## Smoothing bathymetry

### Motivation

The interpolated PRISM bathymetry introduced a small number of grid cells with very steep local depth gradients, particularly in continental shelf regions. Although these features are physically plausible in the source reconstruction, they can generate large pressure-gradient errors and unstable barotropic motions when represented on the relatively coarse ORCA1 grid.

UKCM2/NEMO Pliocene spin up run systematically crashed with extreme sea-surface-height anomalies (`sossheig`) concentrated in a few shelf cells in Antarctica, Artic, and ITF regions. Diagnostics suggested that the instability was associated with local bathymetric gradients rather than with tracer fields or freshwater forcing.

To improve numerical stability, an straightforward solution was to decrease NEMO timestep. However, further crashes and expensive cost of reducing timestep motivated a bathymetry smoothing.

---

### 1. Create a smoothed bathymetry

Run:

```text
python smooth_bathymetry.py
```

Update the input/output filenames as required.

The script:

- reads the interpolated PRISM bathymetry;
- applies local smoothing only where required;
- preserves major ocean gateways and large-scale bathymetric structure;
- minimises changes away from problematic regions.

Output:
```text
plio_enh_topo_v1.0_new_reggridded_v6_local_cluster_r060_s200_d050_p75.nc
```

#### Compare original and smoothed bathymetry
 
Run:

```text
python bathy_smooth_diagnostics.py
```

This script quantifies the bathymetric modifications and produces diagnostics showing:

- maximum depth changes;
- spatial distribution of modifications;
- affected ocean area;
- preservation of major ocean basins and gateways.

The goal is to confirm that smoothing only alters the problematic regions while retaining the intended Pliocene paleogeography.

figure

**Note: Rebuild the NEMO mesh (`domain_cfg.nc`)**

After modifying bathymetry, the NEMO mesh must be regenerated.

Follow the standard NEMO mesh-generation workflow already documented in `step1_make_domain_cfg` in this repository to produce a new `domain_cfg.nc` for the smoothed bathymetry.

---

### 2. Generating a Pliocene ocean cold-start file

### Motivation

After rebuilding the mesh, some ocean cells became newly wet because of changes to the local partial-step representation.

A direct restart using the old restart file would therefore leave missing values in these newly wet cells.

To avoid this issue, a new cold-start temperature/salinity file was created by combining:

- Pliocene temperature and salinity from the original restart;
- the new ocean mask from the smoothed `domain_cfg.nc`.

### 1. Create the cold-start file

Run:

```text
python make_pliocene_coldstart.py
```
Inputs:

```text
eb987o_30820101_restart.nc
domain_cfg_v6.nc
domain_cfg_v6_smoothed.nc
EN4_v1.1.1995_2014.monthlymean_eORCA1T_NEMO_L75_teos10.nc
```

The script:

- constructs the 3-D ocean masks from the old and new domain_cfg.nc files;
- identifies newly wet ocean cells;
- copies the original Pliocene restart temperature and salinity unchanged on all existing ocean cells;
- fills newly wet cells using inverse-distance-weighted interpolation from nearby ocean points at the same vertical level;
- stores the reconstructed field in a NEMO-compatible temperature/salinity file.

For the current setup:

```text
Old wet cells     : 4,001,399
New wet cells     : 4,003,481
Newly wet cells   : 2,082
Newly dry cells   : 0
```

The largest interpolation distance was approximately:

```text
3.16 grid cells
```

indicating that all reconstructed values were obtained from nearby ocean points.

Output:

```text
Pliocene_T_and_S_coldstart_eORCA1_L75.nc
```

### 2. Verify the cold-start file

Run:

```text
python verify_pliocene_coldstart.py
```

The verification script checks:

file dimensions and metadata;
horizontal coordinates;
vertical coordinate consistency;
preservation of original restart values;
successful filling of newly wet cells;
dry-cell handling;
monthly structure;
NEMO fldread compatibility.

For the current setup the verification confirmed:

```text
Original restart wet cells preserved : OK
Newly wet cells filled               : 2,082
Newly dry cells                      : 0
Non-finite T/S values                : 0
Monthly records                      : identical
NEMO-compatible structure            : OK
```

The resulting file can therefore be used as an initial temperature/salinity condition when starting UKCM2/NEMO with the smoothed bathymetry.

**Notes**

- The EN4 file is used only as a convenient NEMO-compatible template container.
- All physical temperature and salinity values originate from the Pliocene UKCM2 restart.
- The reconstructed file is intended for cold starts, not for continuation from a previous simulation.
- If the bathymetry is modified again, both the mesh generation and cold-start generation steps must be repeated.
