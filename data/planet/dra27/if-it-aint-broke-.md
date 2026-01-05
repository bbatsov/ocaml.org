---
title: "If it ain\u2019t broke, \u2026"
description: "Someone was kind enough a few weeks ago to comment on Hacker News that
  \u201COpam on Windows is a masterpiece of engineering\u201D. There\u2019s certainly
  a lot which has to go on under the hood to create what I believe is the necessary
  baseline experience. Unfortunately, it doesn\u2019t always work out."
url: https://www.dra27.uk/blog/platform/2025/12/01/if-it-aint-broke.html
date: 2025-12-01T00:00:00-00:00
preview_image:
authors:
- ""
source:
---

<p>Someone was kind enough a few weeks ago to comment on <a href="https://news.ycombinator.com/item?id=45855726">Hacker News</a>
that “Opam on Windows is a masterpiece of engineering”. There’s certainly a lot
which has to go on under the hood to create what I believe is the necessary
baseline experience. Unfortunately, it doesn’t <em>always</em> work out.</p>

<p>10 days ago, a new user popped up on our <a href="https://discuss.ocaml.org/t/when-following-install-instructions-for-windows-topkg-and-ocp-indent-fail-to-compile/17514">Discourse forum</a>
with a failure of some core packages to work on Windows. The OP was using MSYS2,
which at the moment has slightly less support than the recommended Cygwin, but
it looked to me like either my <a href="https://github.com/ocaml/opam-repository/pull/28897">ocaml/opam-repository#28897</a>
or <a href="https://github.com/ocaml/ocamlfind/pull/112">ocaml/ocamlfind#112</a> ought to
fix things.</p>

<p>Except it didn’t, and it wasn’t until today that I was able to reproduce the
problem, and figure out what was going on. As it happens, once I could reproduce
it, this was easy for <em>me</em> to work out, but it shines an interesting light on to
a subtle bit of opam’s Windows internals and consequently something that either
the OP or a packager has misunderstood and it also finally goads me into
standing on my soapbox and shouting from the rafters:</p>

<p><strong>Code should be changed if and only if a bug is being fixed or an actual
user-facing feature is being implemented! The risk of regression ALWAYS
outweighs the benefit of refactoring UNLESS you’re actually changing
something else.</strong></p>

<p>The target of my particular ire is the wonderful <a href="https://www.shellcheck.net/">ShellCheck</a>
tool. If you don’t use it already, then:</p>

<ol>
  <li>You don’t have to write shell scripts, and should continue not having to
write shell scripts for as long as possible! However, you may have to one
day, so:</li>
  <li>You should absolutely start using it <em>on the next shell script you have to
write</em>. You should also use it to check changes you make to existing scripts,
too.</li>
</ol>

<p>But what I’m finally going to come out and just say, is that you should <em>never</em>
<strong>ever</strong> change a shell script just to please ShellCheck unless you actually
have a bug report. If ShellCheck grumbles and warns something <em>might</em> cause a
bug, then take it as a challenge - figure out the scenario in which that could
be triggered, test it, fix it. If, as is often the case, it’s warning about an
unhygienic construct which <em>could</em> go wrong but actually never will because, for
example, the file names concerned don’t contain spaces and never will, then
leave that script alone!</p>

<p>What was actually going on here? The OP was seeing this effect when trying to
install <a href="https://ocaml.org/p/topkg/latest">topkg</a>:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C:\Users\DRA&gt;opam install topkg
The following actions will be performed:
=== install 1 package
  ∗ topkg      1.1.1

