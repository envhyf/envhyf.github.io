title: '[WRF-Chem学习笔记②]仅含沙尘源的模拟练习'
date: 2016-09-05 16:26:26

categories:

- Research

tags: 

- Linux
- Model

---

本文是我学习官网练习的记录，其模拟区域为北非，中东以及欧洲部分地区，无嵌套设置。共分为5个练习，分别为: 

> 1. 初始场添加沙尘排放源的模拟
> 2. 采用GOCART全球气溶胶排放清单的模拟
> 3. 加入生物源排放的化学模拟(MEGAN引入)
> 4. 加入气溶胶直接/间接辐射效应的模拟
> 5. WRF-Chem数值预报实验  

下文记录Exercise 1的流程和注意事项
<!--more-->

### <font color="grey"> WPS运行</font>    

```
-------
cd /wrf/WPS/
rm geogrid.log ungrib.log metgrid.log
1. 地面静态资料数据生成
cd geogrid
ln -sf GEOGRID.TBL.ARW_CHEM GEOGRID.TBL
cd ..
./geoerid.exe
-------
2. 气象资料读取
./link_grib.csh /disk2/data/ncep/fnl_2015071*
./ungrib.exe >& ungrib.log
-------
3. 数据整合
mpirun -np 8  metgrid.exe
-------
```

注意此处geogrid.exe输出结果会包含沙尘信息:
    
      Processing EROD  
      Processing CLAYFRAC
      Processing SANDFRAC  

### <font color="green"> WRF运行</font>     

在`WRFV3/test`文件夹下复制`em_real`文件夹，新建`tutor`文件夹。此处主要进行WRF运行过程中边界场和初始场文件的生成，__chem_opt__(化学选项) = 401， 表示仅考虑沙尘。

```
链接WPS的输出结果
ln -svf /disk2/hyf
mpirun -np 16 ./real.exe
mpirun -np 16 ./wrf.exe  
```
成功运行，会在`rsl.out.xxxx`出现 __wrf: SUCCESS COMPLETE WRF__字段。

### <font color="orange"> 绘制输出结果</font>    

利用Python对输出文件进行读取和绘制图像，如下图中的温度:  

[![enter image description here][1]][1]


[1]: http://i.stack.imgur.com/8ozjq.png


参考资料 

[Exercise 1](http://ruc.noaa.gov/wrf/WG11/wrf_tutorial_exercises_v35/exercise_1.html)  