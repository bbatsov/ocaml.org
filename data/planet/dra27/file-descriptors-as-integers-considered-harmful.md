---
title: File descriptors as integers considered harmful
description: "Continuing the previous foray into file descriptors, and trying to remove
  Obj.magic both from ocaml-uring\u2019s code itself, and from the recommended way
  for using it."
url: https://www.dra27.uk/blog/platform/2025/10/01/int-file-descriptors-considered-harmful.html
date: 2025-10-01T00:00:00-00:00
preview_image:
authors:
- ""
source:
---

<p>Continuing <a href="https://www.dra27.uk/blog/platform/2025/09/30/file-descriptors-are-not-integers.html">the previous foray into file descriptors</a>,
and trying to remove <code class="language-plaintext highlighter-rouge">Obj.magic</code> both from ocaml-uring’s code itself, and from
the recommended way for using it.</p>

<p>I expected this to be a fairly simple application of the C API features added in
<a href="https://github.com/dra27/ocaml/commits/file_descr-interop">dra27/ocaml#file_descr-interop</a>.
It’s not quite what happened, though - as I don’t actually think that any of the
changes I propose for the Unix library in OCaml are needed for ocaml-uring. What
piqued my interest in this rather deep rabbit-hole was that what I would contend
are slightly poor design decisions stem from the corrupting notion in C that
file descriptors are integers - i.e. because file descriptors in C <em>are</em>
integers, it becomes dangerously tempting to keep them as that in OCaml, because
OCaml <a href="https://ocaml.org/manual/5.3/intfc.html#s:c-value">treats integers specially</a>.</p>

<p><a href="https://ocaml.org/p/uring/latest/doc/index.html">ocaml-uring’s documentation</a>
states that it “aims to provide a thin type-safe layer for use in higher-level
interfaces”. I think adopting Xavier’s <em>veto</em> on manipulating file descriptors
as integers <em>in OCaml</em> (i.e. <em>never</em> doing it) leads to greater type safety,
even in thin bindings, without compromising performance. Let’s see! 🤓</p>

<p>The <a href="https://github.com/ocaml-multicore/ocaml-uring">sources for ocaml-uring</a>
use <code class="language-plaintext highlighter-rouge">Obj.magic</code> both in the library itself in one place, and at various points
in both the tests and the documentation, exclusively for duping the type system
into believing an <code class="language-plaintext highlighter-rouge">int</code> is a <code class="language-plaintext highlighter-rouge">Unix.file_descr</code>.</p>

<p>Consider <a href="https://man7.org/linux/man-pages/man2/mkdirat.2.html">mkdirat(2)</a>.
This is part of a family of *at syscalls which accept a file descriptor as an
argument and which will use the directory name of the file referred to by that
file descriptor as the base for a relative path. There is a special value
<code class="language-plaintext highlighter-rouge">AT_FDCWD</code> which can be passed instead of an open file descriptor (it instructs
<code class="language-plaintext highlighter-rouge">mkdirat</code> instead to interpret paths relative to the current working directory),
and it’s the representation of this <code class="language-plaintext highlighter-rouge">AT_FDCWD</code> which causes the <code class="language-plaintext highlighter-rouge">Obj.magic</code>.
When used in C, <code class="language-plaintext highlighter-rouge">AT_FDCWD</code> is an illegal file descriptor value (it’s -100, as it
happens). Continuing with <code class="language-plaintext highlighter-rouge">mkdirat</code> as an example, we have this C declaration:</p>

<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">void</span> <span class="nf">io_uring_prep_mkdirat</span><span class="p">(</span><span class="k">struct</span> <span class="n">io_uring_sqe</span> <span class="o">*</span><span class="n">sqe</span><span class="p">,</span>
                           <span class="kt">int</span> <span class="n">dirfd</span><span class="p">,</span> <span class="k">const</span> <span class="kt">char</span> <span class="o">*</span><span class="n">path</span><span class="p">,</span> <span class="n">mode_t</span> <span class="n">mode</span><span class="p">);</span>
