## Building vegetation and soil input dumps

PRISM4 provides a vegetation reconstruction grouped in 8 major biomes. The UM13.8/JULES vegetation field, on the other hand, 
is based on plant functional types(PFTs), consisting of 9 types of leaf, grasses and ground that are combined to build up a biome. 
Each grid cell includes the proportion of each PFTs present (or absent) in that grid cell. For this reason, we reconstruct each biome in the model as a mix of PFTs (Table 1). For example, in the Late Pliocene simulation, each grid cell of PMISM4 boreal forests is represented in the
UKCM2 as 70% of needleleaf, 20% C3 grass, 2.5 shrubs and 7.5% bareground (see full conversion below). 
This table is mostly similar to the one provided by [Charles Williams in his Idiot's Guide](https://github.com/PalaeoClimateModellingUK/PalaeoClimateModellingUK.github.io/blob/main/resources.md);
however with some minor changes to make sure the dominant PFTs of each biome are consistent with the PI distribution.


| Pliocene (PRISM4) | JULES |
|---|---|
| Land ice | ice |
| Dry Tundra or Tundra | 40% shrubs + 60% bareground |
| Boreal Forest | 70% needleleaf + 20% C3 grass + 2.5% shrubs + 7.5% bareground |
| Temperate forest | 75% needleleaf + 10% C3 grass + 10% shrubs + 5% bareground |
| Desert | 100% bareground |
| Grassland | 5% broadleaf + 55% C3 grass + 30% shrubs + 10% bareground |
| Savanna | 18% broadleaf + 5% C4 grass + 67% shrubs + 10% bareground |
| Warm-temperate forest | 75% broadleaf + 10% C3 grass + 5% C4 grass + 10% shrubs |
| Tropical forest | 100% broadleaf |
| Lake | 100% lake |

### 1. Creating Pliocene vegetation file

Use whatever tools/language is are more comfortable with to generate your vegetation file. I do not provide a code for as I recommend you do it interactively to visually inspect changes in all biomes/PFTs distributions as you build them up.

For me it basically consisted of:

- Interpolating the PRISM biome reconstruction onto the UM grid (similarly to the way it is done in `step6_interp_input_dumps_noVeg_mule.py` script in the step before this)
- Applying the conversion table to the interpolated `qrparm.veg.frac.LP`
  - This basically identify grid cells belonging to each specific PRISM mega biome and replacing them with PFTs as indicated in the conversion table
- Save a Pliocene PFT distribution in netCDF and in the same structure as the original UM dump.
 
This gives the following result:

[figure to be added]

### 2. Creating PI-to-LP index matrix

Nonetheless, this is not your final vegetation field to be added to the input dump. 
To ensure consistent with the PI setup, the nearest grid cell in the PI setup with similar
characteristics to the generated 'Pliocene PFTs distribution' is adopted in the Late Pliocene setup.

For this, the script

`step7_reform_veg.py`

generates an index matrix (`remapping_idx_veg.nc`) with the corresponding PI grid cell for each Pliocene grid cell.

> [!TIP]
> It is recommended to check isave and jsave to inspect that they're xonally and meridionally consistently, respectively. So that, confirming that the nearest grid cell is indeed being used.
> [figure to be added]

### 3. Building soil and input dumps

After obtaining the `remapping_idx_veg.nc` file, you're ready to generate the new input dumps for vegetation and soil fields.
Here I modify soil fields in the same way as vegetation as most soil properties are linked to a certain vegetation type. This procedure therefore modifies all vegetation (i.e., canopy heigh and leaf area index) and soil (i.e., dust) properties while ensuring consistency the PI setup.

To do this, apply the `step8a_build_input_dumps_from_veg_index.py` to the following input dumps:




