---
title: Creating `ocaml.nvim` to Bring Neovim Support to OCaml's LSP Server
description: Introducing the `ocaml.nvim` plugin, which integrates OCaml's Neovim
  editor with the LSP.
url: https://tarides.com/blog/2025-12-10-creating-ocaml-nvim-to-bring-neovim-support-to-ocaml-s-lsp-server
date: 2025-12-10T00:00:00-00:00
preview_image: https://tarides.com/blog/images/nvim-post-1360w.webp
authors:
- Tarides
source:
---

<p>We are happy to announce the release of the new <code>ocaml.nvim</code> plugin! It provides Neovim users with access to advanced <code>ocaml-lsp</code> features without getting involved in complicated editor-side logic. Think of it as the <a href="https://ocaml.org/backstage/2025-10-14-ocaml-nvim-a-neovim-plugin-for-ocaml">Neovim sibling</a> of the Emacs plugin <code>ocaml-eglot</code> – which we have also <a href="https://tarides.com/blog/2025-11-27-bringing-emacs-support-to-ocaml-s-lsp-server-with-ocaml-eglot/">covered on the blog</a> – simplifying maintenance while still providing custom OCaml features. This is part of Tarides' ongoing effort to improve the OCaml user experience by introducing new features and improving existing ones.</p>
<p>In this post, we provide a brief overview of the relationship between Merlin, LSP, and Neovim in OCaml, the goals and features of <code>ocaml.nvim</code>, and offer some insight into how the engineers approached this project. We encourage you to install the plugin if you’re a NeoVim user (or just curious!) and give it a try. The <a href="https://github.com/tarides/ocaml.nvim">repository is public</a> and comes with documentation about its features and how to install the plugin. To help the team out, you can share your feedback on the <a href="https://discuss.ocaml.org/">OCaml Discuss forum</a>, in the repo, or by emailing <a href="mailto:charlene@tarides.com">Charlène</a> directly.</p>
<h2>Merlin, LSP, &amp; NeoVim</h2>
<p>The <a href="https://microsoft.github.io/language-server-protocol/">Language Server Protocol</a> (LSP) was created to address a programming landscape of a growing number of incompatible editors. The open protocol standardises the interactions between an editor and a server providing IDE services, creating a standard model supporting any compatible editor.</p>
<p>Created before the LSP, OCaml’s editor-agnostic server, Merlin, has offered advanced in-editor functionalities to the language for a long time. It quickly gained popularity within the community and was supported by several editors, including Emacs and Vim. However, supporting and maintaining each individual editor was resource intensive, as they required a tailored approach whenever they, OCaml, or Merlin updated.</p>
<p>Now, thanks to the fact that most modern editors use LSP to communicate with programming languages, OCaml can use LSP defaults to integrate Merlin with each editor’s LSP plugins. Neovim is one of the most popular editors (alongside Emacs, Vim, and VSCode) in the OCaml community. The new <code>ocaml.nvim</code> extension enables Neovim to support custom commands that are not possible with the standard LSP. The plugin <code>ocaml.nvim</code> works with generic Neovim LSP to provide access to advanced <code>ocaml-lsp</code> features, including all the advanced Merlin commands not supported by generic LSP clients. This approach addresses the so-called ‘editor burnout’ problem, which we have described in greater detail in our <a href="https://tarides.com/blog/2025-11-27-bringing-emacs-support-to-ocaml-s-lsp-server-with-ocaml-eglot/">post on <code>ocaml-eglot</code></a>, <code>ocaml.nvim</code>’s ‘sibling’.</p>
<h2>OCaml.nvim: Creating a New Plugin</h2>
<p>The goal of the project was to create a modern and idiomatic Neovim plugin that would implement the server’s custom requests in a way that respected the design of the editor. We wanted to create a good foundation for community and industry users to build on, with a minimalist and well-documented plugin.</p>
<p>The <code>ocaml.nvim</code> plugin, of course, also supports key custom OCaml features, including advanced type-enclosing functionality, syntax-aware navigation, search by type, construct, and switching between the corresponding <code>.ml</code> and <code>.mli</code>. Check out <a href="https://github.com/tarides/ocaml.nvim">the <code>ocaml.nvim</code></a> repository to read more about the supported features.</p>
<h3>Code Actions and Custom Requests</h3>
<p>Since this plugin aimed to enhance the standard LSP features, we needed to rely on an additional protocol to link specific information to a particular command.
LSP offers two types of special requests: <code>code actions</code> and <code>custom requests</code>.</p>
<p>The LSP already lists <code>code actions</code>, but when there are numerous options, this special request can be confusing to use.
Moreover, <code>code actions</code> prevent the use of parameters. They do not, for example, let the user select between multiple choices.
This is where <code>custom requests</code> come into play.</p>
<p><code>Custom requests</code> allow the language server to converse with the user as much as needed, for example, to collect choices. As a consequence, they require an extra plugin to translate the conversation from the server to the user, as the LSP cannot generalise the infinity of commands we could imagine in all languages.</p>
<p>Every function is available on the Merlin side and accessible via <a href="https://github.com/ocaml/ocaml-lsp"><code>ocaml-lsp</code></a>.
There are two types of calls.</p>
<ul>
<li><code>Ocaml-lsp</code> API: This is the language server for OCaml. It provides the default features to the LSP. It also offers more than that. We can connect to the API to request information based on the cursor location or the server's knowledge of the current file.</li>
<li><code>MerlinCallCompatible</code>: This is also provided by <code>ocaml-lsp</code>, and allows us to invoke the Merlin commands. It requires additional arguments and returns the result in a character string format; here, we used <a href="https://www.json.org/json-en.html">JSON</a>.</li>
</ul>
<h3>UI Elements</h3>
<p>Our objective was to provide a clear interface. To achieve this, we implemented four main UI elements for the entire plugin.</p>
<ul>
<li><strong>Selector</strong>: opens a floating window where the focus is initialised at the first line. Then, you can use the arrows and enter to move and select the wanted line.</li>
</ul>
<p><img src="https://tarides.com/blog/images/selector-1360w~tJ_bf9zaVtOkbV95LyD-Ww.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/selector-170w~589GpSmVs6YS-VbF_Nhp0Q.webp 170w, /blog/images/selector-340w~EzbLKgR7p83XYF3qDlO29Q.webp 340w, /blog/images/selector-680w~YePRvKGvQbaDCkm_ED8PWg.webp 680w, /blog/images/selector-1360w~tJ_bf9zaVtOkbV95LyD-Ww.webp 1360w" alt="The floating window with the first line ‘sys.command string -> int’ highlighted"></p>
<ul>
<li><strong>New Window</strong>: This is built-in with <code>nvim</code>, and splits the current window to open a new buffer.</li>
</ul>
<p><img src="https://tarides.com/blog/images/new_window-1360w~0lEM8ZGtonWzGSH1d4hCTg.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/new_window-170w~h6LsFJSI9CF7yHpdz6mqUQ.webp 170w, /blog/images/new_window-340w~cyswlvU5RrErWY2Fg6DF3A.webp 340w, /blog/images/new_window-680w~H7BnMN2qTb51Y1U_V0jWdQ.webp 680w, /blog/images/new_window-1360w~0lEM8ZGtonWzGSH1d4hCTg.webp 1360w" alt="The floating window with a split and a new buffer"></p>
<ul>
<li><strong>One-line Display</strong>: This is the simplest UI, simply a <code>print</code>. If it takes less than one line, it is displayed where you enter the commands.</li>
</ul>
<p><img src="https://tarides.com/blog/images/one_line_display-1360w~tReqPqPAiLYzRuopW_Rqhg.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/one_line_display-170w~k8rvKa-yENBmZ4vB7AiuHQ.webp 170w, /blog/images/one_line_display-340w~5sQVUMZrDhiFTVIdwiJqWw.webp 340w, /blog/images/one_line_display-680w~ZiqO-tgu2v3vfRJVBzkMfw.webp 680w, /blog/images/one_line_display-1360w~tReqPqPAiLYzRuopW_Rqhg.webp 1360w" alt="The floating window displaying the string in a single line"></p>
<ul>
<li><strong>Multi-line Display</strong>: By default, if the string in <code>print</code> exceeds one line, it displays it in multiple lines. However, this behaviour is not ideal because it requires pressing Enter to erase the text in an unappealing manner. So, the plugin provides an alternative that is similar to the UI selector.</li>
</ul>
<p><img src="https://tarides.com/blog/images/multi_line_display-1360w~Z0lM3F-VKdOC4JPnSoYwvQ.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/multi_line_display-170w~JqOJD_ZncXAc8ncLnVJFDw.webp 170w, /blog/images/multi_line_display-340w~nVOGTpIjwzDKkwjU-mHZIg.webp 340w, /blog/images/multi_line_display-680w~8EKMFXwcThDbOfN6vEsZkg.webp 680w, /blog/images/multi_line_display-1360w~Z0lM3F-VKdOC4JPnSoYwvQ.webp 1360w" alt="The floating window with a box containing the information from the string"></p>
<p>Regarding customisation, since everyone has their own keymaps, we provided default ones and allow users to modify them to suit their needs and preferences.</p>
<h2>Try it Out!</h2>
<p>Visit the <a href="https://github.com/tarides/ocaml.nvim"><code>ocaml.nvim</code> repository on GitHub</a> to get started with the new plugin. The repo includes the installation instructions, a straightforward process using <code>lazy.nvim</code>, as well as a features list and some examples.</p>
<p>Since the team is working towards a 1.0 release, any and all feedback is welcome. Remember to share your thoughts, suggestions, and questions on <a href="https://discuss.ocaml.org">Discuss</a> or in the repo through issues and commits.</p>
<p>You can connect with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, <a href="https://www.threads.net/@taridesltd">Threads</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up for our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

