# How each number is computed

Every number in the [drop reel](../drops/index.html) and the run log, defined: what it is, why it can be trusted (or where it cannot), and the code that computes it. File and line refer to the project's code repository at the time this page was generated (2026-10-31 19:48); the excerpts are copied from that code, not paraphrased.

## In contact (a touch)

**Definition.** At each 5 ms time step the craft is in contact when the resultant contact force the solver applied to it is non-zero: |R| > 1e-12 N.

**Why it can be trusted.** This is the solver's own statement that some contact transmitted force in that step. Bodies that are merely close (within the detection distance) carry zero force and do not count. Non-smooth contact dynamics resolves even an instantaneous collision as an impulse inside its time step, so no touch can fall between two steps.

**Limits.** A resultant can in principle cancel when the craft is squeezed equally from opposite sides; the contact count column of the trajectory (n_box_contacts) would show such a case, and none has been seen.

`src/milani/impact.py`, line 211:

```python
            fn = float(np.linalg.norm(F[:3]))
            in_contact = fn > dc.contact_eps
```

`src/milani/config.py`, line 415:

```python
    contact_eps: float = 1.0e-12           # N; |reaction| above this = in contact
```

## First contact and arrival velocity

**Definition.** The first step in contact. The arrival velocity is the craft's velocity one step before it, the last free-flight state.

**Why it can be trusted.** Taking the state before the touch keeps the arrival velocity free of the impact itself.

`src/milani/impact.py`, line 227:

```python
            if in_contact and first is None:
                pv, pw, pwb = (prev if prev is not None else (v, w_g, v6[3:6]))
                v_in, w_in, wb_in = pv.copy(), pw.copy(), pwb.copy()
                I = s.contacts()
                geo = _contact_geometry(_craft_contacts(I, craft), craft, x, M,
                                        craft_vertices(), piece_of)
                first = {"t": k * dt, "x": x.copy(), "geo": geo}
            if in_contact and k % 5 == 0:
```

## Touches (contact episodes)

**Definition.** How many times the craft goes from free to in contact during the drop.

`src/milani/detectors.py`, line 79:

```python
            if not s._in_contact:
                s.episodes += 1
            s.last_contact_t = s.t
```

## Last contact

**Definition.** The last step at which the craft was in contact before the drop ended.

**Why it can be trusted.** Within the simulated time this is exact (every step is tested). Whether a later touch would have come is what the detachment test and the replay check below settle.

`src/milani/detectors.py`, line 81:

```python
            s.last_contact_t = s.t
```

## Exit state (exit speed, spin, elevation)

**Definition.** The craft's velocity and spin at the step its contact ended for the last time. Each separation overwrites it, so at the end it is the last one. Exit elevation is the angle of the exit velocity above the horizontal of the bed (not of the local surface).

`src/milani/impact.py`, line 253:

```python
            if was_in_contact and not in_contact:
                exit_state = {"t": k * dt, "x": x.copy(), "v": v.copy(), "w_g": w_g.copy(),
                              "w_b": v6[3:6].copy(), "M": M.copy()}
```

## Bounced off: when a drop is over

**Definition.** A drop ends as 'bounced off' when the craft has been free for 0.5 s, is rising, and EITHER no point of it can reach the terrain anywhere along its path over the free crop, OR it flies out over the crop's edge. The first test: a sphere of the craft's reach around its centre of mass, moving on the exact ballistic arc (no force acts on a free craft but gravity), must stay above the bed's top surface — its 1 cm height map, raised by the largest grain displacement so far and a 1 cm margin — until the centre leaves the crop.

**Why it can be trusted.** Every vertex of the craft is inside that sphere whatever the rotation, so the test is a proof that no touch can happen over the crop; it does not have to predict the tumbling. The test it replaced (2026-10-31) projected the rotating craft 3-30 s ahead; a free craft's spin axis wobbles, and on bed 1 a drop declared over at 1.3 s grazed a boulder with a wing tip at 20.5 s. Every drop since has been re-simulated with the new test.

**Limits.** Beyond the crop edge nothing is simulated: a craft that flies out of the crop low could touch the terrain just outside it. Grains still creeping after the test (mm/s) are covered only by the replay check.

`src/milani/detectors.py`, line 192:

