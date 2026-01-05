---
title: Conex, securing the opam-repository, is now in production
description:
url: https://hannes.robur.coop/Posts/ConexRunning
date: 2025-11-06T13:29:33-00:00
preview_image:
authors:
- hannes
source:
---

<p>TL;DR: The long-awaited conex timestamp and snapshot service is now in beta. Test it, we need feedback :)</p>
<h2>Introduction</h2>
<p>The community-maintained <a href="https://github.com/ocaml/opam-repository">opam-repository</a> is a git repository hosted on GitHub, which dozens people and bots have write access to. The <a href="https://opam.ocaml.org">opam.ocaml.org</a> servers, which deliver the opam-repository by default, are operated on scaleway machines -- and we couldn't find any documentation of how this service is run and who has which privileges on it. The opam-repository contains package metadata in the form of package names, their dependencies, and their source tarballs with sha256 (or sha512) checksums of these. An attacker could either modify the GitHub repository (e.g. changing a checksum, introduce a dependency, introduce an upper bound to a vulnerable version, ...); the opam.ocaml.org servers; or sit on the network connection between you and the server and prevent any update. Your opam client won't notice this.</p>
<p><a href="https://github.com/robur-coop/conex">Conex</a> is a security system and implementation, based on <a href="https://theupdateframework.io/">tuf</a>, to prevent a lot of attack vectors (see <a href="https://theupdateframework.github.io/specification/latest/#goals-to-protect-against-specific-attacks">TUF attacks</a> to get an overview). We're proud that after a decade of our initial announcement we got it into shape.</p>
<p>What we announce here is only the first minimal viable version: a service that serves the opam-repository, updating hourly, and signing this repository. The timestamp service signs the repository every 5 minutes. So, security-wise this means: instead of trusting the opam.ocaml.org servers, you have to trust the opam-mirror, and <a href="https://robur.coop">us at Robur</a> for running it without introducing malicious data.</p>
<p>With this minimal version, we're not yet at a stage where OCaml package authors sign their releases, but we're making good progress towards securing the opam-repository. For the end-to-end signing we need to figure out workflows, especially taking the amazing work of the opam-repository maintainers into account.</p>
<h2>Security</h2>
<p>As mentioned, the <em>conex.robur.coop</em> service is run as a MirageOS unikernel by Robur. The <a href="https://git.robur.coop/robur/opam-mirror/src/branch/conex">source code is public</a>. The running binary is <a href="https://builds.robur.coop/job/conex-opam-mirror/">available from our reproducible build infrastructure</a>.</p>
<p>What we prevent in this version are <em>Fast-forward attacks</em>: an attacker who compromised signing keys can increase the version to the maximum value. This means a client would not be able to update to a version recovering from the key compromise. We implemented the <em>epoch</em> system, which requires a more privileged key to sign, and can recover from that maximum version. We can also rotate keys when they are compromised.</p>
<p>We also make <em>Indefinite freeze attacks</em> noticable: the timestamp service signs every 5 minutes its data -- so if your client receives a signature that is outdated, this can be detected.</p>
<p>The <em>Mix-and-match attacks</em> are avoided by our snapshot service, which signs all signatures of the opam-repository (at the moment, we only have a single maintainer key). This means an attacker cannot provide you with old metadata of package "a" and newer metadata of package "b".</p>
<p><em>Rollback attacks</em> are also protected, since we have version numbers in the signing metadata, and any version decrement (or modification without increment) leads to a verification failure.</p>
<p><em>Vulnerability to key compromises</em> are avoided, since we can rotate keys. The root key is kept offline.</p>
<p>If you have any questions about the security implications, please reach out.</p>
<h2>Setup your conex repo</h2>
<p>To setup opam with conex on your machine, you need to install conex (and/or conex-mirage-crypto, see below) - currently only available as opam package (sorry about the bootstrap problem - hopefully it will be integrated into opam at some time).</p>
<p>Then, in your "~/.opam/config" you need to specify the <a href="https://opam.ocaml.org/doc/Manual.html#configfield-repository-validation-command">repository-validation-command</a>:</p>
<pre><code>repository-validation-command: [
  "/path/to/conex_verify_mirage_crypto"
  "--quorum=%{quorum}%"
  "--trust-anchors=%{anchors}%"
  "--repository=%{repo}%" {incremental}
  "--dir=%{dir}%" {!incremental}
  "--patch=%{patch}%" {incremental}
  "--incremental" {incremental}
]
</code></pre>
<p>Instead of "conex_verify_mirage_crypto", you can as well use "conex_verify_openssl" (fewer dependencies, calls out to OpenSSL as the cryptographic provider, is slower).</p>
<p>Then you can run <code>opam repo add conex-robur https://conex.robur.coop 1 sha256=ad5eb0e4a77abfbc6c1bb5661eba46049404e0222588dd059c87f12436d41a28</code>. Thereafter you can <code>opam repo remove default</code>, and then you're only using signed metadata. The "1" is the quorum of root keys for signatures to be valid, the "sha256=ad5eb0e4a77abfbc6c1bb5661eba46049404e0222588dd059c87f12436d41a28" is the hash of the public root key.</p>
<h2>Set it up yourself</h2>
<p>You can as well run your own conex opam-repository. You can either download the binary from above, or compile from source:</p>
<pre><code class="language-shell">$ git clone git://git.robur.coop/robur/opam-mirror -b conex
$ cd opam-mirror/mirage
$ mirage configure -t &lt;my-target&gt; # use unix, hvt, spt, xen, virtio, ...
$ make
</code></pre>
<h3>Setup cryptographic keys</h3>
<p>The next step is to setup cryptographic keys:</p>
<pre><code class="language-shell"># generate root key, and root file
$ conex_key --id root
$ conex_root create &amp;&amp; conex_root add-key --id root

