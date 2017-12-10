---
layout: main
title: De novo genome assembly
categories: [genomics, genome assembly]
permalink: /genomics/de_novo_assembly
---

{% include _genomics_de_novo_assembly_toc.html %}

{% include _side_tab_genomics.html %}

<br>
<center><img src="{{ site.url }}/images/under_construction.jpeg"></center>

<br>
<h1><center>Under construction...</center></h1>
<br>

---
---
<br>

---
---
<br>

---
---
<br>

Here we're going to run through some of the things I do when assembling and analyzing a newly sequenced isolate genome. But first, the important part:

<div class="warning">
<h2>ATTENTION!</h2>
This is not an authoritative, exhaustive, or standard workflow for working with a newly sequenced genome! No such thing exists. All genomes, datasets, and goals are different, and new tools are constantly being developed. The point of this page is just to give examples of some of the things you can do, for people who may be completely new to the arena and would benefit from walking through some of these things just for the exposure. Don't let anything here, or anywhere, constrain your science to doing only what others have done!</div>

Now that that's out of the way, let's get to it.  
<br>

---
---
<br>
# Tools used here
We'll be using a variety of tools that are listed here with the versions I employed running things on either [my personal computer (MacOSX - Darwin) and/or on a server (Linux - Ubuntu)](https://www.quora.com/Whats-the-difference-between-Mac-OS-X-Darwin-OS-and-a-popular-Linux-distribution-like-Ubuntu-What-can-be-done-on-Darwin). Help for installing, or links to already available really helpful installation instructions, can be found [here](/bash/installing_tools). 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [FastQC v0.11.5](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [Trimmomatic v0.36](http://www.usadellab.org/cms/?page=trimmomatic)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [SPAdes v3.11.0](http://cab.spbu.ru/software/spades/)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [MegaHit v1.1.2](https://github.com/voutcn/megahit)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [QUAST v4.5](https://github.com/ablab/quast)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [bowtie2 v2.2.5](https://github.com/BenLangmead/bowtie2)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [anvi'o v3](http://merenlab.org/software/anvio/)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [FastTree v2.1.10](http://www.microbesonline.org/fasttree/)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; • [RAxML v8.2.9](https://github.com/stamatak/standard-RAxML)  
<br>

---
<br>
# The data
The practice data we're going to use here was provided by colleagues at the [J. Craig Venter Institute](http://www.jcvi.org/). In working out the details for a rather large-scale project which in part involves sequencing a bunch of *Burkholderia* isolates from the ISS and performing de novo genome assemblies, [Aubrie O'Rourke](https://www.linkedin.com/in/aubrie-o-rourke-94975a6a/) and her team put an already sequenced isolate – [*Burkholderia cepacia* (ATCC 25416)](https://www.atcc.org/products/all/25416.aspx) – through their pipeline in order to refine as needed and to have something to benchmark their expectations against. The sequencing was done with Illumina's Nextera kit as paired-end 2x150 bps, with ~350bp insert.

If you'd like to follow along rather than just reading through, you can copy and paste the following commands into your terminal to download the sequence data we're working with. The downloaded directory contains subdirectories with both a full dataset and a subsampled dataset, and then within those a subdirectory the starting data and one containing all intermediate and result files. Working with the full dataset can take a decent amount of time for some steps, and some may require more memory than a personal computer offers. So you have lots of choices regarding how much you'd like to commit yourself here 🙂 .
 
If you're not sure what the following commands are doing, you can find an explanation [here](/bash/basics#bottom), and since almost all of this work happens at the command line, you should probably already be somewhat comfortable with [*bash*](/bash/). 

```
cd ~
curl -O https://AstrobioMike.github.io/tutorial_files/genomics_de_novo_temp.tar.gz
tar -xvf genomics_de_novo_temp.tar.gz
rm genomics_de_novo_temp.tar.gz
cd genomics_de_novo_temp
```  
<br>

---
<br>
# Quality filtering
Assessing the quality of your sequence data and filtering appropriately should pretty much always be the first thing you do with your dataset. A great tool to get an overview of what you're starting with is [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).
<br>
## FastQC
[FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) scans the fastq files you give it to generage a broad overview of some summary statistics, and has several screening modules that test for some commonly occurring problems. (But as the developers note, its modules are expecitng random sequence data, and any warning or failure notices the program generates should be interpreted within the context of your experiment.) It produces an html output for each fastq file of reads it is given (they can be gzipped), and can be run like such:

```bash
fastqc B_cepacia_raw_R1.fastq.gz B_cepacia_raw_R2.fastq.gz
```

The resulting html output files can be opened and explored showing all of the modules FastQC scans. Some are pretty straightforward and some take some time to get used to to interpret. It's also hard to know what to expect when  you are looking at the output for the first time without any real baseline experience of what good vs bad examples look like, but the developers provide some files to demonstrate some circumstances. For instance, [here](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/good_sequence_short_fastqc.html) is a good example output, and [here](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/bad_sequence_fastqc.html) is a relatively poor one. You should also look over the helpful links about each module provided [here](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/). 

Looking at our output from the forward reads (B_cepacia_raw_R1_fastqc.html), not too much stands out other than the quality scores are pretty mediocre from about 1/3 of the way through the read on: 

<center><img src="{{ site.url }}/images/fastqc_before.png"></center>  

<br>
Here the read length is stretched across the x-axis, the blue line is the mean quality score of all reads at the corresponding positions, red line is the median, and the yellow boxplots represent the interquartile range, and the whiskers the 10th and 90th percentiles. The reverse reads look very similar, you can open that html file (R2) as well if you'd like. Sometimes this will reveal there are still adapters from the sequencing run mixed in, which would wreak havoc on assembly efforts downstream. Getting this type of information from FastQC helps us determine what parameters we want to set for our quality filtering/read trimming.
<br>
## Trimmomatic
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) is a pretty flexible tool that enables you to trim up your sequences based on several quality thresholds and some other metrics (like minimum length or removing adapters and such). Since the summary from [FastQC]() wasn't all that terrible other than semi-low quality scores, for a first pass I just ran Trimmomatic with pretty generic, but stringent settings:

```bash
java -jar ~/happy_bin/Trimmomatic-0.36/trimmomatic-0.36.jar PE B_cepacia_raw_R1.fastq.gz B_cepacia_raw_R2.fastq.gz BCep_R1_paired.fastq.gz BCep_R1_unpaired.fastq.gz BCep_R2_paired.fastq.gz BCep_R2_unpaired.fastq.gz LEADING:10 TRAILING:10 SLIDINGWINDOW:5:20 MINLEN:151 -threads 20
```

The syntax for how to run Trimmomatic can be found in their [manual](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf), but our filtering thresholds here start with "LEADING:10". This says cut the bases off the start of the read if their quality score is below 10, and we have the same set for the end with "TRAILING:10". Then the sliding window parameters are 5 followed by 20, which means starting at base 1, look at a window of 5 bps and if the average quality score drops before 20, truncate the read at that position and only keep up to that point. The stringent part comes in with the MINLEN:151 at the end. Since the reads are already only 151 bps long, this means if any part of the read is truncated due to those quality metrics set above the entire read will be thrown away.  

The output from that shows us that only about 14% of the read pairs (both forward and reverse from the same fragment) passed, leaving us with only ~600,000 read pairs. That sounds low, but since we know what we're working with here (meaning it's an isolate of a known genus and not a metagenome or something completely unknown), we can pretty quickly estimate if this could even possibly be enough depth for us to assemble the genome we're expecting. Assuming those reads were perfect quality and perfectly evenly distributed (which they're not), that would be (600,000 paired reads) * (302 bps per paired read) = 181.2 Mbps covered. Most *Burkholderia* are around 8.5 Mbps, meaning we'd have around 20X coverage right now, if all was perfect. This confirms that this is a little low and we should probably adjust our stringency on filtering – I don't think there are solid rules on this either that always hold true, but in my (albeit limited) experience ~50–100X coverage is more around where you want to be for de novo assembly of a typical prokaryotic genome.  

