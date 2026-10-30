## Initialization & Teardown

---

## `Initialize()`

**What it does:** Bootstraps the LMGC90 Fortran core — initialises timer,
parameter-ID mapping, and all subsystem state.  Must be the very first chipy
call.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.Initialize()
```

**Notes:** Must be called before any other chipy function.  Paired with
`Finalize()` at end of script.

---

## `Finalize()`

**What it does:** Releases all Fortran-side memory and shuts down the LMGC90
core cleanly.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.Finalize()
```

**Notes:** Call after `CloseDisplayFiles()` and `ClosePostproFiles()`.  Omitting
it can leave HDF5 files corrupted or partially written.

---

## `checkDirectories()`

**What it does:** Creates `OUTBOX/`, `POSTPRO/`, and `DISPLAY/` subdirectories
if they do not already exist.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.checkDirectories()
```

**Notes:** Call immediately after `Initialize()`.  Harmless if directories already
exist.

---

## `utilities_DisableLogMes()`

**What it does:** Suppresses verbose per-step log output from the Fortran core to
stdout.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.utilities_DisableLogMes()
```

**Notes:** Highly recommended for long runs; without it, terminal output is
overwhelming.  Does not suppress Python-level `print()` calls.

---

