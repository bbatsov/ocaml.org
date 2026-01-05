---
title: Jane Street and Docker on moving to OCaml 5 at ICFP/SPLASH 2025
description: Jane Street's production deployment of OCaml 5 and Docker's migration
  to direct-style programming with Eio presented at ICFP.
url: https://anil.recoil.org/notes/icfp25-ocaml5-js-docker
date: 2025-10-07T00:00:00-00:00
preview_image: https://anil.recoil.org/images/icfp-29.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>This is part 3 of 5 of a <a href="https://anil.recoil.org/notes/icfp25">series</a> of posts<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> about ICFP 2025.</p>
<p>It's been about six years since we wrote the papers on <a href="https://anil.recoil.org/papers/2020-icfp-retropar">parallelism</a> and <a href="https://anil.recoil.org/papers/2021-pldi-retroeff">effects</a>,
and four years since we helped to <a href="https://anil.recoil.org/notes/recapping-ocaml-22">release</a>
upstream OCaml 5.0 with multicore support, a <a href="https://tarides.com/blog/2023-03-02-the-journey-to-ocaml-multicore-bringing-big-ideas-to-life/">mammoth
effort</a>
that took up years of work for my <a href="https://anil.recoil.org/projects/ocamllabs">OCaml Labs</a> and
<a href="https://tarides.com">Tarides</a> crew. After the release came out, I focussed on
building applications using OCaml 5 for my own work on <a href="https://anil.recoil.org/projects/plancomp">planetary computing</a>, for example on <em>using</em> the new features with the
fledgling <a href="https://anil.recoil.org/papers/2023-ocaml-eio">Eio library</a> to get some experience with
direct-style OCaml programming.</p>
<p>Meanwhile, big OCaml users have also been adapting their codebases to shift
from OCaml 4 to 5. Jane Street have expanded their tools and compiler team and
driven through their <a href="https://anil.recoil.org/news.xml#the-path-to-ocaml-5-in-production-at-jane-street">production switch</a> to the multicore runtime, and Docker for
Desktop is progressing with <a href="https://anil.recoil.org/news.xml#functional-networking-at-docker">their switch</a> to direct-style code via Eio for
hundreds of millions of users! Read on to learn more...</p>
<h2><a href="https://anil.recoil.org/news.xml#the-path-to-ocaml-5-in-production-at-jane-street" class="anchor" aria-hidden="true"></a>The Path to OCaml 5 in Production at Jane Street</h2>
<p>Although it was the last talk of the entire conference week, I'm discussing
this first as it was the most exciting thing I learnt at ICFP!  At the
<a href="https://conf.researchr.org/home/icfp-splash-2025/rebase-2025">REBASE</a> <sup><a href="https://anil.recoil.org/news.xml#fn:2" class="footnote">[2]</a></sup> workshop,
<a href="https://github.com/yminsky">Yaron Minsky</a> <a href="https://www.youtube.com/live/UI1wApT2t1w?t=20700s">announced</a> that Jane Street's production servers are now running
on the OCaml 5 runtime!</p>
<p><a href="https://www.youtube.com/live/UI1wApT2t1w?t=20700s"> <img src="https://anil.recoil.org/images/icfp-29.webp" alt="%c" title="Yaron Minsky showing the timeline of Jane Street's recent OCaml usage"> </a></p>
<p>That was the good news (our runtime is trading trillions of dollars, wow). The
bad news is that there was a bumpy road internally for Jane Street to go from
the version that was first released (OCaml 5.0 on <a href="https://ocaml.org/releases">Dec
2022</a>) to their current tree.  Ron gave a really
good roundup of some of the effort that went into release engineering their
internal rollout. Since the entire OCaml runtime was
<a href="https://github.com/ocaml/ocaml/pull/10831">rewritten</a> as part of the multicore
runtime upgrade, Jane Street encountered <a href="https://github.com/ocaml/ocaml/pulls?q=is:pr%20pacing%20label:Performance%20is:closed">GC pacing
issues</a>
and other unexpected changes in resource usage as a result of the new runtime
behaviours.  This took some significant design and engineering effort and to fix, which Ron
covers in detail in <a href="https://www.youtube.com/live/UI1wApT2t1w?t=20700s">his talk</a>.</p>
<h3><a href="https://anil.recoil.org/news.xml#diagnosing-the-ocaml-5-performance-bumps" class="anchor" aria-hidden="true"></a>Diagnosing the OCaml 5 performance bumps</h3>
<p>So what was the root cause of this bumpiness? I think a lot of it was just
normal release engineering; we clearly
<a href="https://discuss.ocaml.org/t/ocaml-5-0-first-normal-alpha-release/10216">signalled</a>
when releasing OCaml 5.0 that it was not yet feature complete, and that the
4.x runtime would continue to be supported for some years.</p>
<blockquote>
<p>The developer team released OCaml 5.0.0 in December 2022. OCaml 5.x features
a full rewrite of its runtime system for shared-memory parallel programming
using domains and native support for concurrent programming using effect
handlers.</p>
<p>Owing to the large number of changes, especially to the garbage collector,
OCaml 4.14 (the final release in the OCaml 4.x series, originally released in
March 2022) remains supported for the time being. Maintainers of existing
codebases are strongly encouraged to evaluate OCaml 5.x and to report any
performance degradations on our issue tracker.
<cite>-- <a href="https://github.com/ocaml/ocaml/blob/85cd5fd3dc0c1763926378a571ef215ce9512908/README.adoc">ocaml/ocaml README</a></cite></p>
</blockquote>
<p>The latest OCaml 5.4.0 release that came out just a <a href="https://discuss.ocaml.org/t/ocaml-5-4-0-released/17365">couple of
weeks</a> ago is the first
release with full feature parity with the OCaml 4.x LTS branch. Features such
as <a href="https://tarides.com/blog/2025-03-06-feature-parity-series-statmemprof-returns/">statmemprof</a>,
the <a href="https://tarides.com/blog/2025-01-15-using-clang-cl-with-ocaml-5/">MSVC port</a>, <a href="https://tarides.com/blog/2024-09-11-feature-parity-series-compaction-is-back/">GC compaction</a>, <a href="https://tarides.com/blog/2024-08-21-how-tsan-makes-ocaml-better-data-races-caught-and-fixed/">thread sanitizer</a>, <a href="https://github.com/ocaml/ocaml/pull/11418">RISC-V</a> and
<a href="https://github.com/ocaml/ocaml/pull/11712">S390X</a> architecture support all had to be engineered back in. OCaml development
in recent years has been very developer intensive because of the need to not only
reintroduce these features, and <em>also</em> keep up with the torrent of new features being introduced to OCaml 5.x.
All in all, I think it's been a very successful few years for the language to have kept steadily improving!</p>
<h3><a href="https://anil.recoil.org/news.xml#should-we-maintain-multiple-language-runtimes-in-ocaml" class="anchor" aria-hidden="true"></a>Should we maintain multiple language runtimes in OCaml?</h3>
<p>However, Ron's talk was an excellent chance for us to reflect on what we might
have done differently with the benefit of hindsight. <a href="https://github.com/dra27">David Allsopp</a> posits that we should
have maintained the OCaml 4 and 5 runtimes simultaneously:</p>
<blockquote>
<p>One of the early ideas was to merge just the runtime changes as a separate
runtime, leaving all the language changes to a subsequent update.  The main
thing here would have been to upstream the immense changes to the allocator
and garbage collector along with the domains and fibers machinery, while not
yet exposing it.</p>
<p>I remember the concern being that having essentially a runtime variant (not
unlike the debug runtime) might lead to very slow uptake at actually testing
it and possibly a maintenance burden.  i.e. we were concerned at maintaining
two runtimes. This would probably have resulted in something like OCaml
4.15.0, with an experimental official multicore-aware runtime.
<cite>-- <a href="https://www.dra27.uk/blog/platform/2025/10/18/icfp-2025.html">Reflections on ICFP 2025</a>, David Allsopp</cite></p>
</blockquote>
<p>I agree with this. Although we weren't sure in 2021 that it would be possible
to have two simultaneous runtimes<sup><a href="https://anil.recoil.org/news.xml#fn:3" class="footnote">[3]</a></sup> it was clear by the time we were
engineering the 5.0 PR that this would be possible. Still, a few years to stabilise
multicore performance <em>vs</em> the decade-old 4.x runtime isn't bad going at all, so I don't have
deep regrets about the approach we did take!</p>
<h3><a href="https://anil.recoil.org/news.xml#we-need-continuous-continuous-performance-engineering" class="anchor" aria-hidden="true"></a>We need continuous continuous performance engineering</h3>
<p>The other area where all of us agreed more effort is necessary is on continuous performance engineering.
In the runup to 2022, <a href="https://github.com/ctk21">Tom Kelly</a>, <a href="https://kcsrk.info">KC Sivaramakrishnan</a> and I set up <a href="https://github.com/ocaml-bench/sandmark">Sandmark</a>, which
was a body of microbenchmarks and some macrobenchmarks. Tarides setup a <a href="https://discuss.ocaml.org/t/ann-sandmark-nightly-benchmarking-as-a-service/10174">continuous benchmarking service</a> around this, and I hosted a bunch of <a href="https://github.com/ocaml-bench/ocaml_bench_scripts#notes-on-hardware-and-os-settings-for-linux-benchmarking">carefully tuned machines</a> with specific BIOS settings in the Cambrige Computer Lab.</p>
<p>Roll on six years, and the maintainence cost of this service becomes clear.
It's quite a bit of effort to maintain the old machines (one is running Ubuntu
16.04! I'm not telling you which one) to keep consistency in previous results.
New machines all come with a proportional configuration effort, and their results
have to be interpreted. Operating systems have to be upgraded, and tuned afresh.
Continuous benchmarking is itself a continuous process of engineering, and should be treated as such!</p>
<p>Our discussions at ICFP centred around the idea that we should not only maintain this
benchmarking infrastructure, but add an incentive to <em>macro</em> projects to submit
representative tests of their performance. Rocq, Why3,
<a href="https://semgrep.dev/blog/2025/upgrading-semgrep-from-ocaml-4-to-ocaml-5/">Semgrep</a>,
or Frama-C, for example, should all have test cases run within this
infrastructure that move beyond microbenchmarks to realistic performance
patterns that will show up issues with GC pacing in a way that microbenchmarks
do not.</p>
<p>The challenge with doing this so far has been the difficulty of getting many of these
big projects to compile on a random OCaml trunk snapshot (essential to test them against
a pre-release OCaml compiler). Solving this will take some thought (particularly around
ppx usage), but the effort seems worthwhile as we move into a new phase of OCaml 5
engineering now that feature parity has been achieved. Stay tuned for more on this from
Tarides!</p>
<h2><a href="https://anil.recoil.org/news.xml#functional-networking-at-docker" class="anchor" aria-hidden="true"></a>Functional Networking at Docker</h2>
<p>I also had the opportunity to share both a retrospective on the work <a href="https://github.com/djs55">Dave Scott</a>
and I have been doing on <a href="https://www.docker.com/products/docker-desktop/">Docker Desktop</a> for some years, and
<em>also</em> the efforts from new OCamlers like <a href="https://patrick.sirref.org">Patrick Ferris</a> and <a href="https://ryan.freumh.org">Ryan Gibb</a> on helping us
to port some aging OCaml code over to OCaml 5.</p>
<p>We got a paper accepted to ICFP on the topic, and so I had a lot of fun
<a href="https://www.youtube.com/watch?v=j84ocjlj1JA&amp;t=12880s">presenting</a>
"<a href="https://anil.recoil.org/papers/2025-docker-icfp">Functional Networking for Millions of Docker Desktops</a>" to the mainline ICFP audience!  I first discussed the
past; how we <a href="https://anil.recoil.org/notes/docker-buys-unikernel-systems">joined Docker</a> and came up with <a href="https://www.docker.com/blog/docker-unikernels-open-source/">HyperKit and
VPNKit</a> to solve
scaling problems that Docker was facing early in its growth.</p>
<p><a href="https://www.youtube.com/watch?v=j84ocjlj1JA&amp;t=12880s"> <img src="https://anil.recoil.org/images/icfp-37.webp" alt="%c" title="Me on stage at ICFP in a verrrry cold venue"> </a></p>
<p>Our experience report makes the broad case for library operating sytems and
functional programming being a good fit, especially with strict languages like
OCaml which offer thin interfaces to the OS interfaces.</p>
<blockquote>
<p>Our use of library-oriented programming to deliver Docker for Desktop is [...] a very
useful way to build the "invisible systems glue" code that is pervasively needed in many systems
programming tasks.</p>
<p>There are an ever-growing number of hardware and software interfaces to
access the outside world, most obviously with GPUs for machine learning workloads but also
FPGAs and new storage and persistent memory devices. These usually require significant
retrofitting to work with existing codebases, and so building translation adapters like VPNkit and
using library VMMs like Hyperkit will become more common in the future.
<cite>-- <a href="https://doi.org/10.1145/3747525">Functional Networking for Millions of Docker Desktops</a>, 2025</cite></p>
</blockquote>
<p>And looking into recent specifics, <a href="https://patrick.sirref.org">Patrick Ferris</a>'s <a href="https://github.com/moby/vpnkit/pull/646">contributions</a> to VPNKit also allow
this codebase to move to OCaml 5, and take advantage of direct-style IO! In a nutshell, it lets
old code like this:</p>
<pre><code class="language-ocaml">module Make_packet_proxy
(I: Mirage_flow.S) (O: Mirage_flow.S) = struct
 let run incoming outgoing =
  let rec loop () =
   I.read incoming &gt;&gt;= function
   | Error err -&gt; fail "%a" I.pp_error err
   | Ok `Eof -&gt; Lwt.return_unit
   | Ok (`Data buf) -&gt; begin
      O.write outgoing buf &gt;&gt;= function
      | Ok () -&gt; loop ()
      | Error err -&gt; fail "%a" O.pp_error err
     end
  in loop ()
