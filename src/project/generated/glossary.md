## Terms of the Milani bounce study

_The same definitions as in the drop reel's Terms panel._

### angle from vertical

0 degrees: straight down; 40 degrees: arriving obliquely, moving sideways at 64 % of its speed.

### azimuth

The compass direction of the approach, in degrees from the bed's +x axis.

### ballast

A massless box inside the deployed craft's bus. LMGC90 turns a body made of several pieces about the centre of their volumes, not of their masses; the ballast's volume moves that point onto the craft's true centre of mass.

### bed

A patch of simulated terrain: grains dropped into a square box and left to settle under gravity, saved once, then reused by every drop on it. Its 'footprint' is the box's side (4.5 m for production beds).

### bed 1-6

The six beds of the study, each one change from the last: 1 uniform boulders; 2 Weibull boulders; 3 deeper (1.5 m); 4 Weibull boulders + 8 cm fines veneer; 5 fines substrate + boulders + veneer; 6 boulders + 25 cm of fines. 4.5 m square, settled on the lab Mac.

### boulders

The large grains: long semi-axis 0.10-0.35 m (0.2-0.7 m across). On Dimorphos they cover most of the surface.

### centre of mass

The point a free body turns about and whose path is a ballistic arc. For the deployed craft it lies 6 mm from the bus centre toward the wings.

### confirmation window

500 more settle steps after the first rest with nothing changed, to show the bed stays at rest. If it blows up anyway, the state saved at the first rest is the bed (card: rest_window WARN).

### conservative advancement

Moving a body forward in steps no longer than its distance to the nearest obstacle divided by the speed of its fastest point, so it cannot pass through anything between two tests.

### contact law

The rule LMGC90 applies at each pair of touching bodies. All use Coulomb friction (friction 0.6 while settling). The gap law (GAP_SGR_CLB) pushes overlapping bodies apart; the velocity law (RST_CLB) only stops them moving into each other and gives a bounce restitution.

### convex

A shape without dents: the straight line between any two of its points stays inside it. Every grain and every piece of the craft is convex, which makes the exact distance between two of them a small, reliable calculation.

### crop

A drop does not simulate the whole bed: it cuts a circle (the free crop, radius 1.2 m for the deployed craft) around the landing spot whose grains can move, plus the ring of grains touching it held fixed (the frozen ring). Grains beyond are left out.

### deployed / stowed

The craft's two shapes in the simulations. Deployed: bus plus both wings unfolded (1.47 m span, reach 0.76 m from its centre of mass). Stowed: the bare bus (reach 0.23 m). Same mass, 12 kg.

### Dimorphos

The 150 m moon of the asteroid Didymos, hit by NASA's DART in 2022. Its surface is a rubble pile of boulders with finer material between and on them. Surface gravity about 5e-5 m/s^2, 200 000 times weaker than Earth's.

### drop

One simulated landing of the craft on one spot of one settled bed, from just before the first touch until the bounce is over. Also called a run.

### drop set

A list of drops run together from one config (a campaign), e.g. the standard 36 drops on one bed.

### energy kept

Mechanical energy of the craft when it left, divided by its energy on arrival (kinetic + rotational, potential counted from the first-contact height).

### fines

The small grains between and on the boulders: long semi-axis 4-10 cm. On Dimorphos this is everything below DART's ~3 cm image resolution, so how much there is, and where, is an assumption the beds vary.

### free flight

The craft between touches or after its last one. Only gravity acts on it, and gravity exerts no torque, so its path and its tumbling can be computed exactly into the future.

### frozen ring

The grains just outside the free crop, held still so the crop does not spill out. They are part of the terrain the craft can hit, but they cannot move.

### lab Mac

The 24-core Mac Studio in the lab that runs the production beds and drop sets. The laptop runs only small tests and the analyses on these pages.

### landing spot

Where the craft is aimed, in metres from the bed's centre (x, y).

### LMGC90

The open-source contact-dynamics code (Montpellier) that simulates every grain and the craft as rigid bodies colliding with friction. Everything here runs through its Python interface (pylmgc90).

### micro-g

Short for the tiny surface gravity of Dimorphos (5e-5 m/s^2) used in every drop. A craft leaving the surface at 1 cm/s rises for 200 s before falling back.

### Milani

The 12 kg CubeSat (6U) that ESA's Hera mission releases at the Didymos system. Its bus is a 13 x 25 x 37 cm box; with its two solar wings unfolded it is 1.47 m across. At the end of its mission it may touch down on Dimorphos: these simulations ask how it bounces.

