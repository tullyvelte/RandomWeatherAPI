

```python
# WeatherPy
# In this example, you'll be creating a Python script to visualize the weather of 500+ cities across the world of varying distance from the equator. 
# To accomplish this, you'll be utilizing a simple Python library, the OpenWeatherMap API, and a little common sense to create a representative model of weather across world cities.
# Your objective is to build a series of scatter plots to showcase the following relationships:
# Temperature (F) vs. Latitude
# Humidity (%) vs. Latitude
# Cloudiness (%) vs. Latitude
# Wind Speed (mph) vs. Latitude
```


```python
# Import Dependencies

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import requests as req
from citipy import citipy
import random
import owmkeys
```


```python
# Randomly select at least 500 unique **(non-repeat)** cities based on latitude and longitude.


#lat = random.sample(range(-91, 91), 100) # Change Ranges to 500 or over later
#lon = random.sample(range(-181, 181), 100)

lat = np.random.randint(-90, 90, size=2500)
lon = np.random.randint(-180, 180, size=2500)

#print(lat)
#print(lon)

zip_coords = zip(lat, lon)
rand_coords = list(zip(lat, lon))
# print(rand_coords)

coords_df = pd.DataFrame((rand_coords), columns=["Latitude", "Longitude"])

# Add Columns to Capture Place Data

coords_df["Closest City"] = ""
coords_df["Country Code"] = ""

coords_df.head()
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
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Closest City</th>
      <th>Country Code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-62</td>
      <td>70</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>72</td>
      <td>117</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>-22</td>
      <td>-98</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>78</td>
      <td>-168</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>88</td>
      <td>-132</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
# Iterate & Store City/Country Data

for index,row in coords_df.iterrows():
    city = citipy.nearest_city(row["Latitude"],row["Longitude"])
    coords_df.set_value(index,"Closest City",city.city_name)
    coords_df.set_value(index,"Country Code",city.country_code)
    
coords_df.head()
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
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Closest City</th>
      <th>Country Code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-62</td>
      <td>70</td>
      <td>saint-philippe</td>
      <td>re</td>
    </tr>
    <tr>
      <th>1</th>
      <td>72</td>
      <td>117</td>
      <td>saskylakh</td>
      <td>ru</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-22</td>
      <td>-98</td>
      <td>puerto ayora</td>
      <td>ec</td>
    </tr>
    <tr>
      <th>3</th>
      <td>78</td>
      <td>-168</td>
      <td>lavrentiya</td>
      <td>ru</td>
    </tr>
    <tr>
      <th>4</th>
      <td>88</td>
      <td>-132</td>
      <td>tuktoyaktuk</td>
      <td>ca</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check For Duplicates - We have city names & country so can lose random lat/lon numbers

cities_df = coords_df.drop(coords_df.columns[[0, 1]], axis=1) 
#df.drop(df.columns[[0, 1, 3]], axis=1) 

cities_list = cities_df.drop_duplicates()
cities_list.shape
#cities_list = cities_list.sample(20)
```




    (849, 2)




```python
# Connect to Weather API

base_url = "http://api.openweathermap.org/data/2.5/weather"

params = {'appid': owmkey,
          'q': '',
          'units': 'imperial',
          'mode': 'json'}

```


```python
# Create cols to hold needed values

cities_list.loc[:,"Latitude"] = ""
cities_list.loc[:,"Longitude"] = "" # might as well
cities_list.loc[:,"Temperature"] = ""
cities_list.loc[:,"Humidity"] = ""
cities_list.loc[:,"Cloudiness"] = ""
cities_list.loc[:,"Wind Speed"] = ""
#cities_list
```

    /anaconda3/envs/PythonData/lib/python3.6/site-packages/pandas/core/indexing.py:337: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      self.obj[key] = _infer_fill_value(value)
    /anaconda3/envs/PythonData/lib/python3.6/site-packages/pandas/core/indexing.py:517: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      self.obj[item] = s



