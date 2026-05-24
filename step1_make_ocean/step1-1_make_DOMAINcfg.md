## Interpolating bathymetry

1. Create make_ocean directory on Jasmin, which will conatin all scripts and files necessary to generate the NEMO ancils.
   
```text
cd /gws/pw/j25/past2future/users/gpontes/HadGEM3-GC5/LP_exp_bc/
mkdir make_ocean
cd make_ocean
```

2. Transfer the `domain_cfg.nc` file from `[Archer2]` to `[Jasmin]` using Globus
   The `domain_cfg.nc` file can be found on `[Archer2]` at:

   ```text
   cylc-run/u-do332/run1/share/data/etc/ancil_nemo/
   ```

3. run script make_topog.py. Modify input/ouput files. This scripts interpolates the palaeo grid onto the NEMO ORCA grid. This script is not yet working because ideally we want to use conservative remapping and the `domain_cfg.nc` does not provide the grid corners.
   But the script works for Bilinear interpolation.
   To start, I've proceeded with the interpolated bathymetry (also bilinear) file from Charlie.
   Bathymetry from Charlie can be found at

   ```text
   cp /gws/ssde/j25a/ncas_climate/vol2/users/cwilliams2011/data1_for_use.d/pliod.d/new_plio.d/build.d/ancils.d/bathy.d/plio_enh_topo_v1.0_new.nc_regridded_bilinear.nc .
   ```
   
4. run adjust_topog.py. This gets rid of the isolated ocean grid cells (lakes and narrow straits), and makes a few manual adjustments.
   How to use it: after running the scripts identifies grid cell that isolated grid cells that need to befilled with land value (zeroed) and grid cells in whcih the depth is shallow than 40 m. This ouputs the file straits.txt which contain a list of those grid cells.
   Copy and paste the list of cells from the straits.txt script into the adjust_topog.py nad run it again. Re-do this untill the generated straits.txt file is empty.
   Even if the straits.nc file is empty you might want to do further manual adjustments. Look for locations that could potentially generate numerical instabilities in the model (i.e., too narrow seaways, enclosed bays, unsmoothed coastlines).
   Further use the adjust_topog.py script to make aditional adjustments at these locations.
   To do this, the most pratical way is to open the adjusted bathymetry file in ncview. Set the range to 0-5m so that the land-sea mask is highlighted. Make anote of the grid cell inidices you want to change (either fill with on land or 'dig' to make it ocean) and add to adjust_topog.py script.
   Re-do this till you're happy with the land-sea mask/bathymetry.
   You know have your final bathymetry file.

---

## Generating the domain_cfg.nc and mesh_mask.nc files for the new bathymetry

5. On your `[Archer2]` work/ directory create a similar directory as that of `[Jasmin]`:

  ```text
  cd /work/n02/n02/gpontes/LP_exp_bc
  mkdir make_ocean
  cd make_ocean
  ```

6. fcm checkout NEMO util scripts from MetOffice (you need a MORS account)

  ```text
  fcm co https://code.metoffice.gov.uk/svn/nemo/utils utils
  cd utils
  ```

7. load the demanded modules:

  ```text
   module use /work/y07/shared/umshared/moci/modules/modules
   module load GC5-PrgEnv
  ```

8. update the archived environment set for ARCHER2:

  ```text
   cp build/arch/NOC/arch-X86_ARCHER2-Cray.fcm build/arch/NOC/arch-X86_ARCHER2-Cray-updated.fcm
  ```

make the following changes:

  ```text
   35c35
   < %XIOS_HOME           /work/n01/shared/acc/xios-trunk
   > %XIOS_HOME           /work/n01/shared/nemo/xios-trunk
  ```

9. We now need to run the script `tools/maketools`, but, in this configuration of NEMO utils the `arch` and `mk` directories were placed into the `build` direcotry without updating the `tools/maketools` script. So place them back into the `utils` directory:

   ```text
   mv build/arch .
   mv build/mk .
  ```
 
Then run maketools:

  ```text
   cd tools
   ./maketools -m X86_ARCHER2-Cray-updated -n DOMAINcfg
  ```

Thisscripts cretes the `DOMAINcfg` directory under `./tools/`.

10. update the `./tools/DOMAINcfg/namelist_cfg` file with file from Zikun. Also cpy the original ORCA grid file from Zikun:

   ```text
   cd tools/DOMAINcfg/
   cp /work/n02/n02/an25872/NEMO_SRC/utils/tools/DOMAINcfg/namelist_cfg .
   cp cp /work/n02/n02/an25872/NEMO_SRC/utils/tools/DOMAINcfg/eORCA1_coordinates_nc4.nc_from_MR.nc .
  ```

Update the Bathymetry input file in namelist_cfg with your onw final interpolated bathymetry file from Jasmin (use Globus to transfer the file from Jasmin to this directory). Update the name of the Bathymetry variable in the script if needed.
    
11. 

   ```text

  ```


