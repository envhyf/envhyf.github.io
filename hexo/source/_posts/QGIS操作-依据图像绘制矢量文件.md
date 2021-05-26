title: 'QGIS: 依照现有图片资料绘制矢量文件'
date: 2019-02-28 16:26:26

categories:

- Tech

tags:

- GIS

---

有时，我们无法直接获取某些地理数据的精确位置信息，仅能从网络中下载他人的地图作品。因而，我请教了一名规划师同学，参考网络图像，通过适配地理范围，手工获取其实际地理属性。本文以我国集中采暖南北分界线的绘制为例，简述QGIS软件的具体实现步骤。

<!--more-->

上世纪50年代，由于能源缺乏，我国以南北地理界限（秦岭-淮河线）为划分依据，在苏联的帮助下，在该线以北建立了集中供暖系统，沿用至今。下图可见，我国的集中供暖系统并不以省为界限，如四川省的甘孜和阿坝位于供暖线以北，江苏省的徐州位于供暖线以北，而河南省的信阳则没有集中供暖。    


<img src='https://i.stack.imgur.com/GA9Ai.png' width=400>

这条分界线具有多重地理学意义，它还是
 * 我国800毫米等降水量线
 * 1月份0℃等温线
 * 温带季风与亚热带季风气候分界线
 * 温带落叶阔叶林与亚热带常绿阔叶林带分界线
 * 湿润与半湿润区分界线
(摘录自 [国家气象局 气象科普园](http://www.cma.gov.cn/kppd/kppdrt/201712/t20171215_458231.html))。    

同时，该分界线同时具有特殊的历史意义，其东段是我国南北朝、金朝/蒙元-南宋时期的南北政权边界。



## 1. 安装Freehand raster georeferencer

QGIS的[Freehand第三方工具包](http://gvellut.github.io/FreehandRasterGeoreferencer/)可载入图片图层并简单处理，主要用途应是卫星遥感图像的像元分析。该工具包的功能界面非常简洁，工具栏 <img src="https://i.stack.imgur.com/7u3xr.png" width=250>从左至右，功能依次为“载入”，“移动”，“旋转”，“放缩”，“复合操作”，“透明度减小”，“透明度增加”，“输出” 。

## 2. 绘制Polyline

我们分别导入中国分省地图shapefile文件和jpeg格式底图。利用Freehand不断调整合图像位置、大小，使二者基本重合（tip: 按住“⌘”或"ctrl"可按原始比例缩放），如下图。可以看出南方各省份吻合较好，但台湾与海南有较大的偏差，可能是原始图片投影系统上的差异。此处作为案例，未深究其原因。

<img src='https://i.stack.imgur.com/VNsAw.png' width = 400>





选择"Layer"->"Creat Layer"->"New Shapefile Layer"新建point格式的矢量文件，利用 <img src='https://i.stack.imgur.com/7Ntd6.png' width =150>依照图片勾勒界限，如下图：

<img src='https://i.stack.imgur.com/HrPua.png' width=400>

## 3. 后续应用

利用Python语言读取编写好的矢量文件，通过Cartopy, Fiona, Shapely等工具包可做进一步分析。

例如，以该线为依据将我国空气质量监测站点进行分类，探究分采暖/非采暖城市间大气污染特征的差异。该工作可由简单的代码指令完成，如下：

```python
import warnings
warnings.filterwarnings('ignore')
from shapely.geometry import shape
from shapely.geometry import LineString
# loading the boundary layer
import fiona
fname = './N-S_boundary.shp'
line1 = fiona.open(fname)
line1 = shape(line1.next()['geometry'])
# set a end point which is the southernmost for all stations.
end_point  = (dy[dy['lat']==dy['lat'].min()]['lon'].values[0],dy[dy['lat']==dy['lat'].min()]['lat'].values[0])
# loop all monitoring stations for classification
dy['NS']= np.nan
for i in range(0,len(dy),1):
    start_point = (dy['lon'].iloc[i],dy['lat'].iloc[i])
    line2       = LineString([start_point, end_point])
    if line1.intersection(line2).is_empty:
        dy["NS"].iloc[i]='S'
    else:
        dy["NS"].iloc[i]='N'        
```
按供暖分界线区分后，大气采样站点的分布如图所示

<img src='https://i.stack.imgur.com/0LaR2.png' width = 500>
