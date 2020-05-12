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

I like wine. Lots of people like wine. I tend to drink the same wines
over and over again, from an embarrassingly small list of varieties and
regions. But there are hundreds of varieties of wine produced in dozens
of regions in countries ranging from Armenia to Uruguay, with a dizzying
array of naming conventions that can be downright confusing. And I, like
many people, am interested in learning more about wines, maybe in the
hope of finding a cheap, underrated bargain at my local liquor store.
Luckily, there are myriad magazines and websites devoted entirely to the
discussion, reviewing, and rating of wines. One of the most popular of
such magazines and websites is *Wine Enthusiast*.

*Wine Enthusiast* was founded in 1988 by Adam and Sybil Strum, and now
has a circulation of more than 250,000. One of the most popular features
of the Wine Enthusiast magazine and website (www.winemag.com) are the
wine reviews, which contain a wealth of information about each wine,
both useful and not so useful. Information found in a given review
includes: the country and region of origin, the name of the vintner or
winery, the variety (e.g. Pinot Noir), the price (arguably the most
important feature), the name of the reviewer, and even the reviewer’s
*Twitter* handle. In addition to descriptive information, each review
contains a rating or “points” awarded to the wine based on a variety of
categories. The ratings are purported to be on a 100-point scale,
although in reality it is just a 20-point scale from 80 to 100 points.

Wine Enthusiast’s 100-point wine-scoring scale:

    98–100 – Classic
    94–97 – Superb
    90–93 – Excellent
    87–89 – Very good
    83–86 – Good
    80–82 – Acceptable

(<https://www.wine-searcher.com/critics-17-wine+enthusiast>)

Another feature of each review is a text description or “tasting notes”;
these are often full of flowery adjectives and borderline ridiculous
imagery to intended to evoke the actual experience of tasting a wine,
with comments about the wine’s appearance, aroma, flavor, etc. Here is
an example description, chosen at random from the dataset:

*“Fragrant and fresh, this Grillo opens with alluring scents of acacia
flower, beeswax and white stone fruit. The succulent palate offers
creamy white peach, juicy nectarine, almond and mineral framed in tangy
acidity. A note of chopped herb closes the lingering finish.”*

How many people know what acacia flowers and beeswax smell like?

### Research Questions:

The aim of this analysis was to investigate a dataset consisting of
nearly 130,000 wine reviews that were originally scraped (not by me)
from the *Wine Enthusiast* website in 2017 and posted on *Kaggle*
(www.kaggle.com). A number of questions are investigated, including:

  - Which countries have the most reviewed wines?

  - Which varieties are the most reviewed?

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
statistical and graphical techniques using R.

### Data

As mentioned previously, the dataset for this research consists of a
comma separated file downloaded from *Kaggle*, and comprising nearly
130,000 wine reviews that were scraped from the *Wine Enthusiast*
website (www.winemag.com) in 2017. The data contain 14 variables:

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

Here is a link to download the data from *Kaggle*:

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
library(plotly)
```

``` r
#install.packages("qdap")
#install.packages("qdap", INSTALL_opts = "--no-multiarch")
library(qdap)
```

Next, the dataset file is loaded into R and the dimensions inspected:

``` r
wine <- read.csv("winemag_data_130k_v2.csv", encoding="UTF-8" )
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
wine$description[sample(1:129971, 1)]
```

    ## [1] Made from grapes sourced from Domaines Devillard's 12 acres of premier cru Mercurey vineyards, this rich, weighty wine has ripe red fruits that are layered with new wood and tannins. A structured effort that's meant to age for five years and more.
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

The United States, France, and Italy are the countries of origin of the
bulk of the wines reviewed. A substantial number of countries have very
few wines reviewed. So I am going to filter out the countries with few
reviews before plotting.

``` r
wine_country <- wine_country %>%
  filter(Count>50)
```

Below is a visualization of the distribution of wines by country after
filtering.

``` r
ggplot(data=wine_country, aes(x=reorder(country, -Count), y=Count)) + geom_bar(stat='identity', fill='dodgerblue') + labs(title="Wine Review Counts by Country",x="Country", y = "Count") + theme(axis.text.x=element_text(angle=90, hjust=1), plot.title = element_text(hjust = 0.5))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-9-1.png)<!-- -->

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
```

``` r
ggplot(data=wine_varieties, aes(x=reorder(variety, -count), y=count)) + geom_bar(stat='identity', fill='dodgerblue') + labs(title="Wine Review Counts by Variety",x="Variety", y = "Count") + theme(axis.text.x=element_text(angle=90, hjust=1), plot.title = element_text(hjust = 0.5))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-12-1.png)<!-- -->

As we can see above, the four most reviewed wines are Pinot Noir,
Chardonnay, Cabernet Sauvignon, and Red Blend. This distribution is not
nearly as concentrated in a few varieties as was the case with the
countries of origin.

Now let’s look into the prices of wines, and the ratings or “points”
awarded them. First, let’s filter out the very high priced wines that
are out of virtually everyone’s price range (over $500 a bottle).

``` r
mod_price <- wine %>%
  filter(price < 500)
```

