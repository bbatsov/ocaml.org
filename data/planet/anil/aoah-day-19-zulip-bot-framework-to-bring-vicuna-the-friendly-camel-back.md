---
title: 'AoAH Day 19: Zulip bot framework to bring Vicuna the friendly camel back'
description: Building an OCaml Zulip bot framework with functional handlers, and pivoting
  from TOML to INI codecs for Python configparser compatibility
url: https://anil.recoil.org/notes/aoah-2025-19
date: 2025-12-19T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-zulip-regress-2.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After building <a href="https://anil.recoil.org/notes/aoah-2025-18">tomlt</a> yesterday for TOML 1.1 parsing, I proceeded to integrate it with my group's <a href="https://zulip.com">Zulip</a> chat <a href="https://eeg.zulipchat.com">server</a>. I then discovered that Zulip actually uses Python's <a href="https://docs.python.org/3/library/configparser.html">configparser</a> INI format for its <code>.zuliprc</code> files rather than TOML, woops! But this gave me the perfect opportunity to attempt to quickly replicate the tomlt experience with a <em>third</em> config format codec library for Windows-style INI files as well.</p>
<p>So today I released both <strong><a href="https://tangled.org/anil.recoil.org/ocaml-zulip">ocaml-zulip</a></strong> for Zulip API integration and <strong><a href="https://tangled.org/anil.recoil.org/ocaml-init">ocaml-init</a></strong> for INI file codecs that are compatible with Pythonic features such as variable interpolation. Along the way, I developed a new regression test mechanism by writing a Zulip bot that tests the Zulip API using OCaml Zulip!</p>
<h2><a href="https://anil.recoil.org/news.xml#zulip-organized-chat-for-distributed-teams" class="anchor" aria-hidden="true"></a>Zulip: Organized chat for distributed teams</h2>
<p>Zulip is an open-source "async first" messaging app that strikes a nice balance between immediate and thoughtful conversations:</p>
<blockquote>
<ul>
<li>In Zulip, channels determine who gets a message. Each conversation within a channel is labeled with a topic, which keeps everything organized.</li>
<li>You can read Zulip one conversation at a time, seeing each message in context, no matter how many other conversations are going on.</li>
<li>If anything is out of place, it's easy to move messages, rename and split topics, or even move a topic to a different channel
<cite>-- <a href="https://zulip.com/why-zulip/">Why Zulip?</a>, 2024</cite></li>
</ul>
</blockquote>
<p>Zulip itself is fully open source and has a pretty straightforward REST API to communicate with the server, and so I deployed my <a href="https://anil.recoil.org/notes/aoah-2025-13">requests library</a> as well the various API codecs to interface with it. I used the <a href="https://github.com/zulip/python-zulip-api">Zulip Python SDK</a> and the <a href="https://github.com/zulip/zulip-js">Zulip JavaScript</a> library to give me two API specifications. Unlike previous libraries I've vibecoded, there's no language-agnostic test suite so I needed to get a bit more creative to verify correctness by building a live bot to test itself.</p>
<h3><a href="https://anil.recoil.org/news.xml#the-zulip-ini-config-format" class="anchor" aria-hidden="true"></a>The Zulip INI config format</h3>
<p>Zulip's <code>.zuliprc</code> file looks like this:</p>
<pre><code>[api]
email = bot@example.com
key = your-api-key-here
site = https://your-domain.zulipchat.com
</code></pre>
<p>This is classic INI format as used by Python's configparser module. It's
simpler than TOML but isn't fully compatible as it has quirks like case-insensitive
keys, multiline value support via continuation lines, and basic variable
interpolation with a <code>%(name)s</code> syntax.</p>
<p>I couldn't find a feature complete implementation of Python's module, so I quickly
reused the <a href="https://anil.recoil.org/notes/aoah-2025-18">tomlt</a> approach to build <a href="https://tangled.org/anil.recoil.org/ocaml-init">ocaml-init</a> with bidirectional
codecs. The resulting API is unsurprisingly extremely similar to manipulate this format file:</p>
<pre><code class="language-ocaml">type server_config = { host : string; port : int; debug : bool }

