# All about scATAC-seq's quality control

One main challenge in scATAC-seq analysis is to distinguish the good cells (high-quality cells), the bad cells(low-quality cells), and the ugly cells (doublets cells).

<img src="/assets/image/2024-04-21-scATAC-quality-control/1713701483232.png" width="600">

## The bad---- low quality cell

### Why should I perform quality control

The low-quality cells and the doublet cells can influence the downstream analysis, such as clustering (forming a unique cluster), trajectory analysis (creating artificial trajectories), and so on. [OSCA book](https://bioconductor.org/books/3.18/OSCA.basic/quality-control.html#ref-lun2018overcoming) has a great explanation about this issue.

### What metrics are used to distinguish between the good and the bad

Different frameworks prefer different metrics to perform quality control. For example, [ArchR](https://www.archrproject.com/bookdown/per-cell-quality-control.html) and [SnapATAC2](https://kzhang.org/SnapATAC2/tutorials/pbmc.html) utilized TSS enrichment and several unique fragments. And [Muon](https://muon.readthedocs.io/en/latest/omics/atac.html#id1) uses TSS enrichment and nucleosome signal to perform quality control. To compare the best strategy to perform quality control, I first classified the metrics into the following categories.

#### Sequencing depth

Similar to scRNAseq datasets, scATAC has sequencing depth, which is related to total number of fragments in peaks, unique fragment number, unique peak number, and so on. Low sequencing depth means low quality. High sequencing depth may related to doublets, nuclei clumps, or other artifacts.

#### Transcriptional start site (TSS) enrichment score

TSS enrichment score is the ratio of fragments centered at the TSS to fragments in TSS-flanking regions. This metric is used in nearly every scATAC framework. Extremely high TSS scores and extremely low TSS scores are both not good signs.

![1713770714671](/assets//image/2024-04-21-scATAC-quality-control/1713770714671.png)

#### Fraction of fragments in peaks (FRiP)

Represents the fraction of all fragments that fall within ATAC-seq peaks. This is a good metric to show the signal-noise ratio.

#### Nucleosome banding pattern

Because chromosomes are organized into units called nucleosomes, fragment size can exhibit periodic distribution correlated with nucleosome size. Fragments in high-quality cells are nucleosome-free (<147 bp).

<img src="/assets/image/2024-04-21-scATAC-quality-control/1713771076735.png" width="900">

#### Others

For example, duplication fraction in [pycistopic](https://pycistopic.readthedocs.io/en/latest/tutorials.html), ratio reads in genomic blacklist regions in [signac](https://stuartlab.org/signac/articles/pbmc_vignette).

![1713772397798](/assets/image/2024-04-21-scATAC-quality-control/1713772397798.png)

### How to choose quality control metrics?

A good metric typically effectively distinguishes between good and bad cells. So, we begin by exploring whether the above metrics are indicative.

![1713775198151]/assets/image/2024-04-21-scATAC-quality-control/1713775198151.png)

We observed that the four metrics separate bad cells (red box) and good cells well. If we combine these metrics, the separation is more clear.

![1713775982399](/assets/image/2024-04-21-scATAC-quality-control/1713775982399.png)

### How to filter cells based on QC metrics

While low-quality cells can be discovered for different reasons (i.e. low sequencing depth, high nucleosome signal), we noticed that high-quality cells are usually clustered together. This gives us an intuition, that we can select cells based on unsupervised clustering methods, like Kmeans.

Based on this thought, I apply Kmeans on QC metrics to assess whether it can select high-quality cells.

I select TSS enrichment, nucleosome signal, percent of reads in peaks, and number of fragments to be the feature. After normalization, I apply Kmeans.

![1713785029703](/assets/image/2024-04-21-scATAC-quality-control/1713785029703.png)

As we can see, the Kmeans method is excellent at selecting high-quality cells. Low-quality cells are grouped to form low-sequencing-depth clusters, high-nucleosome-signal clusters, and low-TSS score clusters.

![1713784536547](/assets/image/2024-04-21-scATAC-quality-control/1713784536547.png)

I tried different numbers of clusters in Kmeans. As we can see, this method is very robust, high-quality cells are always clustered together until the cluster number increases to 6.

### What about other QC methods

I compared different QC methods provided by various frameworks. The results are shown below.

<img src="/assets/image/2024-04-21-scATAC-quality-control/1713786089347.png" width="900">

Not surprisingly, the results in different methods are similar. But what are the best quality control methods? I used total cell number - unique cell number to roughly estimate the performance of different methods.

![1713787023636](/assets/image/2024-04-21-scATAC-quality-control/1713787023636.png)

As we can see, Kmeans method has the best performance in our datasets.

### What are the filter methods of cellrangers?

You may notice that Cellranger's quality control is very strict. While other methods leave 18000-20000 cells, cellranger's qc only leaves 11577 cells. You can find the filter results in `filtered_peak_bc_matrix.h5` and `singlecell.csv`.  From their [pipeline](https://support.10xgenomics.com/single-cell-atac/software/pipelines/2.0/map/cr-atac), we can see it removes doublet and low-quality cells with its methods.

![1713789253939](/assets/image/2024-04-21-scATAC-quality-control/1713789253939.png)

It uses [cell-calling heuristic](https://kb.10xgenomics.com/hc/en-us/articles/360001892491-What-is-the-difference-between-the-filtered-and-raw-gene-barcode-matrix) to filter low sequencing depth cells. Due to its overly stringent filtering outcomes, I don't believe that using cellranger's filtering results from the outset is a good idea.

### Should I use a strict QC threshold or a loose QC threshold?

I suggest using the loose method at first. The worst outcome with a loose QC threshold is an artificial cluster/trajectory. Usually, you can find them at downstream. However, if you begin with strict QC at first, you may never know what opportunities you've missed in terms of discoveries.

## The ugly ----- doublets

### Common strategy to find doublets

There are two strategies to detect doublets. One approach is based on simulation, the other is based on coverage. [Here](https://www.sc-best-practices.org/chromatin_accessibility/quality_control.html#doublet-detection) describes the difference between the two methods.

### What's the best way to detect doublets

As suggested by the author of scDblFinder, it's better to combine the two methods (coverage-based and simulation-based) to discover doublets. As most frameworks' doublets detection methods are simulation-based, I compared the results run by scDblFinder, ArchR ([demuxlet](https://github.com/statgen/demuxlet)), and SnapATAC2 (scrublet). The more related to amulet (coverage-based method), the results are better.

![1713792040000](/assets/image/2024-04-21-scATAC-quality-control/1713792040000.png)

As we can see, except ArchR showed bad performance, SnapATAC2 and scDblFinder both exhibited good performance.

<img src="/assets/image/2024-04-21-scATAC-quality-control/1713792200267.png" width="500">

The results of SnapATAC2 (scrublet) and scDblFinder are similar. If you prefer strict quality control, you can use scDblFinder; else, you can use SnapATAC2.

## Take home message

- Kmeans are a good strategy to filter low-quality cells.
- Better not use cellranger filtered methods.
- Do not use ArchR to detect doublets.
- Combine Amulet and scDblFinder/SnapATAC2 to detect doublets.
