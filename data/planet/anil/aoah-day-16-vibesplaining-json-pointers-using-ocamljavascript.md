---
title: 'AoAH Day 16: Vibesplaining JSON Pointers using OCaml/Javascript'
description: Building interactive OCaml tutorials that compile to JavaScript, using
  agents to generate executable documentation that teaches protocols like JSON Pointer
  while you code review.
url: https://anil.recoil.org/notes/aoah-2025-16
date: 2025-12-16T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-json-ss-2.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After the successful <a href="https://anil.recoil.org/notes/aoah-2025-15">HTML5 translation</a> yesterday, I realised
that I know next to nothing about HTML5 parsing and had leant extremely heavily
on agentic coding. This approach has also been useful to help me explore
<a href="https://anil.recoil.org/notes/aoah-2025-13">diverse codebases</a> in a combination of languages. So today I
set my sights on understanding the pedagogical impacts of agentic coding a bit
more. Can we use coding agents to help us iteratively explore complex
protocols?</p>
<p>I decided to build a <a href="https://jmap.io">JMAP email</a> client implementation in
OCaml that I need for myself<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup>  but with the added twist of seeing how I
could engineer agents to "vibesplain" a protocol to me that I'm unfamiliar
with.</p>
<p>OCaml has superb tooling to help with this; it can not only compile to efficient native code but also to <a href="https://github.com/ocsigen/js_of_ocaml">JavaScript and WASM</a> that runs standalone in the browser. I turned to my colleagues <a href="https://jon.recoil.org">Jon Ludlam</a>, <a href="https://patrick.sirref.org">Patrick Ferris</a> and <a href="https://github.com/art-w">Arthur Wendling</a> for help with the tooling, since they've been leading the way on <a href="https://patrick.sirref.org/var/propl2025.pdf">scientific programming</a>, <a href="https://jon.recoil.org/blog/2025/12/an-svg-is-all-you-need.html">visualisations</a> and <a href="https://art-w.github.io/x-ocaml/">webcomponents</a> in OCaml.</p>
<p>Today's work resulted in an <strong><a href="https://tangled.org/anil.recoil.org/ocaml-json-pointer">ocaml-json-pointer</a></strong> (<a href="https://datatracker.ietf.org/doc/html/rfc6901">RFC6901</a>) implementation along with an <a href="https://www.cl.cam.ac.uk/~avsm2/json-pointer/">interactive notebook tutorial</a> that bundles the entire OCaml compiler toolchain alongside it. There's even another one for <a href="https://www.cl.cam.ac.uk/~avsm2/yamlrw-doc">Yaml</a> just to illustrate how easy this is to replicate once we've built the first one.</p>
<h2><a href="https://anil.recoil.org/news.xml#why-do-we-need-to-vibesplain-specifications" class="anchor" aria-hidden="true"></a>Why do we need to vibesplain specifications?</h2>
<p>I first began by informally scanning the <a href="https://www.rfc-editor.org/rfc/rfc8620">JMAP RFCs</a> with a cup of tea, and noticing that it needed support for something called <a href="https://datatracker.ietf.org/doc/html/rfc6901">JSON Pointer</a>. I know nothing about this, and so I used my <a href="https://tangled.org/anil.recoil.org/claude-ocaml-internet-rfc">Claude Internet RFC skill</a> to sketch out an <a href="https://tangled.org/anil.recoil.org/ocaml-json-pointer">OCaml implementation</a> based on the specification.</p>
<p>This is straightforward, but I didn't know how to evaluate the generated interface for fitness since I'm not familiar with the topic. Ideally, I could use the coding agent to explain JMAP pointers to me <em>by using the generated OCaml code illustratively</em> so I could do a code review of its output at the same time as educating myself. For this to work, we would have to machine check the generated documentation and code to make sure it all compiles.
This seems like a really good use to put coding agents to!</p>
<p>I dub <strong>vibesplaining</strong> as using an agent to generate executable code <em>and an executable tutorial</em> that can be interacted with by the prompter in order to assist with both understanding the code and iterating on the implementation.</p>
<p>When writing <a href="https://dev.realworldocaml.org">Real World OCaml</a> about a decade ago, we built it using a tool called <a href="https://github.com/realworldocaml/mdx">mdx</a> that allowed for embedding OCaml sections within a Markdown document. Mdx can compile these markdown documents by executing the embedded OCaml, and promote the outputs directly into the document itself. You can see an example in the <a href="https://github.com/realworldocaml/book/tree/master/book/guided-tour">Guided Tour chapter</a> where every single OCaml phrase has been machine generated. <a href="https://github.com/yminsky">Yaron Minsky</a> has talked extensively about how successful this "expect test" approach is within Jane Street: they use it for <a href="https://blog.janestreet.com/using-ascii-waveforms-to-test-hardware-designs/">hardware waveforms</a>, <a href="https://blog.janestreet.com/repeatable-exploratory-programming/">exploratory programming</a> and <a href="https://blog.janestreet.com/the-joy-of-expect-tests/">interactive specification</a>.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/json-pointer/"> <img src="https://anil.recoil.org/images/aoah-json-ss-2.webp" alt="%c" title="What we'll explore in the rest of this post is how having an interactive JS notebook tutorial of OCaml code is now straightforward!"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#executable-ocaml-documentation-within-ocaml-comments" class="anchor" aria-hidden="true"></a>Executable OCaml documentation within OCaml comments</h2>
<p>Upon consulting <a href="https://jon.recoil.org">Jon Ludlam</a>, he told me that <a href="https://jon.recoil.org/blog/2025/04/this-site.html">odoc v3 supports executable code fragments</a> already, so that we don't even need a separate Markdown file anymore. The <em>comments</em> in OCaml code can themselves contain OCaml code that can be compiled into JavaScript and run in a browser!</p>
<p>I had never even looked at the tooling for this aspect of odoc, but <a href="https://patrick.sirref.org">Patrick Ferris</a>
has been building a <a href="https://github.com/patricoferris/ocaml-bibtex">BibTeX OCaml</a> that is extensively using
this workflow. His <a href="https://github.com/patricoferris/ocaml-bibtex/blob/main/src/dune">dune file</a> just
adds a new stanza:</p>
<pre><code>(mdx
  (files index.mld bib.mli)
  (libraries bib bytesrw astring))
