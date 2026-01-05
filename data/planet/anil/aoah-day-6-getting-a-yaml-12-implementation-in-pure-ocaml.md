---
title: 'AoAH Day 6: Getting a Yaml 1.2 implementation in pure OCaml'
description: Implementing a pure OCaml Yaml 1.2 parser using bytesrw by synthesizing
  from the specification and existing C library behavior, passing thousands of test
  suite cases while being 20% faster than the C-based implementation.
url: https://anil.recoil.org/notes/aoah-2025-6
date: 2025-12-06T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-yamlrw-tests.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>I did the palate cleanser of <a href="https://anil.recoil.org/notes/aoah-2025-5">Bytesrw-eio</a> yesterday for a good reason. Back in 2017, I wrote the <a href="https://github.com/avsm/ocaml-yaml/blob/master/CHANGES.md">OCaml Yaml</a> bindings that a lot of projects use in the OCaml ecosystem, and I'm having trouble maintaining it.</p>
<p>Since Yaml is an monstrously convoluted spec, I opted back then to bind to the <a href="https://github.com/yaml/libyaml">C libyaml</a> using <a href="https://anil.recoil.org/papers/2018-socp-modular-ffi">ocaml-ctypes</a>.  This was a good decision a decade ago, but maintaining this has been a nightmare due to the complexity of vendoring the C library, dealing with security issues there, and exposing a reasonable OCaml interface. The ocaml-yaml implementation also doesn't pass the full Yaml test suite.</p>
<p>And the worst thing is, I <em>cannot</em> find the motivation to figure out how Yaml really works. It's the world's <a href="https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell">worst serialisation format</a>, with lots of corner cases and <a href="https://en.wikipedia.org/wiki/Billion_laughs_attack">memory blowups</a> inherent in how it works. So I decided to dive in and see if I could build a <em>pure OCaml Yaml 1.2</em> implementation using <a href="https://github.com/dbuenzli/bytesrw">bytesrw</a> and the <a href="https://yaml.org/spec/1.2.2/">source spec</a>.</p>
<p>TL;DR: it worked. It actually seems to have come up with a reasonable, <a href="https://tangled.org/anil.recoil.org/ocaml-yamlrw">pure OCaml implementation</a> that I'm now using! It needs more validation and external code review, but this has been on my TODO list for <em>years</em> now.</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>As with previous projects, I carefully set up the source directory using all the previous libraries to act as style guides, along with the source code to the key dependency of bytesrw and the associated <a href="https://anil.recoil.org/notes/aoah-2025-5">bytesrw-eio</a> libraries.
Since the yaml spec is so complex, I <em>also</em> added in the source code to my original <a href="https://github.com/avsm/ocaml-yaml">ocaml-yaml</a> library, which includes the vendored version of the C libyaml.</p>
<p>In a slight twist, I also instructed the agent to look at the Git history for ocaml-yaml, since there have been a decade of bug reports about bad ways of interpreting yaml <a href="https://github.com/avsm/ocaml-yaml/issues?q=is:issue">reported</a>, including <a href="https://github.com/avsm/ocaml-yaml/issues/82">one from Martin Jambon</a> that I haven't gotten around to looking at yet for the main library. There have also been frequent requests for a pure OCaml version to make <a href="https://github.com/avsm/ocaml-yaml/issues/81">cross compilation to iOS</a> easier, as well as compilation on <a href="https://github.com/avsm/ocaml-yaml/issues/78">OpenBSD</a> (failing due to the vendoring of the C library), and of course <a href="https://github.com/avsm/ocaml-yaml/issues/73">C memory leaks</a> and <a href="https://github.com/avsm/ocaml-yaml/issues/72">spec violations</a>.  <strong>All of these</strong> would disappear with a spec-compliant implementation, so I fed the agent these to use as regression examples from user bug reports.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/yaml-test-results.html"> <img src="https://anil.recoil.org/images/aoah-yamlrw-tests.webp" alt="%c" title="It's easy to generate an HTML visualisation of the test suite"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h2>
<p>The key to the development loop succeeding was adding integration to an external test suite. Yaml has the <a href="https://github.com/yaml/yaml-test-suite">yaml-test-suite</a> which has <a href="https://github.com/yaml/yaml-test-suite/tree/data">thousands</a> of little examples of good and bad specs, along with the expected outputs in both JSON and Yaml.  For example, <a href="https://github.com/yaml/yaml-test-suite/tree/data/36F6">test 36F6</a> takes this input yaml:</p>
<pre><code>plain: a
 b

 c
