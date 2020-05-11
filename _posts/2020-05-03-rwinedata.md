---
title: "Wine Data: Exploratory Analysis in R"
date: 2020-05-03
tags: [k-means, clustering, unsupervised learning, well log]
header:
 image: "/images/elle_hughes_wine_pic.jpg"
excerpt: "Exploratory Data Analysis in R"
mathjax: "true"
---

### Introduction

Like . There are hundreds of varieties
of wine produced in countries ranging from Armenia to Uruguay. A number
of websites and periodicals compile information about wines in order to
help consumers choose wines and learn more about them. The magazine
*Wine Enthusiast* was founded in 1988 by Adam and Sybil Strum, and now
has a circulation of more than 250,000.

One of the most popular features of the Wine Enthusiast magazine and
website (www.winemag.com) are the wine reviews, which contain important
information about each wine, such as its country and region of origin,
the vintner or winery that produced it, the variety (e.g. Pinot Noir),
as well as the name of the reviewer and other descriptive information.
In addition, each review contains a rating or “points” awarded to the
wine based on a variety of categories. The ratings are purported to be
on a 100-point scale, although in reality it is a 20-point scale from 80
to 100 points. Here is a description of the corresponding qualities to
each points range:

Wine Enthusiast’s 100-point wine-scoring scale:

    98–100 – Classic
    94–97 – Superb
    90–93 – Excellent
    87–89 – Very good
    83–86 – Good
    80–82 – Acceptable

