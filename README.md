# miRNA_analysis

## Introduction

This doc is a record of my work on profiling miRNAs isolated from serum of sheep infected with *Fasciola hepatica*. It acts both as a reference point for when I return to this analysis, a tutorial for anyone else who is interested in similar work and as a transparency measure for any publications that stem from this research. 

## Sample Deets

RNA was isolated from 9 serum samples, collected from sheep experimentally infected with _Fasciola hepatica_. The samples were collected in 2017 and stored at -80c. See the below image for more sample details. ![Screenshot from 2021-11-04 11-22-00](https://user-images.githubusercontent.com/75036690/140305398-9f14615f-30a1-4f19-af23-265c145cdbc1.png)

## Sequencing

RNA was isolated using [Norgen Plasma/Serum Circulating and Exosomal RNA Purification Kit (Slurry Format)](https://norgenbiotek.com/product/plasmaserum-circulating-and-exosomal-rna-purification-kit-slurry-format) before pre-sequencing quality control was carried out by the [Genomics CTU](https://www.qub.ac.uk/sites/core-technology-units/Genomics/) - No issues were flagged so we continued with sequencing. Following on from this the Genomics CTU completed library prep using a [QIAseq miRNA Library Kit
](https://www.qiagen.com/us/products/discovery-and-translational-research/next-generation-sequencing/metagenomics/qiaseq-mirna-ngs/) prior to sequencing with an Illumina NGS system. 

## Post-Sequencing QC

Post-sequencing we were provided with FASTQ files to which we applied multi-fastQC which was generally fine but showed adapter contamination. 
![fastqc_per_sequence_quality_scores_plot](https://user-images.githubusercontent.com/75036690/141108457-dd6352d7-a638-44ad-a47e-b98fa1a39c36.png)
![fastqc_adapter_content_plot](https://user-images.githubusercontent.com/75036690/141108491-423a627b-b91c-4482-8130-0d96e841ebd6.png)

Therefore we trimmed the adapters using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) - This was carried out on the Queen's HPC [Kelvin](https://www.qub.ac.uk/directorates/InformationServices/Services/HighPerformanceComputing/) using the following code: 

```
java -jar <path_to_trimmomatic.jar> SE <name_of_fastq_to_be_trimmed> <name_for_trimmed_file.fastq.gz> ILLUMINACLIP:TruSeq3-SE:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 -trimlog <name_of_trimlog_file>
```

Illummina adapters are built in to trimmomatic and so don't need to be specified beyond ILLUMINACLIP:TruSeq3 but for trimming other adapters, a path to a fasta file containing the adapter sequences must be specified - see [Trimmomatic Manual](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf) for full details. 

Post trimming the FastQ files looked much better in terms of adapter contamination 

![LrQjhyz](https://user-images.githubusercontent.com/75036690/141110331-a4d58153-cb36-43c1-a0bc-58161e957112.png)

But the sequence length distributions were concerning, seeing as we're looking for miRNAs, which should be ~20-25nts in length

![fastqc_sequence_length_distribution_plot (3)](https://user-images.githubusercontent.com/75036690/141110479-ae2d5a29-c0be-4602-8b68-20259c2fc9fa.png)

Peaks at 42 and 60/62 NT is not what we would want to be seeing! FastQC does not check for every single adapter and as it turns out the Qiagen Adapter used during library prep has not yet been removed. This is slightly more complicated as we need to provide Trimmomatic with a FASTA file containing the qiagen adapter sequence. 

A fasta file is simply a text file containing sequence data - for each sequence there is a header/title/identifier line 



## A quick note...

To save you from great frustration and time loss, I recommend running the below line of code on every single fasta/fastq file that miRDeep2 will be using - including your unindexed genomes prior to indexing with bowtie... miRDeep2 HATES white space - even white space that I'm 99% sure doesn't exist... but nethertheless this line of code will find any whitespace and replace it with a "\_" and even 
