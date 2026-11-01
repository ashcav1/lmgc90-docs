# Decision log (append-only)

Format: date — decision — evidence. Gates D0–D10 are defined in the plan
artifact (rev 7). A gate is CLOSED only when its criterion has data.

- 2026-08-30 — **Workflow**: lab_mac is the canonical branch; Claude codes +
  verifies locally on the personal Mac, pushes; Ashish pulls and runs on the
  lab Mac; small results return via `lab.py sync` (reports/ + context/).
- 2026-08-30 — **Serial solver everywhere**: the OpenMP build's parallel
  NLGS diverges on the 24-core Studio (diagnosed earlier); every simulation
  child process forces OMP_NUM_THREADS=1. Local dev build compiled with
  -DWITH_OPENMP=OFF.
- 2026-08-30 — **D0 novelty (closed by rev-5 lit review)**: no published
  grain-resolved CubeSat-on-Dimorphos DEM; ISAE-SUPAERO PhD (Chrono) in
  progress. Re-scan at study start.
- 2026-08-30 — **Scope (rev 7, Ashish)**: transfer-function instrument OUT
  of Paper 1; F2BP loop is the centerpiece; cohesion stays.
- 2026-08-30 — **Teleport semantics (empirical, harness)**: RBDY3
  put_vector REJECTS 'Coor_' (Fortran fatal abort). Repositioning is done by
  shifting X____ and Xbeg_ by the delta and verifying via Coor_ readback.
  The api_enriched.md claim that Coor_ is puttable is WRONG (correction
  pending).
- 2026-08-30 — **Restitution sanity (harness)**: sphere-on-plane RST_CLB
  measures e = 0.5993 vs configured 0.6 at dt = 2e-4 — the contact law
  behaves as configured in the serial build.
- 2026-08-30 — **Upstream pre bug worked around**: rigidPolyhedron(seed=int)
  crashes (NameError in getRandomPolyhedronVertices). Convention: seed numpy
  globally, pass seed=None (v3 did the same).
- 2026-08-30 — **Runbook location**: context/RUNBOOK.md is the tracked
  original; a tracked file inside runs/ breaks `git pull` on the lab Mac
  (runs is a symlink there — reproduced in review). doctor writes an
  untracked copy to runs/README_LABMAC.md.
- 2026-08-30 — **Impact restarts read settle/last.h5** (the converged +
  micro-g final state), via short LOCAL symlink names: LMGC90's Fortran h5
  reader fails on long absolute paths that h5py opens fine (rediscovered;
  the reason v3 symlinked into cwd).
- 2026-08-30 — **Cohesion activation mechanism** (pre-D10): IQS_MOHR_DS_CLB
  bonds only at NStep==1 with virgin statuses; cohesive impact runs reset
  the step counter (TimeEvolution_SetInitialStep(0)) and skip RecupRloc on
  step 1. Without this, cohesive == cohesionless silently (verified against
  mod_nlgs_3D.f90). D10's harness scene still owed before cohesive
  study.
- 2026-08-30 — **Gen determinism**: depositInBox3D accepts a seed (audited)
  + global np.random seeded at generate() start → regen is byte-identical
  (verified by registry sha). A registry FINGERPRINT travels settle→impact
  and mismatches are hard errors.
- 2026-08-30 — **Engine SIGINT semantics**: handler (not KeyboardInterrupt)
  sets STOP + terminates children; drill-verified 2 s exit, resume clean,
  no catalog dupes. PRPRx_SetCpCundallBox does not exist in this build (the
  v3 hasattr call was inert too) — removed.
- 2026-08-30 — **Deposit root cause (source autopsy, mod_deposit3D.f90)**:
  the upstream algorithm's ONLY stochastic placement path exists for grains
  2-4 (`i_list <= n_init+3`, line ~1124); from grain 5 on, only
  deterministic tangency solvers remain, and a failed trip changes NOTHING
  (same radius, same references) — so one failure right after the random
  phase freezes the count at 4-5 forever. Acceptance additionally hinges on
  `norm1 > 0.D0` where norm1 is analytically zero (float round-off coin
  flip, ~line 595), which is why the failure is seed-dependent and bimodal.
  Past 100 grains, the first failed trip `exit`s the WHOLE deposit. Two
  `stop` paths (>110 references, >2000 Verlet neighbours) hard-kill the
  Python process — a real hazard for strongly polydisperse beds. Return
  contract verified: radii_out is an exact PREFIX of the input (truncation,
  not filtering); np.random.seed never touches the Fortran RNG (only the
  `seed` list argument does; unseeded = wall-clock). lz interacts only as
  the benign "container full" stop — the 4-grain collapse is not
  height-related.
- 2026-08-30 — **Legacy v3 layering bug found during the audit** (NOT
  fixed; v3 is frozen legacy): scripts/v3/gen.py lines ~185-188 rebind the
  deposit results, so the bottom layer is DISCARDED and the "top" layer is
  deposited into an empty box — that script's two-layer path never actually
  layered. (Laney's layered runs used gen_shape.py's two-deposit z-shift
  approach, which did layer.) Any old result traced to v3 gen.py's layered
  path has no bottom layer.
- 2026-08-30 — **Deposit replaced (v4 only)**: pre.depositInBox3D bails
  catastrophically on ~35-45% of seeds regardless of fit (measured: 4/200
  placed in a roomy 5x5x4 m box on 7/20 seeds; 1-26/26 spread on the smoke
  box). v4 uses milani/deposit.py — deterministic drop-and-roll, identical
  return contract (nb, corner-origin coor, radii in input order),
  overlap-free by construction, graceful capping when the box is genuinely
  full (600 asked into a ~236-capacity box -> 224-251 placed, stable,
  never a collapse). Verified: 20/20 seeds full placement in all feasible
  regimes, byte-identical determinism, harness checks added, 8/8 gen seeds
  first-try, full smoke + campaign green. v3 scripts still call upstream
  (legacy, unchanged); the LMGC90 source tree is untouched (it is
  gitignored, so a fix there could never reach the lab Mac anyway).
