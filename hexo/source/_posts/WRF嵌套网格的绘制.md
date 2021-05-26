title: WRF嵌套网格的绘制
date: 2020-08-04 06:00:00

categories:

- Research

tags:

- Python
- WRF
- Visualization

---

提要：定义嵌套网格是运行WRF模型的重要工作。本文基于Python 3.7，在[Salem](https://salem.readthedocs.io/en/v0.2.3/)工具包基础上加以改进，结合Cartopy对中国国界线进行替换，同时实现修改默认底图。
<!--more-->

## <font color="blue"> 1. 替换底图</font> 
Salem是用于地理数据处理和可视化的Python小工具，其功能强大，使用简便。例如，Salem可直接读取WRF模型的namelist.input，绘制网格嵌套图。    

```python
import salem
from salem.utils import get_demo_file
from salem import geogrid_simulator
fpath = r'./namelist.wps'
g, maps = geogrid_simulator(fpath)
maps[0].set_rgb(natural_earth='lr')  # add a background image
maps[0].visualize(title='WRF Simlated Domains 1 to 3')
```

[![enter image description here][1]][1]   

  [1]: https://i.stack.imgur.com/9W9Lz.png


Cartopy底图存放位置
`/User/HYF/anaconda3/lib/⁨python3.7/⁨site-packages⁩/cartopy/⁨data⁩/raster⁩`
高清图像下载地址
* [natural Earth I](https://thomasguymer.co.uk/blog/2018/2018-01-15/)
* [Natural Earth III](http://www.shadedrelief.com/natural3/pages/textures.html)
* [Natural Earth III with no clouds](http://shadedrelief.com/natural3/ne3_data/8192/textures/2_no_clouds_8k.jpg)
⁩
下载新image后，需修改相同文件夹下的image.json，举例如下:    

```javascript
{"__comment__": "JSON file specifying the image to use for a given type/name and resolution. Read in by cartopy.mpl.geoaxes.read_user_background_images.",
  "ne_shaded": {
    "__comment__": "Natural Earth shaded relief",
    "__source__": "http://www.naturalearthdata.com/downloads/50m-raster-data/50m-natural-earth-1/",
    "__projection__": "PlateCarree",
    "low": "50-natural-earth-1-downsampled.png",
    "high": "natural-earth-1_large8192px.png",
    "medium": "natural-earth-1_large4096px.png"},
  "ne_shaded": {
    "__comment__": "Natural Earth shaded relief III",
    "__source__": "http://www.shadedrelief.com/natural3/pages/textures.html⁩",
    "__projection__": "PlateCarree",
    "high": "natural-earth-3_large4096px"}
}
```

## <font color="blue"> 2. 中国行政边界重新绘制</font> 
原始Salem工具包的中国行政地图存在错误。因此，自行绘制国家边界，以准确绘制图形。具体实现相对简单，首先应用Salem工具读取WRF模型的namelist文件，然后应用Cartopy工具绘制，并换用上一节介绍的更为清晰的地形底图。

```python
import salem
from salem import geogrid_simulator
from salem import wgs84

fpath = r'./namelist.wps'
g, maps,rect= geogrid_simulator(fpath) 

x0,y0 = pyproj.transform(g[0].proj,wgs84,g[0].extent[0], g[0].extent[2])
x1,y1 = pyproj.transform(g[0].proj,wgs84,g[0].extent[1], g[0].extent[3])  

def background(ax):   
    fname = './bou2_4p.shp' ## 省界shapefile
    shape_feature = ShapelyFeature(Reader(fname).geometries(),
                                    ccrs.PlateCarree(),facecolor='none',edgecolor ='k',linewidth = 0.25)
    ax.add_feature(shape_feature)
    
    fname = './中国轮廓_含南海.shp' ## 国界shapefile
    shape_feature = ShapelyFeature(Reader(fname).geometries(),ccrs.PlateCarree(),facecolor='none',edgecolor ='k',linewidth = 0.75)
    ax.add_feature(shape_feature)
    
    fname = './country1.shp' ## 各国边界shapefile
    shape_feature = ShapelyFeature(Reader(fname).geometries(),ccrs.PlateCarree(),facecolor='none',edgecolor ='k',linewidth = 0.75)
    ax.add_feature(shape_feature) 

    ax.set_extent([x0,125,y0+3,55], crs=ccrs.PlateCarree())     
    xticks = [80,90,100,110,120,130,140]
    yticks = [10,20,30,40,50,60]
    fig.canvas.draw()
    ax.gridlines(xlocs=xticks, ylocs=yticks, color='gray', alpha=0.5, linestyle='--',linewidth = 0.75)
    ax.xaxis.set_major_formatter(LONGITUDE_FORMATTER) 
    ax.yaxis.set_major_formatter(LATITUDE_FORMATTER)
    lambert_xticks(ax, xticks)
    lambert_yticks(ax, yticks) 
## Lambert投影系统下网格线绘制代码参考https://gist.github.com/ajdawson/dd536f786741e987ae4e
def find_side(ls, side):
    """
    Given a shapely LineString which is assumed to be rectangular, return the
    line corresponding to a given side of the rectangle.
    
    """
    minx, miny, maxx, maxy = ls.bounds
    points = {'left': [(minx, miny), (minx, maxy)],
              'right': [(maxx, miny), (maxx, maxy)],
              'bottom': [(minx, miny), (maxx, miny)],
              'top': [(minx, maxy), (maxx, maxy)],}
    return sgeom.LineString(points[side])
def lambert_xticks(ax, ticks):
    """Draw ticks on the bottom x-axis of a Lambert Conformal projection."""
    te = lambda xy: xy[0]
    lc = lambda t, n, b: np.vstack((np.zeros(n) + t, np.linspace(b[2], b[3], n))).T
    xticks, xticklabels = _lambert_ticks(ax, ticks, 'bottom', lc, te)
    ax.xaxis.tick_bottom()
    ax.set_xticks(xticks)
    ax.set_xticklabels([ax.xaxis.get_major_formatter()(xtick) for xtick in xticklabels])  
def lambert_yticks(ax, ticks):
    """Draw ricks on the left y-axis of a Lamber Conformal projection."""
    te = lambda xy: xy[1]
    lc = lambda t, n, b: np.vstack((np.linspace(b[0], b[1], n), np.zeros(n) + t)).T
    yticks, yticklabels = _lambert_ticks(ax, ticks, 'left', lc, te)
    ax.yaxis.tick_left()
    ax.set_yticks(yticks)
    ax.set_yticklabels([ax.yaxis.get_major_formatter()(ytick) for ytick in yticklabels])
def _lambert_ticks(ax, ticks, tick_location, line_constructor, tick_extractor):
    """Get the tick locations and labels for an axis of a Lambert Conformal projection."""
    outline_patch = sgeom.LineString(ax.outline_patch.get_path().vertices.tolist())
    axis = find_side(outline_patch, tick_location)
    n_steps = 30
    extent = ax.get_extent(ccrs.PlateCarree())
    _ticks = []
    for t in ticks:
        xy = line_constructor(t, n_steps, extent)
        proj_xyz = ax.projection.transform_points(ccrs.Geodetic(), xy[:, 0], xy[:, 1])
        xyt = proj_xyz[..., :2]
        ls = sgeom.LineString(xyt.tolist())
        locs = axis.intersection(ls)
        if not locs:
            tick = [None]
        else:
            tick = tick_extractor(locs.xy)
        _ticks.append(tick[0])
    # Remove ticks that aren't visible:    
    ticklabels = copy(ticks)
    while True:
        try:
            index = _ticks.index(None)
        except ValueError:
            break
        _ticks.pop(index)
        ticklabels.pop(index)
    return _ticks, ticklabels    

proj = salem.proj_to_cartopy(g[0].proj)

fig = plt.figure(figsize=(6.,5), frameon=True)
ax1 = plt.subplot(111, projection=proj)
cs = cn_back_plot(ax1)
ax1.background_img(name='ne_shaded3',resolution='low')
ax1.add_geometries([rect[0][0]], crs = proj,facecolor='none', edgecolor='k', lw=2.5)
plt.savefig("./模拟嵌套网格.png",dpi=400)
plt.show()
```

[![enter image description here][2]][2]

  [2]: https://i.stack.imgur.com/usoLA.png


### 参考
1. [从NetCDF中提取局部数据](https://beiyuan.me/clip-netcdf/)    
2. [WRF projection](https://fabienmaussion.info/2018/01/06/wrf-projection/)
3. [Python绘制实用地图](Python绘制气象实用地图(附代码和测试数据))
4. [WRF和WPS部分配置选项简介](https://blog.qlsmc.xyz/2018/12/01/wrf-configuration/)
5. [Springer Cartographics LLC](https://springercarto.com/)
