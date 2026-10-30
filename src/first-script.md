# A first simulation, line by line

A cube dropped on a plane. Two files: a deck builder (pre) and a driver
(chipy). Everything else in this book is a variation of this.

```python
# build.py — writes DATBOX/
from pylmgc90 import pre
import numpy as np

mats, mods = pre.materials(), pre.models()
rock = pre.material(name="ROCKx", materialType="RIGID", density=3000.0)
wall = pre.material(name="TDURx", materialType="RIGID", density=1000.0)
mats.addMaterial(rock, wall)
mod = pre.model(name="rigid", physics="MECAx", element="Rxx3D", dimension=3)
mods.addModel(mod)

bodies = pre.avatars()
floor = pre.rigidPlan(axe1=2.0, axe2=2.0, axe3=0.01, center=[0, 0, -0.01],
                      material=wall, model=mod, color="FLOOR")
floor.imposeDrivenDof(component=[1, 2, 3, 4, 5, 6], dofty="vlocy")   # fixed
bodies += floor

h = 0.1
v = np.array([[sx, sy, sz] for sx in (-h, h) for sy in (-h, h) for sz in (-h, h)])
cube = pre.rigidPolyhedron(model=mod, material=rock, color="BOULD",
                           generation_type="vertices", vertices=v + [0, 0, 0.4])
bodies += cube

tacts = pre.tact_behavs()
law = pre.tact_behav(name="rockx", law="RST_CLB", fric=0.5, rstn=0.5, rstt=0.0)
tacts += law
svs = pre.see_tables()
svs += pre.see_table(CorpsCandidat="RBDY3", candidat="POLYR", colorCandidat="BOULD",
                     CorpsAntagoniste="RBDY3", antagoniste="PLANx", colorAntagoniste="FLOOR",
                     behav=law, alert=0.01)

post = pre.postpro_commands()
pre.writeDatbox(3, mats, mods, bodies, tacts, svs, post=post, gravy=[0, 0, -9.81],
                datbox_path="DATBOX")
```

```python
# run.py — drives the kernel
from pylmgc90 import chipy
import numpy as np

chipy.Initialize()
chipy.checkDirectories()
chipy.utilities_DisableLogMes()      # warnings go to the void: see the crash guide
chipy.SetDimension(3, 0)
chipy.TimeEvolution_SetTimeStep(1e-4)
chipy.Integrator_InitTheta(0.5)
chipy.ReadDatbox(deformable=False)
chipy.OpenDisplayFiles()
chipy.OpenPostproFiles()
chipy.PRPRx_UseCpCundallDetection(5000)   # polyhedra need a detection method
chipy.PRPRx_LowSizeArrayPolyr(10)
chipy.ComputeMass()

for k in range(5000):
    chipy.IncrementStep()
    chipy.ComputeFext()
    chipy.ComputeBulk()
    chipy.ComputeFreeVelocity()
    chipy.SelectProxTactors()
    chipy.RecupRloc()
    chipy.ExSolver("Stored_Delassus_Loops         ", "Quad ", 1.666e-4, 1.0, 10, 1000)
    chipy.UpdateTactBehav()
    chipy.StockRloc()
    chipy.ComputeDof()
    chipy.UpdateStep()
    chipy.WritePostproFiles()
    if k % 100 == 0:
        v = chipy.RBDY3_GetBodyVector("V____", 2)      # body 2 = the cube; 1-based
        print(k, np.round(v[:3], 4))

chipy.CloseDisplayFiles()
chipy.ClosePostproFiles()
chipy.Finalize()
```

Things to notice:

- Body ids are 1-based in the order the avatars were added; the floor is 1,
  the cube 2.
- `imposeDrivenDof(..., dofty="vlocy")` with no values makes a body fixed.
- Without the `see_table` row the cube falls through the floor and nothing
  complains. Every pair that must interact needs a row.
- `ExSolver`'s last two arguments are *sweeps between convergence checks*
  and *maximum checks*: 10 × 1000 = up to 10,000 Gauss–Seidel iterations.
- `utilities_DisableLogMes` also hides the "NLGS did not converge" warning:
  `POSTPRO/SOLVER_INFORMATIONS.DAT` is the only witness.
