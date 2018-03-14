---
title: "Visualization of peaks"
author: "Meeta Mistry"
date: "Thursday July 29th, 2017"
---

Approximate time: 45 minutes

## Learning Objectives
* Visualizing peak locations with respect to the TSS
* Generate bigWig files
* Use IGV to visualize BigWig, BED and data from ENCODE

## Visualization of ChIP-seq data

The first part of ChIP-sequencing analysis uses common processing pipelines, which involves the alignment of raw reads to the genome, data filtering, and identification of enriched signal regions (peak calling). In the second stage, individual programs allow detailed analysis of those peaks, biological interpretation, and visualization of ChIP-seq results.


There are various strategies for visualizing enrichment patterns and we will explore a few of them. To start, we will create bigWig files for our samples, a standard file format commonly used for ChIP-seq data visulaization.

### Creating bigWig files

The first thing we want to do is take our alignment files (BAM) and convert them into bigWig files. The bigWig format is an indexed binary format useful for dense, continuous data that will be displayed in a genome browser as a graph/track, but also is used as input for some of the visualization commands we will be running in `deepTools`. 

[`deepTools`](http://deeptools.readthedocs.org/en/latest/content/list_of_tools.html), is a suite of Python tools developed for the efficient analysis of high-throughput sequencing data, such as ChIP-seq, RNA-seq or MNase-seq. `deepTools` has a wide variety of commands that go beyond those that are covered in this lesson. We encourage you to look through the docuementation and explore on your own time. 

To create our bigWig files there are two tools that can be useful: `bamCoverage` and `bamCompare`. The former will take in a single BAM file and return to you a bigWig file. The latter allows you to normalize two files to each other (i.e. ChIP sample relative to input).

<img src="../img/bam_to_bigwig.png">

*Image aquired from [deepTools documentation](http://deeptools.readthedocs.io/en/latest/content/tools/bamCoverage.html?highlight=bigwig) pages*

We will begin by creating a directory for the visualization output and loading the required modules to run `deepTools`.

```bash
$ cd ~/chipseq/results/
$ mkdir visualization
```

```bash
$ module load gcc/6.2.0  python/2.7.12
$ module load deeptools/2.5.3 

```

Create the bigWig files:

For just one file and copy over the full dataset ones.

```bash
$ bamCompare -b1 bowtie2/H1hesc_Nanog_Rep1_chr12_aln.bam -b2 bowtie2/H1hesc_Input_Rep1_chr12_aln.bam -o visualization/Nanog_Rep1_chr12.bw 2> visualization/Nanog_Rep1_bamcompare.log

$ bamCompare -b1 bowtie2/H1hesc_Nanog_Rep2_chr12_aln.bam -b2 bowtie2/H1hesc_Input_Rep2_chr12_aln.bam -o visualization/Nanog_Rep2_chr12.bw 2> visualization/Nanog_Rep2_bamcompare.log
```

```bash
$ bamCompare -b1 bowtie2/H1hesc_Pou5f1_Rep1_chr12_aln.bam -b2 bowtie2/H1hesc_Input_Rep1_chr12_aln.bam -o visualization/Pou5f1_Rep1_chr12.bw 2> visualization/Pou5f1_Rep1_bamcompare.log

$ bamCompare -b1 bowtie2/H1hesc_Pou5f1_Rep2_chr12_aln.bam -b2 bowtie2/H1hesc_Input_Rep2_chr12_aln.bam -o visualization/Pou5f1_Rep2_chr12.bw 2> visualization/Pou5f1_Rep2_bamcompare.log
```

Compute the matrix (for TSS):
- plot Nanog and Pou5f1 on same plot
- plot them separately each with a heatmap
- 


Compute matrix (for specific binding sites):
- look at Nanog enrichment on Pou5f1 bidning sites
- look at Pou5f1 enrichment on Nanog sites
- 









### Differential enrichment
To provide a more complex picture of biological processes in a cell, many studies aim to compare different datasets obtained by ChIP-seq. 








### IGV: Viewing files in a genome browser

* Copy over the bigWig files to your laptop using filezilla or scp. 
* Copy over the BEDtools overlap/intersect files to your computer.

* Start IGV and load the 2 rep1 files, and the overlap BED files. You will notice that there are positive and negative values on the track, what do you think this denotes in the context of normalization?

> You can generate a simple, non-normalized bigWig with `bamCoverage` and you won't see any negative values. 

* Now load the `Nanog_vs_Pou5f1_edgeR_sig.bed` and `Nanog_vs_Pou5f1_deseq2_sig.bed` (output of DiffBind, in your chipseq R project) into IGV.

* Finally, we are going to visually compare our output to the output from the full dataset from ENCODE, by loading that data from the IGV server.

***
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*
