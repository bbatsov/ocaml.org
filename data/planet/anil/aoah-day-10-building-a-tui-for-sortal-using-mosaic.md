---
title: 'AoAH Day 10: Building a TUI for Sortal using Mosaic'
description: Building a simpler single-process terminal UI for Sortal using Mosaic's
  effects-based direct-style API, with Eio integration and discovering multimodal
  image debugging for terminal layouts.
url: https://anil.recoil.org/notes/aoah-2025-10
date: 2025-12-10T00:00:00-00:00
preview_image: https://anil.recoil.org/images/mosaic-cli-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After building a reasonably complete <a href="https://anil.recoil.org/notes/aoah-2025-8">Sortal contacts manager</a> and
trying out <a href="https://anil.recoil.org/notes/aoah-2025-9">OxCaml's Bonsai_term</a>, I thought I'd have a second go
at a terminal UI using a newly announced <a href="https://discuss.ocaml.org/t/ann-mosaic-a-modern-terminal-user-interface-framework-for-ocaml-early-preview/17572">Mosaic library</a> by <a href="https://github.com/tmattio">Thibaut Mattio</a>.</p>
<p>I first noticed this library when Thibaut presented his <a href="https://watch.ocaml.org/w/oTv8j7T7eGrtHxpzaRe1LZ">OCaml coding with AI</a> talk at FunOCaml. It's quite different from <a href="https://github.com/janestreet/bonsai_term">Bonsai</a> in that Mosaic uses OCaml's effects to provide a more direct-style API, and so seems worth experimenting with. So today's task is to port Sortal to use Mosaic and see what this terminal UI looks like!</p>
<p><a href="https://github.com/tmattio/mosaic"> <img src="https://anil.recoil.org/images/mosaic-cli-1.webp" alt="%c" title="Thibaut has big plans for an ML training TUI using Mosaic"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>The environmental setup for Mosaic was similarly complicated to <a href="https://anil.recoil.org/notes/aoah-2025-9">yesterday's with Bonsai</a>, but for different reasons. Mosaic requires OCaml 5.4.0 or higher, so I had to relax some constraints in dependent packages. But it also requires <a href="https://github.com/tmattio/mosaic/issues/7">pinning vendored packages</a>, which I did manually (specifically, the vendored <code>tree-sitter</code> package). I later automated this vendoring with <a href="https://anil.recoil.org/notes/aoah-2025-23">unpac</a>.</p>
<p>Once this manual package messing was done, the rest was straightforward. I left my remote Linux system vibing with access to all the relevant source code (important since this is a bleeding edge package so the parametric memory of the agent will know nothing about it).  Luckily, the interaction between Eio and Mosaic is far easier, so we ended up with a single-process OCaml binary for the TUI; a relief after yesterday's JSON-RPC gymnastics.</p>
<p>Once again, the ability to <a href="https://anil.recoil.org/notes/aoah-2025-9">paste in images</a> is key to debugging terminal UIs. I wish I had the time to automate this via a skill, but I'll come back to this later in the month perhaps.</p>
<p><img src="https://anil.recoil.org/images/sortal-mosaic-ss-1.webp" alt="%c" title="I don't think Mosaic has themes like Bonsai term, but the agent picked really ugly colours by default."></p>
<p><img src="https://anil.recoil.org/images/sortal-mosaic-ss-2.webp" alt="%c" title="However pasting in an image allowed the agent to pick a better greyscale baseline quickly."></p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>The working <a href="https://tangled.org/anil.recoil.org/sortal-term">terminal application</a> was far simpler than the earlier Bonsai version by virtue of being a single binary. In fact, the code is pretty <a href="https://tangled.org/anil.recoil.org/sortal-term/blob/main/lib/sortal_mosaic.ml">readable</a> and lives in a single file.  Like with Bonsai, what really helped was specifying that the agent should use the <a href="https://github.com/tmattio/mosaic/tree/main/mosaic/examples">Tea example components</a> as inspiration, and it used those to both fix the theming but also introduced Markdown rendering to make the default UI look really smart.</p>
<p><img src="https://anil.recoil.org/images/sortal-mosaic-ss-3.webp" alt="%c" title="The UI before running through the example components with just the agent guessing"></p>
<p><img src="https://anil.recoil.org/images/sortal-mosaic-ss-4.webp" alt="%c" title="The UI after deriving knowledge from the Mosaic examples, with nicer rendering!"></p>
<p>There was also an amusing keybinding bug that both Bonsai and Mosaic suffered from in the first cut of the agent code. When I searched for "/sadiq" it would crash consistently on the last letter of his name. Why?!! About 15 minutes of debugging code ensued, but at one point I realised that "q" was also the "quit" keybinding, and the app was just exiting cleanly. The agent didn't spot this at all, so some human intuition was useful in building the UI logic!</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>This Mosaic terminal is also very usable, and some things like terminal mouse clicking works too. It's less plug-and-play than Bonsai in the longer term as I'll have to build more components like text editors. However, the easier integration with the wider OCaml ecosystem really helps here since there is only one problem to solve ("build me a TUI against an existing data model") rather than two ("port all your dependencies to OxCaml") as well.</p>
<p>Both of these TUIs will require more focussed attention from me to learn their data models and rendering logic; neither Bonsai nor Mosaic particularly benefited from asking the agent to explain their core architectures to me. Only excellent human-written documentation will achieve that.</p>
<p>I am very tempted to build a <a href="https://anil.recoil.org/notes/aoah-2025-4">Claudeio</a> TUI using Mosaic as one of the future advent day projects, though!  But tomorrow, I'm going to switch gears in <a href="https://anil.recoil.org/notes/aoah-2025-11">Day 11</a> and work on HTTP cookies towards a HTTP downloader tool.</p>

