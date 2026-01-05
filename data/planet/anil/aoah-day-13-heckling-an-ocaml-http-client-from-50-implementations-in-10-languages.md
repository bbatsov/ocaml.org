---
title: 'AoAH Day 13: Heckling an OCaml HTTP client from 50 implementations in 10 languages'
description: Agentically synthesising a batteries-included OCaml HTTP client by gathering
  recommendations from fifty open-source implementations across JavaScript, Python,
  Java, Rust, Swift, Haskell, Go, C++, PHP and shell.
url: https://anil.recoil.org/notes/aoah-2025-13
date: 2025-12-13T00:00:00-00:00
preview_image: https://anil.recoil.org/images/awesome-http.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Now I had some <a href="https://anil.recoil.org/notes/aoah-2025-11">prerequisite</a> <a href="https://anil.recoil.org/notes/aoah-2025-12">libraries</a>, I turned my attention to having a batteries-included OCaml HTTP tool with features like request throttling and redirect loop detection. I've hacked on <a href="https://github.com/mirage/ocaml-cohttp">OCaml HTTP protocol</a> libraries since 2011, but these higher level features weren't necessary in things like <a href="https://anil.recoil.org/papers/2025-docker-icfp">Docker's VPNKit</a>. The problem with building one now is that there are <em>loads</em> of random quirks needed in real-world HTTP, which would take ages to figure out if I start from scratch.</p>
<p>Luckily, there's an <a href="https://anil.recoil.org/papers/2025-internet-ecology">entire ecology</a> of HTTP clients built in <em>other</em> languages that could use for inspiration as well! Today, I <strong>gathered <em>fifty</em> open-source HTTP clients from a variety of other language ecosystems, and agentically synthesised a specification <em>across</em> all of them into one OCaml client</strong> using Eio.</p>
<p>I'm not sure what the collective verb is for a group of HTTP clients, so dubbed this whole process a 'heckle' of HTTP coding!</p>
<h2><a href="https://anil.recoil.org/news.xml#condensing-an-http-implementation-from-fifty-other-implementations" class="anchor" aria-hidden="true"></a>Condensing an HTTP implementation from fifty other implementations</h2>
<p>The first thing is to find the heckle of other implementations, which I grabbed from <a href="https://raw.githubusercontent.com/easybase/awesome-http/refs/heads/main/resources.json">this awesome-http</a> repository. I vibed up a <a href="https://tangled.org/anil.recoil.org/ocaml-requests/blob/claude-test/tools/clone_repos.ml">git cloner</a> that fetched the sources to my local dev repo, so the agent could access everything.</p>
<p><a href="https://github.com/easybase/awesome-http"> <img src="https://anil.recoil.org/images/awesome-http.webp" alt="%c" title="To paragraph the Fifth Element: don't want one library, want ALL the libraries!"> </a></p>
<p>Then came the big task of differentially coming up with a spec. I used my earlier <a href="https://anil.recoil.org/notes/aoah-2025-4">Claudeio</a> OCaml library to build a <a href="https://tangled.org/anil.recoil.org/ocaml-requests/blob/claude-test/tools/summarise_recommendations.ml">custom agent</a> that iterates over each of the fifty repos and uses the <a href="https://tangled.org/anil.recoil.org/ocaml-claudeio/blob/main/lib/structured_output.mli">structured JSON output</a> to emit the recommendations using a simple schema. I left this churning for an hour while I went for my morning run, and came back to records like this:</p>
<pre><code class="language-json">"priority_features": [ {
  "priority_rank": 1,
  "category": "Security &amp; Spec Compliance",
  "title": "Strip sensitive headers on cross-origin redirects",
  "description": "When redirecting to a different origin (host/port/scheme),
    automatically strip Authorization, Cookie, Proxy-Authorization, and
    WWW-Authenticate headers to prevent credential leakage to
    unintended domains. Also strip headers on HTTPS-&gt;HTTP protocol downgrade.",
  "rfc_references": [ "RFC 9110 Section 15.4 (Redirection)" ],
  "source_libraries": [
    "reqwest", "got", "node-fetch", "okhttp", "axios", "superagent",
    "http-client", "needle" ],
  "affected_files": [ "lib/requests.ml", "lib/one.ml", "lib/http_client.ml" ],
  "implementation_notes": "Implement same_origin check comparing scheme, host,
   and port. Create a list of sensitive headers to strip. Call
   strip_sensitive_headers before following any redirect where origin changes.",
  "cross_language_consensus": 8 },
</code></pre>
<p>There were <a href="https://tangled.org/anil.recoil.org/ocaml-requests/commit/dc03b221b36a1c53d853e628cc3746bbf370e3d4">hundreds of recommendations</a>, ranked by their class (security, new or missing features, etc), but also how many other language ecosystems viewed
this as important along with a list of specific libraries. Each recommendation also included
implementation notes specific to OCaml, since the agent had been instructed to differentially
compare with the strawman OCaml implementation.</p>
<p>At this point, I did a manual scan and dropped a few language ecosystems that weren't immediately useful.
For example, I'd added in a few <a href="https://github.com/algebraic-dev/http">Lean HTTP clients</a> after
reading <a href="https://martin.kleppmann.com">Martin Kleppmann</a> predict that <a href="https://martin.kleppmann.com/2025/12/08/ai-formal-verification.html">AI will make formal verification go mainstream</a>. However, the Lean clients weren't quite feature complete enough yet, and also don't seem to include any actual verification (it's using Lean as a conventional programming language) so just reverting to Haskell as the reference there seemed fine.</p>
<p>The agent came back with a nice summary when asked to focus on just Python, Rust and Lean and helped confirm my hypothesis:</p>
<blockquote>
<p>Key gaps compared to Python requests and Rust reqwest:</p>
<div role="region"><table>
<tbody><tr>
<th>Gap</th>
<th>Impact</th>
</tr>
<tr>
<td>No proxy support</td>
<td>Blocks enterprise/corporate adoption</td>
</tr>
<tr>
<td>No compression</td>
<td>Larger downloads, slower performance</td>
</tr>
<tr>
<td>No Response.json()</td>
<td>Extra boilerplate for common use case</td>
</tr>
<tr>
<td>No auth stripping on redirects</td>
<td>Potential credential leakage</td>
</tr>
<tr>
<td>No custom CA certs</td>
<td>Can't use with self-signed/internal PK</td>
</tr>
<tr>
<td>No client certs (mTLS)</td>
<td>Missing enterprise auth scenario</td>
</tr>
</tbody></table></div><p>The Lean HTTP library is interesting for its strong typing but is
server-focused and lacks many client features. It's not a good model for your
"batteries included" goal.</p>
</blockquote>
<p>The full list of projects I ended up using was pretty epic: JavaScript (<a href="https://github.com/axios/axios">Axios</a>, <a href="https://github.com/node-fetch/node-fetch">node-fetch</a>, <a href="https://github.com/sindresorhus/got">Got</a>, <a href="https://github.com/visionmedia/superagent">superagent</a>, <a href="https://github.com/tomas/needle">Needle</a>),
Python (<a href="https://github.com/psf/requests">Requests</a>, <a href="https://github.com/urllib3/urllib3">urllib3</a>, <a href="https://github.com/httplib2/httplib2">httplib2</a>, <a href="https://github.com/spyoungtech/grequests">GRequests</a>, <a href="https://github.com/prkumar/uplink">Uplink</a>), Java (<a href="https://github.com/eclipse/jetty.project">Eclipse Jetty</a>, <a href="https://github.com/square/okhttp">OkHttp</a>, <a href="https://github.com/internetarchive/heritrix3">Heritrix</a>, <a href="https://github.com/apache/httpcomponents-client">Apache HttpClient</a>, <a href="https://github.com/googleapis/google-http-java-client">Google HTTP Client</a>, <a href="https://github.com/kevinsawicki/http-request">Http Request</a>), Rust (<a href="https://github.com/seanmonstar/reqwest">reqwest</a>, <a href="https://github.com/hyperium/hyper">hyper</a>, <a href="https://github.com/sagebind/isahc">Isahc</a>, <a href="https://github.com/http-rs/surf">Surf</a>, <a href="https://github.com/alexcrichton/curl-rust">curl-rust</a>), Swift (<a href="https://github.com/Alamofire/Alamofire">Alamofire</a>, <a href="https://github.com/daltoniam/SwiftHTTP">SwiftHTTP</a>, <a href="https://github.com/nghialv/Net">Net</a>, <a href="https://github.com/Moya/Moya">Moya</a>, <a href="https://github.com/dduan/Just">Just</a>, <a href="https://github.com/onevcat/Kingfisher">Kingfisher</a>), Haskell (<a href="https://github.com/mrkkrp/req">Req</a>, <a href="https://github.com/snoyberg/http-client">http-client</a>, <a href="https://github.com/haskell-servant/servant">servant-client</a>, <a href="https://github.com/aesiniath/http-streams">http-streams</a>), Go (<a href="https://github.com/imroc/req">Req</a>, <a href="https://github.com/go-resty/resty">Resty</a>, <a href="https://github.com/dghubble/sling">Sling</a>, <a href="https://github.com/asmcos/requests">requests</a>), C++ (<a href="https://github.com/apache/serf">Apache Serf</a>, <a href="https://github.com/libcpr/cpr">cpr</a>, <a href="https://github.com/cpp-netlib/cpp-netlib">cpp-netlib</a>, <a href="https://github.com/sprinfall/webcc">Webcc</a>, <a href="https://github.com/facebook/proxygen">Proxygen</a>, <a href="https://github.com/yhirose/cpp-httplib">cpp-httplib</a>, <a href="https://github.com/spotify/NFHTTP">NFHTTP</a>, <a href="https://github.com/sony/easyhttpcpp">EasyHttp</a>), PHP (<a href="https://github.com/guzzle/guzzle">Guzzle</a>, <a href="https://github.com/php-http/httplug">HTTPlug</a>, <a href="https://github.com/amphp/http-client">HTTP Client</a>, <a href="https://github.com/sendgrid/php-http-client">SendGrid HTTP Client</a>, <a href="https://github.com/kriswallsmith/Buzz">Buzz</a>), Shell/C (<a href="https://github.com/httpie/httpie">HTTPie</a>, <a href="https://github.com/curl/curl">curl</a>, <a href="https://github.com/aria2/aria2">aria2</a>, <a href="https://github.com/httpie/http-prompt">HTTP Prompt</a>, <a href="https://github.com/micha/resty">Resty</a>, <a href="https://github.com/jonaslu/ain">Ain</a>). Thank you to all the authors for publishing your respective open-source code!</p>
<h3><a href="https://anil.recoil.org/news.xml#crunching-together-thousands-of-recommendations-into-a-spec" class="anchor" aria-hidden="true"></a>Crunching together thousands of recommendations into a spec</h3>
<p>The second phase was to build a <a href="https://tangled.org/anil.recoil.org/ocaml-requests/blob/claude-test/tools/summarise_recommendations.ml#L213">summarisation agent</a> that took the <a href="https://tangled.org/anil.recoil.org/ocaml-requests/commit/dc03b221b36a1c53d853e628cc3746bbf370e3d4">hundreds of recommendations</a> and crunched them up into a unified priority list. Rather than do this by hand, it's very convenient to let the agent tool use take care of scanning the JSON files, backed up by some forceful prompting to make sure it covers them all (there is no guarantee it will, but hey, this is AI). I also used my <a href="https://tangled.org/anil.recoil.org/claude-ocaml-internet-rfc">Claude Internet RFC skill</a> to fetch the relevant specifications so they could also be cross linked from the recommendations.</p>
<p>The summariser crunched through them in about a minute using the latest Claude Opus 4.5 model. The <a href="https://tangled.org/anil.recoil.org/ocaml-requests/commit/46235dac9d4c575e2546751757e5a779870a61ce">resulting recommendations</a> was a very clean list of features in priority order. Here's an example of another security feature I'd never considered before:</p>
<blockquote>
<p>Implement Header Injection Prevention (Newline Validation)</p>
<p>Validate that user-provided header names and values do not contain newlines
(CR/LF) which could enable HTTP request smuggling attacks. Reject headers
containing these characters with a clear error message.</p>
<ul>
<li><strong>RFC References:</strong>
<ul>
<li>RFC 9110 Section 5.5 (Field Syntax)</li>
<li>RFC 9112 Section 2.2 (Message Format)</li>
</ul>
</li>
<li><strong>Cross-Language Consensus:</strong> 5 libraries</li>
<li><strong>Source Libraries:</strong> haskell/http-client, rust/hyper, php/guzzle, java/okhttp, go/req</li>
<li><strong>Affected Files:</strong> <code>lib/headers.ml</code> <code>lib/http_client.ml</code> <code>lib/error.ml</code></li>
<li><strong>Implementation Notes:</strong> Add validation in Headers.set/Headers.add that
rejects header names/values containing <code>\r</code> or <code>\n</code> characters. Add InvalidHeader
error variant with the offending header name for debugging.</li>
</ul>
</blockquote>
<p>After this, the rest was 'normal' agentic coding, whereby I put the agent in a loop
building the recommendations in order, regularly using my <a href="https://tangled.org/anil.recoil.org/claude-ocaml-tidy-code">ocaml-tidy-code</a>
to clean up the generated mess (somewhat), and afterwards running it through a module refactoring.</p>
<p><img src="https://anil.recoil.org/images/aoah-heckle-ss-1.webp" alt="%c" title="The TODO lists for the agent coding OCaml came out of fifty other libraries' features"></p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>The resulting <a href="https://tangled.org/anil.recoil.org/ocaml-requests">ocaml-requests library</a> does work, but I feel this is the first time I've lost the thread on the exact architecture of the library, so this will take some careful code review.</p>
<p>The basic library fulfills its original mandate fairly well; there is a simple high level interface that's direct style:</p>
<pre><code class="language-ocaml">Eio_main.run @@ fun env -&gt;
Switch.run @@ fun sw -&gt;
let req = Requests.create ~sw env in
Requests.set_auth req (Requests.Auth.bearer "your-token");
let user, repos = Eio.Fiber.both
  (fun () -&gt; Requests.get req "https://api.github.com/user")
  (fun () -&gt; Requests.get req "https://api.github.com/user/repos") in
let user_data = Response.body user |&gt; Eio.Flow.read_all in
let repos_data = Response.body repos |&gt; Eio.Flow.read_all in
...
</code></pre>
<p>This is straightforward, and there is also a stateless <code>Requests.One</code> module that contains the one-shot equivalents. There's also an <code>ocurl</code> binary that exercises it all via the CLI. For example, here's ocurl downloading a <a href="https://anil.recoil.org/papers/2024-food-life">recent paper</a> and following redirect chains:</p>
<pre><code>&gt; dune exec -- bin/ocurl.exe https://doi.org/10.1038/s43016-025-01224-w -Iv
ocurl: Creating new connection pool (max_per_endpoint=10, max_idle=60.0s, max_lifetime=300.0s)
ocurl: Created Requests session with connection pools (max_per_host=10, TLS=true)
GET https://doi.org/10.1038/s43016-025-01224-w
ocurl: &gt; GET https://doi.org/10.1038/s43016-025-01224-w HTTP/1.1
ocurl: &gt; Request Headers:
ocurl: &gt;   Accept-Encoding: gzip, deflate
ocurl: &gt;   User-Agent: ocaml-requests/0.1.0 (OCaml 5.4.0)
ocurl: Creating endpoint pool for doi.org:443 (max_connections=10)
ocurl: TLS connection established to doi.org:443
ocurl: &lt; HTTP/1.1 302
ocurl: &lt; Response Headers: &lt;trimmed&gt;
ocurl: &lt;   date: Sun, 14 Dec 2025 14:22:18 GMT
ocurl: &lt;   expires: Sun, 14 Dec 2025 14:48:37 GMT
ocurl: &lt;   location: https://www.nature.com/articles/s43016-025-01224-w
ocurl: 
ocurl: Following redirect (10 remaining)
ocurl: 
ocurl: Request to https://www.nature.com/articles/s43016-025-01224-w ===
ocurl: &gt; GET https://www.nature.com/articles/s43016-025-01224-w HTTP/1.1
ocurl: &gt; Request Headers:
ocurl: &gt;   Accept-Encoding: gzip, deflate
ocurl: &gt;   User-Agent: ocaml-requests/0.1.0 (OCaml 5.4.0)
ocurl: &gt; &lt;trimmed&gt;
ocurl: Following redirect (9 remaining)
ocurl: 
ocurl: &gt; GET https://idp.nature.com/authorize?response_type=cookie&amp;client_id=grover&amp;redirect_uri=https://www.nature.com/articles/s43016-025-01224-w HTTP/1.1
ocurl: &gt; Request Headers:
ocurl: &gt;   Accept-Encoding: gzip, deflate
ocurl: &gt;   User-Agent: ocaml-requests/0.1.0 (OCaml 5.4.0)
ocurl: 
ocurl: Following redirect (8 remaining)
ocurl: &gt; GET https://idp.nature.com/transit?redirect_uri=https://www.nature.com/articles/s43016-025-01224-w&amp;code=99850362-a71b-426f-a325-887d4ebc2346 HTTP/1.1
ocurl: &gt; Request Headers:
ocurl: &gt;   Accept-Encoding: gzip, deflate
ocurl: &gt;   Cookie: idp_session=sVERSION_1fdd8b701-8a14-4c8d-b31f-01af7f3653a2; idp_session_http=hVERSION_149b210b2-c08c-40b1-949f-4125872fff82; idp_marker=4792551b-f6f8-4b89-a02b-9032ea177821
ocurl: &gt;   User-Agent: ocaml-requests/0.1.0 (OCaml 5.4.0)
ocurl: 
ocurl: Following redirect (7 remaining)
ocurl: 
ocurl: Request completed in 0.972 seconds
ocurl: Request completed with status 200
</code></pre>
<p>Quite verbose, but you can see there's a lot going on with a seemingly simple HTTP request beyond just one request!</p>
<h3><a href="https://anil.recoil.org/news.xml#better-cram-tests-with-httpbin" class="anchor" aria-hidden="true"></a>Better cram tests with httpbin</h3>
<p>Where it gets a little messier is the internals, which were built as a result
of iterative agentic work. I don't entirely have confidence (beyond peering at
the code) that all those edge cases identified from 50 other libraries are all
tested in my implementation.</p>
<p>To mitigate this, I put together a <a href="https://httpbin.org">httpbin</a> set of <a href="https://tangled.org/anil.recoil.org/ocaml-requests/blob/claude-test/test/httpbin.t">ocaml-requests cram tests</a> which use an ocurl binary (which models upstream curl, but using this OCaml code) in order to run
a better of command line tests. These do things like:</p>
<pre><code class="language-sh">Test cookie setting endpoint:

  $ ocurl --verbosity=error "$HTTPBIN_URL/cookies/set?session=abc123" | \
  &gt;   grep -o '"session": "abc123"'
  "session": "abc123"

Test setting multiple cookies:

  $ ocurl --verbosity=error "$HTTPBIN_URL/cookies/set?session=abc123&amp;user=testuser" | \
  &gt;   grep '"cookies"' -A 4
    "cookies": {
      "session": "abc123",
      "user": "testuser"
    }
  }