let server_codec = Init.Section.(
  obj (fun host port debug -&gt; { host; port; debug })
  |&gt; mem "host" Init.string ~enc:(fun c -&gt; c.host)
  |&gt; mem "port" Init.int ~enc:(fun c -&gt; c.port)
  |&gt; mem "debug" Init.bool ~dec_absent:false ~enc:(fun c -&gt; c.debug)
  |&gt; finish
)
</code></pre>
<p>There's a <code>bool</code> codec in this library that follows Python's configparser
semantics exactly, accepting yes/no/true/false/on/off/1/0 as boolean values.
This was important for compatibility with existing Zulip configuration files.</p>
<h2><a href="https://anil.recoil.org/news.xml#the-zulip-bot-framework" class="anchor" aria-hidden="true"></a>The Zulip bot framework</h2>
<p>With configuration parsing sorted, I turned to building the actual bot
framework. Our research group at <a href="https://eeg.zulipchat.com">eeg.zulipchat.com</a>
has been wanting an Atom feed bot to post updates from our blogs' Atom/RSS sources,
so this seemed like a good excuse to knock up a bot.</p>
<p>I prompted the agent to follow the basic <a href="https://zulip.com/api/deploying-bots">Python
botserver</a> considerations but adapted to
a more Eio and OCaml idiomatic style. This resulted in a nice design where a
bot handler is just a function:</p>
<pre><code class="language-ocaml">type handler =
  storage:Storage.t -&gt; identity:identity -&gt;
  Message.t -&gt; Response.t
</code></pre>
<p>The Zulip library provides modules for <code>storage</code> for persisting state (on the Zulip server side), <code>identity</code> containing functions to access the bot's email and user ID, and the incoming <code>Message.t</code>. The handler returns a <code>Response.t</code> which can be a reply in the same context (DM or channel/topic), or a post to a specific channel, or a direct message, or an indication that the bot's ignoring the event.</p>
<p>There's an echo bot handler that's executable that shows the API quite simply:</p>
<pre><code class="language-ocaml">let echo_handler ~storage ~identity msg =
  let bot_email = identity.Bot.email in
  let sender_email = Message.sender_email msg in

  (* Ignore our own messages *)
  if sender_email = bot_email then Response.silent
  else
    (* Remove bot mention and echo back *)
    let cleaned_msg = Message.strip_mention msg ~user_email:bot_email in
    if cleaned_msg = "" then
      Response.reply (Printf.sprintf "Hello %s!" (Message.sender_full_name msg))
    else
      Response.reply (Printf.sprintf "Echo: %s" cleaned_msg)
</code></pre>
<p>After this running the bot in an Eio environment is a single function call:</p>
<pre><code class="language-ocaml">let () =
  Eio_main.run @@ fun env -&gt;
  Eio.Switch.run @@ fun sw -&gt;
  let config = Zulip_bot.Config.load ~fs:(Eio.Stdenv.fs env) "echo-bot" in
  Zulip_bot.Bot.run ~sw ~env ~config ~handler:echo_handler
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#tying-it-all-together-with-requests" class="anchor" aria-hidden="true"></a>Tying it all together with Requests</h2>
<p>One nice payoff from this <a href="https://anil.recoil.org/notes/aoah-2025">advent</a> series is seeing how
the libraries begin to compose. The Zulip OCaml package depends on
<a href="https://anil.recoil.org/notes/aoah-2025-13">Requests</a> for HTTPS communication back to the server:</p>
<pre><code class="language-ocaml">let create ~sw env auth =
  let session =
    Requests.create ~sw
      ~default_headers:(Requests.Headers.of_list [
        ("Authorization", Auth.to_basic_auth_header auth);
        ("User-Agent", "OCaml-Zulip/1.0");
      ])
      ~follow_redirects:true
      ~verify_tls:true
      env
  in
  { auth; session }
</code></pre>
<p>This shows how the session abstraction in Requests can persist the common auth
tokens required, making subsequent API calls very syntactically succinct.  The
bot framework also uses <a href="https://anil.recoil.org/notes/aoah-2025-3">XDGe</a> for configuration directory
resolution, <a href="https://anil.recoil.org/notes/aoah-2025-2">jsont</a> for JSON parsing of API responses, and of
course Eio throughout for async operations. The dependency graph is starting to
look like actual infrastructure!</p>
<h2><a href="https://anil.recoil.org/news.xml#testing-with-a-regression-bot" class="anchor" aria-hidden="true"></a>Testing with a regression bot</h2>
<p>Remember that problem I mentioned earlier about Zulip lacking a
language-agnostic test suite? My cunning solution was recursive; let's just
build a Zulip bot that can test itself!
I built a <a href="https://tangled.org/anil.recoil.org/ocaml-zulip/blob/main/examples/regression_test.ml">regression test bot</a> that exercises as
much of the Zulip API as possible when triggered via a direct message:</p>
<pre><code class="language-ocaml">let make_handler ~env ~channel =
  fun ~storage ~identity:_ msg -&gt;
    let content = String.lowercase_ascii (Message.content msg) in
    let sender_email = Message.sender_email msg in
    (* Only respond to DMs containing "regress" *)
    if Message.is_private msg &amp;&amp; String.starts_with ~prefix:"regress" content
    then (
      let client = Storage.client storage in
      let summary = run_tests ~env ~client ~channel ~trigger_user:sender_email in
      Response.reply summary)
    else Response.silent
