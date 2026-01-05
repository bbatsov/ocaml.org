---
title: What I learnt at ICFP/SPLASH 2025 about OCaml, Hazel and FP
description: Highlights from ICFP/SPLASH 2025 including Hazel live programming, OCaml
  AI tooling, formally verified GC, and cross-community discussions between Haskell
  and OCaml.
url: https://anil.recoil.org/notes/icfp25-what-i-learnt
date: 2025-10-09T00:00:00-00:00
preview_image: https://anil.recoil.org/images/icfp-18.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>This is part 5 of a <a href="https://anil.recoil.org/notes/icfp25">series</a> of posts<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> about ICFP 2025.</p>
<p>In addition to giving a bunch of talks about
<a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">Docker</a>, <a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX</a> and
<a href="https://anil.recoil.org/notes/icfp25-propl">planetary computing</a>, the greatest fun at a huge conference
like ICFP and SPLASH is seeing talks given by my students (they grow up so
fast!) and collaborators, and generally floating around random talks trying to
deceipher ancient Greek lambdas floating on a projector.</p>
<h2><a href="https://anil.recoil.org/news.xml#hazel-live-programming-and-type-level-debugging" class="anchor" aria-hidden="true"></a>Hazel live programming and type level debugging</h2>
<p>I've been wanting to try to do something with <a href="https://hazel.org">Hazel</a> ever
since <a href="https://web.eecs.umich.edu/~comar/">Cyrus Omar</a> showed it to me at <a href="https://watch.eeg.cl.cam.ac.uk/w/3nGExywoVm6XFRBA2zYxSL">last year's PROPL</a>.
<a href="https://maxcarroll0.github.io/blog/">Max Carroll</a> picked up the idea of doing <a href="https://anil.recoil.org/ideas/gradual-type-error-debugging">gradual type-level debugging</a> for his Part II undergraduate project
at the Computer Lab. He not only aced his project, but wrote up a
<a href="https://maxcarroll0.github.io/assets/papers/Carroll-Decomposable_Type_Highlighting.pdf">paper</a>
for the <a href="https://conf.researchr.org/home/icfp-splash-2025/hatra-2025">HATRA</a>
workshop:</p>
<blockquote>
<p>We explore how to provide programmers with an interactive interface for
explaining the process by which static types and dynamic casts are derived,
with the goal of improving the debugging of static and dynamic type errors.</p>
<p>To this end, we define mathematical foundations for a decomposable
highlighting system within a bidirectional system, and show how these can be
propagated through dynamic types in a cast system. Our prototype
implementation in the gradually typed Hazel language includes a web-based
user interface, through which we highlight the importance of type level
debugging.
<cite>-- <a href="https://maxcarroll0.github.io/assets/papers/Carroll-Decomposable_Type_Highlighting.pdf">Decomposable Type Highlighting for Bidirectional Type and Cast Systems</a>, Carroll 2025</cite></p>
</blockquote>
<p><a href="https://youtu.be/P-x1msRL7XU?t=3994"> <img src="https://anil.recoil.org/images/icfp-18.webp" alt="%c" title="Max Carroll presenting his work on gradual type-level debugging in Hazel"> </a></p>
<p><a href="https://maxcarroll0.github.io/blog/">Max Carroll</a> delivered a fantastic first conference talk, complete with a live
demo demonstrating type-level debugging in action; give it a watch if you're
interested in live programming!</p>
<p>One issue we had during this project was finding a decent corpus of functional
code <em>with errors</em> to use to test out Max's debugger. Hazel's a pretty young
language, and finding a large codebase is difficult, let alone a bunch of code
with errors. <a href="https://patrick.sirref.org">Patrick Ferris</a> decided to accelerate this process by building a
<a href="https://github.com/patricoferris/hazel_of_ocaml">hazel_of_ocaml</a> and
presenting this work at the <a href="https://conf.researchr.org/home/icfp-splash-2025/tyde-2025#program">TyDE workshop</a>.</p>
<p><a href="https://www.youtube.com/watch?v=VJM5-IVQ8lw&amp;t=21045s"> <img src="https://anil.recoil.org/images/icfp-pf341-tyde.webp" alt="%c" title="Patrick Ferris presents Hazel-of-OCaml at TyDE 2025"> </a></p>
<p>With Patrick's transpiler, we grabbed <a href="https://eric.seidel.io/">Eric Seidel</a>'s
corpus of <a href="https://zenodo.org/records/806814">ill-typed OCaml</a> that he built
for his research on <a href="https://dl.acm.org/doi/10.1145/2951913.2951915">dynamic witnesses for static type
errors</a>. Max successfully used
this translated corpus to build his type-level debugger, and is planning to
continue to work on this in his Part III project this year.</p>
<h2><a href="https://anil.recoil.org/news.xml#three-steps-for-ocaml-to-crest-the-ai-humps" class="anchor" aria-hidden="true"></a>Three Steps for OCaml to Crest the AI Humps</h2>
<p>I've been <a href="https://anil.recoil.org/notes/claude-copilot-sandbox">spending a lot of time with my friend Claude</a> recently, and so have <a href="https://toao.com">Sadiq Jaffer</a> and
<a href="https://jon.recoil.org">Jon Ludlam</a>. We <a href="https://anil.recoil.org/papers/2025-ocaml-ai">wrote up</a> our experiences with interfacing
OCaml with coding agents, and Sadiq <a href="https://youtu.be/Xh5PNe0SxDY?t=24042">presented it</a> to an interactive crowd at the <a href="https://conf.researchr.org/home/icfp-splash-2025/ocaml-2025">OCaml Workshop</a>.</p>
<p><a href="https://youtu.be/Xh5PNe0SxDY?t=24042"> <img src="https://anil.recoil.org/images/icfp-26.webp" alt="%c" title="Sadiq couldnt resist a good pun for his OCaml Workshop talk"> </a></p>
<p>Aside from the very sensible
<a href="https://jon.recoil.org/blog/2025/08/ocaml-lsp-mcp.html">guidance</a> on MCP and
tools, I discovered a couple of things from this work:</p>
<ul>
<li><a href="https://toao.com">Sadiq Jaffer</a> found <a href="https://www.tbench.ai/">terminal-bench</a> and added an <a href="https://toao.com/blog/gc-debug-terminal-bench">OCaml GC debugging task</a>. This has the effect of getting the frontier AI labs to point their mega training tasks at OCaml-related problems, thus making a rising tide for everyone! And looking at <a href="https://github.com/laude-institute/terminal-bench/commits/main/tasks/fix-ocaml-gc">the history of the task</a>, other labs are raising the timeout on Sadiq's task, meaning that fixing bugs in the OCaml GC is right at the top end of difficulty. Let's get more problems into terminal-bench!</li>
<li>It also surprised me just how good the <a href="https://qwen.ai/home">Qwen coder</a> models are <a href="https://toao.com/blog/ocaml-local-code-models">on simple OCaml tasks</a>. Local models are fairly far behind Claude's, but the gap is closing as the innovation moves to the agentic context management. I'm excited to see <a href="https://github.com/tmattio">Thibaut Mattio</a>'s work on <a href="https://getspice.dev/">Spice</a> (see his <a href="https://youtu.be/e8Dkj47nxbg?t=99">FunOCaml talk</a>) as that combines these local models with OCaml-specific context management.</li>
</ul>
<h2><a href="https://anil.recoil.org/news.xml#formally-verified-garbage-collector-for-ocaml" class="anchor" aria-hidden="true"></a>Formally verified garbage collector for OCaml</h2>
<p>Sheera Shamsu gave a fantastic <a href="https://youtu.be/Xh5PNe0SxDY?t=4364">talk</a> on building a formally
specified garbage collector for OCaml to a very crowded room! This was rather
topical given our <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">musings on multiple runtimes</a> in
the shift from OCaml 4 to 5.</p>
<p><a href="https://youtu.be/Xh5PNe0SxDY?t=4364"> <img src="https://anil.recoil.org/images/icfp-21.webp" alt="%c" title="Sheera Shamsu on a mechanically verified GC for OCaml"> </a></p>
<blockquote>
<p>[...] we propose a strategy for crafting a correct, proof-oriented GC from
scratch, designed to evolve over time with additional language features. Our
approach neatly separates abstract GC correctness from OCaml-specific GC
correctness, offering the ability to integrate further GC optimizations,
while preserving core abstract GC correctness. As an initial step to
demonstrate the viability of our approach, we have developed a verified
stop-the-world mark-and- sweep GC for OCaml. The approach is mechanized in Fstar
and its low-level subset Lowstar.
<cite>-- <a href="https://link.springer.com/article/10.1007/s10817-025-09721-0">A Mechanically Verified Garbage Collector for OCaml</a>, Shamsu et al 2025</cite></p>
</blockquote>
<p>Chatting to <a href="https://kcsrk.info">KC Sivaramakrishnan</a> afterwards, it seems that there's interest in shifting to
Lean from Fstar to investigate if the ergonomics of the proofs are better. But
as baselines go, the mechanically verified collector always beat the
conservative Boehmm-GC, which means it's no worse than the current more
conservative choice. That's good work!</p>
<p><img src="https://anil.recoil.org/images/icfp-22.webp" alt="%c" title="A full room for verified GCs!"></p>
<h2><a href="https://anil.recoil.org/news.xml#haskell-and-ocaml-the-twain-shall-meet" class="anchor" aria-hidden="true"></a>Haskell and OCaml, the Twain Shall Meet?</h2>
<p>Most "wonderful ICFP experiences" usually include crossing the lines
to go hang out with <em>other</em> language communities.</p>
<p>Back in 2014, I stayed up all night before my <a href="https://anil.recoil.org/videos/ed84b2eb-1b93-4dc3-b746-63a4af13d4ea">keynote</a> to the Haskell Symposium
trying to encode OCaml functors as Haskell typeclasses and even got help on
stage from friendly Haskellers.  This year, <a href="https://richarde.dev/">Richard Eisenberg</a> was my absolute
highlight with a <a href="https://youtu.be/IlQQElKaFvM?t=13184">superb session</a> on what
he's learnt from being the rare breed of someone steeped deeply <em>both</em> in
Haskell and OCaml.  The room was so packed for this talk that they had to
create an overflow room streaming it in the corridors!</p>
<p><a href="https://youtu.be/IlQQElKaFvM?t=13184"> <img src="https://anil.recoil.org/images/icfp-23.webp" alt="%c" title="Richard Eisenberg setting up in a crowded room for his keynote"> </a></p>
<p>Richard talked about his experiences with being <em>both</em> an OCaml and Haskeller,
and went through a series of examples illustrating the differences between the
two. He didn't get very far before the audience got involved, with both
Haskellers and OCamlers putting their 2c in! For that reason, the stream
recording might not work so well.</p>
<p><a href="https://youtu.be/IlQQElKaFvM?t=13184"> <img src="https://anil.recoil.org/images/icfp-25.webp" alt="%c"> </a></p>
<p>It's worth watching the talk rather than me going through each of his examples,
but I did have a long morning coffee with <a href="https://en.wikipedia.org/wiki/Simon_Peyton_Jones">Simon Peyton Jones</a> when I got back to Cambridge about what
the essential difference is between OCaml and Haskell. Laziness seems like a
detail, but purity is absolutely key; it percolates through every other design
decision (like ordering of variables, or module generativity, and so on) since
side-effects lurk everywhere in OCaml.</p>
<p><img src="https://anil.recoil.org/images/icfp-19.webp" alt="%c" title="Jane Street had a fun 'corridor track' where they contrasted Haskell and OCaml to passerbys as well, including an unfortunate wedding party that happened to be on the same floor as us."></p>
<p>I think it's really important to have these cross-community in-person moments. One call to action in Richard's talk was for us to consider having a unified "Haskell/ML Symposium" where long-form research papers could be shared, with shorter language-specific workshops. One audience member asked why this couldn't just be the <a href="https://conf.researchr.org/home/icfp-splash-2025/mlsymposium-2025#event-overview">ML Workshop</a>, and Richard promptly pointed out that it has "Higher-order, Typed, Inferred, <strong>Strict</strong>" in the title! Just excising one word might unify two communities long split for decades...</p>
<p>Aside from language matters, I think it would be a good idea to bring more of the functional programming community together more often outside of the "main ICFP track" (which is high pressure and quite squeezed for time with little discussion outside the corridor tracks). I really miss <a href="https://cufp.org">CUFP</a>, since for a decade this was where the functional hackers would all meet up towards the tail end of the main conference. This year however, the workshops were run in parallel with the main ICFP and OOPSLA, which I think sadly diluted the community bonding a bit.</p>
<p><img src="https://anil.recoil.org/images/icfp-24.webp" alt="%c" title="KC is the other person who's done both OCaml and Haskell hacking, so it was kind of adorable to see him sitting beside SPJ during the talk!"></p>
<h2><a href="https://anil.recoil.org/news.xml#i-got-shriramed-about-our-cambridge-teaching" class="anchor" aria-hidden="true"></a>I got Shriram'ed about our Cambridge teaching</h2>
<p>Speaking of teaching, noone in the world can school me better than <a href="https://cs.brown.edu/~sk/">Shriram Krishnamurthi</a> when
it comes to matters of computer science pedagogy. I grabbed him at the lunch
break and asked him for advice on our upcoming reform of the Cambridge Computer
Science Tripos (I teach the <a href="https://anil.recoil.org/notes/focs">first course</a>). His opinions were legion,
and he kindly gave me a quick spin around what they are working on at Brown.</p>
<p>The SMoL (Standard Model of Languages) has a nice <a href="https://blog.brownplt.org/2024/04/12/behavior-misconceptions.html">web interface</a> and quiz, just like the one he helped on for our <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a>. SMoL is deliberately language agnostic:</p>
<blockquote>
<ul>
<li>If students master SMoL, they have a good handle on the core of several of these languages.</li>
<li>Students may find it easier to port their knowledge between languages: instead of being lost in a sea of different syntax, they can find familiar signposts in the common semantic features. This may also make it easier to learn new languages.</li>
<li>The differences between the languages are thrown into sharper contrast.</li>
<li>Students can see that, by going beyond syntax, there are several big semantic ideas that underlie all these languages, many of which we consider “best practices” in programming language design.
<cite>-- <a href="https://blog.brownplt.org/2024/04/12/behavior-misconceptions.html">Fixing Standard Misconceptions about Program Behaviour</a>, 2024</cite></li>
</ul>
</blockquote>
<p>Much like <a href="https://richarde.dev/">Richard Eisenberg</a>'s talk on Haskell/OCaml, the SMoL tutor shows multiple
languages for the same problem, rotating across Python, Scala, JavaScript and
so on. I like this idea <em>a lot</em> for our Foundations of CS course, as I've been
considering rotating in <a href="https://hazel.org">Hazel</a> into the mix to ease the
syntactic shock of using OCaml. SMoL takes this concept much further, and is
backed by serious <a href="https://cs.brown.edu/~sk/Publications/Papers/Published/">user studies</a> on students.</p>
<p>I also really liked the <a href="https://pyret.org/">Pyret</a> approach of starting to
teach using tables as a core datastructure, and not lists or arrays. However,
I'll need to think hard about how this teaching model would work under Cambridge's
quirky <a href="https://www.undergraduate.study.cam.ac.uk/supervisions-and-assessment">supervision model</a>.</p>
<p>This is on my queue to work on over the winter, while <a href="https://jon.recoil.org">Jon Ludlam</a> kindly <a href="https://jon.recoil.org/blog/2025/09/giving-hub-cl-an-upgrade.html">covers</a> my undergraduate <a href="https://www.cl.cam.ac.uk/teaching/2526/FoundsCS/">lectures</a> for this year while I'm on sabbatical!
On my reading list from chatting to him:</p>
<ul>
<li><a href="https://cacm.acm.org/opinion/data-centricity/">Data-Centricity: A Challenge and Opportunity for Computing Education</a>, CACM 2025.</li>
<li><a href="https://iase-pub.org/ojs/SERJ/article/view/190/95">Modeling as a core component of structuring data</a>, Konold 2017.</li>
</ul>
<p><img src="https://anil.recoil.org/images/icfp-27.webp" alt="%c" title="Not a bracket out of place when Shriram is demoing PyRet!"></p>
<h2><a href="https://anil.recoil.org/news.xml#deterministic-wasm" class="anchor" aria-hidden="true"></a>Deterministic WASM</h2>
<p>Webassembly has also gone a long way since I <a href="https://anil.recoil.org/notes/wasm-on-exotic-targets">last looked</a> into it.  I had a long chat with <a href="https://www.doc.ic.ac.uk/~pg/">Phillipa Gardner</a> on the
<a href="https://bsky.app/profile/ningkeli.bsky.social/post/3m2y4ncoiae2n">nature hike</a>
to learn about her work on <a href="https://dl.acm.org/doi/10.1145/3656440">SpecTec</a>,
which is a single source-of-truth DSL that describes both the
<a href="https://webassembly.org">Wasm</a> specification <em>and</em> the artefacts like the
interpreter.</p>
<p>After that, <a href="https://s3d.cmu.edu/people/core-faculty/titzer-ben.html">Ben Titzer</a> told me about <a href="https://arxiv.org/abs/2312.03858">WALI</a> which is an alternative approach to <a href="https://wasi.dev/">WASI</a> that simply exposes Linux kernel interfaces straight to the wasm runtime.  I'm rather amenable to this given my <a href="https://anil.recoil.org/notes/icfp25-post-posix">case for shared memory IO</a> earlier in the week at VMIL, so this is now on my list of things to investigate! <a href="https://kcsrk.info">KC Sivaramakrishnan</a>, <a href="https://tyconmismatch.com/code.html">Chris Casinghino</a> and I discussed what an <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml</a> wasm unikernel might look like (a lot of buzzwords, I know), and we are pretty close to OxCaml making it possible to write runtimes using itself -- it just needs support for "external memory", which is a topic the Jane Street <a href="https://blog.janestreet.com/wrought-2025/#ref-counted-objects-in-shared-memory">interns worked on</a> over their summer projects.</p>
<h2><a href="https://anil.recoil.org/news.xml#wrapup-thoughts-on-singapore" class="anchor" aria-hidden="true"></a>Wrapup thoughts on Singapore</h2>
<p>Overall, I had a brilliant -- if exhausting! -- week in ICFP in Singapore. I
loved the city, I loved the vibes around the conference, and it was totally
worth the trip. Huge thanks to Ilya Sergey and the organising team for making
this happen!</p>
<p><img src="https://anil.recoil.org/images/icfp-20.webp" alt="%c" title="The vegetarian food was amazing and my diet is in tatters."></p>
<p><img src="https://anil.recoil.org/images/icfp-5.webp" alt="%c" title="The coffee was 'ok'; I wonder what Satnam Singh thought about it!"></p>
<p><img src="https://anil.recoil.org/images/icfp-17.webp" alt="%c" title="The views were spectacular. Singaporean architecture is ridiculous."></p>
<p><small class="credits"> <em>10th Oct 2025: Typo fixes spotted by Shriram.</em> </small></p>

