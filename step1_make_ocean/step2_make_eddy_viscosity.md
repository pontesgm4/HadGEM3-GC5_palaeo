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

### 1. Transfer required files from Archer2 to Jasmin
Note: I suppose step1_make_domain_cfg can be entirely run on Jasmin, but I haven't tested. 

Copy:

```bash
domain_cfg.nc 
mesh_mask.nc
```

Once you have copied the new files above into your work directory, run `make_ahmcoef.py`, located at:

```text
cp /gws/pw/j25/past2future/users/gpontes/HadGEM3-GC5-0/LP_exp_bc/make_ocean/make_ahmcoef.py .
```

The run:

```bash
python make_ahmcoef.py
```

Note: this script requires the netCDF4 python library, which is not installed in the latest Jasmin jaspy. So need to use your python environment with netCDF4 installed.

This writes:

```text
ahmcoef.nc
```

Check:

```bash
ncview ahmcoef.nc
```

---

## 2. Generating the `eddy_viscosity_3D.nc` file

### Parameter values

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

I have updated Dave Storkey’s script to be consistent with the new set of variables in the domain_cfg.nc and mesh_mask.nc files for GC5:

```text
/gws/pw/j25/past2future/users/gpontes/HadGEM3-GC5-0/LP_exp_bc/make_ocean/make_eorca1_eddy_viscosity_3D_file.py
```

Already containing the necessary input values and files.

Make executable if preferred:

```bash
chmod +x make_eorca1_eddy_viscosity_3D_file.py
```

Then, run:

```bash
python make_eorca1_eddy_viscosity_3D_file.py     -m 20000     -t 1000     -d domain_cfg.nc     -k mesh_mask.nc     -c ahmcoef.nc
```

This writes:

```text
eddy_viscosity_3D.nc
```

![Pliocene bathymetry on the NEMO ORCA1 grid](https://github.com/user/repo/assets/....png)

---

### 3. Check output

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
ahm1_3d
```

Quick visual check:

```bash
ncview eddy_viscosity_3D.nc
```

Useful checks:

- surface viscosity lowest near the equator
- smooth transition poleward
- coastal values follow the new land–sea mask
- no strong gradients (both meridionally and zonally)


