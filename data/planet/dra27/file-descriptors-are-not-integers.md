---
title: File descriptors are not integers
description: "There was a flurry of activity on ocaml-multicore/ocaml-uring this month
  leading to a release (ocaml/opam-repository#28604). ocaml-uring provides bindings
  to the Linux\u2019s io_uring, which allows batching various syscalls to the kernel
  for it to execute out-of-order, and in parallel. Its principal use at the moment
  is for the high-performance Linux backend of Eio."
url: https://www.dra27.uk/blog/platform/2025/09/30/file-descriptors-are-not-integers.html
date: 2025-09-30T00:00:00-00:00
preview_image:
authors:
- ""
source:
---

<p>There was a flurry of activity on <a href="https://github.com/ocaml-multicore/ocaml-uring">ocaml-multicore/ocaml-uring</a>
this month leading to a release (<a href="https://github.com/ocaml/opam-repository/pull/28604">ocaml/opam-repository#28604</a>).
ocaml-uring provides bindings to the Linux’s <a href="https://github.com/axboe/liburing">io_uring</a>,
which allows batching various syscalls to the kernel for it to execute
out-of-order, and in parallel. Its principal use at the moment is for the
high-performance Linux backend of <a href="https://github.com/ocaml-multicore/eio">Eio</a>.</p>

<p>Various of the syscalls available in io_uring return Unix file descriptors, and
the design of ocaml-uring as a low-level interface to it leads to some slightly
unfortunate recommendations in its instructions:</p>

<div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">#</span> <span class="k">let</span> <span class="n">fd</span> <span class="o">=</span>
    <span class="k">if</span> <span class="n">result</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="k">then</span> <span class="n">failwith</span> <span class="p">(</span><span class="s2">"Error: "</span> <span class="o">^</span> <span class="n">string_of_int</span> <span class="n">result</span><span class="p">);</span>
    <span class="p">(</span><span class="nn">Obj</span><span class="p">.</span><span class="n">magic</span> <span class="n">result</span> <span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span><span class="p">);;</span>
<span class="k">val</span> <span class="n">fd</span> <span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span> <span class="o">=</span> <span class="o">&lt;</span><span class="n">abstr</span><span class="o">&gt;</span>
</code></pre></div></div>

<p>Anil was wondering if we could dust off some of the code in an old failed OCaml
pull request of mine (<a href="https://github.com/ocaml/ocaml/pull/1990">ocaml/ocaml#1990</a>)
to get rid of some of the magic. Pulling out the C code from that PR wasn’t
entirely mechanical, and Anil wondered if we might put it in a library. As it
happened, there’d been some discussions about the PR at developer meetings, and
there was a consensus that we ought to have an official C API for converting a
C file descriptor (i.e. an <code class="language-plaintext highlighter-rouge">int</code>) to an OCaml <code class="language-plaintext highlighter-rouge">Unix.file_descr</code>. I wasn’t that
keen on producing an external library without sorting out the compiler’s library
at the same time, so I figured I’d dust that change off and see where it went.</p>

<p>That original PR attempted to add primitives to the Unix module to allow OCaml
code to convert an <em>OCaml</em> <code class="language-plaintext highlighter-rouge">int</code> to <code class="language-plaintext highlighter-rouge">Unix.file_descr</code>, so the ocaml-uring
example would instead just be <code class="language-plaintext highlighter-rouge">Unix.descr_of_fd result</code> with no <code class="language-plaintext highlighter-rouge">Obj.magic</code>.
However, that approach had an absolute veto from <a href="https://github.com/ocaml/ocaml/pull/1990#issuecomment-413464113">Xavier</a>:</p>

<p><em>“File descriptors are not integers”</em></p>

<p>I started off on this little rabbit-hole of changes agreeing philosophically,
but not really agreeing, but ended up in total agreement, and ending with the
feeling that the fact they are integers - and that OCaml treats integers
specially - encourages possibly poorer library design.</p>

<p>On Unix, a <code class="language-plaintext highlighter-rouge">Unix.file_descr</code> is just the OCaml representation of the C <code class="language-plaintext highlighter-rouge">int</code>,
but it’s absolutely not that on Windows, where the implementation is much more
complicated. So the <code class="language-plaintext highlighter-rouge">Obj.magic</code> “trick” is a route to a segfault on Windows.
That’s obviously not important for a Linux-only library like ocaml-uring, but
<code class="language-plaintext highlighter-rouge">Obj.magic</code> is a wart. The Windows complexity exists because we have both CRT
file descriptors (which are just C <code class="language-plaintext highlighter-rouge">int</code>s, the same as on Unix) and also OS
file handles (which are Win32 <code class="language-plaintext highlighter-rouge">HANDLE</code>s - a pointer). There’s some added
book-keeping complexity needed, but that’s not important right now. The key
thing is that we have a notion of an “Operating System” file descriptor and a
“C Runtime Library” file descriptor. On Unix, they happen to be the same thing;
on Windows, they’re not. On Windows, given one, it is always possible to obtain
the other (the functions <a href="https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/get-osfhandle"><code class="language-plaintext highlighter-rouge">_get_osfhandle</code></a>
and <a href="https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/open-osfhandle"><code class="language-plaintext highlighter-rouge">_open_osfhandle</code></a>
are provided for this.</p>

<p>That gives a straightforward portable C API: <code class="language-plaintext highlighter-rouge">caml_unix_file_descr_of_os</code> which
takes an <code class="language-plaintext highlighter-rouge">int</code> (on Unix) or a <code class="language-plaintext highlighter-rouge">HANDLE</code> (on Windows) and returns an OCaml
<code class="language-plaintext highlighter-rouge">Unix.file_descr</code> representing it. It’s similarly straightforward to provide
<code class="language-plaintext highlighter-rouge">caml_unix_file_descr_of_fd</code> and <code class="language-plaintext highlighter-rouge">caml_unix_fd_of_file_descr</code> for getting the
CRT file descriptor on both, and we then have the portable primitives required
(<code class="language-plaintext highlighter-rouge">caml_unix_os_of_file_descr</code> is not necessary, but the reasons are left as an
exercise for the avid portable C stub code author).</p>

<p>To this, I then added a quick commit to solve another issue in this area, which
is tracked in <a href="https://github.com/ocaml/ocaml/issues/9052">ocaml/ocaml#9052</a>.
While there’s the veto on <em>converting</em> a <code class="language-plaintext highlighter-rouge">Unix.file_descr</code> to an <code class="language-plaintext highlighter-rouge">int</code>, being
able to debug the values in logs and so forth is useful.</p>

<p>At this point, the scab of this old PR well and truly picked, I paused and
thought a bit about the original problem I’d been trying to solve in <a href="https://github.com/ocaml/ocaml/pull/1990">#1990</a>,
and which remained not-entirely-satisfactorily solved. It’s also described in
<a href="https://github.com/ocaml/ocaml/issues/6948">ocaml/ocaml#6948</a>, and while
looking through it, I realised <a href="https://www.dra27.uk/blog/platform/2025/04/03/cloexec.html">some of my previous work</a>
was related to this. In particular, that passing specific file descriptors from
one process to another is <em>not</em> a Unix-specific operation, and you can do it on
Windows as well, it’s just less common (i.e. you can start a Windows process
with file descriptor 3 connected to a control channel if you want, just as you
can on Unix).</p>

<p>Which got me thinking some more about having an OCaml function for this, and
about “file descriptors are not integers”, and I realised that while this is
<em>mostly</em> true, it’s not <em>always</em> true. For a start, there are three well-known
values, 0 for “input”, 1 for “output”, and 2 for “logging”, known as the
“Standard Input, Output and Error” handles. These are exposed in the Unix
library as <code class="language-plaintext highlighter-rouge">Unix.stdin</code>, <code class="language-plaintext highlighter-rouge">Unix.stdout</code> and <code class="language-plaintext highlighter-rouge">Unix.stderr</code> (and in the Standard
Library itself, for the channels API), and you can also specify them when
spawning processes. Stepping back, let’s consider if instead of treating file
descriptors as <code class="language-plaintext highlighter-rouge">int</code>, <a href="http://cs.bell-labs.co/who/ken/">Thompson</a> and <a href="https://9p.io/who/dmr/index.html">Ritchie</a>
had instead used <a href="https://en.wikipedia.org/wiki/Opaque_pointer#C">opaque pointers</a><sup role="doc-noteref"><a href="https://www.dra27.uk/feed.xml#fn:note" class="footnote" rel="footnote">1</a></sup>
for <code class="language-plaintext highlighter-rouge">open</code>, <code class="language-plaintext highlighter-rouge">read</code>, and so forth in the first version of Unix.</p>

<p>In this hypothetical scenario, virtually all C programs would be just fine:
<code class="language-plaintext highlighter-rouge">STDIN_FILENO</code> and so forth would still be there, and virtually all C code using
file descriptors doesn’t actually care about the precise value. However, the
implementation would face the same issue as we have in OCaml when trying to
spawn other processes if we needed to <em>pass</em> file descriptors to a process. At
that point, it dawned on me: “file descriptors are <strong>indeed</strong> not integers<strong>,
but at program startup, there is a partial map from integers to file
descriptors</strong>”. Most processes are created with at least <code class="language-plaintext highlighter-rouge">0</code>, <code class="language-plaintext highlighter-rouge">1</code> and <code class="language-plaintext highlighter-rouge">2</code> added
to that map, and they can then be retrieved with <code class="language-plaintext highlighter-rouge">Unix.stdin</code>/<code class="language-plaintext highlighter-rouge">STDIN_FILENO</code>,
etc. Given that Windows <em>does</em> support the notion of passing additional file
descriptors, that means that this operation <strong>is portable</strong>, and the inability
to pass <em>abstract</em> file descriptors <strong>using</strong> <em>integers</em> seems like a gap. This
fits with a TODO item I noted <a href="https://www.dra27.uk/blog/platform/2025/04/03/cloexec.html">in April</a>:
once <code class="language-plaintext highlighter-rouge">Unix.create_process</code> et al are correctly <em>inheriting</em> file descriptors on
Windows, it makes even more sense to be able to do this too.</p>

<p><a href="https://github.com/ocaml/ocaml/issues/6948">#6948</a> proposed adding a
<code class="language-plaintext highlighter-rouge">file_descr list</code> argument to functions like <code class="language-plaintext highlighter-rouge">create_process</code>, but that’s the
wrong API here. It <em>assumes</em> that each descriptor is being passed using its
current file descriptor number, which is wrong for two reasons. Firstly, it
breaks the abstraction (we have just treated a <code class="language-plaintext highlighter-rouge">Unix.file_descr</code> as an <code class="language-plaintext highlighter-rouge">int</code>).
Secondly, as noted in the C API above, if the <code class="language-plaintext highlighter-rouge">Unix.file_descr</code> was created from
an OS file descriptor on Windows, it doesn’t necessarily <em>have</em> a CRT descriptor
associated with it. The fix is easy: pass a <code class="language-plaintext highlighter-rouge">(int * file_descr) list</code>. We
preserve the abstraction, and specify the mapping.</p>

<p>With this extra list, when we spawn a process, we now have the ability to pass,
say, the input portion of a pipe (created with <code class="language-plaintext highlighter-rouge">Unix.pipe</code>) as file
descriptor 3. How can we pick that up on the OCaml side? As it happens, on
Windows we really could trivially enumerate them, and so have
<code class="language-plaintext highlighter-rouge">Unix.startup_file_descrs : (int * file_descr) list</code> or some such, but there’s
no way to do that portably on Unix. For this specific case, there is a concrete
reason to have a function <code class="language-plaintext highlighter-rouge">int -&gt; Unix.file_descr</code> (indeed, this is exactly what
the Unix library does internally to create <code class="language-plaintext highlighter-rouge">stdin</code>, <code class="language-plaintext highlighter-rouge">stdout</code> and <code class="language-plaintext highlighter-rouge">stderr</code>).
However, there is <em>no case</em> for a function <code class="language-plaintext highlighter-rouge">Unix.file_descr -&gt; int</code>.</p>

<p>What to call this function, therefore? Having <code class="language-plaintext highlighter-rouge">file_descr_of_fd</code> (as in C) but
not <code class="language-plaintext highlighter-rouge">fd_of_file_descr</code> seems very strange. But what does the type tell us? We’re
taking an <code class="language-plaintext highlighter-rouge">int</code> - a <em>low-level</em> descriptor and returning a <code class="language-plaintext highlighter-rouge">file_descr</code> - a
<em>high-level</em> descriptor. C already has two levels - the stream API (“<code class="language-plaintext highlighter-rouge">FILE *</code>”)
and provides a function to go from the low level to the stream API. It’s
<a href="https://pubs.opengroup.org/onlinepubs/9799919799/functions/fdopen.html"><code class="language-plaintext highlighter-rouge">fdopen</code></a>.
The C streams API isn’t really about abstraction, so C also has a function to go
the other way, <a href="https://pubs.opengroup.org/onlinepubs/9799919799/functions/fileno.html"><code class="language-plaintext highlighter-rouge">fileno</code></a>,
but the nice property is that <code class="language-plaintext highlighter-rouge">fdopen</code> and <code class="language-plaintext highlighter-rouge">fileno</code> do not sound like reverse
operations.</p>

<p>Fixing <code class="language-plaintext highlighter-rouge">Unix.create_process</code> on Windows is definitely a job for another day, but
adding <code class="language-plaintext highlighter-rouge">Unix.fdopen</code> is easy, so I did that. The three commits are sat in
<a href="https://github.com/dra27/ocaml/commits/fdopen">dra27/ocaml#fdopen</a>. Next up,
applying all this to ocaml-uring to get rid of the magic…</p>

<div class="footnotes" role="doc-endnotes">
  <ol>
    <li role="doc-endnote">
      <p>I couldn’t find a precise reference for exactly when in the early 1970s this became possible, relative to the first releases of Unix!&nbsp;<a href="https://www.dra27.uk/feed.xml#fnref:note" class="reversefootnote" role="doc-backlink">↩</a></p>
    </li>
  </ol>
</div>
