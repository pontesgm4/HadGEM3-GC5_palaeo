## Smoothing bathymetry

### Motivation

The interpolated PRISM bathymetry introduced a small number of grid cells with very steep local depth gradients, particularly around the Antarctic continental shelf. Although these features are physically plausible in the source reconstruction, they can generate large pressure-gradient errors and unstable barotropic motions when represented on the relatively coarse ORCA1 grid.

Initial UKCM2/NEMO tests crashed shortly after startup with extreme sea-surface-height anomalies (`sossheig`) concentrated in a few Antarctic shelf cells. Diagnostics suggested that the instability was associated with local bathymetric gradients rather than with tracer fields or freshwater forcing.

To improve numerical stability, a locally smoothed bathymetry was generated before rebuilding the NEMO mesh.

---

### 1. Create a smoothed bathymetry

Run:

```text
python smooth_bathymetry.py
```

Update the input/output filenames as required.

The script:

reads the interpolated PRISM bathymetry;
applies local smoothing only where required;
preserves major ocean gateways and large-scale bathymetric structure;
minimises changes away from problematic regions.

Output:
```text
plio_enh_topo_v1.0_new_reggridded_v6_local_cluster_r060_s200_d050_p75.nc
```
---

### 2. Compare original and smoothed bathymetry

Run:

```text
python bathy_smooth_diagnostics.py
```

This script quantifies the bathymetric modifications and produces diagnostics showing:

maximum depth changes;
spatial distribution of modifications;
affected ocean area;
preservation of major ocean basins and gateways.

The goal is to confirm that smoothing only alters the problematic regions while retaining the intended Pliocene paleogeography.

figure

*** Note: Rebuild the NEMO mesh (`domain_cfg.nc`) ***

After modifying bathymetry, the NEMO mesh must be regenerated.

Follow the standard NEMO mesh-generation workflow already documented in `step1_make_domain_cfg` in this repository to produce a new `domain_cfg.nc` for the smoothed bathymetry:



