---
title: "Mapping San Francisco Crime Incidents using Python and Folium"
date: 2020-05-19
tags: [folium, python, map]
header:
 image: "/images/gordon-mak-_fSrmawumtQ-unsplash.jpg"
excerpt: "map of San Francisco crime data using the folium library"
mathjax: "true"
---

## Introduction:

Full disclosure: this notebook is a variation on a class project from the IBM Data Science Professional Certificate program through Coursera that I completed in  the fall of 2019. This particular project was part of course 7 of the series, entitled _Data Visualization with Python_.
<br>
<br>
A map of San Francisco will be constructed using the **folium** library.  Then crime incidents will be posted to the map from a csv file downloaded from the city of San Francisco consisting of all criminal incidents since 2018.  That file contains too many data points to comfortably view on a map, so I will reduce the number of incidents before plotting them.  Then I will add descriptive information to the map about each crime, accessible as a popup when a particular incident marker is clicked.


```python
# In case folium needs to be installed...
#!conda install -c conda-forge folium=0.10.0 --yes
```


```python
import pandas as pd
import numpy as np
import folium
```

### Data:
As noted earlier, the crime data for this notebook can be downloaded from the city of San Francisco's open data site.  A link to the data is provided below.
<br>
Data source:
<br>
https://data.sfgov.org/Public-Safety/Police-Department-Incident-Reports-2018-to-Present/wg3w-h783


