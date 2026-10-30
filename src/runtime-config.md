## Configuration

---

## `SetDimension(dim, mhyp)`

**What it does:** Sets the problem dimension and mechanical hypothesis for the
entire simulation.

**Arguments:**
- `dim` (int): spatial dimension — `2` or `3`
- `mhyp` (int): mechanical hypothesis
  - `0` — rigid bodies in 3D (our use case)
  - `1` — plane strain (2D)
  - `2` — plane stress (2D)
  - `3` — axisymmetric (2D)

**Returns:** None

**As used in our code:**
```python
chipy.SetDimension(3, 0)
```

**Notes:** Must be called before `ReadDatbox`.  For all 3D rigid body simulations
use `dim=3, mhyp=0`.

---

## `TimeEvolution_SetTimeStep(dt)`

**What it does:** Sets the time step size `dt` (seconds) used in every subsequent
time-integration step.

**Arguments:**
- `dt` (float, seconds): time step size

**Returns:** None

**As used in our code:**
```python
chipy.TimeEvolution_SetTimeStep(1e-3)
```

**Notes:** Implemented in `chipy/lmgc90.py` as a direct SWIG wrapper (not in
macro.py).  Set before `Integrator_InitTheta`.

---

## `Integrator_InitTheta(theta)`

**What it does:** Initialises the theta time integrator.  `theta=0.5` gives the
Crank-Nicolson midpoint rule (second-order, energy-conserving for rigid bodies).

**Arguments:**
- `theta` (float, 0–1): integration parameter; `0.5` = midpoint, `1.0` = fully
  implicit (more damping)

**Returns:** None

**As used in our code:**
```python
chipy.Integrator_InitTheta(0.5)
```

**Notes:** Implemented in `chipy/lmgc90.py` as a SWIG wrapper.  Call after
`TimeEvolution_SetTimeStep`.

---

