# Crash & divergence field guide

*Written 2026-08-30 from a source dive into `src/Core/kernel/mod_nlgs_3D.f90`
(6,057 lines), `src/ChiPy/kernel/wrap_nlgs_3D.f90`, and the ChiPy macro
layer, for the 2025.rc1 tree this project runs. Purpose: when a run crashes,
start here. Line numbers refer to that tree.*

---

## 1. Where our solver call actually goes

`chipy.ExSolver("Stored_Delassus_Loops         ", norm, tol, relax, gs_it1,
gs_it2)` → `nlgs_3D_ExSolver` (`wrap_nlgs_3D.f90:397`). The orchestration
(`:437-506`):

```
set_nlgs_parameter(norm, tol, relax)
prep_nlgs(SDL?)                      # build local W operators (Delassus)
do block = 1 .. nb_block_iter:       # ← this is our gs_it2 (!)
    [quick_scramble / reverse if enabled]
    do i = 1 .. nb_iter_check:       # ← this is our gs_it1 (!)
        solve_nlgs(i_iter)           # one full Gauss-Seidel sweep
    prep_check_nlgs(iconv)           # 0 contacts → RETURN immediately
    solve_nlgs(i_check)              # extra sweep accumulating residuals
    comp_check_nlgs(iconv)           # 0 = converged, -1 = DIVERGED
    display_check_nlgs
    exit when iconv is 0 or -1
RnodHRloc + solve_nlgs(i_post)       # push reactions to bodies
if iconv == -1:  "@ NLGS diverged at ITERATION: n"  → faterr → PROCESS DIES
if never converged: quiet log "WARNING : NLGS not converged" and CARRY ON
```

Note the ordering: on divergence the NaN is detected at the loop `EXIT`,
but `RnodHRloc` + the `i_post` sweep still run — **the NaN reactions are
pushed into the global body `Ireac` arrays first, then the faterr fires**
(`wrap_nlgs_3D.f90:487-501`). The dying process's state is already
poisoned; anything that would catch-and-continue (see §6b) must treat the
whole simulation state as lost.

**Gotcha #1 — the argument names are swapped relative to the docstring.**
The Python docstring says gs_it1 = number of convergence checks, gs_it2 =
sweeps per check. The wrap wires it the other way: **arg5 (our `GS_IT1=50`)
is the number of sweeps BETWEEN checks; arg6 (our `GS_IT2=1000`) is the max
number of check blocks.** So our settings mean: up to 1000 checks × 50
sweeps = 50,000 sweeps max, checking every 50. Harmless in practice, but
read logs accordingly: "NLGS ITERATION: n" counts sweeps.

**Gotcha #2 — non-convergence is NOT an error.** If the loop exhausts
without converging, the solver logs a quiet warning and the step proceeds
with whatever reactions it has. Only *divergence* is fatal. A campaign can
silently run steps that never converged — watch `SOLVER INFORMATIONS` /
violation postpro, not just crashes.

**Gotcha #2b — our runs can't see the warnings.** `LOGMES`
(`mod_utilities.f90:333-342`) prints only when verbose mode is on or the
message is forced. Our sims call `utilities_DisableLogMes()`, and the
"NLGS not converged" warning and per-check residual displays are NOT
forced — so in v3/v4 logs they are **silently suppressed**. The forced
"diverged" message still appears. Practical rule: non-convergence is only
visible through postpro (`SOLVER INFORMATIONS`, violations) — never expect
it in `log.txt`.

**Gotcha #3 — norm strings are matched exactly against 5 characters**
(`'Quad '`, `'Maxm '`, `'QM/16'`, `'QuaN '`, `mod_nlgs_3D.f90:5065-5074`)
**and an unrecognized string is silently ignored** — the `SELECT CASE` has
no `default`, so `i_checktype` keeps its previous value (module default
`=1` at `:264`). A typo in NORM does not error; it just checks a different
norm than you think. Bonus hazard: the wrap copies **all 5 bytes** of the
C string with no null-check (`wrap_nlgs_3D.f90:433-436`), so a 4-char
Python literal reads one byte past its terminator — always pass the full
5 characters including the trailing space.

**Gotcha #4 — never set GS_IT2 ≤ 0.** `iconv` in `ExSolver`
(`wrap_nlgs_3D.f90:414`) is **uninitialized** until the first block
iteration assigns it; with `nb_block_iter <= 0` the loop body never runs
and the final `if (iconv == -1)` reads garbage — a nondeterministic
spurious "NLGS diverged" abort with zero sweeps performed.

## 2. What "NLGS diverged" literally is

`comp_check_nlgs` (`mod_nlgs_3D.f90:5343-5371`):

```fortran
call check_convergence_nlgs(... , converged)
if (converged) iconv = 0
s = any(isnan(this(:)%rls)); t = any(isnan(this(:)%rlt)); n = any(isnan(this(:)%rln))
if ( s .or. t .or. n ) iconv = -1
```