</code></pre>
<p>This stanza handles the build logic for odoc and mdx to interface and extract
executable code out of <a href="https://github.com/patricoferris/ocaml-bibtex/blob/main/src/bib.mli">interfaces
files</a>
like:</p>
<pre><code class="language-ocaml">(** ... 
  Use {! decode} to read your Bibtex data and {! encode} to write
  it. For example, here is a little program to format a Bibtex file
  from a string.

  {@ocaml[
    open Bytesrw

    let format s =
      let reader = Bytes.Reader.of_string s in
      let writer = Bytes.Writer.of_out_channel Out_channel.stdout in
      Bib.encode (Bib.decode reader) writer
  ]}

  You can then use it with something like:

  {@ocaml[
    # format "@string{  hello=world }";;
    @string{
      hello=world
    }
    - : unit = ()
  ]}
</code></pre>
<p>OCaml inception right here! The <code>#</code> at the start of the fragment indicates that it's an <a href="https://ocaml.org/docs/toplevel-introduction">OCaml toplevel phrase</a>, otherwise it's library code.</p>
<h3><a href="https://anil.recoil.org/news.xml#vibesplaining-a-json-pointer-tutorial" class="anchor" aria-hidden="true"></a>Vibesplaining a JSON pointer tutorial</h3>
<p>I used this expert knowledge to then vibesplain up a
<a href="https://tangled.org/anil.recoil.org/ocaml-json-pointer/blob/main/doc/tutorial.mld">tutorial.mld</a>,
which is a standalone ocamldoc fragment that explains JSON Pointers to me.
Here's an illustrative fragment:</p>
<pre><code class="language-sh">From RFC 6901, Section 1:

{i JSON Pointer defines a string syntax for identifying a specific value
within a JavaScript Object Notation (JSON) document.}

In other words, JSON Pointer is an addressing scheme for locating values
inside a JSON structure. Think of it like a filesystem path, but for JSON
documents instead of files.

For example, given this JSON document:

{x@ocaml[
# let users_json = parse_json {|{
    "users": [
      {"name": "Alice", "age": 30},
      {"name": "Bob", "age": 25}
    ]
  }|};;
val users_json : Jsont.json =
  {"users":[{"name":"Alice","age":30},{"name":"Bob","age":25}]}
]x}

The JSON Pointer [/users/0/name] refers to the string ["Alice"]:

