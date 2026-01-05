---
title: Four Ps for Building Massive Collective Knowledge Systems
description: "Design principles for collective knowledge systems\u2014permanence,
  provenance, permission, and placement\u2014that enable robust networks for evidence-based
  decision making."
url: https://anil.recoil.org/notes/principles-for-collective-knowledge
date: 2025-11-23T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aria-ci-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>I've been building some big <a href="https://anil.recoil.org/notes/rs-future-of-publishing">collective knowledge</a> systems recently, both for <a href="https://anil.recoil.org/papers/2025-evidence-tap">scholarly literature</a> or to power large-scale <a href="https://anil.recoil.org/papers/2025-tessera">observational foundation models</a>.  While the modalities of
knowledge in these systems are very different, they share a common set of design principles I've noticed while building
<a href="https://anil.recoil.org/notes/geotessera-python">individual</a> <a href="https://anil.recoil.org/notes/uk-national-data-lib">pieces</a>. A good computer architecture is one that can be re-used, and I've been mulling over what this exactly is for some time.</p>
<p>I found the perfect place to codify this at the <a href="https://www.aria.org.uk/opportunity-spaces/collective-flourishing">ARIA Workshop on Collective Flourishing</a> that <a href="https://toao.com">Sadiq Jaffer</a> and I attended in Birmingham last week. I posit there are <em>"4 P's"</em> needed for any collective knowledge system to be robust and accurate: <strong>permanence, provenance, permission and placement</strong>. If these properties exist throughout our knowledge graph, we can make robust networks for rapid <a href="https://anil.recoil.org/notes/ai-for-evidence-synthesis-workshop">evidence-based decision making</a>. They also form a dam against the wave of <a href="https://anil.recoil.org/notes/claude-copilot-sandbox">agentic AI</a> that is going to dominate the Internet next year in a big way.</p>
<p><em>Will building these collective knowledge systems be a transformative capability for human society?</em> Hot on the heels of COP30 <a href="https://www.bbc.co.uk/news/articles/cp84m16mdm1o">concluding indecisively</a>, I've been getting excited by decision making towards <em>biodiversity</em> going down a more positive path in <a href="https://www.cbd.int/sbstta/ipbes.shtml">IPBES</a>. We could empower decisionmakers at all scales (local, country, international) to be able to move <a href="https://fivetimesfaster.org/">five times faster</a> on actions about global species extinctions, unsustainable wildlife trade and food security, while rapidly assimilating extraordinarily complex evidence chains. I'll talk about this more while explaining the principles...</p>
<p>This post is split up into a few parts. First, let's <a href="https://anil.recoil.org/news.xml#collective-flourishing-at-aria">introduce ARIA's convening role</a> in this. Then I'll <a href="https://anil.recoil.org/news.xml#the-4ps-for-collective-knowledge-systems">introduce the principles</a> of <a href="https://anil.recoil.org/news.xml#p1-permanence-aka-dois-for-all-with-the-rogue-scholar">permanence</a>, <a href="https://anil.recoil.org/news.xml#p2-provenance-is-it-ai-poison-or-precious">provenance</a>, <a href="https://anil.recoil.org/news.xml#p3-permission-not-everything-needs-to-go-into-the-borg">permission</a>, and <a href="https://anil.recoil.org/news.xml#p4-placement-data-has-weight-and-geopolitics-matters">placement</a>. The post does assume some knowledge of Internet protocols; I'll write a more accessible one for a general audience at a future date when the ideas are more baked!</p>
<h2><a href="https://anil.recoil.org/news.xml#collective-flourishing-at-aria" class="anchor" aria-hidden="true"></a>Collective Flourishing at ARIA</h2>
<p>The ARIA workshop was held in lovely Birmingham, hosted by programme manager <a href="https://scholar.google.com/citations?user=7MTl9CAAAAAJ&amp;hl=en">Nicole Wheeler</a>. It explored four core beliefs that ARIA had published about this opportunity space in collective flourishing:</p>
<blockquote>
<ol>
<li>Navigating towards a better future requires clarity on direction and path &gt; <strong>we need the capability to make systemic complexity legible so we can envision and deliberate over radically different futures.</strong></li>
<li>Simply defining our intent for the future is not enough → <strong>we need a means of negotiating our fragmented values into shared, actionable plans for collective progress.</strong></li>
<li>Our current cognitive, emotional, and social characteristics are not immutable constants → <strong>human capacity can and will change over time, and we need tools to figure out together how we navigate this change.</strong></li>
<li>Capabilities that augment our vision, action, and capacity are powerful and can have unintended consequences → <strong>we must balance the pressing need for these tools with the immense responsibility they entail.</strong>
<cite>-- <a href="https://www.aria.org.uk/opportunity-spaces/collective-flourishing">ARIA Collective Flourishing Opportunity Space</a>, 2025</cite></li>
</ol>
</blockquote>
<p>I agree with these values, and translating these into concrete
systems concepts seems a useful exercise.  The workshop was under <a href="https://www.chathamhouse.org/about-us/chatham-house-rule">Chatham
House rules</a>, which
hamstrings my ability to credit individuals, but the gathering was a
useful and eclectic mix of social scientists and technologists.  There was also
a real sense of collective <em>purpose</em>: a desire to <a href="https://ukfoundations.co/">reignite UK growth</a> and <a href="https://www.bensouthwood.co.uk/p/regional-inequality-also-a-housing">decrease inequality</a>.</p>
<p><img src="https://anil.recoil.org/images/aria-ci-1.webp" alt="%c" title="While I cannot confirm nor deny any individual's presence at the workshop, there was much banter late into the evening at the pub afterwards!"></p>
<h2><a href="https://anil.recoil.org/news.xml#the-4ps-for-collective-knowledge-systems" class="anchor" aria-hidden="true"></a>The 4P's for Collective Knowledge Systems</h2>
<p>First, why come up with these system design principles at all? I believe strongly both in building systems from the groundup, and also in
<a href="https://en.wikipedia.org/wiki/Eating_your_own_dog_food">eating my own dogfood</a>
and using whatever I build. I also define knowledge broadly: not just academic
papers, but also geospatial datasets, blogs and other more conventionally
"informal" knowledge sources that are increasingly complementing scholarly
publishing as a source of timely knowledge.<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup></p>
<p>Towards this, several of my colleagues such as <a href="https://jonmsterling.com">Jon Sterling</a> have been building systems like <a href="https://www.forester-notes.org/tfmt-000V/index.xml">Forester</a>, and I've got my own homebrew <a href="https://anil.recoil.org/notes/bushel-lives">Bushel</a>, and <a href="https://patrick.sirref.org">Patrick Ferris</a> has <a href="https://patrick.sirref.org/weekly-2025-w45/index.xml">Graft</a>, and <a href="https://mynameismwd.org">Michael Dales</a> has decades of <a href="https://digitalflapjack.com/blog/">Atom/RSS</a> on his sites. These sites are very loosely coupled -- they're built by different people over many years -- but there is already the beginnings of a rich mesh of hyperlinks across them.</p>
<p>To get to the next level of collective meshing (not just for these personal sites but for <a href="https://anil.recoil.org/papers/2025-biodiversity-9recs">biodiversity data</a> as well), I posit we need to explicitly engineer in support for permanence, provenance, permission and placement right into the way we access data across the Internet. If we build these mechanisms via Internet <em>protocols</em>, collective knowledge can be meshed without any one entity requiring central control. I view this as being vital for the Internet as a whole to <a href="https://anil.recoil.org/papers/2025-internet-ecology">evolve and adapt</a> into the coming decades and combat <a href="https://en.wikipedia.org/wiki/Enshittification">enshittification</a>.</p>
<p>I'll now dive into the four principles: (i) <a href="https://anil.recoil.org/news.xml#p1-permanence-aka-dois-for-all-with-the-rogue-scholar">permanence</a>; (ii) <a href="https://anil.recoil.org/news.xml#p2-provenance-is-it-ai-poison-or-precious">provenance</a>; (iii) <a href="https://anil.recoil.org/news.xml#p3-permission-not-everything-needs-to-go-into-the-borg">permission</a>; and (iv) <a href="https://anil.recoil.org/news.xml#p4-placement-data-has-weight-and-geopolitics-matters">placement</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#p1-permanence-aka-dois-for-all-with-the-rogue-scholar" class="anchor" aria-hidden="true"></a>P1: Permanence (aka DOIs for all with the Rogue Scholar!)</h2>
<p>Firstly, knowledge that is spread around the world needs a way to be retrieved
reliably. Scholarly publications, especially open-access ones, are distributed
both digitally and physically and often <a href="https://www.researchgate.net">replicated</a>. While papers are "big" enough
pieces of work to warrant this effort, what about all the other outputs we have
such as (micro)blogs, social media posts, and datasets?  A reliable addressing
system is essential to be retrieve these too, and we can do this via standard
Internet protocols such as HTTP and DNS.</p>
<p>The eagle-eyed among you might notice that my site now has a unique DOI for most post.
A <a href="https://doi.org">Digital Object Identifier</a> is something you more conventionally
see associated with academic papers, but thanks to the hard work of <a href="https://front-matter.de/team">Martin Fenner</a>
we can now have them for other forms of content!  Martin set up the <a href="https://rogue-scholar.org/">Rogue Scholar</a>
which permits <em>any</em> standards-compliant site with an Atom or JSONFeed to be assigned a DOI automatically.</p>
<p>This post, for example, has the
<a href="https://doi.org/10.59350/418q4-gng78">10.59350/418q4-gng78</a> DOI assigned to
it, which forms its unique DOI identifier.  This can be resolved into the real
location by retrieving the DOI URL, which issues a HTTP redirect to this post:</p>
<pre><code>$ curl -I https://doi.org/10.59350/418q4-gng78
HTTP/2 302
date: Wed, 26 Nov 2025 12:48:43 GMT
location: https://anil.recoil.org/notes/fourps-for-collective-knowledge
</code></pre>
<p>Crucially, this DOI URL is not the <em>only</em> identifier for this post, as you can still
also use my original homepage URL. However, it's an identifier that can
be redirected to a new location if the content moves, and <em>also</em> has extra metadata associated
with it that help with keeping track of networks of knowledge.</p>
<p><a href="https://rogue-scholar.org"> <img src="https://anil.recoil.org/images/aria-ci-4.webp" alt="%c" title="You too can sign up to the Rogue Scholar with your blog!"> </a></p>
<p>Let's look at some more details of the extra useful metadata, by peering at one of my <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">recent posts</a> to see how Rogue Scholar <a href="https://rogue-scholar.org/records/kqwjw-cjb76">augments the metadata for it</a>. They (i) <a href="https://anil.recoil.org/news.xml#tracking-authorship-metadata">track author identities</a>; (ii) build a <a href="https://anil.recoil.org/news.xml#forming-a-reference-mesh">reference mesh</a> across items; and (iii) <a href="https://anil.recoil.org/news.xml#archiving-and-versioning-posts">archive clean versions</a> to replicate content with an open license.</p>
<h3><a href="https://anil.recoil.org/news.xml#tracking-authorship-metadata" class="anchor" aria-hidden="true"></a>Tracking authorship metadata</h3>
<p>Firstly, the authorship information helps to identify me concretely across name variations. My own <a href="https://orcid.org/0000-0001-8954-2428">ORCID</a> forms a unique identifier for my own scholarly publishing, and this is now <a href="https://blog.front-matter.de/posts/rogue-scholar-links-records-via-orcid-and-doi/">tied</a> to my blog post.
You can then <a href="https://rogue-scholar.org/search?q=orcid:0000-0001-8954-2428&amp;l=list&amp;p=1&amp;s=10&amp;sort=newest">search for my ORCID</a> and find my posts, but <em>also</em> find it in other indexing systems such as <a href="https://search.crossref.org/?from_ui=&amp;q=0000-0001-8954-2428">CrossRef</a> which index scholarly metadata.  <a href="https://blog.openalex.org/were-rebuilding-openalex-while-its-running-heres-whats-changing/">OpenAlex has just rewritten</a> their codebase and released it a few weeks ago, with 10s of millions of new types of works indexed.</p>
<p>Curating databases that are this large across decades clearly leads to some inconsistencies as people move around jobs and change their circumstances. Identifying "who" has done something is therefore a surprisingly tricky metadata problem. This is one of the areas where <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">ATProto, Bluesky and Tangled</a> have a lot to offer, by allowing the social graph to be shared among multiple differentiated services (e.g. microblogging or code hosting).</p>
<p><a href="https://arxiv.org/pdf/2402.03239"> <img src="https://anil.recoil.org/images/aria-ci-5.webp" alt="%rc" title="Bluesky is a highly reusable social authentication mechanism"> </a></p>
<h3><a href="https://anil.recoil.org/news.xml#forming-a-reference-mesh" class="anchor" aria-hidden="true"></a>Forming a reference mesh</h3>
<p>Secondly, references from links within this post are extracted out and linked
to <em>other</em> DOIs. I do this by generating a <a href="https://anil.recoil.org/perma.json">structured JSONFeed</a>
which breaks out metadata for each post by scanning the links within my source
Markdown. For example, here is an excerpt for one of the <code>"references"</code> fields
in my post:</p>
<pre><code class="language-json">{ "url": "https://doi.org/10.33774/coe-2025-rmsqf",
  "doi": "10.33774/coe-2025-rmsqf",
  "cito": [ "citesAsSourceDocument" ] }, {
  "url": "https://doi.org/10.59350/hasmq-vj807",
  "doi": "10.59350/hasmq-vj807",
  "cito": [ "citesAsRelated" ] },
