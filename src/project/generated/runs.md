# Every run so far

Generated from the reports in the code repository; see [Where things stand](state.md) for what they mean and what is next. Production beds are the six beds of the study; everything else is an experiment that answered one question (its question is the first line of its config). Settle time is wall time on the machine named.

## Beds settled on the lab Mac

### Test beds (smoke checks)

| bed | what it is for | size | grains | settle | QA |
|---|---|---|---|---|---|
| `micro` seed 1 | test-only: ~40 boulders, settles in seconds | 0.9 m | 232 | 3,300 steps, 0.0 h | PASS |
| `micro` seed 1 | test-only: ~40 boulders, settles in seconds | 0.9 m | 232 | 1,400 steps, 0.0 h | PASS |
| `smoke` seed 1 | 8-run engine drill on the smoke bed (not science) | 2.4 m | 419 | 4,000 steps, 0.1 h | WARN (overlap_rel_fines WARN) |
| `smoke` seed 1 | 8-run engine drill on the smoke bed (not science) | 2.4 m | 827 | 7,700 steps, 0.2 h | WARN (overlap_rel_fines WARN) |

## Beds settled on the laptop

### Production beds

| bed | what it is for | size | grains | settle | QA |
|---|---|---|---|---|---|
| bed 1 (`ladder_L1_uniform`) seed 1 | uniform boulders 0.15-0.35 m, 0.9 m deep | 4.5 m | 1,043 | 5,000 steps, 0.8 h | PASS |

### Experiment beds

| bed | what it is for | size | grains | settle | QA |
|---|---|---|---|---|---|
| `exp_3m_capwin` seed 1 | 3 m bed 1 geometry, cap kept through the window, rest at 3 mm/s | 3.0 m | 463 | 4,800 steps, 0.3 h | PASS |
| `exp_3m_tol4` seed 1 | timing: 3 m version of study bed 1 | 3.0 m | 463 | 5,600 steps, 1.4 h | PASS |
| `exp_4m` seed 1 | laptop rehearsal of the drop set configuration: 4 m bed-1 bed | 4.0 m | 824 | 9,600 steps, 1.5 h | PASS |
| `exp_4m` seed 2 | laptop rehearsal of the drop set configuration: 4 m bed-1 bed | 4.0 m | 827 | 9,500 steps, 1.6 h | WARN (stage1_rest_window WARN) |
| `exp_4m_capwin` seed 1 | 4 m bed 1 geometry, cap kept through the window, rest at 3 mm/s | 4.0 m | 824 | 5,700 steps, 0.8 h | PASS |
| `exp_base_v12` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 5,500 steps, 0.1 h | PASS |
| `exp_bed2_3m` seed 1 | bed 2 at 3 m: 90 cm of Weibull boulders | 3.0 m | 932 | 15,100 steps, 4.7 h | PASS |
| `exp_bed4q_2m` seed 1 | quick fines bed at 2 m: 50 cm of 0.10-0.25 m boulders + 8 cm fines veneer, split law | 2.0 m | 1,132 | 14,100 steps, 6.6 h | WARN (overlap_rel_fines WARN, solver_unconverged_fraction WARN, fines_coverage WARN) |
| `exp_fines` seed 1 | experiment: fines veneer cost and overlap | 2.0 m | 1,415 | 12,200 steps, 1.1 h | FAIL (overlap_rel_fines FAIL) |
| `exp_fines_only` seed 1 | experiment: fines-only layer, velocity rule vs gap rule | 1.0 m | 353 | 1,500 steps, 0.0 h | WARN (overlap_rel_fines WARN, bare_floor_fraction WARN) |
| `exp_fines_only_gap` seed 1 | experiment: fines-only layer, velocity rule vs gap rule | 1.0 m | 353 | 1,800 steps, 0.0 h | WARN (bare_floor_fraction WARN) |
| `exp_fines_thick` seed 1 | experiment: 25 cm of 4-8 cm fines only (bed 6 top layer, bed 5 substrate depth): velocity rule vs gap rule | 1.0 m | 882 | 1,900 steps, 0.0 h | WARN (overlap_rel_fines WARN) |
| `exp_fines_thick_gap` seed 1 | experiment: 25 cm of 4-8 cm fines only (bed 6 top layer, bed 5 substrate depth): velocity rule vs gap rule | 1.0 m | 882 | 5,000 steps, 0.9 h | PASS |
| `exp_fric03` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 8,600 steps, 0.1 h | PASS |
| `exp_rst` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 5,100 steps, 0.0 h | FAIL (overlap_rel_boulders FAIL) |
| `exp_smoke_v20` seed 1 | plumbing-test bed: 2.4 m square, ~400 boulders + ~400 fines | 2.4 m | 419 | 3,100 steps, 0.0 h | WARN (overlap_rel_fines WARN) |
| `exp_smoke_v8` seed 1 | plumbing-test bed: 2.4 m square, ~400 boulders + ~400 fines | 2.4 m | 419 | 9,300 steps, 0.5 h | WARN (overlap_rel_fines WARN) |
| `exp_stab_base` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 25,000 steps, 0.3 h | PASS |
| `exp_stab_cap015` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 8,200 steps, 0.1 h | WARN (stage1_rest_window WARN) |
| `exp_stab_dt3` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 2,900 steps, 0.0 h | WARN (stage1_rest_window WARN) |
| `exp_stab_dt4` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 4,300 steps, 0.0 h | WARN (stage1_rest_window WARN) |
| `exp_stab_scramble` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 26,900 steps, 0.3 h | PASS |
| `exp_stab_theta1` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 25,200 steps, 0.4 h | PASS |
| `exp_stab_tol3e4` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 5,900 steps, 0.1 h | WARN (stage1_rest_window WARN) |
| `exp_stab_tol4` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 27,600 steps, 1.0 h | PASS |
| `exp_stab_tol4_dt3` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 18,800 steps, 1.0 h | WARN (stage1_rest_window WARN, solver_unconverged_fraction WARN) |
| `exp_stab_tol4_mc100` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 7,800 steps, 0.1 h | WARN (stage1_rest_window WARN) |
| `exp_v20` seed 1 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 77 | 5,600 steps, 0.0 h | PASS |
| `exp_win_base` seed 2 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 7,900 steps, 0.1 h | PASS |
| `exp_win_base` seed 3 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 4,600 steps, 0.0 h | PASS |
| `exp_win_base` seed 4 | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders | 1.5 m | 116 | 6,100 steps, 0.1 h | PASS |

