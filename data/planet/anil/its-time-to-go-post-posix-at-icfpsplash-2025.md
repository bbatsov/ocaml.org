---
title: It's time to go post-POSIX at ICFP/SPLASH 2025
description: VMIL keynote arguing for post-POSIX shared memory interfaces like io_uring
  in language runtimes for high-performance concurrent computing.
url: https://anil.recoil.org/notes/icfp25-post-posix
date: 2025-10-08T00:00:00-00:00
preview_image: https://anil.recoil.org/images/icfp-38.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>This is part 4 of 5 of a <a href="https://anil.recoil.org/notes/icfp25">series</a> of posts<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> about ICFP 2025.</p>
<p>After the excitement of presenting my <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">Docker experience report</a>, I went straight into giving a <a href="https://youtu.be/tOMF69dP2-I?t=14187">keynote talk</a> at <a href="https://conf.researchr.org/home/icfp-splash-2025/vmil-2025">VMIL
2025</a>.  This talk
bubbled up intrusive thoughts I've had resulting in the past 25 years: every
system I've worked on, ranging from <a href="https://anil.recoil.org/papers/2010-icfp-xen">Xen</a> to
<a href="https://anil.recoil.org/papers/2025-docker-icfp">Docker</a> on all seem to boil down to "make shared memory go
fast".</p>
<p>I'd started to believe it was time for change in the way we approach IO about 12 years ago when I talked about <a href="https://www.youtube.com/watch?v=Ss4pUbq09Lw">wierd IO
behaviour</a> to a packed audience at
FOSDEM, and now I believe it's even more true in 2025.</p>
<p>So I made one key <a href="https://anil.recoil.org/slides/vmil25-keynote.pdf">argument</a> to the audience: it's time to accept that standards
such as POSIX are now holding back the development of good language runtimes,
and we need to embrace the diversity of highly concurrent, shared-memory
interfaces. And unfortunately, there's no portable subset amongst these, and so
this may require a rethink of our frontend language interfaces as well.</p>
<p><a href="https://youtu.be/tOMF69dP2-I?t=14187"> <img src="https://anil.recoil.org/images/icfp-38.webp" alt="%c" title="The leaning tower of operating system layers"> </a></p>
<p>After explaining <a href="https://github.com/ocaml-multicore/ocaml-uring">io_uring</a> on
Linux, and then the Windows and macOS variants, I showed how we're trying to
support these in our OCaml 5 <a href="https://github.com/ocaml-multicore/eio">Eio library</a>. While Eio has plenty of
really <a href="https://tarides.com/blog/2024-03-20-eio-1-0-release-introducing-a-new-effects-based-i-o-library-for-ocaml/">cool features</a>,
the defining one for me is that it abstracts away IO operations (like
<a href="https://ocaml.org/p/eio/1.3/doc/eio/Eio/Flow/index.html#val-copy">Flow.copy</a>)
sufficiently that the backend can do highly parallel dispatch to the kernel
over shared memory interfaces like uring.</p>
<p>Crucially, we treat the shared memory interface in Eio as the first-class citizen,
with the POSIX(ish) backends relegated to a compatibility role.  This way, future
interfaces to the programmer can be "parallel first".</p>
<h2><a href="https://anil.recoil.org/news.xml#posix-shouldnt-be-relegated-just-yet" class="anchor" aria-hidden="true"></a>POSIX shouldn't be relegated just yet</h2>
<p>The discussion with the audience after the talk was just fantastic. <a href="https://www.humprog.org/~stephen/">Stephen Kell</a>,
who has
<a href="https://www.humprog.org/~stephen/blog/devel/native-debugging-part-2.html">thought</a>
<a href="https://www.humprog.org/~stephen/blog/research/seven-type-sins.html">deeply</a>
about the interaction between the kernel and userspace asked whether I was
being too harsh on POSIX, which has served us faithfully for many years. And
indeed, I agree with Stephen! POSIX gives us a fine boot layer, and a fine
interaction layer (for terminals anyway), and a great single threaded
interface. Where it falls over is the highly concurrent and parallel world of
high performance computing, where must align our data paths and not have any
interference from third party code.</p>
<p>So perhaps we need to restructure runtimes to explicitly have a "boot phase"
(POSIX) where they are establishing their resources, and then switch into a
"steady phase" (uring and friends) where they are blasting data at high speeds.
These are really quite distinct modes of operation, and both are useful. Note
that "high speed" here applies to embedded systems as well; if I do less work
on those systems in the CPU, then I'll get better energy usage, so low-overhead
mechanisms like uring are useful there too.</p>
<p>One other thought I had from Stephen's excellent question was the important
role of POSIX in portability for many years. Moving forward into the next
decade, what standards body will help developers write portable software for
all these different IO interfaces? It seems inevitable that this will fragment
into per-language specifications instead of the operating system (or C-level)
interfaces we've had for the past 50 years.</p>
<h2><a href="https://anil.recoil.org/news.xml#building-more-uring-based-low-level-examples" class="anchor" aria-hidden="true"></a>Building more uring-based low-level examples</h2>
<p>I was also slightly surprised by how few people had used <code>io_uring</code>, in a room
full of language VM developers. One of the most useful things I did when
developing the OCaml <a href="https://github.com/ocaml-multicore/ocaml-uring">uring
bindings</a> was to build a
<a href="https://github.com/ocaml-multicore/ocaml-uring/blob/main/tests/urcp_lib.ml">parallel file
cp</a>,
but I never got around to any real networking code.</p>
<p>So I've started to build a "raw" OCaml 5 + uring HTTP server that is completely
non-portable, but serves to show how it works at the lowest level. This should
also give us a nice benchmark against which to test higher level interfaces.  My plan
is to make this work with OCaml 5 first, and subsequently add <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml support</a>. <a href="https://toao.com">Sadiq Jaffer</a> pointed
me to the magic <a href="https://oxcaml.org/documentation/stack-allocation/intro/">caml_alloc_local</a> FFI function
added in OxCaml that allows allocation directly into the OCaml stack from C, which should be all I need to
make the shared memory interface never allocate into the heap.</p>
<p><a href="https://patrick.sirref.org">Patrick Ferris</a> also spent <a href="https://patrick.sirref.org/icfp-2025/index.xml">some ICFP time</a> hacking on integrating OxCaml into the OCaml uring bindings:</p>
<blockquote>
<p>The idea being that a completion queue entry (a notification that some operation has completed) could be fully represented using 64 bits (two 32-bit, unboxed values).
<cite>-- <a href="https://patrick.sirref.org/icfp-2025/index.xml">Patrick at ICFP 2025</a></cite></p>
</blockquote>
<p>Having had my regular morning coffee with <a href="https://en.wikipedia.org/wiki/Simon_Peyton_Jones">Simon Peyton Jones</a> back in Cambridge, he then pointed out
that Haskell <em>also</em> has a uring <a href="https://gitlab.haskell.org/ghc/ghc/-/issues/18390">backend
PR</a> lingering for years, and
so perhaps we should do the same exercise there too, to understnad it all! I
can't say no to Simon, but if a Haskell expert is interested please do get in
touch so I don't have to inflict my OCaml-style Haskell on the world...</p>
<p><small class="credits"> <em>3rd Nov 2025: Add link to Patrick's uring bindings.</em> </small></p>

