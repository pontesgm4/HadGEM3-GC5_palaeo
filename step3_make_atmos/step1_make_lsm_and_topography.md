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

---

### 3. Smoothing the Antarctic topography

The interpolated atmospheric topography can contain strong gradients near the Antarctic coast and the southern boundary of the atmospheric grid. These gradients can cause numerical problems in the UM.

This step applies a dedicated smoothing procedure to the southernmost part of the atmospheric topography while leaving the remainder of the field unchanged.

Run:

```text
python step3_smooth_antarc.py
```

The script reads:

`LP_topog_atmos.nc`

containing the atmospheric topography generated in Step 2.

**Notes**

- Only the southernmost nine latitude rows are modified by the moving-average smoothing.
- The first three rows are first flattened to a common mean value.
- The smoothing uses an 11 × 5 longitude–latitude window.
- Longitude is treated periodically when constructing the smoothing window.
- The Pliocene land-sea mask is reapplied after smoothing.
- All atmospheric topography north of the first nine latitude rows is unchanged from `LP_topog_atmos.nc`.

---

### 4. Scaling the orographic gradient fields

The UM orographic gravity-wave drag scheme uses several fields derived from surface topography. Because the Pliocene topography differs from the original UM topography, the corresponding orographic parameters need to be adjusted consistently.

This step derives the relationship between surface elevation and the original UM orographic parameters, then applies these relationships to the reconstructed Pliocene topography.

The fields treated are:

- `grad_xx`
- `grad_yy`
- `silhouette`
- `peak_trough`

**Input files**

The script requires the following files:

`../../orig_ancils/orig_ancil_um/qrparm.orog_wavedrag.nc`

Original UM topography and orographic gradient fields (grad_xx and grad_yy).

`../../orig_ancils/orig_ancil_um/qrparm.ash.nc`

Original UM silhouette and peak_trough fields.

`LP_lsm.nc`

Pliocene land-sea mask generated in Step 1.

`LP_topog_atmos_antarc.nc`

Pliocene atmospheric topography generated in Step 3.

**Running the script**

Run:

`python step4_scale_xx_yy_orog_grads.py`

The script:

- Calculates the cosine-latitude-weighted mean of the original orographic fields in 100 m elevation bins from 0 to 5500 m.
- Fits a smoothing B-spline to the relationship between elevation and each orographic field.
- Evaluates the fitted relationships using the Pliocene atmospheric topography.
- Sets negative values to zero.
- Applies the Pliocene land-sea mask.

**Output files**

The main output is:

`LP_xx_yy_scaled.nc`

containing:

- grad_xx
- grad_yy
- silhouette
- peak_trough

The script also produces the diagnostic plot:

`spline_fit_gradients.pdf`

when `test_plot = True`.

![spline_fit_gradients](https://github.com/pontesgm4/HadGEM3-GC5_palaeo/blob/main/step1_make_ocean/spline_fit_gradients.png)

Notes

- The scaling is based on the relationship between the original UM orographic fields and surface elevation.
- The Pliocene land-sea mask is applied to the resulting fields.
- `peak_trough` is generated but is currently not included in the diagnostic plot.
