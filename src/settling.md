# Settling a granular bed

The bed is the terrain every impact lands on. Settling it once, correctly,
turned out to be the hardest part of the pipeline. What follows is the
recipe that works and the evidence behind each rule.

## Recipe

1. **Deposit real shapes, densely.** Drop each grain's own polyhedron onto
   the height map of what is already there, at the lowest of ~64 random
   candidate spots. The loose column is then ~1.7× the final layer. Proxy
   spheroids gave 5–6× columns (a 12-vertex 1:0.7:0.5 grain fills a quarter
   of its spheroid) and a night of settling that never finished.
2. **Position rule for boulders, speed rule for pairs with small stones**
   ([Contact laws](contact-laws.md)).
3. **Cap speeds while depositing** (`RBDY3_PartialDamping(1, vmax)` every
   step): 0.3 m/s at dt = 2 ms keeps every grain within 0.6 mm per step,
   well inside the alert distance. On a 0.9 m-thick test layer, 0.3 m/s
   rested in 6,800 steps and 1.5 m/s in 10,000: harder landings agitate the
   pile longer.
4. **Never lift the cap.** A pile at rest under the cap (max speed 8 mm/s
   against 0.3 m/s — the clip is inert) blew apart to 1–3 m/s within 500
   steps of removing it, four times, with any solver budget. The uncapped
   contact dynamics of a deep polyhedral pile are unstable; the cap is a
   harmless stabiliser at rest, so it stays through the final phase.
   *Check that it really does*: in our pipeline the call sat behind a phase
   flag from a rebuild and every 500-step check ran uncapped for two
   days, until a 1,853-boulder study bed blew apart 100 steps after
   its first rest (2026-10-30). The `capped` column of the metrics made it
   obvious in hindsight.
5. **One time step for the whole settle.** Changing dt at release (4 → 2 ms)
   threw 100 of 116 resting boulders to 1–6 m/s: stored reactions scale
   with dt.
6. **Full NLGS budget on thick layers** (`max_checks` 1000 × 10 sweeps).
   A 1,000-iteration budget reached rest just as fast and was *not* the
   cause of the explosions, but the final state should be a converged one.
7. **Rest is structural, not a velocity threshold.** A pile several grains
   deep chatters at cm/s indefinitely (the common-plane contact set flickers
   by ±5 contacts per 50 steps in a resting bed of 1,400 contacts) while its
   geometry is static. Declare rest when the median grain moved < 3 mm over
   the last second of simulated time, 95 % < 6 cm, none > 30 cm — or when
   99 % are below 2 mm/s. Then a 500-step check with nothing changed
   must pass the same test again. The threshold was 1 mm at first: a 6 m,
   1,853-boulder bed reached 3 mm/s at step 6,600 and crept to 1 mm/s by
   19,800 — ten hours of 1 g creep that a micro-g drop never sees (the
   creep timescale stretches by ~450 at 5e-5 g). On 27 saved traces the
   3 mm/s rest comes at the same step for small beds and 2–3× earlier for
   the 4–6 m ones, always with zero overlap under the position rule.
8. **Hard invariants, not warnings:** no grain outside the box; at least one
   grain–grain contact per grain; a layer that lost > 40 % of its expected
   height has fallen through itself. All three were silent once.

## Rest under the position rule is metastable — save it when you have it

Held at rest for longer (a 20,000-step 500-step check on the 0.9 m
test layer), every variant at the usual NLGS tolerance (Quad 1e-3) burst
and blew apart with nothing changed in the run: the baseline at ~9,600
steps, θ = 1 at ~7,500, scrambled Gauss–Seidel order at ~6,800, a 1 ms
time step at ~21,000. Lifting the cap, switching the solver budget or
changing dt only trigger it sooner. (Correction, 2026-10-30: every one of
these windows had in fact lifted the speed limit at the first rest — the phase-flag
bug above — so the holds and bursts below describe *uncapped* piles. With
the speed limit kept, the 3 m bed at tolerance 1e-3 held its 500-step window and
passed QA at the first try; whether a long capped hold or a budget switch
alone detonates a pile is untested.) The one variant that held the whole
window was **tolerance 1e-4** (rest at 7,700, then 20,000 steps at 1 mm/s;
~8× the cost per step); 3e-4 is not enough, and a second baseline run held
where the first had exploded — at 1e-3 the hold is a coin flip. A larger
time step (3–4 ms) or a gentler cap (0.15 m/s) explode as well. That is the mechanism: residual penetrations
within the tolerance are ejected at `gap/dt` every step, and in a deep
pile the kicks accumulate faster than friction removes them unless they
are made ten times smaller. Consequences:

