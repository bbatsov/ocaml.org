---
title: 'AoAH Day 22: Assembling monorepos for agentic OCaml development'
description: Materialising opam metadata into git submodules and monorepos, enabling
  cross-cutting fixes and unified odoc3 documentation across dozens of OCaml libraries.
url: https://anil.recoil.org/notes/aoah-2025-22
date: 2025-12-22T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-monopam-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Over the past three weeks, I've accumulated dozens of OCaml repositories as
part of this <a href="https://anil.recoil.org/notes/aoah-2025">series</a>. Keeping them coordinated has become a real challenge;
when I fix something in one library, dependent packages need updating, and
agents working on one repo have no visibility into related code.  Ideally,
I could have all my code in one place and see what agents can do with a lot of local context.</p>
<p>Today I'm switching tacks to address this with a monorepo workflow built around dune's
<a href="https://www.dra27.uk/blog/platform/2018/08/15/dune-vendoring.html">excellent vendoring support</a>.
I last visited this when building <a href="https://anil.recoil.org/papers/rwo">RWOv2</a> and its <a href="https://github.com/realworldocaml/book">monorepo</a> when I built a <a href="https://github.com/tarides/opam-monorepo">duniverse</a> tool that turned into the <a href="https://github.com/tarides/opam-monorepo">opam-monorepo</a> plugin that <a href="https://mirage.io">MirageOS</a>
now uses. Let's see what happens in today's agentic world instead!</p>
<p>I also wanted to explore the small group dynamic around vibecoding tools. For today's tool, I first asked <a href="https://tarides.com/blog/author/mark-elvers/">Mark Elvers</a> to spend a few hours sketching out the sort of tool he might want, and then <a href="https://jon.recoil.org">Jon Ludlam</a> has been <a href="https://jon.recoil.org/blog/2025/12/claude-and-dune.html">using Claude</a> to build up <a href="https://github.com/ocaml/dune/pull/12995">complex odocv3 rules</a>. The way we work together with agentic code is quite different from when we've handcrafted a project, with the code itself now being more throwaway as we pass the baton among each other. I'm lightheartedly calling this 'vibrating' amongst each other to reflect the new speed of agentic iterations, and to differentiate from the more thoughtful process of pair programming. Today's tool <strong><a href="https://tangled.org/anil.recoil.org/repo-tool">monopam</a></strong> helps to manage OCaml monorepos for cross-cutting code and documentation.</p>
<h2><a href="https://anil.recoil.org/news.xml#the-git-repo-coordination-problem" class="anchor" aria-hidden="true"></a>The Git repo coordination problem</h2>
<p>The OCaml libraries I've built are designed to be standalone, but obviously have interdependencies among each other. <a href="https://anil.recoil.org/notes/aoah-2025-13">Requests</a> depends on <a href="https://anil.recoil.org/notes/aoah-2025-12">conpool</a> for connection management and HTTP cookie logic from <a href="https://anil.recoil.org/notes/aoah-2025-11">cookeio</a>. The codec libraries like <a href="https://anil.recoil.org/notes/aoah-2025-7">yamlt</a>, <a href="https://anil.recoil.org/notes/aoah-2025-18">tomlt</a>, and <a href="https://anil.recoil.org/notes/aoah-2025-19">init</a> all have optional dependencies on bytesrw for serialisation. Meanwhile, <a href="https://anil.recoil.org/notes/aoah-2025-15">html5rw</a> depends on <a href="https://anil.recoil.org/notes/aoah-2025-20">langdetect</a> and has optional dependencies on the wasm and JavaScript compiler stack.</p>
<p>Here's the full inventory of what I've built in the last few weeks:</p>
<div role="region"><table>
<tbody><tr>
<th>Day</th>
<th>Library</th>
<th>Description</th>
</tr>
<tr>
<td>1</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-crockford">ocaml-crockford</a></td>
<td>Crockford Base32 encoding</td>
</tr>
<tr>
<td>2</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed">ocaml-jsonfeed</a></td>
<td>JSONFeed 1.1 implementation</td>
</tr>
<tr>
<td>3</td>
<td><a href="https://tangled.org/anil.recoil.org/xdge">xdge</a></td>
<td>XDG directories with Eio capabilities</td>
</tr>
<tr>
<td>4</td>
<td><a href="https://tangled.org/anil.recoil.org/claudeio">claudeio</a></td>
<td>Claude OCaml/Eio SDK</td>
</tr>
<tr>
<td>5</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-bytesrw-eio">ocaml-bytesrw-eio</a></td>
<td>Bytesrw/Eio adapter</td>
</tr>
<tr>
<td>6</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-yamlrw">ocaml-yamlrw</a></td>
<td>Pure OCaml Yaml 1.2 parser</td>
</tr>
<tr>
<td>7</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-yamlt">ocaml-yamlt</a></td>
<td>jsont codecs for Yaml</td>
</tr>
<tr>
<td>8</td>
<td><a href="https://tangled.org/anil.recoil.org/sortal">sortal</a></td>
<td>Contacts management CLI</td>
</tr>
<tr>
<td>11</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-punycode">ocaml-punycode</a></td>
<td>Punycode RFC3492 implementation</td>
</tr>
<tr>
<td>11</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-publicsuffix">ocaml-publicsuffix</a></td>
<td>Public suffix list for cookies</td>
</tr>
<tr>
<td>11</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-cookeio">ocaml-cookeio</a></td>
<td>HTTP cookie handling</td>
</tr>
<tr>
<td>12</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-conpool">ocaml-conpool</a></td>
<td>TCP/TLS connection pooling</td>
</tr>
<tr>
<td>13</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-requests">ocaml-requests</a></td>
<td>HTTP client library</td>
</tr>
<tr>
<td>14</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-karakeep">ocaml-karakeep</a></td>
<td>Karakeep bookmark API</td>
</tr>
<tr>
<td>15</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-html5rw">ocaml-html5rw</a></td>
<td>HTML5 parser and validator</td>
</tr>
<tr>
<td>16</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-json-pointer">ocaml-json-pointer</a></td>
<td>JSON Pointer RFC6901</td>
</tr>
<tr>
<td>16</td>
<td><a href="https://tangled.org/anil.recoil.org/odoc-xo">odoc-xo</a></td>
<td>odoc extras for notebooks</td>
</tr>
<tr>
<td>17</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-jmap">ocaml-jmap</a></td>
<td>JMAP email client</td>
</tr>
<tr>
<td>18</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-tomlt">ocaml-tomlt</a></td>
<td>TOML 1.1 codecs</td>
</tr>
<tr>
<td>19</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-zulip">ocaml-zulip</a></td>
<td>Zulip bot framework</td>
</tr>
<tr>
<td>19</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-init">ocaml-init</a></td>
<td>INI file codecs</td>
</tr>
<tr>
<td>20</td>
<td><a href="https://tangled.org/anil.recoil.org/ocaml-langdetect">ocaml-langdetect</a></td>
<td>Language detection</td>
</tr>
</tbody></table></div><p>And the Claude skills I've developed along the way:</p>
<div role="region"><table>
<tbody><tr>
<th>Skill</th>
<th>Purpose</th>
</tr>
<tr>
<td><a href="https://tangled.org/anil.recoil.org/claude-ocaml-metadata">claude-ocaml-metadata</a></td>
<td>Automate opam package setup</td>
</tr>
<tr>
<td><a href="https://tangled.org/anil.recoil.org/claude-ocaml-internet-rfc">claude-ocaml-internet-rfc</a></td>
<td>Fetch and integrate IETF RFCs</td>
</tr>
<tr>
<td><a href="https://tangled.org/anil.recoil.org/claude-ocaml-tidy-code">claude-ocaml-tidy-code</a></td>
<td>Refactor generated OCaml</td>
</tr>
<tr>
<td><a href="https://tangled.org/anil.recoil.org/claude-ocaml-to-npm">claude-ocaml-to-npm</a></td>
<td>Publish js_of_ocaml to NPM</td>
</tr>
</tbody></table></div><p>So far, I've been publishing each of these as individual Git repositories, but maintaining an <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo">overlay opam repo</a> that a user can add to gain access to consistent metadata that makes the dev packages installable. Unfortunately, incorrect interdependencies are already creeping in; <a href="https://github.com/samoht">Thomas Gazagnaire</a> asked me today why my yamlt library depends on webassembly, and I'm sure it shouldn't -- I've clearly got a stray missing dependency somewhere in my metadata.</p>
<p>When an agent works on just one repository, it has no visibility into
how changes might benefit (or break) dependent code. It also can't make fixes
for documentation <em>across</em> repositories. I noticed quite often in the past month that I was
cloning source packages temporarily into my workspace for the agent to access, and then
deleting them.  All this motivates me to investigate alternatives to having lots of small git repos for my day-to-day agentic development.</p>
<h2><a href="https://anil.recoil.org/news.xml#dunes-vendoring-is-amazing-for-monorepos" class="anchor" aria-hidden="true"></a>Dune's vendoring is amazing for monorepos</h2>
<p>Dune has a fantastic but underappreciated feature: it automatically discovers and builds any OCaml code in subdirectories. As <a href="https://github.com/dra27">David Allsopp</a> explained <a href="https://www.dra27.uk/blog/platform/2018/08/15/dune-vendoring.html">back in 2018</a>, you can simply clone dependencies into your project tree and dune will build them together.</p>
<p>Note that this only works if all the packages contain dune files. Since OCaml is all about choice, there's no hard mandate to use one build tool: it's perfectly fine to use <a href="https://github.com/ocaml/ocamlbuild">ocamlbuild</a> or Makefiles, as long as your libraries install a <a href="https://dune.readthedocs.io/en/stable/reference/findlib.html">findlib META file</a>. Dune will also gain support for <a href="https://dune.readthedocs.io/en/latest/explanation/package-management.html">opam package installation</a> next year to help make this even easier.</p>
<p>Years ago I built a tool called <a href="https://github.com/ocamllabs/duniverse">duniverse</a> to automate this vendoring workflow. It worked, but required a lot of manual repository management. With agents now doing the heavy lifting, though, I thought it might be easier now and so decided to revisit it.</p>
<p>Today's work ended up extending <a href="https://tarides.com/blog/author/mark-elvers/">Mark Elvers</a> initial foray into monorepos to release <strong><a href="https://tangled.org/anil.recoil.org/repo-tool">monopam</a></strong>: a little CLI tool that reads opam metadata from a local repository (like <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo">aoah-opam-repo</a>), resolves the dependency graph, materialises the sources as git submodules, and produces a single dune workspace that builds everything together. For now, it depends on an opam local switch to work, but if someone wants to try it with dune package management I'd love to hear how it goes.</p>
<h2><a href="https://anil.recoil.org/news.xml#materialising-aoah-opam-repo" class="anchor" aria-hidden="true"></a>Materialising aoah-opam-repo</h2>
<p>The <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo">aoah-opam-repo</a> contains all the packages I've built during this series, maintained using the <a href="https://anil.recoil.org/notes/aoah-2025-5">opam metadata skill</a>. Let's turn it into a unified source tree using monopam:</p>
<pre><code class="language-bash">$ monopam --opam-overlay aoah-opam-repo -o aoah-vendor --submodules
Scanning opam overlay at aoah-opam-repo
Found 21 repositories to process
Initialized empty Git repository in aoah-vendor/.git/
Using git submodules for vendor dependencies
Cloning into ...
remote: Enumerating objects: 14, done.
remote: Counting objects: 100% (14/14), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 14 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
# ...etc
Output written to aoah-vendor
  opam-repository/ - opam package definitions
  vendor/          - source code
  setup.sh         - run to pin packages and install deps
