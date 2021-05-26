title: 中国空气质量历史数据整理(HDF5格式)
date: 2018-04-04 23:32:26

categories:

- Research

tags: 

- Air quality
- Python

---

全国空气质量历史数据是大气环境研究的重要基础资料，而我国[官方平台](http://106.37.208.233:20035/)仅提供实时监测信息，而且还只支持IE浏览器(╮(╯_╰)╭)。

我曾尝试采用爬虫工具获取并存储逐时信息，但限于权限，未能在实验室服务器上连续长时间运行。互联网上直接提供数据或API端口的网站有很多，如[环境云](http://www.envicloud.cn/pages/product.html#apis)，[PM25.in](http://pm25.in/), [青悦开放环境数据中心](http://data.epmap.org/)等。其中，[beijingair](http://beijingair.sinaapp.com/)最具分享精神,提供了2013年至今的详尽历史数据，且完全免费。

由衷感谢[@王_晓磊](http://weibo.com/xiaoleiwang)的出色工作和无私分享，极大地推动了我国环境数据的公开与透明。其数据格式为每日一份csv文件存储当日所有站点/城市的逐时信息。在长时间尺度的数据分析时，需逐一阅读各原始文件。此处，我考虑将全年数据文件整合为多维度数据存储格式(HDF5)文件，便于调用和处理。


<!--more-->

## <font color="blue"> 1. 全站点数据整合 </font> 

截至2018年，我国367座城市共设置有1437处国控空气质量监测站点。下文为全年数据打理流程


```python
#library
import pandas as pd
from pandas import HDFStore, DataFrame
from pandas import read_hdf
import os,sys,string
import numpy as np

### CREAT A EMPTY HDF5 FILE

hdf = HDFStore("site_2017_whole_year.h5")

### READ THE CSV FILES AND SAVE IT INTO HDF5 FORMAT
os.chdir("./site_2017/")
files = os.listdir("./")
files.sort()

print files[0]
test_file= "./../china_sites_20170101.csv"
test_f = pd.read_csv(test_file,encoding='utf_8')
site_columns = list(test_f.columns[3:])
print site_columns[1]

### AFTER 2017_2 PUTUO SITE IN SHANGHAI (1141A) is in.
### 注意，2017年2月后上海市增加了普陀站点(1141A),需在site_columns中补充上
#site_columns = site_columns.append(u'1141A')

feature =  ['pm25','pm10','O3','O3_8h','CO',"NO2",'SO2',"aqi"]
fe_dict = {"pm25":1,"aqi":0, 'pm10':3, 'SO2':5,'NO2':7, 'O3':9,"O3_8h":11, "CO": 13}
for k in range(0,len(feature),1):
    data_2017 = {"date":[],'hour':[],}
    for i in range(0,len(site_columns),1):
        data_2017[site_columns[i]] = []
    for file in files[0:]:
        print file
        filename,extname = os.path.splitext(file)
        if (extname == ".csv"):
            datafile =file
            f_day = pd.read_csv(datafile,encoding='utf_8')
             #site_columns = list(f_day.columns[3:])
            for i in range(0,len(f_day),15):
                datetime = str(f_day["date"].iloc[i])
                hour = "%02d" % ((f_day["hour"].iloc[i]))
                data_2017["date"].append(datetime)
                data_2017["hour"].append(hour)
                for t in range(0,len(site_columns),1):
                    data_2017[site_columns[t]].append(f_day[site_columns[t]].iloc[i+fe_dict[feature[k]]])
    print feature[k]
    data_2017  = pd.DataFrame(data_2017)
    hdf.put(feature[k], data_2017, format='table', encoding="utf-8")

```


## <font color="blue"> 2. 全城市数据整合 </font> 
方法类似，每次写入367座城市某一类污染物信息

```python
### CREAT A EMPTY HDF5 FILE
hdf = HDFStore("city_2017_whole_year.h5")

### READ THE CSV FILES AND SAVE IT INTO HDF5 FORMAT
os.chdir("./city_2017/")
files = os.listdir("./")
files.sort()

print files[0]
test_file= "china_cities_20170101.csv"
test_f = pd.read_csv(test_file,encoding='utf_8')
city_columns = list(test_f.columns[3:])
print city_columns[1]

feature =  ['pm25','pm10','O3','O3_8h','CO',"NO2",'SO2',"aqi"]
fe_dict = {"pm25":1,"aqi":0, 'pm10':3, 'SO2':5,'NO2':7, 'O3':9,"O3_8h":11, "CO": 13}
for k in range(0,len(feature),1):
    data_2017 = {"date":[],'hour':[],}
    for i in range(0,len(city_columns),1):
        data_2017[city_columns[i]] = []
    for file in files[0:]:
        filename,extname = os.path.splitext(file)
        if (extname == ".csv"):
            datafile =file
            f_day = pd.read_csv(datafile,encoding='utf_8')
            city_columns = list(f_day.columns[3:])
            for i in range(0,len(f_day),15):
                datetime = str(f_day["date"].iloc[i])
                hour = "%02d" % ((f_day["hour"].iloc[i]))
                data_2017["date"].append(datetime)
                data_2017["hour"].append(hour)
                for t in range(0,len(city_columns),1):
                    data_2017[city_columns[t]].append(f_day[city_columns[t]].iloc[i+fe_dict[feature[k]]])
    print feature[k]
    data_2017  = pd.DataFrame(data_2017)
    hdf.put(feature[k], data_2017, format='table', encoding="utf-8")
```

在此处分享我已转制的数据文件，欢迎大家学习、使用：

* 2017年 [所有站点](https://drive.google.com/open?id=1-Un7PwJ-F3zKm9axmvLt_ImnXzvylud2)/[所有城市](https://drive.google.com/file/d/1YjC73kZuWSXxPaSc4BQs0RGehw6RMN0w/view?usp=sharing)
* 2016年 [所有站点](https://drive.google.com/open?id=1YJng8C8a-qqsKP81SV2ubEEoumsFzIzT)/[所有城市](https://drive.google.com/open?id=1ZDrpXBcR5LneIbVBanC7XA9i_EJ1C8io)
* 2015年 [所有站点](https://drive.google.com/open?id=1J16zum1t4YHSLwTc3oekFRNaDccveYfP)/[所有城市](https://drive.google.com/open?id=1kST06WbMfPtxZ3oQMwbLgaU36vAHOsK1)   
* [全国国控环境监测点列表及经纬度](https://drive.google.com/file/d/1OWMp6Z8VBEjkuAZNbA7h_yidAKxmiiQG/view?usp=sharing)

   
## <font color="blue"> 3. HDF读取应用例举 </font> 

## 提取某个城市的站点浓度信息

```python
import pandas as pd
from pandas import ExcelWriter
import sys,csv,os

#### SELECT THE SPECIES ####
feature = ['pm25', 'pm10', 'O3', 'O3_8h', 'CO', 'NO2', 'SO2', 'aqi']
writer = ExcelWriter('./city_2017.xlsx')
for i in range(0,len(feature),1):
    df = pd.read_hdf("./site_2017_whole_year.h5",\
                     feature[i], encoding = 'utf-8')
    df = df[['date','hour','1744A','1745A','1746A','1747A']]
    df.to_excel(writer,feature[i], index = False)
writer.save()
```



Carbonaceous aerosol in urban and rural European atmospheres: estimation of secondary organic carbon concentrations







###  全国PM2.5年均浓度分布

   ```python
#library
from mpl_toolkits.axes_grid1 import make_axes_locatable
import cartopy
from cartopy.io.shapereader import Reader
from cartopy.feature import ShapelyFeature
from mpl_toolkits.basemap import cm as base_cm


#Load the data select the variable
data_city = {"city":[],'pm25':[],'pm10':[]}
pm25 = pd.read_hdf("./../data/city_2017_whole_year.h5",'pm25', encoding = 'utf-8')
pm10 = pd.read_hdf("./../data/city_2017_whole_year.h5",'pm10', encoding = 'utf-8')

for i,t in enumerate(pm25.columns[2:]):
    data_city['city'].append(t) 
    data_city['pm25'].append(pm25[t].mean())
    data_city['pm10'].append(pm10[t].mean())    
data_city = pd.DataFrame(data_city)    

## Add the lon,lat for each city
## 加载所有国控站点名称，位置的xls文件，已上传至google drive，下载链接见上文
Df = pd.ExcelFile("./站点列表含经纬度-1497个.xlsx", )
df = Df.parse(u'Sheet1')
data_city['lon'] =  [df[df[u'城市'].str[0:2]==k[0:2]][u'经度'].mean() for k in data_city['city']]
data_city['lat'] =  [df[df[u'城市'].str[0:2]==k[:2]][u'纬度'].mean() for k in data_city['city']]

#Plotting 
fig = plt.figure(figsize=(6, 5))
ax = plt.subplot(projection=ccrs.PlateCarree())
ax.set_extent([73, 135, 17, 55])
ax.coastlines(linewidth = 0.2)
ax.add_feature(cartopy.feature.OCEAN)
ax.add_feature(cartopy.feature.BORDERS,linewidth = 0.4)

pm25_plot = ax.scatter(data_city['lon'],data_city['lat'],s = 45,zorder = 2,c = data_city['pm25'],\
                           cmap=base_cm.GMT_seis_r,  lw = 0,alpha = 0.9)
    

   ```

 例图  

 [![enter image description here][1]][1]


[1]: https://i.stack.imgur.com/OPbHd.png