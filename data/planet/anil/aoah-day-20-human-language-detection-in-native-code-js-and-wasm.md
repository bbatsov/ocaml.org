---
title: 'AoAH Day 20: Human language detection in native code, JS and wasm'
description: Porting the Nu HTML Validator's language detection to OCaml, then optimizing
  from 115MB to 28MB and fixing WASM array limits for browser deployment.
url: https://anil.recoil.org/notes/aoah-2025-20
date: 2025-12-20T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-langdetect-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>I took a break from yesterday's <a href="https://anil.recoil.org/news.xml">bot hacking</a> to continue the <a href="https://anil.recoil.org/notes/aoah-2025-15">HTML5 parsing</a> in OCaml adventure. Vibespiling seems to have taken off, with <a href="https://simonwillison.net/">Simon Willison</a> <a href="https://simonwillison.net/2025/Dec/18/swift-justhtml/">reporting</a> that there's a <a href="https://github.com/kylehowells/swift-justhtml">Swift version</a> now as well. I got curious about how far I could push the vibespiling support: could we go beyond "just" parsing to also do <em>complete HTML5 validation</em>? The <a href="https://validator.github.io/validator/">Nu HTML Validator</a> is where I went next, which is a bunch of Java code used by the W3C to apply some seriously complex rules for HTML5 validation.</p>
<p>I decided to split this work into two days, and started with a simple problem: HTML5 validation includes the need for automated language detection to validate that the <code>lang</code> attribute on HTML elements matches the actual content. This is important for accessibility, as screen readers use language hints to select the correct pronunciation.</p>
<p>The W3C validator uses the <a href="https://github.com/shuyo/language-detection">Cybozu langdetect</a> algorithm, so I vibespiled this into pure OCaml code as <strong><a href="https://tangled.org/anil.recoil.org/ocaml-langdetect">ocaml-langdetect</a></strong>. However, I decided to push harder by compiling this to <em>three</em> different backends: native code OCaml, JavaScript via <a href="https://github.com/ocsigen/js_of_ocaml">js_of_ocaml</a> and then into <a href="https://tarides.com/blog/2025-02-19-the-first-wasm-of-ocaml-release-is-out/">modern WebAssembly using wasm_of_ocaml</a>. As a fun twist, I got the regression tests running as interactive "<a href="https://anil.recoil.org/notes/aoah-2025-17">vibesplained</a>" online notebooks that can do language detection in the browser.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/langdetect-js/langdetect.html"> <img src="https://anil.recoil.org/images/aoah-langdetect-ss-1.webp" alt="%c" title="The JavaScript version is interactive so you can test it out directly as you read this post."> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#the-n-gram-frequency-algorithm" class="anchor" aria-hidden="true"></a>The n-gram frequency algorithm</h2>
<p>Language detection via <a href="https://en.wikipedia.org/wiki/N-gram">n-gram analysis</a> is surprisingly simple. The algorithm first trains profiles for each language, by analysing a corpus of text and counting the frequency of sequences of 1-3 characters. This creates a statistical fingerprint of the language.
Then, when given unknown text it extracts its n-grams and compares against all trained profiles using Bayesian probability. The language whose profile best matches the text wins.</p>
<p>It turns out that n-gram frequencies are <a href="https://web.stanford.edu/~jurafsky/slp3/3.pdf">remarkably
stable</a> across different texts
in the same language. To pick an obvious example, the word "the" appears
frequently in English texts, giving bigrams "th" and "he" high frequencies.
Similarly, "qu" is common in French, "sch" in German, etc etc.  The algorithm
uses multiple trials with randomized sampling to avoid overfitting to any
particular part of the text. Each trial adjusts the smoothing parameter
slightly using a Gaussian distribution, all of which should be straightforward
to implement in OCaml.</p>
<h2><a href="https://anil.recoil.org/news.xml#implementing-langdetect-in-ocaml" class="anchor" aria-hidden="true"></a>Implementing langdetect in OCaml</h2>
<p>I grabbed the <a href="https://github.com/validator/validator/tree/main/langdetect">validator/validator/langdetect</a> directory and vibespiled it from Java to OCaml, which is straightforward now with all the earlier Claude skills I've developed this month. The major hurdle to leap that's different from the other libraries is where to stash the precomputed ngram statistics for all the different languages. I wrote <a href="https://github.com/mirage/ocaml-crunch/blob/main/CHANGES.md">ocaml-crunch</a> for Mirage back in 2011 which just generates OCaml modules, but it's still <a href="https://github.com/tarides/hackocaml/issues/21">surprisingly difficult</a> to be more efficient and store precomputed data. <a href="https://www.cst.cam.ac.uk/people/jdy22">Jeremy Yallop</a> <a href="https://discuss.ocaml.org/t/generating-a-map-at-compile-time/16217/7?u=avsm">noted</a> back in March that his <a href="https://www.cl.cam.ac.uk/~jdy22/projects/modular-macros/">modular macros</a> project should support this sort of usecase but it's not quite ready yet. Similarly, using OCaml Marshal requires stashing the marshalled datastructure somewhere, which is hard to do portably.</p>
<p>Without a clear optimisation strategy, I prompted the agent to just precompute the profiles directly into OCaml
code.  The initial port worked immediately thanks to the clear structure of the
Java code it was being vibespiled from. The static library was 115MB, but I
didn't really notice as the regression tests all passed. The language profiles
contain 172,000 unique n-grams across 47 languages, and the naive approach of
generating one OCaml module per language with string literals duplicated
n-grams across profiles.</p>
<p>The native code library provides a straightforward interface to query the ngrams via a cmdliner binary:</p>
<pre><code class="language-sh">$ dune exec langdetect
Hello Thomas Gazagnaire, I'm finally learning French! Just kidding, I don't know anything about it.
en 1.0000

