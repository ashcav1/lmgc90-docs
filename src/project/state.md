# Where things stand

_Written Sunday 1 November 2026, evening. This page is rewritten at the end
of every work session; the tables marked "regenerated" are rebuilt from the
code repository each time the site is published._

**The short version.** We simulate Hera's [Milani](../glossary.md#milani)
CubeSat landing on ground like [Dimorphos](../glossary.md#dimorphos)'s and
measure how it bounces. This weekend:

- **The end of a drop is now exact,** and a centre-of-mass error in every
  drop with the panels open was found and fixed. All 108 laptop drops were
  re-simulated.
- **These pages were rewritten in plain words,** and grain sizes are now
  lengths. They had been half-lengths, so every boulder looked half its size.
- **How big a bed must be**, worked out and tested: the study beds' 4.5 m is
  right; bigger buys nothing, and 2 m is too small for drops (below).

The lab Mac has been settling the six study beds since Friday, but has sent
nothing to GitHub since Friday morning.

## At a glance (regenerated)

{{#include generated/snapshot.md}}

## What this project is

Milani may end its mission by touching down on Dimorphos. Whether it stays,
hops or bounces away depends on the ground it hits, which nobody has seen
at the scale of the craft. We build that ground in
[LMGC90](../glossary.md#lmgc90) as piles of rigid angular
[grains](../glossary.md#grain), drop the craft onto it, and record what
happens. The plan, in order:

1. **Settle six study beds**, each one change more realistic than the last
   (table below). Each is settled once and reused.
2. **Fire the same [36 drops](../glossary.md#36-drop-set) at every bed**
   and see which change moves the bounce. That decides how realistic the
   ground has to be.
3. **Run the position study** on the chosen bed: the same arrival at many
   [landing spots](../glossary.md#landing-spot), to map where Milani stays
   and where it bounces away.
4. **Write the paper**: bounce-outcome maps for Milani on Dimorphos.

We are between steps 1 and 2.

## The beds, in plain terms

A [bed](../glossary.md#bed) is grains poured into a square box and left to
settle. Sizes are lengths, a grain's longest side.
[Boulders](../glossary.md#boulders) are 20-70 cm long.
[Small stones](../glossary.md#small-stones) are 8-16 cm long and stand in
for Dimorphos's finer material. Nothing finer is simulated: sand or gravel
would take millions of grains.

| bed | box | what is in it | where, state |
|---|---|---|---|
| **study bed 1** | 4.5 m x 4.5 m | 0.9 m of boulders 30-70 cm long, [even size mix](../glossary.md#even-size-mix) (every length equally likely) | lab Mac, settling. A laptop copy is settled and has its 36 drops |
| **study bed 2** | 4.5 m x 4.5 m | 0.9 m of boulders 20-70 cm long, [realistic size mix](../glossary.md#realistic-size-mix) (many small, few big, as measured on Dimorphos) | lab Mac, settling |
| **study bed 3** | 4.5 m x 4.5 m | bed 2, 1.5 m deep | lab Mac, settling |
| **study bed 4** | 4.5 m x 4.5 m | bed 2 + a [thin top layer](../glossary.md#thin-top-layer) of small stones (8 cm, about one stone deep: boulder tops stick out) | lab Mac, settling |
| **study bed 5** | 4.5 m x 4.5 m | a [bottom layer](../glossary.md#bottom-layer) of small stones (30 cm, stones 10-20 cm long), then bed 4 on top | lab Mac, settling |
| **study bed 6** | 4.5 m x 4.5 m | bed 2 + a [thick top layer](../glossary.md#thick-top-layer) of small stones (25 cm) | lab Mac, settling |
| laptop: bed 1 at 4 m | 4 m x 4 m | bed 1's recipe | settled, 36 drops |
| laptop: bed 2's boulders at 3 m | 3 m x 3 m | bed 2's boulders, no small stones | settled, 36 drops (panels folded) |
| laptop: quick bed with small stones | 2 m x 2 m | 0.5 m of boulders 20-50 cm long + a thin top layer | settled, 12 drops (panels folded) |
| laptop: bed 1's boulders with a thin top layer | 3 m x 3 m | bed 1's boulders + a thin top layer | **settling now**, its 36 drops start on their own |

## How big must a bed be?

A drop does not simulate the whole bed. Only the grains in a circle around
the landing spot can move: the [patch](../glossary.md#patch). Around it a
[fixed ring](../glossary.md#fixed-ring) of grains is held still, and the
rest of the bed is fixed ground. So what a bed needs is room for:

- **the patch**, which grows with the craft: radius 1.2 m with the panels
  open (the craft is 1.47 m across), 0.7 m with them folded (37 cm long);
- **0.75 m on every side between the patch and the box walls**: one largest
  boulder (70 cm) plus 5 cm. Next to the walls the grains packed against a
  flat wall, not like natural ground;
- **the spread of the landing spots**: the 36-drop set uses a 3 x 3 grid
  with spots 30 cm apart, 0.6 m in all.

Side of the box = 2 x (patch radius + 0.75 m) + spread of the spots. With
the panels open: 2 x 1.95 + 0.6 = **4.5 m**, the size of the study beds. With
them folded and spots 5 cm apart: 2 x 1.45 + 0.1 = **3.0 m**, the laptop beds.
A 2 m box fits a patch of only 0.25 m with these boulders, about the folded
craft's own reach (0.23 m) with no room to slide, so **2 m beds are not used
for drops any more**.

The box size only costs settling time, once per bed. A drop's cost depends
on the patch, not on the box. So the six study beds stay at 4.5 m: that is
the smallest box that fits the open-panel craft and the 3 x 3 grid of spots.

Is the patch itself big enough? A test on the laptop's copy of bed 1 ran
the same 20 drops with patches of 0.9, 1.2 and 1.45 m (panels open) and
0.45, 0.7 and 1.0 m (panels folded); every first touch was identical.

- **Panels open: the 1.2 m patch is enough.** The median
  [speed ratio](../glossary.md#speed-ratio-e) was 0.22 / 0.32 (straight
  down / 40 degrees) at every patch size. Drop by drop it changed by less
  than the difference between landing spots 4 cm apart. A 0.9 m patch would
  save half the computing, but twice as many drops slid out of it (6 of 20
  against 3). A 1.45 m patch bought nothing. **So no bigger bed is needed.**
- **Panels folded: single drops are very sensitive.** The folded craft hits
  with one edge or corner, and the same drop's speed ratio moved by up to
  0.27 between patch sizes. The open craft lands over several touches,
  which averages this out. The 0.7 m patch gave medians 0.04 lower than the
  1.0 m one, but the 0.45 m patch did not, so this looks like chance in 20
  drops rather than a real patch effect. It is not settled, so the 3 m
  laptop beds are for comparing beds, not for paper numbers
  ([details](notebook/2026-11-01.md)).

## The state of every piece of work

| piece of work | where | state | what happens next |
|---|---|---|---|
| The six study beds, three seeds each for beds 1-4 | lab Mac | settling since Fri 30 Oct, about 11:15. **Nothing pushed to GitHub since Fri 07:57** | First look when you are there: are the windows running, and does `python3 lab.py sync --reports-only` push? |
| The 36-drop sets on the lab Mac's beds | lab Mac | start on their own as each bed settles | Made with Friday's code: the open-panel craft's centre of mass is off and the end-of-drop test is the old one. After `git pull`, re-simulate each drop set when it finishes (commands below). |
| Study bed 1, laptop copy (same bed ID as the lab Mac's) | laptop | settled 31 Oct, passed the quality checks | The laptop's reference for checking the lab Mac's result. |
| Its 36 drops | laptop | re-simulated 1 Nov and checked: 20 last contacts confirmed, 11 next touch off the patch, 1 long hop back onto it, 4 left the patch | In the [drop reel](drop-reel.md). |
| The 4 m and 3 m test sets | laptop | re-simulated 1 Nov and checked: 4 m 24 confirmed + 8 off the patch, 3 m 30 + 2; no drop in any set ended too early | In the reel. |
| The quick 2 m bed with small stones | laptop | 12 drops, all checked | In the reel. |
| Bed 1's boulders with a thin top layer, 3 m | laptop | settling: grains still moving 12 mm/s against the 3 mm/s rest test, 11.5 s per step | Reaches rest around 5 am Monday at this pace; its 36 drops start on their own. |
| Friction test: does settling at higher friction help? | laptop | **done: no.** Every run at friction 0.8 or 0.9 made the solver fail, and the one that finished took half again as many steps | Settling stays at friction 0.6, the same as the drops. |
| Patch-size test (is the patch big enough?) | laptop | **done**: with the panels open the 1.2 m patch is enough and the 4.5 m beds stay; with them folded single drops are very sensitive | 3 m laptop beds only for comparing beds. |

## What changed this week, and why

The details are in the [lab notebook](notebook.md); the short reasons:

- **29-30 Oct: the first six-bed night settled nothing.** A leftover setting
  removed the [speed limit](../glossary.md#speed-limit) at the first rest,
  so beds blew apart. Small stones made LMGC90 switch some boulders to a
  contact detection that produced NaNs, and 6 m beds cost ten times the
  estimate. All fixed; beds are now 4.5 m and stop at 3 mm/s
  ([what the first night taught](notebook/bed-study.md)).
- **30 Oct: small stones.** Boulders sank 9 cm into small stones settled
  with the speed rule; the [split contact rule](../glossary.md#split-contact-rule)
  keeps them on top.
- **31 Oct, day: drops made usable.** Movies of every drop, the
  [drop reel](drop-reel.md), and a first round of fixes to how a drop ends.
- **31 Oct, night: the exact last contact.** A free craft feels only
  gravity, so its path and its tumbling are known exactly. They match the
  simulation's own free flights to 0.001 mm and 0.06 degrees. A drop now
  ends when that [exact free flight](methods.md#exact-free-flight) will not
  come back down on a grain the drop can move, and the prediction is then
  checked against the whole bed ([how](methods.md#last-contact-check)).
- **31 Oct, night: the craft's centre of mass.** LMGC90 puts the centre of a
  body made of several pieces at the centre of their volumes, not of their
  masses. Milani's panels are much lighter for their size than its body, so
  with the panels open the craft turned about a point 10.2 mm from its true
  centre of mass. A massless [ballast](../glossary.md#ballast) box inside the
  body now puts the centre in the right place; every drop checks it (to
  1e-11 m). The folded craft (one piece) was never affected.
- **1 Nov: plain words and true sizes.** The pages used the code's names
  (veneer, fines, crop, frozen ring, stowed, deployed) and gave half-lengths
  as sizes. Every page now uses the [glossary](../glossary.md)'s words, and
  sizes are lengths.

## What the drops show so far

From the re-simulated laptop drops (medians):

- With the panels open the craft does not bounce once: it lands on a panel,
  tips, and leaves after 3-4 [touches](../glossary.md#touch) spread over
  seconds. Folded, it hits an edge or corner and is gone within a second.
- Arriving at 40 degrees from vertical keeps more speed than straight down:
  speed ratio about 0.38 against 0.23 on bed 1.
- Where the craft lands matters more than how fast it arrives, and single
  drops are sensitive: moving the centre of mass by 1 cm changed single
  drops' speed ratio by up to ±0.1, but the medians by at most 0.02.
- No bounce ended too early. In 21 of 96 the craft's next touch is on the
  bed just outside the patch: for 19, 15 s to 8 minutes later (a new hop),
  for 2 within 6 s, right at the patch's edge.
- On the thin top layer the craft's first touch was a boulder in all 12
  drops: the layer is one stone deep and the boulder tops stick out.

## What is next

1. **When you are next at the lab Mac** (the settles keep running through
   all of it):
   - `git pull`
   - When each drop set has finished:
     `python3 lab.py campaign rerun battery_L1 --stale --workers 8`, then
     `python3 lab.py check-last-contact battery_L1 --workers 8`
   - Then `python3 lab.py site --publish`.
2. Tonight on the laptop: the 3 m bed with a thin top layer settles, then
   its 36 drops run, are checked and get movies.
3. Through the week: the lab Mac's beds finish and their drop sets run.
   Then compare the beds: which change moves the bounce, by how much, at
   what cost.
4. Choose the bed for the position study and its grid of landing spots.

## Decisions that need you

- **This site is public.** The documentation repository is public, so the
  drop reel and these notes are visible to anyone with the link. Say if they
  should move somewhere private before the paper.
- **Re-simulating the lab Mac's weekend drops is necessary**: the
  open-panel craft's centre of mass was wrong. With the new end-of-drop test
  a set of 36 takes about two CPU-hours.
- **Hops that land just outside the patch.** For about one bounce in five
  (21 of 96), the craft's next touch is on the bed outside the patch. These
  count as bounced off, with the next touch recorded in the reel. I
  recommend keeping that for the 19 that land 15 s or more later (the bounce
  is complete; the hop is the orbit model's job), and flagging the 2 that
  touch the patch edge within 6 s as "patch too small".
- **The position study's landing area.** On a 4.5 m bed the landing spots
  fit in a 0.6 m x 0.6 m square, less than one craft-width of ground. A
  wider area on the same bed needs a wider box: 6 m gives a 2.1 m x 2.1 m
  square. That is a choice for when the bed for the position study is
  picked.
- **ParaView on the lab Mac**: movies need it (or the run folders copied to
  the laptop). Installing it there is a five-minute job.

## Where to find things

- [The drop reel](drop-reel.md): every drop, with its movie and analysis,
  organized by bed type or start conditions.
- [Every run so far](runs.md) (regenerated): every bed, drop set and test, on
  both machines.
- [How each number is computed](methods.md): definitions, justifications,
  limits and the code, for every number on these pages.
- [Lab notebook](notebook.md): the dated notes, the study plan, the decision
  log, the failure catalogue.
- [Glossary](../glossary.md): every term these pages use, in plain words.
- The LMGC90 chapters of this guide: [settling](../settling.md),
  [impacts](../impacts.md), [failure modes](../failure-modes.md).

## How this page stays current

At the end of each work session Claude rewrites the text above and runs
`python3 lab.py site --publish`. That regenerates the run log, the snapshot,
the methods page and the notebook from the code repository and pushes them
here; the site rebuilds itself within a few minutes. The lab Mac's morning
script can run the same command, so its results appear without anyone at
the laptop.
