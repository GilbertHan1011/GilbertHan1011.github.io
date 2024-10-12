---
layout: post
title: All about scATAC-seq's quality control
date: 2024-04-21 21:01:00
description: I described a new way for scATAC-seq quality control
tags: formatting images
categories: sample-posts
thumbnail: assets/img/2024-04-21-scATAC-quality-control/1713701483232.png
---
One main challenge in scATAC-seq analysis is to distinguish the good cells (high quality cells), the bad cells(low quality cells), and the ugly cells (doublet  cells).

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713701483232.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>

## The bad---- low quality cell

### Why should I perform quality control

The low quality cells and the doublet cells can influence the downstream analysis, such as clustering (form a unique cluster), trajectory analysis (create artificial trajectories) and so on. [OSCA book](https://bioconductor.org/books/3.18/OSCA.basic/quality-control.html#ref-lun2018overcoming) has great explaination about this issue.

### What metrics are used to distinguish between the good and the bad

Different frameworks perfer different metrics to perform quality control. For example, [ArchR](https://www.archrproject.com/bookdown/per-cell-quality-control.html) and [SnapATAC2](https://kzhang.org/SnapATAC2/tutorials/pbmc.html) utilized TSS enrichment and number of unique fragments. And [Muon](https://muon.readthedocs.io/en/latest/omics/atac.html#id1) use TSS enrichment and nucleosome signal to perform quality control. To compare what's best strategy to preform quality control, I firstly classified the metrics to following categories.

#### Sequencing depth

Similar with scRNAseq datasets, scATAC has sequencing depth, which related with total number of fragments in peaks, unique fragment number, unique peak number, and so on. Low sequencing depth means low quailty. High sequencing depth may related with doublets, nuclei clumps, or other artefacts.

#### Transcriptional start site (TSS) enrichment score

TSS enrichment score is the ratio of fragments centered at the TSS to fragments in TSS-flanking regions. This metrics is used in nearly every scATAC framework. Extremely high TSS scores and extremely low TSS scores are both not good signs.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713770714671.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


#### Fraction of fragments in peaks (FRiP)

Represents the fraction of all fragments that fall within ATAC-seq peaks. This is a good metric to show signal-noise ratio.

#### Nucleosome banding pattern

Because chromosomes are organized into units called nucleosomes, fragment size can exhibit periodic distribution correlated with nucleosome size. Fragments in high quality cells are nucleosome-free (<147 bp).



<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713771076735.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


#### Others

For example, duplication fraction in [pycistopic](https://pycistopic.readthedocs.io/en/latest/tutorials.html), ratio reads in genomic blacklist regions in [signac](https://stuartlab.org/signac/articles/pbmc_vignette).


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713772397798.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


### How to choose quality control metrics?

A good metric typically effectively distinguishes between good and bad cells. So, we begin by exploring whether the above metrics are indicative.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713775198151.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


We observed that the four metrics seperate bad cells (red box) and good cells well. If we combine these metrics, the seperation are more clear.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713775982399.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


### How to filter cells base on QC metrics

While low quailty cells can be discovered with different reasons (i.e. low sequencing depth, high nucleosome signal), we noticed that high quality cells are usually clustered together. This give us a intuation, that we can select cells base on unspervise clustering methods, like Kmeans.

Base on this thought, I apply kmeans on QC metrics to assess wether it can select high quality cells.

I select TSS enrichment, nucleosome signal, percent of reads in peaks, number of fragment to be the feature. After normalization, I apply Kmeans.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713785029703.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


As we can see, Kmeans method is excellent at select high quality cells. Low quality cells are grouped together to form low-sequencing-depth cluster, high-nuleosome-signal cluster, and low-TSS score clusters.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713784536547.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


I tried different number of clusters in Kmeans. As we can see, this methods is very robust, high quality cells are always cluster togehter until cluster number increase to 6.

### What about other QC methods

I compared different QC methods provided by various framework. The results are show below.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713786089347.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


Not suprising, the results in different methods are similar. But what's the best quality control methods? I used total cell number - unique cell number to roughly estimate the performance of different methods.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713787023636.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>



As we can see, Kmeans methods has best performance in our datasets.

### What's filter methods of cellrangers?

You may notice that cellranger's quality control is very strict. While other methods leaves 18000-20000 cells, cellranger's qc only leaves 11577 cells. You can find the filter results in `filtered_peak_bc_matrix.h5` and `singlecell.csv`.  From their [pipeline](https://support.10xgenomics.com/single-cell-atac/software/pipelines/2.0/map/cr-atac), we can see it remove doublet and low quality cells with it's own methods.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713789253939.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


It use [cell-calling heuristic](https://kb.10xgenomics.com/hc/en-us/articles/360001892491-What-is-the-difference-between-the-filtered-and-raw-gene-barcode-matrix) to filter low sequencing depth cells. Due to its overly stringent filtering outcomes, I don't believe that using cellranger's filtering results from the outset is a good idea.

### Should I use strict QC threshold or loose QC threshold?

I suggest to use loose method at first. The worst outcome with a loose QC threshold is an arificial cluster/trajectory. Usually you can find them at downstream. However, if you begin with strict QC at first, you may never know what opportunities you've missed in terms of discoveries.

## The ugly ----- doublets

### Common strategy to find doublets

There are two strategies to detect doublets. One approch is based on simulation, the other is based on coverage. [Here](https://www.sc-best-practices.org/chromatin_accessibility/quality_control.html#doublet-detection) describe the difference between the two methods.

### What's the best way to detect doublets

As suggested by author of scDblFinder, it's better to combine the two methods (coverage-based and simulation-based) to discover doublets. As most frameworks' doublets detection methods are simulation-based, I compared the results run by scDblFinder, ArchR ([demuxlet](https://github.com/statgen/demuxlet)), and SnapATAC2 (scrublet). The more related with amulet (coverage-based method), the results are better.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713792040000.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>



As we can see, except ArchR showed bad performance, SnapATAC2 and scDblFinder both exhibit good performance.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2024-04-21-scATAC-quality-control/1713792200267.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The good, the bad, and the ugly in scATAC-seq
</div>


Actually, results of SnapATAC2(scrublet) and scDblFinder are similar. If you perfer strict quality control, you can use scDblFinder; else, you can use SnapATAC2.

## Take home message

- Kmeans are good strategy to filter low quality cells.
- Better not use cellranger filtered methods.
- Do not use ArchR to detect doublets.
- Combine Amulet and scDblFinder/SnapATAC2 to detect doublets.
