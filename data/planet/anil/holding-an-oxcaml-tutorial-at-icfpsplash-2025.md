---
title: Holding an OxCaml tutorial at ICFP/SPLASH 2025
description: Tutorial at ICFP 2025 on OxCaml extensions for performance engineering
  with modes and locals.
url: https://anil.recoil.org/notes/icfp25-oxcaml
date: 2025-10-06T00:00:00-00:00
preview_image: https://anil.recoil.org/images/oxcaml-codespace.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>This is part 2 of 5 of a <a href="https://anil.recoil.org/notes/icfp25">series</a> of posts<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> about ICFP 2025.</p>
<p>Several extensions to "oxidize" OCaml (Rust performancew with ML ergonomics!) have been developing rapidly
in a <a href="https://github.com/oxcaml/oxcaml">fork</a> called
<a href="https://oxcaml.org">OxCaml</a>. I helped an intrepid crew from Jane Street,
IIT-M, Tarides, Brown and Cambridge pull together a really fun tutorial in ICFP
2025 that you can try out too!  <strong>TL;DR:</strong> Work through the online
<a href="https://gavinleroy.com/oxcaml-tutorial-icfp25/">slides</a>, try the
<a href="https://github.com/oxcaml/tutorial-icfp25">activities</a>, and take the
<a href="https://gavinleroy.com/oxcaml-icfp-activity/">quiz</a> to give us feedback.</p>
<p><a href="https://github.com/oxcaml/tutorial-icfp25"> <img src="https://anil.recoil.org/images/oxcaml-codespace.webp" alt="%c" title="Just click on the tutorial repo to get an online environment"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#where-oxcaml-came-from" class="anchor" aria-hidden="true"></a>Where OxCaml came from</h2>
<p>If you watch <a href="https://github.com/yminsky">Yaron Minsky</a>'s talk about <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">moving to OCaml 5</a>, towards the end he discusses the language
extensions Jane Street needs to eke out every ounce of performance while
writing code in a safe, functional OCaml style. These extensions currently materialise in
a forked version of the mainline OCaml compiler.  Earlier in the year, a bunch
of us from Cambridge and Tarides <a href="https://tarides.com/blog/2025-07-09-introducing-jane-street-s-oxcaml-branch/">helped</a>
Jane Street get the first public release out the door, and to setup opam
repositories and tutorials for this fork to be developed <a href="https://github.com/oxcaml/oxcaml">in the open</a>.</p>
<p><a href="https://blog.janestreet.com/introducing-oxcaml/"> <img src="https://anil.recoil.org/images/oxcaml-release-summer25.webp" alt="%c" title="An unruly crowd shove an oxidising compiler out the door on a hot summer day"> </a></p>
<p><a href="https://github.com/yminsky">Yaron Minsky</a> <a href="https://bsky.app/profile/yminsky.bsky.social/post/3lrimpjimjs2w">summarised</a> the OxCaml release thusly:</p>
<blockquote>
<p>OxCaml's extensions make OCaml a better language for performance engineering.
It also supports data race free parallel programming, and a bunch of other
goodies.</p>
<p>OxCaml is in an interesting spot. It's an experimental language, whose
extensions will change quickly and mercilessly.</p>
<p>But it's also a production quality compiler. Indeed it's the compiler we use
in production everyday.</p>
<p>So why are we doing this? Our primary hope is to use this to build awareness
about our work, and to help pave the way for getting these extensions
upstreamed to mainline OCaml.</p>
<p>But we're also interested in building adoption among enthusiasts and
researchers who don't mind working with a language that is changing quickly
under their feet. We think there's a ton to learn from the collaboration.
<cite>-- <a href="https://bsky.app/profile/yminsky.bsky.social/post/3lrimpjimjs2w">Yaron Minsky on BlueSky</a>, June 2025</cite></p>
</blockquote>
<p>So the status in the summer of 2025 was that there existed an open source code
drop of OxCaml, but it had a lot of rough edges and it would take some effort
to make it usable outside of the walls of Jane Street.</p>
<h2><a href="https://anil.recoil.org/news.xml#creating-an-oxcaml-tutorial-from-outside-jane-street" class="anchor" aria-hidden="true"></a>Creating an OxCaml tutorial from outside Jane Street</h2>
<p>The language extensions in OxCaml fall into three broad categories of features that:</p>
<ul>
<li>are upstreamable to OCaml, like <a href="https://discuss.ocaml.org/t/ocaml-5-4-0-released/17365#p-73063-labelled-tuples-1">labelled tuples</a> and <a href="https://tarides.com/blog/2025-10-10-ocaml-5-4-release-new-features-fixes-and-more/">immutable arrays</a></li>
<li>are still moving targets but candidates for upstreaming later (like <a href="https://www.dra27.uk/blog/platform/2025/10/18/icfp-2025.html">local modes</a>)</li>
<li>Jane Street specific extensions which are unlikely to ever make it upstream (like <a href="https://github.com/oxcaml/oxcaml/pull/4826">block indices for direct record access</a>)</li>
</ul>
<p>Right now, all of these are all developed in the OxCaml <a href="https://github.com/oxcaml/oxcaml">monorepo</a> by a
growing number of developers within Jane Street.  Without access to the
internal Jane Street developer resources, <a href="https://kcsrk.info">KC Sivaramakrishnan</a> and I started to work with
<a href="https://richarde.dev/">Richard Eisenberg</a> and <a href="https://tyconmismatch.com/code.html">Chris Casinghino</a> to submit a tutorial proposal to ICFP; there's no
better way to learn something than a deadline looming over our heads!</p>
<p>Writing the tutorial proved much more difficult than I'd expected, as OxCaml
has a very <a href="https://oxcaml.org/documentation/">large number</a> of extensions
(such as modes and kinds) that are not only evolving fast, but also sometimes
interact differently in combination. It not only requires learning the compiler extensions,
but also keeping up with the <a href="https://github.com/oxcaml/opam-repository">OxBase libraries</a> that
compose them into usable interfaces. <a href="https://kcsrk.info">KC Sivaramakrishnan</a> and I struggled to pack in all the
different concepts; I spent a whole weekend just attempting to get a parallel quicksort
compiling!</p>
<p>Luckily, <a href="https://richarde.dev/">Richard Eisenberg</a> had the idea to bring in experts in programming language
pedagogy in the form of <a href="https://cs.brown.edu/~sk/">Shriram Krishnamurthi</a>, <a href="https://willcrichton.net/">Will Crichton</a> and <a href="https://gavinleroy.com/">Gavin Gray</a> to save the day.
Gavin completely redesigned our slides to radically simplify the examples
(beginning with a <code>gensym</code> rather than a sorting algortithm), and <em>also</em>
designed a quiz to test user knowledge before and after. <a href="https://thenumb.at/">Max Slater</a>, <a href="https://github.com/mgndv">Megan Del Vecchio</a> and
<a href="https://www.linkedin.com/in/nadia-razek">Nadia Razek</a> from Jane Street also leapt in to give us the inside line on new
developments.</p>
<p>This collaboration made for a nice split in efforts; <a href="https://kcsrk.info">KC Sivaramakrishnan</a>, <a href="https://patrick.sirref.org">Patrick Ferris</a> and I
focussed on the longer handson activity <a href="https://github.com/oxcaml/tutorial-icfp25/tree/main/handson_activity">examples</a>, while Gavin finished the <a href="https://slipshow.readthedocs.io/en/stable/">Slipshow</a>-based <a href="https://gavinleroy.com/oxcaml-tutorial-icfp25/">slides</a> to deliver the presentation itself.  I got a <a href="https://github.com/oxcaml/tutorial-icfp25">OxCaml GitHub DevContainer</a> working that allowed participants to spin up a full OxCaml environment in just a few minutes, to ensure that as many people could participate during the conference as possible.</p>
<h2><a href="https://anil.recoil.org/news.xml#of-course-we-had-to-release-a-new-compiler-the-night-before-right" class="anchor" aria-hidden="true"></a>Of course we had to release a new compiler the night before... right?</h2>
<p>Some excitement ensued when we realised that there hadn't been a public release
of the OxCaml packages since our first public release earlier in the year! Meanwhile,
hundreds of improvements had accumulated upstream, including a number of significant
interface and type system changes. It seemed a little regressive
to present an out-of-date version of the tutorial to a demanding ICFP audience.</p>
<p>So, in a late night call after we arrived in Singapore, with <a href="https://icfp25.sigplan.org/profile/dianakalinichenko">Diana Kalichenko</a> working
tirelessly on compilation fixes from New York, we refreshed all the tutorial
examples and <a href="https://github.com/oxcaml/opam-repository/pull/18">released the latest minus19
compiler</a> version of the
compiler with four months of developments! We fixed the compilation
problems that resulted in our slides, and the new Devcontainer images finally rebuilt
around 10 seconds before the tutorial started. No sweat.</p>
<p><img src="https://anil.recoil.org/images/icfp-4.webp" alt="%c" title="Gavin, KC and me look to the heavens for inspiration to get the tutorial working"></p>
<h2><a href="https://anil.recoil.org/news.xml#the-tutorial-day-arrives" class="anchor" aria-hidden="true"></a>The tutorial day arrives</h2>
<p>The tutorial itself held at NUS went fantastically! Both sessions were completely full,
with participants online as well (Edwin Torok in particular gave all of it a thorough
spin on Discord, thanks!).</p>
<p>The broad sweep of audience feedback centred around stuff like this:</p>
<ul>
<li>The interaction between portable and non-portable functions, which stemmed from a confusion around the fact that Base functions are annotated with modes, but the OCaml stdlib is not. The answer right now is to always use Base with OxCaml.</li>
<li>Whether exclaves can be used to allocate in the caller caller's region or not. Exclaves are transitive, which lets you build this.</li>
<li>What stops the Capsule API from leaking the access keys to outside the interface? The answer is that other modes (like local) work together to give safety to this aspect of the interface. It's difficult to only use one of the modes axes in isolation in a real interface.</li>
<li>Are OxCaml annotations erasable so that the programs are runnable using OCaml? Answer is "mostly erasable".</li>
<li>Are <code>local</code> and <code>stack_</code> inferred? The compiler does this analysis by default and will locally allocate when possible, but it wasn't clear to the tutorial attendees that this is the case.</li>
</ul>
<p><img src="https://anil.recoil.org/images/icfp-1.webp" alt="%c" title="Full tutorial room at the NUS computing department with an engaged audience!"></p>
<h2><a href="https://anil.recoil.org/news.xml#should-you-use-oxcaml-right-now" class="anchor" aria-hidden="true"></a>Should you use OxCaml right now?</h2>
<p>The answer depends on if you want to get into compiler and language design or
not!  The number of mode axes in the OxCaml compiler are growing rapidly as
more usecases are covered, so we'll clearly need to develop this tutorial
further, and the "final form" of OxCaml is by no means stable yet. Jane Street
iterates quickly on language changes since they control all the code in their monorepo
that uses it, and have extensive production engineering infrastructure. From the outside,
it'll be hard to keep a large codebase in sync... for now.</p>
<p>My high-level takeaway from discussions with the developers is that there's
another 12-24 months of active language evolution left, so it'll continue to be
a moving target for a while. But some features like locals have been around for
longer than others features and are more stable. Knowing this
really helps to plan out how I'm going to use OxCaml "from the outside".</p>
<p>It's also reassuring to see that Jane Street is serious about investing in
education resources for the language as well. <a href="https://github.com/yminsky">Yaron Minsky</a> <a href="https://bsky.app/profile/yminsky.bsky.social/post/3m3tmp56yzc2k">posts</a>:</p>
<blockquote>
<p>We've had an exciting couple of weeks full of opportunities to teach people about the exciting (and mildly bewildering) features of OxCaml.
And...we're looking to hire an experienced educator to help us in this work. Please share this with anyone you think might be a good fit!
<cite>-- <a href="https://bsky.app/profile/yminsky.bsky.social/post/3m3tmp56yzc2k">Yaron Minsky on Bluesky</a>, Oct 2025</cite></p>
</blockquote>
<p><img src="https://anil.recoil.org/images/icfp-3.webp" alt="%c" title="There were a healthy contingent of Jane Street OxCaml developers to answer questions as well."></p>
<p>If you want to have a go at the tutorial and quiz yourself, then it's all still
open for participation! Follow the
<a href="https://gavinleroy.com/oxcaml-tutorial-icfp25/">slides</a> and then take the
<a href="https://gavinleroy.com/oxcaml-icfp-activity/">quiz</a>. And most importantly,
share your improbable stunts online so we can see what's going on. I'm hacking on a
<a href="https://anil.recoil.org/notes/icfp25-post-posix">io_uring oxhttpserver</a> myself, and I heard rumours that <a href="https://kcsrk.info">KC Sivaramakrishnan</a>
has been peering into the eBPF sources...</p>

