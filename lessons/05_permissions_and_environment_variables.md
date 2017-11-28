---
title: "Permissions and Environment variables"
author: "Christina Koch, Radhika Khetani"
date: "Wednesday, September 20, 2017"
---

Approximate time: 40 minutes

## Learning Objectives
 
* How to grant or restrict access to files on a multi-user UNIX system
* What is an "Environment Variable" in a shell.
* What is $PATH, and why I should care.

## **Permissions**

Unix controls who can read, modify, and run files using *permissions*.

Users of a multi-user UNIX system can belong to any number of groups.

Let's see what groups we all belong to:

```bash
$ groups
```

Depending on our affiliation, we all belong to at least a couple of groups. I belong to 4 groups,
* rsk27
* bcbio
* hbctraining
* Domain_Users

As you can imagine, on a shared system it is important to protect each user's data. To start, every file and directory on a Unix computer belongs to one owner and one group. Along with each file's content, the operating system stores the information about the user and group that own it, which is the "metadata" for a given file.

The user-and-group model means that for each file every user on the system falls into one of three categories:

* the owner of the file
* a member of the group the file belongs to
* and everyone else.

For each of these three categories, the computer keeps track of whether people in that category can read the file, write to the file, or execute the file (i.e., run it if it is a program).

Let's look at this model in action by running the command `ls -l /n/groups/hbctraining/`, to list the files in that directory:

```bash
$ ls -l /n/groups/hbctraining/

drwxrwsr-x  4 mm573 hbctraining 831 Feb 29  2016 bcbio-rnaseq
drwxrwsr-x 12 mm573 hbctraining 318 May 24 11:13 chip-seq
-rw-r--r--  1 root  hbctraining   0 Apr  5  2015 copy_me.txt
drwxrwsr-x  3 rsk27 hbctraining 201 Apr  5  2015 exercises
drwxrwsr-x  6 rsk27 hbctraining 293 Oct 27 09:40 for_chipseq
drwxrwsr-x  4 mp298 hbctraining  51 Dec  6  2016 mep-data
drwxrwsr-x  4 rsk27 hbctraining  53 Jun  2 15:57 ngs_course
drwxrwsr-x  4 rsk27 hbctraining  53 Nov  2  2016 ngs-course_backup_Nov1-2016
drwxrwsr-x  6 mm573 hbctraining 107 Mar 24  2016 ngs-data-analysis2016
.
.
.
.
```

The `-l` flag tells `ls` to give us a long-form listing. It's a lot of information, so let's go through the columns in turn.

On the right side, we have the files' names. Next to them, moving left, are the times and dates they were last modified. Backup systems and other tools use this information in a variety of ways, but you can use it to tell when you (or anyone else with permission) last changed a file.

Next to the modification time is the file's size in bytes and the names of the user and group that owns it. In this case, it is the eCommons IDs denoting either Mary, Meeta or me as an owner and `hbctraining` is the associated group. 

Now, take a look at the `unix_workshop` directory in your home directory to explore that first column a little more:

```bash
ls -l ~/unix_workshop/

drwxrwsr-x 2 rsk27 rsk27  78 Aug 22 21:08 genomics_data
drwxrwsr-x 2 rsk27 rsk27 725 Aug 22 21:16 other
drwxrwsr-x 2 rsk27 rsk27 256 Aug 22 21:16 raw_fastq
-rw-rw-r-- 1 rsk27 rsk27 377 Sep 22 10:00 README.txt
drwxrwsr-x 2 rsk27 rsk27  62 Aug 22 21:07 reference_data
```

Who is the owner of the files in this directory? Which group do the files belong to?

Basically, O2 has you (your account ID) listed both as an owner and a group, and this is usually the assignment for the files and folders in your personal directory.

We'll skip over the second column for now (the one showing `1` for each file), because it's the first column that we care about most. This shows the file's permissions, i.e., who can read, write, or execute it.