**Divergence == a NaN appeared in some contact reaction.** Nothing subtler.
Every historical "NLGS diverged at ITERATION n" was NaN contamination in
`rls/rlt/rln`, detected at the *next* convergence check (hence the iteration
number is a multiple of the sweeps-per-check, not the true birth moment of
the NaN). Consequences for debugging:

- The divergence iteration tells you the check block, not the cause. The
  NaN was born somewhere in the preceding ≤50 sweeps.
- **Two concrete NaN factories live in `prep_nlgs` itself, and both warn
  only via suppressed logmes.** (a) A degenerate Delassus diagonal:
  `Wnn ≤ 1e-18` (bad inertia, zero mass, broken contactor) prints a
  diagnostic at `mod_nlgs_3D.f90:1005-1016` — **logmes, run continues** —
  then the solver divides by it (`ron = 1/Wnn` :2574, `invWnn` :2591,
  `rlniki = -vvlocfreenik/WWnnik` :3048) → Inf/NaN a few sweeps later.
  (Two FIXED bodies touching give exactly Wnn = 0 — never give frozen
  bodies a rule between them.)
  (b) A near-singular local 3×3: `det < 1e-24` (`:2564`, `:2613`) — same
  pattern, warn-and-continue into a division. Under our
  `DisableLogMes` these precursors are invisible; the first symptom we
  ever see is the diverged faterr.
- Other candidates: garbage state from a corrupted restart, and — the
  historical big one — data races in the threaded build (next section).
- Because it's `faterr`, the Python process dies uncatchably. The engine's
  retry ladder (fresh → dt/2 → tol×3) is the correct response shape:
  smaller dt shrinks per-step impulses (better-conditioned local solves);
  looser tol ends the solve earlier (fewer sweeps in which to blow up).

## 3. Why the OpenMP build diverges on many cores (our M2-Ultra story, formalized)

`solve_nlgs` runs the sweep as (`mod_nlgs_3D.f90:2719-2741`):

```fortran
!$OMP PARALLEL DEFAULT(SHARED) PRIVATE(...~60 locals...)
!$OMP DO SCHEDULE(RUNTIME) REDUCTION(+:...counters...) REDUCTION(MAX/MIN:...)
DO ikk = 1, nb_CDAN
   ik = IALEAT(ikk)          ! randomized contact order
   ... read neighbour reactions from SHARED this(:) ... update this(ik)%rl* ...
```

The Gauss–Seidel update of contact `ik` reads the reactions of its
neighbours from the **shared** `this(:)` array — the same array other
threads are concurrently writing. There is no synchronization on those
reads/writes:

- **Serial:** true Gauss–Seidel — every read sees the freshest value.
  Deterministic, fastest convergence.
- **Threaded:** each read sees an unpredictable old/new mix (a chaotic
  asynchronous iteration), and a read racing a write of a `real(8)` can in
  principle observe a torn/inconsistent value. More threads ⇒ more
  interleavings ⇒ larger effective "staleness" ⇒ occasional NaN ⇒ the §2
  detector fires. That is why 24 threads (lab M2 Ultra) diverged reliably
  while ~14 (dev M4 Pro) mostly survived, and why `OMP_NUM_THREADS=1` /
  the serial build fixed it. **Our engine forces serial in every child.**

The kernel sweep found the threaded sweep is racy in **three independent
ways**, not one:

1. **Stale/torn neighbour reads** — the mechanism above: `this(:)%rl*` is
   read and written concurrently with no ATOMIC/CRITICAL/colouring
   anywhere.
2. **Lost updates on body forces** — the harder one. Each contact pushes
   its reaction to its two bodies via `injj_` → `add_reac`
   (`mod_RBDY3.f90:3121-3141`): a bare `bdyty%Ireac = bdyty%Ireac + xxreac`
   with no atomic. Two contacts touching the *same body* on different
   threads simply lose increments. Same exposure for `Vaux` in `vitrad_`,
   and `nullify_reac_`/`nullify_vlocy_` zero those arrays from inside the
   loop. In a dense bed (every grain has many neighbours) this is
   systematically wrong physics, not just noise.
3. **Locals that never made the PRIVATE list** — under `DEFAULT(SHARED)`,
   five variables written inside the loop are shared across threads:
   `is_traction` (`:3521/:3549`), `snmax` (brittle-failure threshold,
   `:2963`), `kkk`, `mu_g`, and the 80-char message buffer `cout`. These
   only bite the CZM/brittle law families (not our RST/IQS/MOHR set), but
   they mean even "embarrassingly parallel-looking" laws race. Related
   oddity: `WRRmin/WRRmax` are declared with initializers (`:2680`), which
   in Fortran makes them implicitly `SAVE` — so they live in REDUCTION
   clauses *and* persist across calls; at best implementation-defined.
- `SCHEDULE(RUNTIME)` means chunking follows the `OMP_SCHEDULE` env var —
  one more way parallel runs are irreproducible.
