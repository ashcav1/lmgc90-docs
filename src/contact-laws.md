# Contact laws: what each one really does

Parameters are those of `pre.tact_behav(name, law, **params)`. "Measured"
refers to dated experiments on real decks (rigid polyhedra, 2026).

## `RST_CLB` — restitution + Coulomb

`fric`, `rstn`, `rstt`. Newton restitution on the normal (and tangential)
relative velocity, Coulomb friction. The law activates when a pair is in
contact at the *beginning* of the step (`gapTTbegin ≤ 0`) and then imposes
`vn ≥ −rstn · vn_begin`.

- Measured e = 0.5993 for rstn = 0.5 at dt = 2e-4 (box on plane).
- **It never corrects an overlap.** Dense packings of polyhedra settled under
  it drift 5–16 mm into each other and no time step, tolerance or alert
  distance removes it (our bed QA fails at > 0.1 × smallest semi-axis).
  Use it for the spacecraft–rock pair and for pairs involving small grains
  where the position rule is worse (see below); not for settling boulders.

## `GAP_SGR_CLB` — gap-level Signorini + Coulomb

`fric`. Imposes `gap_end ≥ 0`: an overlap left by one step is pushed out on
the next (at `gap/dt`). Measured overlap after settling: ~1e-19 m.

- The reference law for settling rigid boulders.
- `pre` whitelists it only for deformable contactors; the kernel is generic,
  so our deck writer admits it for rigid pairs.
- Small grains under the position rule **drain through the pores** of an
  overlap-free boulder packing for thousands of steps; small-stone pairs stay on
  `RST_CLB`.
- Friction 0.9 with this law on rigid polyhedra made the NLGS diverge (NaN)
  once; keep settle friction ≤ 0.7.

## `IQS_MOHR_DS_CLB` / `GAP_MOHR_DS_CLB` — Mohr–Coulomb cohesion

`cohn`, `coht`, `dyfr`, `stfr`: normal and tangential cohesion **stresses**
(Pa), dynamic and static friction. In the solver (`mod_nlgs_3D`):

- a pair in contact at step 1 with a virgin status becomes `Mstck`
  (bonded): `IF (Nstep == 1 .AND. statusBEGIN == i_nknow)`;
- the bond force is `cohn × cd_surf × H` where `cd_surf` is the contact area
  the detection measured — a **stress times an area**, not a force;
- the bond holds while the status stays `Mstck`; once a contact slides or
  opens it is lost for good;
- the laws are inelastic (no restitution).

Measured (scene `cohesion`, two cubes under upward gravity):

| setup | result |
|---|---|
| `cohn` given as a force (470 N for a 0.04 m² face) | both cubes lift off: the bond is 19 N |
| `cohn` in Pa, `coht = 0` | the bonded cube holds exactly its weight for one step, status turns `slide`, then free |
| `cohn = coht` in Pa, polyhedron–polyhedron | holds (0.0 mm in 2 s) at 2× weight, lifts off at 0.5× |
| any cohesion, polyhedron–**plane** | status `Mstck`, reaction 0: plane contacts have no area |

Restart decks: bonds form only at step 1 of a deck. Writing a fresh deck
from a settled state (our composer) bonds every touching pair; restarting
through HDF5 with stored statuses does not, unless the step counter is reset
and `RecupRloc` skipped at the first step.

Scale: 10 Pa over a 0.05 m² boulder face is 0.5 N — a thousand times a 12 kg
craft's weight at Dimorphos gravity. A bed with any cohesion at all is a
bonded crust at landing speeds of cm/s; measured restitution of the craft
rose from 0.40 to 0.455 and was the same at 10 and 100 Pa.

## Others present in the build

`IQS_CLB`, `IQS_DS_CLB`, `IQS_CLB_g0`, `GAP_SGR_CLB_g0`, `GAP_SGR_DS_CLB`,
`IQS_WET_DS_CLB` / `GAP_WET_DS_CLB` (non-smooth Lennard-Jones attraction,
`Wthk` thickness), `IQS_STICK`, `IQS_CAP_MOHR_DS_CLB` (with tension/compression
caps), `…_WEAR`, `ELASTIC_REPELL_CLB`, `MAC_CZM` (cohesive zones for
deformables). We have not measured these.
