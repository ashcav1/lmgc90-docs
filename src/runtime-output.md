## Output Functions

---

## `WriteOut(nsteps=1)`

**What it does:** Writes the current state (DOF + contact reactions) to the HDF5
file (or ASCII files if HDF5 is not initialised), but only on steps where
`current_step % nsteps == 0`.

**Arguments:**
- `nsteps` (int, default `1`): write every `nsteps` steps

**Returns:** None

**As used in our code:**
```python
chipy.WriteOut(500)   # write every 500 steps
```

**Notes:**
- Writes to the file specified in `InitHDF5`.
- The `nsteps` check is done internally — no need for an `if` guard in the loop.
- Each write creates one "record" in the HDF5 file; `ReadIni(record=-1)` reads
  the last record.

---

## `WriteDisplayFiles(freq=1)`

**What it does:** Writes VTK display files to `DISPLAY/` every `freq` steps.

**Arguments:**
- `freq` (int): write every `freq` steps

**Returns:** None

**As used in our code:**
```python
chipy.WriteDisplayFiles(50)   # write every 50 steps
```

**Notes:** Frame files are named with sequential integers continuing from the
`restart` index given to `OpenDisplayFiles`.

---

## `WritePostproFiles()`

**What it does:** Writes post-processing output to `POSTPRO/` for this step.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.WritePostproFiles()
```

---

## `WriteLast(filename=None)`

**What it does:** Writes a "last-state" snapshot — the most recent step's DOF,
velocities, and contact reactions — for crash recovery.

**Arguments:**
- `filename` (str or None): if `InitHDF5` was called with a filename, and a
  filename is provided here, writes to that HDF5 file as a special "last"
  record.  Also writes a separate compacted copy.  If `None`, falls back to
  ASCII `.LAST` files.

**Returns:** None

**As used in our code:**
```python
chipy.WriteLast("last.h5")              # inside loop, every 50 steps
chipy.WriteLast("last.h5")              # final write after loop
```

**Notes:**
- `last.h5` is the crash-recovery file; `ReadIni(record=-1, h5_file="last.h5")`
  reads its last state.
- Also called immediately after `RBDY3_PutBodyVector` to persist the initial drop
  velocity before any time steps run.
- Writing frequently (every 50 steps) is cheap compared to the cost of losing
  hours of simulation on a crash.

---

## `ReadIni(record=0, h5_file='')`

**What it does:** Reads a saved state (DOF + contact reactions) into the running
simulation.  Used for restarts.

**Arguments:**
- `record` (int): which record to read:
  - `< 1` (e.g. `-1`) — reads the **last** record in the HDF5 file
  - `0` — reads ASCII `.INI` files (legacy)
  - `> 0` — reads that specific record index from HDF5
- `h5_file` (str): path to the HDF5 file (e.g. `"poly_settle.h5"`).  If empty,
  uses ASCII files.

**Returns:** None

**As used in our code:**
```python
# Robust wrapper used in command_impact.py:
chipy.ReadIni(record=-1, h5_file="last.h5")
chipy.ReadIni(record=-1, h5_file="poly_drop.h5")
chipy.ReadIni(record=-1, h5_file="poly_settle.h5")
```

**Notes:**
- Must be called **after** `ReadDatbox` (bodies must already exist) and **before**
  `OpenDisplayFiles` (so frame numbering works correctly).
- Restores: DOF (positions/velocities), VlocRloc (contact reactions, if
  tact_behav defined), GPV (if MAILx deformable — not our case).
- In `command_impact.py`, a `try/except` wrapper tries `record=-1` then
  `record=max(nb_record-1, 0)` as a fallback.
- Priority order for restart input: `last.h5` > `poly_drop.h5` > `poly_settle.h5`.

---

