# Polyhedron contact detection

Polyhedron–polyhedron (`PRPRx`) detection must be selected explicitly;
without a call to one of the `PRPRx_Use…Detection` functions no POLYR pair is
ever detected.

## Modes

| call | method | what we measured |
|---|---|---|
| `PRPRx_UseCpCundallDetection(nb_iter, cd_shrink=0, an_shrink=0, delta=0)` | Cundall's iterative common plane; contact points are the vertices projected near the plane; a face–face pair yields up to four points with an area | **the one that works** for random convex hulls; the contact set of a resting pile flickers by a few contacts per 50 steps (see [Settling](settling.md)) |
| `PRPRx_UseCpF2fExplicitDetection(tol, …)` | face-to-face combinatory: face pairs whose normals agree within `tol` | **zero grain–grain contacts** on our triangulated random hulls (no two triangles parallel within tol): every grain fell through the pile onto the floor |
| `PRPRx_UseCpF2fDetection(tol, iter, …)` | F2F where faces match, Cundall for the rest | same result as above in our hands: zero contacts |
| `PRPRx_UseNcDetection(gdist)`, `…NcF2f…` | non-convex shapes | not used |
| `PRPRx_UseStoDetection`, `…TrianglesIntersection…` | other strategies | not used |

Rule: never switch detection without a scene that counts `POLYR-POLYR`
contacts afterwards. Our bed QA fails any settle with fewer than one
grain–grain contact per grain.

## Companion settings

- `PRPRx_LowSizeArrayPolyr(size)`: initial per-polyhedron candidate array;
  10 is generous for 12–20-vertex grains.
- `PRPRx_SetCundallNeighbor(ratio)`: width of the band around the common
  plane whose projected vertices become contact points (0.1 reduced the
  contact-set flicker slightly in one test: 4.0 vs 5.4 contacts per 50
  steps; not conclusive).
- `PRPRx_ShrinkPolyrFaces(shrink)`: shrinks candidate faces for detection.
- `PRPRx_SetTolRecupRloc(tol)`: how far a contact point may move between
  steps and still inherit its previous reaction (warm start).
- `PRPRx_SetInternalSurface(itype, value)`: a surface value for the
  triangle-intersection detection only.

## Alert distance

The see-table `alert` is the detection radius: pairs closer than it become
candidates. Too small and a fast body steps past its neighbour before being
detected (then the position rule ejects it); too large and the candidate lists
explode. We use a tenth of the smallest semi-axis of the kind involved and
cap speeds while a layer deposits so that no grain moves more than 0.4 of
that per step.

## Other pairs

- `PRPLx` (polyhedron–plane) works out of the box; its contacts carry **no
  area**, so a cohesive law on a plane pair bonds in status only (zero force).
- `SPPRx` (sphere–polyhedron) misses approaching contacts in this build:
  spheres sink a full radius into boulders. We use polyhedra for everything.
- `SPSPx` (sphere–sphere) is fine but has no rolling resistance law.
