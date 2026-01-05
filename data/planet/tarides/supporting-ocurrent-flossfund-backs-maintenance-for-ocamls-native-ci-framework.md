---
title: 'Supporting OCurrent: FLOSS/Fund Backs Maintenance for OCaml''s Native CI Framework'
description: Announcing FLOSS Fund's support for maintenance of OCaml's native CI
  and workflow framework OCurrent
url: https://tarides.com/blog/2025-10-30-supporting-ocurrent-floss-fund-backs-maintenance-for-ocaml-s-native-ci-framework
date: 2025-10-30T00:00:00-00:00
preview_image: https://tarides.com/blog/images/floss-fund-1360w.webp
authors:
- Tarides
source:
---

<p>We’re pleased to share that <a href="https://floss.fund">FLOSS/Fund</a> has provided Tarides with a grant to support the ongoing maintenance of <strong>OCurrent</strong>, the OCaml-native CI and workflow framework. This support is part of the <a href="https://floss.fund/blog/second-tranche-2025-anniversary/">second tranche of FLOSS Fund’s 2025 round</a>, and it will help us focus on ensuring OCurrent’s stability, improving its infrastructure, strengthening its documentation, and continuing community support.</p>
<h2>What is OCurrent?</h2>
<p><a href="https://www.ocurrent.org/">OCurrent</a> is both a small embedded domain-specific language (eDSL) for describing workflows and pipelines, and the wider CI infrastructure that powers much of the OCaml ecosystem. The OCurrent library enables developers to express build, test, and deployment logic directly in OCaml, with automatic dependency tracking and selective re-execution of steps when inputs change. This makes it particularly well-suited for long-running, continuously evolving systems where correctness and reproducibility are key. Pipelines written in OCurrent are self-adjusting: when a Git branch, Docker image, or external dependency is updated, the pipeline reacts automatically.</p>
<p>OCurrent underpins the continuous integration and build infrastructure for the OCaml community. The <a href="https://github.com/ocurrent/ocaml-ci">ocaml-ci</a> service, which provides CI for OCaml projects hosted on GitHub, is implemented using OCurrent. Similarly, <a href="https://github.com/ocurrent/opam-repo-ci">opam-repo-ci</a> tests submissions to the <code>opam</code> repository. The same infrastructure is used for building and maintaining <a href="https://hub.docker.com/r/ocaml/opam">Docker base images</a> and services like <em>ocaml-docs-ci</em> and <em>ocaml-multicore-ci</em>. Together, they keep the OCaml ecosystem’s packages, documentation, and base environments current across compiler versions, architectures, and operating systems.</p>
<h2>How Will We Use the FLOSS Fund Grant?</h2>
<p>This grant will enable us to dedicate time to maintaining OCurrent’s core components and the CI infrastructure, enhancing their performance and reliability, and refining the developer experience. The funding will help sustain the infrastructure that powers <em>ocaml-ci</em> and <em>opam-repo-ci</em>, as well as the automated build and deployment pipelines that many OCaml projects rely on.</p>
<p>For an ecosystem like OCaml’s, with its diversity of compiler versions, platforms, and tooling, having a reliable and type-safe pipeline engine is crucial. OCurrent enables reproducible builds and continuous integration, reducing friction for developers and ensuring that the ecosystem remains healthy and up to date. As described in <a href="https://tarides.com/blog/2023-07-12-ocaml-ci-renovated/">our earlier post on the renovated ocaml-ci</a>, OCurrent powers the “zero-configuration” CI experience that has become a cornerstone of OCaml’s development workflow.</p>
<h2>A Note of Thanks</h2>
<p>We’re grateful to FLOSS Fund for recognising the importance of this work and for supporting the continued development of the open-source infrastructure that keeps OCaml projects running smoothly. Thanks also to the many contributors, users, and maintainers of OCurrent, <em>ocaml-ci</em>, <em>opam-repo-ci</em>, and related projects. Your participation and feedback continue to shape the future of the OCaml ecosystem.</p>
<p>You can connect with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, <a href="https://www.threads.net/@taridesltd">Threads</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up for our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