So, I went back and altered how I was filtering a bit. Since the worst part of the reads, quality-wise, is at the end, I decided to chop off the last few bps of each read *before* beginning to do the filtering steps (the parameters you enter into Trimmomatic are carried out in the order in which they appear – though this is usually *not* the case with most programs at the command line). The length we want to trim the reads down to counts from the 5' end of each read, and is set with the `CROP:140` parameter in the following. And note that the minimum length is now changed too, otherwise nothing would have made it through. 

```bash
java -jar ~/happy_bin/Trimmomatic-0.36/trimmomatic-0.36.jar PE B_cepacia_raw_R1.fastq.gz B_cepacia_raw_R2.fastq.gz BCep_R1_paired.fastq.gz BCep_R1_unpaired.fastq.gz BCep_R2_paired.fastq.gz BCep_R2_unpaired.fastq.gz CROP:140 LEADING:10 TRAILING:10 SLIDINGWINDOW:5:20 MINLEN:140 -threads 20
```

These settings allowed ~36% of paired reads through (~1.5 million pairs), which by the same quick estimation we did above suggests we could possibly have around 50X coverage. 36% is still a low amount of our total starting data, but of course having less "good data", is better than having more "bad data" – especially when bad sequence data could severly inhibit our assembly efforts.  

I decided to move forward with this here. But keep in mind this process doesn't need to be linear. While you can't try *everything* in the world trying to get the best out of your data, you can try *a lot* of things. So just like we're going to run a few different assemblers below with different settings and compare them, we could just as easily test different assemblers with different quality-filtered data going into them.  

And just for a peek at the FastQC output after our trimming:

<center><img src="{{ site.url }}/images/fastqc_after.png"></center>  

<br>
Things still don't look perfect, but they look much cleaner than before – now our interquartile boxes (yellow) are much more snuggly sitting up top telling us our distribution of higher qualities across the end of the reads is much better. And though they weren't much of a factor here, don't forget to keep an eye on all the other modules from FastQC with any data you throw into it. They are not designed to be perfect assessments, because different experimental conditions can lead to warnings and such for expected reasons as mentioned above, but they are very useful in that they can point you toward potential problems you might not otherwise see. When something does catch your eye, open up the manual for that module [here](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/) and revisit exactly what it can mean and what can cause it.

<br>

