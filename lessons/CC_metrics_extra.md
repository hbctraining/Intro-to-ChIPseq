---
title: "Cross-correlation Metrics"
authors: "Mary Piper and Meeta Mistry"
date: "October 2, 2018"
---

## Quality Metrics Based on Cross-Correlation


### Cross-correlation plot

Strand cross-correlation is computed as the Pearson's linear correlation between the minus strand and the plus strand, after shifting minus strand by k base pairs. In the end we will have a cross-correlation value for each shift value, and they can be plotted against each other to generate a cross-correlation plot as shown below. The **cross-correlation plot typically produces two peaks**: a peak of enrichment corresponding to the predominant fragment length (highest correlation value) and a peak corresponding to the read length (“phantom” peak).

<img src="../img/cc-example.png">

### Metrics based on the cross-correlation plot

There are two metrics that are computed using the cross-correlation described below. If you are interested in computing these and drawing cross-correlation plots outside of the ChIPQC package you can use [phantompeakqualtools](https://github.com/kundajelab/phantompeakqualtools). 

#### Normalized strand cross-correlation coefficent (NSC):

The ratio of the maximal cross-correlation value divided by the background cross-correlation (minimum cross-correlation value over all possible strand shifts). 

<img src="https://latex.codecogs.com/gif.latex?\frac{max(CC&space;values)}{min(CCvalues)}" title="\frac{max(CC values)}{min(CCvalues)}" />

- higher NSC values indicate more enrichment (better signal:noise)
- low signal-to-noise: NSC values < 1.1
- minimum possible NSC value: 1 (no enrichment) 

#### Relative strand cross-correlation coefficient (RSC):

The ratio of the fragment-length cross-correlation value minus the background cross-correlation value, divided by the phantom-peak cross-correlation value minus the background cross-correlation value. 

<img src="https://latex.codecogs.com/gif.latex?\frac{max(CCvalues)&space;-&space;background}{phantomCCvalue&space;-&space;background}" title="\frac{max(CCvalues) - background}{phantomCCvalue - background}" />

- high enrichment: RSC values > 1
- low signal-to-noise: RSC values < 0.8
- minimum possible RSC value: 0 (no enrichment)

> **NOTE:** Low NSC and RSC values can be due to failed and poor quality ChIP, low read sequence quality and hence lots of mismappings, shallow sequencing depth or a combination of these. Also, datasets with few binding sites (< 200) which could be due to biological reasons (i.e. a factor that truly binds only a few sites in a particular tissue type) would output low NSC and RSC scores.


### Fragment length 

The shift value at which we observe the highest correlation value is considered to be the estimated fragment length. Some tools will report to you the top three fragment length values if the peak is not entirely clear. You will want the fragment length estimate to roughly resemble the actual fragment length you had decided on when size selecting during your library preparation.


### Why is it that we observe this 'phantom peak' at read length?

The concept is that if you have a read that maps uniquely on strand `x` at position `i` (where `i` is the starting position); it follows that you will have a read mapping to strand `y` at position `i+r`. Because the way the counts are stored, with the number of reads starting at each coordinate, you will get a bunch of reads at `x[i]` and a bunch of reads at `y[i+r]` that are `r` distance away from each other.


### `phantompeakqualtools` 

The [`phantompeakqualtools`](https://code.google.com/archive/p/phantompeakqualtools/) package is a tool used to compute cross-correlation plots and the quality measures described above for ChIP-seq data [[1](http://www.g3journal.org/content/4/2/209.full)]. We have some instructions below if youare interested in trying it.

#### **Set up**

The `phantompeakqualtools` package is written as an R script, that uses `samtools` as a dependency. The package has various options that need to be specified when running from the command line. To get set up, we will need to start an interactive session, load the necessary modules and set up the directory structure:

```
$ srun --pty -p interactive -t 0-12:00 --mem 8G --reservation=HBC bash	

$ module load gcc/6.2.0 R/3.4.1 samtools/1.3.1

$ cd ~/chipseq/results

$ mkdir chip_qc

$ cd chip_qc
```

**We have downloaded the software for you, and have a copy you can use.**  The directory contains several files.

```
$ ls -l /n/groups/hbctraining/chip-seq/phantompeakqualtools/
```

> **NOTE:**  You can download the `phantompeakqualtools` package, directly from [GitHub](https://github.com/kundajelab/phantompeakqualtools), if you wanted your own local version. This repo is maintained by the developer Anshul Kundaje.


In this folder there should be a `README.txt` which contains all the commands, options, and output descriptions. Let's check out the `README.txt`:

```
$ less /n/groups/hbctraining/chip-seq/phantompeakqualtools/README.md
```

#### **Using R libraries**

In the README you will have noticed an *INSTALLATION* section. We will need to install the R package, `spp` and `caTools`, into our personal R library to run the script. Since this is a bit more involved, in the interest of time we have created the libraries and shared them for you to use. To use our libraries, you will need to setup an environmental variable called `R_LIBS_USER` and point it to the location on O2 where our libraries reside:

```
$  export R_LIBS_USER="/n/groups/hbctraining/R/library/"
```

> **NOTE: Testing libraries**
>
> If you want to check and see that this is working, you can open up R by typing R and pressing Enter:
> 
```
$ R
```
> 
> And then once in R, try loading the libraries:

```
R version 3.4.1 (2017-06-30) -- "Single Candle"
Copyright (C) 2017 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> library(spp)
> library(caTools)
```
>
> To exit, type: `q()` in the console.

#### **Running `phantompeakqualtools`**

To obtain quality measures based on cross-correlation plots, we will be running the `run_spp.R` script from the command line which is a package built on SPP. This modified SPP package allows for determination of the cross-correlation peak and predominant fragment length in addition to peak calling. We will be using this package solely for obtaining these quality measures (no peak calling). 

The options that we will be using include:

* `-c`: full path and name (or URL) of tagAlign/BAM file
* `-savp`: save cross-correlation plot
* `-out`: will create and/or append to a file several important characteristics of the dataset described in more detail below.

```
## DO NOT RUN THIS
## THIS SCRIPT IS FOR COMPUTING METRICS ON A SINGLE FILE
$ Rscript /n/groups/hbctraining/chip-seq/phantompeakqualtools/run_spp.R -c=<tagAlign/BAMfile> -savp -out=<outFile>
```
>_**NOTE:** Even though the script is called `run_spp.R`, we aren't actually performing peak calling with SPP._

From within the `phantompeakqualtools` directory, we will create output directories and use a 'for loop' to **run the script on every Nanog and Pouf51 BAM file**:

```
$ mkdir -p logs qual

$ for bam in ../bowtie2/*Nanog*aln.bam ../bowtie2/*Pou5f1*aln.bam
do 
bam2=`basename $bam _aln.bam`
Rscript /n/groups/hbctraining/chip-seq/phantompeakqualtools/run_spp.R -c=$bam -savp -out=qual/${bam2}.qual > logs/${bam2}.Rout
done
```

The for loop generates **three output files**. The **quality metrics** are written in a tab-delimited text file, and the **log files** contains the standard output text. A third file is created in the same directory as the BAM files. These are pdf files that contain the **cross-correlation** plot for each sample. Let's move those files into the appropriate output directory:

```
$ mv ../bowtie2/*pdf qual  

```

To visualize the quality metrics (.qual) files more easily, we will concatenate the files together to create a single summary file that you can move over locally and open up with Excel.

```
$ cat qual/*qual > qual/phantompeaks_summary.xls
```
Let's use Filezilla or `scp` to move the summary file over to our local machine for viewing. Open up the file in Excel and take a look at our NSC and RSC values. 

#### **`phantompeakqualtools`: quality metrics output**

The qual files are tab-delimited with the columns containing the following information:

- COL1: Filename: tagAlign/BAM filename 
- COL2: numReads: effective sequencing depth (i.e. total number of mapped reads in input file)
- COL3: estFragLen: comma separated strand cross-correlation peak(s) in decreasing order of correlation. (**NOTE:** The top 3 local maxima locations that are within 90% of the maximum cross-correlation value are output. In almost all cases, the top (first) value in the list represents the predominant fragment length.) 
- COL4: corr_estFragLen: comma separated strand cross-correlation value(s) in decreasing order (col2 follows the same order) 
- COL5: phantomPeak: Read length/phantom peak strand shift 
- COL6: corr_phantomPeak: Correlation value at phantom peak 
- COL7: argmin_corr: strand shift at which cross-correlation is lowest 
- COL8: min_corr: minimum value of cross-correlation 
- COL9: Normalized strand cross-correlation coefficient (NSC) = COL4 / COL8 
- COL10: Relative strand cross-correlation coefficient (RSC) = (COL4 - COL8) / (COL6 - COL8) 
- COL11: QualityTag: Quality tag based on thresholded RSC (codes: -2:veryLow,-1:Low,0:Medium,1:High,2:veryHigh)

> **NOTE:** The most important metrics we are interested in are the values in columns 9 through 11, however these numbers are computed from values in the other columns.

**How do the values compare to the thresholds mentioned above?** All samples have quite high NSC values indicating more enrichment, a good signal to noise and a fair number of peaks. Nanog-rep2 has a comparably higher NSC value which might explain the increased number of peaks for that sample compared to the others. The RSC and quality tags further indicate good chip signal and a quality IP, yielding a very high quality tag. Based on these metrics, the samples look good for further analysis.



