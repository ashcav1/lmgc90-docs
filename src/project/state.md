# Where things stand

_Written 2 November 2026. The tables are regenerated from what the lab Mac pushes
to GitHub._

**The short version.** We simulate Hera's [Milani](../glossary.md#milani) CubeSat
landing on ground like [Dimorphos](../glossary.md#dimorphos)'s and measure how it
bounces. The study now has one job: settle [three standard beds](beds.md), run
[the same drops](results.md) on each, and compare.

{{#include generated/status.md}}

**What the lab Mac is doing.** It runs by itself. It settles the three beds and two small
test beds. No impact is simulated until all three beds have settled. Then the drops start, followed by the last-contact check, the movies, the
reports and the [drop reel](drop-reel.md). Everything is pushed to GitHub when a step ends, whether
it succeeded or failed, and every 2 hours while it runs.

**What we know so far:**

- **Bed 3 has settled.** Its 22,572 stones came to rest in 1 hour 45 minutes (1,900 steps), level,
  with stones on all of the surface.
- **Bed 2's recipe works in its 2 m test.** It came to rest in 40 minutes, with stones on
  96 % of the surface and the boulders' tops a median 5 cm below the stones around them.
  The full bed is settling and should take about 4 hours.
- **Bed 1 is still settling,** with a typical grain moving 4.5 mm/s at the last report. At rest is under 3 mm/s.
- **"Failed" in the table above means one [quality check](../glossary.md#quality-checks-qa).**
  Small stones sink slightly into each other while settling, and the worst pair's overlap
  was over the old limit: 73 % of the thinnest stone's half-thickness on bed 3, against a limit of 50 %.
  The limit is now 100 %, with a warning that stays on the bed ([why](plan.md)). The table
  changes when the run restarts after bed 1 finishes; those beds are then graded
  again without settling again.

**What's next:** [the plan](plan.md).

## Where to find things

- [The three beds](beds.md): the recipes, the real pictures and numbers for every seed.
- [Drop results](results.md): every drop set, and the three beds side by side.
- [The drop reel](drop-reel.md): every drop with its movie and analysis.
- [The bed library](library.md): every settled bed, and how to name, check and move one.
- [How each number is computed](methods.md), and the [glossary](../glossary.md).
- [Legacy](legacy.md): everything before 2 November.
