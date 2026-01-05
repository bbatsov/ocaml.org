---
title: 'AoAH Day 21: Complete dynamic HTML5 validation in OCaml and the browser'
description: Porting the W3C's Nu HTML Validator from Java to OCaml and running in
  the browser dynamically
url: https://anil.recoil.org/notes/aoah-2025-21
date: 2025-12-21T00:00:00-00:00
preview_image: https://anil.recoil.org/images/aoah-html5v-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>With <a href="https://anil.recoil.org/notes/aoah-2025-20">language detection</a> now working in OCaml, I completed vibespiling the <a href="https://validator.github.io/validator/">Nu HTML Validator</a> from Java to OCaml. This is the official <a href="https://validator.w3.org">W3C validator</a> used to check HTML5 conformance, and it's a substantial codebase with thousands of validation rules.  I set out to see what a few <em>days</em> of agentic processing would do to transform the complex Java codebase into a more functionally structured pure OCaml codebase.</p>
<p>The result is a pure OCaml HTML5 conformance checker that integrates with the <a href="https://anil.recoil.org/notes/aoah-2025-15">parser</a> I built last week, all published as <strong><a href="https://tangled.org/anil.recoil.org/ocaml-html5rw">ocaml-html5rw</a></strong>. Having the logic in pure OCaml meant that I could <em>also</em> <a href="https://anil.recoil.org/notes/aoah-2025-20">compile it</a> into standalone JavaScript and WASM.  Dynamic conformance checking works even better than server-side filtering since live JavaScript executing on the page (and modifying the DOM) can <em>also</em> be checked. I <a href="https://www.npmjs.com/package/html5rw-jsoo">published</a> this to NPM using a <a href="https://tangled.org/anil.recoil.org/claude-ocaml-to-npm/blob/main/SKILL.md">new Claude skill</a>, and coded a live panel overlay to live debug HTML5 issues that I use on my own website now.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/html5rw-validate/"> <img src="https://anil.recoil.org/images/aoah-html5v-ss-1.webp" alt="%c" title="My conformance checker now runs the OCaml straight in the browser on my dev website and highlights errors along with explanations."> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#full-html5-validation-in-ocamljavascriptwasm" class="anchor" aria-hidden="true"></a>Full HTML5 validation in OCaml/JavaScript/WASM</h2>
<p>I'm going to talk about my results in reverse today, since I thought the outcomes were so unexpectedly useful.  I took yesterday's <a href="https://anil.recoil.org/notes/aoah-2025-20">session</a> and asked Claude to build an <a href="https://tangled.org/anil.recoil.org/claude-ocaml-to-npm/blob/main/SKILL.md">ocaml-to-npm</a> skill, which I used to publish the html5rw JavaScript and wasm <a href="https://www.npmjs.com/package/html5rw-jsoo">to npm</a>.</p>
<p>This runs the HTML5 validation OCaml code by serialising the live DOM tree and then collecting the various validation errors <em>along with the source</em>. This is sufficient to populate an overlay panel that can not only list the errors, but also highlighting the offending DOM node with a red box. This spotted lots of dynamic errors in my website!</p>
<p>Publishing on NPM is quite convenient as there are several CDNs that serve the JavaScript directly. I integrate this into the development version of my blog as simply as:</p>
<pre><code>&lt;script src="https://cdn.jsdelivr.net/npm/html5rw-jsoo@1.0.0/htmlrw.js"&gt;&lt;/script&gt;
&lt;script&gt;
function validateWithPanel() {
  const result = html5rw.validateAndShowPanel(document.documentElement, {
    // Annotation options
    annotation: {
      addDataAttrs: true,
      addClasses: true,
      showTooltips: true,
      tooltipPosition: 'auto',
      highlightOnHover: true
    },
    // Panel options
    panel: {
      initialPosition: 'topRight',
      draggable: true,
      collapsible: true,
      groupBySeverity: true,
      clickToHighlight: true,
      showSelectorPath: true,
      theme: 'auto'
    }
});
&lt;/script&gt;
</code></pre>
<p>I'll probably wrap this in a <a href="https://webcomponents.org">webcomponent</a> in the
future as <a href="https://github.com/art-w">Arthur Wendling</a> did with <a href="https://github.com/art-w/x-ocaml">x-ocaml</a> but for
now this is already useful on my own website. If anyone has any pointers for
what the right CSS patterns are for adding these debug overlay panels to
websites with minimal intrusion, I'd be grateful. I'm extremely unfamiliar with how
modern frontend programming works...</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/html5rw-validate/"> <img src="https://anil.recoil.org/images/aoah-html5v-ss-2.webp" alt="%c" title="Did you know that you're not really supposed to have more than one h1 tag? Neither did I..."> </a></p>
<p>And of course, if you do prefer to stick to the server-side, then you get fast native code OCaml via a command-line binary provided by the package:</p>
<pre><code class="language-bash">$ dune exec -- html5check test.html
test.html:126.73: error [no-p-element-in-scope]: No “p” element in scope but a
“p” end tag seen.
test.html:113.72: error [missing-alt]: An “img” element must have an “alt”
attribute, except under certain conditions. For details, consult guidance on
providing text alternatives for images.
test.html:120.27: error [duplicate-id]: Duplicate ID “duplicate-id”.
test.html:123.36: error [disallowed-child]: Element “div” not allowed as child
of element “span” in this context. (Suppressing further errors from this
subtree.)
test.html:152.8: info [multiple-h1]: Consider using only one “h1” element per
document (or, if using “h1” elements multiple times is required, consider using
the “headingoffset” attribute to indicate that these “h1” elements are not all
top-level headings).
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#a-few-days-of-vibespiling" class="anchor" aria-hidden="true"></a>A few days of vibespiling</h2>
<p>The reason this took a few days of background vibespiling comes down to the sheer size of the problem. The <a href="https://github.com/validator/validator">Nu Validator</a> is a mature Java application that is built around Java's <a href="https://docs.oracle.com/javase/tutorial/jaxp/sax/parsing.html">SAX event model</a>, which I last used in 2000 when I worked on <a href="https://en.wikipedia.org/wiki/Chello">Chello's website</a>. Looking through the validator Java code brought back "fond" memories of building <a href="https://getyarn.io/yarn-clip/796493b5-d8f6-42fa-9252-3d3803379653">factories of factory makers</a>.  In the Nu validators, there are lots of rules checkers that iterate through an HTML5 parse tree and extend a base <code>Checker</code> class:</p>
<pre><code>public final class TableChecker extends Checker {
private Table current;
private final LinkedList&lt;Table&gt; stack = new LinkedList&lt;&gt;();

@Override
public void startElement(String uri, String localName,
                         String qName, Attributes atts)
  throws SAXException {
  if ("http://www.w3.org/1999/xhtml".equals(uri)) {
    if ("table".equals(localName)) { push(); } else
    if (current != null) {
      if ("td".equals(localName) || "th".equals(localName)) {
        current.cell(atts, localName);
      }
      // ... more element handling
    }
  } } }
</code></pre>
<p>Since the number of rules was massive, a single run of the agent wasn't enough. Instead, I knocked up a <a href="https://anil.recoil.org/notes/aoah-2025-4">Claudeio</a> wrapper that ran the agent iteratively exhorting it to continually sample the rules and iterate on a good architecture. This is only possible since the validator has a <a href="https://github.com/validator/validator/blob/main/tests/messages.json">massive test suite</a> with expected outputs. The goal of the agent was therefore to try OCaml code architectures that maximised the number of passing rules, and then combining them towards getting 100% pass rate.</p>
<p>After a few days, this converged and hit 100% pass rate with a bit of human prompt massaging from me.  The OCaml version replaces inheritance with <a href="https://dev.realworldocaml.org/first-class-modules.html">first-class modules</a> instead, and each checker implements the <a href="https://tangled.org/anil.recoil.org/ocaml-html5rw/blob/main/lib/check/checker.mli">Checker.S</a> signature:</p>
<pre><code>module type S = sig
  type state
  val create : unit -&gt; state
  val reset : state -&gt; unit
  val start_element : state -&gt; element:Element.t -&gt; Message_collector.t -&gt; unit
  val end_element : state -&gt; tag:Tag.element_tag -&gt; Message_collector.t -&gt; unit
  val characters : state -&gt; string -&gt; Message_collector.t -&gt; unit
  val end_document : state -&gt; Message_collector.t -&gt; unit
end
</code></pre>
<p>This gives us the same flexibility to compose checkers, but with abstract state
types rather than hidden mutable fields scattered across a class hierarchy.</p>
<h3><a href="https://anil.recoil.org/news.xml#browsing-the-giant-html5-test-suite" class="anchor" aria-hidden="true"></a>Browsing the giant HTML5 test suite</h3>
<p>I extended the visual HTML test suite generator I built a few days ago to include the thousands of validation tests, and the library now outputs one <a href="https://www.cl.cam.ac.uk/~avsm2/html5rw-check/">epic HTML file</a> that lists each of the thousands of tests.</p>
<p>Having these tests was essential when doing refactoring, as a small change in one checker affected others. Without the comprehensive test oracle, the agents would quickly diverge out of control.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/html5rw-check/"> <img src="https://anil.recoil.org/images/aoah-html5v-ss-4.webp" alt="%c" title="It's quite fun just browsing around the expect tests to see what's going on with HTML5"> </a></p>
<h3><a href="https://anil.recoil.org/news.xml#a-case-study-on-the-table-checker" class="anchor" aria-hidden="true"></a>A case study on the table checker</h3>
<p>The table checker is one of the more complex validators, tracking cell spans,
detecting overlaps, and validating that <code>headers</code> attributes reference valid
<code>th</code> elements. The Java version spreads this across multiple files, but the
OCaml version consolidates everything into a <a href="https://tangled.org/anil.recoil.org/ocaml-html5rw/blob/main/lib/check/specialized/table_checker.ml">single module</a> with explicit types.</p>
<pre><code class="language-ocaml">type cell = {
  mutable left : int;
  mutable right : int;
  mutable bottom : int;
  headers : string list;
  element_name : string;
}

type row_group = {
  mutable current_row : int;
  mutable insertion_point : int;
  cells_in_effect : ((int * int), cell) Hashtbl.t;
  mutable cells_on_current_row : cell array;
  row_group_type : string option;
}

type table = {
  mutable state : table_state;
  mutable column_count : int;
  header_ids : (string, unit) Hashtbl.t;
  cells_with_headers : cell list ref;
  mutable current_row_group : row_group option;
  (* ... *)
}
</code></pre>
<p>However, the agent didn't fundamentally change the algorithmic structure; we still have the same basic state machine but with much more succinct variant types.</p>
<h2><a href="https://anil.recoil.org/news.xml#typed-error-codes" class="anchor" aria-hidden="true"></a>Typed error codes</h2>
<p>One significant quality of life improvement came from refactoring how the error messages are collected for rendering. The Java code uses string formatting throughout to directly output messages, but the OCaml <a href="https://tangled.org/anil.recoil.org/ocaml-html5rw/blob/main/lib/check/error_code.mli">error_code.mli</a> module defines a polymorphic variant hierarchy that's exhaustively checkable:</p>
<pre><code class="language-ocaml">type table_error = [
  | `Cell_overlap
  | `Cell_spans_rowgroup
  | `Row_no_cells of [`Row of int]
  | `Column_no_cells of [`Column of int] * [`Elem of string]
]

type attr_error = [
  | `Not_allowed of [`Attr of string] * [`Elem of string]
  | `Missing of [`Elem of string] * [`Attr of string]
  | `Bad_value of [`Elem of string] * [`Attr of string] *
                  [`Value of string] * [`Reason of string]
  | `Duplicate_id of [`Id of string]
  (* ... *)
]
</code></pre>
<p>This allows clients to pattern match on specific classes of errors easily:</p>
<pre><code class="language-ocaml">match err with
| `Attr (`Duplicate_id (`Id id)) -&gt; handle_duplicate id
| `Img `Missing_alt -&gt; suggest_alt_text ()
| `Table `Cell_overlap -&gt; report_overlap ()
| _ -&gt; default_handler err
</code></pre>
<p>This also means we can add new error categories without changing existing code,
and the compiler tells us if we miss any cases. This is a pretty classic
usecase for OCaml that both <a href="https://github.com/yminsky">Yaron Minsky</a> and I talked about (in <a href="https://www.youtube.com/watch?v=hKcOkWzj0_s">Caml
Trading</a> and
<a href="https://anil.recoil.org/papers/2010-icfp-xen">XenServer</a>), but sometimes it's good to remember why I love
OCaml so much!</p>
<h2><a href="https://anil.recoil.org/news.xml#reflections" class="anchor" aria-hidden="true"></a>Reflections</h2>
<p>While the OCaml code generated is by no means sparkling clean, it is useful and
operational already.  The typed error hierarchy is perhaps the biggest win, as
it lifts up the abstraction level to be more idiomatic to OCaml style and
eventually makes it easier to perhaps jump over to Haskell or Lean for even
more purity and formal specification work.  This is by far the biggest agentic
coding translation I've attempted so far, to the point where it used up all my
Claude 20x Max credits in a matter of days. I have two accounts now!</p>
<p>What also surprised me was how little the agent struggled with the architectural
transformation <em>across programming languages</em>. Given examples of OCaml first
class modules (from Real World OCaml and the Jane Street OCaml code), it
produced well-structured code.</p>
<p><img src="https://anil.recoil.org/images/aoah-html5v-ss-3.webp" alt="%c" title="Claude can introspect its session context to update a skill to match what its learnt"></p>
<p>I still have no idea how I'm going to maintain this code in the long term, but
do let me know if the HTML5 checker is useful to you. One little Claude trick
that remains handy is that after a prompting session, I prompt the agent to fix
its own skill based on the feedback its received this session. That helps to
generalise the skills as more projects get to using it.</p>

