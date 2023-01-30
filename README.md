> **NOTE**: The materials in this repository are **no longer actively maintained.**  More recent content can be found at: https://hbctraining.github.io/Intro-to-ChIPseq-flipped/

## OLD - Introduction to ChIP-seq using high performance computing


| Audience | Computational Skills | Prerequisites | Duration |
:----------|:----------|:----------|:----------|
| Biologists | Beginner/Intermediate | None | 3-day workshop (~19.5 hours of trainer-led time)|

### Description

This repository has teaching materials for a 3-day Introduction to ChIP-sequencing data analysis workshop. This workshop focuses on teaching basic computational skills to enable the effective use of an high-performance computing environment to implement a ChIP-seq data analysis workflow. It includes an introduction to shell (bash) and shell scripting. In addition to running the ChIP-seq workflow from FASTQ files to peak calls and nearest gene annotations, the workshop covers best practice guidlelines for ChIP-seq experimental design and data organization/management and quality control.

> These materials were developed for a trainer-led workshop, but are also amenable to self-guided learning.

### Learning Objectives

1.	Understand the necessity for, and use of, the command line interface (bash) and HPC for analyzing high-throughput sequencing data.
2.	Understand best practices for designing a ChIP-seq experiment and analysis the resulting data.

### Lessons
**[Click here](schedule/2-day.md) for links to lessons and the suggested schedule**

### Dataset

### Installation Requirements

Download the most recent versions of R and RStudio for your laptop:

 - [R](http://lib.stat.cmu.edu/R/CRAN/) (version 3.5.0 or above)
 - [RStudio](https://www.rstudio.com/products/rstudio/download/#download)
 
> **NOTE**: When installing the following packages, if you are asked to select (a/s/n) or (y/n), please select “a” or "y" as applicable.

(1) Install the below packages on your laptop from CRAN. You DO NOT have to go to the CRAN webpage; you can use the following function to install them:


```r
install.packages("BiocManager")
install.packages("tidyverse")
```

**Note that these package names are case sensitive!**


(2) Install the below packages from Bioconductor. Load BiocManager, then run BiocManager's `install()` function 7 times for the 7 packages:

```r
library(BiocManager)
install("insert_first_package_name_in_quotations")
install("insert_second_package_name_in_quotations")
& so on ...
```

Note that these package names are case sensitive!

```r
ChIPQC
ChIPseeker
DiffBind
clusterProfiler
AnnotationDbi
TxDb.Hsapiens.UCSC.hg19.knownGene
EnsDb.Hsapiens.v75
org.Hs.eg.db
```

> **NOTE:** The library used for the annotations associated with genes (here we are using `TxDb.Hsapiens.UCSC.hg19.knownGene` and `EnsDb.Hsapiens.v75`) will change based on organism (e.g. if studying mouse, would need to install and load `TxDb.Mmusculus.UCSC.mm10.knownGene`). The list of different organism packages are given [here](https://github.com/hbctraining/Training-modules/raw/master/DGE-functional-analysis/img/available_annotations.png).

(3) Finally, please check that all the packages were installed successfully by **loading them one at a time** using the `library()` function.  

```r
library(tidyverse)
library(ChIPQC)
library(ChIPseeker)
library(DiffBind)
library(clusterProfiler)
library(AnnotationDbi)
library(TxDb.Hsapiens.UCSC.hg19.knownGene)
library(EnsDb.Hsapiens.v75)
```

(4) Once all packages have been loaded, run sessionInfo().  

```r
sessionInfo()
```

***
*These materials have been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

* *Some materials used in these lessons were derived from work that is Copyright © Data Carpentry (http://datacarpentry.org/). 
All Data Carpentry instructional material is made available under the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).*

