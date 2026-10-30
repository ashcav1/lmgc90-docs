## Material and Model Creation

---

## `material(name, materialType, density=None, **kwargs)`

**What it does:** Defines a material with a given constitutive law.

**Arguments:**
- `name` (str, **exactly 5 characters**): material identifier used in DAT files
  (e.g. `"TDURx"`, `"MOUxx"`)
- `materialType` (str): constitutive law — `"RIGID"` for rigid bodies, `"ELAS"`
  for linear elastic, etc.
- `density` (float, kg/m³): mass density

**Returns:** `material` instance

**As used in our code:**
```python
tdur = pre.material(name="TDURx", materialType="RIGID", density=1000.0)
pdur = pre.material(name="MOUxx", materialType="RIGID", density=100.0)
```

**Notes:**
- Name **must be exactly 5 characters** — will cause silent errors or format
  mismatches if wrong length.
- Density is stored as `Umas=` in `BULK_BEHAV.DAT`.
- For our simulation: walls use `TDURx` (density 1000), bed particles use
  `MOUxx` (density 100).  Both are `RIGID`.

---

## `model(name, physics, element, dimension=None, **kwargs)`

**What it does:** Defines a finite-element or rigid-body model type.

**Arguments:**
- `name` (str, **exactly 5 characters**): model identifier (e.g. `"rigid"`)
- `physics` (str): physics module — `"MECAx"` for mechanical
- `element` (str): element type — `"Rxx3D"` for 3D rigid body
- `dimension` (int): spatial dimension — `3` for 3D

**Returns:** `model` instance

**As used in our code:**
```python
mod = pre.model(name="rigid", physics="MECAx", element="Rxx3D", dimension=3)
```

**Notes:**
- Name must be exactly 5 characters.
- `"Rxx3D"` is the only element type for 3D rigid bodies.
- `"MECAx"` is the standard mechanical physics module.

---

