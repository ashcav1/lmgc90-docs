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

No settled beds yet.
