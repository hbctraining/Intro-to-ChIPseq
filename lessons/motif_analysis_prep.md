---
title: "Motif Analysis Prep using Bedtools"
author: " Meeta Mistry"
date: "Thursday, April 25th, 2019"
---

Approximate time: 20 minutes


## Preparing files for Motif Analysis

Once you have your set of enriched regions which represent binding sites from your protein of interest, a next logical question is within those regions are there any **similar patterns of sequences that are over-represented**. These sequences are often referred to as **motifs**.

Tools for motif analyis often require sequence information for each of your binding regions, and so this lesson will show you how to use `bedtools` to obtain that.

Start an interactive session:	

```bash
$ srun --pty -p interactive -t 0-12:00 --mem 1G --reservation=HBC2 /bin/bash
```	

Since the motif analyses are unlikely to give reliable results using only the 32.8 Mb of reads mapping to chr12, **we will use the full set of peak calls output from the IDR analysis**. Move into your project directory, and copy over the IDR bed files:

```bash
$ cd ~/chipseq/results

$ cp /n/groups/hbctraining/chip-seq/full-dataset/idr/*.bed .

```

Extract the **first three columns** of the IDR peak calls for the Nanog file:

```bash
	$ cut -f 1,2,3 Nanog-idr-merged.bed  > Nanog-idr-merged-simple.bed
```


To extract the sequences corresponding to the peak coordinates for motif discovery, we will use the [bedtools](http://bedtools.readthedocs.org/en/latest/content/bedtools-suite.html) suite of tools. **The `getfasta` command extracts sequences from a reference fasta file for each of the coordinates defined in a BED/GFF/VCF file**. 	


 ```bash	
$ module load gcc/6.2.0 bedtools/2.27.1	

$ bedtools getfasta -fi \
/n/groups/shared_databases/igenome/Homo_sapiens/UCSC/hg19/Sequence/WholeGenomeFasta/genome.fa \
-bed Nanog-idr-merged-simple.bed \
-fo Nanog-idr-merged-dreme.fasta
```	

 Using `scp` or **FileZilla** on your local computer, transfer `Nanog-idr-merged-dreme.fasta` to your Desktop.

***

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