```python
# Perform a weather check on each of the cities using a series of successive API calls.
# Include a print log of each city as it's being processed with the city number, city name, and requested URL.
counter = 0

for index,row in cities_list.iterrows():
    counter += 1
    params["q"] =f'{row["Closest City"]},{row["Country Code"]}'
    city_weather_resp = req.get(base_url, params=params).json()
    print(f"Getting Record {counter}")
    print(f"Processing weather information for {params['q']}")
   # print(f'{params}')
   # Add new Lat/Lon/Temp/Wind/Humidity/Clouds to Cities DF
    cities_list.set_value(index,"Latitude",city_weather_resp.get("coord",{}).get("lat"))
    cities_list.set_value(index,"Longitude",city_weather_resp.get("coord",{}).get("lon"))
    cities_list.set_value(index,"Temperature",city_weather_resp.get("main",{}).get("temp_max"))
    cities_list.set_value(index,"Wind Speed",city_weather_resp.get("wind",{}).get("speed"))
    cities_list.set_value(index,"Humidity",city_weather_resp.get("main",{}).get("humidity"))
    cities_list.set_value(index,"Cloudiness",city_weather_resp.get("clouds",{}).get("all"))
```

    Getting Record 1
    Processing weather information for saint-philippe,re
    Getting Record 2
    Processing weather information for saskylakh,ru
    Getting Record 3
    Processing weather information for puerto ayora,ec
    Getting Record 4
    Processing weather information for lavrentiya,ru
    Getting Record 5
    Processing weather information for tuktoyaktuk,ca
    Getting Record 6
    Processing weather information for illoqqortoormiut,gl
    Getting Record 7
    Processing weather information for milkovo,ru
    Getting Record 8
    Processing weather information for busselton,au
    Getting Record 9
    Processing weather information for hovd,mn
    Getting Record 10
    Processing weather information for ust-barguzin,ru
    Getting Record 11
    Processing weather information for arraial do cabo,br
    Getting Record 12
    Processing weather information for morant bay,jm
    Getting Record 13
    Processing weather information for pandan,ph
    Getting Record 14
    Processing weather information for ushuaia,ar
    Getting Record 15
    Processing weather information for punta arenas,cl
    Getting Record 16
    Processing weather information for northam,au
    Getting Record 17
    Processing weather information for bluff,nz
    Getting Record 18
    Processing weather information for okmulgee,us
    Getting Record 19
    Processing weather information for dikson,ru
    Getting Record 20
    Processing weather information for barrow,us
    Getting Record 21
    Processing weather information for flinders,au
    Getting Record 22
    Processing weather information for aleksandrovka,ru
    Getting Record 23
    Processing weather information for avarua,ck
    Getting Record 24
    Processing weather information for ribeira grande,pt
    Getting Record 25
    Processing weather information for mar del plata,ar
    Getting Record 26
    Processing weather information for yellowknife,ca
    Getting Record 27
    Processing weather information for port elizabeth,za
    Getting Record 28
    Processing weather information for ambon,id
    Getting Record 29
    Processing weather information for hobart,au
    Getting Record 30
    Processing weather information for mataura,pf
    Getting Record 31
    Processing weather information for mackenzie,ca
    Getting Record 32
    Processing weather information for laguna,br
    Getting Record 33
    Processing weather information for albany,au
    Getting Record 34
    Processing weather information for cherskiy,ru
    Getting Record 35
    Processing weather information for carnarvon,au
    Getting Record 36
    Processing weather information for poum,nc
    Getting Record 37
    Processing weather information for andarab,af
    Getting Record 38
    Processing weather information for atuona,pf
    Getting Record 39
    Processing weather information for chuy,uy
    Getting Record 40
    Processing weather information for kapaa,us
    Getting Record 41
    Processing weather information for butaritari,ki
    Getting Record 42
    Processing weather information for rikitea,pf
    Getting Record 43
    Processing weather information for provideniya,ru
    Getting Record 44
    Processing weather information for cape town,za
    Getting Record 45
    Processing weather information for east london,za
    Getting Record 46
    Processing weather information for necochea,ar
    Getting Record 47
    Processing weather information for vieste,it
    Getting Record 48
    Processing weather information for vaitupu,wf
    Getting Record 49
    Processing weather information for rawlins,us
    Getting Record 50
    Processing weather information for mahebourg,mu
    Getting Record 51
    Processing weather information for jamestown,sh
    Getting Record 52
    Processing weather information for thompson,ca
    Getting Record 53
    Processing weather information for pennagaram,in
    Getting Record 54
    Processing weather information for bandarbeyla,so
    Getting Record 55
    Processing weather information for vaini,to
    Getting Record 56
    Processing weather information for bredasdorp,za
    Getting Record 57
    Processing weather information for talnakh,ru
    Getting Record 58
    Processing weather information for mehamn,no
    Getting Record 59
    Processing weather information for madimba,tz
    Getting Record 60
    Processing weather information for bengkulu,id
    Getting Record 61
    Processing weather information for lorengau,pg
    Getting Record 62
    Processing weather information for ust-tsilma,ru
    Getting Record 63
    Processing weather information for excelsior springs,us
    Getting Record 64
    Processing weather information for ponto novo,br
    Getting Record 65
    Processing weather information for kazachinskoye,ru
    Getting Record 66
    Processing weather information for mackay,au
    Getting Record 67
    Processing weather information for ilulissat,gl
    Getting Record 68
    Processing weather information for maputo,mz
    Getting Record 69
    Processing weather information for la ronge,ca
    Getting Record 70
    Processing weather information for holetown,bb
    Getting Record 71
    Processing weather information for kodiak,us
    Getting Record 72
    Processing weather information for rocha,uy
    Getting Record 73
    Processing weather information for georgetown,sh
    Getting Record 74
    Processing weather information for iracoubo,gf
    Getting Record 75
    Processing weather information for sterling,us
    Getting Record 76
    Processing weather information for tura,ru
    Getting Record 77
    Processing weather information for erenhot,cn
    Getting Record 78
    Processing weather information for hermanus,za
    Getting Record 79
    Processing weather information for guerrero negro,mx
    Getting Record 80
    Processing weather information for fulitun,cn
    Getting Record 81
    Processing weather information for san lorenzo,ar
    Getting Record 82
    Processing weather information for pisco,pe
    Getting Record 83
    Processing weather information for coquimbo,cl
    Getting Record 84
    Processing weather information for galbshtadt,ru
    Getting Record 85
    Processing weather information for manali,in
    Getting Record 86
    Processing weather information for opuwo,na
    Getting Record 87
    Processing weather information for formosa,ar
    Getting Record 88
    Processing weather information for uch,pk
    Getting Record 89
    Processing weather information for chimore,bo
    Getting Record 90
    Processing weather information for odweyne,so
    Getting Record 91
    Processing weather information for sur,om
    Getting Record 92
    Processing weather information for inhambane,mz
    Getting Record 93
    Processing weather information for kaitangata,nz
    Getting Record 94
    Processing weather information for ponta do sol,cv
    Getting Record 95
    Processing weather information for cairns,au
    Getting Record 96
    Processing weather information for airai,pw
    Getting Record 97
    Processing weather information for lebu,cl
    Getting Record 98
    Processing weather information for labutta,mm
    Getting Record 99
    Processing weather information for clyde river,ca
    Getting Record 100
    Processing weather information for kruisfontein,za
    Getting Record 101
    Processing weather information for yamada,jp
    Getting Record 102
    Processing weather information for qaanaaq,gl
    Getting Record 103
    Processing weather information for isangel,vu
    Getting Record 104
    Processing weather information for bambous virieux,mu
    Getting Record 105
    Processing weather information for tuatapere,nz
    Getting Record 106
    Processing weather information for vestmannaeyjar,is
    Getting Record 107
    Processing weather information for vardo,no
    Getting Record 108
    Processing weather information for victoria,sc
    Getting Record 109
    Processing weather information for tubruq,ly
    Getting Record 110
    Processing weather information for mirandola,it
    Getting Record 111
    Processing weather information for hilo,us
    Getting Record 112
    Processing weather information for khatanga,ru
    Getting Record 113
    Processing weather information for tasiilaq,gl
    Getting Record 114
    Processing weather information for upernavik,gl
    Getting Record 115
    Processing weather information for hasaki,jp
    Getting Record 116
    Processing weather information for nikolskoye,ru
    Getting Record 117
    Processing weather information for serra,br
    Getting Record 118
    Processing weather information for sioux lookout,ca
    Getting Record 119
    Processing weather information for tibiri,ne
    Getting Record 120
    Processing weather information for chokurdakh,ru
    Getting Record 121
    Processing weather information for manta,ec
    Getting Record 122
    Processing weather information for rio grande,br
    Getting Record 123
    Processing weather information for lompoc,us
    Getting Record 124
    Processing weather information for new norfolk,au
    Getting Record 125
    Processing weather information for kaeo,nz
    Getting Record 126
    Processing weather information for ancud,cl
    Getting Record 127
    Processing weather information for angoche,mz
    Getting Record 128
    Processing weather information for kasempa,zm
    Getting Record 129
    Processing weather information for iquique,cl
    Getting Record 130
    Processing weather information for shizunai,jp
    Getting Record 131
    Processing weather information for scarborough,gb
    Getting Record 132
    Processing weather information for norman wells,ca
    Getting Record 133
    Processing weather information for karachi,pk
    Getting Record 134
    Processing weather information for aksarka,ru
    Getting Record 135
    Processing weather information for geraldton,au
    Getting Record 136
    Processing weather information for salalah,om
    Getting Record 137
    Processing weather information for torbay,ca
    Getting Record 138
    Processing weather information for najran,sa
    Getting Record 139
    Processing weather information for sisimiut,gl
    Getting Record 140
    Processing weather information for severo-kurilsk,ru
    Getting Record 141
    Processing weather information for hithadhoo,mv
    Getting Record 142
    Processing weather information for meyungs,pw
    Getting Record 143
    Processing weather information for lumeje,ao
    Getting Record 144
    Processing weather information for kiunga,pg
    Getting Record 145
    Processing weather information for karangampel,id
    Getting Record 146
    Processing weather information for cayenne,gf
    Getting Record 147
    Processing weather information for ferrol,es
    Getting Record 148
    Processing weather information for shingu,jp
    Getting Record 149
    Processing weather information for grand centre,ca
    Getting Record 150
    Processing weather information for camacha,pt
    Getting Record 151
    Processing weather information for ponta do sol,pt
    Getting Record 152
    Processing weather information for soyo,ao
    Getting Record 153
    Processing weather information for shchigry,ru
    Getting Record 154
    Processing weather information for cedar city,us
    Getting Record 155
    Processing weather information for bo phloi,th
    Getting Record 156
    Processing weather information for itarema,br
    Getting Record 157
    Processing weather information for belushya guba,ru
    Getting Record 158
    Processing weather information for boatlaname,bw
    Getting Record 159
    Processing weather information for barrhead,ca
    Getting Record 160
    Processing weather information for kamsack,ca
    Getting Record 161
    Processing weather information for pacific grove,us
    Getting Record 162
    Processing weather information for khatra,in
    Getting Record 163
    Processing weather information for amga,ru
    Getting Record 164
    Processing weather information for mys shmidta,ru
    Getting Record 165
    Processing weather information for grand forks,us
    Getting Record 166
    Processing weather information for nemuro,jp
    Getting Record 167
    Processing weather information for ballitoville,za
    Getting Record 168
    Processing weather information for richards bay,za
    Getting Record 169
    Processing weather information for barentsburg,sj
    Getting Record 170
    Processing weather information for bonavista,ca
    Getting Record 171
    Processing weather information for katsuura,jp
    Getting Record 172
    Processing weather information for glendive,us
    Getting Record 173
    Processing weather information for codo,br
    Getting Record 174
    Processing weather information for port alfred,za
    Getting Record 175
    Processing weather information for karratha,au
    Getting Record 176
    Processing weather information for nanakuli,us
    Getting Record 177
    Processing weather information for mana,gf
    Getting Record 178
    Processing weather information for tateyama,jp
    Getting Record 179
    Processing weather information for bangassou,cf
    Getting Record 180
    Processing weather information for ahuimanu,us
    Getting Record 181
    Processing weather information for kavieng,pg
    Getting Record 182
    Processing weather information for coihaique,cl
    Getting Record 183
    Processing weather information for hirara,jp
    Getting Record 184
    Processing weather information for gayny,ru
    Getting Record 185
    Processing weather information for los llanos de aridane,es
    Getting Record 186
    Processing weather information for pitimbu,br
    Getting Record 187
    Processing weather information for chardara,kz
    Getting Record 188
    Processing weather information for lazaro cardenas,mx
    Getting Record 189
    Processing weather information for mitsamiouli,km
    Getting Record 190
    Processing weather information for rincon,an
    Getting Record 191
    Processing weather information for souillac,mu
    Getting Record 192
    Processing weather information for dzhebariki-khaya,ru
    Getting Record 193
    Processing weather information for nambucca heads,au
    Getting Record 194
    Processing weather information for milos,gr
    Getting Record 195
    Processing weather information for narsaq,gl
    Getting Record 196
    Processing weather information for dzaoudzi,yt
    Getting Record 197
    Processing weather information for kijang,id
    Getting Record 198
    Processing weather information for ruatoria,nz
    Getting Record 199
    Processing weather information for bismarck,us
    Getting Record 200
    Processing weather information for taolanaro,mg
    Getting Record 201
    Processing weather information for leningradskiy,ru
    Getting Record 202
    Processing weather information for port blair,in
    Getting Record 203
    Processing weather information for barcelos,br
    Getting Record 204
    Processing weather information for boddam,gb
    Getting Record 205
    Processing weather information for monrovia,lr
    Getting Record 206
    Processing weather information for harper,lr
    Getting Record 207
    Processing weather information for svetlogorsk,ru
    Getting Record 208
    Processing weather information for denpasar,id
    Getting Record 209
    Processing weather information for zhuhai,cn
    Getting Record 210
    Processing weather information for kainantu,pg
    Getting Record 211
    Processing weather information for sao filipe,cv
    Getting Record 212
    Processing weather information for louisbourg,ca
    Getting Record 213
    Processing weather information for saleaula,ws
    Getting Record 214
    Processing weather information for mount gambier,au
    Getting Record 215
    Processing weather information for taoudenni,ml
    Getting Record 216
    Processing weather information for durban,za
    Getting Record 217
    Processing weather information for ostrovnoy,ru
    Getting Record 218
    Processing weather information for margherita,in
    Getting Record 219
    Processing weather information for kamenka,ru
    Getting Record 220
    Processing weather information for port augusta,au
    Getting Record 221
    Processing weather information for cap-haitien,ht
    Getting Record 222
    Processing weather information for karibib,na
    Getting Record 223
    Processing weather information for oktyabrskiy,ru
    Getting Record 224
    Processing weather information for platteville,us
    Getting Record 225
    Processing weather information for andros,gr
    Getting Record 226
    Processing weather information for tiznit,ma
    Getting Record 227
    Processing weather information for keta,gh
    Getting Record 228
    Processing weather information for nguiu,au
    Getting Record 229
    Processing weather information for machico,pt
    Getting Record 230
    Processing weather information for novyye burasy,ru
    Getting Record 231
    Processing weather information for saint-francois,gp
    Getting Record 232
    Processing weather information for katangli,ru
    Getting Record 233
    Processing weather information for hamilton,bm
    Getting Record 234
    Processing weather information for ndioum,sn
    Getting Record 235
    Processing weather information for gat,ly
    Getting Record 236
    Processing weather information for padang,id
    Getting Record 237
    Processing weather information for gamboma,cg
    Getting Record 238
    Processing weather information for kegayli,uz
    Getting Record 239
    Processing weather information for samusu,ws
    Getting Record 240
    Processing weather information for pevek,ru
    Getting Record 241
    Processing weather information for san patricio,mx
    Getting Record 242
    Processing weather information for bathsheba,bb
    Getting Record 243
    Processing weather information for komsomolskiy,ru
    Getting Record 244
    Processing weather information for kibaya,tz
    Getting Record 245
    Processing weather information for orje,no
    Getting Record 246
    Processing weather information for mporokoso,zm
    Getting Record 247
    Processing weather information for fortuna,us
    Getting Record 248
    Processing weather information for aybak,af
    Getting Record 249
    Processing weather information for sitka,us
    Getting Record 250
    Processing weather information for cockburn town,tc
    Getting Record 251
    Processing weather information for olafsvik,is
    Getting Record 252
    Processing weather information for cidreira,br
    Getting Record 253
    Processing weather information for amderma,ru
    Getting Record 254
    Processing weather information for hearst,ca
    Getting Record 255
    Processing weather information for taltal,cl
    Getting Record 256
    Processing weather information for halalo,wf
    Getting Record 257
    Processing weather information for novikovo,ru
    Getting Record 258
    Processing weather information for faanui,pf
    Getting Record 259
    Processing weather information for tabuk,sa
    Getting Record 260
    Processing weather information for rio gallegos,ar
    Getting Record 261
    Processing weather information for matay,eg
    Getting Record 262
    Processing weather information for manadhoo,mv
    Getting Record 263
    Processing weather information for qasigiannguit,gl
    Getting Record 264
    Processing weather information for la palma,pa
    Getting Record 265
    Processing weather information for mokobeng,bw
    Getting Record 266
    Processing weather information for sentyabrskiy,ru
    Getting Record 267
    Processing weather information for qandala,so
    Getting Record 268
    Processing weather information for saldanha,za
    Getting Record 269
    Processing weather information for athabasca,ca
    Getting Record 270
    Processing weather information for san quintin,mx
    Getting Record 271
    Processing weather information for christchurch,nz
    Getting Record 272
    Processing weather information for saint-augustin,ca
    Getting Record 273
    Processing weather information for caravelas,br
    Getting Record 274
    Processing weather information for husavik,is
    Getting Record 275
    Processing weather information for bodden town,ky
    Getting Record 276
    Processing weather information for omboue,ga
    Getting Record 277
    Processing weather information for hongjiang,cn
    Getting Record 278
    Processing weather information for castro,cl
    Getting Record 279
    Processing weather information for nizhneyansk,ru
    Getting Record 280
    Processing weather information for umzimvubu,za
    Getting Record 281
    Processing weather information for ajdabiya,ly
    Getting Record 282
    Processing weather information for lagoa,pt
    Getting Record 283
    Processing weather information for kaoma,zm
    Getting Record 284
    Processing weather information for dutlwe,bw
    Getting Record 285
    Processing weather information for tual,id
    Getting Record 286
    Processing weather information for kargasok,ru
    Getting Record 287
    Processing weather information for othonoi,gr
    Getting Record 288
    Processing weather information for pochutla,mx
    Getting Record 289
    Processing weather information for batagay-alyta,ru
    Getting Record 290
    Processing weather information for nome,us
    Getting Record 291
    Processing weather information for deputatskiy,ru
    Getting Record 292
    Processing weather information for monte patria,cl
    Getting Record 293
    Processing weather information for caraz,pe
    Getting Record 294
    Processing weather information for sobolevo,ru
    Getting Record 295
    Processing weather information for goma,cd
    Getting Record 296
    Processing weather information for port lincoln,au
    Getting Record 297
    Processing weather information for nerchinskiy zavod,ru
    Getting Record 298
    Processing weather information for hobyo,so
    Getting Record 299
    Processing weather information for dera bugti,pk
    Getting Record 300
    Processing weather information for adzope,ci
    Getting Record 301
    Processing weather information for portland,au
    Getting Record 302
    Processing weather information for labuhan,id
    Getting Record 303
    Processing weather information for kambove,cd
    Getting Record 304
    Processing weather information for aykhal,ru
    Getting Record 305
    Processing weather information for san cristobal,ec
    Getting Record 306
    Processing weather information for say,ne
    Getting Record 307
    Processing weather information for broken hill,au
    Getting Record 308
    Processing weather information for ketchikan,us
    Getting Record 309
    Processing weather information for nicoya,cr
    Getting Record 310
    Processing weather information for jaipur hat,bd
    Getting Record 311
    Processing weather information for cabo san lucas,mx
    Getting Record 312
    Processing weather information for vitorino freire,br
    Getting Record 313
    Processing weather information for mount isa,au
    Getting Record 314
    Processing weather information for kidal,ml
    Getting Record 315
    Processing weather information for xingyi,cn
    Getting Record 316
    Processing weather information for koungheul,sn
    Getting Record 317
    Processing weather information for anadyr,ru
    Getting Record 318
    Processing weather information for banda aceh,id
    Getting Record 319
    Processing weather information for yangjiang,cn
    Getting Record 320
    Processing weather information for tecoanapa,mx
    Getting Record 321
    Processing weather information for southbridge,us
    Getting Record 322
    Processing weather information for sibu,my
    Getting Record 323
    Processing weather information for te anau,nz
    Getting Record 324
    Processing weather information for yenagoa,ng
    Getting Record 325
    Processing weather information for miranorte,br
    Getting Record 326
    Processing weather information for westport,nz
    Getting Record 327
    Processing weather information for kaka,tm
    Getting Record 328
    Processing weather information for binzhou,cn
    Getting Record 329
    Processing weather information for paradwip,in
    Getting Record 330
    Processing weather information for marcona,pe
    Getting Record 331
    Processing weather information for krasnoborsk,ru
    Getting Record 332
    Processing weather information for marawi,sd
    Getting Record 333
    Processing weather information for matara,lk
    Getting Record 334
    Processing weather information for touros,br
    Getting Record 335
    Processing weather information for ust-kuyga,ru
    Getting Record 336
    Processing weather information for satitoa,ws
    Getting Record 337
    Processing weather information for camrose,ca
    Getting Record 338
    Processing weather information for klaksvik,fo
    Getting Record 339
    Processing weather information for las vegas,us
    Getting Record 340
    Processing weather information for grand river south east,mu
    Getting Record 341
    Processing weather information for gorontalo,id
    Getting Record 342
    Processing weather information for brae,gb
    Getting Record 343
    Processing weather information for kutum,sd
    Getting Record 344
    Processing weather information for longyan,cn
    Getting Record 345
    Processing weather information for zhaotong,cn
    Getting Record 346
    Processing weather information for taunton,gb
    Getting Record 347
    Processing weather information for firminy,fr
    Getting Record 348
    Processing weather information for innisfail,au
    Getting Record 349
    Processing weather information for payo,ph
    Getting Record 350
    Processing weather information for port hardy,ca
    Getting Record 351
    Processing weather information for agva,tr
    Getting Record 352
    Processing weather information for tyup,kg
    Getting Record 353
    Processing weather information for henties bay,na
    Getting Record 354
    Processing weather information for hualmay,pe
    Getting Record 355
    Processing weather information for tshikapa,cd
    Getting Record 356
    Processing weather information for nyimba,zm
    Getting Record 357
    Processing weather information for kyaikto,mm
    Getting Record 358
    Processing weather information for verkhoyansk,ru
    Getting Record 359
    Processing weather information for kahului,us
    Getting Record 360
    Processing weather information for mandan,us
    Getting Record 361
    Processing weather information for mecca,sa
    Getting Record 362
    Processing weather information for tomatlan,mx
    Getting Record 363
    Processing weather information for okhotsk,ru
    Getting Record 364
    Processing weather information for pontianak,id
    Getting Record 365
    Processing weather information for palabuhanratu,id
    Getting Record 366
    Processing weather information for usinsk,ru
    Getting Record 367
    Processing weather information for duminichi,ru
    Getting Record 368
    Processing weather information for miandrivazo,mg
    Getting Record 369
    Processing weather information for staryy biser,ru
    Getting Record 370
    Processing weather information for sahrak,af
    Getting Record 371
    Processing weather information for don sak,th
    Getting Record 372
    Processing weather information for esperance,au
    Getting Record 373
    Processing weather information for makakilo city,us
    Getting Record 374
    Processing weather information for qazvin,ir
    Getting Record 375
    Processing weather information for qaqortoq,gl
    Getting Record 376
    Processing weather information for wamba,cd
    Getting Record 377
    Processing weather information for uhlove,ua
    Getting Record 378
    Processing weather information for hornepayne,ca
    Getting Record 379
    Processing weather information for marsh harbour,bs
    Getting Record 380
    Processing weather information for svetlaya,ru
    Getting Record 381
    Processing weather information for iqaluit,ca
    Getting Record 382
    Processing weather information for lasa,cn
    Getting Record 383
    Processing weather information for tupancireta,br
    Getting Record 384
    Processing weather information for naze,jp
    Getting Record 385
    Processing weather information for port macquarie,au
    Getting Record 386
    Processing weather information for resen,mk
    Getting Record 387
    Processing weather information for colares,pt
    Getting Record 388
    Processing weather information for hambantota,lk
    Getting Record 389
    Processing weather information for flin flon,ca
    Getting Record 390
    Processing weather information for mogadishu,so
    Getting Record 391
    Processing weather information for belaya gora,ru
    Getting Record 392
    Processing weather information for longyearbyen,sj
    Getting Record 393
    Processing weather information for asau,tv
    Getting Record 394
    Processing weather information for toungoo,mm
    Getting Record 395
    Processing weather information for fort nelson,ca
    Getting Record 396
    Processing weather information for caruray,ph
    Getting Record 397
    Processing weather information for watertown,us
    Getting Record 398
    Processing weather information for tessalit,ml
    Getting Record 399
    Processing weather information for san carlos de bariloche,ar
    Getting Record 400
    Processing weather information for aklavik,ca
    Getting Record 401
    Processing weather information for hammerfest,no
    Getting Record 402
    Processing weather information for kailua,us
    Getting Record 403
    Processing weather information for saint-joseph,re
    Getting Record 404
    Processing weather information for meaux,fr
    Getting Record 405
    Processing weather information for ukiah,us
    Getting Record 406
    Processing weather information for porto walter,br
    Getting Record 407
    Processing weather information for berdigestyakh,ru
    Getting Record 408
    Processing weather information for tilichiki,ru
    Getting Record 409
    Processing weather information for mnogovershinnyy,ru
    Getting Record 410
    Processing weather information for bereda,so
    Getting Record 411
    Processing weather information for tadine,nc
    Getting Record 412
    Processing weather information for buariki,ki
    Getting Record 413
    Processing weather information for junin,ar
    Getting Record 414
    Processing weather information for mayor pablo lagerenza,py
    Getting Record 415
    Processing weather information for montepuez,mz
    Getting Record 416
    Processing weather information for lubben,de
    Getting Record 417
    Processing weather information for asfi,ma
    Getting Record 418
    Processing weather information for beidao,cn
    Getting Record 419
    Processing weather information for sembe,cg
    Getting Record 420
    Processing weather information for yabrud,sy
    Getting Record 421
    Processing weather information for luderitz,na
    Getting Record 422
    Processing weather information for san fernando,mx
    Getting Record 423
    Processing weather information for viligili,mv
    Getting Record 424
    Processing weather information for shache,cn
    Getting Record 425
    Processing weather information for stolin,by
    Getting Record 426
    Processing weather information for seoul,kr
    Getting Record 427
    Processing weather information for nizwa,om
    Getting Record 428
    Processing weather information for norton shores,us
    Getting Record 429
    Processing weather information for birao,cf
    Getting Record 430
    Processing weather information for bubaque,gw
    Getting Record 431
    Processing weather information for bayangol,ru
    Getting Record 432
    Processing weather information for teknaf,bd
    Getting Record 433
    Processing weather information for santiago del estero,ar
    Getting Record 434
    Processing weather information for pombas,cv
    Getting Record 435
    Processing weather information for port-gentil,ga
    Getting Record 436
    Processing weather information for songea,tz
    Getting Record 437
    Processing weather information for houma,us
    Getting Record 438
    Processing weather information for puerto cabezas,ni
    Getting Record 439
    Processing weather information for santa isabel,mx
    Getting Record 440
    Processing weather information for heyang,cn
    Getting Record 441
    Processing weather information for belmonte,br
    Getting Record 442
    Processing weather information for taburao,ki
    Getting Record 443
    Processing weather information for krasnoselkup,ru
    Getting Record 444
    Processing weather information for novosibirsk,ru
    Getting Record 445
    Processing weather information for jati,pk
    Getting Record 446
    Processing weather information for yingkou,cn
    Getting Record 447
    Processing weather information for nha trang,vn
    Getting Record 448
    Processing weather information for craig,us
    Getting Record 449
    Processing weather information for bethel,us
    Getting Record 450
    Processing weather information for hainburg,at
    Getting Record 451
    Processing weather information for fuerte olimpo,py
    Getting Record 452
    Processing weather information for constitucion,cl
    Getting Record 453
    Processing weather information for shimoda,jp
    Getting Record 454
    Processing weather information for dakar,sn
    Getting Record 455
    Processing weather information for codrington,ag
    Getting Record 456
    Processing weather information for mayo,ca
    Getting Record 457
    Processing weather information for tsihombe,mg
    Getting Record 458
    Processing weather information for beyneu,kz
    Getting Record 459
    Processing weather information for boyolangu,id
    Getting Record 460
    Processing weather information for novolabinskaya,ru
    Getting Record 461
    Processing weather information for haibowan,cn
    Getting Record 462
    Processing weather information for bilibino,ru
    Getting Record 463
    Processing weather information for morropon,pe
    Getting Record 464
    Processing weather information for keetmanshoop,na
    Getting Record 465
    Processing weather information for ponta delgada,pt
    Getting Record 466
    Processing weather information for baculin,ph
    Getting Record 467
    Processing weather information for sao joao da barra,br
    Getting Record 468
    Processing weather information for megion,ru
    Getting Record 469
    Processing weather information for spencer,us
    Getting Record 470
    Processing weather information for truro,ca
    Getting Record 471
    Processing weather information for houlton,us
    Getting Record 472
    Processing weather information for itacarambi,br
    Getting Record 473
    Processing weather information for madingou,cg
    Getting Record 474
    Processing weather information for port hueneme,us
    Getting Record 475
    Processing weather information for lichinga,mz
    Getting Record 476
    Processing weather information for nishihara,jp
    Getting Record 477
    Processing weather information for alta floresta,br
    Getting Record 478
    Processing weather information for beringovskiy,ru
    Getting Record 479
    Processing weather information for kabugao,ph
    Getting Record 480
    Processing weather information for attawapiskat,ca
    Getting Record 481
    Processing weather information for pangnirtung,ca
    Getting Record 482
    Processing weather information for havre-saint-pierre,ca
    Getting Record 483
    Processing weather information for tocopilla,cl
    Getting Record 484
    Processing weather information for suez,eg
    Getting Record 485
    Processing weather information for saint george,bm
    Getting Record 486
    Processing weather information for liverpool,ca
    Getting Record 487
    Processing weather information for charlottetown,ca
    Getting Record 488
    Processing weather information for dombarovskiy,ru
    Getting Record 489
    Processing weather information for oranjemund,na
    Getting Record 490
    Processing weather information for roma,au
    Getting Record 491
    Processing weather information for isla mujeres,mx
    Getting Record 492
    Processing weather information for ijaki,ki
    Getting Record 493
    Processing weather information for agdam,az
    Getting Record 494
    Processing weather information for strezhevoy,ru
    Getting Record 495
    Processing weather information for eenhana,na
    Getting Record 496
    Processing weather information for dalvik,is
    Getting Record 497
    Processing weather information for hailar,cn
    Getting Record 498
    Processing weather information for celestun,mx
    Getting Record 499
    Processing weather information for skjervoy,no
    Getting Record 500
    Processing weather information for thinadhoo,mv
    Getting Record 501
    Processing weather information for nouadhibou,mr
    Getting Record 502
    Processing weather information for nazare,br
    Getting Record 503
    Processing weather information for sukabumi,id
    Getting Record 504
    Processing weather information for kautokeino,no
    Getting Record 505
    Processing weather information for baisha,cn
    Getting Record 506
    Processing weather information for port hedland,au
    Getting Record 507
    Processing weather information for bhikangaon,in
    Getting Record 508
    Processing weather information for isabela,us
    Getting Record 509
    Processing weather information for george,za
    Getting Record 510
    Processing weather information for gumdag,tm
    Getting Record 511
    Processing weather information for zyryanka,ru
    Getting Record 512
    Processing weather information for shambu,et
    Getting Record 513
    Processing weather information for santa cruz del sur,cu
    Getting Record 514
    Processing weather information for stupino,ru
    Getting Record 515
    Processing weather information for quatre cocos,mu
    Getting Record 516
    Processing weather information for tezu,in
    Getting Record 517
    Processing weather information for ibate,br
    Getting Record 518
    Processing weather information for goure,ne
    Getting Record 519
    Processing weather information for jiaonan,cn
    Getting Record 520
    Processing weather information for yuksekova,tr
    Getting Record 521
    Processing weather information for tarauaca,br
    Getting Record 522
    Processing weather information for marica,br
    Getting Record 523
    Processing weather information for hanover,us
    Getting Record 524
    Processing weather information for jumla,np
    Getting Record 525
    Processing weather information for cernat,ro
    Getting Record 526
    Processing weather information for amberley,nz
    Getting Record 527
    Processing weather information for nong chik,th
    Getting Record 528
    Processing weather information for vila franca do campo,pt
    Getting Record 529
    Processing weather information for sistranda,no
    Getting Record 530
    Processing weather information for gunnedah,au
    Getting Record 531
    Processing weather information for marzuq,ly
    Getting Record 532
    Processing weather information for nanortalik,gl
    Getting Record 533
    Processing weather information for vangaindrano,mg
    Getting Record 534
    Processing weather information for petatlan,mx
    Getting Record 535
    Processing weather information for aguimes,es
    Getting Record 536
    Processing weather information for ilula,tz
    Getting Record 537
    Processing weather information for sedelnikovo,ru
    Getting Record 538
    Processing weather information for yerbogachen,ru
    Getting Record 539
    Processing weather information for tres picos,mx
    Getting Record 540
    Processing weather information for maniitsoq,gl
    Getting Record 541
    Processing weather information for samarai,pg
    Getting Record 542
    Processing weather information for sandakan,my
    Getting Record 543
    Processing weather information for yelizovo,ru
    Getting Record 544
    Processing weather information for saint anthony,ca
    Getting Record 545
    Processing weather information for colorado springs,us
    Getting Record 546
    Processing weather information for waitati,nz
    Getting Record 547
    Processing weather information for coahuayana,mx
    Getting Record 548
    Processing weather information for ulaanbaatar,mn
    Getting Record 549
    Processing weather information for cartagena del chaira,co
    Getting Record 550
    Processing weather information for adelaide,au
    Getting Record 551
    Processing weather information for olga,ru
    Getting Record 552
    Processing weather information for rawson,ar
    Getting Record 553
    Processing weather information for caraballeda,ve
    Getting Record 554
    Processing weather information for tashla,ru
    Getting Record 555
    Processing weather information for srednekolymsk,ru
    Getting Record 556
    Processing weather information for boa vista,br
    Getting Record 557
    Processing weather information for vaxjo,se
    Getting Record 558
    Processing weather information for tobias barreto,br
    Getting Record 559
    Processing weather information for ban tak,th
    Getting Record 560
    Processing weather information for ballina,au
    Getting Record 561
    Processing weather information for ardahan,tr
    Getting Record 562
    Processing weather information for bowen,au
    Getting Record 563
    Processing weather information for ritto,jp
    Getting Record 564
    Processing weather information for surt,ly
    Getting Record 565
    Processing weather information for walvis bay,na
    Getting Record 566
    Processing weather information for tiksi,ru
    Getting Record 567
    Processing weather information for longkou,cn
    Getting Record 568
    Processing weather information for emerald,au
    Getting Record 569
    Processing weather information for bol,td
    Getting Record 570
    Processing weather information for kemijarvi,fi
    Getting Record 571
    Processing weather information for kodinsk,ru
    Getting Record 572
    Processing weather information for poyarkovo,ru
    Getting Record 573
    Processing weather information for saint-pierre,pm
    Getting Record 574
    Processing weather information for eten,pe
    Getting Record 575
    Processing weather information for avera,pf
    Getting Record 576
    Processing weather information for tarudant,ma
    Getting Record 577
    Processing weather information for sheridan,us
    Getting Record 578
    Processing weather information for phan rang,vn
    Getting Record 579
    Processing weather information for radcliff,us
    Getting Record 580
    Processing weather information for oistins,bb
    Getting Record 581
    Processing weather information for teya,ru
    Getting Record 582
    Processing weather information for milici,ba
    Getting Record 583
    Processing weather information for alakurtti,ru
    Getting Record 584
    Processing weather information for leshan,cn
    Getting Record 585
    Processing weather information for sao sebastiao,br
    Getting Record 586
    Processing weather information for yialos,gr
    Getting Record 587
    Processing weather information for tumannyy,ru
    Getting Record 588
    Processing weather information for altamont,us
    Getting Record 589
    Processing weather information for buala,sb
    Getting Record 590
    Processing weather information for clervaux,lu
    Getting Record 591
    Processing weather information for chute-aux-outardes,ca
    Getting Record 592
    Processing weather information for samana,do
    Getting Record 593
    Processing weather information for chiredzi,zw
    Getting Record 594
    Processing weather information for edson,ca
    Getting Record 595
    Processing weather information for dicabisagan,ph
    Getting Record 596
    Processing weather information for axim,gh
    Getting Record 597
    Processing weather information for odienne,ci
    Getting Record 598
    Processing weather information for khanpur,pk
    Getting Record 599
    Processing weather information for yulara,au
    Getting Record 600
    Processing weather information for vostok,ru
    Getting Record 601
    Processing weather information for lyngseidet,no
    Getting Record 602
    Processing weather information for red bluff,us
    Getting Record 603
    Processing weather information for nantucket,us
    Getting Record 604
    Processing weather information for sedkyrkeshch,ru
    Getting Record 605
    Processing weather information for samalaeulu,ws
    Getting Record 606
    Processing weather information for kavaratti,in
    Getting Record 607
    Processing weather information for ibra,om
    Getting Record 608
    Processing weather information for khandyga,ru
    Getting Record 609
    Processing weather information for datong,cn
    Getting Record 610
    Processing weather information for havoysund,no
    Getting Record 611
    Processing weather information for dalbandin,pk
    Getting Record 612
    Processing weather information for honiara,sb
    Getting Record 613
    Processing weather information for dunedin,nz
    Getting Record 614
    Processing weather information for mantua,cu
    Getting Record 615
    Processing weather information for evensk,ru
    Getting Record 616
    Processing weather information for kariba,zw
    Getting Record 617
    Processing weather information for cabedelo,br
    Getting Record 618
    Processing weather information for kalaleh,ir
    Getting Record 619
    Processing weather information for almaznyy,ru
    Getting Record 620
    Processing weather information for osoyoos,ca
    Getting Record 621
    Processing weather information for paitan,ph
    Getting Record 622
    Processing weather information for pleshanovo,ru
    Getting Record 623
    Processing weather information for port-cartier,ca
    Getting Record 624
    Processing weather information for rabo de peixe,pt
    Getting Record 625
    Processing weather information for kudahuvadhoo,mv
    Getting Record 626
    Processing weather information for awjilah,ly
    Getting Record 627
    Processing weather information for two rivers,us
    Getting Record 628
    Processing weather information for lubaczow,pl
    Getting Record 629
    Processing weather information for volovo,ru
    Getting Record 630
    Processing weather information for sidi ali,dz
    Getting Record 631
    Processing weather information for natchitoches,us
    Getting Record 632
    Processing weather information for porto novo,cv
    Getting Record 633
    Processing weather information for nalut,ly
    Getting Record 634
    Processing weather information for caohai,cn
    Getting Record 635
    Processing weather information for shorapani,ge
    Getting Record 636
    Processing weather information for simao,cn
    Getting Record 637
    Processing weather information for college,us
    Getting Record 638
    Processing weather information for revelstoke,ca
    Getting Record 639
    Processing weather information for bima,id
    Getting Record 640
    Processing weather information for dudinka,ru
    Getting Record 641
    Processing weather information for sambava,mg
    Getting Record 642
    Processing weather information for namatanai,pg
    Getting Record 643
    Processing weather information for songjianghe,cn
    Getting Record 644
    Processing weather information for namibe,ao
    Getting Record 645
    Processing weather information for aransas pass,us
    Getting Record 646
    Processing weather information for roald,no
    Getting Record 647
    Processing weather information for dokka,no
    Getting Record 648
    Processing weather information for leh,in
    Getting Record 649
    Processing weather information for kyzyl-suu,kg
    Getting Record 650
    Processing weather information for morondava,mg
    Getting Record 651
    Processing weather information for martapura,id
    Getting Record 652
    Processing weather information for tingi,tz
    Getting Record 653
    Processing weather information for wilmington,us
    Getting Record 654
    Processing weather information for liepaja,lv
    Getting Record 655
    Processing weather information for khash,ir
    Getting Record 656
    Processing weather information for loveland,us
    Getting Record 657
    Processing weather information for yarmouth,ca
    Getting Record 658
    Processing weather information for sao jose da coroa grande,br
    Getting Record 659
    Processing weather information for japura,br
    Getting Record 660
    Processing weather information for zhengjiatun,cn
    Getting Record 661
    Processing weather information for aginskoye,ru
    Getting Record 662
    Processing weather information for lata,sb
    Getting Record 663
    Processing weather information for petropavlovsk-kamchatskiy,ru
    Getting Record 664
    Processing weather information for devils lake,us
    Getting Record 665
    Processing weather information for toyooka,jp
    Getting Record 666
    Processing weather information for santa maria,cv
    Getting Record 667
    Processing weather information for sosenskiy,ru
    Getting Record 668
    Processing weather information for merritt island,us
    Getting Record 669
    Processing weather information for malatya,tr
    Getting Record 670
    Processing weather information for bayan,kw
    Getting Record 671
    Processing weather information for vestmanna,fo
    Getting Record 672
    Processing weather information for kisesa,tz
    Getting Record 673
    Processing weather information for teno,cl
    Getting Record 674
    Processing weather information for valkeakoski,fi
    Getting Record 675
    Processing weather information for qujing,cn
    Getting Record 676
    Processing weather information for kangaba,ml
    Getting Record 677
    Processing weather information for ostersund,se
    Getting Record 678
    Processing weather information for yumen,cn
    Getting Record 679
    Processing weather information for khuzhir,ru
    Getting Record 680
    Processing weather information for dipkarpaz,cy
    Getting Record 681
    Processing weather information for winnemucca,us
    Getting Record 682
    Processing weather information for suicheng,cn
    Getting Record 683
    Processing weather information for abdanan,ir
    Getting Record 684
    Processing weather information for bardiyah,ly
    Getting Record 685
    Processing weather information for bad wildungen,de
    Getting Record 686
    Processing weather information for alice springs,au
    Getting Record 687
    Processing weather information for gremyachye,ru
    Getting Record 688
    Processing weather information for dingle,ie
    Getting Record 689
    Processing weather information for puerto escondido,mx
    Getting Record 690
    Processing weather information for metro,id
    Getting Record 691
    Processing weather information for arroio grande,br
    Getting Record 692
    Processing weather information for malwan,in
    Getting Record 693
    Processing weather information for tabou,ci
    Getting Record 694
    Processing weather information for male,mv
    Getting Record 695
    Processing weather information for marsassoum,sn
    Getting Record 696
    Processing weather information for nabire,id
    Getting Record 697
    Processing weather information for lagunas,pe
    Getting Record 698
    Processing weather information for kaupanger,no
    Getting Record 699
    Processing weather information for vitim,ru
    Getting Record 700
    Processing weather information for tahoua,ne
    Getting Record 701
    Processing weather information for hay river,ca
    Getting Record 702
    Processing weather information for san vicente,ph
    Getting Record 703
    Processing weather information for wanning,cn
    Getting Record 704
    Processing weather information for anchorage,us
    Getting Record 705
    Processing weather information for garowe,so
    Getting Record 706
    Processing weather information for qeshm,ir
    Getting Record 707
    Processing weather information for grand gaube,mu
    Getting Record 708
    Processing weather information for huangnihe,cn
    Getting Record 709
    Processing weather information for mangrol,in
    Getting Record 710
    Processing weather information for hede,cn
    Getting Record 711
    Processing weather information for mahibadhoo,mv
    Getting Record 712
    Processing weather information for tarko-sale,ru
    Getting Record 713
    Processing weather information for eucaliptus,bo
    Getting Record 714
    Processing weather information for sayyan,ye
    Getting Record 715
    Processing weather information for kilindoni,tz
    Getting Record 716
    Processing weather information for lalpur,in
    Getting Record 717
    Processing weather information for mubi,ng
    Getting Record 718
    Processing weather information for lamu,ke
    Getting Record 719
    Processing weather information for cangucu,br
    Getting Record 720
    Processing weather information for lokosovo,ru
    Getting Record 721
    Processing weather information for araguacu,br
    Getting Record 722
    Processing weather information for meulaboh,id
    Getting Record 723
    Processing weather information for segezha,ru
    Getting Record 724
    Processing weather information for mazamitla,mx
    Getting Record 725
    Processing weather information for glenwood springs,us
    Getting Record 726
    Processing weather information for sangar,ru
    Getting Record 727
    Processing weather information for faya,td
    Getting Record 728
    Processing weather information for ciudad guayana,ve
    Getting Record 729
    Processing weather information for beloha,mg
    Getting Record 730
    Processing weather information for palmer,us
    Getting Record 731
    Processing weather information for egvekinot,ru
    Getting Record 732
    Processing weather information for keti bandar,pk
    Getting Record 733
    Processing weather information for urazovo,ru
    Getting Record 734
    Processing weather information for champerico,gt
    Getting Record 735
    Processing weather information for kassala,sd
    Getting Record 736
    Processing weather information for joson,ph
    Getting Record 737
    Processing weather information for cuenca,es
    Getting Record 738
    Processing weather information for madera,mx
    Getting Record 739
    Processing weather information for harer,et
    Getting Record 740
    Processing weather information for kieta,pg
    Getting Record 741
    Processing weather information for puerto narino,co
    Getting Record 742
    Processing weather information for ocos,gt
    Getting Record 743
    Processing weather information for playas,ec
    Getting Record 744
    Processing weather information for umm lajj,sa
    Getting Record 745
    Processing weather information for ust-omchug,ru
    Getting Record 746
    Processing weather information for karkaralinsk,kz
    Getting Record 747
    Processing weather information for tobolsk,ru
    Getting Record 748
    Processing weather information for elizabeth city,us
    Getting Record 749
    Processing weather information for arlit,ne
    Getting Record 750
    Processing weather information for kurchum,kz
    Getting Record 751
    Processing weather information for gonbad-e qabus,ir
    Getting Record 752
    Processing weather information for orda,ru
    Getting Record 753
    Processing weather information for enshi,cn
    Getting Record 754
    Processing weather information for tazmalt,dz
    Getting Record 755
    Processing weather information for machala,ec
    Getting Record 756
    Processing weather information for blackfoot,us
    Getting Record 757
    Processing weather information for siedlce,pl
    Getting Record 758
    Processing weather information for warqla,dz
    Getting Record 759
    Processing weather information for merauke,id
    Getting Record 760
    Processing weather information for sosnogorsk,ru
    Getting Record 761
    Processing weather information for bud,no
    Getting Record 762
    Processing weather information for lufilufi,ws
    Getting Record 763
    Processing weather information for fairbanks,us
    Getting Record 764
    Processing weather information for awbari,ly
    Getting Record 765
    Processing weather information for ossora,ru
    Getting Record 766
    Processing weather information for riviere-au-renard,ca
    Getting Record 767
    Processing weather information for abha,sa
    Getting Record 768
    Processing weather information for astoria,us
    Getting Record 769
    Processing weather information for lamont,ca
    Getting Record 770
    Processing weather information for sechura,pe
    Getting Record 771
    Processing weather information for biak,id
    Getting Record 772
    Processing weather information for portree,gb
    Getting Record 773
    Processing weather information for oyo,ng
    Getting Record 774
    Processing weather information for borama,so
    Getting Record 775
    Processing weather information for ust-ilimsk,ru
    Getting Record 776
    Processing weather information for terrasini,it
    Getting Record 777
    Processing weather information for izumo,jp
    Getting Record 778
    Processing weather information for vila,vu
    Getting Record 779
    Processing weather information for bolshiye uki,ru
    Getting Record 780
    Processing weather information for kampot,kh
    Getting Record 781
    Processing weather information for hervey bay,au
    Getting Record 782
    Processing weather information for ahome,mx
    Getting Record 783
    Processing weather information for yuanping,cn
    Getting Record 784
    Processing weather information for halmstad,se
    Getting Record 785
    Processing weather information for khuchni,ru
    Getting Record 786
    Processing weather information for christiana,za
    Getting Record 787
    Processing weather information for phan thiet,vn
    Getting Record 788
    Processing weather information for zhigansk,ru
    Getting Record 789
    Processing weather information for bolungarvik,is
    Getting Record 790
    Processing weather information for addi ugri,er
    Getting Record 791
    Processing weather information for osinovo,ru
    Getting Record 792
    Processing weather information for ternate,id
    Getting Record 793
    Processing weather information for saravan,la
    Getting Record 794
    Processing weather information for labuan,my
    Getting Record 795
    Processing weather information for tazovskiy,ru
    Getting Record 796
    Processing weather information for sorland,no
    Getting Record 797
    Processing weather information for half moon bay,us
    Getting Record 798
    Processing weather information for isla vista,us
    Getting Record 799
    Processing weather information for yar-sale,ru
    Getting Record 800
    Processing weather information for dauriya,ru
    Getting Record 801
    Processing weather information for chicama,pe
    Getting Record 802
    Processing weather information for salta,ar
    Getting Record 803
    Processing weather information for erdemli,tr
    Getting Record 804
    Processing weather information for izhma,ru
    Getting Record 805
    Processing weather information for cururupu,br
    Getting Record 806
    Processing weather information for burica,pa
    Getting Record 807
    Processing weather information for khandagayty,ru
    Getting Record 808
    Processing weather information for sao gotardo,br
    Getting Record 809
    Processing weather information for kleck,by
    Getting Record 810
    Processing weather information for saint-georges,gf
    Getting Record 811
    Processing weather information for cootamundra,au
    Getting Record 812
    Processing weather information for suleja,ng
    Getting Record 813
    Processing weather information for muroto,jp
    Getting Record 814
    Processing weather information for inderborskiy,kz
    Getting Record 815
    Processing weather information for malakal,sd
    Getting Record 816
    Processing weather information for istok,ru
    Getting Record 817
    Processing weather information for sorvag,fo
    Getting Record 818
    Processing weather information for gaya,ne
    Getting Record 819
    Processing weather information for kuryk,kz
    Getting Record 820
    Processing weather information for raudeberg,no
    Getting Record 821
    Processing weather information for albacete,es
    Getting Record 822
    Processing weather information for azimur,ma
    Getting Record 823
    Processing weather information for gangelt,de
    Getting Record 824
    Processing weather information for coari,br
    Getting Record 825
    Processing weather information for hofn,is
    Getting Record 826
    Processing weather information for sovetskaya,ru
    Getting Record 827
    Processing weather information for ixtapa,mx
    Getting Record 828
    Processing weather information for nhulunbuy,au
    Getting Record 829
    Processing weather information for zig,az
    Getting Record 830
    Processing weather information for tigil,ru
    Getting Record 831
    Processing weather information for sakakah,sa
    Getting Record 832
    Processing weather information for chatra,in
    Getting Record 833
    Processing weather information for polovinnoye,ru
    Getting Record 834
    Processing weather information for san jeronimo,mx
    Getting Record 835
    Processing weather information for kununurra,au
    Getting Record 836
    Processing weather information for galesong,id
    Getting Record 837
    Processing weather information for smithers,ca
    Getting Record 838
    Processing weather information for nuevo progreso,mx
    Getting Record 839
    Processing weather information for kitui,ke
    Getting Record 840
    Processing weather information for torit,sd
    Getting Record 841
    Processing weather information for plettenberg bay,za
    Getting Record 842
    Processing weather information for tuljapur,in
    Getting Record 843
    Processing weather information for comodoro rivadavia,ar
    Getting Record 844
    Processing weather information for iskateley,ru
    Getting Record 845
    Processing weather information for daru,pg
    Getting Record 846
    Processing weather information for alugan,ph
    Getting Record 847
    Processing weather information for broome,au
    Getting Record 848
    Processing weather information for ulkan,ru
    Getting Record 849
    Processing weather information for tabiauea,ki



