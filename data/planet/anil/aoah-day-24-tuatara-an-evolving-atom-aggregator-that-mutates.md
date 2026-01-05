---
title: 'AoAH Day 24: Tuatara, an evolving Atom aggregator that mutates'
description: Tuatara is a feed aggregator that integrates Claude to evolve and patch
  its own code when encountering parsing errors, embodying the concept of self-healing
  software.
url: https://anil.recoil.org/notes/aoah-2025-24
date: 2025-12-24T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-tuatara-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>My original purpose for starting this <a href="https://anil.recoil.org/notes/aoah-2025">AoAH</a> series was to build a
feed aggregator for my group website, so I had to finish up with something to show!</p>
<p>I'm not sure if taking the <a href="https://www.goodreads.com/quotes/6001-think-you-re-escaping-and-run-into-yourself-longest-way-round">longest way around</a>
was wise here but I ended up building <strong><a href="https://tangled.org/anil.recoil.org/tuatara">tuatara</a></strong>, an
aggregator to pull together all my colleagues' writing into one place.
They're a quirky bunch with many diverse homegrown feeds in various
states of brokenness, so it's difficult to build a one-size-fits-all tool.</p>
<p>So given it's the end of the year and I'm sozzled on Christmas eve on
mulled wine, I decided to make Tuatara <strong>mutate its own code</strong> by linking with my <a href="https://anil.recoil.org/notes/aoah-2025-4">Claudeio</a> library to
force it to evolve and modify itself as it runs across feed errors. Every deployment of
Tuatara is meant to be slightly <a href="https://anil.recoil.org/papers/2025-internet-ecology">different</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#evolving-code-like-its-2026" class="anchor" aria-hidden="true"></a>Evolving code like it's 2026</h2>
<p>The initial generation of the code was pretty straightforward, using Sqlite to
store a database with all the posts and importing metadata from my previously
created <a href="https://anil.recoil.org/notes/aoah-2025-8">Sortal</a> contacts manager.</p>
<pre><code class="language-bash">&gt; tuatara import-sortal
Sortal Import Results:

  Total contacts scanned: 420
  Contacts with feeds: 15
  Feeds imported: 16
  Feeds skipped (already exist): 0

Run 'tuatara fetch' to download posts from the imported feeds.
</code></pre>
<p>But when we actually get the feeds, I rapidly realised that there are lots of
parsing quirks needed:</p>
<pre><code class="language-bash">&gt; tuatara fetch
Fetching Anil Madhavapeddy...
  340 posts (0 new)
Fetching David Allsopp...
  Not modified
Fetching Jessica Man...
  Not modified
Fetching Jon Ludlam...
  28 posts (0 new)
Fetching Jon Sterling...
  Not modified
Fetching Mark Elvers...
  Not modified
Fetching Martin Kleppmann...
  Error: Feed parse error: document MUST contains exactly one &lt;feed&gt; element at l.0 c.0
  URL: http://feeds.feedburner.com/martinkl
Fetching Onkar Gulati...
  Error: Not_found
  URL: https://onkargulati.com/feed.xml
Fetching Patrick Ferris...
  Error: Feed parse error: &lt;entry&gt; elements MUST contains at least an &lt;author&gt; element or &lt;feed&gt; element MUST contains one or more &lt;author&gt; elements at l.1460 c.7
  URL: http://patrick.sirref.org/weeklies/atom.xml
Fetching Richard Mortier...
  79 posts (79 new)
Fetching Ryan Gibb...
  38 posts (38 new)
Fetching Sadiq Jaffer...
  10 posts (10 new)

Total: 127 new posts (3 errors)
</code></pre>
<p>Either we skip content, or talk to the people involved to fix their feeds, but
it's Christmas eve so that's unlikely. And anyway, we want to be <a href="https://en.wikipedia.org/wiki/Robustness_principle">liberal in
what we accept</a> so why
can't I fix my own software first?!</p>
<p><img src="https://anil.recoil.org/images/aoah-tuatara-ss-1.webp" alt="%c" title="Like the amazing Tuatara, why don't we build evolution directly into our software? Time to take off the semantics seatbelt..."></p>
<h2><a href="https://anil.recoil.org/news.xml#medice-cura-te-ipsum" class="anchor" aria-hidden="true"></a>Medice, cura te ipsum</h2>
<p>The non-obvious and probably-terrible answer here is to use our fancy coding
models to force the Tuatara source code to heal itself.  I added an <code>--evolve</code>
flag to allow tuatara to invoke <a href="https://anil.recoil.org/notes/aoah-2025-4">Claude/OCaml</a> upon errors and patch its own code.
Turns out self help works, as Martin's blog feed was just fine!</p>
<pre><code class="language-bash">Fetching Martin Kleppmann...
  Error: Feed parse error: document MUST contains exactly one &lt;feed&gt; element at l.0 c.0
  URL: http://feeds.feedburner.com/martinkl

Invoking Claude Code to fix parse error...

The feed from `http://feeds.feedburner.com/martinkl` is an **RSS 2.0 feed** (it
starts with `&lt;rss version="2.0"&gt;`), but tuatara was incorrectly detecting it as
an **Atom feed**. This caused the Syndic Atom parser to fail with "document
MUST contains exactly one \&lt;feed\&gt; element".