$ dune exec langdetect
Bonjour Thomas Gazagnaire, j'apprends enfin le français ! Je plaisante, je n'y connais rien.
fr 1.0000

$ dune exec langdetect
Hello Thomas Gazagnaire, I'm finally learning French! Just kidding, I don't know anything about it.
Bonjour Thomas Gazagnaire, j'apprends enfin le français ! Je plaisante, je n'y connais rien.
en 0.5714
fr 0.4286
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#the-115mb-problem-for-javascript" class="anchor" aria-hidden="true"></a>The 115MB problem for JavaScript</h2>
<p>But then, when I compiled it to JavaScript using the <a href="https://dune.readthedocs.io/en/latest/reference/dune/executable.html#jsoo-field">dune
stanzas</a>
the massive size was a little too big, with very long compilation times.  The
<a href="https://tangled.org/anil.recoil.org/ocaml-langdetect/commit/69e99a9c342957eee8db079137c803b0895e63fa">fix</a> was simply to pack everything into a shared data structure across <em>all</em>
languages, looking something like this:</p>
<pre><code class="language-ocaml">(* Shared string table for all 172K unique n-grams *)
let ngram_table = [| "the"; "th"; "he"; ... |]

(* Flat int array: (ngram_index, frequency) pairs for all languages *)
let profile_data = [| 0; 15234; 1; 8921; ... |]

