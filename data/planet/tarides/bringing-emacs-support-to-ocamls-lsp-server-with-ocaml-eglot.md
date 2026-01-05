---
title: Bringing Emacs Support to OCaml's LSP Server with `ocaml-eglot`
description: Discover the `ocaml-eglot` project, a plugin that integrates OCaml's
  Emacs editor support with the LSP.
url: https://tarides.com/blog/2025-11-27-bringing-emacs-support-to-ocaml-s-lsp-server-with-ocaml-eglot
date: 2025-11-27T00:00:00-00:00
preview_image: https://tarides.com/blog/images/emacs-image-1360w.webp
authors:
- Tarides
source:
---

<p>The team of people working on editors and editor support at Tarides is excited to announce the release of <code>ocaml-eglot</code>! The project is part of Tarides’ efforts to improve the OCaml developer experience across different platforms and workflows, a high-priority goal continuously evolving with community feedback.</p>
<p>Bringing Emacs integration to OCaml’s LSP server benefits both the user and the maintainer. If you use Emacs and want to start using OCaml, or switch to a more simplified setup, check out the <a href="https://github.com/tarides/ocaml-eglot"><code>ocaml-eglot</code> repository</a> on GitHub to try the new Emacs minor mode.</p>
<p>This post will give you some background to the development of the new tool, as well as the benefits and limitations of LSP, and the features of <code>ocaml-eglot</code>. Let’s dive in!</p>
<h2>The Problem: ‘Editor Burnout’</h2>
<p>The goal of the <code>ocaml-eglot</code> project was to address a problem the engineers had dubbed <em><code>editor burnout</code></em>. Developers rely on editors to simplify their coding workflow, and over the years, the creation of more and more editor features has transformed editors into sophisticated, feature-rich development environments. However, all these features need to be added and maintained in every editor. Maintaining support for so many different features across different editors, including updating the support every time something changes on the language server's end, can quickly become untenable. ‘Editor burnout’ refers to the pressure this puts on maintainers.</p>
<p>In OCaml, the editor-agnostic server <a href="https://ocaml.github.io/merlin/">Merlin</a> is used to provide IDE-like services. By providing contextual information about the code, Merlin lets developers use simple text editors to write OCaml and benefit from features that typically come from a fully integrated IDE. However, Merlin also had a high maintenance cost due to each code editor needing its own integration layer.</p>
<p>So, now that we understand the problem, what is the solution?</p>
<h2>LSP and OCaml</h2>
<p>LSP, or the <em>Language Server Protocol</em>, is a widely documented open protocol that standardises the interactions between an editor and a server providing IDE services. LSP defines a collection of standard features across programming languages, which has contributed to its widespread adoption. This adoption has made LSP a standard protocol across editors, including <a href="https://code.visualstudio.com/">Visual Studio Code</a>, <a href="https://www.vim.org">Vim</a>, <a href="https://www.gnu.org/software/emacs/">Emacs</a>, and many more.</p>
<p>The language server implementation for LSP in OCaml is <code>ocaml-lsp</code>. It uses Merlin as a library. It was originally designed to integrate with <a href="https://code.visualstudio.com">Visual Studio Code</a> when paired with the <code>vscode-ocaml-platform</code> plugin. We can significantly reduce the maintenance burden by relying on LSP's defaults for editor compatibility and only providing support for OCaml-specific features. This benefits not only the maintainers, but also the user by ensuring the plugins remain performant, compatible, maintainable, and up-to-date.</p>
<p>LSP aims to be compatible with as many languages as possible, making some assumptions about how those languages are structured and function. Inevitably, these assumptions cannot cover all the features of every language. This is true of OCaml, where the editing experience relies on custom features outside the scope of the LSP.</p>
<p>The solution to this incompatibility is to create a <em>client-side extension</em> that covers what the editor’s standard LSP support does not. That way, we have both the basic LSP compatibility and an extension that adds support for OCaml-specific features. As we’ve hinted above, this has the added benefit of keeping the maintenance workload on the editor side down by delegating the standard LSP handling to the generic LSP plug-ins.</p>
<p>Some of these OCaml-specific editor features include <a href="https://github.com/tarides/ocaml-eglot?tab=readme-ov-file#type-enclosings">type-enclosing</a>, <a href="https://github.com/tarides/ocaml-eglot?tab=readme-ov-file#construct-expression">construct and navigation between holes</a>, <a href="https://github.com/tarides/ocaml-eglot?tab=readme-ov-file#destruct-or-case-analysis">destruct</a>, and <a href="https://github.com/tarides/ocaml-eglot?tab=readme-ov-file#search-for-values">search by types</a>.</p>
<h2>OCaml-Eglot</h2>
<p>As an editor popular with the OCaml community, let’s take a brief look at how Emacs and OCaml work together. In Emacs, developers can attach a "buffer"/file to a major mode to handle a feature of a language like OCaml: features like syntax highlighting, for example. One file is always attached to just one major mode.</p>
<p>OCaml has four major modes:</p>
<ul>
<li><code>caml-mode</code>: the original,</li>
<li><code>tuareg</code>: a full reimplementation of <code>caml-mode</code> and the most common choice by users,</li>
<li><code>ocaml-ts-mode</code>: an experimental version of <code>caml-mode</code> based on tree-sitter grammar,</li>
<li><code>neocaml</code>: an experimental full reimplementation of <code>tuareg</code> based on tree-sitter grammar.</li>
</ul>
<p>Now, we can also attach one or multiple <code>minor-mode</code>s to a file, and this is where <code>ocaml-eglot</code> comes into play. For example, we can use a major mode (we generally recommend Tuareg) and link <code>ocaml-eglot</code> to it as a minor mode, thereby attaching LSP features to all files in which Tuareg is active.</p>
<p>Eglot is the default LSP client bundled with Emacs, and <code>ocaml-eglot</code> provides full OCaml language support in Emacs as an alternative to Merlin integration. (By the way, thanks to the <code>ocaml-eglot</code> client using LSP’s defaults, its code size is a lot smaller than the traditional OCaml <code>Emacs</code> mode, which also makes it easier to maintain!).</p>
<p>The ideal user of <code>ocaml-eglot</code> is someone who is already an active Emacs user and wants to start using OCaml with minimal start-up hassle. The simplified configuration, automated setup, and consistency across different editors and languages are helpful both to people new to OCaml and to seasoned users with multiple editors, since they improve the workflow. The plugin supports all the features of the integration of Merlin into Emacs, <code>merlin.el</code>, meaning that users don’t lose any functionality with the new system. The <code>ocaml-eglot</code> project is also actively maintained, and users can expect regular future updates and a tool that evolves with the times.</p>
<h3>Creating OCaml-Eglot</h3>
<p>Let's peek behind the curtain at the development of <code>ocaml-eglot</code>. There are two common approaches that developers who implement server languages tend to use to add features outside of the LSP. These are <a href="https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#textDocument_codeAction">Code Actions</a> and Custom Requests:</p>
<ul>
<li>Code Action: A contextual action that can be triggered from a document perspective, can perform a file modification, and potentially broadcast a command that can be interpreted by the client. Code Actions are more
‘integrated’, which means that they sometimes even work ‘out of the box’ with the client. However, they are limited in terms of interactions and the command lifecycle.</li>
<li>Custom Request: Not formally part of the protocol, but since LSP is a protocol layered on top of a regular server that can handle JSON RPC messages and responses, developers can still use arbitrary requests to provide extra features. Custom Requests give developers more power to add interactions and experiences, but always need specific editor integration.</li>
</ul>
<p>The design process behind OCaml-eglot essentially boiled down to identifying all the features offered by <code>merlin.el</code> that were not covered by the LSP, and then adding them using Code Actions or Custom Requests. During this process, the developers asked themselves two questions to help them decide which approach to use:</p>
<ul>
<li>Should the feature be configured by arguments that are <strong>independent of the context:</strong> If the answer is yes, they used a Custom Request; if no, they used a Code Action.</li>
<li>Does the feature require <strong>additional interaction</strong> such as choosing one option from a set of possible results?: If yes, they used a Custom Request; if no, they used a Code Action.</li>
</ul>
<p>Of course, things were a little more complicated than this in reality, but it still gives you a good idea of the types of technical decisions the team made during development.</p>
<h2>Try it Out!</h2>
<p>Install <code>ocaml-eglot</code> by checking out its <a href="https://github.com/tarides/ocaml-eglot">GitHub repository</a> and following the instructions. When you have had a chance to test it out in your projects, please share your experience on <a href="https://discuss.ocaml.org">OCaml Discuss</a> to give other users an idea of what to expect and the maintainers an idea on what to improve!</p>
<p>Installing <code>ocaml-eglot</code> is just like installing a regular Emacs package. It is available on <a href="https://melpa.org/#/ocaml-eglot">Melpa</a> and can be installed in many different ways, for example with GNU’s <code>use package</code>. More detailed instructions are available <a href="https://github.com/tarides/ocaml-eglot?tab=readme-ov-file#ocaml-eglot">in the repo’s readme</a>, including instructions on recommended configurations for <code>ocaml-eglot</code>.</p>
<h3>Features</h3>
<p>Some of the features that <code>ocaml-eglot</code> comes with are:</p>
<ul>
<li>Error navigation: Quickly jump to the next or previous error(s).</li>
<li>Type information: Display types under cursor with adjustable verbosity and navigate enclosing expressions.</li>
<li>Code generation: Pattern match construction, case completion, and wildcard refinement via the ‘destruct’ feature.</li>
<li>Navigation: Jump between language constructs like let, module, function, match, and navigate phrases and pattern cases.</li>
<li>Search: Find definitions, declarations, and references. The team also recently introduced a new Xref Backend inspired by one used by Jane Street for years.</li>
</ul>
<p>Check out the project's readme to discover the full list of commands offered by <code>ocaml-eglot</code>. The new mode is ‘agile’, meaning that the team can also incubate new features quickly, like the <a href="https://github.com/tarides/ocaml-eglot/pull/65">refactor extract at toplevel</a>.</p>
<h2>Until Next Time</h2>
<p>For some really helpful background that goes into more detail than we did in this post, I recommend that you read the paper <a href="https://conf.researchr.org/details/icfp-splash-2025/ocaml-2025-papers/7/A-New-Era-of-OCaml-Editing-Powered-by-Merlin-Delivered-via-LSP">“A New Era of OCaml Editing Powered by Merlin, Delivered by LSP”</a> by Xavier van de Woestyne, Sonja Heinze, Ulysse Gérard, and Muluh Godson.</p>
<p>You can connect with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, <a href="https://www.threads.net/@taridesltd">Threads</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up to our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

