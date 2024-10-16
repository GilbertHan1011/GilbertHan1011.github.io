---
layout: page
title: linkSet
description: R package for handling promoter-enhancer interactions.
img: assets/img/linkSet_log.png
importance: 2
category: work
giscus_comments: true
---
Recently, I’m working on integrating epigenomic data and single-cell data to identify long-distance enhancers. During this process, I discovered a gap in existing data structures for managing promoter-enhancer interactions. To address this, I developed a
lightweight yet powerful R package designed to efficiently handle all aspects of enhancer-gene regulation.

Now, this software are handy to import from various formats, like GRange and Interaction Set. It also enables basic operation like subsetting and GRange methods. The full documents are avaliable [here](https://gilberthan1011.github.io/linkSet/articles/linkSet.html). Recently, I'll add statistical methods and visualization methods soon!


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/proj_linkset/overview.png" title="overview" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Overview of LinkSet
</div>
