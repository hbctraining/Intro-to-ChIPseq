---
title: "FastQC and alignment"
author: "Mary Piper, Radhika Khetani"
date: "June 28th, 2017"
---

Contributors: Mary Piper, Radhika Khetani

Approximate time: 60 minutes

## Learning Objectives

* Become familiar wih NGS best practices
* Evaluate the quality of your NGS data using FastQC
* Describe and use the Trimmomatic tool to perform quality trimming
* Describe parameters for performing alignment using Bowtie2

## Best practices for NGS Analysis 

Ok so now you are all set up and have begun your analysis. You have followed best practices to set up your analysis directory structure in a way such that someone unfamiliar with your project should be able to look at it and understand what you did and why. But there is more...:

1. **Make sure to use the appropriate software.** Do your research and find out what is best for the data you are working with. Don't just work with tools that you are able to easily install. Also, make sure you are using the most up-to-date versions! If you run out-of-date software, you are probably introducing errors into your workflow; and you may be missing out on more accurate methods.

2. **Keep up with the literature.** Bioinformatics is a fast-moving field and it's always good to stay in the know about recent developments. This will help you determine what is appropriate and what is not.  

3. **Do not re-invent the wheel.** If you run into problems, more often than not someone has already encountered that same problem. A solution is either already available or someone is working on it -- so find it! Ask colleagues or search/ask online forums such as [BioStars](https://www.biostars.org/).

4. **Testing is essential.** If you are using a tool for the first time, test it out on a single sample or a subset of the data before running your entire dataset through. This will allow you to debug quicker and give you a chance to also get a feel for the tool and the different parameters.

## Quality control of sequence reads

<img src="../img/chip_workflow_june2017_step1.png" width="700">

Now that we have our files and directory structure, we are ready to begin our ChIP-Seq analysis. For any NGS analysis method, our first step in the workflow is to explore the quality of our reads prior to aligning them to the reference genome and proceeding with downstream analyses. 

We will use FastQC to get a good idea of the overall quality of our data. We will use FastQC to identify whether any samples appear to be outliers, to examine our data for contamination.


### FASTQC

Let's run FastQC on all of our files. 

Start an interactive session with 2 cores if don't have one going, and change directories to the `raw_data` folder.

```bash
$ srun --pty -p short -t 0-12:00 --mem 8G -n 2 --reservation=HSPH bash

$ cd ~/chipseq/raw_data 

```

Now we need to load the FASTQC module to use the tool. Then we can run FASTQC on the Input Replicate 1 sample:

```bash
$ module load fastqc/0.11.3

$ fastqc H1hesc_Input_Rep1_chr12.fastq 
```

Now, move all of the `fastqc` files to the `results/fastqc` directory:

```bash
$ mv *fastqc* ../results/fastqc/
```

Transfer the FastQC zip file for Input replicate 1 to your local machine using FileZilla and view the report.

![fastqc](../img/fastqc_input_rep1.png)


Based on the sequence quality plot, we see across the length of the read the quality drops into the low range. Trimming could be performed from both ends of the sequences, or we can use an alignment tool that can acccount for this. 

> **NOTE:** If you are interested in learning more about the FASTQ file format and information on how to interpret the FASTQC reports we have a [QC lesson](https://hbctraining.github.io/Intro-to-rnaseq-hpc-O2/lessons/02_assessing_quality.html) as part of the RNA-seq workflow.
>
> FastQC has a really well documented [manual page](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) with [more details](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) about all the plots in the report. We also recommend looking at [this post](http://bioinfo-core.org/index.php/9th_Discussion-28_October_2010) for more information on what bad plots look like and what they mean for your data.

## Alignment

Now that we have assessed the quality of our sequence data, we are ready to align the reads to the reference genome. Most ChIP-seq experiments do not require gapped alignments because the sequenced reads do not contain them, unlike exon junctions in RNA-seq analyses; therefore, we do not need a splice-aware aligner. We can use a traditional short-read aligner to quickly and accurately align reads to the genome.

[Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) is a fast and accurate alignment tool that indexes the genome with an FM Index based on the Burrows-Wheeler Transform to keep memory requirements low for the alignment process. *Bowtie2* supports gapped, local and paired-end alignment modes and works best for reads that are at least 50 bp (shorter read lengths should use Bowtie1). By default, Bowtie2 will perform a global end-to-end read alignment, which is best for quality-trimmed reads. However, it also has a local alignment mode, which will perform soft-clipping for the removal of poor quality bases or adapters from untrimmed reads. We will use this option since we did not trim our reads.

> _**NOTE:** Our reads are only 36 bp, so technically we should explore alignment Bowtie1 to see if it is better. However, since it is rare that you will have sequencing reads with less than 50 bp, we will show you how to perform alignment using Bowtie2._

### Creating Bowtie2 index

To perform the Bowtie2 alignment, a genome index is required. **We previously generated the genome indices for you**, and they exist in the `reference_data` directory.

However, if you needed to create a genome index yourself, you would use the following command:

```bash
# DO NOT RUN

bowtie2-build <path_to_reference_genome.fa> <prefix_to_name_indexes>

```

> A quick note on shared databases for human and other commonly used model organisms. The O2 cluster has a designated directory at `/n/groups/shared_databases/` in which there are files that can be accessed by any user. These files contain, but are not limited to, genome indices for various tools, reference sequences, tool specific data, and data from public databases, such as NCBI and PDB. So when using a tool that requires a reference of sorts, it is worth taking a quick look here because chances are it's already been taken care of for you. 
>
>```bash
>$ ls -l /n/groups/shared_databases/igenome/
>```

### Aligning reads with Bowtie2

Since we have our indices already created, we can get started with read alignment. Change directories to the `bowtie2` folder:

```bash
$ cd ~/chipseq/results/bowtie2
```

Now let's load the module. We can find out more on the module on O2:

```bash
$ module spider bowtie2
```
You will notice that before we load this module we also need to load the gcc compiler (as will be the case for many of the NGS analysis tools on O2. Always check `module spider` first.)

```bash
$ module load gcc/6.2.0 bowtie2/2.2.9
```

We will perform alignment on our single raw FASTQ file, `H1hesc_Input_Rep1_chr12.fastq`. Details on Bowtie2 and its functionality can be found in the [user manual](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml); we encourage you to peruse through to get familiar with all available options.

The basic options for aligning reads to the genome using Bowtie2 are:

* `-p`: number of processors / cores
* `-q`: reads are in FASTQ format
* `--local`: local alignment feature to perform soft-clipping
* `-x`: /path/to/genome_indices_directory
* `-U`: /path/to/FASTQ_file
* `-S`: /path/to/output/SAM_file

```bash
$ bowtie2 -p 2 -q --local \
-x ~/chipseq/reference_data/chr12 \
-U ~/chipseq/raw_data/H1hesc_Input_Rep1_chr12.fastq \
-S ~/chipseq/results/bowtie2/H1hesc_Input_Rep1_chr12_aln_unsorted.sam

```

## Filtering reads

An important issue concerns the inclusion of multiple mapped reads (reads mapped to multiple loci on the reference genome). **Allowing for multiple mapped reads increases the number of usable reads and the sensitivity of peak detection; however, the number of false positives may also increase** [[1]](https://www.ncbi.nlm.nih.gov/pubmed/21779159/). Therefore we need to filter our alignment files to **contain only uniquely mapping reads** in order to increase confidence in site discovery and improve reproducibility. Since there is no parameter in Bowtie2 to keep only uniquely mapping reads, we will need to perform the following steps to generate alignment files containing only the uniquely mapping reads:

1. Change alignment file format from SAM to BAM
2. Sort BAM file by read coordinate locations
3. Filter to keep only uniquely mapping reads (this will also remove any unmapped reads)

### 1. Changing file format from SAM to BAM

While the SAM alignment file output by Bowtie2 is human readable, we need a BAM alignment file for downstream tools. Therefore, we will use [Samtools](http://samtools.github.io) to convert the file formats. The command we will use is `samtools view` with the following parameters:

* `-h`: include header in output
* `-S`: input is in SAM format
* `-b`: output BAM format
* `-o`: /path/to/output/file

```bash
$ samtools view -h -S -b \
-o H1hesc_Input_Rep1_chr12_aln_unsorted.bam \
H1hesc_Input_Rep1_chr12_aln_unsorted.sam
```

You can find additional parameters for the samtools functions in the [manual](http://www.htslib.org/doc/samtools-1.2.html).

### 2. Sorting BAM files by genomic coordinates

Before we can filter to keep the uniquely mapping reads, we need to sort our BAM alignment files by genomic coordinates. To perform this sort, we will use [Sambamba](http://lomereiter.github.io/sambamba/index.html), which is a tool that quickly processes BAM and SAM files. It is similar to SAMtools, but has unique functionality.
The command we will use is `sambamba sort` with the following parameters:

* `-t`: number of threads / cores
* `-o`: /path/to/output/file

```bash
$ sambamba sort -t 2 \
-o H1hesc_Input_Rep1_chr12_aln_sorted.bam \
H1hesc_Input_Rep1_chr12_aln_unsorted.bam 
```
> **Can we use `samtools` to do the sorting instead?** We could have also used `samtools` to perform the sort, however using `sambamba` gives us dual functionality. List the contents of the directory -- what do you see? The advantage to using `sambamba` is that along with the sort an index file is generated. If we used `samtools` this would have been a two-step process.

### 3. Filtering uniquely mapping reads

Finally, we can filter the uniquely mapped reads. We will use the `sambamba view` command with the following parameters:

* `-t`: number of threads / cores
* `-h`: print SAM header before reads
* `-f`: format of output file (default is SAM)
* `-F`: set [custom filter](https://github.com/lomereiter/sambamba/wiki/%5Bsambamba-view%5D-Filter-expression-syntax) - we will be using the filter to remove multimappers and unmapped reads.

```bash
$ sambamba view -h -t 2 -f bam \
-F "[XS] == null and not unmapped " \
H1hesc_Input_Rep1_chr12_aln_sorted.bam > H1hesc_Input_Rep1_chr12_aln.bam
```
We filtered out unmapped reads by specifying in the filter `not unmapped`. Also, among the reads that were aligned, we filtered out multimappers by specifying `[XS] == null`. 'XS' is a tag generated by Bowtie2 that gives an alignment score for the second-best alignment, and it is only present if the read is aligned and more than one alignment was found for the read.

Now that the alignment files contain only uniquely mapping reads, we are ready to perform peak calling.

> _**NOTE:** After performing read alignment, it's often useful to generate QC metrics for the alignment using tools such as [QualiMap](http://qualimap.bioinfo.cipf.es/doc_html/analysis.html#bam-qc) or [MultiQC](http://multiqc.info) prior to moving on to the next steps of the analysis._ 

***
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

