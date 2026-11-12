# Parameters and their values

Every number the simulations use, with its value. This page is built from the project's configuration files and code each time the site is published, so the values are the ones the runs read. Values marked *measured* come from the finished beds and drops. The results are on [Drop results](results.md), and the sources are listed at the end.

## Contact during a drop

LMGC90 treats every body as rigid and every contact as nonsmooth (contact dynamics). A contact has no stiffness, damping or overlap: it is described by its friction coefficient and two restitution coefficients only. RST_CLB is Newton's restitution with Coulomb friction.

| Contact | Law | Friction μ | Normal restitution e<sub>n</sub> | Tangential restitution e<sub>t</sub> | Basis |
|---|---|---|---|---|---|
| Grain–grain and grain–floor | RST_CLB | 0.6 | 0.5 | 0 | μ: DART fit 0.55, range 0.4–0.7 (Raducan et al. 2024); boulder shapes 0.64 (Robin et al. 2024). e<sub>n</sub>: usual value in lander simulations (Thuillet et al. 2018) |
| Craft–grain and craft–floor | RST_CLB | 0.5 | 0.5 | 0 | Provisional; still to check against MASCOT's bounces on Ryugu (Brisset et al. 2020) |

| Parameter | Value | Note |
|---|---|---|
| Cohesion | 0 Pa | Dimorphos's surface: 0 to a few Pa (Raducan et al. 2024) |
| Rolling resistance | None | The grains are polyhedra, so their shapes resist rolling on their own |
| Frozen ring contacts | With free grains and the craft only | Frozen grains never touch each other or the floor |

## Contact while the beds settle

The beds are poured and settled under Earth gravity before any drop. Settling uses its own contact laws: they only shape the pile, and every drop then uses the laws above.

| Contact | Law | Friction μ | Restitution | How it works | Beds |
|---|---|---|---|---|---|
| Boulder–boulder, boulder–floor and boulder–wall | GAP_SGR_CLB | 0.6 | None (the law has none) | Position-level: an overlap left by one step is pushed out on the next | Beds 1 and 2 |
| Stone–stone, stone–floor and stone–wall | RST_CLB | 0.6 | e<sub>n</sub> = e<sub>t</sub> = 0 | Velocity-level, with no bounce | Beds 2 and 3 |
| Boulder–stone | GAP_SGR_CLB | 0.6 | None (the law has none) | Position-level: an overlap left by one step is pushed out on the next | Bed 2 |

## Contact solver and time step

| Parameter | Drops | Settling | Note |
|---|---|---|---|
| Method | Contact dynamics (LMGC90), rigid bodies | Same | Sánchez et al. 2021 |
| Time integration | θ-method, θ = 0.5 | θ = 0.5 |  |
| Contact solver | Nonlinear Gauss–Seidel (NLGS) | Same | Sweeps over every contact until the forces settle |
| Convergence tolerance | 1.666 × 10<sup>−4</sup> | 10<sup>−3</sup> | LMGC90's quadratic norm ("Quad"): the root-mean-square change in the contact velocities between checks, relative to their size |
| Convergence check | Every 10 sweeps | Every 10 sweeps |  |
| Sweeps allowed per time step | 10,000 | 1,000 while the grains fall into place, 10,000 in the final phase | A step that reaches the limit moves on unconverged; the measured share is below |
| Relaxation | 1 (none) | 1 (none) |  |
| Contact order in a sweep | The same every sweep | The same every sweep | No random reordering |
| Contact detection | Every step; polyhedron pairs by Cundall's common-plane method, up to 5,000 iterations | Same |  |
| Contact search distance (alert distance) | The larger of 0.2 s and 2.5 × 1.25 v<sub>in</sub> Δt: 1 cm on bed 1; 0.4 cm on bed 3 | A tenth of the smallest semi-axis: 5 mm for boulders; 2.4 mm for stones | s is half the smallest long semi-axis r<sub>min</sub>: 5 cm on bed 1, 2 cm on beds 2 and 3 |
| Time step Δt | 0.02 s / (1.25 v<sub>in</sub>), kept between 0.2 and 5 ms at Dimorphos and between 0.1 and 2.5 ms at Didymos (table below) | 2 ms, never changed during a settle | Below the limits, a body at 1.25 v<sub>in</sub> moves 2% of s per step |
| Gravity | The body's (next section) | 9.81 m/s² | A cohesionless pile of rigid grains settles to the same shape at any gravity, only faster |

