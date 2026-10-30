# A pipeline on top: the Milani bounce study

The code this book was written against is a Python pipeline for CubeSat
touchdowns on boulder beds (private repository `ashcav1/CSML_LMGC90`). Its
structure is a reasonable template for any LMGC90 study of rigid grains:

```
src/lmgcx/      LMGC90 made safe: session (boot order, step, getters, cap),
                deck (contact tables by color, laws, postpro), geometry
                (convex hulls, deposition, height maps), QA, crash signatures
src/milani/     the study: config dataclasses (hashed into run ids), beds,
                composer (crop + spacecraft deck), impact loop, detectors,
                recorder, campaigns (shards, resumable catalog), plots,
                reports, estimates, a scripting API
configs/        beds, impactors, contact laws, campaigns — commented TOML
tests/          unit tests, LMGC90 scenes (restitution, spin frame,
                invisible walls, frozen pairs, deployed body, cohesion,
                round trip), a smoke chain
```

## Session: one object, the right call order

```python
from lmgcx.session import Session, SolverSettings, DetectionSettings
s = Session(Path("DATBOX"), SolverSettings(dt=2e-3, tol=1e-3, max_checks=1000),
            DetectionSettings(cundall_iter=5000)).boot()
for k in range(n):
    s.step()                 # IncrementStep … UpdateStep in the documented order
    s.cap_speed(0.3)         # RBDY3_PartialDamping, inert below the cap
V = s.vectors("V____")       # (nb, 6) all bodies; body-frame spin in 4:6
I = s.contacts()             # getInteractions table
s.save_state("state.h5"); s.finalize()
```

`boot()` performs the initialisation sequence of [Initialisation order](init-order.md),
including `PRPRx_UseCpCundallDetection`, and refuses to run without the
`polytopes_area_oob` patch.

## Deck: contact tables by kind

```python
from lmgcx.deck import ContactTable, Group, make_laws
table = ContactTable()
table.allow(Group("BOULD", "POLYR"), Group("BOULD", "POLYR"), "rockx", alert=0.01)
table.allow(Group("BOULD", "POLYR"), Group("FLOOR", "PLANx"), "rockx", alert=0.01)
tacts, laws = make_laws(pre, {"rockx": {"law": "GAP_SGR_CLB", "fric": 0.6}})
pre.writeDatbox(3, mats, mods, bodies, tacts, table.to_pre(pre, laws), post=..., gravy=[0, 0, -9.81])
```

Every pair that must interact is listed; frozen–frozen pairs are not.

## The bed library

`runs/beds/<id>/` + `LIBRARY.json` + a `MANIFEST.json` of checksums per bed;
`lab.py beds` lists / verifies / exports / imports; `@name` or an id in a
campaign pins the folder; `ensure_bed` verifies the manifest and refuses a
modified bed.

## The scripting layer

```python
from milani import api
bed = api.settle("ladder_L2_weibull")          # settled once, reused by id
run = api.drop(bed, speed=0.06, angle=30, attitude="fixed",
               impactor="milani_deployed", laws="regolith_cohesive_10pa", crop=1.2)
print(run.outcome, run.e_speed); run.plot()
camp = api.campaign("battery_L2", workers=8)   # resumable sweep with a shared catalog
```

Run identity is a hash of the fully resolved physics configuration plus a
physics version string, so a result is never mistaken for another; every
run writes a 20 Hz trajectory, every contact on the craft, grain motion,
the contact network, energies and solver health, and the standard figures
are drawn from those files.
