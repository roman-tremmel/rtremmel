---
layout: post
title: Statistics using the rstats R package
subtitle: Simple and pipe-friendly framework for basic statistic tests
tags: [statistics, tidyverse, R, pairwise, tests, pvalue]
---

Install the package via cran or from
(here)\[<a href="https://github.com/kassambara/rstatix" class="uri">https://github.com/kassambara/rstatix</a>\]


    library(tidyverse)
    library(rstatix)
	
Simple descriptive statistics

    mtcars %>% 
      as_tibble() %>% 
      rstatix::get_summary_stats(cyl, disp)

    ## # A tibble: 2 x 13
    ##   variable     n   min   max median    q1    q3   iqr    mad   mean     sd     se     ci
    ##   <chr>    <dbl> <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ## 1 cyl         32   4       8     6     4      8    4    2.96   6.19   1.79  0.316  0.644
    ## 2 disp        32  71.1   472   196.  121.   326  205. 140.   231.   124.   21.9   44.7

or pairwise tests

    mtcars %>% 
      as_tibble() %>% 
      rstatix::pairwise_wilcox_test(disp ~ cyl)

    ## # A tibble: 3 x 9
    ##   .y.   group1 group2    n1    n2 statistic         p     p.adj p.adj.signif
    ## * <chr> <chr>  <chr>  <int> <int>     <dbl>     <dbl>     <dbl> <chr>       
    ## 1 disp  4      6         11     7         1 0.000795  0.000795  ***         
    ## 2 disp  4      8         11    14         0 0.0000276 0.0000828 ****        
    ## 3 disp  6      8          7    14         0 0.000287  0.000574  ***

    # and possible with a grouped data.frame as well

    mtcars %>% 
      as_tibble() %>% 
      group_by(am) %>% 
      rstatix::pairwise_wilcox_test(disp ~ cyl) 

    ## # A tibble: 6 x 10
    ##      am .y.   group1 group2    n1    n2 statistic     p p.adj p.adj.signif
    ## * <dbl> <chr> <chr>  <chr>  <int> <int>     <dbl> <dbl> <dbl> <chr>       
    ## 1     0 disp  4      6          3     4         0 0.05  0.05  *           
    ## 2     0 disp  4      8          3    12         0 0.011 0.022 *           
    ## 3     0 disp  6      8          4    12         0 0.004 0.013 *           
    ## 4     1 disp  4      6          8     3         0 0.019 0.056 ns          
    ## 5     1 disp  4      8          8     2         0 0.044 0.089 ns          
    ## 6     1 disp  6      8          3     2         0 0.139 0.139 ns

which can be easily added to boxplots using ggplot and package
`ggsignif`

    # get pvalues and add x-y- positions for plotting
    stats <- mtcars %>% 
      as_tibble() %>% 
      rstatix::pairwise_wilcox_test(disp ~ cyl) %>% 
      rstatix::add_xy_position()  

    #use like
    ggplot(mtcars, aes(factor(cyl), disp)) +
       geom_boxplot() +
       ggsignif::geom_signif(annotations = stats$p, 
                             y_position = stats$y.position, 
                             xmin = stats$xmin, xmax = stats$xmax)

![](test_files/figure-markdown_strict/ggsignif-1.png)

Get all comparisons of a factor

    mtcars %>% 
      as_tibble() %>% 
      rstatix::get_comparisons(cyl)

    ## $V1
    ## [1] "4" "6"
    ## 
    ## $V2
    ## [1] "4" "8"
    ## 
    ## $V3
    ## [1] "6" "8"

    # which replaces this one very well
    combn(unique(mtcars$cyl), 2, simplify = F)

    ## [[1]]
    ## [1] 6 4
    ## 
    ## [[2]]
    ## [1] 6 8
    ## 
    ## [[3]]
    ## [1] 4 8

And correlations are possible as well.

    mtcars %>% 
      as_tibble() %>% 
      rstatix::cor_mat()

    ## # A tibble: 11 x 12
    ##    rowname   mpg   cyl  disp    hp   drat     wt   qsec     vs     am   gear   carb
    ##  * <chr>   <dbl> <dbl> <dbl> <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
    ##  1 mpg      1    -0.85 -0.85 -0.78  0.68  -0.87   0.42   0.66   0.6    0.48  -0.55 
    ##  2 cyl     -0.85  1     0.9   0.83 -0.7    0.78  -0.59  -0.81  -0.52  -0.49   0.53 
    ##  3 disp    -0.85  0.9   1     0.79 -0.71   0.89  -0.43  -0.71  -0.59  -0.56   0.39 
    ##  4 hp      -0.78  0.83  0.79  1    -0.45   0.66  -0.71  -0.72  -0.24  -0.13   0.75 
    ##  5 drat     0.68 -0.7  -0.71 -0.45  1     -0.71   0.091  0.44   0.71   0.7   -0.091
    ##  6 wt      -0.87  0.78  0.89  0.66 -0.71   1     -0.17  -0.55  -0.69  -0.580  0.43 
    ##  7 qsec     0.42 -0.59 -0.43 -0.71  0.091 -0.17   1      0.74  -0.23  -0.21  -0.66 
    ##  8 vs       0.66 -0.81 -0.71 -0.72  0.44  -0.55   0.74   1      0.17   0.21  -0.570
    ##  9 am       0.6  -0.52 -0.59 -0.24  0.71  -0.69  -0.23   0.17   1      0.79   0.058
    ## 10 gear     0.48 -0.49 -0.56 -0.13  0.7   -0.580 -0.21   0.21   0.79   1      0.27 
    ## 11 carb    -0.55  0.53  0.39  0.75 -0.091  0.43  -0.66  -0.570  0.058  0.27   1

and similar to the `Hmisc::rcorr()` function the pvalues are pre
calculated and can be easily extracted as well:

    mtcars %>% 
      as_tibble() %>% 
      rstatix::cor_mat(method = "sp")  %>% 
      rstatix::cor_get_pval()

    ## # A tibble: 11 x 12
    ##    rowname      mpg       cyl     disp        hp     drat       wt     qsec        vs       am    gear      carb
    ##    <chr>      <dbl>     <dbl>    <dbl>     <dbl>    <dbl>    <dbl>    <dbl>     <dbl>    <dbl>   <dbl>     <dbl>
    ##  1 mpg     0.       4.69e- 13 6.37e-13 5.09e- 12  5.38e-5 1.49e-11  7.06e-3 6.19e-  6  8.16e-4 1.33e-3 4.34e-  5
    ##  2 cyl     4.69e-13 7.44e-232 2.28e-14 1.87e- 12  1.94e-5 3.57e-10  6.20e-4 1.48e-  8  2.18e-3 7.68e-4 5.02e-  4
    ##  3 disp    6.37e-13 2.28e- 14 0.       6.79e- 10  1.61e-5 3.35e-12  8.11e-3 2.86e-  6  1.35e-4 3.33e-4 1.43e-  3
    ##  4 hp      5.09e-12 1.87e- 12 6.79e-10 2.27e-236  2.28e-3 1.95e- 7  3.11e-5 7.13e-  7  4.16e-2 6.39e-2 1.80e-  6
    ##  5 drat    5.38e- 5 1.94e-  5 1.61e- 5 2.28e-  3  0.      7.59e- 7  6.17e-1 1.02e-  2  1.43e-5 1.01e-6 4.95e-  1
    ##  6 wt      1.49e-11 3.57e- 10 3.35e-12 1.95e-  7  7.59e-7 0.        2.15e-1 4.13e-  4  1.45e-6 2.16e-5 3.58e-  3
    ##  7 qsec    7.06e- 3 6.20e-  4 8.11e- 3 3.11e-  5  6.17e-1 2.15e- 1  0.      6.86e-  8  2.64e-1 4.18e-1 4.15e-  5
    ##  8 vs      6.19e- 6 1.48e-  8 2.86e- 6 7.13e-  7  1.02e-2 4.13e- 4  6.86e-8 7.44e-232  3.57e-1 1.17e-1 9.88e-  5
    ##  9 am      8.16e- 4 2.18e-  3 1.35e- 4 4.16e-  2  1.43e-5 1.45e- 6  2.64e-1 3.57e-  1  0.      2.30e-8 7.26e-  1
    ## 10 gear    1.33e- 3 7.68e-  4 3.33e- 4 6.39e-  2  1.01e-6 2.16e- 5  4.18e-1 1.17e-  1  2.30e-8 0.      5.31e-  1
    ## 11 carb    4.34e- 5 5.02e-  4 1.43e- 3 1.80e-  6  4.95e-1 3.58e- 3  4.15e-5 9.88e-  5  7.26e-1 5.31e-1 7.44e-232

Further nice functions are `wilcox_effsize()`, `shapiro_test()` or
`adjust_pvalue()`. Check
(github)\[<a href="https://github.com/kassambara/rstatix" class="uri">https://github.com/kassambara/rstatix</a>\]
for more examples.
