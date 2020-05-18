---
title: "General Social Survey Data: Statistical Inference using R"
date: 2020-05-03
tags: [statistical inference, hypothesis testing, gss]
header:
 image: "/images/aditya-romansa-5zp0jym2w9M-unsplash.jpg"
excerpt: "GSS Data: Statistical Inference"
mathjax: "true"
---

### Abstract

This project investigates the General Social Survey (GSS) dataset from the University of Chicago. I will use the statistical analysis capabilities of R to investigate the question of whether there is a significant racial difference among US women in the average age at which a woman’s first child is born. Along the way, I will do some exploratory data analysis, and see what special packages and functions R has for statistical inference.

### Load packages

``` r
library(ggplot2)
library(dplyr)
library(statsr)
```

### Load data

``` r
load("gss_data.Rdata")
```

## Introduction

This analysis will investigate a subset of the data collected for the
General Social Survey, a project of the National Opinion Research Center
(NORC) at the University of Chicago.

According to the NORC:

*“The GSS aims to gather data on contemporary American society in order
to monitor and explain trends and constants in attitudes, behaviors, and
attributes; to examine the structure and functioning of society in
general as well as the role played by relevant subgroups; to compare the
United States to other societies in order to place American society in
comparative perspective and develop cross-national models of human
society; and to make high-quality data easily accessible to scholars,
students, policy makers, and others, with minimal cost and waiting.”*

The focus of my analysis will be racial differences in the average ages
of women at the time of the birth of their first child. The question of
interest is, *is there a statistically significant difference between
white and black women in the average age at which they have their first
child?*

This question is interesting because it could shed light on cultural and
economic factors that could affect when in their lives women in the
United States transition to being mothers.

-----

## Part 1: Data

``` r
dim(gss)
```

    ## [1] 57061   114

This dataset, the General Social Survey, contains 57,061 observations
(rows) and 114 variables (columns).

Let’s take a look at the variable
    names:

``` r
colnames(gss)
```

    ##   [1] "caseid"   "year"     "age"      "sex"      "race"     "hispanic"
    ##   [7] "uscitzn"  "educ"     "paeduc"   "maeduc"   "speduc"   "degree"  
    ##  [13] "vetyears" "sei"      "wrkstat"  "wrkslf"   "marital"  "spwrksta"
    ##  [19] "sibs"     "childs"   "agekdbrn" "incom16"  "born"     "parborn" 
    ##  [25] "granborn" "income06" "coninc"   "region"   "partyid"  "polviews"
    ##  [31] "relig"    "attend"   "natspac"  "natenvir" "natheal"  "natcity" 
    ##  [37] "natcrime" "natdrug"  "nateduc"  "natrace"  "natarms"  "nataid"  
    ##  [43] "natfare"  "natroad"  "natsoc"   "natmass"  "natpark"  "confinan"
    ##  [49] "conbus"   "conclerg" "coneduc"  "confed"   "conlabor" "conpress"
    ##  [55] "conmedic" "contv"    "conjudge" "consci"   "conlegis" "conarmy" 
    ##  [61] "joblose"  "jobfind"  "satjob"   "richwork" "jobinc"   "jobsec"  
    ##  [67] "jobhour"  "jobpromo" "jobmeans" "class"    "rank"     "satfin"  
    ##  [73] "finalter" "finrela"  "unemp"    "govaid"   "getaid"   "union"   
    ##  [79] "getahead" "parsol"   "kidssol"  "abdefect" "abnomore" "abhlth"  
    ##  [85] "abpoor"   "abrape"   "absingle" "abany"    "pillok"   "sexeduc" 
    ##  [91] "divlaw"   "premarsx" "teensex"  "xmarsex"  "homosex"  "suicide1"
    ##  [97] "suicide2" "suicide3" "suicide4" "fear"     "owngun"   "pistol"  
    ## [103] "shotgun"  "rifle"    "news"     "tvhours"  "racdif1"  "racdif2" 
    ## [109] "racdif3"  "racdif4"  "helppoor" "helpnot"  "helpsick" "helpblk"

