---
title: ICFP/SPLASH 2025
description:
url: https://ryan.freumh.org/2025-icfp-splash.html
date: 2025-11-06T00:00:00-00:00
preview_image:
authors:
- Ryan T. Gibb
source:
---

<div class="container">
    
    <span>Published 6 Nov. 2025.</span>
    
    
</div>

<div> Tags: <a href="https://ryan.freumh.org/conference.html" title="All pages tagged 'Conference'." rel="tag">Conference</a>. </div>


<p><span>I attended the co-located <a href="https://icfp25.sigplan.org/">International Conference on
Functional Programming (ICFP)</a> and <a href="https://2025.splashcon.org/">International Conference on Systems,
Programming, Languages and Applications: Software for Humanity
(SPLASH)</a> as a co-author of a <a href="https://ryan.freumh.org/papers/2025-icfp-docker.html">couple</a> <a href="https://ryan.freumh.org/papers/2025-ocaml-ai.html">of</a> papers, and was presenting ‘<a href="https://conf.researchr.org/details/icfp-splash-2025/propl-2025-papers/13/Spatial-Programming-for-Environmental-Monitoring">Spatial
Programming for Environmental Monitoring</a>’ at the <a href="https://conf.researchr.org/home/icfp-splash-2025/propl-2025">International
Workshop on Programming for the Planet (PROPL 2025)</a>. While a lot of
the type-theory papers are beyond my ken ICFP is a great gathering place
for a lot of like-minded individuals interested in programming languages
from a range of perspectives including the practical side. As <a href="https://patrick.sirref.org/icfp-2025/index.xml">Patrick says</a>,
‘the best track is <u>hallway</u> track’. As such, there were too many
conversation to document here but I’ve tried to recall some
highlights.</span></p>
<h2>The Hike</h2>
<p><span>Straight after arriving to Singapore
having been awake for ~30hrs Patrick and I had the ill-conceived idea to
join a 13km hike through the hot and humid Singaporean
jungle.</span></p>
<p><img src="https://ryan.freumh.org/images/2025-10-10-singapore-hike.png"></p>
<p><span>We did see some long-tailed macaques
though!</span></p>
<p><img src="https://ryan.freumh.org/images/2025-10-11-monkeys.jpg"></p>
<h2>Geowiki Datastructure</h2>
<p><span>I had a great chat with <a href="https://www.cse.iitd.ac.in/~aseth/">Aadi</a> after <a href="https://conf.researchr.org/details/icfp-splash-2025/propl-2025-papers/5/GPU-accelerated-Hydrology-Algorithms-for-On-prem-Computation-Flow-accumulation-Drai">our</a>
<a href="https://conf.researchr.org/details/icfp-splash-2025/propl-2025-papers/3/STACD-STAC-Extension-with-DAGs-for-Geospatial-Data-and-Algorithm-Management">respective</a>
<a href="https://conf.researchr.org/home/icfp-splash-2025/propl-2025">talks</a>
at <a href="https://conf.researchr.org/home/icfp-splash-2025/propl-2025">PROPL</a>
on how Brigraphs might just be the foundation for a datastructure behind
a unified view of physical world. I’ve typed up my thoughts on this
below.</span></p>
<blockquote>
<p><span>The vast about of data about
our physical world lies in isolated silos. In order to use this data
across administrative domains and organisations, from a plurality of
sources, with real-time updates and monitoring, we need a shared model
of the world. A bigraph-like structure with a spatial hierarchy with
connections between nodes is a natural fit for such a model.</span></p>
<p><span>For example, watershed
datasets naturally describes a hierarchy from raster data, water bodies
and farm plots, micro water sheds, to water sheds. Irrigation, flow, and
aquifer data connects these nodes. Different use cases may call for
different hierarchies in parallel, such as forestry stands and
administrative boundaries (reminiscent of bigraphs with
sharing).</span></p>
<p><span>These nodes can be associated
with static data (e.g. polygons) and temporal data (e.g. water levels
over time). With a unified view of this world, we can make inferences
using data from across the board, e.g. what happens to the watershed
<em>here</em> if there’s reforestation <em>there</em>. We can aggregated
data from a variety of sources, like as remote earth observation, ground
source observation, and sensors in the environment. Changes can be peer
reviewed up the spatial hierarchy, with access control based on roles
and membership organisation.</span></p>
<p><span>This enables a unified space
for real-time monitoring. So one organisation can publish remote sensing
data to the structure, another can publish sensor data, and this can be
aggregated by authorities to provide real-time flood warnings to a
spatial domain in a pub-sub system. This requires code to live alongside
the data in a processing pipeline with incremental
computation.</span></p>
</blockquote>
<p><span>An open question I have is
what the architecture of this system would be: centralized like
Wikipedia? Federated like ActivityPub? Delegated like the DNS? Given the
intrinsic spatial hierarchy, and countries strict regulation around this
data, the latter seems the most appropriate.</span></p>
<h2>Pac</h2>
<p><span>People were excited to hear about the work
behind <a href="https://github.com/fn06/pac">Pac</a>.</span></p>
<p><span>WIP</span></p>

