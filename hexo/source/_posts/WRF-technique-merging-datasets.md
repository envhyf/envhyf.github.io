title: 'WRF post processing 1: Merging datafiles for specific spots'
date: 2019-03-11 00:29:26

categories:

- Research

tags:

- Python
- WRF

---
When you've got the all the files from  WRF simulations, you might want to merge them without the spin-up frames (to reach a balanced state with the boundary conditions, i.e., 12 hours for a 5-day simulation). Meanwhile, the variables/grids which are not focused on can be ignore. Therefore, a general workflow in pythonic way is presented. I will also rewritten this function as my first Python Package. Please note the updates on my website.
<!--more-->

 # 1. Pretreatment
 In this post, the technique applied here is to extracting five meteorological variables for several grids. Those variables contain _wind speed (ws), wind direction (wd), relative humidity (RH), planetal boundary layer height (PBLH), Temperature at 2 m above ground (T2) and precipitation (PREP)_. Noticed that _ws, wd_ and _rh_ are not directly derived from standard output, some essential steps should be implemented.

```python
"""
calulating wind speed by U and V components
"""
def calc_ws(fc,y,x):
    u = fc.variables["U10"][:,y,x]
    v = fc.variables["V10"][:,y,x]
    return  (u**2+v**2)**0.5

"""
calulating wind directionn by U and V components
"""    
def calc_wd(fc,y,x): # fc: input data, y/x: the index in y/x axis of targeted grid
    """
    Reference: http://colaweb.gmu.edu/dev/clim301/lectures/wind/wind-uv
    """
    u = fc.variables["U10"][:,y,x]
    v = fc.variables["V10"][:,y,x]
    wd = []
    for i in range(0,len(u),1):
        if u[i]==0:
            if v[i]==0:
                wd.append(np.nan)
            if v[i]>0:
                wd.append(180.0)
            if v[i]<0:
                wd.append(0)
        if u[i]>0:
            if v[i]>=0:
                wd.append(270-np.arctan(v[i]/u[i])/2/np.pi*360.0)
            if v[i]<0:
                wd.append(270+np.arctan(v[i]/u[i])/2/np.pi*360.0)
        if u[i]<0:
            if v[i]>=0:
                wd.append(90+np.arctan(v[i]/u[i])/2/np.pi*360.0)
            if v[i]<0:
                wd.append(90-np.arctan(v[i]/u[i])/2/np.pi*360.0)
    return wd

# calculate the relative humidity
def calc_rh(fc,y,x):
    """
    referencing:
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
    e_s = e_0 * np.exp(K1*(K2 - K3))
    w_s = (0.622 * e_s)/(p - e_s)
    return (qv/w_s)*100.0
```
Those functions were wrote in an independent file (function_lib.py) for further importing.

# 2. Main function
Now that we have the pre-defined tools, we can proceed to read and merge all the outputs into one __hdf5__ file.

```python
from netCDF4 import Dataset
import pandas as pd
from pandas import HDFStore, DataFrame
from pandas import read_hdf
import os,sys,string
import numpy as np
# the Pretreatment functions
import function_lib as func

# targeted girds with specific names
site_= ['YZ','TY','HD','SS','XS','HS']
loc_dic = {site_[0]:[37,65],site_[1]:[38,64],
           site_[2]:[37,66],site_[3]:[28,75],
           site_[4]:[34,63],site_[5]:[38,68]}

# creat an empty h5 file
hdf = HDFStore("simulated_meteo_chifeng.h5")
feature = ['WS','WD','PBLH','RH','T2','PREP']

# loop all the original files
Dir = "/disk2/hyf/wrf/WRFV3/Chifeng_output/post-process/d03/"
files = os.listdir(Dir)
files.sort()
files = files[0:]

# write the varibales into h5 file
for k in range(0,len(feature),1):
    data = {'date':[],site_[0]:[],site_[1]:[],site_[2]:[],
                      site_[3]:[],site_[4]:[],site_[5]:[],}
    for file in files:
        filename,extname = os.path.splitext(file)
        if filename[7:10] =='d03': ## restrict to specific nested domain
            fc = Dataset(Dir+file)
            data['date'].extend([''.join(t) for t in fc.variables['Times']])
            for i in range(0,len(site_),1):
                x,y = loc_dic[site_[i]][1],loc_dic[site_[i]][0]
                if feature[k]=='WD':
                    data[site_[i]].extend(func.calc_wd(fc,y,x))
                if feature[k]=='WS':
                    data[site_[i]].extend(func.calc_ws(fc,y,x))
                if feature[k]=='PBLH':
                    data[site_[i]].extend(fc.variables['PBLH'][:,y,x])
                if feature[k]=='RH':
                    data[site_[i]].extend(func.calc_rh(fc,y,x))
                if feature[k]=='T2':
                    data[site_[i]].extend(fc.variables['T2'][:,y,x])
                if feature[k]=='PREP':
                    #RAINC is the rain calculation by cumulus scheme and RAINNC is the rain calculation by micrphysics scheme.                  
                    rain = (fc.variables['RAINC'][:,y,x]+fc.variables['RAINNC'][:,y,x])*1000.0/997.0
                    data[site_[i]].extend(rain)
    data  = pd.DataFrame(data)
    # a efficient way to deal with those spin-up frames.
    data = data.drop_duplicates(subset='date', keep='first', inplace=False)
    print len(data)
    data = data.reset_index()
    hdf.put(feature[k], data, format='table', encoding="utf-8")
```

And after those scripts completed successfully, we have merged all the targetd variables for the spots of interest. In our furture post, the technique dealing with a 2-d griided system (i.e., a city) will be presented.
