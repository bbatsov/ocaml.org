---
title: 'AoAH Day 3: XDG filesystem paths using Eio capabilities'
description: Building an XDG Base Directory Specification library with Eio capabilities
  and Cmdliner integration, providing sandboxed filesystem access patterns with full
  environment variable and CLI override support.
url: https://anil.recoil.org/notes/aoah-2025-3
date: 2025-12-03T00:00:00-00:00
preview_image: https://anil.recoil.org/images/cross-xdg.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>By Day 3 of the <a href="https://anil.recoil.org/notes/aoah-2025">Advent of Agentic Humps</a>, I now have the
confidence to build a slightly more complex library that uses
<a href="https://github.com/ocaml-multicore/eio">Eio</a> to implement the <a href="https://specifications.freedesktop.org/basedir/latest/">XDG Base
Directory Specification</a> with a twist: let's use <a href="https://anil.recoil.org/papers/2023-ocaml-eio">Eio capabilities</a> to sandbox XDG paths by default.</p>
<p><img src="https://anil.recoil.org/images/cross-xdg.webp" alt="%c" title="Lots of other languages have cross-platform XDG implementations, so I wanted one in OCaml for Eio"></p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>The <a href="https://specifications.freedesktop.org/basedir/latest/">XDG Spec</a> is
comprehensive, but full of lots of informal rules about directories. Some of
the rules are pretty easy to follow:</p>
<blockquote>
<p><code>$XDG_DATA_DIRS</code> defines the preference-ordered set of base directories to
search for data files in addition to the <code>$XDG_DATA_HOME</code> base directory. The
directories in <code>$XDG_DATA_DIRS</code> should be separated with the separator used
for <code>$PATH</code> on the platform (typically this is a colon <code>:</code>).</p>
</blockquote>
<p>While others are harder to enforce in code:</p>
<blockquote>
<p>The directory MUST be on a local file system and not shared with any other
system. The directory MUST be fully-featured by the standards of the
operating system. More specifically, on Unix-like operating systems <code>AF_UNIX</code>
sockets, symbolic links, hard links, proper permissions, file locking, sparse
files, memory mapping, file change notifications, a reliable hard link count
must be supported, and no restrictions on the file name character set should
be imposed. Files in this directory MAY be subjected to periodic clean-up. To
ensure that your files are not removed, they should have their access time
timestamp modified at least once every 6 hours of monotonic time or the
'sticky' bit should be set on the file.</p>
</blockquote>
<p>We can do bits of that, but POSIX doesn't really expose everything we need to
mechanically verify some aspects of filesystem support. Still, we have a big
long spec, so let's see what happens if we throw an agent at it!</p>
<p>My general approach with Claude was to download a copy of the XDG spec, instruct
the agent to digest it. But then, I also supplied the <a href="https://anil.recoil.org/notes/aoah-2025-1">previous</a>
<a href="https://anil.recoil.org/notes/aoah-2025-2">two</a> libraries as examples of "my OCaml style" that it could draw
from. Having more code available to act as an oracle to guide the model towards
something I find acceptable is a useful way to avoid lots of prompting.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>The first attempts with using completely crashed and burnt as the agent failed
to grok the admittedly very complicated Eio types for capabilities (which do involve
subtyping and phantom polymorphic variants). I then cloned the Eio source code
and specifically instructed the agent to read the <a href="https://github.com/ocaml-multicore/eio/blob/main/README.md">Eio README</a>,
as it has extensive information about best practises to use the library. This is
similar to my earlier trick with <a href="https://anil.recoil.org/notes/aoah-2025-2">jsont</a> to instruct it to read
the cookbooks.</p>
<p>Things got much better after this; the agent had to iterate quite a few times,
but did eventually converge on the right types.</p>
<pre><code class="language-ocaml">val config_dir : t -&gt; Eio.Fs.dir_ty Eio.Path.t
(** [config_dir t] returns the path to user-specific configuration files.

    {b Purpose:} Store user preferences, settings, and configuration files.
    Configuration files should be human-readable when possible.

    {b Environment Variables:}
    - [${APP_NAME}_CONFIG_DIR]: Application-specific override (highest priority)
    - [XDG_CONFIG_HOME]: XDG standard variable
    - Default: [$HOME/.config/{app_name}]

    @see &lt;https://specifications.freedesktop.org/basedir-spec/latest/#variables&gt;
      XDG_CONFIG_HOME specification *)