</code></pre></div></div>

<p>and <a href="https://github.com/ocaml-multicore/ocaml-uring/blob/main/lib/uring/uring.ml#L341">this</a>
corresponding primitive in ocaml-uring:</p>
<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">external</span> <span class="n">submit_mkdirat</span> <span class="o">:</span> <span class="n">t</span> <span class="o">-&gt;</span> <span class="n">id</span>
                          <span class="o">-&gt;</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span> <span class="o">-&gt;</span> <span class="nn">Sketch</span><span class="p">.</span><span class="n">ptr</span> <span class="o">-&gt;</span> <span class="kt">int</span>
                          <span class="o">-&gt;</span> <span class="kt">bool</span> <span class="o">=</span> <span class="s2">"ocaml_uring_submit_mkdirat"</span> <span class="p">[</span><span class="o">@@</span><span class="n">noalloc</span><span class="p">]</span>
</code></pre></div></div>

<p>There’s a close correspondence between the two - the OCaml <code class="language-plaintext highlighter-rouge">t</code> gets passed to
<a href="https://man7.org/linux/man-pages/man3/io_uring_get_sqe.3.html"><code class="language-plaintext highlighter-rouge">io_uring_get_sqe</code></a>
which gives us the first parameter for the prep call. The <code class="language-plaintext highlighter-rouge">id</code> is separately
passed to <a href="https://man7.org/linux/man-pages/man3/io_uring_sqe_set_data.3.html"><code class="language-plaintext highlighter-rouge">io_uring_set_sqe_data</code></a>
(that’s part of a mechanism in liburing to be able to identify which syscalls
have completed - more on that later). The remaining arguments in this primitive
(the <code class="language-plaintext highlighter-rouge">Unix.file_descr</code>, the <code class="language-plaintext highlighter-rouge">Sketch.ptr</code> and the <code class="language-plaintext highlighter-rouge">int</code>) correspond to the
<code class="language-plaintext highlighter-rouge">dirfd</code>, <code class="language-plaintext highlighter-rouge">path</code> and <code class="language-plaintext highlighter-rouge">mode</code> arguments to the syscall (the liveness of buffers is
complicated here - the <code class="language-plaintext highlighter-rouge">Sketch.ptr</code> type is essentially allowing a string to be
passed from OCaml to C without subsequently having to worry about it being moved
by the GC).</p>

<p>It looks like a good thin layer over the syscall. Every one of these prep calls
will necessarily have a <code class="language-plaintext highlighter-rouge">io_uring_set_sqe_data</code> call, and it’s always good to
minimise the number of crossings between C and OCaml when writing C bindings.</p>

<p>It naturally becomes tempting to have a <code class="language-plaintext highlighter-rouge">Unix.file_descr</code> value for <code class="language-plaintext highlighter-rouge">AT_FDCWD</code>,
and that’s indeed <a href="https://github.com/ocaml-multicore/ocaml-uring/blob/main/lib/uring/uring.ml#L468">the implementation</a>:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">let</span> <span class="n">at_fdcwd</span> <span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span> <span class="o">=</span> <span class="nn">Obj</span><span class="p">.</span><span class="n">magic</span> <span class="nn">Config</span><span class="p">.</span><span class="n">at_fdcwd</span>
</code></pre></div></div>

