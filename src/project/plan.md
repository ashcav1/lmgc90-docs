# The plan

_Written the night of 1-2 November 2026. Where we want to end up, where we
are, and the steps in between, each with the check that decides the next.
[Where things stand](state.md) has the day-to-day detail._

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
| **A: bare boulders** | 0.9 m of [boulders](../glossary.md#boulders) 20-70 cm long in the [realistic size mix](../glossary.md#realistic-size-mix) (study bed 2's recipe), 4.5 m x 4.5 m. If that mix stays too slow to settle, the [even size mix](../glossary.md#even-size-mix) 30-70 cm (study bed 1's) | settled once per seed |
| **B: buried boulders** | bed A with [small stones](../glossary.md#small-stones) poured on until they fill the gaps between the boulders | bed A's settled boulders, copied, then the small stones |

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
- **What a buried bed costs.** Bed B needs a layer of small stones about 25
  cm thick, poured over the boulders: about **18,000 stones** of 8-16 cm
  (or about 9,000 of 10-20 cm). At today's settling speed that is more than
  a week per bed on one core. Splitting one simulation over several cores
  is ruled out: LMGC90's multi-core solver has a known flaw (cores overwrite
  each other's numbers) that blew runs apart on the lab Mac, so every run is
  held to one core.
So everything needed for the paper works except one thing: **getting beds
to rest in hours, not days**: the realistic size mix, and above all a bed
full of small stones.

## Making beds settle fast

Five tests run on the laptop tonight, each a 2 m copy of the 3 m bed that
would not rest (study bed 1's boulders and a thin top layer). Each changes
one thing, for both stages (the boulders, then the small stones):

| test | the change | why it might help |
|---|---|---|
| `settle_budget` | at most 1,000 solver passes per time step (now up to 10,000) | each step costs a fifth; the drops start from zero speed in near-zero gravity, so the bed's shape matters, not perfectly converged forces at 1 g |
| `settle_g1` | settle at 1 m/s² instead of 9.81 | the stones' weight presses them 10 times less into each other. Without stickiness, a bed settled at lower gravity packs the same way, just more slowly, and the drops run at 0.00005 m/s² anyway |
| `settle_budget_g1` | both | |
| `settle_budget_quench` | budget, and every grain's speed set to zero every 200 steps until the first rest | takes out the energy the jostling keeps alive |
| `settle_budget_coarse` | budget, with stones 10-20 cm long | half as many stones; does a coarser layer rest sooner? |

After the lab Mac's results came in, four more, on study bed 2's boulders
(the realistic size mix) at 2 m x 2 m:

| test | what it shows |
|---|---|
| `settle_rmix_base` | how long these boulders take settled the old way (52 hours at 4.5 m on the lab Mac) |
| `settle_rmix_budget` | the same with the solver budget: does it help the realistic size mix too? |
| `settle_rmix_budget_t15`, `_t25` | the budget, with 15 or 25 cm of small stones on these boulders: **the first measure of how much cover a thick layer gives**, and of how long it takes |

**First result (the boulder stage, 22:15).** With the solver budget the
boulders came to rest in 4,700 steps and 4.9 minutes, against 8,400 steps
and 13.7 minutes for the same boulders settled the old way, and passed their
500-step check cleanly (the old way overran it). Nearly three times faster.
The budget with the speed zeroing took 5,600 steps. **Settling at 1 m/s²
is out:** everything moves three times slower, and the boulders had not
rested after 20,000 steps, so both 1 m/s² tests stopped there. The
small-stone stages run through the night.

**How we pick.** The settling method that reaches rest in the least time and passes
the quality checks wins, if it is at least three times faster than today.
Less than that does not make bed B affordable. The tests stop at 06:15 on
2 November at the latest, finished or not.

## The steps

| when | what | where | the check that decides the next step |
|---|---|---|---|
| tonight | the five settling tests | laptop | a settling method at least 3x faster that passes the quality checks |
| Mon 2 Nov | measure how buried the boulders are on every bed (cover seen from above, how far boulders stand out) and print it on each bed's report | code | done when every settled bed shows it |
| Mon 2 Nov | **how thick must the stone layer be?** Bed A's boulders at 2 m x 2 m with 15, 25 and 35 cm of small stones, settled with the winning method | lab Mac, one core each | the thinnest layer that meets "buried" (70 % cover, boulders 5 cm or less above) |
| Mon 2 Nov | **bed A, three seeds**, settled with the winning method (so that bed B can be built on it). Realistic size mix first | lab Mac | rests within about a day; if not, bed A uses the even size mix (hours) |
| Tue 3 - Thu 5 Nov | **bed B, three seeds**, on bed A's boulders, with the layer thickness found above | lab Mac | settled, quality checks passed, "buried" measured on the real bed |
| as each bed lands | the [36-drop set](../glossary.md#36-drop-set) with the panels open, then the exact last-contact check, movies, this site | lab Mac | under an hour for a bed's three seeds on 16 cores |
| end of week | A against B: how much do the small stones change the bounce, and how much of that is seed-to-seed spread? | | |
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

- **No settling method is three times faster:** use the 10-20 cm stones (half as many)
  and one seed of bed B first. If that is still too slow, settle bed B at
  3.9 m x 3.9 m (one landing spot instead of nine, a quarter fewer stones).
- **The layer has to be thicker than 35 cm:** pour the stones in two layers
  (the code settles a bed one layer at a time), so each pour rests before
  the next.
- **The lab Mac is down or not reporting:** it is the first thing to check
  on Monday; nothing in this plan runs without it.

## Decisions that need you

1. **Is "buried" right?** At least 70 % of the surface small stones, and the
   boulders that show standing 5 cm or less above them.
2. **Stone size for bed B:** 8-16 cm long (as fine as we can afford, twice
   the cost) or 10-20 cm (half the cost). Tonight's coarse-stone test
   informs it; I recommend 8-16 cm if the settling-speed test succeeds.
3. **Stop the lab Mac's unfinished settles** (recommended above), and if
   the realistic size mix stays slow with the new method, **bed A in the
   even size mix** instead.
4. **The position study's landing area** (unchanged from
   [Where things stand](state.md)): 0.6 m x 0.6 m fits on a 4.5 m bed; a
   wider area needs a wider bed.
