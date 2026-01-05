---
title: 'AoAH Day 23: Unpac unifies git branching with package management'
description: Introducing unpac, a tool that unifies git and package management into
  a single workflow where all code dependencies live in one repository as trackable
  branches.
url: https://anil.recoil.org/notes/aoah-2025-23
date: 2025-12-23T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-unpac-ss-6.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Yesterday's <a href="https://anil.recoil.org/notes/aoah-2025-22">monopam</a> workflow used git submodules to combine
vendored packages, but was awkward to use for crosscutting changes involving
lots of vendored git repositories. Today I asked what agentic code development
would look like if we could unify <em>all code</em> into a single git repository,
where upstream packages become branches instead of submodules. I've
open-sourced the <strong><a href="https://tangled.org/anil.recoil.org/unpac">unpac</a></strong> CLI to
explore this, and have begun using it myself.</p>
<p>Coding agents work best when all relevant code is locally available so
they can grep and make <a href="https://anil.recoil.org/notes/aoah-2025-21">cross-cutting changes</a>. I first noticed this when building <a href="https://anil.recoil.org/notes/aoah-2025-9">Bonsai terminal UIs</a> and <a href="https://anil.recoil.org/notes/aoah-2025-10">Mosaic</a>, where I had to manually assemble monorepos just to get the agent working.
Things come to a crashing halt when package management
gets involved; the tool calls for web search are far slower and unreliable.
This means that the agent doesn't really have a good view on what third-party
packages might be useful to solve a problem, leading to the common complaint
that LLMs <a href="https://ryan.freumh.org/claude-code.html">reinvent the wheel</a>.</p>
<p>To fix this, unpac parses package metadata and materialises it into a git branch
structure <em>in a single repository</em> to make vendoring, patching, and updating
a native git workflow. Local changes can later be exported into git patches for
sending upstream, but in the meanwhile our agents can work on a single
repository.</p>
<p>The secret sauce to working on so many branches is to use <a href="https://git-scm.com/docs/git-worktree">git worktrees</a>, which allow multiple
branches to be checked out simultaneously from one git repo! I'll explain how unpac works next, and you can
browse a <a href="https://tangled.org/anil.recoil.org/unpac-work">working unpac tree</a>. You do end up with a
lot of git branches, which got me <a href="https://x.com/rhatr/status/1012001138110029824">banned from GitHub</a> back when I announced <a href="https://www.theregister.com/2017/01/17/docker_adds_continuous_integration_to_datakit/">Docker DataKit</a>. Luckily this time around I am hosting on <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">Tangled</a> where I host my own Git remotes and so don't have to worry about third-party SLAs!</p>
<p><a href="https://tangled.org/anil.recoil.org/unpac-work"> <img src="https://anil.recoil.org/images/aoah-unpac-ss-6.webp" alt="%c" title="All the dependent code is in separate branches in the git repo, managed by unpac"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#the-unpac-branching-model" class="anchor" aria-hidden="true"></a>The unpac branching model</h2>
<p>unpac organises code and dependencies using a lot of unrelated git branches, with
careful merging across them.  The <code>main</code> branch only holds the unpac metadata about which projects exist and which opam remotes to use.
While this defaults to the <a href="https://github.com/ocaml/opam-repository">upstream opam-repository</a>, I'm also using this with the <a href="https://github.com/oxcaml/opam-repository">oxcaml/opam-repository</a> and my <a href="https://tangled.org/anil.recoil.org/aoah-opam-repo">aoah-opam-repo</a> overlays (maintained via the <a href="https://anil.recoil.org/notes/aoah-2025-5">opam metadata skill</a>) to help track community forks.</p>
<pre><code>[opam]
compiler = "5.4.0"

[[opam.repositories]]
name = "opam"
path = "/workspace/opam/opam-repository"

