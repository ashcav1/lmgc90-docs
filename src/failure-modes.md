# Failure modes catalogue

_Started 2026-10-29 from the first production night and the laptop
experiments that followed; each entry is something that actually happened.
Add to it whenever a run fails for a new reason. The QA checks in
`milani/pipeline.py` and the settle invariants in `milani/beds.py` turn most
of these into hard failures with a message, so a silent wrong result is the
failure to fear most — several below used to be silent._

## Settling beds

| what happened | how it showed | cause | avoid / detect |
|---|---|---|---|
| 16 beds settled all night, none finished | every window still "depositing" at 07:00; 20–30 s per step | grains were deposited as proxy spheroids: a 6 m column for a 1.2 m layer, 29k contacts × 2–4k NLGS iterations per step, 16 processes sharing the Mac | real-shape deposition (`deposit_polyhedra`, 64 candidates: column 1.7× the layer); one bed per window; `lab.py estimate` before a slate |
| thick bed never reaches "every grain < 2 mm/s" | p99 speed 0.1–0.3 m/s for ever, 130–260 of 463 boulders "moving", no overlap, no escape, solver converged | a pile several grains deep chatters: the common-plane contact set flickers ±5 contacts per 50 steps and re-kicks the pile; thin beds damp it, thick ones don't | rest is structural (`rest_disp_m`: median displacement over 1 s < 1 mm); `n_moving`, `d50/d95` columns in `settle_metrics.csv` show it |
| bed at rest explodes within a second | 150–355 boulders at 0.6–3.4 m/s right after `capped` turns 0 | the speed cap was lifted; uncapped contact dynamics of a deep polyhedral pile are unstable (clip is inert at rest, yet removing it blows the pile up — four times, any solver budget) | never lift the cap; the final phase only switches to the full solver |
| bed at rest explodes at release (variant) | same, with `deposit_dt` ≠ `dt` | the time step changed mid-settle (stored reactions scale with dt) | one dt per settle (`deposit_dt` removed) |
| velocity-law boulders fail QA | `overlap_rel_boulders FAIL 0.196` (limit 0.10) after a clean settle | RST_CLB only stops approach velocity; dense polyhedra drift 5–16 mm into each other | boulders settle under GAP_SGR_CLB (gap law, ~0 overlap); RST stays for fines pairs (gap-law fines drain through pores) |
| settle crashes with NaN | `nlgs_diverged: a NaN appeared in contact reactions`, stage retried | friction 0.9 with the gap law on rigid polyhedra: the NLGS diverged | keep settle friction ≤ 0.6–0.7 (Dimorphos angle of repose 30–35° → μ 0.55–0.7); the crash detector names it; one retry with the same settings |
| bed looks settled but every boulder lies on the floor | surface 5.6 cm for a 17 cm layer, `POLYR-POLYR` contacts = 0, QA only warned | face-to-face detection (`PRPRx_UseCpF2fDetection` / `...Explicit`) finds no contacts on triangulated random hulls | Cundall common-plane detection only; QA now FAILS on < 1 grain–grain contact per grain or a layer that lost > 40 % of its height |
| budget "fix" that wasn't | with a 1,000-iteration NLGS budget the pile also exploded at release | the budget was a false lead: the explosion is the cap removal, not the solve | keep `deposit_max_checks = 0` on thick layers anyway (converged reactions); thin test beds may pin 100 |
| a rerun doubles the cost and loses the evidence | after `max_steps` the stage was wiped and rerun at half dt / double steps | old retry policy | only crashes/stalls retry; a settle that ran out of steps fails with its stage dir kept |
| bed id changes under a running campaign | the campaign builds a *second* bed with a new id; old runs orphaned | a field was added to / removed from a hashed config dataclass (SettleSpec, BedSpec, …) | never change hashed dataclasses while beds or runs exist; configs pin values instead; `PHYSICS_VERSION` bump only when results must change |
| 8-vertex grains chatter longer than 12-vertex | more "moving" grains for longer on the same bed | fewer, larger faces → contact points jump more | 12 vertices (default); 20 settles equally well at ~1.4× the cost (see experiments) |
| smaller time step does not help rest | dt 1 ms: still chattering after 16 s simulated, twice the steps | chatter is a contact-set effect, not a dt effect | keep dt 2 ms for settling |

## Impacts

| what happened | how it showed | cause | avoid / detect |
|---|---|---|---|
| spacecraft flies through a wall | box leaves the bed with no contact | no see-table row for that pair → pair never detected, no error | every pair the physics needs gets a row (scene `invisible_walls` pins it) |
| NaN at the first step | `Wnn = 0` | two FIXED bodies with a contact rule between them | frozen–frozen and frozen–floor pairs get no rule (scene `frozen_pairs`) |
| craft starts inside the terrain | `start_overlap` | aim point too close to a boulder top | the composer ray-casts every vertex and starts 2 cm clear; the check still runs |
| `left_patch` | the box left the free core low | crop too small for the bounce | rerun with a larger crop (`--crop`), use `shell_mode = "adjacency"` |
| crop too close to a wall | `target … is X m from a wall; the crop needs Y m` | bed margin: wall-affected grains | usable half-width from `describe`; sites inside it |
| spheres sink into boulders | SPPRx misses approaching contacts | LMGC90 sphere–polyhedron detection | all grains are polyhedra |
| random SIGBUS in detection | crash in `PRPRx_SelectProxTactors` | `polytopes_area_oob` in LMGC90 | `lab.py patch-lmgc90` (required on every machine) |

## Cohesion (gate D10, scene `cohesion`)

| what happened | how it showed | cause | avoid / detect |
|---|---|---|---|
| no bond at all | both cubes rose the free-flight distance | `cohn` is a **stress** (Pa) × detected contact area, not a force | give cohesion in Pa; the scene pins the bond = weight × 2 holding |
| bond for one step, then free | reaction = exactly the weight at step 1, then 0; status `slide` | tangential cohesion 0 → first tangential tug → `slide` → a sliding contact loses its bond | `coht = cohn` |
| bond on the floor carries no force | status `Mstck`, reaction 0 | polyhedron–plane contacts have no measured area | cohesion only between polyhedra (PRPRx); never rely on floor cohesion |
| cohesion silently equal to no cohesion in restarts | — | bonds form only at step 1 on virgin statuses | cohesive impact decks must reset the step counter and skip `RecupRloc` at step 1 (DECISIONS 2026-08-30) |

## Operations

| what happened | how it showed | avoid |
|---|---|---|
| a window blocks for days | shard window waiting on the bed lock of a rung another window is settling | `job.sh` asks `lab.py campaign ready` and revisits every 10 min |
| `pkill -f "exec bed-settle"` killed the waiting chain | the chain's own command text matched | stop runs by config name: `pkill -INT -f "lab.py bed --bed .*NAME.toml"` |
| OneDrive sync eats 1.6 cores on the laptop | every run-folder write synced | keep `runs/` outside OneDrive on a production machine (lab Mac: external drive) |
