---
title: Programming for the Planet at ICFP/SPLASH 2025
description: Report on second Programming for the Planet workshop featuring papers
  on climate modeling, geospatial computation and planetary-scale collaborative systems.
url: https://anil.recoil.org/notes/icfp25-propl
date: 2025-10-05T00:00:00-00:00
preview_image: https://anil.recoil.org/images/propl25-header.640.webp
authors:
- Anil Madhavapeddy
source:
---

<p>This is part 1 of 5 of a <a href="https://anil.recoil.org/notes/icfp25">series</a> of posts<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> about ICFP 2025.</p>
<p>The <a href="https://popl24.sigplan.org/home/propl-2024">first outing</a> of PROPL was
last year in London, and this time around <a href="https://dorchard.github.io">Dominic Orchard</a> and I invited <a href="https://kcsrk.info">KC Sivaramakrishnan</a> to be
the PC chair and <a href="https://anil.recoil.org/notes/propl-at-splash">held it at ICFP/SPLASH</a>. The uptake was
encouraging, and we got enough submissions to have a proper <a href="https://dl.acm.org/doi/proceedings/10.1145/3759536">published
proceedings</a> in the ACM
Digital Library for the first time! Our <a href="https://anil.recoil.org/papers/2025-propl">proceedings summary</a>
is a quick read to give you an idea of the breadth of the papers and talks this year.</p>
<p><a href="https://dl.acm.org/action/showFmPdf?doi=10.1145/3759536"> <img src="https://anil.recoil.org/images/propl25-header.webp" alt="%c" title="A summary of the 6 papers, 9 talks and provocations that appeared."> </a></p>
<p>The workshop itself had slightly less in-person attendance than last year, but
this was due to the heavily multi-track structure of ICFP. We had less <a href="https://conf.researchr.org/home/icfp-splash-2025/propl-2025#program">total
time</a> than
last year as the morning slot was taken up by the ICFP keynote (the
awesome <a href="https://raintown.org">Satnam Singh</a>), and with six parallel sessions people were filtering in and
out of all the events. The online attendance made up for it, with quite a few
people tracking the <a href="https://www.youtube.com/watch?v=IIRJeleXeuU">SIGPLAN live
stream</a>.</p>
<p>The <a href="https://dl.acm.org/doi/proceedings/10.1145/3759536">papers</a> were exactly
what I'd dreamed would happen -- a variety of practitioners describing their
computational challenges mixed together with solutions. I'll summarise the day's
proceedings next!</p>
<p><a href="https://anil.recoil.org/slides/propl25-intro.pdf"> <img src="https://anil.recoil.org/images/icfp-6.webp" alt="%c" title="Dominic Orchard opening the PROPL25 workshop"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#computational-challenges" class="anchor" aria-hidden="true"></a>Computational challenges</h2>
<p>The first batch of talks I'll cover were about how to specify some core
computational models related to climate and biodiversity science.</p>
<p>Firstly, climate modelling was up with Chinmayi Prabhu
<a href="https://www.youtube.com/watch?v=IIRJeleXeuU&amp;t=18092s">presenting</a> a paper on
<a href="https://dl.acm.org/doi/10.1145/3759536.3763801">climate model coupler verification</a> that discussed
the difficulty of folding multiple global climate models into combined ones,
something that is normally done via (underspecified and somewhat black magic)
<a href="https://www.metoffice.gov.uk/research/climate/understanding-climate/coupled-modelling">coupler</a>
components. Chinmayi had found some bugs in production coupler code, and
described a hybrid verification strategy that used both static and runtime
techniques to improve the state of affairs.</p>
<blockquote>
<p>The continuous exchange of data through couplers creates the risk of subtle
errors propagating across components, potentially distorting scientific
conclusions. In this paper, we argue for lightweight formal verification
techniques applied at the coupler interface to improve both coupler and model
correctness.
<cite>-- <a href="https://dl.acm.org/doi/10.1145/3759536.3763801">Towards Modelling and Verification of Coupler Behaviour in Climate Models</a></cite></p>
</blockquote>
<p><a href="https://www.youtube.com/watch?v=IIRJeleXeuU&amp;t=18092s"> <img src="https://anil.recoil.org/images/icfp-11.webp" alt="%c" title="Chinmayi Prabhu Baramashetru presenting her work on climate coupler verification"> </a></p>
<p>Then we <a href="https://youtu.be/IIRJeleXeuU?t=12711">heard</a> about <a href="https://dl.acm.org/doi/10.1145/3759536.3763805">hydrology modeling using GPUs</a> over in India, where
algorithms to trace the path of surface water flows (e.g. flow accumulation,
watershed delineation or runoff simulation) are hard to execute for large areas
at reasonably fine spatial and temporal resolutions.</p>
<blockquote>
<p>Libraries like GDAL that use multi-threaded CPU-based implementations running
on a single host may be slow, and distributed infrastructures like Google
Earth Engine may not support the kind of computational primitives required by
these algorithms.</p>
<p>We have developed a GPU-accelerated framework that
re-engineers these four algorithms and is able to process areas as large as
river basins of 250,000 km2 on commodity GPU workstations.
<cite>-- <a href="https://dl.acm.org/doi/10.1145/3759536.3763805">GPU-Accelerated Hydrology Algorithms for On-Prem Computation</a></cite></p>
</blockquote>
<p><a href="https://www.cse.iitd.ac.in/~aseth/">Aadi Seth</a> explained not only the basics of flow algorithms, but why it's
essential to GPU accelerate them to get a reasonable spatial scale and
resolution. They ended up with a set of small parallelizable primitives that are either
pixel independent, or a sequence of 'long pixel' operations that need more
coordination.</p>
<p><a href="https://youtu.be/IIRJeleXeuU?t=12711"> <img src="https://anil.recoil.org/images/icfp-13.webp" alt="%c" title="Aadi Seth presenting his work on hydrology modeling in India"> </a></p>
<p>Continuing on in the previous theme of novel computation models, <a href="https://mynameismwd.org">Michael Dales</a> presented "<a href="https://anil.recoil.org/papers/2025-yirgacheffe">Yirgacheffe: A Declarative Approach to Geospatial Data</a>",
a new <a href="https://yirgacheffe.org">Python library</a> that
allows spatial algorithms to be implemented concisely and supports parallel
execution and resource management. <em>(<a href="https://digitalflapjack.com/weeknotes/2025-10-13/">read more...</a>)</em></p>
<p>Michael used our work on the <a href="https://anil.recoil.org/papers/2024-life">LIFE biodiversity metric</a> as the
motivating usecase <a href="https://digitalflapjack.com/weeknotes/2025-10-13/">for his demo</a>, as we have petabytes
of rasters processed using Yirgacheffe.  He also set up a nice <a href="https://yirgacheffe.org">new
homepage</a> for the project and used PROPL to launch it,
including <a href="https://digitalflapjack.com/blog/marimo/">adopting</a> a shiny new
executable notebook called <a href="https://marimo.io">Marimo</a> which I liked the look
of!</p>
<p><a href="https://www.youtube.com/live/IIRJeleXeuU?t=23927s"> <img src="https://anil.recoil.org/images/icfp-36.webp" alt="%c" title="Michael Dales showing the LIFE biodiversity map calculated using Yirgacheffe"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#geospatial-data-management" class="anchor" aria-hidden="true"></a>Geospatial data management</h2>
<p>Switching tack from computation to managing large-scale datasets, we had a
number of papers discussing how to orchestrate these both for full execution
but also in a developer friendly way for local use.</p>
<p>The first extremely ambitious <a href="https://youtu.be/IIRJeleXeuU?t=4536">talk</a> was from <a href="https://orcid.org/0009-0007-3826-1125">Jean-Michel Lord</a> who
presented <a href="https://anil.recoil.org/papers/2025-programming-gbon">"Programming Opportunities for the Global Biodiversity Observation Network"</a>. GEOBON is a <a href="https://earthobservations.org/groups/geo-biodiversity-observation-network">global network</a> of researchers dedicated to improving the acquisition, coordination and delivery of biodiversity information at a global scale. This PROPL collaboration came up up after I met <a href="https://www.thegonzalezlab.org/">Andrew Gonzalez</a> at the
<a href="https://anil.recoil.org/notes/nas-rs-biodiversity">NAS</a> earlier in the year and learnt about <a href="https://boninabox.geobon.org/">BON-in-a-Box</a>, which uses <a href="https://www.tunbury.org/2025/07/02/bon-in-a-box/">Docker under the hood</a>!</p>
<p><a href="https://youtu.be/IIRJeleXeuU?t=4536"> <img src="https://anil.recoil.org/images/icfp-8.webp" alt="%c" title="Jean-Michel Lord illustrates the technology stack behind BON-in-a-Box"> </a></p>
<p>Jean-Michel described how off-the-shelf software could (almost) be enough to integrate the world's biodiversity dataset pipelines, but needed some help from their maintainers. Most notably, BON-in-a-box facilitates peer review of <em>computation pipelines</em> (as opposed to the science underpinning them), which is the first time I've seen peer review applied to scientific code. This "connecting the dots" across diverse biodiversity datasets is vital towards building a comprehensive model of life on this planet, and computer science is a crucial piece of the puzzle to make sense of all the data. <a href="https://www.thegonzalezlab.org/">Andrew Gonzalez</a> also just announced that all the major players are getting together later this month at the <a href="https://www.livingdata2025.com/">Living Data 2025</a> conference in Colombia, further underlying how timely getting <a href="https://www.linkedin.com/posts/andrew-gonzalez-23589146_livingdata2025-boninabox-biodiversitymonitoring-activity-7386243770066817024-SV57">involved</a> with BON-in-a-Box is.</p>
<p><a href="https://www.cse.iitd.ac.in/~aseth/">Aadi Seth</a> had a second paper on the challenge of building <a href="https://dl.acm.org/doi/10.1145/3759536.3763803">spatio-temporal catalogue dataflow
graphs</a>. These
<a href="https://stacspec.org/en/">catalogues</a> are really important in environmental
science to <a href="https://anil.recoil.org/notes/nas-rs-biodiversity">connect the dots</a> in the often "gappy" field
datasets. The STAC-D extension proposes dataflow pipelines to help compose
these together via YAML specifications that describe dependencies between the
(large) datasets involved.</p>
<blockquote>
<p>We propose STACD (STAC extension with DAGs), an extension to STAC
specifications that incorporates Directed Acyclic Graph (DAG) representations
along with defining algorithms and version changes in the workflows. We also
provide a reference implementation on Apache Airflow to demonstrate STACD
capabilities such as selective recomputation when some datasets or algorithms
in a DAG are updated, complete lineage construction for a dataset, and
opportunities for improved collaboration and distributed processing that
arise with this standard.
<cite>-- <a href="https://dl.acm.org/doi/10.1145/3759536.3763803">STAC Extension with DAGs</a><cite></cite></cite></p>
</blockquote>
<p>This one really reminded me of the work I did ages ago with <a href="https://research.google/people/derekmurray/?&amp;type=google">Derek G Murray</a> and <a href="https://cs.brown.edu/people/malte/">Malte Schwarzkopf</a>
on the <a href="https://anil.recoil.org/papers/2011-nsdi-ciel">CIEL execution engine</a>. Dynamic dataflow graphs seem
to come up again and again as the beating heart of coordination languages, and
we've made surprisingly little progress in making them more ergonomic. Some
combination of Docker and <a href="https://simon.peytonjones.org/assets/pdfs/build-systems-jfp.pdf">Build Systems à la Carte</a> would
go a long way here!</p>
<p>The third <a href="https://www.youtube.com/watch?v=IIRJeleXeuU&amp;t=6053s">talk</a> was from the the <a href="https://catalysts.org">Catalysts Foundation</a>
from India, and highlighted the importance of data science for promoting health
and wellbeing in some of the most vulnerable rural communities, who are at
serious risks of increasing intensity due to climate change.</p>
<blockquote>
<p>Climate change presents multifaceted public health challenges, from
heat-related mortality and vector-borne disease expansion to water
contamination and respiratory ailments. The <a href="https://doi.org/10.1016/S0140-6736(22)01540-9">2022 Lancet Countdown Report</a>
demonstrates a host of health effects of climate change ranging from
heat-related illness and mortality to the spread of vector-borne and
water-borne pathogens, to rising food insecurity as cropping patterns change.
Current public health systems lack integrated, real-time data capabilities to
identify vulnerable populations and coordinate timely responses to these
climate-induced health threats, particularly in resource-constrained
settings.
<cite>-- <a href="https://conf.researchr.org/details/icfp-splash-2025/propl-2025-papers/9/Precision-Action-Towards-Climate-and-Health-PATCH-">Precision Action Towards Climate and Health (PATCH)</a></cite></p>
</blockquote>
<p><a href="https://www.youtube.com/watch?v=IIRJeleXeuU&amp;t=6053s"> <img src="https://anil.recoil.org/images/icfp-9.webp" alt="%c" title="Prerak Shah shows the increasing risks faced by rural Indian communities"> </a></p>
<p>Prerak talked about the difficulty of combining geospatial data with machine
learning inference, and keeping track of the resulting outputs in a systematic
way. What I found particularly interesting about their "PATCH" system is that it not
only has core computing facilities (a health reporting platform, a spatial counterfactual
map for interventions and a communications channel for different stakeholders), but they
also extensively partner with local state governments in India (like the India <a href="https://en.wikipedia.org/wiki/India_Meteorological_Department">Meteorological Department</a> and the <a href="https://www.aiims.edu/index.php/en">All India Institutes of Medical Sciences</a>.</p>
<p><a href="https://patrick.sirref.org">Patrick Ferris</a> formalised many of the above problems with his thoughtful <a href="https://youtu.be/IIRJeleXeuU?t=15374">seminar</a> on "what we talk about when we talk about scientific programming". Patrick came up with three axes along which the scientific method of hypothesis falsification needs to operate when dealing with computational data: dynamism, scale and access controls. If you'd like to read more about Patrick's thoughts on this topic, his paper last year on "<a href="https://anil.recoil.org/papers/2024-uncertainty-cs">Uncertainty at scale: how CS hinders climate research</a>" is also well worth a read.</p>
<p><a href="https://youtu.be/IIRJeleXeuU?t=15374"> <img src="https://anil.recoil.org/images/icfp-31.webp" alt="%c" title="Patrick Ferris discusses difficult usecases for geospatial scientific computing"> </a></p>
<p>All these talks highlighted the difficulty of managing large and often very messy datasets in practise. So how do we move towards more principled platforms to fix the situation? That's what the next group of talks covered!</p>
<h2><a href="https://anil.recoil.org/news.xml#towards-a-giant-planetary-wiki-of-code" class="anchor" aria-hidden="true"></a>Towards a giant planetary wiki of code</h2>
<p>By far my favourite aspect of PROPL was he sheer ambition on display when it
comes to leveraging the network effects around computer technology to
accelerate the pace of environmental action. The next batch of papers is all
about evolving notebooks to be global scale!</p>
<p><a href="https://web.eecs.umich.edu/~comar/">Cyrus Omar</a> led the charge with his <a href="https://youtu.be/IIRJeleXeuU?t=16533">call</a> to action for "<a href="https://anil.recoil.org/papers/2025-fairground">A FAIR Case for a Live Computational Commons</a>", which
he described as a live planetary wiki! What is needed to turn something like
Wikipedia into a live, updated computation graph with incremental, real-time
visualisation? This would effectively be one giant program running and being
updated by tens of thousands of contributors in real time.</p>
<blockquote>
<p>This paper proposes Fairground, a computational commons designed as a
collaborative notebook system where thousands of scientific artifacts are
authored, collected, and maintained together in executable form in a manner
that is <a href="https://en.wikipedia.org/wiki/FAIR_data">FAIR</a>, reproducible, and
live by default. Unlike existing platforms, Fairground notebooks can
reference each other as libraries, forming a single planetary-scale live
program executed by a distributed scheduler.
<cite>-- <a href="https://anil.recoil.org/papers/2025-fairground.pdf">A FAIR Case for a Live Computational Commons</a>, Omar et al 2025</cite></p>
</blockquote>
<p>Many of the answers to how to do this lay in the talks at this week's
ICFP/SPLASH: programming languages with clean semantics for incremental
compilation, purely functional with effect tracking, and mergeable semantics
for managing large scale data structures. Cyrus' own <a href="https://hazel.org">Hazel</a>
language is a perfect example of an ergonomic interactive language that has
clean, functional semantics while retaining usability.</p>
<p><a href="https://youtu.be/IIRJeleXeuU?t=16533"> <img src="https://anil.recoil.org/images/icfp-32.webp" alt="%c" title="Cyrus Omar lays out his vision for a FAIR planetary wiki"> </a></p>
<p><a href="https://dynamicaspects.org/research/">Roly Perera</a> then switched tack and
<a href="https://youtu.be/IIRJeleXeuU?t=13998">discussed</a> how we might achieve more
transparent climate reporting, via a new class of "transparent programming
languages" like his own <a href="https://f.luid.org/">Fluid</a> project.</p>
<blockquote>
<p>With traditional print media, the figures, text and other content are
disconnected from the underlying data, making them hard to understand,
evaluate and trust. Digital media, such as online papers and articles,
present an opportunity to make visual artifacts which are connected to data
and able to reveal those fine-grained relationships to an interested user.
This would enable research outputs, news articles and other data-driven
artifacts to be more transparent, self-explanatory and explorable .
<cite> -- <a href="https://f.luid.org/">fluid, explorable, self-explanatory research outputs</a></cite></p>
</blockquote>
<p>Roly showed in his talk how the latest advances in Fluid helped to automate
providing "drill down" explanations for topics in the energy transition and
decarbonisation, adaptation to climate change, or risk mitigation strategies
that required policy changes justified by data.</p>
<p><a href="https://youtu.be/IIRJeleXeuU?t=13998"> <img src="https://anil.recoil.org/images/icfp-30.webp" alt="%c" title="Roly Perera presents his Fluid language"> </a></p>
<p>The <a href="https://f.luid.org/">Fluid website</a> has loads of interactive examples
for you to explore, so continue on there if interested in this topic.</p>
<p><a href="https://www.gla.ac.uk/schools/computing/staff/cristianurlea/">Cristian Urlea</a> then discussed the crucial problem of <a href="https://dl.acm.org/doi/10.1145/3759536.3763804">Bridging Disciplinary Gaps in
Climate Research</a>, since the
point of all these planetary computing systems is to make them accessible to
non-computer-science-expert users (the "<a href="https://dl.acm.org/doi/pdf/10.1145/3480947">vernacular programmers</a>").</p>
<blockquote>
<p>Current scientific computing practices pose major barriers to entry,
particularly for interdisciplinary researchers and those in low and
middle-income countries (LMICs). Challenges include steep learning curves,
limited access to expert support, and difficulties with legacy or
under-documented software. Drawing on real-world experiences, we identify
recurring obstacles in the usability, accessibility, and sustainability of
scientific software.
<cite>-- <a href="https://dl.acm.org/doi/10.1145/3759536.3763804">Bridging Disciplinary Gaps in Climate Research</a>, 2025</cite></p>
</blockquote>
<p>I greatly enjoyed the framing of this paper of "reimagining scientific
programming as a shared public good", a point also made by Roberto Di Cosmo
recently in his <a href="https://www.nature.com/articles/d41586-025-03196-0">Nature comment</a> that we must stop
treating code like an afterthought, and instead record, share and value it.</p>
<h2><a href="https://anil.recoil.org/news.xml#onto-programming-the-planet" class="anchor" aria-hidden="true"></a>Onto Programming the Planet!</h2>
<p>Once we have all these planetary scale notebooks, what sorts of new programs
might we run on them? The last group of talks covered some radically different
ideas here, and I was involved with all of them!</p>
<p>First, <a href="https://toao.com">Sadiq Jaffer</a> and <a href="https://www.cst.cam.ac.uk/people/ray25">Robin Young</a> <a href="https://youtu.be/IIRJeleXeuU?t=21133">presented</a> our own <a href="https://github.com/ucam-eo/geotessera">TESSERA</a> project, which is a new geospatial foundation model for programming with observations of the planet from space. This is pretty scifi stuff!</p>
<blockquote>
<p>Remote sensing observations from satellites are critical for scientists to
understand how our world is changing in the face of climate change,
biodiversity loss, and desertification. However, working directly with this
data is difficult. For any given satellite constellation, there are a
multitude of processed products, data volume is considerable, and for optical
imagery, users must contend with data sparsity due to cloud cover. This
complexity creates a significant barrier for domain experts who are not
specialists.</p>
<p>Pre-trained, self-supervised foundation models such as <a href="https://arxiv.org/abs/2506.20380">TESSERA</a> aim to solve this by offering pre-computed
global embeddings. These rich embeddings can be used in-place of raw remote
sensing data in a powerful “embedding-as-data” approach. For example, a
single 128-dimensional TESSERA embedding for a 10-meter point on Earth can
substitute for an entire year of optical and radar imagery, representing its
temporal and spectral characteristics. While this could democratise access to
advanced remote sensing-derived analytics, it also creates a new programming
challenge: a lack of tools designed for this new approach.</p>
<p><cite>-- <a href="https://conf.researchr.org/details/icfp-splash-2025/propl-2025-papers/14/Challenges-in-Practice-Building-a-Usable-Library-for-Planetary-Scale-Embeddings">Building a Usable Library for Planetary-Scale Embeddings</a>, Jaffer 2025</cite></p>
</blockquote>
<p><a href="https://youtu.be/IIRJeleXeuU?t=21133"> <img src="https://anil.recoil.org/images/icfp-12.webp" alt="%c" title="Sadiq Jaffer shows how to detect all the solar farms on the planet using TESSERA"> </a></p>
<p>Sadiq did a live demo by demonstrating the
<a href="https://github.com/ucam-eo/geotessera">Geotessera</a> Python library I've been
<a href="https://anil.recoil.org/notes/geotessera-python">hacking on</a>. He went one step further and coded up a
simple classifier that found most of the solar farms on the planet, with a demo
that ran in minutes on his laptop!</p>
<p>After this, <a href="https://ancazugo.github.io/">Andres Zuñiga-Gonzalez</a> <a href="https://youtu.be/IIRJeleXeuU?t=25009">showed</a> how he's been using airborne data to uncover socioeconomic stratification of urban nature in England, all via a pipeline that combines all sorts of geospatial algorithms to distill metrics about modern urban life.</p>
<blockquote>
<p>Using high-resolution LiDAR (Vegetation Object Model), Sentinel 2 imagery,
and open geospatial datasets for over 28 million buildings across England, we
integrate raster, vector, and socioeconomic data within a scalable
computational framework. Tree segmentation was performed using adaptive
local-maximum filtering, canopy cover estimated at 1 m resolution, and park
accessibility derived from network-based walking distances.</p>
<p>Inequality in access to nature was quantified via Gini coefficients and
modelled with spatial error regressions against socioeconomic deprivation.
Our results reveal that while most urban areas meet the 3-tree proximity
rule, fewer than 3% achieve 30% canopy cover, and only a minority satisfy all
three components simultaneously.
<cite>-- <a href="https://anil.recoil.org/papers/2025-england-nature">Airborne assessment uncovers socioeconomic stratification of urban nature in England</a></cite></p>
</blockquote>
<p><a href="https://youtu.be/IIRJeleXeuU?t=25009"> <img src="https://anil.recoil.org/images/icfp-14.webp" alt="%c" title="Andres sketches out the 3-30-300 rule of urban nature access"> </a></p>
<p>You can read more about this in his <a href="https://anil.recoil.org/papers/2025-england-nature">preprint</a> that
explains in detail the programming pipeline (including route finding across
tree canopies across the whole of England), and the bigger picture behind
<a href="https://anil.recoil.org/ideas/urban-vegetation">3-30-300 mapping</a>.</p>
<p>And last but definitely not least, <a href="https://ryan.freumh.org">Ryan Gibb</a> <a href="https://youtu.be/IIRJeleXeuU?t=22918">presented</a> possibly the most topical
talk for a programming language venue: repurposing the classic Robin Milner
<a href="https://en.wikipedia.org/wiki/Bigraph">bigraphs</a> for environmental goodness!
Ryan (and <a href="https://profiles.imperial.ac.uk/joshua.millar22">Josh Millar</a>) observe that existing distributed computing models poorly
capture spatial structure, hindering dynamic collaboration and access control.
They argue that that space must be treated as a first-class concept in
programming models, and use this to better coordinate the complex sensor
systems we need for environmental science.</p>
<p><a href="https://youtu.be/IIRJeleXeuU?t=22918"> <img src="https://anil.recoil.org/images/icfp-35.webp" alt="%c" title="Ryan Gibb goes full bigraph using Roy Ang's OSM translation!"> </a></p>
<p>It was also wonderful to see <a href="mailto:ra652@cam.ac.uk">Roy Ang</a> attend the workshop, as it was his <a href="https://anil.recoil.org/ideas/bigraphs-real-world">Part II project</a> which imported
<a href="https://openstreetmap.org">OpenStreetMap</a> into <a href="https://uog-bigraph.bitbucket.io/">BigraphER</a> that sparked off the idea in the first place.
You may also be interested in our preprint "<a href="https://anil.recoil.org/papers/2025-bifrost">An Architecture for Spatial Networking</a>" that explores this concept of spatial networking in more detail.</p>
<h1><a href="https://anil.recoil.org/news.xml#reflections-on-the-2nd-propl" class="anchor" aria-hidden="true"></a>Reflections on the 2nd PROPL</h1>
<p>As always, the corridor track of discussions after the conference was the most valuable part of attending PROPL. We had the opportunity to put up some posters during the main banquet session, and it was busy!</p>
<p></p><div class="video-center"><iframe title="PROPL poster session at ICFP/SPLASH banquet" src="https://crank.recoil.org/videos/embed/83e457b1-76d0-4e7e-b65f-be35544750c7" frameborder="0" allowfullscreen="" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="aspect-ratio: 16/9; width: 100%;"></iframe></div><p></p>
<p>One thing that leapt out at me from the discussions was the need for a <em>hosted</em> service with the ergonomics of Docker, the interactive flexibility of Jupyter, the peer community of Wikipedia, and the semantic cleanliness of Hazel. This is overwhelmingly difficult to do in a topdown manner, but we do have now a growing community of practitioners and computer scientists who share a vision of making this happen. I had long conversations with most of the attendees of PROPL about how we might make this a reality, and my plan is spend a good chunk of my sabbatical time this year hacking on this.</p>
<p>The other really energizing thing was seeing all the side hacking going on. I
spotted <a href="https://patrick.sirref.org">Patrick Ferris</a> porting all the Python Geotessera code over to OCaml in a
skunkworks effect of which I heartily approve. I looked over the Hazel UI with
<a href="https://web.eecs.umich.edu/~comar/">Cyrus Omar</a> to figure out what a planetary view might look like. <a href="https://www.cse.iitd.ac.in/~aseth/">Aadi Seth</a> figured
out spatial datastructures for the CoRE stack with <a href="https://ryan.freumh.org">Ryan Gibb</a> and <a href="https://ancazugo.github.io/">Andres Zuñiga-Gonzalez</a> while we
stood around in the blazing Singapore heat. <a href="https://cseweb.ucsd.edu/~mcoblenz/">Michael Coblenz</a> lead a <a href="https://conf.researchr.org/details/icfp-splash-2025/hatra-2025-papers/8/Discussion">session at
HATRA</a>
on user centric approaches to types and reasoning assistants.</p>
<p>All of these strands could weave together powerfully into groundup systems that
solve the problems we've been defining at PROPL! I'm feeling
<a href="https://youtu.be/IIRJeleXeuU?t=26691">energized</a> and tired, and look forward
to continuing the discussions started in London (2024) and Singapore (2025)
into real systems in 2026!</p>
<p><img src="https://anil.recoil.org/images/icfp-33.webp" alt="%c" title="Ilya Sergey had the genius idea of having a fridge magnet booth
at the reception, which the organising committee for PROPL took full advantage of. See my fridge for more."></p>
<p><img src="https://anil.recoil.org/images/icfp-34.webp" alt="%c" title="As a metaphor about the race we are running to mitigate the worst effects of climate change using computer science, I left Sadiq in the dust at the Skyline Luge. We were both outclassed soundly by a bunch of ten year olds who had just finished school, however."></p>

<div class="footnotes"><ol><li><p></p><p>See also in the <a href="https://anil.recoil.org/notes/icfp25">ICFP25</a> series: <a href="https://anil.recoil.org/notes/icfp25-propl">chairing PROPL25</a>, the <a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a>, <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">multicore at Jane Street and Docker</a>, <a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX IO</a> and <a href="https://anil.recoil.org/notes/icfp25-what-i-learnt">what I learnt</a>.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Eyres et al (2025). LIFE: A metric for mapping the impact of land-cover change on global extinctions. <a href="https://doi.org/10.1098/rstb.2023.0327" target="_blank"><i>10.1098/rstb.2023.0327</i></a></li>
<li>Madhavapeddy (2025). What I learnt at the National Academy of Sciences US-UK Forum on Biodiversity. <a href="https://doi.org/10.59350/j6zkp-n7t82" target="_blank"><i>10.59350/j6zkp-n7t82</i></a></li>
<li>Dales et al (2025). Yirgacheffe: A Declarative Approach to Geospatial Data. Association for Computing Machinery. <a href="https://doi.org/10.1145/3759536.3763806" target="_blank"><i>10.1145/3759536.3763806</i></a></li>
<li>Madhavapeddy (2025). Holding an OxCaml tutorial at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/55bc5-x4p75" target="_blank"><i>10.59350/55bc5-x4p75</i></a></li>
<li>Madhavapeddy (2025). What I learnt at ICFP/SPLASH 2025 about OCaml, Hazel and FP. <a href="https://doi.org/10.59350/w1jvt-8qc58" target="_blank"><i>10.59350/w1jvt-8qc58</i></a></li>
<li>Millar et al (2025). An Architecture for Spatial Networking. arXiv. <a href="https://doi.org/10.48550/arXiv.2507.22687" target="_blank"><i>10.48550/arXiv.2507.22687</i></a></li>
<li>Madhavapeddy et al (2025). Proceedings of the 2nd ACM SIGPLAN International Workshop on Programming for the Planet. <a href="https://doi.org/10.1145/3759536" target="_blank"><i>10.1145/3759536</i></a></li>
<li>Madhavapeddy (2025). It's time to go post-POSIX at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/mch1m-8a030" target="_blank"><i>10.59350/mch1m-8a030</i></a></li>
<li>Madhavapeddy (2025). A Roundup of ICFP/SPLASH 2025 happenings. <a href="https://doi.org/10.59350/4jf5k-01n91" target="_blank"><i>10.59350/4jf5k-01n91</i></a></li>
<li>Omar et al (2025). A FAIR Case for a Live Computational Commons. Association for Computing Machinery. <a href="https://doi.org/10.1145/3759536.3763802" target="_blank"><i>10.1145/3759536.3763802</i></a></li>
<li>Madhavapeddy (2025). 2nd Programming for the Planet workshop CFP out. <a href="https://doi.org/10.59350/728q9-5ct54" target="_blank"><i>10.59350/728q9-5ct54</i></a></li>
<li>Madhavapeddy (2025). GeoTessera Python library released for geospatial embeddings. <a href="https://doi.org/10.59350/7hy6m-1rq76" target="_blank"><i>10.59350/7hy6m-1rq76</i></a></li>
<li>Madhavapeddy (2025). Jane Street and Docker on moving to OCaml 5 at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/3jkaq-d3398" target="_blank"><i>10.59350/3jkaq-d3398</i></a></li>
<li>Zuniga-Gonzalez et al (2025). Airborne assessment uncovers socioeconomic stratification of urban nature in England. arXiv. <a href="https://doi.org/10.48550/arXiv.2510.13861" target="_blank"><i>10.48550/arXiv.2510.13861</i></a></li>
<li>Romanello et al (2022). The 2022 report of the Lancet Countdown on health and climate change: health at the mercy of fossil fuels. The Lancet. <a href="https://doi.org/10.1016/S0140-6736(22)01540-9" target="_blank"><i>10.1016/S0140-6736(22)01540-9</i></a></li>
<li>Baramashetru et al (2025). Towards Modelling and Verification of Coupler Behaviour in Climate Models. <a href="https://doi.org/10.1145/3759536.3763801" target="_blank"><i>10.1145/3759536.3763801</i></a></li>
<li>Kumar et al (2025). GPU-Accelerated Hydrology Algorithms for On-Prem Computation: Flow Accumulation, Drainage Lines, Watershed Delineation, Runoff Simulation. <a href="https://doi.org/10.1145/3759536.3763805" target="_blank"><i>10.1145/3759536.3763805</i></a></li>
<li>Laud et al (2025). STACD: STAC Extension with DAGs for Geospatial Data and Algorithm Management. <a href="https://doi.org/10.1145/3759536.3763803" target="_blank"><i>10.1145/3759536.3763803</i></a></li>
<li>Urlea et al (2025). Bridging Disciplinary Gaps in Climate Research through Programming Accessibility and Interdisciplinary Collaboration. <a href="https://doi.org/10.1145/3759536.3763804" target="_blank"><i>10.1145/3759536.3763804</i></a></li>
<li>Shaw (2020). Myths and mythconceptions: what does it mean to be a programming language, anyhow?. Proceedings of the ACM on Programming Languages. <a href="https://doi.org/10.1145/3480947" target="_blank"><i>10.1145/3480947</i></a></li>
<li>Cosmo et al (2025). Stop treating code like an afterthought: record, share and value it. Nature. <a href="https://doi.org/10.1038/d41586-025-03196-0" target="_blank"><i>10.1038/d41586-025-03196-0</i></a></li>
<li>Feng et al (2025). TESSERA: Temporal Embeddings of Surface Spectra for Earth Representation and Analysis. arXiv. <a href="https://doi.org/10.48550/arXiv.2506.20380" target="_blank"><i>10.48550/arXiv.2506.20380</i></a></li></ul>
