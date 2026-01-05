---
title: Combining Static Model Checking with Dynamic Enforcement Using the Statecall
  Policy Language
description: Paper on DSL for specifying temporal protocol automata presented at ICFEM
  2009.
url: https://anil.recoil.org/notes/2009-icfem-spl-1
date: 2009-11-01T00:00:00-00:00
preview_image: https://anil.recoil.org/images/papers/2009-icfem-spl.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Paper on a DSL for specifying temporal protocol automata at ICFEM 2009. The Statecall Policy Language (SPL) provides a practical middle ground between ad-hoc coding and full formal verification for complex protocol implementations. SPL lets programmers embed automata directly in their code which can be both statically model-checked using SPIN and dynamically enforced at runtime with minimal performance overhead. I demonstrated the approach with an SSH server written entirely in OCaml/SPL, showing how the automata provide higher-level debugging capabilities while maintaining the benefits of formal verification.</p>