(* Offsets: (lang_code, start_index, num_pairs) *)
let profile_offsets = [|
  ("en", 0, 4521);
  ("fr", 9042, 3892);
  ...
|]
</code></pre>
<p>This reduced the binary from 115MB to around 28MB, which is a reasonable
reduction without having to resort to compression.  Many n-grams appear in
multiple languages (consider Latin alphabet characters) so deduplicating into a
shared string table eliminated quite a bit of redundancy.</p>
<p>At this point, I prompted the agent to build me a full Javascript based
regression test that took the native code, and gave me a browser based
version instead.</p>
<p>One minor hiccup was that the regression tests failed due to JavaScript
integers overflowing <em>vs</em> native integers, but the fix was simple and the
regression tests in the browser made debugging them easy for the agentic loop. Without them, there would have been a lot of human cut-and-pasting which is quite tedious!</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/langdetect-js/langdetect.html"> <img src="https://anil.recoil.org/images/aoah-langdetect-ss-2.webp" alt="%c" title="The JavaScript version also executes regression tests in the browser environment derived from the native tests."> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#the-wasm-array-limit" class="anchor" aria-hidden="true"></a>The WASM array limit</h2>
<p>With the JavaScript size under control, I turned to WASM compilation via <code>wasm_of_ocaml</code>. The first attempt failed with a cryptic error about exceeding operands and a parse error. It turns out WASM's <code>array_new_fixed</code> instruction has a limit of 10000 operands, and our profile data array had 662,000 elements.</p>
<p>The <a href="https://tangled.org/anil.recoil.org/ocaml-langdetect/commit/6f25190ebfb2edc4697b3a2ab05e6b33ae1cba3b">solution</a> was to chunk the arrays and concatenate at runtime, which incurs runtime overhead but <a href="https://github.com/konsoletyper/teavm/issues/971">is a common enough solution</a>.  The generated code now includes 74 chunks for the profile data and 20 chunks for the n-gram string table, but clocks in at around 20MB and could probably be reduced further with some browser compression.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/langdetect-js/langdetect.html"> <img src="https://anil.recoil.org/images/aoah-langdetect-ss-3.webp" alt="%c" title="Wasm mode is effectively the same as JavaScript, but more modern."> </a></p>
<p>Now, the browser tests include the ability to switch between Wasm and
JavaScript in the same test HTML. There was no real performance difference here, but the dataset is small. The most observable difference is that the wasm needs to be served via a web server and not local filesystem, as otherwise browsers reject it. The browser also must serve <code>.wasm</code> files as mime type <code>application/wasm</code> or it's promptly rejected.</p>
<h2><a href="https://anil.recoil.org/news.xml#browser-demo" class="anchor" aria-hidden="true"></a>Browser demo</h2>
<p>The OCaml <code>langdetect-js</code> package provides a browser-ready API using Brr callbacks from HTML to register them with the JavaScript:</p>
<pre><code>// Detect language
const lang = langdetect.detect("Hello, world!");  // "en"

// Get probability scores
const result = langdetect.detectWithProb("Bonjour le monde");
// { lang: "fr", prob: 0.9987 }

// Get all candidates
const all = langdetect.detectAll("这是中文");
// [{ lang: "zh-cn", prob: 0.85 }, { lang: "zh-tw", prob: 0.12 }, ...]
</code></pre>
<p>I also <a href="https://www.npmjs.com/package/langdetect-jsoo">published this to npm</a> so that the JavaScript is conveniently available via a CDN like <a href="https://cdn.jsdelivr.net/npm/langdetect-jsoo@1.0.0/langdetect.js">jsDeliver</a>.  Publishing to npm required putting a <a href="https://tangled.org/anil.recoil.org/ocaml-langdetect/tree/npm">npm branch</a> in the repo with the npm <code>package.json</code>. I followed the convenient guide by <a href="https://simonwillison.net/">Simon Willison</a> to <a href="https://til.simonwillison.net/npm/npm-publish-github-actions">get a minimal package.json</a> for the project.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>This was a good intermediate port to work on since it let me exercise Webassembly a bit more, and understand the tradeoffs in OCaml compilation to these other backends.  The process of getting the agent to systematically port first to native code (from Java), and then compile to JavaScript and debug platform-specific issues like the integer overflows, and then go to wasm was quite good.</p>
<p>The agent was particularly helpful for the tedious work of generating the
chunked array code and debugging the Unicode normalization edge cases.</p>
<p>For future hacking, there are several language optimisations coming up in <a href="https://oxcaml.org">OxCaml</a> that should make this even more efficient; support for compile time metaprogramming (so I could for example compute a perfect hash statically for all the ngrams), and also for smaller integer sizes so I dont need to use a full 31-bit range for the ngram values. However, I couldn't quite get the wasm_of_ocaml constraints on the oxcaml branch working so I ran out of time today to get this going. Package management takes me out of the flow zone yet again!</p>
<p>Now that langdetect works, we'll go onto the full HTML5 validator in <a href="https://anil.recoil.org/notes/aoah-2025-21">Day 21</a>!</p>

