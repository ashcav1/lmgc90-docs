# How a run is organised

## Bodies, contactors, colors

A **body** (`RBDY3` for 3-D rigids) carries mass, inertia and a frame. A
**contactor** is the geometry attached to it that can touch things:
`POLYR` (convex polyhedron), `PLANx` (a thick plane), `SPHER`, `CYLND`, …
A body can carry several contactors (a spacecraft bus plus two wings as one
rigid body: three `POLYR` on one `RBDY3`, each with its own volume, inertia
and shift so the rigid properties stay exact).

Every contactor has a **color**, a 5-character tag. Colors are how contact
rules are addressed: a **see table** row says "candidate contactor type
`POLYR` of color `BOULD` on body type `RBDY3` may touch antagonist `PLANx`
of color `FLOOR` with law `rockx` within alert distance 0.01 m". No row, no
contact, no error.

## Laws

A **tact_behav** is a contact law with parameters: `RST_CLB` (restitution +
Coulomb), `GAP_SGR_CLB` (gap-level Signorini + Coulomb), `IQS_MOHR_DS_CLB`
(Mohr–Coulomb cohesion), … — see [Contact laws](contact-laws.md) for what
they really do.

## The time step

NSCD is implicit at the velocity level. A step (`θ`-method, θ = 0.5 by
default) computes free velocities from external forces, detects candidate
contacts within the alert distance, solves all contact impulses at once by
Gauss–Seidel sweeps (NLGS) under each pair's law, then updates velocities
and positions. There is no stiffness, no overlap spring, no stability limit
from contact stiffness: the time step is set by how far bodies move per step
relative to the alert distance and by how well the NLGS converges.

## Restart, display, postpro

- `WriteLast` / `ReadIni` (HDF5) store and restore a state (positions,
  velocities, contact reactions). We prefer writing a fresh deck from a
  settled state through `pre` when the set of bodies changes (cropping).
- `WriteDisplayFiles` writes ParaView `.vtu`/`.pvd` under `DISPLAY/`.
- `WritePostproFiles` writes the text tables selected in
  `postpro_commands` under `POSTPRO/` (solver information, energies,
  violation, coordination number).
- `getInteractions()` returns the current contact table as a structured
  array: body ids, contactor ranks, status, gap, local reactions, frame.

## Processes

One simulation = one process. Nothing in the kernel can be re-initialised
in-process; a crash is contained to the run; parallelism is many processes.
