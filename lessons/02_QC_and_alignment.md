---
title: "FastQC and alignment"
author: "Mary Piper, Radhika Khetani"
date: "March 14th, 2018"
---

Contributors: Mary Piper, Radhika Khetani

Approximate time: 60 minutes

## Learning Objectives

* Evaluate the quality of your sequencing data using FastQC
* Describe parameters for performing alignment using Bowtie2
* Become familiar with data analysis best practices

## Best practices for NGS Analysis 

Ok so now you are all set up and have begun your analysis. You have followed best practices to set up your analysis directory structure in a way such that someone unfamiliar with your project should be able to look at it and understand what you did and why (reproducibility). But there is more...

1. **Use appropriate software.** Do your research and find out what is best for the data you are working with. Don't just work with tools that you are able to easily install. Also, make sure you are using the most up-to-date versions, and keeping track of the version number! If you run out-of-date software, you are probably introducing errors into your workflow; and you may be missing out on more accurate methods.

2. **Keep up with the literature.** Bioinformatics is a fast-moving field and it's always good to stay in the know about recent developments. This will help you determine what is appropriate and what is not.  

3. **Do not re-invent the wheel.** If you run into problems, more often than not someone has already encountered that same problem. A solution is either already available or someone is working on it -- so find it! Ask colleagues or search/ask online forums such as [BioStars](https://www.biostars.org/).

4. **Testing is essential.** If you are using a tool for the first time, test it out on a single sample or a subset of the data before running your entire dataset through. This will allow you to debug quicker and give you a chance to also get a feel for the tool and the different parameters.

## Quality control of sequence reads

<img src="../img/chip_workflow_march2018_step1.png" width="700">

Now that we have our files and directory structure, we are ready to begin our ChIP-Seq analysis. For any NGS analysis method, our first step in the workflow is to explore the quality of our reads prior to aligning them to the reference genome and proceeding with downstream analyses. 

### Understanding the Illumina sequencing technology

Before we can assess the quality of our reads, it would be helpful to know a little bit about how these reads were generated. Since our data was sequenced on an Illumina sequencer we will introduce you to their Sequencng by Synthesis methodology, however keep in mind there are other technologies and the way reads are generated will vary (as will the associated biases observed in your data). An **animation of the Sequencing by Synthesis is most helpful** (rather than reading through lines of text), and so we would like you to take five minutes and watch [this YouTube video](https://www.youtube.com/watch?v=fCd6B5HRaZ8&t=3s) from Illumina.

### Unmapped read data (FASTQ)

The [FASTQ](https://en.wikipedia.org/wiki/FASTQ_format) file format is the defacto file format for sequence reads generated from next-generation sequencing technologies. This file format evolved from FASTA in that it contains sequence data, but also contains quality information. Similar to FASTA, the FASTQ file begins with a header line. The difference is that the FASTQ header is denoted by a `@` character. For a single record (sequence read) there are four lines, each of which are described below:

|Line|Description|
|----|-----------|
|1|Always begins with '@' and then information about the read|
|2|The actual DNA sequence|
|3|Always begins with a '+' and sometimes the same info in line 1|
|4|Has a string of characters which represent the quality scores; must have same number of characters as line 2|

Let's use the following read as an example:

```
@HWI-ST330:304:H045HADXX:1:1101:1111:61397
CACTTGTAAGGGCAGGCCCCCTTCACCCTCCCGCTCCTGGGGGANNNNNNNNNNANNNCGAGGCCCTGGGGTAGAGGGNNNNNNNNNNNNNNGATCTTGG
+
@?@DDDDDDHHH?GH:?FCBGGB@C?DBEGIIIIAEF;FCGGI#########################################################
```

As mentioned previously, line 4 has characters encoding the quality of each nucleotide in the read. The legend below provides the mapping of quality scores (Phred-33) to the quality encoding characters. *Different quality encoding scales exist (differing by offset in the ASCII table), but note the most commonly used one is fastqsanger.*

 ```
 Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHI
                   |         |         |         |         |
    Quality score: 0........10........20........30........40                                
```
 
Using the quality encoding character legend, the first nucelotide in the read (C) is called with a quality score of 31 and our Ns are called with a score of 2. **As you can tell by now, this is a bad read.** 

Each quality score represents the probability that the corresponding nucleotide call is incorrect. This quality score is logarithmically based and is calculated as:

	Q = -10 x log10(P), where P is the probability that a base call is erroneous

These probabaility values are the results from the base calling algorithm and dependent on how much signal was captured for the base incorporation. The score values can be interpreted as follows:

|Phred Quality Score |Probability of incorrect base call |Base call accuracy|
|:-------------------|:---------------------------------:|-----------------:|
|10	|1 in 10 |	90%|
|20	|1 in 100|	99%|
|30	|1 in 1000|	99.9%|
|40	|1 in 10,000|	99.99%|
|50	|1 in 100,000|	99.999%|
|60	|1 in 1,000,000|	99.9999%|

Therefore, for the first nucleotide in the read (C), there is less than a 1 in 1000 chance that the base was called incorrectly. Whereas, for the the end of the read there is greater than 50% probabaility that the base is called incorrectly.

## Assessing quality with FastQC

Now we understand what information is stored in a FASTQ file, the next step is to examine quality metrics for our data.

[FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) provides a simple way to do some quality control checks on raw sequence data coming from high throughput sequencing pipelines. It provides a modular set of analyses which you can use to give a quick impression of whether your data has any problems of which you should be aware before doing any further analysis.

The main functions of FastQC are:

* Import of data from BAM, SAM or FastQ files (any variant)
* Providing a quick overview to tell you in which areas there may be problems
* Summary graphs and tables to quickly assess your data
* Export of results to an HTML based permanent report
* Offline operation to allow automated generation of reports without running the interactive application

### Run FastQC  

Let's run FastQC on all of our files. 

Change directories to the `raw_data` folder and check the contents

```bash
$ cd ~/chipseq/raw_data 

$ ls -l
```

Before we start using any software, we either have to check if it's available on the cluster, and if it is we have to load it into our environment (or `$PATH`). On the O2 cluster, we can check for, and load packages (or modules) using the **LMOD** system. 

If we check which modules we currently have loaded, we should not see FastQC.

```bash
$ module list
```

This is because the FastQC program is not in our $PATH (i.e. its not in a directory that unix will automatically check to run commands/programs).

```bash
$ echo $PATH
```

To find the FastQC module to load we need to search the versions available:

```bash
$ module spider
```

Then we can load the FastQC module:

```bash
$ module load fastqc/0.11.3
```

Once a module for a tool is loaded, you have essentially made it directly available to you like any other basic UNIX command.

```bash
$ module list

$ echo $PATH
```

FastQC will accept multiple file names as input, so we can use the `*.fq` wildcard.

```bash
$ fastqc *.fastq
```

*Did you notice how each file was processed serially? How do we speed this up?*

Exit the interactive session and once you are on a "login node," start a new interactive session with 6 cores. Now we can use the multi-threading functionality of FastQC to speed this up by running 6 jobs at once, one job for one file.

```bash
$ exit  #exit the current interactive session

$ srun --pty -n 6 -p short -t 0-12:00 --mem 8G --reservation=HBC /bin/bash  #start a new one with 6 cpus (-n 6) and 8G RAM (--mem 8G)

$ module load fastqc/0.11.3  #reload the module for the new session

$ cd ~/chipseq/raw_data

$ fastqc -t 6 *.fastq  #note the extra parameter we specified for 6 threads
```

How did I know about the -t argument for FastQC?

```bash
$ fastqc --help
```

Now, move all of the `fastqc` files to the `results/fastqc` directory:

```bash
$ mv *fastqc* ../results/fastqc/
```

### FastQC Results
   
Let's take a closer look at the files generated by FastQC:
   
`$ ls -lh ../results/fastqc/`

#### HTML reports
The .html files contain the final reports generated by fastqc, let's take a closer look at them. Transfer the file for `H1hesc_Input_Rep1_chr12.fastq` over to your laptop via *FileZilla*.

##### Filezilla - Step 1

Open *FileZilla*, and click on the File tab. Choose 'Site Manager'.
 
![FileZilla_step1](../img/Filezilla_step1.png)

##### Filezilla - Step 2

Within the 'Site Manager' window, do the following: 

1. Click on 'New Site', and name it something intuitive (e.g. O2)
2. Host: transfer.rc.hms.harvard.edu 
3. Protocol: SFTP - SSH File Transfer Protocol
4. Logon Type: Normal
5. User: training_account
6. Password: password for training_account
7. Click 'Connect'

<img src="../img/Filezilla_step2.png" width="500">	
	
The **"Per base sequence quality"** plot is the most important analysis module in FastQC for ChIP-Seq; it provides the distribution of quality scores across all bases at each position in the reads. This information can help determine whether there were any problems at the sequencing facility during the sequencing of your data. Generally, we expect a decrease in quality towards the ends of the reads, but we shouldn't see any quality drops at the beginning or in the middle of the reads.

![FastQC_seq_qual](../img/FastQC_seq_qual.png)

Based on the sequence quality plot, we see the majority of the reads have high quality, but the whiskers drop into the poor quality regions, indicating that a significant number of reads have low quality bases across the reads. The poor quality reads in the middle of the sequence would be concerning if this was our dataset, and we would probably want to contact the sequencing facility. However, this dataset was created artifically, so does not indicate a problem at the sequencing facility. Trimming could be performed from both ends of the sequences, or we can use an alignment tool that can ignore these poor quality bases at the ends of reads (soft clip). 

This is the main plot explored for ChIP-seq, but if you would like to go through the remaining plots/metrics, FastQC has a really well documented [manual page](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) with [more details](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) about all the plots in the report. We recommend looking at [this post](http://bioinfo-core.org/index.php/9th_Discussion-28_October_2010) for more information on what bad plots look like and what they mean for your data. Also, FastQC is just an indicator of what's going on with your data, don't take the "PASS"es and "FAIL"s too seriously.

> **We also have a [slidedeck](https://github.com/hbctraining/Intro-to-rnaseq-hpc-O2/raw/master/lectures/error_profiles_mm.pdf) of error profiles for Illumina sequencing, where we discuss specific FASTQC plots and possible sources of these types of errors.**

## Alignment to Genome

Now that we have assessed the quality of our sequence data, we are ready to align the reads to the reference genome. [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) is a fast and accurate alignment tool that indexes the genome with an FM Index based on the Burrows-Wheeler Transform method to keep memory requirements low for the alignment process. *Bowtie2* supports gapped, local and paired-end alignment modes and works best for reads that are at least 50 bp (shorter read lengths should use Bowtie1). By default, Bowtie2 will perform a global end-to-end read alignment, which is best for quality-trimmed reads. However, it also has a local alignment mode, which will perform soft-clipping for the removal of poor quality bases or adapters from untrimmed reads. We will use this option since we did not trim our reads.

> _**NOTE:** Our reads are only 36 bp, so technically we should explore alignment with [bwa](http://bio-bwa.sourceforge.net/) or Bowtie1 to see if it is better. However, since it is rare that you will have sequencing reads with less than 50 bp, we will show you how to perform alignment using Bowtie2._

### Creating Bowtie2 index

To perform the Bowtie2 alignment, a genome index is required. The index is analagous to the index in the back of a book. By indexing the genome, we have organized it in a manner that now allows for efficient search and retrieval of matches of the query (sequence read) to the genome. **We previously generated the genome indices for you**, and they exist in the `reference_data` directory.

However, if you needed to create a genome index yourself, you would use the following command:

```bash
# DO NOT RUN

bowtie2-build <path_to_reference_genome.fa> <prefix_to_name_indexes>
```

> A quick note on shared databases for human and other commonly used model organisms. The O2 cluster has a designated directory at `/n/groups/shared_databases/` in which there are files that can be accessed by any user. These files contain, but are not limited to, genome indices for various tools, reference sequences, tool specific data, and data from public databases, such as NCBI and PDB. So when using a tool that requires a reference of sorts, it is worth taking a quick look here because chances are it's already been taken care of for you. 

>```bash
>$ ls -l /n/groups/shared_databases/igenome/
>```

### Aligning reads to the genome with Bowtie2

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

## Alignment file format: SAM/BAM

The output we requested from the Bowtie2 aligner is an unsorted SAM file, also known as **Sequence Alignment Map format**. The SAM file, is a **tab-delimited text file** that contains information for each individual read and its alignment to the genome. While we will go into some features of the SAM format, the paper by [Heng Li et al](http://bioinformatics.oxfordjournals.org/content/25/16/2078.full) provides a lot more detail on the specification.

The file begins with a **header**, which is optional. The header is used to describe source of data, reference sequence, method of alignment, etc., this will change depending on the aligner being used. Each section begins with character ‘@’ followed by **a two-letter record type code**.  These are followed by two-letter tags and values. Example of some common sections are provided below:

```
@HD  The header line
VN: format version
SO: Sorting order of alignments

@SQ  Reference sequence dictionary
SN: reference sequence name
LN: reference sequence length
SP: species

@PG  Program
PN: program name
VN: program version
```

Following the header is the **alignment section**. Each line that follows corresponds to alignment information for a single read. Each alignment line has **11 mandatory fields for essential mapping information** and a variable number of other fields for aligner specific information. 

![SAM1](../img/sam_bam.png)

An example read mapping is displayed above. *Note that the example above spans two lines, but in the file it is a single line.* Let's go through the fields one at a time. First, you have the read name (`QNAME`), followed by a `FLAG` 

The `FLAG` value that is displayed can be translated into information about the mapping. 

| Flag | Description |
| ------:|:----------------------:|
| 1 | read is mapped |
| 2 | read is mapped as part of a pair |
| 4 | read is unmapped |
| 8 | mate is unmapped |
| 16| read reverse strand|
| 32 | mate reverse strand |
| 64 | first in pair |
| 128 | second in pair |
| 256 | not primary alignment |
| 512 | read fails platform/vendor quality checks |
| 1024| read is PCR or optical duplicate |

* For a given alignment, each of these flags are either **on or off** indicating the condition is **true or false**. 
* The `FLAG` is a combination of all of the individual flags (from the table above) that are true for the alignment 
* The beauty of the flag values is that **any combination of flags can only result in one sum**.

**There are tools that help you translate the bitwise flag, for example [this one from Picard](https://broadinstitute.github.io/picard/explain-flags.html)**

Moving along the fields of the SAM file, we then have `RNAME` which is the reference sequence name. The example read is from chromosome 1 which explains why we see 'chr1'. `POS` refers to the 1-based leftmost position of the alignment. `MAPQ` is giving us the alignment quality, the scale of which will depend on the aligner being used. 

`CIGAR` is a sequence of letters and numbers that represent the *edits or operations* required to match the read to the reference. The letters are operations that are used to indicate which bases align to the reference (i.e. match, mismatch, deletion, insertion), and the numbers indicate the associated base lengths for each 'operation'.

| Operation | Description |
| ------:|:----------------------:|
| M | sequence match or mismatch |
| I | insertion to the reference |
| D | deletion from reference |
| N | skipped region from the reference|


Now to the remaning fields in our SAM file:

![SAM1](../img/sam_bam3.png)

The next three fields are more pertinent to paired-end data. `MRNM` is the mate reference name. `MPOS` is the mate position (1-based, leftmost). `ISIZE` is the inferred insert size.

Finally, you have the data from the original FASTQ file stored for each read. That is the raw sequence (`SEQ`) and the associated quality values for each position in the read (`QUAL`).

Let;'s take a quick peek at our SAM file that we just generated. Since it is just a text file, we can browse through it using `less`:

``` bash
$ less H1hesc_Input_Rep1_chr12_aln_unsorted.sam
```

**Does the information you see line up with the fields we described above?**

## Filtering reads

An important issue with ChIP-seq data concerns the inclusion of multiple mapped reads (reads mapped to multiple loci on the reference genome). **Allowing for multiple mapped reads increases the number of usable reads and the sensitivity of peak detection; however, the number of false positives may also increase** [[1]](https://www.ncbi.nlm.nih.gov/pubmed/21779159/). Therefore we need to filter our alignment files to **contain only uniquely mapping reads** in order to increase confidence in site discovery and improve reproducibility. Since there is no parameter in Bowtie2 to keep only uniquely mapping reads, we will need to perform the following steps to generate alignment files containing only the uniquely mapping reads:

1. Change alignment file format from SAM to BAM
2. Sort BAM file by read coordinate locations
3. Filter to keep only uniquely mapping reads (this will also remove any unmapped reads)

### 1. Changing file format from SAM to BAM

While the SAM alignment file output by Bowtie2 is human readable, we need a BAM alignment file for downstream tools. Therefore, we will use [Samtools](http://samtools.github.io) to convert the file formats.

To use `samtools` we will need to load the module:

```bash
module load samtools/1.3.1
```

The command we will use is `samtools view` with the following parameters:

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

Before we can filter to keep the uniquely mapping reads, we need to sort our BAM alignment files by genomic coordinates (instead of by name). To perform this sort, we will use [Sambamba](http://lomereiter.github.io/sambamba/index.html), which is a tool that quickly processes BAM and SAM files.

The command we will use is `sambamba sort` with the following parameters:

* `-t`: number of threads / cores
* `-o`: /path/to/output/file

```bash
$ sambamba sort -t 2 \
-o H1hesc_Input_Rep1_chr12_aln_sorted.bam \
H1hesc_Input_Rep1_chr12_aln_unsorted.bam 
```

> **NOTE: This tool is not available as a module on O2.** You will only be able to use this as part of the tools available in the `bcbio` pipeline. In a previous lesson, you had added this to your $PATH by modifying your `.bashrc` file. **If the command above does not work for you, run this line below:**
> 
> `export PATH=/n/app/bcbio/tools/bin:$PATH`


We could have also used `samtools` to perform the above sort, however using `sambamba` gives us dual functionality. List the contents of the directory -- what do you see? The advantage to using `sambamba` is that along with the newly sorted file, an index file is generated. If we used `samtools` this would have been a two-step process.

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

> _**NOTE:** After performing read alignment, it's useful to generate QC metrics for the alignment using tools such as [MultiQC](http://multiqc.info) prior to moving on to the next steps of the analysis._

***
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

