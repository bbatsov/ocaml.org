---
title: 'AoAH Day 7: Converting between JSON and Yaml with yamlt'
description: Building yamlt to enable jsont codec definitions to work with both JSON
  and Yaml, providing data manipulation with location tracking and good error messages
  for both formats.
url: https://anil.recoil.org/notes/aoah-2025-7
date: 2025-12-07T00:00:00-00:00
preview_image: https://anil.recoil.org/images/yamlt-tangled.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After the excitement of building an entire <a href="https://anil.recoil.org/notes/aoah-2025-6">Yaml 1.2 parser yesterday</a>, I began to put it to use. Since I've been steadily converting all my JSON parsers <a href="https://anil.recoil.org/notes/aoah-2025-2">to use jsont</a> codecs, it would be convenient if a single JSONt codec definition could <em>also</em> convert that schema to Yaml. In theory, Yaml is a superset of JSON, except <a href="https://metacpan.org/pod/JSON::XS#JSON-and-YAML">it isn't actually</a>. But it's close enough that we <em>should</em> be able to build a <a href="https://tangled.org/anil.recoil.org/ocaml-yamlt">yamlt library</a> that can accept a <a href="https://github.com/dbuenzli/jsont">jsont codec</a> and spit out Yaml (or the reverse).</p>
<p><a href="https://tangled.org/anil.recoil.org/ocaml-yamlt"> <img src="https://anil.recoil.org/images/yamlt-tangled.webp" alt="%c"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>I used all the previous tricks learnt so far, with all the dependent libraries available in the source tree for the agent to browse.  The source code to jsont was absolutely essential, since the agent needed to learn how to traverse the jsont codecs to build a different (yaml) translator.</p>
<p>One thing that was essential was guidance on how to do certain conversions where the results are ambiguous. For example, null handling in Yaml is much more permissive than in JSON, so I opted for any dictionary or array types to resolve nulls in the Yaml to the empty dictionary. It would also be ok to just raise an error in that case, but it seems safer to try to recover from what a human might do.</p>
<h2><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h2>
<p>As <a href="https://anil.recoil.org/notes/aoah-2025-2">before</a>, cram tests are a very effective way of exercising codepaths. I got the agent to generate a <a href="https://tangled.org/anil.recoil.org/ocaml-yamlt/tree/main/tests/cram">comprehensive suite</a> of cram tests that do things like:</p>
<pre><code>Encode arrays to JSON and YAML formats

  $ test_arrays encode
  JSON: {"numbers":[1,2,3,4,5],"strings":["hello","world"]}
  YAML Block:
  numbers:
    - 1
    - 2
    - 3
    - 4
    - 5
  strings:
    - hello
    - world
  YAML Flow: {numbers: [1, 2, 3, 4, 5], strings: [hello, world]}
</code></pre>
<p>And also negative results:</p>
<pre><code>Attempting to decode an object file with an array codec should fail

  $ test_arrays int ../data/objects/simple.yml
  JSON: int_array: ERROR: Missing member values in Numbers object
  File "-", line 1, characters 0-28:
  YAML: int_array: ERROR: Missing member values in Numbers object
  File "-":
</code></pre>
<p>Which is a convenient way of checking that location tracking is working.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>This all worked fairly straightforwardly. The Yamlt interface even includes support for multidocument Yaml files:</p>
<pre><code class="language-ocaml">val decode_all :
  ?layout:bool -&gt;
  ?locs:bool -&gt;
  ?file:Jsont.Textloc.fpath -&gt;
  ?max_depth:int -&gt;
  ?max_nodes:int -&gt;
  'a Jsont.t -&gt;
  Bytes.Reader.t -&gt;
  ('a, string) result Seq.t
(** [decode_all t r] decodes all documents from a multi-document YAML stream.
    Returns a sequence where each element is a result of decoding one document.
    Parameters are as in {!val-decode}. Use this for YAML streams containing
    multiple documents separated by [---]. *)
</code></pre>
<p>There's a lot going on this function, but it's easy to use. The optional parameters control details like Yaml layout and location tracking and defence against billion laughs attacks, but ultimately accept the jsont codec and a Yaml bytesrw source and give the result back as a lazy Seq.t.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>Using this library is very easy. The example code illustrates this:</p>
<pre><code class="language-ocaml">module Config = struct
  type t = { name : string; port : int }

  let make name port = { name; port }

  let jsont =
    Jsont.Object.map ~kind:"Config" make
    |&gt; Jsont.Object.mem "name" Jsont.string ~enc:(fun c -&gt; c.name)
    |&gt; Jsont.Object.mem "port" Jsont.int ~enc:(fun c -&gt; c.port)
    |&gt; Jsont.Object.finish
end

(* Use the same codec for both JSON and YAML *)
let from_json = Jsont_bytesrw.decode_string Config.jsont json_str
let from_yaml = Yamlt.decode_string Config.jsont yaml_str
</code></pre>
<p>I've started using yamlt in my website code without issue. It's very convenient
being able to describe a format as a jsont codec (which take care of conversion
to OCaml types and a concrete wire format), but then also exposing this as Yaml
for easier human editing. The line number tracking means that it's much easier
for me to trace back errors, thanks to jsont being so careful about this in its
implementation and interface.</p>
<p>I also notice that <a href="https://patrick.sirref.org">Patrick Ferris</a> has been <a href="https://patrick.sirref.org/graft-and-bib-update/index.xml">hacking</a> on a <a href="https://github.com/patricoferris/ocaml-bibtex">BibTeX codec</a> which is based on the same principles as jsont and also uses Bytesrw. I hope to also integrate this into my website code soon too!  But first, in <a href="https://anil.recoil.org/notes/aoah-2025-8">Day 8</a> we will build a complete CLI application that uses some of these libraries we've built so far.</p>

