---
title: 'AoAH Day 14: Debugging a Karakeep CLI against the live service'
description: Vibe coding an OCaml library for the Karakeep bookmarking service by
  giving an agent a live API key and letting it debug jsont codecs against the real
  service.
url: https://anil.recoil.org/notes/aoah-2025-14
date: 2025-12-14T00:00:00-00:00
preview_image: https://anil.recoil.org/images/karakeep-ss.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>With the <a href="https://anil.recoil.org/notes/aoah-2025-13">Requests</a> library under my belt, I finally got to what
I actually need for myself: vibe coding OCaml library interfaces to my
<a href="https://anil.recoil.org/news.xml">##selfhosted</a> services that contain most of my data.</p>
<p>To start with, I use
<a href="https://karakeep.app">Karakeep</a> across all my devices to bookmark things, and
I'd like to be able to programmatically search through tags, for example by
taking all outbound links from the blogs that I read and autosynching them with
my remote service. Karakeep on the server side does some cool things like
screenshot links and create local webarchives.</p>
<p>Unfortunately, Karakeep doesn't publish an OCaml interface.  Fortunately, my
new bestie Claude helped me build
<a href="https://tangled.org/anil.recoil.org/ocaml-karakeep">ocaml-karakeep</a> without
much input from me!</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p><a href="https://karakeep.app"> <img src="https://anil.recoil.org/images/karakeep-ss.webp" alt="%c" title="I keep a ton of bookmarks in Karakeep, and its Reader mode is convenient to quickly scan papers and such."> </a></p>
<p>With a ton of pre-Christmas organising to do this weekend, I didn't have a lot of
spare time. So I thought I'd use everything I've learnt so far and try to
one-shot this as bravely/foolishly as possible. I setup a repo with:</p>
<ul>
<li>Access to all the previous libraries, including the <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed">jsonfeed</a> to act as an 'exemplar' for using jsont, as well as <a href="https://tangled.org/anil.recoil.org/ocaml-requests">requests</a>.</li>
<li>Cloned the <a href="https://github.com/karakeep-app/karakeep">karakeep source code</a> which has the details of the API somewhere in the Node source. It uses something called <a href="https://trpc.io">tRPC</a> that I'm not familiar with.</li>
<li>I took a ZFS snapshot of my karakeep deployment, and gave my agent a <strong>live API key</strong> to access my node. Yes, this is the 'take the seatbelt off' moment.</li>
</ul>
<p>And then I prompted the agent to build jsont specifications derived from the
Karakeep source, and then hook this up to Requests, and then iterate using my
live key to debug failures until the API spec was right.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>This almost worked first time! While the testcases generated worked, I <em>also</em> got the agent to generate a karakeep CLI command. I'd made sure that, as with <a href="https://anil.recoil.org/notes/aoah-2025-3">xdge</a>, the Requests library also exposes Cmdliner terms that permit easy integration of HTTP requests into any CLI (to configure logging, proxies, that sort of thing).</p>
<p><img src="https://anil.recoil.org/images/karakeep-ss-3.webp" alt="%c" title="I could just invoke the new karakeep CLI and try out different API functionality directly."></p>
<p>Since I didn't have time to do a full coverage test of the API, I asked the agent to exercise the functionality and do live debugging. This is where the jsont codec was incredibly useful, since it provided all the right hints to the agent to take action. Good error messages aren't just for humans any more!</p>
<p>A typical error, for example was:</p>
<pre><code>&gt; main.exe bookmarks summarize wqa1sc1z42xcxfwwrsxeggbl
main.exe: [ERROR] Karakeep error: JSON error: Missing members in bookmark object:
 content
 createdAt
 id
File "-", line 1, characters 0-615: 
</code></pre>
<p>The agent had enough data from that error message to figure things out and make the jsont codec more permissive with option types.</p>
<p><img src="https://anil.recoil.org/images/karakeep-ss-2.webp" alt="%c" title="The agent live debugs the missing JSON fields, looks up the original Karakeep source, and fixes the jsont codecs"></p>
<p>The library also made good use of the stateful HTTP interface from Requests, for example by modifying all the default requests to use the API key:</p>
<pre><code class="language-ocaml">let create ~sw ~env ~base_url ~api_key =                                                                                                                      
  let session = Requests.create ~sw env in                                                                                                                    
  let session =                                                                                                                                               
    Requests.set_auth session (Requests.Auth.bearer ~token:api_key)                                                                                           
  in                                                                                                                                                          
  { session; base_url }
</code></pre>
<p>The <code>karakeep.proto</code> subpackage has all the dedicated jsont decoders, which take care of conversion to-and-from OCaml types. I found these quite readable:</p>
<pre><code class="language-ocaml">(** Type of content a bookmark can have *)                                                                                                                    
type bookmark_content_type =                                                                                                                                  
  | Link  (** A URL to a webpage *)                                                                                                                           
  | Text  (** Plain text content *)                                                                                                                           
  | Asset  (** An attached asset (image, PDF, etc.) *)                                                                                                        
  | Unknown  (** Unknown content type *)                                                                                                                      
                                                                                                                                                              
val bookmark_content_type_jsont : bookmark_content_type Jsont.t
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>The new agent trick I learnt today is the power of debugging against live services. The jsont descriptions derived from the Karakeep source code provided just enough information in error messages that the agent could fix them by trying to run the command against my live service. Luckily, nothing <a href="https://www.reddit.com/r/google_antigravity/comments/1p82or6/google_antigravity_just_deleted_the_contents_of/">got deleted</a> this time, but in the future projects like <a href="https://patrick.sirref.org/weekly-2025-w49/index.xml">Shelter</a> will be ever more important.</p>
<p><a href="https://mynameismwd.org">Michael Dales</a> <a href="https://eeg.zulipchat.com/#narrow/channel/522690-Blogs/topic/My.202025.20Advent.20of.20Agentic.20Humps.3A.20a.20new.20library.20daily/near/563008635">asked</a> an interesting question: <em>"Why OCaml over an even more strict language?"</em>.  My instinct is that OCaml is pretty much the best choice here; I think it's pretty hard to find a usable language with a stronger typing discipline. The more exotic the type systems get, the harder it is to do proof discharge. However, I believe that with some reinforcement learning magic, <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml's lifetimes</a> might be gamechanging when combined with agents that can also reason about performance and memory in addition to conventional type safety.</p>
<p>Anyway, I now have a <a href="https://tangled.org/anil.recoil.org/ocaml-karakeep">karakeep binary and library</a> that's working great for my search needs. Back to bookmarking Christmas presents I needed to have obtained yesterday!</p>

