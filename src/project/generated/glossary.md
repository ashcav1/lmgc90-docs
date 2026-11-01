## Terms of the Milani bounce study

_The same definitions as in the drop reel's Terms panel._

### 36-drop set

The same 36 landings on every study bed: 9 landing spots (a 3 x 3 grid 30 cm apart) x 2 arrival speeds (4 and 8 cm/s) x 2 angles (0 and 40 degrees from vertical), the craft always in the same pose.

### 500-step check

After a bed first comes to rest, 500 more steps with nothing changed; the bed must still be at rest at the end. If it blows apart instead, the state saved at the first rest is the bed, with a warning.

### angle from vertical

0 degrees: straight down. 40 degrees: arriving obliquely, moving sideways at 64 % of its speed.

### approach direction

The compass direction the craft comes from, in degrees from the bed's +x axis. (The code calls it the azimuth.)

### at rest

How a settle decides the grains have stopped: over the last simulated second the median grain moved less than 3 mm, 95 % moved less than 6 cm, none more than 30 cm. Piles of angular grains never stop jiggling completely, so 'at rest' means the pile's shape no longer changes.

### ballast

A massless box inside the craft's body (panels open). LMGC90 turns a body made of several pieces about the centre of their volumes, not of their masses; the ballast's volume moves that point onto the craft's true centre of mass.

### bed

Simulated ground: grains poured into a square box, left to settle under Earth's gravity, saved once, and reused by every drop on it. A bed's size is the side of its box: 4.5 m x 4.5 m for the study beds, 2-4 m for laptop tests.

### bed size

The side of a bed's box. A drop needs the patch plus 0.75 m on every side kept away from the walls (the largest boulder, 70 cm, plus 5 cm), where grains packed against the walls: side = 2 x (patch radius + 0.75 m) + the spread of the landing spots. With the panels open (1.2 m patch) that is at least 3.9 m; with them folded (0.7 m patch), 2.9 m.

### bottom layer

30 cm of small stones under the boulders (study bed 5): boulders resting on loose stones instead of on the box's rigid floor. (The code calls it the substrate.)

### boulders

Grains 20-70 cm long (30-70 cm in study bed 1). Boulders of this size cover most of Dimorphos.

### centre of mass

The point a free body turns about and whose path is a ballistic arc. With the panels open it lies 6 mm from the centre of the craft's body, toward the panels.

### conservative advancement

Moving a body forward in steps no longer than its distance to the nearest obstacle divided by the speed of its fastest point, so it cannot pass through anything between two tests.

### contact dynamics (NSCD)

LMGC90's method: rigid bodies whose collisions are solved as impulses within each time step, not as springs, which allows long time steps (5 ms in drops, 2 ms while settling). NSCD stands for non-smooth contact dynamics.

### contact rule

How LMGC90 treats two touching bodies; every rule includes friction. Position rule (GAP_SGR_CLB): at the end of each step no two bodies may overlap, and an overlap is pushed out; nothing bounces. Speed rule (RST_CLB): bodies stop moving into each other and bounce back with a set fraction of their approach speed (none while settling, half in drops); an overlap that already exists stays.

### contact solver (NLGS)

The part of LMGC90 that finds every contact force in a time step. It sweeps over the contacts until the forces stop changing (to a tolerance) or a budget of 10 000 sweeps runs out; a step that uses the whole budget is 'unconverged'. NLGS stands for non-linear Gauss-Seidel.

### convex

A shape without dents: the straight line between any two of its points stays inside it. Every grain and every piece of the craft is convex, which makes the exact distance between two of them a small, reliable calculation.

### Dimorphos

The 150 m moon of the asteroid Didymos, hit by NASA's DART spacecraft in 2022. Its surface is a pile of boulders, with smaller stones and finer material between and on them.

### Dimorphos gravity

5e-5 m/s^2, 200 000 times weaker than Earth's, used in every drop. A craft leaving the surface at 1 cm/s rises for 200 s before it falls back. (Also written micro-g.)

### drop

One simulated landing of the craft on one spot of one settled bed, from just before the first touch until the bounce is over. (The code calls it a run.)

### drop set

Drops run together from one file. (The code calls it a campaign.)

### energy kept

The craft's energy when it left, divided by its energy on arrival (motion plus spin, height counted from the first touch).

### even size mix

Every length between the smallest and the largest is equally likely (study bed 1: 30-70 cm). (The code calls it uniform.)

### fixed ring

The grains just outside the patch that touch it, held still so the patch keeps its support. The craft can hit them; they do not move. (The code calls them frozen.)

### free flight

The craft between touches or after its last one. Only gravity acts on it, and gravity cannot make it turn faster or slower, so its path and its tumbling can be computed exactly.

### friction

Friction coefficient 0.6 between grains (a grain slides on a slope steeper than 31 degrees) and 0.5 between the craft and a grain. Settling uses the same 0.6: higher friction made the contact solver fail (1 November).

### grain

One boulder or small stone. Every grain is a random convex polyhedron (a solid with flat faces): 12 corners for boulders, 10 for small stones, about 1 : 0.7 : 0.5 in length : width : thickness, flat and angular like the boulders DART imaged.

### grain size

Sizes on these pages are lengths: a grain's longest dimension. The bed files give half of it (r_min_m and r_max_m, the 'long semi-axis'). Measured on the settled beds, real grains come out about 10 % shorter than twice r, because only 10-12 corners outline each one.

### how a drop ends

