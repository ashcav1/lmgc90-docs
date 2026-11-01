# The six beds: how much terrain realism does the bounce need?

_2026-10-29, revised 2026-10-30 after the first night settled nothing (see
"What the first night taught", below). Everything here is one command
(`OVERNIGHT_HOURS=60 bash overnight/launch.sh`); this note says what it will
do, why, what it costs and what to look at afterwards._

## Why this study

The first study night tried to settle sixteen 8 m beds at once and
finished none of them (DECISIONS 2026-10-29). Two lessons shaped this study:

1. **A bed is settled once and reused.** Every impact patches its own patch of
   the same settled bed. The advisor's question, "same arrival, different
   landing spot, same bed", needs exactly one bed per terrain model.
2. **We do not know how much realism matters.** A Dimorphos bed can be
   described at many levels of fidelity: boulders in the even size mix; the real
   (Weibull) size distribution; the depth below the surface; a thin top layer of
   small stones on top; small stones *under* the boulders; small stones filling the pores. Each
   step costs more to settle and to simulate. The six beds measures, bed by
   bed, what each one changes in the bounce statistics and what it costs,
   so the position study can be run at the lowest fidelity that is enough.

**Terms.** Sizes are lengths (a grain's longest dimension; the bed files
give half of it). *Boulders* are the beds' 20–70 cm grains, the small end of
the boulder field DART imaged on Dimorphos (its images count boulders down
to about 0.5 m). *Small stones* are the 8–20 cm grains between and on the
boulders. They stand in for Dimorphos's finer material (gravel, sand,
dust), which lies below DRACO's ~3 cm resolution, so its amount and
placement are assumptions; real fine-grained material is far too numerous
to simulate. In the pipeline both are polyhedral grains, and their kind
(`boulder` or `fines`) picks their contact rule and QA.

## Terms used below

| term | meaning |
|---|---|
| **bed** | a settled patch of terrain: grains dropped into a box and left to come to rest under gravity, saved once, reused by every impact (`runs/beds/<bed id>/`) |
| **settle** | letting the dropped grains fall and stop; a bed is "settled" when its geometry has stopped changing |
| **layer** | one deposited material of a bed, listed bottom-up in its config (`[[layers]]`): kind `boulder` or `fines`, thickness, sizes |
| **small stones** | grains 8–20 cm long between and on the boulders, standing in for Dimorphos's finer material; not fine-grained (sand and gravel are far too small and numerous to simulate) |
| **the six beds** | six beds of increasing realism (config files `ladder_L1` … `L6` — historical name); "bed k" is the k-th of them |
| **36-drop set** | the same 36 landings fired at every bed: 9 landing spots × 2 speeds × 2 angles, same pose (config files `battery_L1` … `L6` — historical name) |
| **impact / run** | one spacecraft drop on one spot of one bed (`runs/single/`, or one row of a campaign) |
| **patch / shell** | the free patch of the bed simulated around the landing spot, and the ring of grains around it held fixed |
| **campaign** | a list of impacts run from one config file, resumable, with a shared results table (`catalog.csv`) |
| **shard K/N** | one terminal's 1/N share of a campaign's runs (`campaign start X --shard K/N`) |
| **window** | one Terminal window = one simulation process = one core (overnight/) |
| **outcome** | how a run ended: `detached` (bounced off), `captured` (stuck, barely moving), `rest`, `left_patch` (left the patch on the ground: rerun with a larger patch; a craft flying out over the edge after its bounce is `detached`), `capped` (time limit) |
| **e_speed** | speed out ÷ speed in (the restitution number) |
| **position rule / speed rule** | two of LMGC90's built-in contact rules (`GAP_SGR_CLB`, `RST_CLB`). The position rule pushes two grains apart whenever they overlap — used for settling boulders (no overlaps left), but those small pushes are what makes a resting thick pile blow up later. The speed rule only stops grains from moving into each other and never pushes an existing overlap apart — used for impacts (calm) and for pairs with small stones |
| **rest test** | how the code decides a pile has stopped: either every grain is slower than 2 mm/s, or the grains have stopped moving anywhere (half of them moved less than 3 mm in the last second of simulated time, 95 % less than 6 cm, none more than 30 cm). The state at that first rest is saved; a short 500-step check follows with the speed limit still on |
| **QA** | the automatic checks on every settled bed before it may be used — no grain outside the box, grain-to-grain overlap below a limit (small-stone pairs graded where the craft can reach them; buried small stones only warn), every grain touching others, the layer height as expected — and on every run (patch edge disturbance, outcome); PASS / WARN / FAIL, and a FAIL bed is never used. A bed graded FAIL under older rules is re-graded from its settled layers, never re-settled |

## The six beds

All beds are 4.5 m × 4.5 m: a landing spot needs the 1.2 m patch plus two of
the largest grains (0.7 m) to the nearest wall, and the 3 × 3 grid of spots
sits at ±0.3 m, so 2.25 − 0.3 = 1.95 m ≥ 1.9 m. (The first night ran 6 m
beds with spots at ±0.6 m: three times the cost, and no bed settled.) Each
bed is one config file in `configs/beds/`, a list of `[[layers]]` from the
bottom up. Beds 1–4 are settled **three times** (seeds 1, 2, 3: terrain
varies the bounce more than any contact law does, DECISIONS 2026-10-29) and
the 36-drop set run on all three; beds 5 and 6 once (days per bed).

| bed | config | what it adds | layers (bottom → top) | grains (est.) | settled beds |
|---|---|---|---|---|---|
| 1 | `ladder_L1_uniform` | the simplest terrain | 90 cm of boulders 30–70 cm long, even size mix | 1,040 | 3 |
| 2 | `ladder_L2_weibull` | **real size distribution** (Pajola et al. 2024: many small, few large; 20–70 cm long) | 90 cm of boulders | 2,100 | 3 |
| 3 | `ladder_L3_deep` | **depth**: does what lies under the top boulders matter? | 150 cm of boulders | 3,500 | 3 |
| 4 | `ladder_L4_veneer` | **small stones on top** (the THIN state) | 90 cm of boulders + an 8 cm layer of small stones 8–16 cm long | 7,800 | 3 |
| 5 | `ladder_L5_substrate` | **layered regolith**: boulders resting on small stones, not on a rigid floor | 30 cm of small stones 10–20 cm long + 90 cm of boulders + 8 cm of small stones | 18,800 | 1 |
| 6 | `ladder_L6_thick` | **more small stones**, filling the pores (the THICK state) | 90 cm of boulders + 25 cm of small stones | 20,000 | 1 |

Beds 1–3 are bare boulder fields; 4–6 add small stones in three different ways.
Bed 2 is the reference: every later bed is bed 2 plus one change — literally:
beds 4 and 6 of a seed start from bed 2's settled boulders of that seed
(the pipeline copies the finished layer, `pipeline.stage_key`), so the
paired comparison 2 → 4 and 2 → 6 is the same terrain with and without
small stones. Pairs with small stones use the split contact rule (`boulder_fines_law`, below).

How a bed is built (`milani/beds.py`, `build_layer`): layer 1 is deposited
on the floor, settled under gravity until at rest; layer 2 is deposited on
the height map of the settled layer 1 and the whole thing settled again;
and so on. Deposition drops each grain's real shape onto the surface below
it at the lowest of 64 candidate spots (a column ~1.7× the final layer, no
overlaps: exact convex-intersection tests in the unit tests). While a layer
falls into place its speeds are capped at 0.3 m/s (measured 2026-10-29: a
0.9 m boulder layer rested in 6,800 steps at 0.3 m/s and 10,000 at 1.5 m/s —
harder landings agitate the pile longer; `speed_cap = 0` selects the
automatic rule, 0.4 of the smallest contact-detection distance per step).
The settle uses the position rule for boulder pairs (no interpenetration), the
position rule for boulder–stone pairs (the split contact rule of 2026-10-30: the velocity
law left −20 mm there) and the speed rule among small stones (the position rule among
small stones never rests over boulders; DECISIONS 2026-10-28/30). A layer is at
rest when its geometry stops changing — the median grain moved less than
3 mm over the last second of simulated time, 95 % of grains less than 6 cm
and none more than 30 cm — or the same for the load-bearing half of the
grains alone (those at or above the median size: on a realistic-size-mix pile the
small grains rattle in their pores for ever while the larger half the craft
meets is still, 2026-10-31) — or when every grain is slower than 2 mm/s.
The threshold was 1 mm until 2026-10-30: bed 1 of the first night reached 3 mm/s
at step 6,600 and crept to 1 mm/s by step 19,800, ten hours of 1 g creep
that a drop at 5e-5 g never sees (the creep timescale stretches by ~450:
ten seconds of 1 g compaction is over an hour at Dimorphos gravity, the
drops last 300 s). On the 27 saved laptop traces the 3 mm/s rest comes at
the same step for small beds (ratio 0.9–1.0) and 2–3× earlier for the 4 m
and 6 m ones, always with zero overlap. The state at that first rest is
saved at once, and a 500-step 500-step check follows in which nothing
changes — not the speed limit, not the solver. (Until 2026-10-30 the window
ran *uncapped*: a guard from the 10-28 rebuild survived the 10-29 "never
lift the cap" fix, and bed 1 blew apart 100 steps into its window exactly
as the 10-29 experiment predicted for an uncapped pile.) If the pile blows
up during the window the saved first-rest state is the bed and the event is
recorded in its card (`rest_window: WARN`). A first-rest bed is a valid packing (QA grades it),
and impacts never reproduce the instability: they use the speed rule,
start from zero velocity, at Dimorphos gravity. (Thick polyhedral piles never stop chattering at cm/s
in LMGC90 while their shape is static — DECISIONS 2026-10-29; the bed is
used at micro-g with its velocities zeroed, so static geometry is what
"settled" means.) A settle that puts a grain outside the box, or ends with
fewer than one grain–grain contact per grain, fails the bed outright.

