---
title: 'AoAH Day 18: TOML 1.1 codecs directly from the spec and paper'
description: Building tomlt, a pure OCaml TOML 1.1 parser with bidirectional codecs
  following the jsont design patterns
url: https://anil.recoil.org/notes/aoah-2025-18
date: 2025-12-18T00:00:00-00:00
preview_image: https://anil.recoil.org/images/jsont-paper.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After getting my <a href="https://anil.recoil.org/notes/aoah-2025-17">email</a> interfaces automated yesterday, I turned my attention to <a href="https://eeg.zulipchat.com">Zulip</a> integration. But first, I took a segway into another format that it required known as <a href="https://toml.io">TOML</a>. I noticed <a href="https://lobste.rs/s/h50lml/toml_1_1_0_released">TOML 1.1.0 was released</a> today and so I built <strong><a href="https://tangled.org/anil.recoil.org/ocaml-tomlt">ocaml-tomlt</a></strong> today.</p>
<p>What I wanted to explore with this library is whether I could use a coding agent to build a complex functional abstraction from scratch. After building <a href="https://anil.recoil.org/notes/aoah-2025-6">yamlrw</a> and <a href="https://anil.recoil.org/notes/aoah-2025-7">yamlt</a>, I settled on the technique <a href="https://erratique.ch">Daniel Bünzli</a> developed with <a href="https://github.com/dbuenzli/jsont">jsont</a> in his <a href="https://github.com/dbuenzli/jsont/blob/main/paper/soup.pdf">paper</a>.</p>
<p><a href="https://github.com/dbuenzli/jsont/blob/main/paper/soup.pdf"> <img src="https://anil.recoil.org/images/jsont-paper.webp" alt="%c" title="Daniel wrote a nice paper about the combinator magic behind jsont"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#why-toml-instead-of-yaml-or-json" class="anchor" aria-hidden="true"></a>Why TOML instead of Yaml or JSON?</h2>
<p>TOML has become a popular configuration format for other language ecosystems like Rust and Python. Unlike <a href="https://anil.recoil.org/notes/aoah-2025-6">Yaml 1.2</a>, TOML is actually a <a href="https://toml.io/en/v1.1.0">reasonable human-editable format</a> without the terrifying corner cases and denial of service traps hidden in Yaml.</p>
<p>Since Toml 1.1 was just released today, there are existing OCaml libraries that fully supported. In addition, I need one that is pure OCaml with no C dependencies (like <a href="https://anil.recoil.org/notes/aoah-2025-6">yamlrw</a>) and that uses <a href="https://github.com/dbuenzli/bytesrw">Bytesrw</a> for streaming I/O so that it composes well with my other libraries from this month's coding.</p>
<h3><a href="https://anil.recoil.org/news.xml#the-data-soup-paper" class="anchor" aria-hidden="true"></a>The data soup paper</h3>
<p>The implementation of tomlt was prompted from <a href="https://raw.githubusercontent.com/dbuenzli/jsont/refs/heads/main/paper/soup.tex">"An Alphabet for Your Data Soups"</a> which accompanies his jsont library. Working with untyped data formats like TOML in strongly-typed languages like OCaml requires a lot of tedious dynamic marhsalling, and I'd like to switch to conventional OCaml <a href="https://dev.realworldocaml.org/records.html">records</a> or other static types as soon as possible.</p>
<p>Daniel's solution is to define a <a href="https://dev.realworldocaml.org/gadts.html">generalised algebraic datatype</a> whose values represent
bidirectional mappings between subsets of the wire format and my chosen OCaml
types. Waaay back in 2010 when <a href="https://github.com/samoht">Thomas Gazagnaire</a> and I worked on <a href="https://anil.recoil.org/papers/2010-dyntype-wgt">camlp4-based serialisation</a>, we converted into a generic intermediate
representation for OCaml types and values. More recently <a href="https://www.cst.cam.ac.uk/people/jdy22">Jeremy Yallop</a> has been
working on <a href="https://dl.acm.org/doi/10.1145/3607851">MacoCaml</a> which performs
this transformation at compile time via hygenic macros.</p>
<p>Unlike any of these approaches, the functional pearl Daniel came up with allows the programmer to
define direct functional transformations that work in both directions. It's a
bit more work at runtime and so a bit slower, but in return you get excellent
error messages for malformed messages. The core Toml type therefore becomes:</p>
<pre><code class="language-ocaml">(* A codec encapsulates both decoding and encoding *)
type 'a t = {
  kind : string;
  doc : string;
  dec : Toml.t -&gt; ('a, codec_error) result;
  enc : 'a -&gt; Toml.t;
}
</code></pre>
<p>This means you write your schema once and get both directions for free, and
user functions can be placed at every coding step to allow the programmer to
<em>interpose</em> custom functionality such as transformation or validation.</p>
<h2><a href="https://anil.recoil.org/news.xml#using-tomlt-in-practise" class="anchor" aria-hidden="true"></a>Using tomlt in practise</h2>
<p>A Toml config file might look something like this:</p>
<pre><code>[server]                                                                                                                                                    
  host = "localhost"                                                                                                                                          
  port = 8080                                                                                                                                                 
                                                                                                                                                              
[database]                                                                                                                                                  
  connection_max = 5000
</code></pre>
<p>Here's what using tomlt to parse this looks like in practice:</p>
<pre><code class="language-ocaml">type config = { host : string; port : int; debug : bool }

