---
title: "Florian\u2019s compiler news, 28 October 2025"
description:
url: https://gallium.inria.fr/blog/florian-cn-2025-10-28
date: 2025-01-13T08:00:00-00:00
preview_image:
authors:
- GaGallium
source:
---



  <p>This series of blog post aims to give a short weekly glimpse into my
(Florian Angeletti) work on the OCaml compiler. This week subject is my
personal retrospective on the release of OCaml 5.4.0.</p>


  

  
  <h2>PRs</h2>
<h3>Stdlib</h3>
<ul>
<li><p>(<em>breaking change</em>) <a href="https://github.com/ocaml/ocaml/issues/13570">#13570</a>, <a href="https://github.com/ocaml/ocaml/issues/13794">#13794</a>: Format,
add an out_width function to Format device for approximating unicode
width. (Florian Angeletti, review by Nicolás Ojeda Bär, Daniel Bünzli,
and Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13569">#13569</a>:
add a <code>Format.format_text</code> which adds break hints to format
literals. (Florian Angeletti, review by Nicolás Ojeda Bär, Daniel
Bünzli, and Gabriel Scherer)</p></li>
</ul>
<h3>CLI</h3>
<ul>
<li><a href="https://github.com/ocaml/ocaml/issues/13764">#13764</a>, <a href="https://github.com/ocaml/ocaml/issues/13779">#13779</a>: add
missing “-keywords” flag to ocamldep and ocamlprof (Florian Angeletti,
report by Prashanth Mundkur, review by Gabriel Scherer)</li>
</ul>
<h3>OCamldoc</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13877">#13877</a>:
ocamldoc, add a <code>-latex-escape-underscore</code> flag to control
the escaping of <code>_</code> underscore in latex references (in order
to be able to match odoc behaviour). (Florian Angeletti, review by
Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13896">#13896</a>,
<a href="https://github.com/ocaml/ocaml/issues/14098">#14098</a>:
ocamldoc, do not wrap module description in a paragraph tag inside the
table of modules (Florian Angeletti, report by John Whitington, review
by Gabriel Scherer)</p></li>
</ul>
<h3>Error messages</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13702">#13702</a>,
<a href="https://github.com/ocaml/ocaml/issues/13865">#13865</a>:
Specialized error messages for functors appearing in contexts where
non-functors were expected <code>module A: sig ... end = Set.Make</code>
(and the reverse) (Florian Angeletti, report by Jeremy Yallop, review by
Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14070">#14070</a>:
also point to label mismatches in error messages for labelled tuples
(Florian Angeletti, review by Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13788">#13788</a>,
<a href="https://github.com/ocaml/ocaml/issues/13813">#13813</a>: Keep
the module context in spellchecking hints. <code>Fun.protact</code> now
prompts <code>Did you mean "Fun.protect?"</code> rather than
<code>Did you mean "protect?"</code>. (Florian Angeletti, suggestion by
Daniel Bünzli, review by Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13817">#13817</a>:
align spellchecking hints with the possibly misspelled identifier/
Error: Unbound type constructor “aray” Hint: Did you mean “array”?
(Florian Angeletti, suggestion by Daniel Bünzli, review by Gabriel
Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13563">#13563</a>,
lighter inline code styling for output without bold support: inline code
is no longer printed as “…” to avoid confusion with OCaml strings.
(Florian Angeletti, review by Richard Eisenberg)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13568">#13568</a>,
composable formatting for warning and alert messages (Florian Angeletti,
review by Richard Eisenberg)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13818">#13818</a>:
better delimited hints in error message (Florian Angeletti, review by
Gabriel Scherer)</p></li>
</ul>
<h3>Typechecker bug fixes:</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13778">#13778</a>,
<a href="https://github.com/ocaml/ocaml/issues/13811">#13811</a>: do not
warn for unused type declarations when the type is used in a first-class
module type (<code>module S with type t = int)</code>. (Florian
Angeletti, report by Nicolás Ojeda Bär, review by Gabriel
Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14135">#14135</a>:
Fix a rare internal typechecker error when combining recursive modules,
polymorphic fields or methods, and constrained type parameters. (Florian
Angeletti, review by Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14214">#14214</a>,
<a href="https://github.com/ocaml/ocaml/issues/14221">#14221</a>: fix a
confused error message for module inclusions, functor error messages
were missing some type equalities potentially leading to nonsensical
“type t is not compatible with type t” submessage (Florian Angeletti,
report by Basile Clément, review by Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14108">#14108</a>:
toplevel, fix a typo in directive type mismatch (Florian Angeletti,
review by Gabriel Scherer)</p></li>
</ul>
<h3>Runtime bug fix</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14101">#14101</a>,
<a href="https://github.com/ocaml/ocaml/issues/14139">#14139</a>: define
atomic helper types inside <code>caml/misc.h</code> to improve header
compatibility with C++ (Florian Angeletti, report by Kate Deplaix,
review by Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14169">#14169</a>:
runtime, fix cache miss within the stack fragments cache (Florian
Angeletti, review by Gabriel Scherer)</p></li>
</ul>
<h2>Reviews</h2>
<h3>Standard library:</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13760">#13760</a>:
Add String.{edit_distance,spellcheck} (Daniel Bünzli, review by wikku,
Nicolás Ojeda Bär, Gabriel Scherer and Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13753">#13753</a>,
<a href="https://github.com/ocaml/ocaml/issues/13755">#13755</a>: Add
Stdlib.Repr: Repr.phys_equal and Repr.compare are more explicit than
(==) and <code>compare</code>. (Kate Deplaix, Thomas Blanc and Léo
Andrès, review by Gabriel Scherer, Florian Angeletti, Nicolás Ojeda Bär,
Daniel Bünzli and Jeremy Yallop)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13796">#13796</a>:
Add Uchar.utf_8_decode_length_of_byte and Uchar.max_utf_8_decode_length.
(Daniel Bünzli, review by Nicolás Ojeda Bär and Florian
Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13768">#13768</a>:
Add Either.get_left and Either.get_right (T. Kinsart, review by Nicolás
Ojeda Bär and Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13310">#13310</a>:
Add Stdlib.Pair (Victoire Noizet, review by Nicolás Ojeda Bär, Daniel
Bünzli, Xavier Van de Woestyne, Jeremy Yallop and Florian
Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13620">#13620</a>:
Avoid copying the string in String.concat, String.sub and
String.split_on_char when the full string is returned. (Christophe
Raffalli, review by Nicolás Ojeda Bär and Gabriel Scherer and Hugo
Heuzard)</p></li>
</ul>
<h3>Tools:</h3>
<ul>
<li><a href="https://github.com/ocaml/ocaml/issues/12642">#12642</a>, <a href="https://github.com/ocaml/ocaml/issues/13536">#13536</a>, <a href="https://github.com/ocaml/ocaml/issues/14184">#14184</a>, <a href="https://github.com/ocaml/ocaml/issues/14192">#14192</a>: in the
toplevel, print shorter paths for constructors and labels when only some
modules along their path are open. (Gabriel Scherer, review by Florian
Angeletti)</li>
</ul>
<h3>Manual and documentation:</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/12452">#12452</a>:
Add examples to Stdlib.Fun documentation. (Hazem ElMasry, review by
Florian Angeletti and Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13924">#13924</a>:
Document how to put <span class="citation" data-cites="deprecated">[@deprecated]</span> on let bindings,
constructors, etc in the manual (Valentin Gatien-Baron, review by
Florian Angeletti)</p></li>
</ul>
<h3>Compiler user-interface
and warnings:</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13663">#13663</a>:
Improve the error message when GADT parameter variance cannot be
checked. (Stefan Muenzel, review by Gabriel Scherer and Florian
Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13646">#13646</a>:
Improve the error messages when a recursive module type references
another recursive module type. (Stefan Muenzel, review by Florian
Angeletti and Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13809">#13809</a>:
Distinguish <code>(module M : S)</code> and
<code>(module M) : (module S)</code> and change locations of error
messages when <code>S</code> is ill-typed in <code>(module S)</code>
(Samuel Vivien, review by Florian Angeletti and Gabriel
Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13814">#13814</a>,
13898: Add an <code>unused-type-declaration</code> warning when using a
<code>t as 'a</code> with no other occurences of <code>'a</code> (Samuel
Vivien, review by Florian Angeletti, Kate Deplaix)</p></li>
</ul>
<h3>Internal/compiler-libs
changes:</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13302">#13302</a>,
<a href="https://github.com/ocaml/ocaml/issues/14236">#14236</a>: Store
locations of longidents components (Ulysse Gérard and Jules Aguillon,
review by Jules Aguillon and Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13460">#13460</a>:
introduce a variant of all predefined types (Gabriel Scherer, review by
Ulysse Gérard and Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13457">#13457</a>,
<a href="https://github.com/ocaml/ocaml/issues/13537">#13537</a>:
Annotate alloc/free open/close pairs of functions with compiler
attributes for static analysis. (Antonin Décimo, review by Gabriel
Scherer and Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13612">#13612</a>:
Refactor <code>type_application</code> (Ulysse Gérard, Leo White, review
by Antonin Décimo, Gabriel Scherer, Samuel Vivien, Florian Angeletti and
Jacques Garrigue)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13744">#13744</a>:
Refactor in <code>collect_apply_args</code> (Samuel Vivien, review by
Florian Angeletti and Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13820">#13820</a>:
Add a new option -i-variance to print the variance of every type
parameter; bivariance is printed as <code>+-</code>, and for
consistency, parser is modified too to accept <code>+-</code> and
<code>-+</code> as <code>type_variance</code>. (Takafumi Saikawa and
Jacques Garrigue, review by Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13848">#13848</a>:
Add all paths components to the cmt files indexes (Ulysse Gérard, review
by Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13854">#13854</a>:
Make the parser set loc_ghost more correctly, for
<code>keyword%extension</code> syntax (Valentin Gatien-Baron, review by
Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13856">#13856</a>:
Add a new indirection in types AST called <code>package</code> that
stores the content of a <code>Tpackage</code> node (Samuel Vivien,
review by Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13866">#13866</a>:
Modified occurence check that prevents recursive types for it to see the
checked type as a graph rather than a tree (Samuel Vivien, report by
Didier Remy, review by Florian Angeletti and Jacques Garrigue)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13884">#13884</a>
Correctly index modules in constructors and labels paths (Ulysse Gérard,
review by Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13946">#13946</a>:
refactor the #install_printer code in the debugger and toplevel (Pierre
Boutillier, review by Gabriel Scherer and Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13971">#13971</a>:
Keep generalized structure from patterns when typing <code>let</code>
(Leo White, review by Samuel Vivien and Florian Angeletti)</p></li>
</ul>
<h3>Build system:</h3>
<ul>
<li><a href="https://github.com/ocaml/ocaml/issues/13431">#13431</a>:
Simplify github action responsible for flagging PRs with the
<code>parsetree-changes</code> label and extend it to mention the <span class="citation" data-cites="ppxlib-dev">@ppxlib-dev</span> team.
(Nathan Rebours, review by Florian Angeletti)</li>
</ul>
<h3>Bug fixes:</h3>
<ul>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13957">#13957</a>:
Allow ‘effect’ as attribute id. (Pieter Goetschalckx, review by Nicolás
Ojeda Bär and Florian Angeletti)</p></li>
<li><p>(<em>breaking change</em>) <a href="https://github.com/ocaml/ocaml/issues/13605">#13605</a>: Fix
ungenerated constraints when they where impossible due to polyvars
issues (Samuel Vivien, review by Florian Angeletti, Richard Eisenberg
and Jacques Garrigue)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13710">#13710</a>:
Support unicode identifiers in comments. (Pieter Goetschalckx, review by
Florian Angeletti and Gabriel Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13845">#13845</a>:
Fix bug in untypeast/pprintast for value bindings with polymorphic type
annotations. (Chris Casinghino, review by Florian Angeletti and Gabriel
Scherer)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13172">#13172</a>,
<a href="https://github.com/ocaml/ocaml/issues/13829">#13829</a>: Fix a
missing check of illegal recursive module-type definitions (Clement
Blaudeau, review by Florian Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13541">#13541</a>,
<a href="https://github.com/ocaml/ocaml/issues/13777">#13777</a>: Using
C++11 <code>thread_local</code> causes name-mangling issues when linking
with flexlink on Cygwin. (Antonin Décimo and David Allsopp, report by
Kate Deplaix)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/13956">#13956</a>
Fix a regression introduced in <a href="https://github.com/ocaml/ocaml/issues/13308">#13308</a> triggering
wrong unused warnings. (Ulysse Gérard, review by Florian
Angeletti)</p></li>
<li><p><a href="https://github.com/ocaml/ocaml/issues/14105">#14105</a>:
Fix a loop in Pprintast that could result in a hang when printing
constructor <code>(::)</code> in isolation. (Ulysse Gérard, review by
Nicolás Ojeda Bär and Florian Angeletti)</p></li>
</ul>


  