**Time step used, in ms,** by impact speed v<sub>in</sub> (cm/s):

| v<sub>in</sub> (cm/s) | Bed 1, Dimorphos | Bed 1, Didymos | Bed 3, Dimorphos | Bed 3, Didymos |
|---|---|---|---|---|
| 2 | 5 | – | 5 | – |
| 4 | 5 | 2.5 | 5 | 2.5 |
| 6 | 5 | – | 5 | – |
| 8 | 5 | 2.5 | 4 | 2.5 |
| 12 | – | 2.5 | – | 2.5 |
| 16 | 5 | 2.5 | 2 | 2 |
| 32 | – | 2.5 | – | 1 |

Bed 2 has the same s as bed 3, so its drops used the same time steps and search distance.

**How well the solver converged** (measured on every finished drop): sweeps per time step, and the share of time steps that reached the sweep limit without meeting the tolerance.

| Drops | Number | Sweeps per step, median drop | Steps at the limit, median drop | Steps at the limit, worst drop |
|---|---|---|---|---|
| Bed 1, Dimorphos | 720 | 64 | 0.3% | 10.2% |
| Bed 1, Didymos | 720 | 80 | 0.3% | 7.3% |
| Bed 2, Dimorphos | 107 | 115 | 0.3% | 2.1% |
| Bed 2, Didymos | 144 | 96 | 0.3% | 2.5% |
| Bed 3 (1.2 m patch), Dimorphos | 48 | 222 | 0.4% | 3.3% |
| Bed 3 (1.2 m patch), Didymos | 35 | 216 | 0.3% | 2.6% |
| Bed 3 (1.0 m patch), Dimorphos | 97 | 162 | 0.4% | 3.9% |
| Bed 3 (1.0 m patch), Didymos | 77 | 111 | 0.2% | 2.6% |

## Gravity and the two bodies

| Parameter | Dimorphos | Didymos | Note |
|---|---|---|---|
| Gravity g | 5 × 10<sup>−5</sup> m/s² | 2 × 10<sup>−4</sup> m/s² | Didymos's is 4 times Dimorphos's |
| Range of g over the surface | 4.5–5.1 × 10<sup>−5</sup> m/s² | 0.3–2.6 × 10<sup>−4</sup> m/s² | Dimorphos: Juventas's gravimeter (GRASS) abstracts and our own check. Didymos: our own calculation from its mass, size and 2.26 h spin; none is published |
| Escape speed near the surface | 3.3–6.5 cm/s over the surface (5 cm/s in the middle) | About 40 cm/s, provisional (44 cm/s without the spin, lower toward the equator) |  |
| Impact speeds v<sub>in</sub> | 2, 4, 6, 8 and 16 cm/s | 4, 8, 12, 16 and 32 cm/s | Didymos's are twice Dimorphos's, so every speed pairs at the same Froude number Fr = v<sub>in</sub>/√(gL) |
| Pour of each bed | First (seed 1) | Second (seed 2) | Each gravity drops onto its own pour of the same recipe |
| Clock factor | 1 | 0.5 | Every time in a drop's settings is multiplied by this and every speed limit divided by it, so drops at equal Fr are the same motion on a different clock |
| Time limit per drop | 300 s | 150 s |  |

## The craft: Milani with its solar panels open

