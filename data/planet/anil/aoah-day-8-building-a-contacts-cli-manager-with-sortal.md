---
title: 'AoAH Day 8: Building a contacts CLI manager with Sortal'
description: Creating Sortal, a CLI contacts management application using Yaml storage,
  XDG directories, Git-based synchronization, and integrating all previously built
  libraries into a cohesive CLI tool.
url: https://anil.recoil.org/notes/aoah-2025-8
date: 2025-12-08T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>I've been accumulating a <em>lot</em> of contacts that I use to write cross references
on my website. This works by using
<a href="https://erratique.ch/software/cmarkit">Cmarkit</a> to parse my custom Markdown,
and spot entries like <code>[@sadiqj]</code> and convert those into a full reference like
<a href="https://toao.com">Sadiq Jaffer</a>.</p>
<p>Today, I want to build a full CLI application that stores all my
contacts as Yaml files in my home directory using XDG conventions, and give me
a simple search interface so I can quickly autocomplete these posts from my
editor.  I call this little application "<a href="https://tangled.org/anil.recoil.org/sortal">Sortal</a>".</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>The data model I want behind sortal is that I can have a flat set of Yaml files somewhere
in my XDG path, and that the CLI and library will build those up into data structures for
querying and printing. I have around 2500 contacts or so, so this is very manageable without
a fancy database.</p>
<p>Sortal uses many of the previous libraries I've been building up so far. I prompted
the agent to generate a standalone OCaml project, and to analyse my existing (extremely hacked together)
website code to determine a reasonable semantic for Sortal's schema, and then to design
a CLI that uses <a href="https://anil.recoil.org/notes/aoah-2025-7">yamlt</a> and <a href="https://anil.recoil.org/notes/aoah-2025-3">xdge</a> along with jsont/cmdliner
to plan a user interface with subcommands. I also blended in <a href="https://github.com/dbuenzli/fmt">Fmt</a>
and <a href="https://github.com/dbuenzli/logs">Logs</a> to get nice colours in my terminal. I'm using a lot of libraries from <a href="https://erratique.ch">Daniel Bünzli</a>, which is no coincidence as the model needs to have far less context if it only has to use a few, well-designed and modular dependencies.</p>
<p>Architecturally, I used Claude's <a href="https://code.claude.com/docs/en/common-workflows">planning mode</a> for this
with the best Opus 4.5 model, along with instructions to maintain a separation between the core
jsont schemas, the library logic, and the cmdliner terms. A useful tip is to ensure that you prompt
Claude to "ask for any clarifications" after working, and it'll drop you into a custom terminal user
interface that structures followup questions. This is a very convenient way of batching answers to the
code model.</p>
<p><img src="https://anil.recoil.org/images/aoah-ss-1.webp" alt="%c"></p>
<p>I did have to do some prompting to refine how the agent designed the xdge and cmdliner integration, specifically by selecting <em>only</em> the XDG dirs that Sortal actually uses.  It does work beautifully with the CLI once fixed, as the man page shows:</p>
<p><img src="https://anil.recoil.org/images/sortal-ss-1.webp" alt="%c"></p>
<p>I also prompted the model to use the man pages to guide a better CLI design, and it came up with a reasonable set of subcommands:</p>
<p><img src="https://anil.recoil.org/images/sortal-ss-2.webp" alt="%c"></p>
<h2><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h2>
<p>I'm actually opting not to do any fancy testing for this library, since it's basically a bunch of data munging at this stage and I'm going to using it myself for my own data.</p>
<p>The one thing I did do was to prompt the agent to have an automated import script from my existing Yaml formats, so I could run both in parallel. But aside from that, just keeping an eye the model's inferred success criteria was helpful:</p>
<pre><code>- dune build @check succeeds
- All tests pass (dune runtest)
- Documentation builds without warnings (dune build @doc)
- No __ references in generated HTML docs
- CLI executable works correctly
- sortal.schema builds without eio/xdge dependencies
- Existing API unchanged (backward compatible)
</code></pre>
<p>This is all the sort of thing I would have done myself by hand, but the model has now enough previous libraries and <a href="https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool">memory</a> and the <a href="https://tangled.org/anil.recoil.org/claude-ocaml-metadata">ocaml-metadata skill</a> to figure out my preferred style of libraries.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>The proof is in the pudding, so after knocking up a quick import script for my existing contacts, here's it showing <a href="https://toao.com">Sadiq Jaffer</a>.</p>
<p><img src="https://anil.recoil.org/images/sortal-ss-3.webp" alt="%c"></p>
<p>The associated Yaml is very straightforward, out of my <code>~/.local/share/sortal</code> directory:</p>
<pre><code class="language-yaml">version: 1
kind: person
handle: sadiqj
names:
  - Sadiq Jaffer
emails:
  - address: sj514@cam.ac.uk
    type: personal
  - address: sadiq@toao.com
organizations:
urls:
  - url: https://toao.com
services:
  - url: https://github.com/sadiqj
    kind: github
    handle: sadiqj
    primary: false
orcid: 0009-0006-4120-3244
feeds:
  - type: atom
    url: https://toao.com/feeds/posts.atom.xml
</code></pre>
<p>As a giant quality of life improvement, I also coded up a <code>Git_store</code> module that automated the commit and push of the XDG directory to a private repo on every CLI change. This gives me a super quick way of synching my contacts.</p>
<p>I also left the version field in there to allow for future schema evolution. I guess this would need a jsont codec that <em>only</em> reads the version field and preserves <a href="https://erratique.ch/software/jsont/doc/cookbook.html#unknown_members">unknown object members</a> and passes them to a versioning module. Something for the future!</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>Well, it works! I'm using this now for this website, and I'm going to add more quality-of-life things as I need it, such as support for thumbnails.</p>
<p>It's very encouraging to see an emerging workflow in OCaml: this language is brilliant at sketching out a big complicated service, and then refactoring to break it up into composable libraries. Having said that, I don't think the agentic coding exhibits particularly good taste in library design, and is nowhere near capable of building libraries of the quality of <a href="https://erratique.ch">Daniel Bünzli</a>, but they are very good at <em>using</em> those libraries.</p>
<p>Also of note is that I have zero PPX in this CLI, which I would have used if I were building by hand. Instead, building up boilerplate combinator functions (like jsont codecs) is done pretty well by the coding agent, and results in semantically richer code.  I very much appreciate <a href="https://patrick.sirref.org">Patrick Ferris</a> maintaining ppxlib, but I do not miss having a dependency on it!</p>
<p>Tomorrow in <a href="https://anil.recoil.org/notes/aoah-2025-9">Day 9</a> we'll try Bonsai_term for a terminal UI, and then <a href="https://anil.recoil.org/notes/aoah-2025-10">Mosaic</a> to see how alternative approaches might work.</p>