Let's have a closer look at one of those permission strings for README.txt:

```bash
-rw-rw-r--
```

**The first character** tells us whether the listing is a regular file `-` or a directory `d`, or there may be some other character meaning more esoteric things. In this case, it is `-` which means it is a regular file.

The next 9 characters are usually some combination of the following in the order listed below:

<button>r</button> = read permission

<button>w</button> = write/modify permission
 
<button>x</button> = execute permission (run a script/program or traverse a directory).

> Sometimes the `x` is replaced by another character, but it is beyond the scope of today's class. You can [get more information here](https://en.wikipedia.org/wiki/File_system_permissions#Notation_of_traditional_Unix_permissions), if you are interested.
>
> To see an example of a file that is actually executable, try `ls -l /bin/ls`.

The **next three characters** tell us what permissions the file's **owner** has. Here, the owner can read and write the file: `rw-`. 

The **middle triplet** shows us the **group's permissions**. If the permission is turned off, we see a dash, so `rw-` means "read and write, but not execute". (In this case the group and the owner are the same so it makes sense that this is the same for both.)

The **final triplet** shows us what everyone who isn't the file's owner, or in the file's group, can do. In this case, it's `r--` again, so **everyone else** on the system can read the file's contents.

Now, if we take a look at the permissions for directories (e.g. `drwxrwsr-x`): the `x` for the permissions here indicates that "execute" is turned on. What does that mean, given that a directory isn't a program or an executable file, we can't "execute" it? 

Well, `x` means something different for directories. It gives someone the right to *traverse* the directory, but not to look at its contents. This is beyond the scope of today's class, but note that you can give access to a specific file that's deep inside a directory structure without giving them access to all the files and sub-directories within.

### Changing permissions

To change permissions, we use the `chmod` command (whose name stands for "change mode"). Let's make our README.txt file **inaccessible** to all users other than you and the group the file belong to (you, in this case), currently they are able to read it:

```bash
$ ls -l ~/unix_workshop/README.txt

-rw-rw-r-- 1 rsk27 rsk27 377 Oct  6 10:28 /home/rsk27/unix_workshop/README.txt

$ chmod o-rw ~/unix_workshop/README.txt         # the "-" after o denotes removing that permission

$ ls -l ~/unix_workshop/README.txt

-rw-rw---- 1 rsk27 rsk27 377 Oct  6 10:28 /home/rsk27/unix_workshop/README.txt
```

The `o` signals that we're changing the privileges of "others".

Let's change it back to allow it to be readable by others:

```bash
$ chmod o+r ~/unix_workshop/README.txt         # the "+" after o denotes adding/giving that permission

$ ls -l ~/unix_workshop/README.txt

-rw-rw-r-- 1 rsk27 rsk27 377 Oct  6 10:28 /home/rsk27/unix_workshop/README.txt
```