```python
# Drop cities with missing information
cities_list = cities_list.dropna()
cities_list = cities_list.sample(500)
cities_list.shape
#cities_list.head()
```




    (500, 8)




```python
# Build a scatter plot for each data type
# You must use proper labeling of your plots, including aspects like:
# Plot Titles (with date of analysis) and Axes Labels.

# Scatterplot Temperature (F) vs. Latitude

plt.scatter(cities_list["Latitude"], cities_list["Temperature"], marker="o",
            Facecolors="Red", Edgecolors="Gray", alpha=.5)

# Incorporate the other graph properties
plt.title("Temperature by Latitude in Random Sample of World Cities December, 3 2017")
plt.ylabel("Temperature (F)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim( -90, 90)
plt.axvline(0, color='black',alpha=0.5)

# Save the figure
plt.savefig("TemperatureInWorldCities.png")

# Show plot
plt.show()
```


![png](output_9_0.png)



```python
# Scatterplot Humidity (%) vs. Latitude

plt.scatter(cities_list["Latitude"], cities_list["Humidity"], marker="o",
            Facecolors="Green", Edgecolors="Gray", alpha=.5)

# Incorporate the other graph properties
plt.title("Humidity by Latitude in Random Sample of World Cities December, 3 2017")
plt.ylabel("Humidity %")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim( -90, 90)
plt.axvline(0, color='black',alpha=0.5)

# Save the figure
plt.savefig("HumidityInWorldCities.png")

# Show plot
plt.show()
```


