# Introduction

LMGC90 is an open-source contact-dynamics code (Non-Smooth Contact Dynamics,
NSCD) developed at LMGC, Université de Montpellier. It integrates collections
of rigid or deformable bodies with unilateral contact and Coulomb friction
through an implicit, velocity-level formulation solved by a Gauss–Seidel
sweep over the contacts (NLGS). Its Python layer, `pylmgc90`, has two halves:
`pre` builds the input deck (`DATBOX/`) and `chipy` drives the Fortran kernel
step by step.

This book is the guide we wished we had when we started simulating a 12 kg
CubeSat landing on a bed of polyhedral boulders at Dimorphos gravity. It
covers the API as it is actually used in such a pipeline, the solver's
behaviour as measured on real decks, and — at length — the ways a run can
fail, because most failures in LMGC90 are silent: a wrong see table, a
contact law that never bonds, a detection mode that finds no contacts, a
settling pile that looks at rest and is not.

What it is not: the official manual (the LMGC90 user repository carries the
reference documentation and examples), a tutorial on NSCD theory, or a
description of deformable-body (FEM) features, which we do not use.

Conventions: all code is Python 3 against `lmgc90_user_2025.rc1`; function
names are the `chipy`/`pre` names; units are SI; "we measured" means a
dated experiment recorded in the pipeline's decision log.
