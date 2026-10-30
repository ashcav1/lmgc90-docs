## Body Creation

---

## `rigidPlan(axe1, axe2, axe3, center, material, model, color)`

**What it does:** Creates a rigid planar body (a flat rectangular wall/floor) as
an avatar.

**Arguments:**
- `axe1` (float): half-length along first axis
- `axe2` (float): half-length along second axis
- `axe3` (float): half-thickness
- `center` (list of 3 floats): `[x, y, z]` position of the plan centre
- `material`: a `material` instance
- `model`: a `model` instance
- `color` (str, exactly 5 characters): display colour label (e.g. `"VERTx"` for
  green walls)

**Returns:** `avatar` instance (a rigid plan body)

**As used in our code:**
```python
left  = pre.rigidPlan(axe1=20, axe2=10, axe3=0.01, center=[0, 20, 0],
                      material=tdur, model=mod, color="VERTx")
down  = pre.rigidPlan(axe1=200, axe2=200, axe3=0.01, center=[0., 0., 0.],
                      material=tdur, model=mod, color="VERTx")
```

**Notes:**
- Color string must be exactly 5 characters (padded with `x` if needed).
- The plan is a flat box; its orientation is set by subsequent `rotate()` calls.
- In the settle box: floor (`down`) is horizontal; side walls are vertical after
  rotating ±90° around their edge nodes.

---

## `rigidPolyhedron(model, material, center, color, generation_type, nb_vertices, radius, vertices=None, faces=None, tol=0.0, number=None)`

**What it does:** Creates a single rigid convex polyhedral body.

**Arguments:**
- `model`: a `model` instance
- `material`: a `material` instance
- `center` (list of 3 floats): `[x, y, z]` position of the polyhedron centre
- `color` (str, 5 chars): display colour (e.g. `"BLUEx"`)
- `generation_type` (str): how to generate vertices:
  - `"random"` — randomly distributes exactly `nb_vertices` vertices on a sphere
    of the given `radius`; vertex count is exact
  - `"regular"` — icosphere/cube subdivision; vertex count is approximate
  - `"vertices"` — provide explicit `vertices` + `faces` arrays
  - `"full"` / `"bevel"` — other algorithmic modes
- `nb_vertices` (int): number of vertices (for `"random"` or `"regular"`)
- `radius` (float): circumscribed sphere radius
- `vertices` (array or None): explicit vertex coordinates (only for
  `generation_type="vertices"`)
- `faces` (array or None): face connectivity (only for `generation_type="vertices"`)
- `tol` (float, default `0.0`): geometric tolerance
- `number` (int or None): optional explicit body number

**Returns:** `avatar` instance (a rigid polyhedron body)

**As used in our code:**
```python
grain = pre.rigidPolyhedron(
    model=mod, material=tdur, center=c.tolist(), color="BLUEx",
    generation_type="random",
    nb_vertices=int(nbv),      # nbv=18
    vertices=None, faces=None,
    radius=float(r),
    tol=0.0, number=None
)
bodies += grain

impactor = pre.rigidPolyhedron(
    model=mod, material=tdur, center=[0.0, 0.0, 18.0], color="BLUEx",
    generation_type="random",
    nb_vertices=10,
    vertices=None, faces=None,
    radius=5.0,
    tol=0.0, number=None
)
```

**Notes:**
- `"random"` gives exactly `nb_vertices` vertices; `"regular"` may give a
  different count than requested (depends on icosphere subdivision level).
- Use `"random"` for bed particles to guarantee consistent geometry.
- `center` must be a Python `list`, not a numpy array — use `.tolist()` when
  pulling from numpy.
- For the impactor: `nb_vertices=10` gives a rougher shape; increase for a
  smoother sphere-like impactor.

---

