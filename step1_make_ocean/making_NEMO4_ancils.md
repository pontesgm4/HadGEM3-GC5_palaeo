# Interpolating PRISM Pliocene bathymetry onto the NEMO (HadGEM-GC5) ocean grid

## Overview

To build the Pliocene boundary conditions for **HadGEM-GC5 / NEMO4**, the reconstructed PRISM bathymetry is horizontally interpolated onto the native NEMO ORCA grid defined in `domain_cfg.nc`.

The interpolated field is used as the starting point for:
- updating the ocean bathymetry (`bathy_metry`),
- defining the land–sea mask,
- regenerating vertical levels (`bottom_level` / partial cells),
- manual regional adjustments (gateways, shelves, isolated basins).

---

## Target grid

Ideally we want to use conservative remapping, but I couldn't do that because the NEMO `domain_cfg.nc` file does not contain the grid corners (see below). As such, I used the interpolation from PRISM to NEMO grid done by Charlie.

The `domain_cfg.nc` can be found on Archer2, after running any suite, on 

`[Archer2] ~/cylc-run/<suite-id>/runX/share/data/etc/ancil_nemo/`


### Why conservative remapping was not used

Conservative remapping was initially tested (following previous workflows with MOM/ACCESS-ESM1.5), but is not used here for the NEMO ORCA grid.

#### 1. ORCA corner geometry is not explicit

`xesmf` conservative remapping requires cell corners:

```text
lon_b(y+1,x+1)
lat_b(y+1,x+1)
```

or equivalent vertex bounds.

NEMO `domain_cfg.nc` provides:
- T-point centres (`glamt`, `gphit`)
- staggered F points (`glamf`, `gphif`)

but does **not** provide an explicit `(ny+1,nx+1)` corner grid directly usable by ESMF.

#### 2. ORCA tripolar geometry is difficult for ESMF

The ORCA grid includes:
- Arctic fold / tripolar geometry
- longitude discontinuities
- highly distorted polar cells

These can cause ESMF conservative remapping to fail (e.g. `ESMC_FieldRegridStore rc=506`) because invalid polygons may be generated.

---

## Post-processing

After interpolation:

1. convert negative elevations to land (`depth <= 0 → land`)
2. inspect regional gateways manually
3. remove isolated inland seas if needed

---

## Summary

For the Pliocene HadGEM-GC5 ocean setup:

| Step | Method |
|---|---:|
| Horizontal interpolation | `xESMF` |
| Target grid | `glamt / gphit` |
| Preferred method | `nearest_s2d` (or `bilinear`) |
| Conservative remapping | tested but not used |
| Manual QC | required |

This approach prioritises robust coastline/gateway representation over strict area conservation and is generally more suitable for paleo bathymetry reconstruction on the NEMO ORCA grid.