Bounced off: left the ground, and its exact free flight comes back down on none of the grains the drop can move. Left the patch: slid or hopped out of the patch while on the ground, so the result is cut off. Captured: stayed down, creeping. Came to rest. Ran out of time: 300 s simulated.

### lab Mac

The 24-core Mac Studio in the lab that settles the study beds and runs their drop sets. The laptop runs small tests and the analyses on these pages.

### landing spot

Where the craft is aimed, in metres from the bed's centre (x, y).

### laptop test beds

Smaller or copied beds settled on the laptop to try things quickly: bed 1 itself (the same bed as the lab Mac's); bed 1's recipe at 4 m x 4 m; bed 2's boulders at 3 m x 3 m; 20-50 cm boulders, 0.5 m deep, with a thin top layer, at 2 m x 2 m; bed 1's boulders with a thin top layer at 3 m x 3 m.

### layers of a bed

A bed is poured in layers (boulders, then small stones), each settled before the next. A finished layer is reused by every bed that starts with the same layers. (The code calls them stages.)

### LMGC90

The open-source contact-dynamics code (University of Montpellier) that simulates every grain and the craft as rigid bodies colliding with friction, run through its Python interface (pylmgc90).

### Milani

ESA's 12 kg CubeSat, released by the Hera mission at the asteroid Didymos and its moon Dimorphos. Its body is a 13 x 25 x 37 cm box with two solar panels that fold out to 1.47 m across. It may land on Dimorphos at the end of its mission; these simulations ask how it would bounce.

### next touch

Where and when the craft would touch the bed again after a drop has ended, from its exact free flight. 'Off the patch': on the fixed ring or the rest of the bed, the landing of a new hop.

### panels open / folded

The craft's two shapes in the simulations, both 12 kg. Panels open: the body with both solar panels unfolded, 1.47 m across. Panels folded: the body alone, 37 cm long. (The code calls them deployed and stowed.)

### ParaView

The open-source 3-D viewer. LMGC90 writes frames for it (every 0.1 s for the first 15 s of a drop, then every second); the movies are rendered by ParaView itself.

### patch

A drop does not simulate the whole bed: only grains in a circle around the landing spot can move. That circle is the patch: radius 1.2 m with the panels open, 0.7 m with them folded on 3 m beds. (The code calls it the crop.)

### quality checks (QA)

Automatic checks on every bed and drop, graded PASS, WARN (look at it) or FAIL (not used). Beds: overlaps, escaped grains, contacts per grain, layer height. Drops: how it ended, that the first touch was on a grain that can move, no energy created, grain motion at the patch edge, the solver.

### reach

The largest distance from the craft's centre of mass to any point of the craft: 0.76 m with the panels open, 0.23 m folded. Whatever the craft's rotation, all of it lies within this distance.

### realistic size mix

Many small boulders and few large ones, in the proportions measured on DART's images of Dimorphos: the share of boulders longer than D falls like exp(-(D / 0.033 m)^0.39), a Weibull distribution (Pajola et al. 2024, fitted to boulders 1-16 m long). The beds keep only 20-70 cm, so most boulders are near 20 cm and a few are large. Study beds 2-6. (The code calls it weibull.)

### replay

Running a drop again from its saved setup. The simulation is deterministic, so the replay matches step for step (94 of 94 checked); movies and checks are made this way after the fact.

### rest of the bed

Everything beyond the fixed ring: left out of the drop, and treated as fixed ground by the checks made after it.

### restitution

The share of the velocity across a surface that survives a bounce (normal restitution), or along it (tangential).

### settle

Letting poured grains fall and stop. Beds are settled under Earth's gravity (9.81 m/s^2): a pile stops within seconds of simulated time instead of hours, and the drops then run at Dimorphos gravity. A bed is settled when it is at rest and still at rest after the 500-step check.

### small stones

Grains 8-16 cm long (10-20 cm in study bed 5's bottom layer). They stand in for the smaller material between and on Dimorphos's boulders. They are not fine-grained: sand or gravel would take millions of grains per square metre, far beyond what can be simulated, so no bed has fine-grained material. (The code calls them fines.)

### speed limit

While settling no grain may move faster than 0.3 m/s; without it piles blow apart.

### speed ratio e

Exit speed divided by arrival speed. 0: the craft stopped; 1: it left as fast as it came. The main bounce number.

### split contact rule

How beds with small stones are settled: the position rule between boulders and between a boulder and a small stone (so boulders do not sink into the stones), the speed rule between two small stones (with the position rule there they never come to rest on the boulders). Used since 30 October.

### start conditions

What is set before a drop: the bed, the landing spot, the arrival speed, the angle from vertical, the approach direction, the craft's shape and its pose.

### study beds 1-6

The six beds of the study, each one change from the one before; all 4.5 m x 4.5 m with 0.9 m of boulders unless stated, settled on the lab Mac. 1: boulders, even size mix (30-70 cm). 2: boulders, realistic size mix (20-70 cm). 3: bed 2, 1.5 m deep. 4: bed 2 plus a thin top layer of small stones. 5: a bottom layer of small stones, then bed 4. 6: bed 2 plus a thick top layer of small stones.

### thick top layer

25 cm of small stones on the boulders (study bed 6).

### thin top layer

Small stones poured on the boulders, enough for an 8 cm layer (study beds 4 and 5). The stones are 5-10 cm thick, so the layer is about one stone deep and the tops of the boulders stick out through it. (The code calls it the veneer.)

### touch

One period of contact between the craft and the ground, from the step the solver first puts a force on the craft to the step the force is gone. A drop can have several: the craft tips onto a panel, rocks, skips.
