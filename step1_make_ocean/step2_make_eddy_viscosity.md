## Generating the `eddy_viscosity_3D.nc` file

The GC5/NEMO4 configuration requires an ancillary file containing the **3D lateral eddy viscosity fields** (`ahmt_3d` and `ahmf_3d`). These are defined on the NEMO grid and vary horizontally (with latitude and coastline geometry) and vertically (with depth).

For the Palaeo setup this file should be regenerated using the new bathymetry and mesh generated in the previous steps.

### Required input files

This step uses:

- `domain_cfg.nc`  
  Generated in the previous step with `DOMAINcfg`. Provides:
  - `nav_lon`
  - `nav_lat`
  - `gdept_1d`

- `mesh_mask.nc`  
  Generated in the previous step. Used to create the `ahmcoef` coefficient field.

- `ahmcoef.nc`  
  NetCDF file containing:

  ```text
  icof(y,x)
  ```

  This file is generated using Charles Williams’ script and is based on the new `mesh_mask.nc`.

---

### 1. Trasnfer required files from Archer2 to Jasmin
Note: I suppose step1 can be entirely run on Jasmin, but I haven't tested. 

Copy:

```bash
domain_cfg.nc 
mesh_mask.nc
```

Copy Charles Williams’ `ahmcoef` generation script and run it.

Notes: I needed to do small modifications in Charles' script because the new variables within the new mesh_mask.nc for GC5 is different from GC3 (i.e., does not include nav_lat)
my updated script is located on Jasmin at:

```text
/gws/pw/j25/past2future/users/gpontes/HadGEM3-GC5-0/LP_exp_bc/make_ocean/ahmcoef.py
```
The run:

```bash
python ahmcoef.py
```
Note: this script requires the netCDF4 python library, which is not installed in the latest Jasmin jaspy. So need to use your python environment with netCDF4 installed.

Check:

```bash
ncview ahmcoef.nc
```

---

### 2. Generating the `eddy_viscosity_3D.nc` file

# Parameter values

Following guidance from Dave Storkey (Met Office), use:

```text
aht0 = 1000
ahm0 = 20000
```

where:

- `aht0` = viscosity at the equator
- `ahm0` = viscosity outside the tropics

The script applies:

- **horizontal variation**
  - low viscosity at the equator
  - smooth transition between 2.5°–20°
  - higher viscosity poleward

- **coastal variation**
  - modified using `icof`

- **vertical variation**
  - viscosity increases with depth
  - transition from ~800–3000 m

---

# Dave Storkey’s script is located at:

```text
/gws/pw/j25/past2future/users/gpontes/HadGEM3-GC5-0/LP_exp_bc/make_ocean/make_eorca1_eddy_viscosity_3D_file.py
```
Already containing the necessary input values and files.

Make executable if preferred:

```bash
chmod +x make_eorca1_eddy_viscosity_3D_file.py
```
Then, simply run:

```bash
python make_eorca1_eddy_viscosity_3D_file.py
```

This writes:

```text
eddy_viscosity_3D.nc
```

---

### 5. Check output

Inspect:

```bash
ncdump -h eddy_viscosity_3D.nc
```

Expected variables:

```text
nav_lon
nav_lat
ahmt_3d
ahmf_3d
```

Quick visual check:

```bash
ncview eddy_viscosity_3D.nc
```

Useful checks:

- surface viscosity lowest near the equator
- smooth transition poleward
- values increase with depth
- coastal values follow the new land–sea mask

---

### Notes / caveats

- `ahmcoef.nc` depends on the new `mesh_mask.nc`, so regenerate it whenever coastlines or gateways are modified.

- The script was originally written for **eORCA1 / NEMO4** and works directly with GC5.

- For major palaeogeographic changes (e.g. Panama, Indonesian Throughflow, Arctic gateways), inspect the resulting viscosity field carefully around narrow passages and coastlines.

