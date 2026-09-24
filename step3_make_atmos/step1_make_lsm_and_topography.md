## Creating the atmospheric land-sea mask and topographic fields

The Pliocene paleogeography modifies the distribution of land and ocean relative to the present day. The Unified Model therefore requires an updated land-sea mask consistent with the reconstructed Pliocene geography.

The coupled setup provides a land-fraction ancillary (lsmask) containing fractional land coverage for each atmospheric grid cell. Several UM components, including river-routing and land-surface processes, additionally require a binary land-sea mask.

This step generates:

- a binary land-sea mask (LP_lsm.nc);

---

### 1. Generate the binary land-sea mask

Run:

python step1_make_lsm.py

Input:

atmo_mask_fracarea_anc_ns.nc

The script:

reads the fractional land mask (lsmask);
replaces missing values with zero;
applies a land-fraction threshold of:
0.01
classifies grid cells as:
land (1) if land fraction ≥ 0.01;
ocean (0) otherwise;
writes the resulting binary land-sea mask.

Output:

LP_lsm.nc

The resulting file contains:

variable: lsm
values: 0 = ocean, 1 = land

and can be used directly by downstream UM ancillary-generation tools.

2. Generate the corrected land-fraction ancillary

The source ancillary required a north-south orientation correction before being used by the UM workflow.

The same script therefore also creates a corrected version of the original land-fraction file.

The script:

copies all dimensions, variables and metadata from the source file;
applies the north-south latitude correction;
updates the latitude coordinate;
preserves the original fractional land values;
appends provenance information to the file history.

Output:

atmo_mask_fracarea_anc_ns_flipped.nc

This file retains fractional land coverage and is used by subsequent ancillary-generation steps.

Outputs

After completion the workflow produces:

LP_lsm.nc
atmo_mask_fracarea_anc_ns_flipped.nc

where:

File	Purpose
LP_lsm.nc	Binary land-sea mask used by UM ancillary tools
atmo_mask_fracarea_anc_ns_flipped.nc	Corrected fractional land-mask ancillary consistent with Pliocene geography
Notes
A threshold of 1% land fraction is used when constructing the binary mask.
Missing land-fraction values are converted to ocean (0).
The binary mask is derived entirely from the reconstructed Pliocene land fraction field.
This step should be rerun whenever the Pliocene land-sea distribution is modified.
