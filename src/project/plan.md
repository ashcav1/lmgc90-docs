# The plan

_Written the night of 1-2 November 2026; the results of that night's
settling tests added on 2 November at 06:30. Where we want to end up, where
we are, and the steps in between, each with the check that decides the
next. [Where things stand](state.md) has the day-to-day detail._

## Where we want to end up

A paper on what decides whether [Milani](../glossary.md#milani) stays, hops or
bounces away when it lands on ground like [Dimorphos](../glossary.md#dimorphos)'s:
the landing speed and angle, the ground, and where on that ground it lands.

Nobody has seen Dimorphos's ground at the scale of the craft. So we bracket
it with two kinds of ground, the same everywhere in the study:

- **bare boulders**, and
- **boulders buried in smaller stones**, where most of the surface is small
  stones and the boulders show only their tops.

For that we need **standard beds**: a few beds, each settled once, at rest,
passing the [quality checks](../glossary.md#quality-checks-qa), big enough
for the craft with its panels open (4.5 m x 4.5 m), and used for every drop.

## The standard beds

| bed | what it is | built how |
|---|---|---|
| **A: bare boulders** | 0.9 m of [boulders](../glossary.md#boulders) 20-70 cm long in the [realistic size mix](../glossary.md#realistic-size-mix) (study bed 2's recipe), 4.5 m x 4.5 m. If that mix stays too slow to settle, the [even size mix](../glossary.md#even-size-mix) 30-70 cm (study bed 1's) | settled once per seed with the solver budget (below); config `standard_A`, its drops `battery_A` |
| **B: buried boulders** | bed A with [small stones](../glossary.md#small-stones) poured on until they fill the gaps between the boulders **and** cover them | bed A's settled boulders, copied, then the small stones. How thick a pour that takes is the open question: the stones fall into the gaps first (below) |

B is built on A's own boulders (same seed), so A against B is the same
ground with and without small stones. Three seeds of each, as for the study
beds.

**"Buried" needs a number.** We measure it on the bed's height map (the top
of the bed seen from above, on a 1 cm grid), and propose:

- small stones cover **at least 70 %** of the surface seen from above, and
- the boulders that still show stand **5 cm or less** (median) above the
  small stones around them.

The only settled bed with a [thin top layer](../glossary.md#thin-top-layer)
is far from that: small stones cover **24 %** of its surface, and the
boulders that show stand a median 9 cm above the stones around them (1 in
20 stand more than 23 cm above). That is why the craft hit a boulder first
in all 12 drops on it. An older test bed did reach 77 % cover with a 10 cm
layer, but only because its boulders sank 9 cm into the stones under the
contact rule we have since replaced, which is not real.

**How fine can "fine" be?** The smallest stones we can afford are 8-16 cm
long, a fifth to a half of the craft body's length (36.6 cm). Halving the
stone size multiplies their number by eight: a layer of 4-8 cm stones on a
4.5 m bed would be about 140,000 stones, far beyond these machines. So
"buried in smaller stones" in this study means buried in stones 8-16 cm
long. Material finer than that, if Dimorphos has it, is a limit we state in
the paper.

## Where we are

- **Drops: ready.** A drop now ends at the exact moment no part of the craft
  can touch the ground again, the open-panel craft turns about its true
  centre of mass, and the [patch](../glossary.md#patch) is the right size:
  1.2 m with the panels open, so 4.5 m beds are enough (tested on 1
  November). A drop costs about 4 minutes on one core.
- **Beds no longer blow apart, but most take far too long to come to
  rest.** The lab Mac's weekend (59 hours, 16 windows, reported Sunday 21:19):
  - study bed 1 (even size mix): all three seeds settled, in 1.5, 2.8 and 4
    hours; 46 of its 108 drops ran;
  - study bed 2 (realistic size mix): **one seed settled, after 52 hours**
    (26,400 steps at 7 s each); the other two were still settling after 59;
  - study bed 3 (bed 2, 1.5 m deep): all three still settling after 59 hours;
  - study beds 4 and 6 (with small stones): still settling, or waiting for
    bed 2's boulders to copy; bed 5 failed in its first 84 minutes.

  So the realistic size mix is slow even without small stones, and small
  stones make every bed slower again:
  - the quick 2 m bed rested after 6.3 hours;
  - the 3 m bed with study bed 1's boulders and a thin top layer never
    rested: after 32 hours its median grain still moved 11 mm/s, against
    the 3 mm/s that counts as [at rest](../glossary.md#at-rest). We stopped
    it on 1 November at 22:00;
  - its 2 m version was still moving 18 mm/s after 8 hours.

  In all of them the small stones keep jostling: 1 grain in 100 sits at
  the [speed limit](../glossary.md#speed-limit) the whole time, and the
  contact solver needs thousands of passes per time step.
- **Small stones poured on a boulder layer fall into its gaps.** Measured
  on 2 November on the end state of the overnight tests: of a layer one
  stone deep poured on study bed 1's boulders (30-70 cm, 0.9 m deep), **92 %
  of the stones ended up in the lower half of the boulder layer and a third
  on the floor**; small stones cover 1 % of the surface seen from above. On
  the quick bed's smaller boulders (20-50 cm, 0.5 m deep) a third fell to
  the lower half, and the cover is 24 %. So a pour does not stay on top as
  a layer: it fills the gaps from the bottom up first, and only what is
  left covers the boulders. That is very likely also why these beds never
  rest: stones keep trickling down through the gaps.
- **What a buried bed costs.** Bed B needs a layer of small stones about 25
  cm thick, poured over the boulders: about **18,000 stones** of 8-16 cm
  (or about 9,000 of 10-20 cm). At today's settling speed that is more than
  a week per bed on one core. Splitting one simulation over several cores
  is ruled out: LMGC90's multi-core solver has a known flaw (cores overwrite
  each other's numbers) that blew runs apart on the lab Mac, so every run is
  held to one core.
So everything needed for the paper works except two linked things:
**getting beds to rest in hours, not days** (the realistic size mix, and
above all a bed full of small stones), and **finding a pour of small stones
that buries the boulders instead of draining into them**.

## Making beds settle fast

Five tests ran on the laptop on the night of 1-2 November, each a 2 m copy of the 3 m bed that
would not rest (study bed 1's boulders and a thin top layer). Each changes
one thing, for both stages (the boulders, then the small stones):

| test | the change | why it might help |
|---|---|---|
| `settle_budget` | at most 1,000 solver passes per time step (now up to 10,000) | each step costs a fifth; the drops start from zero speed in near-zero gravity, so the bed's shape matters, not perfectly converged forces at 1 g |
| `settle_g1` | settle at 1 m/s² instead of 9.81 | the stones' weight presses them 10 times less into each other. Without stickiness, a bed settled at lower gravity packs the same way, just more slowly, and the drops run at 0.00005 m/s² anyway |
| `settle_budget_g1` | both | |
| `settle_budget_quench` | budget, and every grain's speed set to zero every 200 steps until the first rest | takes out the energy the jostling keeps alive |
| `settle_budget_coarse` | budget, with stones 10-20 cm long | half as many stones; does a coarser layer rest sooner? |

After the lab Mac's results came in, four more ran, on study bed 2's boulders
(the realistic size mix) at 2 m x 2 m:

| test | what it shows |
|---|---|
| `settle_rmix_base` | how long these boulders take settled the old way (52 hours at 4.5 m on the lab Mac) |
| `settle_rmix_budget` | the same with the solver budget: does it help the realistic size mix too? |
| `settle_rmix_budget_t15`, `_t25` | the budget, with 15 or 25 cm of small stones on these boulders: **the first measure of how much cover a thick layer gives**, and of how long it takes |

### What the tests showed (2 November, morning)

Each bed settles in stages: the boulders first, then the small stones
poured on them. "At rest" means the median grain moved less than 3 mm/s
over the last second, five checks in a row, then a 500-step check.

| test | boulders | small stones |
|---|---|---|
| the old way (`bed4u_2m_mu06`, 1 Nov) | at rest: 8,400 steps, 13.7 min | **not at rest**: 18 mm/s after 7,200 steps and 8 hours (4.0 s a step) |
| `settle_budget` | at rest: 4,700 steps, 4.9 min | **not at rest**: 6 mm/s after 20,000 steps and 5.9 hours (1.1 s a step) |
| `settle_budget_quench` (speeds zeroed every 200 steps) | at rest: 5,600 steps, 5.5 min | **not at rest**: 5 mm/s after 20,000 steps and 6.1 hours |
| `settle_budget_coarse` (stones 10-20 cm) | shared with `settle_budget` | **crashed** twice at step 2,100: a grain's speed became infinite |
| `settle_g1`, `settle_budget_g1` (1 m/s²) | **not at rest** after 20,000 steps (11 and 15 mm/s) | not reached |
| `settle_rmix_base` (realistic size mix, the old way) | at rest: 14,500 steps, 106 min; the 500-step check overran, so the bed is the first-rest snapshot (QA WARN) | none |
| `settle_rmix_budget` (realistic size mix, budget) | at rest: 13,400 steps, 63 min, 500-step check clean, **QA PASS** | none |
| `settle_rmix_budget_t15` (15 cm of stones) | copied from `settle_rmix_budget` | **stopped at 06:15**: 11 mm/s after 9,800 steps and 6.7 hours (2.5 s a step) |
| `settle_rmix_budget_t25` (25 cm of stones) | copied | **stopped at 06:15**: 14 mm/s after 5,700 steps and 6.7 hours (4.2 s a step) |

What that means:

- **The solver budget is the one change that helps, and we adopt it for
  every new bed.** It does not change how a bed settles step by step: at
  step 7,200 the small stones moved 17.5 mm/s with the budget and 17.9 mm/s
  without. But each step costs a quarter as much, so the budget got there
  in 1.9 hours instead of 8. On the boulders it is 2.8 times faster for the
  even size mix and 1.7 times for the realistic size mix, and the realistic
  mix's 500-step check passed cleanly where the old way's overran.
- **By the rule we set (at least 3 times faster to rest, quality checks
  passed), no method qualifies**, because no layer of small stones came to
  rest in any test. With the budget the stones' median move fell from 18 to
  about 5 mm/s over 20,000 steps and then levelled off between 4 and 6;
  3 mm/s was never reached.
- **Zeroing the speeds barely helps** (5 against 6 mm/s at the end) and
  stays off. **The 10-20 cm stones crashed** and cannot be used until that
  is understood. **1 m/s² is out.**
- **The thick-layer tests were stopped half-way** and saved no end state,
  so they do not yet say how much cover 15 or 25 cm gives. They now run to
  their 20,000-step limit on the lab Mac, which saves the end state even
  when the stones are not at rest, and `lab.py burial` measures it.

**What decides bed B now.** Two questions, both answered by the thickness
tests on the lab Mac:

1. **How much of a pour drains into the realistic size mix?** Its many
   20-30 cm boulders fill the gaps between the big ones, so less should
   drain than into study bed 1's boulders (92 % in the lower half). If 25
   or 35 cm of stones reaches 70 % cover, bed B is that pour on bed A.
2. **Does a thick layer ever rest?** If not even 35 cm rests within 20,000
   steps, bed B needs a looser rest test for its small stones (for example
   6 mm/s, which the budget run reached for five checks in a row at step
   13,800), justified by what the drops see: every grain starts the drop at
   zero speed in a gravity 200,000 times weaker.

**What bed B costs.** At 4.5 m, 25 cm of stones is about 18,000 of them.
From the 2 m test's 4.2 s a step and the cost rule (cost per step grows as
grains^1.3), that is about 30 s a step: 20,000 steps would take about
**a week per seed** on one core, three seeds in parallel. That is the price
if nothing better turns up, and it is why bed B is built once and kept.

## The steps

| when | what | where | the check that decides the next step |
|---|---|---|---|
| ~~1-2 Nov night~~ | ~~the nine settling tests~~ **done**: the solver budget is adopted; no small-stone layer rested | laptop | |
| ~~Mon 2 Nov~~ | ~~measure how buried the boulders are~~ **done**: on every bed report, and `lab.py burial` for any saved stage, at rest or not | code | |
| Mon 2 Nov | **how thick must the stone layer be?** `settle_rmix_budget_t15`, `_t25`, `_t35`: bed A's boulders at 2 m x 2 m with 15, 25 and 35 cm of small stones, solver budget, each to its 20,000-step limit (about 15, 25 and 35 hours) | lab Mac, one core each | `lab.py burial` on each: the thinnest layer with 70 % cover and boulders 5 cm or less above; and whether any of them rests |
| Mon 2 Nov | **bed A, three seeds** (`standard_A`, solver budget). Study bed 2 took 52 hours the old way; the budget's 1.7 times suggests about 30 | lab Mac | rests within about a day and a half; if not, bed A uses the even size mix (hours) |
| as bed A lands | the 36-drop set on it (`battery_A`) | lab Mac | under an hour for three seeds on 16 cores |
| Wed 4 Nov | **bed B's recipe**, from the thickness tests: the pour, and the rest test for its stones | you and me | |
| Wed 4 Nov - about Wed 11 Nov | **bed B, three seeds**, on bed A's boulders | lab Mac, three cores for about a week | settled, quality checks passed, "buried" measured on the real bed |
| as each bed lands | the [36-drop set](../glossary.md#36-drop-set) with the panels open, then the exact last-contact check, movies, this site | lab Mac | under an hour for a bed's three seeds on 16 cores |
| when bed B lands | A against B: how much do the small stones change the bounce, and how much of that is seed-to-seed spread? Meanwhile, study bed 1 against bed A (does the size mix matter?) | | |
| next | the position study on the beds that matter, then the paper's figures | lab Mac | |

## What happens to the six study beds

They were the first try at bracketing the ground, before we knew what small
stones cost.

- **Keep what has settled**: bed 1's three seeds and its 46 drops, bed 2's
  seed 2. The drops ran with Friday's code: re-simulate them with the new
  end of drop (`campaign rerun --stale`) and run bed 1's other 62 drops.
  Bed 1 against bed 2 answers whether the size mix matters.
- **Stop everything still settling** (beds 2 seeds 1 and 3, bed 3, beds 4
  and 6), unless a window's last progress line ends in "confirmation
  window": that is its 500-step check, the last stretch before rest. With
  the old settings each would need days more; the standard beds are
  settled with the new method anyway. Bed B replaces bed 6, and bed 4's
  thin top layer leaves most boulders bare (24 % cover on the 2 m test
  bed), so it is not buried ground. Bed 3's question (does depth matter?)
  waits until there is a fast method.
- **Bed A is study bed 2's recipe, settled again** with the new settling
  method. It has to be re-settled so that bed B can be built on its boulders (a bed
  can only be copied when both were settled the same way).

## If it does not work

- **A week per seed of bed B is too long:** settle one seed of bed B first
  and run its drops while the other two settle; or settle bed B at 3.9 m x
  3.9 m (one landing spot instead of nine, a quarter fewer stones). The
  10-20 cm stones (half as many) need their crash understood first.
- **Even 35 cm of stones mostly drains into the boulders:** pour the
  stones in two layers (the code settles a bed one layer at a time), so the
  first fills the gaps and rests before the second covers the boulders; or
  give bed B a thinner boulder layer under the stones (less gap to fill),
  at the price of A and B no longer sharing their boulders.
- **The lab Mac is down or not reporting:** it is the first thing to check
  on Monday; nothing in this plan runs without it.

## Decisions that need you

1. **Is "buried" right?** At least 70 % of the surface small stones, and the
   boulders that show standing 5 cm or less above them.
2. **Stone size for bed B:** 8-16 cm long. The 10-20 cm stones crashed in
   the overnight test, so for now there is no choice to make.
3. **Stop the lab Mac's unfinished settles** (recommended above), and if
   bed A in the realistic size mix is not at rest after about a day and a
   half, **bed A in the even size mix** instead.
4. **A looser rest test for bed B's small stones** (6 mm/s instead of 3),
   if the thickness tests show a thick layer never rests: the drops start
   every grain at zero speed in near-zero gravity, so the creep does not
   reach them. Decided on Wednesday with the thickness results.
5. **The position study's landing area** (unchanged from
   [Where things stand](state.md)): 0.6 m x 0.6 m fits on a 4.5 m bed; a
   wider area needs a wider bed.
