# miRNA Analyis Pipeline: From FastQ to DEseq

## Introduction

The document is intended to be used as a reference point for miRNA analysis and profiling - There are other methods and tools available that will achieve similar goals but this is how I did it. In all, this document has several uses:

* Allows me to jot down what I've done and why so I don't forget if I need to re-do any analysis.
* Acts as a tutorial for others unfamiliar with miRNA analysis.
* Offers transparency of methods if my work is published - It's a huge pet peeve of mine that when you read a paper, the bioinformatics methods almost always lack detail, making it extremely difficult to replicate. 

## My Samples/Data

RNA was isolated from 9 serum samples of sheep experimentally infected with liver fluke (*Fasciola hepatica*). The serum was collected ~15 weeks post infection and stored at -80c prior to use in this study. See the below table for more details on the samples (NOTE - confirming details of sample 1, hence the ?s in some fields)

![sample_deets](https://user-images.githubusercontent.com/75036690/137309059-f0bcd8d4-89c1-4f7c-8c8c-ef71c23830db.png)

The RNA was isolated using a [Norgen Biotek Kit](https://norgenbiotek.com/product/plasmaserum-circulating-and-exosomal-rna-purification-kit-slurry-format) 
