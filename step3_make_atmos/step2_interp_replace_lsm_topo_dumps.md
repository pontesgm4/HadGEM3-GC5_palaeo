## Applying new fields to input dumps

This step will replace all generated orography fields into their respective input dumps. However, some required fields have not been created, simply because I don't know how to create them. So, for these I have simply interpolated the PI fields onto the Pliocene land-sea mask.

Script: `step6_interp_input_dumps_noVeg_mule.py`

**Inputs**

`lsm_pi_file` -> `do332a.da22791201_00.nc` (Here I take this from a PI restart dump)

`lsm_file` -> `LP_lsm.nc` (generated in step1)

`orog_file` -> `LP_topog_atmos_antarc.nc` (final orog file after smoothing Antarctica generated in step1)

`stddev_file` -> `LP_orog_std.nc` (generated in step1)

`gradient_file` -> `LP_xx_yy_scaled.nc` (generated in step1)

`netcdf_landfrac` -> `../make_coupler/atmo_mask_fracarea_anc_ns.nc` (generated in step2_make_coupler)

`mask_file` -> `../make_coupler/um_masks.nc` (generated in step2_make_coupler)

**How to run**

`step6` is an arg parser script and must be run as following:

```text
python step6_interp_input_dumps_noVeg_mule.py qparm.ash -o qrparm.ash.LP
python step6_interp_input_dumps_noVeg_mule.py qparm.landfrac -o qrparm.landfrac.LP
python step6_interp_input_dumps_noVeg_mule.py qparm.mask -o qrparm.mask.LP
python step6_interp_input_dumps_noVeg_mule.py qparm.orog_mean -o qrparm.ororg_mean.LP
python step6_interp_input_dumps_noVeg_mule.py qparm.orog_radiation_parameters -o qrparm.orog_radiation_parameters.LP
python step6_interp_input_dumps_noVeg_mule.py qparm.orog_wavedrag -o qrparm.ororg_wavedrag.LP
```
