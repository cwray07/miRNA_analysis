# miRNA_analysis

## Introduction

This doc is a record of my work on profiling miRNAs isolated from serum of sheep infected with *Fasciola hepatica*. It acts both as a reference point for when I return to this analysis, a tutorial for anyone else who is interested in similar work and as a transparency measure for any publications that stem from this research. 

## Sample Deets

RNA was isolated from 9 serum samples, collected from sheep experimentally infected with _Fasciola hepatica_. The samples were collected in 2017 and stored at -80c. See the below image for more sample details. ![Screenshot from 2021-11-04 11-22-00](https://user-images.githubusercontent.com/75036690/140305398-9f14615f-30a1-4f19-af23-265c145cdbc1.png)

## Sequencing

RNA was isolated using [Norgen Plasma/Serum Circulating and Exosomal RNA Purification Kit (Slurry Format)](https://norgenbiotek.com/product/plasmaserum-circulating-and-exosomal-rna-purification-kit-slurry-format) before pre-sequencing quality control was carried out by the [Genomics CTU](https://www.qub.ac.uk/sites/core-technology-units/Genomics/) - No issues were flagged so we continued with sequencing. Following on from this the Genomics CTU completed library prep using a [QIAseq miRNA Library Kit
](https://www.qiagen.com/us/products/discovery-and-translational-research/next-generation-sequencing/metagenomics/qiaseq-mirna-ngs/) prior to sequencing with an Illumina NGS system. 

## Post-Sequencing QC

Post-sequencing we were provided with FASTQ files to which we applied multi-fastQC which showed adapter contamination. 
![fastqc_per_sequence_quality_scores_plot](https://user-images.githubusercontent.com/75036690/141108457-dd6352d7-a638-44ad-a47e-b98fa1a39c36.png)
![fastqc_adapter_content_plot](https://user-images.githubusercontent.com/75036690/141108491-423a627b-b91c-4482-8130-0d96e841ebd6.png)

Therefore we trimmed the adapters using [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) - This was carried out on the Queen's HPC [Kelvin](https://www.qub.ac.uk/directorates/InformationServices/Services/HighPerformanceComputing/) using the following code: 


