

```python
#Colleen Karnas-Haines
#3/26/2018
#Equator Data
#Added an extra scatter-geo map
#Also added some nice colors to help visualize the data
#I chose to use take the absolute latitude degrees to truly show distance from the equator as opposed to position on the globe

#Observations
# 1. As suspected, cities near the equator have higher noontime temperatures than cities far away from the equator.
#It should be noted though, that there is a lack of cities in the southern hemishpere, so seraching by city creates
#a surge of cities at the southern tip of South America and Africa, whereas North America and Europe's sampled cities can be
#more spread out
# 2. At various times I noticed a more prominent trend in wind speed--the wind speed would increase
#as the distance from the equator increased. The current data reading are less pronouced, particularly midway between the 
#equator and the poles.
# 3. There is less variation in the temperatures around the equator. If you look at the temperatures at -30 to 30 degrees latitude
#there is between 40 to 50 degree difference in temperature. However, 60-90 and -60 to -90 degrees latitude
#both see temperatures with a 75 to 85 degree range.
```


```python
from config import api_key
import random
import pandas as pd
import numpy as np
from citipy import citipy
from pprint import pprint
import requests
import matplotlib.pyplot as plt
#https://openweathermapy.readthedocs.io/en/latest/
import openweathermapy.core as owm
```


```python
#define blocks so that we have an equal representation across the globe

#Five lng ranges hit too many oceans and I could not find enough *unique* cities in a timely fashion
#lat_ranges={"A":(-90,-45),"B":(-44,0),"C":(1,45),"D":(46,90)}
#lng_ranges={"a":(-180,-108),"b":(-107,-36),"c":(-35,36),"d":(37,108),"e":(109,180)}

#Three lng ranges gives us more options to find cities
lng_ranges={"a":(-180,-61),"b":(-60,60),"c":(61,180)}
lat_ranges={"A":(-90,-31),"B":(-30,30),"C":(31,90)}
```


```python
#create data frame
loc_dict={}
locations = pd.DataFrame(columns=["latitude","longitude","Nearest City","Country Code","Noon Temp(f)",
                                  "Humidity","Clouds","Wind Speed","Distance From Equator"])
```


```python
#We have 9 lat/lng combinations which does not go evenly into 500, so we will get 504
#####
#####
#Replace with 56
#####
#####
for i in range(56):
    for lat_loc in lat_ranges:
        for lng_loc in lng_ranges:
            duplicate=True
            while duplicate:          
                lat=random.uniform(*lat_ranges[lat_loc])
                lng=random.uniform(*lng_ranges[lng_loc])
                city=citipy.nearest_city(lat,lng)
                c_name=city.city_name 
                country=city.country_code
                loc_dict={"latitude":lat,"longitude":lng,"Nearest City":c_name,"Country Code":country}
                if c_name not in locations["Nearest City"].values:
                    locations=locations.append(loc_dict, ignore_index=True)
                    duplicate=False
                    
             
            
locations.head()                
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>latitude</th>
      <th>longitude</th>
      <th>Nearest City</th>
      <th>Country Code</th>
      <th>Noon Temp(f)</th>
      <th>Humidity</th>
      <th>Clouds</th>
      <th>Wind Speed</th>
      <th>Distance From Equator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-33.381618</td>
      <td>-170.591883</td>
      <td>vaini</td>
      <td>to</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-45.713471</td>
      <td>-12.066371</td>
      <td>jamestown</td>
      <td>sh</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-62.270670</td>
      <td>116.297216</td>
      <td>albany</td>
      <td>au</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-10.734625</td>
      <td>-93.916516</td>
      <td>puerto ayora</td>
      <td>ec</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>25.107179</td>
      <td>34.769792</td>
      <td>safaga</td>
      <td>eg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Since we have 4 extra cities, we randomly drop 4 of them
#Reread insruction, we need *at least* 500
#to_drop=[random.randint(0,(len(locations)-1)) for i in range(4)]
#for i in to_drop:
#    locations=locations.drop(i)
#locations=locations.reset_index(drop=True)
```


```python
base_url="http://api.openweathermap.org/data/2.5/forecast?appid="+api_key+"&q="
units="&units=imperial"

```