</code></pre>
<p>...migrate to direct-style code Eio like this:</p>
<pre><code>module Proxy = struct
 let run incoming outgoing =
  try
    while true do
      Eio.Flow.copy incoming outgoing
    done
  with
  | End_of_file -&gt; ()
  | Write_error err -&gt;
      fail "%a" pp_write_error err
  | Read_error err -&gt;
      fail "%a" pp_read_error err
end
</code></pre>
<p>The old code<sup><a href="https://anil.recoil.org/news.xml#fn:4" class="footnote">[4]</a></sup> had monadic concurrency, functors for parameterising the IO
drivers, and error handling duplicated across OCaml exceptions and the
concurrency monad. The new code uses direct control flow constructs like
<code>while</code>, and also one form of error handling.  This is all still a
work-in-progress, but looking a solid approach with no blockers except hacking
time to get it merged.</p>
<p>Read the <a href="https://doi.org/10.1145/3747525">ICFP paper</a> to learn more about
this, or <a href="https://anil.recoil.org/slides/icfp-docker-25.pdf">browse my slides</a>!  It's exciting to see
production code here get simpler <em>and</em> faster as we move to OCaml 5.  The reasons
why this happens are explored further in my <a href="https://anil.recoil.org/notes/icfp25-post-posix">VMIL keynote talk</a>
that I gave the next day, where I make a case for runtimes focussing on
post-POSIX IO! And beyond that, we have <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml waiting in the wings</a>
for even more performance gains. OCaml is living in exciting times.</p>

