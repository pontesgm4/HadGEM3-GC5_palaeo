## Building vegetation and soil input dumps

PRISM4 provides a vegetation reconstruction in terms of **broad biome classes** (e.g., boreal forest, tundra, savanna). 
In contrast, the UM13.8/JULES land-surface scheme represents vegetation using **Plant Functional Types (PFTs)**. 
Each land grid cell contains fractional coverage of several PFTs, and 
these fractions collectively define the vegetation characteristics of that grid cell.

Consequently, the PRISM4 biome reconstruction cannot be used directly by the model. 
Instead, each PRISM4 biome must first be translated into an equivalent mixture of JULES PFTs. 
Table 1 shows the conversion adopted in this workflow.

The conversion is largely based on the mapping proposed by Charles Williams in the [Idiot's Guide](https://github.com/PalaeoClimateModellingUK/PalaeoClimateModellingUK.github.io/blob/main/resources.md), 
with minor modifications to ensure that the dominant PFTs assigned to each biome remain consistent with the pre-industrial (PI) 
vegetation distribution used by UKCM2.

**Table 1. Conversion from PRISM4 biomes to JULES PFT fractions**

| Pliocene (PRISM4) | JULES |
|---|---|
| Land ice | ice |
| Dry Tundra or Tundra | 40% shrubs + 60% bareground |
| Boreal Forest | 70% needleleaf + 20% C3 grass + 2.5% shrubs + 7.5% bareground |
| Temperate Forest | 75% needleleaf + 10% C3 grass + 10% shrubs + 5% bareground |
| Desert | 100% bareground |
| Grassland | 5% broadleaf + 55% C3 grass + 30% shrubs + 10% bareground |
| Savanna | 18% broadleaf + 5% C4 grass + 67% shrubs + 10% bareground |
| Warm-temperate Forest | 75% broadleaf + 10% C3 grass + 5% C4 grass + 10% shrubs |
| Tropical Forest | 100% broadleaf |
| Lake | 100% lake |

### 1. Creating the Pliocene vegetation file

The first step is to construct a Pliocene PFT distribution on the UM grid.

No dedicated script is provided for this stage. Instead, I recommend building the vegetation field interactively using whichever tools or programming language you are most comfortable with. This allows visual inspection of the resulting biome and PFT distributions at each stage of the workflow.

The procedure consists of:

1. Interpolating the PRISM4 biome reconstruction onto the UM grid (following a similar approach to that used in `step6_interp_input_dumps_noVeg_mule.py`).
2. Applying the biome-to-PFT conversion shown in Table 1.
   - For each grid cell, identify the PRISM4 biome class.
   - Replace that biome class with the corresponding mixture of JULES PFT fractions.
3. Saving the resulting PFT fractions in a NetCDF file using the same structure and dimensions as the original UM vegetation ancillary (`qrparm.veg.frac`).

The resulting file represents a first-order Pliocene vegetation reconstruction expressed in terms of JULES PFTs.

> [!TIP]
> Visually inspect each generated PFT layer before proceeding. Errors introduced during interpolation or biome conversion are often easier to identify at this stage than after the vegetation has been incorporated into the UM ancillaries.

![prism4](https://github.com/pontesgm4/HadGEM3-GC5_palaeo/blob/main/step3_make_atmos/prism4.png)
![LP](https://github.com/pontesgm4/HadGEM3-GC5_palaeo/blob/main/step3_make_atmos/LP_veg.png)
![PI](https://github.com/pontesgm4/HadGEM3-GC5_palaeo/blob/main/step3_make_atmos/PI_veg.png)

---

### 2. Creating the PI-to-LP remapping index

The PFT distribution generated in the previous step is **not yet the final vegetation field used by the model**.

Rather than directly adopting the newly generated PFT fractions, the workflow proposed here identifies the most 
similar grid cell in the pre-industrial (PI) setup and transfers the associated vegetation and soil properties from that location. 
This preserves internal consistency among vegetation-related parameters that are difficult to reconstruct independently.

The script

```text
step7_reform_veg.py
```

creates a remapping matrix:

```text
remapping_idx_veg.nc
```

which stores, for every Pliocene grid cell, the indices of the corresponding PI grid cell selected as the best match.

> [!TIP]
> Inspect the `isave` and `jsave` fields produced by the script. These fields contain the zonal and meridional source indices used in the remapping and provide a useful diagnostic to verify that neighbouring grid cells are generally mapped to nearby locations rather than to physically unrealistic source regions.

![isave_jsave](https://github.com/pontesgm4/HadGEM3-GC5_palaeo/blob/main/step3_make_atmos/isave_jsave.png)

---

### 3. Building vegetation and soil ancillaries

Once the remapping matrix (`remapping_idx_veg.nc`) has been generated, it can be used to construct the final vegetation and soil ancillaries.

The philosophy behind this step is that many soil properties are closely linked to vegetation type. Therefore, vegetation and soil fields are remapped together using the same PI-to-LP correspondence. This approach preserves physically consistent combinations of vegetation and soil characteristics inherited from the PI configuration.

The script

```text
step8a_build_input_dumps_from_veg_index.py
```

should be applied to each of the vegetation- and soil-related ancillaries listed below:

```bash
python step8a_build_input_dumps_from_veg_index.py qparm.hyddtop \
    -o qrparm.hyddtop.LP

python step8a_build_input_dumps_from_veg_index.py qparm.soil.dust \
    -o qrparm.soil.dust.LP

python step8a_build_input_dumps_from_veg_index.py qparm.soil \
    -o qrparm.soil.LP

python step8a_build_input_dumps_from_veg_index.py qparm.soil_roughness \
    -o qrparm.soil_roughness.LP

python step8a_build_input_dumps_from_veg_index.py qparm.veg.frac \
    -o qrparm.veg.frac.LP

python step8a_build_input_dumps_from_veg_index.py qparm.veg.func \
    -o qrparm.veg.func.LP
```

This procedure generates Pliocene versions of the original PI ancillaries while maintaining consistency among vegetation fractions, vegetation functional parameters, soil properties, dust parameters and other land-surface characteristics.

---

### 4. Checking PFT consistency

As a final quality-control step, verify that the PFT fractions sum to unity in every grid cell.

Run:

```bash
python step8b_PFT_check_ancil.py
```

The script checks that the sum of all PFT fractions equals 1.0 (within numerical precision) for every land grid cell.