```python

for row in range(len(locations)):
    print("Processing record #"+str(row)+" | "+locations.iloc[row,2])
    url="http://api.openweathermap.org/data/2.5/forecast?appid="+api_key+"&lat="+str(int(locations.iloc[row][0]))+"&lon="+str(int(locations.iloc[row][1]))+units
    response=requests.get(url,"Error").json()
    print(url)
    
    #I had this code because osme of the cities were unknown to the weather api, but the lat/long were always good.
    #I decided just to use lat/long exclusively.
    
    #response=requests.get(base_url+locations.iloc[row][2]+","+locations.iloc[row][3]+units).json()
    #if response["cod"]=="404":
    #    response=requests.get("http://api.openweathermap.org/data/2.5/forecast?appid="+api_key+
    #                          "&lat="+str(int(locations.iloc[row][0]))+"&lon="+str(int(locations.iloc[row][1]))+units).json() 
    #    print("http://api.openweathermap.org/data/2.5/forecast?appid="+api_key+
    #                          "&lat="+str(int(locations.iloc[row][0]))+"&lon="+str(int(locations.iloc[row][1]))+units)
    #else:
    #    print(base_url+locations.iloc[row][2]+","+locations.iloc[row][3]+units)

    #The weather forecast is every 3 hours: 0,6am,9am,12pm. So the [4] gives us the noon temperature
    locations.iloc[row,4]=response["list"][4]["main"]["temp_max"]
    locations.iloc[row,5]=response["list"][4]["main"]["humidity"]
    locations.iloc[row,6]=response["list"][4]["clouds"]["all"]
    locations.iloc[row,7]=response["list"][4]["wind"]["speed"]



```

    Processing record #0 | vaini
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-170&units=imperial
    Processing record #1 | jamestown
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-45&lon=-12&units=imperial
    Processing record #2 | albany
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-62&lon=116&units=imperial
    Processing record #3 | puerto ayora
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-10&lon=-93&units=imperial
    Processing record #4 | safaga
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=25&lon=34&units=imperial
    Processing record #5 | samarai
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-11&lon=150&units=imperial
    Processing record #6 | qaanaaq
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=87&lon=-88&units=imperial
    Processing record #7 | illoqqortoormiut
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=89&lon=-25&units=imperial
    Processing record #8 | leningradskiy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=89&lon=175&units=imperial
    Processing record #9 | rikitea
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=-111&units=imperial
    Processing record #10 | laguna
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-40&units=imperial
    Processing record #11 | busselton
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-81&lon=75&units=imperial
    Processing record #12 | atuona
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-4&lon=-135&units=imperial
    Processing record #13 | kuruman
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-26&lon=23&units=imperial
    Processing record #14 | banda aceh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=4&lon=91&units=imperial
    Processing record #15 | yellowknife
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=88&lon=-106&units=imperial
    Processing record #16 | pudozh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=62&lon=36&units=imperial
    Processing record #17 | amderma
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=70&lon=63&units=imperial
    Processing record #18 | mataura
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-74&lon=-159&units=imperial
    Processing record #19 | arraial do cabo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=-29&units=imperial
    Processing record #20 | taolanaro
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-83&lon=62&units=imperial
    Processing record #21 | moerai
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-20&lon=-150&units=imperial
    Processing record #22 | vila velha
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-29&lon=-25&units=imperial
    Processing record #23 | letpadan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=17&lon=95&units=imperial
    Processing record #24 | lewistown
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=47&lon=-109&units=imperial
    Processing record #25 | ilulissat
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=76&lon=-47&units=imperial
    Processing record #26 | pevek
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=83&lon=167&units=imperial
    Processing record #27 | ushuaia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-71&lon=-80&units=imperial
    Processing record #28 | port alfred
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-89&lon=54&units=imperial
    Processing record #29 | geraldton
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=95&units=imperial
    Processing record #30 | paita
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-6&lon=-85&units=imperial
    Processing record #31 | amapa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=-47&units=imperial
    Processing record #32 | osmena
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=10&lon=120&units=imperial
    Processing record #33 | provideniya
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=40&lon=-178&units=imperial
    Processing record #34 | vawkavysk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=24&units=imperial
    Processing record #35 | mikhaylovka
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=44&lon=71&units=imperial
    Processing record #36 | punta arenas
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-70&lon=-106&units=imperial
    Processing record #37 | cape town
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-71&lon=-10&units=imperial
    Processing record #38 | bluff
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-79&lon=170&units=imperial
    Processing record #39 | hilo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=-139&units=imperial
    Processing record #40 | cuamba
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-14&lon=36&units=imperial
    Processing record #41 | tadine
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-21&lon=169&units=imperial
    Processing record #42 | fallon
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=-117&units=imperial
    Processing record #43 | nanortalik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=-45&units=imperial
    Processing record #44 | heishan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=41&lon=122&units=imperial
    Processing record #45 | avarua
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=-164&units=imperial
    Processing record #46 | east london
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-80&lon=56&units=imperial
    Processing record #47 | flinders
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=135&units=imperial
    Processing record #48 | hobe sound
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=27&lon=-79&units=imperial
    Processing record #49 | manakara
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-22&lon=49&units=imperial
    Processing record #50 | bengkulu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-19&lon=88&units=imperial
    Processing record #51 | nome
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=71&lon=-165&units=imperial
    Processing record #52 | westport
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=58&lon=-13&units=imperial
    Processing record #53 | kazalinsk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=44&lon=61&units=imperial
    Processing record #54 | avera
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-156&units=imperial
    Processing record #55 | tsihombe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=43&units=imperial
    Processing record #56 | saint-philippe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-60&lon=67&units=imperial
    Processing record #57 | freeport
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=26&lon=-78&units=imperial
    Processing record #58 | ponta do sol
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=20&lon=-31&units=imperial
    Processing record #59 | kununurra
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-19&lon=126&units=imperial
    Processing record #60 | yarmouth
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=42&lon=-67&units=imperial
    Processing record #61 | klaksvik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=77&lon=-5&units=imperial
    Processing record #62 | nikolskoye
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=43&lon=173&units=imperial
    Processing record #63 | ancud
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=-74&units=imperial
    Processing record #64 | chuy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-66&lon=-21&units=imperial
    Processing record #65 | mount gambier
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-43&lon=136&units=imperial
    Processing record #66 | guerrero negro
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=-123&units=imperial
    Processing record #67 | qandala
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=10&lon=50&units=imperial
    Processing record #68 | labuhan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-22&lon=92&units=imperial
    Processing record #69 | norman wells
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=-121&units=imperial
    Processing record #70 | sorvag
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=61&lon=-11&units=imperial
    Processing record #71 | baruun-urt
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=46&lon=113&units=imperial
    Processing record #72 | constitucion
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-74&units=imperial
    Processing record #73 | cidreira
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=-36&units=imperial
    Processing record #74 | hobart
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-60&lon=148&units=imperial
    Processing record #75 | alofi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-18&lon=-166&units=imperial
    Processing record #76 | namibe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-15&lon=9&units=imperial
    Processing record #77 | kaeng khlo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=15&lon=102&units=imperial
    Processing record #78 | lac du bonnet
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=-94&units=imperial
    Processing record #79 | tabas
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=34&lon=56&units=imperial
    Processing record #80 | omsukchan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=62&lon=155&units=imperial
    Processing record #81 | castro
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-51&lon=-97&units=imperial
    Processing record #82 | hermanus
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-64&lon=1&units=imperial
    Processing record #83 | kaitangata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-61&lon=177&units=imperial
    Processing record #84 | mutata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=7&lon=-76&units=imperial
    Processing record #85 | mbandaka
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=0&lon=17&units=imperial
    Processing record #86 | kotaparh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=19&lon=82&units=imperial
    Processing record #87 | havelock
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=33&lon=-75&units=imperial
    Processing record #88 | sorland
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=10&units=imperial
    Processing record #89 | yermakovskoye
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=92&units=imperial
    Processing record #90 | lebu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-100&units=imperial
    Processing record #91 | mar del plata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-54&lon=-48&units=imperial
    Processing record #92 | nelson bay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=159&units=imperial
    Processing record #93 | huamachuco
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-7&lon=-77&units=imperial
    Processing record #94 | grand baie
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-16&lon=55&units=imperial
    Processing record #95 | hasaki
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=23&lon=160&units=imperial
    Processing record #96 | florence
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=33&lon=-110&units=imperial
    Processing record #97 | grindavik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=52&lon=-26&units=imperial
    Processing record #98 | srednekolymsk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=73&lon=154&units=imperial
    Processing record #99 | rio gallegos
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-50&lon=-67&units=imperial
    Processing record #100 | port elizabeth
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-65&lon=33&units=imperial
    Processing record #101 | new norfolk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-74&lon=126&units=imperial
    Processing record #102 | pisco
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-18&lon=-87&units=imperial
    Processing record #103 | luanda
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-10&lon=10&units=imperial
    Processing record #104 | katsuura
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=24&lon=148&units=imperial
    Processing record #105 | fortuna
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=37&lon=-136&units=imperial
    Processing record #106 | rabat
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=34&lon=-7&units=imperial
    Processing record #107 | komsomolskiy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=62&units=imperial
    Processing record #108 | villarrica
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=-70&units=imperial
    Processing record #109 | luderitz
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=4&units=imperial
    Processing record #110 | mahebourg
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=71&units=imperial
    Processing record #111 | villagarzon
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=0&lon=-76&units=imperial
    Processing record #112 | takoradi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=0&lon=-1&units=imperial
    Processing record #113 | kupang
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-10&lon=123&units=imperial
    Processing record #114 | kapaa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=33&lon=-168&units=imperial
    Processing record #115 | milazzo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=38&lon=15&units=imperial
    Processing record #116 | beidao
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=34&lon=105&units=imperial
    Processing record #117 | puerto madryn
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-42&lon=-65&units=imperial
    Processing record #118 | bredasdorp
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-86&lon=22&units=imperial
    Processing record #119 | tuatapere
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-49&lon=162&units=imperial
    Processing record #120 | santa rosa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-17&lon=-61&units=imperial
    Processing record #121 | nchelenge
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-9&lon=27&units=imperial
    Processing record #122 | butaritari
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=19&lon=167&units=imperial
    Processing record #123 | barrow
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=74&lon=-157&units=imperial
    Processing record #124 | ribeira grande
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=31&lon=-29&units=imperial
    Processing record #125 | tokur
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=56&lon=132&units=imperial
    Processing record #126 | coihaique
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-47&lon=-72&units=imperial
    Processing record #127 | umzimvubu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=33&units=imperial
    Processing record #128 | dunedin
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-54&lon=178&units=imperial
    Processing record #129 | cartagena
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=11&lon=-76&units=imperial
    Processing record #130 | dakar
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=16&lon=-18&units=imperial
    Processing record #131 | mangrol
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=20&lon=69&units=imperial
    Processing record #132 | moose factory
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=51&lon=-80&units=imperial
    Processing record #133 | yevlax
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=40&lon=47&units=imperial
    Processing record #134 | batagay-alyta
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=129&units=imperial
    Processing record #135 | san carlos de bariloche
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-42&lon=-68&units=imperial
    Processing record #136 | necochea
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-42&lon=-56&units=imperial
    Processing record #137 | esperance
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=127&units=imperial
    Processing record #138 | halalo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-16&lon=-177&units=imperial
    Processing record #139 | lolodorf
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=10&units=imperial
    Processing record #140 | shingu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=27&lon=137&units=imperial
    Processing record #141 | laurel
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=45&lon=-108&units=imperial
    Processing record #142 | longyearbyen
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=77&lon=20&units=imperial
    Processing record #143 | ola
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=56&lon=152&units=imperial
    Processing record #144 | rawson
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-46&lon=-61&units=imperial
    Processing record #145 | kruisfontein
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-63&lon=27&units=imperial
    Processing record #146 | port macquarie
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=152&units=imperial
    Processing record #147 | san quintin
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=22&lon=-127&units=imperial
    Processing record #148 | mandera
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=4&lon=41&units=imperial
    Processing record #149 | mount isa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-24&lon=142&units=imperial
    Processing record #150 | lavrentiya
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=-170&units=imperial
    Processing record #151 | barentsburg
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=80&lon=-1&units=imperial
    Processing record #152 | nizhneyansk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=84&lon=134&units=imperial
    Processing record #153 | rio cuarto
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-63&units=imperial
    Processing record #154 | sao joao da barra
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-25&units=imperial
    Processing record #155 | te anau
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=166&units=imperial
    Processing record #156 | saleaula
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=1&lon=-166&units=imperial
    Processing record #157 | itaueira
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-7&lon=-43&units=imperial
    Processing record #158 | kavieng
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=4&lon=155&units=imperial
    Processing record #159 | kodiak
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=59&lon=-154&units=imperial
    Processing record #160 | paamiut
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=59&lon=-51&units=imperial
    Processing record #161 | saskylakh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=84&lon=120&units=imperial
    Processing record #162 | san luis
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-67&units=imperial
    Processing record #163 | rio grande
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-47&units=imperial
    Processing record #164 | port lincoln
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=136&units=imperial
    Processing record #165 | fare
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-15&lon=-149&units=imperial
    Processing record #166 | bonthe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=-15&units=imperial
    Processing record #167 | kaeo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-28&lon=176&units=imperial
    Processing record #168 | tuktoyaktuk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=85&lon=-128&units=imperial
    Processing record #169 | talah
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=35&lon=8&units=imperial
    Processing record #170 | khatanga
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=82&lon=95&units=imperial
    Processing record #171 | rosario
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-61&units=imperial
    Processing record #172 | richards bay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=35&units=imperial
    Processing record #173 | okato
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=172&units=imperial
    Processing record #174 | vaitupu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-3&lon=-177&units=imperial
    Processing record #175 | maceio
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-13&lon=-30&units=imperial
    Processing record #176 | grand river south east
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-21&lon=70&units=imperial
    Processing record #177 | thunder bay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=49&lon=-89&units=imperial
    Processing record #178 | ust-tsilma
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=66&lon=49&units=imperial
    Processing record #179 | tiksi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=81&lon=125&units=imperial
    Processing record #180 | viedma
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=-63&units=imperial
    Processing record #181 | margate
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=38&units=imperial
    Processing record #182 | portland
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-51&lon=134&units=imperial
    Processing record #183 | westpunt
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=15&lon=-69&units=imperial
    Processing record #184 | omboue
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-6&lon=2&units=imperial
    Processing record #185 | veraval
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=68&units=imperial
    Processing record #186 | taber
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=50&lon=-112&units=imperial
    Processing record #187 | lagoa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=45&lon=-27&units=imperial
    Processing record #188 | ust-maya
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=60&lon=134&units=imperial
    Processing record #189 | valparaiso
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-75&units=imperial
    Processing record #190 | sao lourenco do sul
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-51&units=imperial
    Processing record #191 | griffith
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=145&units=imperial
    Processing record #192 | cockburn harbour
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=23&lon=-71&units=imperial
    Processing record #193 | salalah
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=19&lon=53&units=imperial
    Processing record #194 | akyab
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=90&units=imperial
    Processing record #195 | attawapiskat
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=-85&units=imperial
    Processing record #196 | develi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=37&lon=35&units=imperial
    Processing record #197 | bolshegrivskoye
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=52&lon=74&units=imperial
    Processing record #198 | general pico
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=-63&units=imperial
    Processing record #199 | treinta y tres
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-55&units=imperial
    Processing record #200 | souillac
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-45&lon=70&units=imperial
    Processing record #201 | chicama
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-7&lon=-79&units=imperial
    Processing record #202 | khor
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=26&lon=52&units=imperial
    Processing record #203 | carnarvon
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-24&lon=113&units=imperial
    Processing record #204 | soddy-daisy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=35&lon=-85&units=imperial
    Processing record #205 | upernavik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=79&lon=-56&units=imperial
    Processing record #206 | chokurdakh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=82&lon=144&units=imperial
    Processing record #207 | linares
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=-69&units=imperial
    Processing record #208 | saldanha
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=10&units=imperial
    Processing record #209 | launceston
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=148&units=imperial
    Processing record #210 | ewa beach
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=-158&units=imperial
    Processing record #211 | carutapera
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=7&lon=-41&units=imperial
    Processing record #212 | marihatag
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=8&lon=127&units=imperial
    Processing record #213 | scottsbluff
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=41&lon=-103&units=imperial
    Processing record #214 | chernyy yar
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=47&lon=45&units=imperial
    Processing record #215 | chagda
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=59&lon=129&units=imperial
    Processing record #216 | neuquen
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=-69&units=imperial
    Processing record #217 | port shepstone
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=38&units=imperial
    Processing record #218 | burnie
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-43&lon=141&units=imperial
    Processing record #219 | tayoltita
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=24&lon=-105&units=imperial
    Processing record #220 | bassila
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=8&lon=1&units=imperial
    Processing record #221 | airai
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=12&lon=142&units=imperial
    Processing record #222 | smithers
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=57&lon=-127&units=imperial
    Processing record #223 | larsnes
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=64&lon=4&units=imperial
    Processing record #224 | dikson
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=83&lon=68&units=imperial
    Processing record #225 | comodoro rivadavia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-45&lon=-68&units=imperial
    Processing record #226 | rocha
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-43&lon=-45&units=imperial
    Processing record #227 | bambous virieux
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=82&units=imperial
    Processing record #228 | cabo san lucas
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=9&lon=-115&units=imperial
    Processing record #229 | longido
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-3&lon=36&units=imperial
    Processing record #230 | lorengau
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=2&lon=143&units=imperial
    Processing record #231 | mys shmidta
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=76&lon=-172&units=imperial
    Processing record #232 | ozinki
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=49&lon=50&units=imperial
    Processing record #233 | anadyr
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=61&lon=175&units=imperial
    Processing record #234 | villa carlos paz
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-64&units=imperial
    Processing record #235 | beloha
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=42&units=imperial
    Processing record #236 | ngunguru
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=177&units=imperial
    Processing record #237 | mazatlan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=22&lon=-106&units=imperial
    Processing record #238 | tabou
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-2&lon=-6&units=imperial
    Processing record #239 | kloulklubed
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=136&units=imperial
    Processing record #240 | sioux lookout
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=50&lon=-90&units=imperial
    Processing record #241 | velyka mykhaylivka
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=47&lon=30&units=imperial
    Processing record #242 | severo-kurilsk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=44&lon=162&units=imperial
    Processing record #243 | san rafael
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-66&units=imperial
    Processing record #244 | tres arroyos
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=-59&units=imperial
    Processing record #245 | mildura
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=142&units=imperial
    Processing record #246 | upata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=6&lon=-62&units=imperial
    Processing record #247 | caluquembe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-14&lon=14&units=imperial
    Processing record #248 | lata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-8&lon=167&units=imperial
    Processing record #249 | kenai
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=65&lon=-155&units=imperial
    Processing record #250 | bolungarvik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=-27&units=imperial
    Processing record #251 | bolshaya rechka
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=52&lon=104&units=imperial
    Processing record #252 | villa maria
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-63&units=imperial
    Processing record #253 | knysna
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=22&units=imperial
    Processing record #254 | hokitika
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=166&units=imperial
    Processing record #255 | puerto escondido
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=12&lon=-97&units=imperial
    Processing record #256 | porto novo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=14&lon=-32&units=imperial
    Processing record #257 | rungata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-1&lon=178&units=imperial
    Processing record #258 | thompson
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=66&lon=-96&units=imperial
    Processing record #259 | swarzedz
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=52&lon=17&units=imperial
    Processing record #260 | teya
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=62&lon=92&units=imperial
    Processing record #261 | valdivia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=-73&units=imperial
    Processing record #262 | pinheiro machado
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-53&units=imperial
    Processing record #263 | northam
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=118&units=imperial
    Processing record #264 | cumana
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=10&lon=-64&units=imperial
    Processing record #265 | cayenne
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=16&lon=-42&units=imperial
    Processing record #266 | meulaboh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-3&lon=88&units=imperial
    Processing record #267 | eureka
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=41&lon=-137&units=imperial
    Processing record #268 | noceto
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=44&lon=10&units=imperial
    Processing record #269 | zhigansk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=122&units=imperial
    Processing record #270 | cauquenes
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-72&units=imperial
    Processing record #271 | tapes
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-51&units=imperial
    Processing record #272 | christchurch
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-43&lon=172&units=imperial
    Processing record #273 | faanui
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-12&lon=-154&units=imperial
    Processing record #274 | inhapim
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-19&lon=-41&units=imperial
    Processing record #275 | hithadhoo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-2&lon=76&units=imperial
    Processing record #276 | narsaq
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=80&lon=-62&units=imperial
    Processing record #277 | qasigiannguit
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=67&lon=-48&units=imperial
    Processing record #278 | petropavlovsk-kamchatskiy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=44&lon=164&units=imperial
    Processing record #279 | punta alta
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=-61&units=imperial
    Processing record #280 | carmelo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-58&units=imperial
    Processing record #281 | perth
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=114&units=imperial
    Processing record #282 | inirida
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=-64&units=imperial
    Processing record #283 | kindu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-3&lon=25&units=imperial
    Processing record #284 | olutanga
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=7&lon=122&units=imperial
    Processing record #285 | iqaluit
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=61&lon=-74&units=imperial
    Processing record #286 | uddevalla
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=58&lon=11&units=imperial
    Processing record #287 | haibowan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=107&units=imperial
    Processing record #288 | coihueco
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=-71&units=imperial
    Processing record #289 | dolores
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=-56&units=imperial
    Processing record #290 | ahipara
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=169&units=imperial
    Processing record #291 | san francisco
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-29&lon=-62&units=imperial
    Processing record #292 | tamboril
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-4&lon=-40&units=imperial
    Processing record #293 | sabang
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=6&lon=92&units=imperial
    Processing record #294 | pangnirtung
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=61&lon=-61&units=imperial
    Processing record #295 | saint anthony
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=-52&units=imperial
    Processing record #296 | sherlovaya gora
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=50&lon=115&units=imperial
    Processing record #297 | rafaela
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-61&units=imperial
    Processing record #298 | cangucu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-52&units=imperial
    Processing record #299 | waipawa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-44&lon=175&units=imperial
    Processing record #300 | sucua
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-3&lon=-77&units=imperial
    Processing record #301 | conde
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-12&lon=-36&units=imperial
    Processing record #302 | loa janan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=0&lon=116&units=imperial
    Processing record #303 | crawfordsville
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=-86&units=imperial
    Processing record #304 | bud
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=67&lon=6&units=imperial
    Processing record #305 | seoul
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=127&units=imperial
    Processing record #306 | vilcun
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=-72&units=imperial
    Processing record #307 | plettenberg bay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-42&lon=23&units=imperial
    Processing record #308 | ulladulla
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=157&units=imperial
    Processing record #309 | lazaro cardenas
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=11&lon=-105&units=imperial
    Processing record #310 | urucara
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=0&lon=-59&units=imperial
    Processing record #311 | malwan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=16&lon=72&units=imperial
    Processing record #312 | slave lake
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=-113&units=imperial
    Processing record #313 | manbij
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=35&lon=39&units=imperial
    Processing record #314 | batagay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=68&lon=134&units=imperial
    Processing record #315 | coquimbo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-75&units=imperial
    Processing record #316 | rivera
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-55&units=imperial
    Processing record #317 | colac
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=143&units=imperial
    Processing record #318 | salta
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-24&lon=-67&units=imperial
    Processing record #319 | montes altos
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-5&lon=-47&units=imperial
    Processing record #320 | innisfail
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-17&lon=146&units=imperial
    Processing record #321 | port hardy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=51&lon=-127&units=imperial
    Processing record #322 | dipkarpaz
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=35&lon=35&units=imperial
    Processing record #323 | fengcheng
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=40&lon=124&units=imperial
    Processing record #324 | san antonio
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-72&units=imperial
    Processing record #325 | george
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=22&units=imperial
    Processing record #326 | batemans bay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=156&units=imperial
    Processing record #327 | acevedo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=1&lon=-75&units=imperial
    Processing record #328 | farafangana
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-22&lon=47&units=imperial
    Processing record #329 | mata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=21&lon=111&units=imperial
    Processing record #330 | bethel
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=-159&units=imperial
    Processing record #331 | sarno
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=40&lon=14&units=imperial
    Processing record #332 | gulshat
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=45&lon=74&units=imperial
    Processing record #333 | lincoln
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=-62&units=imperial
    Processing record #334 | zarate
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-59&units=imperial
    Processing record #335 | masterton
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-41&lon=176&units=imperial
    Processing record #336 | makakilo city
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=5&lon=-166&units=imperial
    Processing record #337 | itaguai
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-22&lon=-43&units=imperial
    Processing record #338 | dhidhdhoo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=8&lon=73&units=imperial
    Processing record #339 | shelburne
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=-64&units=imperial
    Processing record #340 | sondrio
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=46&lon=9&units=imperial
    Processing record #341 | zhuanghe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=38&lon=123&units=imperial
    Processing record #342 | cordoba
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-63&units=imperial
    Processing record #343 | oranjemund
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=7&units=imperial
    Processing record #344 | corowa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=146&units=imperial
    Processing record #345 | san carlos
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=17&lon=-90&units=imperial
    Processing record #346 | goure
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=13&lon=10&units=imperial
    Processing record #347 | ingham
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-18&lon=145&units=imperial
    Processing record #348 | whitehorse
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=61&lon=-130&units=imperial
    Processing record #349 | belushya guba
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=80&lon=50&units=imperial
    Processing record #350 | kyzyl-suu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=41&lon=77&units=imperial
    Processing record #351 | bahia blanca
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=-62&units=imperial
    Processing record #352 | artigas
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-56&units=imperial
    Processing record #353 | lakes entrance
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=150&units=imperial
    Processing record #354 | temascalcingo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=19&lon=-100&units=imperial
    Processing record #355 | tchaourou
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=8&lon=2&units=imperial
    Processing record #356 | kanniyakumari
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=6&lon=77&units=imperial
    Processing record #357 | egvekinot
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=56&lon=-179&units=imperial
    Processing record #358 | izhevskoye
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=54&lon=41&units=imperial
    Processing record #359 | de-kastri
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=51&lon=141&units=imperial
    Processing record #360 | mercedes
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-65&units=imperial
    Processing record #361 | saint-joseph
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=57&units=imperial
    Processing record #362 | victor harbor
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=137&units=imperial
    Processing record #363 | santa eulalia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-11&lon=-76&units=imperial
    Processing record #364 | los llanos de aridane
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=27&lon=-20&units=imperial
    Processing record #365 | singaraja
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-6&lon=115&units=imperial
    Processing record #366 | walla walla
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=46&lon=-117&units=imperial
    Processing record #367 | torbay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=44&lon=-41&units=imperial
    Processing record #368 | yar-sale
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=71&lon=70&units=imperial
    Processing record #369 | lujan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-69&units=imperial
    Processing record #370 | swellendam
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=21&units=imperial
    Processing record #371 | kiama
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=157&units=imperial
    Processing record #372 | celestun
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=21&lon=-93&units=imperial
    Processing record #373 | gat
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=23&lon=12&units=imperial
    Processing record #374 | mawlaik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=24&lon=94&units=imperial
    Processing record #375 | atikokan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=48&lon=-92&units=imperial
    Processing record #376 | constantine
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=37&lon=5&units=imperial
    Processing record #377 | talnakh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=69&lon=94&units=imperial
    Processing record #378 | general roca
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=-66&units=imperial
    Processing record #379 | scottsburgh
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=34&units=imperial
    Processing record #380 | newcastle
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=152&units=imperial
    Processing record #381 | acapulco
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=-105&units=imperial
    Processing record #382 | bathsheba
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=14&lon=-52&units=imperial
    Processing record #383 | ojhar
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=20&lon=74&units=imperial
    Processing record #384 | saint george
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=31&lon=-61&units=imperial
    Processing record #385 | snina
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=49&lon=22&units=imperial
    Processing record #386 | sentyabrskiy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=46&lon=151&units=imperial
    Processing record #387 | talcahuano
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-81&units=imperial
    Processing record #388 | stellenbosch
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=18&units=imperial
    Processing record #389 | bunbury
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=115&units=imperial
    Processing record #390 | bucerias
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=20&lon=-107&units=imperial
    Processing record #391 | prado
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-16&lon=-38&units=imperial
    Processing record #392 | balaghat
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=21&lon=80&units=imperial
    Processing record #393 | hamilton
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=32&lon=-69&units=imperial
    Processing record #394 | andenes
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=70&lon=15&units=imperial
    Processing record #395 | troitsk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=61&units=imperial
    Processing record #396 | rivadavia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-67&units=imperial
    Processing record #397 | adelaide
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=26&units=imperial
    Processing record #398 | napier
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-39&lon=177&units=imperial
    Processing record #399 | samusu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-4&lon=-161&units=imperial
    Processing record #400 | tawkar
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=38&units=imperial
    Processing record #401 | gopalpur
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=85&units=imperial
    Processing record #402 | haines junction
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=59&lon=-143&units=imperial
    Processing record #403 | portpatrick
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=54&lon=-5&units=imperial
    Processing record #404 | stoyba
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=131&units=imperial
    Processing record #405 | osorno
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=-73&units=imperial
    Processing record #406 | santa vitoria do palmar
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-52&units=imperial
    Processing record #407 | broken hill
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=144&units=imperial
    Processing record #408 | hihifo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-16&lon=-175&units=imperial
    Processing record #409 | goderich
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=5&lon=-16&units=imperial
    Processing record #410 | manokwari
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=1&lon=132&units=imperial
    Processing record #411 | deming
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=32&lon=-107&units=imperial
    Processing record #412 | saint-augustin
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=-58&units=imperial
    Processing record #413 | yinchuan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=38&lon=107&units=imperial
    Processing record #414 | san juan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-67&units=imperial
    Processing record #415 | butterworth
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=29&units=imperial
    Processing record #416 | coromandel
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=175&units=imperial
    Processing record #417 | porto walter
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-9&lon=-71&units=imperial
    Processing record #418 | marienburg
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=7&lon=-54&units=imperial
    Processing record #419 | dicabisagan
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=18&lon=128&units=imperial
    Processing record #420 | prince rupert
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=47&lon=-136&units=imperial
    Processing record #421 | cascais
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=37&lon=-13&units=imperial
    Processing record #422 | manzhouli
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=48&lon=117&units=imperial
    Processing record #423 | los angeles
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=-72&units=imperial
    Processing record #424 | stutterheim
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=27&units=imperial
    Processing record #425 | fairlie
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-43&lon=170&units=imperial
    Processing record #426 | san cristobal
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-11&lon=-91&units=imperial
    Processing record #427 | kandi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=10&lon=3&units=imperial
    Processing record #428 | mlonggo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-5&lon=110&units=imperial
    Processing record #429 | nipawin
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=53&lon=-104&units=imperial
    Processing record #430 | kalundborg
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=11&units=imperial
    Processing record #431 | hecun
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=36&lon=113&units=imperial
    Processing record #432 | calbuco
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-42&lon=-73&units=imperial
    Processing record #433 | middelburg
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=24&units=imperial
    Processing record #434 | cowra
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=149&units=imperial
    Processing record #435 | hualmay
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-14&lon=-84&units=imperial
    Processing record #436 | victoria
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-5&lon=53&units=imperial
    Processing record #437 | qui nhon
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=14&lon=112&units=imperial
    Processing record #438 | la ronge
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=-105&units=imperial
    Processing record #439 | tasiilaq
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=57&lon=-36&units=imperial
    Processing record #440 | udachnyy
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=67&lon=111&units=imperial
    Processing record #441 | bell ville
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=-62&units=imperial
    Processing record #442 | calvinia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=19&units=imperial
    Processing record #443 | sydney
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-34&lon=151&units=imperial
    Processing record #444 | rioja
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-5&lon=-77&units=imperial
    Processing record #445 | lukulu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-14&lon=23&units=imperial
    Processing record #446 | padang
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-6&lon=94&units=imperial
    Processing record #447 | sayville
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=-72&units=imperial
    Processing record #448 | zile
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=40&lon=35&units=imperial
    Processing record #449 | mombetsu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=45&lon=143&units=imperial
    Processing record #450 | salamanca
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-70&units=imperial
    Processing record #451 | paysandu
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-58&units=imperial
    Processing record #452 | mandurah
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=112&units=imperial
    Processing record #453 | baracoa
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=20&lon=-73&units=imperial
    Processing record #454 | minab
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=25&lon=58&units=imperial
    Processing record #455 | ratnagiri
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=16&lon=72&units=imperial
    Processing record #456 | charlestown
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=39&lon=-71&units=imperial
    Processing record #457 | yefremov
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=52&lon=38&units=imperial
    Processing record #458 | nyurba
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=67&lon=117&units=imperial
    Processing record #459 | mulchen
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=-71&units=imperial
    Processing record #460 | young
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-57&units=imperial
    Processing record #461 | ruatoria
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-36&lon=179&units=imperial
    Processing record #462 | santa isabel do rio negro
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=1&lon=-63&units=imperial
    Processing record #463 | jiddah
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=21&lon=38&units=imperial
    Processing record #464 | naze
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=23&lon=133&units=imperial
    Processing record #465 | dolbeau
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=52&lon=-70&units=imperial
    Processing record #466 | qafsah
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=34&lon=8&units=imperial
    Processing record #467 | kamiiso
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=42&lon=139&units=imperial
    Processing record #468 | illapel
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-31&lon=-73&units=imperial
    Processing record #469 | la plata
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-57&units=imperial
    Processing record #470 | collie
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=116&units=imperial
    Processing record #471 | san patricio
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=5&lon=-117&units=imperial
    Processing record #472 | malindi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-3&lon=41&units=imperial
    Processing record #473 | mahibadhoo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=3&lon=66&units=imperial
    Processing record #474 | fort saint james
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=-124&units=imperial
    Processing record #475 | kamenka
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=51&lon=42&units=imperial
    Processing record #476 | kondinskoye
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=60&lon=68&units=imperial
    Processing record #477 | longavi
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-35&lon=-71&units=imperial
    Processing record #478 | kirkwood
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-33&lon=25&units=imperial
    Processing record #479 | kawhia
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=174&units=imperial
    Processing record #480 | maneadero
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=29&lon=-119&units=imperial
    Processing record #481 | garowe
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=7&lon=47&units=imperial
    Processing record #482 | zhicheng
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=26&lon=118&units=imperial
    Processing record #483 | aklavik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=77&lon=-143&units=imperial
    Processing record #484 | dalvik
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=67&lon=-18&units=imperial
    Processing record #485 | kirensk
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=57&lon=107&units=imperial
    Processing record #486 | trelew
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-42&lon=-66&units=imperial
    Processing record #487 | maldonado
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-37&lon=-53&units=imperial
    Processing record #488 | traralgon
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-38&lon=146&units=imperial
    Processing record #489 | boa vista
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=1&lon=-61&units=imperial
    Processing record #490 | ginir
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=7&lon=42&units=imperial
    Processing record #491 | hue
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=15&lon=107&units=imperial
    Processing record #492 | palmer
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=56&lon=-144&units=imperial
    Processing record #493 | sidi ali
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=35&lon=1&units=imperial
    Processing record #494 | gazli
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=41&lon=62&units=imperial
    Processing record #495 | la union
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-40&lon=-73&units=imperial
    Processing record #496 | tacuarembo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=-56&units=imperial
    Processing record #497 | wellington
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-32&lon=148&units=imperial
    Processing record #498 | lompoc
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=25&lon=-130&units=imperial
    Processing record #499 | tessalit
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=24&lon=3&units=imperial
    Processing record #500 | lolua
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=-5&lon=171&units=imperial
    Processing record #501 | mayo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=63&lon=-138&units=imperial
    Processing record #502 | vila franca do campo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=37&lon=-24&units=imperial
    Processing record #503 | vengerovo
    http://api.openweathermap.org/data/2.5/forecast?appid=74bf24768501a41ba7a1d66c2054a799&lat=55&lon=76&units=imperial
    


