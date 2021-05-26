title: '[WRF-Chem学习笔记①]排放预处理模块的安装与使用'
date: 2016-09-04 16:26:26

categories:

- Research 

tags: 

- Linux
- Model

---

气象场和化学物质排放是大气化学模式的两项最重要的前驱资料。在实验之余，我将着力学习WRF-Chem的有关内容，并记录、发布。

按照[WRF-Chem Tutorial](http://ruc.noaa.gov/wrf/WG11/Tutorial.html)中的说明，在[Learning to Run WRF-Chem](http://ruc.noaa.gov/wrf/WG11/tutorialexercises.htm)之前，我先学习[How to Generate Emission for WRF-Chem](http://ruc.noaa.gov/wrf/WG11/tutorialemissions.htm)

本文记录Exercise 1的实现过程，可供有兴趣的同学参考。
<!--more-->

运行WRF和WRF-Chem的一个重要区别: 排放源清单的输入。__Exercise 1__将会介绍建立WRF-Chem可读取的源清单文件的最基本方法(转置现有全球清单)，其核心工具为__PREP_CHEM_SOURCES__。


## <font color="blue"> 安装过程</font>   

### <font color="black"> HDF5安装</font> 

该程序的顺利运行需要依赖HDF5, NetCDF, zlib等标准库。之前在安装WRF-Chem时已经安装了NetCDF和zlib，这里只列出HDF5和szip的安装步骤。



```bash
#1. szip安装
wget http://www.hdfgroup.org/ftp/lib-external/szip/2.1/src/szip-2.1.tar.gz
tar -xvzf szip-2.1.tar.gz
cd szip-2.1
./configure --prefix=/The_path/you/want/to/install/
make & make check & make install
#2. HDF5安装
wget http://www.hdfgroup.org/ftp/HDF5/current/bin/linux-centos7-x86_64-gcc485/hdf5-1.8.17-linux-centos7-x86_64-gcc485-shared.tar.gz
#解压并进入文件夹
./configure --prefix=/disk2/hyf/lib/hdf5 --enable-fortran \
 --with-szlib=/szip/install/path\
 --with-zlib=/zlib/install/path  ## 此处注意与后文prep_chem_source的编译器应对应上，否则lib中的文件无法编译成功
make & make check & make install
```

### <font color="black"> 排放预处理程序安装</font>  

下载并解压后，进入`/bin/build`文件夹，修改与编译器对应的配置文件，如我将采用gfortran编译器编译，则`vim include.mk.gfortran`, 修改其中的NETCDF与HDF5库对应位置，按以下命令进行编译:

```bash
`make OPT=gfortran CHEM=RADM_WRF_FIM`
```

其中`OPT=`对应编译器类别(i.e, intel, pgi,gfortran, ...)，`CHEM=`对应特定化学机理，此处选择__RADM__机理。这将决定WRF-Chem读取源清单文件的化学物质分类格式。  

值得注意的是，按上述操作，将在编译__edgar_emission.f90__过程出错，WRF论坛中也有用户提交了同样的报错，幸有大神回复并解决，解决方法是调整某些行的字符串的长度，可参考[该网页](http://forum.wrfforum.com/viewtopic.php?f=39&t=9199)进行修改。

修改完成后，再次编译，若出现`Finished building === ../prep_chem_sources_RADM_WRF_FIM_.exe`，说明编译成功!    
ヽ(✿ﾟ▽ﾟ)ノ

PS: 安装完成后，编辑~/.bashrc   

```bash
# 添加以下几行
export HDF5="$DIR/hdf5" # $DIR为所有库存放位置
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HDF5/lib
export NETCDF="$DIR/netcdf"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NETCDF/lib
```


## <font color="blue"> 案例</font>   


### <font color="black"> 排放预处理程序安装</font>  

按照Exercise 1中的input文件，替换`prep_chem_src\bin`文件夹下原有的input文件，并按照自己的global_emission文件夹修改读取原始排放数据的路径。   

运行`$./prep_chem_sources_RADM_WRF_FIM_.exe`           

运行成功后，会出现五个输出文件，其中有三个后缀名为`.bin`的文件，分别为:
      
```bash
  WRF-TUTORIAL-T-2010-07-14-000000-g1-ab.bin ##  人为源排放清单
  WRF-TUTORIAL-T-2010-07-14-000000-g1-bb.bin ##  生物质燃烧清单
  WRF-TUTORIAL-T-2010-07-14-000000-g1-gocartBG.bin ## GOCART的全球背景化学场资料
```

将`prep_chem_src\bin`文件夹中三个`.bin`后缀的输出文件移动到`WRF-Chem`文件夹中, 此处我在`WRFV3\test\`文件夹内新建`em_ex1`文件夹，内容复制同目录下的`em_real`。 

运行`./convert_emiss.exe`

按照论坛上大家的观点，目前WRF-Chem 3.4版本以上会报错，[参考1](http://forum.wrfforum.com/viewtopic.php?f=41&t=8921)[参考2](http://forum.wrfforum.com/viewtopic.php?f=41&t=5863)。 

若运行成功，则会生成三个排放源文件，其中

(1) __wrfchemi_d01__文件对应设计模拟区域内各类污染物的排放清单结果,包括CO, NH3, PM10, PM2.5及各类有机物(按照RACM机理分类)。   
(2) __wrffirechemi_d01__文件对应生物质燃烧排放情况.     
(3)__wrfchemi_gocart_bg_d01__文件对应二甲基硫(DMS), NO3, OH, H2O2的背景浓度分布。DMS单位为nM/L,其他物种单位为体积混合比(volume mixing ratio)。 


依据上述的步骤      
>  `原始全球清单 → 模拟区域排放二进制文件 → WRF-Chem可读取的nc格式排放源文件`
完成了排放源资料的制作

### <font color="black"> 初始场与边界场文件的生成</font>   

此处链接同模拟区域、时间的`met_em.xxx`文件(由WPS模型完成)[链接](ftp://aftp.fsl.noaa.gov/divisions/taq/wrfchemv35_tutorial/met_em_v3.5.tar.gz)，运行`./real.exe`, 生成初始场和边界场文件,内部包括了化学物质和气象资料信息。

### <font color="black"> 输出文件生成</font>   

运行`./wrf.exe`，生成wrfout文件。下图为PM2.5的分布情形，可以看出如阿尔卑斯山、安纳陀利亚高原等高海拔地区浓度相对低。

[![enter image description here][1]][1]








附图: 参考[曼彻斯特大学教程](http://wiki.rac.manchester.ac.uk/community/WRF/)，绘制WRF-Chem运行流程如下:  

此处需要运行`./real.exe`两次，目前我还没有弄明白。

[![enter image description here][2]][2]



[1]: http://i.stack.imgur.com/ZAZNE.png
[2]: http://i.stack.imgur.com/GbD3C.png

  

参考资料 

[convert_emiss.exe](https://sites.google.com/a/utexas.edu/qinjian-jin/codes/climate-models)  