### next touch

Where and when the craft would touch the bed again after a drop has ended, from its exact free flight. 'Off the patch': on the frozen ring or the bed beyond the free crop, the landing of a new hop.

### NLGS

Non-linear Gauss-Seidel, the iterative solver LMGC90 uses each time step to find all contact forces at once. It sweeps the contacts until the forces stop changing (to a tolerance) or a budget of sweeps runs out ('solver steps at budget').

### NSCD

Non-smooth contact dynamics, LMGC90's method: rigid bodies, contacts that either transmit force or open, and collisions solved as impulses within each time step instead of as springs. It allows large time steps (5 ms for drops, 2 ms for settling).

### outcome

How a drop ended: bounced off (left the surface; its exact free flight comes back down on none of the grains the simulation moves), left the patch (slid or hopped out of the free crop on the ground: the result is cut off), captured (stayed down, creeping), came to rest, or time limit (300 s).

### ParaView

The open-source 3-D viewer; LMGC90 writes frames for it (every 0.1 s for the first 15 s of a drop, then every second). The movies are rendered by ParaView itself.

### polyhedral grain

Every boulder and fines grain is a random convex polyhedron (12 vertices for boulders, 10 for fines) with axis ratios about 1 : 0.7 : 0.5 — flat, angular, like the boulders DART imaged.

### QA

Automatic checks on every bed and every drop: PASS, WARN (look at it) or FAIL (never used). For beds: overlaps, escaped grains, contacts per grain, layer height. For drops: outcome, first contact on free grains, energy not created, crop disturbance, solver.

### reach

The largest distance from the craft's centre of mass to any point of the craft (0.76 m deployed, 0.23 m stowed). Whatever the craft's rotation, all of it is within this sphere.

### replay

Running a drop again from its saved job. The simulation is deterministic: the replay is identical, step for step (94 of 94 checked), which lets movies and checks be made after the fact.

### restitution

How much of the velocity component normal to a surface survives a bounce (normal restitution), or along it (tangential).

### settle

Letting the dropped grains fall and stop. A bed is settled when its shape has stopped changing (structural rest), checked once more in a confirmation window, then saved.

### speed cap

While settling, no grain may move faster than 0.3 m/s. It keeps the deposit calm and the pile stable; until 2026-10-30 a bug removed it at the first rest, and beds blew apart.

### speed ratio e

Exit speed divided by arrival speed. 0: the craft stopped; 1: it left as fast as it came. The main bounce number.

### split law

Settling beds with fines: the gap law between boulders and fines (no boulder sinks into the fines), the velocity law among fines (the gap law there never comes to rest over boulders). Used for beds 4-6 since 2026-10-30.

### stage

One layer of a bed being deposited and settled; a bed with fines on boulders has two stages. A finished stage is reused by any bed that shares those first layers.

### standard 36 drops

The same 36 landings fired at every bed: 9 landing spots x 2 speeds (4 and 8 cm/s) x 2 angles (0 and 40 degrees from vertical), craft in the same fixed pose.

### start conditions

What is set before a drop: the landing spot, the arrival speed, the angle from vertical, the approach azimuth, the craft's pose (attitude) and shape, and the bed.

### structural rest

The test that ends a settle: over the last second of simulated time the median grain moved less than 3 mm, 95 % less than 6 cm and none more than 30 cm. Thick piles of polyhedra never stop jiggling completely, so 'at rest' means the shape no longer changes, not that every grain is still.

### substrate

A layer of fines under the boulders (30 cm in bed 5): boulders resting on soft ground instead of a rigid floor.

### touch

One period of contact between the craft and the terrain: from the step the solver first puts a force on the craft until the step the force is gone. A drop can have several touches (the craft tips onto a wing, rocks, skips).

### uniform size distribution

Every grain size between the smallest and the largest is equally likely (bed 1: long semi-axis 0.15-0.35 m).

### veneer

A thin layer of fines poured on top of the boulders (8 cm in beds 4 and 5).

### Weibull size distribution

A size distribution where small boulders are many and large ones rare. The share of boulders larger than diameter D falls like exp(-(D/lambda)^k); with the values fitted to DART's images of Dimorphos (Pajola et al. 2024: k = 0.39, lambda = 0.033 m) the tail is heavy. The beds keep only 0.2-0.7 m diameters (long semi-axis 0.10-0.35 m), so within that range most boulders are near the small end and a few are large. Beds 2-6 use it.
