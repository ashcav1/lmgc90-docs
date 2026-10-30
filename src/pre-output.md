## Output

---

## `writeDatbox(dim, mats, mods, bodies, tacts, svs, post=None, gravy=None, datbox_path='DATBOX')`

**What it does:** Writes all pre-processing data to a DATBOX directory as a set
of `.DAT` and `.INI` files consumed by chipy at runtime.

**Arguments:**
- `dim` (int): spatial dimension (`3`)
- `mats`: `materials` container
- `mods`: `models` container
- `bodies`: `avatars` container
- `tacts`: `tact_behavs` container
- `svs`: `see_tables` container
- `post` (optional): `postpro_commands` container
- `gravy` (list of 3 floats, optional): gravity vector `[gx, gy, gz]` written
  to `BULK_BEHAV.DAT`; if not provided, no gravity section is written
- `datbox_path` (str, default `'DATBOX'`): path to the output directory
  (created if it doesn't exist)

**Returns:** None

**As used in our code:**
```python
# Write DROP datbox first (impactor free):
pre.writeDatbox(3, mats, mods, bodies, tacts, svs,
                post=post, gravy=[0., 0., -98.1],
                datbox_path="DATBOX_DROP")

# Then freeze impactor and write SETTLE datbox:
impactor.imposeDrivenDof(component=[1,2,3,4,5,6], dofty="vlocy")
pre.writeDatbox(3, mats, mods, bodies, tacts, svs,
                post=post, gravy=[0., 0., -98.1],
                datbox_path="DATBOX_SETTLE")
```

**Notes:**
- `gravy` values end up as `grv1`, `grv2`, `grv3` in `BULK_BEHAV.DAT`.
- Gravity at impact time is overridden by regex-patching `BULK_BEHAV.DAT` in
  `command_impact.py` — so the value here only matters for the settle phase.
- Write ORDER matters: `DATBOX_DROP` first (impactor unfrozen), then
  `DATBOX_SETTLE` (impactor frozen via `imposeDrivenDof`).  `imposeDrivenDof`
  is irreversible — it cannot be undone after the call.
- The generated files include: `BODIES.DAT`, `BULK_BEHAV.DAT`, `TACT_BEHAV.DAT`,
  `DEM_BODIES.INI` (initial positions), `DRIVEN_DOF.DAT` (driven constraints),
  `CONTACT_3D.DAT`, and possibly `POSTPRO.DAT`.

---

## `visuAvatars(bodies)`

**What it does:** Opens an interactive VTK visualisation window showing all
bodies in the `avatars` container.

**Arguments:**
- `bodies`: `avatars` container

**Returns:** None (opens a window)

**As used in our code:**
```python
try:
    pre.visuAvatars(bodies)
except Exception:
    pass
```

**Notes:**
- Requires a display (fails headlessly); always wrap in `try/except` for
  non-interactive or remote use.
- Useful for sanity-checking the scene geometry before running the simulation.
- Blocking call — the script pauses until the VTK window is closed.

---