```python
incidents = pd.read_csv('Police_Department_Incident_Reports__2018_to_Present.csv')
incidents.head()
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
      <th>Incident Datetime</th>
      <th>Incident Date</th>
      <th>Incident Time</th>
      <th>Incident Year</th>
      <th>Incident Day of Week</th>
      <th>Report Datetime</th>
      <th>Row ID</th>
      <th>Incident ID</th>
      <th>Incident Number</th>
      <th>CAD Number</th>
      <th>...</th>
      <th>SF Find Neighborhoods</th>
      <th>Current Police Districts</th>
      <th>Current Supervisor Districts</th>
      <th>Analysis Neighborhoods</th>
      <th>HSOC Zones as of 2018-06-05</th>
      <th>OWED Public Spaces</th>
      <th>Central Market/Tenderloin Boundary Polygon - Updated</th>
      <th>Parks Alliance CPSI (27+TL sites)</th>
      <th>ESNCAG - Boundary File</th>
      <th>Areas of Vulnerability, 2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>2020/02/03 02:45:00 PM</td>
      <td>2020/02/03</td>
      <td>14:45</td>
      <td>2020</td>
      <td>Monday</td>
      <td>2020/02/03 05:50:00 PM</td>
      <td>89881675000</td>
      <td>898816</td>
      <td>200085557</td>
      <td>200342870.0</td>
      <td>...</td>
      <td>41.0</td>
      <td>10.0</td>
      <td>8.0</td>
      <td>16.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2020/02/03 03:45:00 AM</td>
      <td>2020/02/03</td>
      <td>03:45</td>
      <td>2020</td>
      <td>Monday</td>
      <td>2020/02/03 03:45:00 AM</td>
      <td>89860711012</td>
      <td>898607</td>
      <td>200083749</td>
      <td>200340316.0</td>
      <td>...</td>
      <td>53.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2020/02/03 10:00:00 AM</td>
      <td>2020/02/03</td>
      <td>10:00</td>
      <td>2020</td>
      <td>Monday</td>
      <td>2020/02/03 10:06:00 AM</td>
      <td>89867264015</td>
      <td>898672</td>
      <td>200084060</td>
      <td>200340808.0</td>
      <td>...</td>
      <td>19.0</td>
      <td>5.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>35.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2020/01/19 05:12:00 PM</td>
      <td>2020/01/19</td>
      <td>17:12</td>
      <td>2020</td>
      <td>Sunday</td>
      <td>2020/02/01 01:01:00 PM</td>
      <td>89863571000</td>
      <td>898635</td>
      <td>206024187</td>
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
      <td>4</td>
      <td>2020/01/05 12:00:00 AM</td>
      <td>2020/01/05</td>
      <td>00:00</td>
      <td>2020</td>
      <td>Sunday</td>
      <td>2020/02/03 04:09:00 PM</td>
      <td>89877368020</td>
      <td>898773</td>
      <td>200085193</td>
      <td>200342341.0</td>
      <td>...</td>
      <td>103.0</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>30.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 36 columns</p>
</div>




```python
cols = list(incidents.columns.values)
cols
```




    ['Incident Datetime',
     'Incident Date',
     'Incident Time',
     'Incident Year',
     'Incident Day of Week',
     'Report Datetime',
     'Row ID',
     'Incident ID',
     'Incident Number',
     'CAD Number',
     'Report Type Code',
     'Report Type Description',
     'Filed Online',
     'Incident Code',
     'Incident Category',
     'Incident Subcategory',
     'Incident Description',
     'Resolution',
     'Intersection',
     'CNN',
     'Police District',
     'Analysis Neighborhood',
     'Supervisor District',
     'Latitude',
     'Longitude',
     'point',
     'SF Find Neighborhoods',
     'Current Police Districts',
     'Current Supervisor Districts',
     'Analysis Neighborhoods',
     'HSOC Zones as of 2018-06-05',
     'OWED Public Spaces',
     'Central Market/Tenderloin Boundary Polygon - Updated',
     'Parks Alliance CPSI (27+TL sites)',
     'ESNCAG - Boundary File',
     'Areas of Vulnerability, 2016']



I will also want to include some information about the type of criminial incident.  I will take a quick look at the "Incident Category" column to see what kind of information it contains.


```python
incidents['Incident Category'].head(10)
```




    0                              Missing Person
    1                             Stolen Property
    2                                Non-Criminal
    3                               Lost Property
    4                 Miscellaneous Investigation
    5                 Miscellaneous Investigation
    6    Offences Against The Family And Children
    7                               Larceny Theft
    8                         Other Miscellaneous
    9                                Non-Criminal
    Name: Incident Category, dtype: object




```python
incidents.shape
```




    (341327, 36)



This is a huge file; there is no way I could post 340,000 + incidents on a single map.  So I will narrow it down to just show a couple hundred of the most recent incidents.  But it appears the file is not sorted in time order, so I will sort it now, descending by the DateTime column.


```python
incidents.sort_values(by=['Incident Datetime'], inplace=True, ascending=False)
```


```python
incidents.head()
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
      <th>Incident Datetime</th>
      <th>Incident Date</th>
      <th>Incident Time</th>
      <th>Incident Year</th>
      <th>Incident Day of Week</th>
      <th>Report Datetime</th>
      <th>Row ID</th>
      <th>Incident ID</th>
      <th>Incident Number</th>
      <th>CAD Number</th>
      <th>...</th>
      <th>SF Find Neighborhoods</th>
      <th>Current Police Districts</th>
      <th>Current Supervisor Districts</th>
      <th>Analysis Neighborhoods</th>
      <th>HSOC Zones as of 2018-06-05</th>
      <th>OWED Public Spaces</th>
      <th>Central Market/Tenderloin Boundary Polygon - Updated</th>
      <th>Parks Alliance CPSI (27+TL sites)</th>
      <th>ESNCAG - Boundary File</th>
      <th>Areas of Vulnerability, 2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>28731</td>
      <td>2020/04/29 12:55:00 AM</td>
      <td>2020/04/29</td>
      <td>00:55</td>
      <td>2020</td>
      <td>Wednesday</td>
      <td>2020/04/29 12:55:00 AM</td>
      <td>92313007041</td>
      <td>923130</td>
      <td>200265317</td>
      <td>201200104.0</td>
      <td>...</td>
      <td>21.0</td>
      <td>5.0</td>
      <td>10.0</td>
      <td>36.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>28912</td>
      <td>2020/04/29 12:34:00 AM</td>
      <td>2020/04/29</td>
      <td>00:34</td>
      <td>2020</td>
      <td>Wednesday</td>
      <td>2020/04/29 12:35:00 AM</td>
      <td>92314312026</td>
      <td>923143</td>
      <td>200266070</td>
      <td>201200080.0</td>
      <td>...</td>
      <td>88.0</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>28680</td>
      <td>2020/04/29 12:34:00 AM</td>
      <td>2020/04/29</td>
      <td>00:34</td>
      <td>2020</td>
      <td>Wednesday</td>
      <td>2020/04/29 12:35:00 AM</td>
      <td>92314304083</td>
      <td>923143</td>
      <td>200266070</td>
      <td>201200080.0</td>
      <td>...</td>
      <td>88.0</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>28771</td>
      <td>2020/04/29 12:34:00 AM</td>
      <td>2020/04/29</td>
      <td>00:34</td>
      <td>2020</td>
      <td>Wednesday</td>
      <td>2020/04/29 12:35:00 AM</td>
      <td>92314304011</td>
      <td>923143</td>
      <td>200266070</td>
      <td>201200080.0</td>
      <td>...</td>
      <td>88.0</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
    </tr>
    <tr>
      <td>28927</td>
      <td>2020/04/29 12:24:00 PM</td>
      <td>2020/04/29</td>
      <td>12:24</td>
      <td>2020</td>
      <td>Wednesday</td>
      <td>2020/04/29 12:24:00 PM</td>
      <td>92331606362</td>
      <td>923316</td>
      <td>200267056</td>
      <td>201201480.0</td>
      <td>...</td>
      <td>17.0</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>13.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 36 columns</p>
