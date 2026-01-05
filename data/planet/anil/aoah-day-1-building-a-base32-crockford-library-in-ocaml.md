---
title: 'AoAH Day 1: Building a Base32 Crockford library in OCaml'
description: Building a Base32 Crockford encoding library in OCaml using Claude Code,
  establishing the development workflow with sandboxed Docker containers and local
  development environments.
url: https://anil.recoil.org/notes/aoah-2025-1
date: 2025-12-01T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-ss-5.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Let's start day 1 of the <a href="https://anil.recoil.org/notes/aoah-2025">Advent of Agentic Humps</a> with a gentle introduction to agentic coding. Firstly, I've chosen to exclusively use <a href="https://www.claude.com/product/claude-code">Claude Code</a> for this since it's CLI driven. I tried some of the other Copilot and Cursor IDEs, but I just couldn't adjust to how busy the displays were.</p>
<p>With Claude, my setup first involved a custom devcontainer using Docker on a Linux host, and my local Mac laptop. I coordinate both of these via Git repositories hosted up at <a href="https://tangled.org">Tangled</a> with a <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">self-hosted knot</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#remote-sandboxed-docker-devcontainer" class="anchor" aria-hidden="true"></a>Remote sandboxed Docker devcontainer</h2>
<p>I setup a remote Docker container on a Linux host where I can leave the agent running unattended in a sandbox. I coded up a <a href="https://tangled.org/anil.recoil.org/slop/tree/main/.devcontainer">custom Claude devcontainer</a> for this which takes care of firewalling the agent at the DNS level and installing <a href="https://tangled.org/anil.recoil.org/slop/blob/main/.devcontainer/setup-ocaml.sh">relevant OCaml packages</a>.</p>
<p>With this setup, I can run <code>claude --dangerously-skip-permissions</code> and have it only access my repositories on <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">Tangled</a> and GitHub.</p>
<p>As a bit of fun, the wrapper <a href="https://tangled.org/anil.recoil.org/slop/blob/main/slopper">slopper script</a> to start the session passes the session shell output back to Claude afterwards, to give it an opportunity to rewrite the Dockerfile with additional libraries I installed. It's shaky, but it occasionally does something useful.</p>
<p><img src="https://anil.recoil.org/images/aoah-ss-5.webp" alt="%c" title="The Claude devcontainer works great with tmux and Docker sandboxing"></p>
<h2><a href="https://anil.recoil.org/news.xml#local-development" class="anchor" aria-hidden="true"></a>Local development</h2>
<p>On my Mac and Linux desktops, I just use Claude code directly with my usual vi-based OCaml coding setup. This is just a straightforward addition to my usual workflow with Claude running in a terminal.</p>
<p>The only major change is that I assume that all code generated is slop by default, and is so until "promoted" via review and manual editing to the next level. I therefore stage all my work-in-progress slop code in a dedicate <a href="https://tangled.org/anil.recoil.org/slop">slop repository</a> which I'll reset and delete regularly. This is a convenient way of doing coordination, nothing more.</p>
<h2><a href="https://anil.recoil.org/news.xml#getting-started-with-the-first-library-crockford-base32" class="anchor" aria-hidden="true"></a>Getting started with the first library: Crockford base32</h2>
<p><strong>Problem.</strong> I've integrated my blog <a href="https://anil.recoil.org/notes/principles-for-collective-knowledge">with Rogue Scholar</a> but the mechanism for doing so required generating <a href="https://www.crockford.com/base32.html">Base32 encoded random DOIs</a> for the Rogue Scholar DOI assigner to pick up from my blog feed. There is no existing Base32 library in OCaml, so let's make one.</p>
<p><strong>Approach.</strong>
I looked over at the <a href="https://www.crockford.com/base32.html">spec</a>, and also (with permission from Martin Fenner) at his reference <a href="https://pkg.go.dev/github.com/front-matter/commonmeta/crockford">Go implementation</a> on GitHub. I then instructed Claude to synthesise a spec from the online spec <em>and</em> the Go implementation, and used Sonnet 4.5 to come up with a plan. I then instructed it to write the OCaml implementation as a single module with no dependencies other than the stdlib. I then <em>separately</em> had a session to take just the OCaml interface file and synthesise alcotest-based checks from the spec, to act as test cases.</p>
<p><strong>Results.</strong> The <a href="https://tangled.org/anil.recoil.org/ocaml-crockford">crockford code</a> is up with <a href="https://ocaml.org/p/crockford/latest">online docs</a>. I assigned co-copyright to Front Matter due to cross-generating the code with their implemenation as a reference. Unsure if this is the right thing to do, but it seems better to be generous about copyright assignment in this case.</p>
<p><strong>Reflections.</strong> This is a pretty simple day 1 task, as it's a single module of OCaml and relatively easy to test. Still, I had to be careful to not just generate the tests in the same session as the code to ensure that the agent didn't just specialise the tests to the code it had written.</p>
<p>Onto <a href="https://anil.recoil.org/notes/aoah-2025-2">Day 2</a> then, where we'll build a more substantive JSONfeed library!</p>

