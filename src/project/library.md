# The bed library

_Regenerated from the machines' reports every time the site is published._

A settled bed can take days of computer time, so we keep each one and reuse
it. The bed library is how: it treats a settled bed as a protected file you
can name, check, and carry to another machine.

**What it does for you**

- **Every settled bed is registered.** When a bed passes its
  [quality checks](../glossary.md#quality-checks-qa) (PASS or WARN), it goes
  into the library with its name, seed, grain count, grade and date.
- **Beds are protected.** The files a drop reads are made read-only, and
  their checksums are saved. Every use checks them first: a changed or
  damaged bed is refused with a message, never used silently.
- **You can name a bed directly.** A drop set can say which bed it wants by
  name (`@ladder_L1_uniform:1`, name and seed) or by its id, instead of
  through the recipe. That bed is used as it is, even if the recipe or a
  default changes later.
- **Beds can move between machines.** Export one to a file, copy it, import
  it. A bed settled for days on the lab Mac can then be used on the laptop.

**The commands**

| to do this | run |
|---|---|
| see every bed on this machine | `python3 lab.py beds list` |
| check that none has changed | `python3 lab.py beds verify` |
| pack a bed into one file | `python3 lab.py beds export <bed id> <file>.tar.gz` |
| bring it in on another machine | `python3 lab.py beds import <file>.tar.gz` |

A worked example is in the code repository, `examples/08_bed_library.sh`. The
library lives in `src/milani/bedlib.py`, and its list on each machine is
`runs/beds/LIBRARY.json`.

**A bed's id is its recipe.** The eight characters at the end of an id are a
hash of the recipe, so a drop can never run on the wrong ground by accident.
When a recipe's meaning changes, the id changes with it: that is how the
beds poured wrongly before 2 November ([the bug](plan.md#a-bug-in-the-pour-found-2-november))
stay in the library but are no longer picked by their recipes. They can still
be reached by naming them, so check the "pour" column below first.

**The code underneath.** All of this sits on `src/lmgcx`, our library that
makes LMGC90 safe to drive from Python (boot order, contact tables, pouring,
height maps, quality checks), and `src/milani`, the study built on it. The
guide's chapter [A pipeline on top](../pipeline-api.md) explains both, with
code.

## Study and standard beds on the lab Mac

| bed id | seed | size | grains | settle | QA | pour |
|---|---|---|---|---|---|---|
| `ladder_L1_uniform-s1-016730e7` | 1 | 4.5 m × 4.5 m | 1,043 | 5,000 steps, 1.4 h | PASS | good |
| `ladder_L1_uniform-s2-c0102e9e` | 2 | 4.5 m × 4.5 m | 1,046 | 7,900 steps, 2.8 h | WARN | good |
| `ladder_L1_uniform-s3-322f7e49` | 3 | 4.5 m × 4.5 m | 1,044 | 10,000 steps, 4.0 h | WARN | good |
| `ladder_L2_weibull-s1-f18a8ca2` | 1 | 4.5 m × 4.5 m | 2,097 | 35,600 steps, 68.6 h | WARN | **lopsided pour, do not use** |
| `ladder_L2_weibull-s2-761bed88` | 2 | 4.5 m × 4.5 m | 2,116 | 26,400 steps, 52.3 h | WARN | **lopsided pour, do not use** |

## Study and standard beds on the laptop

| bed id | seed | size | grains | settle | QA | pour |
|---|---|---|---|---|---|---|
| `ladder_L1_uniform-s1-016730e7` | 1 | 4.5 m × 4.5 m | 1,043 | 5,000 steps, 0.8 h | PASS | good |

## Experiment beds

Settled for one test each; kept because a settled bed is expensive.

| machine | bed id | seed | size | grains | settle | QA | pour |
|---|---|---|---|---|---|---|---|
| laptop | `exp_3m_capwin-s1-6fdae8a7` | 1 | 3.0 m × 3.0 m | 463 | 4,800 steps, 0.3 h | PASS | good |
| laptop | `exp_3m_tol4-s1-5272f472` | 1 | 3.0 m × 3.0 m | 463 | 5,600 steps, 1.4 h | PASS | good |
| laptop | `exp_4m-s1-773cdcc5` | 1 | 4.0 m × 4.0 m | 824 | 9,600 steps, 1.5 h | PASS | good |
| laptop | `exp_4m-s2-4457e602` | 2 | 4.0 m × 4.0 m | 827 | 9,500 steps, 1.6 h | WARN | good |
| laptop | `exp_4m_capwin-s1-898a237d` | 1 | 4.0 m × 4.0 m | 824 | 5,700 steps, 0.8 h | PASS | good |
| laptop | `exp_base_v12-s1-bdea77c6` | 1 | 1.5 m × 1.5 m | 116 | 5,500 steps, 0.1 h | PASS | good |
| laptop | `exp_bed2_3m-s1-a7060bb1` | 1 | 3.0 m × 3.0 m | 932 | 15,100 steps, 4.7 h | PASS | **lopsided pour, do not use** |
| laptop | `exp_bed4q_2m-s1-c962607f` | 1 | 2.0 m × 2.0 m | 1,132 | 14,100 steps, 6.6 h | WARN | **lopsided pour, do not use** |
| laptop | `exp_fines_only-s1-effa0302` | 1 | 1.0 m × 1.0 m | 353 | 1,500 steps, 0.0 h | WARN | good |
| laptop | `exp_fines_only_gap-s1-042219c9` | 1 | 1.0 m × 1.0 m | 353 | 1,800 steps, 0.0 h | WARN | good |
| laptop | `exp_fines_thick-s1-87337ac8` | 1 | 1.0 m × 1.0 m | 882 | 1,900 steps, 0.0 h | WARN | good |
| laptop | `exp_fines_thick_gap-s1-115dac60` | 1 | 1.0 m × 1.0 m | 882 | 5,000 steps, 0.9 h | PASS | good |
| laptop | `exp_fric03-s1-be42dd53` | 1 | 1.5 m × 1.5 m | 116 | 8,600 steps, 0.1 h | PASS | good |
| laptop | `exp_settle_rmix_base-s1-91a3f0ca` | 1 | 2.0 m × 2.0 m | 414 | 14,500 steps, 1.8 h | WARN | **lopsided pour, do not use** |
| laptop | `exp_settle_rmix_budget-s1-f02aa087` | 1 | 2.0 m × 2.0 m | 414 | 13,400 steps, 1.0 h | PASS | **lopsided pour, do not use** |
| laptop | `exp_smoke_v20-s1-c5f17fe7` | 1 | 2.4 m × 2.4 m | 419 | 3,100 steps, 0.0 h | WARN | good |
| laptop | `exp_smoke_v8-s1-dc0e523a` | 1 | 2.4 m × 2.4 m | 419 | 9,300 steps, 0.5 h | WARN | good |
| laptop | `exp_stab_base-s1-593b15bf` | 1 | 1.5 m × 1.5 m | 116 | 25,000 steps, 0.3 h | PASS | good |
| laptop | `exp_stab_cap015-s1-bfbb8eb4` | 1 | 1.5 m × 1.5 m | 116 | 8,200 steps, 0.1 h | WARN | good |
| laptop | `exp_stab_dt3-s1-e973b10e` | 1 | 1.5 m × 1.5 m | 116 | 2,900 steps, 0.0 h | WARN | good |
| laptop | `exp_stab_dt4-s1-fc38cd8d` | 1 | 1.5 m × 1.5 m | 116 | 4,300 steps, 0.0 h | WARN | good |
| laptop | `exp_stab_scramble-s1-d6c5d154` | 1 | 1.5 m × 1.5 m | 116 | 26,900 steps, 0.3 h | PASS | good |
| laptop | `exp_stab_theta1-s1-2027a327` | 1 | 1.5 m × 1.5 m | 116 | 25,200 steps, 0.4 h | PASS | good |
| laptop | `exp_stab_tol3e4-s1-cc919f61` | 1 | 1.5 m × 1.5 m | 116 | 5,900 steps, 0.1 h | WARN | good |
| laptop | `exp_stab_tol4-s1-21d58f00` | 1 | 1.5 m × 1.5 m | 116 | 27,600 steps, 1.0 h | PASS | good |
| laptop | `exp_stab_tol4_dt3-s1-8b683f34` | 1 | 1.5 m × 1.5 m | 116 | 18,800 steps, 1.0 h | WARN | good |
| laptop | `exp_stab_tol4_mc100-s1-8dce3e40` | 1 | 1.5 m × 1.5 m | 116 | 7,800 steps, 0.1 h | WARN | good |
| laptop | `exp_v20-s1-513d10b3` | 1 | 1.5 m × 1.5 m | 77 | 5,600 steps, 0.0 h | PASS | good |
| laptop | `exp_win_base-s2-3c1316b3` | 2 | 1.5 m × 1.5 m | 116 | 7,900 steps, 0.1 h | PASS | good |
| laptop | `exp_win_base-s3-a74e734e` | 3 | 1.5 m × 1.5 m | 116 | 4,600 steps, 0.0 h | PASS | good |
| laptop | `exp_win_base-s4-0ad8b350` | 4 | 1.5 m × 1.5 m | 116 | 6,100 steps, 0.1 h | PASS | good |

