---
layout: post
title: "From Raw Weather Data to Forecasts: A Complete WRF-WPS Workflow Guide"
date: 2026-06-02
author: "Your Name"
categories: [WRF, HPC, Weather Modeling]
tags: [WRF, WPS, NetCDF, HPC, Python, Geospatial, NWP]
excerpt: "A complete walkthrough of the WRF-WPS pipeline — from installing dependencies and configuring domains to running real.exe, wrf.exe, and reading forecast outputs in Python."
---

The Weather Research and Forecasting (WRF) model is one of the most widely used numerical weather prediction systems in the world. It is used by meteorological agencies, research institutions, climate scientists, and industry for weather forecasting, climate studies, renewable energy forecasting, and atmospheric research.

This article walks through the complete WRF workflow — from installing dependencies to generating forecast outputs. The goal is not only to explain *how* to run WRF, but also *why* each component exists in the forecasting pipeline.

---

## What is WRF?

WRF (Weather Research and Forecasting Model) is a mesoscale numerical weather prediction model that solves the governing equations of atmospheric motion on a discretized grid.

Unlike machine learning models, WRF is a physics-based simulator. It computes future atmospheric states using conservation laws of mass, momentum, energy, and moisture. Given appropriate initial and boundary conditions, WRF can generate forecasts ranging from local thunderstorms to regional weather systems.

---

## Understanding the WRF Workflow

WRF cannot directly consume weather datasets such as GFS or ERA5. Instead, meteorological data must first be converted into a format suitable for the model using the **WRF Preprocessing System (WPS)**. The complete pipeline looks like this:

```text
Global Weather Data (GFS / ERA5 / GDAS)
          ↓
       Ungrib  →  Intermediate Files
          ↓
       Metgrid  →  met_em Files
          ↓
       real.exe  →  wrfinput / wrfbdy
          ↓
       wrf.exe  →  Forecast Output (wrfout_*)
```

---

## Components of the WRF Ecosystem

| Component | Purpose |
|-----------|---------|
| Geogrid   | Generates terrain, land use, soil, and geographical fields |
| Ungrib    | Converts GRIB weather files into WPS intermediate format |
| Metgrid   | Interpolates meteorological data onto model domains |
| real.exe  | Creates initial and boundary conditions |
| wrf.exe   | Performs atmospheric simulation |
| wrfout    | Final forecast outputs in NetCDF format |

---

## Understanding Domain Nesting

One of the most powerful features of WRF is **nested domains** — running multiple grids at different resolutions within the same simulation.

```text
d01 : India       (27 km)
 └── d02 : North India  (9 km)
       └── d03 : Delhi  (3 km)
```

This approach captures large-scale atmospheric patterns at the outer domain while providing high-resolution forecasts over the region of interest, at a fraction of the cost of running a global high-resolution simulation.

---

## System Requirements

Recommended environment:

- Ubuntu 22.04+
- GCC / GFortran
- OpenMPI or MPICH
- NetCDF, Jasper, libpng, zlib

WRF can be built on local Linux systems, WSL2, HPC clusters, or cloud VMs.

---

## Installing Dependencies

```bash
sudo apt update
sudo apt install -y \
  build-essential gcc g++ gfortran \
  m4 perl tcsh csh wget
```

Verify your compilers:

```bash
gcc --version
gfortran --version
m4 --version
```

---

## Compiler Compatibility Tests

Before building WRF, verify that Fortran and C compilers can communicate, and that shell and Perl support exist. Download the UCAR test suite:

```bash
mkdir -p $HOME/TESTS && cd $HOME/TESTS
wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/Fortran_C_tests.tar
tar -xvf Fortran_C_tests.tar
```

Run each test and verify successful execution before proceeding.

---

## Installing Required Libraries

```bash
mkdir -p $HOME/Build_WRF/LIBRARIES
```

| Library | Purpose |
|---------|---------|
| NetCDF  | Scientific data storage |
| MPICH   | Parallel execution |
| zlib    | Compression |
| libpng  | PNG support |
| Jasper  | GRIB2 support |

---

## Environment Variables

Add the following to `~/.bashrc`:

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

Then reload: `source ~/.bashrc`

---

## Building WPS

```bash
git clone https://github.com/wrf-model/WPS.git && cd WPS
./configure   # Select: Linux x86_64, gfortran (dmpar)
./compile >& compile.log &
tail -f compile.log
```

A successful build produces `geogrid.exe`, `metgrid.exe`, and `ungrib.exe`.

---

## Downloading Geographical Data

