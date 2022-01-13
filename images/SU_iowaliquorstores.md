

```python
%load_ext sql
%matplotlib inline

```

    The sql extension is already loaded. To reload it, use:
      %reload_ext sql
    


```python
import matplotlib.pyplot as plt
import zipfile
import pandas as pd
import geopandas
from sqlalchemy import create_engine
import sqlalchemy.sql
from shapely.geometry import Point, Polygon
import matplotlib
import numpy as np

from shapely import wkt
import squarify #used for treemap plot
```

## Datasets from other sources

* [A shapefile of the state boundaries from the US Census Bureau](https://www.census.gov/geo/maps-data/data/cbf/cbf_state.html)

* [Iowa tract boundaries from the US Census Bureau](https://catalog.data.gov/dataset/tiger-line-shapefile-2016-state-iowa-current-census-tract-state-based)
* [Iowa primary roads from the US Census Bureau](https://catalog.data.gov/dataset/tiger-line-shapefile-2015-state-iowa-primary-and-secondary-roads-state-based-shapefile)


```python
states = geopandas.read_file("zip://./cb_2017_us_state_5m.zip") # map of US -apply a filter for iowa 
iowa = geopandas.read_file('zip://./tl_2016_19_tract.zip') #provides polygon of each census tract in iowa
iowa_roads = geopandas.read_file('zip://./tl_2015_19_prisecroads.zip')#primary roads in iowa
```


```python
#Table generated from big query-liquor store GIS location
iowacityPoints = pd.read_csv('iowacityPoints.csv')

print(type(iowacityPoints.loc[0,'store_location']))
iowacityPoints['store_location'] = iowacityPoints['store_location'].apply(wkt.loads) #change to geometry datatype (import wkt)
print(type(iowacityPoints.loc[0,'store_location']))
```

    <class 'str'>
    <class 'shapely.geometry.point.Point'>
    


```python
iowacityPoints.head(3)
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
      <th>store_name</th>
      <th>zip_code</th>
      <th>store_location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hy-Vee Wine and Spirits / Iowa City</td>
      <td>52240</td>
      <td>POINT (-91.53046300000001 41.642764)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Hy-Vee Food Store / Iowa City</td>
      <td>52240</td>
      <td>POINT (-91.518868 41.676095)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fareway Stores #034 / Iowa City</td>
      <td>52240</td>
      <td>POINT (-91.48043 41.629617)</td>
    </tr>
  </tbody>
</table>
</div>




```python
#change to geopandas df with point to geometry datatype
gdf_iowacityP = geopandas.GeoDataFrame(iowacityPoints, geometry = 'store_location')
#gdf_iowacityP['store_location'].head(3)
gdf_iowacityP.head(3)

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
      <th>store_name</th>
      <th>zip_code</th>
      <th>store_location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hy-Vee Wine and Spirits / Iowa City</td>
      <td>52240</td>
      <td>POINT (-91.53046 41.64276)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Hy-Vee Food Store / Iowa City</td>
      <td>52240</td>
      <td>POINT (-91.51887 41.67609)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fareway Stores #034 / Iowa City</td>
      <td>52240</td>
      <td>POINT (-91.48043 41.62962)</td>
    </tr>
  </tbody>
</table>
</div>




```python

df_JCountyShape = iowa[iowa.COUNTYFP == '103']  #johnson county = 103

#must supply tracts which are part of Iowa city manually
tractName = [1,5,6,11,12,13,14,15,16,17,18.02,21,23]

tractName_str = []
for i in tractName:
    tractName_str.append(str(i))
tractName_str    

tractNameStr=[str(i) for i in tractName]
tractNameStr

pdf_iowa = pd.DataFrame()
#pdf_iowa = pd.DataFrame(columns=['STATEFP', 'COUNTYFP', 'TRACTCE','GEOID','NAME','NAMELSAD','MTFCC','FUNCSTAT','ALAND','AWATER','INTPTLAT','INTPTLON','geometry'])

for i in range(len(df_JCountyShape)):
    #print(df_JCountyShape.iloc[i,4])
    for name in tractNameStr:
        
        #print(name)
        if name==df_JCountyShape.iloc[i,4]:
            #print(name,' ', df_JCountyShape.iloc[i,4])
            x=df_JCountyShape.iloc[i,:]
            x=pd.DataFrame(x)
            xtranspose = x.T
            pdf_iowa = pd.concat([pdf_iowa, xtranspose])
            break
            
gdf_iowacity = geopandas.GeoDataFrame(pdf_iowa)
gdf_iowacity.head(1)
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
      <th>STATEFP</th>
      <th>COUNTYFP</th>
      <th>TRACTCE</th>
      <th>GEOID</th>
      <th>NAME</th>
      <th>NAMELSAD</th>
      <th>MTFCC</th>
      <th>FUNCSTAT</th>
      <th>ALAND</th>
      <th>AWATER</th>
      <th>INTPTLAT</th>
      <th>INTPTLON</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>64</th>
      <td>19</td>
      <td>103</td>
      <td>001802</td>
      <td>19103001802</td>
      <td>18.02</td>
      <td>Census Tract 18.02</td>
      <td>G5020</td>
      <td>S</td>
      <td>3352097</td>
      <td>78507</td>
      <td>+41.6366549</td>
      <td>-091.5242066</td>
      <td>POLYGON ((-91.53796 41.64070, -91.53797 41.641...</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt = gdf_iowacity.plot(color = 'red',edgecolor='grey')  

```


![png](output_8_0.png)



```python
ax = states[states.STATEFP == '19'].plot( edgecolor='k', figsize=(10,20))
gdf_iowacity.plot(ax=ax, color = 'red')
ax.set(xlim=(-97,-90), ylim=(40,44))
```




    [(40, 44), (-97, -90)]




![png](output_9_1.png)



```python
ax=states[states.STATEFP == '19'].plot(figsize=(10,20))
iowa.plot(ax=ax, color = 'none', edgecolor = 'grey') #displays census tracts 
gdf_iowacity.plot(ax=ax, color='red', edgecolor='grey')
gdf_iowacityP.plot(ax=ax, color='black')
iowa_roads.plot(ax=ax, color = 'black')
ax.set(xlim=(-91.6,-91.45), ylim=(41.6,41.72))
```




    [(41.6, 41.72), (-91.6, -91.45)]




![png](output_10_1.png)



```python
df_iowaAgg = pd.read_csv('iowacityAggregateData2.csv')
```


```python
df_iowaAgg
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
      <th>year</th>
      <th>pop_over25</th>
      <th>stores_num</th>
      <th>sales</th>
      <th>people_store_ratio</th>
      <th>dollars_store_ratio</th>
      <th>min</th>
      <th>max</th>
      <th>ave</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2012</td>
      <td>17191</td>
      <td>22</td>
      <td>9370466</td>
      <td>781</td>
      <td>425930</td>
      <td>3118.0</td>
      <td>3388920.0</td>
      <td>446213.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2013</td>
      <td>17344</td>
      <td>21</td>
      <td>9285364</td>
      <td>826</td>
      <td>442160</td>
      <td>18755.0</td>
      <td>3481457.0</td>
      <td>442160.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2014</td>
      <td>17452</td>
      <td>25</td>
      <td>9949581</td>
      <td>698</td>
      <td>397983</td>
      <td>2747.0</td>
      <td>3866877.0</td>
      <td>397983.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2015</td>
      <td>17250</td>
      <td>26</td>
      <td>10390854</td>
      <td>663</td>
      <td>399648</td>
      <td>3694.0</td>
      <td>4155665.0</td>
      <td>399648.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016</td>
      <td>17677</td>
      <td>27</td>
      <td>10832127</td>
      <td>655</td>
      <td>401190</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
import matplotlib.pyplot as plt
```


```python
fig,axes = plt.subplots(nrows=2, ncols=2, figsize=(9,6), sharey=False, sharex=True)

labels = [str(i) for i in df_iowaAgg['year']] #year 2012 to 2016
tick_arr = np.arange(len(labels))

pop = df_iowaAgg['pop_over25']
stores =  df_iowaAgg['stores_num']
sales =  df_iowaAgg['sales']
ratio =  df_iowaAgg['people_store_ratio']

axes[0,0].bar(tick_arr, pop, align='center')
axes[0,0].set_ylim([16000,18000])
axes[0,0].set_title('Population over 25 yrs')
axes[0,0].set_xlabel('Year')
axes[0,0].set_ylabel('Population')

axes[0,1].bar(tick_arr, stores, align='center')
axes[0,1].set_title('Number of Liquor Stores')
axes[0,1].set_xlabel('Year')
axes[0,1].set_ylabel('Number of Stores')

axes[1,0].bar(tick_arr, sales, align='center')
axes[1,0].set_title('Total Liquor Sales')
axes[1,0].set_xlabel('Year')
axes[1,0].set_ylabel('Dollars')

axes[1,1].bar(tick_arr, ratio, align='center')
axes[1,1].set_title('People to Store Ratio')
axes[1,1].set_xlabel('Year')
axes[1,1].set_ylabel('Number of Persons over 25')

# Set labels on both axes
plt.setp(axes, xticks=tick_arr, xticklabels=labels);

plt.show()
plt.style.use('default')
```


![png](output_14_0.png)



```python

labels = [str(i) for i in df_iowaAgg['year']] #year 2012 to 2016
min = df_iowaAgg['min']
max = df_iowaAgg['max']
ave = df_iowaAgg['ave']

x = np.arange(len(labels))  # the label location
width = 0.5  # the width of the bars

fig, ax = plt.subplots(figsize= (8,4))

rects1 = ax.bar(x, min, width, label='min')
rects2 = ax.bar(x, max, width, label='max')
rects3 = ax.bar(x, ave, width, label='ave')

# Add some text for labels, title and custom x-axis tick labels, etc.
ax.set_ylabel('Dollars')
ax.set_title('Iowa City Store Wide Spirits Aggregate Sales Data')
ax.set_xticks(x)
ax.set_ylim(0,4500000)
ax.set_xticklabels(labels)
ax.legend(loc=1)

#code credits: https://matplotlib.org/3.1.1/gallery/lines_bars_and_markers/barchart.html
def autolabel(rects):
    """Attach a text label above each bar in *rects*, displaying its height."""
    for rect in rects:
        height = rect.get_height()
        ax.annotate('{}'.format(height),
                    xy=(rect.get_x() + rect.get_width() / 2, height),
                    xytext=(0, 3),  # 3 points vertical offset
                    textcoords="offset points",
                    ha='center', va='bottom')

autolabel(rects1)
autolabel(rects2)
autolabel(rects3)

plt.show()
```


![png](output_15_0.png)



```python
#NOTE 'iowacitySalesbyStore2012_15.csv' has been ordered according to highest sales for years 2012-2015
df_stores = pd.read_csv('iowacitySalesbyStore2012_15.csv')
df_stores.head()
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
      <th>store_name</th>
      <th>yearlysales</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hy-Vee Wine and Spirits / Iowa City</td>
      <td>4155665.47</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Liquor Downtown / Iowa City</td>
      <td>882003.77</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>2</th>
      <td>John's Grocery</td>
      <td>777551.65</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hy-Vee Food Store / Iowa City</td>
      <td>750013.20</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hy-Vee Food Store #2 / Iowa City</td>
      <td>650767.64</td>
      <td>2015</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_stores2015 = df_stores[df_stores.year==2015]
df_stores2014 = df_stores[df_stores.year==2014]
df_stores2013 = df_stores[df_stores.year==2013]
df_stores2012 = df_stores[df_stores.year==2012]

```


```python
fig,axes = plt.subplots(nrows=4, ncols=1, figsize=(10,16), sharey=False, sharex=False)

labels2015 = df_stores2015.iloc[0:9,0]
labels2014 = df_stores2014.iloc[0:9,0]
labels2013 = df_stores2013.iloc[0:9,0]
labels2012 = df_stores2012.iloc[0:9,0]


sales2015 = df_stores2015.iloc[0:9,1]
sales2014 = df_stores2014.iloc[0:9,1]
sales2013 = df_stores2013.iloc[0:9,1]
sales2012 = df_stores2012.iloc[0:9,1]

axes[0].barh(labels2015,sales2015, align='center')
axes[0].set_title('2015 Stores with Highest Sales')
axes[0].set_xlabel('Sales')

axes[1].barh(labels2014,sales2014, align='center')
axes[1].set_title('2014 Stores with Highest Sales')
axes[1].set_xlabel('Sales')

axes[2].barh(labels2013,sales2013, align='center')
axes[2].set_title('2013 Stores with Highest Sales')
axes[2].set_xlabel('Sales')

axes[3].barh(labels2012,sales2012, align='center')
axes[3].set_title('2012 Stores with Highest Sales')
axes[3].set_xlabel('Sales')

plt.show()
plt.style.use('default')
```


![png](output_18_0.png)


## Connect to Google Cloud SQL database - create table with employment stats for a given liquor store location 


```python
import sqlalchemy

s = "mysql+mysqlconnector://root@35.239.4.71/iowaDb"
engine = sqlalchemy.create_engine(s)
engine.connect()
con0=engine.connect()
```


```python
storeNom =  'Hy-Vee Wine and Spirits / Iowa City' 


#get data from table'2016empStats'
def getEmpStats(storeNom):
        
        cmd = sqlalchemy.sql.text('''select * from 2016empStats
            where store_name = :nom;
            ''' )
        res_list = con0.execute(cmd, nom = storeNom).fetchall() #for some reason cannot pass storeNom directly, must also use secondary variable
        return res_list
    
def plotTreeMap(storeNom):
    res = getEmpStats(storeNom)
    print(storeNom)
    col1 = ['agr', 'entertain', 'constr', 'education', 'fince/insur', 'it-info', 'manuf', 'admin_nonPub', \
           'admin_Pub', 'retail', 'science', 'trade', 'whsle']
    col2 = []
    for j in range(2, len(res[0])):  
        col2.append(int(res[0][j]))

    df = pd.DataFrame({'#persons':col2, 'occupation':col1 })
    df=df[df['#persons']>0]  #get rid of jobs where count=0 

    squarify.plot(sizes=df['#persons'], label=df['occupation'],alpha=.8  )
    plt.title('2016 Occupation Breakdown by Neigborhood Tract')
    plt.axis('off')
    plt.show()

plotTreeMap(storeNom)
    
```

    Hy-Vee Wine and Spirits / Iowa City
    


![png](output_21_1.png)



```python
plotTreeMap('Liquor Downtown / Iowa City')
```

    Liquor Downtown / Iowa City
    


![png](output_22_1.png)



```python
plotTreeMap('Walgreens #05077 / Iowa City')
```

    Walgreens #05077 / Iowa City
    


![png](output_23_1.png)



```python
plotTreeMap('Hartig Drug Store #10')
```

    Hartig Drug Store #10
    


![png](output_24_1.png)


## Connect to Google Cloud PostgreSQL database - Create table w/ grocery store GIS information


```python
df_groc = pd.read_csv('grocerystores.csv')
df_groc.head(5)
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
      <th>store_name</th>
      <th>lat</th>
      <th>lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>natural grocers</td>
      <td>41.644995</td>
      <td>-91.531962</td>
    </tr>
    <tr>
      <th>1</th>
      <td>johns grocery</td>
      <td>41.663399</td>
      <td>-91.529915</td>
    </tr>
    <tr>
      <th>2</th>
      <td>new pioneer food co-op</td>
      <td>41.660514</td>
      <td>-91.528508</td>
    </tr>
    <tr>
      <th>3</th>
      <td>asian market</td>
      <td>41.653424</td>
      <td>-91.530093</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hy-vee waterfront</td>
      <td>41.642639</td>
      <td>-91.529444</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create Point(lon, lat) in the df
df_groc['geom'] = list(zip(df_groc['lon'], df_groc['lat']))
print(df_groc['geom'].head(2))
df_groc['geom'] = df_groc['geom'].apply(Point)
print(type(df_groc['geom'][0]))
dfg_groc = geopandas.GeoDataFrame(df_groc, geometry = 'geom')
dfg_groc.head(5)
```

    0    (-91.531962, 41.644995)
    1    (-91.529915, 41.663399)
    Name: geom, dtype: object
    <class 'shapely.geometry.point.Point'>
    




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
      <th>store_name</th>
      <th>lat</th>
      <th>lon</th>
      <th>geom</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>natural grocers</td>
      <td>41.644995</td>
      <td>-91.531962</td>
      <td>POINT (-91.53196 41.64500)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>johns grocery</td>
      <td>41.663399</td>
      <td>-91.529915</td>
      <td>POINT (-91.52992 41.66340)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>new pioneer food co-op</td>
      <td>41.660514</td>
      <td>-91.528508</td>
      <td>POINT (-91.52851 41.66051)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>asian market</td>
      <td>41.653424</td>
      <td>-91.530093</td>
      <td>POINT (-91.53009 41.65342)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hy-vee waterfront</td>
      <td>41.642639</td>
      <td>-91.529444</td>
      <td>POINT (-91.52944 41.64264)</td>
    </tr>
  </tbody>
</table>
</div>




```python
s="postgres+psycopg2://postgres:iowa@104.197.94.138:5432/icityDb"
engine = sqlalchemy.create_engine(s)
con1=engine.connect()
#con1.execute("CREATE EXTENSION postgis")  
```


```python

con1.execute("drop table if exists grocstore")
con1.execute('''create table grocstore(
    store_name character varying(128),
    lon float,
    lat float,
    geom geography)
    ''')

```




    <sqlalchemy.engine.result.ResultProxy at 0x1ea7e263e48>




```python
dict_groc = dfg_groc.to_dict('records') #what does 'records denote?'

print(dict_groc[0])
for item in dict_groc:
    item['wkt'] = item['geom'].wkt
    cmd = sqlalchemy.sql.text('''INSERT INTO grocstore(store_name, lon, lat, geom)\
        VALUES(:store_name, :lon, :lat, ST_GeogFromText(:wkt))''')
    con1.execute(cmd,item)
```

    {'store_name': 'natural grocers', 'lat': 41.644995, 'lon': -91.531962, 'geom': <shapely.geometry.point.Point object at 0x000001EA7F498A58>}
    


```python
con1.execute("select * from grocstore limit 5").fetchall()
```




    [('natural grocers', -91.531962, 41.644995, '0101000020E6100000BE2D58AA0BE256C0B18A37328FD24440'),
     ('johns grocery', -91.529915, 41.663399, '0101000020E61000003BAA9A20EAE156C07C992842EAD44440'),
     ('new pioneer food co-op', -91.528508, 41.660514, '0101000020E610000029EB3713D3E156C06B4606B98BD44440'),
     ('asian market', -91.530093, 41.653424, '0101000020E6100000ABB5300BEDE156C0F435CB65A3D34440'),
     ('hy-vee waterfront', -91.529444, 41.642639, '0101000020E61000000F441669E2E156C02C11A8FE41D24440')]




```python

con1.execute("drop table if exists liqstore")
con1.execute('''create table liqstore(
    store_name character varying(128),
    zip_code character varying(16),
    store_location geography)
    ''')

```




    <sqlalchemy.engine.result.ResultProxy at 0x1ea7e26ca90>




```python
iowacityP_dict = gdf_iowacityP.to_dict('records')
print(iowacityP_dict[0])
for item in iowacityP_dict:
    item['wkt']=item['store_location'].wkt
    cmd = sqlalchemy.sql.text('''INSERT INTO liqstore(store_name, zip_code, store_location)\
        VALUES(:store_name, :zip_code, ST_GeogFromText(:wkt))''')
    con1.execute(cmd, item)

```

    {'store_name': 'Hy-Vee Wine and Spirits / Iowa City', 'zip_code': 52240, 'store_location': <shapely.geometry.point.Point object at 0x000001EA7E30B9E8>}
    


```python
con1.execute("select * from liqstore limit 5").fetchall()
```




    [('Hy-Vee Wine and Spirits / Iowa City', '52240', '0101000020E61000003D2F151BF3E156C0E8853B1746D24440'),
     ('Hy-Vee Food Store / Iowa City', '52240', '0101000020E610000038BC202235E156C09BFEEC478AD64440'),
     ('Fareway Stores #034 / Iowa City', '52240', '0101000020E61000001B81785DBFDE56C0B900344A97D04440'),
     ('New Pioneer Food Co-op / Iowa City', '52240', '0101000020E61000009EB30584D6E156C0541EDD088BD44440'),
     ('Hy-Vee Food Store #2 / Iowa City', '52240', '0101000020E6100000F2F7414C78E256C0B59D00D41ED34440')]




```python
#IMPORTANT data type must be 'geography' for ST_Intersects(ST_Buffer(),_) to work!
con1.execute('''SELECT DISTINCT grocstore.store_name FROM grocstore, liqstore 
              WHERE ST_Intersects(ST_Buffer(grocstore.geom,300), liqstore.store_location)''').fetchall()
```




    [('aldi',),
     ('asian market',),
     ('fareway grocery commerce',),
     ('fareway grocery westwind',),
     ('hy-vee 1st st',),
     ('hy-vee waterfront',),
     ('iowa city african and oriental market',),
     ('johns grocery',),
     ('natural grocers',),
     ('new pioneer food co-op',),
     ('walmart',)]




```python
ax = states.plot(color='None', edgecolor='k', figsize=(10,20))
#roads.plot(ax = ax, color='black')
iowa_roads.plot(ax=ax, color = 'black')
iowa.plot(ax=ax, edgecolor = 'grey')
gdf_iowacity.plot(ax=ax, color='red', edgecolor='grey')
gdf_iowacityP.plot(ax=ax, color='green', alpha=0.7)
dfg_groc.plot(ax=ax, color='yellow', alpha=0.6)
ax.set(xlim=(-91.6,-91.45), ylim=(41.6,41.72))
```




    [(41.6, 41.72), (-91.6, -91.45)]




![png](output_36_1.png)



```python

```