[[opam.repositories]]
name = "aoah"
path = "/workspace/opam/aoah-opam-repo"
</code></pre>
<p>Each third-party opam package then has <em>three</em> branches in my unpac repo:</p>
<ul>
<li><code>upstream/opam/&lt;pkg&gt;</code> holds the unmodified upstream code and history</li>
<li><code>vendor/opam/&lt;pkg&gt;</code> is upstream history relocated to a <code>vendor/opam/&lt;pkg&gt;/</code> prefix using <a href="https://github.com/newren/git-filter-repo">git-filter-repo</a>.</li>
<li><code>patches/opam/&lt;pkg&gt;</code> is the vendor branch with any local changes applied.</li>
</ul>
<p>The projects you are working on are all in a <code>project/&lt;name&gt;</code> orphan git
branches, independent of all the others.  Adding a dependency to a project is a mere matter of
<em>merging</em> the vendor branch for a given dependency into the <code>project</code> branch.</p>
<p>This merging will materialise the dependency code in the project branch
conflict-free under <code>vendor/opam/</code>. This allows us to build a monorepo of OCaml
code that's maintained its git history across all the different developers,
while also allowing local commits to be held and rebased. The agent has a ton
of context available to it now without having to go to the outside world!</p>
<h3><a href="https://anil.recoil.org/news.xml#working-on-multiple-branches-simultaneously-with-worktrees" class="anchor" aria-hidden="true"></a>Working on multiple branches simultaneously with worktrees</h3>
<p>Before showing off the unpac CLI, it's worth explaining git worktrees as I'd
never used them before today.  Normally, you can only have one git branch
checked out at a time, but worktrees free us of that restriction:</p>
<blockquote>
<p>A git repository can support multiple working trees, allowing you to check
out more than one branch at a time. With <code>git worktree add</code> a new working
tree is associated with the repository, along with additional metadata that
differentiates that working tree from others in the same repository. The
working tree, along with this metadata, is called a "worktree".</p>
<p>This new worktree is called a "linked worktree" as opposed to the "main
worktree" prepared by git-init or git-clone. A repository has one main
worktree (if it's not a bare repository) and zero or more linked worktrees.
When you are done with a linked worktree, remove it with <code>git worktree remove</code>.
<cite>-- <a href="https://git-scm.com/docs/git-worktree">git-worktree documentation</a>, 2023</cite></p>
</blockquote>
<p>Creating them is pretty straightforward using <code>git worktree add</code>.  This creates
a new checkout with the <code>.git</code> entry being a file containing an entry
like:</p>
<pre><code>gitdir: /workspace/git/worktrees/tuatara
</code></pre>
<p>Without worktrees, agents fall over themselves switching branches due to the
requirement that all files be committed before switching. With worktrees, you
can have uncommitted stuff in multiple branches, meaning we can simultaneously
view upstream code while making patches, compare vendor and patches branches
side-by-side with diff, and work on multiple projects from the same repository.
I used to have these affordances back when I used Mercurial and Perforce (with
different distribution models, admittedly), so it's great to have it back!</p>
<h2><a href="https://anil.recoil.org/news.xml#using-unpac-via-the-cli" class="anchor" aria-hidden="true"></a>Using unpac via the CLI</h2>
<p>This elaborate git schema is all very good, but not something I'd want to manage
manually. The unpac CLI takes care of all the gruntwork involved,
including integrating an opam solver to create 100s of branches with one
command. Let's take a look at an example project:</p>
<pre><code class="language-bash">$ unpac init

# Vendors in packages from opam including dependencies
$ unpac add opam eio --solve
</code></pre>
<p>We now have a bunch of vendor branches in our local repository, and need to
create a project to use them:</p>
<pre><code class="language-bash"># Create a new project branch
$ unpac project new myapp

# Merge in the patch branches of these dependencies into project/myapp/vendor
$ unpac opam merge eio --solve myapp

# Hax0r like it's 2026 on your project
$ cd project/myapp
</code></pre>
<p><img src="https://anil.recoil.org/images/aoah-unpac-ss-1.webp" alt="%c" title="The unpac CLI solves package constraints and merges individual branches into a project"></p>
<h2><a href="https://anil.recoil.org/news.xml#doing-agentic-monorepo-development" class="anchor" aria-hidden="true"></a>Doing agentic monorepo development</h2>
<p>We can now do a simple <code>dune build</code> in the <code>project/myapp</code> directory since all our code is present in one working branch.
A typical unpac project looks like:</p>
<pre><code>.
└── dune-project
├── vendor/
│   ├── eio/            # vendored eio source
│   ├── lwt/            # vendored lwt source
│   └── ...
├── src/                # your project source
</code></pre>
<p>Dune <a href="https://anil.recoil.org/notes/aoah-2025-22">automatically discovers and builds</a> the vendored packages. No special configuration is needed beyond standard dune files.</p>
<p>However, some packages don't build with dune since the upstream projects don't use dune but have other build systems (quite reasonably! Choice is important). In the past, I have maintained over 50 <a href="https://github.com/dune-universe">dune ports</a> via opam overlays, mostly by hand. However, I can now easily use my coding agent to do all the porting automatically.</p>
<p><img src="https://anil.recoil.org/images/aoah-unpac-ss-4.webp" alt="%c" title="Claude can spawn parallel subagents using git worktrees to do the ports independently."></p>
<p>You can see some of the diffs in the patch branches in my working tree: <a href="https://tangled.org/anil.recoil.org/unpac-work/tree/opam%2Fpatches%2Flogs">patches/logs</a> or <a href="https://tangled.org/anil.recoil.org/unpac-work/tree/opam%2Fpatches%2Fcmdliner">patches/cmdliner</a> or <a href="https://tangled.org/anil.recoil.org/unpac-work/tree/opam%2Fpatches%2Fbos">patches/bos</a> for example. Since the agent has a clean local interface to work with, it can keep its commits neatly organised.</p>
<p>An <code>unpac vendor status</code> command neatly summarises the status of which packages have been patched, and which project they're merged into:</p>
<pre><code class="language-bash">$ unpac vendor status
Package                    Patches   Merged into
----------------------------------------------------------------------
angstrom                         0   -
asn1-combinators                 0   myapp
astring                          0   -
base64                           0   myapp
bigstringaf                      0   -
bos                              1   -
bytesrw                          0   -
bytesrw-eio                      0   -
ca-certs                         0   -
checkseum                        0   -
cmdliner                         1   tuatara
conpool                          0   -
cookeio                          0   -
csexp                            0   myapp
cstruct                          0   -
decompress                       0   -
digestif                         0   myapp, tuatara
domain-local-await               0   -
domain-name                      0   myapp
</code></pre>
<p>You can see here that <code>cmdliner</code> has been patched and merged into the tuatara
project, but <code>bos</code> has been patched and is unmerged. This is all calculated
internally via git commands, so there's no separate metadata store to get out
of sync.</p>
<p>I haven't completed porting all the third-party packages to use dune just yet,
but I've left it running overnight. When that's done, the big feature we gain
is that a dune build can seamlessly cross-compile binaries since all the OCaml
code and C bindings are in one place. This is what <a href="https://mirage.io/docs/mirage-4">MirageOS
4</a> does, and we can reap the benefits now for
conventional binaries too. Windows builds should also be a lot easier as long
as the dune rules don't have too many Unixisms.</p>
<h3><a href="https://anil.recoil.org/news.xml#importing-existing-projects" class="anchor" aria-hidden="true"></a>Importing existing projects</h3>
<p>The unpac CLI was self-explaining enough that another agent session could import
an OCaml project by analysing that project and running the sequence of unpac
commands.</p>
<p><img src="https://anil.recoil.org/images/aoah-unpac-ss-2.webp" alt="%c" title="Importing and vendoring all the code needed for an existing project using Claude"></p>
<p>In order to reduce the load on external git clones, unpac also supports having a local "git branch cache" which pulls remotes just once, and then all unpac invocations pull from that local store. As an experiment over the holidays, I've left a session doing a slow clone of <em>all</em> opam git remotes, to see how well git scales to a few thousand branches.</p>
<h3><a href="https://anil.recoil.org/news.xml#pushing-the-results" class="anchor" aria-hidden="true"></a>Pushing the results</h3>
<p>We do end up with 100s of local branches, and so an <code>unpac push</code> command checks which ones need pushing and takes care of it for you.</p>
<p><img src="https://anil.recoil.org/images/aoah-unpac-ss-3.webp" alt="%c"></p>
<p>You can browse one of my working unpac repositories on
<a href="https://tangled.org/anil.recoil.org/unpac-work">tangled/anil.recoil.org/unpac-work</a>
to get a sense of the structure.</p>
<p>I'm still working on the pulling/rebasing functionality, but the basic idea is
all the same: pull from the outside world into a pristine branch, relocate the
directory, and then have local patch branches.</p>
<h2><a href="https://anil.recoil.org/news.xml#integrating-ocaml-with-other-languages-in-one-repo" class="anchor" aria-hidden="true"></a>Integrating OCaml with other languages in one repo</h2>
<p>The current unpac focuses on opam, but <a href="https://ryan.freumh.org">Ryan Gibb</a> has been leading the research on a <a href="https://anil.recoil.org/papers/2025-hyperres">generalised packaging language</a> that can describe package management <em>across</em> ecosystems. Imagine something like this in a future unpac:</p>
<pre><code class="language-toml"># Works with opam, npm, cargo, pip...
dependencies = [
  { source = "opam", name = "eio", version = "&gt;=1.0" },
  { source = "npm", name = "d3", version = "^7.0" },
  { source = "cargo", name = "tokio", version = "1.0" },
]
</code></pre>
<p>I've already had need for this last week when I <a href="https://anil.recoil.org/notes/aoah-2025-13">vibespiled 50 HTTP libraries</a> across 10 languages into an OCaml implementation. I really want to be able to more easily draw from other language ecosystems, and unpac's git branch model works regardless of the package manager (hence the <code>opam/</code> suffix for <code>vendor/</code> branches).</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p><a href="https://x.com/rhatr/status/1012001138110029824"> <img src="https://anil.recoil.org/images/aoah-unpac-ss-5.webp" alt="%rc" title="I did not get banned from anything while writing this post"> </a></p>
<p>Unpac's branching model actually doesn't work hugely well with GitHub due to the storage limits on an account being hit pretty fast, but it's peachy when used with self-hosted Git services. I'm sure we could do something with Git object alternates as well to improve on this in the future.</p>
<p>There's quite a lot of work required to make unpac production grade, but I'm astounded by how quickly I could put this prototype together in a day. Sketching out CLI tools and cram tests is extraordinarily fun as well, as I could specify my desired user interface and then engage in a Socratic dialogue with the agent to refine the specification.</p>
<p>I'm also having subversive thoughts now about issue management. I've been a fan for many years of <a href="https://github.com/janestreet/iron">Jane Street's Iron</a> code review system. However, despite having talked to Stephen Weeks and <a href="https://github.com/yminsky">Yaron Minsky</a> extensively about it over the years, I've never found the bandwidth to build an equivalent for open source. But with coding agents being able to interpret natural language alongside code, it seems like a really obvious extension to also store issues within branches as well as code, and to unify our agent context horizon. Something for the 2026 queue!</p>
<p>I'd love to hear any feedback on unpac's model from other projects. I wouldn't use the tool I've released just yet as it's only about 18 hours old, but I'll work on it more in the new year as well and do a proper release once it's self hosting. Many thanks to <a href="https://ryan.freumh.org">Ryan Gibb</a> and <a href="https://patrick.sirref.org">Patrick Ferris</a> (who came up with the name) for several design discussions that lead to this post.</p>
<h2><a href="https://anil.recoil.org/news.xml#feedback-25th-dec-2025" class="anchor" aria-hidden="true"></a>Feedback (25th Dec 2025)</h2>
<p>Some useful comments about this post:</p>
<p>First, <a href="https://github.com/edwintorok">Török Edwin</a> from the <a href="https://anil.recoil.org/projects/xen">XenServer</a> team <a href="https://discuss.systems/@edwintorok/115777291897330349">reports</a> that:</p>
<blockquote>
<p>XAPI has 2 invalid commits in its commit history (invalid email address
containing a duplicate of the author name and email), so it is impossible to
push its history into a brand new GitHub repo, you can only do that by
forking the original repo through the API.  Although I see you are not using
GitHub, so you might be fine. Still might be a good idea to run <code>git fsck</code> on
the final repo, XAPI may not be the only project with some invalid commits in
its history, and that could create problems later (e.g. if <code>tangled</code> starts
running <code>git fsck</code>)".</p>
</blockquote>
<p>Something to investigate for the new year: antagonistic git remotes! Then, <a href="https://mynameismwd.org">Michael Dales</a> <a href="https://toot.mynameismwd.org/@michael/statuses/01KDAB3RCJB7FVZQK17WZTJ767">notes that</a>:</p>
<blockquote>
<p>For any production system I’ve had to deliver, I’d always vendor in third
part code using git submodules (and those taken from a local mirror of each
repo). You just can’t use external dependencies if you need to know you can
ship updates to customers at the drop of a hat, and you never know when an
open source project might go away.</p>
</blockquote>
<p>And then <a href="https://github.com/djs55">Dave Scott</a> reminded me how much this proposed branch scheme makes it easier for shipping
products to comply with open-source licenses. In <a href="https://anil.recoil.org/papers/2025-docker-icfp">Docker for Desktop</a> we have an elaborate <a href="https://github.com/moby/vpnkit/tree/master/scripts">license gathering
script</a> for the 'about' box
to credit contributors.</p>

