---

layout: post
title: "From Raw Weather Data to Forecasts: A Complete WRF-WPS Workflow Guide"
date: 2026-06-02
categories: [WRF, WPS, Weather, HPC]
------------------------------------

# From Raw Weather Data to Forecasts: A Complete WRF-WPS Workflow Guide

The Weather Research and Forecasting (WRF) model is one of the most widely used numerical weather prediction systems in the world. It is used by meteorological agencies, research institutions, climate scientists, and industry for weather forecasting, climate studies, renewable energy forecasting, and atmospheric research.

This article walks through the complete WRF workflow—from installing dependencies to generating forecast outputs. The goal is not only to explain *how* to run WRF, but also *why* each component exists in the forecasting pipeline.

---

# What is WRF?

WRF (Weather Research and Forecasting Model) is a mesoscale numerical weather prediction model that solves the governing equations of atmospheric motion on a discretized grid.

Unlike machine learning models, WRF is a physics-based simulator. It computes future atmospheric states using conservation laws of:

* Mass
* Momentum
* Energy
* Moisture

Given appropriate initial and boundary conditions, WRF can generate forecasts ranging from local thunderstorms to regional weather systems.

---

# Understanding the WRF Workflow

WRF cannot directly consume weather datasets such as GFS or ERA5.

Instead, meteorological data must first be converted into a format suitable for the model using the WRF Preprocessing System (WPS).

The complete workflow is:

```text
Global Weather Data
(GFS / ERA5 / GDAS)
          |
          v
       Ungrib
          |
          v
   Intermediate Files
          |
          v
       Metgrid
          |
          v
      met_em Files
          |
          v
      real.exe
          |
          v
 wrfinput / wrfbdy
          |
          v
      wrf.exe
          |
          v
     Forecast Output
```

---

# Components of the WRF Ecosystem

| Component | Purpose                                                    |
| --------- | ---------------------------------------------------------- |
| Geogrid   | Generates terrain, land use, soil, and geographical fields |
| Ungrib    | Converts GRIB weather files into WPS intermediate format   |
| Metgrid   | Interpolates meteorological data onto model domains        |
| real.exe  | Creates initial and boundary conditions                    |
| wrf.exe   | Performs atmospheric simulation                            |
| wrfout    | Final forecast outputs                                     |

---

# Understanding Domain Nesting

One of the most powerful features of WRF is nested domains.

Instead of running a high-resolution simulation over an entire continent, WRF allows multiple grids with different resolutions.

Example:

```text
d01 : India (27 km)
 |
 +-- d02 : North India (9 km)
       |
       +-- d03 : Delhi (3 km)
```

Advantages:

* Captures large-scale atmospheric patterns
* Provides high-resolution forecasts over regions of interest
* Significantly reduces computational cost

---

# System Requirements

Recommended environment:

* Ubuntu 22.04+
* GCC/GFortran
* OpenMPI or MPICH
* NetCDF
* Jasper
* libpng
* zlib

WRF can be built on:

* Local Linux systems
* WSL2
* HPC clusters
* Cloud virtual machines

---

# Installing Dependencies

```bash
sudo apt update

sudo apt install -y \
build-essential \
gcc \
g++ \
gfortran \
m4 \
perl \
tcsh \
csh \
wget
```

Verify compiler installation:

```bash
gcc --version
gfortran --version
m4 --version
```

---

# Compiler Compatibility Tests

Before building WRF, it is useful to verify that:

* Fortran compiler works
* C compiler works
* C and Fortran can communicate
* Shell scripting support exists
* Perl support exists

Download UCAR test suite:

```bash
mkdir -p $HOME/TESTS
cd $HOME/TESTS

wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/Fortran_C_tests.tar

tar -xvf Fortran_C_tests.tar
```

Run each test and verify successful execution.

---

# Installing Required Libraries

Create a build directory:

```bash
mkdir -p $HOME/Build_WRF/LIBRARIES
```

Required libraries:

* NetCDF
* MPICH/OpenMPI
* Jasper
* libpng
* zlib

These libraries provide:

| Library | Purpose                 |
| ------- | ----------------------- |
| NetCDF  | Scientific data storage |
| MPICH   | Parallel execution      |
| zlib    | Compression             |
| libpng  | PNG support             |
| Jasper  | GRIB2 support           |

---

# Environment Variables

Add the following to `~/.bashrc`.

```bash
export DIR=$HOME/Build_WRF/LIBRARIES

export CC=gcc
export CXX=g++

export FC=gfortran
export F77=gfortran

export PATH=$DIR/netcdf/bin:$PATH
export NETCDF=$DIR/netcdf

export PATH=$DIR/mpich/bin:$PATH

export LDFLAGS=-L$DIR/grib2/lib
export CPPFLAGS=-I$DIR/grib2/include

export JASPERLIB=$DIR/grib2/lib
export JASPERINC=$DIR/grib2/include
```