```python
def reach_clear(centre, v, g: float, reach: float, hm, lift: float, margin: float,
                target_xy, crop_r: float, t_max: float, step_s: float = 0.25) -> bool:
    """True when NO point of the craft can touch the terrain while it is over
    the free crop: a sphere of radius `reach` (the craft's circumradius about
    its centre of mass — every vertex is inside it whatever the rotation)
    centred on the centre of mass, which moves on an exact ballistic arc
    once the craft is free, stays `margin` + `lift` above the bed's top
    surface (height map `hm`: z grid, x0, y0, dx, floor_z) from now until the
    centre leaves the crop circle (radius `crop_r` about `target_xy`) or
    until `t_max`.

    Why not predict the rotation (2026-10-31): the spinning look-ahead
    (`terrain_clear`, `ballistic_clear` with `w`) assumed a fixed spin axis;
    a free asymmetric body's axis wobbles (torque-free precession), and on
    bed 1 a drop declared detached at 1.3 s grazed a boulder with a wing tip
    at 20.5 s — inside the 30 s horizon. This test needs no rotation model:
    it is a proof, up to the height map's 1 cm resolution and grain motion
    after now (`lift` = the largest grain displacement so far; the
    `lab.py check-last-contact` replay tests what is left)."""
    import math
    import numpy as np
    c = np.asarray(centre, dtype=float)
    vv = np.asarray(v, dtype=float)
    tx, ty = float(target_xy[0]), float(target_xy[1])
    # time for the centre to leave the crop circle (horizontal motion is uniform)
    px, py = c[0] - tx, c[1] - ty
    a = vv[0] ** 2 + vv[1] ** 2
    if a < 1e-14:
        T = t_max
    else:
        b = 2.0 * (px * vv[0] + py * vv[1])
```

`src/milani/impact.py`, line 265:

```python
            def clear_ahead():
                # detached only when no point of the craft can reach the terrain
                # while it is over the free crop (detectors.reach_clear: a sphere
                # of the craft's circumradius on the exact ballistic arc of its
                # centre of mass, against the bed's height map raised by the
                # largest grain displacement so far)
                if hm is None:
                    return False
                if k - clear_memo["k"] < clear_every:
                    return clear_memo["ok"]
                Call = s.vectors("Coor_")[:, :3]
                lift = float(np.linalg.norm(Call[free] - C0[free], axis=1).max()) if len(free) else 0.0
                ok = reach_clear(x, v, g, reach, hm, lift, dc.detach_margin_m, target,
                                 st.crop_radius_m, st.max_sim_time_s - k * dt)
                clear_memo.update(k=k, ok=ok)
                return ok

            outcome_kind = det.update(
                in_contact, float(np.linalg.norm(v)), float(np.linalg.norm(v6[3:6])),
```

## Last-contact check (replay)

**Definition.** Each bounced-off drop is replayed from its saved job with the detachment test switched off, and the replay continues until the craft leaves the crop, stops, or 150 s past the original end. Up to the original end the replay is the original run (deterministic); any touch after it shows up as a later last contact or one more touch.

**Why it can be trusted.** This tests the proof's two loose ends (height-map resolution, grains moving later) directly, drop by drop. Verdicts: confirmed; touched again; left the patch at detachment (nothing beyond the crop can be checked).

`src/milani/checks.py`, line 37:

```python
def check_last_contact(run_dir: Path, extra_s: float = 150.0, keep: bool = False) -> dict:
    from . import pipeline
    run_dir = Path(run_dir)
    rec = json.loads((run_dir / "outcome.json").read_text())
    ct = rec.get("contact") or {}
    out = {"run_id": rec["run_id"], "t_last_contact": ct.get("last_t"),
           "t_detached": rec.get("sim_time_s"), "episodes": ct.get("episodes"),
           "detached_how": ct.get("detached_how")}
    if rec["outcome"] != "detached":
        out["verdict"] = "not applicable"
    elif str(ct.get("detached_how") or "").startswith("patch_edge"):
        out["verdict"] = "left the patch at detachment"
    else:
        job = json.loads((run_dir / "job.json").read_text())
        job["detector"] = dict(job.get("detector") or {}, detach_min_time_s=NEVER)
        job["impact"] = dict(job["impact"], max_sim_time_s=min(
            float(job["impact"].get("max_sim_time_s", 300.0)), float(rec["sim_time_s"]) + extra_s))
        tmp = run_dir / "_lastcheck"
        if tmp.exists():
            shutil.rmtree(tmp)
        try:
            for phase, stall in (("compose", 900), ("impact", 3600)):
                rc = pipeline.run_phase(phase, tmp, job, stall_seconds=stall)
                if rc != 0:
                    out["verdict"] = f"replay failed ({phase} exit {rc})"
                    break
            else:
                new = json.loads((tmp / "outcome.json").read_text())
                nct = new.get("contact") or {}
                same_start = (abs(float(nct.get("first_t") or -1) - float(ct.get("first_t") or -2)) < 1e-9)
                later = (nct.get("last_t") or 0) > (ct.get("last_t") or 0) + 1e-9
                more = (nct.get("episodes") or 0) > (ct.get("episodes") or 0)
                ex0, ex1 = rec.get("exit") or {}, new.get("exit") or {}
                z0 = (ex0.get("position_rel_target") or [0, 0, 0])[2]
                z1 = (ex1.get("position_rel_target") or [0, 0, 0])[2]
                out.update({
                    "replay_outcome": new["outcome"], "replay_end_t": new["sim_time_s"],
                    "replay_last_t": nct.get("last_t"), "replay_episodes": nct.get("episodes"),
                    "replay_same_first_contact": same_start,
                    "extra_free_flight_s": round(float(new["sim_time_s"]) - float(rec["sim_time_s"]), 3),
                    "height_gain_m": round(float(z1) - float(z0), 4),
```

