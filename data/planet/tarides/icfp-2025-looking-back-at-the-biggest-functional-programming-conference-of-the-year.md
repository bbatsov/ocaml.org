---
title: 'ICFP 2025: Looking Back at the Biggest Functional Programming Conference of
  the Year'
description: Check out our summary of everything OCaml at ICFP 2025, including insights
  from the Tarides engineers who attended!
url: https://tarides.com/blog/2025-12-04-icfp-2025-looking-back-at-the-biggest-functional-programming-conference-of-the-year
date: 2025-12-04T00:00:00-00:00
preview_image: https://tarides.com/blog/images/singapore-icfp-1360w.webp
authors:
- Tarides
source:
---

<p>The biggest functional programming conference on the calendar took place in Singapore from October 12 to 18 this year. With a jam-packed schedule featuring talks, workshops, and events, ICFP 2025 brought together passionate functional programming developers from around the world. As an active contributor to the OCaml ecosystem, Tarides is committed to supporting the growth of this vibrant community and exploring new applications for OCaml. We look forward to discusing, presenting, and discovering the latest developments in the language at the conferece every year.</p>
<p>If you didn’t get the chance to go this year, or if you just want to relive it, this post has you covered! We’re going to take a look at everything OCaml at the conference – from OxCaml to improved editors and establishing a community code of conduct  – and hear first-hand from some of our engineers who attended.</p>
<p>Next year’s ICFP is in Indianapolis, and you should keep an eye <a href="https://icfp26.sigplan.org/">on the website</a> to register and submit your papers.</p>
<h2>Experience Reports</h2>
<p>Several of my colleagues attended ICFP, and just like <a href="https://tarides.com/blog/2024-10-23-looking-back-on-our-experience-at-icfp/">last year</a>, I asked them a bunch of questions about their experience when they returned! Firstly, I asked them all why they enjoy ICFP and why they choose to attend it year after year. David Allsopp highlighted the social networking opportunities: “For those of us lucky enough to be able to attend them, conferences grease the social gears of collaboration”. As an example, David mentioned how a talk on range analysis in Standard ML inspired him and Ryan Gibb (who incidentally work next door to each other in Cambridge) to start hacking on package-management solving right then and there! Read more about <a href="https://www.dra27.uk/blog/platform/2025/10/18/icfp-2025.html">David’s time at ICFP in his blog post</a>.</p>
<p>Sudha Parimala explained that “ICFP has many different tracks that are highly relevant to Functional Programming in general and also to OCaml. It's a good place to meet folks working on similar things and discuss ideas.” In addition to the various tracks, ICFP also hosts events that provide opportunities to meet new people. “Even though I couldn't make it this time, I'd highly recommend the ‘Women in PL’ dinner hosted at ICFP. It gives you a good platform to connect with other women programmers and researchers.” If you’re attending ICFP next year, keep an eye out for these organised events.</p>
<p>Speaking of next year, the submission deadline for papers is on February 19, so, in the words of Sudha: “If you're working on cool OCaml stuff, consider submitting it to the OCaml workshop next year!” and we look forward to seeing you there!</p>
<h2>The OCaml Workshop and Talks</h2>
<p>Let’s dive into everything OCaml (and some exciting honourable mentions!) that happened at ICFP this year. Where possible, I’ve linked the recordings of the talks so you can recreate the conference experience from home!</p>
<p>The OCaml Workshop was great, as usual, and took place on October 17. The following talks were presented by developers of many different affiliations, including IIT Madras, NIT Trichy, Tarides, the University of Illinois Urbana-Champaign, Jane Street, Bloomberg, and the University of Cambridge.</p>
<h3>OCaml Workshop Talks</h3>
<ul>
<li>
<p><a href="https://www.youtube.com/watch?v=ekbLoe24iXg">A Mechanically Verified Garbage Collector for OCaml</a> by Sheera Shamsu, Dipesh Kafle, Dhruv Maroo, Kartik Nagar, Karthikeyan Bhargavan, and KC Sivaramakrishnan. OCaml’s garbage collector (GC) is crucial to the functioning of its runtime system and overall correctness and safety. This talk shared details about a project to create a correct, proof-oriented GC that can evolve with the language over time.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=cgpnBdXsW2c">OCaml Package Management with (only!) Dune</a> by Stephen Sherratt, Marek Kubica, and Rudi Grinberg. This talk showcased how developers can now use Dune to download and install packages without relying on any other tool (like <code>opam</code>).</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=CIS_ljgqSQw">How the OCaml Community Established Its Code of Conduct</a> by Sudha Parimala. As Sudha said, “OCaml has had a thriving open source community for almost three decades now, but a code of conduct was only introduced in 2022.” Her talk described the process behind establishing that code of conduct, including creating the team, gathering feedback, and lessons learned.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=m_fBmuoGwtM">Embedding WebAssembly in OCaml for Safe Program Construction</a> by Hunter DeMeyer. This talk proposed WasML: a library enforcing syntactic and semantic constraints in <code>wasm</code> programs via OCaml types.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=Kqs5DXwcyVI">smaws: An AWS SDK for OCaml</a> by Chris Armstrong. Introduced a new Amazon Web Services-based software development kit for OCaml, <code>smaws</code>, exploring the challenges and design choices behind the new library.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=PekeGxGlc3Q">Toward a More Secure OCaml Ecosystem</a> by Maksim Grankin. All about the new OCaml Security Team, launched by the OCaml Software Foundation (OCSF), the motivations behind creating the team, its goals, and the security challenges facing language ecosystems today.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=Ub8k1BcSRLQ">Three Steps for OCaml to Crest the AI Humps</a> by Sadiq Jaffer, Jonathan Ludlam, Ryan Gibb, Thomas Gazagnaire, and Anil Madhavapeddy. The talk looked at how well-represented OCaml is amongst open-weight models and what could make it stand out, outlining potential changes the ecosystem can make to work better with AI coding assistants.</p>
<p>In his <a href="https://toao.com/blog/ai-existential-ocaml">blog post about the talk</a>, Sadiq described how “The gap between coding agent performance on mainstream versus niche languages could prove fatal to smaller language communities. New developers increasingly judge a programming language not just on its traditional tooling (compilers, debuggers, libraries) but also on how well AI coding agents support it. If agents struggle with OCaml, fewer new developers will choose to learn it, creating a vicious cycle.”</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=q4oEKMTeXk4">A New Era of OCaml Editing: Powered by Merlin, Delivered via LSP</a> by Xavier Van de Woestyne, Sonja Heinze, Ulysse Gérard, and Muluh Godson. Explored how the OCaml ecosystem is managing the maintenance burden resulting from an increasing number of features available for editors by adopting the generic open standard Language Server Protocol (LSP). In his own words, Xavier said that “I presented some work carried out in the name of ‘ecumenism’ for OCaml editor support! How to make Merlin and OCaml-LSP-server interact to reduce the need for (heavy) maintenance between the wide variety of editors. We had to walk the line between minimising maintenance without losing functionality. The entire work is described in the paper: <a href="https://conf.researchr.org/details/icfp-splash-2025/ocaml-2025-papers/7/A-New-Era-of-OCaml-Editing-Powered-by-Merlin-Delivered-via-LSP">A New Era of OCaml Editing Powered by Merlin, Delivered via LSP</a>”.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=It0i9xFqCtE">Taming the Flat Float Array Optimisation: Tracking Separability in the Type System</a> by Diana Kalinichenko and Richard A. Eisenberg. This talk presented an approach to flat float array optimisation that is used in <a href="https://oxcaml.org/">OxCaml</a>, which enabled previously rejected non-separable types to be used while maintaining compatibility with existing code and unlocking new optimisations for arrays of known non-float types.</p>
</li>
</ul>
<p>Of course, there are more talks at the conference outside of the workshop, and this year, attendees were spoiled for choice with plenty of OCaml topics to explore!</p>
<h3>Miscellaneous OCaml Talks</h3>
<ul>
<li>
<p><a href="https://www.youtube.com/watch?v=qTSpEKZohKY">A Guided Tour Through Oxidised OCaml</a> by Gavin Gray, Anil Madhavapeddy, KC Sivaramakrishnan, Will Crichton, Shriram Krishnamurthi, Chris Casinghino, and Richard A. Eisenberg. This collaborative tutorial was a combined effort by members from Brown University, the University of Cambridge, Jane Street, IIT Madras, and Tarides. It took participants on a tour of the most significant extensions of OxCaml, Jane Street’s production compiler for performance-oriented programming, including fearless concurrency, data layout, and location control. The creators of the workshop encourage you to work through the <a href="https://gavinleroy.com/oxcaml-tutorial-icfp25/">slides online</a>, try the <a href="https://github.com/oxcaml/tutorial-icfp25">activities</a>, and give them feedback by filling in the <a href="https://gavinleroy.com/oxcaml-icfp-activity/">quiz</a>.</p>
<p>In his <a href="https://anil.recoil.org/notes/icfp25-oxcaml">blog post on the tutorial</a>, Anil shared that “the tutorial itself [...] went fantastically! Both sessions were completely full, with participants online as well”.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=1_Y6hmj-VpY&amp;t=1s">Functional Networking for Millions of Docker Desktops (Experience Report)</a> by Anil Madhavapeddy, David J. Scott, Patrick Ferris, Ryan Gibb, and Thomas Gazagnaire from the University of Cambridge, Docker, and Tarides at the ‘Applications and SRC Talks’. This talk reflected on a decade of functional OCaml code in production as part of Docker’s container architecture across millions of desktops.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=AxiqBFf4zqg">Implicit Modules, a Middle Step Towards Modular Implicits</a> by Samuel Vivien and Didier Rémy from Inria and PSL in the ML Family Workshop. Described a new proposal to extend OCaml with implicit module arguments, a long-term project with the first step, modular explicits, about to be integrated with mainstream OCaml.</p>
<p>This was one of Xavier’s most memorable talks of the conference, and he described it as “A careful presentation that outlined the status of the project, what has been done, what remains to be done, and which is very promising for the future of OCaml!”</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=abDWZ9D8kEE">The Wild West of Post-POSIX IO Interfaces</a> by Anil Madhavapeddy from the University of Cambridge as a keynote speech in the Language Semantics &amp; Type Systems track. The talk tracked the evolution of asynchronous and shared-memory I/O, from the POSIX programming model to OCaml 5’s Eio library.</p>
<p><a href="https://anil.recoil.org/notes/icfp25-post-posix">Anil distilled the most important part of his talk</a> as follows:  “So I made one key argument to the audience: it's time to accept that standards such as POSIX are now holding back the development of good language runtimes, and we need to embrace the diversity of highly concurrent, shared-memory interfaces.”</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=5pQcX3XMhhE">Generating Hazel Programs From Ill-Typed OCaml Programs</a> by Patrick Ferris and Anil Madhavapeddy from the University of Cambridge at the Type-Driven Development workshop. Highlighted the work on Hazel, a young programming language, and the development of a compiler to Hazel that can generate ill-typed code to help its development and testing.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=tg-5liTtCe4">OCaml Blockly</a>  by Kenichi Asai of Ochanomizu University from the presentations for papers in the Journal of Functional Programming. This talk introduced OCaml Blockly, which is a block-based programming environment based on Google Blockly, but for OCaml. It is useful because it knows the scoping and type rules of OCaml: meaning that for any complete program in OCaml Blockly, its OCaml counterpart will compile successfully.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=5y-SGiA8ycs">Formal Semantics and Program Logics for a Fragment of OCaml</a> by Remy Seassau, Irene Yoon, Jean-Marie Madiot, and François Pottier from Inria in the ICFP Papers track. Their talk shared work towards a formal definition of OCaml and a foundational program verification environment for the language by presenting a formal definition of a nontrivial sequential fragment of OCaml: OLang.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=b0SIWDdmRVU">A Tale of Two Lambdas: A Haskeller’s Journey Into OCaml</a> by Richard A. Eisenberg from Jane Street at the Haskell track. Richard shared his experience with both the Haskell and OCaml ecosystems and communities, offering insights on how they are similar, how they are different and, most importantly, what they can learn from each other.</p>
<p>This talk was a highlight for both Anil and David! Anil wrote in his <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">blog post series</a> that “This year, Richard Eisenberg was my absolute highlight with a superb session on what he's learnt from being the rare breed of someone steeped deeply both in Haskell and OCaml. The room was so packed for this talk that they had to create an overflow room streaming it in the corridors!” and David enjoyed the entire Haskell track, saying “Haskell did very nicely this year, with Richard Eisenberg’s likewise excellent keynote on the Friday”.</p>
</li>
</ul>
<p>In addition to all the amazing OCaml talks, there were, of course, plenty of presentations on other topics and programming languages!</p>
<h3>Honourable Mentions: Non-OCaml Talks</h3>
<ul>
<li>
<p><a href="https://www.youtube.com/live/IIRJeleXeuU?si=F1fY7arkGJpm44mB">Programming for the Planet</a>. This was the second ever PROPL workshop after the first one was held in London last year. It hosted several talks centred around using functional programming methods to address challenges to biodiversity, climate, and the vast amounts of planetary data that the sector yields. <a href="https://anil.recoil.org/notes/icfp25-propl">Anil shared that</a> “By far my favourite aspect of PROPL was the sheer ambition on display when it comes to leveraging the network effects around computer technology to accelerate the pace of environmental action”.</p>
</li>
<li>
<p><a href="https://conf.researchr.org/details/icfp-splash-2025/olivierfest-2025-papers/7/Continuations-in-Music">Continuations in Music</a>  by Youyou Cong at ‘Continuations at Work’ explored the usefulness of continuations in computer programming languages and music. David thought it was “thought-provoking, and potentially (even) more to go further back in musical history to late mediaeval and early Tudor, where the pattern work of composition and imitative properties are even more abundant (continuations with combinators for inversion, canon, etc.)”.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=elhNJyLgjDU">Polynomial-Time Program Equivalence for Machine Knitting</a> by Nathan Hurtig, Jenny Han Lin, Thomas S. Price, Adriana Schulz, James McCann, and Gilbert Bernstein from the University of Washington, University of Utah, and Carnegie Mellon University. Presented their algorithm at the ‘Applications and SRC Talks’ to canonicalise algebraic representations of the topological semantics of machine-knitting programs. David commented that it stood out to him “not because I can follow the category theory, but because it was a beautiful exposition”.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=9S_VY_PVnZk">Join Points in Practice</a> was a keynote by Simon Peyton Jones all about join points as a powerful optimisation tool. Both Xavier and David especially emphasised how enjoyable Simon’s presentation style is: “One should always attend a Simon P-J talk, and his packed Haskell keynote was no exception”, David said, and Xavier commented: “His presentation, his gift of the gab, and his enthusiasm make him the most impressive and enjoyable speaker I have ever seen. So seeing him live was a must for me, and I wasn't disappointed by the delivery”.</p>
</li>
<li>
<p><a href="https://www.youtube.com/watch?v=dtMOicQfUtI">Freer Arrows and Why You Need Them in Haskell</a> by Grant VanDomelen, Gan Shen, Lindsey Kuper, and Yao Li at the Haskell track. This talk discussed freer arrows, an expressive structure that is amenable to static analysis. Xavier said: “I use arrows extensively for my website, and I thought it was great to see their defunctionalised (freer) representation with very clear examples!”</p>
</li>
<li>
<p><a href="https://www.youtube.com/live/F_7S90vFEsE?si=4q2joS4bcglm_rWd&amp;t=1">Functional Art, Music, Modelling and Design (FARM)</a> This workshop welcomed submissions across art, crafting, and design, and hosted talks on music, vector graphics drawing, and even coats of arms! My colleague Stephen Sherratt attended the workshop and presented the talk <a href="https://www.youtube.com/watch?v=TNYRrrc2J0Y">Software-defined Declarative Synthesiser Live-Coding in a Jupyter Notebook</a>, where he generated music in real-time by writing Rust code into a Jupyter notebook. He also particularly enjoyed the talks <a href="https://youtu.be/XW9Gc5UGCvM?si=GEi8ZZ0M3bKT2ZYG&amp;t=14254">Weft – Enabling Tidal on the Web</a> by Matthew Kaney and William Payne, and <a href="https://www.youtube.com/watch?v=PTkj7LM5QXI">Generalising Turtle Geometry: An Extensible Language for Vector Graphics Drawing</a> by Alice Rixte.</p>
</li>
</ul>
<h2>ICFP 2026: See You Next Year!</h2>
<p>As mentioned above, next year’s ICFP will take place in Indianapolis, Indiana, from August 24 to 29. It will be another great opportunity to meet developers, discuss everything functional programming and OCaml, and stay up to date with the latest changes and releases. Keep an eye <a href="https://icfp26.sigplan.org/">on the website</a> and <a href="https://bsky.app/profile/icfp-conference.bsky.social">ICFP’s socials</a> to register, and, of course, submit your talks before the February deadline. We hope to see you there!</p>
<p>You can connect with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, <a href="https://www.threads.net/@taridesltd">Threads</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up for our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