</code></pre>
<p>This structured list of references also includes <a href="https://purl.archive.org/spar/cito">CITO</a> conventions to also list <em>how</em> the citation should be interpreted, which may be useful input to LLMs that are interpreting a document. I've <a href="https://ocaml.org/p/jsonfeed/latest">published</a> an <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed/blob/main/lib/cito.mli">OCaml-JSONFeed</a> library that conveniently lists all the citation structures possible.
This reference metadata is hoovered up by databases such as <a href="https://search.crossref.org/search/works?q=10.59350/hasmq-vj807&amp;from_ui=yes">CrossRef</a> which use them to maintain their giant graph databases that associate posts, papers and anything else with a DOI with each other.</p>
<p>To make this as easy as possible to do with any blog content online, Rogue Scholar has augmented <a href="https://blog.front-matter.de/posts/rogue-scholar-references-learn-new-tricks/">how it scans posts</a> so that just adding a "References" header to your content is enough to make this just work. We now have an interconnected mesh of links between diverse blogs and papers and datasets, all using simple URLs!</p>
<h3><a href="https://anil.recoil.org/news.xml#archiving-and-versioning-posts" class="anchor" aria-hidden="true"></a>Archiving and versioning posts</h3>
<p>Thirdly, the metadata and Atom feeds are used to archive the contents of the post via the <a href="https://blog.front-matter.de/posts/rogue-scholar-blog-posts-archived-by-internet-archive/">Internet Archive Archive-It service</a>. This is also not as straightforward as you might expect; the problem with archiving HTML straight from the source is that the web pages you read are usually quite a mess of JavaScript and display logic, whereas the <em>essence</em> of the page is hidden.</p>
<p>For example, look at the <a href="https://web.archive.org/web/20250909194331/https://anil.recoil.org/notes/owntracks-and-lifecycle">archive.org version</a> of one of my posts vs the <a href="https://rogue-scholar.org/records/wm49x-a9q51">Rogue Scholar version</a> of the same post.  The latter is significantly cleaner, since the "archival" version actually uses my <a href="https://anil.recoil.org/perma.json">blog feed</a> instead of the original HTML.
The feed reader version strips out all the unnecessary display gunk so that it can be read by clients like <a href="https://netnewswire.com/">NetNewsWire</a> or Thunderbird. There is some work that needs to happen on the Atom feed generation side to really make this clean; for example, I learnt about how to <a href="https://simonwillison.net/2024/Aug/1/footnotes-that-work-in-rss-readers/">lay out footnotes to be feed-reader friendly</a>.</p>
<p><a href="https://web.archive.org/web/20250909194331/https://anil.recoil.org/notes/owntracks-and-lifecycle"> <img src="https://anil.recoil.org/images/aria-ci-2.webp" alt="%c" title="The raw archive version of the post, which keeps all the display HTML"> </a></p>
<p><a href="https://rogue-scholar.org/records/wm49x-a9q51"> <img src="https://anil.recoil.org/images/aria-ci-3.webp" alt="%c" title="The feed reader version of the post, with a clean reader view that is easier to mechanically interpret"> </a></p>
<p>To wrap up the first P of Permanence, we've seen that it's a bit more involved than "simply archiving it". Some metadata curation and formatting flexibility really helps to clean up the connections. If you have your own blog, you should sign up to <a href="https://rogue-scholar.org">Rogue Scholar</a>. Martin has just incorporated it as a <a href="https://doi.org/10.53731/rftfk-qv692">German non-profit organisation</a>, showing he's thinking about the long-term sustainability of such ventures as well.</p>
<h2><a href="https://anil.recoil.org/news.xml#p2-provenance-is-it-ai-poison-or-rare-literature" class="anchor" aria-hidden="true"></a>P2: Provenance (is it AI poison or rare literature?)</h2>
<p>The enormous problem we're facing with collective intelligence right now is that the Internet is getting flooded by AI generated slop. While there are obvious dangers to our collective sanity and attention spans, there's also the pragmatic problem that <a href="https://www.nature.com/articles/s41586-024-07566-y">recursive training causes model collapse</a>. If we just feed our models the output of other language models, we greatly dilute the quality of the resulting LLMs and the overall quality of collective knowledge.</p>
<p>We observed the societal implications in our <a href="https://anil.recoil.org/papers/2025-ai-poison">recent Nature comment</a>:</p>
<blockquote>
<p>The publication of ever-larger numbers of problematic papers, including fake ones generated by artificial intelligence, represents an existential crisis for the established way of doing evidence synthesis. But with a new approach, AI might also save the day.
<cite>-- <a href="https://rdcu.be/evkfj">Will AI speed up literature reviews or derail them entirely?</a>, 2025</cite></p>
</blockquote>
<p>We urgently need to build accurate provenance information into our collective knowledge networks to distinguish <em>where</em> some piece of knowledge came from. Efforts like Rogue Scholar and <a href="https://blog.kagi.com/small-web">Kagi Small Web</a> do this by human judgement: a community keeps an eye on the feeds and filters out the obviously bad actors. <a href="https://www.shaneweisz.com">Shane Weisz</a> also pointed out to me that crowdsourcing communities also often self-organise like this. For example, <a href="https://www.inaturalist.org/journal/gcwarbler/115290-cv-and-geomodel-predictions-visibility-identifiability-seasonality-apples-oranges">iNaturalist volunteers</a> painstakingly critique AI output <em>vs</em> human experts for species detection. Provenance in these systems would help them scale their efforts without burning out.</p>
<p>Luckily though, we do have some partial solutions already for keeping track of
provenance:</p>
<ul>
<li>Code can be versioned through Git, now widely adopted, but also <a href="https://anil.recoil.org/notes/tangled-and-ci">federated via Tangled</a>.</li>
<li>Data can be traced through services like <a href="https://zenodo.org">Zenodo</a> and even <a href="https://help.zenodo.org/docs/deposit/describe-records/reserve-doi/">given DOIs</a> just like Rogue Scholar has been doing. This is not perfect yet since it's difficult to <a href="https://anil.recoil.org/papers/2025-yirgacheffe">continuously update large datasets</a>, but technology is steadily advancing here.</li>
<li>Code <em>and</em> data can be versioned through dataflow systems, of which there are many out there include several we discussed at <a href="https://anil.recoil.org/notes/icfp25-propl">PROPL 2025</a>, such as <a href="https://www.cse.iitd.ac.in/~aseth/">Aadi Seth</a>'s <a href="https://dl.acm.org/doi/10.1145/3759536.3763803">dynamic STACs</a> or our own <a href="https://anil.recoil.org/news.xml">OCurrent</a>, or <a href="https://codeocean.com/resources/nature-partnership">Nature+CodeOcean</a> for scientific computation.</li>
<li>Rogue Scholar <a href="https://doi.org/10.53731/nxp08-a9947">supports DOI versioning</a> of posts to allow intentional edits of the same content.</li>
</ul>
<p>What's missing is a provenance <em>protocol</em> by which each of these "islands of provenance" can interoperate across each other's boundaries. Almost every project runs its own CI systems that never share the details of how they got their data and code. Security organisations are now recommending <a href="https://www.ncsc.gov.uk/blog-post/sboms-and-the-importance-of-inventory">Software Bill of Materials</a> be generated for all software, and <a href="https://www.docker.com/products/hardened-images/">Docker Hardened Images</a> are acting as an anchor for wider efforts in this space. The IETF <a href="https://blog.aayushg.com/standards/">is moving to advance standards of provenance</a> but perhaps too slowly and conservatively given the <a href="https://anil.recoil.org/notes/ai-ietf-aiprefs">rapid rise of AI crawlers</a>.</p>
<p>An area I'm going to investigate in the future is how HTTP-based provenance headers
might help glue these together, so that a collective knowledge crawler doesn't
need to build a global provenance graph (which would be overwhelmingly massive)
to filter out non-trusted primary content.</p>
<p><a href="https://rdcu.be/evkfj"> <img src="https://anil.recoil.org/images/davidparkins-ai-poison.webp" alt="%c" title="AI poisoning the literature in a legendary cartoon. Credit: David Parkins, Nature"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#p3-permission-not-everything-needs-to-go-into-the-borg" class="anchor" aria-hidden="true"></a>P3: Permission (not everything needs to go into the Borg)</h2>
<p>The Internet is pretty good about building giant public databases, and it's
also pretty good at supporting storing secret data. However, it's <em>terrible</em>
at supporting semi-private access to remote sites.</p>
<p>Consider a really obvious collective knowledge case: I want to expose my draft
papers that I'm working on with a diverse group of people. I collaborate with
dozens of people all over the world, and so want to selectively grant them
access to my works-in-progress.  Why is this so difficult to do?</p>
<p>It's currently easy to use <em>individual</em> services to grant access; for example,
I might share my <a href="https://overleaf.com">Overleaf</a> or my Google Drive for a
project, but propagating those access rights across services is near impossible
as soon as you cross a project or API boundary.  There are a few directions we
could go to break this problem down into easier to solve chunks:</p>
<ul>
<li>If we make it easier to self-host services, for example via initiatives like <a href="https://doi.org/10.59350/s621r-eg143">Eilean</a>, then having access to the databases directly makes it much easier to take nuanced decisions about which bits of the data to grant access to. I run, for example, three separate video hosting sites: one for <a href="https://watch.ocaml.org">OCaml</a>, for the <a href="https://watch.eeg.cl.cam.ac.uk">EEG</a> and another <a href="https://crank.recoil.org">personally</a>. Each of these federates across each other via <a href="https://activitypub.rocks">ActivityPub</a>, but still supports private videos.</li>
<li>There was research into distributed permission protocols like <a href="https://research.google/pubs/macaroons-cookies-with-contextual-caveats-for-decentralized-authorization-in-the-cloud/">Macaroons</a> at the height of the cloud boom a decade ago, but they've all been swallowed up into the bottomless pit of pain that is <a href="https://oauth.net/2/">oAuth</a>. It's high time we resurrected some of the more nuanced work on fine-grained authentication that doesn't give access to absolutely everything and/or SMS you at 2am requesting a verification code.</li>
<li>Rather than 'yes/no' decisions, we could also share different <em>views</em> of the data depending on who's asking. This used to be difficult due to the combinatorics involved, but you could imagine nowadays applying a local LLM to figure out the rich context. The <a href="https://github.com/google-deepmind/concordia">DeepMind Concordia</a> project takes this idea even further with social simulations based on the same principles.</li>
</ul>
<p>When we look at the <a href="https://anil.recoil.org/notes/rs-future-of-publishing">current state of the publishing industry</a>, it becomes clearer why a few publishers are hoovering up smaller journals. Maintaining the infrastructure for open/closed access is just a lot easier in a centralised setup than it is when distributed. However, it's crucial for expanding the ceiling on our collective knowledge that we support such federated access to semi-private data. Let's consider another sort of data to see why...</p>
<h3><a href="https://anil.recoil.org/news.xml#biodiversity-needs-spatial-permissioning" class="anchor" aria-hidden="true"></a>Biodiversity needs spatial permissioning</h3>
<p>Zooming out to a global usecase, biodiversity data is a prime example of where
everything can't be open. Economically motivated rational actors (i.e.
poachers) are highly incentivised to use all available data to figure out where
to snarf a rare species, and so some of this presence data is vital to keep
tight control over. But the pendulum swings both ways, and without robust
permissions mechanisms to share the data with well intentioned actors, we
cannot make evidence-driven planning decisions for global topics such as <a href="https://anil.recoil.org/notes/exploring-food-impacts">food consumption</a>.</p>
<p>I asked <a href="https://www.cambridgeconservation.org/about/people/professor-neil-burgess/">Neil Burgess</a>, the Chief Scientist of <a href="https://www.unep-wcmc.org/en">UNEP-WCMC</a>, and <a href="https://scholar.google.com/citations?user=fK2N8doAAAAJ&amp;hl=en">Violeta Muñoz-Fuentes</a> about their views on how biodiversity data might make an impact if connected together. They gave me a remarkable list of databases they maintain (an excerpt reproduced with permission here):</p>
<blockquote>
<ul>
<li><a href="https://www.protectedplanet.net">Protected Planet</a>, 10s of thousands of users. The world's most trusted, up-to-date, and complete source of information on protected areas and other effective area-based conservation measures (OECMs). Includes effectiveness of protected and conserved area management; updated monthly with submissions from governments, NGOs, landowners, and communities.</li>
<li><a href="https://unbiodiversitylab.org/en/">UN Biodiversity Lab</a>, 1000s of governments, NGO users.  A geospatial platform with 400+ of the world’s best data layers on nature, climate change, and sustainable development. Supports country-led efforts for planning, monitoring, and reporting; linked to the Convention on Biological Diversity's global nature agreements.</li>
<li><a href="https://tradeview.cites.org/">CITES Wildlife Trade View</a>, 1000s of government and NGO users. Visualizes legal wildlife trade globally, by species and by country.</li>
<li><a href="https://trade.cites.org/">CITES Wildlife Trade Database</a>, 10000s of government, NGO, research users.  Contains records of all legal wildlife trade under CITES (&gt;40,900 species globally).</li>
<li><a href="https://www.ibat-alliance.org/">IBAT</a>, 1000s of businesses.  Spatial tool for businesses to calculate potential impacts on nature. IBAT is an alliance of BirdLife International, Conservation International, IUCN, and UNEP-WCMC.
<cite>-- An excerpt of UNEP-WCMC tools and systems (N. Burgess, personal communication, 2025)</cite></li>
</ul>
</blockquote>
<p>This is just a short excerpt from the list, and many of these involve illegal activities (tracking them, not doing them!). The value in connecting them together and making them safely accessible by both humans and AI agents would be transformative to the global effort to save species from extinction, for example by carefully picking and choosing what trade agreements are signed between countries. A real-time version could change the course of human history for pivotal global biodiversity conferences where negotiations decide the future of many.</p>
<p>So I make a case that we <em>must</em> engineer robust permission protocols into the heart of how we share data, and not just for copyright and legal reasons. Some data must stay private for security, economic or geopolitical reasons, but that act of hiding knowledge currently makes it very difficult to take part in a collective knowledge network with our current training architectures. Perhaps <a href="https://en.wikipedia.org/wiki/Federated_learning">federated learning</a> will be one breakthrough, but I'm betting on <a href="https://anil.recoil.org/papers/2024-hope-bastion">agentic permissions</a> being where this goes instead.</p>
<p><a href="https://unbiodiversitylab.org/en"> <img src="https://anil.recoil.org/images/aria-ci-6.webp" alt="%c" title="The UN Biodiversity Lab has spatial biodiversity information for thousands of species across the world, collected by many, many people over the years. (source: Violeta Muñoz-Fuentes, UNEP-WCMC)"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#p4-placement-data-has-weight-and-geopolitics-matters" class="anchor" aria-hidden="true"></a>P4: Placement (data has weight, and geopolitics matters)</h2>
<p>The final P is one that we thought we wouldn't need to worry about thanks to
<a href="https://anil.recoil.org/news.xml">the cloud</a> back in the day: placement. A lot of the digital data
involved in our lives is spatial in nature (e.g. our movement data), but also
must be <em>accessed</em> only from some locations. If we don't engineer in location as
a first-class element of how we treat collective knowledge, it'll never be a truely
useful knowledge companion to humans.</p>
<h3><a href="https://anil.recoil.org/news.xml#physical-location-matters-a-lot-for-knowledge-queries" class="anchor" aria-hidden="true"></a>Physical location matters a lot for knowledge queries</h3>
<p>We explained some spatial ideas in our recent <a href="https://anil.recoil.org/papers/2025-bifrost">Bifrost</a> paper:</p>
<blockquote>
<p>Physical containment creates a natural network hierarchy, yet we do not currently take advantage of this. Even local interactions between devices often require traversal over a wide-area network (WAN), with consequences for privacy, robustness, and latency.</p>
<p>Instead, devices in the same room should communicate directly, while physical barriers should require explicit networking gateways. We call this spatial networking: instead of overlaying virtual addresses over physical network connections, we use physical spaces to constrain virtual network addresses.</p>
<p>This lets users point at two devices and address them by their physical relationship; devices are named by their location, policies are scoped
by physical boundaries, and spaces naturally compose while maintaining local autonomy.
<cite>-- <a href="https://anil.recoil.org/papers/2025-bifrost">An Architecture for Spatial Networking</a>, Millar et al, 2025</cite></p>
</blockquote>
<p>Think about all the times in your life that you've wanted to pull up some specific knowledge about the region you're in, and how bad our digital systems currently are at dealing with fine-grained location. I go to the gym every Sunday morning like clockwork with <a href="https://github.com/djs55">Dave Scott</a>, and yet my workout app treats it like a whole new experience every single time I open it.</p>
<p>Similarly, if you have a group of people in a meeting room, they should be able to use their physical proximity to take advantage of that inherent trust! For example, photos weren't allowed the Collective Intelligence workshop due to the Chatham House rules, but it would have been <em>really</em> useful to be able to get a copy of other people's photos for me to have a personal record of all the amazing whiteboarding brainstorming that was going on.</p>
<p>Protocol support for placement, combined with permissioning above, would allow us to build a personal knowledge network that actually fits into our lives based on where we physically are.</p>
<h3><a href="https://anil.recoil.org/news.xml#where-code-and-data-is-hosted-also-matters" class="anchor" aria-hidden="true"></a>Where code and data is hosted also matters</h3>
<p>When I started working on <a href="https://anil.recoil.org/projects/plancomp">planetary computing</a>, the most obvious
change was just how vast the datasets involved are. We've just installed a
<a href="https://www.tunbury.org/2025/10/18/quick-look-at-ceph/">multi petabyte cluster</a> in the
Computer Lab just to deal with the embeddings from <a href="https://anil.recoil.org/papers/2025-tessera">TESSERA</a>,
and syncing those takes weeks even on a gigabit link. This data can't just
casually move, which means that in turn we can't use many cloud-based services
like GitHub for our hosting. And this, in turn, disconnects us from all the
collective knowledge training happening there for their code foundation models.</p>
<p>An alternative is to decouple the names of the code and data from where it's hosted.
This is a feature explicitly supported by the <a href="https://arxiv.org/abs/2402.03239">AT Protocol</a> that underpins Bluesky.
There are <a href="https://anil.recoil.org/notes/atproto-for-fun-and-blogging">dozen of alternative services</a> that are springing up
that can reuse the authentication infrastructure, but allow users to choose <em>where</em> their data is hosted.</p>
<p>The one we are using most here in my group is <a href="https://tangled.org">Tangled</a>, which is a code hosting service
that I've <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">described before</a> and <a href="https://tangled.org/anil.recoil.org">use regularly</a>.
What makes this service different from other code hosting is that while I can remain social and share, the
actual code is stored on a "knot" that I host. I run several: one on my personal <a href="https://recoil.org">recoil.org</a> domain,
and another for my colleagues in the Cambridge Computer Lab. Code that sits there can also be run using <a href="https://anil.recoil.org/notes/tangled-and-ci">local CI</a>
which can access private data stored in our local network, by virtue of the fact that we run our own infrastructure.</p>
<p>To wrap up the principle of placement, I've made a case for why explicit control over locations (of people, of code, of data, of predictions) matter a lot for collective intelligence, and should be factored into any system architecture for this. If you don't believe me, try asking your nearest LLM for what the best pub is near you, and watch it hallucinate and burn!</p>
<p><img src="https://anil.recoil.org/images/okavango-2.webp" alt="%c" title="Offline access also still matters, for latency and battery life, or you're simply in the middle of nowhere"></p>
<h2><a href="https://anil.recoil.org/news.xml#next-directions" class="anchor" aria-hidden="true"></a>Next Directions</h2>
<p>I jotted down these four principles to help organise my thoughts, and they're by no means set in stone. I am reasonably convinced that the momentum building around <a href="https://atprotocol.dev/atmosphereconf/">ATProto usage</a> worldwide makes it a compelling place to focus prototyping and research efforts on, and they are working on plugging gaps such as <a href="https://atproto.wiki/en/working-groups/private-data">permission support</a> already. If you'd like to work on this or have pointers for me, please do let me know! I'll update this post as they come in.</p>
<p><small class="credits">(I'd like to thank many people for giving me input and ideas into this post, many of whom are cited above. In particular, Sadiq Jaffer, Shane Weisz, Michael Dales, Patrick Ferris, Cyrus Omar, Aadi Seth, Michael Coblenz, Jon Sterling, Nate Foster, Aurojit Panda, Ian Brown, Srinivasan Keshav, Jon Crowcroft, Ryan Gibb, Josh Millar, Hamed Haddadi, Sam Reynolds, Alec Christie, Bill Sutherland, Violeta Muñoz-Fuentes and Neil Burgess have all poured in relevant ideas, along with the wider ATProto and ActivityPub communities)</small></p>

<div class="footnotes"><ol><li><p></p><p>As I write this, the UK's budget announcement was released <a href="https://www.bbc.co.uk/news/live/cy8vz032qgpt">an hour early</a> by the OBR, throwing markets into real-time uncertainty.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy et al (2025). Steps towards an Ecology for the Internet. Association for Computing Machinery. <a href="https://doi.org/10.1145/3744169.3744180" target="_blank"><i>10.1145/3744169.3744180</i></a></li>
<li>Madhavapeddy (2025). Royal Society's Future of Scientific Publishing meeting. <a href="https://doi.org/10.59350/nmcab-py710" target="_blank"><i>10.59350/nmcab-py710</i></a></li>
<li>Feng et al (2025). TESSERA: Temporal Embeddings of Surface Spectra for Earth Representation and Analysis. arXiv. <a href="https://doi.org/10.48550/arXiv.2506.20380" target="_blank"><i>10.48550/arXiv.2506.20380</i></a></li>
<li>Madhavapeddy (2025). A fully AI-generated paper just passed peer review; notes from our evidence synthesis workshop. <a href="https://doi.org/10.59350/k540h-6h993" target="_blank"><i>10.59350/k540h-6h993</i></a></li>
<li>Madhavapeddy (2025). Oh my Claude, we need agentic copilot sandboxing right now. <a href="https://doi.org/10.59350/aecmt-k3h39" target="_blank"><i>10.59350/aecmt-k3h39</i></a></li>
<li>Dales et al (2025). Yirgacheffe: A Declarative Approach to Geospatial Data. Association for Computing Machinery. <a href="https://doi.org/10.1145/3759536.3763806" target="_blank"><i>10.1145/3759536.3763806</i></a></li>
<li>Madhavapeddy (2025). Thoughts on the National Data Library and private research data. <a href="https://doi.org/10.59350/fk6vy-5q841" target="_blank"><i>10.59350/fk6vy-5q841</i></a></li>
<li>Madhavapeddy (2025). What I learnt at ICFP/SPLASH 2025 about OCaml, Hazel and FP. <a href="https://doi.org/10.59350/w1jvt-8qc58" target="_blank"><i>10.59350/w1jvt-8qc58</i></a></li>
<li>Jaffer et al (2025). AI-assisted Living Evidence Databases for Conservation Science. Cambridge Open Engage. <a href="https://doi.org/10.33774/coe-2025-rmsqf" target="_blank"><i>10.33774/coe-2025-rmsqf</i></a></li>
<li>Millar et al (2025). An Architecture for Spatial Networking. arXiv. <a href="https://doi.org/10.48550/arXiv.2507.22687" target="_blank"><i>10.48550/arXiv.2507.22687</i></a></li>
<li>Reynolds et al (2025). Will AI speed up literature reviews or derail them entirely?. Nature Publishing Group. <a href="https://doi.org/10.1038/d41586-025-02069-w" target="_blank"><i>10.1038/d41586-025-02069-w</i></a></li>
<li>Madhavapeddy (2025). Socially self-hosting source code with Tangled on Bluesky. <a href="https://doi.org/10.59350/r80vb-7b441" target="_blank"><i>10.59350/r80vb-7b441</i></a></li>
<li>Madhavapeddy (2025). Programming for the Planet at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/hasmq-vj807" target="_blank"><i>10.59350/hasmq-vj807</i></a></li>
<li>Madhavapeddy (2025). The AIETF arrives, and not a moment too soon. <a href="https://doi.org/10.59350/agfta-8wk09" target="_blank"><i>10.59350/agfta-8wk09</i></a></li>
<li>Madhavapeddy (2025). Arise Bushel, my sixth generation oxidised website. <a href="https://doi.org/10.59350/0r62w-c8g63" target="_blank"><i>10.59350/0r62w-c8g63</i></a></li>
<li>Madhavapeddy (2025). Exploring the biodiversity impacts of what we choose to eat. <a href="https://doi.org/10.59350/xj427-y3q48" target="_blank"><i>10.59350/xj427-y3q48</i></a></li>
<li>Madhavapeddy (2025). GeoTessera Python library released for geospatial embeddings. <a href="https://doi.org/10.59350/7hy6m-1rq76" target="_blank"><i>10.59350/7hy6m-1rq76</i></a></li>
<li>Madhavapeddy (2025). mlgpx is the first Tangled-hosted package available on opam. <a href="https://doi.org/10.59350/7267y-nj702" target="_blank"><i>10.59350/7267y-nj702</i></a></li>
<li>Madhavapeddy (2025). Using AT Proto for more than just Bluesky posts. <a href="https://doi.org/10.59350/32rdt-zny05" target="_blank"><i>10.59350/32rdt-zny05</i></a></li>
<li>Fenner (2025). Rogue Scholar is becoming a German Non-Profit Organization. Front Matter. <a href="https://doi.org/10.53731/rftfk-qv692" target="_blank"><i>10.53731/rftfk-qv692</i></a></li>
<li>Fenner (2025). Rogue Scholar starts supporting versioning. Front Matter. <a href="https://doi.org/10.53731/nxp08-a9947" target="_blank"><i>10.53731/nxp08-a9947</i></a></li>
<li>Gibb (2025). Eilean. Front Matter. <a href="https://doi.org/10.59350/s621r-eg143" target="_blank"><i>10.59350/s621r-eg143</i></a></li>
<li>Shumailov et al (2024). AI models collapse when trained on recursively generated data. Nature. <a href="https://doi.org/10.1038/s41586-024-07566-y" target="_blank"><i>10.1038/s41586-024-07566-y</i></a></li>
<li>Laud et al (2025). STACD: STAC Extension with DAGs for Geospatial Data and Algorithm Management. <a href="https://doi.org/10.1145/3759536.3763803" target="_blank"><i>10.1145/3759536.3763803</i></a></li>
<li>Kleppmann et al (2024). Bluesky and the AT Protocol: Usable Decentralized Social Media. Proceedings of the ACM Conext-2024 Workshop on the Decentralization of the Internet. <a href="https://doi.org/10.1145/3694809.3700740" target="_blank"><i>10.1145/3694809.3700740</i></a></li></ul>