## The 36-drop set: the same landings on every bed

`configs/campaigns/battery_L1.toml` … `battery_L6.toml` differ only in the
bed. Each is:

- **9 landing spots**: a 3 × 3 grid at x, y ∈ {−0.3, 0, +0.3} m from the
  bed centre (±0.6 m on the first night's 6 m beds);
- **2 speeds** (4 and 8 cm/s) × **2 angles** (0° and 40° from vertical);
- **the same pose and approach in every run** (`attitude = "fixed"`, bus
  axes aligned with the bed, long axis up, solar panels sideways; approach along
  +x); Dimorphos gravity 5e-5 m/s²; 1.2 m patch with the ring of touching grains
  (frozen = the grains touching the patch); 300 s cap.

So within a bed the only thing that varies is the landing spot, and
between beds the only thing that varies is the bed. The 36 drops run on
each of a type's settled beds (`bed_seeds = [1, 2, 3]`: 108 runs per type,
36 for bed 5). A bed is "read" by its 36 outcomes: how many detached, the median speed ratio with a
bootstrap CI, exit directions and spins, and `--compare` pairs each run
with the same run on another bed.

## The beds are kept

Each settled bed is registered in the lab Mac's bed library
(`runs/beds/LIBRARY.json`, `python3 lab.py beds`), checksummed and made
read-only. Everything that follows — the 36-drop set, the position
study, any future campaign — names a bed from the library
(`bed = "@ladder_L2_weibull"`) or by its id and only reads it. Nothing
re-settles a bed that exists, and a bed that was modified is refused.

## What comes out

After the night, `bash overnight/finish.sh` pushes to GitHub:

- `reports/labmac/studies/bed_ladder/summary.md` — one row per bed: grains,
  settle hours, surface mean height and roughness, what forms the top,
  worst overlap, QA; standard-drop runs, detached share, captured + rest share,
  median e with CI, minutes per run. (`python3 lab.py study beds`)
- `reports/labmac/beds/<bed id>/` — the bed card and figures for each bed
  (surface, settling curves, sizes, cross-section, 3-D, contact network).
- `reports/labmac/campaigns/battery_Lk/` — the campaign figures: sites,
  bounce map, restitution, exit state, energy, contact, terrain, qa.
- `python3 lab.py plot --compare battery_L2 battery_Lk` — the paired
  difference of bed k against the reference bed 2, Δe with a bootstrap
  95 % CI and the outcome agreement matrix.

The questions the table answers, in order: Did the size distribution change
the outcomes (1 → 2)? Did depth (2 → 3)? Did small stones on top (2 → 4)? Does it
matter whether the boulders sit on small stones (4 → 5)? Does more small stones change
it further (4 → 6)? And at what cost in settle hours and minutes per run?

## Cost

Measured on the lab Mac's first study night (2026-10-29, six beds of
6 m settling at once, one process per bed) and on the laptop:

| deck | grains | ms per step | NLGS iterations per step | steps to rest (1 mm/s) | steps to rest (3 mm/s) |
|---|---|---|---|---|---|
| laptop 3 m, boulders in the even size mix | 463 | 239 | ~520 | 7,200 | 4,800 |
| laptop 4 m, boulders in the even size mix | 824 | 553 | ~650 | 9,200 | 4,700 |
| lab Mac bed 1, 6 m, even size mix | 1,853 | 2,800 | ~1,000 | 19,800 | 6,600 |
| lab Mac bed 2, 6 m, realistic size mix | 3,729 | 8,500 | ~1,300 | not reached in 16.7 h (6,800 steps) | — |
| lab Mac bed 3, 6 m, realistic size mix, 1.5 m | 6,214 | 13,000 | ~1,500 | not reached in 16.7 h (4,600 steps) | — |
| lab Mac bed 5 bottom layer, 19,502 small stones, speed rule | 19,502 | 2,900 | 10–20 | 3,100 | — |

