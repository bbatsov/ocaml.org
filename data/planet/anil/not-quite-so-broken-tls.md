---
title: Not-Quite-So-Broken TLS
description: Paper on rebuilding TLS securely but practically presented at USENIX
  Security 2015.
url: https://anil.recoil.org/notes/2015-usenixsec-nqsb-1
date: 2015-08-01T00:00:00-00:00
preview_image: https://anil.recoil.org/images/papers/2015-usenixsec-nqsb.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>Paper on rebuilding TLS securely but practically at USENIX Security 2015, presented with David Kaloper-Mersinjak, Hannes Mehnert and Peter Sewell. Not-Quite-So-Broken TLS was a from-scratch implementation in OCaml that served dual roles: both an executable specification for testing other TLS implementations and a production-ready library. By using a memory-safe language and modular design, we excluded entire classes of security vulnerabilities by construction. The implementation achieved reasonable performance (73-84% of OpenSSL throughput) despite the safety guarantees, and could be compiled into tiny Xen unikernels with a 4% TCB compared to Linux/OpenSSL stacks.</p>