{@ocaml[
# let ptr = of_string_nav "/users/0/name";;
val ptr : nav t = [Mem "users"; Nth 0; Mem "name"]
# get ptr users_json;;
- : Jsont.json = "Alice"
]}
</code></pre>
<p>This tutorial is not only available in the code repo, but it's also present in the
generated HTML library documentation for the JSON Pointer library as well.  It will,
for example, appear on the central <a href="https://ocaml.org">ocaml.org</a> if this package
is ever submitted to the official package repository.</p>
<h3><a href="https://anil.recoil.org/news.xml#interactive-vibesplaining-with-javascript" class="anchor" aria-hidden="true"></a>Interactive vibesplaining with JavaScript</h3>
<p>However, the tutorial would be even useful to me if it was fully interactive so
I can mess around with it while learning the codebase.
<a href="https://github.com/art-w">Arthur Wendling</a> recently released a <a href="https://www.webcomponents.org">webcomponent</a> for OCaml called
<a href="https://github.com/art-w/x-ocaml">x-ocaml</a> that allows a full OCaml compiler to be embedded
into a webpage. This is how I built the recent
<a href="https://anil.recoil.org/notes/icfp25-oxcaml">ICFP 2025 OxCaml tutorial</a>, but I figured it would also be an easy
way of redistributing a fully standalone version of the JSON Pointer library.</p>
<p>Being a webcomponent, <a href="https://art-w.github.io/x-ocaml/">using x-ocaml</a> is as simple as including a script header and adding
an <code>&lt;x-ocaml&gt;</code> tag with the OCaml code.  The only extra thing I needed was to create a Dockerfile to install my local library
into a separate opam switch and then run the <code>x-ocaml</code> shell script to generate the libraries blob.</p>
<p>We still need to compile our odoc source tutorial into HTML with the right <code>x-ocaml</code> tags. <a href="https://jon.recoil.org">Jon Ludlam</a> explained to me odoc mld can be compiled straight to HTML using a sequence of commands:</p>
<pre><code class="language-sh">$ odoc compile &lt;file.mld&gt; 
$ odoc html-generate page-file.odoc -o . 
$ odoc support-files -o 
</code></pre>
<p>I then used my <a href="https://anil.recoil.org/notes/aoah-2025-15">html5rw library</a> from yesterday to write a utility to transform the odoc HTML output by rewriting to introduce the <code>&lt;x-ocaml&gt;</code> tags. I've published this as a <a href="https://tangled.org/anil.recoil.org/odoc-xo">odoc-xo</a> binary.  This then sit inside some dune rules in my JSON Pointer repository to automate the conversion from mld to interactive Javascript. You can see the <a href="https://tangled.org/anil.recoil.org/ocaml-json-pointer/blob/main/doc/dune">promotion rules here</a>; I'll wrap them into a Claude skill later this week.</p>
<p>You can browse the <a href="https://www.cl.cam.ac.uk/~avsm2/json-pointer/">interactive tutorial
here</a> for JSON Pointer, which I
found very useful to exploring the specification interactively. I did have a
quick go at compiling my earlier <a href="https://anil.recoil.org/notes/aoah-2025-4">Claude OCaml bindings</a> in order
to be able to invoke Claude during my explorations, but I got interrupted by a
very pleasant mince pie celebration so I'll leave that as an exercise to the
reader.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>There are still a few tooling rough edges to smooth out here, but nothing that can't be automated with a Claude skill and some more binaries like <a href="https://tangled.org/anil.recoil.org/odoc-xo">odoc-xo</a>. In <a href="https://anil.recoil.org/notes/aoah-2025-17">Day 17</a> I'll finish up the JMAP implementation as well.</p>
<p>More generally, what I find fascinating about agentic skills is that once you apply a template somewhere, it can be reused very quickly. It took me a few hours and a <em>lot</em> of input from <a href="https://jon.recoil.org">Jon Ludlam</a> and <a href="https://patrick.sirref.org">Patrick Ferris</a> to come up with the first version today, but then it took less than 10 minutes to generate an equivalent one for other repositories. For example, here's the <a href="https://anil.recoil.org/notes/aoah-2025-6">Yaml</a> library <a href="https://www.cl.cam.ac.uk/~avsm2/yamlrw-doc">explainer tutorial</a> if you really want to know more.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/yamlrw-doc"> <img src="https://anil.recoil.org/images/aoah-yamlrw-doc-1.webp" alt="%c" title="All you ever wanted to interactively be vibesplained to about... Yaml"> </a></p>
<p>These tutorials aren't high quality writing, especially compared to one where I've really thought it through and spent time on for specific teaching outcomes. However, agent generated "vibesplained" tutorials can also be very personalised even if not redistributed. For example, I want to know about recent advances in TCP, and so I am feeding the agent my <a href="https://github.com/mirage/mirage-tcpip">mirage-tcpip</a> stack and the latest RFCs, and requesting a tutorial about new concepts that I might have missed in recent years. Having the capability to provide contexts and generate <em>executable</em> tutorial notebooks is a new capability I've discovered today.</p>
<p>This does, of course, depend on having a high quality baseline of documentation in my dependent libraries, and of exemplar human-written rules and documentation. Thank you to <a href="https://jon.recoil.org">Jon Ludlam</a>, <a href="https://patrick.sirref.org">Patrick Ferris</a>, <a href="https://github.com/art-w">Arthur Wendling</a> and <a href="https://erratique.ch">Daniel Bünzli</a> for going to the (long) effort of giving me the base material to work from here. I am very appreciative of your contributions to OCaml open source.</p>

<div class="footnotes"><ol><li><p></p><p>Being a professor is not accurately measured by research or teaching outputs, but by how overloaded your INBOX is.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div>