Three facts set the cost:

- **The cost per step grows faster than the grain count** (about
  grains^1.3): the NLGS needs more iterations on a bigger pile, and the
  gap-law boulder pairs set the iteration count for the whole deck — in a
  mixed deck the small-stone contacts (cheap on their own: 10–20 iterations under
  the speed rule) are swept 1,000+ times too. Bed 5's boulder layer over
  its small stones would have run at ~40 s per step at 6 m.
- **The steps to rest grow with the pile** (about the square root of the
  grain count): a wide bed creeps longer before its median move falls
  under the threshold. Hence 3 mm/s (above).
- **Deposit** (`deposit_polyhedra`, 64 candidates) is negligible: 0.4 s for
  463 boulders, 20 s for 31,800 small stones. The 0.3 m/s cap makes the rain-down
  from the column (1.7–2.9× the layer) last 6–13 s simulated; it is a
  minority of the steps.

`python3 lab.py estimate <bed config>` applies this model layer by layer
(±50 %). For the 4.5 m beds, one core each:

| bed | layers | hours (model) | note |
|---|---|---|---|
| 1 | 1,040 boulders | ~2 | |
| 2 | 2,100 boulders | ~8 | |
| 3 | 3,500 boulders | ~20 | |
| 4 | bed 2 copied + 5,700 small stones | ~25 (+0 for the copy) | the small-stone layer dominates: 7,800 grains swept at the boulders' iteration count |
| 5 | 10,900 small stones + 2,100 boulders + 5,700 small stones | ~160 | the boulder layer alone ~80 h at ~40 s per step; runs into the week |
| 6 | bed 2 copied + 17,900 small stones | ~85 | runs into the week |

So beds 1–3 (three seeds each) are the first day's result, bed 4 the
second day's; beds 5 and 6 keep settling past the weekend and their drops
run when they land (the windows never stop a settle at the cutoff, and
`launch.sh` the next time skips what is done). The small-stone beds are
expensive because the small stones are many (an 8 cm top layer of stones 8–16 cm long is
5,700 of them on 4.5 m); coarser or fewer small stones, or a smaller bed width
for beds 5–6 only, are the levers if the week is too long.

**Impacts** (measured on the laptop's 4 m rehearsal, 8 drops, and in
`lab.py estimate` for a campaign): a 1.2 m patch with the ring of touching grains
holds ~600 bodies and a drop takes 2 min (median) to 5 min (p90), 29 min
at the 300 s cap. The 576 runs of the slate (108 per type for beds 1–4 and
6, 36 for bed 5) are ~30 CPU-hours: an afternoon on 16 windows once the
beds exist.

## How to run it

On the lab Mac (overnight/README.md Parts A–B; the repo is at
`~/Projects/CSML_LMGC90`). First stop anything left from the previous
slate (Ctrl+C in each old window), then:

```bash
cd ~/Projects/CSML_LMGC90
```
```bash
git pull --rebase --autostash origin lab_mac
```
```bash
OVERNIGHT_HOURS=60 bash overnight/launch.sh
```

`OVERNIGHT_HOURS=60` lets new drops start all weekend (the default is 11 h;
a settle that has started always runs to its end regardless). What the 16
windows do (`overnight/plan.sh`): windows 1–14 each settle one bed, once —
bed 1 seeds 1–3, bed 2 seeds 1–3, bed 3 seeds 1–3, bed 4 seeds 1–3, bed 5
seed 1, bed 6 seed 1 — then join the drops; windows 15–16 go straight to the
drops and come back every 10 min until a bed's three seeds are settled.
The bed 4 and bed 6 windows print `waiting to copy it` and idle until bed 2
of their seed has settled (~8 h), then copy that layer and settle their
small stones on top. Expect: bed 1's 108 drops by the first evening, bed 2's that
night, bed 3's and bed 4's over the second day, beds 5 and 6 during the
week.

In the morning (README Part D): the regrade loop, then `bash
overnight/finish.sh`, then tell Claude. The six-beds table is in `reports/labmac/studies/bed_ladder/summary.md`.
`bash overnight/status.sh` at any time shows each bed's progress in words
and each campaign's counts.

## What the first night taught (2026-10-29 → 30)

The slate of 2026-10-29 (six 6 m beds, 16 windows, 16.7 h) settled no bed.
Reports: `reports/labmac/overnight/2026-10-29_1514/`, the per-bed traces in
`reports/labmac/debug/night_2026-10-29/`. Six causes, all fixed on
2026-10-30 (DECISIONS):

