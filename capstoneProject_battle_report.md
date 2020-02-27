
# Capstone Project - The Battle of Neighborhoods | Coursera

## ToC

* [Introduction](#intro)
* [Data](#data)
* [Methodology and Analysis](#analysis)
* [Results and Discussion](#result)
* [Conclusion](#conclusion)

## Introduction <span id="intro"></span>
This report is for Capstone Project in Coursera [Applied Data Science Capstone](https://www.coursera.org/learn/applied-data-science-capstone).

### Business Problem
Assume that we want to open a successful Chinese restaurant in a U.S. city.

To achieve this goal, first of all we need to choose a city in U.S. Since it is a Chinese restaurant, a multi-cultural metropolis sounds promising. So we would like to select San Francisco.
![SF Map](https://ljmoore.files.wordpress.com/2013/02/san-francisco-autofill-map1.jpg?w=800)

The second step is to choose an ideal location of the restaurant in the city. We can leverage the Foursquare location data and machine learning skills to learn which are the most popular areas for Chinese restaurants. 

### Who would be interested in this project?
Someone who has interest in opening a successful business in a U.S. city.


## Data  <span id="data"></span>

Our target is to find the most popular areas for Chinese restaurants in San Francisco.

So the first thing is to divide San Francisco city into certain areas. We can do that based on on-line postal code (ZIP) information.

The second is to get necessary location data in all the areas utilizing Foursquare API. After data cleaning, we can use the data to cluster the areas into different types. Then we pick up the cluster where Chinese restaurants are more popular. 


## Methodology and Analysis <span id="analysis"></span>

### A Basic View on San Francisco City


```python
import pandas as pd
import numpy as np
```


```python
# get the coordinate of city
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

city = "San Francisco City"

geolocator = Nominatim(user_agent="city_explorer")
location = geolocator.geocode(city)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinate of San Francisco City are {}, {}.'.format(latitude, longitude))
```

    The geograpical coordinate of San Francisco City are 37.7790262, -122.4199061.
    


```python
# get the postal code and coordinate info
#   from united-states.postcode.info
from urllib.request import urlopen
import re

url = "http://united-states.postcode.info/california/san-francisco"
html = urlopen(url).read().decode("utf-8")

# example: California, <a href="/p/94102">94102</a> San Francisco, San Francisco, GPS coordinates: 37.7813,-122.4167
res = re.findall(r'>(\d+)</a>.* GPS coordinates: (.+),(.+)\r', html)
print("Coordinates: ", res[0])
```

    Coordinates:  ('94101', '37.7848', '-122.7278')
    


```python
# create the postal code and coordinate dataframe

hash_sf = {'PostalCode': [], 
           'Latitude': [], 
           'Longitude': []
          }

for row in res:
    hash_sf['PostalCode'].append(int(row[0]))
    hash_sf['Latitude'].append(float(row[1]))
    hash_sf['Longitude'].append(float(row[2]))


column_names = ['PostalCode', 'Latitude', 'Longitude'] 

# instantiate the dataframe
df_sf = pd.DataFrame(columns=column_names)
df_sf = pd.DataFrame(hash_sf)
df_sf.head()
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
      <th>PostalCode</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>94101</td>
      <td>37.7848</td>
      <td>-122.7278</td>
    </tr>
    <tr>
      <th>1</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
    </tr>
    <tr>
      <th>2</th>
      <td>94103</td>
      <td>37.7725</td>
      <td>-122.4147</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94104</td>
      <td>37.7915</td>
      <td>-122.4018</td>
    </tr>
    <tr>
      <th>4</th>
      <td>94105</td>
      <td>37.7864</td>
      <td>-122.3892</td>
    </tr>
  </tbody>
</table>
</div>




```python
# remove abnormal postal codes
# in the dataframe, multiple codes share the same coodinate: [37.7848, -122.7278]
# in addition, this location is in the gulf
# so we remove these abnormal codes

df_sf = df_sf[df_sf['Longitude']!=-122.7278].reset_index(drop=True)
df_sf
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
      <th>PostalCode</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
    </tr>
    <tr>
      <th>1</th>
      <td>94103</td>
      <td>37.7725</td>
      <td>-122.4147</td>
    </tr>
    <tr>
      <th>2</th>
      <td>94104</td>
      <td>37.7915</td>
      <td>-122.4018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94105</td>
      <td>37.7864</td>
      <td>-122.3892</td>
    </tr>
    <tr>
      <th>4</th>
      <td>94107</td>
      <td>37.7621</td>
      <td>-122.3971</td>
    </tr>
    <tr>
      <th>5</th>
      <td>94108</td>
      <td>37.7929</td>
      <td>-122.4079</td>
    </tr>
    <tr>
      <th>6</th>
      <td>94109</td>
      <td>37.7917</td>
      <td>-122.4186</td>
    </tr>
    <tr>
      <th>7</th>
      <td>94110</td>
      <td>37.7509</td>
      <td>-122.4153</td>
    </tr>
    <tr>
      <th>8</th>
      <td>94111</td>
      <td>37.7974</td>
      <td>-122.4001</td>
    </tr>
    <tr>
      <th>9</th>
      <td>94112</td>
      <td>37.7195</td>
      <td>-122.4411</td>
    </tr>
    <tr>
      <th>10</th>
      <td>94114</td>
      <td>37.7587</td>
      <td>-122.4330</td>
    </tr>
    <tr>
      <th>11</th>
      <td>94115</td>
      <td>37.7856</td>
      <td>-122.4358</td>
    </tr>
    <tr>
      <th>12</th>
      <td>94116</td>
      <td>37.7441</td>
      <td>-122.4863</td>
    </tr>
    <tr>
      <th>13</th>
      <td>94117</td>
      <td>37.7712</td>
      <td>-122.4413</td>
    </tr>
    <tr>
      <th>14</th>
      <td>94118</td>
      <td>37.7812</td>
      <td>-122.4614</td>
    </tr>
    <tr>
      <th>15</th>
      <td>94121</td>
      <td>37.7786</td>
      <td>-122.4892</td>
    </tr>
    <tr>
      <th>16</th>
      <td>94122</td>
      <td>37.7593</td>
      <td>-122.4836</td>
    </tr>
    <tr>
      <th>17</th>
      <td>94123</td>
      <td>37.7999</td>
      <td>-122.4342</td>
    </tr>
    <tr>
      <th>18</th>
      <td>94124</td>
      <td>37.7309</td>
      <td>-122.3886</td>
    </tr>
    <tr>
      <th>19</th>
      <td>94127</td>
      <td>37.7354</td>
      <td>-122.4571</td>
    </tr>
    <tr>
      <th>20</th>
      <td>94128</td>
      <td>37.6216</td>
      <td>-122.3929</td>
    </tr>
    <tr>
      <th>21</th>
      <td>94129</td>
      <td>37.8005</td>
      <td>-122.4650</td>
    </tr>
    <tr>
      <th>22</th>
      <td>94130</td>
      <td>37.8231</td>
      <td>-122.3693</td>
    </tr>
    <tr>
      <th>23</th>
      <td>94131</td>
      <td>37.7450</td>
      <td>-122.4383</td>
    </tr>
    <tr>
      <th>24</th>
      <td>94132</td>
      <td>37.7211</td>
      <td>-122.4754</td>
    </tr>
    <tr>
      <th>25</th>
      <td>94133</td>
      <td>37.8002</td>
      <td>-122.4091</td>
    </tr>
    <tr>
      <th>26</th>
      <td>94134</td>
      <td>37.7190</td>
      <td>-122.4096</td>
    </tr>
    <tr>
      <th>27</th>
      <td>94143</td>
      <td>37.7631</td>
      <td>-122.4586</td>
    </tr>
    <tr>
      <th>28</th>
      <td>94158</td>
      <td>37.7694</td>
      <td>-122.3867</td>
    </tr>
    <tr>
      <th>29</th>
      <td>94199</td>
      <td>37.7750</td>
      <td>-122.4183</td>
    </tr>
  </tbody>
</table>
</div>




```python
# render the map for areas marked by postal code
import folium

map_sf = folium.Map(location=[latitude, longitude], zoom_start=12)

# add markers to map
for lat, lng, pcode in zip(df_sf['Latitude'], df_sf['Longitude'], df_sf['PostalCode']):
    label = '{}'.format(pcode)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_sf)
    
map_sf
```

In case the rendered map is not shown, please see this snapshot.  
![sf_postalcode](sf_postalcode.png)

### Request Data Using Foursquare API


```python
# define foursqure info
CLIENT_ID = 'DTM4ERJ5PLBBUI13LZJO44E0Z2XV1MMXNSZ4WFXDI2S0CXWR' # your Foursquare ID
CLIENT_SECRET = 'ONQOLBNCXJKP3OF51AWOD1RZC4GYFQRGOFMGZIVEHJKE4X5P' # your Foursquare Secret
VERSION = '20180605' # Foursquare API version
```


```python
# define API request parameters and function
import requests # library to handle requests

LIMIT = 100
radius = 1000

def getNearbyVenues(codes, latitudes, longitudes, radius=500):
    
    venues_list=[]
    for code, lat, lng in zip(codes, latitudes, longitudes):
        print(code)
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            LIMIT)
            
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        
        # return only relevant information for each nearby venue
        venues_list.append([(
            code, 
            lat, 
            lng, 
            v['venue']['name'], 
            v['venue']['location']['lat'], 
            v['venue']['location']['lng'],  
            v['venue']['categories'][0]['name']) for v in results])

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['PostalCode', 
                  'PostalCode Latitude', 
                  'PostalCode Longitude', 
                  'Venue', 
                  'Venue Latitude', 
                  'Venue Longitude', 
                  'Venue Category']
    
    return(nearby_venues)
```


```python
# API request
sf_venues = getNearbyVenues(df_sf['PostalCode'],
                            df_sf['Latitude'], 
                            df_sf['Longitude'],
                            radius)
sf_venues.head()
```

    94102
    94103
    94104
    94105
    94107
    94108
    94109
    94110
    94111
    94112
    94114
    94115
    94116
    94117
    94118
    94121
    94122
    94123
    94124
    94127
    94128
    94129
    94130
    94131
    94132
    94133
    94134
    94143
    94158
    94199
    




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
      <th>PostalCode</th>
      <th>PostalCode Latitude</th>
      <th>PostalCode Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Asian Art Museum</td>
      <td>37.780178</td>
      <td>-122.416505</td>
      <td>Art Museum</td>
    </tr>
    <tr>
      <th>1</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Philz Coffee</td>
      <td>37.781433</td>
      <td>-122.417073</td>
      <td>Coffee Shop</td>
    </tr>
    <tr>
      <th>2</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Ales Unlimited: Beer Basement</td>
      <td>37.782751</td>
      <td>-122.415656</td>
      <td>Beer Bar</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Saigon Sandwich</td>
      <td>37.783084</td>
      <td>-122.417650</td>
      <td>Sandwich Place</td>
    </tr>
    <tr>
      <th>4</th>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Whitechapel</td>
      <td>37.782230</td>
      <td>-122.418884</td>
      <td>Cocktail Bar</td>
    </tr>
  </tbody>
</table>
</div>




```python
# save the dataframe
csv_file = "sf_venues.csv"
sf_venues.to_csv(csv_file)
```


```python
# restore the dataframe
csv_file = "sf_venues.csv"
sf_venues = pd.read_csv(csv_file)

print(sf_venues.shape)
sf_venues.head()
```

    (2573, 8)
    




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
      <th>Unnamed: 0</th>
      <th>PostalCode</th>
      <th>PostalCode Latitude</th>
      <th>PostalCode Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Asian Art Museum</td>
      <td>37.780178</td>
      <td>-122.416505</td>
      <td>Art Museum</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Philz Coffee</td>
      <td>37.781433</td>
      <td>-122.417073</td>
      <td>Coffee Shop</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Ales Unlimited: Beer Basement</td>
      <td>37.782751</td>
      <td>-122.415656</td>
      <td>Beer Bar</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Saigon Sandwich</td>
      <td>37.783084</td>
      <td>-122.417650</td>
      <td>Sandwich Place</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>94102</td>
      <td>37.7813</td>
      <td>-122.4167</td>
      <td>Whitechapel</td>
      <td>37.782230</td>
      <td>-122.418884</td>
      <td>Cocktail Bar</td>
    </tr>
  </tbody>
</table>
</div>



### Data Analysis


```python
# one hot encoding
sf_onehot = pd.get_dummies(sf_venues[['Venue Category']], prefix="", prefix_sep="")

# add code column back to dataframe
sf_onehot['PostalCode'] = sf_venues['PostalCode'] 

# move code column to the first column
fixed_columns = [sf_onehot.columns[-1]] + list(sf_onehot.columns[:-1])
sf_onehot = sf_onehot[fixed_columns]

print(sf_onehot.shape)
sf_onehot.head()
```

    (2573, 323)
    




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
      <th>PostalCode</th>
      <th>Acai House</th>
      <th>Accessories Store</th>
      <th>Afghan Restaurant</th>
      <th>African Restaurant</th>
      <th>Airport</th>
      <th>Airport Lounge</th>
      <th>Airport Service</th>
      <th>Alternative Healer</th>
      <th>American Restaurant</th>
      <th>...</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Vineyard</th>
      <th>Wagashi Place</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>94102</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>94102</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>94102</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94102</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>94102</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 323 columns</p>
</div>




```python
# group by postal code
sf_grouped = sf_onehot.groupby('PostalCode').mean().reset_index()

sf_grouped
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
      <th>PostalCode</th>
      <th>Acai House</th>
      <th>Accessories Store</th>
      <th>Afghan Restaurant</th>
      <th>African Restaurant</th>
      <th>Airport</th>
      <th>Airport Lounge</th>
      <th>Airport Service</th>
      <th>Alternative Healer</th>
      <th>American Restaurant</th>
      <th>...</th>
      <th>Video Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Vineyard</th>
      <th>Wagashi Place</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>94102</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>94103</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.030000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>94104</td>
      <td>0.01</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0100</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94105</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.01</td>
      <td>0.020000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>94107</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0200</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>94108</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>94109</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.0100</td>
      <td>0.01</td>
      <td>0.0000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>94110</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>94111</td>
      <td>0.01</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.040000</td>
      <td>0.0100</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>94112</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.011364</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.034091</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>94114</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0100</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>94115</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>94116</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.012346</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.012346</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>94117</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>94118</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.030000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0200</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>94121</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.040000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0100</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>16</th>
      <td>94122</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.060000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>94123</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.030000</td>
      <td>0.0100</td>
      <td>0.00</td>
      <td>0.0100</td>
      <td>0.030000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>94124</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.022727</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>94127</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.018868</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.018868</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>20</th>
      <td>94128</td>
      <td>0.00</td>
      <td>0.022989</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.011494</td>
      <td>0.08046</td>
      <td>0.08046</td>
      <td>0.00</td>
      <td>0.011494</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.022989</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.022989</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.011494</td>
    </tr>
    <tr>
      <th>21</th>
      <td>94129</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.016667</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>22</th>
      <td>94130</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.033333</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>23</th>
      <td>94131</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.025000</td>
      <td>...</td>
      <td>0.0125</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0125</td>
      <td>0.00</td>
      <td>0.0125</td>
      <td>0.012500</td>
    </tr>
    <tr>
      <th>24</th>
      <td>94132</td>
      <td>0.00</td>
      <td>0.012195</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.012195</td>
    </tr>
    <tr>
      <th>25</th>
      <td>94133</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>26</th>
      <td>94134</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.050000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>27</th>
      <td>94143</td>
      <td>0.00</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
    <tr>
      <th>28</th>
      <td>94158</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.020833</td>
      <td>0.0000</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.020833</td>
    </tr>
    <tr>
      <th>29</th>
      <td>94199</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.010000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.050000</td>
      <td>0.0100</td>
      <td>0.00</td>
      <td>0.0000</td>
      <td>0.010000</td>
    </tr>
  </tbody>
</table>
<p>30 rows × 323 columns</p>
</div>



### Cluster Postal Code Areas


```python
# import k-means from clustering stage
from sklearn.cluster import KMeans

# set number of clusters
k = 5

sf_grouped_clustering = sf_grouped.drop('PostalCode', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=k, random_state=0).fit(sf_grouped_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10] 
```




    array([0, 0, 0, 0, 3, 0, 0, 3, 0, 3])




```python
# add coordinate info
sf_merged = sf_grouped.join(df_sf.set_index('PostalCode'), on='PostalCode')

# add clustering labels
sf_merged.insert(0, 'ClusterLabel', kmeans.labels_)

print(sf_merged.shape)
sf_merged.head() # check the last columns!
```

    (30, 326)
    




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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>Acai House</th>
      <th>Accessories Store</th>
      <th>Afghan Restaurant</th>
      <th>African Restaurant</th>
      <th>Airport</th>
      <th>Airport Lounge</th>
      <th>Airport Service</th>
      <th>Alternative Healer</th>
      <th>...</th>
      <th>Vineyard</th>
      <th>Wagashi Place</th>
      <th>Whisky Bar</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>94102</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>...</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>37.7813</td>
      <td>-122.4167</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>94103</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>...</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.03</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>37.7725</td>
      <td>-122.4147</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>94104</td>
      <td>0.01</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>...</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>37.7915</td>
      <td>-122.4018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>94105</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>...</td>
      <td>0.01</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.01</td>
      <td>37.7864</td>
      <td>-122.3892</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>94107</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>...</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.02</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>37.7621</td>
      <td>-122.3971</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 326 columns</p>
</div>




```python
# render the map for clusters marked by postal code
import folium
# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=12)

# set color scheme for the clusters
x = np.arange(k)
ys = [i + x + (i*x)**2 for i in range(k)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, code, cluster in zip(sf_merged['Latitude'], sf_merged['Longitude'], sf_merged['PostalCode'], sf_merged['ClusterLabel']):
    label = folium.Popup('Code ' + str(code) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```


In case the rendered map is not shown, please see this snapshot.  
![sf_cluster](sf_cluster.png)

### Examine Area Top Venues


```python
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    
    return row_categories_sorted.index.values[0:num_top_venues]
```


```python
# sort top 10 venues for each area
num_top_venues = 10

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['PostalCode']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
sf_venues_sorted = pd.DataFrame(columns=columns)
sf_venues_sorted['PostalCode'] = sf_grouped['PostalCode']

for ind in np.arange(sf_grouped.shape[0]):
    sf_venues_sorted.iloc[ind, 1:] = return_most_common_venues(sf_grouped.iloc[ind, :], num_top_venues)

sf_venues_sorted.head()
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
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>94102</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Vietnamese Restaurant</td>
      <td>Beer Bar</td>
      <td>Cocktail Bar</td>
      <td>Music Venue</td>
      <td>Sushi Restaurant</td>
      <td>Sandwich Place</td>
      <td>Performing Arts Venue</td>
      <td>Speakeasy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>94103</td>
      <td>Coffee Shop</td>
      <td>Cocktail Bar</td>
      <td>Beer Bar</td>
      <td>Wine Bar</td>
      <td>New American Restaurant</td>
      <td>Motorcycle Shop</td>
      <td>Gym</td>
      <td>Gay Bar</td>
      <td>Art Gallery</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>94104</td>
      <td>Coffee Shop</td>
      <td>Men's Store</td>
      <td>Boutique</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Park</td>
      <td>Gym / Fitness Center</td>
      <td>Gym</td>
      <td>Shoe Store</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94105</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Gym</td>
      <td>Scenic Lookout</td>
      <td>Park</td>
      <td>Art Gallery</td>
      <td>Seafood Restaurant</td>
      <td>New American Restaurant</td>
      <td>Burger Joint</td>
      <td>Cycle Studio</td>
    </tr>
    <tr>
      <th>4</th>
      <td>94107</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Brewery</td>
      <td>Park</td>
      <td>Sushi Restaurant</td>
      <td>Breakfast Spot</td>
      <td>Sandwich Place</td>
      <td>Gym</td>
      <td>Bakery</td>
      <td>Bar</td>
    </tr>
  </tbody>
</table>
</div>




```python
# add clustering labels
sf_venues_sorted.insert(0, 'ClusterLabel', kmeans.labels_)

print(sf_venues_sorted.shape)
sf_venues_sorted.head()
```

    (30, 12)
    




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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>94102</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Vietnamese Restaurant</td>
      <td>Beer Bar</td>
      <td>Cocktail Bar</td>
      <td>Music Venue</td>
      <td>Sushi Restaurant</td>
      <td>Sandwich Place</td>
      <td>Performing Arts Venue</td>
      <td>Speakeasy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>94103</td>
      <td>Coffee Shop</td>
      <td>Cocktail Bar</td>
      <td>Beer Bar</td>
      <td>Wine Bar</td>
      <td>New American Restaurant</td>
      <td>Motorcycle Shop</td>
      <td>Gym</td>
      <td>Gay Bar</td>
      <td>Art Gallery</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>94104</td>
      <td>Coffee Shop</td>
      <td>Men's Store</td>
      <td>Boutique</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Park</td>
      <td>Gym / Fitness Center</td>
      <td>Gym</td>
      <td>Shoe Store</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>94105</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Gym</td>
      <td>Scenic Lookout</td>
      <td>Park</td>
      <td>Art Gallery</td>
      <td>Seafood Restaurant</td>
      <td>New American Restaurant</td>
      <td>Burger Joint</td>
      <td>Cycle Studio</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>94107</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Brewery</td>
      <td>Park</td>
      <td>Sushi Restaurant</td>
      <td>Breakfast Spot</td>
      <td>Sandwich Place</td>
      <td>Gym</td>
      <td>Bakery</td>
      <td>Bar</td>
    </tr>
  </tbody>
</table>
</div>



### Examine Clusters <span id="examine_cluster"></span>


```python
# CLuster 0
sf_venues_sorted.loc[sf_venues_sorted['ClusterLabel']==0]
# In this cluster, The most common venues are coffee shops & bars
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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>94102</td>
      <td>Coffee Shop</td>
      <td>Theater</td>
      <td>Vietnamese Restaurant</td>
      <td>Beer Bar</td>
      <td>Cocktail Bar</td>
      <td>Music Venue</td>
      <td>Sushi Restaurant</td>
      <td>Sandwich Place</td>
      <td>Performing Arts Venue</td>
      <td>Speakeasy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>94103</td>
      <td>Coffee Shop</td>
      <td>Cocktail Bar</td>
      <td>Beer Bar</td>
      <td>Wine Bar</td>
      <td>New American Restaurant</td>
      <td>Motorcycle Shop</td>
      <td>Gym</td>
      <td>Gay Bar</td>
      <td>Art Gallery</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>94104</td>
      <td>Coffee Shop</td>
      <td>Men's Store</td>
      <td>Boutique</td>
      <td>Seafood Restaurant</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Park</td>
      <td>Gym / Fitness Center</td>
      <td>Gym</td>
      <td>Shoe Store</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>94105</td>
      <td>Coffee Shop</td>
      <td>Café</td>
      <td>Gym</td>
      <td>Scenic Lookout</td>
      <td>Park</td>
      <td>Art Gallery</td>
      <td>Seafood Restaurant</td>
      <td>New American Restaurant</td>
      <td>Burger Joint</td>
      <td>Cycle Studio</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0</td>
      <td>94108</td>
      <td>Coffee Shop</td>
      <td>Hotel</td>
      <td>Men's Store</td>
      <td>Boutique</td>
      <td>Restaurant</td>
      <td>Shoe Store</td>
      <td>Cocktail Bar</td>
      <td>Church</td>
      <td>Szechuan Restaurant</td>
      <td>New American Restaurant</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0</td>
      <td>94109</td>
      <td>Coffee Shop</td>
      <td>Vietnamese Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Grocery Store</td>
      <td>Steakhouse</td>
      <td>Wine Bar</td>
      <td>American Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>Clothing Store</td>
      <td>Bakery</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>94111</td>
      <td>Coffee Shop</td>
      <td>Cocktail Bar</td>
      <td>Wine Bar</td>
      <td>New American Restaurant</td>
      <td>Men's Store</td>
      <td>Seafood Restaurant</td>
      <td>Scenic Lookout</td>
      <td>Dessert Shop</td>
      <td>Restaurant</td>
      <td>Italian Restaurant</td>
    </tr>
    <tr>
      <th>29</th>
      <td>0</td>
      <td>94199</td>
      <td>Cocktail Bar</td>
      <td>Wine Bar</td>
      <td>Coffee Shop</td>
      <td>Performing Arts Venue</td>
      <td>Dessert Shop</td>
      <td>New American Restaurant</td>
      <td>Beer Bar</td>
      <td>French Restaurant</td>
      <td>Marijuana Dispensary</td>
      <td>Optical Shop</td>
    </tr>
  </tbody>
</table>
</div>




```python
# CLuster 1
sf_venues_sorted.loc[sf_venues_sorted['ClusterLabel']==1]
# In this cluster, The most common venues are restaurants
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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>94116</td>
      <td>Chinese Restaurant</td>
      <td>Sandwich Place</td>
      <td>Park</td>
      <td>Dumpling Restaurant</td>
      <td>Pizza Place</td>
      <td>Bubble Tea Shop</td>
      <td>Sushi Restaurant</td>
      <td>Korean Restaurant</td>
      <td>Café</td>
      <td>Bus Stop</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1</td>
      <td>94118</td>
      <td>Japanese Restaurant</td>
      <td>Bakery</td>
      <td>Thai Restaurant</td>
      <td>Korean Restaurant</td>
      <td>Italian Restaurant</td>
      <td>Coffee Shop</td>
      <td>Chinese Restaurant</td>
      <td>Burmese Restaurant</td>
      <td>Sushi Restaurant</td>
      <td>Vietnamese Restaurant</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1</td>
      <td>94121</td>
      <td>Café</td>
      <td>Chinese Restaurant</td>
      <td>Grocery Store</td>
      <td>Sushi Restaurant</td>
      <td>Korean Restaurant</td>
      <td>Vietnamese Restaurant</td>
      <td>Playground</td>
      <td>Pizza Place</td>
      <td>Bakery</td>
      <td>Japanese Restaurant</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1</td>
      <td>94122</td>
      <td>Chinese Restaurant</td>
      <td>Bubble Tea Shop</td>
      <td>Vietnamese Restaurant</td>
      <td>Bakery</td>
      <td>Japanese Restaurant</td>
      <td>Bank</td>
      <td>Dim Sum Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Deli / Bodega</td>
      <td>Bar</td>
    </tr>
  </tbody>
</table>
</div>




```python
# CLuster 2
sf_venues_sorted.loc[sf_venues_sorted['ClusterLabel']==2]
# In this cluster, The most common venues are food trucks and sports
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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>22</th>
      <td>2</td>
      <td>94130</td>
      <td>Food Truck</td>
      <td>Athletics &amp; Sports</td>
      <td>Park</td>
      <td>Rugby Pitch</td>
      <td>Baseball Field</td>
      <td>Music Venue</td>
      <td>Breakfast Spot</td>
      <td>History Museum</td>
      <td>Bus Station</td>
      <td>American Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python
# CLuster 3
sf_venues_sorted.loc[sf_venues_sorted['ClusterLabel']==3]
# In this cluster, The most common venues are parks and pizza places
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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>94107</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Brewery</td>
      <td>Park</td>
      <td>Sushi Restaurant</td>
      <td>Breakfast Spot</td>
      <td>Sandwich Place</td>
      <td>Gym</td>
      <td>Bakery</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3</td>
      <td>94110</td>
      <td>Mexican Restaurant</td>
      <td>Grocery Store</td>
      <td>Coffee Shop</td>
      <td>Latin American Restaurant</td>
      <td>New American Restaurant</td>
      <td>Dive Bar</td>
      <td>Italian Restaurant</td>
      <td>Cocktail Bar</td>
      <td>Bar</td>
      <td>Bakery</td>
    </tr>
    <tr>
      <th>9</th>
      <td>3</td>
      <td>94112</td>
      <td>Mexican Restaurant</td>
      <td>Pizza Place</td>
      <td>Liquor Store</td>
      <td>Bakery</td>
      <td>Latin American Restaurant</td>
      <td>Sandwich Place</td>
      <td>Vietnamese Restaurant</td>
      <td>Bank</td>
      <td>Park</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>10</th>
      <td>3</td>
      <td>94114</td>
      <td>Gay Bar</td>
      <td>Park</td>
      <td>Thai Restaurant</td>
      <td>Coffee Shop</td>
      <td>Bakery</td>
      <td>New American Restaurant</td>
      <td>Café</td>
      <td>Japanese Restaurant</td>
      <td>Deli / Bodega</td>
      <td>Pizza Place</td>
    </tr>
    <tr>
      <th>11</th>
      <td>3</td>
      <td>94115</td>
      <td>Cosmetics Shop</td>
      <td>Bakery</td>
      <td>Gift Shop</td>
      <td>Ice Cream Shop</td>
      <td>Boutique</td>
      <td>Sandwich Place</td>
      <td>Tea Room</td>
      <td>Yoga Studio</td>
      <td>Park</td>
      <td>Spa</td>
    </tr>
    <tr>
      <th>13</th>
      <td>3</td>
      <td>94117</td>
      <td>Park</td>
      <td>Coffee Shop</td>
      <td>Gift Shop</td>
      <td>Pizza Place</td>
      <td>Ice Cream Shop</td>
      <td>Yoga Studio</td>
      <td>Liquor Store</td>
      <td>Bakery</td>
      <td>Sushi Restaurant</td>
      <td>Bookstore</td>
    </tr>
    <tr>
      <th>17</th>
      <td>3</td>
      <td>94123</td>
      <td>Italian Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>French Restaurant</td>
      <td>Cosmetics Shop</td>
      <td>Yoga Studio</td>
      <td>Burger Joint</td>
      <td>Salad Place</td>
      <td>Sandwich Place</td>
      <td>Wine Bar</td>
      <td>Taco Place</td>
    </tr>
    <tr>
      <th>18</th>
      <td>3</td>
      <td>94124</td>
      <td>Southern / Soul Food Restaurant</td>
      <td>Park</td>
      <td>Bakery</td>
      <td>Pizza Place</td>
      <td>Light Rail Station</td>
      <td>Mexican Restaurant</td>
      <td>Playground</td>
      <td>Bistro</td>
      <td>Theater</td>
      <td>Gym</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3</td>
      <td>94127</td>
      <td>Park</td>
      <td>Grocery Store</td>
      <td>Convenience Store</td>
      <td>Burger Joint</td>
      <td>Breakfast Spot</td>
      <td>Mexican Restaurant</td>
      <td>Café</td>
      <td>Bar</td>
      <td>Playground</td>
      <td>Mediterranean Restaurant</td>
    </tr>
    <tr>
      <th>20</th>
      <td>3</td>
      <td>94128</td>
      <td>Rental Car Location</td>
      <td>Airport Lounge</td>
      <td>Airport Service</td>
      <td>Spa</td>
      <td>Bookstore</td>
      <td>Japanese Restaurant</td>
      <td>Boutique</td>
      <td>Museum</td>
      <td>Gift Shop</td>
      <td>Exhibit</td>
    </tr>
    <tr>
      <th>21</th>
      <td>3</td>
      <td>94129</td>
      <td>Trail</td>
      <td>Tunnel</td>
      <td>Café</td>
      <td>Park</td>
      <td>Historic Site</td>
      <td>Museum</td>
      <td>General Entertainment</td>
      <td>Baseball Field</td>
      <td>Scenic Lookout</td>
      <td>Fried Chicken Joint</td>
    </tr>
    <tr>
      <th>23</th>
      <td>3</td>
      <td>94131</td>
      <td>Park</td>
      <td>Coffee Shop</td>
      <td>Playground</td>
      <td>Trail</td>
      <td>Gift Shop</td>
      <td>Café</td>
      <td>Bakery</td>
      <td>Bookstore</td>
      <td>Scenic Lookout</td>
      <td>Grocery Store</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3</td>
      <td>94132</td>
      <td>Bakery</td>
      <td>Pizza Place</td>
      <td>Food Truck</td>
      <td>Clothing Store</td>
      <td>Gym</td>
      <td>Sandwich Place</td>
      <td>Juice Bar</td>
      <td>Mobile Phone Shop</td>
      <td>Cosmetics Shop</td>
      <td>Coffee Shop</td>
    </tr>
    <tr>
      <th>25</th>
      <td>3</td>
      <td>94133</td>
      <td>Italian Restaurant</td>
      <td>Pizza Place</td>
      <td>Cocktail Bar</td>
      <td>Coffee Shop</td>
      <td>Park</td>
      <td>Café</td>
      <td>Chinese Restaurant</td>
      <td>Scenic Lookout</td>
      <td>New American Restaurant</td>
      <td>Bakery</td>
    </tr>
    <tr>
      <th>27</th>
      <td>3</td>
      <td>94143</td>
      <td>Park</td>
      <td>Pizza Place</td>
      <td>Art Gallery</td>
      <td>Garden</td>
      <td>Breakfast Spot</td>
      <td>Coffee Shop</td>
      <td>Ice Cream Shop</td>
      <td>Sushi Restaurant</td>
      <td>Playground</td>
      <td>Sandwich Place</td>
    </tr>
    <tr>
      <th>28</th>
      <td>3</td>
      <td>94158</td>
      <td>Park</td>
      <td>Coffee Shop</td>
      <td>Bar</td>
      <td>Food Truck</td>
      <td>Gym / Fitness Center</td>
      <td>Soccer Field</td>
      <td>Miscellaneous Shop</td>
      <td>Bubble Tea Shop</td>
      <td>Brewery</td>
      <td>Yoga Studio</td>
    </tr>
  </tbody>
</table>
</div>




```python
# CLuster 4
sf_venues_sorted.loc[sf_venues_sorted['ClusterLabel']==4]
# In this cluster, The most common venues are parks and spas
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
      <th>ClusterLabel</th>
      <th>PostalCode</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>26</th>
      <td>4</td>
      <td>94134</td>
      <td>Park</td>
      <td>Spa</td>
      <td>Bus Station</td>
      <td>Baseball Field</td>
      <td>Cantonese Restaurant</td>
      <td>Bakery</td>
      <td>Coffee Shop</td>
      <td>Trail</td>
      <td>Library</td>
      <td>Convenience Store</td>
    </tr>
  </tbody>
</table>
</div>



## Results and Discussion <span id="result"></span>  

As illustrated in section [Examine Clusters](#examine_cluster), Chinese restaurants are more popular in cluster 1.  
So, to open a successful Chinese restaurant in San Francisco, we recommend to select its location from the following areas:

| Area | PostalCode |
| - | - |
| Lake Merced | PostalCode 94116 |
| Sunset | PostalCode 94122 |
| Richmond | PostalCode 94118/94121 |

![SF Map](https://ljmoore.files.wordpress.com/2013/02/san-francisco-autofill-map1.jpg?w=800)

## Conclusion <span id="conclusion"></span>  

The purpose of this report is specific, recommending the best location for a successful Chinese restaurant in a U.S. city.  

However, the same methodology can be adapted to many business domains.

