---
layout: post
title: Indentification of genomic intervals using R
subtitle: Matching SNPs to genes or vice versa
#cover-img: assets/img/app_overview.png
tags: [interval, genomics, R]
---

One of my repeating tasks in daily work is to investigate whether genetic variations including single nucleotide variants or larger structural variants are located or cover gene regions. 

Of course, when working with vcf files from e.g. NGS projects mostly the annotation already have been done by the company/bioinformatics using their own bash-pipeline. And if I'm working in my linux command line, I will also use tools such as [SnpSift](http://snpeff.sourceforge.net/SnpSift.html), [ANNOVAR](https://doc-openbio.readthedocs.io/projects/annovar/en/latest/) or even the webinterface [wannovar](http://wannovar.wglab.org/). But, there are a lot of situations when I have a small list of variants in indefinite format in R and need very fast -without any transformation- the overlap with genomic ranges. 

In the last years I used for these tasks the `library(GenomicRanges)` packages, but now there is a more `tidyverse`-one available: [valr](https://cran.r-project.org/web/packages/valr/index.html) 


Assume that we have 50 snps:


```r
snps = readRDS("snps_hg19.RDS")
snps
# A tibble: 50 x 4
   chrom chromStart  chromEnd name       
   <chr>      <int>     <int> <chr>      
 1 chr7   117768468 117768478 rs113541589
 2 chr7   117587439 117587440 rs75409059 
 3 chr7   117523461 117523481 rs59327040 
 4 chr7   117621876 117621877 rs7777391  
 5 chr7   117888644 117888645 rs148453644
 6 chr7   117682409 117682410 rs111804712
 7 chr7   117379902 117379903 rs36073009 
 8 chr7   117625407 117625408 rs57338631 
 9 chr7   117312349 117312350 rs11764070 
10 chr7   117380621 117380622 rs4730802
```
