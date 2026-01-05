---
title: 'AoAH Day 15: Porting a complete HTML5 parser and browser test suite'
description: Vibespiling JustHTML from Python to pure OCaml, achieving 100% pass rate
  on the browser html5lib test suite using agentic workflows.
url: https://anil.recoil.org/notes/aoah-2025-15
date: 2025-12-15T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-html5-ss-tests.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After my success with <a href="https://anil.recoil.org/notes/aoah-2025-6">Yaml 1.2</a> in pure OCaml, I found <a href="https://github.com/EmilStenstrom/justhtml">JustHTML</a>, a new Python library for parsing HTML5 by <a href="https://friendlybit.com">Emil Stenström</a> <em>(via <a href="https://simonwillison.net/">Simon Willison</a> <a href="https://simonwillison.net/2025/Dec/14/justhtml/">posting</a> about it)</em>.
Emil wrote JustHTML <a href="https://friendlybit.com/python/writing-justhtml-with-coding-agents/">using coding agents</a> as well, and then Simon <a href="https://simonwillison.net/2025/Dec/15/porting-justhtml/">ported it to JavaScript in a few hours</a>.</p>
<p>My question, though, is how difficult is to go in the <em>other</em> direction and move towards a strongly typed interface like OCaml's. Could we ultimately distill down the extremely complex set of rules around parsing HTML all the way into a proof assistant like Lean, but hopping via OCaml and Haskell to provide convenient executable pitstops?</p>
<p>Today's task was to vibespile the Python into <strong><a href="https://tangled.org/anil.recoil.org/ocaml-html5rw">ocaml-html5rw</a></strong>, a pure OCaml HTML5 parser and serialiser that passes the <a href="https://github.com/html5lib/html5lib-tests">browser test suite</a> 100%.</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>I took a very similar approach to my earlier <a href="https://anil.recoil.org/notes/aoah-2025-6">Yaml 1.2</a> port,
depending purely on <a href="https://github.com/dbuenzli/bytesrw">Bytesrw</a> as the
string decoding/encoding codec. I instructed the agent not to use any other
external libraries, and to build up a test suite that could decode the
html5lib-tests to act as an external oracle. I also used my <a href="https://anil.recoil.org/notes/aoah-2025-11">earlier</a> Claude skills to tidy up OCaml code that was generated.</p>
<p>I also used an earlier trick to get my freshly generated OCaml test suite (which is itself parsing the
third-party html5lib expect tests) to output a standalone HTML report. This was
extremely useful to see progress, but also for me to understand what sort of thing HTML5 parsing involves. You can <a href="https://www.cl.cam.ac.uk/~avsm2/html5rw/">browse a snapshot</a> to judge for yourself.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/html5rw/"> <img src="https://anil.recoil.org/images/aoah-html5-ss-tests.webp" alt="%c" title="The HTML5 test suite is quite scary, but this is what browser developers have to deal with."> </a></p>
<p>I'd never actually realised before doing this that, unlike many parsers, HTML5
parsing never actually fails. The <a href="https://html.spec.whatwg.org/multipage/">WHATWG
specification</a> is a living standard
that defines error recovery rules for almost every possible malformed input,
ensuring all HTML documents produce a valid DOM tree just like browsers do.
So, the biggest danger here is that we parse a minor syntax error into a nonsensical
DOM that is "far away" from the author intention.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>The HTML5 port took a few hours, and the resulting library seems to pass the
HTML5 tests without much drama. One footgun is that the test runner itself was
quite complex, and hidden in there was some skipping of test cases. So my
review of this library actually happens in reverse: I read the test runners
first to figure out what's going on, only working backwards to the library
itself. A very strange workflow...</p>
<p><img src="https://anil.recoil.org/images/aoah-html5-ss-1.webp" alt="%c" title="The planning using subagents churned through the tests fairly quickly"></p>
<p>There's nothing too surprising so far; after all <a href="https://simonwillison.net/">Simon Willison</a> ported it to
JavaScript in no time at all. So what makes this interesting to do in OCaml
<em>vs</em> Python or Javascript? The obvious thing is modules and ease of
refactoring, so I spent some time critically going through the resulting
module structure of the library itself.</p>
<h3><a href="https://anil.recoil.org/news.xml#avoid-reinventing-the-wheel" class="anchor" aria-hidden="true"></a>Avoid reinventing the wheel</h3>
<p>A lot of the code inside the library looked suspiciously like it was reinventing the Unicode wheel, with lots of character-encoding specific manipulation. So I cloned some key libraries from the OCaml ecosystem that are both fairly standalone and well engineered: <a href="https://github.com/dbuenzli/astring">astring</a>, <a href="https://github.com/dbuenzli/uutf">uutf</a>, and <a href="https://github.com/dbuenzli/uunf">uunf</a> by <a href="https://erratique.ch">Daniel Bünzli</a>.</p>
<p><img src="https://anil.recoil.org/images/aoah-html5-ss-3.webp" alt="%c" title="The planning process uses parallel agents to explore each library, to minimise context usage.">
<img src="https://anil.recoil.org/images/aoah-html5-ss-4.webp" alt="%c" title="The agent then returns with results of the search including recommendations."></p>
<p>I cloned a few candidate libraries, and the agentic analyse determined that only some of them were relevant to the problem at hand, so I discarded the rest and focussed on the top set.</p>
<p>The <a href="https://tangled.org/anil.recoil.org/ocaml-html5rw/commit/958671e9df25f94f38795ada0d291aa5355f024a">resulting diff</a> crunched the size of the library down considerably. Since we had extensive test coverage, the 100% pass rate at the end built up some confidence that semantics weren't changed too badly. The existence of OCaml interface files also meant I could inspect those separately in the diff and be satisfied that only implementations had changed.</p>
<h3><a href="https://anil.recoil.org/news.xml#types-making-browsing-the-specification-fun" class="anchor" aria-hidden="true"></a>Types making browsing the specification fun</h3>
<p><a href="https://dev.realworldocaml.org/files-modules-and-programs.html">Modules</a> and types are defining feature of OCaml, so I pointed the agent at the WHATWG standard and asked it to introduce explanations directly into the interface files themselves. This is quite different from an informal specification; the guidelines can be browsed directly and navigated around via the <a href="https://www.cl.cam.ac.uk/~avsm2/htmlrw-doc/html5rw/Html5rw/index.html">odoc HTML output</a>.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/htmlrw-doc/html5rw/Html5rw/index.html"> <img src="https://anil.recoil.org/images/aoah-html5-ss-odoc.webp" alt="%c" title="Learn all sorts of random HTML5 facts like context sensitive fragment parsing by browsing the OCaml docs!"> </a></p>
<p>When I was browsing the types, I realised that there were too many strings involved in parsing errors, and so the agent helped synthesise them into an <a href="https://tangled.org/anil.recoil.org/ocaml-html5rw/commit/c9a783be220f20922d1bf5106852f8ff7711cc19">extensible OCaml variant</a> that describes things much more precisely.  This has gotten me thinking about 'doing verification in reverse'; just like <a href="https://www.quantamagazine.org/mathematical-beauty-truth-and-proof-in-the-age-of-ai-20250430/">AI for maths</a> is shifting what it means to do math, this sort of thing is shifting what it means to do formal specification.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>I'll end by asking the same <a href="https://simonwillison.net/2025/Dec/15/porting-justhtml/">questions</a> that Simon did:</p>
<blockquote>
<p>I’ll end with some open questions:</p>
<ul>
<li>Does this library represent a legal violation of copyright of either the Rust library or the Python one?</li>
<li>Even if this is legal, is it ethical to build a library in this way?</li>
<li>Does this format of development hurt the open source ecosystem?</li>
<li>Can I even assert copyright over this, given how much of the work was produced by the LLM?</li>
<li>Is it responsible to publish software libraries built in this way?</li>
<li>How much better would this library be if an expert team hand crafted it over the course of several months?
<cite>-- <a href="https://simonwillison.net/2025/Dec/15/porting-justhtml/">I just ported JustHTML from Python to JavaScript</a>, Simon Willison, 2025</cite></li>
</ul>
</blockquote>
<p>I feel the last question is answered most easily: an expert team that has access to these tools <em>and</em> the domain knowledge about HTML5 should be able to do a good job. While I have no experimental evidence in this domain about that fact, we did find earlier this year that <a href="https://anil.recoil.org/papers/2024-ce-llm">expert-level retrieval of conservation evidence could be significantly boosted</a> via access to <a href="https://anil.recoil.org/papers/2025-evidence-tap">living evidence databases</a>. I feel agentic search sits alongside the same 'needle-in-a-haystack' productivity boost; scanning through thousands of HTML5 test cases to find the problem is something that agents are better at than humans who do not have domain knowledge (like me, in this case).</p>
<p>The question of copyright and licensing is difficult. I definitely did <em>some</em>
editing by hand, and a fair bit of prompting that resulted in targeted code
edits, but the vast amount of architectural logic came from JustHTML. So I
opted to make the <a href="https://tangled.org/anil.recoil.org/ocaml-html5rw/blob/main/LICENSE.md">LICENSE a joint one</a>
with <a href="https://friendlybit.com">Emil Stenström</a>. I did not follow the transitive dependency through to the Rust
one, which I probably should.</p>
<p>I'm also extremely uncertain about every releasing this library to the central
opam repository, especially as there are <a href="https://github.com/aantron/lambdasoup">excellent HTML5
parsers</a> already available. I haven't
checked if those pass the HTML5 test suite, because this is wandering into the
agents <em>vs</em> humans territory that I ruled out in my <a href="https://anil.recoil.org/notes/aoah-2025">groundrules</a>.
Whether or not this agentic code is better or not is a moot point if releasing
it drives away the human maintainers who are the source of creativity in the code!</p>
<p>I note that throughout my entire AoAH adventure so far, most of the code I've
generated has been spectacularly unoriginal. I've received tons of help and
examples of how to use tools from colleagues as <em>input</em>, but the aggregate set
of OCaml code <em>output</em> that I would class as refreshingly interesting from me
is pretty minimal.
So in the long term, I don't think this process is "helping the core" of the
community by coming up with beautiful functional pearls.</p>
<p>However, the libraries <em>are</em> satisfying an important need for utility: some
things like Yaml and HTML5 are fundamentally such ugly formats that I find it
hard to argue for elegant solutions therein, and yet we need to manipulate them
to live in the real world. So I'm ending up with a utilitarian plea, with some
angst that this might be smothering the functional flame that make's OCaml so
much fun in the first place. I must ask <a href="https://www.educ.cam.ac.uk/people/staff/gibson/">Jenny Gibson</a> for her views on where
play fits into programming the next time I see her!</p>
<p>Tomorrow in <a href="https://anil.recoil.org/notes/aoah-2025-16">Day 16</a> we'll use this new library to help with
"vibesplaining" code via executable notebooks.</p>

