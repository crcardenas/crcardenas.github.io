---
layout: single
title:  "Code I wish I knew 3 years ago"
date:   2022-04-10 9:00:00 -0500
categories: bioinformatics
author_profile: true
show_date: true
---

I've been "playing" with publicly available data (found on [NCBI](https://www.ncbi.nlm.nih.gov/ "National Center for Biotechnology Information")), in order to help me wrap my head around some pipelines I will be using for my thesis project. For my project I will be using ultra conserved elements to reconstruct the phylogeny of *Calosoma* [(Weber 1801)](https://www.biodiversitylibrary.org/bibliography/8639 "Biodiversity Heritage Library") *sensu lato* based on [Jenealls 1940](https://bibliotheques.mnhn.fr/EXPLOITATION/infodoc/ged/viewportalpublished.ashx?eid=IFD_FICJOINT_MEMMN_S000_1940_T013_N001_1 "PDF from Muséum National D'Histoire Naturalle Paris") hypothesis of the sub-generic groups. There is actually quite a bit to break down here, so I will probably save that context for a different post. In brief, the taxonomy in *Calosoma* is a wreck. This arises from over splitting species into sub species and subgenera groups being para/poly-phyletic ([See my advisors paper](https://toussaintlab.files.wordpress.com/2018/02/toussaint-and-gillett-2018-biol-j-linn-soc-rekindling-jeannel_s-gondwanan-vision-phylogenetics-and-evolution-of-carabinae-with-a-focus-on-calosoma-caterpillar-hunter-beetles.pdf "link to pdf") on it if you're curious). Whether or not this has been the correct approach, I've got lots of work ahead of me to resolve these relationships.

Needless to say this is the bread and butter of my thesis, and will really drive my research focus in the future. But, I am going on a tangent! In approaching these pipelines I've learned some really useful command line functions that I wish I knew about during my masters studies. So here are three of them.

## Process substitution
`<()` or `>()`

You can use this as a variable in your code for standard output from a function. For example, given a compressed file `foo.txt.gz`, you cannot just

    $ cat foo.txt.gz

Of course, this is because its compressed. It simply cannot take its input! However, you can call it like this to count lines with `>` in them.

    gunzip -c foo.txt.gz | grep > | wc -l

However, process substition is really useful in for loops or programs that don't accept zipped files as input. This is because you can place the function where a standard input file might exist.

Basically, this _won't_ work:

    $ program -options fast -input foo.txt.gz

But his _will_ work:

    $ program -options fast -input <(gunzip -c foo.txt.gz)

A more technical explanation of processes substation can be found at the [gnu.org manual](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html "process substitution") or nicely explained in this [stackexchange post.](https://unix.stackexchange.com/a/609384 "by Stéphane Chazelas")

## Parsing a CSV file

OK, so I will admit, I don't know exactly what is going on here. I just know it works really well for what I need it to do. A little breif background on why  I use this approach. I learned from Dr. Gerry Carter at OSU while I was, albeit briefly, advising someone on a project. Dr. Carter insisted on a "master" file with data/info we point to in programming languages (R, python, linux) rather than manipulating that original file. Basically, once you've collected your data, don't manipulate that file any further! Taking that philosophy forward to my bioinformatics work I figure I can use one master csv file to work with.

Say you have a csv file with accession numbers you want to call from [NCBI's sequencing database](https://www.ncbi.nlm.nih.gov/sra "Sequencing Read Archive") called example.csv and you want to use information from that csv to run some code. Your example file has a header with column names, and the content we are interested in. Now we might not need all the data at every step, but its important to keep it associated as we might need that information down the line.

example.csv:

    accessionNo,taxon,dataType
    SRR12339051,Adelotopus_paroensis_SRR12339051,hybrid-capture
    SRR12339069,Broscus_cephalotes_SRR12339069,hybrid-capture
    SRR12339144,Calathus_sp_SRR12339144,hybrid-capture
    SRR12339143,Calophaena_bicincta_SRR12339143,hybrid-capture

Whats really cool is we can iterate over rows and pull data from the column using the following code that downloads SRA's using [NCBI's sratoolkit](https://github.com/ncbi/sra-tools "sratools gibhub") fastq-dump.

    exec < ./example.csv
    read header
    while IFS="," read -r accessionNo taxon dataType
    do
        fastq-dump --outdir ./fastq/$taxon --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip $accessionNo
    done


Here is my understanding of what is going on:

`exec < ./example.csv`

We are telling bash to look at this file for the following commands:

`read header`

Read the previously "exec'd" files header:

`while IFS="," read -r accesionNo taxon datatype`

Here we start our while loop. While reading the file, use these fields. Those fields are indicated using IFS, the input field separator. In this case, its a comma (csv: comma seperated values). As an important note, call all the header columns or it breaks!

    fastq-dump --outdir ./fastq/$taxon --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip $accessionNo


This is, IMO, where the code is really helpful. You are able to call the columns information by row into the function, rather than calling them from a list. So our first pass of the loop would look like this:

    fastq-dump --outdir ./fastq/Adelotopus_paroensis_SRR12339051 --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip SRR12339051

Each subsequent loop would move to the next line and populate the $taxon and $accessionNo with that rows information. There are probably lots of really useful applications for this. But I still need to tease out what each part of this code is doing before the while loop and why.

## Create a log file from a for loop

    for i in `cat foo.txt'
    do
        ...
    done |& tee -a function.log

This is really useful at the end of a for loop. Some times a program doesn't write useful logs, or the information you want what is printed to the terminal and not the log file. So here, at the end of a loop with a program that has terminal output, you can forward that output to a log file (function.log, function-log.txt, or what ever!) without overwriting what the previous loop did.


## Keep in mind
1) I am not a computer scientist, at best these clunky commands are just helpful to solve problems for me.

2) There are always other really efficient ways to solve these problems.

3) This is a way for me to document the code I hae been using so I wont forget it! ;)

Hopefully by the time this post is up, I will have figured out how to enable comments on this blog. If so, maybe you could let me know how you might do things differently or if you know what some of this code is actually doing you can point me in the right direction.

I hope you found this helpful, until next time!
