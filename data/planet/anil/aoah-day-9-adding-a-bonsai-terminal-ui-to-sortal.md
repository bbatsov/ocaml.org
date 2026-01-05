---
title: 'AoAH Day 9: Adding a Bonsai terminal UI to Sortal'
description: Experimenting with OxCaml's bonsai_term framework for Sortal's terminal
  UI, navigating Eio-Async interoperability challenges through JSON-RPC while discovering
  image-based debugging techniques for terminal applications.
url: https://anil.recoil.org/notes/aoah-2025-9
date: 2025-12-09T00:00:00-00:00
preview_image: https://anil.recoil.org/images/x-bonsai.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After building a reasonably complete <a href="https://anil.recoil.org/notes/aoah-2025-8">Sortal contacts manager</a>,
I decided to try to do a proper job of a terminal user interface. The first
option for a modern UI is something that <a href="https://github.com/yminsky">Yaron Minsky</a> announced last week:
<a href="https://github.com/janestreet/bonsai_term">bonsai_term</a>, which also gives me
a chance to dip into the <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml</a> ecosystem with my agentic hacking!</p>
<h2><a href="https://anil.recoil.org/news.xml#approach-to-using-oxcaml" class="anchor" aria-hidden="true"></a>Approach to using OxCaml</h2>
<p><a href="https://x.com/avsm/status/1994022009978925518"> <img src="https://anil.recoil.org/images/x-bonsai.webp" alt="%rc" title="I did dip my toes into this a couple of weeks ago very quickly"> </a></p>
<p>Switching to the OxCaml compiler is a relatively straightforward process by following the <a href="https://oxcaml.org/get-oxcaml/">installation instructions</a>. There is a custom <a href="https://github.com/oxcaml/opam-repository">oxcaml/opam-repository</a> remote that contains all the bleeding edge packages required.</p>
<p>However, stepping outside of the Jane Street OxCaml package universe and interfacing with bleeding edge upstream OCaml packages does require a minor act of god to succeed. Luckily, <a href="https://github.com/dra27">David Allsopp</a> is one such OCaml god that's been <a href="https://github.com/oxcaml/opam-repository/pull/27">helping me</a> work through a <a href="https://github.com/oxcaml/opam-repository/pull/23">bunch of constraints</a> to get a consistent package universe. This is a topic for a future note, but I just wanted to warn the casual reader that this aspect of OxCaml isn't quite ready for casual use yet: if you want to play with it, then stick with the Jane Street package set for now and you'll make fine progress.</p>
<h3><a href="https://anil.recoil.org/news.xml#making-eio-and-async-play-nicely" class="anchor" aria-hidden="true"></a>Making Eio and Async play nicely</h3>
<p>I however, thought I'd go in the deep end and make Bonsai work with Sortal, including the Romeo-and-Juliet-esque blending of Bonsai's use of <a href="https://github.com/janestreet/async">Async</a> with Sortal's <a href="https://github.com/ocaml-multicore/eio">Eio</a>. This is complex because they each have their own run queues that don't compose cleanly -- one of the motivating problems we wrote about in <a href="https://anil.recoil.org/papers/2017-tfp-effecthandlers">early OCaml effects</a> work back in 2017.</p>
<p>There have been attempts since then; <a href="https://github.com/https://roscidus.com">Thomas Leonard</a> built an <a href="https://github.com/talex5/async_eio">async_eio bridge</a> and even a <a href="https://github.com/talex5/async-eio-lwt-chimera">lwt/async/eio chimera</a> that could run all three in one process!</p>
<p>I gave this a quick shot, but there has been a lot of bitrot and code movement in the intervening years since these prototypes, especially as OxCaml has zoomed off with all of <a href="https://anil.recoil.org/notes/icfp25-oxcaml">its extensions</a>. So after my morning espresso I decided to instruct the coding agent to instead use JSON-RPC to communicate between a Sortal server (that it spawns) and a Bonsai terminal client. After all, we've got really good <a href="https://anil.recoil.org/notes/aoah-2025-2">jsont codecs</a> so JSON-RPC shouldn't be too difficult.</p>
<h3><a href="https://anil.recoil.org/news.xml#setting-up-the-agentic-environment" class="anchor" aria-hidden="true"></a>Setting up the agentic environment</h3>
<p>The only environment that really worked here was giving the agent a monorepo with all the relevant Jane Street packages checked out with the OxCaml switch configured, and my <a href="https://github.com/oxcaml/opam-repository/pull/23">eio OxCaml patched packages</a>. I later automated this monorepo assembly with <a href="https://anil.recoil.org/notes/aoah-2025-23">unpac</a>.</p>
<p>I then had to build the application in two phases:</p>
<ul>
<li>first get a client/server JSON-RPC working with test cases to check that there is a useful information flow</li>
<li>then, with reference to the <a href="https://github.com/janestreet/bonsai_term_examples">bonsai_term_examples</a> build the user interface.</li>
</ul>
<p><img src="https://anil.recoil.org/images/sortal-claude-ss-planning.webp" alt="%c" title="The agent went through a fairly complex TODO list to do the client/server split"></p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>There was some drama, but I did end up with a working terminal UI here, although perhaps one that requires a lot more iteration before I consider it sound. The most obvious result is that I have no idea how to code review this, as I need to spend more time learnings the basics of Bonsai and <em>then</em> get to terminal coding and <em>then</em> apply it my specific Sortal usecase. But, as the only user of this thing, the artefact that came out after an hour's agentic coding isn't completely bad.</p>
<p>I was only confident enough <a href="https://tangled.org/anil.recoil.org/sortal-term/tree/bonsai">to push this to a branch and not main</a> until I do more experimentation, but I did learn a few fun things.</p>
<h3><a href="https://anil.recoil.org/news.xml#how-does-a-coding-agent-debug-a-ui" class="anchor" aria-hidden="true"></a>How does a coding agent debug a UI?</h3>
<p>One big drawback with coding agents is that they don't have a spatial sense of how UI elements can be laid out. There are workaround for browsers such as using headless Chrome, but often the results of a bad web layout just need repeated iteration.</p>
<p>However, <a href="https://toao.com">Sadiq Jaffer</a> pointed out to me that Claude Code also supports multimodal <em>images</em> as part of its prompts, which could be fed into it to debug the UI! It's as simple as dragging and dropping images into the Claude Code prompt.</p>
<p>For example, look at the two screenshots below that represent before and after triggering a bug:</p>
<p><img src="https://anil.recoil.org/images/sortal-claude-ss-before.webp" alt="%c" title="Before I triggered a UI corruption bug"></p>
<p><img src="https://anil.recoil.org/images/sortal-claude-ss-after.webp" alt="%c" title="After the bug, the contents are all over the place"></p>
<p>Simply dragging these two images allowed the agent to figure out (visually) that there was a layout problem, and then apply it to a reasonable looking code fix that refactored the component layout logic.</p>
<p><img src="https://anil.recoil.org/images/sortal-claude-ss-result.webp" alt="%c" title="The result of the before/after image prompting"></p>
<p>It seems really worthwhile to build a proper Claude Skill that wraps this process up. I could imagine taking <a href="https://github.com/orangekame3/awesome-terminal-recorder">your favourite terminal recording library</a> and hooking it up to an interactive Pty to automate this prompting process of UI layout.</p>
<h3><a href="https://anil.recoil.org/news.xml#bonsai-itself-seems-to-have-a-lot-of-power" class="anchor" aria-hidden="true"></a>Bonsai itself seems to have a lot of power</h3>
<p>The other fun thing was experimenting with <a href="https://github.com/janestreet/bonsai_term_components">Bonsai terminal components</a>, since there are some incredibly nice ones present. These range from <a href="https://github.com/janestreet/bonsai_term_components/tree/with-extensions/bar_chart/src">TUI bar charts</a> to full blown <a href="https://github.com/janestreet/bonsai_term_components/tree/with-extensions/text_editor/src">text editor components</a> (with vi bindings!) to <a href="https://github.com/janestreet/bonsai_term_components/tree/with-extensions/tree_view/src">tree views</a>.</p>
<p>Check out some of the example gifs <a href="https://github.com/janestreet/bonsai_term">in the main repo</a> like the weighted tree one below.</p>
<figure class="image-center"><img src="https://www.cl.cam.ac.uk/~avsm2/weighted-tree.gif"></figure>
<p>So I think working through the constraint problems are well worth it in order to use so many cool components. The text editor especially is extremely usable out of the box as a replace for ed or nvi, which is incredible! There are also pager replacement components which remove the need for less/more, but with the same bindings.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>I call today a "negative but worthwhile result". The Bonsai-term architecture is too complex with the client-server split, although agentic coding made it possible to try it out really quickly. I think in the future it's worth investing in Eio and Async interoperability properly so that everything can run in one process, which dramatically simplifies a TUI. <a href="https://github.com/https://roscidus.com">Thomas Leonard</a> has already done the groundwork here, so it's a matter of putting time in the integration and keeping it working.</p>
<p>Bonsai itself shows how powerful TUIs can be, and in the <a href="https://anil.recoil.org/notes/aoah-2025-10">Day 10</a> I'll look at yet another new TUI library just announced a few days ago!</p>

