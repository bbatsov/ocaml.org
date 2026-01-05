---
title: Testing a priority queue with Monolith
description:
url: https://cambium.inria.fr/blog/testing-a-priority-queue-with-Monolith
date: 2025-11-19T08:00:00-00:00
preview_image:
authors:
- GaGallium
source:
---



  <p>A priority queue is a data structure whose specification is
non-deterministic: indeed, if a priority queue contains several
key-value pairs whose key is minimal, then any such pair can be legally
returned by <code>pop</code>. In this blog post, I describe how to test
an OCaml implementation of a priority queue using Monolith.</p>


  


<p>Suppose that we have written (or someone has given us) an
implementation of (immutable) priority queues. The signature of this
module might look like this (<a href="https://github.com/fpottier/bitsets/blob/main/src/LeftistHeap.mli">LeftistHeap.mli</a>):</p>
<div class="sourceCode"><pre class="sourceCode ocaml"><code class="sourceCode ocaml"><span><a href="http://gallium.inria.fr/blog/index.rss#cb1-1" aria-hidden="true" tabindex="-1"></a><span class="kw">module</span> Make (Key : <span class="kw">sig</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-2" aria-hidden="true" tabindex="-1"></a>  <span class="kw">type</span> t</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-3" aria-hidden="true" tabindex="-1"></a>  <span class="kw">val</span> <span class="dt">compare</span>: t -&gt; t -&gt; <span class="dt">int</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-4" aria-hidden="true" tabindex="-1"></a><span class="kw">end</span>) : <span class="kw">sig</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-5" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-6" aria-hidden="true" tabindex="-1"></a>  <span class="co">(**A key. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-7" aria-hidden="true" tabindex="-1"></a>  <span class="kw">type</span> key = Key.t</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-8" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-9" aria-hidden="true" tabindex="-1"></a>  <span class="co">(**An immutable priority queue. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-10" aria-hidden="true" tabindex="-1"></a>  <span class="kw">type</span> 'a t</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-11" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-12" aria-hidden="true" tabindex="-1"></a>  <span class="co">(**</span><span class="ot">[</span>empty<span class="ot">]</span><span class="co"> is the empty queue. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-13" aria-hidden="true" tabindex="-1"></a>  <span class="kw">val</span> empty : 'a t</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-14" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-15" aria-hidden="true" tabindex="-1"></a>  <span class="co">(**</span><span class="ot">[</span>singleton k v<span class="ot">]</span><span class="co"> is a singleton queue containing the key-value</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-16" aria-hidden="true" tabindex="-1"></a><span class="co">     pair </span><span class="ot">[</span>(k, v)<span class="ot">]</span><span class="co">. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-17" aria-hidden="true" tabindex="-1"></a>  <span class="kw">val</span> singleton : key -&gt; 'a -&gt; 'a t</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-18" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-19" aria-hidden="true" tabindex="-1"></a>  <span class="co">(**</span><span class="ot">[</span>merge q1 q2<span class="ot">]</span><span class="co"> merges the queues </span><span class="ot">[</span>q1<span class="ot">]</span><span class="co"> and </span><span class="ot">[</span>q2<span class="ot">]</span><span class="co">. The result is a</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-20" aria-hidden="true" tabindex="-1"></a><span class="co">     new queue. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-21" aria-hidden="true" tabindex="-1"></a>  <span class="kw">val</span> merge : 'a t -&gt; 'a t -&gt; 'a t</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-22" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-23" aria-hidden="true" tabindex="-1"></a>  <span class="co">(**</span><span class="ot">[</span>pop q<span class="ot">]</span><span class="co"> extracts a key-value pair whose key is minimal out of the queue</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-24" aria-hidden="true" tabindex="-1"></a><span class="co">     </span><span class="ot">[</span>q<span class="ot">]</span><span class="co">. </span><span class="ot">[</span><span class="dt">None</span><span class="ot">]</span><span class="co"> is returned only in the case where </span><span class="ot">[</span>q<span class="ot">]</span><span class="co"> is empty. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-25" aria-hidden="true" tabindex="-1"></a>  <span class="kw">val</span> pop: 'a t -&gt; ((key * 'a) * 'a t) <span class="dt">option</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-26" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb1-27" aria-hidden="true" tabindex="-1"></a><span class="kw">end</span></span></code></pre></div>
<p>The implementation of this module might be based on Chris Okasaki’s
leftist heaps, a simple and beautiful data structure (<a href="https://github.com/fpottier/bitsets/blob/main/src/LeftistHeap.ml">LeftistHeap.ml</a>).
But this does not matter. Let’s see how to test this implementation as a
black box using <a href="https://gitlab.inria.fr/fpottier/monolith/">Monolith</a> (<a href="https://cambium.inria.fr/~fpottier/publis/pottier-monolith-2021.pdf">paper</a>).</p>
<p>Monolith expects us to provide a reference implementation of a
priority queue. This reference implementation does not need to be very
efficient; what matters is that it should be simple and correct. The
simplest possible approach is to use an unsorted list of key-value
pairs, along the following lines (<a href="https://github.com/fpottier/bitsets/blob/8fc708ffdf095bec0a8017dead4bd56fed13b204/test/LeftistHeap/reference.ml#L46">reference.ml</a>)
:</p>
<div class="sourceCode"><pre class="sourceCode ocaml"><code class="sourceCode ocaml"><span><a href="http://gallium.inria.fr/blog/index.rss#cb2-1" aria-hidden="true" tabindex="-1"></a>  <span class="kw">type</span> t =</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-2" aria-hidden="true" tabindex="-1"></a>    (Key.t * Val.t) <span class="dt">list</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-3" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-4" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> empty : t =</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-5" aria-hidden="true" tabindex="-1"></a>    []</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-6" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-7" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> singleton k v : t =</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-8" aria-hidden="true" tabindex="-1"></a>    [(k, v)]</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-9" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-10" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> merge q1 q2 : t =</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-11" aria-hidden="true" tabindex="-1"></a>    q1 @ q2</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-12" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb2-13" aria-hidden="true" tabindex="-1"></a>  <span class="co">(* missing [pop], for now *)</span></span></code></pre></div>
<p>Here comes the interesting part. <code>pop</code> is a
non-deterministic operation: if a priority queue contains several
minimal key-value pairs, then any of them can be legally returned by
<code>pop</code>.</p>
<p>In such a situation, the candidate implementation of <code>pop</code>
makes a choice. It would not make sense for the reference implementation
of <code>pop</code> to also make a choice: because the two choices might
be different, the candidate and the reference could become out of
sync.</p>
<p>Instead, we want the reference implementation of <code>pop</code> to
check that the choice made by the candidate implementation is legal and
to obey this choice, that is, to make the same choice.</p>
<p>Therefore, the reference implementation of <code>pop</code> cannot
have type <code>t -&gt; ((Key.t * Val.t) * t) option</code>, as one
might expect. Instead, it must take two arguments, namely the queue on
the reference side and the result returned by <code>pop</code> on the
candidate side. It is expected to return a diagnostic, that is, an
indication of whether this candidate result is valid or invalid. The
type <code>diagnostic</code> is defined by Monolith (<a href="https://cambium.inria.fr/~fpottier/monolith/doc/monolith/Monolith/#spec:fun:nondet">documentation</a>);
its constructors are <code>Valid</code> and <code>Invalid</code>.</p>
<p>The reference implementation of <code>pop</code> can be written as
follows (<a href="https://github.com/fpottier/bitsets/blob/f049d28fea7111a0b812ab6abd9a82abd81f153a/test/LeftistHeap/reference.ml#L129">link</a>):</p>
<div class="sourceCode"><pre class="sourceCode ocaml"><code class="sourceCode ocaml"><span><a href="http://gallium.inria.fr/blog/index.rss#cb3-1" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> pop (q : t) (<span class="dt">result</span> : ((Key.t * Val.t) * _) <span class="dt">option</span>)</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-2" aria-hidden="true" tabindex="-1"></a>  : (((Key.t * Val.t) * t) <span class="dt">option</span>) diagnostic =</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-3" aria-hidden="true" tabindex="-1"></a>    <span class="kw">match</span> <span class="dt">result</span> <span class="kw">with</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-4" aria-hidden="true" tabindex="-1"></a>    | <span class="dt">Some</span> (kv, _cq) -&gt;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-5" aria-hidden="true" tabindex="-1"></a>        <span class="co">(* The candidate has extracted the key-value pair [kv] and has returned</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-6" aria-hidden="true" tabindex="-1"></a><span class="co">           a candidate queue [_cq] that we cannot inspect, as it is an abstract</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-7" aria-hidden="true" tabindex="-1"></a><span class="co">           data structure. Fortunately, there is no need to inspect it. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-8" aria-hidden="true" tabindex="-1"></a>        <span class="co">(* Check that the key-value pair [kv] chosen by the candidate is a</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-9" aria-hidden="true" tabindex="-1"></a><span class="co">           minimal element of the queue [q], and return [q] minus [kv]. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-10" aria-hidden="true" tabindex="-1"></a>        handle @@ <span class="kw">fun</span> () -&gt;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-11" aria-hidden="true" tabindex="-1"></a>        <span class="kw">let</span> q = remove_minimal kv q <span class="kw">in</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-12" aria-hidden="true" tabindex="-1"></a>        valid (<span class="dt">Some</span> (kv, q))</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-13" aria-hidden="true" tabindex="-1"></a>    | <span class="dt">None</span> -&gt;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-14" aria-hidden="true" tabindex="-1"></a>        <span class="co">(* The candidate has returned [None]. Check that the reference queue [q]</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-15" aria-hidden="true" tabindex="-1"></a><span class="co">           is empty; if it isn't, fail. *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-16" aria-hidden="true" tabindex="-1"></a>        <span class="kw">if</span> q = [] <span class="kw">then</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-17" aria-hidden="true" tabindex="-1"></a>          valid <span class="dt">None</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-18" aria-hidden="true" tabindex="-1"></a>        <span class="kw">else</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-19" aria-hidden="true" tabindex="-1"></a>          invalid @@ <span class="kw">fun</span> _doc -&gt;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb3-20" aria-hidden="true" tabindex="-1"></a>          <span class="dt">format</span> <span class="st">"(* candidate returns None, yet queue is nonempty *)"</span></span></code></pre></div>
<p>The auxiliary function <code>remove_minimal kv kvs</code> (<a href="https://github.com/fpottier/bitsets/blob/f049d28fea7111a0b812ab6abd9a82abd81f153a/test/LeftistHeap/reference.ml#L93">link</a>)
checks that the key-value pair <code>kv</code> is a minimal element of
the list <code>kvs</code> and returns this list deprived of this
element. It fails, by raising an exception, if <code>kv</code> is not in
the list or not minimal.</p>
<p>The auxiliary function <code>handle</code> (<a href="https://github.com/fpottier/bitsets/blob/f049d28fea7111a0b812ab6abd9a82abd81f153a/test/LeftistHeap/reference.ml#L106">link</a>)
handles the exceptions raised by <code>remove_minimal</code> and returns
an <code>invalid</code> diagnostic in these cases.</p>
<p>The reference implementation is now complete. There remains to tell
Monolith about the operations that we want to test. For each operation,
we must provide a specification (that is, roughly, a type), a reference
implementation, and a candidate implementation. This is done as follows
(<a href="https://github.com/fpottier/bitsets/blob/main/test/LeftistHeap/test.ml">test.ml</a>):</p>
<div class="sourceCode"><pre class="sourceCode ocaml"><code class="sourceCode ocaml"><span><a href="http://gallium.inria.fr/blog/index.rss#cb4-1" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> spec = t <span class="kw">in</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-2" aria-hidden="true" tabindex="-1"></a>  declare <span class="st">"empty"</span> spec R.empty C.empty;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-3" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-4" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> spec = value ^&gt; key ^&gt; t <span class="kw">in</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-5" aria-hidden="true" tabindex="-1"></a>  declare <span class="st">"singleton"</span> spec R.singleton C.singleton;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-6" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-7" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> spec = t ^&gt; t ^&gt; t <span class="kw">in</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-8" aria-hidden="true" tabindex="-1"></a>  declare <span class="st">"merge"</span> spec R.merge C.merge;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-9" aria-hidden="true" tabindex="-1"></a></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-10" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> spec = t ^&gt; nondet (<span class="dt">option</span> ((key *** value) *** t)) <span class="kw">in</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb4-11" aria-hidden="true" tabindex="-1"></a>  declare <span class="st">"pop"</span> spec R.pop C.pop;</span></code></pre></div>
<p>In the case of <code>pop</code>, we have used the combinator
<code>nondet</code> (<a href="https://cambium.inria.fr/~fpottier/monolith/doc/monolith/Monolith/#spec:fun:nondet">documentation</a>)
to tell Monolith that <code>pop</code> is a non-deterministic operation,
whose reference implementation is written in the unusual style that we
have shown above.</p>
<p>There remains to run the test, which runs forever and prints the
problematic scenarios that it detects.</p>
<p>It is worth noting that the result of <code>pop</code> is a composite
value: it is partly concrete, partly abstract, as it is an (optional)
pair of a key-value pair (concrete data that can be observed) and a
priority queue (abstract data that cannot be directly observed). This
does not create any problem: Monolith is able to automatically decompose
this composite value and submit the priority queue component to further
testing.</p>
<p>To illustrate this, let us intentionally introduce the following bug
in the candidate implementation:</p>
<div class="sourceCode"><pre class="sourceCode ocaml"><code class="sourceCode ocaml"><span><a href="http://gallium.inria.fr/blog/index.rss#cb5-1" aria-hidden="true" tabindex="-1"></a>  <span class="kw">let</span> pop q =</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb5-2" aria-hidden="true" tabindex="-1"></a>    <span class="kw">match</span> pop q <span class="kw">with</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb5-3" aria-hidden="true" tabindex="-1"></a>    | <span class="dt">Some</span> (kv, q') -&gt;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb5-4" aria-hidden="true" tabindex="-1"></a>        <span class="dt">ignore</span> q';</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb5-5" aria-hidden="true" tabindex="-1"></a>        <span class="dt">Some</span> (kv, q) <span class="co">(* return the original queue, unchanged *)</span></span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb5-6" aria-hidden="true" tabindex="-1"></a>    | <span class="dt">None</span> -&gt;</span>
<span><a href="http://gallium.inria.fr/blog/index.rss#cb5-7" aria-hidden="true" tabindex="-1"></a>        <span class="dt">None</span></span></code></pre></div>
<p>Then, the test immediately fails and prints the following
scenario:</p>
<pre><code>(* @03: Failure in an observation: candidate and reference disagree. *)
          open Bitsets.LeftistHeap.Make(Int);;
          #require "monolith";;
          module Sup = Monolith.Support;;
(* @01 *) let x0 = singleton 6 11;;
(* @02 *) let (Some ((_, _), x1)) = pop x0;;
(* @03 *) let observed = pop x1;;
          (* candidate returns (6, 11), which does not exist *)</code></pre>
<p>The priority queue returned by the first <code>pop</code> operation
is extracted, named <code>x1</code>, and submitted to a second
<code>pop</code> operation, where an observable problem appears.</p>


  
