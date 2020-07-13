---
layout: post
title: Indentification of genomic intervals using R
subtitle: Matching SNPs to genes or vice versa
tags: [interval, genomics, R]
---

One of my repeating tasks at daily work is to investigate whether genetic variations including single nucleotide variants or larger structural variants are located or cover gene regions. 

Of course, when working with vcf files from e.g. NGS projects mostly the annotation already have been done by the company/bioinformatics using their own bash-pipeline. And if I'm working in my linux command line environment, I will also use tools such as [SnpSift](http://snpeff.sourceforge.net/SnpSift.html), [ANNOVAR](https://doc-openbio.readthedocs.io/projects/annovar/en/latest/) or even the webinterface of the latter one [wannovar](http://wannovar.wglab.org/). But, there are a lot of situations when I have a small list of variants in indefinite format in `R` and need very fast -without any transformation- the overlap with some genomic ranges and intervals. 

In the last years I used the `library(GenomicRanges)` package for that, but now there is a more `tidyverse`-one available: [valr](https://cran.r-project.org/web/packages/valr/index.html) 

Let's start. Assume that we have a dataframe of 10 snps:


```r
library(tidyverse)
library(valr)

snps = readRDS("snps_hg19.RDS")
snps
# A tibble: 10 x 4
   chrom chromStart  chromEnd name       
   <chr>      <int>     <int> <chr>      
 1 chr7   117377028 117377029 rs35683365 
 2 chr7   117449996 117449997 rs115455707
 3 chr7   117380621 117380622 rs4730802  
 4 chr7   117682409 117682410 rs111804712
 5 chr7   117898504 117898505 rs77731080 
 6 chr7   117587439 117587440 rs75409059 
 7 chr7   117699243 117699244 rs115306340
 8 chr7   117337208 117337209 rs61689363 
 9 chr7   117715479 117715480 rs80270370 
10 chr7   117607212 117607213 rs147971215
```

and now we want to know which one is within a gene or not?!

Thus, we first need to download gene regions, if we haven't already a target gene list. I'm using the [UCSC table bowser](https://genome.ucsc.edu/cgi-bin/hgTables?hgsid=858920105_KjYALqAZR68IuW3xGVRCt0Z3oDdc&clade=mammal&org=Human&db=hg19&hgta_group=genes&hgta_track=refSeqComposite&hgta_table=0&hgta_regionType=range&position=chr7%3A117%2C253%2C000-118%2C034%2C999&hgta_outputType=primaryTable&hgta_outFileName=) and the NCBI RefSeq Track for that. 

After downloading our file (chr7:117,253,000-118,034,999) we can load the file directly in R

```r
genes <- read.table("valr/genes_hg19.txt", header = T, stringsAsFactors = F, comment.char = "$") %>% as_tibble
genes
# A tibble: 7 x 16
    bin name  chrom strand txStart  txEnd cdsStart cdsEnd exonCount exonStarts exonEnds score name2
  <int> <chr> <chr> <chr>    <int>  <int>    <int>  <int>     <int> <chr>      <chr>    <int> <chr>
1   184 NM_0~ chr7  +       1.17e8 1.17e8   1.17e8 1.17e8        27 117120078~ 1171202~     0 CFTR 
2     2 NM_0~ chr7  -       1.17e8 1.18e8   1.17e8 1.18e8        23 117350704~ 1173518~     0 CTTN~
3     2 NM_0~ chr7  -       1.17e8 1.18e8   1.17e8 1.17e8        23 117350704~ 1173518~     0 CTTN~
4     2 NM_0~ chr7  -       1.17e8 1.18e8   1.17e8 1.18e8        23 117350704~ 1173518~     0 CTTN~
5     2 NM_0~ chr7  -       1.17e8 1.18e8   1.17e8 1.17e8        23 117350704~ 1173518~     0 CTTN~
6   185 NM_0~ chr7  +       1.18e8 1.18e8   1.18e8 1.18e8         4 117824217~ 1178243~     0 LSM8 
7  1484 NM_0~ chr7  +       1.18e8 1.18e8   1.18e8 1.18e8         7 117864757~ 1178650~     0 ANKR~
# ... with 3 more variables: cdsStartStat <chr>, cdsEndStat <chr>, exonFrames <chr>
```

Now we can check the function `valr::bed_closest()` to find closest intervals.

```r
bed_closest(snps, genes)
Fehler in check_names(x, expect_names) : 
  expected 3 required names, missing: start, end

# Ok, add required columns to both dataframes

snps_gr = snps %>% 
 select(chrom,  start=chromStart,  end=chromEnd, name) 

genes_gr = genes %>% 
 select(chrom, start=txStart, end=txEnd, gene=name2, id=name)

valr::bed_intersect(snps_gr, genes_gr)
# A tibble: 12 x 9
   chrom   start.x     end.x name.x        start.y     end.y gene.y  id.y           .overlap
   <chr>     <int>     <int> <chr>           <int>     <int> <chr>   <chr>             <int>
 1 chr7  117377028 117377029 rs35683365  117350704 117512274 CTTNBP2 NM_001363349.1        1
 2 chr7  117377028 117377029 rs35683365  117350704 117512274 CTTNBP2 NM_001363350.1        1
 3 chr7  117377028 117377029 rs35683365  117350704 117513495 CTTNBP2 NM_033427.3           1
 4 chr7  117377028 117377029 rs35683365  117350704 117513495 CTTNBP2 NM_001363351.1        1
 5 chr7  117449996 117449997 rs115455707 117350704 117512274 CTTNBP2 NM_001363349.1        1
 6 chr7  117449996 117449997 rs115455707 117350704 117512274 CTTNBP2 NM_001363350.1        1
 7 chr7  117449996 117449997 rs115455707 117350704 117513495 CTTNBP2 NM_033427.3           1
 8 chr7  117449996 117449997 rs115455707 117350704 117513495 CTTNBP2 NM_001363351.1        1
 9 chr7  117380621 117380622 rs4730802   117350704 117512274 CTTNBP2 NM_001363349.1        1
10 chr7  117380621 117380622 rs4730802   117350704 117512274 CTTNBP2 NM_001363350.1        1
11 chr7  117380621 117380622 rs4730802   117350704 117513495 CTTNBP2 NM_033427.3           1
12 chr7  117380621 117380622 rs4730802   117350704 117513495 CTTNBP2 NM_001363351.1        1

```

Nice, the output (".x" is added to all `snps_gr` columns and ".y" is added to `genes_gr`)shows that three SNPs overlap with four transcripts of *CTTNBP2*, respectively. 


Result is confirmed using the `GenomicRanges` package: 

```r
GenomicRanges::findOverlaps(GenomicRanges::makeGRangesFromDataFrame(snps_gr), GenomicRanges::makeGRangesFromDataFrame(genes_gr)) %>% as.data.frame() %>% bind_cols(snps_gr[.$queryHits,], genes_gr[.$subjectHits,]) %>% head
  queryHits subjectHits chrom...3 start...4   end...5        name chrom...7 start...8   end...9    gene
1         1           4      chr7 117377028 117377029  rs35683365      chr7 117350704 117513495 CTTNBP2
2         1           5      chr7 117377028 117377029  rs35683365      chr7 117350704 117513495 CTTNBP2
3         1           2      chr7 117377028 117377029  rs35683365      chr7 117350704 117512274 CTTNBP2
4         1           3      chr7 117377028 117377029  rs35683365      chr7 117350704 117512274 CTTNBP2
5         2           4      chr7 117449996 117449997 rs115455707      chr7 117350704 117513495 CTTNBP2
6         2           5      chr7 117449996 117449997 rs115455707      chr7 117350704 117513495 CTTNBP2
              id
1    NM_033427.3
2 NM_001363351.1
3 NM_001363349.1
4 NM_001363350.1
5    NM_033427.3
6 NM_001363351.1
```

Whats about the other variants? Find closets gene region using the `valr::bed_closest` function

```r
valr::bed_closest(snps_gr, genes_gr)
# A tibble: 27 x 10
   chrom   start.x     end.x name.x       start.y     end.y gene.y  id.y           .overlap  .dist
   <chr>     <int>     <int> <chr>          <int>     <int> <chr>   <chr>             <int>  <int>
 1 chr7  117337208 117337209 rs61689363 117350704 117512274 CTTNBP2 NM_001363349.1        0  13496
 2 chr7  117337208 117337209 rs61689363 117350704 117512274 CTTNBP2 NM_001363350.1        0  13496
 3 chr7  117337208 117337209 rs61689363 117350704 117513495 CTTNBP2 NM_033427.3           0  13496
 4 chr7  117337208 117337209 rs61689363 117350704 117513495 CTTNBP2 NM_001363351.1        0  13496
 5 chr7  117377028 117377029 rs35683365 117350704 117512274 CTTNBP2 NM_001363349.1        1      0
 6 chr7  117377028 117377029 rs35683365 117350704 117512274 CTTNBP2 NM_001363350.1        1      0
 7 chr7  117377028 117377029 rs35683365 117350704 117513495 CTTNBP2 NM_033427.3           1      0
 8 chr7  117377028 117377029 rs35683365 117350704 117513495 CTTNBP2 NM_001363351.1        1      0
 9 chr7  117377028 117377029 rs35683365 117120078 117308719 CFTR    NM_000492.4           0 -68310
10 chr7  117380621 117380622 rs4730802  117350704 117512274 CTTNBP2 NM_001363349.1        1      0

```

As you can see, the variants within the genes showed zero distance `.dist`. The other ones showed either positive values reflecting upstream variant position according to gene region, or negative values reffering to a downstream location.


The package comes along with many more nice features and it is very good documented on [github](https://github.com/rnabioco/valr). Used example files can be downloaded [here for snps](/assets/img/snps_hg19.RDS) and [here for the genes](/assets/img/genes_hg19.txt)