Reload:

```bash
source ~/.bashrc
```

---

# Building WPS

Clone WPS:

```bash
git clone https://github.com/wrf-model/WPS.git
cd WPS
```

Configure:

```bash
./configure
```

Select:

```text
Linux x86_64, gfortran (dmpar)
```

Compile:

```bash
./compile >& compile.log &
tail -f compile.log
```

Expected executables:

```bash
geogrid.exe
metgrid.exe
ungrib.exe
```

---

# Downloading Geographical Data

Geographical data contains:

* Terrain height
* Land use categories
* Soil types
* Vegetation information

Download:

```bash
mkdir -p $HOME/Build_WRF/geog_data

cd $HOME/Build_WRF/geog_data

wget http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz

tar -xvzf geog_high_res_mandatory.tar.gz
```

---

# Building WRF

Clone the model:

```bash
git clone https://github.com/wrf-model/WRF.git

cd WRF
```

Configure:

```bash
./configure
```

Select:

```text
GNU (gfortran/gcc) (dmpar)
```

Compile:

```bash
./compile em_real >& compile.log &
```

Monitor:

```bash
tail -f compile.log
```

Successful compilation generates:

```bash
real.exe
wrf.exe
ndown.exe
tc.exe
```

---

# WPS Workflow

## Step 1: Geogrid

Purpose:

Generate static geographical fields.

Run:

```bash
./geogrid.exe
```

Outputs:

```bash
geo_em.d01.nc
geo_em.d02.nc
```

---

## Step 2: Ungrib

Purpose:

Decode meteorological GRIB files.

Link appropriate Vtable:

```bash
ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable
```

Link weather data:

```bash
./link_grib.csh /path/to/gfs/files
```

Run:

```bash
./ungrib.exe
```

Outputs:

```bash
FILE:2024-01-02_00
FILE:2024-01-02_03
...
```

---

## Step 3: Metgrid

Purpose:

Interpolate meteorological fields onto model domains.

Run:

```bash
./metgrid.exe
```

Outputs:

```bash
met_em.d01.*
met_em.d02.*
```

---

# Running WRF

## Step 1: real.exe

Purpose:

Generate model initial and boundary conditions.

Run:

```bash
mpirun -np 4 ./real.exe
```

Outputs:

```bash
wrfinput_d01
wrfinput_d02
wrfbdy_d01
```

---

## Step 2: wrf.exe

Purpose:

Perform atmospheric simulation.

Run:

```bash
mpirun -np 4 ./wrf.exe
```

Outputs:

```bash
wrfout_d01_*
wrfout_d02_*
```

---

# Understanding WRF Outputs

WRF outputs are stored in NetCDF format.

Important variables:

| Variable | Description               |
| -------- | ------------------------- |
| T2       | 2-meter temperature       |
| U10      | 10-meter zonal wind       |
| V10      | 10-meter meridional wind  |
| PSFC     | Surface pressure          |
| Q2       | Specific humidity         |
| RAINNC   | Accumulated precipitation |

---

# Reading WRF Outputs in Python

```python
import xarray as xr

ds = xr.open_dataset(
    "wrfout_d01_2024-01-02_00:00:00"
)

print(ds["T2"])
```

---

# Computational Considerations

Runtime depends on:

* Grid resolution
* Number of domains
* Number of vertical levels
* Forecast duration
* Available CPU cores

Example:

| Domain | Resolution |
| ------ | ---------- |
| d01    | 27 km      |
| d02    | 9 km       |

A 24-hour forecast can take:

* Several hours on a laptop
* Tens of minutes on an HPC cluster

---

# Common Errors and Troubleshooting

## CFL Error

Cause:

Time step too large.

Guideline:

```text
time_step ≈ 6 × dx(km)
```

Example:

```text
dx = 9 km
time_step ≈ 54 seconds
```

---

## geogrid.exe Fails

Check:

* geog_data_path
* Geography dataset installation
* Permissions

---

## real.exe Fails

Verify:

* Date ranges
* met_em files
* Nesting configuration

---

## wrf.exe Crashes

Possible causes:

* CFL violation
* Insufficient memory
* Incorrect physics configuration

---

# Key Takeaways

WRF consists of two major components:

1. WPS for preprocessing
2. WRF for numerical simulation

The complete forecasting pipeline is:

```text
Geographical Data
+
Meteorological Data
        |
        v
       WPS
        |
        v
    met_em Files
        |
        v
     real.exe
        |
        v
     wrf.exe
        |
        v
 Forecast Outputs
```

Understanding this workflow is essential before moving toward advanced topics such as data assimilation, climate downscaling, ensemble forecasting, or machine-learning-based WRF emulation.
