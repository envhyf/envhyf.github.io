title: 'Plotting topography map with hillside'
date: 2018-11-06 16:26:26

categories:

- Tech

tags:

- Python
- Visualization

------

Hillshade is the representation of the earth's surface under the radiation of sun. A terrain raster data can be better visualized by adding the information of hillshades. This blog will introduce some procrdures to overlay the hillshade with terrain for a nice picture.  

<!--more-->

First, let's import the required packages.  

```python
from osgeo import gdal 
import numpy as np 
import matplotlib.pyplot as plt 
import elevation
%matplotlib inline
```

Second, we download and read a sample data 

```python
!eio --product SRTM3 clip -o DEM.tif --bounds -122.4 41.2 -122.1 41.3
from osgeo import gdal
raster = gdal.Open("./DEM.tif",  gdal.GA_ReadOnly)
dem = raster.GetRasterBand(1).ReadAsArray()
dem = dem[::-1]

```

Then we calculated the hillshade using the function I referenced from [Download and Process DEMs in Python](http://geologyandpython.com/dem-processing.html)

```python
def hillshade(array, azimuth, angle_altitude):
    # Source: http://geoexamples.blogspot.com.br/2014/03/shaded-relief-images-using-gdal-python.html
    x, y = np.gradient(array)
    slope = np.pi/2. - np.arctan(np.sqrt(x*x + y*y))
    aspect = np.arctan2(-x, y)
    azimuthrad = azimuth*np.pi / 180.
    altituderad = angle_altitude*np.pi / 180.
    shaded = np.sin(altituderad) * np.sin(slope) \
     + np.cos(altituderad) * np.cos(slope) \
     * np.cos(azimuthrad - aspect)
    return 255*(shaded + 1)/2
```

Finally, we plotted three figures, one is the simple dem plot, the other two  are the topography overlayed by hillshade using different alpha. 

```python
fig = plt.figure(figsize=(12, 4))
extent =[-122.4 ,41.2, -122.1, 41.3]
ax = fig.add_subplot(131,projection=ccrs.Mercator())
cax = ax.contourf(dem, np.arange(vmin, vmax, 10),extent=extent, 
                  cmap=terrain_cmap, vmin=vmin, vmax=vmax, origin='image')
plt.title(r'$Original$',fontsize =12)
ax = fig.add_subplot(132,projection=ccrs.Mercator())
ax.matshow(hillshade(dem, 30, 30), extent=extent, cmap='Greys', alpha=.3, zorder=10)

cax = ax.contourf(dem, np.arange(vmin, vmax, 10),extent=extent, 
                  cmap=terrain_cmap, vmin=vmin, vmax=vmax, origin='image')
plt.title(r'$\alpha=0.3$',fontsize =12)
ax = fig.add_subplot(133,projection=ccrs.Mercator())
ax.matshow(hillshade(dem, 30, 30), extent=extent, cmap='Greys', alpha=.8, zorder=10)
cax = ax.contourf(dem, np.arange(vmin, vmax, 10),extent=extent, 
                  cmap=terrain_cmap, vmin=vmin, vmax=vmax, origin='image')
plt.title(r'$\alpha=0.8$',fontsize =12)

```

![](https://i.stack.imgur.com/4pQAH.png)

__References__

[Create a Hillshade from a Terrain Raster in Python](https://www.neonscience.org/create-hillshade-py)

[Earth analytics python](https://www.earthdatascience.org/courses/earth-analytics-python/lidar-raster-data/overlay-raster-maps/)

[Download and Process DEMs in Python](http://geologyandpython.com/dem-processing.html)