<p>However, there really is no need for this, and if we step back from the fact
it’s an <code class="language-plaintext highlighter-rouge">int</code>, and instead consider what <code class="language-plaintext highlighter-rouge">AT_FDCWD</code> is <em>modelling</em>, a more
obvious solution emerges which doesn’t require magic. <code class="language-plaintext highlighter-rouge">AT_FDCWD</code> is essentially
be used to mean “no file descriptor” (and there’s then an interpretation for
what that means <code class="language-plaintext highlighter-rouge">mkdirat</code> should do). i.e. <code class="language-plaintext highlighter-rouge">AT_FDCWD</code> means <code class="language-plaintext highlighter-rouge">None</code>, which is
then being modelled by -100. An actual file descriptor is <code class="language-plaintext highlighter-rouge">Some fd</code>, which is
being modelled by its number (which can never be -100). In other words, we could
pass a <code class="language-plaintext highlighter-rouge">Unix.file_descr option</code> to the C stub, instead of a <code class="language-plaintext highlighter-rouge">Unix.file_descr</code>.
This falls out even more naturally when we look at the actual <a href="https://github.com/ocaml-multicore/ocaml-uring/blob/main/lib/uring/uring.ml#L498-L502"><code class="language-plaintext highlighter-rouge">Uring.mkdirat</code></a>
exposed to the library user:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">let</span> <span class="n">mkdirat</span> <span class="n">t</span> <span class="o">~</span><span class="n">mode</span> <span class="o">?</span><span class="p">(</span><span class="n">fd</span><span class="o">=</span><span class="n">at_fdcwd</span><span class="p">)</span> <span class="n">path</span> <span class="n">user_data</span> <span class="o">=</span>
</code></pre></div></div>

<p>It actually already has a <code class="language-plaintext highlighter-rouge">Unix.file_descr option</code> because it’s using an
optional argument to convey it! It’s quite an <a href="https://github.com/dra27/ocaml-uring/commit/00ca9ecd88216ae4fdd039d92c58619686e4b4af">easy adjustment</a>
just to allow the C stub to resolve this, it has the benefit that the value of
<code class="language-plaintext highlighter-rouge">AT_FDCWD</code> no longer has to be determined from OCaml, and it doesn’t involve any
additional allocations of memory on the OCaml side (because the optional
argument meant there was already one).</p>

<p>So far, so hopefully good. Now on to the one in the README:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">#</span> <span class="k">let</span> <span class="n">fd</span> <span class="o">=</span>
    <span class="k">if</span> <span class="n">result</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="k">then</span> <span class="n">failwith</span> <span class="p">(</span><span class="s2">"Error: "</span> <span class="o">^</span> <span class="n">string_of_int</span> <span class="n">result</span><span class="p">);</span>
    <span class="p">(</span><span class="nn">Obj</span><span class="p">.</span><span class="n">magic</span> <span class="n">result</span> <span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span><span class="p">);;</span>
<span class="k">val</span> <span class="n">fd</span> <span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span> <span class="o">=</span> <span class="o">&lt;</span><span class="n">abstr</span><span class="o">&gt;</span>
</code></pre></div></div>

<p>Well, it wasn’t my intention, but the proposed <code class="language-plaintext highlighter-rouge">Unix.fdopen</code> <a href="https://github.com/dra27/ocaml-uring/commit/6c14ffd68be586e48fa4293df76338db32a28c12">could deal with
that</a>.
Except that this is not “<code class="language-plaintext highlighter-rouge">fdopen</code>” in the spirit it was added for - in fact,
it’s precisely what was <em>not</em> supposed to be done. We’ve got an <code class="language-plaintext highlighter-rouge">int</code> in OCaml
that is <em>actually</em> a file descriptor, and we’re trying to coerce it to be a
<code class="language-plaintext highlighter-rouge">Unix.file_descr</code>. We’ve got a layering violation - we’ve allowed an abstract
value from the world of C that is represented in C as an <code class="language-plaintext highlighter-rouge">int</code> to leak up to the
world of OCaml as an OCaml <code class="language-plaintext highlighter-rouge">int</code>, which should be a number!</p>

