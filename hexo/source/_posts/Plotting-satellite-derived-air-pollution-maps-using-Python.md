title: 'Plotting air pollution maps using satellite data'
date: 2018-11-25 15:41:33

categories:

- Research

tags: 

- Python
- Air quality
- Visualization

---

I'm writing today about downloading, handling, and plotting satellite derived air pollution maps with cartopy and fiona using Python. One key task in this post is to clip a raster-like (2-d array) dataset with a polygon in pure Python environment (i.e., no need for ArcGIS or QGIS GUI-based software).         

The satellite sensor can offer critical supplementary data of several atmospheric species, e.g., SO<sub>2</sub>, NO<sub>2</sub>, PM<sub>2.5</sub>. Comparaing to ground-based monitoring which might be sparse in some areas (e.g., Africa, South America, oceans), the satellite observation offers a full picture for better understanding the spatiotemporal patterns of some air pollutants.

Below is an excerpt of a NO<sub>2</sub> column maps within  Chengyu urabn agglomeration in China.

<!--more-->   

The  tropospheric NO<sub>2</sub>  concentration derived from satellite remote sensing techniques have been used on global, regional and urban scales since the mid-nineties. After the launch of the Ozone Monitoring Instrument (OMI) in 2004, the spatial resolution (up to 13 x 24 km<sup>2</sup>) ave been largely improved. The vetical column NO<sub>2</sub> dataset  have contributed to reveal the hotspots of air pollution around the world, or show the episodes with long-range transport.

The TEMIS website provide the image and data for  NO<sub>2</sub>  concentration maps in individual days and monthly averages derived from different satellite sensors (e.g., OMI, GOME-2). Herein, I present the general processes to obtain DOMINO version 2.0 data

## Obtaining the datasets

We start by downloading the essential data files using function of wget.

```python
#Creat an loop for downloading the monthly averages from 2013-2017
import os
for i in ['2013','2014','2015','2016','2017']:
	for t in ['01','02','03','04','05','06','07','08','09','10','11','12']:
		filename = 'http://www.temis.nl/airpollution/no2col/data/omi/data_v2/'+i+'/'+t+'/'+'no2_'+i+t+ '.grd.gz'
		os.system('wget %s' %filename)
print "DOWNLOADING COMPLETE"
```


## Merging the datasets

We then define a reading function and open those files to obtain essential information.  In the case of 2013, there are 12 monthly averages need to read and merge for further analysis.

```python
import numpy as np
def read_grd(filename):
    with open(filename) as infile:
        ncols = int(infile.readline().split()[1])
        nrows = int(infile.readline().split()[1])
        xllcorner = float(infile.readline().split()[1])
        yllcorner = float(infile.readline().split()[1])
        cellsize = float(infile.readline().split()[1])
        nodata_value = int(infile.readline().split()[1])
        version = float(infile.readline().split()[1])
    longitude = xllcorner + cellsize * np.arange(ncols)
    latitude = yllcorner + cellsize * np.arange(nrows)
    value = np.loadtxt(filename, skiprows=7)
    return longitude, latitude, value,nodata_value

set_year = 2013
for i in range(0,12,1):
    # setting the file path which contains those original data.
    file_path = './NO2_POMINO/no2_'+str(set_year)+ '%02d' %(i+1)+'.grd'
    lon,lat,value,nodata_value = read_grd(file_path)
    value = value[::-1]
    value[value == nodata_value] = np.nan        
    if i ==0:
        value_ = value.reshape(1,value.shape[0],value.shape[1])
    else:
        value_ = np.vstack([value_,value[None, ...]])
## np.nanmean() is utilized to compute the arithmetic mean igonring NaNs
no2_2013 = np.nanmean(value_,axis = 0)
```

##Cut 2-d array by shapefile 

Next,  we  import the necessary library and the specific shapefile to clip the global 2-d map.

