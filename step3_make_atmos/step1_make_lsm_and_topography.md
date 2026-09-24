## Creating the atmospheric land-sea mask and topographic fields

The Pliocene paleogeography modifies the distribution of land and ocean relative to the present day. The Unified Model therefore requires an updated land-sea mask consistent with the reconstructed Pliocene geography.

The coupled setup provides a land-fraction ancillary (lsmask) containing fractional land coverage for each atmospheric grid cell. Several UM components, including river-routing and land-surface processes, additionally require a binary land-sea mask.

This step generates:

- a binary land-sea mask (LP_lsm.nc);

---

### 1. Generate the binary land-sea mask

Run:

```text
python step1_make_lsm.py
```

Input:

```text
atmo_mask_fracarea_anc_ns.nc
```

The script:

- reads the fractional land mask (lsmask);
- replaces missing values with zero;
- applies a land-fraction threshold of: 0.01
- classifies grid cells as:
  - land (1) if land fraction ≥ 0.01;
  - ocean (0) otherwise;
- writes the resulting binary land-sea mask.

Output:

```text
LP_lsm.nc
```

The resulting file contains:

- variable: lsm
  - values: 0 = ocean, 1 = land

- and can be used directly by downstream UM ancillary-generation tools.

**Notes**
- A threshold of 1% land fraction is used when constructing the binary mask.
- Missing land-fraction values are converted to ocean (0).
- The binary mask is derived entirely from the reconstructed Pliocene land fraction field.
- This step should be rerun whenever the Pliocene land-sea distribution is modified.

---

### 2. Creating the atmospheric topography

The atmospheric UM configuration requires topography on the atmospheric model grid. The Pliocene topography is first expressed as a change relative to the modern topography, then interpolated to the UM atmospheric grid and added to the original UM topography.

This approach preserves the original UM topography while applying the reconstructed Pliocene topographic changes.

Run:

```text
python step2_interp_topog_atmos.py
```

The script reads:

```text
LP_topo_v1.0.nc
Modern_std_topo_v1.0.nc
```

containing the Pliocene and modern topography, respectively.

The Pliocene topographic anomaly is calculated as:

`Pliocene anomaly = Pliocene topography − modern topography`

This anomaly is then used for the atmospheric interpolation.

**Output**

The resulting atmospheric topography is written to:

```text
LP_topog_atmos.nc
```

The file contains:

`variable: ht`

and is used as the Pliocene atmospheric topography for subsequent UM ancillary generation.

**Notes**
- The Pliocene topographic anomaly is calculated relative to the modern topography before interpolation.
- Conservative-normalised regridding is used to transfer the anomaly to the UM atmospheric grid.
- The Pliocene LSM generated in Step 1 is used to remove topographic anomalies over ocean cells.
- A 3 × 3 atmospheric-grid-cell smoothing is applied after interpolation.
- The original UM topography is used as the baseline, with the Pliocene anomaly added to it.
- Negative resulting elevations are set to zero.
- This step should be rerun whenever the Pliocene topography or land-sea mask is modified.