<p>Funnily enough, ocaml-uring has another minor layering violation like this in
the translation of Unix error values to <code class="language-plaintext highlighter-rouge">Unix.error</code> values in the
<a href="https://github.com/ocaml-multicore/ocaml-uring/blob/main/lib/uring/uring.ml#L651-L652"><code class="language-plaintext highlighter-rouge">Uring.error_of_errno</code> function</a>:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">let</span> <span class="n">error_of_errno</span> <span class="n">e</span> <span class="o">=</span>
  <span class="nn">Uring</span><span class="p">.</span><span class="n">error_of_errno</span> <span class="p">(</span><span class="n">abs</span> <span class="n">e</span><span class="p">)</span>
</code></pre></div></div>

<p>I didn’t want to abuse my proposed <code class="language-plaintext highlighter-rouge">Unix.fdopen</code> this way, so soon after
conceiving it for a more noble and even slightly-typed purpose, so instead I
opted for bouncing these numbers back to C by instead adding <a href="https://github.com/dra27/ocaml-uring/pull/3/files"><code class="language-plaintext highlighter-rouge">Uring.file_descr_of_result</code></a>:</p>

<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="n">value</span> <span class="nf">ocaml_uring_file_descr_of_result</span><span class="p">(</span><span class="n">value</span> <span class="n">v_result</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">CAMLparam0</span><span class="p">();</span>
  <span class="n">CAMLlocal2</span><span class="p">(</span><span class="n">result</span><span class="p">,</span> <span class="n">val</span><span class="p">);</span>
  <span class="k">if</span> <span class="p">(</span><span class="n">Int_val</span><span class="p">(</span><span class="n">v_result</span><span class="p">)</span> <span class="o">&lt;</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{</span>
    <span class="n">val</span> <span class="o">=</span> <span class="n">unix_error_of_code</span><span class="p">(</span><span class="o">-</span><span class="n">Int_val</span><span class="p">(</span><span class="n">v_result</span><span class="p">));</span>
    <span class="n">result</span> <span class="o">=</span> <span class="n">caml_alloc_small</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
  <span class="p">}</span> <span class="k">else</span> <span class="p">{</span>
    <span class="n">val</span> <span class="o">=</span> <span class="n">caml_unix_file_descr_of_fd</span><span class="p">(</span><span class="n">Int_val</span><span class="p">(</span><span class="n">v_result</span><span class="p">));</span>
    <span class="n">result</span> <span class="o">=</span> <span class="n">caml_alloc_small</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="p">}</span>
  <span class="n">Field</span><span class="p">(</span><span class="n">result</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span> <span class="o">=</span> <span class="n">val</span><span class="p">;</span>
  <span class="n">CAMLreturn</span><span class="p">(</span><span class="n">result</span><span class="p">);</span>
<span class="p">}</span>
</code></pre></div></div>

<p>which handles the pattern in C, where it belongs. The result certainly looks
nicer in some of the test code, for example, the numeric matching in one of the
tests:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">if</span> <span class="n">result</span> <span class="o">&gt;=</span> <span class="mi">0</span> <span class="k">then</span>
  <span class="k">let</span> <span class="n">fd</span> <span class="o">=</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">fdopen</span> <span class="n">result</span> <span class="k">in</span>
  <span class="nn">Unix</span><span class="p">.</span><span class="n">close</span> <span class="n">fd</span>
<span class="k">else</span>
  <span class="k">let</span> <span class="n">error</span> <span class="o">=</span> <span class="nn">Uring</span><span class="p">.</span><span class="n">error_of_errno</span> <span class="n">fd</span> <span class="k">in</span>
  <span class="k">raise</span> <span class="p">(</span><span class="nn">Unix</span><span class="p">.</span><span class="nc">Unix_error</span><span class="p">(</span><span class="n">error</span><span class="o">,</span> <span class="c">(* ... *)</span><span class="p">))</span>
</code></pre></div></div>

<p>becomes a slightly more idiomatic:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">match</span> <span class="nn">Uring</span><span class="p">.</span><span class="n">file_descr_of_result</span> <span class="n">result</span> <span class="k">with</span>
<span class="o">|</span> <span class="nc">Ok</span> <span class="n">fd</span> <span class="o">-&gt;</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">close</span> <span class="n">fd</span>
<span class="o">|</span> <span class="nc">Error</span> <span class="n">error</span> <span class="o">-&gt;</span> <span class="k">raise</span> <span class="p">(</span><span class="nn">Unix</span><span class="p">.</span><span class="nc">Unix_error</span><span class="p">(</span><span class="n">error</span><span class="o">,</span> <span class="c">(* ... *)</span><span class="p">))</span>
</code></pre></div></div>

<p>but there are two things that still feel bad - those <code class="language-plaintext highlighter-rouge">int</code>s are <em>still there</em>
in OCaml! Worse, we’re now incurring an extra C call to process them.</p>

<p>The veto tells us that the processing belongs in C. Is there a way to do this
where we don’t <em>ever</em> have the general <code class="language-plaintext highlighter-rouge">int</code> result exposed in OCaml, but we’re
still a thin wrapper around the API <em>and</em> at no cost?</p>

<p>Not without breaking the API, but I think we can have our API cake and also eat
it in this case. A first stab sits on <a href="https://github.com/dra27/ocaml-uring/pull/2/files">my fork</a>.
The result of an io_uring operation is always a (C) <code class="language-plaintext highlighter-rouge">int</code>. Each of the syscalls
uses one of three encodings of that integer. In all three cases, a negative
number corresponds to an error, where the absolute value of the number is the
Unix error number (from <code class="language-plaintext highlighter-rouge">errno.h</code>). Some functions then just return 0 on
success. The other functions return a number ≥ 0 and for some of those, that
number is a file descriptor.</p>

<p>Now, the choice of encoding is known <em>when we prep the call</em>. So the trick I
tried instead was to encode that in the user-data attached to the ring entry.
The ocaml-uring implementation is using an array index as the user-data, so even
on a 32-bit system we can comfortably steal two bits to encode how to treat the
result. The three wait functions now instead of returning:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">type</span> <span class="n">cqe_option</span> <span class="o">=</span> <span class="n">private</span>
<span class="o">|</span> <span class="nc">Cqe_none</span>
<span class="o">|</span> <span class="nc">Cqe_some</span> <span class="k">of</span> <span class="p">{</span> <span class="n">user_data_id</span> <span class="o">:</span> <span class="n">id</span><span class="p">;</span> <span class="n">res</span><span class="o">:</span> <span class="kt">int</span> <span class="p">}</span>
</code></pre></div></div>

<p>return a slightly richer:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">type</span> <span class="n">cqe_option</span> <span class="o">=</span> <span class="n">private</span>
<span class="o">|</span> <span class="nc">Cqe_none</span>
<span class="o">|</span> <span class="nc">Cqe_unit</span> <span class="k">of</span> <span class="p">{</span> <span class="n">user_data_id</span> <span class="o">:</span> <span class="n">id</span><span class="p">;</span> <span class="n">result</span><span class="o">:</span> <span class="kt">unit</span> <span class="p">}</span>
<span class="o">|</span> <span class="nc">Cqe_int</span> <span class="k">of</span> <span class="p">{</span> <span class="n">user_data_id</span> <span class="o">:</span> <span class="n">id</span><span class="p">;</span> <span class="n">result</span><span class="o">:</span> <span class="kt">int</span> <span class="p">}</span>
<span class="o">|</span> <span class="nc">Cqe_fd</span> <span class="k">of</span> <span class="p">{</span> <span class="n">user_data_id</span> <span class="o">:</span> <span class="n">id</span><span class="p">;</span> <span class="n">result</span><span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span> <span class="p">}</span>
<span class="o">|</span> <span class="nc">Cqe_error</span> <span class="k">of</span> <span class="p">{</span> <span class="n">user_data_id</span> <span class="o">:</span> <span class="n">id</span><span class="p">;</span> <span class="n">result</span><span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">error</span> <span class="p">}</span>
</code></pre></div></div>

<p>A little bit of additional plumbing is needed for cancellation. I opted to have
the C functions themselves return the amended user-data value, as it’s only
needed for cancellation. That keeps the considerations of C in C-land, but it
might be better to push that into the <code class="language-plaintext highlighter-rouge">Heap</code> implementation which backs the IDs
and allow both the job ID and heap pointer to be the same number.</p>

<p>I didn’t push this all the way back to Eio, but I did update all the tests and
documentation, and I have to say the results looked nice to me (and still
low-level) - in particular, the copying tests seem to get a clearer delineation
of error path. I’m not sure if the separation of <code class="language-plaintext highlighter-rouge">unit</code>/<code class="language-plaintext highlighter-rouge">int</code> was strictly
buying much, and given that various of the error codes which may want to come
aren’t Posix and so map to <code class="language-plaintext highlighter-rouge">Unix.UNKNOWNERR</code> (which allocates), it would
possibly be worth looking at a richer error type which could itself be converted
to a <code class="language-plaintext highlighter-rouge">Unix.error</code> expensively, but on-demand. However, what was nice was that
the return from the wait from functions still allocates just one two-field OCaml
word, and the file descriptor - if there is one - incurs no further conversion,
the value comes from C with the correct types already.</p>

<p>Entertainingly, the final version of the patch, while breaking the API, does not
actually need any alterations to OCaml’s Unix library at all.</p>

<p>There’s one last possibly fun OxCaml follow-up to look at. The value which comes
back from the C wait functions is a <code class="language-plaintext highlighter-rouge">cqe_option</code> (see above) whose shape is a
single constant constructor and four 2-field constructors. The first field in
these cases is the io_uring user-data, which we then translate back to the
OCaml value to yield one of these:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">type</span> <span class="k">'</span><span class="n">a</span> <span class="n">completion_option</span> <span class="o">=</span>
  <span class="o">|</span> <span class="nc">None</span>
  <span class="o">|</span> <span class="nc">Unit</span> <span class="k">of</span> <span class="p">{</span> <span class="n">result</span><span class="o">:</span> <span class="kt">unit</span><span class="p">;</span> <span class="n">data</span><span class="o">:</span> <span class="k">'</span><span class="n">a</span> <span class="p">}</span>
  <span class="o">|</span> <span class="nc">Int</span> <span class="k">of</span> <span class="p">{</span> <span class="n">result</span><span class="o">:</span> <span class="kt">int</span><span class="p">;</span> <span class="n">data</span><span class="o">:</span> <span class="k">'</span><span class="n">a</span> <span class="p">}</span>
  <span class="o">|</span> <span class="nc">FD</span> <span class="k">of</span> <span class="p">{</span> <span class="n">result</span><span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span><span class="p">;</span> <span class="n">data</span><span class="o">:</span> <span class="k">'</span><span class="n">a</span> <span class="p">}</span>
  <span class="o">|</span> <span class="nc">Error</span> <span class="k">of</span> <span class="p">{</span> <span class="n">result</span><span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">error</span><span class="p">;</span> <span class="n">data</span><span class="o">:</span> <span class="k">'</span><span class="n">a</span> <span class="p">}</span>
</code></pre></div></div>

<p>Which has the same shape! Now, we could do an evil <code class="language-plaintext highlighter-rouge">Obj.magic</code> trick here to
re-use the block and simply replace the field which contained the io_uring user
data with the OCaml value (we’d need to swap the fields around in the types, but
that’s a minor detail). However, this kind of lying can be dangerous when the
OCaml optimiser comes into play… but can the uniqueness modes be used here to
allow <em>the compiler</em> to determine that the <code class="language-plaintext highlighter-rouge">cqe_option</code> block is now available
and actually compile it down to just that field assignment? We’d then halve the
allocations arising from a ring wait operation 🤔</p>

<p>Anyway, file descriptors are not integers. I’m a new disciple. I think we should
make T-Shirts.</p>