- Interesting: a **`JACOBI_SOLVER` mode exists** (`:2711-2717`) that
  snapshots all reactions before the sweep (race-free by construction,
  at the price of Jacobi's slower convergence; the code even warns that
  Jacobi with RELAX=1 may not converge). The parallel-safe path exists —
  the default GS path just isn't it.

## 4. The convergence test, decoded

`check_convergence_nlgs` (`:5375-5413`). All residuals are **pre-normalized
by `tol`** during the check sweep, so "< 1" means "within tolerance":

| quantity | meaning |
|---|---|
| `QuadDV` / `MaxmDV` | RMS / max change in local relative velocity between sweeps |
| `QuadDVR` / `MaxmDVR` | same, reaction-weighted |
| `MeanDVoR` | mean complementarity residual (ΔV·R against ΣWRR) |

- `Quad ` (ours): mean-square test — `|MeanDVoR|, QuadDV, QuadDVR < 1`.
  Tolerant of a few bad contacts (they average out).
- `Maxm `: same but on the max — every single contact must be within tol.
  Much stricter on outliers; typically needs more iterations.
- `QM/16`: Quad **plus** a leash on the max at 16.666× tol — mean accuracy
  with a cap on how bad any individual contact may be. Good middle ground
  and a sensible upgrade if we ever suspect a few rogue contacts hiding
  under a converged mean.

`display_check_nlgs` prints these each check; in a stalling run, watch
whether the Quad values plateau (structural non-convergence — dt too big,
bad geometry) or oscillate (cycling statuses; scramble/relax help).

## 5. Mechanisms worth knowing inside the sweep

- **Restitution (RST_CLB) is a velocity-RHS trick** (`:1242-1261`): at prep,
  if the contact's gap at *step start* is ≤ 0, the law sets
  `covfree* = rst · v*BEGIN`, and the Signorini condition is then enforced
  on `v + covfree` — i.e. Newton restitution `v⁺ = −e·v⁻`. If the gap
  starts positive, the pair is `forecast = noact` for this step — **a
  closing contact reacts one step late** (first step: free flight closes
  the gap; next step: shock law applies). Explains the harness's clean
  e ≈ rstn measurement and why detection alert distances matter.
- **Contact-skip heuristic** (`:2775-2781`): a contact repeatedly in
  `noctc` status gets skipped for `iskip(iskip+1)/2` sweeps — triangular
  backoff. Cheap sweeps on quiet beds; also means a long-quiet contact
  wakes with slight latency.
- **Radjai Gap Rescue** (`Nb_RGR`, `:283-286`): a rescue branch for
  contacts with pathological gaps; its activation count is reported in the
  check counters. Nonzero RGR counts = the solver is fire-fighting
  geometry (interpenetrated states, e.g. after a bad restart or teleport).
- **Scramble** (`quick_scramble_nlgs:5698`, `IALEAT`): permutes the GS
  contact order per block. Breaks order-induced cycling; costs
  reproducibility of the iteration path (not of physics). We don't enable
  it by default; it's a lever for oscillating non-convergence.
- **SDL vs ELG**: `Stored_Delassus_Loops` precomputes and stores each
  contact's local W and its couplings (`Wab`, `:179`) — fast sweeps, memory
  ∝ contacts × adjacency. `Exchange_Local_Global` recomputes on the fly —
  slower, tiny memory. Big beds with dense contact networks pay SDL memory;
  if allocation ever fails at prep on huge scenes, ELG is the escape hatch.
- **Diagonal resolution** (`diagonal_resolution`, `:290`;
  `chipy.nlgs_3D_DiagonalResolution()`): solve each contact with only the
  diagonal of its local W (ignore in-sweep coupling refinement). The
  archive scripts used it; v3/v4 don't. Faster per sweep, weaker coupling
  handling.

## 6. Crash taxonomy — what kills a run and what to do

*(Merged from the hand dive + a kernel-wide fatal-path sweep of every
`faterr`/`stop` in the rigid-3D runtime. Grouped by subsystem; the
Appendix has the exhaustive per-module lists. §7 has our case studies.)*

### 6.1 Solver (nlgs_3D)

