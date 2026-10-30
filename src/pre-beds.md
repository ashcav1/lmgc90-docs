## Bed Generation

---

## `granulo_Random(nb, r_min, r_max)`

**What it does:** Generates a 1D array of particle radii drawn from a uniform
distribution in number (not volume) between `r_min` and `r_max`.

**Arguments:**
- `nb` (int): number of particles to generate
- `r_min` (float): minimum radius
- `r_max` (float): maximum radius

**Returns:** 1D numpy array of `nb` floats

**As used in our code:**
```python
radii = pre.granulo_Random(5000, 0.2, 1.0)
```

**Notes:**
- Uniform in number ≠ uniform in volume; large particles are under-represented
  by volume.  For a volume-uniform distribution, use `granulo_Uniform` or
  `granulo_TwoSizes` if available.
- The returned array may have length < `nb` after `depositInBox3D` truncates
  the tail (see the corrected contract below — it is a prefix, not a filter).

---

## `depositInBox3D(radii, lx, ly, lz, d_radii=None, d_coor=None, seed=None, logmes=False)`

**What it does:** Places spheres of the given radii inside a box of
`lx × ly × lz`. **Corrected 2026-08-30 (source autopsy):** this is NOT pure
Python — the Python layer is a thin marshaller over compiled Fortran
(`mod_deposit3D.f90`, via the ChiPy binding `deposit3D_InContainer`).

**KNOWN INSTABILITY (root-caused 2026-08-30):** the Fortran algorithm's only
stochastic placement path exists for grains 2–4; from grain 5 onward only
deterministic tangency solvers remain, and a failed trip retries the SAME
radius against the SAME references — so one failure right after the random
phase freezes the deposit at 4–5 grains forever (measured: ~35–45% of seeds,
even in roomy boxes). Acceptance also hinges on `norm1 > 0.D0` where norm1
is analytically zero — a round-off coin flip. Past 100 placed grains, the
first failed trip aborts the ENTIRE deposit. Two internal `stop` calls
(>110 reference particles; >2000 Verlet neighbours) hard-kill the Python
process — beware strongly polydisperse beds. **v4 does not call this
function; use `milani/deposit.py` (deterministic, graceful capping, same
contract).** With `logmes=True` the only visible failure signal is the
console line `Depot plante a : <n>`.

**Arguments:**
- `radii` (array): radii from `granulo_Random` or similar
- `lx` (float): box x-dimension (box spans `[0, lx]`)
- `ly` (float): box y-dimension (box spans `[0, ly]`)
- `lz` (float): box z-dimension (box spans `[0, lz]`)
- `d_radii`, `d_coor` (arrays, optional): pre-deposited particles to pack on
  top of (this is the supported way to build layered beds — NOT two separate
  calls, which deposit into independent empty boxes)
- `seed` (list of ints, optional): seeds the Fortran RNG. Must have at least
  the compiler's `random_seed` size (convention: `list(range(33))`); a SHORT
  list is a Fortran fatal abort. `np.random.seed` has NO effect on this RNG.
  Unseeded calls are wall-clock-seeded (non-deterministic across runs);
  a seeded failing configuration fails REPRODUCIBLY.
- `logmes` (bool): print deposit diagnostics

**Returns:** tuple `(nb_placed, coor, placed_radii)`
- `nb_placed` (int): number of particles successfully placed
- `coor` (array, shape `[nb_placed, 3]`): particle centres, with x in `[0,lx]`,
  y in `[0,ly]`, z in `[0,lz]`
- `placed_radii` (array, shape `[nb_placed]`): **exactly `radii[:nb_placed]`
  — an input-order PREFIX (tail truncation), never a filtered subset or a
  permutation.** Consequence: feeding `placed_radii` back into a retry
  silently caps that retry at the previous count (the 4-grain trap).
  `zip(coor, placed_radii)` is always correct.

**As used in our code:**
```python
nb_comp, coor, radii = pre.depositInBox3D(radii, LX=35, LY=35, LZ=15)

# Centre bed in simulation coordinates:
coor[:, 0] -= 0.5 * LX    # shift x: [0,35] -> [-17.5, 17.5]
coor[:, 1] -= 0.5 * LY    # shift y: [0,35] -> [-17.5, 17.5]
coor[:, 2] += 0.1          # lift off floor slightly
```

**Notes:**
- **Box origin is at (0,0,0)** — the output coordinates are all positive.  You
  must translate to your simulation coordinate system after the call (subtract
  half-dimensions to centre).
- `nb_comp` may be less than the requested `NB_PARTICLES` if not all particles
  fit; print a warning if this happens.
- Results are cached to `.npy` files so this expensive call runs only once per
  geometry change (`CLEAR_CACHE = True` forces regeneration).

---

