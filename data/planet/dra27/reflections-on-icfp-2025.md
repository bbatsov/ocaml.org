---
title: Reflections on ICFP 2025
description: "I spent last week at ICFP 2025. A nice (if exhausting!) week, as ever.
  Amusingly, the most reflections were actually sparked by Yaron\u2019s talk which
  was right at the end (you can see the talk itself on YouTube)."
url: https://www.dra27.uk/blog/platform/2025/10/18/icfp-2025.html
date: 2025-10-18T00:00:00-00:00
preview_image:
authors:
- ""
source:
---

<p>I spent last week at <a href="https://icfp25.sigplan.org/">ICFP 2025</a>. A nice (if
exhausting!) week, as ever. Amusingly, the most reflections were actually
sparked by <a href="https://conf.researchr.org/details/icfp-splash-2025/rebase-2025-papers/2/The-Saga-of-Multicore-OCaml">Yaron’s talk</a>
which was right at the end (you can see the talk itself <a href="https://www.youtube.com/watch?v=UI1wApT2t1w&amp;list=PLyrlk8Xaylp5ihrTVeOSaylaBe4ZORkSI&amp;t=20694s">on YouTube</a>).</p>

<p>I’ve been involved from both a Tarides perspective and from general day-to-day
upstream work on the OCaml runtime with some of the experiences Jane Street had
switching to OCaml 5 (because they’re using <a href="https://oxcaml.org">OxCaml</a>, you’ll
more often hear this referred to as “Runtime 5”, but it essentially means
Multicore OCaml). It’s interesting to reflect on the decisions we made when
merging Multicore OCaml in the light of these subsequent experiences not, of
course, as a navel-gazing exercise in the benefits of hindsight, but in terms of
what we can potentially learn for the road towards OxCaml becoming OCaml.</p>

<p><span>Unsurprisingly</span>, there was <em>tons</em> of preparation, planning, and hard work leading
up to opening and merging of Multicore in <a href="https://github.com/ocaml/ocaml/pull/10831">ocaml/ocaml#10831</a>
in January 2022. One of the early ideas was to merge just the runtime changes as
a separate runtime, leaving all the language changes to a subsequent update. The
main thing here would have been to upstream the immense changes to the allocator
and garbage collector along with the domains and fibers machinery, while not yet
exposing it. I remember the concern being that having essentially a runtime
variant (not unlike the debug runtime) might lead to very slow uptake at
actually testing it and possibly a maintenance burden. i.e. we were concerned at
maintaining two runtimes. This would probably have resulted in something like
OCaml 4.15.0, with an experimental <em>official</em> multicore-aware runtime.</p>

<p>The decisions moved on to being more, “no, it’s all or nothing - let’s take it”.
From there gradually we moved more towards this being OCaml 5.00.0; a major
version bump. The previous rebase of Multicore had been on 4.12, and that had
intentionally separated off the concept of effects to a separate version, as
this had the benefit that the surface syntax (especially where keywords were
concerned) was unaltered. During the various discussions, there was (slightly
unexpected!) enthusiasm to go the whole hog, and bring the effect system in as
well, but leaving the changing of the surface syntax to a subsequent release.
That this would be OCaml 5 was cemented, and we had a plan.</p>

<p>As a side-note, bumping the major release comes with a few other things. In
particular, it was suggested we should deal with the accumulation of
deprecations in the Standard Library, finally removing the unsafe string mode,
and so forth. Potential risks, but the practical idea was that programs which
were being updated to work with OCaml 5 which still hadn’t updated to these, in
some cases, very old deprecations would have bigger worries.</p>

<p>During 2022, until the release of OCaml 5.0 on 15 December, I had the role of
release manager for all of Tarides’ efforts towards OCaml 5. The simile between
OCaml 4 / OCaml 5 and Python 2 / Python 3 weighed heavily on a lot of what we
did, coupled with the very real fear that we <em>might</em> still find a fundamental
problem that could cause the multicore change to need to be reverted, with more
work required. A key thing which was introduced was that apart from those
deprecations, and a few bits of installation tidying, we froze <em>all</em> other work
on OCaml during the OCaml 5.0 dev cycle. This was such a big deal, we named it
the “Sequential Glaciation”, and it meant an unprecedented level of
compatibility for OCaml 4.14 programs when running on OCaml 5.0. Essentially, it
meant that <em>any</em> program which compiled without warnings on OCaml 4.14 should
compile without change on OCaml 5.0 (with some corner cases for Standard Library
replacements, etc.), an unusual level of commitment.</p>