- declare rest by the structural test and **save the state immediately**;
  a short 500-step check (500 steps) can follow, but the instability
  can strike inside it (a 4 m bed did, 400 steps in, for both seeds), so
  keep the first-rest snapshot and fall back to it if the window explodes —
  unless the tolerance is 1e-4 (a 3 m, 463-boulder bed then held its window
  at ~3× the step cost: on a deep pile the full solve at 1e-3 already runs
  hundreds of iterations, so the 8× seen on a thin bed shrinks), which is
  the option when a bed must hold its window, be held, or be built on;
- never change anything between phases;
- the impact decks are immune: they use the speed rule, which never
  ejects an overlap, start with zeroed velocities at micro-g, and their
  free grains move < 15 µm before the craft arrives (measured).

## Keep what you settled

A settled bed is the expensive asset; impacts are cheap and only read it.
Keep beds as immutable, checksummed folders (the state file, the deck, the
geometry, the height map, the card), make them read-only, register them in
a library by name and id, and let every consumer verify the checksums
before use. Name beds by id in campaigns — an id that hashes the settle
settings is exactly what makes a bed invisible after a default changes,
so pin the folder, not the config. Export/import as a tar to move beds
between machines.

## Costs (laptop, M4 Pro, one process)

| bed | grains | contacts | ms per step | steps to rest |
|---|---|---|---|---|
| 0.9 m × 0.9 m, 15 cm layer | 232 | 1,400 | 27–34 | 1,700–3,300 |
| 1.5 m × 1.5 m, 0.9 m layer | 116 | 600 | 6 (budget 1,000 it.) / ~15 (full) | 5,100–6,800 |
| 3 m × 3 m, 0.9 m layer | 463 | 3,300 | 55–80 (budget) / 150–200 (full) | 5,500 |
| 3 m × 3 m, 1.2 m layer, tolerance 1e-4 | 463 | — | 875 (full) | 5,100 (500-step check held) |
| 4 m × 4 m, 0.9 m layer | 821–827 | — | 550–590 (full) | 9,200–9,400 (500-step check without the speed limit exploded, snapshot used) |
| 3 m × 3 m, 0.9 m layer, speed limit kept, rest at 3 mm/s | 463 | — | 215 (full) | 4,300 (500-step check held, QA PASS) |
| 4.5 m × 4.5 m, 0.9 m layer, even size mix (study bed 1), speed limit kept, 3 mm/s | 1,043 | — | 585 (laptop, shared) | 4,500 (500-step check held, QA PASS) |
| 3 m × 3 m, 0.9 m layer, **realistic size mix**, speed limit kept, 3 mm/s | 934 | — | 480–1,100 (laptop, shared) | ~13,700 (median move 4–8 mm/s from 5,000 to 13,000: the whole pile churns; 500-step check held, QA PASS) |
| 6 m × 6 m, 0.9 m layer (lab Mac, 6 beds at once) | 1,853 | 13,000 | 2,800 (full, ~1,000 iterations) | 19,800 at 1 mm/s (6,600 at 3 mm/s); 500-step check without the speed limit exploded |
| 6 m × 6 m, 0.9 m layer, realistic size mix (lab Mac) | 3,729 | 25,500 | 8,500 (~1,300 iterations) | not reached in 6,800 steps (16.7 h) |
| 6 m × 6 m, 1.5 m layer, realistic size mix (lab Mac) | 6,214 | 32,000 | 13,000 (~1,500 iterations) | not reached in 4,600 steps |
| 6 m × 6 m, 30 cm of small stones 10–20 cm long, speed rule (lab Mac) | 19,502 | 110,800 | 2,900 (10–20 iterations) | 3,100 |
| 2 m × 2 m, 30 cm of small stones 10–20 cm long + 0.9 m boulders in the realistic size mix, speed rule on boulder–stone | 2,167 + 418 | 18,900 | 1,000–1,900 | not in 3,000 (boulders 67 mm into the small stones) |
| the same with the position rule on boulder–stone pairs only | 2,167 + 418 | 18,900 | 1,000–3,100 | ~4,000 at 1 mm/s median move (boulder–stone overlap 0.0 mm, stone–stone 16 mm) |
| 1 m × 1 m, 10 cm of small stones 8–16 cm long only | 353 | — | 19 (speed rule) / 51 (position rule) | 1,000 / 1,300 |
| 1 m × 1 m, 25 cm of small stones 8–16 cm long only | 882 | — | 70 (speed rule) / 656 (position rule) | 1,400 / 3,500 |
| 2 m × 2 m, 0.5 m boulders + 10 cm small stones | 327 + 1,415 | 10,460 | 588 (small-stone stage, speed rule) / 4,000–4,400 (position rule, still draining) | 5,100 / — |

