# Install, build, patch

## Building the kernel

LMGC90 builds with CMake (Fortran + C + Python/SWIG). Points that cost us
time:

- **Serial, not OpenMP.** The OpenMP NLGS races on many cores (we saw the
  solver diverge on a 24-core Mac Studio); every simulation is one serial
  process, parallelism is one process per core. Build with
  `-DWITH_OPENMP=OFF`, or force `OMP_NUM_THREADS=1` in every child.
- **Install prefix.** Re-running `cmake` inside a Python virtual environment
  resets the install prefix to `/usr/local`; pass `-DCMAKE_INSTALL_PREFIX`
  every time.
- **Git branch check.** The build aborts if the enclosing git branch changed
  since configuration; re-run `cmake .`.
- **HDF5.** The Fortran HDF5 reader fails on long absolute paths that h5py
  opens fine: restart files are referred to by short relative names from the
  run directory.

## Patch you need: polytopes_area_oob

`mod_DiscreteGeometry.f90` (around line 4572) sums `carea(1:nb_pts)` past a
per-polygon array on every face-to-face polyhedron contact. The result is a
random SIGBUS in `PRPRx_SelectProxTactors` — about one settle in two on a
230-grain bed. The fix is `area = sum(carea)`; physics is unchanged
(bit-identical results). Apply it before any production work.

## Python side

`pylmgc90` is imported from the build directory (`build/`), so put that on
`PYTHONPATH` ahead of anything a shell profile might add; an old build
silently wins otherwise. A per-machine config file that pins the build path
is worth having.

## Sanity checks before trusting a build

1. A box dropped on a plane with `RST_CLB`, restitution 0.5, rebounds with
   e ≈ 0.5 (we get 0.5993 → 0.60 at dt = 2e-4 and 0.4996 at 1e-4).
2. A spacecraft with no see-table row for the wall passes through it; with a
   row it stops.
3. Two fixed bodies touching with a contact rule produce `Wnn = 0` → NaN;
   without a rule they coexist.
4. A settled bed written back through `pre` reproduces every vertex to
   < 10 µm.
5. A cohesive pair holds exactly its weight (see [Contact laws](contact-laws.md)).
