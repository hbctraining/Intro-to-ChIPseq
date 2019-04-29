---
title: "The Shell"
author: "Mary Piper, Radhika Khetani, Meeta Mistry"
date: "April 2019"
---

## Learning Objectives
- Review shell commands
- Review HPC concepts 

## Setting up

Let's look at some of the basic HPC concepts with the **first few slides in [this slide deck](https://hbctraining.github.io/Intro-to-rnaseq-hpc-salmon/lectures/HPC_intro_O2_review.pdf)**, then we will come back and get some practice.

### Connecting to a *login* node on O2

Type in the following command with your username to login:

```bash
ssh username@o2.hms.harvard.edu
```

You will receive a prompt for your password, and you should type in your associated password; note that the cursor will *not move* as you type in your password.

A warning might pop up the first time you try to connect to a remote machine, type "Yes" or "Y". 

Once logged in, you should see the O2 icon, some news, and the command prompt: 

```
[rc_training10@login01 ~]$ 
```

> `ssh` stands for secure shell. All of the information (like your password) going between your computer and the O2 login computer is encrypted when using `ssh`.
>
> A "node" on a cluster is essentially a computer in the cluster of computers.

**A login node's only function is to enable users to log in to a cluster, it is not meant to be used for any actual work/computing.**

### Connecting to a *compute* node on O2

There are multiple ways to connect with, and do work on, a compute node; a compute node is where all work should be performed. To connect to a compute node, users have to interact with a job scheduler like *slurm* using commands like `srun` or `sbatch`, and by specifying what resources they require.

1. The `srun` command with a few mandatory parameters will create an "interactive session" on O2. This is essentially a way for us to do work on the compute node directly from the terminal. If the connectivity to the cluster is lost in the middle of a command being run that work will be lost in an interactive session.

2. The `sbatch` command with a few mandatory parameters + a specialized shell script will result in the script being run on a compute node. This "job" will not be accessible directly from the Terminal and will run in the background. Users do not need to remain connected to the cluster when such a "batch job" is running.

You will get practice with running batch jobs, for now we are going to start an interactive session on O2 using `srun`. 

```bash
$ srun --pty -p interactive -t 0-8:00 --mem 1G --reservation=HBC2 /bin/bash
```

In the above command the parameters we are using are requesting specific resources:
* `--pty` - Start an interactive session
* `-p interactive` - on the "partition" called "interactive" (a partition is a group of computers dedicated to certain types of jobs, interactive, long, short, high-memory, etc.)
* `-t 0-8:00` - time needed for this work: 0 days, 8 hours, 0 minutes.
* `--mem 1G` - memory needed - 1 gigabyte
* `--reservation=HBC2` - ***this is only for this workshop, make sure you don't use it in the future with your own accounts***
* `/bin/bash` - You want to interact with the compute node using the *bash* shell

> These resources are listed slightly differently in the specialized script that is submitted directly using `sbatch`. We will be reviewing the arguments above and what that specialized script looks like at the end of this lesson.

Make sure that your command prompt is now preceded by a character string that contains the word "compute":

```
[rc_training10@compute-a-16-163 ~]$
```

### Copying example data folder

Your accounts were erased after the command-line workshop, so we are starting fresh this time, let's copy over the same data folder we worked with in the shell workshop to our home directories:

```bash
$ cp -r /n/groups/hbctraining/unix_lesson/ .
```

****

**Exercise**

1. In the above command, what does the `.` at the end mean? Is it essential?
2. Why did we have to run the `cp` command with `-r`?
3. Is the path to the `unix_lesson/` directory a "full" path or a "relative" path?

****

## Reviewing shell commands

We are going to start this review with more exercises, this time hands on! Remember, there are likely multiple ways to do the same thing and we will try to cover at least a few.

****

**Exercises**

**Shell basics**

1. Change directory into the `unix_lesson/` directory.
2. Use the `tree` command to get a directory structure of `unix_lesson/`.
3. Take a quick look at the `Mov10_oe_1.subset.fq` file using `less` from `unix_lesson/`, without changing directories.
4. Move up to your home directory (parent of `unix_lesson/`).
5. With a single command change directories to the `raw_fastq/` folder.
6. With a shortest possible command change directories back to the home directory.
7. What does the `~` in the command prompt mean?
8. What is the full path to your home directory?
9. List, in long listing format, the contents of `/n/groups/hbctraining/intro_rnaseq_hpc/full_dataset/` **using tab completion**.
10. Modify the above command using the `*` wildcard to only list those files that have "oe" in their names.
11. How many and which commands have you run so far today?

**Loops and shell scripts**

16. Use the `for` loop to iterate over each FASTQ file in `~/unix_lesson/raw_fastq/` and do the following:
      * Print the name of the current file
      * Dump out the first 40 lines into a new file that will be saved in `~/unix_lesson/shell_review/`
17. Place the above `for` loop into a shell script using `vim` and run it.

**Environment variables**

19. Display the contents of the `$HOME` variable.
20. Use the `which` command to check where the executable file for the `pwd` command lives in the directory structure.
21. How does shell know where to find the executable file for the `pwd` command?
22. Display the contents of the variable that stores the various paths to folders containing executable command files.
23. Can you run the `bowtie2` command? What do you think you might need to do to run this command?

**LMOD system**

24. Load the `gcc/6.2.0` module.
25. Has `$PATH` changed? 
26. Load the `bowtie2/2.3.4.3` module.
27. List the modules that are loaded.

****

## Some setting up for the rest of the workshop

### Add a path to `$PATH`

We need to use one tool that is unavailable as a module on O2, but it is available in a folder on O2, so we are going to add it to our $PATH. If we just add it using the `export` command, it will only be available to us in this specific interactive session. However, if we place that export command in a script that is run everytime a new interactive session is started, it is more efficient.

* Use `vim` to open `~/.bashrc`
* Add the following line at the end of the file `export PATH=/n/app/bcbio/tools/bin:$PATH`
* Save and quit out of `vim`

### Resources on O2 and asking Slurm for them

Finally, let's review some of the information for O2 and slurm in [the rest of the slides](https://hbctraining.github.io/Intro-to-rnaseq-hpc-salmon/lectures/HPC_intro_O2_review.pdf)

****

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*
