---
title: Odoc bugs
description:
url: https://jon.recoil.org/blog/2025/09/odoc-bugs.html
date: 2025-09-22T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
---

<section><h1><a href="https://jon.recoil.org/atom.xml#odoc-bugs" class="anchor"></a>Odoc bugs</h1><ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-09-22</p></li></ul><ul class="at-tags"><li class="libs"><span class="at-tag">libs</span> <p>odoc.model</p></li></ul><p>This post is a brief write-up of a couple of bugs in odoc that I've been working on over the past 2 weeks. I was convinced at the start of this that I was actually fixing one bug, but although they both had the same backtrace and similar immediate causes, they're actually quite different. They both involve <em>expansion</em>, which is the process that odoc uses to work out the contents of a module from its expression - what allows you to see the contents of a module such as <code>module M = Map.Make(String)</code>.</p></section><p>Continue reading <a href="https://jon.recoil.org/blog/2025/09/odoc-bugs.html">here</a></p>