&lt;&gt;&lt;&gt; Processing actions &lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;  🐫
⬇ retrieved topkg.1.1.1  (https://opam.ocaml.org/cache)
[ERROR] The compilation of topkg.1.1.1 failed at "ocaml pkg/pkg.ml build --pkg-name topkg --dev-pkg false".

#=== ERROR while compiling topkg.1.1.1 ========================================#
# context     2.5.0 | win32/x86_64 | ocaml.4.14.2 | https://opam.ocaml.org#9bc1691da5d727ede095f83c019b92087a7da33e
# path        C:\Devel\Roots\msys2-testing-native\default\.opam-switch\build\topkg.1.1.1
# command     C:\Devel\Roots\msys2-testing-native\default\bin\ocaml.exe pkg/pkg.ml build --pkg-name topkg --dev-pkg false
# exit-code   125
# env-file    C:\Devel\Roots\msys2-testing-native\log\topkg-8308-179b77.env
# output-file C:\Devel\Roots\msys2-testing-native\log\topkg-8308-179b77.out
### output ###
# Exception: Fl_package_base.No_such_package ("findlib", "").
</code></pre></div></div>

<p>This error gets readily seen when using OCaml 5.x on MSYS2. It’s not a problem
with topkg, it’s actually that the <a href="https://ocaml.org/p/ocamlfind/latest">ocamlfind</a>
library manager is not installed correctly. There are fixes pending for that in
<a href="https://github.com/ocaml/ocamlfind/pull/112">ocaml/ocamlfind#112</a>, but the
issue there is to do with OCaml <strong>5.x</strong> - this build is OCaml 4.14.2.</p>

<p>The actual issue is readily apparent if we rebuild ocamlfind:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C:\Users\DRA&gt;opam reinstall ocamlfind -v
The following actions will be performed:
=== recompile 1 package
  ↻ ocamlfind 1.9.8

&lt;&gt;&lt;&gt; Processing actions &lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;&lt;&gt;  🐫
⬇ retrieved ocamlfind.1.9.8  (cached)
+ C:\Devel\Roots\msys2-testing-native\default\.opam-switch\build\ocamlfind.1.9.8\./configure "-bindir" "C:\\Devel\\Roots\\msys2-testing-native\\default\\bin" "-sitelib" "C:\\Devel\\Roots\\msys2-testing-native\\default\\lib" "-mandir" "C:\\Devel\\Roots\\msys2-testing-native\\default\\man" "-config" "C:\\Devel\\Roots\\msys2-testing-native\\default\\lib/findlib.conf" "-no-custom" "-no-camlp4" (CWD=C:\Devel\Roots\msys2-testing-native\default\.opam-switch\build\ocamlfind.1.9.8)
- Welcome to findlib version 1.9.8
- Configuring core...
&lt;snip snip snip&gt;
- Configuration for threads written to site-lib-src/threads/META
- Configuration for str written to site-lib-src/str/META
- Configuration for bytes written to site-lib-src/bytes/META
- Access denied - SRC
- File not found - -NAME
- File not found - META.IN
- File not found - -TYPE
- File not found - F
- File not found - -EXEC
- File not found - SH
- File not found - -C
- File not found - SED -E 'S/@VERSION@/1.9.8/G'         -E 'S/@REQUIRES@//G'    \$1\
- File not found -  &gt; \${1%.IN}\
- File not found - SH
- File not found - {}
- File not found - ;
- Detecting compiler arguments: (extractor built) ok
</code></pre></div></div>

<p>What’s going on at the end of that snippet? It turns out ocamlfind’s <code class="language-plaintext highlighter-rouge">configure</code>
script contains a call to what it expects to be POSIX <a href="https://pubs.opengroup.org/onlinepubs/9799919799/utilities/find.html"><code class="language-plaintext highlighter-rouge">find</code></a>,
but what it’s actually getting is the Windows <code class="language-plaintext highlighter-rouge">find</code> utility. This problem is as
old as the hills, and affects <code class="language-plaintext highlighter-rouge">sort</code> as well:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C:\Users\DRA&gt;find /?
Searches for a text string in a file or files.

FIND [/V] [/C] [/N] [/I] [/OFF[LINE]] "string" [[drive:][path]filename[ ...]]

  /V         Displays all lines NOT containing the specified string.
  /C         Displays only the count of lines containing the string.
  /N         Displays line numbers with the displayed lines.
  /I         Ignores the case of characters when searching for the string.
  /OFF[LINE] Do not skip files with offline attribute set.
  "string"   Specifies the text string to find.
  [drive:][path]filename
             Specifies a file or files to search.

If a path is not specified, FIND searches the text typed at the prompt
or piped from another command.
</code></pre></div></div>

<p>versus:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C:\Users\DRA&gt;C:\msys64\usr\bin\find --version
find (GNU findutils) 4.10.0
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later &lt;https://gnu.org/licenses/gpl.html&gt;.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Eric B. Decker, James Youngman, and Kevin Dalley.
Features enabled: D_TYPE O_NOFOLLOW(enabled) LEAF_OPTIMISATION FTS(FTS_CWDFD) CBO(level=1)
</code></pre></div></div>

<p>Now, it turns out that the OP’s system is misconfigured - which is part of the
reason why <a href="https://cygwin.com">Cygwin</a>, <a href="https://www.msys2.org">MSYS2</a> and
<a href="https://gitforwindows.org/">Git for Windows</a> don’t put their Unix shell
utilities into <code class="language-plaintext highlighter-rouge">PATH</code> by default. The Unix utilities are in <code class="language-plaintext highlighter-rouge">C:\msys64\usr\bin</code>,
but this entry in <code class="language-plaintext highlighter-rouge">PATH</code> appears after <code class="language-plaintext highlighter-rouge">C:\WINDOWS\system32</code>, so a few
utilities get shadowed. For things like <code class="language-plaintext highlighter-rouge">curl</code> and <code class="language-plaintext highlighter-rouge">tar</code>, this is <em>usually</em>
benign, but for <code class="language-plaintext highlighter-rouge">find</code> and <code class="language-plaintext highlighter-rouge">sort</code> it’s pretty terminal!</p>

<p>In its default mode, where opam manages the Cygwin or MSYS2 environment for
you - it’s part of a mildly complex process in <a href="https://github.com/ocaml/opam/pull/5832">ocaml/opam#5832</a>,
as opam ensures that either the Cygwin or MSYS2 <code class="language-plaintext highlighter-rouge">bin</code> directory is just “left”
enough in <code class="language-plaintext highlighter-rouge">PATH</code> to ensure that some key utilities are not shadowed, but not so
far to the left that it shadows things like Git for Windows!</p>

<p>However, if you’ve set-up <code class="language-plaintext highlighter-rouge">PATH</code> yourself, as the OP had, you’re on your own,
I’m afraid!</p>

<p>So why is all this making me rant about ShellCheck? Well, the problem is that
<em>downgrading</em> ocamlfind to a previous version works just fine, and so it’s the
regression in ocamlfind 1.9.8 that I want to focus on. The failing part of the
script is:</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># create META from META.in in POSIX-compatible &amp; safe way</span>
<span class="c"># see: https://www.shellcheck.net/wiki/SC2044</span>
<span class="nv">meta_subst</span><span class="o">=</span><span class="s2">"sed -e 's/@VERSION@/</span><span class="nv">$version</span><span class="s2">/g' </span><span class="se">\</span><span class="s2">
        -e 's/@REQUIRES@/</span><span class="k">${</span><span class="nv">req_bytes</span><span class="k">}</span><span class="s2">/g' </span><span class="se">\</span><span class="s2">
        </span><span class="se">\"\$</span><span class="s2">1</span><span class="se">\"</span><span class="s2"> &gt; </span><span class="se">\"\$</span><span class="s2">{1%.in}</span><span class="se">\"</span><span class="s2">"</span>
find src <span class="nt">-name</span> <span class="s1">'META.in'</span> <span class="nt">-type</span> f <span class="nt">-exec</span> sh <span class="nt">-c</span> <span class="s2">"</span><span class="nv">$meta_subst</span><span class="s2">"</span> sh <span class="o">{}</span> <span class="se">\;</span>
</code></pre></div></div>

<p>but which was originally (from <a href="https://github.com/ocaml/ocamlfind/commit/cbc4a7db226670c12ee2891213593559bd694bbf">cbc4a7d</a>):</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for </span>part <span class="k">in</span> <span class="sb">`</span><span class="nb">cd </span>src<span class="p">;</span> <span class="nb">echo</span> <span class="k">*</span><span class="sb">`</span><span class="p">;</span> <span class="k">do
    if</span> <span class="o">[</span> <span class="nt">-f</span> <span class="s2">"src/</span><span class="nv">$part</span><span class="s2">/META.in"</span> <span class="o">]</span><span class="p">;</span> <span class="k">then
        </span><span class="nb">sed</span> <span class="nt">-e</span> <span class="s2">"s/@VERSION@/</span><span class="nv">$version</span><span class="s2">/g"</span> <span class="se">\</span>
            <span class="nt">-e</span> <span class="s2">"s/@REQUIRES@/</span><span class="k">${</span><span class="nv">req_bytes</span><span class="k">}</span><span class="s2">/g"</span> <span class="se">\</span>
            src/<span class="nv">$part</span>/META.in <span class="o">&gt;</span>src/<span class="nv">$part</span>/META
    <span class="k">fi
done</span>
</code></pre></div></div>

<p>ShellCheck reports two issues with that original loop:</p>
<ol>
  <li>It dislikes the <a href="https://www.shellcheck.net/wiki/SC2006">use of backticks rather than <code class="language-plaintext highlighter-rouge">$(...)</code></a></li>
  <li>It worries that the <a href="https://www.shellcheck.net/wiki/SC2164">exit code of <code class="language-plaintext highlighter-rouge">cd</code> is unchecked</a></li>
</ol>

<p>The first is because this script is very old, and it was written at a time when
there were not only versions of <code class="language-plaintext highlighter-rouge">sh</code> which didn’t support <code class="language-plaintext highlighter-rouge">$(...)</code> notation,
there were people using OCaml on them! In fact, it’s only comparatively recently
that the default version of <code class="language-plaintext highlighter-rouge">sh</code> in (Open)Solaris/Illumos, etc. actually
supports it.</p>

<p>The second is true, but:</p>
<ol>
  <li>It’s never going to happen (the <code class="language-plaintext highlighter-rouge">src</code> directory be missing from the
tarball?!)</li>
  <li>Even if it it does, the loop will <em>correctly</em> do nothing, and the build will
fail later</li>
</ol>

<p>So, while <code class="language-plaintext highlighter-rouge">for part in `cd src; echo *`; do</code> is somewhat showing its age, in the
context of this specific fragment of this specific shell script, that will
<strong>never ever</strong> cause a problem. Writing a new script? Don’t write that! Adding a
new part to an existing script? Don’t write that!</p>

<p>However, it is now broken, and my proposed fix is that actually the use of the
wildcard was completely unnecessary, as we already compute the relevant list of
directories which can possibly contain a <code class="language-plaintext highlighter-rouge">META.in</code> file slightly later in the
script (taken from <a href="https://github.com/ocaml/ocamlfind/pull/112/commits/129a259141fa3f1af157eef3100b74866bf90795">1291259</a>):</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for </span>part <span class="k">in</span> <span class="nv">$parts</span><span class="p">;</span> <span class="k">do
  if</span> <span class="o">[</span> <span class="nt">-f</span> src/<span class="s2">"</span><span class="nv">$part</span><span class="s2">"</span>/META.in <span class="o">]</span><span class="p">;</span> <span class="k">then
    </span><span class="nb">sed</span> <span class="nt">-e</span> <span class="s2">"s/@VERSION@/</span><span class="nv">$version</span><span class="s2">/g"</span> <span class="se">\</span>
        <span class="nt">-e</span> <span class="s2">"s/@REQUIRES@/</span><span class="k">${</span><span class="nv">req_bytes</span><span class="k">}</span><span class="s2">/g"</span> <span class="se">\</span>
        src/<span class="s2">"</span><span class="nv">$part</span><span class="s2">"</span>/META.in <span class="o">&gt;</span> src/<span class="s2">"</span><span class="nv">$part</span><span class="s2">"</span>/META
  <span class="k">fi
done</span>
</code></pre></div></div>

<p>A piece of code got changed to fix neither a bug nor add a feature, and
something which worked before became broken. It took a non-trivial amount of
(wallclock) time to diagnose and fix the issue. That’s a lot of negative
consequences, and so far at least, there are <em>no</em> positive consequences.</p>

<p>If it ain’t broke, don’t fix it!</p>
