title: 'Altering the resolution of a raster data using Python'
date: 2019-04-07 23:26:26

categories:

- Tech

tags:

- Python
- GIS

---

Sometimes, we need to transform the original raster dataset to a different resolution (usually coarser). In this post, I will introduce the simple workflow for cell resampling using Python. <!--more-->

# 1. Data import / preprocessing

To get started, first load the required library and then import the original raster data. In this case, we downloaded the Gross Domestic Product (GDP) distribution of China in 2010 with the resolution of 1 km x 1 km as the input. What's more, a subset of China outlined by a shapefile was also used for clipping that raster.

```Python
import warnings
warnings.filterwarnings('ignore')
import cartopy
import matplotlib.pyplot as plt
from osgeo import gdal
import rasterio
from rasterio.mask import mask
import geopandas as gpd
from shapely.geometry import mapping
import skimage.transform as st

# original data import
from osgeo import gdal
pathToRaster = r"./cngdp2010.tif"
input_raster = gdal.Open(pathToRaster)
prj=input_raster.GetProjection()
srs=osr.SpatialReference(wkt=prj)
if srs.IsProjected:
    print srs.GetAttrValue('projcs')
print srs.GetAttrValue('geogcs')

# Albers_Conic_Equal_Area
# GCS_Unknown_datum_based_upon_the_Krassowsky_1940_ellipsoid

# transforming the projection of the input_raster
output_raster = r'./cngdp2010_adjust.tif'
gdal.Warp(output_raster,input_raster,dstSRS='EPSG:4326',dstNodata = -9999)

# clipping the raster by polygon
input_shape = r'./boundary.shp'
shapefile = gpd.read_file(input_shape)
shapefile = shapefile.to_crs(epsg=4326)
input_raster = r'./cngdp2010_adjust.tif' # input raster
geoms = shapefile.geometry.values # list of shapely geometries
output_raster = r'./2+26city_gdp.tif' # output raster
geometry = geoms[0] # shapely geometry
geoms = [mapping(geoms[0])]
with rasterio.open(input_raster) as src:
    out_image, out_transform = mask(src, geoms, crop=True) ## 出现了不配套的情况，怎么办
    out_meta = src.meta.copy()        
out_meta.update({"driver": "GTiff",
                 "height": out_image.shape[1],
                 "width": out_image.shape[2],
                 "transform": out_transform})
with rasterio.open(output_raster, "w", **out_meta) as dest:
    dest.write(out_image)  
```

![][1]


# 2. Resampling the data
There are many different approaches to adjust the resolution of the raster file. Here, I use one simple method suppoerted by [skikit-image package](https://scikit-image.org/). With the resize function, you are simply create a new 2-d array with a predefined mesh grid.  

```Python
pathToRaster  = r'./2+26city_gdp.tif'
with rasterio.open(pathToRaster) as src:
    arr = src.read()
    arr[arr ==-9999] = np.nan
    arr  = arr[0,:,:]
    # In this case, the orignial resolution was adjusted to 3 km x 3 km.
    new_shape = (round(arr.shape[0]/3.0),round(arr.shape[1]/3.0))
    newarr= st.resize(array, new_shape, mode='constant')
```
![][2]

# 3. Exporting Data

When the resampling works are done, the generated numpy array can be exported to geotiff format file for further uses. Luckily, some sufficient information for geotiff file can be easily assesed from the original one.


```Python
import cartopy.crs as ccrs
import rasterio
pathToRaster  = r'./2+26city_gdp.tif'
raster = gdal.Open(pathToRaster,  gdal.GA_ReadOnly)
gt     = raster.GetGeoTransform()
```

Since the mesh grid has been altered, the GeoTransform information should be editted. The values in __gt__ includes:

* gt[0] = top left x
* gt[1] = w-e pixel resolution
* gt[2] = 0
* gt[3] = top left y
* gt[4] = 0
* gt[5] = n-s pixel resolution (negative value)

In this case, gt[1] and gt[5] should be revised (i.e., multiplied by 3)

```Python
output =  r'./2+26city_3km.tif'
cols, rows =newarr.shape[1],newarr.shape[0]
driver = gdal.GetDriverByName("GTiff")
outDs = driver.Create(output, cols, rows, 1, gdal.GDT_Float32)
r, float, etc).
outDs.SetGeoTransform((gt[0],gt[1]*3.0,gt[2],gt[3],gt[4],gt[5]*3.0))
outDs.SetProjection(raster.GetProjection())
outDs.GetRasterBand(1).WriteArray(newarr[::-1])
outDs.GetRasterBand(1).SetNoDataValue(-9999)
del outDs
```

Done!



# PS: 

_参考资料_

[Export Numpy Arrays to Geotiff Format Using Rasterio and Python
](https://www.earthdatascience.org/courses/earth-analytics-python/multispectral-remote-sensing-in-python/export-numpy-array-to-geotiff-in-python/)  

[1]:https://i.stack.imgur.com/uiBFi.png
[2]: https://i.stack.imgur.com/xQ2Y6.png

