# Where things stand

_Written Saturday 31 October 2026, late night. This page is rewritten at the
end of every work session; the tables marked "regenerated" are rebuilt from
the code repository each time the site is published._

**The short version.** We simulate Hera's [Milani](../glossary.md#milani)
CubeSat landing on [Dimorphos](../glossary.md#dimorphos)-like rubble and
measure how it bounces. Tonight's question was "how do we know a drop's last
contact really is the last?". It now has an exact answer: once the craft is
free, its future is computed exactly and checked against the whole bed. On
the way, that check exposed a modelling bug. In every drop with the wings
deployed, the craft had been turning about a point 1 cm from its true centre
of mass. Both are fixed, and all 108 laptop drops were re-simulated and
checked. The lab Mac has been settling the six study beds since Friday, but
nothing from it has reached GitHub since Friday morning. Its drops were made
with the old code and need the same re-simulation.

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
| The six production beds, three seeds each for beds 1-4 | lab Mac | settling since Fri 30 Oct, about 11:15. **Nothing pushed to GitHub since Fri 07:57** | First look when you are there: are the windows running, and does `python3 lab.py sync --reports-only` push? |
| The standard 36 drops on the lab Mac's beds | lab Mac | start on their own as each bed settles | Made with Friday's code: the deployed craft's centre of mass is off and the end-of-drop test is the old one. After `git pull`, re-simulate each drop set when it finishes (commands below). |
| Bed 1 settled on the laptop (same bed id as the lab Mac's) | laptop | done 31 Oct, QA PASS | The laptop's reference for checking the lab Mac's result. |
| The 36 drops on bed 1 | laptop | **re-simulated tonight** with both fixes and checked: 20 last contacts confirmed, 11 with the next touch off the patch, 1 long hop back onto it, 4 left the patch | In the [drop reel](drop-reel.md) with new movies. |
| The 4 m bed and 3 m Weibull bed test sets | laptop | **re-simulated tonight** and checked: 4 m 24 confirmed + 8 off the patch, 3 m 30 + 2; no drop in any set ended too early | In the reel. |
| The first drops onto fines (2 m bed, fines veneer) | laptop | bed settled tonight; its 12 drops are running, already with tonight's code | Added to the reel when they finish. |
| A 3 m bed with a fines veneer | laptop | still settling (its fines layer, slowly) | Overnight; its 36 drops start on their own. |

## What changed this week, and why

The details are in the [lab notebook](notebook.md); the short reasons:

- **29-30 Oct: the first six-bed night settled nothing.** A leftover guard
  released the [speed cap](../glossary.md#speed-cap) at the first rest, so beds
  blew apart. Fines made LMGC90 switch some boulders to a contact detection
  that produced NaNs, and 6 m beds cost ten times the estimate. All fixed;
  beds are now 4.5 m and rest at 3 mm/s
  ([what the first night taught](notebook/bed-study.md)).
- **30 Oct: fines.** Boulders sank 9 cm into fines settled with the velocity
  law; the [split law](../glossary.md#split-law) keeps them on top.
- **31 Oct, day: drops made usable.** Movies of every drop, the
  [drop reel](drop-reel.md), and a first round of fixes to how a drop ends.
- **31 Oct, night: the exact last contact.** A free craft feels only gravity,
  so its path and its tumbling are known exactly. They match the
  simulation's own free flights to 0.001 mm and 0.06°. A drop now ends when
  that [exact free flight](methods.md#exact-free-flight) will not come back
  down on a grain the simulation can move. The prediction is then checked
  against the whole bed, including the parts the simulation leaves out
  ([how](methods.md#last-contact-check)). Drops end sooner, with the same
  results where nothing else changed.
- **31 Oct, night: the craft's centre of mass.** That exact check disagreed
  with the solver by a centimetre. The reason is how LMGC90 builds a body
  made of several pieces: it puts the body's centre at the centre of the
  pieces' volumes, not of their masses. Milani's wings are much lighter for
  their size than its bus, so the deployed craft turned about a point
  10.2 mm from its true centre of mass in every deployed drop so far. A
  massless [ballast](../glossary.md#ballast) box hidden inside the bus now
  puts LMGC90's centre in the right place; every drop checks it (to
  1e-11 m) ([the craft model](methods.md#the-craft-model)). The stowed box
  was never affected.

## What the drops show so far

From the re-simulated laptop drops (medians):

- The deployed craft does not bounce once: it lands on a wing, tips, and
  leaves after 3-4 [touches](../glossary.md#touch) spread over seconds. The
  stowed box hits an edge or corner and is gone within a second.
- Arriving at 40 degrees from vertical keeps more speed than straight down:
  [speed ratio](../glossary.md#speed-ratio-e) about 0.38 against 0.23 on bed 1.
- Where the craft lands matters more than how fast it arrives, and the
  deployed craft is sensitive: moving its centre of mass by 1 cm changed
  single drops' e by up to ±0.1, but the medians by at most 0.02.
- No bounce ended too early. In 21 of 96 the craft's next touch is on the
  bed just outside the simulated patch: for 19, 15 s to 8 minutes later (a
  new hop), for 2 within 6 s, right at the patch's edge.

## What is next

1. **When you are next at the lab Mac** (the settles keep running through
   all of it):
   - `git pull`
   - When each drop set has finished:
     `python3 lab.py campaign rerun battery_L1 --stale --workers 8`, then
     `python3 lab.py check-last-contact battery_L1 --workers 8`
   - Then `python3 lab.py site --publish`.
2. Overnight on the laptop: the fines drops, the 3 m veneer bed, and a full
   replay check of the 3 m set to test the shortcut the check uses.
3. Through the week: beds 2-6 finish and their drop sets run. Then compare
   the beds: which change moves the bounce, by how much, at what cost.
4. Choose the bed for the position study and configure its grid of landing
   spots.

## Decisions that need you

- **This site is public.** The documentation repository is public, so the
  drop reel and these notes are visible to anyone with the link. Say if they
  should move somewhere private before the paper.
- **Re-simulating the lab Mac's weekend drops is now necessary**, not
  optional: the deployed craft's centre of mass was wrong. With the new
  end-of-drop test a set of 36 takes about two CPU-hours.
- **Hops that land just outside the patch.** For about one bounce in five
  (21 of 96), the craft's next touch is on the bed outside the simulated
  patch. These count as bounced off, with the next touch recorded in the
  reel. I recommend keeping it that way for the 19 that land 15 s or more
  later (the bounce is complete; the hop is the orbit model's job), and
  flagging the 2 that touch the patch edge within 6 s as "crop too small".
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
`python3 lab.py site --publish`. That regenerates the run log, the snapshot,
the methods page and the notebook from the code repository and pushes them
here; the site rebuilds itself within a few minutes. The lab Mac's morning
script can run the same command, so its results appear without anyone at
the laptop.
