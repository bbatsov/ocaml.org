---
title: '2025 Advent of Agentic Humps: Building a useful O(x)Caml library every day'
description: An exploration of agentic programming through building useful OCaml libraries
  daily using Claude Code while establishing groundrules for responsible development.
url: https://anil.recoil.org/notes/aoah-2025
date: 2025-12-01T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-ss-2.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Agentic programming has been getting a <a href="https://devclass.com/2025/11/27/ocaml-maintainers-reject-massive-ai-generated-pull-request/">hilariously</a> <a href="https://mastodon.social/@regehr/115606922116794760">bad</a> <a href="https://news.ycombinator.com/item?id=46039274">rap</a> in the OCaml community recently, but it's definitely here to stay despite the <a href="https://anil.recoil.org/notes/claude-copilot-sandbox">security</a> and <a href="https://github.com/ocaml/ocaml/pull/14052#discussion_r2565290229">legal</a> concerns. I realised that to form a useful opinion on all this, I needed to really get into <a href="https://anil.recoil.org/papers/2025-ocaml-ai">using Claude with OCaml</a> for real outputs and not just toy code. So this holiday month, I'm going to release a new <em>useful</em> OCaml library per day until Christmas using Claude Code: the advent of agentic humps is here!</p>
<ul>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-1">Day 1: Crockford</a></strong> for <a href="https://www.crockford.com/base32.html">Crockford Base32</a> encoding.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-2">Day 2: Jsonfeed</a></strong> for an implementation of the <a href="https://www.jsonfeed.org">JSONFeed 1.1</a> spec.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-3">Day 3: XDGe</a></strong> for a <a href="https://specifications.freedesktop.org/basedir/latest/">XDG Directory specifiction</a> with Eio capabilities.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-4">Day 4: Claudeio</a></strong> for a Claude OCaml/Eio SDK so I can use Claude to write more Eio.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-5">Day 5: Bytesrw-eio</a></strong> Bytesrw/Eio adapter and automate opam metadata via a custom Claude skill.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-6">Day 6: Yamlrw</a></strong> for a pure OCaml Yaml 1.2 library, to replace <a href="https://github.com/avsm/ocaml-yaml">ocaml-yaml</a>'s C binding.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-7">Day 7: Yamlt</a></strong> to allow jsont codecs to be serialised to Yaml as well as JSON.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-8">Day 8: Sortal</a></strong>: a contacts management CLI using Yaml, Git and Cmdliner.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-9">Day 9: Sortal-Bonsai</a></strong>: adding a <code>Bonsai_term</code> terminal UI to Sortal via Async.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-10">Day 10: Sortal-Mosaic</a></strong>: adding a <code>Mosaic</code> terminal UI to Sortal via Eio.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-11">Day 11: Cookeio, Public-suffix, Punycode</a></strong>: parsing Internet RFCs to build cookie libraries.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-12">Day 12: Conpool</a></strong>: Eio TLS/TCP connection pooling and self-contained performance viz.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-13">Day 13: Requests</a></strong>: Heckling an OCaml HTTP client from 50 other implementations.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-14">Day 14: Karakeep</a></strong>: Live agentic API construction for the Karakeep app.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-15">Day 15: Htmlrw</a></strong>: Vibespiling Rust/Python into a 100% compliant HTML5 manipulation library.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-16">Day 16: Json-pointer</a></strong>: Vibesplaining specifications by generating OCaml Javascript notebooks.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-17">Day 17: Jmap</a></strong>: Vibemailing little CLI agents to bring my JMAP messages under control.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-18">Day 18: Tomlt</a></strong>: Elegant TOML 1.1 codecs inspired by the jsont data soup paper.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-19">Day 19: Zulip, INIt</a></strong>: Zulip bot framework and INI codecs compatible with Python configparser.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-20">Day 20: Langdetect</a></strong>: Statistical detection for human languages in OCaml, JavaScript and wasm.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-21">Day 21: Html5rw_check</a></strong>: Vibespiling the Nu HTML Validator from Java to typed OCaml checkers.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-22">Day 22: Monopam</a></strong>: Monorepo workflow with dune vendoring for cross-cutting fixes.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-23">Day 23: Unpac</a></strong>: Unifying git and opam package management with branch-based monorepos.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-24">Day 24: Tuatara</a></strong>: Tuatara, an evolving Atom aggregator that mutates its own code.</li>
<li><strong><a href="https://anil.recoil.org/notes/aoah-2025-25">Day 25: OCaml Claude Marketplace</a></strong>: Wrapping up my Claude skills into a reusable bundle.</li>
</ul>
<p><img src="https://anil.recoil.org/images/aoah-ss-2.webp" alt="%c" title="Claude is also very good at automating non-coding tasks like opam metadata"></p>
<p>I'm working through a large backlog of ideas that I'll figure out as each days goes on. Ideas thrown on the pile by colleagues include TCP connection reuse and pooling library with TLS support, HTTP cookie jar handling using Eio, Batteries-include HTTP(S) client library with redirect/cookies, digest vast amounts of Git and summarise it (see a <a href="https://thicket.dev">preview</a>), Zulip bindings using Requests and Eio, Kitty graphics protocol to show graphics in your terminal, client bindings for the JMAP protocol, client bindings for the Immich self hosted photo service, client bindings for the Peertube video service, generate image srcsets in various resolutions for websites, DOI resolution of papers to structured metadata, and a Parquet library in pure OCaml. I'm also working on an <code>io_uring</code> <a href="https://oxcaml.org">OxCaml</a> webserver if I can get the Linux kernel not crashing on me before Santa visits...</p>
<p>My overall goal is to accelerate the heck out of how I manage the growing data in this website. I've been building it as <a href="https://anil.recoil.org/notes/bushel-lives">homebrew infrastructure</a> for the past twenty five years, and now I want it to move from ad-hoc scripts to principled data management. I am also using the libraries to do data processing in the day job for the <a href="https://anil.recoil.org/projects/rsn">remote sensing of nature</a> or <a href="https://anil.recoil.org/projects/ce">evidence synthesis</a>. I'll edit the above list every day to link to what I actually did.</p>
<p>I've picked these choices fairly carefully as they're not "core" libraries that
are difficult to write and require functional ingenuity, but are instead
problems that involve a fair amount of boilerplate code that is typically quite
tedious to write in OCaml. Hand writing code might be on the ropes, but not
quite out of action just yet! But first, let's establish some groundrules for if
this is a good idea or not.</p>
<h2><a href="https://anil.recoil.org/news.xml#isnt-this-just-more-ai-slop-code" class="anchor" aria-hidden="true"></a>Isn't this just more AI slop code?</h2>
<p>There's a definite gag reflex involved with releasing so much code: by prioritising quantity over quality, aren't I just contributing to the world of <a href="https://anil.recoil.org/notes/ai-poisoning">AI slop</a>? However, the hypothesis I am exploring is that the software engineering process fundamentally changes when using agents towards specification driven development, which has always been the <a href="https://deepspec.org/main">holy grail of functional programming</a>.</p>
<p>There's been extensive discussion recently about the role of <a href="https://rfd.shared.oxide.computer/rfd/0576">LLMs in open source</a> elsewhere that informed my thinking. I liked <a href="https://github.com/tmattio">Thibaut Mattio</a> stating how he's <a href="https://discuss.ocaml.org/t/ann-mosaic-a-modern-terminal-user-interface-framework-for-ocaml-early-preview/17572/5">approaching</a> his <a href="https://www.youtube.com/watch?v=BAvXqd0QeVM">own</a> agentic software development:</p>
<blockquote>
<p>AI writes a significant amount of the initial code, and I review, revise, and iterate on a large portion of it. That’s how I work these days. But the architecture, design, and core logic are very much the result of deliberate iteration and manual refinement.
<cite>-- <a href="https://discuss.ocaml.org/t/ann-mosaic-a-modern-terminal-user-interface-framework-for-ocaml-early-preview/17572/5">Thibaut Mattio</a>, OCaml Discuss, 2025</cite></p>
</blockquote>
<p>Bryan Cantrill came up with a superb set of principles for <a href="https://rfd.shared.oxide.computer/rfd/0576">LLM Usage at Oxide</a>. In particular, he separates out using LLMs for reading, writing and coding. I totally agreed with him that I hate people sending me LLM-generated writing for me to review; I would rather get the raw prompt and use my own LLM+context rather than read through other people's slop.</p>
<blockquote>
<p>LLM-generated prose undermines a social contract of sorts: absent LLMs, it is presumed that of the reader and the writer, it is the writer that has undertaken the greater intellectual exertion. (That is, it is more work to write than to read!) For the reader, this is important: should they struggle with an idea, they can reasonably assume that the writer themselves understands it — and it is the least a reader can do to labor to make sense of it.
<cite>-- <a href="https://rfd.shared.oxide.computer/rfd/0576">Using LLMs at Oxide</a>, RFD0576, Dec 2025</cite></p>
</blockquote>
<p>However, there is an undeniable (and growing) power in the ability to generate code at scale using LLMs. I've been doing a lot of this <a href="https://anil.recoil.org/news.xml">with Python</a> in recent months, but I find myself increasingly frustrated by the lack of typing guardrails involved with agentic coding there.</p>
<p>I believe that a strongly typed, modular language like OCaml could become one of the <em>best</em> languages for agentic coding in the longer term, with <a href="https://arxiv.org/abs/2508.04865">advances</a> happening rapidly to cure the data deficiency problem for relatively obscure languages with smaller corpuses. Also, with <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml on the horizon</a>, getting help with increasingly complex (but rewarding) code annotations such as modes and kinds sems essential.</p>
<h2><a href="https://anil.recoil.org/news.xml#groundrules-for-the-advent-of-agentic-humps" class="anchor" aria-hidden="true"></a>Groundrules for the Advent of Agentic Humps</h2>
<p>After reflecting on the recent discussions, I decided on these for my little
December experiment:</p>
<ul>
<li><strong>No AI-driven contributions to other people's code.</strong> All my slop stays in my
own lane unless the other person agrees. Luckily my own research group is
easy to bribe with some festive beer so I hope to get them (or you, my dear reader) to voluntarily help me judge the success or failure.</li>
<li><strong>Read every line of code that's tagged for release</strong>. Even if I haven't written it all, it's vital to look for howlers. However, intermediate pushes may have slop in them, so stick to the tagged releases.</li>
<li><strong>The library has to be used somewhere</strong> in my production code stack, for example this website. Time to <a href="https://en.wikipedia.org/wiki/Eating_your_own_dog_food">eat my own</a> agentic slop on my own knowledge bases!</li>
<li><strong>Build on great human designed code.</strong> LLMs do not replace or compete with well designed foundation libraries in the OCaml ecosystem like <a href="https://github.com/ocaml-multicore/eio">Eio</a>, <a href="https://dev.realworldocaml.org">Core</a>, <a href="http://github.com/ocsigen/lwt/blob/master/CHANGES">Lwt</a> or the <a href="https://erratique.ch/contact.en">Bunzli-verse</a>. Each of these have different design ethoses, but if they didn't exist there is no scaffolding over which to compose LLM-driven code outputs. So this is <em>not</em> a competition to beat them, but rather to use them more effectively.</li>
</ul>
<p>And overall, this process should not help me learn more about agentic workflows
but also contribute to the wider discussion, so I'll capture what I learn in
this blog series at the end.</p>
<p>Some non-rules:</p>
<ul>
<li>Keeping agentic code separate from my "real code" seems pointless nowadays,
with LLMs everywhere. I tried that earlier in the year, but I fear the
poisoning will have to be dealt with by other means.</li>
<li>I'm trying to keep this specific to my own OCaml workflow, and not
generalising this for a hypothetical other user. But you should feel free to
fork this stuff.</li>
<li>I have no idea how I'm going to maintain all these libraries once released. A
problem for 2026. I'm not particularly attached to any of these libraries, so
maintainance/rewrite offers are all fine by me.</li>
<li>There's a reasonable chance some of this has some bad bugs, since it's not
going through peer review. I'll do my best to handle test coverage, but
please be tolerant. Bug reports are welcome.</li>
<li>I've done my best to manually scan code and attribute copyright where possible,
but there remains a chance I have horribly screwed up. Any errors in attribution
are my own, but I'm going to press on and take the risk.</li>
</ul>
<p>If anyone else wants to join in the Advent of Agentic Humps, ping me on
whatever communication medium you like. Just remember the groundrules: don't waste
other maintainer's time without their permission first.</p>