<div class="footnotes"><ol><li><p></p><p>See also in the <a href="https://anil.recoil.org/notes/icfp25">ICFP25</a> series: <a href="https://anil.recoil.org/notes/icfp25-propl">chairing PROPL25</a>, the <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a>, <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">multicore at Jane Street and Docker</a>, <a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX IO</a> and <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">what I learnt</a>.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy (2025). Oh my Claude, we need agentic copilot sandboxing right now. <a href="https://doi.org/10.59350/aecmt-k3h39" target="_blank"><i>10.59350/aecmt-k3h39</i></a></li>
<li>Madhavapeddy (2025). Foundations of Computer Science. <a href="https://doi.org/10.59350/qms3q-ymn65" target="_blank"><i>10.59350/qms3q-ymn65</i></a></li>
<li>Madhavapeddy (2025). Holding an OxCaml tutorial at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/55bc5-x4p75" target="_blank"><i>10.59350/55bc5-x4p75</i></a></li>
<li>Madhavapeddy (2025). It's time to go post-POSIX at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/mch1m-8a030" target="_blank"><i>10.59350/mch1m-8a030</i></a></li>
<li>Madhavapeddy (2025). A Roundup of ICFP/SPLASH 2025 happenings. <a href="https://doi.org/10.59350/4jf5k-01n91" target="_blank"><i>10.59350/4jf5k-01n91</i></a></li>
<li>Madhavapeddy (2025). Programming for the Planet at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/hasmq-vj807" target="_blank"><i>10.59350/hasmq-vj807</i></a></li>
<li>Madhavapeddy (2025). Webassembly on exotic architectures (a 2025 roundup). <a href="https://doi.org/10.59350/ycqj1-b3996" target="_blank"><i>10.59350/ycqj1-b3996</i></a></li>
<li>Madhavapeddy (2025). Jane Street and Docker on moving to OCaml 5 at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/3jkaq-d3398" target="_blank"><i>10.59350/3jkaq-d3398</i></a></li>
<li>Seidel et al (2016). Dynamic witnesses for static type errors (or, ill-typed programs usually go wrong). <a href="https://doi.org/10.1145/2951913.2951915" target="_blank"><i>10.1145/2951913.2951915</i></a></li>
<li>Shamsu et al (2025). A Mechanically Verified Garbage Collector for OCaml. Journal of Automated Reasoning. <a href="https://doi.org/10.1007/s10817-025-09721-0" target="_blank"><i>10.1007/s10817-025-09721-0</i></a></li>
<li>Youn et al (2024). Bringing the WebAssembly Standard up to Speed with SpecTec. Artifact for "Bringing the WebAssembly Standard up to Speed with SpecTec". <a href="https://doi.org/10.1145/3656440" target="_blank"><i>10.1145/3656440</i></a></li>
<li>Ramesh et al (2025). Empowering WebAssembly with Thin Kernel Interfaces. Proceedings of the Twentieth European Conference on Computer Systems. <a href="https://doi.org/10.1145/3689031.3717470" target="_blank"><i>10.1145/3689031.3717470</i></a></li></ul>
