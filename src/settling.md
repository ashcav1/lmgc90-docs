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
2. **Gap law for boulders, velocity law for pairs with fines**
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
5. **One time step for the whole settle.** Changing dt at release (4 → 2 ms)
   threw 100 of 116 resting boulders to 1–6 m/s: stored reactions scale
   with dt.
6. **Full NLGS budget on thick layers** (`max_checks` 1000 × 10 sweeps).
   A 1,000-iteration budget reached rest just as fast and was *not* the
   cause of the explosions, but the final state should be a converged one.
7. **Rest is structural, not a velocity threshold.** A pile several grains
   deep chatters at cm/s indefinitely (the common-plane contact set flickers
   by ±5 contacts per 50 steps in a resting bed of 1,400 contacts) while its
   geometry is static. Declare rest when the median grain moved < 1 mm over
   the last second of simulated time, 95 % < 2 cm, none > 10 cm — or when
   99 % are below 2 mm/s. Then a final phase under the full solver must pass
   the same test again.
8. **Hard invariants, not warnings:** no grain outside the box; at least one
   grain–grain contact per grain; a layer that lost > 40 % of its expected
   height has fallen through itself. All three were silent once.

## Rest under the gap law is metastable — save it when you have it

Held at rest for longer (a 20,000-step confirmation window on the 0.9 m
test layer), every variant at the usual NLGS tolerance (Quad 1e-3) burst
and blew apart with nothing changed in the run: the baseline at ~9,600
steps, θ = 1 at ~7,500, scrambled Gauss–Seidel order at ~6,800, a 1 ms
time step at ~21,000. Lifting the cap, switching the solver budget or
changing dt only trigger it sooner. The one variant that held the whole
window was **tolerance 1e-4** (rest at 7,700, then 20,000 steps at 1 mm/s;
~8× the cost per step); 3e-4 is not enough, and a second baseline run held
where the first had exploded — at 1e-3 the hold is a coin flip. A larger
time step (3–4 ms) or a gentler cap (0.15 m/s) explode as well. That is the mechanism: residual penetrations
within the tolerance are ejected at `gap/dt` every step, and in a deep
pile the kicks accumulate faster than friction removes them unless they
are made ten times smaller. Consequences:

- declare rest by the structural test and **save the state immediately**;
  a short confirmation window (500 steps) can follow, but the instability
  can strike inside it (a 4 m bed did, 400 steps in), so keep the
  first-rest snapshot and fall back to it if the window explodes — unless
  the tolerance is 1e-4, which is the option when a bed must be held or
  built on;
- never change anything between phases;
- the impact decks are immune: they use the velocity law, which never
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
| 0.9 m square, 15 cm layer | 232 | 1,400 | 27–34 | 1,700–3,300 |
| 1.5 m square, 0.9 m layer | 116 | 600 | 6 (budget 1,000 it.) / ~15 (full) | 5,100–6,800 |
| 3 m square, 0.9 m layer | 463 | 3,300 | 55–80 (budget) / 150–200 (full) | 5,500 |

Vertices: 20-vertex grains settle in the same number of steps as 12 at
~1.4× the cost per step; 8-vertex grains chatter longer. A 1 ms time step
does not reach rest sooner in simulated time and doubles the steps.
Friction 0.3 settles in more steps (grains roll further); 0.9 diverged once.

## What the metrics file holds

Every `check_every` steps: max / mean / 99th-percentile grain speed, the
phase flag, number of grains faster than 2 cm/s, max spin, deepest overlap,
overlapping pairs, escaped grains, and the median / 95th / 99th-percentile /
max displacement over the last second. The `settle` figure of a bed draws
them; the overnight status reads them in words.
