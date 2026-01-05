---
title: 'AoAH Day 12: Eio Connection pooling and event tracing'
description: Building a TCP/TLS connection pooling library for Eio with DNS-based
  load balancing, stacked error handling, and self-contained HTML visualisations for
  stress test results.
url: https://anil.recoil.org/notes/aoah-2025-12
date: 2025-12-12T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-vis-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>After yesterday's <a href="https://anil.recoil.org/notes/aoah-2025-11">library bonanza</a> for HTTP cookie handling, I implemented a TCP/TLS <a href="https://tangled.org/anil.recoil.org/ocaml-conpool">connection pooling library</a>. This is useful for an HTTP client as it provides the network-level mechanisms for keeping track of outgoing network connections <em>by their DNS name</em>.  This allows for more flexible outgoing connection management without worrying about overloading remote endpoints.</p>
<p>For example, <code>github.io</code> has four A records:</p>
<pre><code>&gt; host github.io
github.io has address 185.199.110.153
github.io has address 185.199.109.153
github.io has address 185.199.108.153
github.io has address 185.199.111.153
</code></pre>
<p>With this new connection pooling library, my application should be able to
connect to the <code>github.io</code> name and keep track of all the outgoing connections
on the basis of it being called <code>github.io</code> and load balance the number of
outgoing connections accordingly.</p>
<p>In the interests of exploring something new, I also decided to add in visualisation
support to figure out what the library is spending its time on.
I decided to generate <a href="https://jon.recoil.org/blog/2025/12/an-svg-is-all-you-need.html">self-contained visualisations</a>,
inspired by <a href="https://jon.recoil.org">Jon Ludlam</a> rediscovering the joy of SVGs yesterday!</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>The core library itself is pretty straightforward, as it's reasonably similar
to the <a href="https://github.com/mirage/ocaml-conduit">ocaml-conduit</a> mechanism of
providing a name-based resolver. The interface should be a matter of creating a
connection pool to keep track of state and then requesting a connection from it
to a hostname:</p>
<pre><code>module Endpoint: sig
 type t
 val make : host:string -&gt; port:int -&gt; t
end

type connection_ty = [Eio.Resource.close_ty | Eio.Flow.two_way_ty]                                                                                            
type connection = connection_ty Eio.Resource.t
val connection : sw:Eio.Switch.t -&gt; t -&gt; Endpoint.t -&gt; connection
</code></pre>
<p>This uses Eio's <a href="https://github.com/ocaml-multicore/eio?tab=readme-ov-file#provider-interfaces">resource mechanism</a> to allow this connection to be used like any other.</p>
<h3><a href="https://anil.recoil.org/news.xml#stacking-io-errors" class="anchor" aria-hidden="true"></a>Stacking IO errors</h3>
<p>One of the coolest things about Eio's error handling is the ability to <a href="https://github.com/ocaml-multicore/eio?tab=readme-ov-file#provider-interfaces">stack errors</a> by re-raising exceptions and adding more context to it.  I prompted the agent to also create connection-specific errors for Eio, but to integrate them into the <a href="https://ocaml.org/p/eio/1.3/doc/eio/Eio/index.html#exception-Io">Eio.Io extensible type</a>. This allows errors from failures to look like this:</p>
<pre><code>[WARNING] Connection attempt 3 to localhost:8088 failed:
  Eio.Io Net Connection_failure Refused Unix_error
  (Connection refused, "connect-in-progress", ""),
   connecting to tcp:127.0.0.1:8088,
   connecting to localhost:8088,
   after 3 retry attempts
</code></pre>
<p>The conpool library can keep a lightweight reporting stack of what it's been
doing while propagating the error, which makes a big different to the quality
of the end logging.</p>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<h3><a href="https://anil.recoil.org/news.xml#self-contained-visualisations-work-well" class="anchor" aria-hidden="true"></a>Self-contained visualisations work well</h3>
<p>I activated <a href="https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design">Claude Marketplace's front end module</a>
and instructed it to generate me a self-contained HTML file with the results of
the output of the stress test.</p>
<p>This...just worked. Here's a <a href="https://www.cl.cam.ac.uk/~avsm2/conpool-stress.html">HTML
snapshot</a> of the results
of a conpool run, with all the various configurations tests pushed together
into a visualisation. I've also seen <a href="https://toao.com">Sadiq Jaffer</a> do the same thing when iterating
on <a href="https://anil.recoil.org/papers/2025-tessera">TESSERA</a> CNN visualisations for his intermediate runs.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/conpool-stress.html"> <img src="https://anil.recoil.org/images/aoah-vis-ss-1.webp" alt="%c" title="Standalone visualiation of the connection pool stress tests on localhost"> </a></p>
<h3><a href="https://anil.recoil.org/news.xml#a-negative-result-with-event-tracing" class="anchor" aria-hidden="true"></a>A negative result with event tracing</h3>
<p>I figured I'd have a go at integrating at <a href="https://ocaml.org/manual/5.2/api/Runtime_events.html">event tracing</a>, but the various
packaging problems around the tools defeated my quick attempt. There's
<a href="https://github.com/ocaml-multicore/meio">meio</a> which I really want to use, but
it requires various upstream PRs merging, and I'll also need to figure out what
to do if the ring buffer overflows. I'll have to come back to this later; for
now the self-contained visualisations are fine.</p>
<h3><a href="https://anil.recoil.org/news.xml#another-claude-skill-for-tidying-ocaml-code" class="anchor" aria-hidden="true"></a>Another Claude skill for tidying OCaml code</h3>
<p>I found myself doing the same prompting repeatedly to tidy up generated code to
make it more idiomatic, so I had Claude go back over my history to compact my
instructions into a reusable skill, which I uploaded to
<a href="https://tangled.org/anil.recoil.org/claude-ocaml-tidy-code">ocaml-tidy-code</a>.
Running this over all the code is good to do <em>after</em> test cases have been
generated, as then it's fairly easy to verify that things are working well.</p>
<p><img src="https://anil.recoil.org/images/aoah-cleanup-ss-1.webp" alt="%c" title="The cleanup agent does a reasonable job of pulling out reusable functions after several generation passes"></p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>I find the Eio types to be quite complex after we stripped out objects, and so
having an agent code up the idiomatic boilerplate required for registering Eio
exceptions, pretty printers and resources was quite nice. It's also good to
have another Claude skill to further refine my workflow for cleaning up code.</p>
<p>The self-contained visualisations are also extremely useful; in addition to the
standalone HTML one, I also vibed up a full event logging system using the
<a href="https://ocaml.org/manual/5.2/api/Runtime_events.html">Runtime_events</a> that
<a href="https://toao.com">Sadiq Jaffer</a> implemented. I decided not to go with that due to some difficulties
with dealing with full ring buffers, but I'll come back to in the future.
There's clearly a need for a library that can register library-level events and
dispatch them to the OCaml custom events buffer, to remote logging endpoints,
and to clients like <a href="https://anil.recoil.org/notes/aoah-2025-9">terminal UIs</a> or web monitors.</p>
<p>Tomorrow, we'll pull this all together into a <a href="https://anil.recoil.org/notes/aoah-2025-13">Day 13 HTTP client</a>!</p>