</code></pre>
<p>While httpbin is extremely convenient to providing a local endpoint that understands HTTP, this still doesn't cover the full battery of tests. Ideally in the future, we should be able to <em>use the 50 test suites</em> from other language's libraries somehow. I haven't quite thought this through, but it would be extremely powerful if we could bridge OCaml over to a wider set of test suites.</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>I've accomplished what I set out to do at the start of the day, but I feel like
I've gone further off the 'conventional coding' path than ever before. On one
hand, it's incredible to have churned through <em>fifty</em> other libraries in a
matter of hours. On the other hand, I have almost no intuition about the
detailed structure of my resulting library without going through it
line-by-line.</p>
<p>There are many important details like the intermingling of connection handling,
exceptions and resource cleanups that are likely shaky. A lot of common issues
will be taken care of by the fact that <a href="https://ocaml-multicore.github.io/eio/eio/Eio/Switch/index.html">Eio Switches</a>
handle resource cleanup very well, but it's unlikely to be 100%. This definitely requires
human attention, which I'll do over the next few months.</p>
<p>However, I've also got very little <em>emotional</em> attachment to the structure of the
new Requests library, unlike other libraries I've written by hand -- it's so
easy to refactor with a bit more agentic coding! <a href="https://asbradbury.org">Alex Bradbury</a>
<a href="https://www.linkedin.com/feed/update/urn:li:activity:7404566293862887425?commentUrn=urn:li:comment:(activity:7404566293862887425%2C7404578370950307840)&amp;dashCommentUrn=urn:li:fsd_comment:(7404578370950307840%2Curn:li:activity:7404566293862887425)">asked</a> me if I'm enjoying this whole process, and today I realised that I <em>am</em>. If I
viewed this as conventional coding, then I would hate it. But agentic coding is
extraordinarily different from conventional coding, much more akin to top-down
specification and <a href="https://doi.org/10.1007/11691372_34">counterexample driven abstract refinement</a>
using natural language!</p>
<p>I'm doing a lot of <a href="https://anil.recoil.org/news.xml">machine learning research</a> elsewhere, and this agentic
coding adventure reminds me of a wonderful quote from <a href="https://en.wikiquote.org/wiki/Emma_Goldman">Emma Goldman</a>
that I heard while listening to <a href="https://www.bbc.co.uk/sounds/play/m002n7rf">3rd Reith Lecture</a> today:</p>
<blockquote>
<p>"If can't dance, it's not my revolution!"
<cite>-- <a href="https://en.wikiquote.org/wiki/Emma_Goldman">Emma Goldman</a>, 1931</cite></p>
</blockquote>
<p>It's important to find joy in whatever we're working on, and I'm glad I am enjoying myself
here, even though I remain highly uncertain about how to integrate this with the open source
practises I've experienced in the past three decades.
I'm going to forge ahead with using Requests for a while on some real-world
APIs that I need access to, and see how it goes. But this is definitely not
ready for wider use just yet!</p>
<p>In <a href="https://anil.recoil.org/notes/aoah-2025-14">Day 14</a> we'll take a look at using Requests to build OCaml
interfaces to some of the <a href="https://anil.recoil.org/news.xml">##selfhosted</a> services I use.</p>

