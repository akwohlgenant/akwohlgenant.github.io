---
title: "Clustering of Well Log Data with R"
date: 2021-06-09
tags: [clustering, R, well log data]
header:
 image: "/images/2021-06-09-well_log_clustering_r/matt-artz-layers-unsplash.jpg"
excerpt: ""
mathjax: "true"
---

*Photo by Matt Artz on Unsplash*

About a year ago, I posted about using the k-means clustering algorithm in Python for well log data from oil and gas fields.  Since then, I've been spending some time learning R, so I wanted to post a companion piece showing how to do some of the same things using R.  The general process is very similar, and I will use the same data as the previous post. But I will demonstrate how little code it takes to do some of the same things in R. 

I want to thank Jared Lander and his great book *R for Everyone*.  The basic workflow for k-means clustering in R that I show here is pretty much straight out of his book, with some minor tweaks around displaying the results in a format that is familiar to folks who've worked in the oil patch.

First step, import the packages we will need. For now, all we really
need are the `tidyverse` and `useful` packages.  The `useful` package is, well, useful; it contains some handy tools for clustering and for visualizing the clusters in two dimensions.

``` r
library(tidyverse)
library(useful)
```

Next, I’ll read in the data using the `read_csv()` function from the tidyverse package.  The data can be downloaded from the SEG Github: https://github.com/seg/tutorials-2016/tree/master/1610_Facies_classification.

``` r
logs <- read_csv("kgs_log_data.csv")
head(logs)
```

    ## # A tibble: 6 x 11
    ##   Facies Formation `Well Name` Depth    GR ILD_log10 DeltaPHI PHIND    PE  NM_M
    ##    <dbl> <chr>     <chr>       <dbl> <dbl>     <dbl>    <dbl> <dbl> <dbl> <dbl>
    ## 1      3 A1 SH     SHRIMPLIN   2793   77.4     0.664      9.9  11.9   4.6     1
    ## 2      3 A1 SH     SHRIMPLIN   2794.  78.3     0.661     14.2  12.6   4.1     1
    ## 3      3 A1 SH     SHRIMPLIN   2794   79.0     0.658     14.8  13.0   3.6     1
    ## 4      3 A1 SH     SHRIMPLIN   2794.  86.1     0.655     13.9  13.1   3.5     1
    ## 5      3 A1 SH     SHRIMPLIN   2795   74.6     0.647     13.5  13.3   3.4     1
    ## 6      3 A1 SH     SHRIMPLIN   2796.  74.0     0.636     14    13.4   3.6     1
    ## # ... with 1 more variable: RELPOS <dbl>

Now I will plot the logs for one of the wells. With `ggplot`, I can use
the `geom_path()` geometric object in the `ggplot` call which will
connect the data points in order of increasing depth, and I’ll reverse
the y-scale so that depth is increasing downward. Let’s try this first
for just one well, *SHRIMPLIN*, and one log curve,
*GR*.

``` r
shrimplin <- logs %>% filter(`Well Name` == "SHRIMPLIN") %>% select(Depth, GR, ILD_log10, DeltaPHI, PHIND, PE)

ggplot(shrimplin, aes(x=GR, y=Depth)) + geom_path() + theme_bw() + scale_y_reverse()
```

![](/images/2021-06-09-well_log_clustering_r/unnamed-chunk-3-1.png)<!-- -->

Now I can take advantage of the `facet_wrap()` option in `ggplot` to plot
each curve side-by-side. For this to work, I will need to have each
curve name in the data as a value for a single variable rather than as individual variables in separate columns. This can be accomplished using the `pivot_longer()`
function from `tidyr`, which is included in the `tidyverse` package.

I'll be making the dataframe *longer* (more rows) by losing some columns (the log curve names).  The `facet_wrap()` option can only take a single column argument to facet by, so we will be faceting by "curve_name", the new column we create with `pivot_longer`. I'll also *free* the x-scales so that they will default to the ranges of the individual curves rather than being fixed to a single scale.