(<https://www.wine-searcher.com/critics-17-wine+enthusiast>)

Another useful feature of each review is a text description of the wine
under review. These descriptions, or “tasting notes”, are often full of
descriptive adjectives and creative imagery to describe the wine’s
appearance, aroma, flavor, etc. Here is an example description, chosen
at random from the dataset:

*“Fragrant and fresh, this Grillo opens with alluring scents of acacia
flower, beeswax and white stone fruit. The succulent palate offers
creamy white peach, juicy nectarine, almond and mineral framed in tangy
acidity. A note of chopped herb closes the lingering finish.”*

### Research Questions:

The aim of this analysis was to investigate a dataset consisting of
nearly 130,000 wine reviews that were scraped from the Wine Enthusiast
website in 2017 and posted on the website *kaggle* (www.kaggle.com). A
number of questions are investigated, including:

  - What is the distribution of country of origin? Which countries have
    the most reviewed wines?

  - What is the distribution of wine varieties? Which varieties are the
    most reviewed?

  - What does the price distribution look like? Is it a normal
    distribution, or is it more heavily skewed toward lower priced
    wines?

  - What about the distribution of ratings or “points”? How are they
    distributed?

  - Is there any relationship between price of wine and its
    corresponding rating in points? Are higher priced wines rated higher
    than cheap wines?

  - What are the most commonly used words in the wine descriptions? How
    can those be visualized?

These questions will be investigated by using a number of different
statistical and graphical techniques in R.

### Data

As mentioned previously, the dataset for this research consists of a
comma separated file downloaded from the *kaggle* website, and
comprising nearly 130,000 wine reviews that were scraped from the Wine
Enthusiast website (www.winemag.com) in 2017. The data contain 14
variables or columns, that include the following information about each
wine:

  - Unique identifier (numerical, column name “X”)
  - Country of origin
  - Province (if country is US, this is the state of origin)
  - Region (there are two region variables, *region\_1* and *region\_2*)
  - Designation (this appears to be some kind of secondary descriptor)
  - Title (generally the name of producer, variety and vintage)
  - Description (text description or “tasting notes” for each wine)
  - Variety (e.g. Pinot Noir or Chardonnay)
  - Winery
  - Price (US Dollars)
  - Points (80-100 points)
  - Taster name
  - Taster Twitter handle

Here is a link to the website from which the data was obtained:

<https://www.kaggle.com/zynicide/wine-reviews>

### Methods

The dataset is investigated using a number of different descriptive,
statistical, and graphical methods available in R and it’s associated
packages, including *dplyr*, *ggplot2*, *tm*, and others. Methods used
include:

  - Grouping by country and variety and summarizing counts.
  - Plotting histograms of wine price and points.
  - Binning wines into price range bins (e.g. $10-$20, $20-$30, etc.)
    and visualizing price distribution
  - Visualizing wine ratings as a function of price bins; addressing
    question of whether there is a relationship between price and rating
    points.
  - Using word frequency counting and word cloud to visualize the
    frequency of words used in wine descriptions.

### Analysis

First, the necessary packages are included for the analysis.

``` r
library(dplyr) # for a variety of analysis functions
library(ggplot2) # for plotting
# the packages below are included to aid in construction of the word cloud, among other functions
library(tm) 
library(SnowballC)
library(wordcloud)
library(RColorBrewer)
library(tidytext)
library(stringr)
library(tidyverse)
library(tmap)
```

Next, the dataset file is loaded into R and the dimensions inspected:

``` r
wine <- read.csv("winemag_data_130k_v2.csv")
dim(wine)
```

    ## [1] 129971     14

The dataset contains 129,971 observations, and 14 variables. Next the
variable names are
    listed:

``` r
colnames(wine)
```

    ##  [1] "X"                     "country"               "description"          
    ##  [4] "designation"           "points"                "price"                
    ##  [7] "province"              "region_1"              "region_2"             
    ## [10] "taster_name"           "taster_twitter_handle" "title"                
    ## [13] "variety"               "winery"

Let’s see what a “description” looks like for a randomly chosen wine in
the
    dataset:

``` r
wine$description[846]
```

    ## [1] Fragrant and fresh, this Grillo opens with alluring scents of acacia flower, beeswax and white stone fruit. The succulent palate offers creamy white peach, juicy nectarine, almond and mineral framed in tangy acidity. A note of chopped herb closes the lingering finish.
    ## 119955 Levels: "Chremisa," the ancient name of Krems, is commemorated in this wine that comes from Krems vineyards. It has tight, tangy apple-driven acidity, with a bright, light, citrusy character. Not for aging. ...

This description is full of interesting adjectives and imagery meant to
elicit in the reader a sense of the appearance, aroma, and flavor of the
wine. Later, the words used will be more thoroughly investigated and
visualized.

Now the countries of origin will be investigated. What are the unique
countries in this dataset?

``` r
countries <- unique(wine$country)
length(countries)
```

    ## [1] 44

So 44 unique countries are represented in the dataset. Let’s get a
listing of these countries, and summarise them by their respective
counts in the dataset:

``` r
wine_country <- wine %>%
  group_by(country) %>%
  summarise(Count=n()) %>%
  arrange(-Count)
wine_country
```

    ## # A tibble: 44 x 2
    ##    country   Count
    ##    <fct>     <int>
    ##  1 US        54504
    ##  2 France    22093
    ##  3 Italy     19540
    ##  4 Spain      6645
    ##  5 Portugal   5691
    ##  6 Chile      4472
    ##  7 Argentina  3800
    ##  8 Austria    3345
    ##  9 Australia  2329
    ## 10 Germany    2165
    ## # ... with 34 more rows

``` r
(54504+22093+19540)/129971
```

    ## [1] 0.7396804

The United States, France, and Italy are the countries of origin of the
bulk of the wines reviewed, together making up approximately 74% of the
total wines reviewed. Below is a visualization of the distribution of
wines by
country:

``` r
ggplot(data=wine_country, aes(x=reorder(country, -Count), y=Count)) + geom_bar(stat='identity', fill='blue') + theme(axis.text.x=element_text(angle=90, hjust=1))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-8-1.png)<!-- -->

As we can see, the top three countries make up the bulk of the wines,
and the counts tail off rapidly in the rest of the countries
represented, such that many of the countries are difficult to see on the
plot because their numbers are so low in comparison.

Next, a similar analysis is performed with wine varieties:

``` r
varieties <- unique(wine$variety)
length(varieties)
```

    ## [1] 708

The dataset contains 708 unique varieties of wine. In order to visualize
the varieties that make up the dataset, it is easiest to filter out wine
varieties with the fewest reviews. In this case, wines with fewer than
500 reviews will be removed before plotting the visualization.

``` r
wine_varieties <- wine %>%
  group_by(variety) %>%
  summarise (count=n()) %>%
  filter(count>500) %>%
  arrange(-count)
ggplot(data=wine_varieties, aes(x=reorder(variety, -count), y=count)) + geom_bar(stat='identity', fill='red') + theme(axis.text.x=element_text(angle=90, hjust=1))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-10-1.png)<!-- -->

As we can see above, the four most reviewed wines are Pinot Noir,
Chardonnay, Cabernet Sauvignon, and Red Blend. This distribution is not
nearly as concentrated in a few varieties as was the case with the
countries of origin.

Now let’s look into the prices of wines, and the ratings or “points”
awarded them. First we can make a histogram of the prices to see the
distribution:

``` r
hist(wine$price)
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-11-1.png)<!-- -->

This is not a very informative plot, because the right tail is so long
and consists of very few observations for each bin. So let’s limit the
plot to wines with prices less than $500.

``` r
mod_price <- wine %>%
  filter(price < 500)
hist(mod_price$price)
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-12-1.png)<!-- -->

Here it apparent that the prices are highly concentrated in the low
range, with a long tail extending out into the higher prices
(right-skewed). We will do more thorough investigation of this
observation.

``` r
summary(wine$price)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##    4.00   17.00   25.00   35.36   42.00 3300.00    8996

From the statistical summary above, we can see that the range in price
is very large, from $4 to $3,300, with a median value of $25, which
reinforces what was intuitive from the histogram, that the prices are
very concentrated in the low end of the range.

Now a histogram of the wine ratings represented by the variable
*points*:

``` r
hist(wine$points)
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-14-1.png)<!-- -->

This distribution appears to be nearly normal, with a peak around 87-88
points. Below is a statistical summary:

``` r
summary(wine$points)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   80.00   86.00   88.00   88.45   91.00  100.00

In this near-normal distribution, the mean and median are very close to
one another at 88 and 88.45 points, respectively.

We will now do a more thorough investigation of wine prices and compare
them with the wine ratings to see if a relationship exists between the
two variables that could be used to predict one from the other. To do
this, we will divide the wines into buckets or bins based on their price
in $10 increments ($0-$10, $10-$20, etc.) Then we can see the
distribution of wine counts by these bin designations.

First we create a vector of the values to be the “breaks” between bins,
and labels or “tags” to attach to each of these breaks.

``` r
breaks1 <- c(0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 100000)
tags <- c('$0-10', '$10-20', '$20-30', '$30-40', '$40-50', '$50-60', '$60-70', '$70-80', '$80-90', '$90-100', '>$100')
```

Then we can use the *cut* function to divide the price variable into
bins separated by the breaks we just created, and summarize the counts
for each
bin:

``` r
price_bins <- cut(wine$price, breaks=breaks1, include.lowest=TRUE, right=FALSE, labels=tags)
summary(price_bins)
```

    ##   $0-10  $10-20  $20-30  $30-40  $40-50  $50-60  $60-70  $70-80  $80-90 $90-100 
    ##    2841   36560   29103   17299   12064    7593    5111    3163    1898    1392 
    ##   >$100    NA's 
    ##    3951    8996

Above it is apparent that the largest bin of prices is the $10-$20 bin,
which containis 36,560 observations. We can visualize this distribution
better with a bar
    graph:

``` r
ggplot(data=as_tibble(price_bins), mapping=aes(x=value)) + geom_bar()
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-18-1.png)<!-- -->

Next, we can use these price bins as a way to compare with wine points
and investigate the question of whether a relationship exists between
price and points. To do this we can create a new column called
price\_bin:

``` r
wine_pp <- wine %>% select(price, points) %>% filter(!is.na(price))
wb_group <- as_tibble(wine_pp) %>%
  mutate(price_bin = case_when(
    price < 10 ~ tags[1],
    price >= 10 & price < 20 ~ tags[2],
    price >= 20 & price < 30 ~ tags[3],
    price >= 30 & price < 40 ~ tags[4],
    price >= 40 & price < 50 ~ tags[5],
    price >= 50 & price < 60 ~ tags[6],
    price >= 60 & price < 70 ~ tags[7],
    price >= 70 & price < 80 ~ tags[8],
    price >= 80 & price < 90 ~ tags[9],
    price >= 90 & price < 100 ~ tags[10],
    price >= 100 ~ tags[11]
  ))
summary(wb_group)
```

    ##      price             points        price_bin        
    ##  Min.   :   4.00   Min.   : 80.00   Length:120975     
    ##  1st Qu.:  17.00   1st Qu.: 86.00   Class :character  
    ##  Median :  25.00   Median : 88.00   Mode  :character  
    ##  Mean   :  35.36   Mean   : 88.42                     
    ##  3rd Qu.:  42.00   3rd Qu.: 91.00                     
    ##  Max.   :3300.00   Max.   :100.00

Now we can visualize the wine rating (points) as a functio of price by
plotting box plots of points for each price
bin:

``` r
ggplot(data=wb_group, mapping = aes(x=price_bin, y=points)) + geom_jitter(color='red', alpha=0.1) + 
  geom_boxplot(fill='bisque', color='black', alpha=0.3) + theme_minimal()
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-20-1.png)<!-- -->

We can see from this visualization not only the spread in the underlying
price data for each bin, but also the progression of the median points
for each price bin. It is apparent from this plot that overall there is
an increase in rating points associated with an increase in price. This
effect is more dramatic in the lower price bins from zero to $50, and
the effect is flattened with higher prices, such that the median rating
for a $90-$100 wine is not significantly higher than that for a $70-$80
wine.

This raises an interesting causal question: are the more expensive wines
rated higher because they are better? Or are the tasters biased by their
own knowledge of the wine’s price, and therefore more likely to rate the
wine higher if it is more expensive? This question can not be answered
from these data, but the relationship could be further investigated
using a dataset of blind tastings where the tasters are ignorant of the
price when assigning the ratings.

The final question will now be investigated: what words are most
commonly used to describe the wines in the description variable? To do
this, word frequencies will be counted, and a word cloud will be
generated to visualize the results.

First a *corpus* is created, which is a collection of natural language
text. In this case, it is created from the *description* column of the
dataset.

``` r
wine_corpus = Corpus(VectorSource(wine$description))
```

Next the corpus is edited to remove upper case letter, numbers,
punctuation, and common words that don’t yield any insights
(e.g. “the”).

``` r
wine_corpus = tm_map(wine_corpus, content_transformer(tolower))
```

    ## Warning in tm_map.SimpleCorpus(wine_corpus, content_transformer(tolower)):
    ## transformation drops documents

``` r
wine_corpus = tm_map(wine_corpus, removeNumbers)
```

    ## Warning in tm_map.SimpleCorpus(wine_corpus, removeNumbers): transformation drops
    ## documents

``` r
wine_corpus = tm_map(wine_corpus, removePunctuation)
```

    ## Warning in tm_map.SimpleCorpus(wine_corpus, removePunctuation): transformation
    ## drops documents

``` r
wine_corpus = tm_map(wine_corpus, removeWords, c("the", "and", "wine", "with", "this", "flavors", "its"))
```

    ## Warning in tm_map.SimpleCorpus(wine_corpus, removeWords, c("the", "and", :
    ## transformation drops documents

``` r
wine_corpus =  tm_map(wine_corpus, stripWhitespace)
```

    ## Warning in tm_map.SimpleCorpus(wine_corpus, stripWhitespace): transformation
    ## drops documents

``` r
inspect(wine_corpus[846])
```

    ## <<SimpleCorpus>>
    ## Metadata:  corpus specific: 1, document level (indexed): 0
    ## Content:  documents: 1
    ## 
    ## [1] fragrant fresh grillo opens alluring scents of acacia flower beeswax white stone fruit succulent palate offers creamy white peach juicy nectarine almond mineral framed in tangy acidity a note of chopped herb closes lingering finish

An inspection above the description for element number 846 in the corpus
shows that the unnecessary features were removed.

``` r
wine_dtm <- DocumentTermMatrix(wine_corpus)
wine_dtm = removeSparseTerms(wine_dtm, 0.99)
```

``` r
freq = data.frame(sort(colSums(as.matrix(wine_dtm)), decreasing=TRUE))
wordcloud(rownames(freq), freq[,1], max.words=50, colors=brewer.pal(1, "Dark2"))
```

    ## Warning in brewer.pal(1, "Dark2"): minimal value for n is 3, returning requested palette with 3 different levels

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-25-1.png)<!-- -->

The word cloud above is limited to 50 words, but gives a good idea of
the kinds of words that are frequently used in the wine descriptions in
the dataset. Additional work coud be performed here to remove additional
words that don’t yield much insight, such as “has”, “that”, “drink”,
“more”, etc.

### Conclusions

The results of this analysis effectively answered the research questions
posed:

  - The countries of origin were highly concentrated in a few countries,
    with about 74% of the reviews coming from the United States, France
    and Italy.

  - The wine varieties were somewhat more evenly distributed than the
    countries, with the most common wines being Pinot Noir, Chardonnay,
    and Cabernet Sauvignon.

  - The wine prices were highly concentrated in the $10 to $50 dollar
    range, with a long tail exending up in price to a maximum price of
    $3,300.

  - The wine ratings or “points” were near normally distributed, with a
    median value of 88 points.

  - There was a relationship noted between price and points, such that
    an increase in price is associated with an increase in awarded
    points. This effect was more dramatic in the lower price bins
    (0-$50) than in the higher price range.

  - And finally, the most frequently used words in the wine desription
    column were calculated and visualized using a word cloud.