---
<br>
# Assembly
Now that we have our reads quality filtered, we're ready to move on to assembling them. There are lots of assembly programs out there, and once again, there is no one-size-fits-all. Your data have a lot to say about which assembler is going to work the "best", and that's not really a straightforward criterion to shoot for either. I've had really good results with [SPAdes](http://cab.spbu.ru/software/spades/) for isolate or enrichment cultures where I'm trying to reconstruct just one or a few genomes. But when working with high diversity metagenomic samples, SPAdes will need too much memory and [MegaHit](https://github.com/voutcn/megahit) is pretty awesome with how well it does with such a small memory footprint, and it's insanely fast.  

Whenever I am working with a new dataset, I generate multiple assemblies testing different programs with different parameters and then compare the results so I can feel at least somewhat confident that I'm doing the best that can currently be done with the data.  

Here I ran a couple with [SPAdes](http://cab.spbu.ru/software/spades/) and a couple with [MegaHit](https://github.com/voutcn/megahit). I will note that I've consistently found that incorporating an error-correction step tends improve assembly results, and the one I happen to use is available through the SPAdes program. So even when I end up using the assembly from another program, I typically put into it error-corrected reads from SPAdes. A default run with the current version will run the error-correction step and save the reads from it so you can then use them elsewhere, but if you don't want to do the assembly with SPAdes you can also run it with the `--only-error-correction` flag set.  

The last thing I'll note is that these assembly steps can take a good chuck of time, so I ran them on a server with multiple cpus. If you're running things on your own computer, it's probably better to be doing this with the subsampled dataset. 
<br>
## SPAdes
As I mentioned above, I've had great results with [SPAdes](http://cab.spbu.ru/software/spades/) before when working with axenic or enrichment cultures, so I anticipate that to be the case here. I first ran an assembly with default settings:

```bash
spades.py -1 BCep_R1_paired.fastq.gz -2 BCep_R2_paired.fastq.gz -t 25 -o spades_default_assembly -m 500
```

And then after doing some scanning of the documentation to see what parameters I'd like to try varying, I saw [this note](http://cab.spbu.ru/files/release3.11.1/manual.html#sec3.4) suggesting assembly settings when using 2x150 paired-end Illumina data. It recommends that if you 50X+ coverage (which we were right around based on our rough calculation above), to try setting the kmer lengths to 21,33,55,77, which is the default **if** your read lengths are 150 (or longer I presume), and to run the assembler in `--careful` mode (which is not the default). Since the trimming of our reads we did above put our reads just under 150 bps long, we'll have to set that parameter ourselves if we want that last kmer of 77 to be run, and we'll have to add the flag for cafeful mode. Also, to save time, I input the error-corrected reads here from the previous run, and added the `--only-assembler` flag, as that step is very slow and we don't need to run it again. 

```bash
spades.py -1 spades_default_assembly/corrected/BCep_R1_paired.fastq.00.0_0.cor.fastq.gz -2 spades_default_assembly/corrected/BCep_R2_paired.fastq.00.0_0.cor.fastq.gz -t 25 -o spades_kmers_set_careful_assembly -m 500 -k 21,33,55,77 --careful
```
## MegaHit
I also ran two assemblies with [MegaHit](https://github.com/voutcn/megahit), both with default settings, but one with the quality-filtered reads as input:

```bash
megahit -m 0.25 -t 20 -1 BCep_R1_paired.fastq.gz -2 BCep_R2_paired.fastq.gz -o megahit_default_assembly
```

And one with the quality-filtered *and* error-corrected reads from SPAdes:

```bash
megahit -m 0.25 -t 20 spades_default_assembly/corrected/BCep_R1_paired.fastq.00.0_0.cor.fastq.gz -2 spades_default_assembly/corrected/BCep_R2_paired.fastq.00.0_0.cor.fastq.gz -o megahit_default_err_corr_assembly
```
Now that we have a handful of assemblies done, let's see how they compare.  
<br>

---
<br>
# Comparing assemblies
Let's just get this out there right off the bat, there is no individual metric that exists to determine if you have a good assembly or not, especially if you have no reference, and *especially* especially if you're working with a metagenomic assembly. There are some general statistics you can look at, like N50 or largest contig, etc. But for one, these don't have any context unless you're comparing multiple assemblies of the same data, and two, I've had metagenomic assemblies with "worse" summary statistics overall, but that enabled me to recover more high-quality bins than an assembly with "better" summary metrics. So you have to keep in mind what your goals are, and know that picking the "best" assembly is not a trivial or straightforward task. Having a reference genome like we do in this case however makes things a lot easier. 
<br>
## QUAST
[QUAST](https://github.com/ablab/quast) is a really nice tool for comparing multiple assemblies, and for metagenome assemblies there is a comparable [MetaQUAST](http://bioinf.spbau.ru/metaquast). We can provide QUAST with: all of our assemblies; a fasta file of our reference genome; and a .gff file of our reference genome that contains information about its genes. I downloaded the two reference files for our *Bulkholderia cepacia* ATCC 25416 from NCBI [here](https://www.ncbi.nlm.nih.gov/genome/10703?genome_assembly_id=255013).  

```bash
quast.py -o quast_B_cep_out -R ref_genomes/BCep_ref.fna -G ref_genomes/BCep_ref.gff -t 5 -l spades_default,spades_kmers_careful,megahit_default,megahit_default_err_corr spades_default_assembly/contigs.fasta spades_kmers_set_careful_assembly/contigs.fasta megahit_default_assembly/final.contigs.fa megahit_default_err_corr_assembly/final.contigs.fa
```
You can find more about the syntax and how to run QUAST [in its documentation](http://quast.bioinf.spbau.ru/manual.html). The output directory contains text files of all the information, but there is also a useful html summary file. Here's a portion of it:

<center><img src="{{ site.url }}/images/quast_output.png"></center>  

<br>
The columns here hold information about each of our 4 assemblies and the rows are different metrics. The majority of the rows starting from the top are in relation to the reference we provided, then the last few starting with "# contigs" are reference-independent. In the interactive html page, you can highlight the row names to get some help on what they mean, and there is more info in the [manual](http://quast.bioinf.spbau.ru/manual.html) of course. The cells are shaded across the assemblies for each row from red to blue, indicating "worst" to "best", but this is only a loose guide to help your eye, as differences can be negligible or up to interpretation, and some rows are more important than others.  

The first thing to notice is that all of them reconstructed about 98.5% of the reference genome, which I think is pretty damn good, but none of them got 100%. This is to be expected for possibly a few reasons, but most likely it's just because short Illumina reads alone aren't able to assemble repetitive regions that extend longer than the paired-read fragment length. We can also see that our assemblies aligned across about 7,550 genes out of the 7,705 that are annotated in the reference genome, and looking at mismatches per 100 kbp we can see we're down around 3-5 SNVs per 100 kbp, which is close enough to be considered monoclonal in my book (is there a consensus definition for prokaryotes on this?). Moving down to the reference-independent section in the table we can see which assemblies cover the most bps with the fewest contigs. This doesn't mean everything either, but fewer contigs covering the same region does provide more information on synteny, which can be very important. 
<br>
## Read recruitment
Another metric to assess the quality of an assemble is to see how well the reads that went into the assembly recruit back to it. It's sort of a way of checking to see how much of the data that went in actually ended up getting used. I usually do this for environmental metagenomes, and it has been informative in some cases there, but here –  with an isolate genome when the few assemblies tested performed pretty similarly – it turns out that they all pretty much recruited reads just as efficiently (with the default settings of [bowtie2 v2.2.5](https://github.com/BenLangmead/bowtie2) at least). So I'm leaving the mention of this in here because it can be helpful with some datasets.  

As far as selecting which of our assemblies to move forward with, since they all performed reasonably well, for the reason of maximizing insight into synteny as mentioned above from the QUAST output, I chose to move forward with the SPAdes assembly done with the `--careful` and specific kmer settings.
<br>
## What about when you don't have a reference?
Again, it's pretty nice here because we have a known reference genome. When that's not that case, it's much harder to be confident you are making the best decisions you can. Then I might go further down the path of processing and analysis with multiple assemblies. Some might be better than others for your particular questions, and that might not be decipherable with summary statistics.  

For instance, if recovering bins from a metagenomes is the goal, you might want to start that process and see which assembly is better suited for that. As I mentioned above, in some cases I've had better results binning out representative genomes from metagenomic assemblies with seemingly worse overall summary statistics. And if I had just stopped at the assembly summary level, and didn't run all of them through my binning process as well, I wouldn't have caught that. Or if you're question is more about the functional potential of the whole community, or even looking for specific genes in a metagenome, and not so much about binning things out, then maybe seeing which assembly gives you better annotations could help you decide. The bottom line is it's difficult, and there is no one answer, but we do the best we can.  
<br>

---
<br>
# Analysis
Now that we/ve selected the assembly we're going to move forward with, we can finaly begin to do some analysis. And because of how damn glorious it is, [anvi'o](http://merenlab.org/software/anvio/) is a great place to start.
<br>
## anvi'o
In one sentence, [anvi'o](http://merenlab.org/software/anvio/) is a very powerful and user-friendly data visualization and exploration platform. It's powerful mostly because its developers perpetually aim to make it inherently as expansive and flexible as possible, and it's user-friendly because they actively provide loads of [well-documented workflows, tutorials, and blog posts](http://merenlab.org/software/anvio/), and there is excellent help on how you can [get anvi'o installed here](http://merenlab.org/2016/06/26/installation-v2/).  

As with all things in this vein, anvi'o isn't meant to be the "one way" you will do things, but rather it is a great platform for integrating many facets of your data. This integration only facilitates your exploration and can help guide you to where you might want to go deeper, but the underlying infrastructure also contains easily accessible, parsed-down tables and files of information ready for you to investigate specific questions at your whim.  

Here we're going to put our isolate genome assembly into the anvi'o framework. Many of the major steps we are going to be performing are laid out in the [metagenomic workflow presented here](http://merenlab.org/2016/06/22/anvio-tutorial-v2/), as much of the processing is the same.  

First we need to generate what anvi'o calls a [contigs database](http://merenlab.org/2016/06/22/anvio-tutorial-v2/#creating-an-anvio-contigs-database). This contains the contigs from our assembly and information about them. The following script will organize our contigs in an anvi'o-friendly way, generate some basic stats about them, and use the program [Prodigal](https://github.com/hyattpd/Prodigal) to identify [open-reading frames](https://en.wikipedia.org/wiki/Open_reading_frame).

```anvio
anvi-gen-contigs-database -f spades_kmers_set_careful_assembly.fa -o contigs.db -n B_cepacia_isolate
```

Now that we have our `contigs.db` that holds our sequences and some basic information about them, we can start adding more. This is one of the places where the flexibility comes into play, but for now we'll move forward with some parts of a general anvi'o workflow, including:

• using the program [HMMER](http://hmmer.org/) with 3 profile hidden Markov models to scan for bacterial single-copy genes [(from Campbell et al. 2013)](http://www.pnas.org/content/110/14/5540.short), archaeal single-copy genes [(from Rinke et al. 2013)](http://www.nature.com/nature/journal/v499/n7459/full/nature12352.html), and bacterial, archaeal, and eukaryotic ribosomal RNAs [(from Tørsten Seemann's Barrnap tool)](https://github.com/tseemann/barrnap) (see the bottom of page 7 [here](http://eddylab.org/software/hmmer3/3.1b2/Userguide.pdf) for a good explanation of what exactly a hidden Markov model is in the realm of sequences)

• using [NCBI COGs](https://www.ncbi.nlm.nih.gov/COG/) to functionally annotate the open-reading frames [Prodigal](https://github.com/hyattpd/Prodigal) predicted with either [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi) or [DIAMOND](https://github.com/bbuchfink/diamond)

• and using a tool called [Centrifuge](https://ccb.jhu.edu/software/centrifuge/manual.shtml#obtaining-centrifuge) taxonomical classification of the found open-reading frames

```bash
  # HMM searching for single-copy genes and rRNAs
anvi-run-hmms -c contigs.db -T 20

  # functional annotation with BLASTp against NCBI's COGs
anvi-run-ncbi-cogs -c contigs.db --num-threads 20 --cog-data-dir ~/happy_bin/anvi_ncbi_cogs/ --search-with blastp

  # exporting Prodigal-identified open-reading frames from anvi'o
anvi-get-dna-sequences-for-gene-calls -c contigs.db -o gene_calls.fa

  # running then through taxonomic classifier
centrifuge -f -x ~/happy_bin/centrifuge_db/nt/nt gene_calls.fa -S centrifuge_hits.tsv -p 10 &

  # importing the taxonomy results into our anvi'o contigs database
anvi-import-taxonomy -c contigs.db -i centrifuge_report.tsv centrifuge_hits.tsv -p centrifuge
```

Ok, great, so we've got quite a bit of info into our contigs database now. The next thing we want to do is recruit our reads that built our assembly, to our assembly, so we can incorporate coverage information. I did the mapping with [bowtie2](https://github.com/BenLangmead/bowtie2) as mentioned above:

```bash
  # building bowtie index from our selected assembly fasta file
bowtie2-build spades_kmers_set_careful_assembly.fa spades_kmers_set_careful_assembly.btindex
  # mapping our reads
bowtie2 -q -x spades_kmers_set_careful_assembly.btindex -1 ../BCep_R1_paired.fastq.gz -2 ../BCep_R2_paired.fastq.gz -p 25 -S spades_kmers_set_careful_assembly.sam
  # converting to a bam file
samtools view -bS spades_kmers_set_careful_assembly.sam > B_cep_assembly.bam  
  # sorting and indexing our bam file (can be done with samtools also)
anvi-init-bam B_cep_assembly.bam -o B_cep.bam
```
And we then integrate this mapping information into anvi'o with the `anvi-profile` program. There is a lot going on with this step and a lot of things you can play with, discussed [here](http://merenlab.org/2016/06/22/anvio-tutorial-v2/#anvi-profile), but for our purposes right now it's mostly about giving us coverage information for each contig:

```bash
anvi-profile -i B_cep.bam -c contigs.db -M 1000 -T 8 --cluster-contigs -o B_cep_profiled/
```
And now that that's done, we're ready to take a look at our assembly in anvi'o's interactive mode:

```bash
anvi-interactive -c contigs.db -p B_cep_profiled/PROFILE.db --title "B. cepacia assembly"
```
<br>

<center><img src="{{ site.url }}/images/fresh_anvi.png"></center>  

<br>
So there is a lot going on here at first glance, especially if you're not yet familiar with how anvi'o organizes things. The interactive interface is extraordinarily expansive and I'd suggest reading about it [here](http://merenlab.org/2016/06/22/anvio-tutorial-v2/#anvi-interactive) and [here](http://merenlab.org/2016/02/27/the-anvio-interactive-interface/) to start, but there's lots more.  

For our purposes here, at the center of the figure is a hierarchical clustering of the contigs from our assembly, here clustered based on tetranucleotide frequency and coverage. So each leaf represents a contig (or a fragment of a contig as each is broken down into a max of ~20,000bps, but for right now I'll just be referring to them as contigs). Then radiating out from the center are layers of information ("Parent", "Taxonomy", "Length", etc.), with each layer displaying information for each contig.  

The first thing that jumps out to me here is the second layer colored light purple, labeled "Taxonomy". There is actually a color for each of contig for whatever taxonomy was assigned to the majority of genes in that particular contig, showing here that the genes in almost the entire assembly were identified as *Burkholderia* – minus the one white bar at ~3:00 o'clock which was not classified as anything. The next thing that stands out is how stable the mean coverage is across all contigs, other than mostly just that same area on the right side, where the 2 identified ribosomal RNAs are found. This is to be expected, because of their highly conserved nature they are difficult to assembly and generally have more non-specific mapping problems. This is great and shows the culture really seems to have been axenic.  

Just for a quick comparison, here is the same typ of figure, but from an enrichment culture:  
<br>

<center><img src="{{ site.url }}/images/other_anvi.png"></center>  

<br>
Here the highlighted contigs, labeled "Bin 1", represent the targeted cultivar, demonstrating a nice example of how anvi'o can help you manually curate bins you're trying derive from assemblies.  

Back to our gloriously clean example, we can select all of our contigs and check estimated levels of completion and redundancy based on the single-copy marker genes we scanned for earlier:  
<br>

<center><img src="{{ site.url }}/images/per_com_anvi.png"></center>  

<br>
Which shows us based on the [Campbell et al. 2013](http://www.pnas.org/content/110/14/5540.short) bacterial single-copy marker genes, we doing pretty good. But of course this approach doesn't actually add up to 100% completion and 0% redundancy in all organisms (any?), so for comparison's sake, I ran the ATCC 25416 reference genome through the same pipeline and it seems these are just the numbers for this genome based on this marker-gene set:  
<br>

<center><img src="{{ site.url }}/images/ref_per_com_anvi.png"></center>  

<br>
Though we still see here this doesn't mean we were able to reconstruct the entire isolate genome, as we are ~150Mbps short, like we saw with the QUAST output above.  

Okay, so we didn't need much manual curation in this case, but it was good to visually inspect the coverage of our assembly to make sure nothing weird was going on. Now we're going to start putting our new isolate genome into some context.  
<br>

---
<br>

<br>
<center><img src="{{ site.url }}/images/under_construction.jpeg"></center>

<br>
<h1><center>Under construction...</center></h1>
<br>



## Phylogenomics
Pull near-relative references to estimate where your newly acquired isolates fit in:
<br>

<center><img src="{{ site.url }}/images/phylo_syn.png"></center>  

<br>

---
<br>
## Distributions
Pull available metagenomes to assess distributions of newly aquired genomic lineages:
<br>

<center><img src="{{ site.url }}/images/dist_syn.png"></center>  

<br>

---
<br>
## Pangenomics
Combine pangenomics and metagenomics to characterize population variability and ecological delineations:
<br>

<center><img src="{{ site.url }}/images/pan_syn.png"></center>  

<br>








