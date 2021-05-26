title: 'Two method to mask the raster data by specific geometry' 
date: 2018-12-06 16:26:26

categories:

- Code

tags: 

- Python
- GIS

------

Sometimes, we need to clip or extract the raster image with polygon features, e.g., only focus on the percipitation within China using global dataset. This post will introduce two methods to mask the 2-d array-like dataset by a specific geometry using Python.  

* The first method use [GeoPandas](geopandas.org) module to test those raster coordidnates  within/out of the shapefile
* The second method which I strongly recommended with faster speed was based on [shapely](https://shapely.readthedocs.io/en/stable/manual.html) module.

<!--more-->

## 1. Geopandas 

```python
# referencing https://stackoverflow.com/questions/47781496/python-using-polygons-to-create-a-mask-on-a-given-2d-grid
import numpy as np
import iris
import geopandas as gpd

# reading the mask shapefile
fname = r'xxx/china.shp'
cn =gpd.read_file(fname)

# four corners
x0,x1 = cn.bounds['minx'].values,cn.bounds['maxx'].values
y0,y1 = cn.bounds['miny'].values,cn.bounds['maxy'].values

# the map background function
def back_main_clean(fig,extent):
    ax = fig.add_subplot(111, projection=ccrs.PlateCarree())
    ax.set_extent(extents=extent, crs=ccrs.Geodetic())
    for i in ax.spines.itervalues():
        i.set_linewidth(0.01)
    fname = r'xxx/china_provinces.shp'
    shape_feature = ShapelyFeature(Reader(fname).geometries(), ccrs.PlateCarree(),facecolor='#FFFFFF', linestyle='--',edgecolor ='k', linewidth = 0.25, alpha =1)
    ax.add_feature(shape_feature, zorder =3)   

# initializing the raster data
x  = np.linspace(x0,x1,30)
y  = np.linspace(y0,y1,20)
fig = plt.figure(figsize=(8, 5))
ax = back_main_clean(fig, extent)
xx,yy = np.meshgrid(x,y)
zz = np.random.random((len(x),len(y)))
plt.scatter(xx.reshape(-1),yy.reshape(-1),zorder =4,transform=ccrs.PlateCarree(), c=zz.reshape(-1),s=8)
```

![F1SOK0.png](https://s1.ax1x.com/2018/12/06/F1SOK0.png)

```python
# creating the mask array and counting the processing time
import time
start = time.time()
lon2 = xx.reshape(-1)
lat2 = yy.reshape(-1)
mask = []
for lat, lon in zip(lat2, lon2):
    this_point = gpd.geoseries.Point(lon,lat)
    res = cn.contains(this_point)
    mask.append(res.values[0])
mask = np.array(mask)
mask= mask.reshape(len(y),len(x))
print "Process time: " + str(time.time() - start)
```

__Process time: 5.11 Seconds__

```python
# plotting the raster data extracted by China
zz_mask = np.ma.masked_array(zz,mask) # noted that ~mask can simply transform to clip function.
fig = plt.figure(figsize=(8, 5))
ax = back_main_clean(fig, extent)
xx,yy = np.meshgrid(x,y)
zz = np.random.random((len(x),len(y)))
plt.scatter(xx.reshape(-1),yy.reshape(-1),zorder =4,transform=ccrs.PlateCarree(), c=zz_mask.reshape(-1),s=8)
```

![F1p9PJ.png](https://s1.ax1x.com/2018/12/06/F1p9PJ.png)

## 2. shapely.vectorized

```python
# referencing https://gist.github.com/pelson/9785576
import fiona
import shapely.vectorized
from shapely.geometry import shape
fname = r'xxx/china.shp'
cn_area = fiona.open(fname)
pol = cn_area.next()
geom = shape(pol['geometry'])

## four corner 
x0,x1 = geom.bounds[0],geom.bounds[2]
y0,y1 = geom.bounds[1],geom.bounds[3]

## creating the mask array

x  = np.linspace(x0,x1,30)
y  = np.linspace(y0,y1,20)
xx,yy = np.meshgrid(x,y)
import time
start = time.time()
mask_ = shapely.vectorized.contains(geom,  xx,yy)
print "Process time: " + str(time.time() - start)
```

__Process time: 0.01 Seconds__  (huge advantage here)

```python
zz = np.random.random((len(x),len(y)))
zz_mask = np.ma.masked_array(zz,~mask_)
fig = plt.figure(figsize=(8, 5))
ax = back_main_clean(fig, extent)
xx,yy = np.meshgrid(x,y)
# zz = np.random.random((len(x),len(y)))
# plt.scatter(xx.reshape(-1),yy.reshape(-1),zorder =4,transform=ccrs.PlateCarree(),)
plt.scatter(xx.reshape(-1),yy.reshape(-1),zorder =4,transform=ccrs.PlateCarree(), c=zz_mask.reshape(-1),s=5)
```

![F1pyZT.png](https://s1.ax1x.com/2018/12/06/F1pyZT.png)

## Some additional tips

```python
# 1. the error during shapefile reading
import fiona
cn_area = fiona.open('xxx.shp')
> Recode from CP936 to UTF-8 not supported, treated as ISO8859-1 to UTF-8.
cn_area = fiona.open('xxx.shp', encoding  ='utf-8')
```

```python
# 2. select the polygon by attributes within geopandas module
fname = r'xxx/china_provinces.shp'
cn =gpd.read_file(fname, encoding ='utf-8')
test_p = cn[cn['NAME']==u'黑龙江']
test_p.plot()
```

![F19R78.png](https://s1.ax1x.com/2018/12/06/F19R78.png)

```python
# 3. select the polygon by attributes within shapely module
with fiona.open(fname, encoding='utf-8') as src:
    filtered = filter(lambda f: f['properties']['NAME']==u'黑龙江', src)
pol = filtered[0]#.next()
geom = shape(pol['geometry'])
geom
```

![F19htg.png](https://s1.ax1x.com/2018/12/06/F19htg.png)

```python
# 4. select and output one polygon from multiple polygons by attributes
import fiona
with fiona.open(fname, encoding='utf-8') as input:
    meta = input.meta
    with fiona.open('test_heilongjiang.shp','w',**meta) as output:
        for feature in input:
            if feature['properties']['NAME']==u'黑龙江':
                output.write(feature)
```