The root cause was that the `detect_feed_type` function prioritized the HTTP
`Content-Type` header over the actual content. FeedBurner (and other feed
aggregators) often serve RSS feeds with an incorrect `application/atom+xml`
content-type header.

This is a generic fix that will work for any feed aggregator or CDN that
mis-labels RSS feeds as Atom (or vice versa), so no domain-specific quirk was
needed.
</code></pre>
<p>But the parsing drama continued, as <a href="https://mynameismwd.org">Michael Dales</a> uses the wrong date format in his
feed (tsk tsk I'd send the RFC police out if it wasn't Christmas), but Tuatara
<a href="https://tangled.org/anil.recoil.org/tuatara/commit/a56c4a877936a1ae49c0fab301fb6afbcc74b3bb">evolves a quirk</a> that gets past it:</p>
<blockquote>
<p>The quirk module converts ISO 8601 dates (2025-10-22T12:24:00-00:00) to RFC
822 format (Wed, 22 Oct 2025 12:24:00 GMT) which is what Syndic's RSS2 parser
expects.</p>
</blockquote>
<p>And <a href="https://www.cst.cam.ac.uk/people/og309">Onkar Gulati</a> and <a href="https://patrick.sirref.org">Patrick Ferris</a> both have an empty author field which would
ordinarily give us a dreaded <code>Not_found</code> exception:</p>
<blockquote>
<p>Fetching Patrick Ferris...Error: Feed parse error: <entry> elements MUST contains at least an <author> element or <feed> element MUST contains one or more <author> elements at l.1460 c.7 URL: http://patrick.sirref.org/weeklies/atom.xml</author></feed></author></entry></p>
</blockquote>
<p>But never fear, the inexorable <code>--evolve</code> flag figures it out and patches its own code!</p>
<p>There were some non-trivial quirks as well; <a href="https://ancazugo.github.io/">Andres Zuñiga-Gonzalez</a> uses Quatro for his website which puts the entire HTML blob into the summary field, but the evolution managed to use <a href="https://anil.recoil.org/notes/aoah-2025-15">html5rw</a> to parse <a href="https://tangled.org/anil.recoil.org/tuatara/commit/7f29b37e1c647f984589e42164a0fc2ec0cda5c4">its way out of this</a>. This sort of fix is very hard to generalise, so it's actually quite useful for the tool to fix itself on demand for our small group.</p>
<h2><a href="https://anil.recoil.org/news.xml#using-the-claude-frontend-design" class="anchor" aria-hidden="true"></a>Using the Claude frontend design</h2>
<p>Then I needed a quick way to do a clean frontend output so I can visualise the
JSONfeed. Claude has a <code>/plugin frontend-design</code> skill that is built in, and
prompting it to give me a few designs let me integrate a <code>--html</code> output.</p>
<p>And because it's Christmas, I added some snowflakes as well. Yay!</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/eeg-xmas"> <img src="https://anil.recoil.org/images/aoah-tuatara-ss-4.webp" alt="%c" title="Ho ho ho merry xmas everyone from the EEG feed that isnt live yet but will be after the new year"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>The paper I enjoyed writing the most this year was <a href="https://anil.recoil.org/papers/2025-internet-ecology">Steps towards an Ecology for the Internet</a> for
<a href="https://anil.recoil.org/notes/ecology-at-aarhus">Aarhus 2025</a>. In the back of my head since has been a desire
to start figuring out what self-evolving software actually might be. It's a
strange, and probably impractical idea, but I'm delighted that I took a tiny
step towards it with this project.</p>
<p>Back in March, I had the honour of being invited to a <a href="https://bellairs.net">Bellairs</a> meeting to discuss a heady combination of semantics and computational science. <a href="https://jonmsterling.com">Jon Sterling</a> demonstrated his wonderfully organised Forester website. And I... showed how my mismash of semi-structured writings can kind of be connected together in a vaguely coherent way to build my website. Next year will have me thinking much harder about the implications of <a href="https://anil.recoil.org/papers/2025-internet-ecology">self-evolving code</a>, of how radically <a href="https://anil.recoil.org/papers/2025-biodiversity-9recs">transformative to global biodiversity</a> semi-structured agentic processing might be, and other heavy matters. But to close this year, I'm disproportionately pleased to have gotten my tiny website under control a little!</p>
<p><img src="https://anil.recoil.org/images/aoah-tuatara-ss-2.webp" alt="%c" title="Sitting indoors in Barbados with a gigantic beach outside: a classic sign of semanticists in the wild"></p>
<p>As I noted in my <a href="https://anil.recoil.org/notes/acm-ai-recs">letter to the ACM</a>, it's important that we can use AI for things that boost the
human condition; I really enjoy reading my colleagues' long form thoughts much
more than doomscrolling on the web, and so making it easier to gather their
thoughts digestibly and easily is a nice end to my <a href="https://anil.recoil.org/news.xml">agentic humps</a>
effort. Tomorrow on <a href="https://anil.recoil.org/notes/aoah-2025-25">Christmas</a> I'll publish all the skills I used so others can try them out.</p>