- 2026-08-30 — **Walls sized from the ACTUAL loose deposit column**, not
  nominal depth: the 2x-headroom deposit could start grains above the wall
  tops -> spill during settle (the observed "deposit gets screwy with
  walls"). generate() now runs geometry first, then sizes walls from the
  true column top + impactor headroom. Deposit box and wall interior share
  scene.bed width by construction; the depositor keeps grains at
  x,y in [r, L-r], so nothing ever starts inside a wall.
- 2026-10-28 — **Clean rebuild (Ashish's go-ahead)**: v2/v3/v4 scripts removed
  from the tree (tag `legacy/lab_mac-final`; other branches tagged
  `legacy/laney`, `legacy/ashish` before deletion). New code: `src/lmgcx`
  (LMGC90 made safe, project-agnostic) + `src/milani` (pipeline) + `lab.py`.
  Old local data moved out of the project to `../LMGC90_legacy_2026-10-28/`.
- 2026-10-28 — **Per-run composed decks replace restart-with-ghost**: beds are
  settled once WITHOUT an impactor; each impact deck is rebuilt by pre
  (`readDatbox(hfile=…)` → `updateReferenceConfig()` → patch/recolour/freeze →
  add the box → `writeDatbox`). Evidence: round-trip test reproduces 232
  settled grains to 5.5e-9 m. Patch = patch + fixed ring + floor, no walls.
- 2026-10-28 — **Frozen bodies get no frozen–frozen or frozen–floor rule**:
  source shows two touching fixed bodies give Wnn = 0 and 0/0 in the solver
  (mod_nlgs_3D.f90 :2574/:2591/:3048). Pinned by the frozen_pairs scene
  (zero motion, no NaN).
- 2026-10-28 — **Walls invisible to the spacecraft = no craft–wall rule**:
  verified in source for PRPRx/PRPLx/SPPLx/SPPRx/SPSPx (missing see-row =
  never detected, no error) and by the invisible_walls scene (craft passes,
  grain stops). The SPPLx "Unknown visibility table" faterr is dead code.
- 2026-10-28 — **All grains are polyhedra (gate D4 closed toward realism)**:
  LMGC90's SPPRx detector (sphere vs polyhedron) projects toward the
  polyhedron centre and misses approaching contacts; in tests sphere small stones
  sank a full radius into boulders (LMGC gap −43 mm; exact hull distance
  confirmed the penetration). Polyhedral small stones: worst exact penetration
  0.9–2.8 mm. Also: LMGC90 has no rolling-resistance law, so sphere small stones
  would roll freely anyway.
- 2026-10-28 — **LMGC90 memory bug found and fixed (polytopes_area_oob)**:
  `DiscreteGeometry::polytopes_intersection_wc` does
  `area = sum(carea(1:nb_pts))` on an array with one entry per polygon →
  out-of-bounds read on every face-to-face contact → random SIGBUS in
  PRPRx_SelectProxTactors (7/7 macOS crash reports identical; ~1 settle in 2
  crashed on a 230-grain bed). Fix `area = sum(carea)` (value unused
  afterwards). Verified: 6/6 runs at the crash-prone setting succeed,
  bit-identical to pre-fix successful runs. Shipped as data
  (`src/milani/lmgc90_patches.py`, `lab.py patch-lmgc90`); simulations refuse
  to start on an unpatched build. Also found: re-running cmake in a
  venv-configured LMGC90 build silently resets the install prefix to
  /usr/local (patch tool passes it explicitly), and LMGC90's build aborts
  when the enclosing git branch changed (patch tool re-confirms with cmake).
- 2026-10-28 — **Spin frame**: V____(4:6) is body-frame (source + spin_frame
  scene: a global spin given at random attitude reads back exactly, the cube
  turns 0.10000 rad about z in 0.1 s). GetBodyMatrix rows = body axes.
  Outcomes store spin in global, body and azimuth-canonical frames.
- 2026-10-28 — **v4 wall overlap bug**: walls were centred on the deposit-box
  faces, so edge grains started ~1 cm inside the wall plates (measured −9 mm
  at step 1). Walls now sit outside the box (inner faces on the box faces).
- 2026-10-28 — **Settling**: boulders deposited lying flat (long axis
  horizontal) as oblate spheroids (exact non-overlap, ~half the loose
  column); small stones deposited onto the settled boulder height map (a real
  thin top layer); speeds capped during deposition (RBDY3_PartialDamping), then an
  uncapped release phase must also come to rest. Restitution sanity: box on
  floor e = 0.4996 vs rstn 0.5.
- 2026-10-28 — **Settle law is GAP_SGR_CLB, not RST_CLB**: the smoke bed
  failed QA with a 15.8 mm boulder–boulder overlap (exact SAT check of the
  two polyhedra: 15.5 mm, so real, not a gap-reporting artefact). On the
  micro bed, RST_CLB with zero restitution left 4.9–7.4 mm overlaps
  whatever the dt (5e-4 to 2e-3), tolerance or alert distance (×3, ×6):
  a velocity-level law stops approach but never pushes an overlap back
  out, so dense polyhedral packings drift. The gap-level GAP_SGR_CLB on the
  same deposit: worst gap −3e-20 m at dt 1e-3 and 2e-3, at similar cost
  (3,300 vs 2,100 steps at dt 2e-3). IQS_CLB (gap-level only for open
  gaps) was worse: −10.6 mm, so the drift is existing contacts sinking,
  which only a gap-level law corrects. pre whitelists GAP_SGR_CLB for
  deformable contactors only; the 3D NLGS treats it generically
  (mod_nlgs_3D.f90:1164-1194), so `lmgcx.deck.import_pre()` admits it for
  rigid pairs. **Small-stone pairs stay on RST_CLB** (`[settle] fines_law`): on
  the overlap-free larger half, gap-law small stones drained through the boulder
  pores for thousands of steps at the speed limit (smoke stage 2 never came
  to rest; 6× the per-step cost). What a THIN "thin top layer" should be — on top,
  or partly in the pores — is gate D5. Bed QA now grades boulder pairs
  (warn 0.02 / fail 0.10 of the boulder semi-axis) apart from pairs with
  small stones (0.15 / 0.5, as before). Impacts keep RST_CLB (restitution is
  physics there). Production settle dt raised to 2e-3 (overlap no longer
  depends on dt; speed limit × dt = 0.6 mm per step).
- 2026-10-28 — **Patch check is kinematic, not force-based**: before the box
  touched anything, frozen-ring grains in a smoke deck carried contact
  impulses ~1e5× gravity scale (0.05 N·s on one contact). That is self-stress:
  rigid grains wedged between immovable ones admit any balanced squeeze, and
  NLGS picks one arbitrarily per step. `crop_shell_load` measured that noise
  (5,070 at patch 0.4 m, 7,109 at 0.6 m — growing with the patch). It is kept
  in the record as information. QA now grades `crop_edge_speed`: the fastest
  free grain in the patch's outer band ÷ impact speed, warn-only (0.25) until
  gate D12 calibrates a fail limit. Patch retries (×1.5) trigger on left_patch
  and first contact on fixed grains only.
- 2026-10-28 — **Rest at asteroid gravity**: at g = 5e-5 a box left at ~1 mm/s
  hops for tens of seconds per flight, so "10 s of unbroken slow contact"
  never happened (smoke impact A ran to the 120 s cap: 406 contact episodes,
  e_speed 0.013). The detector now also counts slow time between touches
  after first contact (speed < rest_speed and spin < rest_spin); a box that
  slow can't clear the terrain, so "detached" can't apply. A rerun then
  skidded at 3 mm/s, spin 0.012 rad/s: friction at g = 5e-5 (μg ≈ 2.5e-5
  m/s²) takes ~2 min to stop it, so every slow landing would run to the time
  cap (600 s, ~30 CPU-min in study). New class **`captured`**: no
  detachment, and the box's fastest point (v + ω × circumradius) stays below
  `capture_fraction` (0.1) × impact speed for rest_time_s. The exit state at
  that moment is recorded, so the creep is not lost.
- 2026-10-29 — **First study night lost to settling; beds rebuilt as
  layers.** All 16 windows were still settling at the 11 h cutoff (7,800
  steps = 15.6 s of settling, 99 % of grains still at the speed limit): the
  deposit was a 6 m loose column for a 1.2 m layer, and each step cost 20–30 s
  at 29,000 contacts × 2,000–4,000 NLGS iterations with 16 settles sharing
  the Mac. Root cause: grains were deposited as the spheroids that
  circumscribe them, and an angular 1:0.7:0.5 polyhedron fills only ~25 % of
  that spheroid, so the column was 5–6× the layer whatever the box height.
  Fixes: (1) `deposit_polyhedra` drops each grain's REAL top/bottom profile
  onto the height map of what is already there (solid fraction 0.34, column
  ~1.7× the layer); (2) a short NLGS budget while speeds are capped
  (`deposit_max_checks`, full budget in the release phase); (3) beds are a
  list of `[[layers]]` (kind, thickness, sizes), each deposited on the
  settled layers below — small stones bottom layer, thin top layers and pore-filling layers
  are all expressible; the classic boulders + thin top layer is the default
  `layer_list()`. PHYSICS_VERSION 2026.10.2 (no finished work existed).
  Settling the same bed 15 times for terrain replication is dropped: one bed
  per fidelity study bed, reused by every run (the advisor's design).
- 2026-10-29 — **The six study beds** (docs/BED_STUDY.md): six 6 m beds of
  increasing realism (even size mix → realistic size mix → deep → thin top layer of small stones → small stones
  bottom layer under the boulders → thick top layer of small stones), settled once each, and the
  same fixed-pose 36-drop set on every one; `lab.py study ladder`
  tabulates settle cost, structure and bounce statistics per study bed; the
  paired `--compare` now pairs across beds. It decides which realism the
  position study needs (gates D5 small stones, D9 depth) and what each costs.
- 2026-10-29 — **Position study design** (docs/STUDY_DESIGN.md). Sweeps can
  now hold the pose and approach fixed (`attitude = "fixed"` +
  `attitude_euler_deg`, `attitude_yaw_deg`, one-entry `azimuth`) so a cell
  differs only by landing spot. Speed knobs measured on smoke impact A and
  REJECTED: contact detection every 2 / 4 steps (penetration 18.6 → 4.5 mm,
  e 0.038 → 0.043, 3× unconverged steps; per-step cost did not fall) and dt
  10 ms (e −30 %, iterations 29 → 144). The harness costs < 10 % of a step.
  Honest levers: bodies per patch and run endings. New `shell_mode =
  "adjacency"` (frozen = grains touching the patch; complete support,
  −31 % frozen bodies on the smoke bed; hash-optional, default unchanged) is
  a gate arm (`deployed_gate_shell_adjacency`). `lab.py estimate` predicts
  campaign time from the host's own records (21–32 µs per body-step on the
  laptop). Usable aim square = L/2 − (patch + shell + 2 r_max + 0.05):
  ±1.35 m on the 8 m bed with the craft with panels open; bigger beds are the way to
  wider coverage of one terrain.
- 2026-10-29 — **D2 closed: Milani with deployed solar panels** (`milani_deployed`).
  Geometry from Tyvak's deployed render in Perez Lissi et al. (Hera CubeSats
  paper, fig. 5): two solar panels of three panels each, hinged on the long edges of
  the big solar face and lying in its plane, panels as tall as the craft's body.
  No Milani span is published; Juventas (same 6U-XL form factor, same 2 × 3
  panels) is quoted at ~1420 mm across the arrays → 0.587 m per solar panel.
  Assumed (edit configs/impactors/milani_deployed.toml): panel thickness
  10 mm, root gap 20 mm, 0.6 kg per solar panel, total 12 kg. Modelled as ONE rigid
  body with three convex POLYR contactors (polyhedron detection needs convex
  pieces; a hull would fill the space between the solar panels and the craft's body); each
  piece gets an effective volume mass/ρ and matching inertia so pre's
  rigid properties come out exact: COM 6 mm toward the solar face, principal
  moments 0.153 / 0.339 / 0.453 kg m² (panels folded: 0.077 / 0.151 / 0.195),
  verified against LMGC90's own values in the `deployed_drop` scene. Span
  1.466 m: patches grow to 1.2–2.0 m (gate arms) / 1.3 m (sweeps), and the
  open-panel campaigns (`deployed_gate_*`, `deployed_sweep`) run before the
  folded-craft `bare_sweep` from the second night on; the folded-craft runs become the
  comparison arm. Two facts learned on the way: `getInteractions` icdtac /
  iantac are the contactor's rank within its body, and LMGC90's body frame
  is the principal frame ordered by ascending moment — a permutation of the
  bus axes, stored as `principal_axes_bus` in each run's craft.json.
  ImpactorSpec grew a `wings` field that is left out of the identity hash
  while None (HASH_OPTIONAL), so every panels folded run id is unchanged.
- 2026-10-29 — **Recorder + standard figures**: every impact run now writes
  `trajectory.csv` (34 columns: box state, attitude, contact force, energy
  budget, grain activity at 20 Hz), `box_contacts.csv`, `grains.npz` (free
  grain positions/speeds), `contacts.npz` (contact-network frames) and
  LMGC90's KINETIC/DISSIPATED ENERGY, VIOLATION EVOLUTION, COORDINATION
  NUMBER postpro; settled beds add their resting `contacts.npz`
  (milani/recorder.py). Cadences are module constants, NOT config fields:
  adding a field to a hashed dataclass (ImpactSettings, DetectorSettings,
  SolverOptions, the bed specs) changes every run/bed id and orphans the
  work already done. `lab.py plot` draws a fixed catalogue of figures for
  any run / bed / campaign folder (milani/plots, rules in docs/PLOTS.md);
  reports use the same code. Measured: `getInteractions` reactions are
  forces (verlet tables hold impulse/H; Σ rn on the box = 0.89 |Reac_|).
- 2026-10-29 (night) — **Cohesion works (gate D10 CLOSED), sensitivity measured,
  instability mechanism located** (docs/EXPERIMENTS_2026-10-29.md): the
  Mohr-Coulomb bond in LMGC90 is a stress x the detected contact area, needs
  `coht = cohn`, forms at step 1 on touching polyhedron pairs only (plane
  contacts carry no area) — scene `cohesion` pins it; cohesive rock-rock laws
  (`IQS_MOHR_DS_CLB`, `configs/laws/regolith_cohesive_*`) are allowed in impact
  decks, never on the craft pair. One drop, seven law arms: rock-rock
  restitution irrelevant (0.399-0.401), friction decisive (0.4 slides out of
  the patch, 0.7 e 0.43), cohesion +14 % and saturated between 10 and 100 Pa.
  Settle variants: 20 vertices as good as 12 at 1.4x cost, 8 worse; friction
  0.9 NaN twice; dt 1 ms exploded from rest WITH the cap on — the cap only
  delays the gap-law ejection instability (gap/dt). Held at rest for
  20,000 steps, every lever (θ = 1, scrambled sweep order, baseline) blew
  within 1,300-4,000 steps, while NLGS tolerance 1e-4 held all 20,000 at 8x
  the step cost: the gap-law rest is metastable through tolerance-sized
  residual penetrations, so the pipeline saves at the first rest test with a
  500-step window and changes nothing between phases — and, since a 4 m bed
  blew 400 steps INTO its window, snapshots the first-rest state and falls back
  to it if the window explodes or overruns (card `rest_window WARN`) — (tol 1e-4 when a bed
  must be held); impact decks (speed rule, zeroed velocities, micro-g)
  are immune — free grains move < 15 µm before first contact. A 20-vertex
  bed changed the drop's e from 0.40 to 0.51: terrain beats every law arm. Parameters for Dimorphos
  with sources: docs/DIMORPHOS_PARAMETERS.md (friction 0.6, cohesion 0 with
  10/100 Pa arms, restitution 0.5, grain density 3,400). Book published:
  https://ashcav1.github.io/lmgc90-docs/ (sources github.com/ashcav1/lmgc90-docs).
- 2026-10-29 — **Settling thick beds: what was measured and what changed** (all on
  the laptop, dt 2 ms, GAP_SGR_CLB, 1 g; thin beds = micro 0.15 m / smoke 0.24 m
  layers; thick = 0.9 m of study bed-1 boulders on 1.5 m and 3 m × 3 m bed):
  1. *Deposit*: `deposit_polyhedra` with 64 candidate spots per grain (was 8)
     gives a column 1.7x the final layer (max 2.2x) instead of 2.1x (2.9x);
     0.4 s for 463 boulders, 20 s for the 31,800 small stones of study bed 6.
  2. *speed limit*: on the thick 1.5 m bed, 0.3 m/s rested in 6,800 steps and
     1.5 m/s (the automatic alert-based rule, `speed_cap = 0`) in 10,000:
     harder landings agitate the pile longer. Configs pin 0.3; the automatic
     rule (`BedSpec.speed_cap_for`, ALERT_TRAVEL x alert / dt) stays available.
  3. *Deposition dt*: a larger step during deposition (4-10 ms) calms the pile
     in the same simulated time, then rattles for 100+ s without passing the
     rest test; switching dt back at release threw 100 of 116 resting boulders
     to 1-6 m/s (spins 40 rad/s). `deposit_dt` removed: one dt per settle.
  4. *The cap is never lifted*: the 3 m bed reached rest four times (e.g.
     step 10,000: median grain 0.05 mm/s, every grain below 7 mm/s against a
     0.3 m/s cap) and blew apart within a second of the cap being removed,
     every time — 150-355 of 463 boulders at 0.6-3.4 m/s — with a
     1,000-iteration NLGS budget and with full convergence alike (so the
     budget is NOT the cause; a false lead for an hour). `RBDY3_PartialDamping`
     is a pure clip of grains faster than the cap (mod_RBDY*.f90): inert at
     rest, so a bed at rest under it is physically unaltered; the uncapped
     contact dynamics of a deep polyhedral pile are simply unstable in
     LMGC90 (thin beds survive the release, which hid it). The settle keeps
     the cap on through its final phase, which only switches to the full
     solver (the six study beds solve fully from the first step, `deposit_max_checks = 0`: measured to reach rest at step 5,500 on the 3 m bed; the three thin test beds pin 100).
  5. *Retry policy*: a settle that runs out of steps is no longer wiped and
     rerun at half dt / double steps (it doubled the cost of the slowest beds
     and destroyed the evidence); only crashes/stalls retry, once.
  6. *Diagnostics* in settle_metrics.csv: n_moving, max_spin, min_gap, n_pen,
     n_out; `final_state.npz` (positions, velocities) with every settle.
     The 3 m bed never rested in 10,000 steps with NO escaped grain, NO
     overlap, converged solver: at 10 s its fastest boulders sit inside the
     pile with 10-14 contacts each, moving at the cap.
  7. *Where the motion comes from*: a resting micro bed's contact set
     flickers by +-5 contact points per 50 steps (of ~1,400; -28 right before
     a burst) under the Cundall common-plane detection; each change
     redistributes impulses. In a 1-grain-deep bed the kicks die out; in a
     4-deep pile (3 m x 0.9 m, 463 boulders, 7 contacts per grain) they
     sustain 130-260 chattering grains at 3-5 cm/s for ever (no cap / budget
     / relaxation / periodic quench changes it; full budget converges at
     650 it/step and chatters the same). The face-to-face detection modes
     (PRPRx_UseCpF2fDetection, ...Explicit) gave ZERO grain-grain contacts on
     our triangulated random hulls — not an option; the QA now fails a bed
     with < 1 grain-grain contact per grain (it only warned).
  8. *Rest is structural* (`SettleSpec.rest_disp_m`, 1 mm): a stage is at
     rest when the median grain moved less than 1 mm over the last second of
     simulated time, 95% of grains less than 2 cm and none more than 10 cm —
     or when the old velocity test passes (on the 3 m bed the 99th percentile
     never fell below ~10 mm/s and the 95th stalled near 14 mm/s while the
     median kept falling: a few percent of chattering grains are the pile's
     steady state). The bed is used at micro-g with its velocities zeroed at
     compose; a static geometry with chatter velocities IS the settled bed.
     Measured on the 3 m bed (463 boulders, 0.9 m deep, full solver, cap 0.3):
     at rest by the structural test at step 5,500 (11 s simulated; median
     grain 0.1 mm/s, one grain moving, 300-1,900 NLGS iterations per
     step), ~0.2 s per step alone on the laptop.
- 2026-10-29 — **Bug: layered beds used the classic `boulders.r_max_m` (0.06 m
  default) for the wall margin and patch size** (`site_half_width`,
  `bodies_in_crop`): the usable landing square of the 6 m study beds was
  reported as ±1.63 m instead of ±1.05 m. `BedSpec.r_max_m` (largest grain
  of any layer) everywhere. 36-drop set sites (±0.6 m) were inside either way.
- 2026-10-29 — **Code organisation pass**: legacy sphere/spheroid depositor
  removed; `outward_faces` -> lmgcx.geometry.shapes; `FileLock`, `jsonable`,
  `as_str` single public copies in lmgcx; `describe_bed` -> milani.describe
  (which now recognises a config by its keys); `examples/01..06` and a
  capabilities README.
- 2026-10-29 — **Reports-only sync never pulls code**: while simulations run,
  `sync --reports-only` (and the campaigns' auto-sync) commits the reports on
  top of the remote tip through a temporary index and pushes that, so a code
  push from the laptop no longer blocks the lab Mac's results from reaching
  GitHub, and nothing changes under the running processes (tests/unit/test_sync.py).
- 2026-10-28 — **Overnight slate + campaign shards**: `campaign start
  --shard K/N` runs every N-th run of the plan, so N terminals (one per
  core) share one campaign. The catalog takes flock'd appends, each shard
  has its own manifest, and only shard 1 auto-syncs. `--hours` stops new
  starts (exit 2, resumable). Git syncs are serialized by a lock.
  `overnight/` launches one window per performance core: THIN bed seed 1,
  15 BARE beds, then BARE gates (D11/D12 first pass) and `bare_sweep`.
- 2026-10-28 — **Grain counts are large with realistic shapes**: elongated
  12-vertex polyhedra hold ~13% of their circumscribed sphere, so the 8 m
  THIN bed is ~4,500 boulders + ~22,500 small stones (small stones 8–16 cm long); THICK is
  ~67,500 small stones. `describe` prints estimates. Small stones resolution is gate D5.
- 2026-10-28 — **Gravity**: homogeneous Naidu ellipsoid surface gravity
  4.98–5.50e-5 m/s^2 (~4.4–5.5e-5 with spin and Didymos tide) — 5e-5 default;
  cohesionless rigid dynamics is self-similar in g, so Froude scaling covers
  site-to-site variation.
- 2026-10-28 — **Run identity**: run_id hashes the fully resolved physics
  config + PHYSICS_VERSION (git SHA recorded, not hashed). Bed id likewise.
- 2026-10-30 — **The first six-bed night settled nothing; six causes, all
  fixed** (reports/labmac/overnight/2026-10-29_1514, debug/night_2026-10-29;
  docs/BED_STUDY.md "What the first night taught", docs/FAILURE_MODES.md):
  1. *The cap is applied on every step, window included* (beds.py). The
     10-29 rule "never lift the cap" had been written into the comments but
     `s.cap_speed()` stayed behind `if capped:`; every 500-step check
     since the rebuild ran uncapped, which re-attributes the 10-29 "window
     explosions" (dt 1 ms, budget switch, 4 m at 1e-3) to the cap lift.
     Bed 1 (1,853 boulders) rested at step 19,800 and blew apart within 100
     steps of release.
  2. *No "big" polyhedra* (lmgcx.session: `POLYR_SetThresholdBigPolyr(1e3)`
     before the bodies load; `big_polyr` tripwire in pipeline). LMGC90
     routes pairs touching a polyhedron larger than 4x the deck's mean radius
     through its non-convex detection; bed 5's 19,502 small stones made 17 boulders
     "big" and the deck died with a NaN (zero-frame contact) at step ~150.
     Mixed decks on the laptop never crossed the ratio (3.4), which is why the
     laptop never saw it; every study deck with small stones would. Reproduced
     the same day with `LMGCX_BIG_POLYR_RATIO=4` on a 2 m deck (45 big
     polyhedra: spins to 520 rad/s, non-finite velocity at step ~400).
  3. *4.5 m beds, spots at ±0.3 m, rest at 3 mm/s* (configs). Measured cost
     ~grains^1.3 per step (NLGS iterations grow with the pile) and steps to
     rest ~sqrt(grains); 6 m beds were 3x a 4.5 m bed and bed 1's last ten
     hours bought a creep from 3 to 1 mm/s that micro-g drops never see
     (timescale x450). 4.5 m is the smallest bed width that keeps 1.9 m
     (patch 1.2 m + 2 r_max) between the nine spots and the walls. Bed ids
     change (values in the hash); nothing settled is orphaned.
  4. *Three settled beds per type* for beds 1-4 (`bed_seeds = [1, 2, 3]`),
     one for beds 5-6; terrain varies the bounce more than laws (10-29).
  5. *A finished layer is copied, not re-settled* (`pipeline.stage_key`:
     bed width, materials, settle laws that the stage's pairs can have,
     layers 1..k, seed; walls excluded). Beds 4 and 6 start from bed 2's
     boulders of the same seed: the 2 -> 4 and 2 -> 6 comparisons are the
     same terrain with and without small stones.
  6. *Split small-stone law set on beds 4-6* (`boulder_fines_law = "GAP_SGR_CLB"`)
     ahead of the lab Mac's 2 m tests: it is the only candidate left (velocity
     law fails QA at -20 mm on boulder–stone, position rule among small stones never rests
     over boulders). QA on the real beds decides; a FAIL removes the line.
  7. *Report sync*: the temporary index is built with `git add -A`
     (`rm --cached` refused INDEX.md from the second sync of a night on).
  8. *Estimator* (`lab.py estimate`): layer-by-layer model calibrated on the
     night, never fitted to a host's small beds (10x undershoot).
  Not fixed, recorded: bed 5's bottom layer small stones overlap -27 mm under the
  speed rule (0.54 of the smallest semi-axis, a QA FAIL) — the position rule on
  a small-stones-only floor layer rests (laptop, ~18 h at 4.5 m) but costs the
  whole bed; decide when bed 5 lands.
- 2026-10-31 — **Small stones overlap is graded where the craft can reach it**
  (pipeline.card, `fines_overlap_by_reach`; QA_VERSION 2). The 2 m bed-5
  probes (docs/BED_STUDY.md, small stones): under the speed rule the boulders sank
  9 cm into the bottom layer of small stones (67 mm at the end, 2.2 of the smallest
  semi-axis); under the split contact rule the boulder–stone overlap is 0.0 mm and
  the pile rests, but the small stones among themselves stay 16 mm into each other
  (0.53) — a 1 g compression under the boulders that a micro-g drop never
  loads. Pairs with a grain in the top layer are graded as before (0.15
  WARN, 0.5 FAIL); buried pairs are `overlap_rel_fines_buried`, WARN at
  most. Judgment call: the craft never touches buried small stones and the
  alternative (position rule among bottom layer small stones) costs ~20x and is untested
  under boulders. A bed whose bed.json says FAIL under an older QA_VERSION
  is re-graded from its settled stages, never re-settled — bed 5 of the
  weekend slate started before this rule.
- 2026-10-31 — **A craft flying out over the patch edge is `detached`**
  (detectors.py). On the 4 m bed, 2 of 9 vertical drops at 8 cm/s ended
  `left_patch` after 15–26 s of free flight, rising at 0.7 cm/s: the
  ballistic look-ahead never cleared them because it uses the boulders'
  bounding spheres, and a wide craft 20 cm above the field is inside
  those spheres until it has risen above the tallest one (a minute at
  micro-g). The exit state at the last contact is the bounce; what
  follows outside the patch is not simulated anyway. Rule: airborne, rising,
  past the detachment time and crossing the edge -> `detached`, recorded as
  `contact.detached_how = "patch_edge"` (vs "ballistic"); on the ground it
  stays `left_patch`. Restitution numbers are unchanged; only the label and
  the run's end time. The look-ahead itself also gained a second test
  (`terrain_clear`): the craft is clear when its vertices along the
  ballistic path stay above the bed's height map — the real top envelope of
  the grains, raised by the largest grain displacement since the impact —
  so a bounce over a boulder field is declared at lift-off, not a minute
  later. Runs recorded before this on the lab Mac keep their
  label; the catalog's e_speed of a `left_patch` run with an airborne exit
  is still the bounce. `python3 lab.py campaign regrade <name>` relabels
  them from their trajectory.csv (3 of the 5 voided drops on the 4 m bed;
  the other 2 left on the ground and stay `left_patch`).
- 2026-10-31 — **Solver non-convergence is graded on the contact phase**
  (lmgcx.postpro.solver_health, outcome.run_qa): a run has a fixed handful
  of steps that hit the 10,000-sweep budget — the patch's own relaxation
  before the touch, the shock steps of the touch (8–13 per run on the 4 m
  bed) — so the old fraction of all steps read 4 % on a 1 s run and 0.06 %
  on a 100 s one for the same solver behaviour. Now: steps from the first
  touch on, WARN above 5 %, FAIL above 20 %; the counts before and after
  the touch are in the message.
- 2026-10-31 — **Rest may be declared on the load-bearing half of
  the pile** (beds.py: `d50_big`, `d95_big`, `dmax_big` over the grains at
  or above the median size, same thresholds, as a third OR-branch of the
  rest test; the metrics carry the three new columns). The 3 m bed-2 twin
  (934 boulders in the realistic size mix, 20–70 cm long) kept a median move of 6–8 mm/s for
  2,400 steps with no trend while the 4.5 m even-size-mix bed 1 had rested at
  4,500: on a polydisperse pile the median grain is a small one rattling
  in a pore between big ones, and that never stops; the larger half that
  carries the surface — what the craft meets — is what has to be still.
  Uniform piles: both medians agree. **Tested the same afternoon on a
  parallel settle of the same bed (`exp_bed2_3m_b`, deterministic twin,
  step for step): the larger half median tracks the overall one (9.6 vs 8.4,
  12.3 vs 10.4 mm/s in the plateau) — the whole pile churns, not just the
  rattlers, so the branch does not fire earlier here.** Kept (harmless, and
  the columns are informative); the real lesson is that a realistic-size-mix pile
  needs ~2.5x the steps of an even-size-mix one to reach 3 mm/s (3 m twin: past
  11,500 against 4,500 for the 4.5 m even-size-mix bed 1), which the estimator
  now carries. Not hashed (a stopping rule, like the window fix).
- 2026-10-31 (evening) — **A drop is over only when no point of the craft
  can reach the terrain over the patch** (detectors.reach_clear, impact.py
  clear_ahead; `lab.py check-last-contact`; `lab.py campaign rerun`).
  Replaying a bed-1 drop that had been declared "detached" at 1.3 s, with
  detachment switched off, showed a panel tip grazing a boulder at 20.5 s
  (1.6 N for one step, horizontal speed -6 %). The look-ahead had projected
  the craft's vertices 3-30 s ahead assuming a constant spin axis; a free
  asymmetric body's axis precesses, and over 19 s the panel tip drifted the
  few centimetres between "clear" and a graze. The new test needs no
  rotation model: the craft is inside a sphere of its circumradius (0.76 m
  with the panels open, 0.23 m folded) about its centre of mass, which moves on an
  exact ballistic arc when free; the sphere must stay above the bed's
  height map (+ largest grain displacement so far + 1 cm + half a sample
  step of travel) from now until the centre leaves the patch, checked
  every 0.5 s of free flight. It is a proof of no contact over the patch up
  to the height map's resolution and later grain creep; the replay check
  tests those directly, drop by drop (verdicts in last_contact_check.json).
  Consequence: bounced-off drops run longer (the craft must climb clear of
  the boulders or reach the patch edge: tens of seconds to minutes of
  simulated time at micro-g), and every laptop drop set has been
  re-simulated in place (`campaign rerun`; old records in
  superseded/). Also: "energy kept" counted potential energy from the box
  floor (energy_ratio ~6 % high on a 1 m bed); now from the first-contact
  height, old records recomputed exactly (outcome.energy_kept). And the
  contact-frame normal restitution is reported only for single-touch drops.
- 2026-11-01 — **The craft with panels open turns about its centre of mass: an
  internal ballast** (impactors.ballast_piece; impact.py records
  `setup.craft_geometry_error_m` for every drop). LMGC90 places a body made
  of several contactors at the volume-weighted centroid of their GEOMETRIC
  volumes (mod_RBDY3 read_bodies: OG = sum(v_i shift_i) / sum(v_i)) and keeps
  the principal inertia it is given; the per-piece "effective volumes" that
  make pre's centre of mass right never reach LMGC90. The craft with panels open
  (bus 10.8 kg, solar panels 0.6 kg each, solar panels far lighter for their volume) so
  turned about a point 10.2 mm from its centre of mass in every drop until
  now; the folded craft (one piece) was right. Found by comparing the craft in
  the run's ParaView frames with craft.json: a constant 10.172 mm offset
  along the craft's body x axis. The fix adds a box inside the craft's body (3.9 x 24 x 36 cm,
  3 mm inside every wall: no grain can reach it before the craft's body) whose
  volume moves the geometric centroid onto the centre of mass, and sets the
  density to 12 kg / total geometric volume. In pre the ballast is massless:
  pre takes a contactor's volume and inertia as given only when an inertia
  entry exceeds 1e-6 (rigidContactor3D), so the ballast gets 2e-6 and the
  bus 2e-6 less; build_avatar asserts pre's centre and inertia equal the
  intended ones. Its interactions are excluded from the craft's contacts.
  Verified in a run: the simulated craft within 1.6e-11 m of craft.json,
  inertia 0.4535 / 0.1531 / 0.3387 kg m^2 (intended 0.4534 / 0.1531 /
  0.3386). One 4 m-bed drop: e 0.300 -> 0.329. Ids are unchanged (the bed
  ids must survive); records carry `impact_model` (milani.IMPACT_MODEL) and
  `lab.py campaign rerun NAME --stale` re-simulates older ones in place.
- 2026-11-01 — **A drop ends on the craft's exact free flight**
  (flight.py; impact.py ahead; detectors.py). Once free, the craft feels
  only gravity: centre on a ballistic arc, rotation by the torque-free
  Euler equations (RK4), validated against the recorded free flights of
  bed 1 to 0.001 mm and 0.06 degrees over 10-70 s, every surface point
  within 0.015 mm after 48 s. When the craft has been free 0.5 s and is
  rising, its flight is scanned over the whole bed, which is kept as three
  layers: the patch's free grains at their current pose, the fixed ring and
  the unsimulated bed as settled. Coarse test: surface points every 2 cm
  against each layer's dilated 1 cm top surface; decision: the exact
  distance between the craft's convex pieces and the grains' convex
  polyhedra (closest points of two hulls, bounded least squares; 2e-13 m
  worst error on 1000 box pairs), with conservative-advancement steps so no
  touch falls between two tests. The drop goes on while the flight comes
  within 1 cm + (fastest free grain's surface speed) x (time ahead) of a
  free grain in the time the run could still take: the solver decides
  those, also after the centre crossed the patch edge with a solar panel still over
  it. Otherwise it ends: `detached_how = "clear"` (no touch on the bed) or
  `"beyond"` (the next touch is on the fixed ring or the bed outside the
  patch; the prediction is stored in `contact.flight`). Replaces reach_clear
  (a sphere of the craft's reach: a proof, but it kept drops running for
  minutes) and the patch-edge rule. Same results, far less computing: a 4 m
  drop now ends 0.5-1 s after its last touch instead of 31 s later.
- 2026-11-01 — **The last-contact check is the exact flight over the whole
  bed, plus a replay where it can add something** (checks.py,
  `lab.py check-last-contact --replay near|all|none`). The flight from the
  last contact is scanned until the run's end against the unsimulated bed
  only (inside the simulation the solver decided), after it against all
  three layers (free grains from the run's last ParaView frame; impact.py
  now writes a frame at the very end) until the craft leaves the bed; a
  touch is an exact distance under 1 mm. The replay (detachment off,
  150 s more) runs where the flight passes within 5 cm of a free grain
  after the end, because only there can the grains' own later motion
  matter; `all` replays every drop that ended over the patch. Verdicts:
  confirmed / next touch off the patch / lands back on the patch (after
  the time limit) / touched again / touched unsimulated terrain. Seconds
  per drop instead of minutes, so the lab Mac's sets can all be checked.
  Re-checking the 93 bounced-off drops made before tonight (with the craft
  where LMGC90 had it): 75 touch nothing on the bed again, 17 next touch
  the fixed ring or the bed beyond 34-290 s later (a new hop), 1 comes
  back down on the patch after 547 s; none ended too early.
- 2026-11-01 — **Settle friction stays 0.6, equal to the drops' friction.**
  Settling bed4u_3m's recipe on a 2 m x 2 m box at friction 0.8 and 0.9
  ended in an NLGS divergence (NaN reactions, "diverged at ITERATION 510")
  in 3 of 3 runs; at 0.6 none did, and the one 0.8 layer that finished took
  12,500 steps against 8,400. A friction switched after settling would also
  leave the bed out of balance at the drops' friction
  (docs/EXPERIMENTS_2026-11-01.md).
- 2026-11-01 — **The pages use plain words, and sizes are lengths.** The
  docs, the site and the config descriptions had used the code's names
  (fines, veneer, substrate, crop, frozen ring, stowed, deployed, gap/velocity
  law, ladder, battery, rung, skeleton) and given the long semi-axis r as a
  grain's size. One vocabulary now: small stones, thin / thick top layer,
  bottom layer, even / realistic size mix, patch, fixed ring, panels open /
  folded, position rule / speed rule, speed limit, 500-step check, study beds
  1-6, 36-drop set. Sizes are lengths (2 r; measured grains are ~10 %
  shorter, 10-12 corners outline each). The mapping lives in
  `site_text.PLAIN` and the glossary in `site_text.GLOSSARY`; code names stay
  in the code, configs' `name` fields are unchanged (they are in bed and run
  ids), descriptions are not hashed.
- 2026-11-01 — **The study beds stay 4.5 m x 4.5 m; 2 m beds are not used
  for drops.** Box side = 2 x (patch radius + 0.75 m) + the spread of the
  landing spots, the 0.75 m being one largest boulder plus 5 cm kept from
  the walls. Open panels (1.2 m patch) and the 3 x 3 grid 30 cm apart: 4.5 m.
  Folded panels (0.7 m patch), spots 5 cm apart: 3.0 m. A 2 m box leaves a
  0.25 m patch with 70 cm boulders. The box only costs settle time; a drop's
  cost is the patch's.
- 2026-11-01 — **Gate D12 (patch size), first pass: 1.2 m is enough with
  the panels open.** Same 20 drops on bed 1, patch 0.9 / 1.2 / 1.45 m: median
  e 0.213-0.229 (0 deg) and 0.315-0.318 (40 deg); paired change against
  1.45 m 0.000 (95 % -0.005..+0.010) at 1.2 m; 0.9 m cuts off twice the
  drops (6 of 20 left the patch, against 3). Folded panels (0.45 / 0.7 /
  1.0 m): single drops change by up to 0.27 with the same first touch; 0.7 m
  sits 0.04 below 1.0 m (95 % 0.004..0.08) and 0.45 m does not, so it is
  not a clean patch effect but not settled either. Study beds and their
  1.2 m patch stay; folded-craft results from 3 m beds carry about ±0.05.
- OPEN GATES: D1 (parameter provenance — provisional regolith_v1), D2 (Milani
  contact model if the panels are open), D5 (small stones resolution / SFD), D7 (budget
  from pilot timing), D8 (MASCOT benchmark — re-derive the target CoR values
  from primary sources first; notes disagree), D9 (bed depth), D10 (cohesion
  harness; cohesion disabled in config validation until then), D11 (dt
  convergence: gate_dt_fine vs gate_crop_small), D12 (patch convergence: first pass done
  2026-11-01, open panels pass at 1.2 m; folded panels unsettled). D3 closed by design (walls for settle,
  patch for impact; periodic boundaries remain an option). D4 closed
  (polyhedral small stones).
