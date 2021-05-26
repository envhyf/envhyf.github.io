title: 'WRF post processing 2: Xarray tricks'
date: 2020-11-01 00:29:26

categories:

- Research

tags:

- Python
- WRF

---
Xarray provides pandas-level convenience for working with ultidimensional data. Xarray has two fundamental data structure: a __DataArray__, which holds a single multi-dimensional variable and its coordinates; 
a __Dataset__, which holds multiple variables that potentially share the same coordinates.     

Moreover, a __DataArray__ has four attributes: 
* values: a numpy.ndarray holding the array’s values (矩阵数值，例如地表温度具体数值)
* dims: dimension names for each axis (e.g., ('x', 'y', 'z')) (维度名称，如经度、纬度、垂直分层、时间等)
* coords: a dict-like container of arrays (coordinates) that label each * point (e.g., 1-dimensional arrays of numbers, datetime objects or strings) (各维度坐标体系，如时间序列)
* attrs: an OrderedDict to hold arbitrary metadata (attributes) (对各属性的描述)
<!--more-->  

 # 1. Subset and Diurnal profiles
 As an illustration, we firstly extract the original file generated from WRF-Chem by specific time range and spatial coverage. Then, the daily averages for those selected grids are calculated and saved as another netCDF file. 

```python
import xarray as xr
filename = "./../wrfout_allBVOCs_d01_2018-06-01_00:00:00"
df = xr.open_dataset(filename)

# 1. add dims
df = df.assign_coords(south_north=('south_north',df.XLAT[0,:,0].values))
df = df.assign_coords(west_east=('west_east',df.XLONG[0,0,:].values))
df = df.assign_coords(Time=('Time',df.XTIME.values))

# 2. selcting the grids with lon in the range of 100-120，lat in the range of 40-50 in the bottom vertical level. The simulation period was chosen in the range of 2018-06-01 to 2018-06-05
da = df.sel(bottom_top = slice(0,1), south_north = slice(40,50),west_east = slice(100,120), Time = slice('2018-06-01T00:00:00.000000000',\
                                                                                                        '2018-06-08T00:00:00.000000000'))
# 3. selecting one/multi variable(s)
var      = 'PM2_5_DRY' 
var_list = ['asoa1j','asoa1i','asoa2j','asoa2i']
# 3.1 Summarizing multiple variables
da['SUM']  = 0
for c in var_list:
    da['SUM'] = da['SUM']+da[c]
    
# 4. Calculating the diurnal averages
dm  = da["SUM"].groupby('Time.hour').mean()
dm.to_netcdf("diurnal.nc")

```

# 2. Rolling window
In the second example, we use resample trick to group the original data, and use rolling trick to calculate the daily maximum of 8-hour ozone concentrations.

```python
da['8h_o3_averege'] = da['o3'].rolling(Time=8, center=False).mean() # center = False->time=end point of the rolling window.
dt = da['8h_o3_averege'].resample(Time ='1D').max()
dt.to_netcdf("./8h_o3_max.nc")
```