A realistic-size-mix pile takes ~3x the steps of an even-size-mix one to reach the same median move: it churns in bursts under the position rule's kicks, and not only its small grains — the median of the load-bearing half tracks the overall one (deterministic twin with the extra columns, 2026-10-31). Budget for it; a gentler speed limit only slows the rain-down.

The lab Mac rows show the scaling that matters for study: the NLGS
iteration count grows with the pile, so the cost per step grows as about
grains^1.3, and the steps to rest as about sqrt(grains); in a mixed deck the
gap-law boulder pairs set the iteration count and the cheap small-stone contacts
are swept 1,000+ times too. A 6 m bed of 3,700 boulders is 2–3 days on one
core; the same terrain at 4.5 m is 8 h.

Boulders deposited on a velocity-law bed with small stones sink into it: 9 cm in the
first second on the 2 m probe, 67 mm left at the end — the law only stops
approach once a pair overlaps, and a heavy boulder pressing into a soft
bed with small stones outruns the detection. The position rule on the boulder–stone pairs
alone (speed rule kept among the small stones, where the position rule never rests
over boulders) leaves that overlap at zero and the pile rests. What remains
is the small stones among themselves, 0.5 of their smallest semi-axis into each
other in a loaded bottom layer: a 1 g artefact, graded in our QA only where
the craft can reach the pair and reported for buried pairs.

One more LMGC90 trap in mixed decks: a POLYR whose radius exceeds
`POLYR_SetThresholdBigPolyr` (default 4) times the deck's *mean* radius is
"big", and every pair touching it is detected with the non-convex algorithm
instead of the common plane. Small stones pull the mean radius down, so the largest
boulders of a deck with small stones cross the ratio (19,502 small stones 10–20 cm long
under 3,762 boulders: mean 0.09 m, 17 boulders "big"), and that path gave
a contact with a zero local frame — `Wnn = 0`, NaN at NLGS iteration 10,
within 150 steps. Set the ratio out of reach before the contactors load
and check the `NB BIG POLYR` banner is 0.

Vertices: 20-vertex grains settle in the same number of steps as 12 at
~1.4× the cost per step; 8-vertex grains chatter longer. A 1 ms time step
does not reach rest sooner in simulated time and doubles the steps.
Friction 0.3 settles in more steps (grains roll further); 0.9 diverged once.
The position rule on small-stone pairs removes the overlap the speed rule leaves
(7.8 mm → 0.0 on the small-stones-only beds, QA PASS) at ~3× the step cost on a
thin layer and ~9× on a 25 cm pile (0.7 ms per body-step — the position rule's
price on any pile several grains deep, boulders included; the velocity
law's is ~0.08). On small stones poured over boulders it is unusable: 5,800 steps
at 4–6 s/step (10×) without rest, the small stones flickering between the
boulders at a median 10 mm/s for ever. The speed rule's failing overlap
there is the boulder–stone pairs (−20 mm; stone–stone only −8 mm), so the
split to try is the position rule for boulder–stone pairs and the speed rule
among small stones (`boulder_fines_law` in the Milani pipeline).

## What the metrics file holds

Every `check_every` steps: max / mean / 99th-percentile grain speed, the
phase flag, number of grains faster than 2 cm/s, max spin, deepest overlap,
overlapping pairs, escaped grains, and the median / 95th / 99th-percentile /
max displacement over the last second. The `settle` figure of a bed draws
them; the overnight status reads them in words.
