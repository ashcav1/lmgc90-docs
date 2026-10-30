# Post-processing: files, units, probes

## `getInteractions()`

A structured array, one row per contact point: `icdbdy`/`ianbdy` (1-based
body ids), `cdtac`/`antac` (contactor types), `icdtac`/`iantac` (contactor
**rank within the body**, not the global index), `status` (`noctc`, `stick`,
`slide`, `Mstck`, …), `gapTT`, `rl` (local reaction, tangential / normal /
spin components), `uc` (local frame; column 1 is the normal from antagonist
to candidate), `coor` (contact point).

**Units:** `rl` from `getInteractions()` are **forces** (N) — the Verlet
tables store impulse / H internally; `Reac_` on a body is a force too. The
sum of the normal reactions on a box is 0.89 × |`Reac_`| in a typical
multi-point contact.

## POSTPRO files (3-D rigid)

| file | columns |
|---|---|
| `SOLVER_INFORMATIONS.DAT` | time, NLGS iterations, three error measures (mean dv, quad dv, quad dvr), contacts |
| `KINETIC_ENERGY.DAT` | t, KE, KE + Edef, −Wext, −Wpot, DE |
| `VIOLATION_EVOLUTION.DAT` | t, mean gap at begin, mean gap, max gap at begin, max gap |
| `COORDINATION_NUMBER.DAT` | contacts per grain (column 1), … |

A step whose iteration count reached the budget did **not** converge; the
warning is suppressed by `DisableLogMes`, so this file is the only witness.
The three error columns are not convergence norms; do not read them as such.

## Body state

`RBDY3_GetAllBodyVector(name)` for `Coor_`, `X____`, `V____`, `Vbeg_`,
`Reac_`, `Fext_`; `RBDY3_GetAllMass()`, `RBDY3_GetAllInertia()` (principal
inertias), `RBDY3_GetBodyMatrix` (frame). `POLYR_GetVertex(i)` gives the
current global vertices of polyhedron `i` (1-based, `POLYR_GetPOLYR2BDYTY`
maps contactor to body).

## HDF5

`io_hdf5_write_last(name)` writes a single-record state usable by
`pre.readDatbox(hfile=…)`. Short relative names only.