<p>This gave an escape hatch: it meant that <em>any</em> code could be being tested on
OCaml 5 to check for performance regressions.</p>

<p>So we invested a phenomenal amount of engineering effort into ensuring that the
public OCaml ecosystem was compatible at launch - that so-called “sequential”
code (i.e. written for OCaml 4) would successfully execute single-domain on
OCaml 5.</p>

<p><span>And then follows an</span> inadvertent mistake, to bring this back round to Jane
Street’s experience.</p>

<p>OCaml 5.1’s Changelog includes <a href="https://github.com/ocaml/ocaml/blob/5.1.0/Changes#L21">25 Standard Library entries</a>
of which 17 add new functions, 5 are non-trivial performance improvements and,
crucially, three <em>breaking changes</em> and non-trivial bug-fixes. That’s the
Standard Library alone!</p>

<p>No such escape hatch now: if you’re only now, as Jane Street were, only just
able to <em>think</em> about investigating OCaml 5 (also bear in mind that by OCaml 5.1
we still hadn’t restored feature parity with OCaml 4.14), there’s no easy way
back if you do hit a problem. Even if you just upgrade a codebase to OCaml 5.1
without using domains and effects, it’s very hard, bordering on impossible, to
go back to 4.14.</p>

<p>It’s important not to put this down to the benefits of hindsight. I don’t
remember us ever discussing the idea of actively supporting OCaml 5 (i.e. with
all the front-end changes included) still being able to run on the OCaml 4.14
runtime. It’s the kind of suggestion I would have expected in a core developers’
meeting to say, “yeah, I can do that”, but to have met with some resistance from
others at the idea of us all having to maintain it. 🙂</p>

<p>However, from a technical perspective, it really wasn’t as difficult as I’d have
expected. In the middle of 2023, I prepared a branch of OCaml 5 for Jane Street
which replayed the OCaml 5.x changes, skipping all of the alterations to the
runtime (a snapshot of it sits in branch <a href="https://github.com/dra27/ocaml/tree/backported-trunk-to-5.1-20230929">backported-trunk-to-5.1-20230929</a>
on my OCaml fork). The work in this formed the basis for Jane Street’s
OxCaml-on-runtime4 changes (it wasn’t called OxCaml back then, of course) - and
when compared with its basis commit of <a href="https://github.com/ocaml/ocaml/commit/5000b93cad2d126d5949c952eb9a93e47bf901a8">5000b93cad</a>,
there’s no OCaml 5 runtime changes in there, just various minor things added to
the 4.x runtime up to its release and a smattering of non-multicore related
things.</p>

<p>With this work, Jane Street managed to get their internal compiler to being the
OCaml 5.1 frontend but still running on their altered version of “Runtime 4”. As
they started investigating switching all the exciting things in OxCaml to be
available on “Runtime 5”, they also considered trying the runtime-variant
approach we’d ruled out earlier in the merge. Turns out our ruling it out was
correct - it didn’t work easily for them, as it requires too many extra flags
and things to be begin plumbed through build systems and so forth.</p>

<p>The trick was to make it a <em>configuration</em> option (which still exists in OxCaml
at the moment, although it’s on notice!). That provides a lot of simplification
in the deployment (we never have to build both runtimes at once, for example),
and making that “escape hatch” slightly more awkward to get to is probably no
bad thing, either. For a very large codebase, I can certainly see the value it
could have provided.</p>

<p>Maybe that’s also something we should look to as OxCaml’s development continues:
focusing in the ecosystem on being able to have that escape hatch back to OCaml
itself, possibly more for benchmarking comparison, than anything else. Although
so much of OxCaml is visible through the front-end (the modes systems, et al),
stack allocation, unboxed types, SIMD, and so forth are all also runtime
changes, and perhaps we need to be considering these escape hatches, <em>before</em> as
well as during any upstreaming effort. If nothing else, it would help
benchmarking. It <em>might</em> also be interesting for things like data-race freedom,
which don’t affect the runtime, to be able to have programs which are data-race
free on OCaml 5.x, even if the final form of DRF in OxCaml isn’t yet known. And
maybe it’ll be a bit too hard to maintain, but I’m always up for a challenge,
and musing is musing!</p>

