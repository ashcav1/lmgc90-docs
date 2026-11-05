# Where things stand

_Written 5 November 2026. The tables are regenerated from what the lab Mac pushes
to GitHub._

**The short version.** We simulate Hera's [Milani](../glossary.md#milani) CubeSat
landing on ground like [Dimorphos](../glossary.md#dimorphos)'s and
[Didymos](../glossary.md#didymos)'s, and measure how it bounces. The study now has one job:
settle [six standard beds](beds.md), run [the same drops](results.md) on each, and compare.

{{#include generated/status.md}}

**What the lab Mac is doing.** The drops in [the impact plan](plan.md#the-impact-plan), approved on 4 November: the drop test (done), then the rest of bed 1 at both gravities on all 15 cores. After that, beds 2 and 3 start side by side, 4 cores per set, with the drops at the centre spot first and in a mixed order, so that by Monday each set has an even sample and a measured cost.

**The first run of bed 1 only landed at the centre spot.** A safety check that the patch fits between the landing spot and the walls used a stricter rule than the one that chose the 9 spots, so it refused the other 8: 160 of 1,440 drops ran, and 1,280 failed in seconds. The check is fixed. Running again keeps the 160 and does only the rest.

**The drop test found little effect from depth.** All 20 pairs had the same outcome with grains free to 30 cm and to 60 cm. The speed ratio typically changed by 0.03, against a spread of about 0.3 from moving the landing spot just 4 cm. Straight down there was no shift on average, and at 40° it was about 0.05 higher at 30 cm. That was on bed 1's boulders only.

**A drop costs more than estimated:** about 0.25 core-hours on bed 1, not 0.13, and the checks and movies add about half as much again.

**What we know so far:**

- **All six beds are ready.** That is three grounds with two pours each. Bed 1 passed every quality check. The others carry a warning for small overlaps between stones.
- **Version D is bed 2.** It has small stones below the rocks and more above them, and it is the only version that met all the targets: stones on 40-70 % of the surface, at least 2 rocks per m² sticking out, by at least 8 cm, and fewer holes than A.

| version | stones on top | rocks sticking out, per m² | typically by | empty space | big holes | deep | settling time |
|---|---|---|---|---|---|---|---|
| A | 37-38 % | 6.3-7.0 | 17-18 cm | 37-38 % | 1.1-1.3 % | 0.31-0.32 m | 4.7-4.8 h |
| B | 31-32 % | 7.5-7.9 | 16-17 cm | 40 % (one pour) | 2.1 % (one pour) | 0.29-0.31 m | 2.2-2.8 h |
| C | 27 % | 3.8-4.1 | 33-35 cm | 37 % (one pour) | 4.4 % (one pour) | 0.34-0.36 m | 3.3-3.5 h |
| **D (picked)** | 53-54 % | 5.4 | 16-18 cm | 34 % | 0.1 % | 0.41-0.42 m | 21.7-24.5 h |
| E | both pours crashed on the last layer | | | | | | |

  "Empty space" is the share of the space under the surface that no grain fills, and "big holes" is the share in holes that fit a 10 cm ball.
- **Version E, with stones between two thin layers of rocks, never finished.** One pour blew up with a stone spinning at about 110 turns a second, and the other stopped with an error just after first coming to rest. Each did the same thing when tried again.
- **Version D is closer to the real ground.** Boulders are 59-61 % of its volume, against 81 % in version A. A study of the DART impact infers at most about 40 % near the surface.
- **About 3 in 10 grains were still moving faster than 2 cm/s when version D passed the rest test,** against 1-2 in 10 on version A. It needs watching in the first drops.
- **Rounder grains worked.** The worst overlap the craft can reach fell from 79 % to 16-20 % of the thinnest stone's half-thickness on bed 2, and from
  73 % to 19-37 % on bed 3. Bed 3 needed about half the stones (12,100, not 22,600) and bed 1 settled in about 1 hour, not 3.5.
- **Gravity does not change a settled pile**, so [each body's gravity goes into its drops](beds.md).
- **The fastest drops were capped.** Planned landings on Dimorphos are about 7-9 cm/s, and a craft leaves again above about 8-10 cm/s, so drops stop at 16 cm/s there and at 32 cm/s on Didymos.

**What's next:** [the plan](plan.md).

## Where to find things

- [The beds](beds.md): the recipes, the real pictures and numbers for every seed.
- [Drop results](results.md): every drop set, and the three beds side by side.
- [The drop reel](drop-reel.md): every drop with its movie and analysis.
- [The bed library](library.md): every settled bed, and how to name, check and move one.
- [How each number is computed](methods.md), and the [glossary](../glossary.md).
- [Legacy](legacy.md): everything before 2 November.