## Speed ratio e

**Definition.** e = |v_out| / |v_in|: exit speed over arrival speed.

`src/milani/outcome.py`, line 109:

```python
        rs["e_speed"] = _ratio(np.linalg.norm(v_out), np.linalg.norm(vi))
        rs["e_n_flat"] = _ratio(v_out[2], -vi[2])
        rs["e_t_flat"] = _ratio(np.linalg.norm(v_out[:2]), np.linalg.norm(vi[:2]))
```

## Normal and tangential restitution (flat frame)

**Definition.** Relative to the bed's horizontal: e_n = v_out,z / (-v_in,z) and e_t = |v_out,horizontal| / |v_in,horizontal|.

**Limits.** The bed's horizontal is not the surface the craft hit: on a 40-degree boulder face these mix normal and tangential.

`src/milani/outcome.py`, line 110:

```python
        rs["e_n_flat"] = _ratio(v_out[2], -vi[2])
        rs["e_t_flat"] = _ratio(np.linalg.norm(v_out[:2]), np.linalg.norm(vi[:2]))
```

## Normal restitution, contact frame

**Definition.** (v_out . n) / (-v_in . n), with n the mean contact normal at the FIRST touch.

**Limits.** Meaningful for single-touch drops only. After several touches v_out leaves a different surface than n describes, and the value can even be negative; the page shows it as 'not meaningful' then.

`src/milani/outcome.py`, line 112:

```python
        if rec["first_contact"] and rec["first_contact"]["normal"]:
            n = np.asarray(rec["first_contact"]["normal"])
            rs["e_n_contact"] = _ratio(v_out @ n, -(vi @ n))
```

## Energy kept

**Definition.** (kinetic + rotational energy at exit + m g (z_exit - z_first contact)) / (kinetic + rotational energy on arrival).

**Why it can be trusted.** Potential energy is counted from the first-contact height, so the number does not depend on where the box floor is. Until 2026-10-31 it was counted from the floor (about 6 % too high on a 1 m deep bed); older records are recomputed exactly from their stored terms.

`src/milani/outcome.py`, line 131:

```python
        e_in = ke_in + (rot_in or 0.0)
        e_out = ke_out + (rot_out or 0.0) + mass * g * (z_out - z_in)
        en = {"ke_in": ke_in, "ke_out": ke_out, "rot_in": rot_in, "rot_out": rot_out,
              "mech_in": e_in, "mech_out": e_out, "potential_reference": "first contact",
              "energy_ratio": round(e_out / e_in, 6) if e_in > 0 else None}
```

`src/milani/outcome.py`, line 218:

```python
def energy_kept(rec: dict):
    """The record's energy ratio with potential energy counted from the
    first-contact height. Records written before 2026-10-31 referenced the
    box floor (mech = ke + rot + m g z); their m g z_in is recovered exactly
    as mech_in - ke_in - rot_in and taken off both sides."""
    en = rec.get("energy") or {}
    if not en or en.get("energy_ratio") is None:
        return None
    if en.get("potential_reference") == "first contact":
        return en["energy_ratio"]
    pe_in = en["mech_in"] - en["ke_in"] - (en.get("rot_in") or 0.0)
    den = en["mech_in"] - pe_in
    return round((en["mech_out"] - pe_in) / den, 6) if den > 0 else None
```

## Outcomes

