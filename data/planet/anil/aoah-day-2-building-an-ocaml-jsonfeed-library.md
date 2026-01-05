---
title: 'AoAH Day 2: Building an OCaml JSONFeed library'
description: Implementing a JSONFeed specification library using jsont codecs, discovering
  how Claude can automate the construction of complex combinators from prose specifications
  with excellent error messages.
url: https://anil.recoil.org/notes/aoah-2025-2
date: 2025-12-02T00:00:00-00:00
preview_image: https://anil.recoil.org/images/jsonfeed.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Day 2 of the <a href="https://anil.recoil.org/notes/aoah-2025">Advent of Agentic Humps</a> dawns with building a slightly more complex library than before, via the <a href="https://www.jsonfeed.org">JSONFeed</a> specification that is a more modern version of Atom.</p>
<p>JSONfeed is a successor to Atom for website feeds, that has a nice <a href="https://www.jsonfeed.org/version/1.1/">informal specification</a> about how to parse it. However, it also has a growing number of <a href="https://github.com/egonw/JSONFeed-extensions/tree/main">extensions</a> which also need to be implemented somehow, as well as some <a href="https://www.jsonfeed.org/mappingrssandatom/">informal rules to map RSS/Atom to JSONFeed</a>.</p>
<p>There is no existing OCaml implementation that I could find, and I need it to integrate my website with <a href="https://rogue-scholar.org">Rogue Scholar</a> more easily for <a href="https://anil.recoil.org/notes/principles-for-collective-knowledge">permanent DOIs</a>.</p>
<p><a href="https://jsonfeed.org"> <img src="https://anil.recoil.org/images/jsonfeed.webp" alt="%c"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>Unlike the <a href="https://anil.recoil.org/notes/aoah-2025-1">Crockford</a> implementation, parsing JSON involves selecting a third-party library dependency cone. By default the agent chose <a href="https://github.com/ocaml-community/yojson">Yojson</a> (presumably because its the most popular in its training set). I would conventionally use my own <a href="https://github.com/mirage/ezjsonm">ezjsonm</a> library that builds over the lower level <a href="https://github.com/dbuenzli/jsonm">jsonm</a>, but I noticed a deprecation notice on jsonm towards a newer library by <a href="https://erratique.ch">Daniel Bünzli</a> called <a href="https://github.com/dbuenzli/jsont">jsont</a>.</p>
<h3><a href="https://anil.recoil.org/news.xml#jsont" class="anchor" aria-hidden="true"></a>Jsont</h3>
<p>After seeing the <a href="https://discuss.ocaml.org/t/ann-jsont-0-1-0-declarative-json-data-manipulation-for-ocaml/15702">announcement of jsont</a> about a year ago, I gave it a quick try:</p>
<blockquote>
<p>Jsont is an OCaml library for declarative JSON data manipulation. It provides:</p>
<ul>
<li>Combinators for describing JSON data using the OCaml values of your choice. The descriptions can be used by generic functions to decode, encode, query and update JSON data without having to construct a generic JSON representation.</li>
<li>A JSON codec with optional text location tracking and layout preservation. The codec is compatible with effect-based concurrency.</li>
</ul>
<p>The descriptions are independent from the codec and can be used by third-party processors or codecs.
<cite>-- <a href="https://github.com/dbuenzli/jsont">jsont</a>, Daniel Bünzli, 2025</cite></p>
</blockquote>
<p>The codec for a JSON type is expressed using combinators, and then separately serialised or deserialised. I found that it had <em>fantastic</em> error messages since it could use the codec to come up with the reason why some input has been rejected. But on the flipside, writing the codecs involved a lot of boilerplate, so I found it quite time consuming.</p>
<p><a href="https://github.com/dbuenzli/jsont/blob/main/paper/soup.pdf"> <img src="https://anil.recoil.org/images/jsont-paper.webp" alt="%c" title="Daniel wrote a nice paper about the combinator magic behind jsont"> </a></p>
<p>But now, with Claude, I could use it to scan a spec and automate codec construction using jsont! So I decided to try a complex coding case where I fed the prose JSONFeed spec to Claude, and instructed it to build jsont codecs. As a separate phase, I then built test cases and serialisers.</p>
<h3><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h3>
<p>The very first run of Claude crashed and burned with jsont as it didn't have enough examples to figure out the interface from scratch, resulting in a lot of type errors and no working code.</p>
<p>So I took a different tack: I ran <code>opam source jsont</code> to get the source code into the current working directory, and then prompted a fresh Claude session to <em>"ultrathink about the interface of jsont.0.2.0, paying particular attention to the cookbook"</em>. The <a href="https://erratique.ch/software/jsont/doc/cookbook.html">cookbook</a> is a section of the (excellent) documentation in Jsont that describes real-world usage, and the agent also picked up on the <a href="https://github.com/dbuenzli/jsont/blob/main/test/json_rpc.ml">jsonrpc</a> testcase in the jsont repository.</p>
<p>Once it had this bit of example-driven context, the agent proceeded to build a very credible set of codecs:</p>
<ul>
<li><a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/author.ml">Author descriptions</a> look about right, with a pretty printer thrown in as a bonus.</li>
<li>The extremely tedious set of <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/cito.ml">CITO citation methods</a> came out in one shot.</li>
<li>It figured out to use <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/rfc3339.ml">Ptime based RFC3339</a> date handling for the feeds, from a combination of the spec and by querying opam locally.</li>
<li>The overall <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/jsonfeed.mli">Jsonfeed.mli</a> interface weaves all these together, exposes the jsont codec value <em>and</em> accessor and pretty printer functions.</li>
<li>As a bonus, i fed it the extensions repository so it can now expose <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/item.ml#L160">structured references</a> in my own <a href="https://anil.recoil.org/perma.json">site's JSON feed</a>.</li>
</ul>
<p>The user exposed API is idiomatic OCaml:</p>
<pre><code class="language-ocaml">type t
val jsont : t Jsont.t
val create :
  title:string -&gt; ?home_page_url:string -&gt;
  ?feed_url:string -&gt; ?description:string -&gt;
  ?user_comment:string -&gt; ?next_url:string -&gt;
  ?icon:string -&gt; ?favicon:string -&gt;
  ?authors:Author.t list -&gt; ?language:string -&gt;
  ?expired:bool -&gt; ?hubs:Hub.t list -&gt;
  items:Item.t list -&gt; ?unknown:Unknown.t -&gt;
  unit -&gt; t
