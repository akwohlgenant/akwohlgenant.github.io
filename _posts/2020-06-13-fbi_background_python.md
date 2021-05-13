---
title: "Investigate FBI National Instant Criminal Background Check System (NICS)"
date: 2020-06-13
tags: [FBI, guns, data wrangling, Python]
header:
 image: "/images/seth-schulte-zYJ9cVjHyZA-unsplash.jpg"
excerpt: "Which states have the most background checks?  Find out with these data from the FBI..."
mathjax: "true"
---

### Table of Contents
<ul>
<li><a href="#intro">Introduction</a></li>
<li><a href="#wrangling">Data Wrangling</a></li>
<li><a href="#eda">Exploratory Data Analysis</a></li>
<li><a href="#conclusions">Conclusions</a></li>
</ul>

<a id='intro'></a>
## Introduction

This project uses Python and several of its associated libraries in a Jupyter Notebook to investigate a dataset derived from the National Instant Background Check System (NICS).  The NICS is a system developed by the United States Federal Bureau of Investigation (FBI) as a means to prevent firearms from falling into the wrong hands.  Here's how it works: when someone tries to buy a firearm in the United States, the seller contacts NICS, either through the internet or by telephone.  The seller submits a form filled out by the prospective buyer, and the system checks to see if the buyer has a criminal record or is otherwise ineligible to purchase a firearm. More details on the NICS system can be found on the FBI website:

 (<https://www.fbi.gov/services/cjis/nics>)

The data that will be utilized for this project was originally published by the FBI in PDF form as a report.  It was later extracted by a third party using code to parse the data and convert it to comma separated values (CSV) format.  Details of the data extraction can be found at: 

(<https://github.com/BuzzFeedNews/nics-firearm-background-checks/blob/master/README.md>)

The dataset contains tabulations by month from 1998 until 2017 for all U.S. states and territories of background checks performed for several categories of firearm, including:

- Handgun: this is a fairly self-explanatory category
- Long Gun: a weapon designed to be fired from the shoulder, such as a rifle or shotgun
- Other: refers to other firearms that are neither handguns nor long guns such as firearms with a pistol grip that expel a shotgun shell

There are also different types of transactions tabulated in the dataset, including pre-pawn checks, and other relatively obscure kinds of firearms transactions.

This project will focus on a single firearm category: **long gun**, with the number of background checks for this type of firearm as the variable of interest.  The project will attempt to answer several questions about long gun background checks:

- Which states tend to have the most background checks for long guns, and which tend to have the least?
- How are states with more long gun background checks different from states with fewer?
- How do the numbers of long gun background checks vary over time?
- What, if any, demographic features of a state are correlated to the number of long gun background checks, and how are they correlated?

It is important to note that there is **not** a one-to-one correlation between background checks and sales of guns, for a variety of reasons including varying state laws.  While background checks are a good indicator of gun sales, the numbers of background checks does not necessarily equal the number of guns sold for a particular category or state.

As mentioned earlier, this analysis is performed using the Python language and several useful open-source libraries for data analysis and visualization, including _pandas_ and _matplotlib_ to name just two.  In the code cell below, the appropriate libraries will be imported, followed by sections on **Data Wrangling**, **Exploratory Data Analysis**, and finally a section containing **Conclusions** based on the analysis.


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
```

<a id='wrangling'></a>
## Data Wrangling

In this section, the NICS background checks dataset will be loaded from an external file in comma separated values (csv) format into a Pandas DataFrame.  The DataFrame containing the data will be inspected for content, general characteristics, and for missing and/or duplicated values, which will be dealt with appropriately.

In addition to the NICS data, a data file containing by-state census data will be loaded into a Pandas DataFrame.  This file will be manipulated to match the format of the NICS data, and ultimately merged with the NICS data variables of interest so that relationships can be investigated between the NICS data and state demographics data.

### General Properties

#### NICS Data


```python
df = pd.read_csv('gun_data_csv.csv')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>month</th>
      <th>state</th>
      <th>permit</th>
      <th>permit_recheck</th>
      <th>handgun</th>
      <th>long_gun</th>
      <th>other</th>
      <th>multiple</th>
      <th>admin</th>
      <th>prepawn_handgun</th>
      <th>...</th>
      <th>returned_other</th>
      <th>rentals_handgun</th>
      <th>rentals_long_gun</th>
      <th>private_sale_handgun</th>
      <th>private_sale_long_gun</th>
      <th>private_sale_other</th>
      <th>return_to_seller_handgun</th>
      <th>return_to_seller_long_gun</th>
      <th>return_to_seller_other</th>
      <th>totals</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>2017-09</td>
      <td>Alabama</td>
      <td>16717.0</td>
      <td>0.0</td>
      <td>5734.0</td>
      <td>6320.0</td>
      <td>221.0</td>
      <td>317</td>
      <td>0.0</td>
      <td>15.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>9.0</td>
      <td>16.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>32019</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2017-09</td>
      <td>Alaska</td>
      <td>209.0</td>
      <td>2.0</td>
      <td>2320.0</td>
      <td>2930.0</td>
      <td>219.0</td>
      <td>160</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>17.0</td>
      <td>24.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6303</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2017-09</td>
      <td>Arizona</td>
      <td>5069.0</td>
      <td>382.0</td>
      <td>11063.0</td>
      <td>7946.0</td>
      <td>920.0</td>
      <td>631</td>
      <td>0.0</td>
      <td>13.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>38.0</td>
      <td>12.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>28394</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2017-09</td>
      <td>Arkansas</td>
      <td>2935.0</td>
      <td>632.0</td>
      <td>4347.0</td>
      <td>6063.0</td>
      <td>165.0</td>
      <td>366</td>
      <td>51.0</td>
      <td>12.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>13.0</td>
      <td>23.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>17747</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2017-09</td>
      <td>California</td>
      <td>57839.0</td>
      <td>0.0</td>
      <td>37165.0</td>
      <td>24581.0</td>
      <td>2984.0</td>
      <td>0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>123506</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 27 columns</p>
</div>



The next code cell will find and print the minimum and maximum dates in the dataset.


```python
print('Minimum date: {},\nMaximum date: {}'.format(min(df['month']), max(df['month'])))
```

    Minimum date: 1998-11,
    Maximum date: 2017-09
    

So these data run from November 1998 until September 2017.


```python
df.shape
```




    (12485, 27)



The dataset contains 12,485 rows and 27 columns.

Below is a listing of the column names, number of non-null entries, and data type for each column:


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 12485 entries, 0 to 12484
    Data columns (total 27 columns):
    month                        12485 non-null object
    state                        12485 non-null object
    permit                       12461 non-null float64
    permit_recheck               1100 non-null float64
    handgun                      12465 non-null float64
    long_gun                     12466 non-null float64
    other                        5500 non-null float64
    multiple                     12485 non-null int64
    admin                        12462 non-null float64
    prepawn_handgun              10542 non-null float64
    prepawn_long_gun             10540 non-null float64
    prepawn_other                5115 non-null float64
    redemption_handgun           10545 non-null float64
    redemption_long_gun          10544 non-null float64
    redemption_other             5115 non-null float64
    returned_handgun             2200 non-null float64
    returned_long_gun            2145 non-null float64
    returned_other               1815 non-null float64
    rentals_handgun              990 non-null float64
    rentals_long_gun             825 non-null float64
    private_sale_handgun         2750 non-null float64
    private_sale_long_gun        2750 non-null float64
    private_sale_other           2750 non-null float64
    return_to_seller_handgun     2475 non-null float64
    return_to_seller_long_gun    2750 non-null float64
    return_to_seller_other       2255 non-null float64
    totals                       12485 non-null int64
    dtypes: float64(23), int64(2), object(2)
    memory usage: 2.6+ MB
    

The goal of this project is to investigate the background checks tabulated in the 'long_gun' feature, so I will make a dataframe containing only that column in addition to month-year and state columns.


```python
df_long = df[['month', 'state', 'long_gun']].copy()
```


```python
df_long.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>month</th>
      <th>state</th>
      <th>long_gun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>2017-09</td>
      <td>Alabama</td>
      <td>6320.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2017-09</td>
      <td>Alaska</td>
      <td>2930.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2017-09</td>
      <td>Arizona</td>
      <td>7946.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2017-09</td>
      <td>Arkansas</td>
      <td>6063.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2017-09</td>
      <td>California</td>
      <td>24581.0</td>
    </tr>
  </tbody>
</table>
</div>



The next two code cells query the dataframe for any duplicated or null values, with the result totals for both printed below the corresponding code cell.


```python
df_long.duplicated().sum()
```




    0




```python
df_long.isnull().sum()
```




    month        0
    state        0
    long_gun    19
    dtype: int64



No duplicate rows were found, but 19 rows were found with null values in the long_gun columns.  Those null data will be dealt with in the next sub-section, _Data Cleaning_.

Now the census data will be loaded and inspected.

#### Census Data


```python
census = pd.read_csv('U.S. Census Data.csv', header=None)
census.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Fact</td>
      <td>Fact Note</td>
      <td>Alabama</td>
      <td>Alaska</td>
      <td>Arizona</td>
      <td>Arkansas</td>
      <td>California</td>
      <td>Colorado</td>
      <td>Connecticut</td>
      <td>Delaware</td>
      <td>...</td>
      <td>South Dakota</td>
      <td>Tennessee</td>
      <td>Texas</td>
      <td>Utah</td>
      <td>Vermont</td>
      <td>Virginia</td>
      <td>Washington</td>
      <td>West Virginia</td>
      <td>Wisconsin</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Population estimates, July 1, 2016,  (V2016)</td>
      <td>NaN</td>
      <td>4863300</td>
      <td>741894</td>
      <td>6931071</td>
      <td>2988248</td>
      <td>39250017</td>
      <td>5540545</td>
      <td>3576452</td>
      <td>952065</td>
      <td>...</td>
      <td>865454</td>
      <td>6651194</td>
      <td>27862596</td>
      <td>3051217</td>
      <td>624594</td>
      <td>8411808</td>
      <td>7288000</td>
      <td>1831102</td>
      <td>5778708</td>
      <td>585501</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Population estimates base, April 1, 2010,  (V2...</td>
      <td>NaN</td>
      <td>4780131</td>
      <td>710249</td>
      <td>6392301</td>
      <td>2916025</td>
      <td>37254522</td>
      <td>5029324</td>
      <td>3574114</td>
      <td>897936</td>
      <td>...</td>
      <td>814195</td>
      <td>6346298</td>
      <td>25146100</td>
      <td>2763888</td>
      <td>625741</td>
      <td>8001041</td>
      <td>6724545</td>
      <td>1853011</td>
      <td>5687289</td>
      <td>563767</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Population, percent change - April 1, 2010 (es...</td>
      <td>NaN</td>
      <td>0.017</td>
      <td>0.045</td>
      <td>0.084</td>
      <td>0.025</td>
      <td>0.054</td>
      <td>0.102</td>
      <td>0.001</td>
      <td>0.06</td>
      <td>...</td>
      <td>0.063</td>
      <td>0.048</td>
      <td>0.108</td>
      <td>0.104</td>
      <td>-0.002</td>
      <td>0.051</td>
      <td>0.084</td>
      <td>-0.012</td>
      <td>0.016</td>
      <td>0.039</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Population, Census, April 1, 2010</td>
      <td>NaN</td>
      <td>4779736</td>
      <td>710231</td>
      <td>6392017</td>
      <td>2915918</td>
      <td>37253956</td>
      <td>5029196</td>
      <td>3574097</td>
      <td>897934</td>
      <td>...</td>
      <td>814180</td>
      <td>6346105</td>
      <td>25145561</td>
      <td>2763885</td>
      <td>625741</td>
      <td>8001024</td>
      <td>6724540</td>
      <td>1852994</td>
      <td>5686986</td>
      <td>563626</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 52 columns</p>
</div>




```python
census.tail(25)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>61</td>
      <td>Veteran-owned firms, 2012</td>
      <td>NaN</td>
      <td>41943</td>
      <td>7953</td>
      <td>46780</td>
      <td>25915</td>
      <td>252377</td>
      <td>51722</td>
      <td>31056</td>
      <td>7206</td>
      <td>...</td>
      <td>8604</td>
      <td>59379</td>
      <td>213590</td>
      <td>18754</td>
      <td>8237</td>
      <td>76434</td>
      <td>49331</td>
      <td>12912</td>
      <td>39830</td>
      <td>6470</td>
    </tr>
    <tr>
      <td>62</td>
      <td>Nonveteran-owned firms, 2012</td>
      <td>NaN</td>
      <td>316984</td>
      <td>56091</td>
      <td>427582</td>
      <td>192988</td>
      <td>3176341</td>
      <td>469524</td>
      <td>281182</td>
      <td>60318</td>
      <td>...</td>
      <td>66219</td>
      <td>469392</td>
      <td>2057218</td>
      <td>219807</td>
      <td>63317</td>
      <td>548439</td>
      <td>461401</td>
      <td>94960</td>
      <td>370755</td>
      <td>51353</td>
    </tr>
    <tr>
      <td>63</td>
      <td>Population per square mile, 2010</td>
      <td>NaN</td>
      <td>94.4</td>
      <td>1.2</td>
      <td>56.3</td>
      <td>56</td>
      <td>239.1</td>
      <td>48.5</td>
      <td>738.1</td>
      <td>460.8</td>
      <td>...</td>
      <td>10.7</td>
      <td>153.9</td>
      <td>96.3</td>
      <td>33.6</td>
      <td>67.9</td>
      <td>202.6</td>
      <td>101.2</td>
      <td>77.1</td>
      <td>105</td>
      <td>5.8</td>
    </tr>
    <tr>
      <td>64</td>
      <td>Land area in square miles, 2010</td>
      <td>NaN</td>
      <td>50645.33</td>
      <td>570640.95</td>
      <td>113594.08</td>
      <td>52035.48</td>
      <td>155779.22</td>
      <td>103641.89</td>
      <td>4842.36</td>
      <td>1948.54</td>
      <td>...</td>
      <td>75811</td>
      <td>41234.9</td>
      <td>261231.71</td>
      <td>82169.62</td>
      <td>9216.66</td>
      <td>39490.09</td>
      <td>66455.52</td>
      <td>24038.21</td>
      <td>54157.8</td>
      <td>97093.14</td>
    </tr>
    <tr>
      <td>65</td>
      <td>FIPS Code</td>
      <td>NaN</td>
      <td>"01"</td>
      <td>"02"</td>
      <td>"04"</td>
      <td>"05"</td>
      <td>"06"</td>
      <td>"08"</td>
      <td>"09"</td>
      <td>"10"</td>
      <td>...</td>
      <td>"46"</td>
      <td>"47"</td>
      <td>"48"</td>
      <td>"49"</td>
      <td>"50"</td>
      <td>"51"</td>
      <td>"53"</td>
      <td>"54"</td>
      <td>"55"</td>
      <td>"56"</td>
    </tr>
    <tr>
      <td>66</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>67</td>
      <td>NOTE: FIPS Code values are enclosed in quotes ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>68</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>69</td>
      <td>Value Notes</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>70</td>
      <td>1</td>
      <td>Includes data not distributed by county.</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>71</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>72</td>
      <td>Fact Notes</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>73</td>
      <td>(a)</td>
      <td>Includes persons reporting only one race</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>74</td>
      <td>(b)</td>
      <td>Hispanics may be of any race, so also are incl...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>75</td>
      <td>(c)</td>
      <td>Economic Census - Puerto Rico data are not com...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>76</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>77</td>
      <td>Value Flags</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>78</td>
      <td>-</td>
      <td>Either no or too few sample observations were ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>79</td>
      <td>D</td>
      <td>Suppressed to avoid disclosure of confidential...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>80</td>
      <td>F</td>
      <td>Fewer than 25 firms</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>81</td>
      <td>FN</td>
      <td>Footnote on this item in place of data</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>82</td>
      <td>NaN</td>
      <td>Not available</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>83</td>
      <td>S</td>
      <td>Suppressed; does not meet publication standards</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>84</td>
      <td>X</td>
      <td>Not applicable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>85</td>
      <td>Z</td>
      <td>Value greater than zero but less than half uni...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>25 rows × 52 columns</p>
</div>



It looks like row 64 is the last valid row; the rest appear to just contain FIPS code and notes.  So rows 65 and greater need to be deleted.

Null values are identified below.


```python
census.isnull().sum().any()
```




    True



Apparently there are null values in the dataset.  Those will be better identified and dealt with after the format manipulations are completed.  The same is true for the duplicated values that are checked below.


```python
census.duplicated().sum().any()
```




    True



There are several issues already apparent in this dataset.  First, the states are in columns while the variables are in rows; this is not ideal for data analysis and doesn't match up well with the NICS dataset in which _state_ is a feature or column of that dataset.  So these data will need to be transposed so that the census features are columns and the state names are a descriptive feature in it's own column.

Another issue is that the names of the features are extremely long and cumbersome.  Those will need to be shortened and have the spaces removed and replaced with underscores.

Finally, the tail of the file appears to contain notes that will be deleted since they have no associated data.

These issues will be dealt with, along with the small amount of cleanup for the NICS data, in the next section, **Data Cleaning**.

### Data Cleaning
#### NICS Data
The NICS dataset is in pretty good shape and requires only minimal cleaning.  The null values identified in the previous section will be eliminated by deleting the rows containing them.  Since there are only 19 null values, it shouldn't affect the analysis.


```python
df_long.dropna(inplace=True)
```


```python
df_long.isnull().sum().any()
```




    False




```python
df_long.shape
```




    (12466, 3)



After deleting the 19 rows containing null values, the NICS dataset moving forward contains 12,466 rows and just the three columns selected in the previous section.

#### Census Data
Now the census dataset will be cleaned up.  First, the rows containing notes and non-data items at the bottom of the file will be deleted.


```python
census.drop(census.index[65:], inplace=True)
census.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>60</td>
      <td>Nonminority-owned firms, 2012</td>
      <td>NaN</td>
      <td>272651</td>
      <td>51147</td>
      <td>344981</td>
      <td>189029</td>
      <td>1819107</td>
      <td>442365</td>
      <td>259614</td>
      <td>54782</td>
      <td>...</td>
      <td>74228</td>
      <td>434025</td>
      <td>1224845</td>
      <td>218826</td>
      <td>70491</td>
      <td>450109</td>
      <td>426697</td>
      <td>104785</td>
      <td>379934</td>
      <td>55397</td>
    </tr>
    <tr>
      <td>61</td>
      <td>Veteran-owned firms, 2012</td>
      <td>NaN</td>
      <td>41943</td>
      <td>7953</td>
      <td>46780</td>
      <td>25915</td>
      <td>252377</td>
      <td>51722</td>
      <td>31056</td>
      <td>7206</td>
      <td>...</td>
      <td>8604</td>
      <td>59379</td>
      <td>213590</td>
      <td>18754</td>
      <td>8237</td>
      <td>76434</td>
      <td>49331</td>
      <td>12912</td>
      <td>39830</td>
      <td>6470</td>
    </tr>
    <tr>
      <td>62</td>
      <td>Nonveteran-owned firms, 2012</td>
      <td>NaN</td>
      <td>316984</td>
      <td>56091</td>
      <td>427582</td>
      <td>192988</td>
      <td>3176341</td>
      <td>469524</td>
      <td>281182</td>
      <td>60318</td>
      <td>...</td>
      <td>66219</td>
      <td>469392</td>
      <td>2057218</td>
      <td>219807</td>
      <td>63317</td>
      <td>548439</td>
      <td>461401</td>
      <td>94960</td>
      <td>370755</td>
      <td>51353</td>
    </tr>
    <tr>
      <td>63</td>
      <td>Population per square mile, 2010</td>
      <td>NaN</td>
      <td>94.4</td>
      <td>1.2</td>
      <td>56.3</td>
      <td>56</td>
      <td>239.1</td>
      <td>48.5</td>
      <td>738.1</td>
      <td>460.8</td>
      <td>...</td>
      <td>10.7</td>
      <td>153.9</td>
      <td>96.3</td>
      <td>33.6</td>
      <td>67.9</td>
      <td>202.6</td>
      <td>101.2</td>
      <td>77.1</td>
      <td>105</td>
      <td>5.8</td>
    </tr>
    <tr>
      <td>64</td>
      <td>Land area in square miles, 2010</td>
      <td>NaN</td>
      <td>50645.33</td>
      <td>570640.95</td>
      <td>113594.08</td>
      <td>52035.48</td>
      <td>155779.22</td>
      <td>103641.89</td>
      <td>4842.36</td>
      <td>1948.54</td>
      <td>...</td>
      <td>75811</td>
      <td>41234.9</td>
      <td>261231.71</td>
      <td>82169.62</td>
      <td>9216.66</td>
      <td>39490.09</td>
      <td>66455.52</td>
      <td>24038.21</td>
      <td>54157.8</td>
      <td>97093.14</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 52 columns</p>
</div>



Next, the 'Fact Note' column will be deleted.


```python
census.drop(census.columns[1], axis=1, inplace=True)
census.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>...</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Fact</td>
      <td>Alabama</td>
      <td>Alaska</td>
      <td>Arizona</td>
      <td>Arkansas</td>
      <td>California</td>
      <td>Colorado</td>
      <td>Connecticut</td>
      <td>Delaware</td>
      <td>Florida</td>
      <td>...</td>
      <td>South Dakota</td>
      <td>Tennessee</td>
      <td>Texas</td>
      <td>Utah</td>
      <td>Vermont</td>
      <td>Virginia</td>
      <td>Washington</td>
      <td>West Virginia</td>
      <td>Wisconsin</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Population estimates, July 1, 2016,  (V2016)</td>
      <td>4863300</td>
      <td>741894</td>
      <td>6931071</td>
      <td>2988248</td>
      <td>39250017</td>
      <td>5540545</td>
      <td>3576452</td>
      <td>952065</td>
      <td>20612439</td>
      <td>...</td>
      <td>865454</td>
      <td>6651194</td>
      <td>27862596</td>
      <td>3051217</td>
      <td>624594</td>
      <td>8411808</td>
      <td>7288000</td>
      <td>1831102</td>
      <td>5778708</td>
      <td>585501</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Population estimates base, April 1, 2010,  (V2...</td>
      <td>4780131</td>
      <td>710249</td>
      <td>6392301</td>
      <td>2916025</td>
      <td>37254522</td>
      <td>5029324</td>
      <td>3574114</td>
      <td>897936</td>
      <td>18804592</td>
      <td>...</td>
      <td>814195</td>
      <td>6346298</td>
      <td>25146100</td>
      <td>2763888</td>
      <td>625741</td>
      <td>8001041</td>
      <td>6724545</td>
      <td>1853011</td>
      <td>5687289</td>
      <td>563767</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Population, percent change - April 1, 2010 (es...</td>
      <td>0.017</td>
      <td>0.045</td>
      <td>0.084</td>
      <td>0.025</td>
      <td>0.054</td>
      <td>0.102</td>
      <td>0.001</td>
      <td>0.06</td>
      <td>0.096</td>
      <td>...</td>
      <td>0.063</td>
      <td>0.048</td>
      <td>0.108</td>
      <td>0.104</td>
      <td>-0.002</td>
      <td>0.051</td>
      <td>0.084</td>
      <td>-0.012</td>
      <td>0.016</td>
      <td>0.039</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Population, Census, April 1, 2010</td>
      <td>4779736</td>
      <td>710231</td>
      <td>6392017</td>
      <td>2915918</td>
      <td>37253956</td>
      <td>5029196</td>
      <td>3574097</td>
      <td>897934</td>
      <td>18801310</td>
      <td>...</td>
      <td>814180</td>
      <td>6346105</td>
      <td>25145561</td>
      <td>2763885</td>
      <td>625741</td>
      <td>8001024</td>
      <td>6724540</td>
      <td>1852994</td>
      <td>5686986</td>
      <td>563626</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 51 columns</p>
</div>



Next, the long feature names in the 'Fact' column will be replaced with shorter names, all lower case, with underscores instead of spaces.  First a list is made of the new feature names.  These names were originally typed into a text editor and pasted into the list below.


```python
features = ['state', 'pop_est_jul_2016', 'pop_est_apr_2010', 'pc_pop_chang_2011_2016','pop_cen_apr_2010', 'pc_under_5yrs_jul_2016', 'pc_under_5yrs_apr_2010', 'pc_under_18yrs_jul_2016', 'pc_under_18yrs_apr_2010', 'pc_65yrs_ jul_2016', 'pc_65yrs_apr_2010', 'pc_female_jul_2016', 'pc_female_apr_2010', 'pc_white_ju_2016', 'pc_black_jul_2016', 'pc_ind_alnat_jul_2016', 'pc_asian_jul_2016', 'pc_haw_pacisl_jul_2016', 'pc_two_or_more_races_jul_2016', 'pc_hispanic_jul_2016', 'pc_white_not_hispanic_jul_2016', 'pop_vets_2011_2015', 'pc_foreign_born_2011_2015', 'hous_units_ jul_2016', 'hous_units_apr_2010', 'pc_owner_occ_2011_2015', 'med_val_own_occ_2011_2015', 'med_monthly_costs_mortg_2011_2015', 'med_monthly_costs_nomortg_2011_2015', 'med_gross_rent_2011_2015', 'build_permits_2016', 'households_2011_2015', 'persons_per_house_2011_2015', 'pc_in_same_house_2011_2015', 'pc_other_than_Engl_spok_home_2011_2015', 'pc_high_school_grad_higher_2011_2015', 'pc_bach_degree_higher_2011_2015', 'pc_with_disability_under_65_2011_2015', 'pc_wout_hlth_insur_under_65_2011_2015', 'pc_in_labor_force_2011_2015', 'pc_in_labor_force_female_2011_2015', 'tot_accom_foodserv_sales_2012', 'tot_hlth_care_rec_div_rev_2012', 'tot_manu_ship_2012', 'tot_merch_wholesale_2012', 'tot_retail_sales_2012', 'tot_retail_percap_2012', 'mean_travel_to_work_min_2011_2015', 'med_hshld_income_2011_2015', 'percap_income_past_12mo_2011_2015', 'pc_in_poverty', 'tot_empl_estab_2015', 'tot_employment_2015', 'tot_annual_payroll_2015', 'pc_change_tot_empl_2014_15', 'tot_nonemp_est_2015', 'all_firms_2012', 'men_owned_firms_2012', 'women_owned_firms_2012', 'minority_owned_firms_2012', 'nonminor_owned_firms_2012', 'vet_owned_firms_2012', 'nonvet_owned_firms_2012', 'pop_per_sqmi_2010', 'land_area_sqmi_2010']
```


```python
print(features)
```

    ['state', 'pop_est_jul_2016', 'pop_est_apr_2010', 'pc_pop_chang_2011_2016', 'pop_cen_apr_2010', 'pc_under_5yrs_jul_2016', 'pc_under_5yrs_apr_2010', 'pc_under_18yrs_jul_2016', 'pc_under_18yrs_apr_2010', 'pc_65yrs_ jul_2016', 'pc_65yrs_apr_2010', 'pc_female_jul_2016', 'pc_female_apr_2010', 'pc_white_ju_2016', 'pc_black_jul_2016', 'pc_ind_alnat_jul_2016', 'pc_asian_jul_2016', 'pc_haw_pacisl_jul_2016', 'pc_two_or_more_races_jul_2016', 'pc_hispanic_jul_2016', 'pc_white_not_hispanic_jul_2016', 'pop_vets_2011_2015', 'pc_foreign_born_2011_2015', 'hous_units_ jul_2016', 'hous_units_apr_2010', 'pc_owner_occ_2011_2015', 'med_val_own_occ_2011_2015', 'med_monthly_costs_mortg_2011_2015', 'med_monthly_costs_nomortg_2011_2015', 'med_gross_rent_2011_2015', 'build_permits_2016', 'households_2011_2015', 'persons_per_house_2011_2015', 'pc_in_same_house_2011_2015', 'pc_other_than_Engl_spok_home_2011_2015', 'pc_high_school_grad_higher_2011_2015', 'pc_bach_degree_higher_2011_2015', 'pc_with_disability_under_65_2011_2015', 'pc_wout_hlth_insur_under_65_2011_2015', 'pc_in_labor_force_2011_2015', 'pc_in_labor_force_female_2011_2015', 'tot_accom_foodserv_sales_2012', 'tot_hlth_care_rec_div_rev_2012', 'tot_manu_ship_2012', 'tot_merch_wholesale_2012', 'tot_retail_sales_2012', 'tot_retail_percap_2012', 'mean_travel_to_work_min_2011_2015', 'med_hshld_income_2011_2015', 'percap_income_past_12mo_2011_2015', 'pc_in_poverty', 'tot_empl_estab_2015', 'tot_employment_2015', 'tot_annual_payroll_2015', 'pc_change_tot_empl_2014_15', 'tot_nonemp_est_2015', 'all_firms_2012', 'men_owned_firms_2012', 'women_owned_firms_2012', 'minority_owned_firms_2012', 'nonminor_owned_firms_2012', 'vet_owned_firms_2012', 'nonvet_owned_firms_2012', 'pop_per_sqmi_2010', 'land_area_sqmi_2010']
    

Then the length of the list of feature names is checked against the dataframe.


```python
len(features)
```




    65




```python
census.shape
```




    (65, 51)



The lengths match, so I will insert the new column containing the shortened feature names into the census dataframe.


```python
#df.insert(loc=idx, column='A', value=new_col)
census.insert(loc=1, column=1, value=features)
census.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Fact</td>
      <td>state</td>
      <td>Alabama</td>
      <td>Alaska</td>
      <td>Arizona</td>
      <td>Arkansas</td>
      <td>California</td>
      <td>Colorado</td>
      <td>Connecticut</td>
      <td>Delaware</td>
      <td>...</td>
      <td>South Dakota</td>
      <td>Tennessee</td>
      <td>Texas</td>
      <td>Utah</td>
      <td>Vermont</td>
      <td>Virginia</td>
      <td>Washington</td>
      <td>West Virginia</td>
      <td>Wisconsin</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Population estimates, July 1, 2016,  (V2016)</td>
      <td>pop_est_jul_2016</td>
      <td>4863300</td>
      <td>741894</td>
      <td>6931071</td>
      <td>2988248</td>
      <td>39250017</td>
      <td>5540545</td>
      <td>3576452</td>
      <td>952065</td>
      <td>...</td>
      <td>865454</td>
      <td>6651194</td>
      <td>27862596</td>
      <td>3051217</td>
      <td>624594</td>
      <td>8411808</td>
      <td>7288000</td>
      <td>1831102</td>
      <td>5778708</td>
      <td>585501</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Population estimates base, April 1, 2010,  (V2...</td>
      <td>pop_est_apr_2010</td>
      <td>4780131</td>
      <td>710249</td>
      <td>6392301</td>
      <td>2916025</td>
      <td>37254522</td>
      <td>5029324</td>
      <td>3574114</td>
      <td>897936</td>
      <td>...</td>
      <td>814195</td>
      <td>6346298</td>
      <td>25146100</td>
      <td>2763888</td>
      <td>625741</td>
      <td>8001041</td>
      <td>6724545</td>
      <td>1853011</td>
      <td>5687289</td>
      <td>563767</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Population, percent change - April 1, 2010 (es...</td>
      <td>pc_pop_chang_2011_2016</td>
      <td>0.017</td>
      <td>0.045</td>
      <td>0.084</td>
      <td>0.025</td>
      <td>0.054</td>
      <td>0.102</td>
      <td>0.001</td>
      <td>0.06</td>
      <td>...</td>
      <td>0.063</td>
      <td>0.048</td>
      <td>0.108</td>
      <td>0.104</td>
      <td>-0.002</td>
      <td>0.051</td>
      <td>0.084</td>
      <td>-0.012</td>
      <td>0.016</td>
      <td>0.039</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Population, Census, April 1, 2010</td>
      <td>pop_cen_apr_2010</td>
      <td>4779736</td>
      <td>710231</td>
      <td>6392017</td>
      <td>2915918</td>
      <td>37253956</td>
      <td>5029196</td>
      <td>3574097</td>
      <td>897934</td>
      <td>...</td>
      <td>814180</td>
      <td>6346105</td>
      <td>25145561</td>
      <td>2763885</td>
      <td>625741</td>
      <td>8001024</td>
      <td>6724540</td>
      <td>1852994</td>
      <td>5686986</td>
      <td>563626</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 52 columns</p>
</div>



Now I can drop the first column with the overly long feature descriptions.


```python
census.drop(census.columns[0], axis=1, inplace=True)
census.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>...</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>state</td>
      <td>Alabama</td>
      <td>Alaska</td>
      <td>Arizona</td>
      <td>Arkansas</td>
      <td>California</td>
      <td>Colorado</td>
      <td>Connecticut</td>
      <td>Delaware</td>
      <td>Florida</td>
      <td>...</td>
      <td>South Dakota</td>
      <td>Tennessee</td>
      <td>Texas</td>
      <td>Utah</td>
      <td>Vermont</td>
      <td>Virginia</td>
      <td>Washington</td>
      <td>West Virginia</td>
      <td>Wisconsin</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <td>1</td>
      <td>pop_est_jul_2016</td>
      <td>4863300</td>
      <td>741894</td>
      <td>6931071</td>
      <td>2988248</td>
      <td>39250017</td>
      <td>5540545</td>
      <td>3576452</td>
      <td>952065</td>
      <td>20612439</td>
      <td>...</td>
      <td>865454</td>
      <td>6651194</td>
      <td>27862596</td>
      <td>3051217</td>
      <td>624594</td>
      <td>8411808</td>
      <td>7288000</td>
      <td>1831102</td>
      <td>5778708</td>
      <td>585501</td>
    </tr>
    <tr>
      <td>2</td>
      <td>pop_est_apr_2010</td>
      <td>4780131</td>
      <td>710249</td>
      <td>6392301</td>
      <td>2916025</td>
      <td>37254522</td>
      <td>5029324</td>
      <td>3574114</td>
      <td>897936</td>
      <td>18804592</td>
      <td>...</td>
      <td>814195</td>
      <td>6346298</td>
      <td>25146100</td>
      <td>2763888</td>
      <td>625741</td>
      <td>8001041</td>
      <td>6724545</td>
      <td>1853011</td>
      <td>5687289</td>
      <td>563767</td>
    </tr>
    <tr>
      <td>3</td>
      <td>pc_pop_chang_2011_2016</td>
      <td>0.017</td>
      <td>0.045</td>
      <td>0.084</td>
      <td>0.025</td>
      <td>0.054</td>
      <td>0.102</td>
      <td>0.001</td>
      <td>0.06</td>
      <td>0.096</td>
      <td>...</td>
      <td>0.063</td>
      <td>0.048</td>
      <td>0.108</td>
      <td>0.104</td>
      <td>-0.002</td>
      <td>0.051</td>
      <td>0.084</td>
      <td>-0.012</td>
      <td>0.016</td>
      <td>0.039</td>
    </tr>
    <tr>
      <td>4</td>
      <td>pop_cen_apr_2010</td>
      <td>4779736</td>
      <td>710231</td>
      <td>6392017</td>
      <td>2915918</td>
      <td>37253956</td>
      <td>5029196</td>
      <td>3574097</td>
      <td>897934</td>
      <td>18801310</td>
      <td>...</td>
      <td>814180</td>
      <td>6346105</td>
      <td>25145561</td>
      <td>2763885</td>
      <td>625741</td>
      <td>8001024</td>
      <td>6724540</td>
      <td>1852994</td>
      <td>5686986</td>
      <td>563626</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 51 columns</p>
</div>



Now the dataframe is looking better, but it still has the features as rows instead of columns.  In the next code cell, I will use the _transpose_ method to flip the axes.


```python
census_trans = census.transpose()
census_trans.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>55</th>
      <th>56</th>
      <th>57</th>
      <th>58</th>
      <th>59</th>
      <th>60</th>
      <th>61</th>
      <th>62</th>
      <th>63</th>
      <th>64</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>state</td>
      <td>pop_est_jul_2016</td>
      <td>pop_est_apr_2010</td>
      <td>pc_pop_chang_2011_2016</td>
      <td>pop_cen_apr_2010</td>
      <td>pc_under_5yrs_jul_2016</td>
      <td>pc_under_5yrs_apr_2010</td>
      <td>pc_under_18yrs_jul_2016</td>
      <td>pc_under_18yrs_apr_2010</td>
      <td>pc_65yrs_ jul_2016</td>
      <td>...</td>
      <td>tot_nonemp_est_2015</td>
      <td>all_firms_2012</td>
      <td>men_owned_firms_2012</td>
      <td>women_owned_firms_2012</td>
      <td>minority_owned_firms_2012</td>
      <td>nonminor_owned_firms_2012</td>
      <td>vet_owned_firms_2012</td>
      <td>nonvet_owned_firms_2012</td>
      <td>pop_per_sqmi_2010</td>
      <td>land_area_sqmi_2010</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Alabama</td>
      <td>4863300</td>
      <td>4780131</td>
      <td>0.017</td>
      <td>4779736</td>
      <td>0.06</td>
      <td>0.064</td>
      <td>0.226</td>
      <td>0.237</td>
      <td>0.161</td>
      <td>...</td>
      <td>322025</td>
      <td>374153</td>
      <td>203604</td>
      <td>137630</td>
      <td>92219</td>
      <td>272651</td>
      <td>41943</td>
      <td>316984</td>
      <td>94.4</td>
      <td>50645.33</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Alaska</td>
      <td>741894</td>
      <td>710249</td>
      <td>0.045</td>
      <td>710231</td>
      <td>0.073</td>
      <td>0.076</td>
      <td>0.252</td>
      <td>0.264</td>
      <td>0.104</td>
      <td>...</td>
      <td>55521</td>
      <td>68032</td>
      <td>35402</td>
      <td>22141</td>
      <td>13688</td>
      <td>51147</td>
      <td>7953</td>
      <td>56091</td>
      <td>1.2</td>
      <td>570640.95</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Arizona</td>
      <td>6931071</td>
      <td>6392301</td>
      <td>0.084</td>
      <td>6392017</td>
      <td>0.063</td>
      <td>0.071</td>
      <td>0.235</td>
      <td>0.255</td>
      <td>0.169</td>
      <td>...</td>
      <td>451951</td>
      <td>499926</td>
      <td>245243</td>
      <td>182425</td>
      <td>135313</td>
      <td>344981</td>
      <td>46780</td>
      <td>427582</td>
      <td>56.3</td>
      <td>113594.08</td>
    </tr>
    <tr>
      <td>5</td>
      <td>Arkansas</td>
      <td>2988248</td>
      <td>2916025</td>
      <td>0.025</td>
      <td>2915918</td>
      <td>0.064</td>
      <td>0.068</td>
      <td>0.236</td>
      <td>0.244</td>
      <td>0.163</td>
      <td>...</td>
      <td>198380</td>
      <td>231959</td>
      <td>123158</td>
      <td>75962</td>
      <td>35982</td>
      <td>189029</td>
      <td>25915</td>
      <td>192988</td>
      <td>56</td>
      <td>52035.48</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 65 columns</p>
</div>



That looks more like it!  Next I will save the transposed file back out to a csv file before moving on.


```python
census_trans.to_csv('census_trans.csv', header=False, index=False)
```

Now I will read the file back into a new dataframe.


```python
census_new = pd.read_csv('census_trans.csv')
census_new.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>pop_est_jul_2016</th>
      <th>pop_est_apr_2010</th>
      <th>pc_pop_chang_2011_2016</th>
      <th>pop_cen_apr_2010</th>
      <th>pc_under_5yrs_jul_2016</th>
      <th>pc_under_5yrs_apr_2010</th>
      <th>pc_under_18yrs_jul_2016</th>
      <th>pc_under_18yrs_apr_2010</th>
      <th>pc_65yrs_ jul_2016</th>
      <th>...</th>
      <th>tot_nonemp_est_2015</th>
      <th>all_firms_2012</th>
      <th>men_owned_firms_2012</th>
      <th>women_owned_firms_2012</th>
      <th>minority_owned_firms_2012</th>
      <th>nonminor_owned_firms_2012</th>
      <th>vet_owned_firms_2012</th>
      <th>nonvet_owned_firms_2012</th>
      <th>pop_per_sqmi_2010</th>
      <th>land_area_sqmi_2010</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Alabama</td>
      <td>4863300</td>
      <td>4780131</td>
      <td>0.017</td>
      <td>4779736</td>
      <td>0.060</td>
      <td>0.064</td>
      <td>0.226</td>
      <td>0.237</td>
      <td>0.161</td>
      <td>...</td>
      <td>322025</td>
      <td>374153</td>
      <td>203604</td>
      <td>137630</td>
      <td>92219</td>
      <td>272651</td>
      <td>41943</td>
      <td>316984</td>
      <td>94.4</td>
      <td>50645.33</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>741894</td>
      <td>710249</td>
      <td>0.045</td>
      <td>710231</td>
      <td>0.073</td>
      <td>0.076</td>
      <td>0.252</td>
      <td>0.264</td>
      <td>0.104</td>
      <td>...</td>
      <td>55521</td>
      <td>68032</td>
      <td>35402</td>
      <td>22141</td>
      <td>13688</td>
      <td>51147</td>
      <td>7953</td>
      <td>56091</td>
      <td>1.2</td>
      <td>570640.95</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Arizona</td>
      <td>6931071</td>
      <td>6392301</td>
      <td>0.084</td>
      <td>6392017</td>
      <td>0.063</td>
      <td>0.071</td>
      <td>0.235</td>
      <td>0.255</td>
      <td>0.169</td>
      <td>...</td>
      <td>451951</td>
      <td>499926</td>
      <td>245243</td>
      <td>182425</td>
      <td>135313</td>
      <td>344981</td>
      <td>46780</td>
      <td>427582</td>
      <td>56.3</td>
      <td>113594.08</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Arkansas</td>
      <td>2988248</td>
      <td>2916025</td>
      <td>0.025</td>
      <td>2915918</td>
      <td>0.064</td>
      <td>0.068</td>
      <td>0.236</td>
      <td>0.244</td>
      <td>0.163</td>
      <td>...</td>
      <td>198380</td>
      <td>231959</td>
      <td>123158</td>
      <td>75962</td>
      <td>35982</td>
      <td>189029</td>
      <td>25915</td>
      <td>192988</td>
      <td>56.0</td>
      <td>52035.48</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>39250017</td>
      <td>37254522</td>
      <td>0.054</td>
      <td>37253956</td>
      <td>0.063</td>
      <td>0.068</td>
      <td>0.232</td>
      <td>0.250</td>
      <td>0.136</td>
      <td>...</td>
      <td>3206958</td>
      <td>3548449</td>
      <td>1852580</td>
      <td>1320085</td>
      <td>1619857</td>
      <td>1819107</td>
      <td>252377</td>
      <td>3176341</td>
      <td>239.1</td>
      <td>155779.22</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 65 columns</p>
</div>




```python
census_new.shape
```




    (50, 65)



Now let's take a quick look at the data for data types and null values.


```python
census_new.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 50 entries, 0 to 49
    Data columns (total 65 columns):
    state                                     50 non-null object
    pop_est_jul_2016                          50 non-null int64
    pop_est_apr_2010                          50 non-null int64
    pc_pop_chang_2011_2016                    50 non-null float64
    pop_cen_apr_2010                          50 non-null int64
    pc_under_5yrs_jul_2016                    50 non-null float64
    pc_under_5yrs_apr_2010                    50 non-null float64
    pc_under_18yrs_jul_2016                   50 non-null float64
    pc_under_18yrs_apr_2010                   50 non-null float64
    pc_65yrs_ jul_2016                        50 non-null float64
    pc_65yrs_apr_2010                         50 non-null float64
    pc_female_jul_2016                        50 non-null float64
    pc_female_apr_2010                        50 non-null float64
    pc_white_ju_2016                          50 non-null float64
    pc_black_jul_2016                         50 non-null float64
    pc_ind_alnat_jul_2016                     50 non-null float64
    pc_asian_jul_2016                         50 non-null float64
    pc_haw_pacisl_jul_2016                    50 non-null float64
    pc_two_or_more_races_jul_2016             50 non-null float64
    pc_hispanic_jul_2016                      50 non-null float64
    pc_white_not_hispanic_jul_2016            50 non-null float64
    pop_vets_2011_2015                        50 non-null int64
    pc_foreign_born_2011_2015                 50 non-null float64
    hous_units_ jul_2016                      50 non-null int64
    hous_units_apr_2010                       50 non-null int64
    pc_owner_occ_2011_2015                    50 non-null float64
    med_val_own_occ_2011_2015                 50 non-null int64
    med_monthly_costs_mortg_2011_2015         50 non-null int64
    med_monthly_costs_nomortg_2011_2015       50 non-null int64
    med_gross_rent_2011_2015                  50 non-null int64
    build_permits_2016                        50 non-null int64
    households_2011_2015                      50 non-null int64
    persons_per_house_2011_2015               50 non-null float64
    pc_in_same_house_2011_2015                50 non-null float64
    pc_other_than_Engl_spok_home_2011_2015    50 non-null float64
    pc_high_school_grad_higher_2011_2015      50 non-null float64
    pc_bach_degree_higher_2011_2015           50 non-null float64
    pc_with_disability_under_65_2011_2015     50 non-null float64
    pc_wout_hlth_insur_under_65_2011_2015     50 non-null float64
    pc_in_labor_force_2011_2015               50 non-null float64
    pc_in_labor_force_female_2011_2015        50 non-null float64
    tot_accom_foodserv_sales_2012             50 non-null int64
    tot_hlth_care_rec_div_rev_2012            50 non-null int64
    tot_manu_ship_2012                        48 non-null float64
    tot_merch_wholesale_2012                  50 non-null int64
    tot_retail_sales_2012                     50 non-null int64
    tot_retail_percap_2012                    50 non-null int64
    mean_travel_to_work_min_2011_2015         50 non-null float64
    med_hshld_income_2011_2015                50 non-null int64
    percap_income_past_12mo_2011_2015         50 non-null int64
    pc_in_poverty                             50 non-null float64
    tot_empl_estab_2015                       50 non-null int64
    tot_employment_2015                       50 non-null int64
    tot_annual_payroll_2015                   50 non-null int64
    pc_change_tot_empl_2014_15                49 non-null float64
    tot_nonemp_est_2015                       50 non-null int64
    all_firms_2012                            50 non-null int64
    men_owned_firms_2012                      50 non-null int64
    women_owned_firms_2012                    50 non-null int64
    minority_owned_firms_2012                 50 non-null int64
    nonminor_owned_firms_2012                 50 non-null int64
    vet_owned_firms_2012                      50 non-null int64
    nonvet_owned_firms_2012                   50 non-null int64
    pop_per_sqmi_2010                         50 non-null float64
    land_area_sqmi_2010                       50 non-null float64
    dtypes: float64(34), int64(30), object(1)
    memory usage: 25.5+ KB
    


```python
census_new.isnull().sum().any()
```




    True



Looks like there are 2 columns containing null values.  Let's get a list of the columns and drop them from the data.


```python
# df.columns[df.isnull().any()].tolist()
census_new.columns[census_new.isnull().any()].tolist()
```




    ['tot_manu_ship_2012', 'pc_change_tot_empl_2014_15']




```python
census_new.drop(['tot_manu_ship_2012', 'pc_change_tot_empl_2014_15'], axis=1, inplace=True)
```


```python
census_new.isnull().sum().any()
```




    False



Lastly, let's check for duplicates.


```python
census_new.duplicated().sum().any()
```




    False



<a id='eda'></a>
## Exploratory Data Analysis

In this section, the data will be investigated using filtering and aggregation.  Summary statistics will be calculated and presented through informative visualizations.

### Which states have the most _long gun_ background checks?

The first question I want to investigate using the NICS data is: which states run the most background checks for the category 'long_gun'?  Recall that the _long gun_ designation refers to:

_"a weapon designed or redesigned, made or remade, and intended to be fired from the shoulder, and designed or redesigned and made or remade to use the energy of the explosive in (a) a fixed metallic cartridge to fire a single projectile through a rifled bore for each single pull of the trigger; or (b) a fixed shotgun shell to fire through a smooth bore either a number of ball shot or a single projectile for each single pull of the trigger."_

This type of gun includes rifles and shotguns, both of which are used extensively for hunting in the United States.  Rifles tend to be used for larger game, such as deer, whereas shotguns are more often used in bird hunting.  I would expect sales of these guns to be most popular in states where hunting is popular, and where there are extensive areas of wilderness in which to hunt wild game.

Let's first group the NICS data by state, and then aggregate the 'long_gun' background checks by summing them at the state level.  Then the results can be listed in descending order, and limited to the top 10 states.


```python
df_long.groupby('state')['long_gun'].sum().sort_values(ascending=False)[0:10]
```




    state
    Pennsylvania      9383642.0
    Texas             7651396.0
    California        5936770.0
    Florida           3829090.0
    Ohio              3646325.0
    Missouri          3071938.0
    North Carolina    2962831.0
    Tennessee         2866345.0
    Virginia          2861010.0
    Michigan          2860539.0
    Name: long_gun, dtype: float64



Let's convert those sums to percentages of the total of all long gun sales in the dataset.  First let's make a dataframe of the sums by state using _groupby_ :


```python
df_long_sum = pd.DataFrame(df_long.groupby('state', as_index=False)['long_gun'].sum())
df_long_sum.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>long_gun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Alabama</td>
      <td>2626029.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>572174.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Arizona</td>
      <td>1480762.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Arkansas</td>
      <td>1663256.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>5936770.0</td>
    </tr>
  </tbody>
</table>
</div>



And now I can compute a new column for each state representing the percent of the total long gun sales from that state.


```python
long_tot = df_long['long_gun'].sum()
df_long_sum['prop'] = df_long_sum['long_gun'] / long_tot * 100
df_long_sum.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>long_gun</th>
      <th>prop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Alabama</td>
      <td>2626029.0</td>
      <td>2.696958</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>572174.0</td>
      <td>0.587628</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Arizona</td>
      <td>1480762.0</td>
      <td>1.520758</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Arkansas</td>
      <td>1663256.0</td>
      <td>1.708181</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>5936770.0</td>
      <td>6.097123</td>
    </tr>
  </tbody>
</table>
</div>



Now let's see the top 10 states, but with percent of total sales listed instead of number:


```python
df_long_sum.sort_values('prop', ascending=False)[0:10].round(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>long_gun</th>
      <th>prop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>40</td>
      <td>Pennsylvania</td>
      <td>9383642.0</td>
      <td>9.6</td>
    </tr>
    <tr>
      <td>46</td>
      <td>Texas</td>
      <td>7651396.0</td>
      <td>7.9</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>5936770.0</td>
      <td>6.1</td>
    </tr>
    <tr>
      <td>9</td>
      <td>Florida</td>
      <td>3829090.0</td>
      <td>3.9</td>
    </tr>
    <tr>
      <td>37</td>
      <td>Ohio</td>
      <td>3646325.0</td>
      <td>3.7</td>
    </tr>
    <tr>
      <td>27</td>
      <td>Missouri</td>
      <td>3071938.0</td>
      <td>3.2</td>
    </tr>
    <tr>
      <td>35</td>
      <td>North Carolina</td>
      <td>2962831.0</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>45</td>
      <td>Tennessee</td>
      <td>2866345.0</td>
      <td>2.9</td>
    </tr>
    <tr>
      <td>50</td>
      <td>Virginia</td>
      <td>2861010.0</td>
      <td>2.9</td>
    </tr>
    <tr>
      <td>24</td>
      <td>Michigan</td>
      <td>2860539.0</td>
      <td>2.9</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_long_sum.sort_values('prop', ascending=False)[0:10]['prop'].sum().round()
```




    46.0



As we can see above, these 10 states account for about 46% of all long gun sales in the United States over the period from 1998 until 2017. 

It is important to note here that a number of states in the top 10 are states with very large populations, including California and Texas.  Later the _per capita_ long gun background checks will be investigated, and differences will be noted.

Let's also look at the bottom 10 states or territories in terms of percent of total long gun background checks.


```python
df_long_sum.sort_values('prop', ascending=False)[-10:].round(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>long_gun</th>
      <th>prop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>23</td>
      <td>Massachusetts</td>
      <td>430886.0</td>
      <td>0.4</td>
    </tr>
    <tr>
      <td>48</td>
      <td>Vermont</td>
      <td>285064.0</td>
      <td>0.3</td>
    </tr>
    <tr>
      <td>7</td>
      <td>Delaware</td>
      <td>242235.0</td>
      <td>0.2</td>
    </tr>
    <tr>
      <td>42</td>
      <td>Rhode Island</td>
      <td>121830.0</td>
      <td>0.1</td>
    </tr>
    <tr>
      <td>41</td>
      <td>Puerto Rico</td>
      <td>31533.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>11</td>
      <td>Guam</td>
      <td>6035.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>8</td>
      <td>District of Columbia</td>
      <td>605.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>49</td>
      <td>Virgin Islands</td>
      <td>431.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>21</td>
      <td>Mariana Islands</td>
      <td>182.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>12</td>
      <td>Hawaii</td>
      <td>35.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



This list is dominated by very small states and islands; these are most likely places where very little hunting is done and, therefore, much fewer long guns are purchased.

Below is a bar graph showing the percentage of total long gun background checks for each state and territory in the dataset, arranged in descending order of proportion of the total.


```python
df_long_sum.sort_values('prop', ascending=False).plot(x='state', y='prop', kind='bar', figsize=(12,8), label='Percent of Total')
plt.title('Percent of Total Long Gun Background Checks by State')
plt.xlabel('State')
plt.ylabel('Percent of Total Background Checks');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_71_0.png)


### Time variation of long gun background checks

- How has the monthly number of long gun background checks varied over the time period represented in the dataset (1998 - 2017)?
- Are there seasonal or annual patterns in the data?
- Is there an overall trend aside from any seasonal variation?

The best way to investigate trends over time is with line graphs over the period of interest.  Let's start by looking at a line graph of all sales over the interval of interest.


```python
long_totals = df_long.groupby('month', as_index=False).sum()
long_totals.plot(x='month', y='long_gun', kind='line', figsize=(12,8))
plt.title('Total Long Gun Background Checks')
plt.xlabel('Year-Month')
plt.ylabel('Total Checks');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_73_0.png)


This dataset contains obvious cyclicity that appears to repeat every year, with an overall trend obscured somewhat by the cyclicity.  It would be nice to smooth out the annual cyclicity and see the overall trend. First let's try to see what an average year looks like by grouping all the common months together and calculating an average by month.  First I will need to separate the month from the year in a new column I will call 'month_only'.


```python
long_totals['month_only'] = pd.DatetimeIndex(long_totals['month']).month
long_totals.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>month</th>
      <th>long_gun</th>
      <th>month_only</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1998-11</td>
      <td>11909.0</td>
      <td>11</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1998-12</td>
      <td>570882.0</td>
      <td>12</td>
    </tr>
    <tr>
      <td>2</td>
      <td>1999-01</td>
      <td>309915.0</td>
      <td>1</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1999-02</td>
      <td>352411.0</td>
      <td>2</td>
    </tr>
    <tr>
      <td>4</td>
      <td>1999-03</td>
      <td>376775.0</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



Now let's see what an average year looks like, with the data average for each calendar month:


```python
long_totals.groupby('month_only', as_index=False)['long_gun'].mean().plot(x='month_only',y='long_gun', kind='line', figsize=(12,8))
plt.title('Annual Average Trend in Long Gun Background Checks')
plt.xlabel('Calendar Month')
plt.ylabel('Average Monthly Background Checks');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_77_0.png)


So now we can see that in a typical year, the long gun sales reach a minimum around June and a maximum in December.

I can try to smooth through some of this using a rolling or moving average calculation on the 'long_gun' variable with a chosen averaging window:


```python
window=12
long_totals['mov_avg'] = long_totals['long_gun'].rolling(window=window).mean()
long_totals.plot('month','mov_avg', kind='line', figsize=(12,8))
plt.title('Total Long Gun Background Checks, 12 Month Moving Average')
plt.xlabel('Year-Month')
plt.ylabel('Total Monthly Background Checks, 12 mo. Moving Avg');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_79_0.png)


Now the spike in sales in 2012 appears much more dramatic!  We can also see that the overall trend has been increasing since 2003 until the end of the dataset in 2017.

Let's locate the month in 2012-2013 where the spike took place:


```python
long_totals[long_totals['long_gun']==long_totals['long_gun'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>month</th>
      <th>long_gun</th>
      <th>month_only</th>
      <th>mov_avg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>169</td>
      <td>2012-12</td>
      <td>1224465.0</td>
      <td>12</td>
      <td>572185.416667</td>
    </tr>
  </tbody>
</table>
</div>



So the big spike in background checks (and presumably sales) for long guns took place in December 2012.  This was just after the election of Barack Obama to his second term as president, and many hunters and gun enthusiasts feared that proposed gun control measures would curtail their ability to buy guns in the future, which in turn caused a spike in gun purchases.

Let's plot a vertical line on the original graph to mark December 2012.


```python
long_totals = df_long.groupby('month', as_index=False).sum()
long_totals.plot(x='month', y='long_gun', kind='line', figsize=(12,8))
plt.title('Total Long Gun Background Checks')
plt.xlabel('Year-Month')
plt.ylabel('Total Checks')
plt.vlines(169, ymin=0, ymax=1300000, linestyles='dashed');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_84_0.png)


### Demographic characteristics of states with high numbers of long gun background checks

- What are some characteristics of the states with the highest number of long gun background checks?

- How do by-state census summary statistics correlate with the by-state aggregated long gun background checks? 

- Are there large differences between demographic characteristics between states with high numbers of long gun background checks and states with low numbers?

We can investigate these questions by utilizing the census data that has been aggregated to the state level.  This dataset contains demographic data by state, as well as data on other human characteristics of the states.  Let's see what demographic characteristics are shared by states with high numbers of long gun background checks.  Also, we can try to quantify how well those characteristics are correlated to the long gun background check data.

First let's revisit the census data from the **Data Wrangling** section of the report.


```python
census_new.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>pop_est_jul_2016</th>
      <th>pop_est_apr_2010</th>
      <th>pc_pop_chang_2011_2016</th>
      <th>pop_cen_apr_2010</th>
      <th>pc_under_5yrs_jul_2016</th>
      <th>pc_under_5yrs_apr_2010</th>
      <th>pc_under_18yrs_jul_2016</th>
      <th>pc_under_18yrs_apr_2010</th>
      <th>pc_65yrs_ jul_2016</th>
      <th>...</th>
      <th>tot_nonemp_est_2015</th>
      <th>all_firms_2012</th>
      <th>men_owned_firms_2012</th>
      <th>women_owned_firms_2012</th>
      <th>minority_owned_firms_2012</th>
      <th>nonminor_owned_firms_2012</th>
      <th>vet_owned_firms_2012</th>
      <th>nonvet_owned_firms_2012</th>
      <th>pop_per_sqmi_2010</th>
      <th>land_area_sqmi_2010</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Alabama</td>
      <td>4863300</td>
      <td>4780131</td>
      <td>0.017</td>
      <td>4779736</td>
      <td>0.060</td>
      <td>0.064</td>
      <td>0.226</td>
      <td>0.237</td>
      <td>0.161</td>
      <td>...</td>
      <td>322025</td>
      <td>374153</td>
      <td>203604</td>
      <td>137630</td>
      <td>92219</td>
      <td>272651</td>
      <td>41943</td>
      <td>316984</td>
      <td>94.4</td>
      <td>50645.33</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>741894</td>
      <td>710249</td>
      <td>0.045</td>
      <td>710231</td>
      <td>0.073</td>
      <td>0.076</td>
      <td>0.252</td>
      <td>0.264</td>
      <td>0.104</td>
      <td>...</td>
      <td>55521</td>
      <td>68032</td>
      <td>35402</td>
      <td>22141</td>
      <td>13688</td>
      <td>51147</td>
      <td>7953</td>
      <td>56091</td>
      <td>1.2</td>
      <td>570640.95</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Arizona</td>
      <td>6931071</td>
      <td>6392301</td>
      <td>0.084</td>
      <td>6392017</td>
      <td>0.063</td>
      <td>0.071</td>
      <td>0.235</td>
      <td>0.255</td>
      <td>0.169</td>
      <td>...</td>
      <td>451951</td>
      <td>499926</td>
      <td>245243</td>
      <td>182425</td>
      <td>135313</td>
      <td>344981</td>
      <td>46780</td>
      <td>427582</td>
      <td>56.3</td>
      <td>113594.08</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Arkansas</td>
      <td>2988248</td>
      <td>2916025</td>
      <td>0.025</td>
      <td>2915918</td>
      <td>0.064</td>
      <td>0.068</td>
      <td>0.236</td>
      <td>0.244</td>
      <td>0.163</td>
      <td>...</td>
      <td>198380</td>
      <td>231959</td>
      <td>123158</td>
      <td>75962</td>
      <td>35982</td>
      <td>189029</td>
      <td>25915</td>
      <td>192988</td>
      <td>56.0</td>
      <td>52035.48</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>39250017</td>
      <td>37254522</td>
      <td>0.054</td>
      <td>37253956</td>
      <td>0.063</td>
      <td>0.068</td>
      <td>0.232</td>
      <td>0.250</td>
      <td>0.136</td>
      <td>...</td>
      <td>3206958</td>
      <td>3548449</td>
      <td>1852580</td>
      <td>1320085</td>
      <td>1619857</td>
      <td>1819107</td>
      <td>252377</td>
      <td>3176341</td>
      <td>239.1</td>
      <td>155779.22</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 63 columns</p>
</div>




```python
len(census_new.columns.values)
```




    63



Wow, 63 features!  That's too many to look through one by one, so let's use some of the handy built-in functionality of pandas to find features that appear to correlate to our variable of interest: long gun background checks.  First, let's summarize the long gun background checks into summary statistics by state, and list the top 10 states, this time by mean number of monthly long gun background checks.


```python
df_sum_stats = df_long.groupby('state')['long_gun'].describe()
df_sum_stats.sort_values('mean', ascending=False)[0:10]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>state</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Pennsylvania</td>
      <td>227.0</td>
      <td>41337.629956</td>
      <td>19618.666658</td>
      <td>8.0</td>
      <td>31805.5</td>
      <td>43052.0</td>
      <td>53338.0</td>
      <td>105826.0</td>
    </tr>
    <tr>
      <td>Texas</td>
      <td>227.0</td>
      <td>33706.590308</td>
      <td>14421.708421</td>
      <td>1349.0</td>
      <td>23145.0</td>
      <td>31314.0</td>
      <td>40248.5</td>
      <td>108058.0</td>
    </tr>
    <tr>
      <td>California</td>
      <td>227.0</td>
      <td>26153.171806</td>
      <td>11601.263578</td>
      <td>950.0</td>
      <td>19258.5</td>
      <td>23807.0</td>
      <td>29544.0</td>
      <td>93224.0</td>
    </tr>
    <tr>
      <td>Florida</td>
      <td>227.0</td>
      <td>16868.237885</td>
      <td>7915.507832</td>
      <td>443.0</td>
      <td>11049.0</td>
      <td>15492.0</td>
      <td>20597.5</td>
      <td>59904.0</td>
    </tr>
    <tr>
      <td>Ohio</td>
      <td>227.0</td>
      <td>16063.105727</td>
      <td>6398.137886</td>
      <td>434.0</td>
      <td>11635.0</td>
      <td>14317.0</td>
      <td>18808.0</td>
      <td>47197.0</td>
    </tr>
    <tr>
      <td>Missouri</td>
      <td>227.0</td>
      <td>13532.766520</td>
      <td>5164.387659</td>
      <td>458.0</td>
      <td>10296.0</td>
      <td>12178.0</td>
      <td>16296.5</td>
      <td>36872.0</td>
    </tr>
    <tr>
      <td>North Carolina</td>
      <td>227.0</td>
      <td>13052.118943</td>
      <td>6091.621816</td>
      <td>695.0</td>
      <td>9255.0</td>
      <td>11406.0</td>
      <td>14765.0</td>
      <td>41758.0</td>
    </tr>
    <tr>
      <td>Tennessee</td>
      <td>227.0</td>
      <td>12627.070485</td>
      <td>6028.096926</td>
      <td>85.0</td>
      <td>9248.5</td>
      <td>10938.0</td>
      <td>15073.0</td>
      <td>43721.0</td>
    </tr>
    <tr>
      <td>Virginia</td>
      <td>227.0</td>
      <td>12603.568282</td>
      <td>5327.054334</td>
      <td>2.0</td>
      <td>8663.0</td>
      <td>11132.0</td>
      <td>15701.5</td>
      <td>39104.0</td>
    </tr>
    <tr>
      <td>Michigan</td>
      <td>227.0</td>
      <td>12601.493392</td>
      <td>5188.161217</td>
      <td>443.0</td>
      <td>8623.0</td>
      <td>11103.0</td>
      <td>14896.5</td>
      <td>28946.0</td>
    </tr>
  </tbody>
</table>
</div>



This looks very similar to the top 10 list in the previous section made from the sum of all long gun background checks across all months in the dataset.  Are means the appropriate statistics to examine?  Let's see the distributions of long gun background checks for several states.  First, let's look at Pennsylvania:


```python
df_long[df_long['state']=='Pennsylvania']['long_gun'].hist()
plt.title('Pennsylvania Long Gun Background Checks')
plt.xlabel('Monthly Background Checks')
plt.ylabel('Frequency');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_91_0.png)


The distribution for Pennsylvania is bimodal and somewhat right-skewed.  Now let's look at Texas, California, and Florida:


```python
df_long[df_long['state']=='Texas']['long_gun'].hist()
plt.title('Texas Long Gun Background Checks')
plt.xlabel('Monthly Background Checks')
plt.ylabel('Frequency');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_93_0.png)



```python
df_long[df_long['state']=='California']['long_gun'].hist()
plt.title('California Long Gun Background Checks')
plt.xlabel('Monthly Background Checks')
plt.ylabel('Frequency');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_94_0.png)



```python
df_long[df_long['state']=='Florida']['long_gun'].hist()
plt.title('Florida Long Gun Background Checks')
plt.xlabel('Monthly Background Checks')
plt.ylabel('Frequency');
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_95_0.png)


These states also have right-skewed distributions, suggesting we should consider using the medians for comparison rather than the means.  In the next code cell, I will make a new dataframe of the long gun NICS data, grouped by state, and with the median value of long gun background checks calculated and added as a column.


```python
df_long_grp_med = df_long.groupby('state', as_index=False)['long_gun'].median()
df_long_grp_med.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>long_gun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Alabama</td>
      <td>9837.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>2391.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Arizona</td>
      <td>6078.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Arkansas</td>
      <td>6118.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>23807.0</td>
    </tr>
  </tbody>
</table>
</div>



Then I will merge it with the census data into a common dataframe with each row as a state.  The resulting dataframe will have a row for every state, but will not include the territories since they are not present in the census data.


```python
census_longgun_mrg = census_new.merge(df_long_grp_med, on='state')
census_longgun_mrg.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>pop_est_jul_2016</th>
      <th>pop_est_apr_2010</th>
      <th>pc_pop_chang_2011_2016</th>
      <th>pop_cen_apr_2010</th>
      <th>pc_under_5yrs_jul_2016</th>
      <th>pc_under_5yrs_apr_2010</th>
      <th>pc_under_18yrs_jul_2016</th>
      <th>pc_under_18yrs_apr_2010</th>
      <th>pc_65yrs_ jul_2016</th>
      <th>...</th>
      <th>all_firms_2012</th>
      <th>men_owned_firms_2012</th>
      <th>women_owned_firms_2012</th>
      <th>minority_owned_firms_2012</th>
      <th>nonminor_owned_firms_2012</th>
      <th>vet_owned_firms_2012</th>
      <th>nonvet_owned_firms_2012</th>
      <th>pop_per_sqmi_2010</th>
      <th>land_area_sqmi_2010</th>
      <th>long_gun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Alabama</td>
      <td>4863300</td>
      <td>4780131</td>
      <td>0.017</td>
      <td>4779736</td>
      <td>0.060</td>
      <td>0.064</td>
      <td>0.226</td>
      <td>0.237</td>
      <td>0.161</td>
      <td>...</td>
      <td>374153</td>
      <td>203604</td>
      <td>137630</td>
      <td>92219</td>
      <td>272651</td>
      <td>41943</td>
      <td>316984</td>
      <td>94.4</td>
      <td>50645.33</td>
      <td>9837.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>741894</td>
      <td>710249</td>
      <td>0.045</td>
      <td>710231</td>
      <td>0.073</td>
      <td>0.076</td>
      <td>0.252</td>
      <td>0.264</td>
      <td>0.104</td>
      <td>...</td>
      <td>68032</td>
      <td>35402</td>
      <td>22141</td>
      <td>13688</td>
      <td>51147</td>
      <td>7953</td>
      <td>56091</td>
      <td>1.2</td>
      <td>570640.95</td>
      <td>2391.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Arizona</td>
      <td>6931071</td>
      <td>6392301</td>
      <td>0.084</td>
      <td>6392017</td>
      <td>0.063</td>
      <td>0.071</td>
      <td>0.235</td>
      <td>0.255</td>
      <td>0.169</td>
      <td>...</td>
      <td>499926</td>
      <td>245243</td>
      <td>182425</td>
      <td>135313</td>
      <td>344981</td>
      <td>46780</td>
      <td>427582</td>
      <td>56.3</td>
      <td>113594.08</td>
      <td>6078.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Arkansas</td>
      <td>2988248</td>
      <td>2916025</td>
      <td>0.025</td>
      <td>2915918</td>
      <td>0.064</td>
      <td>0.068</td>
      <td>0.236</td>
      <td>0.244</td>
      <td>0.163</td>
      <td>...</td>
      <td>231959</td>
      <td>123158</td>
      <td>75962</td>
      <td>35982</td>
      <td>189029</td>
      <td>25915</td>
      <td>192988</td>
      <td>56.0</td>
      <td>52035.48</td>
      <td>6118.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>California</td>
      <td>39250017</td>
      <td>37254522</td>
      <td>0.054</td>
      <td>37253956</td>
      <td>0.063</td>
      <td>0.068</td>
      <td>0.232</td>
      <td>0.250</td>
      <td>0.136</td>
      <td>...</td>
      <td>3548449</td>
      <td>1852580</td>
      <td>1320085</td>
      <td>1619857</td>
      <td>1819107</td>
      <td>252377</td>
      <td>3176341</td>
      <td>239.1</td>
      <td>155779.22</td>
      <td>23807.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 64 columns</p>
</div>




```python
census_longgun_mrg.shape
```




    (50, 64)



The numbers of background checks by state tend to be dominated by the states with large populations.  It would probably be better to look for comparisons based on per capita background checks.  So let's calculate a new feature that is the per capita median number of long gun background checks for each state by dividing the median number of background checks by the estimated state population ('pop_est_jul_2016').  We will also have it be per 1000 people just so the numbers are more manageable.


```python
census_longgun_mrg['longgun_med_percap_thou']=census_longgun_mrg['long_gun'] / (census_longgun_mrg['pop_est_jul_2016']/1000)
```


```python
census_longgun_mrg['longgun_med_percap_thou'].hist()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1b1ab182b48>




![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_103_1.png)


Quite a bimodal distribution!  First let's see what states have the highest per capita number of long gun background checks:


```python
census_longgun_mrg.sort_values('longgun_med_percap_thou', ascending=False)[['state', 'longgun_med_percap_thou']][0:10]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>state</th>
      <th>longgun_med_percap_thou</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>25</td>
      <td>Montana</td>
      <td>3.782182</td>
    </tr>
    <tr>
      <td>40</td>
      <td>South Dakota</td>
      <td>3.540338</td>
    </tr>
    <tr>
      <td>37</td>
      <td>Pennsylvania</td>
      <td>3.367587</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alaska</td>
      <td>3.222832</td>
    </tr>
    <tr>
      <td>47</td>
      <td>West Virginia</td>
      <td>3.147285</td>
    </tr>
    <tr>
      <td>49</td>
      <td>Wyoming</td>
      <td>3.130652</td>
    </tr>
    <tr>
      <td>33</td>
      <td>North Dakota</td>
      <td>3.126847</td>
    </tr>
    <tr>
      <td>11</td>
      <td>Idaho</td>
      <td>2.182231</td>
    </tr>
    <tr>
      <td>36</td>
      <td>Oregon</td>
      <td>2.086985</td>
    </tr>
    <tr>
      <td>18</td>
      <td>Maine</td>
      <td>2.077389</td>
    </tr>
  </tbody>
</table>
</div>



Looks like it's mostly made up of states with relatively low populations and lots of wide open spaces, like my home state of Montana at number 1!  We also see Pennsylvania, which was at the op of previous lists.

Now let's look for correlations.  Let's break out any feature that has an absolute value of correlation coefficient with out target variable that is greater than 0.5.


```python
corr = pd.DataFrame(census_longgun_mrg.corr()['longgun_med_percap_thou']).reset_index()
high_corr = corr[corr['longgun_med_percap_thou'].abs()>.5]
high_corr
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>longgun_med_percap_thou</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>10</td>
      <td>pc_female_jul_2016</td>
      <td>-0.507312</td>
    </tr>
    <tr>
      <td>19</td>
      <td>pc_white_not_hispanic_jul_2016</td>
      <td>0.549060</td>
    </tr>
    <tr>
      <td>21</td>
      <td>pc_foreign_born_2011_2015</td>
      <td>-0.696998</td>
    </tr>
    <tr>
      <td>26</td>
      <td>med_monthly_costs_mortg_2011_2015</td>
      <td>-0.513026</td>
    </tr>
    <tr>
      <td>28</td>
      <td>med_gross_rent_2011_2015</td>
      <td>-0.564111</td>
    </tr>
    <tr>
      <td>33</td>
      <td>pc_other_than_Engl_spok_home_2011_2015</td>
      <td>-0.589257</td>
    </tr>
    <tr>
      <td>45</td>
      <td>mean_travel_to_work_min_2011_2015</td>
      <td>-0.623475</td>
    </tr>
    <tr>
      <td>60</td>
      <td>pop_per_sqmi_2010</td>
      <td>-0.558424</td>
    </tr>
    <tr>
      <td>63</td>
      <td>longgun_med_percap_thou</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



Looks like the highest absolute correlation is with the variable 'pc_foreign_born_2011_2015' which is a negative correlation of -0.697.  Let''s see a scatter plot for each of these (except of course the last one in the list!).


```python
feat_list = list(high_corr.iloc[:-1, 0])
feat_list
```




    ['pc_female_jul_2016',
     'pc_white_not_hispanic_jul_2016',
     'pc_foreign_born_2011_2015',
     'med_monthly_costs_mortg_2011_2015',
     'med_gross_rent_2011_2015',
     'pc_other_than_Engl_spok_home_2011_2015',
     'mean_travel_to_work_min_2011_2015',
     'pop_per_sqmi_2010']




```python
for feat in feat_list:
    census_longgun_mrg.plot(x=feat, y='longgun_med_percap_thou', kind='scatter')
```


![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_0.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_1.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_2.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_3.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_4.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_5.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_6.png)



![png](/images/2020_06_13_fbi_background_python/AndroWohlgenant_Investigate_a_Dataset_10June2020_110_7.png)


The feature with the highest absolute value of correlation coefficient is 'pc_foreign_born_2011_2015' which represents the percentage of people in the state born in a foreign country.  This statistic is inversely correlated with long gun background checks, such that states with few foreign-born people tend to have more per capita long gun background checks. This is not obvious, but it could be that foreign-born Americans tend to live more on the coasts and in urban areas where hunting is not as popular.

To me, the features that make the most intuitive sense as being related to long gun sales are the 'mean_travel_to_work_min_2011_2015' and 'pop_per_sqmi_2010' features, which are both inversely correlated to the target variable.  Highly rural states tend to have very short commute times and low population densities, and also are states where hunting is very popular, and therefore, where I would expect to see a large number of people trying to purchase hunting rifles and shotguns.

I am surprised land area wasn't better correlated to long gun background checks, although California has both large land area and large population, so perhaps it isn't so surprising.


```python
corr[corr['index']=='land_area_sqmi_2010']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>longgun_med_percap_thou</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>61</td>
      <td>land_area_sqmi_2010</td>
      <td>0.324566</td>
    </tr>
  </tbody>
</table>
</div>



<a id='conclusions'></a>
## Conclusions

The exploration and analysis of the NICS and U.S. Census data provided insights into which states tend to have more background checks for purchase of so-called 'long guns', or guns designed to be fired from the shoulder, such as rifles and shotguns.  On a total number basis, the top states were states with fairly large populations (California and Texas) and states with a hunting culture like Pennsylvania.  When these numbers are corrected for state population, a different set of states emerged as the top states in terms of _per capita_ number of background checks for long guns.  This list was dominated by sparsely populated states in the west such as Montana and South Dakota where there is much hunting and wild game.

The overall trend of long gun background checks has been increasing since 2003 until the last year in the dataset, 2017.  There is marked annual cyclicity in the time data, such that background check hit a peak in December every year, and then drop to a minimum in the early summer months.  This is probably driven by a combination of factors:  1) most hunting seasons are in the late fall and winter, and 2) guns may be purchased as Christmas gifts.  After smoothing the cyclical data using a 12-month moving average, it became apparent that the overall trend is increasing, and there was a dramatic spike in gun purchases in December 2012, probably related to the election of Barack Obama to his second presidential term.

The NICS data were combined with the U.S. Census data to look for correlations between long gun background checks and demographic data.  The strongest correlations tended to be inverse correlations between factors related to the population density and also to the percentage of foreign-born people, which may indirectly be a function of population density and distance from major urban areas.  While correlations were found, it is important to note that any relationships noted are merely associations; no rigorous statistical analyses were performed, and no causality can be inferred.