| symptom in log | source | trigger | response |
|---|---|---|---|
| `@ NLGS diverged at ITERATION: n` + faterr | `wrap_nlgs_3D.f90:501` | NaN in contact reactions (§2) | engine retry ladder (fresh → dt/2 → tol×3); if reproducible serial: inspect masses/inertia, restart integrity, first-contact interpenetration; look for the §2 precursors |
| `WARNING : NLGS not converged` (quiet, run continues) | `wrap_nlgs_3D.f90:506` | iteration budget exhausted | not a crash — but check violation/SOLVER postpro; raise gs budget or shrink dt if frequent |
| `Unknown solver type...` faterr | `wrap_nlgs_3D.f90:447` (ExSolver), `:348` (ExPrep) | storage string ≠ `Stored_Delassus_Loops` / `Exchange_Local_Global` | fix the constant (trailing padding fine — trim compare) |
| silent wrong norm | `mod_nlgs_3D.f90:5065` | unmatched 5-char NORM string (no CASE default) | keep `"Quad "` exactly, trailing space included |
| `error allocating Wab` (preceded by `Wab size N`) | `mod_nlgs_3D.f90:857` | SDL Delassus storage = 72 B × Σ adjacencies exhausted RAM | switch storage to `Exchange_Local_Global` — skips the whole Wab block |
| `something is going wrong in compute Wikjl index in Wab storage` | `mod_nlgs_3D.f90:819` | int32 overflow of the Delassus index (~2.4e8 contact-adjacency pairs; Wab would be ~17 GB) | same — ELG storage |
| `error allocating this` / `ialeat` / `Xvlton` … | `mod_nlgs_3D.f90:538-589` | `this(nb_CDAN)` ≈ ≥480 B/contact etc. — RAM | smaller scene or ELG (helps Wab only); check contact count isn't pathological (exploded bed) |
| `mismatch in the entity connectivity for entity …` | `mod_nlgs_3D.f90:688`, `:832` | detection/entity-list bookkeeping inconsistency (see §6.5 — damage may already be done) | call-order audit: `overall_InitEntityList` vs `LoadTactors`; report upstream |
| `default select case <lawty>` | `mod_nlgs_3D.f90:2547` | a law in TACT_BEHAV.DAT that nlgs_3D doesn't implement | fix the law name/deck |
| `this contact element doesn t work with<lawty>` | `mod_nlgs_3D.f90:1483` | `IQS_MOHR_DS_CLB` (Mstck status) on a contactor pair outside its 10-type whitelist | our SPSPx/SPPLx/SPPRx small-stone pairs are all whitelisted; would only fire on an exotic pairing |
| `Wnn(<ik>) = … < 1.D-18` / `det(<ik>) = … < 1.D-25` (logmes — **invisible to us**) | `mod_nlgs_3D.f90:1014`, `:2565`, `:2614` | degenerate local operator | not fatal itself — it's the NaN precursor (§2); if seen with logs on, the named body/contact is the culprit |

### 6.2 Contact detection (PRPRx / SPSPx / SPPLx / SPPRx)

| symptom in log | source | trigger | response |
|---|---|---|---|
| banner `ERROR filling this … PRPRx_LowSizeArrayPolyr(sizefactor) …` + faterr `Error` | `mod_PRPRx.f90:3419` (Cundall, our default path), `:5844` (nc), `:6673`/`:7886`/`:13035` (f2f/wti/STO) | more contact points than `size_factor × max(nb_POLYR, nb_rough_pairs)` — the classic dense-bed abort | raise `PRPRx_LowSizeArrayPolyr` (v4 already sets **100**, default 4). NB the arg is unvalidated — 0/negative = instant nonsense |
| `issue filling boxes` (after `body POLYR <i> out of boxes`) | `mod_PRPRx.f90:2119-2132` (SPSPx twin `:563-570`) | a body's coordinates left the broad-phase AABB **this step** — it exploded / NaN'd / flew unbounded | the detection error is the messenger; autopsy the dynamics (last DOF state, gravity, teleport step) |
| `Estimated max popul limit reached.` | `mod_SPSPx.f90:575` | int32 overflow: `maxpopul = 4(1+Lbox/minray)³` with **no negative guard** (PRPRx/SPPLx have one) — extreme sphere radius ratio | keep small stones SFD bounded (our stratified small stones are); if hit, narrow the radius span |
| `Distance between cd <i> and an <j> … is <tiny>` + `ERROR` / `center of spher <i> within 1.e-18 …` | `mod_PRPRx.f90:3222`, `mod_SPSPx.f90:1178` | two body centres coincident — one tunneled inside another (bad restart, explosion, double-placed grain) | inspect the named pair in the last state; check deposit/registry for duplicates |
| `error allocating box` / `box(...)%which` | `mod_PRPRx.f90:2079`, `mod_SPSPx.f90:542` | broad-phase grid = `Π(1+extent/Lbox)` cells — huge domain + small cells = multi-GB | domain sanity: usually a stray body at huge coordinates inflating the AABB |
| `nickname <behav> … unknown in lawty` | `mod_SPPLx.f90:1499` and peers | a see row names a law that is not defined | define the law; note the `Unknown visibility table` faterr at `mod_SPPLx.f90:1484` is DEAD CODE (no caller) — a pair with no see row is simply never detected, in every module (this is how walls are made invisible to the spacecraft) |
| `unexpected problem while computing nodetoface/edgetoedge/… distance`, `unexpected error in convex hull computation`, `strange id_ptc` | `mod_PRPRx.f90` many sites (§Appendix) | geometry-helper failure on a degenerate polyhedron pairing | usually a mangled/degenerate mesh contactor; check the printed body ids' POLYR |
| *(silent!)* pair produces **no contact** | `mod_PRPRx.f90:9351` | Cundall common-plane iteration budget exhausted → returns 0 → **pair silently dropped**; diagnostic only under a hard-coded `bavard=.false.` | v4 passes 200000 iterations (default 100) making this near-impossible; if impactor ever "ghosts through" a face, suspect this |

