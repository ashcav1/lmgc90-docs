# Parameters for Dimorphos-like regolith

The values below are what the DART results and the lander literature support
for a Dimorphos surface, and how they map onto LMGC90 inputs. The full
table with sources is kept with the pipeline
(`docs/DIMORPHOS_PARAMETERS.md`); the essentials:

| quantity | value | LMGC90 input |
|---|---|---|
| grain density | 3,200–3,500 kg/m³ (chondrite grains) | material density 3,400 |
| internal friction | 0.4–0.7, best 0.55; boulder shapes give θ ≈ 33° (μ 0.64) | `fric` 0.6 (settle and rock–rock) |
| surface cohesion | 0 to a few Pa; < ~50 Pa likely; Didymos tens of Pa | 0 baseline; `IQS_MOHR_DS_CLB` arms at 10 and 100 Pa |
| rock–rock restitution | 0.5 (standard SSDEM choice); 0.3–0.7 bracket | `rstn` 0.5 |
| lander–regolith restitution | MASCOT contact CoR 0.56–0.98; frictional beds 0.3–0.35 | craft–rock `rstn` 0.5, to be calibrated |
| boulders | 4,734 per 0.0132 km² on Dimorphos; median 1.1 m; Weibull k ≈ 0.4 below 5 m, power law −2.5 above; b/a ≈ 0.7 | size distribution and 1:0.7:0.5 axes of the polyhedra |
| surface macroporosity | ≈ 35 % | emerges from gravity settling (~0.6 solid fraction) |
| boulder volume fraction | ≲ 40 vol % (> 2.5 m) | small-stone layers between / on boulders |

Sources: Raducan et al. 2024 (Nature Astronomy), Pajola et al. 2024 (Nature
Communications), Barnouin et al. 2024, the 2024 boulder-morphology study
(Nature Communications), Sánchez & Scheeres 2014, Thuillet et al. 2018 (A&A),
Brisset et al. 2020.