</code></pre>
<p>When someone sends the bot a DM with "regress", it runs through a comprehensive
test suite covering user operations, channel management, message
sending/editing, reactions, message flags, typing indicators, presence updates,
and alert words.  I duly started the harness and DMed Vicuna, and the bot
immediately spat out a number of failures resulting from errors in the codecs.
However, as the logs show, the errors also included useful information about
where the protocol decoding had gone wrong.</p>
<p><img src="https://anil.recoil.org/images/aoah-zulip-regress-1.webp" alt="%c" title="The first run of my Zulip regression bot"></p>
<p>The bot posts a summary to a test channel showing which tests passed or failed,
complete with timing information. This turned out to be more useful than
traditional unit tests since it exercises the actual API against a real Zulip
server. After one round of fixes, the bot successfully posted its results to
a Zulip channel recording success!</p>
<p><img src="https://anil.recoil.org/images/aoah-zulip-regress-2.webp" alt="%c" title="The bot posts the results of its Zulip regression test to Zulip!"></p>
<h2><a href="https://anil.recoil.org/news.xml#composable-command-line-terms" class="anchor" aria-hidden="true"></a>Composable command-line terms</h2>
<p>One pattern I've been developing across these libraries is to expose <a href="https://github.com/dbuenzli/cmdliner">cmdliner</a>
terms that compose together to make it easy to build CLI tools that expose
the configuration needed by a library along with a manual page.</p>
<p>The <code>Zulip_bot.Cmd</code> module provides a <code>config_term</code> that bundles common bot configuration parameters:</p>
<pre><code class="language-ocaml">let config_term default_name env =
  let fs = env#fs in
  Term.(const (fun name config_file verbosity verbose_http -&gt;
        setup_logging ~verbose_http:verbose_http.value verbosity.value;
        load_config ~fs ~name ~config_file)
    $ name_term default_name
    $ config_file_term
    $ verbosity_term
    $ verbose_http_term default_name)
</code></pre>
<p>While this looks complex, all it's doing is to combine various declarations
of command-line parameters.  This combines individual terms for the bot name, config file path, verbosity level,
and HTTP-level debugging into a single composable unit.</p>
<p>The <code>verbose_http_term</code> controls logging sources in the <a href="https://anil.recoil.org/notes/aoah-2025-13">Requests</a>
library, letting you toggle detailed HTTP traces without that being the default verbose output.</p>
<pre><code class="language-ocaml">let verbose_http_term app_name =
  let env_name = String.uppercase_ascii app_name ^ "_VERBOSE_HTTP" in
  let env_info = Cmdliner.Cmd.Env.info env_name in
  Arg.(value &amp; flag &amp; info [ "verbose-http" ] ~env:env_info ~doc)
</code></pre>
<p>Each bot can then define its command with minimal boilerplate:</p>
<pre><code class="language-ocaml">let bot_cmd eio_env =
  let info = Cmd.info "echo_bot" ~version:"2.0.0" ~doc ~man in
  let config_term = Zulip_bot.Cmd.config_term "echo-bot" eio_env in
  Cmd.v info Term.(const (fun config -&gt; run_echo_bot config eio_env) $ config_term)
</code></pre>
<p>The source tracking also helps with debugging by showing where each value
originated from -- whether from the command line, environment variables,
XDG config files, or defaults. This makes it much easier to understand
why a bot is behaving a certain way when deployed.</p>
<p><img src="https://anil.recoil.org/images/aoah-zulip-regress-3.webp" alt="%c" title="The manual pages for a bot are pretty good by default thanks to the cmdliner terms."></p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>It's nice to get to the Zulip bot framework at last, since this is one of the
things I wanted to fix at the start of the month. It uses a number of things
I've built this month, including the Requests library to handle HTTP, the INI
codec for Python configuration, XDG to handle path resolution, and so on. Each
piece is small and focused, and generatively replicated from other
(human-written) exemplar libraries from the OCaml ecosystem.</p>
<p>The only "agentic trick" I learnt today was the value of live debugging, as I
found with both <a href="https://anil.recoil.org/notes/aoah-2025-17">JMAP email</a> and <a href="https://anil.recoil.org/notes/aoah-2025-14">Karakeep</a>. Building
services amenable to this kind of live mocking is something I'll keep in mind
for the future. It's also extremely useful to have good terminal manual pages,
since those can also be interrogated by coding agents as well as be used by humans.</p>

