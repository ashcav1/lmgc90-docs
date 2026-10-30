# Glossary (plain words)

| term | meaning |
|---|---|
| **body** | one rigid object with mass and inertia (`RBDY3`) |
| **contactor** | the shape attached to a body that can touch things: `POLYR` (convex polyhedron), `PLANx` (a thick plane), `SPHER`, … |
| **color** | a 5-character tag on a contactor; contact rules are written per pair of colors |
| **see table** | the list of (candidate color, antagonist color, law, alert distance) pairs allowed to touch; no row = no contact, no error |
| **alert distance** | how close two contactors must be to become a candidate contact |
| **contact law** | the rule for a touching pair. The **gap rule** (`GAP_SGR_CLB`) pushes overlapping grains apart; the **velocity rule** (`RST_CLB`) only stops grains moving into each other and never pushes an overlap apart; **cohesion** (`IQS_MOHR_DS_CLB`) glues touching polyhedra until a traction breaks the bond |
| **NLGS** | the Gauss–Seidel sweep over all contacts that solves the contact forces each step; its tolerance and iteration budget are set in `ExSolver` |
| **settling** | dropping grains into a box and letting them stop; the pile at its first rest is the asset to keep |
| **rest test** | how "stopped" is decided: every grain below a speed, or the geometry no longer changing (median displacement over the last second below a limit) |
| **speed cap** | `RBDY3_PartialDamping`: clips any grain faster than a limit at the start of a step; inert at rest; never lift it during a settle |
| **fines** | the small grains (gravel/sand) between and on the boulders; small polyhedra in the simulations |
| **crop / frozen shell** | the part of a settled bed an impact simulates (free grains around the landing spot) and the ring of grains held fixed around it |
| **QA** | the automatic checks on a bed or run — overlaps, escaped grains, contacts per grain, layer height, crop-edge disturbance; PASS / WARN / FAIL |
| **restitution (e)** | speed out ÷ speed in of the body that bounced |
