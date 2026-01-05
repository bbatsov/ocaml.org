---
title: 'AoAH Day 5: Bytesrw Eio adapters and automating opam metadata'
description: Building Bytesrw-Eio adapters for composable byte stream I/O while discovering
  Claude Skills as a powerful way to automate opam package metadata management through
  reusable workflow templates.
url: https://anil.recoil.org/notes/aoah-2025-5
date: 2025-12-05T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-ss-2.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After the <a href="https://anil.recoil.org/notes/aoah-2025-4">Claude exertions</a> of yesterday, I needed something easier to cool my laptop down. I wanted to learn how to use another new library from <a href="https://erratique.ch">Daniel Bünzli</a> called <a href="https://github.com/dbuenzli/bytesrw">Bytesrw</a>, which provides composable byte stream readers and writers. It supplies ways to serialise Bytesrw to Unix file descriptors, so I figured I'd add in an <a href="https://github.com/ocaml-multicore/eio">Eio</a> library for this. Along the way though, I was generating a growing number of opam packages, so I also learnt how to use <a href="https://www.claude.com/blog/skills">Claude Skills</a> to automate my opam metadata on <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">Tangled</a> as well.</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>The <a href="https://tangled.org/anil.recoil.org/ocaml-bytesrw-eio">Bytesrw-eio</a> library is exceedingly simple and exposes just two functions, so it was pretty easy for the agent to code up given the source repositories already had Unix equivalents.</p>
<pre><code>val bytes_reader_of_flow :
  ?slice_length:int -&gt; _ Eio.Flow.source-&gt;
  Bytesrw.Bytes.Reader.t

val bytes_writer_of_flow :
  ?slice_length:int -&gt; _ Eio.Flow.sink -&gt;
  Bytesrw.Bytes.Writer.t
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>While coding this simple library, I realised my bottleneck lay in managing the
growing number of opam packages that I have lying around! Can I also use agents
to manage these libraries?</p>
<p><img src="https://anil.recoil.org/images/aoah-ss-2.webp" alt="%c" title="The completed list of opam metadata actions, all done with using a custom Claude skill"></p>
<h3><a href="https://anil.recoil.org/news.xml#my-opam-structure" class="anchor" aria-hidden="true"></a>My opam structure</h3>
<p>I'm maintaining all the libraries at a custom <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo">aoah-opam-repository overlay</a> on <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">Tangled</a>. Each individual repository I publish also uses a <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo/blob/main/.tangled/workflows/build.yml">Spindle CI action</a> to build that library on every push. This overlay later became the input to <a href="https://anil.recoil.org/notes/aoah-2025-22">monopam</a> and <a href="https://anil.recoil.org/notes/aoah-2025-23">unpac</a> for assembling monorepos.</p>
<p>For every package I publish, I need to:</p>
<ul>
<li>get the metadata right in the <a href="https://dune.readthedocs.io/en/latest/howto/opam-file-generation.html">dune-project file</a> so that the opam files are generated right</li>
<li>add the right <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/.ocamlformat">.ocamlformat version</a> to the repo</li>
<li>then translate the per-repo opam metadata into an entry that lives in the <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo">aoah-opam-repo</a> so packages can depend on each other even though they're unreleased.</li>
<li>do various other health checks like copyright headers and <a href="https://opam.ocaml.org/packages/opam-dune-lint/">opam-dune-lint</a>.</li>
</ul>
<p>The problem with scripting these up is that there's subtle parameterisation which changes slightly across each library. Some might have an extra external dependency, another might have a custom build step, and others might just have tests that require specific actions. This seems ideal for a coding agent that can take a template of actions and then propose metadata changes automatically.</p>
<h3><a href="https://anil.recoil.org/news.xml#using-claude-skills-to-manage-opam-metadata" class="anchor" aria-hidden="true"></a>Using Claude skills to manage opam metadata</h3>
<p>The idea behind Claude skills is <a href="https://www.claude.com/blog/skills">incredibly simple</a>: you just create a <code>~/.claude/skills/my-skill</code> that contains a <code>SKILL.md</code> and the associated scripts that need to run. Nothing else; no state, no MCP server, just a simple file! The header looks as follows:</p>
<pre><code>—
name: ocaml-metadata
description: Standards for OCaml project metadata files. Use when initialising a new OCaml library/module, preparing for opam release, setting up testing infrastructure, or searching the OCaml ecosystem for dependencies. Not for normal code edits.
license: ISC
—