### 6.3 Decks & laws (tact_behav)

| symptom in log | source | trigger | response |
|---|---|---|---|
| `Unknown interaction law` | `mod_tact_behav.f90:9362` | law string in TACT_BEHAV.DAT not in the enum | typo — the preceding logmes with the name is suppressed for us; diff the deck against a working one |
| `nb of parameters is wrong` | `mod_tact_behav.f90:363` | wrong param count for that law — the 4 explanatory lines (law, nickname, got, expected) are **logmes → invisible to us** | regenerate the deck; if hand-debugging, rerun with logs enabled to see the counts |
| `reading error in law <behav>` | `mod_tact_behav.f90:2312` etc. | malformed number in TACT_BEHAV.DAT | regenerate deck |
| ` lawty <lawty> not implemented` (`get_fric`/`get_rst`) | `mod_tact_behav.f90:6418`, `:6455` | friction/restitution requested from a law that has none (get_rst accepts only RST_CLB / RST_DS_CLB / RST_WET_CLB) | law-vs-parameter mismatch in scene generation |
| *(silent!)* cohesion silently zero | `mod_tact_behav.f90:6528` | `get_coh`'s `case default` is **commented out** — an unsupported law returns coh = 0 without erroring | IQS_MOHR_DS_CLB is supported; but never assume a law bonds — verify (gate D10) |

### 6.4 Restart / HDF5

| symptom in log | source | trigger | response |
|---|---|---|---|
| `hd5::open_read_h5 No file named …` | `mod_lmgc90_hdf5.f90:176` | missing file OR **long absolute path** (fixed-length buffer) | short local names/symlinks — v4's read_restart does this |
| `Error opening file : …` | `mod_lmgc90_hdf5.f90:174` (read), `:215` (write) | file exists but corrupt / not HDF5 / **locked by another process** / permissions | v4 quarantines corrupt h5 via defensive h5py open; lock = two processes on one file (never share run dirs) |
| `Error creating file : …` | `:218` | missing dir, disk full, read-only fs | check the external drive mount + free space (doctor does) |
| `Version not found in file` / `unknown format: X.Y` | `:824`, `mod_h5_format.f90:297` | not an LMGC90 h5 / file written by a **newer build** | rebuild the state locally; never carry h5 across build generations |
| `Please read_bodies before trying to read dof` | `mod_h5_rigid.f90:271` | ReadIni before ReadBodies/LoadTactors | call-order bug |
| `Inconsistent size between already allocated bdyty and data in file` | `mod_h5_rigid.f90:275` | restart body count ≠ BODIES.DAT — a regenerated DATBOX under an old bed | exactly what v4's registry fingerprint guard pre-empts at config time |
| *(silent!)* requested record doesn't exist | `mod_lmgc90_hdf5.f90:466`, hl `:197` | `ReadIni(step)` beyond end-of-file **logs one suppressed line and no-ops** — the sim silently starts from the raw DATBOX state instead of the restart | this is why v4's read_restart counts records with h5py *before* calling ReadIni |

### 6.4b Bugs found in this build (2026-10-28) — read these first

| symptom | source | cause | response |
|---|---|---|---|
| **random SIGBUS** (exit −10), `Fatal Python error: Bus error` in `PRPRx_SelectProxTactors`; macOS crash report shows `polytopes_intersection_wc <- planplan <- detection_common_plane <- wcp_compute_contact_prprx` | `Core/shared/mod_DiscreteGeometry.f90:4572` | `area = sum(carea(1:nb_pts))` indexes the per-POLYGON area array by the number of POINTS → out-of-bounds read on every face-to-face contact; fatal when the tiny C allocation ends a memory page. Same inputs crash or not at random | `python3 lab.py patch-lmgc90` (one-line fix `area = sum(carea)`; value unused, physics unchanged — verified bit-identical). Simulations refuse to start without it |
| spheres sink into polyhedra (LMGC gap ≈ −radius, confirmed by exact hull distance) with no error | `mod_SPPRx.f90` DETECTION_SPPRx → `node_HE_Hdl_proximity` | sphere–polyhedron detection projects from the sphere toward the polyhedron CENTRE and misses approaching contacts on large faces | use polyhedral small stones (the pipeline does); never mix SPHER grains with POLYR boulders in this build |
| `make` fails installing Clipper into `/usr/local/lib` after a `cmake .` re-run | LMGC90 `CMakeLists.txt` VENV_LAST logic | the venv install prefix is applied only on the FIRST configure | re-configure with `-DCMAKE_INSTALL_PREFIX=<venv>` (the patch tool does) |
| `git branch changed... please rerun cmake to confirm` during `make` | `src/get_git_branch.cmake.in` | the build records the git branch of the enclosing checkout | `cmake .` in the build dir (harmless; the patch tool does it) |

### 6.5 Silent killers — dies with no message at all