<p><span>Sticking with OxCaml</span>, I was particularly interested, both from hallway
conversations and from talks, to muse on how the modes system in OxCaml might
make its way upstream. It’s a known design decision of <code class="language-plaintext highlighter-rouge">ocamlopt</code>, which
Richard alluded to in his <a href="https://conf.researchr.org/details/icfp-splash-2025/haskellsymp-2025-papers/2/-A-Tale-of-Two-Lambdas-A-Haskeller-s-Journey-into-OCaml">Haskell (!!) keynote</a>,
that it prefers predictability and a certain measure of simplicity over a
kitchen-sink of optimisations. I want to be able to write high-performance
program in OCaml, and I think others should be able to as well, but here are
three possible takes on why we might end up with a lot of front-end “complexity”
(I put it in quotes, because I tire of “complexity” often begin used as an
attack on solutions to problems) of OxCaml even without performance in mind:</p>

<ol>
  <li>
    <p>Last year at ICFP, we saw in <a href="https://icfp24.sigplan.org/details/icfp-2024-papers/19/Oxidizing-OCaml-with-Modal-Memory-Management">Oxidizing OCaml with Modal Memory Management</a>
the introduction of modes to increase performance by reducing heap
allocation. There’s another neat use of locals, and the regions that come
with them: the same technique allows us to stop accidentally programs like</p>

    <div class="language-ocaml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">let</span> <span class="n">uh_oh</span> <span class="n">file</span> <span class="o">=</span> <span class="nn">In_channel</span><span class="p">.</span><span class="n">with_open_text</span> <span class="n">file</span> <span class="nn">In_channel</span><span class="p">.</span><span class="n">really_input_string</span>
</code></pre></div>    </div>

    <p><em>Performance or no, I think we can all
agree we’d like to write programs where the type checker assures us that not
only are we well-typed but also that we don’t leak resources!</em></p>
  </li>
  <li>
    <p>Thanks to <a href="https://pldi18.sigplan.org/details/pldi-2018-papers/15/Bounding-Data-Races-in-Space-and-Time"><em>Bounding Data Races in Space and Time</em></a>,
OCaml has a much nicer approach to data-races than many languages (certainly
than the C language its runtime sits on!). We can say that OCaml programs
with data races do not “catch fire”, but it’s still chaos! <a href="https://ocaml.org/manual/5.4/api/Stdlib.Atomic.html">Atomics</a>
give us one way of being able to manage this, but we have to actually do it.
The work presented earlier this year in <a href="https://popl25.sigplan.org/details/POPL-2025-popl-research-papers/23/Data-Race-Freedom-la-Mode">Data Race Freedom à la Mode</a>
gives us OxCaml’s <a href="https://github.com/janestreet/portable">capsules</a>… now
we can declare we want to “do it”, and have the type checker ensure we
actually did.</p>

    <p><em>Performance or no, I think we can all agree we’d like to write programs
where the type checker assures us that not only are we well-typed but also
data-race free!</em></p>
  </li>
  <li>
    <p>Thanks to <a href="https://2025.splashcon.org/details/OOPSLA/62/Modal-Effect-Types"><em>Modal Effect Types</em></a>
this year (SPLASH and ICFP ran at the same time this year), we can look
towards typed effects in OCaml.</p>

    <p><em>Performance or no, I think we can all agree
we’d like to write programs where the type checker assures us that not are we
well-typed but also that we handle all our effects!</em></p>
  </li>
</ol>

<p>There’re other approaches to each of these problems as well (indeed, even in
OxCaml, where modal effects are concerned), but I think it’s interesting to see
what’s happening here as not <em>just</em> about high-performance.</p>

<p>That’s enough musing for now… I’ll endeavour to write some more and maybe in a
less musy way about other talks I went to another day!</p>
