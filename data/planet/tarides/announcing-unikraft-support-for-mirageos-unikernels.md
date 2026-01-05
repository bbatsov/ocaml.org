---
title: Announcing Unikraft Support for MirageOS Unikernels
description: Get an overview of the Unikraft backend support for MirageOS, including
  performance benchmarks on different devices.
url: https://tarides.com/blog/2025-11-13-announcing-unikraft-support-for-mirageos-unikernels
date: 2025-11-13T00:00:00-00:00
preview_image: https://tarides.com/blog/images/unikernels-3-1360w.webp
authors:
- Tarides
source:
---

<p>We are happy to announce the release of Unikraft backend support for MirageOS unikernels! Our team, consisting of Fabrice Buoro, Virgile Robles, Nicolas Osborne, and me (Samuel Hym), have been working on this feature for the past year. We’re excited to bring it to the community and hope to see many people try it out.</p>
<p>This post will give you an overview of the release, including some background on Unikraft and performance graphs. You will already be familiar with most of this if you read our <a href="https://discuss.ocaml.org/t/mirageos-on-unikraft/16975">post on Discuss</a>.</p>
<h2>What is Unikraft and Why Did We Choose It?</h2>
<p><a href="https://unikraft.org/">Unikraft</a> is a unikernel development kit: a large <a href="https://unikraft.org/docs/internals/architecture">collection of components</a> that can be combined in any configuration that the user wants in the unikernel tradition of modularity. Unikraft's scope is larger than that of <a href="https://github.com/Solo5/solo5/">Solo5</a> (a backend that <a href="https://tarides.com/blog/2025-02-06-mirageos-on-ocaml-5/">MirageOS also supports</a>). It aims to make it easy to turn any Unix server into an efficient unikernel.</p>
<p>In fact, the initial motivation behind exploring Unikraft as a MirageOS backend was to experiment and see what performance levels we could reach. We were particularly excited about using their Virtio-based network interface, as <code>virtio</code> is currently only implemented for one specific x86_64-only backend in Solo5. Some of the immediate performance differences we observed are detailed further down, but performance is not all we hope to gain from the Unikraft backend in the long run.</p>
<p>Unikraft is on the road to being multicore compatible (i.e., having one unikernel using multiple cores). While this is not ready yet and significant effort is needed to get there, it means that the MirageOS backend will eventually benefit from these efforts and support the full set of OCaml 5 features.</p>
<p>Furthermore, the Unikraft community (which is quite active) is experimenting with a variety of other targets, such as bare-metal for some platforms or new hypervisors (e.g. seL4). Any new target Unikraft supports can then be supported ‘for free’ by MirageOS too. For example, the Unikraft backend has already resulted in Firecracker being a new supported virtual machine monitor (VMM) for MirageOS.</p>
<p>Lastly, since Unikraft is POSIX-compatible (for a large subset of syscalls), it has the potential to enable MirageOS unikernels to embed OCaml libraries that have not been ported yet. This compatibility could be especially useful for large libraries which are hard to port (<a href="https://ocaml.xyz/">owl</a> comes to mind).</p>
<h2>How Does Unikraft Support Work?</h2>
<p>Adding the new MirageOS backend required that we create or modify a series of components:</p>
<ul>
<li>An <a href="https://github.com/mirage/ocaml-unikraft">OCaml cross compiler</a> that can build the new backend by building its corresponding runtime and providing a way to build unikernel images (instead of normal executables).</li>
<li>New libraries for <a href="https://github.com/mirage/mirage-unikraft">Unikraft system support</a>, including its <a href="https://github.com/mirage/mirage-net-unikraft">network</a> and <a href="https://github.com/mirage/mirage-block-unikraft">block</a> devices.</li>
<li><a href="https://github.com/mirage/mirage/pull/1607">Support for the new backends</a> in the <code>mirage</code> tool.</li>
</ul>
<p>Using Unikraft with a QEMU or a Firecracker backend is as simple as choosing the <code>unikraft-qemu</code> or the <code>unikraft-firecracker</code> target when configuring a unikernel.</p>
<h3>The OCaml/Unikraft Cross Compiler</h3>
<p>To build the <a href="https://github.com/mirage/ocaml-unikraft">OCaml cross compiler</a> with Unikraft, we used the <a href="https://unikraft.org/">Unikraft</a> core, Unikraft <a href="https://github.com/mirage/unikraft-lib-musl">lib-musl</a>, and <a href="https://musl.libc.org/">musl</a>. The <a href="https://musl.libc.org/">musl</a> library is the C library recommended by Unikraft for building programs using the POSIX interface. The combination made it easy to build the OCaml 5 runtime, particularly because it provided an implementation of the <code>pthread</code> API, which is now used in many places in the runtime. It could also potentially make it easier to port some libraries that depend on <code>Unix</code> to work on Unikraft backends.</p>
<p>The OCaml cross compiler builds upon the work that has been upstreamed to ease the <a href="https://discuss.ocaml.org/t/building-an-ocaml-cross-compiler-with-ocaml-5-3/15918">creation of cross compilers</a>, using almost the same series of patches as for <code>ocaml-solo5</code>. So the only versions of the compiler that are currently supported for OCaml/Unikraft are OCaml 5.3 and 5.4. All the patches have been upstreamed to OCaml so there should no longer be any patches required by OCaml 5.5.</p>
<p>Note that we didn’t go with the full standard Unikraft POSIX stack, which includes <a href="https://savannah.nongnu.org/projects/lwip/">lwIP</a> to provide network support. We had a prototype at some point relying on <code>lwIP</code> to validate our progress on other building blocks, but it raised many incompatibility issues with the standard MirageOS network stack, so we dropped support for <code>lwIP</code> for now. Instead, we developed the libraries required to plug the MirageOS stacks into the low-level interfaces provided by the Unikraft core.</p>
<h3>The New MirageOS Libraries for Unikraft Support</h3>
<p>Unikraft support comes with packages using the standard names, <code>mirage-block-unikraft</code> and <code>mirage-net-unikraft</code>, to support the block and network devices. These libraries are implemented directly on top of the low-level Unikraft APIs, and therefore use <code>virtio</code> on both QEMU and Firecracker VMMs.</p>
<p>To evaluate the quality of the implementations for these devices, we ran a couple of small benchmarks using OCaml 5.3 and Unikraft 0.18.0. You can find the benchmarks (including the unikernels along with some scripts to set them up and run them) in the <code>benchmarks</code> directory in <a href="https://github.com/Firobe/mirage-skeleton/tree/benchmarks">@Firobe’s fork of mirage-skeleton, benchmarks branch</a>.</p>
<h4>Network Device</h4>
<p>To measure the performance of the network stack, we tweaked the simple <a href="https://github.com/mirage/mirage-skeleton/tree/main/device-usage/network">network skeleton</a> unikernel to compute statistics and used a variable number of clients all sending 512MB of null bytes. We have run this benchmark on both a couple of <code>x86_64</code> laptops and on an LX2160 <code>aarch64</code> board, all running a GNU/Linux OS.</p>
<p>We have observed a lot of variability in the performance of the <code>solo5-spt</code> unikernel (sometimes better, sometimes worse than <code>unikraft-qemu</code>) depending on the actual computer used, so these measurements should be taken with a grain of salt.</p>
<p>On two different <code>x86_64</code> laptops:
<img src="https://tarides.com/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft1-1360w~f2v_2IkapxtU_o7pASCW0w.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft1-170w~GO3z452NUPqB6z9mB8lzKQ.webp 170w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft1-340w~xj6FFSSwPSqo3R6udiFuNg.webp 340w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft1-680w~3TTzsVwszUkYqVxQLQsHqg.webp 680w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft1-1360w~f2v_2IkapxtU_o7pASCW0w.webp 1360w" alt="Network benchmark: Unikraft solo5-spt and solo5-hvt, in decreasing performance order">
<img src="https://tarides.com/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft2-1360w~SYj52Vo8QGKy0GSy04VasQ.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft2-170w~HM4BaGTvSQ98hUIvluAzfg.webp 170w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft2-340w~-Gzp8DJtNeMpJpFIyksJFQ.webp 340w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft2-680w~vtrb3v3a8859pd2oSsza4A.webp 680w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft2-1360w~SYj52Vo8QGKy0GSy04VasQ.webp 1360w" alt="Network benchmark: Unikraft or solo5-spt are fatest, depending on the number of connections, solo5-hvt slower"></p>
<p>On the LX2160 <code>aarch64</code> board:
<img src="https://tarides.com/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft3-1360w~hfvl6i_jK8lOOTXlV0XHqw.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft3-170w~G2yNOcNu_XKMT8pMtJq-xQ.webp 170w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft3-340w~igGIhY56MkXIdb4y0-MA5A.webp 340w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft3-680w~n2XEyuib35mVd2MyZOsFIQ.webp 680w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft3-1360w~hfvl6i_jK8lOOTXlV0XHqw.webp 1360w" alt="Network benchmark: Unikraft solo5-spt and solo5-hvt, in decreasing performance order"></p>
<h4>Block Device</h4>
<p>To measure the performance of the block devices, we wrote a simple unikernel copying data from one disk to another. We can see that the performance of <code>unikraft-qemu</code> is lower than <code>solo5-hvt</code> for small buffer sizes, but fortunately, the situation improves with larger buffer sizes. We only ran this benchmark on an <code>x86_64</code> laptop, as there is currently an <a href="https://github.com/unikraft/unikraft/issues/1622">issue with two block devices</a> on <code>aarch64</code> on Unikraft.
<img src="https://tarides.com/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft4-1360w~U5R7JpfSJ1Bis7jRAxouKw.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft4-170w~kd0STnLNSbxQa4o6q1DpKQ.webp 170w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft4-340w~hRVvEQX35rZKG04bHkaidA.webp 340w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft4-680w~h4RWil_ShQDulcM4llWbiQ.webp 680w, /blog/images/2025-09-10.unikraftmirageos/mirageos-unikraft4-1360w~U5R7JpfSJ1Bis7jRAxouKw.webp 1360w" alt="Block device benchmark: solo5-spt fatest on small buffer sizes, Unikraft-QEMU fatest on larger buffer sizes, Unikraft-Firecracker and solo5-hvt slower"></p>
<p>It is worth mentioning that I/Os can be parallelised, which also yields a significant performance boost. Indeed, <code>mirage-block-unikraft</code> can leverage the parallelised <code>virtio</code> backend of QEMU and Firecracker, which solves the problem of limiting I/Os to what the hardware supports in terms of both parallelism and sector size.</p>
<h3>Current Limitations</h3>
<ol>
<li>In our tests, only Linux appeared to be well supported for compiling Unikraft at the moment. As a result, we have restricted our packages to that OS for now.</li>
<li>Unikraft supports various backends. At the moment, we’ve only added support and tested its two major ones: QEMU and Firecracker.</li>
</ol>
<h2>Try it Out</h2>
<p>To try the new Unikraft backend for MirageOS, you need to use an OCaml 5.3 or 5.4 switch, so that you can install <code>mirage</code> and the OCaml/Unikraft cross compiler. The short version could be:</p>
<pre><code>$ opam switch create unikraft-test 5.4.0 # or 5.3.0, and if needed
$ opam install mirage ocaml-unikraft-backend-qemu ocaml-unikraft-x86_64
</code></pre>
<p>See below for some explanations about the numerous OCaml/Unikraft packages.
From then on, you can follow the standard procedure (see how to <a href="https://mirage.io/docs/install">install MirageOS</a> and how to <a href="https://mirage.io/docs/hello-world">build a hello-world unikernel</a>) to build your unikernel with the Unikraft backend of your choice, which should boil down to something like:</p>
<pre><code>$ mirage configure -t unikraft-qemu
$ make
</code></pre>
<h3>Details About the Various Packages for the OCaml/Unikraft Cross Compiler</h3>
<p>The <a href="https://github.com/mirage/ocaml-unikraft">OCaml cross compiler</a> to Unikraft is split up into 14 packages (see the <a href="https://github.com/ocaml/opam-repository/pull/27856">PR to opam-repository</a> for more details) so that users can:</p>
<ul>
<li>Choose which of the backends (QEMU or Firecracker) and which of the architectures (<code>x86_64</code> and <code>arm64</code>) they want to install, where all combinations can be installed at the same time.</li>
<li>Choose which architecture is generated when they use the <code>unikraft</code> OCamlfind toolchain by installing one of the two <code>ocaml-unikraft-default-&lt;arch&gt;</code> packages.</li>
<li>Install the <code>ocaml-unikraft-option-debug</code> to enable the (really verbose!) debugging messages.</li>
</ul>
<p>Furthermore, virtual packages can be installed to make sure that one of the architecture-specific packages is indeed installed:</p>
<ul>
<li><code>ocaml-unikraft</code> can be installed to make sure that there is indeed a <code>unikraft</code> OCamlfind toolchain installed.</li>
<li><code>ocaml-unikraft-backend-qemu</code> and <code>ocaml-unikraft-backend-firecracker</code> can be installed to make sure that the <code>unikraft</code> OCamlfind toolchain supports the corresponding backend.</li>
</ul>
<p>Those virtual packages will be used by the <code>mirage</code> tool when the target is <code>unikraft-qemu</code> or <code>unikraft-firecracker</code>.</p>
<p>All those packages use one of two version numbers. The backend packages use the Unikraft version number (0.18.0 and 0.20.0 have been tested and packaged) while the latest OCaml cross-compiler packages use version 1.1.0.</p>
<h2>Conclusion</h2>
<p>We are still experimenting with this new backend. We expect to run it in production in the coming months, but it may need improvements nevertheless. Notably absent from this release is an early attempt to leverage Unikraft’s POSIX compatibility to implement Mirage interfaces instead of hooking directly to Unikraft’s internal components. This early version used Unikraft’s <code>lwIP</code>-based network stack instead of Mirage’s (fooling Mirage into thinking it was running on Unix), and it may be interesting to revisit this kind of deployment, in particular for easy inclusion of Unix-only OCaml libraries in unikernels.</p>
<p>We are eager for reviews, comments, and discussion on the implementation, design, and approach of this new Mirage backend, and hope it will be useful to others.</p>
<p>You can connect with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, <a href="https://www.threads.net/@taridesltd">Threads</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up for our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

