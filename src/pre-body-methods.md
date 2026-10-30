## Body Methods

---

## `body.translate(dx=0., dy=0., dz=0.)`

**What it does:** Translates all nodes of the body by the given offsets.

**Arguments:**
- `dx`, `dy`, `dz` (float): displacement in x, y, z

**Returns:** None (modifies in-place)

**As used in our code:**
```python
left.translate(dy=-1.0, dz=2.0)
right.translate(dy= 1.0, dz=2.0)
rear.translate(dx=-1.0, dz=0.5)
front.translate(dx= 1.0, dz=0.5)
```

**Notes:**
- Called after `addAvatar` to position walls correctly.
- All angles/coordinates are in the same units as `center` (model units).

---

## `body.rotate(description='Euler', phi=0., theta=0., psi=0., alpha=0., axis=None, center=None)`

**What it does:** Rotates all nodes of the body around a given center point.

**Arguments:**
- `description` (str): rotation parameterisation:
  - `"Euler"` — use `phi`, `theta`, `psi` (Euler ZXZ angles, **all in radians**)
  - `"axis"` — use `axis` (3-vector) + `alpha` (angle in radians)
- `phi`, `theta`, `psi` (float, radians): Euler angles (for `description="Euler"`)
- `alpha` (float, radians): rotation angle (for `description="axis"`)
- `axis` (list of 3 floats): rotation axis unit vector (for `description="axis"`)
- `center` (list/array of 3 floats, or None): rotation pivot point; if None,
  rotates around the body's own origin

**Returns:** None (modifies in-place)

**As used in our code:**
```python
left.rotate(theta=-0.5*math.pi, center=left.nodes[1].coor)
right.rotate(theta= 0.5*math.pi, center=right.nodes[1].coor)
rear.rotate(description="axis", alpha= 0.5*math.pi,
            axis=[0.,1.,0.], center=rear.nodes[1].coor)
front.rotate(description="axis", alpha=-0.5*math.pi,
             axis=[0.,1.,0.], center=front.nodes[1].coor)
```

**Notes:**
- All angles **must be in radians**.
- `center=body.nodes[1].coor` rotates around the body's first node, which is
  a convenient pivot for wall panels (rotates the panel up from horizontal to
  vertical).
- Use `"axis"` description for rotations around an arbitrary axis (walls
  perpendicular to x vs. y).

---

## `body.imposeDrivenDof(group='all', component=1, dofty='temp')`

**What it does:** Prescribes driven (fixed or velocity-controlled) boundary
conditions on specified DOFs of the body.  **Irreversible** — once set in pre,
cannot be unset for the same DATBOX.

**Arguments:**
- `group` (str, default `'all'`): node group to apply to; `'all'` = all nodes
- `component` (int or list of int): DOF component(s) to constrain:
  - `[1,2,3]` — translation DOFs (x, y, z)
  - `[4,5,6]` — rotation DOFs
  - `[1,2,3,4,5,6]` — all 6 DOFs (fully frozen rigid body)
- `dofty` (str): type of driven DOF:
  - `"vlocy"` — velocity-driven (zero velocity = fixed)
  - `"force"` — force-driven
  - `"temp"` — temperature (for thermal DOFs)
  - `"flux"` — heat flux

**Returns:** None (sets `self.drvDof = True`)

**As used in our code:**
```python
# Walls — fully fixed:
for w in (left, right, down, rear, front):
    w.imposeDrivenDof(component=[1,2,3,4,5,6], dofty="vlocy")

# Impactor frozen for SETTLE datbox (written after DROP):
impactor.imposeDrivenDof(component=[1,2,3,4,5,6], dofty="vlocy")
```

**Notes:**
- **Irreversible in pre** — you cannot remove a driven DOF after calling this.
  This is why `gen_sample.py` writes `DATBOX_DROP` (impactor free) **first**,
  then freezes the impactor and writes `DATBOX_SETTLE`.
- `dofty="vlocy"` with zero velocity = fixed position.  The velocity remains
  zero throughout the simulation.
- The impactor's DOFs are released at the chipy level by simply not loading the
  SETTLE datbox — instead, `command_impact.py` loads `DATBOX_DROP` where the
  impactor was never frozen.

---

## `avatars.addAvatar(av)`

**What it does:** Adds a body to the `avatars` container, assigning it a
sequential internal number.

**Arguments:**
- `av`: an `avatar` instance (rigid body, wall, etc.)

**Returns:** None

**As used in our code:**
```python
bodies.addAvatar(left)
bodies.addAvatar(right)
bodies.addAvatar(down)
```

**Notes:**
- Assigns `av.number = len(self)` (0-based) before appending.
- `bodies += grain` is equivalent and more convenient for single bodies or lists.

---

