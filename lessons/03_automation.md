---
title: "Automating QC and Alignment"
author: "Meeta Mistry, Radhika Khetani"
date: "June 28th, 2017"
---

Contributors: Meeta Mistry, Radhika Khetani

Approximate time: 60 minutes

## Learning Objectives

* Automate a portion of the workflow by grouping a series of sequential commands into a shell script
* Modify the script to submit the workflow as a job to the cluster
* Understanding the difference between serial and parallel jobs


```bash
#!/bin/bash

module load gcc/6.2.0 bowtie2/2.2.9

cd /home/mm573/chipseq/results/bowtie2

for file in /home/mm573/chipseq/raw_data/*.fastq

do

base=`basename $file .fastq`

bowtie2 -q --local \
-x ~/chipseq/reference_data/chr12 \
-U ~/chipseq/raw_data/${base}.fastq \
-S ~/chipseq/results/bowtie2/${base}_unsorted.sam

samtools view -h -S -b \
-o ${base}_unsorted.bam \
${base}_unsorted.sam

sambamba sort -t 2 \
-o ${base}_sorted.bam \
${base}_unsorted.bam 

sambamba view -h -t 2 -f bam \
-F "[XS] == null and not unmapped " \
${base}_sorted.bam > ${base}_aln.bam

done

``` 



***
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*
