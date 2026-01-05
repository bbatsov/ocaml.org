---
title: Linux mode setting, from the comfort of OCaml
description:
url: https://roscidus.com/blog/blog/2025/11/16/libdrm-ocaml/
date: 2025-11-16T09:00:00-00:00
preview_image:
authors:
- Thomas Leonard
source:
---

<p>Linux provides the KMS (Kernel Mode Setting) API to let applications query and configure display settings.
It's used by Wayland compositors and other programs that need to configure the hardware directly.
I found the C API a little verbose and hard to follow so I made <a href="https://github.com/talex5/libdrm-ocaml">libdrm-ocaml</a>,
which lets us run commands interactively in a REPL.</p>

<p>We'll start by discovering what hardware is available and how it's currently configured,
then configure a monitor to display a simple bitmap, and then finally render a 3D animation.
The post should be a useful introduction to KMS even if you don't know OCaml.</p>
<p>( this post also appeared on <a href="https://news.ycombinator.com/item?id=45947822">Hacker News</a> )</p>
<p><strong>Table of Contents</strong></p>
<ul>
<li><a href="https://roscidus.com/#running-it-yourself">Running it yourself</a>
</li>
<li><a href="https://roscidus.com/#querying-the-current-state">Querying the current state</a>
<ul>
<li><a href="https://roscidus.com/#finding-devices">Finding devices</a>
</li>
<li><a href="https://roscidus.com/#listing-resources">Listing resources</a>
</li>
<li><a href="https://roscidus.com/#connectors">Connectors</a>
</li>
<li><a href="https://roscidus.com/#modes">Modes</a>
</li>
<li><a href="https://roscidus.com/#properties">Properties</a>
</li>
<li><a href="https://roscidus.com/#encoders">Encoders</a>
</li>
<li><a href="https://roscidus.com/#crt-controllers">CRT Controllers</a>
</li>
<li><a href="https://roscidus.com/#framebuffers">Framebuffers</a>
</li>
<li><a href="https://roscidus.com/#crtc-planes">CRTC planes</a>
</li>
<li><a href="https://roscidus.com/#expanded-resources-diagram">Expanded resources diagram</a>
</li>
</ul>
</li>
<li><a href="https://roscidus.com/#making-changes">Making changes</a>
<ul>
<li><a href="https://roscidus.com/#non-atomic-mode-setting">Non-atomic mode setting</a>
</li>
<li><a href="https://roscidus.com/#dumb-buffers">Dumb buffers</a>
</li>
<li><a href="https://roscidus.com/#atomic-mode-setting">Atomic mode setting</a>
</li>
</ul>
</li>
<li><a href="https://roscidus.com/#d-rendering">3D rendering</a>
</li>
<li><a href="https://roscidus.com/#linux-vts">Linux VTs</a>
</li>
<li><a href="https://roscidus.com/#debugging">Debugging</a>
</li>
<li><a href="https://roscidus.com/#conclusions">Conclusions</a>
</li>
</ul>
<h2>Running it yourself</h2>
<p>If you want to follow along, you'll need to install <a href="https://github.com/talex5/libdrm-ocaml">libdrm-ocaml</a> and an interactive REPL like <a href="https://github.com/ocaml-community/utop">utop</a>.
With Nix, you can set everything up like this:</p>
<pre><code>git clone https://github.com/talex5/libdrm-ocaml
cd libdrm-ocaml
nix develop
dune utop
</code></pre>
<p>You should see a <code>utop #</code> prompt, where you can enter OCaml expressions.
Use <code>;;</code> to tell the REPL you've finished typing and it's time to evaluate, e.g.</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="mi">1</span><span class="o">+</span><span class="mi">1</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="kt">int</span> <span class="o">=</span> <span class="mi">2</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Alternatively, you can install things using <a href="https://opam.ocaml.org/">opam</a> (OCaml's package manager):</p>
<pre><code>opam install libdrm utop
utop
</code></pre>
<p>Then, at the utop prompt enter <code>#require "libdrm";;</code> (including the leading <code>#</code>).</p>
<h2>Querying the current state</h2>
<p>Before changing anything, we'll start by discovering what hardware is available.</p>
<p>I'll introduce the API as we go along, but you can check the <a href="https://talex5.github.io/libdrm-ocaml/libdrm/Drm/index.html">API reference docs</a>
if you want more information.</p>
<h3>Finding devices</h3>
<p>To list available graphics devices:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Device</span><span class="p">.</span><span class="n">list</span> <span class="bp">()</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Device</span><span class="p">.</span><span class="nn">Info</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line"><span class="o">[{</span><span class="n">primary_node</span> <span class="o">=</span> <span class="nc">Some</span> <span class="s2">"/dev/dri/card0"</span><span class="o">;</span>
</span><span class="line">  <span class="n">render_node</span> <span class="o">=</span> <span class="nc">Some</span> <span class="s2">"/dev/dri/renderD128"</span><span class="o">;</span>
</span><span class="line">  <span class="n">info</span> <span class="o">=</span> <span class="nc">PCI</span> <span class="o">{</span><span class="n">bus</span> <span class="o">=</span> <span class="o">{</span><span class="n">domain</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">bus</span> <span class="o">=</span> <span class="mi">1</span><span class="o">;</span> <span class="n">dev</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">func</span> <span class="o">=</span> <span class="mi">0</span><span class="o">};</span>
</span><span class="line">              <span class="n">dev</span> <span class="o">=</span> <span class="o">{</span><span class="n">vendor_id</span> <span class="o">=</span> <span class="mh">0x1002</span><span class="o">;</span>
</span><span class="line">                     <span class="n">device_id</span> <span class="o">=</span> <span class="mh">0x67ff</span><span class="o">;</span>
</span><span class="line">                     <span class="n">subvendor_id</span> <span class="o">=</span> <span class="mh">0x1458</span><span class="o">;</span>
</span><span class="line">                     <span class="n">subdevice_id</span> <span class="o">=</span> <span class="mh">0x230b</span><span class="o">;</span>
</span><span class="line">                     <span class="n">revision_id</span> <span class="o">=</span> <span class="mh">0xff</span><span class="o">}}}]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>libdrm scans the <code>/dev/dri/</code> directory looking for devices.
It uses <code>stat</code> to find the device major and minor numbers and uses the virtual <code>/sys</code> filesystem to get information about each one.
This is a PCI device, and the information corresponds to the values from <code>lspci</code>, e.g.</p>
<pre><code>$ lspci -nns 0:1:0.0
01:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI]
  Baffin [Radeon RX 550 640SP / RX 560/560X] [1002:67ff] (rev ff)