```bash
mkdir -p $HOME/Build_WRF/geog_data && cd $HOME/Build_WRF/geog_data
wget http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz
tar -xvzf geog_high_res_mandatory.tar.gz
```

This dataset contains terrain height, land use categories, soil types, and vegetation information required by Geogrid.

---

## Building WRF

```bash
git clone https://github.com/wrf-model/WRF.git && cd WRF
./configure   # Select: GNU (gfortran/gcc) (dmpar)
./compile em_real >& compile.log &
tail -f compile.log
```

Successful compilation generates `real.exe`, `wrf.exe`, `ndown.exe`, and `tc.exe`.

---

## WPS Workflow

### Step 1 — Geogrid

Generates static geographical fields (`geo_em.d01.nc`, `geo_em.d02.nc`, …).

```bash
./geogrid.exe
```

### Step 2 — Ungrib

Decodes meteorological GRIB files from GFS/ERA5 into WPS intermediate format.

```bash
ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable
./link_grib.csh /path/to/gfs/files
./ungrib.exe
```

Outputs: `FILE:2024-01-02_00`, `FILE:2024-01-02_03`, …

### Step 3 — Metgrid

Interpolates meteorological fields onto model domains.

```bash
./metgrid.exe
```

Outputs: `met_em.d01.*`, `met_em.d02.*`

---

## Running WRF

### Step 1 — real.exe

Generates initial and boundary conditions from the `met_em` files.

```bash
mpirun -np 4 ./real.exe
```

Outputs: `wrfinput_d01`, `wrfinput_d02`, `wrfbdy_d01`

### Step 2 — wrf.exe

Runs the atmospheric simulation.

```bash
mpirun -np 4 ./wrf.exe
```

Outputs: `wrfout_d01_*`, `wrfout_d02_*`

---

## Understanding WRF Outputs

WRF outputs are stored in **NetCDF** format. Key variables:

| Variable | Description |
|----------|-------------|
| `T2`     | 2-meter temperature |
| `U10`    | 10-meter zonal wind |
| `V10`    | 10-meter meridional wind |
| `PSFC`   | Surface pressure |
| `Q2`     | Specific humidity |
| `RAINNC` | Accumulated precipitation |

---

## Reading WRF Outputs in Python

```python
import xarray as xr

ds = xr.open_dataset("wrfout_d01_2024-01-02_00:00:00")

# Extract 2-metre temperature
t2 = ds["T2"]
print(t2)

# Convert to Celsius and plot
import matplotlib.pyplot as plt
(t2.isel(Time=0) - 273.15).plot(cmap="RdBu_r")
plt.title("2m Temperature (°C)")
plt.savefig("t2_forecast.png", dpi=150)
```

---

## Computational Considerations

Runtime depends on grid resolution, number of domains, vertical levels, forecast duration, and available CPU cores.

| Domain | Resolution | Approx. 24h Forecast Time |
|--------|------------|--------------------------|
| d01    | 27 km      | ~15 min (32 cores) |
| d02    | 9 km       | ~45 min (32 cores) |
| d03    | 3 km       | ~3 hours (32 cores) |

A rule of thumb: **3× resolution increase ≈ 27× more computation** (3D scaling).

---

## Common Errors and Troubleshooting

### CFL Violation

The most common crash. Time step too large for the grid spacing.

```text
time_step ≈ 6 × dx (km)
# Example: dx = 9 km → time_step ≈ 54 seconds
```

### geogrid.exe Fails

Check `geog_data_path` in `namelist.wps`, verify the geography dataset is correctly installed, and confirm file permissions.

### real.exe Fails

Verify that date ranges in `namelist.input` match your `met_em` files and that nesting configuration (grid ratios, parent IDs) is consistent.

### wrf.exe Crashes

Common causes: CFL violation, insufficient memory per MPI rank, incorrect physics suite configuration (e.g., mismatched microphysics and cumulus options).

---

## Key Takeaways

WRF consists of two major systems — **WPS** for preprocessing and **WRF** for numerical simulation. The pipeline is:

```text
Geographical Data + Meteorological Data
        ↓
       WPS  →  met_em Files
        ↓
    real.exe  →  wrfinput / wrfbdy
        ↓
    wrf.exe  →  Forecast Outputs (wrfout_*)
```

Understanding this pipeline thoroughly is essential before moving toward advanced topics such as data assimilation, climate downscaling, ensemble forecasting, or — the direction of my own research — **machine-learning-based WRF emulation** with U-Net and Transformer surrogates.