```python
#The pdf examples had x-axes from -90 to 90, however, in reading the literature about the assignment, the question really asked
#about distance from equator, not placement on globe. Therefore, I did an extra step where I grabbed the absolute value
#of the latitude. This makes the graph look a bit different, but truly evaluates distance from equator vs. placement on globe
for i in range(len(locations)):
    locations.iloc[i,8]=abs(int(locations.iloc[i][0]))
#print(str(abs(int(locations.iloc[0][0]))))
#locations.head()
```


```python

locations=locations.sort_values(by=["Distance From Equator"])
locations.to_csv("EquatorDataResults.csv")
locations.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>latitude</th>
      <th>longitude</th>
      <th>Nearest City</th>
      <th>Country Code</th>
      <th>Noon Temp(f)</th>
      <th>Humidity</th>
      <th>Clouds</th>
      <th>Wind Speed</th>
      <th>Distance From Equator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>310</th>
      <td>-0.138592</td>
      <td>-59.460006</td>
      <td>urucara</td>
      <td>br</td>
      <td>86.60</td>
      <td>64.0</td>
      <td>36.0</td>
      <td>3.27</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>302</th>
      <td>0.082734</td>
      <td>116.045241</td>
      <td>loa janan</td>
      <td>id</td>
      <td>74.90</td>
      <td>94.0</td>
      <td>44.0</td>
      <td>0.69</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>111</th>
      <td>0.950577</td>
      <td>-76.769581</td>
      <td>villagarzon</td>
      <td>co</td>
      <td>79.76</td>
      <td>85.0</td>
      <td>68.0</td>
      <td>2.93</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>112</th>
      <td>-0.869655</td>
      <td>-1.592478</td>
      <td>takoradi</td>
      <td>gh</td>
      <td>83.27</td>
      <td>100.0</td>
      <td>8.0</td>
      <td>7.74</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>85</th>
      <td>-0.404012</td>
      <td>17.992314</td>
      <td>mbandaka</td>
      <td>cd</td>
      <td>78.86</td>
      <td>83.0</td>
      <td>92.0</td>
      <td>2.93</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create a scatter plot that looks at temp according to distance from equator. The colormap helps the eye understand, intuitively,
#what the data is saying. Red=hot!
locations.plot(kind="scatter", x="Distance From Equator", y="Noon Temp(f)", grid=True, figsize=(20,10),
               title="Distance from Equator and Noon Temperature(f)",s=100,c="Noon Temp(f)", cmap="RdYlGn_r",
               xticks=np.arange(0,90,10), colorbar=False)

plt.xlabel('Distance From Equator,\n based on latitude')
plt.savefig('Temperature.png')
plt.show()
```


