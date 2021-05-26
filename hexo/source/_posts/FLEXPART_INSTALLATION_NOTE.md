title: FLEXPART installation notes
date: 2018-08-10 23:32:26

categories:

- Research

tags: 

- Linux

- Model
- Air quality

---

FLEXPART is a Lagrangian particle dispersion model (LPDM) developed by Norwegian Institute of Air Research, Norway. It allows researchers to simulate the long-range transportation, diffusion, dry/wet deposition processes of atmospheric spcecies from their sources.  It also can be utilized for backward calculation based on the observation of receptor to anaysis source-receptor relationships.

This model is coded following the Fortran 95 standard, and can be freely download from the page [here](https://www.flexpart.eu/). Flexpart 8.x/9.x is easy for compilation following the offical [reference](https://www.flexpart.eu/wiki/FpInstall).  I noticed that netCDF-format output (which would make the post-processing easier compared to the original binary output files) has been merged in the newer veision. Therefore, I tried to compile FLEXPART 10.0 beta version in the Linux system, while lots of issues appeared. My installing steps are listed as follows

<!--more-->



![Fig1. Application example: Age composition of the plume](https://i.stack.imgur.com/hWIVz.png)

*<font color='blue'>Fig.1 Application example: CO age composition of the plume arrived at the receptor [source](http://www.cee.mtu.edu/~bzhang3/page1.html)*



## <font color="blue"> 1. Installing required packages</font> 

Compile flag "-fuse-linker-plugin" option is enabled in the makefile of flexaprt. The error `gfortran: error: -fuse-linker-plugin is not supported in this configuration` indicated that this option may not work  in the current system.  The website [Explain Linux Commands](https://www.cleancss.com/explain-command/arm-linux-gnueabi-gcc/67170) shows that:  

> Enables the use of a linker plugin during link-time optimization.  This option relies on the linker plugin support in linker that is available in gold or in GNU ld 2.21 or newer.

Thus, I need to re-compile binutils to upgrade the GNU ld (my current version is 2.17 which is attched in Cent OS 5.9).  Later on, I have to re-compile all the required packages in theo order: binutils $\rightarrow$ gcc $\rightarrow$ netcdf-c/fortran $\rightarrow$ jasper $\rightarrow$ grib_api.    

(PS: since there are many softwares have been compiled in the current environment, I had to edit a new `~/.bashrc_forFlex` as an switch to select the suitable system configurations )

```bash
#Install binutils 
mkdir /disk2/hyf/lib/build
cd /disk2/hyf/lib/build
wget https://ftp.gnu.org/gnu/binutils/binutils-2.27.tar.gz
tar -xvf binutils-2.27.tar.gz
cd binutils-2.27
./configure prefix=/home/hli2/libs/binutils
make
make install
cd ../

vi ~/.bashrc_forFLex
export DIR=/disk2/hyf/lib
export PATH=$DIR/binutils-2.25/bin:$PATH
source ~/.bashrc_forFLex

#Install gcc
#source code in https://ftp.gnu.org/gnu/gcc/
#Referenc in https://gcc.gnu.org/wiki/InstallingGCC
cd gcc-7.3.0
./contrib/download_prerequisites
cd ..
mkdir objdir
cd objdir
../gcc-6.1.0/configure --prefix=$DIR/gcc-6.1.0 --enable-languages=c,c++,fortran,go 
make
make install

vi ~/.bashrc_forFlex
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DIR/gcc-6.1.0/lib/../lib64
export PATH=/disk2/hyf/lib/gcc-6.1.0/bin:$PATH
export CC=gcc
#export CFLAGS='-gdwarf-2 -gstrict-dwarf -02 -fPIC'
export CFLAGS='-gdwarf-2 -gstrict-dwarf'
export CXX=g++
export FC=gfortran
export FCFLAGS=-m64
export F77=gfortran
export FFLAGS=-m64
source ~/.bashrc_forFlex

#Install netCDF
wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-4.6.1.tar.gz
tar -zxvf netcdf-4.6.1.tar.gz
cd netcdf-4.6.1/
wget 'http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD' -O config.guess
wget 'http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD' -O config.sub
./configure --prefix=/disk2/hyf/netcdf-4.6.1 CPPFLAGS="-I/disk2/hyf/lib/hdf5/include -I/diks2/hyf/lib/grib2/include -O3" LDFLAGS="-L/disk2/hyf/lib/hdf5/lib -L/diks2/hyf/lib/grib2/lib" --enable-shared --enable-netcdf-4  --disable-dap --disable-doxygen
make -j
make check
make install

vi~/.bashrc_forFlex
export PATH="$DIR/netcdf-4.6.1/bin:$PATH"
export NETCDF=$DIR/netcdf-4.6.1
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NETCDF/lib
export CFLAGS="-L$HDF5/lib -I$HDF5/include -L$NETCDF/lib -I$NETCDF/include"
export CXXFLAGS="-L$HDF5/lib -I$HDF5/include -L$NETCDF/lib -I$NETCDF/include"
xport FCFLAGS="-L$HDF5/lib -I$HDF5/include -L$NETCDF/lib -I$NETCDF/include"
export CPPFLAGS="-I${HDF5}/include -I${NETCDF}/include"
export LDFLAGS="-L${HDF5}/lib -L${NETCDF}/lib"
source ~/.bashrc_forFlex

#Install netCDF-Fortran
wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-fortran-4.4.4.tar.gz
tar -zxvf netcdf-fortran-4.4.4.tar.gz
./configure  --prefix=${NETCDF} --enable-shared --disable-doxygen
make -j
make check
make install

#Install jasper
./configure --prefix=$DIR/jasper
make
make install

vi~/.bashrc_forFlex
export JASPER=$DIR/jasper
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$JASPER/lib
source ~/.bashrc_forFlex

#Install grib_api
#download page https://confluence.ecmwf.int/display/GRIB/Releases
./configure --prefix=$DIR/grib_api --with-jasper=$DIR/jasper --disable-shared # --enable-sharedv
make
make install
#Noted that if disable-shared is not added into the compile flag, the error "/disk2/hyf/lib/jasper/lib/libjasper.a: error adding symbols: Bad value" would appear.  

vi ~/.bashrc_forFlex
export PATH=$GRIB_API_BIN:$PATH
export GRIB_API=$DIR/grib_api/
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$GRIB_API/lib
export GRIB_API_BIN=$GRIB_API/bin
export GRIB_API_LIB=$GRIB_API/lib
export GRIB_API_INCLUDE=$GRIB_API/include
source ~/.bashrc_forFlex
```



## <font color="blue"> 2. Installing FLEXPART</font>

```bash
#edit the makefile suitable for the own computer
#dowanload page: https://www.flexpart.eu/downloads
tar -xvzf flexpart10.01beta.tar.gz
cd ./flexpart10.01beta/flexpart

# Both ECMWF and GFS data can be used as the inputs for FLEXPART, while the specific model for them should be clarified befoe installation.   
cp -r src/ test1/
cd test1

vi Makefile
# the lines below should be revised depending on the own computer system.   
F90       = $GCC/gcc-6.1.0/bin/gfortran
INCPATH1  = /disk2/hyf/lib/grib_api/include
INCPATH2  = /disk2/hyf/lib/jasper/include
INCPATH3  = /disk2/hyf/lib/netcdf-4.6.1/include
LIBPATH1  = /disk2/hyf/lib/grib_api/lib
LIBPATH2  = /disk2/hyf/lib/jasper/lib
LIBPATH3  = /disk2/hyf/lib/netcdf-4.6.1/lib

vi par_mod.f90
# the default parameters in the par_mod.f90 was set for ECMWF windfield datasets. Therefore, some lines need to be revised. 
# Adjust the following parameters to switch between FNL/ECMWF datasets.
!integer,parameter :: nxmax=361,nymax=181,nuvzmax=92,nwzmax=92,nzmax=92 !FNL XF
!integer,parameter :: nxmax=361,nymax=181,nuvzmax=152,nwzmax=152,nzmax=152 !ECMWF new 
!integer,parameter :: nxmax=361,nymax=181,nuvzmax=92,nwzmax=92,nzmax=92 !ECMWF
!integer,parameter :: nxmax=361,nymax=181,nuvzmax=26,nwzmax=26,nzmax=26
integer,parameter :: nxmax=721,nymax=361,nuvzmax=64,nwzmax=64,nzmax=64
!integer,parameter :: nxmax=1201,nymax=235,nuvzmax=58,nwzmax=58,nzmax=58

!integer,parameter :: nxshift=359 ! for ECMWF
integer,parameter :: nxshift=0     ! for GFS or FNL

make -j 8 gfs
```



## <font color="blue"> 3. Installing FLEXPART-WRF</font>

It has been known that the results of Lagrangian particle dispersion models (LPDM) would affected by the errors in spatio-temporal interpolation of the simulated meteorological field. Furthermore, there are some uncertainties in the aspect of simulating mixing status for particles due to the erros in the vertical velocity. Therefore, the output from mesoscale models which were finer in vertical/horizontial resolution and time interval have been introduced for partially address these issue. In 2006, Flexpart-WRF model has been developed  combining the Flexpart utilities and WRF output. Compared to the standard model using global GFS/ECMWF datasets, Flexpart-WRF can better depict the atmospheric transportation especially for small-scale applications.

```bash
 tar -xvzf ./flexpart_wrf_3.3.2.tar.gz
 cd flexpart_wrf
 cp -r src test
 cd test
 
 vi makefile.mom # set the ${NETCDF} environment variable
 make -f makefile.mom omp

 #Done!
```



## <font color="blue"> 4. Some tips</font>

```bash
# Error1 NetCDF: Invalid dimension ID or name when using forward mode of FLEXPART-WRF
# REFERENCE: https://www.flexpart.eu/ticket/233
# There is a typo error in the original netcdf_out_mod.F90!

Change Line 624
- ncret = NF90_DEF_VAR(ncid,'DRYDEP',NF90_FLOAT,ncdimsid2,ncddvid,&
+ ncret = NF90_DEF_VAR(ncid,'DRYDEP',NF90_FLOAT,ncdimsid22,ncddvid,&


```



#



## 参考资料    

[1. FlXEXPART Offical website](https://www.flexpart.eu/)  

[2. Problem compiling GRIB AP](https://jira.ecmwf.int/browse/SUP-373)

[3. Compile with flag -fPIC](https://stackoverflow.com/questions/629961/how-can-i-set-ccshared-fpic-while-executing-configure)

[4. FLEXPART various compilation problems](https://www.flexpart.eu/ticket/136)