</code></pre>
<p>and expects this error "Multiline plain scalar with empty line" and the following parser events in a custom DSL exposed by the test suite:</p>
<pre><code>+MAP
=VAL :plain
=VAL :a b\nc
-MAP
-DOC
-STR
</code></pre>
<p>So I instructed the agent to also build up a <a href="https://tangled.org/anil.recoil.org/ocaml-yamlrw/blob/main/tests/test_suite_lib/tree_format.ml">test suite DSL</a> that could output in a format compatible with the test suite. A simple <a href="https://tangled.org/anil.recoil.org/ocaml-yamlrw/blob/main/tests/test_suite_lib/test_suite_loader_generic.ml">custom loader</a> and <a href="https://tangled.org/anil.recoil.org/ocaml-yamlrw/blob/main/tests/test_suite_lib/json_format.ml">JSON converter</a> then output in the exact format required by the checked in test suite files.</p>
<p>I also added in some of the pathological tests from my original OCaml yaml, including an implementation of the <a href="https://github.com/avsm/ocaml-yaml/blob/master/tests/yaml/bomb.yml">Yaml bomb</a>:</p>
<pre><code>a: &amp;a ["lol","lol","lol","lol","lol","lol","lol","lol","lol"]
b: &amp;b [*a,*a,*a,*a,*a,*a,*a,*a,*a]
c: &amp;c [*b,*b,*b,*b,*b,*b,*b,*b,*b]
d: &amp;d [*c,*c,*c,*c,*c,*c,*c,*c,*c]
e: &amp;e [*d,*d,*d,*d,*d,*d,*d,*d,*d]
f: &amp;f [*e,*e,*e,*e,*e,*e,*e,*e,*e]
g: &amp;g [*f,*f,*f,*f,*f,*f,*f,*f,*f]
h: &amp;h [*g,*g,*g,*g,*g,*g,*g,*g,*g]
i: &amp;i [*h,*h,*h,*h,*h,*h,*h,*h,*h]
</code></pre>
<p>This simple Yaml file exponentially allocates lols into a <a href="https://en.wikipedia.org/wiki/Billion_laughs_attack">billion
laughs</a>, so I prompted the
agent to also add in depth tracking to terminate parsing after a configurable
number of nodes or depths have been crossed.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>With the test structure setup, it was plain sailing. I prompted the agent to maintain a strict separation between a pure Yaml parser (with a single dependency on bytesrw), and then have Unix and Eio converters using the Bytesrw unix one, or the <a href="https://tangled.org/anil.recoil.org/ocaml-bytesrw-eio">bytesrw-eio</a> one I coded up <a href="https://anil.recoil.org/notes/aoah-2025-5">yesterday</a>.</p>
<p>As a useful aid to debugging, I also prompted the test suite to output nice HTML, which you can <a href="https://www.cl.cam.ac.uk/~avsm2/yaml-test-results.html">browse here</a>. It's convenient to have a rendered version of the entire test suite!</p>
<p>I also coded up a quick <a href="https://github.com/janestreet/core_bench">core-bench</a>
library to differentially test both the original Yaml library and this new one
on the full yaml test suite, and the pure OCaml one seems around 20% faster.
I'm going to do a bit more memory benchmarking in addition to performance
before I'm confident in these results, but the vibe coded smoke test was
reassuring to see that I wasn't dramatically slower. Memory usage remains a
risk of being high, though; something to look at for a future day.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>This was the first day of the advent adventure where I really felt like I'd hit
a breakthrough! Yamlrw is a dropin replacement for <em>all</em> of my uses of
ocaml-yaml now, and I can't find any regressions. After a bit more code review,
I'm going to post on the OCaml forums to request existing users to test this
one and see if they can find any regressions.</p>
<p>It's also nice having a streaming Eio Yaml parser, which will be convenient
for some projects in the remaining days, like my contacts manager. But first,
in <a href="https://anil.recoil.org/notes/aoah-2025-7">Day 7</a> we'll build a nice yamlt codec library...</p>