1. **The 500-step check ran without the speed limit.** Bed 1 (1,853
   boulders) reached rest after 15.4 h and blew apart within 100 steps:
   12 m/s, grains 5 m away. The cap is now applied on every step. Checked
   the same day on the laptop (`configs/experiments/3m_capwin.toml`): the
   3 m bed with the speed limit kept and the 3 mm/s threshold rested at step 4,300
   (1 mm/s took 7,200 on the same bed), held its 500-step window and passed
   QA with no WARN — overlap 1.6e-17, six contacts per grain. The 4 m bed
   (`4m_capwin.toml`), which had exploded in its 500-step check without the speed limit for both
   seeds at 1 mm/s, rested at 4,700, held its window and passed QA too.
2. **LMGC90's "big polyhedron" path.** A polyhedron whose radius exceeds
   4× the deck's *mean* radius is detected with the non-convex algorithm;
   19,502 small stones made bed 5's mean radius 0.09 m, 17 boulders became "big",
   and the deck died with a NaN at step ~150 (a contact with a zero local
   frame). The threshold is now set out of reach for every deck, and a
   deck that still reports a big polyhedron is refused. Reproduced on the
   laptop the same day: a 2 m deck of small stones 8–16 cm long under boulders in the realistic size mix
   run with the default ratio (`LMGCX_BIG_POLYR_RATIO=4`, 45 big
   polyhedra) spun up to 520 rad/s and died at step ~400; the same kind of
   deck with the study threshold runs at 13–40 rad/s.
3. **Cost ten times the estimate** (table above): 6 m → 4.5 m, 1 mm/s →
   3 mm/s, the estimator recalibrated.
4. **Three windows settled the same first layer** (beds 2, 4, 6, same
   seed): a finished layer is now copied.
5. **The 2-hourly report sync failed every time** (a git refusal on
   INDEX.md); fixed with a regression test. The morning sync worked.
6. **Ten windows waited eleven hours** for a bed that was promised in
   1.5 h. The slate now keeps every window busy.

## Small stones: what the 2 m probes of 2026-10-30 showed

Two 2 m decks with bed 5's structure (30 cm of small stones 10–20 cm long, Weibull
boulders on top; `configs/experiments/bigpolyr_probe.toml` and
`bigpolyr_split.toml`), 8,000 steps each on the laptop:

| boulder–stone law | boulder–stone overlap | stone–stone overlap | boulders at rest? |
|---|---|---|---|
| speed rule (bed 5 as first configured) | **67 mm** — the boulders sank 9 cm into the small stones in the first second and stayed there | 13 mm | nearly (median move 2 mm/s at the step cap) |
| split contact rule (position rule on boulder–stone, as beds 4–6 are configured now) | **0.0 mm** | 16 mm | yes (median move ~1 mm/s from step 4,000 on) |

So the split contact rule does what it was meant to: the boulders sit *on* the small stones.
What it cannot touch is the small stones among themselves, which the speed rule
leaves 0.5 of their smallest semi-axis into each other in a loaded 30 cm
bottom layer (0.3–0.35 in a thin top layer). That overlap is a 1 g settling
artefact: the small stones are compressed where the boulders press on them, and a
drop at 5e-5 g adds a load 10⁻⁴ of that. Hence the QA rule since
2026-10-31: small stones overlap is graded on pairs the craft can reach — at
least one grain in the top layer — and buried pairs are reported as
`overlap_rel_fines_buried`, a WARN at most. Under it the split-law probe
grades PASS on reachable pairs (0.00) and WARN on buried ones (0.53); the
velocity-law probe fails outright (2.2). The thin top layer beds 4 and 6 have their
small stones on top, so their stone–stone overlap (~0.33) is graded as before and
warns. A bed 5 graded FAIL by a window that started before this rule is
re-graded from its settled layers the next time anything asks for it
(`lab.py bed`, or the campaign), without re-settling.

## What the result decides

- Which bed the **position study** (docs/STUDY_DESIGN.md) runs on: the
  lowest bed whose outcomes agree with the higher ones within the CIs, at
  the cost the table shows.
- Gate **D5** (small stones): whether small stones matter at all, and if so whether a
  thin top layer, a bottom layer or pore filling is the right model.
- Gate **D9** (depth): whether 0.9 m is enough.
- The honest cost of an 8 m study bed at the chosen bed, from the
  measured settle hours (`python3 lab.py estimate <bed config>` calibrates on
  the settled beds).
