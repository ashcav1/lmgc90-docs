## Body Introspection & Manipulation

---

## `RBDY3_GetNbRBDY3()`

**What it does:** Returns the total number of 3D rigid bodies (`RBDY3`) currently
loaded.

**Arguments:** None

**Returns:** `int` — total body count

**As used in our code:**
```python
imp_id = chipy.RBDY3_GetNbRBDY3()   # impactor is the last created body
```

**Notes:**
- Body IDs in LMGC90 are **1-based**.  The last body has ID equal to this
  return value.
- Impactor is always the last body created in `gen_sample.py`, so its ID is
  reliably `RBDY3_GetNbRBDY3()`.

---

## `RBDY3_PutBodyVector(vect_name, body_id, values)`

**What it does:** Sets a named state vector for a specific rigid body.  Used to
inject velocity (or other state) at runtime.

**Arguments:**
- `vect_name` (str, exactly 5 characters): name of the vector to set:
  - `"V____"` — velocity at current (end-of-step) time level
  - `"Vbeg_"` — velocity at beginning-of-step time level
  - `"Vfree"` — free velocity (unconstrained, used by ExSolver as RHS)
  - `"X____"` / `"Xbeg_"` — cumulative displacement (current / begin-of-step)
  - **`"Coor_"` is GET-only.** Putting it triggers a Fortran fatal error
    (`RBDY3::put_vector unknown id: Coor_`) that aborts the whole process —
    uncatchable from Python. Verified empirically 2026-08-30 (v4 harness).
    To teleport a body: read `Coor_`, add the delta to BOTH `X____` and
    `Xbeg_`, then re-read `Coor_` to verify (see `milani/simcore.teleport`).
- `body_id` (int, 1-based): body index
- `values` (list of 6 floats): `[vx, vy, vz, wx, wy, wz]` for velocity vectors

**Returns:** None

**As used in our code:**
```python
imp_id = chipy.RBDY3_GetNbRBDY3()
v6 = [0.01, 0.0, -4.0,   0.0, 0.0, 0.7]   # [vx,vy,vz, wx,wy,wz]
chipy.RBDY3_PutBodyVector("V____", imp_id, v6)
chipy.RBDY3_PutBodyVector("Vbeg_", imp_id, v6)
chipy.RBDY3_PutBodyVector("Vfree", imp_id, v6)
```

**Notes:**
- **All three vector names must be set** when injecting initial velocity.  The
  theta integrator uses both beginning-of-step (`Vbeg_`) and end-of-step (`V____`)
  values; `Vfree` seeds the contact solver.  Setting only one leaves the others at
  the settled (zero) value, causing a discontinuity.
- Call this **after** `ReadIni` (so the settled positions are loaded) and
  **before** the time loop.
- Use the flag-file pattern to avoid re-applying on restart:
  ```python
  if not Path("drop_initialized.flag").is_file():
      ...set velocity...
      chipy.WriteLast("last.h5")
      Path("drop_initialized.flag").write_text("done")
  ```
- SWIG wrapper; passes directly to `_lmgc90.so`.

---

## `RBDY3_GetBodyVector(vect_name, body_id)`

**What it does:** Reads a named state vector for a specific rigid body.

**Arguments:**
- `vect_name` (str, exactly 5 characters): same names as `PutBodyVector`
- `body_id` (int, 1-based): body index

**Returns:** list/array of 6 floats — `[vx, vy, vz, wx, wy, wz]` for velocity,
or `[x, y, z, phi, theta, psi]` for coordinates

**As used in our code:**
```python
# Not yet used, but needed for post-processing:
pos = chipy.RBDY3_GetBodyVector("Coor_", imp_id)
vel = chipy.RBDY3_GetBodyVector("V____", imp_id)
```

**Notes:**
- Useful for monitoring impactor position/velocity during the impact run.
- Same 1-based indexing as `PutBodyVector`.

---

