## Building vegetation and soil input dumps

PRISM4 provides a vegetation reconstruction grouped in 8 major biomes. The UM13.8/JULES vegetation field, on the other hand, is based on plant functional types(PFTs), consisting of 9 types of leaf, grasses and ground that are combined to build up a biome. Each grid cell includes the proportion of each PFTs present (or absent) in that grid cell. For this reason, we reconstruct each biome in the model as a mix of PFTs (Table 1). For example, in
the Late Pliocene simulation, each grid cell of PMISM4 boreal forests is represented in the
UKCM2 as 70% of needleleaf, 20% C3 grass, 2.5 shrubs and 7.5% bareground (see full conversion below). 
This table was mostly similar to the one provided by [Charles Williams in Idiot's Guide](https://github.com/PalaeoClimateModellingUK/PalaeoClimateModellingUK.github.io/blob/main/resources.md);
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

