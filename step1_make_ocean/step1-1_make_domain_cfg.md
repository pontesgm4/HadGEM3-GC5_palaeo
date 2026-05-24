## Interpolating bathymetry

### 1. Create the `make_ocean` directory on JASMIN

This directory will contain all scripts and files required to generate the NEMO ancillaries.

```bash
cd /gws/pw/j25/past2future/users/gpontes/HadGEM3-GC5/LP_exp_bc/
mkdir make_ocean
cd make_ocean
```

---

### 2. Transfer `domain_cfg.nc` from ARCHER2 to JASMIN

Copy the original `domain_cfg.nc` file using Globus.

On ARCHER2 the file is located at:

```text
cylc-run/u-do332/run1/share/data/etc/ancil_nemo/
```

---

### 3. Interpolate PRISM bathymetry onto the NEMO ORCA grid

Run:

```bash
python make_topog.py
```

Update the input/output filenames in the script as needed.

This script interpolates the reconstructed Pliocene bathymetry onto the NEMO ORCA grid.

**Note on interpolation method**

Conservative remapping was tested initially (following previous workflows using ACCESS-ESM1.5 / MOM), but is not currently used here because:

- `domain_cfg.nc` provides T-point coordinates (`glamt`, `gphit`) but does not provide explicit cell corners in the format required by `xESMF`;
- ORCA tripolar geometry can cause conservative remapping to fail (`ESMF rc=506`);
- for bathymetry, conservative remapping can introduce unrealistic smoothing of narrow gateways and shallow shelves.

At present the workflow uses **bilinear interpolation**.

For the initial setup I used Charlie Williams’ already regridded bathymetry:

```bash
cp /gws/ssde/j25a/ncas_climate/vol2/users/cwilliams2011/data1_for_use.d/pliod.d/new_plio.d/build.d/ancils.d/bathy.d/plio_enh_topo_v1.0_new.nc_regridded_bilinear.nc .
```

---

### 4. Clean and manually adjust bathymetry

Run:

```bash
python adjust_topog.py
```

This script:

- removes isolated ocean grid cells (lakes / disconnected basins),
- identifies narrow straits,
- highlights shallow cells (<40 m), and make them 40m deep
- allows manual editing of key regions.

Workflow:

1. Run the script.
2. The script generates `straits.txt`.
3. Inspect the listed grid cells.
4. Copy the output into `adjust_topog.py`.
5. Re-run.
6. Repeat until `straits.txt` is empty.

Even after `straits.txt` is empty, additional manual adjustments are usually required.

Typical locations to inspect carefully:

- Central American Seaway / Panama
- Indonesian Throughflow
- Arctic gateways
- Southern Ocean passages
- narrow coastal embayments
- enclosed inland seas

A practical way to inspect:

```bash
module load ncview
ncview adjusted_bathy.nc
```

Set the display range to 0–5 m to clearly highlight the land–sea mask.

Record grid indices that should be:

- filled as land (`0`)
- deepened to ocean

then update `adjust_topog.py` and repeat.

Once satisfied, this becomes the **final bathymetry file**.

---

# Generating `domain_cfg.nc` and `mesh_mask.nc`

### 5. Create the equivalent working directory on ARCHER2

```bash
cd /work/n02/n02/gpontes/LP_exp_bc
mkdir make_ocean
cd make_ocean
```

---

### 6. Check out the NEMO utilities from the MetOffice

Requires MORS access.

```bash
fcm co https://code.metoffice.gov.uk/svn/nemo/utils utils
cd utils
```

---

### 7. Load required modules

```bash
module use /work/y07/shared/umshared/moci/modules/modules
module load GC5-PrgEnv
```

---

### 8. Update the ARCHER2 build configuration

Copy the ARCHER2 build file:

```bash
cp build/arch/NOC/arch-X86_ARCHER2-Cray.fcm \
   build/arch/NOC/arch-X86_ARCHER2-Cray-updated.fcm
```

Update:

```text
%XIOS_HOME /work/n01/shared/acc/xios-trunk
```

to:

```text
%XIOS_HOME /work/n01/shared/nemo/xios-trunk
```

---

### 9. Build `DOMAINcfg`

In this NEMO utilities checkout, the `arch/` and `mk/` directories are under `build/`, but `tools/maketools` expects them at the repository root.

Move them:

```bash
mv build/arch .
mv build/mk .
```

Then build:

```bash
cd tools
./maketools -m X86_ARCHER2-Cray-updated -n DOMAINcfg
```

This creates:

```text
tools/DOMAINcfg/
```

---

### 10. Copy template files

Go to:

```bash
cd tools/DOMAINcfg/
```

Copy:

```bash
cp /work/n02/n02/an25872/NEMO_SRC/utils/tools/DOMAINcfg/namelist_cfg .
cp /work/n02/n02/an25872/NEMO_SRC/utils/tools/DOMAINcfg/eORCA1_coordinates_nc4.nc_from_MR.nc .
```

Then transfer your final bathymetry file from JASMIN using Globus.

Update `namelist_cfg`:

- bathymetry filename
- bathymetry variable name (if different)

---

### 11. Submit `make_domain_cfg.exe`

Create:

```bash
vim submit_domaincfg.sh
```

Paste:

```bash
#!/bin/bash -l
#
#SBATCH --job-name=DOMAINcfg_test
#SBATCH --output=logs/DOMAINcfg_%j.out
#SBATCH --error=logs/DOMAINcfg_%j.err
#SBATCH --time=60:00
#SBATCH --chdir=/work/n02/n02/gpontes/LP_exp_bc/make_ocean/utils/tools/DOMAINcfg
#SBATCH --partition=serial
#SBATCH --qos=serial
#SBATCH --account=n02-P2F
#SBATCH --exclusive=mcs
#SBATCH --ntasks=1
#SBATCH --mem=100G
#SBATCH --export=NONE

srun -n 1 ./make_domain_cfg.exe
```

Update:

```bash
#SBATCH --chdir=
```

to your own directory.

Create log directory:

```bash
mkdir -p logs
```

Submit:

```bash
sbatch submit_domaincfg.sh
```

Check status:

```bash
squeue -u <archer2-username>
```

---

### 12. Verify output

Successful completion should generate:

- `domain_cfg.nc`
- `mesh_mask.nc`

Inspect with:

```bash
module load ncview
ncview domain_cfg.nc
```

Check that:

- `bathy_metry` matches the expected Pliocene reconstruction
- land–sea mask is correct
- key gateways are open/closed as intended
- no obvious artefacts remain
