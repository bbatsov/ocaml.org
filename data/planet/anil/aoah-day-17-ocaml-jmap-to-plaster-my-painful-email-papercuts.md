---
title: 'AoAH Day 17: OCaml JMAP to plaster my painful email papercuts'
description: Building an OCaml JMAP client that runs in browsers and CLI, then using
  it to build personalised email workflows for taming notification overload.
url: https://anil.recoil.org/notes/aoah-2025-17
date: 2025-12-17T00:00:00-00:00
preview_image: https://anil.recoil.org/images/shriram-vibe-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After building a <a href="https://anil.recoil.org/notes/aoah-2025-16">JSON Pointer library</a> yesterday, I proceeded
to complete my <a href="https://tangled.org/anil.recoil.org/ocaml-jmap">OCaml JMAP</a>
library today so that I could wrestle my overflowing email inbox under control.
Email is <a href="https://www.ncsc.gov.uk/collection/phishing-scams">central</a> to our digital lives and yet we have mostly <a href="https://www.theverge.com/24113616/gmail-email-20-years-old-internet">ceded control</a> to third-party services for something that unlocks access to almost any service we use.</p>
<p>Luckily, I've been self-hosting my <a href="https://anil.recoil.org/notes/decentralised-stack">own email</a> for some time, so I do have full local access to about three decades worth of messages. However, I've been hampered by existing email clients which are mostly geared towards a temporal view and not towards easy programmability. So today's exercise has been to build an <strong><a href="https://tangled.org/anil.recoil.org/ocaml-jmap">ocaml-jmap</a></strong> that lets me write little agentic programs to help me manage my ever overflowing inbox!</p>
<p><a href="https://x.com/ShriramKMurthi/status/2001361749539459166"> <img src="https://anil.recoil.org/images/shriram-vibe-1.webp" alt="%c" title="Shriram vibes a fair criticism of my current email strategy"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#why-jmap-for-email" class="anchor" aria-hidden="true"></a>Why JMAP for email?</h2>
<p>Historically, the protocol of choice for accessing email has been <a href="https://datatracker.ietf.org/doc/html/rfc3501">IMAP</a>, and for years I used the <a href="https://github.com/nojb/ocaml-imap">Lwt ocaml-imap</a> library that <a href="https://github.com/nojb">Nicolás Ojeda Bär</a> wrote back when he was here in Cambridge. However, IMAP is a pretty convoluted protocol that hasn't evolved much over time, and requires a number of accreted <a href="https://www.ietf.org/archive/id/draft-rjbs-mailmaint-imap-extensions-suggestions-00.html">extensions</a> that are patchily implemented.</p>
<p>The <a href="https://en.wikipedia.org/wiki/JSON_Meta_Application_Protocol">JSON Meta Application Protocol</a> (or JMAP) was developed in response to this. The developers wrote:</p>
<blockquote>
<p>JMAP is the result of efforts to address shortcomings [in existing protocols], providing a modern, efficient, easy-to-use API, built on many years of experience and field testing.
<cite>-- <a href="https://www.ietf.org/blog/jmap/">JMAP, a modern open email protocol</a>, Gondwana and Jenkins, May 2019</cite></p>
</blockquote>
<p>JMAP's exciting because it allows for the reuse of a number of other protocol implementations I've already built these past few weeks. I can use <a href="https://anil.recoil.org/notes/aoah-2025-13">ocaml-requests</a> and <a href="https://anil.recoil.org/notes/aoah-2025-12">conpool</a> for HTTPS connections, and my ever-growing dependence on <a href="https://github.com/dbuenzli/jsont">jsont</a> means that I can get reasonable error messages while debugging the implementation.</p>
<p>Servers that support JMAP are still thin on the ground, but all my email endpoints do now support it. At work in the Cambridge Computer Lab, the enlightened sysadmins gave us <a href="https://app.fastmail.com">Fastmail accounts</a> as an alternative to having Exchange inflicted on us by the central University.  Over at Recoil, my personal mail is switching to <a href="https://github.com/stalwartlabs/stalwart">Stalwart JMAP</a>. While there's still no good JMAP <em>client</em> that replaces my day-to-day email, all these services also expose IMAP for that purpose. This clears the way for me to vibecode up a JSON library in OCaml for the programmatic fragments that I so crave!</p>
<h2><a href="https://anil.recoil.org/news.xml#buildinghhhvibing-ocaml-jmap" class="anchor" aria-hidden="true"></a>Building^H^H^Hvibing OCaml JMAP</h2>
<p>Interestingly, I had also tried to build an OCaml JMAP waaay back when <a href="https://anil.recoil.org/notes/claude-copilot-sandbox">Claude Code was first released</a>, and the difference in nine months of model development is massive; browse my <a href="https://tangled.org/anil.recoil.org/ocaml-jmap/tree/old1">first attempt here</a> and see what a mess it was.</p>
<p>But now that I've got a bunch of examples under my belt, the latest round of vibing of <a href="https://tangled.org/anil.recoil.org/ocaml-jmap">ocaml-jmap</a> was extremely fast. The agent had examples of jsont use from <a href="https://anil.recoil.org/notes/aoah-2025-2">jsonfeed</a> and the <a href="https://anil.recoil.org/notes/aoah-2025-14">Karakeep REST client</a>, and <a href="https://tangled.org/anil.recoil.org/ocaml-requests">ocaml-requests</a> provides direct-style HTTP requests with authentication bearer support. I <a href="https://anil.recoil.org/notes/aoah-2025-16">vibesplained</a> a Javascript tutorial to help me view messages.</p>
<h3><a href="https://anil.recoil.org/news.xml#building-a-browser-based-jmap-client-to-test" class="anchor" aria-hidden="true"></a>Building a browser-based JMAP client to test</h3>
<p>However, I wanted to go a little further than <em>just</em> notebook style tutorials. In addition to compiling OCaml code to JavaScript, there's also really good support for programming browser API directly through libraries like <a href="https://github.com/dbuenzli/brr">Brr</a>. These expose an OCaml interface that allows us to build up browser-specific logic to better integrate within it.</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-1.webp" alt="%c" title="Connecting to fastmail from a browser OCaml JMAP compiled to JavaScript"></p>
<p>So I took the notebook idea one step further and prompted up an entire JMAP client that runs in the browser (albeit a very simple one!). This <a href="https://tangled.org/anil.recoil.org/ocaml-jmap/blob/main/lib/js/jmap_brr.mli">Json_brr</a> library looks just like normal OCaml, except that it uses libraries designed to run in the browser as its dependencies. The interface shows some differences</p>
<pre><code class="language-ocaml">type connection
val api_url : connection -&gt; Jstr.t
(** [api_url conn] returns the API URL for requests. *)

