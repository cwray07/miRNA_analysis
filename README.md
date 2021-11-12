# miRNA_analysis

## Introduction

This doc is a record of my work on profiling miRNAs isolated from serum of sheep infected with *Fasciola hepatica*. It acts both as a reference point for when I return to this analysis, a tutorial for anyone else who is interested in similar work and as a transparency measure for any publications that stem from this research. 

## Sample Deets

RNA was isolated from 9 serum samples, collected from sheep experimentally infected with _Fasciola hepatica_. The samples were collected in 2017 and stored at -80c. See the below image for more sample details. ![Screenshot from 2021-11-04 11-22-00](https://user-images.githubusercontent.com/75036690/140305398-9f14615f-30a1-4f19-af23-265c145cdbc1.png)

## Sequencing

RNA was isolated using [Norgen Plasma/Serum Circulating and Exosomal RNA Purification Kit (Slurry Format)](https://norgenbiotek.com/product/plasmaserum-circulating-and-exosomal-rna-purification-kit-slurry-format) before pre-sequencing quality control was carried out by the [Genomics CTU](https://www.qub.ac.uk/sites/core-technology-units/Genomics/) - No issues were flagged so we continued with sequencing. Following on from this the Genomics CTU completed library prep using a [QIAseq miRNA Library Kit
](https://www.qiagen.com/us/products/discovery-and-translational-research/next-generation-sequencing/metagenomics/qiaseq-mirna-ngs/) prior to sequencing with an Illumina NGS system. 

## Log in to Kelvin

After generating your SSH key pair and getting it authenticated by the kelvin staff you should be able to log in remotely to kelvin using the following line in your terminal:

```
ssh –p 55890 –i <path_to_kelvin_key> <queens_numbers>@login.kelvin.alces.network
```


Before running FASTQC or anything really please ensure you are in an interactive node or else you'll piss everyone at kelvin HQ off bigtime - see [training docs](https://gitlab.qub.ac.uk/qub_hpc/kelvin_training) and [FAQs](https://gitlab.qub.ac.uk/qub_hpc/faq/-/tree/master) for details. As long as you remember to basically never do anything (for example indexing a genome or running a script) within the log in node you'll be grand - Submitting a job via slurm IS done from the login node (see training docs). Also consider things like the amount of memory you'd like to request depending on the task you're doing - fastqc won't need much but bowtie and mirdeep will need much more.

## Post-Sequencing QC

Post-sequencing we were provided with FASTQ files to which we applied multi-fastQC which was generally fine but showed adapter contamination. 
![fastqc_per_sequence_quality_scores_plot](https://user-images.githubusercontent.com/75036690/141108457-dd6352d7-a638-44ad-a47e-b98fa1a39c36.png)
![fastqc_adapter_content_plot](https://user-images.githubusercontent.com/75036690/141108491-423a627b-b91c-4482-8130-0d96e841ebd6.png)

To use FastQC in kelvin you can find the module using 

```
module avail
```

load it using 

```
module load <module>
```

and run FASTQC by entering

```
fastqc <fastq_file>
```

The output will be a couple of different files including a html doc with the results and all relevant figures needed to asses the next steps (as seen above). You can download the files onto your own computer either by using scp (secure copy - see kelvin training docs) or through a third party app such as [filezilla](https://filezilla-project.org/). 

We trimmed the adapters using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) - This was carried out on the Queen's HPC [Kelvin](https://www.qub.ac.uk/directorates/InformationServices/Services/HighPerformanceComputing/) using the following code: 

```
java -jar <path_to_trimmomatic.jar> SE <name_of_fastq_to_be_trimmed> <name_for_trimmed_file.fastq.gz> ILLUMINACLIP:TruSeq3-SE:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 -trimlog <name_of_trimlog_file>
```

Note - Trimmomatic isn't available as a module within kelvin and so can't be loaded in the same way BUT you don't have permission to install new modules, so what can we do? We can get everything we need using wget then simply unzip and run - see [here](https://www.goseqit.com/wp-content/uploads/2018/11/Trimmomatic_tutorial-3.pdf) for details.

Illummina adapters are built in to trimmomatic and so don't need to be specified beyond ILLUMINACLIP:TruSeq3-E but for trimming other adapters, a path to a fasta file containing the adapter sequences must be specified - see [Trimmomatic Manual](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf) for full details. 

Post trimming the FastQ files looked much better in terms of adapter contamination 

![LrQjhyz](https://user-images.githubusercontent.com/75036690/141110331-a4d58153-cb36-43c1-a0bc-58161e957112.png)

But the sequence length distributions were concerning, seeing as we're looking for miRNAs, which should be ~20-25nts in length

![fastqc_sequence_length_distribution_plot (3)](https://user-images.githubusercontent.com/75036690/141110479-ae2d5a29-c0be-4602-8b68-20259c2fc9fa.png)

Peaks at 42 and 60/62 NT is not what we would want to be seeing! FastQC does not check for every single adapter and as it turns out the Qiagen Adapter used during library prep has not yet been removed. This is slightly more complicated as we need to provide Trimmomatic with a FASTA file containing the qiagen adapter sequence. 

A fasta file is simply a text file containing sequence data - for each sequence there is a header/title/identifier line starting with a greater than symbol (>) with subsequent lines containing sequence data. When opening in nano file editor, the fasta file for the Qiagen adapter looks like this:

![Screenshot from 2021-11-10 15-45-30](https://user-images.githubusercontent.com/75036690/141145113-c0dbe704-14a7-43a0-a10b-7fb97aaf5355.png)

Now the following code I use _works_ but I'm confident it's not the way it was intended to be used - Basically I removed the "TruSeq3-SE" section and replaced it with the file name (in the same working directory) of the fasta file containing the qiagen adapter, giving the following code: 

```
java -jar <path_to_trimmomatic.jar> SE <name_of_fastq_to_be_trimmed> <name_for_trimmed_file.fastq.gz> ILLUMINACLIP:qiagen_3\'_adapter.fasta:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 -trimlog <name_of_trimlog_file>
```
The only thing that's changed here is adding 'qiagen_3\\'\_adapter.fasta' where before it stated TruSeq3-SE. Note the use of the backslash before the apostrophe - this is an escape character that essentially tells the computer that the ' is not the start of a quote or anything else but is simply part of the file name. You can read up on escape characters [here](https://en.wikipedia.org/wiki/Escape_character)

So now if we run fastqc we can see the distribution is much nicer

![msedge_9SzgVkXphp](https://user-images.githubusercontent.com/75036690/141295713-83cdc489-4c10-4a34-9c2d-2920bbb9bf22.png)

Note that we have a peak around 30bp - this could be [piRNA](https://en.wikipedia.org/wiki/Piwi-interacting_RNA), another form of small RNA which is known to have peaks around this length but for now we'll be focusing on miRNAs

## A quick note...

To save you from great frustration and time loss, I recommend running the below line of code on every single fasta/fastq file that miRDeep2 will be using - including your un-indexed genomes prior to indexing with bowtie... miRDeep2 HATES white space - even white space that I'm 99% sure doesn't exist... but nevertheless this line of code will find any white-space and replace it with a "\_" and even if there is none it will ensure miRDeep doesn't shout at you a later time - it's important to do this now as if you do it post mapping then you'll have to restart from the beginning (i.e. exactly what happened to me). 

```
module load bbtools/38.63
reformat.sh in=<genome.fa> out=<newname_genome.fa> underscore
```
Note, this can (and should) be used on any fasta file, not just genomes - For example later on you'll need some miRNA reference files from miRBase - these are rife with both real and imaginary white-space so run this code on them before using to avoid the wratch of miRDeep.
Second Note. **module load** is how you load modules in kelvin (amazing I know). other useful ones are **module avail** which lists all modules you can activate (very useful for ones with weird names that you can't remember) and **module unload** which does what it says on the tin - you'll be using this later also to get around a pesky perl error.

## Genome downloads and indexing

Now that we have trimmed fastq files, we need download genomes for our species of interest - in this case the Sheep (_Ovis aries_) and the liver fluke (_Fasciola hepatica_). We can find these easily enough - for mammal genomes you can download them from [Ensembl](http://www.ensembl.org/info/data/index.html) and for parasites you can get them from [WormBase ParaSite](https://parasite.wormbase.org/ftp.html). Rather than downloading them to your laptop and uploading them to kelvin it's best to download them straight into your scratch space.

SSH your way into your kelvin account and navigate to your scratch space with:

```
cd /mnt/scratch2/users/<queens_num>
```

This is where you should have already stored your fastq files (now trimmed) but if not you can mv them here with:

``` 
mv <fastq_file> <destination path>
```

mv can also be used to rename files if instead of providing a different directory for the second option you provide a new filename. You could also move all your fastQ files at once by typing:

``` 
mv *.fq <destination path>
```

check the extension of your files is .fq as it may be .fasta, .fa, .fastq or .gz if it's compressed. Another small tip - if you put the text before the * then the function will be applied to any file that STARTS with the specified characthers rather than ends with. For example, if you wanted to move all files that started with "Sample_1"

```
mv Sample_1* <destination_path>
```

But anyway back to downloading genomes... once you've found the link for your genome of interest right click and copy the link then you can use wget to download it into your current working directory

```
wget <link_to_genome>
```

The genome will probably be with a .gz extension and needs to be unzipped using gunzip, easy as: 

```
gunzip genome
```

Next you'll need to load [bowtie](http://bowtie-bio.sourceforge.net/manual.shtml) which contains a command that allows you to index a genome - this is essential as miRDeep requires an indexed genome for mapping your reads (the next step). You can think of indexing as somewhat analagous to indexing a book - if you know the area within a book a certain setion topic of interest is, you'll find it much quicker than just searching through the whole thing - the same logic applies to certain sequences and an indexed genome, allowing you to narrow down the area to be searched.

Find the module for bowtie within kelvin using module avail - note that there will be a few different versions, we want bowtie NOT bowtie2. 

Once you've found the module load it then you can index your genomes with the following code:

```
bowtie-build <genome.fa> <name_for_indexed_genome>
```

Your name for indexed genome does not need a file extension, this will be applied automaticlly. This will probably take quite a while so you can leave it running if you're in an interactive node or simply wait for it to finish if you've submitted via SLURM (see kelvin training docs)

The output will be a set of 6 files with suffixes .1.ebwt, .2.ebwt, .3.ebwt, .4.ebwt, .rev.1.ebwt, and .rev.2.ebwt.

## miRDeep Mapper

Now that we have trimmed fastq files and indexed genomes we can map our reads to the indexed genomes using [miRDeep2](https://www.mdc-berlin.de/content/mirdeep2-documentation). Firstly you'll need to load the miRDeep2 module so find it using module avail and load it with module load. Then apply the following code:

```
mapper.pl <trimmed_fastq> -e -h -m -p <index_genome> -s <processed_reads_name.fa> -t <mapping_output_name.arf> -v
```

There are a lot of options here! Heres what they mean: -e=fastq input file, -h=parse fastq to fasta, -m=collapse reads, -p=defines the genome file (pre-indexed by bowtie-build), -s=print processed reads to this file, t=print read mappings to this file, v=outputs progress report

For your indexed genome, you'll remember there are 6 different files associated with this - provide the string before the file extension i.e. if your indexed genome contains files like worm.1.ebwt then you would just provide worm for the index_genome option.
