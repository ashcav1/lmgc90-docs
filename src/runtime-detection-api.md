## CpCundall Contact Detection

---

## `PRPRx_SetCpCundallBox(xmin, xmax, ymin, ymax, zmin, zmax)`

**What it does:** Sets the spatial domain for CpCundall hash-based contact
detection.  Only body pairs within this box are checked for POLYR-POLYR contact.

**Arguments:**
- `xmin`, `xmax` (float): x-extent of detection domain
- `ymin`, `ymax` (float): y-extent of detection domain
- `zmin`, `zmax` (float): z-extent of detection domain

**Returns:** None

**As used in our code:**
```python
# command_impact.py uses hasattr check for version safety:
if hasattr(chipy, "PRPRx_SetCpCundallBox"):
    chipy.PRPRx_SetCpCundallBox(-60., 60., -60., 60., -20., 80.)
elif hasattr(chipy, "PRPRx_SetDomain"):
    chipy.PRPRx_SetDomain(-60., 60., -60., 60., -20., 80.)
```

**Notes:**
- Must be called **before** `PRPRx_UseCpCundallDetection`.
- **Not present in the current SWIG wrapper** (`chipy/lmgc90.py`); use `hasattr()`
  before calling to avoid `AttributeError` on versions that lack it.
- `PRPRx_SetDomain` is an alternate name for the same function on some builds.
- Make the domain generously larger than the actual simulation box to avoid
  missing fast-moving bodies.

---

## `PRPRx_UseCpCundallDetection(CPC_NB)`

**What it does:** Enables the CpCundall spatial-hash algorithm for POLYR-POLYR
broad-phase contact detection.  Required whenever two polyhedra may contact each
other.

**Arguments:**
- `CPC_NB` (int): size of the CpCundall hash table.  Should be larger than the
  expected maximum number of active contact pairs.

**Returns:** None

**As used in our code:**
```python
chipy.PRPRx_UseCpCundallDetection(200000)
```

**Notes:**
- Implemented in `chipy/lmgc90.py` (not SWIG-generated; has a real Python body
  in that file).
- Call **after** `ReadDatbox` and after `PRPRx_SetCpCundallBox` (domain must be
  set first).
- Without this, POLYR-POLYR contacts are silently missed.

---

## `PRPRx_LowSizeArrayPolyr(size)`

**What it does:** Sets the initial allocation size for the per-polyhedron contact
candidate array.  A lower value saves memory; LMGC90 will resize dynamically if
needed.

**Arguments:**
- `size` (int): initial candidate-list size per polyhedron

**Returns:** None

**As used in our code:**
```python
chipy.PRPRx_LowSizeArrayPolyr(100)
```

**Notes:**
- Implemented in `chipy/lmgc90.py`.
- `100` is a safe default for particles with ~18 vertices.  Increase for very
  non-convex shapes with many potential contacts.
- Call after `PRPRx_UseCpCundallDetection`.

---