</code></pre>
<p>The full set of constructors and validators can be read in <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/jsonfeed.mli">jsonfeed.mli</a>.</p>
<h3><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h3>
<p>For test cases, I first got Claude to synthesise a <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/tree/main/test/data">variety of JSONFeed corpuses</a> that exercised the spec, and manually inspected it to check it all seemed ok. I then made use of <a href="https://dune.readthedocs.io/en/latest/reference/cram.html">Dune Cram tests</a> to write CLI-based validators that run.</p>
<p>All of the boilerplate around writing the test cases just worked out of the box, with a little bit of manual prompting from me required to guide the agent towards some known edge cases (like extensions handling).</p>
<p>Here's an excerpt from the <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/test/test_locations.t">cram tests</a>:</p>
<pre><code>Missing Required Fields
------------------------

Test missing title field:
  $ ./test_location_errors.exe data/missing_title.json title
  {"status":"error","message":"Missing member title in JSON Feed object",
   "location":{"file":"data/missing_title.json","line":1,"column":1,
   "byte_start":0,"byte_end":65},"context":"$"}
  [1]

Test missing version field:
  $ ./test_location_errors.exe data/missing_version.json title
  {"status":"error","message":"Missing member version in JSON Feed object",
   "location":{"file":"data/missing_version.json","line":1,"column":1,
   "byte_start":0,"byte_end":51},"context":"$"}
  [1]
</code></pre>
<h3><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h3>
<p>The code is available on <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed">anil.recoil.org/ocaml-jsonfeed</a> with <a href="https://ocaml.org/p/jsonfeed/latest">online docs</a>.  My first user for this might be <a href="https://mynameismwd.org">Michael Dales</a> in his <a href="https://digitalflapjack.com/blog/the-partially-dynamic-web/">own website</a>.</p>
<p>The use of Claude has made using jsont the default choice for me now. As a first phase, I can ask the LLM to synthesise down a spec into a combinator-based codec, and then validate that separately. Crucially, the good error messages from jsont help the agent to root-cause why some tests fail, and give me the option of either clarifying the spec or fixing the test case if that was the error.</p>
<p>I did, however, still have to make the judgement call of which libraries to use at the start. The agent also happily spat out Yojson and Ezjsonm based implementations for me, but I simply prefer the jsont approach. If you had other priorities like pure performance, you might go for Yojson instead.</p>
<p>Onto <a href="https://anil.recoil.org/notes/aoah-2025-3">Day 3</a>, where we'll then build our first Eio based library!</p>