</code></pre>
<p>Each graphics device can have a <em>primary</em> and a <em>render</em> node.
The primary node gives full access to the device, including configuring monitors,
while the render node just allows applications to render scenes to memory.
In the <a href="https://roscidus.com/blog/blog/2025/09/20/ocaml-vulkan/">last post</a> I was using the render to node to create a 3D image,
and then sending it to the Wayland compositor for display.
This time we'll be doing the display ourselves, so we need to open the primary node:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">dev</span> <span class="o">=</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">openfile</span> <span class="s2">"/dev/dri/card0"</span> <span class="o">[</span><span class="nc">O_CLOEXEC</span><span class="o">;</span> <span class="nc">O_RDWR</span><span class="o">]</span> <span class="mi">0</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">dev</span> <span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">file_descr</span> <span class="o">=</span> <span class="o">&lt;</span><span class="n">abstr</span><span class="o">&gt;</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>To check the driver version:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Device</span><span class="p">.</span><span class="nn">Version</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Device</span><span class="p">.</span><span class="nn">Version</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line"><span class="o">{</span><span class="n">version</span> <span class="o">=</span> <span class="mi">3</span><span class="o">.</span><span class="mi">61</span><span class="o">.</span><span class="mi">0</span><span class="o">;</span> <span class="n">name</span> <span class="o">=</span> <span class="s2">"amdgpu"</span><span class="o">;</span> <span class="n">date</span> <span class="o">=</span> <span class="s2">"0"</span><span class="o">;</span> <span class="n">desc</span> <span class="o">=</span> <span class="s2">"AMD GPU"</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>If you're familiar with the C API, this corresponds to the <code>drmGetVersion</code> function,
and <code>Drm.Device.list</code> corresponds to <code>drmGetDevices2</code>;
I reorganised things a bit to make better use of OCaml's modules.</p>
<h3>Listing resources</h3>
<p>Let's see what resources we've got to play with:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">resources</span> <span class="o">=</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Kms</span><span class="p">.</span><span class="nn">Resources</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">resources</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Resources</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line">  <span class="o">{</span><span class="n">fbs</span> <span class="o">=</span> <span class="bp">[]</span><span class="o">;</span>
</span><span class="line">   <span class="n">crtcs</span> <span class="o">=</span> <span class="o">[</span><span class="mi">57</span><span class="o">;</span> <span class="mi">60</span><span class="o">;</span> <span class="mi">63</span><span class="o">;</span> <span class="mi">66</span><span class="o">;</span> <span class="mi">69</span><span class="o">];</span>
</span><span class="line">   <span class="n">connectors</span> <span class="o">=</span> <span class="o">[</span><span class="mi">71</span><span class="o">;</span> <span class="mi">78</span><span class="o">;</span> <span class="mi">84</span><span class="o">];</span>
</span><span class="line">   <span class="n">encoders</span> <span class="o">=</span> <span class="o">[</span><span class="mi">70</span><span class="o">;</span> <span class="mi">76</span><span class="o">;</span> <span class="mi">83</span><span class="o">;</span> <span class="mi">86</span><span class="o">;</span> <span class="mi">87</span><span class="o">;</span> <span class="mi">88</span><span class="o">;</span> <span class="mi">89</span><span class="o">;</span> <span class="mi">90</span><span class="o">];</span>
</span><span class="line">   <span class="n">min_width</span><span class="o">,</span><span class="n">max_width</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">16384</span><span class="o">;</span>
</span><span class="line">   <span class="n">min_height</span><span class="o">,</span><span class="n">max_height</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">16384</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Note: The Kernel Mode Setting functions are in the <a href="https://talex5.github.io/libdrm-ocaml/libdrm/Drm/Kms/index.html">Drm.Kms</a> module.
The C API calls these functions <code>drmMode*</code>, but I found that confusing as
e.g. <code>drmModeGetResources</code> sounds like you're asking for the resources of a mode.</p>
<p>A <em>CRTC</em> is a CRT Controller, and typically controls a single monitor
(known as a <a href="https://en.wikipedia.org/wiki/Cathode_ray_tube">Cathode Ray Tube</a> for historical reasons).
<em>Framebuffers</em> provide image data to a CRTC (we create framebuffers as needed).
<em>Connectors</em> correspond to physical connectors (e.g. where you plug in a monitor cable).
An <em>Encoder</em> encodes data from the CRTC for a particular connector.</p>
<p><a href="https://roscidus.com/blog/images/libdrm/arch-simple.svg"><span class="caption-wrapper center"><img src="https://roscidus.com/blog/images/libdrm/arch-simple.svg" title="Resources diagram (simplified)" class="caption"><span class="caption-text">Resources diagram (simplified)</span></span></a></p>
<h3>Connectors</h3>
<p>To save a bit of typing, I'll create an alias for the <code>Drm.Kms</code> module:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">module</span> <span class="nc">K</span> <span class="o">=</span> <span class="nn">Drm</span><span class="p">.</span><span class="nc">Kms</span><span class="o">;;</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>You could also <code>open Drm.Kms</code> to avoid needing any prefix, but I'll keep using <code>K</code> for clarity.</p>
<p>To get details for the first connector (the head of the list):</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
<span class="line-number">16</span>
<span class="line-number">17</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="o">(</span><span class="nn">List</span><span class="p">.</span><span class="n">hd</span> <span class="n">resources</span><span class="o">.</span><span class="n">connectors</span><span class="o">);;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line"><span class="o">{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">71</span><span class="o">;</span> <span class="c">(* DP-1 *)</span>
</span><span class="line"> <span class="n">connector_type</span> <span class="o">=</span> <span class="nc">DisplayPort</span><span class="o">;</span>
</span><span class="line"> <span class="n">connector_type_id</span> <span class="o">=</span> <span class="mi">1</span><span class="o">;</span>
</span><span class="line"> <span class="n">connection</span> <span class="o">=</span> <span class="nc">Connected</span><span class="o">;</span>
</span><span class="line"> <span class="n">mm_width</span><span class="o">,</span><span class="n">mm_height</span> <span class="o">=</span> <span class="mi">700</span><span class="o">,</span><span class="mi">390</span><span class="o">;</span>
</span><span class="line"> <span class="n">subpixel</span> <span class="o">=</span> <span class="nc">Unknown</span><span class="o">;</span>
</span><span class="line"> <span class="n">modes</span> <span class="o">=</span> <span class="o">[</span><span class="mi">3840</span><span class="n">x2160</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line">          <span class="mi">3840</span><span class="n">x2160</span> <span class="mi">30</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line">          <span class="mi">3840</span><span class="n">x2160</span> <span class="mi">29</span><span class="o">.</span><span class="mi">97</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line">          <span class="mi">2560</span><span class="n">x1440</span> <span class="mi">59</span><span class="o">.</span><span class="mi">95</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line">          <span class="o">...];</span>
</span><span class="line"> <span class="n">props</span> <span class="o">=</span> <span class="o">[</span><span class="mi">1</span><span class="o">:</span><span class="mi">77</span><span class="o">;</span> <span class="mi">2</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">5</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">6</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">4</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">34</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">35</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">36</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">37</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">72</span><span class="o">:</span><span class="mi">8</span><span class="o">;</span> <span class="mi">73</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> 
</span><span class="line">          <span class="mi">7</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">74</span><span class="o">:</span><span class="mi">0</span><span class="o">;</span> <span class="mi">75</span><span class="o">:</span><span class="mi">15</span><span class="o">];</span>
</span><span class="line"> <span class="n">encoder_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">70</span><span class="o">;</span>
</span><span class="line"> <span class="n">encoders</span> <span class="o">=</span> <span class="o">[</span><span class="mi">70</span><span class="o">]}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>This is DisplayPort connector 1 (usually called <code>DP-1</code>) and it's currently <code>Connected</code>.
The connector also says which modes are available on the connected monitor.</p>
<p>I was lucky in that the first connector was the one I'm using,
but really we should get all the connectors and filter them to find the connected ones.
<code>List.map</code> can be used to run <code>get</code> on each of them:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">connectors</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">map</span> <span class="o">(</span><span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span><span class="o">)</span> <span class="n">resources</span><span class="o">.</span><span class="n">connectors</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">connectors</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line">  <span class="o">[{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">71</span><span class="o">;</span> <span class="c">(* DP-1 *)</span> <span class="o">...};</span>
</span><span class="line">   <span class="o">{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">78</span><span class="o">;</span> <span class="c">(* HDMI-A-1 *)</span> <span class="o">...};</span>
</span><span class="line">   <span class="o">{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">84</span><span class="o">;</span> <span class="c">(* DVI-D-1 *)</span> <span class="o">...}]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Then to filter:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">is_connected</span> <span class="o">(</span><span class="n">c</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span><span class="o">)</span> <span class="o">=</span> <span class="o">(</span><span class="n">c</span><span class="o">.</span><span class="n">connection</span> <span class="o">=</span> <span class="nc">Connected</span><span class="o">);;</span>
</span><span class="line"><span class="k">val</span> <span class="n">is_connected</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span> <span class="o">-&gt;</span> <span class="kt">bool</span> <span class="o">=</span> <span class="o">&lt;</span><span class="k">fun</span><span class="o">&gt;</span>
</span><span class="line">
</span><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">connected</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">filter</span> <span class="n">is_connected</span> <span class="n">connectors</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">connected</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line">  <span class="o">[{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">71</span><span class="o">;</span> <span class="c">(* DP-1 *)</span> <span class="o">...}]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>We'll investigate <code>c</code>, the first connected one:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">c</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">hd</span> <span class="n">connected</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">c</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line">  <span class="o">{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">71</span><span class="o">;</span> <span class="c">(* DP-1 *)</span> <span class="o">...}</span>
</span></code></pre></td></tr></tbody></table></div></figure><h4>A note on IDs</h4>
<p>In the libdrm C API, IDs are just integers.
To avoid mix-ups, I made them distinct types in the OCaml API.
For example, if you try to use an encoder ID as a connector ID:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="o">(</span><span class="nn">List</span><span class="p">.</span><span class="n">hd</span> <span class="n">resources</span><span class="o">.</span><span class="n">encoders</span><span class="o">);;</span>
</span><span class="line">                           <span class="o">^^^^^^^^^^^^^^^^^^^^^^^^^^^^</span>
</span><span class="line"><span class="nc">Error</span><span class="o">:</span> <span class="nc">This</span> <span class="n">expression</span> <span class="n">has</span> <span class="k">type</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Kms</span><span class="p">.</span><span class="nn">Encoder</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Encoder</span> <span class="o">]</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Id</span><span class="p">.</span><span class="n">t</span>
</span><span class="line">       <span class="n">but</span> <span class="n">an</span> <span class="n">expression</span> <span class="n">was</span> <span class="n">expected</span> <span class="k">of</span> <span class="k">type</span>
</span><span class="line">         <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Connector</span> <span class="o">]</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Id</span><span class="p">.</span><span class="n">t</span>
</span><span class="line">       <span class="nc">These</span> <span class="n">two</span> <span class="n">variant</span> <span class="n">types</span> <span class="n">have</span> <span class="n">no</span> <span class="n">intersection</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Normally this is what you want, but for interactive use it's annoying that you can't just pass a plain integer.
e.g.</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="mi">71</span><span class="o">;;</span>
</span><span class="line">                           <span class="o">^^</span>
</span><span class="line"><span class="nc">Error</span><span class="o">:</span> <span class="nc">The</span> <span class="n">constant</span> <span class="mi">71</span> <span class="n">has</span> <span class="k">type</span> <span class="kt">int</span> <span class="n">but</span> <span class="n">an</span> <span class="n">expression</span> <span class="n">was</span> <span class="n">expected</span> <span class="k">of</span> <span class="k">type</span>
</span><span class="line">         <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Connector</span> <span class="o">]</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Id</span><span class="p">.</span><span class="n">t</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>You can get any kind of ID with <code>Drm.Id.of_int</code> (e.g. <code>K.Connector.get dev (Drm.Id.of_int 71)</code>),
but that's still a bit verbose, so you might prefer to (re)define a prefix operator for it, e.g.</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="o">(</span> <span class="o">!</span> <span class="o">)</span> <span class="o">=</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Id</span><span class="p">.</span><span class="n">of_int</span><span class="o">;;</span>
</span><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="o">!</span><span class="mi">71</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span> <span class="o">{</span><span class="n">connector_id</span> <span class="o">=</span> <span class="mi">71</span><span class="o">;</span> <span class="o">...}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>(note: <code>!</code> is the only single-character prefix operator available in OCaml)</p>
<h3>Modes</h3>
<p>Modes are shown in abbreviated form in the connector output.
To see the full list:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="n">c</span><span class="o">.</span><span class="n">modes</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Mode_info</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line"><span class="o">[</span><span class="mi">3840</span><span class="n">x2160</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">3840</span><span class="n">x2160</span> <span class="mi">30</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">3840</span><span class="n">x2160</span> <span class="mi">29</span><span class="o">.</span><span class="mi">97</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">2560</span><span class="n">x1440</span> <span class="mi">59</span><span class="o">.</span><span class="mi">95</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">1920</span><span class="n">x1200</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1920</span><span class="n">x1080</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1920</span><span class="n">x1080</span> <span class="mi">59</span><span class="o">.</span><span class="mi">94</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1600</span><span class="n">x1200</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">1680</span><span class="n">x1050</span> <span class="mi">59</span><span class="o">.</span><span class="mi">95</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1600</span><span class="n">x900</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1280</span><span class="n">x1024</span> <span class="mi">75</span><span class="o">.</span><span class="mi">02</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1280</span><span class="n">x1024</span> <span class="mi">60</span><span class="o">.</span><span class="mi">02</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">1440</span><span class="n">x900</span> <span class="mi">59</span><span class="o">.</span><span class="mi">89</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1280</span><span class="n">x800</span> <span class="mi">59</span><span class="o">.</span><span class="mi">81</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1152</span><span class="n">x864</span> <span class="mi">75</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1280</span><span class="n">x720</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">1280</span><span class="n">x720</span> <span class="mi">59</span><span class="o">.</span><span class="mi">94</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1024</span><span class="n">x768</span> <span class="mi">75</span><span class="o">.</span><span class="mi">03</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1024</span><span class="n">x768</span> <span class="mi">70</span><span class="o">.</span><span class="mi">07</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">1024</span><span class="n">x768</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">832</span><span class="n">x624</span> <span class="mi">74</span><span class="o">.</span><span class="mi">55</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">800</span><span class="n">x600</span> <span class="mi">75</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">800</span><span class="n">x600</span> <span class="mi">72</span><span class="o">.</span><span class="mi">19</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">800</span><span class="n">x600</span> <span class="mi">60</span><span class="o">.</span><span class="mi">32</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">800</span><span class="n">x600</span> <span class="mi">56</span><span class="o">.</span><span class="mi">25</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">640</span><span class="n">x480</span> <span class="mi">75</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">640</span><span class="n">x480</span> <span class="mi">72</span><span class="o">.</span><span class="mi">81</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">640</span><span class="n">x480</span> <span class="mi">66</span><span class="o">.</span><span class="mi">67</span><span class="n">Hz</span><span class="o">;</span>
</span><span class="line"> <span class="mi">640</span><span class="n">x480</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">640</span><span class="n">x480</span> <span class="mi">59</span><span class="o">.</span><span class="mi">94</span><span class="n">Hz</span><span class="o">;</span> <span class="mi">720</span><span class="n">x400</span> <span class="mi">70</span><span class="o">.</span><span class="mi">08</span><span class="n">Hz</span><span class="o">]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Note: I annotated various pretty-printer functions with <code>[@@ocaml.toplevel_printer]</code>,
which causes utop to use them by default to display values of the corresponding type.
For example, showing a list of modes uses this short summary form.
Displaying an individual mode shows all the information.
Here's the first mode:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
<span class="line-number">16</span>
<span class="line-number">17</span>
<span class="line-number">18</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="o">#</span> <span class="nn">List</span><span class="p">.</span><span class="n">hd</span> <span class="n">c</span><span class="o">.</span><span class="n">modes</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Mode_info</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line"><span class="o">{</span><span class="n">name</span> <span class="o">=</span> <span class="s2">"3840x2160"</span><span class="o">;</span>
</span><span class="line"> <span class="n">typ</span> <span class="o">=</span> <span class="n">preferred</span><span class="o">+</span><span class="n">driver</span><span class="o">;</span>
</span><span class="line"> <span class="n">flags</span> <span class="o">=</span> <span class="n">phsync</span><span class="o">+</span><span class="n">nvsync</span><span class="o">;</span>
</span><span class="line"> <span class="n">stereo_mode</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span>
</span><span class="line"> <span class="n">aspect_ratio</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span>
</span><span class="line"> <span class="n">clock</span> <span class="o">=</span> <span class="mi">533250</span><span class="o">;</span>
</span><span class="line"> <span class="n">hdisplay</span><span class="o">,</span><span class="n">vdisplay</span> <span class="o">=</span> <span class="mi">3840</span><span class="o">,</span><span class="mi">2160</span><span class="o">;</span>
</span><span class="line"> <span class="n">hsync_start</span> <span class="o">=</span> <span class="mi">3888</span><span class="o">;</span>
</span><span class="line"> <span class="n">hsync_end</span> <span class="o">=</span> <span class="mi">3920</span><span class="o">;</span>
</span><span class="line"> <span class="n">htotal</span> <span class="o">=</span> <span class="mi">4000</span><span class="o">;</span>
</span><span class="line"> <span class="n">hskew</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"> <span class="n">vsync_start</span> <span class="o">=</span> <span class="mi">2163</span><span class="o">;</span>
</span><span class="line"> <span class="n">vsync_end</span> <span class="o">=</span> <span class="mi">2168</span><span class="o">;</span>
</span><span class="line"> <span class="n">vtotal</span> <span class="o">=</span> <span class="mi">2222</span><span class="o">;</span>
</span><span class="line"> <span class="n">vscan</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"> <span class="n">vrefresh</span> <span class="o">=</span> <span class="mi">60</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><h3>Properties</h3>
<p>Some resources can also have extra <a href="https://drmdb.emersion.fr/properties">properties</a>.
Use <code>get_properties</code> to fetch them:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get_properties</span> <span class="n">dev</span> <span class="n">c</span><span class="o">.</span><span class="n">connector_id</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Connector</span> <span class="o">]</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Properties</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line"><span class="o">{</span><span class="nc">EDID</span> <span class="o">=</span> <span class="mi">92</span><span class="o">;</span> <span class="nc">DPMS</span> <span class="o">=</span> <span class="nc">On</span><span class="o">;</span> <span class="nc">TILE</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="n">link</span><span class="o">-</span><span class="n">status</span> <span class="o">=</span> <span class="nc">Good</span><span class="o">;</span> <span class="n">non</span><span class="o">-</span><span class="n">desktop</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"> <span class="nc">HDR_OUTPUT_METADATA</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="n">scaling</span> <span class="n">mode</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="n">underscan</span> <span class="o">=</span> <span class="n">off</span><span class="o">;</span>
</span><span class="line"> <span class="n">underscan</span> <span class="n">hborder</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">underscan</span> <span class="n">vborder</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">max</span> <span class="n">bpc</span> <span class="o">=</span> <span class="mi">8</span><span class="o">;</span>
</span><span class="line"> <span class="nc">Colorspace</span> <span class="o">=</span> <span class="nc">Default</span><span class="o">;</span> <span class="n">vrr_capable</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">subconnector</span> <span class="o">=</span> <span class="nc">Native</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Linux only returns a subset of the properties until you enable the <a href="https://talex5.github.io/libdrm-ocaml/libdrm/Drm/Client_cap/index.html#val-atomic">atomic</a> feature.
Let's turn that on now:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Client_cap</span><span class="p">.</span><span class="o">(</span><span class="n">set</span> <span class="n">atomic</span><span class="o">)</span> <span class="n">dev</span> <span class="bp">true</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="o">(</span><span class="kt">unit</span><span class="o">,</span> <span class="nn">Unix</span><span class="p">.</span><span class="n">error</span><span class="o">)</span> <span class="n">result</span> <span class="o">=</span> <span class="nc">Ok</span> <span class="bp">()</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>(<code>Module.(expr)</code> is a short-hand that brings all of <code>Module</code>'s symbols into scope for <code>expr</code>,
so we don't have to repeat the module name for both <code>set</code> and <code>atomic</code>)</p>
<p>And getting the properties again, we now have an extra <code>CRTC_ID</code>,
telling us which controller this connector is currently using:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">c_props</span> <span class="o">=</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">get_properties</span> <span class="n">dev</span> <span class="n">c</span><span class="o">.</span><span class="n">connector_id</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">c_props</span> <span class="o">:</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Connector</span> <span class="o">]</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Properties</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line"><span class="o">{</span><span class="nc">EDID</span> <span class="o">=</span> <span class="mi">92</span><span class="o">;</span> <span class="nc">DPMS</span> <span class="o">=</span> <span class="nc">On</span><span class="o">;</span> <span class="nc">TILE</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="n">link</span><span class="o">-</span><span class="n">status</span> <span class="o">=</span> <span class="nc">Good</span><span class="o">;</span> <span class="n">non</span><span class="o">-</span><span class="n">desktop</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"> <span class="nc">HDR_OUTPUT_METADATA</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="nc">CRTC_ID</span> <span class="o">=</span> <span class="mi">57</span><span class="o">;</span> <span class="n">scaling</span> <span class="n">mode</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span>
</span><span class="line"> <span class="n">underscan</span> <span class="o">=</span> <span class="n">off</span><span class="o">;</span> <span class="n">underscan</span> <span class="n">hborder</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">underscan</span> <span class="n">vborder</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">max</span> <span class="n">bpc</span> <span class="o">=</span> <span class="mi">8</span><span class="o">;</span>
</span><span class="line"> <span class="nc">Colorspace</span> <span class="o">=</span> <span class="nc">Default</span><span class="o">;</span> <span class="n">vrr_capable</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">subconnector</span> <span class="o">=</span> <span class="nc">Native</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><h3>Encoders</h3>
<p><a href="https://www.kernel.org/doc/html/latest/gpu/drm-kms.html">The Linux documentation</a> says:</p>
<blockquote>
<p>Those are really just internal artifacts of the helper libraries used to
implement KMS drivers. Besides that they make it unnecessarily more
complicated for userspace to figure out which connections between a CRTC and
a connector are possible, and what kind of cloning is supported, they serve
no purpose in the userspace API. Unfortunately encoders have been exposed to
userspace, hence can’t remove them at this point. Furthermore the exposed
restrictions are often wrongly set by drivers, and in many cases not powerful
enough to express the real restrictions.</p>
</blockquote>
<p>OK. Well, let's take a look anyway:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">e</span> <span class="o">=</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Encoder</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="o">(</span><span class="nn">Option</span><span class="p">.</span><span class="n">get</span> <span class="n">c</span><span class="o">.</span><span class="n">encoder_id</span><span class="o">);;</span>
</span><span class="line"><span class="k">val</span> <span class="n">e</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Encoder</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line">  <span class="o">{</span><span class="n">encoder_id</span> <span class="o">=</span> <span class="mi">70</span><span class="o">;</span>
</span><span class="line">   <span class="n">encoder_type</span> <span class="o">=</span> <span class="nc">TMDS</span><span class="o">;</span>
</span><span class="line">   <span class="n">crtc_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">57</span><span class="o">;</span>
</span><span class="line">   <span class="n">possible_crtcs</span> <span class="o">=</span> <span class="mh">0x1f</span><span class="o">;</span>
</span><span class="line">   <span class="n">possible_clones</span> <span class="o">=</span> <span class="mh">0x1</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Note: We need <code>Option.get</code> here because a connector might not have an encoder set yet.
Where the C API uses 0 to indicate no resource,
the OCaml API uses <code>None</code> to force us to think about that case.</p>
<p>As the documentation says, the encoder is mainly useful to get the CRTC ID:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">crtc_id</span> <span class="o">=</span> <span class="nn">Option</span><span class="p">.</span><span class="n">get</span> <span class="n">e</span><span class="o">.</span><span class="n">crtc_id</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">crtc_id</span> <span class="o">:</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Kms</span><span class="p">.</span><span class="nn">Crtc</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="mi">57</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>We could instead have got that directly from the connector using its properties:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Properties</span><span class="p">.</span><span class="nn">Values</span><span class="p">.</span><span class="n">get_value_exn</span> <span class="n">c_props</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">crtc_id</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Crtc</span> <span class="o">]</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Id</span><span class="p">.</span><span class="n">t</span> <span class="n">option</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">57</span>
</span></code></pre></td></tr></tbody></table></div></figure><h3>CRT Controllers</h3>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">crtc</span> <span class="o">=</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Crtc</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="n">crtc_id</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">crtc</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Crtc</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line">  <span class="o">{</span><span class="n">crtc_id</span> <span class="o">=</span> <span class="mi">57</span><span class="o">;</span>
</span><span class="line">   <span class="n">fb_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">93</span><span class="o">;</span>
</span><span class="line">   <span class="n">x</span><span class="o">,</span><span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">   <span class="n">width</span><span class="o">,</span><span class="n">height</span> <span class="o">=</span> <span class="mi">3840</span><span class="o">,</span><span class="mi">2160</span><span class="o">;</span>
</span><span class="line">   <span class="n">mode</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">3840</span><span class="n">x2160</span> <span class="mi">60</span><span class="o">.</span><span class="mi">00</span><span class="n">Hz</span><span class="o">}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>An active CRTC has a mode set (presumably from the connector's list of supported modes),
and a framebuffer with the image to be displayed.</p>
<p>If I keep calling <code>Crtc.get</code>, I see that it is sometimes showing framebuffer 93 and sometimes 94.
My Wayland compositor (Sway) updates one framebuffer while the other is being shown, then switches which one is displayed.</p>
<h3>Framebuffers</h3>
<p>My CRTC is currently displaying the contents of framebuffer 93:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">fb_id</span> <span class="o">=</span> <span class="nn">Option</span><span class="p">.</span><span class="n">get</span> <span class="n">crtc</span><span class="o">.</span><span class="n">fb_id</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">fb_id</span> <span class="o">:</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Kms</span><span class="p">.</span><span class="nn">Fb</span><span class="p">.</span><span class="n">id</span> <span class="o">=</span> <span class="mi">93</span>
</span></code></pre></td></tr></tbody></table></div></figure><figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">fb</span> <span class="o">=</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Fb</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span> <span class="n">fb_id</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">fb</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Fb</span><span class="p">.</span><span class="n">t</span> <span class="o">=</span>
</span><span class="line">  <span class="o">{</span><span class="n">fb_id</span> <span class="o">=</span> <span class="mi">93</span><span class="o">;</span>
</span><span class="line">   <span class="n">width</span><span class="o">,</span><span class="n">height</span> <span class="o">=</span> <span class="mi">3840</span><span class="o">,</span><span class="mi">2160</span><span class="o">;</span>
</span><span class="line">   <span class="n">pixel_format</span><span class="o">,</span> <span class="n">modifier</span> <span class="o">=</span> <span class="nc">XR24</span><span class="o">,</span> <span class="nc">None</span><span class="o">;</span>
</span><span class="line">   <span class="n">interlaced</span> <span class="o">=</span> <span class="bp">false</span><span class="o">;</span>
</span><span class="line">   <span class="n">planes</span> <span class="o">=</span> <span class="o">[{</span><span class="n">handle</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="n">pitch</span> <span class="o">=</span> <span class="mi">15360</span><span class="o">;</span> <span class="n">offset</span> <span class="o">=</span> <span class="mi">0</span><span class="o">}]}</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>A framebuffer has up to 4 framebuffer planes (not to be confused with CRTC planes; see later),
each of which references a <em>buffer object</em> (also known as a <em>BO</em> and referenced with a <em>GEM handle</em>).</p>
<p>This framebuffer is using the <code>XR24</code> format, where there is a single BO with 32 bits for each pixel
(8 for red, 8 green, 8 blue and 8 unused).
Some formats use e.g. a separate buffer for each component
(or a different part of the same buffer, using <code>offset</code>).</p>
<p>Modern graphics cards also support format <em>modifiers</em>, but my card is too old so I just get <code>None</code>.
Linux's <a href="https://github.com/torvalds/linux/blob/master/include/uapi/drm/drm_fourcc.h">fourcc.h</a> header file describes the various formats and modifiers.
Modifiers seem to be mainly used to specify the <a href="https://docs.mesa3d.org/isl/tiling.html">tiling</a>.</p>
<p>I don't have permission to see the buffer object, so it appears as (<code>handle = None</code>).
The <code>pitch</code> is the number of bytes from one row to the next (also known as the <em>stride</em>).
Here, the 15360 is simply the width (3840) multiplied by the 4 bytes per pixel.</p>
<h3>CRTC planes</h3>
<p>In fact, <code>Crtc.get</code> is an old API that only covers the basic case of a single framebuffer.
In reality, a CRTC can combine multiple <em>CRTC planes</em>, which for some reason aren't returned with the other resources
and must be requested separately:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">plane_ids</span> <span class="o">=</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">list</span> <span class="n">dev</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">plane_ids</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">id</span> <span class="kt">list</span> <span class="o">=</span> <span class="o">[</span><span class="mi">40</span><span class="o">;</span> <span class="mi">43</span><span class="o">;</span> <span class="mi">46</span><span class="o">;</span> <span class="mi">49</span><span class="o">;</span> <span class="mi">52</span><span class="o">;</span> <span class="mi">55</span><span class="o">;</span> <span class="mi">58</span><span class="o">;</span> <span class="mi">61</span><span class="o">;</span> <span class="mi">64</span><span class="o">;</span> <span class="mi">67</span><span class="o">]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>(note: you need to enable "atomic" mode before requesting planes; we already did that above)</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">planes</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">map</span> <span class="o">(</span><span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">get</span> <span class="n">dev</span><span class="o">)</span> <span class="n">plane_ids</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">planes</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line">  <span class="o">[{</span><span class="n">formats</span> <span class="o">=</span> <span class="o">[</span><span class="nc">XR24</span><span class="o">;</span> <span class="nc">AR24</span><span class="o">;</span> <span class="nc">RA24</span><span class="o">;</span> <span class="nc">XR30</span><span class="o">;</span> <span class="nc">XB30</span><span class="o">;</span> <span class="nc">AR30</span><span class="o">;</span> <span class="nc">AB30</span><span class="o">;</span> <span class="nc">XR48</span><span class="o">;</span> <span class="nc">XB48</span><span class="o">;</span> 
</span><span class="line">               <span class="nc">AR48</span><span class="o">;</span> <span class="nc">AB48</span><span class="o">;</span> <span class="nc">XB24</span><span class="o">;</span> <span class="nc">AB24</span><span class="o">;</span> <span class="nc">RG16</span><span class="o">;</span> <span class="nc">XR4H</span><span class="o">;</span> <span class="nc">AR4H</span><span class="o">;</span> <span class="nc">XB4H</span><span class="o">;</span> <span class="nc">AB4H</span><span class="o">];</span>
</span><span class="line">    <span class="n">plane_id</span> <span class="o">=</span> <span class="mi">40</span><span class="o">;</span>
</span><span class="line">    <span class="n">crtc_id</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span>
</span><span class="line">    <span class="n">fb_id</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span>
</span><span class="line">    <span class="n">crtc_x</span><span class="o">,</span><span class="n">crtc_y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">    <span class="n">x</span><span class="o">,</span><span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">    <span class="n">possible_crtcs</span> <span class="o">=</span> <span class="mh">0x10</span><span class="o">};</span>
</span><span class="line">   <span class="o">...</span>
</span><span class="line">  <span class="o">]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>A lot of these planes aren't being used (don't have a CRTC),
which we can check for with a helper function:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">has_crtc</span> <span class="o">(</span><span class="n">x</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">t</span><span class="o">)</span> <span class="o">=</span> <span class="o">(</span><span class="n">x</span><span class="o">.</span><span class="n">crtc_id</span> <span class="o">&lt;&gt;</span> <span class="nc">None</span><span class="o">);;</span>
</span><span class="line"><span class="k">val</span> <span class="n">has_crtc</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">t</span> <span class="o">-&gt;</span> <span class="kt">bool</span> <span class="o">=</span> <span class="o">&lt;</span><span class="k">fun</span><span class="o">&gt;</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>Looks like Sway is using two planes at the moment:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
<span class="line-number">16</span>
<span class="line-number">17</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">active_planes</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">filter</span> <span class="n">has_crtc</span> <span class="n">planes</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">active_planes</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line">  <span class="o">[{</span><span class="n">formats</span> <span class="o">=</span> <span class="o">[</span><span class="nc">XR24</span><span class="o">;</span> <span class="nc">AR24</span><span class="o">;</span> <span class="nc">RA24</span><span class="o">;</span> <span class="nc">XR30</span><span class="o">;</span> <span class="nc">XB30</span><span class="o">;</span> <span class="nc">AR30</span><span class="o">;</span> <span class="nc">AB30</span><span class="o">;</span> <span class="nc">XR48</span><span class="o">;</span> <span class="nc">XB48</span><span class="o">;</span> 
</span><span class="line">               <span class="nc">AR48</span><span class="o">;</span> <span class="nc">AB48</span><span class="o">;</span> <span class="nc">XB24</span><span class="o">;</span> <span class="nc">AB24</span><span class="o">;</span> <span class="nc">RG16</span><span class="o">;</span> <span class="nc">XR4H</span><span class="o">;</span> <span class="nc">AR4H</span><span class="o">;</span> <span class="nc">XB4H</span><span class="o">;</span> <span class="nc">AB4H</span><span class="o">];</span>
</span><span class="line">    <span class="n">plane_id</span> <span class="o">=</span> <span class="mi">52</span><span class="o">;</span>
</span><span class="line">    <span class="n">crtc_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">57</span><span class="o">;</span>
</span><span class="line">    <span class="n">fb_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">94</span><span class="o">;</span>
</span><span class="line">    <span class="n">crtc_x</span><span class="o">,</span><span class="n">crtc_y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">    <span class="n">x</span><span class="o">,</span><span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">    <span class="n">possible_crtcs</span> <span class="o">=</span> <span class="mh">0x1</span><span class="o">};</span>
</span><span class="line">   <span class="o">{</span><span class="n">formats</span> <span class="o">=</span> <span class="o">[</span><span class="nc">AR24</span><span class="o">];</span>
</span><span class="line">    <span class="n">plane_id</span> <span class="o">=</span> <span class="mi">55</span><span class="o">;</span>
</span><span class="line">    <span class="n">crtc_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">57</span><span class="o">;</span>
</span><span class="line">    <span class="n">fb_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="mi">98</span><span class="o">;</span>
</span><span class="line">    <span class="n">crtc_x</span><span class="o">,</span><span class="n">crtc_y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">    <span class="n">x</span><span class="o">,</span><span class="n">y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">;</span>
</span><span class="line">    <span class="n">possible_crtcs</span> <span class="o">=</span> <span class="mh">0x1</span><span class="o">}]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>More information is available as properties:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="k">let</span> <span class="n">active_plane_ids</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">map</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">id</span> <span class="n">active_planes</span><span class="o">;;</span>
</span><span class="line"><span class="k">val</span> <span class="n">active_plane_ids</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">id</span> <span class="kt">list</span> <span class="o">=</span> <span class="o">[</span><span class="mi">52</span><span class="o">;</span> <span class="mi">55</span><span class="o">]</span>
</span><span class="line">
</span><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">List</span><span class="p">.</span><span class="n">map</span> <span class="o">(</span><span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">get_properties</span> <span class="n">dev</span><span class="o">)</span> <span class="n">active_plane_ids</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="o">[</span> <span class="o">`</span><span class="nc">Plane</span> <span class="o">]</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Properties</span><span class="p">.</span><span class="n">t</span> <span class="kt">list</span> <span class="o">=</span>
</span><span class="line"><span class="o">[{</span><span class="nc">CRTC_H</span> <span class="o">=</span> <span class="mi">2160</span><span class="o">;</span> <span class="nc">CRTC_ID</span> <span class="o">=</span> <span class="mi">57</span><span class="o">;</span> <span class="nc">CRTC_W</span> <span class="o">=</span> <span class="mi">3840</span><span class="o">;</span> <span class="nc">CRTC_X</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="nc">CRTC_Y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line">  <span class="nc">FB_ID</span> <span class="o">=</span> <span class="mi">93</span><span class="o">;</span> <span class="nc">IN_FENCE_FD</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="o">;</span> <span class="nc">SRC_H</span> <span class="o">=</span> <span class="mi">141557760</span><span class="o">;</span> <span class="nc">SRC_W</span> <span class="o">=</span> <span class="mi">251658240</span><span class="o">;</span>
</span><span class="line">  <span class="nc">SRC_X</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="nc">SRC_Y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="n">rotation</span> <span class="o">=</span> <span class="o">[</span><span class="n">rotate</span><span class="o">-</span><span class="mi">0</span><span class="o">];</span> <span class="k">type</span> <span class="o">=</span> <span class="nc">Primary</span><span class="o">;</span> <span class="n">zpos</span> <span class="o">=</span> <span class="mi">0</span><span class="o">};</span>
</span><span class="line"> <span class="o">{</span><span class="nc">CRTC_H</span> <span class="o">=</span> <span class="mi">128</span><span class="o">;</span> <span class="nc">CRTC_ID</span> <span class="o">=</span> <span class="mi">57</span><span class="o">;</span> <span class="nc">CRTC_W</span> <span class="o">=</span> <span class="mi">128</span><span class="o">;</span> <span class="nc">CRTC_X</span> <span class="o">=</span> <span class="mi">3105</span><span class="o">;</span> <span class="nc">CRTC_Y</span> <span class="o">=</span> <span class="mi">1518</span><span class="o">;</span>
</span><span class="line">  <span class="nc">FB_ID</span> <span class="o">=</span> <span class="mi">98</span><span class="o">;</span> <span class="nc">IN_FENCE_FD</span> <span class="o">=</span> <span class="o">-</span><span class="mi">1</span><span class="o">;</span> <span class="nc">SRC_H</span> <span class="o">=</span> <span class="mi">8388608</span><span class="o">;</span> <span class="nc">SRC_W</span> <span class="o">=</span> <span class="mi">8388608</span><span class="o">;</span> <span class="nc">SRC_X</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line">  <span class="nc">SRC_Y</span> <span class="o">=</span> <span class="mi">0</span><span class="o">;</span> <span class="k">type</span> <span class="o">=</span> <span class="nc">Cursor</span><span class="o">;</span> <span class="n">zpos</span> <span class="o">=</span> <span class="mi">255</span><span class="o">}]</span>
</span></code></pre></td></tr></tbody></table></div></figure><ul>
<li>Plane 52 is a <code>Primary</code> plane and is using framebuffer 93 (as we saw before).
</li>
<li>Plane 55 is a <code>Cursor</code> plane, using framebuffer 98 (and the <code>AR24</code> format, with alpha/transparency).
</li>
</ul>
<p>A plane chooses which part of the frame buffer to show (<code>SRC_X</code>, <code>SRC_Y</code>, <code>SRC_W</code> and <code>SRC_H</code>)
and where it should appear on the screen (<code>CRTC_X</code>, <code>CRTC_Y</code>, <code>CRTC_W</code> and <code>CRTC_H</code>).
The source values are in 16.16 format (i.e. shifted left 16 bits).</p>
<p>Oddly, <code>Plane.get</code> returned <code>crtc_x,crtc_y = 0,0</code> for both planes, but
the properties show the correct cursor location (<code>CRTC_X = 3105; CRTC_Y = 1518;</code>).</p>
<p>Having the cursor on a separate plane avoids having to modify the main screen image
whenever the mouse pointer moves, which is good for low latency
(especially if the GPU is busy rendering something else at the time),
power consumption (the GPU can stay powered down),
and allows showing an application's buffer full screen without the compositor
needing to modify the application's buffer.</p>
<p>You might also have some <code>Overlay</code> planes,
which can be <a href="https://zamundaaa.github.io/wayland/2025/10/23/more-kms-offloading.html">useful for displaying video</a>.
My graphics card seems to be too old for that.</p>
<h3>Expanded resources diagram</h3>
<p>Here's an expanded diagram showing some more possibilities:</p>
<p><a href="https://roscidus.com/blog/images/libdrm/arch.svg"><span class="caption-wrapper center"><img src="https://roscidus.com/blog/images/libdrm/arch.svg" title="Expanded resources diagram" class="caption"><span class="caption-text">Expanded resources diagram</span></span></a></p>
<ul>
<li>Some framebuffer formats take the input data from multiple buffers.
</li>
<li>A framebuffer can be shared by multiple CRTCs (perhaps with each plane showing a different part of it).
</li>
<li>A CRTC can have multiple planes (e.g. primary and cursor).
</li>
<li>A single CRTC can show the same image on multiple monitors.
</li>
</ul>
<h2>Making changes</h2>
<p>If I try turning off the CRTC (by setting the mode to <code>None</code>) from my desktop environment it fails:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Crtc</span><span class="p">.</span><span class="n">set</span> <span class="n">dev</span> <span class="n">crtc_id</span> <span class="o">~</span><span class="n">pos</span><span class="o">:(</span><span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">)</span> <span class="o">~</span><span class="n">connectors</span><span class="o">:</span><span class="bp">[]</span> <span class="nc">None</span><span class="o">;;</span>
</span><span class="line"><span class="nc">Exception</span><span class="o">:</span> <span class="nn">Unix</span><span class="p">.</span><span class="nc">Unix_error</span><span class="o">(</span><span class="nn">Unix</span><span class="p">.</span><span class="nc">EACCES</span><span class="o">,</span> <span class="s2">"drmModeSetCrtc"</span><span class="o">,</span> <span class="s2">""</span><span class="o">)</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>The reason is that I'm currently running a graphical desktop and Sway owns the device
(so my <code>dev</code> is not the DRM "master"):</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">utop</span> <span class="o">#</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Device</span><span class="p">.</span><span class="n">is_master</span> <span class="n">dev</span><span class="o">;;</span>
</span><span class="line"><span class="o">-</span> <span class="o">:</span> <span class="kt">bool</span> <span class="o">=</span> <span class="bp">false</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>That can be fixed by switching to a different <a href="https://en.wikipedia.org/wiki/Virtual_console">VT</a> (e.g. with Ctrl-Alt-F2) and running it there.
However, this will result in a second problem: I won't be able to see what I'm doing!</p>
<p>If you have a second computer then you can SSH in and test things out from there, but
for simplicity we'll leave the utop REPL at this point and write some programs instead.</p>
<p>For example, <a href="https://github.com/talex5/libdrm-ocaml/blob/main/examples/query.ml">query.ml</a> shows the information we discovered above:</p>
<pre><code>dune exec -- ./examples/query.exe
</code></pre>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="n">devices</span><span class="o">:</span>                              
</span><span class="line">  <span class="o">[{</span><span class="n">primary_node</span> <span class="o">=</span> <span class="nc">Some</span> <span class="s2">"/dev/dri/card0"</span><span class="o">;</span>
</span><span class="line">    <span class="n">render_node</span> <span class="o">=</span> <span class="nc">Some</span> <span class="s2">"/dev/dri/renderD128"</span><span class="o">;</span>
</span><span class="line"><span class="o">...</span>
</span></code></pre></td></tr></tbody></table></div></figure><h3>Non-atomic mode setting</h3>
<p>Linux provides two ways to configure modes: the old <em>non-atomic</em> API and the newer <em>atomic</em> one.</p>
<p><a href="https://github.com/talex5/libdrm-ocaml/blob/main/examples/nonatomic.ml">examples/nonatomic.ml</a> contains a simple example of the older (but simpler) API.
It starts by finding a device (the first one with a primary node supporting KMS), then
finds all connected connectors (as we did above), and calls <code>show_test_page</code> on each one:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span>
</span><span class="line">  <span class="nn">Utils</span><span class="p">.</span><span class="n">with_device</span> <span class="o">@@</span> <span class="k">fun</span> <span class="n">t</span> <span class="o">-&gt;</span>
</span><span class="line">  <span class="k">let</span> <span class="n">connected</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">filter</span> <span class="nn">Utils</span><span class="p">.</span><span class="n">is_connected</span> <span class="n">t</span><span class="o">.</span><span class="n">connectors</span> <span class="k">in</span>
</span><span class="line">  <span class="nn">Utils</span><span class="p">.</span><span class="n">restoring_afterwards</span> <span class="n">t</span> <span class="o">@@</span> <span class="k">fun</span> <span class="bp">()</span> <span class="o">-&gt;</span>
</span><span class="line">  <span class="nn">List</span><span class="p">.</span><span class="n">iter</span> <span class="o">(</span><span class="n">show_test_page</span> <span class="n">t</span><span class="o">)</span> <span class="n">connected</span><span class="o">;</span>
</span><span class="line">  <span class="nn">Unix</span><span class="p">.</span><span class="n">sleep</span> <span class="mi">2</span>
</span></code></pre></td></tr></tbody></table></div></figure><p><code>restoring_afterwards</code> stores the current configuration, runs the callback,
and then puts things back to normal when that finishes (or you press Ctrl-C).</p>
<p>The program waits for 2 seconds after showing the test page before exiting.</p>
<p><code>show_test_page</code> finds the CRTC (as we did above),
takes the first supported mode, creates a test framebuffer of that size,
and configures the CRTC to display it:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="k">let</span> <span class="n">show_test_page</span> <span class="o">(</span><span class="n">t</span> <span class="o">:</span> <span class="nn">Resources</span><span class="p">.</span><span class="n">t</span><span class="o">)</span> <span class="o">(</span><span class="n">c</span> <span class="o">:</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">t</span><span class="o">)</span> <span class="o">=</span>
</span><span class="line">  <span class="k">match</span> <span class="n">c</span><span class="o">.</span><span class="n">encoder_id</span> <span class="k">with</span>
</span><span class="line">  <span class="o">|</span> <span class="nc">None</span> <span class="o">-&gt;</span> <span class="n">println</span> <span class="s2">"%a has no encoder (skipping)"</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">pp_name</span> <span class="n">c</span>
</span><span class="line">  <span class="o">|</span> <span class="nc">Some</span> <span class="n">encoder_id</span> <span class="o">-&gt;</span>
</span><span class="line">    <span class="k">match</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Encoder</span><span class="p">.</span><span class="n">get</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span> <span class="n">encoder_id</span> <span class="k">with</span>
</span><span class="line">    <span class="o">|</span> <span class="o">{</span> <span class="n">crtc_id</span> <span class="o">=</span> <span class="nc">None</span><span class="o">;</span> <span class="o">_</span> <span class="o">}</span> <span class="o">-&gt;</span>
</span><span class="line">      <span class="n">println</span> <span class="s2">"%a's encoder has no CRTC (skipping)"</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">pp_name</span> <span class="n">c</span>
</span><span class="line">    <span class="o">|</span> <span class="o">{</span> <span class="n">crtc_id</span> <span class="o">=</span> <span class="nc">Some</span> <span class="n">crtc_id</span><span class="o">;</span> <span class="o">_</span> <span class="o">}</span> <span class="o">-&gt;</span>
</span><span class="line">      <span class="n">println</span> <span class="s2">"Showing test page on %a"</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Connector</span><span class="p">.</span><span class="n">pp_name</span> <span class="n">c</span><span class="o">;</span>
</span><span class="line">      <span class="k">let</span> <span class="n">mode</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">hd</span> <span class="n">c</span><span class="o">.</span><span class="n">modes</span> <span class="k">in</span>
</span><span class="line">      <span class="k">let</span> <span class="n">size</span> <span class="o">=</span> <span class="o">(</span><span class="n">mode</span><span class="o">.</span><span class="n">hdisplay</span><span class="o">,</span> <span class="n">mode</span><span class="o">.</span><span class="n">vdisplay</span><span class="o">)</span> <span class="k">in</span>
</span><span class="line">      <span class="k">let</span> <span class="n">fb</span> <span class="o">=</span> <span class="nn">Test_image</span><span class="p">.</span><span class="n">create</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span> <span class="n">size</span> <span class="k">in</span>
</span><span class="line">      <span class="nn">K</span><span class="p">.</span><span class="nn">Crtc</span><span class="p">.</span><span class="n">set</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span> <span class="n">crtc_id</span> <span class="o">(</span><span class="nc">Some</span> <span class="n">mode</span><span class="o">)</span> <span class="o">~</span><span class="n">fb</span> <span class="o">~</span><span class="n">pos</span><span class="o">:(</span><span class="mi">0</span><span class="o">,</span><span class="mi">0</span><span class="o">)</span>
</span><span class="line">        <span class="o">~</span><span class="n">connectors</span><span class="o">:[</span><span class="n">c</span><span class="o">.</span><span class="n">connector_id</span><span class="o">]</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>If the connector doesn't have a CRTC, we could find a suitable one and use that,
but for simplicity the example just skips such connectors.</p>
<p>To run the example (switch away from any graphical desktop first or it won't work):</p>
<pre><code>dune exec -- ./examples/nonatomic.exe
</code></pre>
<h3>Dumb buffers</h3>
<p>Typically the pixel data to be displayed comes from some complex rendering pipeline,
but Linux also provides <em>dumb buffers</em> for simple cases such as testing.
The <code>Test_image.create</code> function used above creates a dumb buffer with a test pattern:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="k">let</span> <span class="n">create_dumb</span> <span class="n">dev</span> <span class="n">size</span> <span class="o">=</span>
</span><span class="line">  <span class="k">let</span> <span class="n">dumb_buffer</span> <span class="o">=</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Buffer</span><span class="p">.</span><span class="nn">Dumb</span><span class="p">.</span><span class="n">create</span> <span class="n">dev</span> <span class="o">~</span><span class="n">bpp</span><span class="o">:</span><span class="mi">32</span> <span class="n">size</span> <span class="k">in</span>
</span><span class="line">  <span class="k">let</span> <span class="n">arr</span> <span class="o">=</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Buffer</span><span class="p">.</span><span class="nn">Dumb</span><span class="p">.</span><span class="n">map</span> <span class="n">dev</span> <span class="n">dumb_buffer</span> <span class="nc">Int32</span> <span class="k">in</span>
</span><span class="line">  <span class="k">for</span> <span class="n">row</span> <span class="o">=</span> <span class="mi">0</span> <span class="k">to</span> <span class="n">snd</span> <span class="n">size</span> <span class="o">-</span> <span class="mi">1</span> <span class="k">do</span>
</span><span class="line">    <span class="k">for</span> <span class="n">col</span> <span class="o">=</span> <span class="mi">0</span> <span class="k">to</span> <span class="n">fst</span> <span class="n">size</span> <span class="o">-</span> <span class="mi">1</span> <span class="k">do</span>
</span><span class="line">      <span class="k">let</span> <span class="n">c</span> <span class="o">=</span>
</span><span class="line">        <span class="o">(</span><span class="n">row</span> <span class="ow">land</span> <span class="mh">0xff</span><span class="o">)</span> <span class="ow">lor</span>
</span><span class="line">        <span class="o">((</span><span class="n">col</span> <span class="ow">land</span> <span class="mh">0xff</span><span class="o">)</span> <span class="ow">lsl</span> <span class="mi">8</span><span class="o">)</span> <span class="ow">lor</span>
</span><span class="line">        <span class="o">(((</span><span class="n">row</span> <span class="n">lsr</span> <span class="mi">8</span><span class="o">)</span> <span class="ow">lor</span> <span class="o">(</span><span class="n">col</span> <span class="n">lsr</span> <span class="mi">8</span><span class="o">))</span> <span class="ow">lsl</span> <span class="mi">18</span><span class="o">)</span>
</span><span class="line">      <span class="k">in</span>
</span><span class="line">      <span class="n">arr</span><span class="o">.{</span><span class="n">row</span><span class="o">,</span> <span class="n">col</span><span class="o">}</span> <span class="o">&lt;-</span> <span class="nn">Int32</span><span class="p">.</span><span class="n">of_int</span> <span class="n">c</span>
</span><span class="line">    <span class="k">done</span><span class="o">;</span>
</span><span class="line">  <span class="k">done</span><span class="o">;</span>
</span><span class="line">  <span class="n">dumb_buffer</span>
</span></code></pre></td></tr></tbody></table></div></figure><p><code>Dumb.create</code> allocates memory for the image data.
<code>Dumb.map</code> makes it appear in host-memory as an OCaml bigarray.
The loop sets each 32-bit int in the image to some colour <code>c</code>.</p>
<p>Then we wrap this data up as an XR24-format framebuffer with a single plane:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="k">let</span> <span class="n">create</span> <span class="n">dev</span> <span class="n">size</span> <span class="o">=</span>
</span><span class="line">  <span class="k">let</span> <span class="n">buffer</span> <span class="o">=</span> <span class="n">create_dumb</span> <span class="n">dev</span> <span class="n">size</span> <span class="k">in</span>
</span><span class="line">  <span class="k">let</span> <span class="n">planes</span> <span class="o">=</span> <span class="o">[</span><span class="nn">K</span><span class="p">.</span><span class="nn">Fb</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">v</span> <span class="n">buffer</span><span class="o">.</span><span class="n">handle</span> <span class="o">~</span><span class="n">pitch</span><span class="o">:</span><span class="n">buffer</span><span class="o">.</span><span class="n">pitch</span><span class="o">]</span> <span class="k">in</span>
</span><span class="line">  <span class="nn">K</span><span class="p">.</span><span class="nn">Fb</span><span class="p">.</span><span class="n">add</span> <span class="n">dev</span> <span class="o">~</span><span class="n">size</span> <span class="o">~</span><span class="n">planes</span> <span class="o">~</span><span class="n">pixel_format</span><span class="o">:</span><span class="nn">Drm</span><span class="p">.</span><span class="nn">Fourcc</span><span class="p">.</span><span class="n">xr24</span>
</span></code></pre></td></tr></tbody></table></div></figure><h3>Atomic mode setting</h3>
<p><a href="https://github.com/talex5/libdrm-ocaml/blob/main/examples/atomic.ml">examples/atomic.ml</a> demonstrates the newer atomic API:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
<span class="line-number">15</span>
<span class="line-number">16</span>
<span class="line-number">17</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="k">let</span> <span class="bp">()</span> <span class="o">=</span>
</span><span class="line">  <span class="nn">Utils</span><span class="p">.</span><span class="n">with_device</span> <span class="o">@@</span> <span class="k">fun</span> <span class="n">t</span> <span class="o">-&gt;</span>
</span><span class="line">  <span class="nn">Drm</span><span class="p">.</span><span class="nn">Client_cap</span><span class="p">.</span><span class="o">(</span><span class="n">set_exn</span> <span class="n">atomic</span><span class="o">)</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span> <span class="bp">true</span><span class="o">;</span>
</span><span class="line">  <span class="k">let</span> <span class="n">connected</span> <span class="o">=</span> <span class="nn">List</span><span class="p">.</span><span class="n">filter</span> <span class="nn">Utils</span><span class="p">.</span><span class="n">is_connected</span> <span class="n">t</span><span class="o">.</span><span class="n">connectors</span> <span class="k">in</span>
</span><span class="line">  <span class="n">println</span> <span class="s2">"Found %d connected connectors"</span> <span class="o">(</span><span class="nn">List</span><span class="p">.</span><span class="n">length</span> <span class="n">connected</span><span class="o">);</span>
</span><span class="line">  <span class="k">let</span> <span class="n">free_planes</span> <span class="o">=</span> <span class="n">ref</span> <span class="o">(</span><span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">list</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span><span class="o">)</span> <span class="k">in</span>
</span><span class="line">  <span class="k">let</span> <span class="n">rq</span> <span class="o">=</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Atomic_req</span><span class="p">.</span><span class="n">create</span> <span class="bp">()</span> <span class="k">in</span>
</span><span class="line">  <span class="nn">List</span><span class="p">.</span><span class="n">iter</span> <span class="o">(</span><span class="n">show_test_page</span> <span class="o">~</span><span class="n">free_planes</span> <span class="n">t</span> <span class="n">rq</span><span class="o">)</span> <span class="n">connected</span><span class="o">;</span>
</span><span class="line">  <span class="n">println</span> <span class="s2">"Checking that commit will work..."</span><span class="o">;</span>
</span><span class="line">  <span class="k">match</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Atomic_req</span><span class="p">.</span><span class="n">commit</span> <span class="o">~</span><span class="n">test_only</span><span class="o">:</span><span class="bp">true</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span> <span class="n">rq</span> <span class="k">with</span>
</span><span class="line">  <span class="o">|</span> <span class="k">exception</span> <span class="nn">Unix</span><span class="p">.</span><span class="nc">Unix_error</span> <span class="o">(</span><span class="n">code</span><span class="o">,</span> <span class="o">_,</span> <span class="o">_)</span> <span class="o">-&gt;</span>
</span><span class="line">    <span class="n">println</span> <span class="s2">"Mode-setting would fail with error: %s"</span> <span class="o">(</span><span class="nn">Unix</span><span class="p">.</span><span class="n">error_message</span> <span class="n">code</span><span class="o">)</span>
</span><span class="line">  <span class="o">|</span> <span class="bp">()</span> <span class="o">-&gt;</span>
</span><span class="line">    <span class="n">println</span> <span class="s2">"Pre-commit test passed."</span><span class="o">;</span>
</span><span class="line">    <span class="nn">Utils</span><span class="p">.</span><span class="n">restoring_afterwards</span> <span class="n">t</span> <span class="o">@@</span> <span class="k">fun</span> <span class="bp">()</span> <span class="o">-&gt;</span>
</span><span class="line">    <span class="nn">K</span><span class="p">.</span><span class="nn">Atomic_req</span><span class="p">.</span><span class="n">commit</span> <span class="n">t</span><span class="o">.</span><span class="n">dev</span> <span class="n">rq</span><span class="o">;</span>
</span><span class="line">    <span class="nn">Unix</span><span class="p">.</span><span class="n">sleep</span> <span class="mi">2</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>The steps are:</p>
<ol>
<li>Use <code>set_exn atomic</code> to enable atomic mode.
</li>
<li>Create an <em>atomic request</em> (<code>rq</code>).
</li>
<li>Use <code>show_test_page</code> to populate it with the desired property changes.
</li>
<li>(optional) Check that it will work (<code>~test_only:true</code>).
</li>
<li>Commit the changes (<code>Atomic_req.commit</code>).
</li>
</ol>
<p>The advantage here is that either all changes are successfully applied at once or nothing changes.
This avoids various problems with flickering or trying to roll back partial changes.</p>
<p><code>show_test_page</code> needs a couple of modifications.
First, we have to find a plane (rather than using the old <code>Crtc.set</code> which assumes a single plane),
and then we set the plane's <code>FB_ID</code> property to the new framebuffer in the request:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="nn">K</span><span class="p">.</span><span class="nn">Atomic_req</span><span class="p">.</span><span class="n">add_property</span> <span class="n">rq</span> <span class="n">plane</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">fb_id</span> <span class="o">(</span><span class="nc">Some</span> <span class="n">fb</span><span class="o">)</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>For the example, I actually set more properties and defined an operator to make the code a bit neater:</p>
<figure class="code"><div class="highlight"><table><tbody><tr><td class="gutter"><pre class="line-numbers"><span class="line-number">1</span>
<span class="line-number">2</span>
<span class="line-number">3</span>
<span class="line-number">4</span>
<span class="line-number">5</span>
<span class="line-number">6</span>
<span class="line-number">7</span>
<span class="line-number">8</span>
<span class="line-number">9</span>
<span class="line-number">10</span>
<span class="line-number">11</span>
<span class="line-number">12</span>
<span class="line-number">13</span>
<span class="line-number">14</span>
</pre></td><td class="code"><pre><code class="ocaml"><span class="line"><span class="k">let</span> <span class="o">(</span> <span class="o">.%{}&lt;-</span> <span class="o">)</span> <span class="n">obj</span> <span class="n">prop</span> <span class="n">value</span> <span class="o">=</span>
</span><span class="line">  <span class="nn">K</span><span class="p">.</span><span class="nn">Atomic_req</span><span class="p">.</span><span class="n">add_property</span> <span class="n">rq</span> <span class="n">obj</span> <span class="n">prop</span> <span class="n">value</span>
</span><span class="line"><span class="k">in</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">fb_id</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="nc">Some</span> <span class="n">fb</span><span class="o">;</span>
</span><span class="line"><span class="c">(* Source region on frame-buffer: *)</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">src_x</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Ufixed</span><span class="p">.</span><span class="n">of_int</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">src_y</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Ufixed</span><span class="p">.</span><span class="n">of_int</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">src_w</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Ufixed</span><span class="p">.</span><span class="n">of_int</span> <span class="o">(</span><span class="n">fst</span> <span class="n">size</span><span class="o">);</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">src_h</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="nn">Drm</span><span class="p">.</span><span class="nn">Ufixed</span><span class="p">.</span><span class="n">of_int</span> <span class="o">(</span><span class="n">snd</span> <span class="n">size</span><span class="o">);</span>
</span><span class="line"><span class="c">(* Destination region on CRTC: *)</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">crtc_x</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">crtc_y</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="mi">0</span><span class="o">;</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">crtc_w</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="n">fst</span> <span class="n">size</span><span class="o">;</span>
</span><span class="line"><span class="n">plane</span><span class="o">.%{</span> <span class="nn">K</span><span class="p">.</span><span class="nn">Plane</span><span class="p">.</span><span class="n">crtc_h</span> <span class="o">}</span> <span class="o">&lt;-</span> <span class="n">snd</span> <span class="n">size</span><span class="o">;</span>
</span></code></pre></td></tr></tbody></table></div></figure><p>In libdrm-ocaml, properties are typed, so you can't forget to convert the source values to fixed point format.</p>
<h2>3D rendering</h2>
<p>The examples above use a dumb-buffer, but it's fairly simple to replace that with a Vulkan buffer.
The code in the last post <a href="https://github.com/talex5/vulkan-test/blob/5a93c76eb4e0205c63e0d0c5f7b4785cd15c208a/vulkan/swap_chain.ml#L42">exported the image memory from Vulkan</a> as a dmabuf FD and sent it to the Wayland compositor.
Now, instead of sending it we just need to import it into our device (with <code>Drm.Dmabuf.to_handle</code>)
and use that handle instead of the dumb-buffer one.</p>
<p>I added a simple <a href="https://github.com/talex5/vulkan-test/commit/731e90d1d61d78b4ac557f9861839dbcc233e06b">surface abstraction</a> to the test code, wrapping the <code>Window</code> module's API
so that the rendering code doesn't need to care whether it's rendering to a Wayland window or directly to the screen.
Then I made a <a href="https://github.com/talex5/vulkan-test/blob/3f767e64f38b75e216a530b6b9747020d958f11b/src/vt.ml">Vt module</a> implementing the new <code>Surface.t</code> type for rendering directly to a Linux VT.</p>
<p>To get the animation working, I used <code>K.Crtc.page_flip</code> to update the framebuffer (I could also have used the atomic API).
The kernel waits until the encoder has finishing sending the current frame before switching to the new one,
which avoids tearing.
We also need to ask the kernel to tell us when this happens, which is done by setting the optional <code>~event</code> argument to some number.
You can read events from the device file and parse them with <a href="https://talex5.github.io/libdrm-ocaml/libdrm/Drm/Event/index.html#val-parse">Drm.Event.parse</a>.</p>
<p>If you want to try it, this should produce an animated room:</p>
<pre><code>git clone https://github.com/talex5/vulkan-test -b kms-3d
cd vulkan-test
nix develop
make download-example
dune exec -- ./src/main.exe 10000 viking_room.obj viking_room.png
</code></pre>
<p>If run with <code>$WAYLAND_DISPLAY</code> set, it will open a Wayland window (as before),
but if run from a text console then it should render the animation directly using KMS.</p>
<h2>Linux VTs</h2>
<p>When the user switches to another virtual terminal (e.g. with Ctrl-Alt-F3),
we should call <code>Drm.Device.drop_master</code> to give up being the master,
allowing the application running on the new terminal to take over.</p>
<p>We should also switch the VT to <code>KD_GRAPHICS</code> mode while using it,
to stop the kernel trying to manage it.</p>
<p>I didn't implement either of these features, but see <a href="https://dvdhrm.wordpress.com/2013/08/24/how-vt-switching-works/">How VT-switching works</a> for details.</p>
<h2>Debugging</h2>
<p>If you get an unhelpful error code from the kernel (e.g. <code>EINVAL</code>), enabling debug messages is often helpful.
Writing <code>4</code> to <code>/sys/module/drm/parameters/debug</code> enables KMS debug messages, which can be seen in the <code>dmesg</code> output.
Write <code>0</code> to the file afterwards to turn the messages off again.
<code>modinfo -p drm</code> lists the various options.</p>
<h2>Conclusions</h2>
<p>I hope you found being able to explore the libdrm API interactively from the OCaml top-level
made it easier to learn about how Linux manages displays.
As when doing <a href="https://roscidus.com/blog/blog/2025/09/20/ocaml-vulkan/">Vulkan in OCaml</a>,
a lot of the noise from C is removed and I think that the essentials of what is going on are easier to see.</p>
<p>I used <a href="https://github.com/yallop/ocaml-ctypes">ocaml-ctypes</a> for the C bindings, and this was my first time using it in "stubs" mode
(where it pre-generates C bindings from OCaml definitions).
This has the advantage that the C type checker checks that the definitions are correct,
and it worked well.
Dune's <a href="https://dune.readthedocs.io/en/latest/foreign-code.html#stub-generation-with-dune-ctypes">Stub Generation</a> feature generates the build rules for this semi-automatically.</p>
<p>Deciding what OCaml types to use for the C types was quite difficult.
For example, C has many different integer types (<code>int</code>, <code>long</code>, <code>uint32_t</code>, etc),
but using lots of types is more painful in OCaml where e.g. <code>+</code> only works on <code>int</code>.
I used OCaml's int type when possible, and other types only when the value might not fit
(e.g. an image size on a 32-bit platform might not fit into an OCaml int, which is one bit shorter).</p>
<p>The C API is somewhat inconsistent about types.
e.g. <code>drmModePageFlipTarget</code> takes a <code>uint32_t target_vblank</code> argument for the sequence number,
while <code>page_flip_handler</code> confirms the event by giving it as <code>unsigned int sequence</code>.
Meanwhile, the <code>sequence_handler</code> event gives it as <code>uint64_t sequence</code>.
I'm not sure what happens if the sequence number gets too large to fit in a 32-bit integer.</p>
<p>Anyway, I think I understand mode setting a lot better now,
and I'm getting faster at debugging graphics problems on Linux
(e.g. when <a href="https://github.com/NixOS/nixpkgs/issues/458132">element-desktop failed to start</a> recently after I updated it).</p>
<p>Thanks to the OCaml Software Foundation for sponsoring this work.</p>