These variable names can be somewhat difficult to comprehend without any
context.

The GSS Codebook, available for download on the website of the National
Opinion Research Center (NORC) at the University of Chicago, provides
in-depth descriptions of the data, how it was acquired, and the meanings
of each category or variable in the dataset. According to the GSS
Codebook, each survey from 1972 to 2004 was an independently drawn
sample of English-speaking persons 18 years of age or over, living in
non-institutional arrangements within the United States, and from 2006
on the survey has also included Spanish-speaking subjects.

The GSS Codebook contains a very verbose section on sampling methodology
that will not be repeated here. However, it is apparent that the sample
was generally a random sample of adults living in the United States.
Therefore it is generalizable to the total population of adults in the
United States, although perhaps with the caveat that there may be bias
in the dataset toward English-speaking respondents; this appears to be
partially corrected for by the inclusioin of Spanish-speaking
respondents starting in 2006. However, Spanish is not the only other
language besides English spoken in the United States, so it is possible
that persons in the United States speaking a language other than English
or Spanish could be underrepresented in this survey, and of course
Spanish-speaking people could be underrepresented in years prior to
2006.

Also it is important to note that these data represent answers to
questions asked of respondents, so they are observational data, not
experimental data. Therefore, there can be no inference of causality;
correlation can be noted, but causation cannot be inferred from such
correlations. Also, it should be noted that since the data are
self-reported, there may be additional bias in the data regarding
subjects the respondents may have been uncomfortable or unwilling to
address in a truthful way.

-----

## Part 2: Research question

The research question of interest: is there a difference in the average
age of black and white women in the United States at time of the birth
of their first child. This is of interest because there may be societal,
cultural and economic factors at play that influence the two races
differently in the United States in terms of when women of different
races transition to having children, whether planned or not.

Hypothesis for testing if the average age of women at the time of the
birth of their first child is different for white and black women:

$$
H_0: \mu_{white} = \mu_{black}\\
H_A: \mu_{white} \ne \mu_{black}
$$

The null hypothesis is there is no difference in the average age of
black and white women at the time of the birth of their first child. The
alternative hypothesis is that there is a difference between the average
age of black and white women at the time of the birth of their first
child.

-----

## Part 3: Exploratory data analysis

Let’s see what years the data were acquired
    in:

``` r
unique(gss$year)
```

    ##  [1] 1972 1973 1974 1975 1976 1977 1978 1980 1982 1983 1984 1985 1986 1987 1988
    ## [16] 1989 1990 1991 1993 1994 1996 1998 2000 2002 2004 2006 2008 2010 2012

This dataset appears to only go through 2012. The GSS survey has
continued to acquire data since then, repeating the survey in 2014, 2016
and 2018; those data are not in this dataset. Let’s see the breakdown of
data counts by year:

``` r
gss_count_yr <- gss %>%
  group_by(year) %>%
  summarise(count=n())
gss_count_yr
```

    ## # A tibble: 29 x 2
    ##     year count
    ##    <int> <int>
    ##  1  1972  1613
    ##  2  1973  1504
    ##  3  1974  1484
    ##  4  1975  1490
    ##  5  1976  1499
    ##  6  1977  1530
    ##  7  1978  1532
    ##  8  1980  1468
    ##  9  1982  1860
    ## 10  1983  1599
    ## # ... with 19 more rows

These are easier to visualize in a bar
plot:

``` r
ggplot(gss_count_yr, aes(x=year, y=count)) + geom_bar(stat='identity', fill='red')
```

![png](/images/2020_05_08_gssdata_images/unnamed-chunk-5-1.png)<!-- -->

It is apparent from this plot that the data were collected either every
year or every other year, but the data counts have increased over time,
with the last 3 collections being pretty consistent at approximately
2000 observations.

Let’s take a look at the breakdown of men versus women in the study:

``` r
gss %>%
  group_by(sex) %>%
  summarise(count=n())
```

    ## # A tibble: 2 x 2
    ##   sex    count
    ##   <fct>  <int>
    ## 1 Male   25146
    ## 2 Female 31915