**Definition.** bounced off: see above. left the patch: the craft's centre left the free crop while on or near the ground (the result is cut off). came to rest: slower than 2 mm/s and 0.01 rad/s for 10 s in contact. captured: its fastest point stayed below 10 % of the arrival speed for 10 s (at micro-g friction takes minutes to stop a creep). time limit: 300 s simulated.

`src/milani/detectors.py`, line 45:

```python
    def update(self, in_contact: bool, speed: float, spin: float, rising: bool,
               clear_ahead, left_patch: bool, point_speed: float | None = None,
               capture_speed: float | None = None) -> str | None:
        """Advance one step. `clear_ahead` is a zero-argument callable (only
        evaluated when detachment is plausible — it is the expensive test).

        "captured": after first contact, the box's fastest point
        (`point_speed`, e.g. v + spin x circumradius) stayed below
        `capture_speed` for rest_time without detaching. At asteroid gravity
        friction stops a creeping box only over minutes (mu g ~ 2.5e-5 m/s^2),
        so this ends runs that can no longer bounce away."""
        s = self.state
        s.t += self.dt
        if s.first_contact_t is not None and left_patch:
            if not in_contact and rising and s.free_time + self.dt >= self.detach_min_time:
                # flying away over the patch edge: the bounce is complete and
                # its exit state is known. The ballistic look-ahead would not
                # have cleared it for a long time — it treats every boulder
                # as its bounding sphere, so a wide craft skimming 20 cm above
                # a boulder field is never "clear" until it has risen above
                # the tallest sphere, minutes at micro-g — and the drift out
                # of the crop came first (2 of 9 drops at 8 cm/s, 0 deg,
                # on the 4 m bed, 2026-10-31: 26 s of free flight, voided)
                s.free_time += self.dt
                s._in_contact = False
                s.detached_how = "patch_edge"
                return "detached"
            # on the ground (sliding, rolling, hopping low) beyond the free
            # crop the box meets frozen grains or bare floor: no longer physical
            s._in_contact = in_contact
            return "left_patch"
        if in_contact:
            if s.first_contact_t is None:
                s.first_contact_t = s.t
            if not s._in_contact:
                s.episodes += 1
            s.last_contact_t = s.t
            s.free_time = 0.0
            if speed < self.rest_speed and spin < self.rest_spin:
                s.rest_time += self.dt
                if s.rest_time >= self.rest_time:
                    s._in_contact = True
                    return "rest"
            else:
                s.rest_time = 0.0
        else:
            if s.first_contact_t is None:
                s.rest_time = 0.0
                if s.t > self.contact_deadline:
                    return "no_contact"
            else:
                s.free_time += self.dt
                if s.free_time >= self.detach_min_time and rising and clear_ahead():
                    s._in_contact = False
                    s.detached_how = "clear"          # no contact possible over the crop (reach_clear)
                    return "detached"
                # at asteroid gravity a box slower than rest_speed hops a few cm
                # for tens of seconds and cannot leave (it could not clear the
                # terrain above): settled, though rarely in unbroken contact
                if speed < self.rest_speed and spin < self.rest_spin:
                    s.rest_time += self.dt
```

`src/milani/impact.py`, line 262:

```python
            left = hd > st.crop_radius_m or (
                hd > st.crop_radius_m - 0.5 * reach and low and not in_contact)

```

## What touched first

**Definition.** The craft piece (bus, wing) and the kind of feature from the contact points of the first touch: 3 or more points a face, 2 an edge, 1 a corner (by where it lies on the box); a wing touch is 'wing'.

`src/milani/impact.py`, line 41:

```python
def _contact_geometry(rows, craft: int, x_craft, M, verts, piece_of=None):
    """Partner ids, mean normal pointing INTO the craft side, contact points,
    which piece of the craft touched (bus / wing+ / wing-) and which box
    feature (face/edge/corner) — heuristic: >=3 points = face, 2 = edge,
    1 = by location of the point on the box. A wing touch is 'wing'."""
    partners, normals, points, gaps, pieces = [], [], [], [], []
    for row in rows:
```

## Site slope and roughness

**Definition.** A plane fitted (least squares) to the bed's height map in a disk around the landing spot (radius: half the craft's smallest dimension, at least 0.1 m): its tilt is the slope, the RMS of heights about it the roughness.

`src/lmgcx/geometry/heightmap.py`, line 57:

