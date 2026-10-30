## I/O Setup

---

## `ReadDatbox(deformable=True)`

**What it does:** Reads all `.DAT` files from the `DATBOX/` directory (which must
be a symlink or real directory containing the pre-processor output).  Loads
bodies, materials, models, contact tables, and contact behaviours.

**Arguments:**
- `deformable` (bool, default `True`): if `False`, skips `ReadModels` /
  `LoadModels` — required for rigid-only simulations to avoid FEM-related errors

**Returns:** None

**As used in our code:**
```python
chipy.ReadDatbox(deformable=False)
```

**Notes:**
- Must be called after `SetDimension`, `TimeEvolution_SetTimeStep`,
  `Integrator_InitTheta`.
- Gravity patch (`BULK_BEHAV.DAT` edit) must happen **before** this call.
- After reading, LMGC90 also writes paranoid copies of the DAT files to `OUTBOX/`
  for audit purposes.
- `DATBOX` must be a directory (or symlink to a directory) in the current working
  directory.

---

## `InitHDF5(filename='')`

**What it does:** Switches output from ASCII files to a single HDF5 file for all
subsequent `WriteOut()` and `WriteLast()` calls.

**Arguments:**
- `filename` (str): path to the HDF5 output file (e.g. `"poly_settle.h5"`)

**Returns:** None

**As used in our code:**
```python
chipy.InitHDF5("poly_settle.h5")
chipy.InitHDF5("poly_drop.h5")
```

**Notes:**
- If the file already exists, LMGC90 calls `h5repack` on it before opening, which
  compacts free space from previous runs.
- Call **after** `ReadDatbox` and **before** `OpenDisplayFiles`.
- Setting a filename here is what enables `WriteLast(filename)` to write to a
  specific file.

---

## `OpenDisplayFiles(restart=1)`

**What it does:** Opens VTK display output files in `DISPLAY/`.  The `restart`
argument controls frame numbering — frame files will start at index `restart`.

**Arguments:**
- `restart` (int, default `1`): first frame index to write; set to
  `max_existing_index + 1` on restart to avoid overwriting previous frames

**Returns:** None

**As used in our code:**
```python
start_display = next_display_index("DISPLAY")  # scans DISPLAY/ for max index
chipy.OpenDisplayFiles(start_display)
```

**Notes:**
- Internally sets `wdf = restart - 1` as the write-file offset.
- On a fresh run, pass `1` (default) so files are `001`, `002`, etc.
- On restart, scan `DISPLAY/` for the highest trailing integer to get the correct
  continuation index — see `next_display_index()` helper in `command_impact.py`.

---

## `OpenPostproFiles()`

**What it does:** Opens post-processing output files in `POSTPRO/`.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.OpenPostproFiles()
```

**Notes:** Call after `OpenDisplayFiles`.  Writes outputs only if
`postpro_commands` were defined in pre-processing.

---

## `CloseDisplayFiles()`

**What it does:** Closes VTK display output files.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.CloseDisplayFiles()
```

**Notes:** In the current build, `macro.py` implements this as `pass` — it is a
no-op.  Still call it for forward compatibility.

---

## `ClosePostproFiles()`

**What it does:** Closes post-processing output files.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.ClosePostproFiles()
```

**Notes:** Call before `Finalize()`.

---