![png](output_10_0.png)



```python
# Scatterplot Cloudiness (%) vs. Latitude

plt.scatter(cities_list["Latitude"], cities_list["Cloudiness"], marker="o",
            Facecolors="Gray", Edgecolors="Brown", alpha=.5)

# Incorporate the other graph properties
plt.title("Cloudiness by Latitude in Random Sample of World Cities December, 3 2017")
plt.ylabel("Cloudiness")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim( -90, 90)
plt.axvline(0, color='black',alpha=0.5)

# Save the figure
plt.savefig("CloudinessInWorldCities.png")

# Show plot
plt.show()
```


![png](output_11_0.png)



```python
# Scatterplot Wind Speed (mph) vs. Latitude

plt.scatter(cities_list["Latitude"], cities_list["Wind Speed"], marker="o",
            Facecolors="Blue", Edgecolors="Gray", alpha=.5)

# Incorporate the other graph properties
plt.title("Wind Speed by Latitude in Random Sample of World Cities December, 3 2017")
plt.ylabel("Wind Speed (MPH)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim( -90, 90)

# Save the figure
plt.savefig("WindSpeedInWorldCities.png")

# Show plot
plt.show()
```


![png](output_12_0.png)



```python
# written description of three observable trends based on the data.

# 1. Temperatures do indeed get higher as you approach the equator
# as seen in arc from highest recorded temperatures peak at lower latitudes.
# The highest temps, those greater than 60, are between the 40th parallels.

# 2. The majority of wind speeds are less the 15 MPH and are distrubuted evenly across measured cities.

# 3. There is no readily conclusive pattern to cloudiness across measured cities other
# than they tend to cluster toward the edges. Most are in the upper or lower quartiles.
```


```python
# Save both a CSV of all data retrieved and png images for each scatter plot.

cities_list.to_csv("City_Weather_Sample_Data.csv")
```


```python
# You must include an exported markdown version of your Notebook called  README.md in your GitHub repository.
```
