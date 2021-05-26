title: 气象模拟文件的后处理PartA-曲折的安装
date: 2018-09-04 23:32:26

categories:

- Research

tags: 

- Linux
- Python

---

以前处理WRF等气象模型的输出文件，总是下载到本地电脑做后处理分析。由于计算量不断增加，模拟生成的文件往往会很大。因而，我考虑直接在服务器中处理数据。本来是很容易的事情，却因为课题组服务器的系统版本较旧，在安装有关工具时耗去了不少时间。在此记录我的探索过程。

气象模拟文件的后处理PartA: 曲折的安装

<!--more-->

服务器的基本信息

```bash
$cat /etc/redhat-release
CentOS release 5.9 (Final)
$gcc --version
gcc (GCC) 4.9.4
$ncdump --version
netcdf library version 4.6.1
```



## <font color="blue"> 1. gdal库安装</font>

gdal是一个常用的地学数据库，可用于读取、编写netCDF等格式的数值文件。尝试利用conda工具安装：

```bash
$conda install -c conda-forge gdal
```

安装完毕后，无法启动python，错误信息如下：

```bash
python: /lib64/libc.so.6: version `GLIBC_2.7' not found (required by /home/hyf/anaconda2/bin/../lib/libpython2.7.so.1.0) 
```

上文已述，服务器系统为Cent OS 5.9版本，并不支持glibc_2.7。可用以下指令进行检查：

```bash
$strings /lib64/libc.so.6 |grep GLIBC_ 
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_PRIVATE
```

同时，gdal安装过程conda环境也升级至Anaconda5，不再适配于CentOS 5.X/glibc_2.5的系统环境，导致Python环境崩坏，无法启动。因而，我只得重新安装Anaconda，并设置安装package阶段不升级conda环境至5.x版本，以避免该错误的再次出现。

```bash
# 已安装的库文件名称存一下
printf '%s\n' ~/anaconda2/pkgs/* | paste -sd "," - > log.csv
# 卸载ananconda
rm -rf ~/anaconda2
rm -rf ~/.condarc ~/.conda ~/.continuum
#重新安装Anaconda，在其下载文件夹内进行
./Anaconda2-4.0.0-Linux-x86_64.sh 
```

参考[conda github issue#6041](https://github.com/conda/conda/issues/6041)，按如下指令对conda环境进行限制，防止其升级

```bash
$conda config --set auto_update_conda false
# 新的错误再次出现
Error: Error key must be one of always_yes, root_dir, channel_alias, add_anaconda_token, add_pip_as_python_dependency, show_channel_urls, update_dependencies, changeps1, allow_softlinks, anaconda_upload, binstar_upload, use_pip, offline, allow_other_channels, add_binstar_token, ssl_verify, always_copy, not auto_update_conda 
```

查询网络资料得知，在conda 4.2版本后，该bug得以修复。因而`conda install conda =4.2`指令后再强制其不升级，不再出现上述错误。

重新安装gdal库文件，python可正常启用，但在调用gdal库函数时，又出现了新的错误（༼ ͠ຈ Ĺ̯ ͠ຈ ༽）：

```python
from osgeo import gdal
>ImportError: /home/hyf/anaconda2/lib/python2.7/site-packages/osgeo/../../../libgdal.so.20: ELF file OS ABI invalid 
```

参考[这篇文章的解释](https://www.lfd.uci.edu/~gohlke/pythonlibs/#gdal)，该错误上文“glibc_2.7”报错是类似的。系统的ldd version同样陈旧(2.5)，无法支持gdal库正常使用。将Cent OS升级成6.x版本可一劳永逸地解决这类问题，但我并无权限，也无决心重新配置整个系统。

在反复安装测试未果后，我决定放弃╮(╯_╰)╭，转向安装其他支持读取netcdf格式的python库。在此期间，我还尝试在服务器上安装了R语言环境和ncdf package，以后有时间专文记述。

## <font color="blue"> 2. netCDF4-python库安装</font>

首先尝试[Anaconda网站](https://anaconda.org/anaconda/netcdf4)提供的方法，直接利用conda工具安装

```bash
$conda install -c anaconda netcdf4
```

安装后，出现python无法启动的错误，具体为“importError: cannot import name md5”。[Stackoverflow](https://stackoverflow.com/questions/46605138/anaconda2-cannot-import-name-md5)上类似题的回答表明，该错误是在anaocnda5.0.1版本中得以修复。但是，我不能安5.x版本的conda啊இдஇ，于是再想其他办法。

重新安装anaconda环境后，我尝试用手动install from source的方法来安装，具体[netcdf4-python的github主页](https://github.com/Unidata/netcdf4-python)中的介绍：

```bash
$git clone https://github.com/Unidata/netcdf4-python.git
cd netcdf4-python
# 修改 setup.cfg中的netcdf, hdf5存放路径
sed -i '/\[directories\]/a \
HDF5_dir = $DIR/hdf5 \
netCDF4_dir = $DIR/netcdf' setup.cfg
# run setup.py
$python setup.py build
$python setup.py install


# run test.py
$cd test
$python run_all.py
```

不出意外，安装过程肯定是要报错的，具体的错误信息如下：

```bash
/usr/bin/ld: /disk2/hyf/lib/netcdf/lib/libnetcdf.a(libdispatch_la-att.o): relocation R_X86_64_32 against `a local symbol' can not be used when making a shared object; recompile with -fPIC
/disk2/hyf/lib/netcdf/lib/libnetcdf.a: could not read symbols: Bad value
collect2: error: ld returned 1 exit status
error: command 'gcc' failed with exit status 1
```

该结果表明，netcdf库在安装过程，关闭了enable-shared，共享库未进行编译，并进一步导致了`"a local symbol" can not be used …`的问题。正好在flexpart安装过程(参见之前的博文[FLEXPART installation notes](http://paisheng.me/2018/08/10/FLEXPART_INSTALLATION_NOTE/))中，我又安装了一套netCDF库，在该环境下重新手动安装，该错误不再显示。

```bash
# change netCDF
$source ~/.bashrc_forFlex
$ncdump --version
netcdf library version 4.6.1

# run setup.py
$python setup.py build
$python setup.py install

# run test.py
$cd test
$python run_all.py
```

成功安装后，在/netcdf4-python/test/文件夹下可运行，并调用netCDF4库函数(`from netCDF4 import Dataset`)。测试结果表明，写netCDF格式文件(mode='w')一切正常，但有两个致命问题：1. 仅能在该文件夹内，方可成功`import netCDF4 `，2. 无法读取现有的nc数据，报错如下：

```python
filepath method not enabled.  To enable, install Cython, make sure you have version 4.1.2 or higher of the netcdf C lib, and rebuild netcdf4-python.
```

Cython我已经安装了，netcdf C 版本也高于4.1.2，到底是哪里出问题了呢？参考[github上netcdf-python issue#263](https://github.com/Unidata/netcdf4-python/issues/263)，我检查了`python setup.py build`时的output，其明确显示

```bash
netcdf lib does not have group rename capability
netcdf lib does not have nc_inq_path function
netcdf lib does not have nc_inq_format_extended function
netcdf lib does not have nc_open_mem function
netcdf lib does not have cdf-5 format capability
netcdf lib does not have netcdf4 parallel functions
```

初步断定"nc_inq_path function" 的缺失，是导致"filepath method not enabled"的根本原因。反复搜索相关资料，也未查询到相似的问题以及解决方法。

完全放弃前，我尝试`conda install netcdf4 `，竟然意外成功，可以正常读取nc格式文件了。

✧*｡٩(ˊᗜˋ*)و✧*｡

✧*｡٩(ˊᗜˋ*)و✧*｡

✧*｡٩(ˊᗜˋ*)و✧*｡

## 参考资料

[1. GCC install](http://stackoverflow.com/questions/9450394/how-to-install-gcc-piece-by-piece-with-gmp-mpfr-mpc-elf-without-shared-libra)      

[2. NetCDF in R](http://geog.uoregon.edu/bartlein/courses/geog607/Rmd/netCDF_01.htm)

[3. Installing NetCDF and R ‘ncdf’](http://mazamascience.com/WorkingWithData/?p=1429)

[4. Aqua-Duct installation guide](https://pythonhosted.org/aquaduct/aquaduct_install.html#netcdf4-mdanalysis-installation-ubuntu-14-04)