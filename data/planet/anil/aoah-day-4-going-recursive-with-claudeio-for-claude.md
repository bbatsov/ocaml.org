---
title: 'AoAH Day 4: Going recursive with Claudeio for Claude'
description: Creating OCaml bindings for the Claude API using Eio and jsont codecs
  by reverse-engineering the JSON-RPC protocol from Python and Go SDKs, enabling Claude
  to write more Claude-powered OCaml code.
url: https://anil.recoil.org/notes/aoah-2025-4
date: 2025-12-04T00:00:00-00:00
preview_image: https://anil.recoil.org/images/claude-ss-perm-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>By this point, I've got three useful libraries and my use of Claude is getting better. So naturally I want to automate my invocations of the <code>claude</code> CLI, but I hit a roadblock: there are no OCaml SDK bindings! However, there appear to be SDKs in <a href="https://github.com/anthropics/claude-agent-sdk-python">Python</a>, <a href="https://github.com/anthropics/anthropic-sdk-go">Go</a> and <a href="https://github.com/anthropics">many others</a>. So today will involve having a stab at generating <a href="https://tangled.org/anil.recoil.org/claudeio">Claude OCaml bindings</a> using Eio, so I can use Claude to write more OCaml!</p>
<h2><a href="https://anil.recoil.org/news.xml#approach" class="anchor" aria-hidden="true"></a>Approach</h2>
<p>I prodded around the <a href="https://github.com/anthropics/claude-agent-sdk-python">Python</a> and noted that the communications protocol between the SDK and the CLI is JSON-RPC. I'd noticed <a href="https://anil.recoil.org/notes/aoah-2025-2">when hacking with jsont</a> that it includes a <a href="https://github.com/dbuenzli/jsont/blob/main/test/json_rpc.ml">json-rpc codec</a>, so adopting the same approach as I did with <a href="https://tangled.org/anil.recoil.org/ocaml-jsonfeed">ocaml-jsonfeed</a> seems reasonable: code up the core protocol using JSONt codecs, and then handle serialisation and process coordination using Eio.</p>
<p>For context to the agent, I gave it the Python and Go Claude SDKs to digest what the actual Claude protocol involves, and then all my previous OCaml libraries and the sources to jsont and Eio (i.e. the lessons learnt from the previous couple of days with xdge and jsonfeed).</p>
<p>One important prompt was to instruct it to <em>first</em> generate a <code>claude.proto</code> subpackage that <em>only</em> has jsont codecs and OCaml types, and then to use that package in the coordination layer with Eio. This avoids mixing up concerns in one giant module, as an unprompted Claude is prone to do.</p>
<h2><a href="https://anil.recoil.org/news.xml#tests" class="anchor" aria-hidden="true"></a>Tests</h2>
<p>Using jsont at the codec layer made all the difference, since I could get the model to debug the wire-level messages independently of the transport layer.  In fact, I left the agent running in a loop where it looked at the error outputs from its own regression tests (against a live Claude instance) and then proceeded to fix them. This was only possible because of the excellent error instructions from jsont. For example, with the structured output test I got:</p>
<pre><code>structured_output_demo.exe: [ERROR] Failed to decode incoming message: Missing member tool_name in Rule object
File "-", line 1, characters 451-515:
File "-", line 1, characters 451-515: at index 0 of
File "-", line 1, characters 450-515: array&lt;Rule object&gt;
File "-": in member rules of
File "-", line 1, characters 423-515: Update object
File "-", line 1, characters 423-515: at index 0 of
File "-", line 1, characters 422-515: array&lt;Update object&gt;
File "-": in member permission_suggestions of
File "-", line 1, characters 88-515: Permission object
File "-": in member request of
File "-", line 1, characters 0-515: ControlRequest object
Line: {"type":"control_request","request_id":"055cb59c-2f9f-457d-8c98-0a2c5a48c577","request":{"subtype":"can_use_tool","tool_name":"Bash","input":{"command":"find /Users/avsm/src/git/knot -type f -name \"*.ml\" -o -name \"*.mli\" -o -name \"*.md\" -o -name \"*.html\" -o -name \"*.go\" -o -name \"dune\" -o -name \"dune-project\" | head -100","description":"List all relevant files in the repository"},"permission_suggestions":[{"type":"addRules","rules":[{"toolName":"Read","ruleContent":"//Users/avsm/src/git/knot/**"}],"behavior":"allow","destination":"session"}],"tool_use_id":"toolu_011w6XYAbALBytxMaLxtnBGd","agent_id":"05bf9384-6c4b-4edd-898f-962d945ff724"}}
</code></pre>
<p>This was enough information for Claude to pick up the problem and address it in the codec:</p>
<pre><code>Claude: I found the issue! The Rule decoder in proto/permissions.ml is expecting
snake_case field names (tool_name, rule_content) but the Claude CLI is sending
camelCase field names (toolName, ruleContent). The rest of the permission
system already uses camelCase consistently.
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#results" class="anchor" aria-hidden="true"></a>Results</h2>
<p>I did find a <em>lot</em> of breakage when using different versions of the upstream
Claude SDKs. For example, permission handling is just...broken... in some
versions, but they seem to quite quickly push changes. I suspect they might be
using a bit too much bleeding edge Claude in developing Claude!</p>
<p><img src="https://anil.recoil.org/images/claude-ss-perm-1.webp" alt="%c" title="I managed to get interactive OCaml callbacks to Claude working after some upstream fixes"></p>
<p>However, their breakage did exercise my agent quite nicely into switching
between the Python and Go SDKs to come up with a good answer, and also
highlighted why agentic coding is so different from one-shot coding LLMs. It's
pretty crazy seeing an agent dynamically introspect itself to come up with the
architecture I specified, against a live service!</p>
<h2><a href="https://anil.recoil.org/news.xml#reflection" class="anchor" aria-hidden="true"></a>Reflection</h2>
<p>It's now quite convenient to have a <a href="https://tangled.org/anil.recoil.org/claudeio">Claude OCaml wrapper</a>, but I stopped short of making a really nice Eio interface as the upstream project is moving so quickly.</p>
<p>I'd like to eventually use this as a basis for a distributed Claude to unify my local and remote Docker development, and also integrate with our local initiatives like <a href="https://ryan.freumh.org">Ryan Gibb</a> and his work on <a href="https://anil.recoil.org/papers/2025-hyperres">package management</a> and <a href="https://patrick.sirref.org">Patrick Ferris</a>'s cool new <a href="https://patrick.sirref.org/weekly-2025-w49/index.xml">Shelter</a>.  For now, I'm holding the fort on just doing simple OCaml invocations of Claude and not trying anything too exotic until the CLI itself settles down and stabilises.</p>
<p>Onto <a href="https://anil.recoil.org/notes/aoah-2025-5">Day 5</a> next, where we use Claude skills for the first time!</p>