``` r
ggplot(data=mod_price, aes(mod_price$price)) + geom_histogram(binwidth = 10, color='black', fill='dodgerblue') + labs(title="Histogram of Wine Prices",x="Price, USD", y = "Count") + theme(plot.title = element_text(hjust = 0.5))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-14-1.png)<!-- -->

Here it apparent that the prices are highly concentrated in the low
range, with a long tail extending out into the higher prices
(right-skewed). We will do more thorough investigation of this
observation.

``` r
summary(mod_price$price)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    4.00   17.00   25.00   34.69   42.00  499.00

Now a histogram of the wine ratings represented by the variable
*points*:

``` r
ggplot(wine, aes(wine$points)) + geom_histogram(binwidth = 1, color='black', fill='dodgerblue') + labs(title="Histogram of Wine Points",x="Points", y = "Count") + theme(plot.title = element_text(hjust = 0.5))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-16-1.png)<!-- -->

``` r
# the 'hjust' is added to center the title, otherwise left-justified
```

This distribution of points appears to be very nearly normal, with a
peak around 87-88 points. Below is a statistical summary:

``` r
summary(wine$points)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   80.00   86.00   88.00   88.45   91.00  100.00

In this near-normal distribution, the mean and median are very close to
one another at 88 and 88.45 points, respectively.

We will now do a more thorough investigation of wine prices and compare
them with the wine ratings to see if a relationship exists between the
two variables. To simplify this analysis, we will divide the wines into
buckets or bins based on their price in $10 increments ($0-$10, $10-$20,
etc.) Then we can see the distribution of wine counts by bin.

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
ggplot(data=as_tibble(price_bins), mapping=aes(x=value)) + geom_bar(fill="dodgerblue") + labs(title="Binned Wine Prices",x="Wine Price Bins, USD", y = "Count") + theme(plot.title = element_text(hjust = 0.5))
```

    ## Warning: Calling `as_tibble()` on a vector is discouraged, because the behavior is likely to change in the future. Use `tibble::enframe(name = NULL)` instead.
    ## This warning is displayed once per session.

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-20-1.png)<!-- -->

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
```

Now we can visualize the wine rating (points) versus price by plotting
box plots of points for each price
bin:

``` r
ggplot(data=wb_group, mapping = aes(x=price_bin, y=points)) + geom_jitter(color='dodgerblue', alpha=0.1) + 
  geom_boxplot(fill='bisque', color='black', alpha=0.3) + theme_minimal()+ labs(title="Boxplots of Wine Price Bins",x="Wine Price Bins, USD", y = "Points") + theme(plot.title = element_text(hjust = 0.5))
```

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-22-1.png)<!-- -->

We can see from this visualization not only the spread in the underlying
price data for each bin, but also the progression of the median points
for each price bin. It is apparent from this plot that overall there is
an increase in rating points associated with an increase in price. This
effect is more dramatic in the lower price bins from zero to 50 dollars,
and the effect is flattened with higher prices, such that the median
rating for a 90-100 dollar wine is not significantly higher than that
for a 70-80 dollar wine.

This raises an interesting causal question: are the more expensive wines
rated higher because they are better? Or are the tasters biased by their
own knowledge of the wine’s price, and therefore more likely to rate the
wine higher if it is more expensive? This question can not be answered
from these data, but the relationship could be further investigated
using a dataset of blind tastings where the tasters are ignorant of the
price when assigning the ratings.

### Word Frequencies in Tasting Notes & Word Cloud

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
nowords <- c("the", "and", "wine", "with", "this", "flavors", "its", "that", "has", "but", "are", "offers", "more", "from")
```

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
wine_corpus = tm_map(wine_corpus, removeWords, nowords)
```

    ## Warning in tm_map.SimpleCorpus(wine_corpus, removeWords, nowords):
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
    ## [1] fragrant fresh grillo opens alluring scents of acacia flower beeswax white stone fruit succulent palate creamy white peach juicy nectarine almond mineral framed in tangy acidity a note of chopped herb closes lingering finish

An inspection above the description for element number 846 in the corpus
shows that the unnecessary features were removed.

Next, I create a document-term matrix for the corpus:

``` r
wine_dtm <- DocumentTermMatrix(wine_corpus)
wine_dtm = removeSparseTerms(wine_dtm, 0.99)
```

Let’s check out the dimensions of the document-term matrix:

``` r
dim(wine_dtm)
```

    ## [1] 129971    454

Then create and sort a dataframe of the sums for each column of the
document-term
matrix.

``` r
freq = data.frame(freqs=sort(colSums(as.matrix(wine_dtm)), decreasing=TRUE))
head(freq, 10)
```

    ##         freqs
    ## fruit   45012
    ## aromas  39613
    ## palate  38083
    ## acidity 34958
    ## finish  34943
    ## tannins 30854
    ## drink   29958
    ## cherry  27380
    ## ripe    26989
    ## black   25388

And finally, generate the word cloud from the frequency
dataframe.

``` r
wordcloud(rownames(freq), freq[,1], max.words=50, colors=brewer.pal(1, "Dark2"))
```

    ## Warning in brewer.pal(1, "Dark2"): minimal value for n is 3, returning requested palette with 3 different levels

![png](/images/2020-05-03-rwinedata_images/unnamed-chunk-30-1.png)<!-- -->

The word cloud above is limited to 50 words, but gives a good idea of
the kinds of words that are frequently used in the wine descriptions in
the dataset. Additional work coud be performed here to remove additional
words that don’t yield much insight, such as “now”.

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
