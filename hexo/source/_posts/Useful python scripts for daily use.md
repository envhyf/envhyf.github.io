title: 'Useful python scripts for daily uses'
date: 2018-07-22 16:26:26

categories:

- Tech

tags: 

- Python
- Visualization

---

In this post, I present some useful tips for how to easily process the dataset using pandas or numpy, some tricks for data visualization, and some other functions. Many of them were collected from [stackoverflow](https://stackoverflow.com)

<!--more-->

## <font color="blue"> 1.Data processing </font> 

```python
#1. Check for repeated samples      
from collections import Counter   
# df is pandas.Dataframe, column 'Sample' contains repeated information重复查询
print [item for item, count in Counter(df["Sample"].values).iteritems() if count >1] 

#2. Build new dataframe based on the user-defined column names
### N: length
def new_df(N, LIST):
	df = pd.DataFrame(np.nan, index=np.arange(0,N,1), columns=['A'])
	for i,t in enumerate(LIST):
		df[LIST[i]] = np.nan
		df = df.drop(['A'],1)  
        
#3. Change the interger to N digits 
## example: transform all values in the column 'Month' (e.g., 2) to 2-digit number (e.g., 02)
metho1: df.MONTH.apply("{0:0=2d}".format)       
method2: "%02d"&(string)

#4. Replace the values in specific columns
pd.options.mode.chained_assignment = None
df.loc[condition function, 'column'] = xxx  

#5. Find the index of the nearest value to a specifc n
def find_nearest(array,value):
	idx = (np.abs(array-value)).argmin()
	return array[idx]   

#6. Test a value if it is NaN:      
def isNaN(num):
	return num != num

#7. Test dataframe if it is empty
df.empty==True

#8. Zip two array
from itertools import product        
c = list(product(df.lon.values, df.lat.values))       

#9. String list function
## Add an element
list.insert(0,'A')
## Delete an element by index
list.pop(2)
## Delete an element by its value
list.remove('A')

#10. assiging value to specific location of a dataframe
## method1 -- the fastest 
df.set_value('row index','column name',value)
## method2 -- moderate speed
df['column name']['row index']=value
## method3 -- the slowest
df.at('column name', 'row index')=value


#11. group the dataframe with user-defined range settings and labels
bins = [0,50, 60, 70, 80, 90,100]
group_names = ['<50', '50-60', '60-70', '70-80','80-90','>90']
df['categories'] = pd.cut(df['RH'], bins, labels=group_names)
## PS: plotting the curve lines of the catagorized data
fig = plt.figure()
for i in range(0,len(categories.unique())-1,1):
	ddf = df[df['categories'] == categories.unique()[i]]
	plt.scatter(ddf['PM2.5'], ddf['VISIBILITY'], alpha = 0.45, label ='RH: '+ roup_names[i])    
	popt, pcov = curve_fit(func, ddf["PM2.5"], ddf["VISIBILITY"])        
	xdata = np.arange(0, 300, 4)
	plt.plot(xdata, func(xdata, *popt),  zorder =3)
plt.legend(ncol = 2)    
plt.show()    

#12. Transform the list to string
import ast
x = u'[ "A","B","C","D"]'
x = ast.literal_eval(x)     
print x
>> ABCD

#13. Selecting the sub-datasets by datetime 
mask = (met['Date'] < '2016-03-01')|(met['Date'] >= '2016-12-01')
met_sub = met.locmask

#14. Sorting the labels by their corresponding values
SE_COL =['Cr', 'Co', 'Ni', 'Cu', 'Zn', 'As', 'Se', 'Mo', 'Cd', 'Pb','V','Mg', 'Ca', 'K', 'Ti','Mn', 'Fe', 'Ba', 'Sr']
SE_COL = [SE_COL[t] for t in np.argsort(np.array([vals.dropna().mean() for col, vals in 
        				pm25_ef.iteritems()]))]    

#15. Drop np.nan and then find the 10th percentile
v  = np.isnan(df['value'])
ra_ = (df['value'].values)[~v]
se_index = ra_.argsort().argsort()[:int(0.1*len(df))]
sorted(ra)[int(0.1*len(ra))]  

#16. Rename the pandas column
df.rename(columns={'oldName1': 'newName1', 'oldName2': 'newName2'}, inplace=True)

#17. Delete specific columns
columns = ['Col1', 'Col2', ...]
df.drop(columns, inplace=True, axis=1)

#18. Add new columns
df[new column name] = pd.Series(Data, index =df.index)

#19. Delete the rows with np.nan
df = df[~np.isfinite('col1')]

#20. Sort the values from the biggest
var_exp = [(i) for i in sorted(eigen_vals, reverse=True)] 

#21. Saving the csv file containing chinese 保存带中文的csv文件
import sys
reload(sys)
sys.setdefaultencoding('utf-8')	

#22 time series dataset
## Add one day to the current datetime, e.g., '2017-11-01' -> '2017-11-02'
pd.to_datetime('2017-11-01')+ pd.to_timedelta(1, 'D')

#23. get the keys and values of a dictornary
s = [value for key,value in dict_.items()] 

#24. Binining the data with two different methods
from pysal.esda.mapclassify import Natural_Breaks as nb
breaks = nb(df.value,k=5)
class_ = pd.DataFrame({'class': breaks.yb}, index=df[df['value'].notnull()].index)
df = df.join(class_)
df.class.fillna(-1, inplace=True)

#25. print pd.dataframe in the format of markdown
from tabulate import tabulate
print(tabulate(response_df, tablefmt="pipe", headers="keys", showindex=False))

#26. saving the 2-d array in txt and read it
np.savetxt("mask_PM_0.1x0.1.txt",mask)
mask = np.loadtxt('./mask_PM_0.1x0.1.txt')

#27. np.array dropna for the NaN of ratios 
t = np.array([(a/b) for a, b in zip(dyn_w[spec].values,dyn_s[spec].values)])
t = t[~np.isnan(t)]

# 28. How to select the data points within specific area
### 长三角
yrd_area = fiona.open("./data/shp/YRD/yrd_b.shp")
pol = yrd_area.next()
from shapely.geometry import shape
geom = shape(pol['geometry'])
poly_data = pol["geometry"]["coordinates"][0][0]#[0:12]
poly = Polygon(poly_data)
yrd_mask = np.array([poly.contains(Point(x,y)) for x,y in zip(data_city['lon'],data_city['lat'],)])
data_yrd = data_city[yrd_mask]
data_yrd = data_yrd.reset_index()

# 29. 存储中文xls报错问题处理
df = pd.read_csv('./xxx.txt', delimiter = "\t",encoding='utf-8')
with pd.ExcelWriter('xxx.xls',options={'encoding':'utf-8'}) as writer:  #
    df.to_excel(writer, sheet_name=u'sheet1')
    
# 30. split the list by specfic string (e.g., quotes)
[s for s in l.split('"') if s.strip() != ''][1]

# 31. replace specific values by condition 
df.loc[df['COLA']!='XXX','COLB']=yyy

# 32. Select subset of netcdf file by specific attributes
import netCDF4
val_ = ['HGT_M', 'XLONG_C','XLAT_C']

with netCDF4.Dataset("geo_em.d01.nc") as src, netCDF4.Dataset("out.nc", "w") as dst:
    # copy global attributes all at once via dictionary
    dst.setncatts(src.__dict__)
    # copy dimensions
    for name, dimension in src.dimensions.items():
        dst.createDimension(
            name, (len(dimension) if not dimension.isunlimited() else None))
    # copy all file data except for the excluded
    for name, variable in src.variables.items():
        if name in val_:
            x = dst.createVariable(name, variable.datatype, variable.dimensions)
            dst[name][:] = src[name][:]
            # copy variable attributes all at once via dictionary
            dst[name].setncatts(src[name].__dict__)

# 33. combining the lists
code_ = []
for pr in pc_code:
    code_ = list(itertools.chain.from_iterable([code_,[t['code'] for t in pr['children']]]))    

# 34. 2-D数据插值
#向新网格进行插值
from scipy import interpolate
hfunc = interpolate.interp2d(lon_dust_sub,lat_dust_sub,PM_dust)
new_dust = np.zeros(len(lat_anthr_sub)*len(lon_anthr_sub))
t =  0 
for i in range(0,len(lat_anthr_sub),1):
    for j in range(0,len(lon_anthr_sub),1):
        new_dust[t] = hfunc(lon_anthr_sub[j],lat_anthr_sub[i])
        t+=1
dust_int = new_dust.reshape(len(lat_anthr_sub),len(lon_anthr_sub))

# 35. 排序获得其序列index
sorted(range(len(s)), key=lambda k: s[k])

# 36. 输出markdown格式的

# 37. 
# -*- coding:utf-8 -*-


# 39. Sort时注意排序
sorter = source_
df_contr_cr.source = df_contr_cr.source.astype("category")
df_contr_cr.source.cat.set_categories(sorter, inplace=True)
df_contr_cr = df_contr_cr.sort_values(["source"]) 

df['categories'].value_counts().reindex(group_names)

# 39. 分钟数据转化为小时数据
df2_re = df2.groupby(np.arange(len(df2))//60.0).mean()

# 40. 有效位数输出
to_clipboard(float_format = '%.4f')

# 41.自然基金报告下载

# 41.自然基金报告下载
import os
for i in range(0,100,1):
    os.system("wget http://output.nsfc.gov.cn/report/41/41205122_%i.png" %i)    
import os
import natsort
from fpdf import FPDF

files = os.listdir('./')
files = natsort.natsorted(files, reverse = False)
files = files[0:]
pdf = FPDF()

for image in files[0:]: 
    if image[-3:]=='png': 
        pdf.add_page()  
        pdf.image(image,0,0,210,297)  
pdf.output("../hh.pdf", "F")   

# 43. 缺失值监测
pd.isnull()
dataframe[xxx].values.isnull()

# 44. If Else in list comprehension
[x for x in list(df.columns[2:]) if x not in ['重庆','山西','西藏']]

# 45. Two variables in list comprehension (double iteration)

# 46. replace or extract element from list every nth

n = 3
result = [[x if i % n else 0 for i, x in enumerate(y)] for y in a]
```

## <font color="blue"> 2. Data statistics  </font>

```python
#1. Curve fitting (y = ax + b)
from scipy import stats
import numpy as np
slope, intercept, r_value, p_value, std_err = stats.linregress(x,y)
##1.1 remove the nan value first!
mask = ~np.isnan(x_) & ~np.isnan(y_)
slope, intercept, r_value, p_value, std_err = stats.linregress(x_[mask], y_[mask])

#2. Fitting by user-defined function 
from scipy.optimize import curve_fit		
def func(x, a,b):
    return ax**2+bx#+0.55
def get_r(f,x,y,popt):
    residuals = y- f(x, *popt)
    ss_res = np.sum(residuals**2)    
    ss_tot = np.sum((y-np.mean(y))**2)    
    r_squared = 1 - (ss_res / ss_tot)
    return r_squared
popt, pcov = curve_fit(func, df['x'], df['y']) 
print "## f(x) = ax*2 +bx ##"
print "## a##"
print popt
print "## r-square ##"
print get_r(func,df['x'], df['y'], popt )

#3. pivot table 
df.groupby(['site']).get_group(df.site.unique()[i])['value'].mean()

#4. Summarize the frequency
clu_d['Cluster'].value_counts().to_frame()
clu_d['Cluster'].value_counts().index.tolist()  

#5. one-way ANOVA test
# 如何统计 seasonal anova difference
spec_ = ['crustal','trace','OM',"EC",'SO42-','NO3-','Cl-','NH4+','n_K',]
val_annual,val_sp,val_su,val_au,val_wi = [],[],[],[],[]
for t in sp_pie:
    val_sp=df[df.season=='Spring'][t].dropna().values
    val_su=df[df.season=='Summer'][t].dropna().values
    val_au=df[df.season=='Autumn'][t].dropna().values
    val_wi=df[df.season=='Winter'][t].dropna().values
    print t
    print stats.f_oneway(val_sp,val_su, val_au,val_wi)

# pearson correlation 
mask = ~np.isnan(df['Ma_re'])&~np.isnan(df['PM25'])
stats.pearsonr(df['Ma_re'][mask],df['PM25'][mask])
```

## <font color="blue"> 3. Data visualization  </font>

```python
#1. Temperature symbol
ax.set_ylabel(u'Temperature (\N{DEGREE SIGN}C)', fontweight='bold')

#2. Match the length of colorbar with the figure
from mpl_toolkits.axes_grid1 import make_axes_locatable
divider = make_axes_locatable(ax1)
cax = divider.append_axes("right", size="5%", pad=0.1)
cbar = plt.colorbar(cf, cax=cax)    
cbarlabel = "xxxx"
cbar.ax.set_xlabel(cbarlabel,size = 8,labelpad=-35)

#3. User-defined colorbar location and ticklabel editting
cbaxes = fig.add_axes([0.83, 0.125, 0.1, 0.01]) 
cbar = plt.colorbar(ss,cax=cbaxes,orientation='horizontal')
loc_ = np.array([5,10,15,20,25])
cbar.set_ticks(loc_)
cbar.set_ticklabels(loc_)
## two methods for ticklabel fontsize
(1) cbar.ax.tick_params(labelsize=8)
(2) cbar.ax.set_xticklabels(cbar.ax.get_xticklabels(), fontsize=8)
cbar.ax.set_xlabel(r'$\mathregular{OC/EC}$',fontsize = 8,labelpad = -28)

#4. Chinese setting
from matplotlib.font_manager import FontProperties 
mpl.rcParams['font.sans-serif'] = ['Microsoft YaHei']# 微软雅黑的中英文混排效果较好
plt.rcParams['axes.unicode_minus'] = False  

#5. User-defined axes setting
def stylize_axes(ax):
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    ax.xaxis.set_tick_params(top='off', direction='out', width=1)
    ax.yaxis.set_tick_params(right='off', direction='out', width=1)

#6. Plotting the first-order regression line
## (x,y) is the original datasets
from scipy import stats
slope, intercept, r_value, p_value, slope_std_error = stats.linregress(x, y)
xx = np.arange(x.min(),x,max(),0.01)
yy = slope*xx+intercept
clabel = # make the font itatilc style
plt.plot(xx,yy,color ='k', lw =1.2, linestyle = "-", label = clabel)

#7. Setting the two subplots shared the same aspect ratio
## Noted that when one subplot using basemap or cartopy (geo-based coordinate), and the other subplot is in basic matplotlib style, there aspect ratio would not be the same. 
def get_aspect(ax):
    xlim = ax.get_xlim()
    ylim = ax.get_ylim()
    aspect_ratio = abs((ylim[0]-ylim[1]) / (xlim[0]-xlim[1]))
    return aspect_ratio
fig = plt.figure(figsize=(12,8))
ax1=plt.subplot(121, projection=ccrs.PlateCarree())
ax2=plt.subplot(122)
ax2.set_aspect(get_aspect(ax1) / get_aspect(ax2))

#8. Fake legend markers
#参考 https://jakevdp.github.io/PythonDataScienceHandbook/04.06-customizing-legends.html
for area in [100, 300, 500]:
    plt.scatter([], [], c='k', alpha=0.3, s=area,
                label=str(area) + ' km$^2$')
plt.legend(scatterpoints=1, frameon=False, labelspacing=1, title='City Area')

#9. Remove the plotting items and retain the label

#10. Remove the tick lines while retain the ticklabels
ax.tick_params(axis=u'both', which=u'both',length=0)

#11. plotting two subplots with different size
fig = plt.figure(figsize=(7,4))
##definitions for the axes
left, width = 0.07, 0.6
bottom, height = 0.1, .8
bottom_h = left_h = left+width+0.05
rect_cones = [left, bottom, width, height]
rect_box = [left_h, bottom, 0.5, height]
ax1 = plt.axes(rect_cones,projection=ccrs.PlateCarree())
ax2 = plt.axes(rect_box)
ax2.set_aspect(get_aspect(ax1) / get_aspect(ax2))

#12. fake circle legend
def plot_legend(title):
    s = [1,3,5,7,9]
    c = plt.cm.binary(np.arange(5)/5.0)
	labels =['A',"B",'C',"D","E"]
    cir1 =  plt.scatter([1], [2], c=c[0], alpha=0.8, s=s[0],label=r'<0.5')
    cir2 =  plt.scatter([1], [2], c=c[1], alpha=0.8, s=s[1],label=labels[1])
    cir3 =  plt.scatter([1], [2], c=c[2], alpha=0.8, s=s[2],label=labels[2])
    cir4 =  plt.scatter([1], [2], c=c[3], alpha=0.8, s=s[3],label=labels[3])
    cir5 =  plt.scatter([1], [2], c=c[4], alpha=0.8, s=s[4],label=labels[4])    
    leg = plt.legend([cir1,cir2,cir3,cir4,cir5],labels,scatterpoints = 1,  \
                     frameon=False, labelspacing=0.9, ncol =2,
                     fontsize=8,title=title , loc = [0.05,0.1])   
    
#13. seperated colors from colormap
c_list = plt.cm.rainbow(np.arange(6)/6.0)

# 14. set twin axis with different color
axu = ax.twinx()
axu.spines["right"].set_position(("axes", 1.25))
so_,=plt.plot(xxx)
axu.yaxis.label.set_color(so_.get_color())
axu.spines['right'].set_color(so_.get_color())
axu.spines["right"].set_edgecolor(so_.get_color())
axu.tick_params(axis='y', colors=so_.get_color())

# 15. add a rectangle
from matplotlib.patches import Rectangle
import matplotlib.patches as mpatches
rec = mpatches.Rectangle((position[i]-
                    width/2.0,bot_[i]),width,hei_[i],linewidth=1,edgecolor='b',facecolor='none',zorder=12)
    ax.add_patch(rec)

# 16. output the color by pre-defined alpha
def make_rgb_transparent(rgb, bg_rgb, alpha):
    return [alpha * c1 + (1 - alpha) * c2
            for (c1, c2) in zip(rgb, bg_rgb)]
from matplotlib import colors
import matplotlib.pyplot as plt

alpha = 0.5

kwargs = dict(edgecolors='none', s=3900, marker='s')
for i, color in enumerate(['red', 'blue', 'green']):
    rgb = colors.colorConverter.to_rgb(color)
    rgb_new = make_rgb_transparent(rgb, (1, 1, 1), alpha)
    print(color, rgb, rgb_new)
    plt.scatter([i], [0], color=color, **kwargs)
    plt.scatter([i], [1], color=color, alpha=alpha, **kwargs)
    plt.scatter([i], [2], color=rgb_new, **kwargs)

# 17. terrain map
def truncate_colormap(cmap, minval=0.0, maxval=1.0, n=100):
    new_cmap = colors.LinearSegmentedColormap.from_list(
    'trunc({n},{a:.2f},{b:.2f})'.format(n=cmap.name, a=minval, b=maxval),
    cmap(np.linspace(minval, maxval, n)))
    return new_cmap
cmap = plt.get_cmap('terrain')
terrain_cmap = truncate_colormap(cmap, 0.15, 0.9)  

# 18. 不等距subplots
import matplotlib.gridspec as gridspec
fig = plt.figure(figsize=(9,6))
gs = gridspec.GridSpec(46,1)
row_xx = 23
# pm25_pm10
ax1 = plt.subplot(gs[0:row_xx, 0])

# 19. two axis one legend
# ask matplotlib for the plotted objects and their labels
lines, labels = ax.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax2.legend(lines + lines2, labels + labels2, loc=0)

# 20. Cartopy longitude
## 方法1
ax.set_xticks([0, 60, 120, 180, 240, 300, 360], crs=ccrs.PlateCarree())
ax.set_yticks([-90, -60, -30, 0, 30, 60, 90], crs=ccrs.PlateCarree())
lon_formatter = LongitudeFormatter(zero_direction_label=True)
lat_formatter = LatitudeFormatter()
ax.xaxis.set_major_formatter(lon_formatter)
ax.yaxis.set_major_formatter(lat_formatter)
## 方法2
ax.set_xticks(np.arange(extent[0]+2, extent[1]+2, 15), crs=ccrs.PlateCarree())
ax.set_yticks(np.arange(extent[2]+3, extent[3], 15), crs=ccrs.PlateCarree())
ax.set_xticklabels([r'$\mathrm{75^o E}$',r'$\mathrm{90^o E}$',r'$\mathrm{105^o E}$',\
                    r'$\mathrm{120^o E}$',r'$\mathrm{135^o E}$',])
ax.set_yticklabels([r'$\mathrm{20^o N}$',r'$\mathrm{35^o N}$',r'$\mathrm{50^o N}$'])

# 21 'GeoAxesSubplot' object has no attribute '_hold'
from matplotlib.axes import Axes
from cartopy.mpl.geoaxes import GeoAxes
GeoAxes._pcolormesh_patched = Axes.pcolormesh

# 22.中英文混排
# # -*- coding: utf-8 -*-
import matplotlib.pyplot as plt
from matplotlib.font_manager import FontProperties

song_ti = FontProperties(fname=r'/Library/Fonts/Songti.ttc', size=20)
times_new_roman = FontProperties(fname=r'/Library/Fonts/Arial Black.ttf', size=15)
ax = plt.gca()
ax.set_title(u'能量随时间的变化', fontproperties=song_ti)
ax.set_xlabel('Time (s)', fontproperties=times_new_roman)
ax.set_ylabel('Energy (J)', fontproperties=times_new_roman)
plt.show()

# 23. no-legend
(label='_nolegend_')

# 25. plot with grey background
ax.set_facecolor("#F5F5F5")

# 26. Midpoint-normalized
import matplotlib.colors as colors
#https://stackoverflow.com/questions/25500541/matplotlib-bwr-colormap-always-centered-on-zero
class MidpointNormalize(colors.Normalize):
    def __init__(self, vmin=None, vmax=None, midpoint=None, clip=False):
        self.midpoint = midpoint
        colors.Normalize.__init__(self, vmin, vmax, clip)

    def __call__(self, value, clip=None):
        # I'm ignoring masked values and all kinds of edge cases to make a
        # simple example...
        x, y = [self.vmin, self.midpoint, self.vmax], [0, 0.5, 1]
        return np.ma.masked_array(np.interp(value, x, y))
    
    
# 27. 月份colorbar
cbaxes = fig.add_axes([0.29, 0.27, 0.18, 0.02]) 
cbar = plt.colorbar(ss,cax=cbaxes, orientation='horizontal')
n = 6
st_po = []
for i in range(0,n,1):
    st_po.append(np.array_split(pd.to_datetime(sorted(date_point)),n)[i].values[0])
cb_ticks = [float(c) for c in st_po]
cbar.ax.set_xticklabels(pd.to_datetime(cb_ticks).strftime('%b')) #B     
cbar.ax.tick_params(labelsize=8.5)  

# 28. nclcmaps

# 29. 创建新的colormap
 cm = LinearSegmentedColormap.from_list('test',plt.cm.BuPu(np.arange(6)/6.0)[1:], N=5)


 # 30. 最好的terrain map
http://chris35wills.github.io/discrete_colourbar/
    
# 31. 对数坐标系统显示实际数值
from matplotlib.ticker import StrMethodFormatter, NullFormatter
ax.yaxis.set_major_formatter(StrMethodFormatter('{x:.3f}'))
ax.yaxis.set_minor_formatter(NullFormatter())  

# 32. loop figure for subplot
fig, axs = plt.subplots(2,4, figsize=(15, 6), facecolor='w', edgecolor='k')#subplot_kw={'projection': ccrs.PlateCarree()}
fig.subplots_adjust(hspace = .5, wspace=.001)
axs = axs.ravel()

for i in range(8):

    axs[i].contourf(np.random.rand(10,10),5,cmap=plt.cm.Oranges)
    axs[i].set_title(str(250+i))
    
# patch_box  
from matplotlib.collections import PatchCollection
from matplotlib.patches import Rectangle
def PATCH_BOX(ax,pos,wid):
    for x in pos[::2]:
        width = wid/2.0
        facecolor = "#F0F0F0" 
        rect = Rectangle((x-width, 0.00), width*4.0,1000000,facecolor=facecolor,linewidth = 0)#           clip_on=False,
        ax.add_patch(rect)     
        
# 33. cartopy extent 报错问题
https://github.com/SciTools/cartopy/issues/837
pip uninstall shapely && pip install --no-binary :all: shapely

# 34. shapely的向量化总是报错
# https://github.com/Toblerity/Shapely/issues/810
pip uninstall shapely & pip install shapely --no-binary shapely==1.7a2
        
 # 35. scatter图的外环
linewidths=1
   
```

## <font color="blue"> 4. Jupyter and Python setting  </font>

```python
#1. Full width displaying all the time
from IPython.core.display import display, HTML
display(HTML("<style>.container { width:100% !important; }</style>"))

#2. Import script.py files
import os,sys
scriptpath = "/Users/HYF/Downloads/stackoverflow_tag_cloud-master/"
##Add the directory containing your module to the Python path (wants absolute paths)
sys.path.append(os.path.abspath(scriptpath))
## Do the import
import stackoverflow_users_taginfo
from stackoverflow_users_taginfo import taginfo
info = taginfo(link = 4918632, num_tags = 400)
WordCloud().generate_from_frequencies(info).to_image().save('TagCloud.pdf')

#3. Check the installing path
import time
time.file

#4. Get the pathname of the file
script_dir = os.path.dirname(os.path.realpath(filename)) + os.sep             

#5. Change the size of fthe uploaded figure
<img src="https://i.stack.imgur.com/kK1LC.png" width="100">

#6. Inquire the information of computer, Python and its package using watermark package
▶pip install watermark
%load_ext watermark 
## Compiler, system, and CPU
%watermark 
## numpy version
%watermark -p numpy

# 7. 取消Warnings显示
import warnings
warnings.filterwarnings('ignore')

```

## <font color="blue"> 5. Some tex characters  </font>

```
\leq  <
```



##   <font color="blue">6. Spatial analysis  </font>

```python
# 1. intersection of two lines
import fiona
from shapely.geometry import shape
from shapely.geometry import LineString
line1 = fiona.open(fname)# fname is the path to a specific polyline-format shapefile.
line1 = shape(line1.next()['geometry']) 
start_point, end_point = (111,22),(33,33)
line2 = LineString([start_point, end_point])
print line1.intersection(line2)
print line1.intersection(line2).is_empty

# read .tif
import warnings
import rasterio
warnings.filterwarnings('ignore')
import skimage.transform as st 
pathToRaster  =r'/Users/HYF/Downloads/2014/per2014/中国年均降水.tif'
xs = np.array([100.5, 12.0])
ys = np.array([25.5, 42.0])
# src.bounds
with rasterio.open(pathToRaster) as src:
    arr = src.read()
    arr[arr<0] = np.nan
    arr = arr[0,:,:]
    rows, cols = rasterio.transform.rowcol(src.transform, xs, ys)    
    
    
# mask point series
mask_= []
for i in range(0,len(df_fire_cn),1):
    xy_point = gpd.geoseries.Point(df_fire_cn.lon.iloc[i],df_fire_cn.lat.iloc[i])
    res = geom.contains(xy_point)
    mask_.append(res)
df_fire_cn = df_fire_cn[mask_]    
```

## <font color="blue">7. Spatial analysis  </font>

```bash
# 1. 下载
import warnings
warnings.filterwarnings('ignore')
from shapely.geometry import shape
from shapely.geometry import LineString
# loading the boundary layer
import fiona
fname = '/Users/HYF/Dropbox/data/geo/中国地图/供暖分界/N-S_boundary.shp'
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
        
## 出现错误:  'Polygon' object is not iterable      
shapeFeature([c.geometry])        
```

<font color="blue">8. Other functions  </font>

```python
# 1. 剪裁pdf
# https://pypi.org/project/pdfcropper/#files
pdf-crop-margins -v -s -u /Users/HYF/Dropbox/thesis/ipynb/5_全国时空分析/output/Sec3/As_空间差异图.pdf    

9. 如何解决自行安装文件无法导入的问题，
python setup.py build
python setup.py install
将安装包和egg装入
#/Users/HYF/anaconda3/lib/python3.7/site-packages
```