<div class="footnotes"><ol><li><p></p><p>See also in the <a href="https://anil.recoil.org/notes/icfp25">ICFP25</a> series: <a href="https://anil.recoil.org/notes/icfp25-propl">chairing PROPL25</a>, the <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a>, <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">multicore at Jane Street and Docker</a>, <a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX IO</a> and <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">what I learnt</a>.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li>
<li><p></p><p>As far as I can tell, REBASE is SPLASH's equivalent of the
venerable <a href="http://cufp.org/">CUFP</a> series that I helped run earlier in the
century.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:2" class="reversefootnote">↩</a><p></p></li>
<li><p></p><p>Our 2020 <a href="https://anil.recoil.org/papers/2020-icfp-retropar">parallelism paper</a> proposed <em>two</em> minor GC
strategies, one of which broke the C FFI and so didn't make the cut in the end
due to the amount of ecosystem churn it would cause.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:3" class="reversefootnote">↩</a><p></p></li>
<li><p></p><p>A fun historical note is that I gave one of the <a href="https://anil.recoil.org/videos/dbd7546a-95d8-40af-b286-3cf930767682">first talks</a> about VPNKit in Jane Street London about a decade ago! Back then we had a deep discussion about whether to use Lwt or Async, and it looks like we'll now meet again via <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml</a>.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:4" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy et al (2025). Functional Networking for Millions of Docker Desktops. <a href="https://doi.org/10.1145/3747525" target="_blank"><i>10.1145/3747525</i></a></li>
<li>Madhavapeddy (2025). Holding an OxCaml tutorial at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/55bc5-x4p75" target="_blank"><i>10.59350/55bc5-x4p75</i></a></li>
<li>Madhavapeddy (2025). What I learnt at ICFP/SPLASH 2025 about OCaml, Hazel and FP. <a href="https://doi.org/10.59350/w1jvt-8qc58" target="_blank"><i>10.59350/w1jvt-8qc58</i></a></li>
<li>Sivaramakrishnan et al (2021). Retrofitting effect handlers onto OCaml. ACM. <a href="https://doi.org/10.1145/3453483.3454039" target="_blank"><i>10.1145/3453483.3454039</i></a></li>
<li>Madhavapeddy (2025). It's time to go post-POSIX at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/mch1m-8a030" target="_blank"><i>10.59350/mch1m-8a030</i></a></li>
<li>Madhavapeddy (2025). A Roundup of ICFP/SPLASH 2025 happenings. <a href="https://doi.org/10.59350/4jf5k-01n91" target="_blank"><i>10.59350/4jf5k-01n91</i></a></li>
<li>Madhavapeddy (2025). Programming for the Planet at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/hasmq-vj807" target="_blank"><i>10.59350/hasmq-vj807</i></a></li>
<li>Sivaramakrishnan et al (2020). Retrofitting parallelism onto OCaml. <a href="https://doi.org/10.1145/3408995" target="_blank"><i>10.1145/3408995</i></a></li></ul>