So the dataset contains 25,146 observations of men, and 31,915
observations of women.

Now let’s take a look at the breakdown of the data by race:

``` r
gss %>%
  group_by(race) %>%
  summarise(count=n())
```

    ## # A tibble: 3 x 2
    ##   race  count
    ##   <fct> <int>
    ## 1 White 46350
    ## 2 Black  7926
    ## 3 Other  2785

The dataset contains 46,350 observations of persons identifying as the
race “White”, 7,926 persons identifying as the race “Black”, and the
remaining 2,785 respondents identifying as races categorized as “Other”.

This analysis will be concerned with investigating potential differences
between white and black, female respondents. Also, rather than utilizing
all of the data since 1972, this analysis will focus just on the data
collected since 2000. Also, missing values in the *agekdbrn* variable
will be removed; this is the variable of interest since it is the
reported age of each respondent at which her first child was born.

``` r
gss_gt1999_f <- gss %>%
  filter(race!="Other", year>1999, sex=="Female", !is.na(agekdbrn))
dim(gss_gt1999_f)
```

    ## [1] 6815  114

This has reduced the number of observations available for analysis to
6,815.

Let’s take a look at the structure of the response variable of interest,
*agekdbrn*:

``` r
str(gss_gt1999_f$agekdbrn)
```

    ##  int [1:6815] 20 19 29 21 22 25 21 29 22 19 ...

And the descriptive categorical variable of interest:

``` r
str(gss_gt1999_f$race)
```

    ##  Factor w/ 3 levels "White","Black",..: 1 1 1 1 1 1 2 1 1 1 ...

This variable is a factor with 3 levels; we only want it to have two
levels so that we can use the *inference* function to compare two means.
So we will need to refactor this variable so that it only has two
levels:

``` r
gss_gt1999_f$race <- factor(gss_gt1999_f$race)
str(gss_gt1999_f$race)
```

    ##  Factor w/ 2 levels "White","Black": 1 1 1 1 1 1 2 1 1 1 ...

Now it only has two levels, so it will work in the inference phase of
the analysis.

Let’s take a quick look at the means and standard deviations of the two
races:

``` r
gss_gt1999_f %>%
  group_by(race) %>%
  summarise(mean=mean(agekdbrn), stddev=sd(agekdbrn))
```

    ## # A tibble: 2 x 3
    ##   race   mean stddev
    ##   <fct> <dbl>  <dbl>
    ## 1 White  23.2   5.02
    ## 2 Black  20.7   4.64

We can see that in the data subset created previously (years \> 1999,
female respondents, race = white or black) that the mean age of first
child born for white respondents is 23.23 years with a standard
deviation of 5.02 years, and the mean age of first child born for black
respondents is 20.73 years with a standard deviation of 4.64 years.

To see additional descriptive statistics by race, I will split into two
datasets, white and black.

``` r
gss_gt1999_f_w <- gss_gt1999_f%>%
  filter(race=="White")
gss_gt1999_f_b <- gss_gt1999_f%>%
  filter(race=="Black")
```

For white respondents:

``` r
summary(gss_gt1999_f_w$agekdbrn)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   11.00   20.00   22.00   23.23   26.00   45.00

And for black respondents:

``` r
summary(gss_gt1999_f_b$agekdbrn)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   12.00   18.00   20.00   20.73   23.00   42.00

Let’s visualize the data, first with a boxplot for the two
races:

``` r
boxplot(agekdbrn ~ race, data=gss_gt1999_f)
```

![png](/images/2020_05_08_gssdata_images/unnamed-chunk-16-1.png)<!-- -->

And now let’s see the histograms for both races:

``` r
ggplot(gss_gt1999_f, aes(x=agekdbrn, color=race)) +
  geom_histogram(fill="gray", alpha=0.5, position="identity")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![png]/images/2020_05_08_gssdata_images/unnamed-chunk-17-1.png)<!-- -->

This plot really illustrates a marked difference between the two races
in the sample. The white respondents histogram reaches a peak at around
23 years, similar to the calculated mean for that group. The black
respondents histogram actually reaches a peak at around 17 years,
significantly lower than the calculated mean of 20.73 years.

It is noted that both histograms are slighty right-skewed. The sample
sizes are both quite large though, so we should be able to assume that
the sampling distributions for both race samples will be normal, and
therefore we can use the t-distribution for inference looking at the
difference between two sample means.

Let’s take a look at a normal probability plot, first for black
respondents:

``` r
qqnorm(gss_gt1999_f_b$agekdbrn, main="Normal Q-Q Plot: Black Respondents")
```

![png](/images/2020_05_08_gssdata_images/unnamed-chunk-18-1.png)<!-- -->

Then for white
respondents:

``` r
qqnorm(gss_gt1999_f_w$agekdbrn, main="Normal Q-Q Plot: White Respondents")
```

![png](/images/2020_05_08_gssdata_images/unnamed-chunk-19-1.png)<!-- -->

Neither variable split plots on a straight line, indicating that the
data are skewed (which we already knew\!)

Let’s move on to the inference part.

-----

## Part 4: Inference

We will be comparing the means of the variable *agekdbrn* for white and
black women in the GSS dataset.

The t-distribution can be used for inference when working with the
standardized diﬀerence of two means if (1) each sample meets the
conditions for using the tdistribution and (2) the samples are
independent.

Based on the description of the data collection in the GSS Codebook, it
is apparent that the observations in the dataset are randomly selected
and independent. It is also obvious that they make up less than 10% of
the overall population and there are more than the minimum required
number.

The data are somewhat right-skewed, but generally appear to be normally
distributed, and so the t-distribution can be used to compare the means.
This will use a theoretical approach, not a simulation, since there are
sufficient number of observations in the data. This will be a two-sided
test since we are looking for differences between the two races.

The hypothesis
test:

``` r
inference(y = agekdbrn, x = race, data = gss_gt1999_f, statistic = "mean", type = "ht", null = 0, 
          alternative = "twosided", method = "theoretical")
```

    ## Response variable: numerical
    ## Explanatory variable: categorical (2 levels) 
    ## n_White = 5552, y_bar_White = 23.2275, s_White = 5.0155
    ## n_Black = 1263, y_bar_Black = 20.7324, s_Black = 4.6382
    ## H0: mu_White =  mu_Black
    ## HA: mu_White != mu_Black
    ## t = 16.9911, df = 1262
    ## p_value = < 0.0001

![png](/images/2020_05_08_gssdata_images/unnamed-chunk-20-1.png)<!-- -->

Recall that the null hypothesis is that there is no difference between
the mean age of white women and black women when their first children
are born, such that any difference seen is due to chance.

The alternative hypothesis is that there IS a difference in the mean age
of white and black women when their first children are born.

In this case, the t-statistic was computed to be t = 16.99 with 1,262
degrees of freedom, and a p-value calculated to be \< 0.0001, very small
indeed. So it appears there is sufficient evidence to reject the null
hypothesis in favor of the alternative hypothesis.

Next we compute the confidence interval for the difference between the
means:

``` r
inference(y = agekdbrn, x = race, data = gss_gt1999_f, statistic = "mean", type = "ci", null = 0, 
          alternative = "twosided", method = "theoretical")
```

    ## Response variable: numerical, Explanatory variable: categorical (2 levels)
    ## n_White = 5552, y_bar_White = 23.2275, s_White = 5.0155
    ## n_Black = 1263, y_bar_Black = 20.7324, s_Black = 4.6382
    ## 95% CI (White - Black): (2.207 , 2.7832)

![png](/images/2020_05_08_gssdata_images/unnamed-chunk-21-1.png)<!-- -->

95% CI (White - Black): (2.207 , 2.7832)

Based on this analysis, we can be 95% confident that white women were on
average 2.21 to 2.78 years older than black women in the United States
at the time of the birth of their first child. The observed difference
between the means does fall within this confidence interval.