# generate maintainer key, and add it to the root file
$ conex_key --id maintainer
$ conex_root add-to-role --role maintainer --id maintainer

# generate snapshot key, and add it to the root file
$ conex_key --id snapshot
$ conex_root add-to-role --role snapshot --id snapshot

# generate timestamp key, and add it to the root file
$ conex_key --id timestamp
$ conex_root add-to-role --role timestamp --id timestamp

# sign the root file with the root key
$ conex_root sign --id root

# truncate the root file (needed for the unikernel to be a multiple of 512 bytes)
$ truncate -s 2k root
</code></pre>
<h3>Start the unikernel</h3>
<p>The unikernel needs another block device (scratch space) to store data to persist across reboots.</p>
<pre><code class="language-shell">$ truncate -s 2G conex-data
</code></pre>
<p>Now you can run the unikernel, first it should initialize the block device (please keep in mind this is for the hvt version of the unikernel, other deployment targets need other command lines):</p>
<pre><code class="language-shell">$ solo5-hvt --net:service=tap0 --block:tar=conex-data --block:root=root -- dist/mirror.hvt --initialize-disk --index-size=100
</code></pre>
<p>And then run the unikernel for good, it will start cloning the opam-repository from GitHub, and add signatures:</p>
<pre><code class="language-shell">$ solo5-hvt --net:service=tap0 --block:tar=conex-data --block:root=root -- dist/mirror.hvt --ipv4=10.0.42.2/24 --ipv4-gateway=10.0.42.1 --skip-download --target-key="..." --snapshot-key="..." --timestamp-key="..."
</code></pre>
<p>Where the keys you provide is the raw data from "~/.conex/timestamp*" -- only the middle line (not the "----- BEGIN PRIVATE KEY-----" and "----- END PRIVATE KEY-----" lines).</p>
<h2>Conclusion and future</h2>
<p>We delivered the minimal version of conex, and are keen to receive feedback. We are also keen to push conex further, and allow opam package authors to sign their releases. This requires additional code, but also documenting and adapting workflows of opam-repository maintainers. We will also integrate conex signatures into commonly used publication tools <a href="https://github.com/ocaml-opam/opam-publish/">opam-publish</a> and <a href="https://github.com/tarides/dune-release">dune-release</a>.</p>
<p>The development of conex is supported by the <a href="https://ocaml-sf.org/">OCaml Software Foundation</a> and by <a href="https://ahrefs.com/">ahrefs</a>.</p>

