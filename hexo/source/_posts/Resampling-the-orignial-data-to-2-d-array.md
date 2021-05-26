title: Resampling the orignial data to 2-d array in specific grid system
date: 2019-06-01 20:32:26

categories:

- Tech

tags:

- Air quality
- Python

---

In this post, I address an common problem in geoscience research: how to arrange the original geodata into pre-defined grid system. Sometimes, we need to unify the resolution of various dataset or summary the scatter data to raster one.

Specifically, this brief tutorial will look at two different original data, and allow you to creat gridded data in python. For better illustration, two practial cases with detailed code are shown:

* Creating an emission inventory based on the emissions from point sources (e.g., power plants, cement plants)
* Remapping a population density map to a coarser resolution

<!--more-->

## <font color="blue"> 1. Scatter data point</font>

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
import cartopy.crs as ccrs
from matplotlib.axes import Axes
from cartopy.mpl.geoaxes import GeoAxes
GeoAxes._pcolormesh_patched = Axes.pcolormesh
from cartopy.mpl.gridliner import LATITUDE_FORMATTER, LONGITUDE_FORMATTER
from scipy.spatial import KDTree

# 1. generating the scatter data
xc1, xc2, yc1, yc2 = 100, 110, 35, 45
N = 1000
x = np.random.uniform(low=xc1, high=xc2, size=(N,))
y = np.random.uniform(low=yc1, high=yc2, size=(N,))
value = np.ravel(np.random.rand(1000,1))
df_points = pd.DataFrame({"x":x, "y":y, "v":value})

## 2. Clustering the scatter point Using KdTree method
XSIZE,YSIZE=50,50 # set the grid resolution
lon,lat  = np.linspace(xc1,xc2,XSIZE),np.linspace(yc1,yc2,YSIZE)
X, Y = np.mgrid[lon.min():lon.max():XSIZE*1j, lat.min():lat.max():YSIZE*1j]

grid = np.c_[X.ravel(), Y.ravel()]
points = np.c_[df_points.x, df_points.y]

from scipy.spatial import KDTree
tree = KDTree(grid)
dist, indices = tree.query(points)
grid_values = df_points.groupby(indices).v.sum()

df_grid = pd.DataFrame(grid, columns=["x", "y"])
df_grid["v"] = grid_values

## 3. Visualization
def plot_xy_tick(ax):
    xticks = [100,105,110]
    yticks = [35,40,45]
    ax.set_xticks(xticks, crs=ccrs.PlateCarree())
    ax.set_yticks(yticks, crs=ccrs.PlateCarree())
    lon_formatter = LongitudeFormatter(number_format='.1f',
                                       degree_symbol='',
                                       dateline_direction_label=True)
    lat_formatter = LatitudeFormatter(number_format='.1f',
                                      degree_symbol='')
    ax.xaxis.set_major_formatter(lon_formatter)
    ax.yaxis.set_major_formatter(lat_formatter)

fig = plt.figure(figsize=(8,3))
ax1 = plt.subplot(121, projection=ccrs.PlateCarree())
ax1.scatter(df_points.x, df_points.y,  alpha=0.2,c=df_points.v,s=15,transform=ccrs.PlateCarree())
ax1.set_extent([xc1, xc2, yc1, yc2], crs=ccrs.PlateCarree())
plot_xy_tick(ax1)
ax2 = plt.subplot(122, projection=ccrs.PlateCarree())
val_ = df_grid.v.values.reshape(len(lat),len(lon))
ax2.pcolormesh(X,Y, val_,transform=ccrs.PlateCarree())
ax1.set_extent([xc1, xc2, yc1, yc2], crs=ccrs.PlateCarree())
plot_xy_tick(ax2)    
plt.show()
```

[![enter image description here][1]][1]


[1]: https://i.stack.imgur.com/6RHnY.png



## <font color="blue"> 2. Raster data</font>

Noted in my last [post](<http://paisheng.me/2019/04/07/Altering-the-resolution-raster-file-using-Pyton/>), the resampling of __tif-format dataset__ to raster file in other resolution relied on the skimage.transform function.  Here, I present the method which directly deal with np.array.

### 2.1 equal coarse ratio on both axis

```python
## creat a original data representing populaiton map
xc1, xc2, yc1, yc2 = 100, 110, 35, 45
XSIZE,YSIZE=100,100
lon,lat  = np.linspace(xc1,xc2,XSIZE),np.linspace(yc1,yc2,YSIZE)
pop = np.random.uniform(low=1000, high=50000, size=(XSIZE*YSIZE,)).reshape(YSIZE,XSIZE)
```

```python
## Method 1 reshape the original data array
shape = np.array(pop.shape, dtype=float)
coarseness = 2 # the new shape is in 50 x 50
new_shape = coarseness * np.ceil(shape/coarseness).astype(int)
# # Create the zero-padded array and assign it with the old density
zp_pop = np.zeros(new_shape)
zp_pop[:int(shape[0]), :int(shape[1])] = pop
temp = zp_pop.reshape((new_shape[0] // coarseness, coarseness,
                                new_shape[1] // coarseness, coarseness))
coarse_pop = np.sum(temp, axis=(1,3))
print (pop.sum())
print (coarse_pop.sum())
```

### 2.2 inequal coarse ratio on both axis

**Method.1 interpolation**

```python
from scipy import interpolate
hfunc = interpolate.interp2d(lon,lat,pop)
XSIZE_n,YSIZE_n=80,60
coarse_pop = np.zeros(XSIZE_n * YSIZE_n)
xx,yy=np.meshgrid(lon_n,lat_n)
grid = np.c_[xx.ravel(),yy.ravel()]
df_grid = pd.DataFrame(grid, columns=["x", "y"])
df_grid['v'] = [hfunc(x,y)[0] for x, y in zip(df_grid['x'].values,df_grid['y'].values)]

print (pop.sum())
print (df_grid['v'].sum())
```

**Method.2 Fourier transform method**

Thanks to help from my new roomate who is expert in math, I learned to transfrom the 2-d array using Fourier transform method

```python
from scipy import fftpack
pop_fft = fftpack.fft2(pop*100*100/(60*80),shape = (60,80)) # 100 x 100 -> 60 x 80
pop_res = fftpack.ifft2(pop_fft).real
print(pop.sum())
print(pop_res.sum())
```
