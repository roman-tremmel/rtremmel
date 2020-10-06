---
layout: post
title: My first self-developped R-package
subtitle: ggfastman: plotting manhattan and QQ plots in a very fast way using R. 
tags: [statistics, ggplot2, R, GWAS, manhattan, pvalue, QQ, qqnorm, qqline]
---

This is my first try to create a R package on my own. Since a long time i had the idea to develop a very fast way to plot bazillions of pvalues in QQ plots and manhattan plots. 
And just recently I stumbeled over the R package (scattermore)[https://github.com/exaexa/scattermore], which allows exactly this: Plotting many datapoints very fast. 
In brief, the data points are rastered using C code and magic. 

Now, it was easy to combine ggplot2 with scattermore functions to generate the plots I wanted. Ans since I need manhattan and QQ plots very often in my daily live, I put all the code in one package for easy installation and use:
(ggfastman)[https://github.com/roman-tremmel/ggfastman]

To install the package you have to run

    devtools::install_github("roman-tremmel/ggfastman", upgrade = "never")
	# and 
	devtools::install_github('exaexa/scattermore', dependencies = F, force = T, upgrade = "never")

Then you can load some example data 


```{r}
library(ggfastman)
data("gwas_data")
```


and runs ggplot2 code to get a nice outout

```{r}
library(tidyverse)
library(ggrepel)
fast_manhattan(gwas_data, build='hg18', speed = "fast", color1 = "pink", color2 = "turquoise", pointsize = 3, pixels = c(1000, 500)) +
  geom_hline(yintercept = -log10(5e-08), linetype =2, color ="darkgrey") + # genomewide significance line
  geom_hline(yintercept = -log10(1e-5), linetype =2, color ="grey")  + # suggestive significance line
  ggrepel::geom_text_repel(data = . %>% group_by(chr) %>% # ggrepel to avoid overplotting
                             top_n(1, -pvalue) %>% # extract highest y values
                             slice(1) %>% # if there are ties, choose the first one
                             filter(pvalue <= 5e-08), # filter for significant ones 
                             aes(label=rsid), color =1) # add top rsid 
```
![boxplot_with_pvalues](/assets/img/manhatten.png)

You can find more examples on (github)[https://github.com/roman-tremmel/ggfastman]