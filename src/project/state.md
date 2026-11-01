# Where things stand

_Written 31 October 2026, evening. This page is rewritten at the end of every
work session; the tables marked "regenerated" are rebuilt from the code
repository each time the site is published._

**The short version.** We simulate Hera's [Milani](../glossary.md#milani)
CubeSat landing on [Dimorphos](../glossary.md#dimorphos)-like rubble and
measure how it bounces. The lab Mac has been settling the six study
[beds](../glossary.md#bed-1-6) since Friday; nothing from it has reached
GitHub yet. On the laptop, the drop simulations now work end to end, every
drop has a movie and a page of analysis in the [drop reel](../drops/index.html),
and today's check of "is the last contact really the last?" found a flaw in
how a drop's end was decided. It is fixed, and every drop is being
re-simulated and verified tonight. Next: the lab Mac's beds finish over the
coming week, its drops get the same re-simulation, and the beds are compared
to choose the one for the position study.

## At a glance

{{#include generated/snapshot.md}}

## What this project is

Milani may end its mission by touching down on Dimorphos. Whether it stays,
hops or bounces away depends on the surface it hits, which nobody has seen at
the scale of the craft. We build that surface in
[LMGC90](../glossary.md#lmgc90) as piles of rigid
[polyhedral grains](../glossary.md#polyhedral-grain), drop the craft onto it,
and record what happens. The plan, in order:

1. **Settle realistic beds.** Six beds, each one change more realistic than
   the last: [uniform](../glossary.md#uniform-size-distribution) boulders,
   [Weibull](../glossary.md#weibull-size-distribution) boulders (the real size
   mix), a deeper pile, a [fines veneer](../glossary.md#veneer), a
   [fines substrate](../glossary.md#substrate) under the boulders, and thick
   fines. Each is settled once and reused.
2. **Fire the [standard 36 drops](../glossary.md#standard-36-drops) at every
   bed** and see which piece of realism changes the bounce. That decides how
   realistic the terrain has to be.
3. **Run the position study** on the chosen bed: the same arrival at many
   [landing spots](../glossary.md#landing-spot), to map where Milani stays and
   where it bounces away.
4. **Write the paper**: bounce-outcome maps for Milani on Dimorphos.

We are between steps 1 and 2.

## The state of every piece of work

| piece of work | where | state | what happens next |
|---|---|---|---|
| The six production beds, three seeds each for beds 1-4 | lab Mac | settling since Fri 30 Oct, about 11:15; no report yet | Bed 1 should be done already; Weibull beds need about a day each, bed 3 about two, beds 5-6 most of the week. They report when their drops start. |
| The standard 36 drops on the lab Mac's beds | lab Mac | start on their own as each bed settles | They run with Friday's code, whose end-of-drop test is the flawed one (below). After a `git pull` they must be re-simulated: one command per drop set. |
| Bed 1 settled on the laptop (same bed id as the lab Mac's) | laptop | done 31 Oct, QA PASS, 49 min | The laptop's reference for checking the lab Mac's result. |
| The 36 drops on bed 1 | laptop | done; being **re-simulated tonight** with the fixed end-of-drop test | Then every bounced-off drop is replayed to confirm its last contact. |
| Two smaller test sets (4 m bed, 3 m Weibull bed) | laptop | done; re-simulation queued after bed 1 | Same as above. |
| The first drops onto fines | laptop | 2 m fines bed settling; its 12 drops start on their own | Added to the reel when they finish. |
| A 3 m bed with a fines veneer | laptop | settling slowly (about 8 s per step) | Overnight. |
| Movies and the drop reel | laptop → this site | 108 drops with movies, figures and full numbers | Rebuilt after the re-simulation; new drop sets added as they finish. |

## What changed this week, and why

The details are in the [lab notebook](notebook.md); the short reasons:

- **29-30 Oct: the first six-bed night settled nothing.** The settle released
  its [speed cap](../glossary.md#speed-cap) at the first rest (a leftover
  guard), so beds blew apart; fines made LMGC90 switch some boulders to a
  contact detection that produced NaNs; and 6 m beds cost ten times the
  estimate. All fixed; beds are now 4.5 m and rest at 3 mm/s
  ([what the first night taught](notebook/bed-study.md)).
- **30 Oct: fines.** Boulders sank 9 cm into fines settled with the velocity
  law. The [split law](../glossary.md#split-law) keeps them on top. Fines
  buried under boulders stay compressed by about half their size, a settling
  artefact the craft never touches, so QA now only warns about it.
- **31 Oct: drops made trustworthy.** The first 36 drops exposed four
  problems with how a drop ends (bounces voided as "left the patch", a
  look-ahead that ignored the craft's spin, solver warnings on short drops,
  a missing bed lookup); all fixed. Movies were being deleted after 95 % of
  drops; they are now kept and the lost ones were rebuilt by replaying the
  drops, every replay bit-identical.
- **31 Oct, evening: the last-contact flaw.** Replaying a "bounced off" drop
  with its end test switched off showed a wing tip grazing a boulder 19 s
  after the run had stopped. The old test predicted the craft's tumble; a
  free craft wobbles, so the prediction drifted. The new test needs no
  prediction: it proves no point of the craft can reach the terrain anywhere
  over the simulated patch ([how](methods.md#bounced-off-when-a-drop-is-over)).
  "Energy kept" was also measured from the box floor; it is now measured from
  the first-contact height. Every bounced-off drop is being re-simulated.

## What the drops show so far

Provisional: these come from drops run before tonight's fix, and change where
the re-simulation finds later touches.

- The deployed craft does not bounce once. It lands on a wing, tips, and
  leaves after 3-4 [touches](../glossary.md#touch) over seconds to tens of
  seconds. The stowed box hits an edge or corner and is gone in under a second.
- Arriving at 40 degrees from vertical always keeps more speed than straight
  down: median [speed ratio](../glossary.md#speed-ratio-e) about 0.38 against
  0.24 on bed 1.
- Where the craft lands matters more than how fast it arrives: straight down
  on bed 1, e ranges from 0.07 to 0.42 across the nine spots.
- The grains barely move: the fastest goes a few mm/s, a few dozen shift by
  more than a millimetre.

## What is next

1. Tonight (laptop): finish the re-simulation and the last-contact replays,
   rebuild the reel with verified drops, add the first fines drops.
2. When you are next at the lab Mac: `git pull`, then re-simulate each
   finished drop set (`python3 lab.py campaign rerun battery_L1`, and so on),
   and publish (`python3 lab.py site --publish`). The settles keep running
   through all of it.
3. Through the week: beds 2-6 finish; their drop sets run; compare the beds
   (which change moves the bounce, by how much, at what cost).
4. Choose the bed for the position study and configure its grid of landing
   spots.

## Decisions that need you

- **This site is public.** The documentation repository is public, so the
  drop reel and these notes are visible to anyone with the link. Say if they
  should move somewhere private before the paper.
- **Re-simulate the lab Mac's weekend drops?** Recommended: their end test is
  the flawed one, and the fix changes some results. Cost: roughly an hour of
  CPU per drop set of 36.
- **Drops that fly out over the crop edge** are counted as bounced off, with
  a note that nothing outside the [crop](../glossary.md#crop) is simulated.
  A larger crop removes the doubt at about twice the cost per drop.
- **ParaView on the lab Mac**: movies need it (or the run folders copied to
  the laptop). Installing it there is a five-minute job.

## Where to find things

- [The drop reel](drop-reel.md): every drop, with its movie and analysis,
  organized by bed type or start conditions.
- [Every run so far](runs.md): every bed, drop set and test, on both machines,
  regenerated from the repository.
- [How each number is computed](methods.md): definitions, justifications,
  limits and the code, for every number on these pages.
- [Lab notebook](notebook.md): the dated notes, the study plan, the decision
  log, the failure catalogue.
- The LMGC90 chapters of this guide: [settling](../settling.md),
  [impacts](../impacts.md), [failure modes](../failure-modes.md), and the
  [glossary](../glossary.md).

## How this page stays current

At the end of each work session Claude rewrites the text above and runs
`python3 lab.py site --publish`, which regenerates the run log, the snapshot,
the methods page and the notebook from the code repository and pushes them
here; the site rebuilds itself within a few minutes. The lab Mac's morning
script can run the same command, so its results appear without anyone at the
laptop.
