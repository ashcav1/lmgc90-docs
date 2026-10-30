# Summary

[Introduction](introduction.md)

# Getting started

- [Install, build, patch](getting-started.md)
- [A first simulation, line by line](first-script.md)
- [How a run is organised](concepts.md)

# Building a deck (pylmgc90.pre)

- [Containers: avatars, materials, models, laws, see tables](pre-containers.md)
- [Materials and models](pre-materials.md)
- [Bodies: planes and polyhedra](pre-bodies.md)
- [Body methods: translate, rotate, drive](pre-body-methods.md)
- [Bed generation (and why we replaced it)](pre-beds.md)
- [Contact laws and see tables](pre-contacts.md)
- [Writing the DATBOX](pre-output.md)
- [BULK_BEHAV.DAT anatomy](bulk-behav.md)

# Running (pylmgc90.chipy)

- [Initialisation order](init-order.md)
- [Initialisation and teardown](runtime-init.md)
- [Configuration: dimension, time step, theta](runtime-config.md)
- [I/O setup: DATBOX, HDF5, display, postpro](runtime-io.md)
- [The time step, call by call](runtime-step.md)
- [Output functions and restarts](runtime-output.md)
- [Bodies: getters and setters](runtime-bodies.md)
- [Polyhedron contact detection](detection.md)
- [Detection API](runtime-detection-api.md)

# Physics as measured

- [Contact laws: what each one really does](contact-laws.md)
- [Settling a granular bed](settling.md)
- [Impacts on a settled bed](impacts.md)
- [Post-processing: files, units, probes](postpro.md)
- [Parameters for Dimorphos-like regolith](parameters.md)
- [A pipeline on top: the Milani bounce study](pipeline-api.md)

# When it goes wrong

- [Failure modes catalogue](failure-modes.md)
- [Crash & divergence field guide](crash-guide.md)
- [FAQ: priority questions answered](faq.md)
