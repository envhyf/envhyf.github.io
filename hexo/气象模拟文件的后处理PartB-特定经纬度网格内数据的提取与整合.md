title: 气象模拟文件的后处理PartB-特定位置气象数据的导出与整合
date: 2018-09-04 23:32:26

menu:

​	Archives: /archives

​	Tech: /categories/Tech

tags: 

- Linux
- Python

------



此处以WRF输出文件为例，介绍的基本流程和

<!--more-->

## <font color="blue"> 1. 常用的预处理函数库</font>

现有的，因而我们可以从，计算获得相对湿度。类似的，我们可以将'U10', 'V10'变量转换为分析中常用的风速(ws)和风向(wd, $0-360^o$)。我们可以将这些常用的函数写成一个.py文件，在每次处理数据提取程序时调用。

```python
import numpy as np
# calculate the wind speed
def calc_ws(fc,y,x):
    u = fc.variables["U10"][:,y,x]
    v = fc.variables["V10"][:,y,x]
    return  (u**2+v**2)**0.5
# calculate the wind direction   
def calc_wd(fc,y,x):
    """
    referencing:http://colaweb.gmu.edu/dev/clim301/lectures/wind/wind-uv
    """
    u = fc.variables["U10"][:,y,x]
    v = fc.variables["V10"][:,y,x]
    if u==0:
        if v==0:
            wd = np.nan
        if v>0:
            wd = 180.0
        if v<0:
            wd = 0
    if u>0:
        if v>=0:
            wd = 270-np.arctan(v/u)/2/np.pi*360.0
        if v<0:
            wd = 270+np.arctan(v/u)/2/np.pi*360.0
    if u<0:
        if v>=0:
            wd = 90+np.arctan(v/u)/2/np.pi*360.0
        if v<0:
            wd = 90-np.arctan(v/u)/2/np.pi*360.0
    return wd
# calculate the relative humidity
def calc_rh(fc,y,x):
    """
    referencing:https://github.com/keltonhalbert/wrftools/blob/master/wrftools/variables/thermo.py
    t:  perturbation potential temperature. "T"
    p:  perturbation pressure. "P"
    pb: base state pressure. "PB"
    qv: Water vapor mixing ratio "QVAPOR"
    noted that all these parameters from WRF have the vertical dimension.
    """
    t = fc.variables['T'][:,0,y,x]
    p0 = fc.variables['P'][:,0,y,x]
    pb = fc.variables['PB'][:,0,y,x]
    qv = fc.variables['QVAPOR'][:,0,y,x]
    theta = t+300
    p = (p0+pb)/100.0
    temp = theta*(p/1000.0)**0.2854
    e_0 = 6.1173 ## mb
    t_0 = 273.16 ## K
    Rv = 461.50 ## J K-1 Kg-1
    Lv_0 = 2.501 * 10**6 ## J Kg-1
    K1 = Lv_0 / Rv ## K 
    K2 = 1 / t_0 ## K-1
    K3 = 1 / temp ## K-1     
    e_s = e_0 * np.exp( K1 * ( K2 - K3 ) )
    w_s = ( 0.622 * e_s ) / (p - e_s )
    return ( qv / w_s ) * 100
```

## <font color="blue"> 2. 读取并整合数据</font> 

wrf输出结果通常有大量文件，

```python
from netCDF4 import Dataset
import pandas as pd
from pandas import HDFStore, DataFrame
from pandas import read_hdf
import os,sys,string
import numpy as np

#准备工序
df1= pd.read_csv("./站点位置及质量浓度.csv")
site_ = [t for t in df1.label]
loc_dic = {site_[0]:[37,65],site_[1]:[38,64],
           site_[2]:[37,66],site_[3]:[28,75],
           site_[4]:[34,63],site_[5]:[38,68]}

#主程序
hdf = HDFStore("simulated_meteo_chifeng.h5")
feature = ['WS','WD','PBLH','RH','T2']
files = os.listdir("/disk2/hyf/wrf/WRFV3/Chifeng_output/201601/")
Dir = "/disk2/hyf/wrf/WRFV3/Chifeng_output/201601/"
files.sort()
for k in range(0,2,1):#len(feature)
    data = {'date':[],site_[0]:[],site_[1]:[],site_[2]:[],
                      site_[3]:[],site_[4]:[],site_[5]:[],}
    for file in files:
        filename,extname = os.path.splitext(file)
        if filename[7:10] =='d03':
            fc = Dataset(Dir+file)
            data['date'].extend([''.join(t) for t in fc.variables['Times']])
            for i in range(0,len(site_),1):
                x,y = loc_dic[site_[i]][0],loc_dic[site_[i]][1]
                if feature[k]=='WS':
                    data[site_[i]].extend(calc_ws(fc,y,x))
                if feature[k]=='WD':
                    data[site_[i]].extend(calc_wd(fc,y,x))                    
    print feature[k]
    data  = pd.DataFrame(data)
    print len(data)
    #去掉重复的时间帧
    data = data.drop_duplicates(subset='date', keep='first', inplace=False)
    print len(data)
    hdf.put(feature[k], data, format='table', encoding="utf-8")
```

## <font color="blue"> 3. 时间分辨率转换</font> 

wrf输出结果通常有大量文件，

此处较为关键的是