### Test beds (smoke checks)

| bed | what it is for | size | grains | settle | QA |
|---|---|---|---|---|---|
| `micro` seed 1 | test-only: ~40 boulders, settles in seconds | 0.9 m | 232 | 1,400 steps, 0.0 h | PASS |
| `micro` seed 7 | test-only: ~40 boulders, settles in seconds | 0.9 m | 231 | 2,200 steps, 0.0 h | PASS |
| `micro` seed 8 | test-only: ~40 boulders, settles in seconds | 0.9 m | 233 | 1,600 steps, 0.0 h | PASS |
| `micro_layers` seed 1 | test-only: fines substrate + boulders + veneer, 0.7 m square (~1,000 grains) | 0.7 m | 209 | 3,000 steps, 0.0 h | WARN (overlap_rel_fines WARN) |
| `smoke` seed 1 | 8-run engine drill on the smoke bed (not science) | 2.4 m | 419 | 4,000 steps, 0.1 h | WARN (overlap_rel_fines WARN) |

## Experiment configs with no bed card here

Stopped on purpose, failed, settled before bed cards existed, still running, or run on the lab Mac (whose cards arrive when it syncs); the lab notes say which.

| config | what it was for |
|---|---|
| `3m_budget100.toml` | timing: 3 m version of study bed 1 |
| `4m_tol4.toml` | 4 m bed 1 settled at NLGS tolerance 1e-4: does the confirmation window hold? |
| `bed2_3m_b.toml` | bed 2 at 3 m: 90 cm of Weibull boulders |
| `bed2_3m_cap015.toml` | bed 2 at 3 m with the speed cap at 0.15 m/s: does a gentler cap end the churn sooner? |
| `bed4_3m.toml` | bed 4 at 3 m: bed 2's boulders + 8 cm fines veneer, split law |
| `bed4u_3m.toml` | bed 4 on uniform boulders at 3 m: 90 cm of 0.15-0.35 m boulders + 8 cm fines veneer, split law |
| `bigpolyr_probe.toml` | 2 m fines substrate + Weibull boulders: mean radius 0.09 m, boulders above 4x it (big POLYR) |
| `bigpolyr_repro.toml` | 2 m: 4-8 cm fines substrate + Weibull boulders: mean radius 0.08 m, largest 4.6x it (big POLYR at the default ratio) |
| `bigpolyr_split.toml` | 2 m fines substrate + Weibull boulders with the split law (bed 5 as configured for the weekend) |
| `dt1.toml` | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders |
| `fines_gap.toml` | experiment: fines veneer cost and overlap |
| `fines_split.toml` | experiment: 2 m layered bed, gap law for boulder-fines pairs only, velocity law among fines |
| `fines_split_substrate.toml` | experiment: 2 m bed, 30 cm fines substrate under 0.5 m of boulders (bed 5 structure), gap law for boulder-fines pairs only |
| `fines_split_thick.toml` | experiment: 2 m layered bed, 25 cm fines veneer (bed 6 top layer), gap law for boulder-fines pairs only |
| `fric09.toml` | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders |
| `v8.toml` | thick-layer test: 1.5 m footprint, 0.9 m of bed-1 boulders |

