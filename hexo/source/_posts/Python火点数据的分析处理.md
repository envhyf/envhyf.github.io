title: 'MODIS卫星火点数据的简易处理与可视化分析'
date: 2017-06-22 16:26:26

categories:

- Research

tags: 

- Air quality
- Python

---

提要：此文介绍利用Python语言处理NASA MODIS火点数据（Global Monthly Fire Location Product，MCD14ML），可实现的基本功能包括：（1）特定时期的火点信息提取；（2）特定区域内的火点信息提取；（3）火点密度空间分布的计算与可视化表达


<!--more-->

### <font color="blue"> 火点数据背景介绍</font>   

生物质(biomass)和石油、煤炭等一样，是一类重要的能源物质。在我国的广大农村地区，生物质能源(如秸秆、薪柴以及牲畜粪便等)占据重要地位，在1979年前，更是占到整个农村能源消费总量的70%以上[^1][1]。同时，由于生物质燃烧过程中，排放大量气态以及颗粒态的污染物，对大气能见度造成影响，危害人体健康，同时具有着一定的气候效益。


环境遥感技术依据中红外波段对高温热源的敏感特性，可对全球陆面的火点信息进行检测，识别生物质开放燃烧事件的具体位置。目前较为成熟的火点数据库主要有两类：[MODIS](https://earthdata.nasa.gov/earth-observation-data/near-real-time/firms/c6-mcd14dl)以及[VIIRS 375m](https://earthdata.nasa.gov/earth-observation-data/near-real-time/firms/v1-vnp14imgt)，以多种数据格式为公众提供历史火点信息。

[![enter image description here][2]][2]

<center>图1. FIRMS Web Fire Mapper截图，2017年6月21日-22日VIIRS 375m数据集火点信息</center>      





### <font color="blue"> 特定时期燃烧事件数据的提取</font> 

此处介绍MODIS数据集中应用最为广泛的__MCD14ML__数据，对基础数据的打理和分析做以基本的介绍。__MCD14ML__数据的单个文件包含某个月的全球火点信息，具体获取方法如下：

```	   python
#### 下载网址及用户名 ###   
#ftp://fuoco.geog.umd.edu/modis/C6/mcd14ml/
#用户名:fire
#密码: burnt  
#----------------------

#### 1. 文件的下载下载
# 此处参考https://stackoverflow.com/questions/7006574/how-to-download-file-from-ftp中的方法进行下载，亦可直接在浏览器中手动下载
    
import mimetypes,os,urllib2,urlparse
def filename_from_url(url):
	 		return os.path.basename(urlparse.urlsplit(url)[2])

def download_file(url):
	"""Create an urllib2 request and return the request plus some useful info"""
	name = filename_from_url(url)
	r = urllib2.urlopen(urllib2.Request(url))
	info = r.info()
	if 'Content-Disposition' in info:
		# If the response has Content-Disposition, we take filename from it
		name = info['Content-Disposition'].split('filename=')[1]
		if name[0] == '"' or name[0] == "'":
			name = name[1:-1]
	
	elif r.geturl() != url:
	# if we were redirected, take the filename from the final url
		name = filename_from_url(r.geturl())
	content_type = None
	if 'Content-Type' in info:
		content_type = info['Content-Type'].split(';')[0]
	# Try to guess missing info
	if not name and not content_type:
		name = 'unknown'
	elif not name:
		name = 'unknown' + mimetypes.guess_extension(content_type) or ''
	elif not content_type:
		content_type = mimetypes.guess_type(name)[0]
	return r, name, content_type
import shutil
def download_file_locally(url, dest):
	req, filename, content_type = download_file(url)        
	if dest.endswith('/'):
		dest = os.path.join(dest, filename)
	with open(dest, 'wb') as f:
		shutil.copyfileobj(req, f)
	req.close()
	     

time = "201609" ## 此处以2016年09月为例
filename = 'MCD14ML.'+time+'.006.01.txt.gz'
download_file_locally('ftp://fire:burnt@fuoco.geog.umd.edu/modis/C6/mcd14ml/',filename)

	   
#### 2. 文件预处理
# 原始文件中存在多类分隔符，需进行预先处理，统一替换为“ ”，便于后续读取。
with open(filename, 'r') as file :
	filedata = file.read()

# Replace the target string
filedata = filedata.replace('   ', ' ')
filedata = filedata.replace(' ', ' ')

#Write the file out again
with open(filename, 'w') as file:
	file.write(filedata)

#### 3. 文件读取
df = pd.read_csv(filename, sep='\s+')
df.head()		
```



### <font color="blue"> 特定区域内燃烧事件的提取</font>     

原始数据中包含该段时期火点位置的有关信息，对于研究生物质燃烧对于局地污染事件发生的影响，应对于特定地理位置内的火点信息进行提取。该工作同样可以借助fiona包，利用简单的判断语句实现。这里我们以四川省为例，对2016年9月发生在该省境内的逐日火点信息进行提取。  

```	python
import fiona
from shapely.geometry import shape,Polygon, Point
# read the shapefile
sc_area = fiona.open("/Users/HYF/Documents/NanChong/GIS/sichuan.shp")
pol = sc_area.next()
from shapely.geometry import shape
geom = shape(pol['geometry'])
poly_data = pol["geometry"]["coordinates"][0]
poly = Polygon(poly_data)

### zip the x coordidate and y coordidate of each hotspot
points = zip(*np.array([df.lon.values, df.lat.values]))
### to identify whether the hotspot is inside the boundary or not
mask = np.array([poly.contains(Point(x, y)) for x, y in points])  

df["MASK"] = mask
select = df[df["MASK"] == True]
select['datetime']  = pd.to_datetime(select.YYYYMMDD,format = '%Y%m%d')

```
### <font color="blue"> 简单的图形绘制</font> 

#### <font color="grey"> 火点分布图</font>

例如，此处我们专门考察2016年9月5日四川省境内的火点分布情况：


```	python
####Only select the biomass events in 2016-09-05
select[select.datetime == '2016-09-05']

m = Basemap(projection='cyl', resolution='l',llcrnrlon = 97.2,llcrnrlat=26.0,urcrnrlon = 109.0,urcrnrlat=34.5)
m.readshapefile('/Users/HYF/Documents/NanChong/GIS/sichuan', 'sichuan', color='b', zorder=3,linewidth=1.5)
m.drawcoastlines(color = '0.15')
lon, lat = m(select.lon,select.lat)
plt.scatter(lon,lat, color = 'red')

parallels = np.arange(26.,34,3.)
m.drawparallels(parallels,labels=[False,True,True,False])
meridians = np.arange(97,109.,3.)
m.drawmeridians(meridians,labels=[True,False,False,True])
plt.show()
```

