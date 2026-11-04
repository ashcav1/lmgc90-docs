# The bed library

_The list below is regenerated from what the machines push to GitHub._

A settled bed can take days of computer time, so we keep each one and reuse
it. The bed library is how: it treats a settled bed as a protected file you
can name, check, and carry to another machine.

**What it does for you**

- **Every settled bed is registered.** When a bed passes its
  [quality checks](../glossary.md#quality-checks-qa) (PASS or WARN), it goes
  into the library with its name, seed, grain count, grade and date.
- **Beds are protected.** The files a drop reads are made read-only, and
  their checksums are saved. Every use checks them first: a changed or
  damaged bed is refused with a message, never used silently.
- **You can name a bed directly.** A drop set can say which bed it wants by
  name (`@bed1_boulders:1`, name and seed) or by its id, instead of
  through the recipe. That bed is used as it is, even if the recipe or a
  default changes later.
- **Beds can move between machines.** Export one to a file, copy it, import
  it. A bed settled for days on the lab Mac can then be used on the laptop.

**The commands**

| to do this | run |
|---|---|
| see every bed on this machine | `python3 lab.py beds list` |
| check that none has changed | `python3 lab.py beds verify` |
| pack a bed into one file | `python3 lab.py beds export <bed id> <file>.tar.gz` |
| bring it in on another machine | `python3 lab.py beds import <file>.tar.gz` |

A worked example is in the code repository, `examples/08_bed_library.sh`. The
library lives in `src/milani/bedlib.py`, and its list on each machine is
`runs/beds/LIBRARY.json`.

**A bed's id is its recipe.** The eight characters at the end of an id are a hash of
the recipe, so a drop can never run on the wrong ground by accident.

**The code underneath.** All of this sits on `src/lmgcx`, our library that
makes LMGC90 safe to drive from Python (boot order, contact tables, pouring,
height maps, quality checks), and `src/milani`, the study built on it. The
guide's chapter [A pipeline on top](../pipeline-api.md) explains both, with
code.

## Every settled bed

| bed id | machine | size | grains | settling time | QA | whole-bed tilt |
|---|---|---|---|---|---|---|
| `bed1_boulders-s1-63ea1d69` | lab Mac | 4 m × 4 m | 852 | 1.1 h | PASS | 0.6° |
| `bed1_boulders-s2-bdb6e0b0` | lab Mac | 4 m × 4 m | 859 | 0.9 h | PASS | 0.6° |
| `bed2a_embedded-s1-7793841c` | lab Mac | 4 m × 4 m | 2,795 | 4.8 h | WARN | 0.5° |
| `bed2a_embedded-s2-b060a218` | lab Mac | 4 m × 4 m | 2,794 | 4.6 h | WARN | 0.5° |
| `bed2b_embedded-s1-a15e1623` | lab Mac | 4 m × 4 m | 2,187 | 2.8 h | WARN | 0.4° |
| `bed2b_embedded-s2-79bef9a4` | lab Mac | 4 m × 4 m | 2,188 | 2.2 h | WARN | 0.6° |
| `bed2c_embedded-s1-05332dba` | lab Mac | 4 m × 4 m | 3,170 | 3.3 h | WARN | 2.3° |
| `bed2c_embedded-s2-80495daa` | lab Mac | 4 m × 4 m | 3,167 | 3.5 h | WARN | 1.5° |
| `bed2d_embedded-s1-aff071fc` | lab Mac | 4 m × 4 m | 7,037 | 24.4 h | WARN | 0.3° |
| `bed2d_embedded-s2-e7260ea7` | lab Mac | 4 m × 4 m | 7,041 | 21.7 h | WARN | 0.6° |
| `bed3_stones-s1-804ae8bd` | lab Mac | 4 m × 4 m | 12,113 | 1.4 h | WARN | 0.0° |
| `bed3_stones-s2-6e2f6de9` | lab Mac | 4 m × 4 m | 12,148 | 1.4 h | WARN | 0.2° |