(** {1 Session Establishment} *)

val get_session :
  url:Jstr.t -&gt;
  token:Jstr.t -&gt;
  (connection, Jv.Error.t) result Fut.t
(** [get_session ~url ~token] establishes a JMAP session.
</code></pre>
<p>Instead of <code>string</code>, this uses <a href="https://erratique.ch/software/brr/doc/Jstr/index.html">Jstr.t</a> for JavaScript strings. Instead of Eio, this also uses <a href="https://erratique.ch/software/brr/doc/Fut/index.html">Fut.t</a> to encode future promises.
But aside from those things, it's just plain OCaml! The <a href="https://tangled.org/anil.recoil.org/ocaml-jmap/blob/main/lib/js/jmap_brr.ml">implementation</a> makes Websocket requests to connect to a remote JMAP server in the browser.</p>
<pre><code class="language-ocaml">let fetch_json ~url ~meth ~headers ?body () =
  Console.(log [str "&gt;&gt;&gt; Request:"; str (Jstr.to_string meth); str (Jstr.to_string url)]);
  (match body with
   | Some b -&gt; Console.(log [str "&gt;&gt;&gt; Body:"; b])
   | None -&gt; Console.(log [str "&gt;&gt;&gt; No body"]));
  let init = Brr_io.Fetch.Request.init
    ~method':meth
    ~headers
    ?body
    ()
  in
  let req = Brr_io.Fetch.Request.v ~init url in
  let* response = Brr_io.Fetch.request req in
</code></pre>
<p>To use <a href="https://tangled.org/anil.recoil.org/ocaml-jmap/tree/main/web">jmap.html</a>, I obtained a (read only!!) API key from my live email server, and then managed to connect to my live email <em>and</em> get a protocol debugger, all from my browser.</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-2.webp" alt="%c" title="The client also dumps the JMAP protocol messages so I can learn more about how it works."></p>
<p>One important performance point about how this works is that some of the base libraries I'm using take advantage of OCaml's modularity in order to expose <em>browser specific backends</em>. For example, <a href="https://github.com/dbuenzli/jsont/blob/main/src/brr/jsont_brr.ml">Jsont_brr</a> uses the native browser JSON parser while still working with Jsont codecs. This sort of casual modularity is an extremely nice and undersung feature of OCaml, and was also key in <a href="https://anil.recoil.org/papers/2019-mirage-functors">MirageOS being portable</a> to so many <a href="https://anil.recoil.org/videos/287364fa-b59c-4b9f-812d-d81cc0c992a5">embedded devices</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#fixing-my-notifications-with-programmable-email" class="anchor" aria-hidden="true"></a>Fixing my notifications with programmable email</h2>
<p>So now we have a working library, I wanted to go beyond the conventional email clients. With agentic coding, I should be able to construct <em>hundreds</em> of domain-specific bits of code that help me manipulate my email <strong>the way I want it done</strong>.  This was the dream of <a href="https://anil.recoil.org/news.xml">##selfhosting</a> and open source in the first place, but it was too much work to write all that glue code... until now.</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-3.webp" alt="%c" title="The sad state of my 10000s of GitHub notifications, which I mostly ignore these days"></p>
<p>One papercut that bugs both me and <a href="https://tarides.com/blog/author/mark-elvers/">Mark Elvers</a> is the flood of email notifications we get that are rapidly out of date. I get thousands a day from GitHub, and ideally they would be marked as read and filed away automatically when I finish with it on the remote service, and not stay in my Inbox.  This is now easily solveable using OCaml JMAP, so I built a proof of concept to filter away my <a href="https://eeg.zulipchat.com">Zulip notifications</a> nicely.</p>
<h3><a href="https://anil.recoil.org/news.xml#jmap-keywords-and-cli-fragments" class="anchor" aria-hidden="true"></a>JMAP keywords and CLI fragments</h3>
<p>I built a domain-specific CLI called <a href="https://tangled.org/anil.recoil.org/ocaml-jmap/blob/main/bin/jmapq.ml">jmapq</a> which exists solely to implement specialist workflows. I'll break this out into a private repo for my own use later, this is just a prototype!</p>
<p>Zulip sends a nicely structured email with a subject like "#Blogs &gt; My 2025 Advent of Agentic Humps: a new library daily [Cambridge Energy &amp; Environment Group]" every so often which I instructed the agent to <a href="https://tangled.org/anil.recoil.org/ocaml-jmap/blob/main/bin/jmapq.ml#L22-46">parse into an OCaml data structure</a>:</p>
<pre><code class="language-ocaml">(** Parsed information from a Zulip notification email subject.
    Subject format: "#Channel &gt; topic [Server Name]" *)
module Zulip_message = struct
  type t = {
    id : string;
    date : Ptime.t;
    thread_id : string;
    channel : string;
    topic : string;
    server : string;
    is_read : bool;
    labels : string list;
  }

  (** Parse a Zulip subject line of the form "#Channel &gt; topic [Server Name]" *)
  let parse_subject subject =
    (* Pattern: #&lt;channel&gt; &gt; &lt;topic&gt; [&lt;server&gt;] *)
    let channel_re = Re.Pcre.regexp {|^#(.+?)\s*&gt;\s*(.+?)\s*\[(.+?)\]$|} in
    match Re.exec_opt channel_re subject with
    | Some groups -&gt;
        let channel = Re.Group.get groups 1 in
        let topic = Re.Group.get groups 2 in
        let server = Re.Group.get groups 3 in
        Some (channel, topic, server)
    | None -&gt; None
</code></pre>
<p>Once we have this, I then added another CLI command to list all the Zulip notifications in my email, which it outputs using nicely structured JSON output that is parseable by <a href="https://github.com/jqlang/jq">jq</a>.</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-4.webp" alt="%c" title="Given a read only API key, this parses out just what I need from Zulip"></p>
<p>Then I instructed the CLI to give me more grouped views, so I can see notifications in my email by topic and by server (I now also get notifications from other Zulips I'm on like the <a href="https://ocaml.zulipchat.com">OCaml</a> or <a href="https://zulip.com/case-studies/lean/">Lean</a>).</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-5.webp" alt="%c" title="The vibe coded CLI can show really specific queries like my Zulip channels"></p>
<p>And then the really sketchy bit. I swapped API keys to have a read/write one (it could delete all my email in theory), and then -- while only mildly sweating -- ran the timeout command. I did add a dry-run mode so I could see the query first before I let it rip.</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-6.webp" alt="%c" title="Running AI against my live email with a vibe coded client is not how I imagined 2025 to go."></p>
<p>And et voila, it added the right keywords and marked things as unread, and my live email view in Fastmail now allows for <a href="https://www.fastmail.help/hc/en-us/articles/360060591213-Searching-your-mail">keyword searches</a> that show me <em>exactly</em> what I want.</p>
<p><img src="https://anil.recoil.org/images/aoah-jmap-ss-7.webp" alt="%c" title="I'm not sure what the limit on keywords is, but Fastmails search interface is great"></p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>Today's been great; I managed to find a papercut that's been bugging me for
years, and code up something quickly that genuinely makes my personal workflows
a little bit better. I used to do this a <a href="https://anil.recoil.org/projects/perscon">lot</a> before the pandemic, but somehow
the shift to online services has really damaged my ability to program my own
digital life.</p>
<p>I feel back in control again; although
<a href="https://tangled.org/anil.recoil.org/ocaml-jmap">ocaml-jmap</a> should be used by
anyone else with extreme caution, I'm rigging up some ZFS snapshots of my own
email so I can code up a few hundred custom agents over things and see how it
goes.  But mostly, I'm happy to have found a glimmer of the <a href="https://resonantcomputing.org">Resonant Computing manifesto</a>
through the medium of strong OCaml types, self hosted services and agentic coding glue:</p>
<blockquote>
<p>Dedicated: Software should work exclusively for you, ensuring contextual
integrity where data use aligns with your expectations. You must be able to
trust there are no hidden agendas or conflicting interests.
<cite>-- <a href="https://resonantcomputing.org">Resonant Computing Manifesto</a>, 2025</cite></p>
</blockquote>
<p>I'm going to work on wrestling GitHub and Netdata under control next. And what about the other side of the Zulip notifications? Well, I'll knock up a Zulip bot in OCaml tomorrow, so stay tuned for that!</p>