| symptom | source | trigger | response |
|---|---|---|---|
| SIGSEGV / garbage during `ReadIniVlocRloc` | `mod_h5_interaction.f90:628` | interaction-type id read from file indexes a fixed stack array with **no bounds check** — h5 from a build with different interaction enums (the `ugly_fix` scenario) | `io_hdf5_FixVersion(0)` for old files; better: never read h5 across builds |
| SIGSEGV / heap corruption in `ExPrep` | `mod_nlgs_3D.f90:415-420` | `entity(%ik)%list` written past its `%nb`-sized end if a detector miscounts — the consistency faterr at `:686` runs **after** the damage | report upstream; in practice: a crash *here* means detection bookkeeping is broken, not your scene |
| process exits code 1, no traceback | any `faterr` (default mode) | `stop 1` (§6b) | the engine's heartbeat/retry machinery is built for exactly this |
| `RBDY3::put_vector unknown id: …` faterr | wrap RBDY3 | invalid Put vector name (e.g. `Coor_` — GET-only) | teleport via X____/Xbeg_; see `lmgc90_api_enriched.md` |
| `input seed too short…` faterr | `mod_utilities.f90:438` | upstream deposit seed list < compiler's `random_seed` size | moot in v4 (own depositor); ≥33 entries if ever calling upstream |
| deposit `stop` (num/verlet array size) | `mod_deposit3D.f90:1604,1661` | >110 reference grains / >2000 neighbours in upstream deposit | moot in v4 — own depositor |

## 6b. `faterr` anatomy — and a possible future upgrade

Every fatal in the tree funnels through **one** routine. `faterr`
(`mod_utilities.f90:344-362`) writes `ERROR [caller]: message` and — with
the default `no_stop = .false.` (`:74`) — executes `stop 1`: process
death, uncatchable from Python. **But** there is a designed escape hatch,
and it is real plumbing, not a vestige:

- `chipy.utilities_setStopMode(False)` flips the flag
  (`wrap_utilities.f90:57` is its only writer); faterr then `f_jump`s —
  a C `longjmp` (`mod_exception.f90:47`, `exception.c`).
- The SWIG layer wraps **every chipy call** in a matching `setjmp` and
  converts the jump into a Python `RuntimeError("ERROR [caller]: message")`
  (`chipy.i.in:138-153`).
- Upstream's own modern driver **already runs this way**:
  `computation.initialize()` calls `utilities_setStopMode(False)` as its
  first statement (`ChiPy/macro/computation.py:25`). So the mode is
  exercised code, not theory.

*Status for us: still not adopted, deliberately.* A zero-body probe
short-circuited before reaching a faterr, so we have no harness proof yet
— and the sweep surfaced hard caveats that shape how it could ever be
used:

1. **Catch-and-exit only, never catch-and-continue.** The longjmp does no
   unwinding: module state (`this`, `Wab`, verlet lists, entity lists,
   open HDF5 handles) is abandoned mid-mutation, and on a divergence the
   NaN reactions were already pushed to the bodies (§1). The only safe
   use is: catch → write a rich `fail.json` with the exact
   `ERROR [caller]: message` → exit nonzero.
2. **Undefined behaviour if the faterr fires inside the OpenMP region**
   (several sites do: `mod_nlgs_3D.f90:4104`, `:4461`, `:4674`, `:5256`,
   `:5842`) — longjmp out of a parallel block from a worker thread.
   Moot for our forced-serial children, fatal caveat for anyone else.
3. Minor: `ExIterJacobi` leaves `JACOBI_SOLVER` stuck `.TRUE.` if a
   faterr escapes mid-loop, and each caught fatal leaks its message
   buffer (`wrap_utilities.f90:82`).

Candidate for the next hardening pass (a harness scene that genuinely
triggers a faterr, e.g. a deliberate bad Put vector, would settle it);
until then, treat every faterr as process death — the heartbeat/retry
machinery already assumes exactly that.

## 7. Our case studies (what actually happened)

1. **Lab M2 Ultra, "NLGS diverged at iteration ~30950", every run; personal
   M4 Pro fine.** Root cause: OpenMP build + 24 threads → §3 races → NaN →
   §2 detector. The likeliest dominant mechanism in a dense bed is race
   family 2 (lost updates on shared-body reaction accumulation) compounded
   by family 1 (stale neighbour reads). Fix: serial solver everywhere
   (`OMP_NUM_THREADS=1` forced per child; dev machine rebuilt with
   `-DWITH_OPENMP=OFF`). Note the iteration number ~30950 was a *sweep*
   count — with 50 sweeps/check the NaN was detected at check #619, born
   within the prior 50 sweeps.
2. **Laney's week-long trade study, sporadic crashes.** Consistent with
   occasional NaN divergence + unattended runs having no restart
   automation. v4's answer: heartbeats, `last.h5` checkpoints, retry
   ladder, resumable queue.