![png](output_10_0.png)



```python
#Scatter plot with distance from equator v. humidty
locations.plot(kind="scatter", x="Distance From Equator", y="Humidity", grid=True, figsize=(20,10),
              title="Distance from Equator and Humidity",s=100, edgecolor="darkblue")
plt.savefig('Humidity.png')
plt.show()
```


    <matplotlib.figure.Figure at 0x2ce07413c50>



![png](output_11_1.png)



```python
#Scatter plot for distance from Equator v. cloud cover. Again, I used a colormap to represent total coverage (covered black)
# versus no cloud coverage (transparent white)
locations.plot(kind="scatter", x="Distance From Equator", y="Clouds", grid=True, figsize=(20,10),
               title="Distance from Equator and Cloud Cover %",s=100, c="Clouds",
               cmap="gist_gray_r", colorbar=False, edgecolor="black")
plt.savefig('CloudCover.png')
plt.show()
```


    <matplotlib.figure.Figure at 0x2ce06d1e048>



![png](output_12_1.png)



```python
#Scatter plot distance from Equator v. wind speed
locations.plot(kind="scatter", x="Distance From Equator", y="Wind Speed", grid=True, figsize=(20,10),
              title="Distance from Equator and Wind Speed",s=100, edgecolor="darkblue")
plt.savefig('WindSpeed.png')
plt.show()
```


    <matplotlib.figure.Figure at 0x2ce0788afd0>