<div class="footnotes"><ol><li><p></p><p>See also in the <a href="https://anil.recoil.org/notes/icfp25">ICFP25</a> series: <a href="https://anil.recoil.org/notes/icfp25-propl">chairing PROPL25</a>, the <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a>, <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">multicore at Jane Street and Docker</a>, <a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX IO</a> and <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">what I learnt</a>.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy et al (2025). Functional Networking for Millions of Docker Desktops. <a href="https://doi.org/10.1145/3747525" target="_blank"><i>10.1145/3747525</i></a></li>
<li>Madhavapeddy (2025). Holding an OxCaml tutorial at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/55bc5-x4p75" target="_blank"><i>10.59350/55bc5-x4p75</i></a></li>
<li>Madhavapeddy (2025). What I learnt at ICFP/SPLASH 2025 about OCaml, Hazel and FP. <a href="https://doi.org/10.59350/w1jvt-8qc58" target="_blank"><i>10.59350/w1jvt-8qc58</i></a></li>
<li>Scott et al (2010). Using functional programming within an industrial product group: perspectives and perceptions. ACM. <a href="https://doi.org/10.1145/1863543.1863557" target="_blank"><i>10.1145/1863543.1863557</i></a></li>
<li>Madhavapeddy (2025). A Roundup of ICFP/SPLASH 2025 happenings. <a href="https://doi.org/10.59350/4jf5k-01n91" target="_blank"><i>10.59350/4jf5k-01n91</i></a></li>
<li>Madhavapeddy (2025). Programming for the Planet at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/hasmq-vj807" target="_blank"><i>10.59350/hasmq-vj807</i></a></li>
<li>Madhavapeddy (2025). Jane Street and Docker on moving to OCaml 5 at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/3jkaq-d3398" target="_blank"><i>10.59350/3jkaq-d3398</i></a></li></ul>