</code></pre>
<p>This solves the opam constraints, finds a cut of dependencies, and then git
submodule adds the lot of them into my target repository. At this point, we run
<code>setup.sh</code> which creates an opam local switch and then <code>dune build</code> just works
using all the locally cloned repos.</p>
<pre><code>.
├── _opam
├── dune
├── dune-project
├── opam-repository
│&nbsp;&nbsp; ├── packages
│&nbsp;&nbsp; └── repo
└── vendor
    ├── dune
    ├── ocaml-bytesrw-eio
    ├── ocaml-claudeio
    ├── ocaml-conpool
    ├── ocaml-cookeio
    ├── ocaml-crockford
    ├── ocaml-html5rw
    ├── ocaml-init
    ├── ocaml-json-pointer
    ├── ocaml-karakeep
    ├── ocaml-langdetect
    ├── ocaml-publicsuffix
    ├── ocaml-punycode
    ├── ocaml-requests
    ├── ocaml-tomlt
    ├── ocaml-yamlrw
    ├── ocaml-yamlt
    ├── odoc-xo
    └── xdge
</code></pre>
<p>The directory structure is straightforward: we have our opam repository, a local switch
and the source code all in one place now, and buildable in a single dune
invocation.</p>
<h2><a href="https://anil.recoil.org/news.xml#cross-cutting-fixes-with-agents" class="anchor" aria-hidden="true"></a>Cross-cutting fixes with agents</h2>
<p>With all the code in one place, agents can now spot opportunities that span
multiple packages. The first thing I did was to build a full documentation set
across all my packages.</p>
<p><img src="https://anil.recoil.org/images/aoah-monopam-ss-1.webp" alt="%c"></p>
<h3><a href="https://anil.recoil.org/news.xml#building-unified-documentation-with-odoc3" class="anchor" aria-hidden="true"></a>Building unified documentation with odoc3</h3>
<p><a href="https://jon.recoil.org">Jon Ludlam</a> has been doing <a href="https://jon.recoil.org/blog/2025/12/claude-and-dune.html">excellent work</a> on odoc3, the modern documentation generator for OCaml. odoc is a composable documentation generator that has a number of <a href="https://ocaml.github.io/odoc/odoc/driver.html">mini-commands</a> that can be called in sequence to build fragments of HTML. Jon's been adding support into dune build rules to build a fully cross-referenced documentation site across an entire dune workspace.</p>
<p>This is where the monorepo approach obviously shines, since we could generate a single site for all my code with types linking directly to their definitions across opam packages. The <a href="https://anil.recoil.org/notes/aoah-2025-16">interactive notebooks</a> I built earlier could reference any type across the whole codebase.</p>
<p>I first pinned Jon's odoc branch and then built the unified docs with the right rules.</p>
<pre><code class="language-bash">$ opam pin add dune https://github.com/jonludlam/dune.git#odoc-v3-rules
$ dune build @doc
$ open _build/default/_doc/_html/index.html
</code></pre>
<p>This generated a working doc page, that also included cross-referenced links
<em>across</em> packages. But even more cool is that if a package doesn't exist in the
local monorepo, it also does a best-effort link straight to the central doc
repository on OCaml.org.</p>
<p>There were a few integration issues that may be bugs in the dune rules. For instance:</p>
<pre><code>&gt; dune build @doc
File "/Users/avsm/src/git/knot/aoah-vendor3/_opam/lib/angstrom/META", line 1, characters 0-0:
Error: Library "angstrom-unix" not found.
-&gt; required by library "angstrom.unix" in
   /Users/avsm/src/git/knot/aoah-vendor3/_opam/lib/angstrom