![png](output_13_1.png)



```python
import plotly.plotly as py
import pandas as pd
import plotly
plotly.tools.set_credentials_file(username='ckarnas', api_key='cAAeEVu4vtfVzgkiW6tN')

df = pd.read_csv('EquatorDataResults.csv')

scl = [50,"rgb(150,0,90)"],[60,"rgb(14,154,34)"],[95,"rgb(255, 111, 0)"]

data = [ dict(
    lat = df['latitude'],
    lon = df['longitude'],
    #text = df['Globvalue'].astype(str) + ' inches',
    marker = dict(
        color = df['Noon Temp(f)'],
        colorscale = scl,
        #reversescale = True,
        opacity = 0.7,
        size = 14,
        colorbar = dict(
            thickness = 10,
            titleside = "right",
            #outlinecolor = "rgba(68, 68, 68, 0)",
            #ticks = "outside",
            #ticklen = 3,
            #showticksuffix = "last",
            #ticksuffix = " inches",
            #dtick = 0.1
        ),
    ),
    type = 'scattergeo'
) ]
# layout = dict(
#     title = 'World Temperature vs Equator location 03-26-2018<br>Source: Open Weather API',
#     geo = dict(
        
#         showframe = False,
#         showcoastlines = False,
#         projection = dict(
#             type = 'Mercator'
#         )
#     )
# )
layout = dict(
    geo = dict(
        scope = 'world',
        showland = True,
        landcolor = "rgb(212, 212, 212)",
        subunitcolor = "rgb(255, 255, 255)",
        countrycolor = "rgb(255, 255, 255)",
        showlakes = True,
        lakecolor = "rgb(255, 255, 255)",
        showsubunits = True,
        showcountries = True,
        resolution = 50,
        
        projection=dict(type="Mercator"),
#         projection = dict(
#             type = 'conic conformal',
#             rotation = dict(
#                 lon = -100
#             )
#         ),
#         lonaxis = dict(
#             showgrid = True,
#             gridwidth = 1,
#             #range= [ -140.0, -55.0 ],
#             dtick = 5
#         ),
#          lataxis = dict (
#              showgrid = True,
#              gridwidth = 1,
#              #range= [ 20.0, 60.0 ],
#              dtick = 5
#         )
     ),
    title = 'World Temperature vs Equator location 03-26-2018<br>Source: Open Weather API',
)
fig = { 'data':data, 'layout':layout }
py.iplot(fig, filename='Equator Temperature Interaction')

```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~ckarnas/2.embed" height="525px" width="100%"></iframe>




```python
py.image.save_as(fig, filename='Equator Temperature Interaction.png')
```
