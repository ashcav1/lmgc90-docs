# Priority Questions — Summary Answers

| Question | Answer |
|----------|--------|
| ExSolver `stype` strings | Must be **exactly 30 chars** with trailing spaces: `"Stored_Delassus_Loops         "` or `"Exchange Local Global         "` |
| ExSolver `norm` strings | Must be **exactly 5 chars**: `"Quad "`, `"Maxm "`, `"QM/16"` |
| `gsit1` vs `gsit2` roles | `gsit1` = outer loop (max convergence checks); `gsit2` = inner loop (NLGS sweeps per check) |
| `RecupRloc(tol)` | `tol` triggers position-based recovery for CSASp/CSPRx; for PRPRx use `PRPRx_SetTolRecupRloc` separately |
| `ReadIni(record=-1)` | Reads **last** record from HDF5; `record=0` reads ASCII .INI files |
| `WriteLast(filename)` | Writes to specified HDF5 file if `InitHDF5` was called; use `"last.h5"` for crash recovery |
| `RBDY3_PutBodyVector` body_id | **1-based** index; impactor = `RBDY3_GetNbRBDY3()` (last created body) |
| Why set V____, Vbeg_, Vfree? | Theta integrator uses beginning (`Vbeg_`) and end (`V____`) of step; solver uses `Vfree` as RHS |
| `PRPRx_SetCpCundallBox` existence | **Not in current SWIG wrapper**; use `hasattr()` check; `PRPRx_SetDomain` is alternate name |
| `rigidPolyhedron("random")` vertex count | Exact — `nb_vertices` vertices are placed on the circumscribed sphere |
| `depositInBox3D` box origin | Box spans `[0, lx] × [0, ly] × [0, lz]`; **must translate** to centre in simulation coordinates |
| `imposeDrivenDof` reversibility | **Irreversible in pre** — write DROP datbox (impactor free) BEFORE calling this |
| `gravy` in `writeDatbox` | Written as `grv1`, `grv2`, `grv3` in `BULK_BEHAV.DAT`; all three components exist |
| `BULK_BEHAV.DAT` gravity field | `grv3=` for vertical; `grv1=`, `grv2=` also present for horizontal components |
| `tact_behav` name length | Exactly **5 characters** |
| `material` name length | Exactly **5 characters** |
| `model` name length | Exactly **5 characters** |
