[Python for post-processing netCDF file]: package installation 

## 尝试gdal库

尝试在Linux服务器上安装gdal库，用于地学数据分析。

```bash
conda install -c conda-forge gdal
```

安装完毕后，无法启动python，错误信息如下：

```bash
python: /lib64/libc.so.6: version `GLIBC_2.7' not found (required by /home/hyf/anaconda2/bin/../lib/libpython2.7.so.1.0) 
```

我所用的Linux系统是CentOS 5版本，最高支持至glibc_2.5版本。可用如下指令查询

```bash
strings /lib64/libc.so.6 |grep GLIBC_ 
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

而gdal安装过程中conda环境也升级至Anaconda5，不再支持CentOS 5/glibc_2.5系统。因而，我只得重新安装conda环境，并强制其不在安装库文件环节升级，避免再次出现该错误。

具体操作如下：

```bash
# 已安装的库文件名称存一下
printf '%s\n' ~/anaconda2/pkgs/* | paste -sd "," - > log.csv
# 卸载ananconda
rm -rf ~/anaconda2
rm -rf ~/.condarc ~/.conda ~/.continuum
#重新安装Anaconda
Anaconda2-4.0.0-Linux-x86_64.sh 
```

参考[conda github issue#6041](https://github.com/conda/conda/issues/6041),用如下指令对conda环境进行限制，防止其升级

```bash
conda config --set auto_update_conda false
```

出现新的错误信息，

```bash
Error: Error key must be one of always_yes, root_dir, channel_alias, add_anaconda_token, add_pip_as_python_dependency, show_channel_urls, update_dependencies, changeps1, allow_softlinks, anaconda_upload, binstar_upload, use_pip, offline, allow_other_channels, add_binstar_token, ssl_verify, always_copy, not auto_update_conda 
```



该错误在conda 4.2版本后得以修复。因此 `conda install conda=4.2` 升级后该问题解决。再次安装gdal，未出现上述错误，但是在python环境下`from osgeo import gdal`报错，错误信息如下:    

> ImportError: /home/hyf/anaconda2/lib/python2.7/site-packages/osgeo/../../../libgdal.so.20: ELF file OS ABI invalid 

问题在于ldd version较为陈旧

https://www.lfd.uci.edu/~gohlke/pythonlibs/#gdal

多次尝试均告负，决定转向安装netCDF4库

## NETCDF4库文件安装

```bash
## 安装指令
conda install -c conda-forge netcdf4=1.2.7 
python
> import netCDF4
/lib64/libc.so.6: version `GLIBC_2.7' not found (required by /home/hyf/anaconda2/lib/python2.7/site-packages/netCDF4/../../../libnetcdf.so.11) 
```

还是出现glibc_2.7的问题。考虑手动安装，

```bash
 python setup.py install
 # 报错为安装netCDF4时并未选择 -fPIC选项
 /usr/bin/ld: /disk2/hyf/lib/netcdf/lib/libnetcdf.a(libdispatch_la-att.o): relocation R_X86_64_32 against `a local symbol' can not be used when making a shared object; recompile with -fPIC
 
 # 重新编译，没办法了
 # 选择--enable-shared版本重新安装netcdf4库，可以成功调用
 # 但是，又有新问题出现，如下
 filepath method not enabled.  To enable, install Cython, make sure you have version 4.1.2 or higher of the netcdf C lib, and rebuild netcdf4-python.
 
 我现在所安装的是netcdf-4.6.1版本，不存在
 
```

不禁思考，在centos 5.x环境下，就不能使用Python语言读取netcdf4文件吗？我不相信，接着试验。