</div>



Now it looks like it's in the right order, so I will extract just the most recent 200 incidents into a new dataframe.


```python
incidents200 = incidents.iloc[0:200, :]
```


```python
incidents200.shape
```




    (200, 36)



And now I will get rid of any rows with null values in the latitude or longitude columns.


```python
incidents200 = incidents200[(incidents200['Latitude'].notna()) | (incidents200['Longitude'].notna())]
```


```python
incidents200.shape
```




    (190, 36)



### Folium Maps: 
Use the folium library to make a basemap centered on the city of San Francisco.


```python
# San Francisco lat/long 37.7775, -122.416389
# https://en.wikipedia.org/wiki/San_Francisco
latitude = 37.7775
longitude = -122.416389
```


```python
# create map and display it
sf_map = folium.Map(location=[latitude, longitude], zoom_start=10)

# display the map of San Francisco
sf_map
```

I haven't figured out how to make the interactive folium map display here yet, so in the meantime, here's a screenshot of the map from the Jupyter Notebook:

![png](/images/sanfran_map1.PNG)<!-- -->

Next, I will create a map layer containing the most recent incident locations:


```python
# create a feature group layer for the incidents
incident_group = folium.map.FeatureGroup()

# use a for loop to add the 200 incidents to the feature group layer
for lat, long, in zip(incidents200.Latitude, incidents200.Longitude):
    incident_group.add_child(
        folium.CircleMarker(
            [lat, long],
            radius=4,
            color='red',
            fill=True,
            fill_color='green',
            fill_opacity=0.5
        )
    )
# add the incidents feature group layer to the map
sf_map.add_child(incident_group)
```


Again, the embedded folium map isn't working here in github pages, but it works fine in the Jupyter Notebook, so here's a screenshot:

![png](/images/sanfran_map2.PNG)<!-- -->

Now I can see where the crimes took place, but I can't see any information about each crime.  Nex I will add information about the posted incidents using the contents of the "Incident Category" column.  That space in the variable name might cause problems, so I will shorten the name of that column from "Incident Category" to just "Category". Then I will add the category description text as a feature on the map that will "popup" when I click on the marker for a particular incident.


```python
# shorten the name of incident category column
incidents200.rename(columns={'Incident Category': 'Category'}, inplace=True)
```


```python
sf_map = folium.Map(location=[latitude, longitude], zoom_start=12)

for lat, lng, label in zip(incidents200.Latitude, incidents200.Longitude, incidents200.Category):
    folium.CircleMarker(
        [lat, lng],
        radius=5, # define how big you want the circle markers to be
        color='red',
        fill=True,
        popup=folium.Popup(label),
        #popup=label, ****this didn't work for some reason; had to replace with above line
        fill_color='green',
        fill_opacity=0.6).add_to(sf_map)

sf_map
```

![png](/images/sanfran_map3.PNG)<!-- -->

Now when I click on a marker, the text from the Category field shows in a popup on the map.  That's all I have on this topic for now, but maybe I will think of something else to add to this map in the future.  Thanks for reading this!
