---
title: 'AoAH Day 11: HTTP Cookies and vibing RFCs for breakfast'
description: Synthesizing three RFC-compliant libraries (punycode, public-suffix,
  and cookeio) directly from Internet RFC specifications, establishing a workflow
  for automating standards implementation with proper cross-referencing to spec sections.
url: https://anil.recoil.org/notes/aoah-2025-11
date: 2025-12-10T00:00:00-00:00
preview_image: https://anil.recoil.org/images/http-cookie-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>I'm switching focus for a few days to build a complete HTTP(S) client to use in my <a href="https://anil.recoil.org/projects/ce">literature downloader</a>. This requires building a few support libraries before we build the full client, so I figured I'd dive in them in the next few days.  First up is <a href="https://en.wikipedia.org/wiki/HTTP_cookie">RFC6264 HTTP Cookie</a> support. There are some excellent existing cookie libraries already on opam, notably <a href="https://github.com/lemaetech/http-cookie">http-cookie</a> and <a href="https://github.com/ulrikstrid/ocaml-cookie">ocaml-cookie</a>, but I wasn't sure what their coverage of the protocol is, and there's no Eio serialisation support.</p>
<p>So I thought I'd have a go at a different approach today using agentic coding: can we synthesise a <em>complete</em> HTTP Cookie implementation purely from the <a href="https://www.rfc-editor.org/rfc/rfc6265">RFC 6265 prose</a> itself, and then differentially compare this OCaml implementation against the others? In theory, running a single test suite across all three libraries might be a good way of discovering how to improve the existing implementations. In the long-term, http-cookie is probably the upstream library I want to use, but I don't want to generate a giant diff against it today due to my <a href="https://anil.recoil.org/notes/aoah-2025">groundrules of not disturbing other maintainers</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#starting-with-rfc-6265-http-state-management" class="anchor" aria-hidden="true"></a>Starting with RFC 6265: HTTP State Management</h2>
<p>I downloaded <a href="https://www.rfc-editor.org/rfc/rfc6265">RFC 6265</a> and gave the agent all my previously generate code in the same directory to act as examples. My first run went verrrrry strangely as it generated a plan for a <a href="https://www.rfc-editor.org/rfc/rfc6264">carrier-grade NAT</a> implementation, until I realised that I'd typoed the RFC number and picked 6264 by mistake.  Ah, such human frailty...</p>
<p>Once I got the right RFC number, I stuck Claude into <a href="https://code.claude.com/docs/en/common-workflows#use-plan-mode-for-safe-code-analysis">planning mode</a> with the RFC text and also instructed it to search through opam to find relevant packages. This opam search went well, as it identified some missing dependent logic that a full implementation of cookies would require. So I went down a sub-rabbithole of implementing those dependent packages first.</p>
<p>First up is <a href="https://en.wikipedia.org/wiki/Punycode">Punycode</a>, as <a href="https://datatracker.ietf.org/doc/html/rfc6265#section-5.1.2">RFC6265 Section 5.1.2</a> requests that:</p>
<blockquote>
<p>Convert each label that is not a Non-Reserved LDH (NR-LDH) label,
to an A-label (see Section 2.3.2.1 of [RFC5890] for the former
and latter), or to a "punycode label" (a label resulting from the
"ToASCII" conversion in Section 4 of [RFC3490]), as appropriate
(see Section 6.3 of this specification).)</p>
</blockquote>
<h2><a href="https://anil.recoil.org/news.xml#diving-into-rfc-3490-punycode" class="anchor" aria-hidden="true"></a>Diving into RFC 3490: Punycode</h2>
<p>I'd never heard of this Punycode before, but perusing
<a href="https://datatracker.ietf.org/doc/html/rfc3490">RFC3490</a> introduces it such:</p>
<blockquote>
<p>Until now, there has been no standard method for domain names to use
characters outside the ASCII repertoire.  This document defines
internationalized domain names (IDNs) and a mechanism called
Internationalizing Domain Names in Applications (IDNA) for handling them in a
standard fashion.  IDNs use characters drawn from a large repertoire
(Unicode), but IDNA allows the non-ASCII characters to be represented using
only the ASCII characters already allowed in so- called host names today.
This backward-compatible representation is required in existing protocols
like DNS, so that IDNs can be introduced with no changes to the existing
infrastructure.  IDNA is only meant for processing domain names, not free
text.
<cite>-- <a href="https://datatracker.ietf.org/doc/html/rfc3490">RFC3490</a>, 2003</cite></p>
</blockquote>
<p>Ok, ok so this is some DNS thing needed to reliably compare cookie domains that
might be in different languages. The RFC is a little unusual in that it <em>embeds</em>
C code and also test vectors, so this seems ideal for an agentic session.</p>
<h3><a href="https://anil.recoil.org/news.xml#result-an-ocaml-punycode-library" class="anchor" aria-hidden="true"></a>Result: an OCaml-punycode library</h3>
<p>I pushed <a href="https://tangled.org/anil.recoil.org/ocaml-punycode/">ocaml-punycode</a> to <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">Tangled</a>. It's fairly straightforward, with one module for the core Unicode (using <a href="https://erratique.ch/software/uunf">uufn</a> for Unicode normalisation and the <a href="https://github.com/hannesm/domain-name">domain-name</a> by <a href="https://github.com/hannesm">Hannes Mehnert</a> for RFC1035 Internet Domain Name handling.
Although we didn't use cram tests for this, the <a href="https://tangled.org/anil.recoil.org/ocaml-punycode/blob/main/test/test_punycode.ml">Punycode alcotest</a> seem to encode the test vectors from the RFC faithfully, and also do roundtrip testing.</p>
<p>One trick that helped get more idiomatic OCaml code here was to do the generation in two passes. First I asked it to transcribe the RFC into OCaml along with the test cases, and then a second pass to refactor the code using higher-order functions and Stdlib combinators. You can see the results of the <a href="https://tangled.org/anil.recoil.org/ocaml-punycode/commit/cb3b948db5e4331c200ef196b41ea35be325cf60">second pass here</a>, which does look like more normal OCaml that I might write.</p>
<h3><a href="https://anil.recoil.org/news.xml#linking-code-to-rfc-sections" class="anchor" aria-hidden="true"></a>Linking code to RFC sections</h3>
<p>Coding with the RFC as the source spec actually resurrected an issue I've been thinking about since 2015. Back then, <a href="https://github.com/lcdunstan">Luke Dunstan</a> wanted to add <a href="https://github.com/mirage/ocaml-dns/issues/28#issuecomment-107215853">multicast DNS support to ocaml-dns</a>, and as part of that he proposed using extension attributes in the code to link the relevant section of the RFC spec to the associated OCaml function.</p>
<p>Coding agents go a long way to automating this. In the punycode ocamldoc, I instructed it to use the online RFC repository as an anchor base, and it successfully links almost every type to where it got the specification constraints from.</p>
<pre><code class="language-ocaml">type error =
| Overflow of position
   (** Arithmetic overflow during encode/decode. This can occur with
       very long strings or extreme Unicode code point values.
       See {{:https://datatracker.ietf.org/doc/html/rfc3492#section-6.4}
       RFC 3492 Section 6.4} for overflow handling requirements. *)
| Invalid_character of position * Uchar.t
   (** A non-basic code point appeared where only basic code points
       (ASCII &lt; 128) are allowed. Per
       {{:https://datatracker.ietf.org/doc/html/rfc3492#section-3.1}
       RFC 3492 Section 3.1}, basic code points must be segregated
       at the beginning of the encoded string. *)
</code></pre>
<p>I've therefore mechanised this insight into a
<a href="https://tangled.org/anil.recoil.org/claude-ocaml-internet-rfc">claude-ocaml-internet-rfc</a>
skill which formalises my future approach to RFCs and allows this part of the
workflow to be reused in future OCaml code.</p>
<p><img src="https://anil.recoil.org/images/http-cookie-1.webp" alt="%c" title="This post was fueled by delicious Swedish mulled wine and 'cookies'"></p>
<h2><a href="https://anil.recoil.org/news.xml#segwaying-into-public-suffix-lists" class="anchor" aria-hidden="true"></a>Segwaying into Public Suffix lists</h2>
<p>But then the exploration of the OCaml RFCs told me that for real compliance, we
needed to support <a href="https://publicsuffix.org/list/public_suffix_list.dat">Public Suffix lists</a>, something else
I'd never heard about!</p>
<blockquote>
<p>A "public suffix" is one under which Internet users can (or historically
could) directly register names. Some examples of public suffixes are com,
co.uk and pvt.k12.ma.us. The Public Suffix List is a list of all known public
suffixes.</p>
<p>The Public Suffix List is an initiative of Mozilla, but is
maintained as a community resource. It is available for use in any software,
but was originally created to meet the needs of browser manufacturers. It
allows browsers to, for example:</p>
<ul>
<li>Avoid privacy-damaging "supercookies" being set for high-level domain name suffixes</li>
<li>Highlight the most important part of a domain name in the user interface</li>
<li>Accurately sort history entries by site
<cite>-- <a href="https://publicsuffix.org/learn/">Public Suffix Lists</a>, 2025</cite></li>
</ul>
</blockquote>
<p>This one's different from the previous library in that there's a large <a href="https://publicsuffix.org/list/public_suffix_list.dat">dataset</a> that needs to be embedded inside the library, and we need reasonably efficient datastructures to traverse them and do the domain comparison (using the Punycode logic from earlier for domain name normalisation).</p>
<p>So my approach for this was to download the dataset and clone the <a href="https://github.com/publicsuffix/list/wiki/">public suffix list wiki</a> and prompt the agent to come up with a generation architecture that would first parse the list into an OCaml data structure and then provide a <code>Public_suffix</code> OCaml interface that we could use without having to download the data as a library user.</p>
<h3><a href="https://anil.recoil.org/news.xml#results-a-public-suffix-opam-package" class="anchor" aria-hidden="true"></a>Results: a public-suffix opam package</h3>
<p>The agent managed all the dune build rules fine for the generated package, so I pushed a <a href="https://tangled.org/anil.recoil.org/ocaml-publicsuffix/">ocaml-publicsuffix</a> Git repository using the <a href="https://anil.recoil.org/notes/aoah-2025-5">opam metadata skill</a>. I <em>also</em> then used the earlier <a href="https://tangled.org/anil.recoil.org/claude-ocaml-internet-rfc">ocaml-internet-rfc skill</a> to add references to the various RFCs we used.</p>
<p>The interface is very simple, since the dataset is embedded as OCaml code:</p>
<pre><code class="language-ocaml">let psl = Publicsuffix.create () in

Publicsuffix.public_suffix psl "www.example.com"
(* Returns: Ok "com" *)

Publicsuffix.public_suffix psl "www.example.co.uk"
(* Returns: Ok "co.uk" *)

Publicsuffix.registrable_domain psl "www.example.com"
(* Returns: Ok "example.com" *
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#and-so-finally-onto-cookeio" class="anchor" aria-hidden="true"></a>And so finally onto Cookeio!</h2>
<p>After these segues, we finally have enough for an Eio-based Cookie library that's derived from the main cookie RFC!</p>
<p>This was pretty plain sailing given the previous libraries. The published <a href="https://tangled.org/anil.recoil.org/ocaml-cookeio">ocaml-cookeio library</a> library has subpackages for the <a href="https://tangled.org/anil.recoil.org/ocaml-cookeio/blob/main/lib/jar/cookeio_jar.mli">Cookie jar</a> interface that provide a pretty simple persistent cookie implementation that we can use in the next few days for our HTTP client.</p>
<pre><code>val create : unit -&gt; t
(** Create an empty cookie jar. *)

val load : clock:_ Eio.Time.clock -&gt; Eio.Fs.dir_ty Eio.Path.t -&gt; t
(** Load cookies from Mozilla format file.

    Loads cookies from a file in Mozilla format, using the provided clock to set
    creation and last access times. Returns an empty jar if the file doesn't
    exist or cannot be loaded. *)

val save : Eio.Fs.dir_ty Eio.Path.t -&gt; t -&gt; unit
(** Save cookies to Mozilla format file. *)

(** {1 Cookie Jar Management} *)

val add_cookie : t -&gt; Cookeio.t -&gt; unit
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>I didn't expect to generate three libraries in one day, but I was massively sped up by creating this <a href="https://tangled.org/anil.recoil.org/claude-ocaml-internet-rfc">Claude skill for Internet RFC handling</a>.  The three libraries today, <a href="https://tangled.org/anil.recoil.org/ocaml-punycode">punycode</a>, <a href="https://tangled.org/anil.recoil.org/ocaml-publicsuffix/">publicsuffix</a> and <a href="https://tangled.org/anil.recoil.org/ocaml-cookeio/">cookeio</a> do a lot of fairly ad-hoc checking that's specified throughout a variety of RFCs, but do benefit from public test sets to check their behaviour.</p>
<p>Now that these are out of the way, I'm going to continue on with another essential library on <a href="https://anil.recoil.org/notes/aoah-2025-12">Day 12</a> tomorrow: Eio TCP/TLS connection pooling.</p>
<p><strong>Update 13th Dec 2025:</strong> <a href="https://github.com/dinosaure">Romain Calascibetta</a> and <a href="https://github.com/hannesm">Hannes Mehnert</a> both kindly alerted me to the fact that there are existing implementations of punycode and publicsuffix. The first one by <a href="https://github.com/cfcs/ocaml-punycode">cfcs/punycode</a> wasn't found by my agent since it's not in the opam repo, and <a href="https://opam.ocaml.org/packages/public-suffix/public-suffix.0.0.1/">public-suffix</a> was missed since my checkout was just a week out of date! But to re-emphasise my <a href="https://anil.recoil.org/notes/aoah-2025">groundrules</a>, this is <em>not</em> a competition of agents <em>vs</em> humans. I will very happily use these "proper" libraries in the long term, but in the short-term I cannot contribute a giant chunk of AI-generated code onto those upstream projects. Figuring out what to do about this is one of the points of this month-long adventure...</p>