-&gt; required by alias vendor/ocaml-karakeep/doc
</code></pre>
<p>This is a package that's present in the local tree, but not installed in opam.
After I opam installed it, the doc generation worked. This probably shouldn't
break a local docs build, so I commented on the GitHub PR.</p>
<p>After this, there were genuine bugs in my own documentation, as evidenced by
warnings emitted by dune. The agents fixed problems and added cross-references across
the documentation, and I could do a single <code>git status</code> to see all the affected
packages.</p>
<pre><code>Changes not staged for commit:
  (use "git add &lt;file&gt;..." to update what will be committed)
  (use "git restore &lt;file&gt;..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)
        modified:   vendor/ocaml-claudeio (modified content)
        modified:   vendor/ocaml-init (modified content)
        modified:   vendor/ocaml-json-pointer (modified content)
        modified:   vendor/ocaml-requests (modified content)
        modified:   vendor/ocaml-tomlt (modified content)
        modified:   vendor/ocaml-yamlrw (modified content)
        modified:   vendor/ocaml-yamlt (modified content)
        modified:   vendor/xdge (modified content)
</code></pre>
<h3><a href="https://anil.recoil.org/news.xml#code-fixing-across-packages" class="anchor" aria-hidden="true"></a>Code fixing across packages</h3>
<p>I then prompted the agents to find opportunities for optimisation <em>across</em>
all the packages. Running this in a fixpoint ended up allowing for backwards
and forwards cross-references: for example, it could add "related libraries"
sections, and also normalise error handling and logging interfaces where
there were inconsistencies.</p>
<p><img src="https://anil.recoil.org/images/aoah-monopam-ss-2.webp" alt="%c" title="Docs fixes from the agent across repositories">
<img src="https://anil.recoil.org/images/aoah-monopam-ss-3.webp" alt="%c" title="And similarly, interface fixes work just as well"></p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>Once I had a consistent monorepo, I could commit the changes and distribute
a batch easily. For example, I uploaded my <a href="https://tangled.org/anil.recoil.org/monopam-odocv3-dune-test">test odocv3 monorepo</a>
and commented on <a href="https://github.com/ocaml/dune/pull/12995">ocaml/dune#12995</a>.</p>
<p>On the other hand, monopam's git submodule workflow is awkward to use due to how separate
submodules are from the main git repository. I had to individually commit and push each of the
changes, and I couldn't get a unified git diff or make commits <em>across</em> the vendored
repositories.  I have a scheme in mind to improve this, which is a topic for tomorrow's post!</p>
<p>Socially speaking, I'm reasonably convinced a monorepo workflow of <em>some</em> sort is the
future for agentic coding. They just work so much better with local tool calls that can
rapidly scan a lot of data instead of making remote calls (which are awkward from a permissions
perspective as well). We'll still need to figure out how the dynamics of 'vibrating' patches
across each other goes; it's early days for the dynamics of agentic pair programming.</p>