3. **"0.03 m small stones — error"** (Laney's slides): most plausibly the deposit
   layer's polydispersity/neighbour limits or detection-scale issues, not
   NLGS proper. v4's own depositor + D5 resolution gate cover this axis.
4. **Restart-integrity family**: corrupt `last.h5` after kill-mid-write
   used to brick all retries (h5py open outside try). v4's `read_restart`
   opens defensively and falls back to the pristine bed state.

## 8. Monitoring cheat-sheet

- `SOLVER INFORMATIONS` postpro (every 25 steps in v4): iterations used,
  error norms, contact counts — a creeping iteration count precedes most
  trouble.
- Check-block print (`display_check_nlgs`): Quad/Maxm residuals per check —
  plateau = structural, oscillation = cycling (try scramble or relax<1).
- Status counters (`Nslide/Nstick/Ncompr/Ntract/NOKsta/Nb_RGR`): NOKsta
  ("questionable status") and RGR activity are early smoke.
- v4 `impact_metrics.csv` logs `reac_norm` per 25 steps — a NaN there
  pinpoints the birth step far better than the solver's check granularity.

---

## Appendix — exhaustive fatal-path inventory (kernel-wide sweep, 2026-08-30)

*Every `faterr`/`stop`/unbounded-write reachable in a rigid-3D
POLYR/SPHER/PLANx run, by module. §6 has the triage; this is the lookup
table when a message isn't there. "logmes" rows are invisible under our
`DisableLogMes` unless marked forced.*

### A1. `mod_nlgs_3D.f90` (prep/solve; `IAM = nlgs_3D::prep_nlgs` / `::solve_nlgs`)

- Allocation faterrs: `ialeat` :538, `ialeatr` :544, `randomlist` :557,
  `Xvlton` :571, `WRRarray` :577, `this` :589, `adjjl` :747, `rev_adj`
  :756, `Wab` :857 (preceded by `Wab size N`).
- Bookkeeping faterrs: entity connectivity mismatch :688/:832; Wikjl
  int32 index overflow :819; `bandwidth and nb_adjac should be the same`
  :825; `unable to find ik in the adjacent contacts of jl` :911.
- Degenerate-operator **logmes** (then continues into a division!):
  `Wnn < 1.D-18` :1005-1016 (→ divisions at :2574/:2591/:3048), `Wtt`/`Wss` clamped
  :1032/:1053, `det < 1.D-25` :2564 (non-diag) / :2613 (diag).
- Law-compatibility faterrs: IQS_MOHR contactor whitelist :1483;
  VISCO_ELASTIC_REPELL_WET :1671; `gapREF < 1.D-18` for
  ROD/WIRE laws :1702-1817; CZM-family "not implemented for this contact
  element" :1864-2410; CRITICAL_VOIGT diag :2239; master `default select
  case <lawty>` :2547.
- Inside the OMP region (see §6b caveat 2): :4104, :4461, :4674
  (NARD), `mu_NG_solver` `A.W+B is not inversible` :5256,
  `coupled_dof_solver` `Non inversible matrix` :5842, `print_info`
  fallthrough :5998.
- No-default hazards: norm SELECT :5065-5075 (silent), `set_nlgs_parameter`.

### A2. Detection modules

**PRPRx** (`mod_PRPRx.f90`): compute_box minray/maxray :450/:452;
box-grid allocs :2079/:2087; `body … out of boxes → issue filling boxes`
:2119-2132; popul limit :2136 (effectively unreachable in PRPRx —
guarded); rough alloc :2667; **the 5 `this`-overflow banners**
:3419 (wcp=ours) / :5844 (nc) / :6673 (f2f) / :7886 (wti) / :13035 (STO);
coincident centres :3222/:6570/:7723/:12868; cohesive-recovery
`not able to recover all cohesive contact points` :3391; geometry-helper
faterrs (`nodetoedge/nodetoface/edgetoedge/edgetoface distance`
:4422-4929, `unsupported combination` :5088, visibility :5453/:8852-8980/
:7156, convex hull :10515-11601, polytope intersection :10698/:11497,
`strange id_ptc` :10773-11601, `unexpected number of contact point`
:11030, `wtf pt_surf` :7879); `nb_ctc greater than max number of
contacts` :12659; verlet allocs :786-822/:1193-1265; adjac allocs
:3684/:6068/:6966/:8070/:12986; periodic-bounds faterrs :629-659,
:1990-2013; index getters :12456-12520. **Silent:** Cundall CP budget
exhausted → pair dropped, `compute_cundall_common_plane = 0` :9351
(diagnostic gated on hard-coded `bavard=.false.`).
Wrap dispatch (`wrap_PRPRx.f90:155-176`): method 0 "You must specify",
1 "Obsolete", 5 "not implemented", default "unknown".
`max_nb_pt_select` (wti) has no default value :271 — only set via
`UseTrianglesIntersectionDetection`.

**SPSPx** (`mod_SPSPx.f90`): minray/maxray :410; box allocs :542/:550;
out-of-boxes :563-570 (note: 4 writes share one `cout` — only the last
line survives); **popul limit :575 with the int32 overflow at :535 (no
negative guard — unique to SPSPx)**; coincident sphere centres
:1178/:2472; verlet :1361-1569; `Very strange … nb_adj vs adjsz` :1705.
`this` is sized exactly — no LowSizeArray-style overflow exists here.

**SPPLx** (`mod_SPPLx.f90`): mirrors SPSPx (:371-579, int64 popul — safe);
`Unknown visibility table` :1484; `nickname <behav> … unknown in lawty`
:1499-1501; verlet :1083-1282.

**SPPRx** (`mod_SPPRx.f90`): mirrors PRPRx; `this` sized 1-per-rough-pair,
overflow banner :2533-2539; HE-proximity faterr :2816.

### A3. `mod_tact_behav.f90`

`Unknown interaction law` :9362; `nb of parameters is wrong` :363 (4
explanatory logmes lines before it); reader errors :2239/:2312/:2699/
:2721; seety errors :584/:2297; duplicate see rows :640;
`get_fric`/`get_rst` defaults :6418/:6455; **`get_coh` default commented
out :6528 — unsupported law = silent zero cohesion**; ~17 more `get_*`
"not implemented" defaults :6832-7345; CZM state machine `incompatible
law` :2946-5826 (not our laws); `Mu_EVOLUTION.DAT` :9493.

### A4. Wrap-layer semantics (`wrap_nlgs_3D.f90`)

`ExPrep` :348 unknown-storage faterr. `ExIter`/`ExIterJacobi` return
nothing. **`AfterIterCheck()` returns int 0 = converged, 1 = not
converged, −1 = NaN — and a −1 is silently ignorable by the caller**:
anyone hand-rolling the iteration loop (archive scripts) must test it.
`ExSolver` :447 unknown-storage, :501 diverged; `iconv` uninitialized if
`nb_block_iter ≤ 0` (§1 Gotcha #4); early `RETURN` at :479 skips
`Nullify_EntityList` when `nb_CDAN == 0` (leak, benign).

### A5. Memory that scales with contacts

| structure | size | abort site |
|---|---|---|
| `this(nb_CDAN)` (solver) | ≈ ≥480 B/contact | `mod_nlgs_3D.f90:589` |
| `Wab` (SDL only) | 72 B per contact-adjacency; int32 index caps ~2.4e8 pairs | :857 / :819 |
| PRPRx `this` | `size_factor × max(nb_POLYR, rough pairs)` × interaction struct | the 5 banners |
| broad-phase `box` grid | `Π(1+extent/Lbox)` cells × `maxpopul` ints | `mod_PRPRx.f90:2079` |
| verlet lists | Σ nb_adj per contactor | various |

Escape hatches: `Exchange_Local_Global` (no Wab at all);
`PRPRx_LowSizeArrayPolyr(N)`; a body at absurd coordinates is usually
what makes the box grid explode.

### A6. HDF5 (`mod_lmgc90_hdf5.f90`, `mod_h5_*.f90`)

Fatal: open-read missing :176 / unreadable :174; open-write :215 /
create :218; `Version not found` :824; version shape :848/:851; group
open :1181/:1187; `Corrupted file, T_ini/T_end` :324/:325;
`unknown format: X.Y` (`mod_h5_format.f90:297/:301` — newer-build file;
message truncated to 20 chars); `read_bodies before dof` + body-count
mismatch (`mod_h5_rigid.f90:271/:275`).
**Silent:** nonexistent record → logmes + no-op read
(`mod_lmgc90_hdf5.f90:466`, `mod_lmgc90_hdf5_hl.f90:197`);
interaction-id OOB stack write on enum-mismatched files
(`mod_h5_interaction.f90:625-629`) → SIGSEGV/corruption — the
`io_hdf5_FixVersion(0)` scenario.

### A7. OpenMP shared-state hazards (`mod_nlgs_3D.f90:2719-4890`)

Race families in §3. Specifics: missing from PRIVATE — `is_traction`
(:3521/:3549 vs :4455/:4491), `snmax` (:2963/:3967), `kkk` (:3364),
`mu_g` (:3359), `cout` (:4495); implicitly-SAVEd reduction vars
`WRRmin/WRRmax` :2680; non-atomic body accumulation
`bdyty%Ireac = bdyty%Ireac + xxreac` (`mod_RBDY3.f90:3121-3141` via
`injj_` :5091, same for `Vaux` via `vitrad_` :5112); `SCHEDULE(RUNTIME)`
:2735; faterrs inside the region (§6b). The threadprivate dispatch in
`mod_inter_handler_3D.f90:285-339` is correct — not the problem.
`WITH_OPENMP` defaults OFF upstream, and the upstream test suite pins
`OMP_NUM_THREADS=1` — the parallel sweep is not a supported-by-testing
configuration.

---

*Project-level decisions derived from this guide (serial-only solver,
retry-ladder shape, restart hygiene) are logged in `context/DECISIONS.md`
(2026-08-30 entries).*
