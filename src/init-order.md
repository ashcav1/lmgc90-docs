# Quick Reference: Initialization Order

```python
# 1. Setup
chipy.Initialize()
chipy.checkDirectories()
chipy.utilities_DisableLogMes()

# 2. Configuration
chipy.SetDimension(3, 0)
chipy.TimeEvolution_SetTimeStep(1e-3)
chipy.Integrator_InitTheta(0.5)

# 3. Load geometry
# [patch BULK_BEHAV.DAT gravity here if needed]
chipy.ReadDatbox(deformable=False)

# 4. Output setup
chipy.InitHDF5("output.h5")

# 5. Contact detection setup (after ReadDatbox, before OpenDisplayFiles)
if hasattr(chipy, "PRPRx_SetCpCundallBox"):
    chipy.PRPRx_SetCpCundallBox(-60., 60., -60., 60., -20., 80.)
chipy.PRPRx_UseCpCundallDetection(200000)
chipy.PRPRx_LowSizeArrayPolyr(100)

# 6. Restart (impact script only)
chipy.ReadIni(record=-1, h5_file="poly_settle.h5")

# 7. Display and postpro
chipy.OpenDisplayFiles(start_frame)
chipy.OpenPostproFiles()
chipy.ComputeMass()

# 8. One-time velocity init (impact script only)
# [set impactor velocity here via RBDY3_PutBodyVector]

# 9. Time loop
for k in range(nb_steps):
    chipy.IncrementStep()
    chipy.ComputeFext()
    chipy.ComputeBulk()
    chipy.ComputeFreeVelocity()
    chipy.SelectProxTactors()
    chipy.RecupRloc(5e-2)
    chipy.ExSolver("Stored_Delassus_Loops         ", "Quad ", 1e-4, 1.0, 50, 1000)
    chipy.UpdateTactBehav()
    chipy.StockRloc()
    chipy.ComputeDof()
    chipy.UpdateStep()
    chipy.WriteOut(500)
    chipy.WriteDisplayFiles(50)
    chipy.WritePostproFiles()
    if k % 50 == 0:
        chipy.WriteLast("last.h5")

# 10. Teardown
chipy.WriteLast("last.h5")
chipy.CloseDisplayFiles()
chipy.ClosePostproFiles()
chipy.Finalize()
```

---