let config_codec =
  Tomlt.(Table.(
    obj (fun host port debug -&gt; { host; port; debug })
    |&gt; mem "host" string ~enc:(fun c -&gt; c.host)
    |&gt; mem "port" int ~enc:(fun c -&gt; c.port)
    |&gt; mem "debug" bool ~enc:(fun c -&gt; c.debug) ~dec_absent:false
    |&gt; finish
  ))

let () =
  match Tomlt.decode_string config_codec {|
    host = "localhost"
    port = 8080
  |} with
  | Ok config -&gt; Printf.printf "Host: %s\n" config.host
  | Error e -&gt; prerr_endline (Tomlt.Toml.Error.to_string e)
</code></pre>
<p>The functional pattern is almost identical to the <a href="https://anil.recoil.org/notes/aoah-2025-7">yamlt</a> or <a href="https://anil.recoil.org/notes/aoah-2025-2">jsont</a> codecs I've been building.
You don't <em>have</em> to define a codec, as tomlt also provides <a href="https://ocaml.org/manual/5.4/indexops.html">custom index operators</a> to navigate tables directly:</p>
<pre><code class="language-ocaml">let config = Toml.of_string {|
  [server]
  host = "localhost"
  port = 8080

  [database]
  connection_max = 5000
|} in
(* Navigate nested tables with .%{} *)
let host = Toml.(config.%{["server"; "host"]} |&gt; to_string) in
let port = Toml.(config.%{["server"; "port"]} |&gt; to_int) in
Printf.printf "Server: %s:%Ld\n" host port;

(* Update values *)
let config' = Toml.(config.%{["database"; "enabled"]} &lt;- bool true) in
print_endline (Toml.to_string config')
</code></pre>
<p>The syntax is a little verbose due to the module opening, but it's still a
pretty nice way to poke around TOML files interactively!</p>
<h3><a href="https://anil.recoil.org/news.xml#datetime-handling" class="anchor" aria-hidden="true"></a>Datetime handling</h3>
<p>Another area where TOML differs from other formats is that it four distinct
datetime formats: offset datetimes, local datetimes, local dates, and local
times. tomlt tries to unify this a little via a single codec that normalises everything to <a href="https://github.com/dbuenzli/ptime">Ptime.t</a>, but allows the codec to supply sensible defaults (e.g. for a missing timezone, or a missing date).</p>
<pre><code class="language-ocaml">(* All of these decode to Ptime.t with sensible defaults *)
(* when = 2024-01-15T10:30:00Z       -&gt; offset datetime *)
(* when = 2024-01-15T10:30:00        -&gt; local datetime *)
(* when = 2024-01-15                 -&gt; date at midnight *)
(* when = 10:30:00                   -&gt; time on today's date *)

let event_codec = Tomlt.(Table.(
  obj (fun name when_ -&gt; { name; when_ })
  |&gt; mem "name" string ~enc:(fun e -&gt; e.name)
  |&gt; mem "when" (ptime ()) ~enc:(fun e -&gt; e.when_)
  |&gt; finish
))
</code></pre>
<p>For applications that need to preserve the exact format, there's also a
<code>ptime_full</code> function which returns a polymorphic variant indicating precisely
what was present in the source config file.</p>
<h2><a href="https://anil.recoil.org/news.xml#testing" class="anchor" aria-hidden="true"></a>Testing</h2>
<p>The secret to vibing seems to be having a specification oracle to guide the
agent, and TOML has a <a href="https://github.com/toml-lang/toml-test">toml-test</a> suite
that's perfect for this purpose:</p>
<blockquote>
<p>toml-test is a language-agnostic test suite to verify the correctness of TOML parsers and writers.</p>
<p>Tests are divided into two groups: "invalid" and "valid". Decoders or
encoders that reject "invalid" tests pass the tests, and decoders that accept
"valid" tests and output precisely what is expected pass the tests. The
output format is JSON, described below.
<cite>-- <a href="https://github.com/toml-lang/toml-test">Toml-test GitHub</a>, 2021</cite></p>
</blockquote>
<p><img src="https://anil.recoil.org/images/aoah-toml-ss-1.webp" alt="%c" title="The Claude coding agent iterated overnight on getting to 100% test on the third party tests"></p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>After building <a href="https://anil.recoil.org/notes/aoah-2025-6">yamlrw</a>, <a href="https://anil.recoil.org/notes/aoah-2025-7">yamlt</a>, and now tomlt,
I'm convinced that the bidirectional codec pattern is a good approach for
<em>agentic</em> OCaml programming. It's a little verbose to express by hand, which
leads down the ppx route for most. But with agentic generation and oracle
specification testing, the coding agent was particularly helpful with both
figuring out the TOML grammar and exposing all the variations of codecs
required for parsing all those datetime variants.</p>
<p>Having the <a href="https://toml.io/en/v1.1.0">TOML 1.1 specification</a> as context and
my earlier <a href="https://anil.recoil.org/notes/aoah-2025-11">Claude OCaml RFC skill</a> helped a lot as well, to
allow the ocamldoc to be cross referenced.  And of course, the key design
insights at the heart of the library came from <a href="https://erratique.ch">Daniel Bünzli</a> publishing jsont and
also uploading his paper. This Tomlt library is a generative clone of his
ideas, but a useful one to my personal workflows this advent!</p>
<p>Tomorrow in <a href="https://anil.recoil.org/notes/aoah-2025-19">Day 19</a>, I'll continue with my original goal of getting
a Zulip bot working!</p>