| Parameter | Value | Basis |
|---|---|---|
| Mass | 12 kg: body 10.8 kg, each solar panel 0.6 kg | 12 kg: ESA. Panel mass assumed |
| Body | 13 × 24.6 × 36.6 cm along x, y and z | The 6U-XL envelope (ESA) |
| Solar panels | Two, each 59 × 36.6 × 1 cm, in the plane of the +x face, 2 cm from the body, tilted 0° | Two wings of three panels, as tall as the body (Tyvak's render). Length from Juventas's 1.42 m across the panels (Perez Lissi et al.). Thickness and gap assumed |
| Width across the panels | 1.466 m |  |
| Center of mass | 6 mm from the body's center along +x | The panels sit on the +x side |
| Moments of inertia about the center of mass | I<sub>xx</sub> = 0.4534, I<sub>yy</sub> = 0.1531, I<sub>zz</sub> = 0.3386 kg m² | x, y and z are principal axes |
| Farthest point from the center | 76 cm, a panel tip |  |
| Length L in Fr | 36.6 cm | The body's long edge |
| Shape in the simulation | Rigid convex blocks: the body, the two panels, and a block hidden inside the body that puts LMGC90's body center on the center of mass | No grain can reach the hidden block |
| Spin at impact | 0 |  |

## The beds

| Parameter | Bed 1 | Bed 2 (stopped November 10) | Bed 3 |
|---|---|---|---|
| Layers, bottom to top (poured thickness) | Boulders 70 cm | Stones 8 cm, then boulders 30 cm, then stones 14 cm | Stones 40 cm |
| Grain length (long axis) | Boulders 20–70 cm | Stones 8–16 cm; boulders 20–70 cm | Stones 8–16 cm |
| Size distribution | Boulders: Weibull in diameter, k = 0.39, λ = 0.033 m | Stones: uniform in length; boulders: Weibull in diameter, k = 0.39, λ = 0.033 m | Stones: uniform in length |
| Axis ratios a : b : c | Boulders 1 : 0.7 : 0.5 | Stones 1 : 0.8 : 0.6; boulders 1 : 0.7 : 0.5 | Stones 1 : 0.8 : 0.6 |
| Vertices per grain | 20 | 20 | 20 |
| Grain density | 3,400 kg/m³ | 3,400 kg/m³ | 3,400 kg/m³ |
| Footprint | 4 m × 4 m, walls 2 cm thick | 4 m × 4 m, walls 2 cm thick | 4 m × 4 m, walls 2 cm thick |
| Solid fraction used to size the pour | Boulders 0.55 | Stones 0.5; boulders 0.55 | Stones 0.5 |
| Pours | Seed 1 (Dimorphos), seed 2 (Didymos) | Seed 1 (Dimorphos), seed 2 (Didymos) | Seed 1 (Dimorphos), seed 2 (Didymos) |
| Grains, measured (each pour) | 852 and 859 | 4,246 and 4,244 | 12,113 and 12,148 |
| Mean depth after settling, measured (each pour) | 0.64 and 0.70 m | 0.42 and 0.41 m | 0.37 and 0.37 m |

Basis: boulder sizes and their mix follow the boulders counted on Dimorphos (Pajola et al. 2024); the axis ratio b/a = 0.7 is theirs, c/a = 0.5 the usual flattening of lying boulders. Grain density: 3,200–3,500 kg/m³ (Raducan et al. 2024). Stones of 8–16 cm are the smallest these machines can afford.

## Settling the beds

Time step, tolerance and gravity are in the solver table above (2 ms, 10<sup>−3</sup>, 9.81 m/s²).

| Parameter | Value | Note |
|---|---|---|
| Speed cap while the grains fall into place | 0.3 m/s on every grain | Keeps fast grains from passing through each other |
| At rest when | The fastest grain is below 2 mm/s, or the median grain moved less than 3 mm over the last second, 5 checks in a row, one every 100 steps (0.2 s) | Thick piles of polyhedra never stop jostling, so a still shape counts as rest |
| Or when the surface is still | Not used (bed 1); the mean height changes by less than 5 mm between measurements, 3 times running (beds 2 and 3) | Surface measured every 2,000 steps (4 s) |
| Release | The last 500 steps (1 s) without the speed cap |  |
| Steps | At least 1,000; at most 60,000 (bed 1); 40,000 (bed 2); 20,000 (bed 3) |  |
| State a drop starts from | Every grain at rest, where settling left it |  |

## Each drop

| Parameter | Value | Note |
|---|---|---|
| Free patch | Grains whose centers lie within R of the aim point across the surface and within 0.6 m below the local surface move freely. R = 1.2 m on beds 1 and 2 and bed 3's first 83 drops; 1.0 m on bed 3 from November 10 | On stones the disturbance dies away near 1 m |
| Frozen ring | The grains touching a free grain, held still | They hold the patch up |
| The rest of the bed | Left out of the drop | Used to check where the craft would land next |
| Patch edge to the wall | At least 0.5 m |  |
| Start | The craft 2 cm clear of the ground along its incoming path |  |
| Impact angle θ<sub>in</sub> from vertical | 0, 20, 40 and 60° |  |
| Azimuth | 0° | The incoming horizontal motion is along the bed's +x |
| Craft poses (roll, pitch, yaw in degrees, about the craft's axes) | upright (0, 0, 0): bottom edges of body and both panels (line); flat (0, 90, 0): solar face and both panels (area); corner (30, 30, 0): corner of one panel (point); back (0, −90, 0): back of the body, panels 13 cm clear (face) | One pose for each kind of first touch, in brackets |
| Landing spots | 9, on a 3 × 3 grid 30 cm apart around the bed's center; the center spot runs first |  |
| Repeats | 1 | A drop is deterministic |
| Drops per bed and gravity | 720 | 5 speeds × 4 angles × 4 poses × 9 spots |
| Recording | The craft every 0.05 s; the grains every 0.25 s for 30 s after the first touch, then every 2.5 s; movie frames every 0.1 s for 15 s, then every 1 s | The same at both gravities |

## When a drop ends

| Ends as | Dimorphos | Didymos | Note |
|---|---|---|---|
| In contact | Normal force on the craft above 10<sup>−12</sup> N | Same |  |
| Flew off | No contact for 0.5 s, rising, and the craft's exact free flight never touches the bed again (or first touches it outside the free patch); a flight passing within 1 cm of a free grain keeps the drop running | The same, after 0.25 s without contact | The flight is the craft's whole shape, turning, under gravity, against the bed as it is |
| Came to rest | Speed below 2 mm/s and spin below 0.01 rad/s for 10 s | Speed below 4 mm/s and spin below 0.02 rad/s for 5 s |  |
| Captured | The craft's fastest point stays below 0.1 v<sub>in</sub> for 10 s without flying off | The same, for 5 s | Fastest point: speed plus spin × the craft's 76 cm reach |
| Time limit | 300 s | 150 s |  |
| Never touched | No contact within 5 × 2 cm / v<sub>in</sub> + 2 s | Same |  |

## Sources

- Raducan et al. 2024: [Physical properties of asteroid Dimorphos as derived from the DART impact, Nature Astronomy 8, 445](https://arxiv.org/abs/2403.00667)
- Pajola et al. 2024: [Evidence for multi-fragmentation and mass shedding of boulders on rubble-pile binary asteroid system (65803) Didymos, Nature Communications 15, 6205](https://www.nature.com/articles/s41467-024-50148-9)
- Robin et al. 2024: [Mechanical properties of rubble pile asteroids (Dimorphos, Itokawa, Ryugu, and Bennu) through surface boulder morphological analysis, Nature Communications 15, 6203](https://www.nature.com/articles/s41467-024-50147-w)
- Thuillet et al. 2018: [Numerical modeling of lander interaction with a low-gravity asteroid regolith surface: application to MASCOT on board Hayabusa2, A&A 615, A41](https://www.aanda.org/10.1051/0004-6361/201832779)
- Brisset et al. 2020: [Regolith behavior under asteroid-level gravity conditions: low-velocity impacts into mm- and cm-sized grain targets](https://arxiv.org/abs/2008.08720)
- Perez Lissi et al.: Main challenges of CubeSat piggyback on an interplanetary mission: the Hera CubeSats (the craft's open solar panels and Juventas's span)
- Sánchez et al. 2021: [LMGC90: a contact dynamics open source code for the simulation of granular asteroid with realistic regolith shapes, EPJ Web of Conferences 249, 14007](https://www.epj-conferences.org/10.1051/epjconf/202124914007)