</code></pre>
<p>You can see one nice feature here that would have taken a while to code by
hand.  The <code>type t</code> for the XDGe library is typically constructed by exposing <a href="https://github.com/dbuenzli/cmdliner">Cmdliner</a> terms to allow
other applications to "plug in" XDG support by just specifying a <a href="https://tangled.org/anil.recoil.org/xdge/blob/v1.0.0/lib/xdge.mli#L347">single term</a>.</p>
<p>This term takes care of adding  of the ordering of environment variable, command-line arguments,
and default values in the right order. The manual page for an example binary shows how this works from the CLI.</p>
<p><img src="https://anil.recoil.org/images/xdge-man-page.webp" alt="%c" title="In this case, the library specifies XDG_EXAMPLE as the app name, but this is easily customised to your app"></p>
<h2><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h2>
<p>You can see the Cmdliner support in action with the test cases, where I adopted
the same cram-based testing strategy as earlier with <a href="https://anil.recoil.org/notes/aoah-2025-2">jsont</a>.</p>
<p>This allows for a nice repository structure where I can simply add in the XDG
Cmdliner term to the test case binaries, and have all the gory details of config setup
handled by the xdge library. For example, in the <a href="https://tangled.org/anil.recoil.org/xdge/blob/main/test/xdg.t">cram tests</a> you can
see how the <em>source</em> of a XDG path is tracked (i.e. did it come from the CLI, from an environment variable or the defaults?):</p>
<pre><code class="language-bash"> $ export HOME=./test_home
  $ unset XDG_CONFIG_DIRS XDG_DATA_DIRS
  $ XDG_CONFIG_HOME=/tmp/xdge/xdg-config \
  &gt; XDG_EXAMPLE_CONFIG_DIR=./app-config \
  &gt; ./xdg_example.exe --config-dir ./cli-config
  === Cmdliner Config ===
  XDG config:
  config_dir: ./cli-config [cmdline]
  
  === XDG Directories ===
  XDG directories for 'xdg_example':
  User directories:
    config: &lt;fs:./test_home/./cli-config&gt; [cmdline]
    data: &lt;fs:./test_home/./test_home/.local/share/xdg_example&gt; [default]
    cache: &lt;fs:./test_home/./test_home/.cache/xdg_example&gt; [default]
    state: &lt;fs:./test_home/./test_home/.local/state/xdg_example&gt; [default]
    runtime: &lt;none&gt; [default]
  System directories:
    config_dirs: [&lt;fs:/etc/xdg/xdg_example&gt;]
    data_dirs: [&lt;fs:/usr/local/share/xdg_example&gt;; &lt;fs:/usr/share/xdg_example&gt;]

Command-line argument overrides both types of environment variables. Even
though both XDG_CONFIG_HOME and XDG_EXAMPLE_CONFIG_DIR are set, the
--config-dir flag takes precedence and shows [cmdline] source. Other
directories fall back to defaults since no other command-line args are
provided.
</code></pre>
<p>I ran the cram tests quite a few times and read through them to make sure
the shell sessions and explanations made sense, and then read through
the xdge source code itself which was pretty simple. There are some
features which Eio doesn't expose functionality for yet that are OS-specific
(like checking the mount type), but they can come in a future iteration.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>Cmdliner is one of the gems in the open-source OCaml community due to how
easy it makes it to build "Unix-style" applications with sensible manual
pages and consistent argument parsing. However, even after using it for
years I can never remember its term language without referring to the manual,
and I always find myself cut-and-pasting from previous code and editing it.</p>
<p>Using the agent definitely helped me out here. A lot of the XDG logic
is fairly boilerplate, but extremely useful to express in a typed way.
I anticipate now using this library in almost every CLI tool I build
in OCaml, as it has enough information exposed in the interface to let
downstream CLI-coding agents pick the right base directories to use by
default.</p>
<p>Onto <a href="https://anil.recoil.org/notes/aoah-2025-4">Day 4</a> then, where we'll go recursive by wrapping Claude in OCaml using Claude!</p>

