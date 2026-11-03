# The plan

_Written 2 November 2026._

## Where we want to end up

A paper on what decides whether [Milani](../glossary.md#milani) stays, hops or
bounces away when it lands on ground like [Dimorphos](../glossary.md#dimorphos)'s:
the landing speed and angle, the ground, and where on that ground it lands.

## The steps

| step | what | how long (estimates) | what decides the next step |
|---|---|---|---|
| 1 | Clean start on the lab Mac: old runs stopped and archived | done | |
| 2 | Settle [the beds](beds.md): three grounds, two pours each, and three versions of bed 2 | done 2 November: bed 1 about 1 hour, bed 3 about 1.5 hours, bed 2 2-5 hours | settled, tilt under 3°, quality checks passed |
| 3 | Settle two more versions of bed 2, with stones below, between and above the rocks, and pick one, which leaves six beds | under a day (estimate) | stones on 40-70 % of the surface, at least 2 rocks per m² sticking out, by at least 8 cm, and fewer holes than version A |
| 4 | **No drop starts until then.** A depth test on bed 1 runs first: do grains free to 0.3 m rather than 0.6 m below the surface change the bounce? | under an hour | if they do, beds 2 and 3 (about 0.3 and 0.4 m deep) need deepening |
| 5 | The drops in [the impact plan](#the-impact-plan), once approved | about 7 days in all (estimate, to be replaced by measured costs) | if bed 3 is too slow, fewer drops on it |
| 6 | Compare the beds and the gravities, write the paper's figures | | |

Steps 2, 4 and 5 run by themselves on the lab Mac.

## The choices behind the beds

- **4 m x 4 m, with the craft's patch 0.5 m from the walls.** The rule until now, one
  largest boulder plus 5 cm (0.75 m), would squeeze the nine landing spots to within
  5 cm of each other. At 0.5 m they stay 30 cm apart. On the beds measured, the
  surface 0.3-0.5 m from a wall is only 1-3 cm lower than at 0.5-0.7 m.
- **Stones 8-16 cm long in beds 2 and 3.** Halving a stone's size multiplies
  the number of stones by eight, so finer material is out of reach. It is a limit
  the paper states.
- **Settled when the surface stops changing.** Small stones can keep jostling for
  ever: in tests the median stone was still moving 4-6 mm/s after 20,000 steps. Beds
  2 and 3 also count as settled when their height map changes by less than 5 mm
  on average, three measurements in a row, 4 seconds of settling apart. Each drop starts
  every grain at zero speed in near-zero gravity, so the jostling never reaches it.
  Beds settled this way carry a WARN in their quality checks.
- **Small stones may overlap a little, and each bed reports by how much.** While
  settling, small stones can sink slightly into each other: their contact rule stops
  them closing further but does not push them back out. The check takes the worst pair
  the craft can reach and compares its overlap with half the thickness of the thinnest
  stone. It warns above 15 % and fails at 100 % (raised from 50 % on 2 November). The first bed 3
  measured 73 %, or 1.9 cm. Most pairs are far below that, and rounder stones cut it to about a third on a small test.
  Drops use the same contact rule, so these overlaps should not kick the craft, but no drop has tested that yet.
- **Boulders make up most of bed 2.** Rocks can only stick out of a bed this shallow if there is more rock than stone: about
  70-85 % by volume, not the 40 % a study of the DART impact infers as a limit near the surface. At 40 %, almost no rock shows.
- **Two pours, one for each gravity.** Gravity does not change a settled pile, so a second pour of each ground adds what one
  pour can't give, the luck of the pour. [Didymos](../glossary.md#didymos)'s gravity is about 4 times
  [Dimorphos](../glossary.md#dimorphos)'s, a provisional number until it is checked against the Hera shape model.

## If it does not work

- **No version of bed 2 meets its target:** change the amount of stones and settle bed 2 again; the other beds are not affected.
- **A bed fails:** its report, settling history and log reach GitHub by themselves and
  the other beds carry on. No drops start until it is fixed, because every bed must be done first.
  A bed that settled but failed only a quality check keeps what it settled: when the rule
  changes, it is graded again without settling again.

## The impact plan

Proposed, not yet approved. `docs/Milani_impact_plan.xlsx` lists about 1,570 drops, with the settings of each and a link to its GitHub folder. It is made from
`configs/impact_plan.toml` by `python3 lab.py impact-sheet`, which also pushes it to GitHub.

| tier | what | drops |
|---|---|---|
| 0 | Depth test on bed 1, runs first and gives measured costs | 40 |
| 1 | Main grid on all six beds: 5 speeds x 3 angles x 3 attitudes x 3 landing spots | 810 |
| 2 | The other six landing spots | 144 |
| 3 | Extras on beds 1 and 2: 60° approach, more attitudes, random attitude, approach direction, spin, panels folded, contact properties, up to 1 m/s | 572 |

Speeds double at each step, to well past escape speed: 2-32 cm/s on Dimorphos and 4-64 cm/s on Didymos. Didymos's gravity is about 4 times
stronger, so each speed there should behave like half that speed on Dimorphos; the two ladders test this.

## How the run is organized

All of this lives in the code repository:

| what | where |
|---|---|
| What runs | `configs/production.toml`, one file |
| The beds | `configs/beds/bed1_boulders.toml`, `bed2a_embedded.toml`, `bed2b_embedded.toml`, `bed2c_embedded.toml`, `bed3_stones.toml` |
| The drops | `configs/campaigns/drops_bed1.toml`, `drops_bed2.toml`, `drops_bed3.toml` |
| Commands | `python3 lab.py production start`, `status`, `stop` |
| What reaches GitHub | `reports/labmac/`: status, reports, figures, settling histories, height maps, grain positions, every drop's records |
