## Contact Definitions

---

## `tact_behav(name, law, **kargs)`

**What it does:** Defines a contact interaction law between two bodies (or body
types).

**Arguments:**
- `name` (str, **exactly 5 characters**): identifier used in DAT files
  (e.g. `"rstc1"`)
- `law` (str): name of the contact law.  Valid laws include:
  - `"RST_CLB"` — restitution (Normal/Tangential) + Coulomb friction
  - `"IQS_CLB"` — implicit quasistatic + Coulomb
  - `"WetxCLB"` — wet cohesion + Coulomb
  - (many others — see `lmgc90dicts.py::listeTactBehav`)
- `**kargs`: law-specific parameters:
  - For `RST_CLB`: `fric` (friction coefficient), `rstn` (normal restitution
    coefficient), `rstt` (tangential restitution coefficient)

**Returns:** `tact_behav` instance

**As used in our code:**
```python
b = pre.tact_behav("rstc1", "RST_CLB", fric=0.3, rstn=0.5, rstt=0.5)
tacts += b
```

**Notes:**
- Name must be exactly 5 characters.
- `RST_CLB` is appropriate for rigid granular bodies with restitution and
  friction.  `rstn=0.5` means 50% energy recovery in normal direction.
- A single behaviour object can be referenced by multiple `see_table` entries.

---

## `see_table(CorpsCandidat, candidat, colorCandidat, CorpsAntagoniste, antagoniste, colorAntagoniste, behav, alert, ...)`

**What it does:** Defines which pairs of body types/colors can come into contact
and what contact law governs them.

**Arguments:**
- `CorpsCandidat` (str): body class of candidate — must be in
  `('RBDY2', 'RBDY3', 'MAILx', 'MBS3D', 'MBS2D')`; use `"RBDY3"` for all
  3D rigid bodies
- `candidat` (str): contactor type of candidate — must be in `rigidContactor`
  list (e.g. `"POLYR"`, `"PLANx"`, `"SPHER"`)
- `colorCandidat` (str, 5 chars): color filter for candidates (e.g. `"BLUEx"`)
- `CorpsAntagoniste` (str): body class of antagonist (same options as Candidat)
- `antagoniste` (str): contactor type of antagonist
- `colorAntagoniste` (str, 5 chars): color filter for antagonists
- `behav`: a `tact_behav` instance defining the contact law
- `alert` (float): detection alert distance — bodies within this distance are
  considered in the proximity list; should be slightly larger than typical
  particle gap

**Returns:** `see_table` instance

**As used in our code:**
```python
svs += pre.see_table(
    CorpsCandidat="RBDY3",    candidat="POLYR",  colorCandidat="BLUEx",
    CorpsAntagoniste="RBDY3", antagoniste="PLANx", colorAntagoniste="VERTx",
    behav=b, alert=0.1
)
svs += pre.see_table(
    CorpsCandidat="RBDY3",    candidat="POLYR",  colorCandidat="BLUEx",
    CorpsAntagoniste="RBDY3", antagoniste="POLYR", colorAntagoniste="BLUEx",
    behav=b, alert=0.051
)
```

**Notes:**
- Two tables are needed: POLYR vs PLANx (particle–wall) and POLYR vs POLYR
  (particle–particle).
- `alert` for POLYR-POLYR should be ~5% of the minimum particle radius.  For
  `RMIN=0.2` → `alert=0.051` is appropriate.
- POLYR-POLYR contacts **require** `PRPRx_UseCpCundallDetection` at runtime.
- Color strings allow selective pairing: only particles of `colorCandidat` are
  checked against antagonists of `colorAntagoniste`.  The impactor (also
  `"BLUEx"`) will therefore contact both bed particles and walls.

---

