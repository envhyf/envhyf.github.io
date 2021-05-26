title: FLEXPART-WRF backward example
date: 2018-08-10 23:32:26
tags: 

- Linux
- Model
- Air quality

------

This article introduces the implemetion of FLEXPART-WRF in the backward mode. After the complete installation, an executable program named as "FP_GFPRTRAN_GFS"(can be user-defined by editting the makefile) is produced. Running the model is simple, which can be divided as three steps.

<!--more-->



## <font color="blue"> 1. Setting the variables</font>

The file __flexwrf.input__ is almost the only file for editting before the modeling.

```bash
=====================FORMER PATHNAMES FILE===================
/disk2/hyf/flexpart/flex_wrf/output_back1/ # the path of output files 
/disk2/hyf/flexpart/flex_wrf/met/          # the path of meteorological datasets
/disk2/hyf/flexpart/flex_wrf/met/AVAILABLE.grid2 # the dictionary connecting meteo files with simulated times
=============================================================
```



```bash
=====================FORMER COMMAND FILE=====================
    -1               LDIRECT:          1 for forward simulation, -1 for backward simulation
    20100518 000000  YYYYMMDD HHMISS   beginning date of simulation
    20100518 110000  YYYYMMDD HHMISS   ending date of simulation
    3600             SSSSS  (int)      output every SSSSS seconds
    3600             SSSSS  (int)      time average of output (in SSSSS seconds)
    180              SSSSS  (int)      sampling rate of output (in SSSSS seconds)
    999999999        SSSSS  (int)      time constant for particle splitting (in seconds)
    180              SSSSS  (int)      synchronisation interval of flexpart (in seconds)
    10.              CTL    (real)     factor by which time step must be smaller than tl
    10               IFINE  (int)      decrease of time step for vertical motion by factor ifine
    1                IOUT              1 concentration, 2 mixing ratio, 3 both, 4 plume traject, 5=1+4
    0                IPOUT             particle dump: 0 no, 1 every output interval, 2 only at end
    0                LSUBGRID          subgrid terrain effect parameterization: 1 yes, 0 no
    0                LCONVECTION       convection: 3 yes, 0 no
    3600.            DT_CONV  (real)   time interval to call convection, seconds
    0                LAGESPECTRA       age spectra: 1 yes, 0 no
    0                IPIN              continue simulation with dumped particle data: 1 yes, 0 no
    0                IFLUX             calculate fluxes: 1 yes, 0 no
    1                IOUTPUTFOREACHREL CREATE AN OUPUT FILE FOR EACH RELEASE LOCATION: 1 YES, 0 NO
    0                MDOMAINFILL       domain-filling trajectory option: 1 yes, 0 no, 2 strat. o3 tracer
    1                IND_SOURCE        1=mass unit , 2=mass mixing ratio unit
    2                IND_RECEPTOR      1=mass unit , 2=mass mixing ratio unit
    0                NESTED_OUTPUT     shall nested output be used? 1 yes, 0 no
    0                LINIT_COND   INITIAL COND. FOR BW RUNS: 0=NO,1=MASS UNIT,2=MASS MIXING RATIO UNIT
    1                TURB_OPTION       0=no turbulence; 1=diagnosed as in flexpart_ecmwf; 2 and 3=from tke.
    1                LU_OPTION         0=old landuse (IGBP.dat); 1=landuse from WRF
    1                CBL SCHEME        0=no, 1=yes. works if TURB_OPTION=1
    0                SFC_OPTION        0=default computation of u*, hflux, pblh, 1=from wrf
   -1                WIND_OPTION       0=snapshot winds, 1=mean winds,2=snapshot eta-dot,-1=w based on divergence
    0                TIME_OPTION       1=correction of time validity for time-average wind,  0=no need
    0                OUTGRID_COORD     0=wrf grid(meters), 1=regular lat/lon grid
    1                RELEASE_COORD     0=wrf grid(meters), 1=regular lat/lon grid
    2                IOUTTYPE          0=default binary, 1=ascii (for particle dump only),2=netcdf
    10                NCTIMEREC (int)   Time frames per output file, only used for netcdf
    0                VERBOSE           VERBOSE MODE,0=minimum, 100=maximum
```

When considering the wet/dry deposition, the should be moved to this, which are supported in the standard FLEXPART files.

