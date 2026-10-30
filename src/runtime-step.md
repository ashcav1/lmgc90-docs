## Per-Step Time Loop

All functions below must be called in the exact order shown.

---

## `ComputeMass()`

**What it does:** Computes and caches mass and inertia tensors for all bodies.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.ComputeMass()
```

**Notes:** Call once after `OpenPostproFiles()`, before the time loop.

---

## `IncrementStep()`

**What it does:** Advances the simulation clock by one `dt`.  Internally calls
`overall_CleanWriteOutFlags()` (resets write-flag state), then
`TimeEvolution_IncrementStep`, then `RBDY3_IncrementStep`.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.IncrementStep()
```

**Notes:** First call in every time-step body.

---

## `ComputeFext()`

**What it does:** Computes external forces (gravity, applied loads) on all bodies.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.ComputeFext()
```

**Notes:** Must follow `IncrementStep()`.

---

## `ComputeBulk()`

**What it does:** Computes bulk (deformable body) internal forces.  For purely
rigid simulations this is a near-no-op but must still be called.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.ComputeBulk()
```

---

## `ComputeFreeVelocity()`

**What it does:** Computes the "free" (unconstrained) velocity of each body, i.e.
what the velocity would be at the end of the step with only external forces and no
contact reactions.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.ComputeFreeVelocity()
```

**Notes:** The output feeds `ExSolver` as the right-hand side of the contact
problem.

---

## `SelectProxTactors(freq_detec=1)`

**What it does:** Updates the neighbour list (broad-phase contact detection) and
identifies proximal body pairs that may be in contact.

**Arguments:**
- `freq_detec` (int, default `1`): recompute neighbourhood every `freq_detec`
  steps.  Values >1 skip detection on intermediate steps (faster but less safe).

**Returns:** None

**As used in our code:**
```python
chipy.SelectProxTactors()
```

**Notes:**
- For POLYR-POLYR contacts, CpCundall detection must be enabled via
  `PRPRx_UseCpCundallDetection` before calling this.
- Skipping detection steps (`freq_detec>1`) is only safe when bodies move slowly
  relative to `dt`.

---

## `RecupRloc(tol=None)`

**What it does:** Recovers the previous-step contact reaction forces (`Rloc`)
from the contact law, to use as a warm-start for the solver.

**Arguments:**
- `tol` (float or None, default `None`): if provided, uses position-based
  recovery (`RecupRlocByPosition`) for CSASp and CSPRx contacts, matching
  reactions by proximity with the given tolerance.  If `None`, uses the default
  index-based recovery.

**Returns:** None

**As used in our code:**
```python
chipy.RecupRloc(5e-2)
```

**Notes:**
- Position-based recovery (`tol` given) is safer on restart or when the contact
  topology changes between steps.
- For PRPRx (POLYR-POLYR) contacts, use `PRPRx_SetTolRecupRloc(tol)` separately
  if position-based recovery is needed — the `tol` argument to `RecupRloc` does
  NOT activate position recovery for PRPRx.
- Always call after `SelectProxTactors()`.

---

## `ExSolver(stype, norm, conv, relax, gsit1, gsit2)`

**What it does:** Runs the Non-Linear Gauss-Seidel (NLGS) contact solver to find
contact reactions consistent with the contact laws and kinematic constraints.

**Arguments:**
- `stype` (str, **exactly 30 characters** including trailing spaces):
  - `"Stored_Delassus_Loops         "` — assembles and stores the Delassus
    operator; faster per NLGS iteration but uses more memory; preferred for
    rigid bodies
  - `"Exchange Local Global         "` — recomputes Delassus on-the-fly; more
    memory-efficient but slower per iteration
- `norm` (str, **exactly 5 characters** including trailing space):
  - `"Quad "` — quadratic norm (RMS); recommended default
  - `"Maxm "` — max norm
  - `"QM/16"` — quadratic norm divided by 16
- `conv` (float): convergence tolerance (e.g. `1e-4`)
- `relax` (float): relaxation parameter; `1.0` = no relaxation
- `gsit1` (int): outer-loop max iterations — maximum number of convergence
  checks before the solver stops regardless of convergence
- `gsit2` (int): inner-loop iterations — number of NLGS sweeps between each
  convergence check

**Returns:** None

**As used in our code:**
```python
chipy.ExSolver(
    "Stored_Delassus_Loops         ",
    "Quad ",
    1.666e-4,   # conv
    1.0,        # relax
    50,         # gsit1
    1000        # gsit2
)
```

**Notes:**
- **String padding is critical**: `stype` must be exactly 30 chars and `norm`
  exactly 5 chars.  Wrong lengths cause a silent Fortran lookup failure.
- Total solver iterations ≈ `gsit1 × gsit2` in the worst case.
- For settling (dense packing, many contacts), increase `gsit2` to `1000` or
  more.  For impact (sparse contacts), `500` may suffice.
- In 3D, dispatches to `nlgs_3D_ExSolver`.

---

## `UpdateTactBehav()`

**What it does:** Updates the state of contact laws (e.g. history variables,
cohesion/debonding state) after the solver has converged.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.UpdateTactBehav()
```

**Notes:** Call immediately after `ExSolver`.

---

## `StockRloc()`

**What it does:** Stores the current contact reactions (`Rloc`) for use as a
warm-start in the next step's `RecupRloc`.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.StockRloc()
```

**Notes:** Must follow `UpdateTactBehav()`.

---

## `ComputeDof()`

**What it does:** Computes the new degrees of freedom (positions and velocities)
for all bodies by integrating the contact-resolved velocities over `dt`.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.ComputeDof()
```

---

## `UpdateStep()`

**What it does:** Finalises the time step — advances internal time-step counters
and promotes "end-of-step" state to "beginning-of-step" for the next iteration.

**Arguments:** None

**Returns:** None

**As used in our code:**
```python
chipy.UpdateStep()
```

**Notes:** Always the last call in the time-step body before output functions.

---