```python
    def site(self, cx: float, cy: float, radius: float) -> dict:
        """Terrain descriptors in a disk: mean/max height, best-fit plane
        slope (deg), RMS roughness about that plane, owner ids present."""
        m = self.disk_mask(cx, cy, radius)
        if not m.any():
            raise ValueError("site disk does not overlap the height map")
        ny, nx = self.z.shape
        xs = self.x0 + self.dx * np.arange(nx)
        ys = self.y0 + self.dx * np.arange(ny)
        X, Y = np.meshgrid(xs, ys)
        x, y, z = X[m], Y[m], self.z[m]
        A = np.column_stack([x - cx, y - cy, np.ones_like(x)])
        coef, *_ = np.linalg.lstsq(A, z, rcond=None)
        resid = z - A @ coef
        slope = float(np.degrees(np.arctan(np.hypot(coef[0], coef[1]))))
        return {"z_mean": float(z.mean()), "z_max": float(z.max()),
                "z_center": float(self.height(cx, cy)),
                "slope_deg": slope, "roughness_rms": float(np.sqrt(np.mean(resid ** 2))),
                "owners": np.unique(self.owner[m]).tolist()}
```

## Grains moved, largest move, fastest grain

**Definition.** Free grains displaced more than 1 mm between the start and the end of the drop; the largest such displacement; the fastest free grain seen (sampled every 5 steps).

`src/milani/impact.py`, line 333:

```python
        bed_gap0=bed_gap0, grains_moved=int(np.sum(disp > 1e-3)),
        max_grain_disp=float(disp.max()) if disp.size else 0.0,
```

## Crop-edge disturbance

**Definition.** The fastest free grain in the crop's outer band (width: half the crop radius or two largest grain radii, whichever is smaller) divided by the arrival speed.

**Why it can be trusted.** If the impact moves grains at the crop's edge, a larger crop could change the result. Measured on the grains' motion, not on forces: rigid grains wedged against the frozen ring carry arbitrary internal stress.

`src/milani/impact.py`, line 153:

```python
    edge = free[d0 > st.crop_radius_m - band]
```

## Solver steps at budget

**Definition.** Time steps, from the first touch on, in which the contact solver used its whole budget of 10 000 sweeps without reaching its tolerance. QA warns above 5 % of those steps.

`src/lmgcx/postpro.py`, line 42:

```python
def solver_health(postpro_dir, max_sweeps: int, first_step: int | None = None) -> dict:
    """Non-convergence is NOT an error in LMGC90 and its warning is
    suppressed by DisableLogMes (crash guide §1 gotcha 2b); the only witness
    is SOLVER_INFORMATIONS: a step whose iteration count reached the budget
    did not converge. With `first_step` (an impact's first contact, 1-based
    row) the count is also split into before and from that step on: a run
    has a fixed handful of such steps — the crop's own relaxation before
    the touch and the shock of the touch itself (8-13 per run on the 4 m
    bed, 2026-10-31) — so a fraction of ALL steps flags a 1 s run and not
    a 100 s one. The contact-phase fraction is the one to grade."""
    d = named(Path(postpro_dir) / "SOLVER_INFORMATIONS.DAT", SOLVER_COLUMNS)
    it = d.get("iterations", np.empty(0))
    if it.size == 0:
        return {"steps": 0, "unconverged_steps": None, "max_iterations": None,
                "mean_iterations": None, "steps_in_contact_phase": None,
                "unconverged_in_contact_phase": None}
    bad = it >= max_sweeps
    out = {"steps": int(it.size), "unconverged_steps": int(bad.sum()),
           "max_iterations": int(it.max()), "mean_iterations": float(it.mean()),
           "steps_in_contact_phase": None, "unconverged_in_contact_phase": None}
    if first_step is not None and 1 <= first_step <= it.size:
        tail = bad[first_step - 1:]
        out["steps_in_contact_phase"] = int(tail.size)
        out["unconverged_in_contact_phase"] = int(tail.sum())
        out["unconverged_before_contact"] = int(bad[:first_step - 1].sum())
    return out
```

## Movies

**Definition.** LMGC90's own ParaView frames of the drop (every 0.1 s for 15 s, then every second), rendered by ParaView 6.0 offscreen and played at 10 frames a second: real time for the first 15 s, then ten times faster. Grain colour is grain speed; the white line is the path of the craft's centre.

`src/milani/tools/pv_drop_movie.py`, line 37:

```python
codes = [0] * (len(bodies) + 1)
for i, b in enumerate(bodies, 1):
    k = b["kind"]
    codes[i] = 3 if k == "craft" else 1 if k.endswith("_frozen") else 2 if k in ("boulder", "fines") else 0

reader = PVDReader(FileName=os.path.join(run_dir, "DISPLAY", "lmgc90.pvd"))
```