```python
#1. transform the shapefile to fiona polygon object
import fiona
from shapely.geometry import shape,Polygon, Point
cf_area = fiona.open("xxx.shp")
pol = cf_area.next()
from shapely.geometry import shape
geom = shape(pol['geometry'])
poly_data = pol["geometry"]["coordinates"][0]
poly = Polygon(poly_data)

#2. substract the dataset that can cover the polygon
x1 =  np.array([t for t,j in (poly_data)]).min()
x2 =  np.array([t for t,j in (poly_data)]).max()
y1 =  np.array([j for t,j in (poly_data)]).min()
y2 =  np.array([j for t,j in (poly_data)]).max()
extent      = x1 -0.5, x2+0.5,y1-0.5, y2+0.5

def find_nearest(array,value):
    idx = (np.abs(array-value)).argmin()
    return array[idx]
nx_st = np.where(lon == (find_nearest(lon,extent[0])))[0]
nx_en = np.where(lon == (find_nearest(lon,extent[1] )))[0]
ny_st = np.where(lat == (find_nearest(lat,extent[2])))[0]
ny_en = np.where(lat == (find_nearest(lat,extent[3] )))[0]
lon_,lat_ = lon[nx_st:nx_en+1], lat[ny_st:ny_en+1]
so2_data= so2_data[ny_st:ny_en+1, nx_st:nx_en+1]  

#3. test the dat points whether or not within the polygon
import time
start_time = time.time()
sh = (len(lat_)*len(lon_),2)
points = np.zeros(len(lat_)*len(lon_)*2).reshape(*sh)
k = 0
for i in range(0,len(lat_),1):
    for j in range(0,len(lon_),1):
        points[k] = np.array([lon_[j],lat_[i]],dtype=float)
        k+=1
mask = np.array([poly.contains(Point(x, y)) for x, y in points])  
mask = mask.reshape(len(lat_),len(lon_))
print("--- %s seconds ---" % (time.time() - start_time))

print mask.shape
np.savetxt("mask_SO2.txt",mask)

#4. mask the data points outsied the polygon

def shape_mask(array, lon_,lat_,mask):
    arr = np.zeros_like(array)
    for i in range(0,len(lat_),1):
        for j in range(0,len( lon_),1):
            if mask[i,j] == 1:
                arr[i,j] = array[i,j]
            else:
                arr[i,j] = -1
    return arr
no2_mask   = shape_mask(no2_data, lon_,lat_,mask)
```

The mask array as the rastered polygon is shown like this.

```python
fig = plt.figure()
ax = back_main_clean(fig,extent)
for spine in plt.gca().spines.values():
	spine.set_visible(False)
ax.outline_patch.set_visible(False)
ax.pcolormesh(mask)
plt.tight_layout()
```

[![enter image description here][1]][1]

## Visualization

```python
from mpl_toolkits.axes_grid1 import make_axes_locatable
fig = plt.figure()
ax = back_main_clean(fig,extent) # the background map was prior-defined by specific uses.
lon_x,lat_y = np.meshgrid(lon_,lat_)

value_mask = np.ma.masked_array(no2_mask, np.isnan(no2_mask))
value_mask = np.ma.masked_less(value_mask, 0.0)
my_cmap = plt.cm.get_cmap('Spectral_r')
my_cmap.set_under('w')
cc = ax.pcolormesh(lon_x,lat_y,value_mask/100.0,cmap =my_cmap,alpha = 0.7,transform=ccrs.PlateCarree(),zorder =2,vmax = 25)#

cbaxes = fig.add_axes([0.58, 0.12, 0.16, 0.015]) 
cbar = fig.colorbar(cc, cax=cbaxes, ticks=[4,12,20],orientation='horizontal')
cbar.ax.set_title(r'$\mathregular{NO_2\ [10^{15}molec/cm^2]}$', fontsize = 8)
cbar.ax.tick_params(color='k', direction='in',labelsize=8)

for spine in plt.gca().spines.values():
    spine.set_visible(False)
ax.outline_patch.set_visible(False)
plt.tight_layout()
```

[![enter image description here][2]][2]





[1]: https://i.stack.imgur.com/a14mC.png
[2]: https://i.stack.imgur.com/b8dRv.png












