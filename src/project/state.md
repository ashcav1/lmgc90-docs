# Where things stand

_Written 2 November 2026. The tables are regenerated from what the lab Mac pushes
to GitHub._

**The short version.** We simulate Hera's [Milani](../glossary.md#milani) CubeSat
landing on ground like [Dimorphos](../glossary.md#dimorphos)'s and measure how it
bounces. The study now has one job: settle [three standard beds](beds.md), run
[the same drops](results.md) on each, and compare.

{{#include generated/status.md}}

**What the lab Mac is doing.** It runs by itself. All three beds have settled, so the next
start runs the drops, followed by the last-contact check, the movies, the reports and the
[drop reel](drop-reel.md). Everything is pushed to GitHub when a step ends, whether it succeeded
or failed, and every 2 hours while it runs.

**What we know so far:**

- **All three beds have settled, each in under 4 hours.** No drop has run on them yet.

| bed | what settled | time | steps | tilt | what the surface is |
|---|---|---|---|---|---|
| 1, boulder field | 1,289 boulders | 3 h 32 min | 9,400 | 0.7° | all boulders |
| 2, embedded boulders | 295 boulders, then 7,533 stones | 3 h 48 min | 1,700, then 2,000 | 0.3° | stones on 98 %; the boulders that show have tops a median 6 cm below the stones around them |
| 3, stone blanket | 22,572 stones | 1 h 45 min | 1,900 | 0.0° | all stones |

- **Bed 2 met its target:** at least 70 % stones on top, and boulders 5 cm or less above them.
- **"Failed" in the table above means one [quality check](../glossary.md#quality-checks-qa)**, on
  beds 2 and 3 and the two test beds. Small stones sink slightly into each other while settling,
  and the worst pair's overlap was over the old limit of 50 % of the thinnest stone's half-thickness:
  79 % (2.4 cm) on bed 2, 73 % (1.9 cm) on bed 3. The limit is now 100 %, with a warning that stays
  on the bed ([why](plan.md)). The table changes when the run restarts; those beds are then graded
  again without settling again. Bed 1 has no small stones and passed every check.

**What's next:** [the plan](plan.md).

## Where to find things

- [The three beds](beds.md): the recipes, the real pictures and numbers for every seed.
- [Drop results](results.md): every drop set, and the three beds side by side.
- [The drop reel](drop-reel.md): every drop with its movie and analysis.
- [The bed library](library.md): every settled bed, and how to name, check and move one.
- [How each number is computed](methods.md), and the [glossary](../glossary.md).
- [Legacy](legacy.md): everything before 2 November.
