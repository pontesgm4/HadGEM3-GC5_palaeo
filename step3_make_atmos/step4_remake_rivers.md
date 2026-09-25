## Remaking river routing for a palaeo setup

The UM river routing scheme requires a two ancillary fields that describe how runoff is transported across the land surface and eventually reaches the ocean. These ancillaries depend strongly on the underlying topography and land-sea mask and therefore need to be regenerated whenever palaeogeography is modified.

The workflow below generates a new river routing ancillary (`qrparm.rivseq.LP`) consistent with the Pliocene topography and land-sea mask.

The overall procedure consists of:

1. Interpolating the atmospheric topography onto the 1° × 1° river-routing grid.
2. Removing artificial topographic depressions ("sinks") that would trap runoff.
3. Interpolating the palaeo land-sea mask onto the same 1° × 1° grid.
4. Computing flow directions and river routing information.
5. Building a new UM river-routing ancillary.

---

### 1. Interpolate atmospheric topography onto the river-routing grid

The UM river routing scheme operates on a **1° × 1° grid**, rather than the native atmospheric grid. Therefore, the first step is to interpolate the palaeo atmospheric topography onto this river-routing grid.

Run:

```bash
python step9a_river_atmos_topog_trip.py
```

#### Input files

| File | Description |
|--------|-------------|
| `../make_coupler/grids.nc` | Target river-routing grid definition |
| `LP_topog_atmos_antarc.nc` | Pliocene atmospheric topography |

#### Output file

| File | Description |
|--------|-------------|
| `topog_new_1x1.nc` | Pliocene topography interpolated onto the 1° × 1° river-routing grid |

#### Notes

- The interpolation only changes the grid resolution and does not modify the topography itself.
- The resulting file will be used in the next step to derive flow directions.

---

### 2. Remove topographic sinks

Real-world topography contains enclosed depressions that can trap water. Some are physically realistic, but many are artefacts introduced by interpolation or discretisation.

Before generating river pathways, these depressions must be removed to ensure that runoff can flow continuously towards the ocean.

This step uses the **RichDEM** package, which applies a depression-filling algorithm to the topography.

#### Installing RichDEM

It is recommended to install RichDEM in a dedicated environment:

```bash
conda create -n richdem python=3.11
conda activate richdem
pip install richdem
```

#### Run the script

```bash
python step9b_river_fill_depression_richdem.py
```

#### Input file

| File | Description |
|--------|-------------|
| `topog_new_1x1.nc` | Interpolated 1° × 1° topography |

#### Output file

| File | Description |
|--------|-------------|
| `topog_no_sink.nc` | Topography after depression filling |

#### Notes

- This step is critical. Failure to remove sinks can lead to unrealistic river networks and internally drained basins.
- The depression-filling algorithm modifies the minimum number of grid cells required to ensure continuous drainage.

After completing the step:

```bash
conda deactivate
```

---

### 3. Interpolate the land-sea mask onto the river-routing grid

The river routing calculations require a land-sea mask defined on the same 1° × 1° grid as the topography.

Run:

```bash
python step9c_river_lsm1x1.py
```

#### Input file

| File | Description |
|--------|-------------|
| `LP_lsm.nc` | Pliocene land-sea mask on the atmospheric grid |

#### Output file

| File | Description |
|--------|-------------|
| `lsm_1x1.nc` | Pliocene land-sea mask on the 1° × 1° river-routing grid |

#### Notes

- Ensure that the resulting land-sea mask is consistent with the interpolated topography.
- Coastal artefacts introduced during interpolation can affect downstream routing calculations.

---

### 4. Generate flow directions and river-routing fields

Once the sink-free topography and land-sea mask are available on the same grid, river pathways can be calculated.

Run:

```bash
python step9d_river_downslope_trip_create_ancil_v2.py
```

#### Input files

| File | Description |
|--------|-------------|
| `topog_no_sink.nc` | Sink-free topography |
| `lsm_1x1.nc` | Pliocene land-sea mask |
| `../../orig_ancils/orig_ancil_um/qrparm.rivseq` | Original UM river-routing ancillary used as template |

#### Output files

| File | Description |
|--------|-------------|
| `river_routing_LP.nc` | Intermediate river-routing diagnostics |
| `qrparm.rivseq.LP` | New Pliocene river-routing ancillary |

#### Notes

- The script calculates downslope flow directions for each land grid cell.
- River pathways are traced until they reach the ocean.
- The original UM ancillary is used as a template to ensure that the final output has the correct format and metadata required by the UM.

> [!TIP]
> Plot the generated river network and compare it with the modern routing scheme. Large drainage basins (e.g., Amazon, Congo, Mississippi, Nile, and Arctic rivers) provide useful diagnostics for identifying interpolation errors or unrealistic flow pathways.

---

### Final outputs

After completing all steps, the main files produced are:

| File | Description |
|--------|-------------|
| `topog_new_1x1.nc` | Topography interpolated to the river-routing grid |
| `topog_no_sink.nc` | Sink-free topography |
| `lsm_1x1.nc` | Land-sea mask on the river-routing grid |
| `river_routing_LP.nc` | Intermediate river-routing diagnostics |
| `qrparm.rivseq.LP` | Final Pliocene river-routing ancillary for UM |

The file `qrparm.rivseq.LP` is the ancillary that should be used in the palaeo UM configuration.