## Drop sets

| drop set | machine | what it is for | drops | outcomes | median e | CPU time | in the reel |
|---|---|---|---|---|---|---|---|
| `exp_battery_4m` | laptop | study drop set on bed 1: 9 spots x 2 speeds x 2 angles, fixed pose | 8 | 7 detached, 1 left patch | 0.425 | 0.3 h |  |
| `exp_battery_L1_s1` | laptop | the standard 36 drops on bed 1 (uniform boulders 0.15-0.35 m, 0.9 m deep) | 36 | 32 detached, 4 left patch | 0.342 | 2.3 h | [open](../drops/index.html#set-exp_battery_L1_s1) |
| `exp_drops_4m_capwin` | laptop | standard drops on the 4 m capped-window bed: 9 spots (+-0.05 m) x 2 speeds x 2 angles | 36 | 34 detached, 2 left patch | 0.312 | 0.8 h |  |
| `exp_drops_4m_capwin_r2` | laptop | the 36 drops again under the rotation-aware look-ahead and the patch-edge rule (2026-10-31) | 36 | 32 detached, 4 left patch | 0.303 | 2.1 h | [open](../drops/index.html#set-exp_drops_4m_capwin_r2) |
| `exp_drops_bed2_3m` | laptop | standard drops on the 3 m bed-2 twin (bare Weibull boulders): 9 spots (+-0.05 m) x 2 speeds x 2 angles, 0.7 m crop — the pair of exp_drops_bed4_3m on the same boulders | 36 | 32 detached, 4 left patch | 0.432 | 0.6 h | [open](../drops/index.html#set-exp_drops_bed2_3m) |
| `exp_drops_bed4q_2m` | laptop | stowed-craft drops on the 2 m quick fines bed: the centre spot x 2 speeds x 2 angles x 3 azimuths, 0.45 m crop | 12 | 8 detached, 4 left patch | 0.307 | 0.4 h | [open](../drops/index.html#set-exp_drops_bed4q_2m) |
| `smoke` | laptop | 8-run engine drill on the smoke bed (not science) | 8 | 5 detached, 3 left patch | 0.186 | 0.1 h |  |

## Single test drops

Sensitivity tests and smoke checks, one drop each (29-30 October).

| drop | machine | outcome | QA | e |
|---|---|---|---|---|
| `2026-10-29_smoke_A` | lab Mac | captured | WARN | 0.031 |
| `2026-10-29_example_2` | laptop | detached | PASS | 0.276 |
| `2026-10-29_froude_g1e-4` | laptop | detached | PASS | 0.400 |
| `2026-10-29_froude_g2.5e-5` | laptop | detached | PASS | 0.400 |
| `2026-10-29_froude_g5e-5` | laptop | detached | PASS | 0.400 |
| `2026-10-29_sensB_regolith_cohesive_100pa` | laptop | left patch | WARN | 0.366 |
| `2026-10-29_sensB_regolith_cohesive_10pa` | laptop | left patch | WARN | 0.372 |
| `2026-10-29_sensB_regolith_fric04` | laptop | left patch | WARN | 0.222 |
| `2026-10-29_sensB_regolith_fric07` | laptop | left patch | WARN | 0.361 |
| `2026-10-29_sensB_regolith_rst03` | laptop | left patch | WARN | 0.352 |
| `2026-10-29_sensB_regolith_rst07` | laptop | detached | WARN | 0.309 |
| `2026-10-29_sensB_regolith_v1` | laptop | left patch | WARN | 0.313 |
| `2026-10-29_sens_regolith_cohesive_100pa` | laptop | detached | PASS | 0.457 |
| `2026-10-29_sens_regolith_cohesive_10pa` | laptop | detached | PASS | 0.455 |
| `2026-10-29_sens_regolith_fric04` | laptop | left patch | WARN | 0.157 |
| `2026-10-29_sens_regolith_fric07` | laptop | detached | PASS | 0.433 |
| `2026-10-29_sens_regolith_rst03` | laptop | detached | PASS | 0.399 |
| `2026-10-29_sens_regolith_rst07` | laptop | detached | PASS | 0.401 |
| `2026-10-29_sens_regolith_v1` | laptop | detached | PASS | 0.400 |
| `2026-10-29_smoke_A` | laptop | detached | WARN | 0.175 |
| `2026-10-29_smoke_A_2` | laptop | detached | WARN | 0.175 |
| `2026-10-29_smoke_A_3` | laptop | detached | WARN | 0.175 |
| `2026-10-29_smoke_A_4` | laptop | detached | WARN | 0.175 |
| `2026-10-29_smoke_B` | laptop | left patch | WARN | 0.421 |
| `2026-10-29_smoke_B_2` | laptop | left patch | WARN | 0.421 |
| `2026-10-29_vert_v20` | laptop | detached | PASS | 0.506 |
| `2026-10-29_vert_v8` | laptop | left patch | WARN | 0.165 |
| `2026-10-30_frames` | laptop | ? | ? | — |
| `2026-10-30_frames2` | laptop | capped | WARN | 0.316 |

## Overnight slates on the lab Mac

### 2026-10-29_1514 (lab Mac)

| terminal | started | how it ended |
|---|---|---|
| job-01 | bed ladder_L1_uniform seed 1 | STOPPED by Ctrl+C |
| job-02 | bed ladder_L2_weibull seed 1 | START  bed ladder_L2_weibull seed 1 |
| job-03 | bed ladder_L3_deep seed 1 | START  bed ladder_L3_deep seed 1 |
| job-04 | bed ladder_L4_veneer seed 1 | STOPPED by Ctrl+C |
| job-05 | bed ladder_L5_substrate seed 1 | DONE   terminal 5 |
| job-06 | bed ladder_L6_thick seed 1 | STOPPED by Ctrl+C |
| job-07 | — | DONE   terminal 7 |
| job-08 | — | DONE   terminal 8 |
| job-09 | — | DONE   terminal 9 |
| job-10 | — | DONE   terminal 10 |
| job-11 | — | DONE   terminal 11 |
| job-12 | — | DONE   terminal 12 |
| job-13 | — | DONE   terminal 13 |
| job-14 | — | DONE   terminal 14 |
| job-15 | — | DONE   terminal 15 |
| job-16 | — | DONE   terminal 16 |

### rehearsal_2026-10-29_1416 (laptop)

| terminal | started | how it ended |
|---|---|---|
| job-01 | campaign smoke shard 1/3 | DONE   terminal 1 |
| job-02 | campaign smoke shard 2/3 | DONE   terminal 2 |
| job-03 | campaign smoke shard 3/3 | DONE   terminal 3 |

### rehearsal_2026-10-29_1450 (laptop)

| terminal | started | how it ended |
|---|---|---|
| job-01 | campaign smoke shard 1/3 | DONE   terminal 1 |
| job-02 | campaign smoke shard 2/3 | DONE   terminal 2 |
| job-03 | campaign smoke shard 3/3 | DONE   terminal 3 |