If we wanted to make this an executable file for ourselves (the file's owners) we would say `chmod u+rwx`, where the `u` signals that we are changing permission for the file's owner. To change permissions for the "group", you'd use the letter `g`, e.g. `chmod g-w`. 

>> The fact that something is marked as executable doesn't actually mean it contains or is a program of some kind. We could easily mark the `~/unix_workshop/raw_fastq/Irrel_kd_1.subset.fq` file as executable using `chmod`. Depending on the operating system we're using, trying to "run" it will fail (because it doesn't contain instructions the computer recognizes, i.e. it is not a script of some type).

****
**Exercise**

If `ls -l myfile.php` returns the following details:

```bash
-rwxr-xr-- 1 caro zoo  2312  2014-10-25 18:30 myfile.php
```
 
Which of the following statements is true?
 
1. caro (the owner) can read, write, and execute myfile.php
2. caro (the owner) cannot write to myfile.php
3. members of caro (a group) can read, write, and execute myfile.php
4. members of zoo (a group) cannot execute myfile.php
****

## **Environment Variables**

Environment variables are, in short, variables that describe the environment in which programs run, and they are predefined for a given computer or cluster that you are on. You can reset them to customize the environment. Two commonly encountered environment variables are `$HOME` and `$PATH`.

* `$HOME` defines the full path for the home directory of a given user.
* `$PATH` defines a list of directories to search in when looking for a command/program to execute.

In the context of the shell the environment variables are usually all in upper case.

First, let's see our list of environmental variables:
```bash
$ env
```

Let's see what is stored in these variables:

```bash
$ echo $HOME

/home/trainingaccount_03
```

Variables, in most systems, are called or denoted with a "$" before the variable name, just like a regular variable.

```bash
$ echo $PATH

/opt/lsf/7.0/linux2.6-glibc2.3-x86_64/bin:/groups/bcbio/bcbio/anaconda/bin:/opt/bcbio/local/bin:/opt/lsf/7.0/linux2.6-glibc2.3-x86_64/etc:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin
```

I have a lot of full/absolute paths in my $PATH variable, which are separated from each other by a ":"; here is the list in a more readable format:

* /n/cluster/bin
* /n/app/bcbio/tools/bin
* /usr/local/bin
* /usr/bin
* /usr/local/sbin
* /usr/sbin
* /opt/puppetlabs/bin

These are the directories that the shell will look through (in the same order as they are listed) for a command or an executable file that you type on the command prompt.

When someone says that a command or an executable file is "in my path", they mean that the parent directory for that command/file is contained in the list within the $PATH variable. 

For any command you execute on the command prompt, you can find out where they are located using the `which` command.

Try it on a few of the basic commands we have learned so far:
```bash
$ which ls
$ which <your favorite command>
$ which <your favorite command>
```

Are the directories listed by the `which` command within `$PATH`?

> #### Modifying Environment Variables
>
> If you are interested in adding a new entry to the path variable, the command to use is `export`. This command is usually executed as follows: 
>
> `export PATH=$PATH:~/opt/bin`, which tells the shell to add the `~/opt/bin` directory to the end of the preexisting list within `$PATH`. Alternatively, if you use `export PATH=~/opt/bin:$PATH`, the same directory will be added to the beginning of the list. The order determines which directory the shell will look in first to find a program.

#### Closer look at the inner workings of the shell, in the context of $PATH
 
The $PATH variable is reset to a set of defaults (/bin:/usr/bin and so on), each time you start a new shell Terminal. To make sure that a command/program you need is always at your fingertips, you have to put it in one of 2 special shell scripts that are always run when you start a new terminal. These are hidden files in your home directory called `.bashrc` and `.bash_profile`. You can create them if they don't exist, and shell will use them.

Check what hidden files exist in our home directory using the `-a` flag:
```bash
$ ls -al ~/
```

We want to add `/n/app/bcbio/tools/bin` to the beginning of the list in `$PATH`, we need tools within this directory for the RNA-Seq workflow. We can add the appropriate `export` command to do this at the end of the `.bashrc` file, this will make it so that when you start a new shell session the location will always be in your path. 

Open the `.bashrc` file using `vim` and at the end of the file add the export command that adds a specific location to the list in `$PATH`. 

```bash
$ vim ~/.bashrc

# at the end of the file type in the following - export PATH=/n/app/bcbio/tools/bin:$PATH
# Don't forget the ":" between the full path and the "$PATH"!
```

**In closing, permissions and environment variables, especially `$PATH`, are very useful and important concepts to understand in the context of UNIX and HPC.**

---
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*

* *The materials used in this lesson were derived from work that is Copyright © Data Carpentry (http://datacarpentry.org/). 
All Data Carpentry instructional material is made available under the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).*
* *Adapted from the lesson by Tracy Teal. Original contributors: Paul Wilson, Milad Fatenejad, Sasha Wood and Radhika Khetani for Software Carpentry (http://software-carpentry.org/)*

