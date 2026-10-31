# Impacts on a settled bed

## Crop, don't simulate the whole bed

A run takes the settled bed, keeps the grains within a radius of the aim
point as **free**, holds the grains that touch them as **frozen** (driven
with zero velocity), and drops the walls. Frozen–frozen and frozen–floor
pairs get **no** contact rule (two fixed bodies with a rule give `Wnn = 0`
and a NaN). Reactions on frozen bodies are meaningless as loads (rigid
grains wedged between driven bodies carry arbitrary self-stress); crop
adequacy is graded kinematically from the speed of the free grains at the
edge.

## Writing the deck from the settled state

Read the settled stage back with `pre.readDatbox(…, hfile="state.h5",
step=-1)`, call `updateReferenceConfig()` on each avatar and zero its dofs,
select the crop, add the spacecraft (one avatar, one or several `POLYR`
contactors with explicit volume / inertia / shift), build the see table with
one law for rock–rock and one for craft–rock, and `writeDatbox`. The
round trip reproduces every vertex to < 10 µm.

## Velocity initialisation

Set `V____`, `Vbeg_` and `Vfree` of the spacecraft with `RBDY3_PutBodyVector`
before the first step. `Coor_` cannot be put (Fortran abort): reposition by
shifting `X____`/`Xbeg_` and verify with a `Coor_` readback. Spin in
`V____(4:6)` is in the **body** frame; `GetBodyMatrix` rows are the body
axes. LMGC90's body frame is the principal-axes frame sorted by ascending
moment of inertia.

## Starting height and ending

Ray-cast every vertex of the craft along its approach against the bed's
height map and start 2 cm before first contact; at 6 cm/s that is 0.3 s of
free flight. End on: detached (rising with a clear ballistic path for 0.5 s),
rest (below 2 mm/s for 10 s), captured (fastest point below 10 % of the
arrival speed for 10 s — friction at 5e-5 m/s² takes minutes to stop a
creep), left the free core, or the time cap.

Two lessons about "clear ballistic path" from 36 drops on a 4 m bed
(2026-10-31):

- **Spheres are the wrong obstacle.** A look-ahead that keeps the craft's
  vertices away from every grain's circumsphere never clears a 1.47 m craft
  skimming 20 cm above a boulder field: it sits inside those spheres until
  it has risen above the tallest one, a minute at micro-g. Vertical drops
  hovered for 40–100 s, or drifted out of the 1.2 m crop first and were
  voided. Use the bed's height map — the real top envelope of the grains,
  raised by the largest grain displacement since the impact — as the
  obstacle, and keep the sphere test only as a second way to pass.
- **Rotate the craft along the path.** A craft that bounces with a slow
  spin (0.05–0.1 rad/s) has a wing coming down within seconds; a
  translation-only look-ahead declared it detached 0.5 s after the first
  touch and cut off the 3–6-touch tumble that follows (first-touch e of
  0.31–0.35 against 0.21–0.26 at the end of the landing). Propagate the
  vertices with the spin (Rodrigues) over at least half a turn.
- A craft that crosses the crop edge airborne and rising, after the
  detachment time, has completed its bounce: call it detached with the exit
  state of its last contact. On the ground it has left the patch.

## The settle's instability does not reach the impact deck

Impact decks use `RST_CLB` for grain pairs: it never ejects a residual
overlap, so the gap-law metastability of a settled pile is absent. Measured:
108 free grains moved a median 0.13 µm (max 14 µm) in the 0.42 s before the
craft's first contact at 5e-5 m/s²; after the bounce, up to 19 mm.

## Time step for impacts

An automatic rule from arrival speed and the smallest grain keeps travel per
step small against the alert distance (5 ms at cm/s on decimetre grains).
Detecting contacts every 2 or 4 steps or doubling dt changed the contact
set and the restitution by 5–30 %: the contact physics is not a speed knob.

## Sensitivity (one drop, 5 cm/s, 20°, stowed 12 kg box, 2.4 m bed)

| arm | e = speed out / in |
|---|---|
| baseline (rock–rock friction 0.6, restitution 0.5) | 0.400 |
| rock–rock restitution 0.3 / 0.7 | 0.399 / 0.401 |
| rock–rock friction 0.7 | 0.433 |
| cohesive bed, 10 Pa / 100 Pa | 0.455 / 0.457 |

The craft–rock pair governs the first bounce; the rock–rock restitution
hardly matters, friction and cohesion stiffen the bed. And the bed itself
matters most: the same drop on beds generated with 8 / 12 / 20-vertex
grains (three random realisations) gave e = 0.17 / 0.40 / 0.51 — the first
one hit fines and slid out of the crop, the others hit a boulder — a larger
spread than any contact-law arm. Where the craft lands dominates what it does, which
is why the study sweeps landing spots on one bed before anything else.
