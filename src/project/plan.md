# The plan

_Written 2 November 2026._

## Where we want to end up

A paper on what decides whether [Milani](../glossary.md#milani) stays, hops or
bounces away when it lands on ground like [Dimorphos](../glossary.md#dimorphos)'s:
the landing speed and angle, the ground, and where on that ground it lands.

## The steps

| step | what | how long (estimates) | what decides the next step |
|---|---|---|---|
| 1 | Clean start on the lab Mac: old runs stopped and archived | minutes | |
| 2 | Settle [the three beds](beds.md), one seed (one random pour) each for now, plus two 2 m test beds | bed 3 hours, bed 1 up to about a day, bed 2 about 3 days | settled, tilt under 3°, quality checks passed. Bed 2 also needs at least 70 % small stones on top and boulders 5 cm or less above them |
| 3 | **No drop starts until all three beds have settled.** Then a depth test on bed 1: do grains free to 0.3 m rather than 0.6 m below the surface change the bounce? | under an hour | if they do, beds 2 and 3 (0.4 m deep) need deepening |
| 4 | [The same 81 drops](results.md) on each bed: 243 drops | bed 1 about 1 hour, bed 2 about half a day, bed 3 about 2 days (a drop's cost grows with the loose grains under the craft: about 310, 2,200 and 6,400). About 2-3 days in all, after the beds | if bed 3 is too slow, fewer landing spots on it |
| 5 | Compare the beds, write the paper's figures | | |

Steps 2 to 4 run by themselves on the lab Mac.

## Three choices behind the beds

- **4 m x 4 m, with the craft's patch 0.5 m from the walls.** The rule until now, one
  largest boulder plus 5 cm (0.75 m), would squeeze the nine landing spots to within
  5 cm of each other. At 0.5 m they stay 30 cm apart. On the beds measured, the
  surface 0.3-0.5 m from a wall is only 1-3 cm lower than at 0.5-0.7 m.
- **Stones 10-20 cm long in bed 2, 8-16 cm in bed 3.** Halving a stone's size multiplies
  the number of stones by eight, so finer material is out of reach. It is a limit
  the paper states.
- **Settled when the surface stops changing.** Small stones can keep jostling for
  ever: in tests the median stone was still moving 4-6 mm/s after 20,000 steps. Beds
  2 and 3 also count as settled when their height map changes by less than 5 mm
  on average, three measurements in a row, 4 seconds of settling apart. Each drop starts
  every grain at zero speed in near-zero gravity, so the jostling never reaches it.
  Beds settled this way carry a WARN in their quality checks.

## If it does not work

- **Bed 2 misses its target:** its 2 m test shows that within about a day. Change the
  pour and restart bed 2; the other beds are not affected.
- **Bed 2 takes more than about a week:** simplify its recipe, and restart it.
- **A bed fails:** its report, settling history and log reach GitHub by themselves and
  the other beds carry on. No drops start until it is fixed, because every bed must be done first.

**One pour of each bed for now.** One pour can't separate a real difference between beds from the
luck of that pour. More seeds can be added later with one line in `configs/production.toml`.

## How the run is organized

All of this lives in the code repository:

| what | where |
|---|---|
| What runs | `configs/production.toml`, one file |
| The beds | `configs/beds/bed1_boulders.toml`, `bed2_embedded.toml`, `bed3_stones.toml` |
| The drops | `configs/campaigns/drops_bed1.toml`, `drops_bed2.toml`, `drops_bed3.toml` |
| Commands | `python3 lab.py production start`, `status`, `stop` |
| What reaches GitHub | `reports/labmac/`: status, reports, figures, settling histories, height maps, grain positions, every drop's records |