``` r
shrimplin_long <- pivot_longer(data=shrimplin, cols=2:6, names_to = "curve_name", values_to = "curve_value")

ggplot(shrimplin_long, aes(x=curve_value, y=Depth)) + 
  geom_path(aes(color=curve_name)) + 
  theme_bw() + 
  scale_y_reverse() + 
  facet_wrap(~ curve_name, nrow=1, scales = "free_x")
```

![](/images/2021-06-09-well_log_clustering_r/unnamed-chunk-4-1.png)<!-- -->

That’s looking pretty good, but let’s reorder the curves in the plot by
setting the factor levels for the *curve* variable. We generally plot a log on the far left that tells us something about lithology, like the gamma-ray (GR).  On the right we usually plot a resistivity curve of some kind (ILD) and a porosity curve of some kind (PHIND). 

Then we can replot the logs for the *SHRIMPLIN* well. I’ll also get rid of the legend since
it’s not really necessary here, and I’ll add a title and a better label
for the x-axis.

``` r
curve_order <- c("GR", "ILD_log10", "PHIND", "DeltaPHI", "PE")
shrimplin_long$curve_name <- factor(shrimplin_long$curve_name, levels=curve_order)

ggplot(shrimplin_long, aes(x=curve_value, y=Depth)) + 
  geom_path(aes(color=curve_name)) + 
  theme_bw() + 
  scale_y_reverse() + 
  theme(legend.position = "none") +
  labs(title="Well: SHRIMPLIN", x="Curve Value") +
  facet_wrap(~ curve_name, nrow=1, scales = "free_x")
```

![](/images/2021-06-09-well_log_clustering_r/unnamed-chunk-5-1.png)<!-- -->

That looks pretty good. Now I’ll move on to the clustering. First I’ll
get rid of null values and select only the numerical variables for clustering.

``` r
colSums(is.na(logs))
```

    ##    Facies Formation Well Name     Depth        GR ILD_log10  DeltaPHI     PHIND 
    ##         0         0         0         0         0         0         0         0 
    ##        PE      NM_M    RELPOS 
    ##       917         0         0

``` r
logs_noNA <- logs %>% filter(!is.na(PE))
logs_train <- logs_noNA %>% select(GR, ILD_log10, PHIND, DeltaPHI, PE)
```

Now I’ll use the `kmeans()` function and specify the number of clusters
to generate. I’ll generate 9 clusters to compare to the 9 facies already defined.

``` rs
set.seed(13)
logs_9clust <- kmeans(x=logs_train, centers=9)
```

Next I can plot the kmeans object I just generated with some help from the `plot.kmeans`
function from the `useful` package. The data will be projected into two principal component dimensions for visualization.

``` r
plot(logs_9clust, data=logs_train)
```

![](/images/2021-06-09-well_log_clustering_r/unnamed-chunk-8-1.png)<!-- -->

Now I can add the cluster labels back to the dataframe that contains the well
name, facies, etc.

``` r
logs_noNA$Cluster <- logs_9clust$cluster
```

I’ll reorder the log curves again, then plot the logs again for the *SHRIMPLIN* well, this time
with an additional track for the clusters I just generated.

``` r
curve_order <- c("GR", "ILD_log10", "PHIND", "DeltaPHI", "PE", "Facies", "Cluster")

logs_noNA %>% filter(`Well Name` == "SHRIMPLIN") %>%
  select(Depth, GR, ILD_log10, PHIND, DeltaPHI, PE, Cluster, Facies) %>%
  pivot_longer(cols=2:8, names_to="curve", values_to="value") %>%
  mutate(curve = factor(curve, levels=curve_order)) %>%
  ggplot(aes(x=value, y=Depth)) + 
  geom_path(aes(color=curve)) + 
  theme_bw() + 
  theme(legend.position = "none") +
  scale_y_reverse() + 
  facet_wrap(~ curve, nrow=1, scales = "free_x") +
  labs(title = "Well: SHRIMPLIN", x = "Curve Value")
```

![](/images/2021-06-09-well_log_clustering_r/unnamed-chunk-10-1.png)<!-- -->

We could do some more optimizing the number of clusters here, but I’ll save that
for a later update.