<div class="footnotes"><ol><li><p></p><p>See also in the <a href="https://anil.recoil.org/notes/icfp25">ICFP25</a> series: <a href="https://anil.recoil.org/notes/icfp25-propl">chairing PROPL25</a>, the <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a>, <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">multicore at Jane Street and Docker</a>, <a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX IO</a> and <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">what I learnt</a>.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy (2025). What I learnt at ICFP/SPLASH 2025 about OCaml, Hazel and FP. <a href="https://doi.org/10.59350/w1jvt-8qc58" target="_blank"><i>10.59350/w1jvt-8qc58</i></a></li>
<li>Madhavapeddy (2025). It's time to go post-POSIX at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/mch1m-8a030" target="_blank"><i>10.59350/mch1m-8a030</i></a></li>
<li>Madhavapeddy (2025). A Roundup of ICFP/SPLASH 2025 happenings. <a href="https://doi.org/10.59350/4jf5k-01n91" target="_blank"><i>10.59350/4jf5k-01n91</i></a></li>
<li>Madhavapeddy (2025). Programming for the Planet at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/hasmq-vj807" target="_blank"><i>10.59350/hasmq-vj807</i></a></li>
<li>Madhavapeddy (2025). Jane Street and Docker on moving to OCaml 5 at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/3jkaq-d3398" target="_blank"><i>10.59350/3jkaq-d3398</i></a></li></ul>
