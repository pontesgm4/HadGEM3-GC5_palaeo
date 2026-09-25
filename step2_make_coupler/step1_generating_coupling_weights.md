*Instructions here have copied from Zikun Ren's [workflow](https://github.com/PalaeoClimateModellingUK/paleoclimate_HadGEM3_UoB/blob/main/Coupling_weights_generation/GC3.1_coupling_weights_generation_workflow.md).*

# generate coupling weights from the mesh_mask.nc

## 1. Move mesh_mask.nc to MONSOON3
`rsync -avu mesh_mask.nc <monsoon3 host name>:<your directory to store mesh_mask>`
## 2. check out updated coupling_weight suite.
`rosie co u-dy998`
## 3. change the IGRID1 file with your mesh_mask.nc path
change the grid information file located at `suite conf > common > IGRID1` as <the path of your mesh_mask>.
## 4. run the suite
```
cd ~/roses/u-dy998
cylc vip
```
check the state of the tasks:
```
cylc tui
```
set the task `suite_info` as successful, by the action `set`.

## 5. check the output 
Output files are written to ~/cylc-run/u-dy998/share/data/20130105T0000Z     
  
After files made, renamed with    
rename DSTAREA DESTAREA *     
rename CONSERVE CONSERV *    
rename BILINEAR BILINEA *    
because the model expects different names from what the suite produces.      

Caveat:
Please note that this work flow doesn't generate `atmo_mask_fracarea_anc_ns` in ancillary form, but only output `atmo_mask_fracarea_anc_ns.nc`.
If you demand `atmo_mask_fracarea_anc_ns` in ancillary form , please use `xancil` to convert it.
