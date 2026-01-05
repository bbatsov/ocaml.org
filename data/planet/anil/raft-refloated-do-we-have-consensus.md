---
title: 'Raft Refloated: Do We Have Consensus?'
description: Paper reproducing Raft consensus protocol with optimizations and empirical
  validation using clean-slate OCaml implementation.
url: https://anil.recoil.org/notes/2014-sigops-raft-1
date: 2015-01-01T00:00:00-00:00
preview_image: https://anil.recoil.org/images/papers/2014-sigops-raft.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Paper on reproducing the Raft consensus protocol, published in ACM SIGOPS Operating Systems Review with Heidi Howard, Malte Schwarzkopf and Jon Crowcroft. We developed a clean-slate OCaml implementation of Raft and built an event-driven simulation framework to test it. The paper proposed several optimizations to the protocol and empirically validated Raft's correctness invariants, while also evaluating whether it lived up to its claims of being more understandable than Paxos.</p>