# OCaml Project Metadata Standards

## When to Use This Skill

Invoke this skill when:

1. **Initializing a new OCaml project** - Setting up dune-project, LICENSE, README, CI, etc.
2. **Preparing for opam release** - Ensuring all metadata is correct for publication
3. **Setting up testing infrastructure** - Especially for Eio-based libraries that need mock testing
4. **Searching the OCaml ecosystem** - Finding and fetching dependency sources for reference
5. **Adding third-party source references** - Using `opam source` to study library implementations

**Do not use for:**
- Regular code edits or bug fixes
- Simple function additions
- Refactoring existing code
</code></pre>
<p>The Yaml frontmatter is the only thing loaded by Claude at startup (thus saving context space). Then it uses that information to load the rest when needed on demand. For example, for the <a href="https://anil.recoil.org/notes/aoah-2025-4">claudeio</a> repository it spins it up on demand.</p>
<p><img src="https://anil.recoil.org/images/aoah-ss-4.webp" alt="%c" title="The OCaml metadata skill being loaded"></p>
<p>If you browse the <a href="https://tangled.org/anil.recoil.org/claude-ocaml-metadata">full skill repository</a> you will also find even more detailed (and personalised) instructions about how to structure tests and retrieve sources to my laptop. My workflow is that if Claude gets something wrong workflow-wise, I now prompt it to <em>also fix the skill</em> after its done. The agent then revises the workflow information in this repository and (hopefully) doesn't make the same mistake twice.</p>
<p><img src="https://anil.recoil.org/images/aoah-ss-3.webp" alt="%c" title="The skill working through bytesrw to fix up the metadata"></p>
<h2><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h2>
<p>Getting back to the problem at hand, testing the bytesrw-eio adapter was a simple matter of prompting the agent to
come up with some <a href="https://tangled.org/anil.recoil.org/ocaml-bytesrw-eio/blob/main/test/test_bytesrw_eio.ml">read and write tests</a>,
and then I deliberately introduced an error to sanity check they were failing
when expected and then refined the tests to include those failures as well.</p>
<p>One frustration here I had is a long-standing problem in OCaml where we don't
have a single agreed IO representation. Eio uses
<a href="https://github.com/mirage/ocaml-cstruct">Cstruct</a> which is backed by Bigarray
(which lives out-of-heap), and then others use <code>bytes</code>/<code>string</code> (which live
in-heap). The reason is usually because out-of-heap values can be non-moving
(allowing for zero-copy IO), but in return it results in <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">GC pacing issues</a> that are hard to fix.  This is relevant to
this Bytesrw-eio library because we introduce a copy between Bigarray (from
Eio) to Bytesrw (which obviously uses <code>bytes</code>). At some point in the future, I
reckon we need to get a non-moving <code>bytes</code> mechanism into OCaml 5.x so that we
can all just use one IO type and avoid these dratted copies.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>The bytesrw-eio package itself was simple enough, and it added to my long-term
todo list to <a href="https://anil.recoil.org/notes/icfp25-post-posix">revisit OCaml IO abstractions</a> again in the
future. The big breakthrough today though, was figuring out Claude skills. I'm
not using MCP at all any more, and will be creating more domain-specific skills
for OCaml actions as I proceed through the month.</p>
<p>Next, onto <a href="https://anil.recoil.org/notes/aoah-2025-6">Day 6</a> where we'll build a pure Yaml codec using this library!</p>

