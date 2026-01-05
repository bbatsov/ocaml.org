---
title: Claude and Dune
description:
url: https://jon.recoil.org/blog/2025/12/claude-and-dune.html
date: 2025-12-18T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
---

<section><h1><a href="https://jon.recoil.org/atom.xml#claude-and-dune" class="anchor"></a>Claude and Dune</h1><ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-12-18</p></li></ul><ul class="at-tags"><li class="notanotebook"><span class="at-tag">notanotebook</span> </li></ul><p>Back in March of this year we released <a href="https://ocaml.github.io/odoc/odoc/index.html">odoc 3.0.0</a>, a major new version of the OCaml documentation generator. It had a whole load of <a href="https://discuss.ocaml.org/t/ann-odoc-3-beta-release/16043">new features</a>, many of which came with new demands on the build system driving it. We decided when working on it to build a new driver for odoc so that we could adjust it as we were building the new features, and this driver is now used to <a href="https://jon.recoil.org/07/odoc-3-live-on-ocaml-org.html" title="odoc-3-live-on-ocaml-org">build the documentation</a> that appears on <a href="https://ocaml.org/p/base/latest/doc/index.html">ocaml.org</a>. However, it was always the plan to integrate the new features into <a href="https://dune.build">Dune</a> so that everyone could just run <code>dune build @doc</code> and be able to use all of the new odoc 3 features.</p><p>So over the last few weeks I have been wrestling with getting Claude to update the odoc rules in Dune to support <i>some</i> of the new features of odoc v3. What began as a background experiment during a lecture series has turned into a multi-week effort to turn mostly-working code into a clean, reviewable patch. AI-developed software is clearly going to be a big part of our future, and Anil is showing us all the way with his <a href="https://anil.recoil.org/notes/aoah-2025-1">Advent of Agentic Humps</a> by building <i>new</i> software, but upstreaming AI-generated changes to an existing, well established code base <a href="https://github.com/ocaml/ocaml/pull/14369">hasn't got off to a good start</a> in the OCaml community, so I wanted to be extra careful to get this right.</p></section><p>Continue reading <a href="https://jon.recoil.org/blog/2025/12/claude-and-dune.html">here</a></p>
