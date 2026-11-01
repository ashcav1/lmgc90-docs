# How each number is computed

Every number in the [drop reel](../drops/index.html) and the run log, defined: what it is, why it can be trusted (or where it cannot), and the code that computes it. File and line refer to the project's code repository at the time this page was generated (2026-10-31 23:59); the excerpts are copied from that code, not paraphrased.

## The craft model

**Definition.** Milani is rigid convex boxes: a 13 x 24.6 x 36.6 cm bus of 10.8 kg and, deployed, two solar wings 1 cm thick and 59 x 36.6 cm, 0.6 kg each: 12 kg in all. The stowed craft is the bus alone at 12 kg. Mass, centre of mass and inertia come from the pieces' own masses. Inside the deployed bus sits a massless ballast box (3.9 x 24 x 36 cm, 3 mm inside every wall): LMGC90 places a body made of several pieces at the centre of their volumes rather than of their masses (mod_RBDY3, reading the bodies), and the ballast's volume moves that point onto the centre of mass.

**Why it can be trusted.** Without it the deployed craft turned about a point 10.2 mm from its centre of mass: the wings are far lighter for their volume than the bus. Found on 1 November by comparing the craft in the simulation's own frames with its design. Every drop now checks that the simulated craft sits within 0.1 mm of its design (it does, to 1e-11 m). No grain can reach the ballast without pushing 3 mm into the bus first.

**Limits.** Drops before 1 November (the first laptop sets, the lab Mac's weekend drops) have the 10 mm offset and are re-simulated (lab.py campaign rerun --stale). On one 4 m bed drop the fix moved e from 0.300 to 0.329; the stowed craft is unaffected.

`src/milani/impactors.py`, line 159:

```python
def ballast_piece(pcs, inset: float = BALLAST_INSET_M):
    """An internal box that puts the craft's VOLUME centroid on its centre of
    mass. LMGC90 places a multi-contactor body's centre at the volume-weighted
    centroid of its contactors' geometric volumes (mod_RBDY3, read_bodies:
    OG = sum(v_i shift_i) / sum(v_i)) and keeps the principal inertia it is
    given, so a craft whose pieces differ in density (deployed: bus 10.8 kg,
    wings 0.6 kg each) turned about a point 10.2 mm from its centre of mass
    until 2026-11-01. The ballast lies `inset` inside the bus on every side,
    so no grain can reach it before the bus; None when the volumes already
    balance (one piece, or uniform density)."""
    M = sum(p.mass for p in pcs)
    com = sum(p.mass * p.centre for p in pcs) / M
    S = sum(float(np.prod(p.dims)) * (p.centre - com) for p in pcs)
    if float(np.linalg.norm(S)) < 1e-12:
        return None
    bus = next(p for p in pcs if p.name == "bus")
    if not np.allclose(bus.axes, np.eye(3)):
        raise NotImplementedError("ballast: the bus must be aligned with the bus frame")
    k = int(np.argmax(np.abs(S)))
    if float(np.abs(np.delete(S, k)).max()) > 1e-9 * abs(float(S[k])):
        raise NotImplementedError(f"ballast: volume imbalance off the bus axes {S}")
    h = 0.5 * np.asarray(bus.dims, dtype=float) - inset
    side = -float(np.sign(S[k]))                  # the ballast sits opposite the imbalance
    face = float(bus.centre[k]) + side * h[k]     # inner face of the bus wall on that side
    A = float(np.prod(np.delete(2.0 * h, k)))
    a = side * (face - float(com[k]))             # from the centre of mass to that face, > 0
    need = abs(float(S[k])) / A                   # the ballast's length L solves L (a - L/2) = need
    disc = a * a - 2.0 * need
    if disc < 0:
        raise ValueError("ballast: the bus cannot hold enough volume to centre the craft")
    L = a - math.sqrt(disc)
    dims = 2.0 * h
    dims[k] = L
    centre = np.asarray(bus.centre, dtype=float).copy()
    centre[k] = face - side * 0.5 * L
    return _box_piece("ballast", 0.0, tuple(dims), centre, np.eye(3))


def _craft_info(spec: ImpactorSpec, pcs, R, centre, av, rho, ballast=None) -> dict:
    axes = np.asarray(av.getBulkFrame(), dtype=float)     # columns = principal axes (global)
    com = np.asarray(av.nodes[1].coor, dtype=float)
```

`src/milani/impact.py`, line 167:

```python
    craft_geometry_error = None
    if craft_info is not None:
        Vsim = craft_vertices()
        Vjs = (np.vstack([np.asarray(p["vertices"], dtype=float) for p in craft_info["pieces"]])
               @ s.axes(craft) + s.vector("Coor_", craft)[:3])
        craft_geometry_error = float(max(np.linalg.norm(Vjs - v, axis=1).min() for v in Vsim))
        if craft_geometry_error > 1e-4:
            print(f"[impact] WARNING: the simulated craft is {1000 * craft_geometry_error:.2f} mm "
                  f"from its craft.json geometry", flush=True)
    # free grains in the outer band of the crop: if the impact moves THEM, the
    # disturbance reached the boundary (kinematic crop check; frozen-ring forces
```

## In contact (a touch)

**Definition.** At each 5 ms time step the craft is in contact when the resultant contact force the solver applied to it is non-zero: |R| > 1e-12 N.

**Why it can be trusted.** This is the solver's own statement that some contact transmitted force in that step. Bodies that are merely close (within the detection distance) carry zero force and do not count. Non-smooth contact dynamics resolves even an instantaneous collision as an impulse inside its time step, and contact detection looks 15 mm ahead (the alert distance) while the craft moves at most 0.4 mm per step at 8 cm/s, so no touch can fall between two steps.

**Limits.** A resultant can in principle cancel when the craft is squeezed equally from opposite sides; the contact count column of the trajectory (n_box_contacts) would show such a case, and none has been seen.

`src/milani/impact.py`, line 239:

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

`src/milani/impact.py`, line 255:

```python
            if in_contact and first is None:
                pv, pw, pwb = (prev if prev is not None else (v, w_g, v6[3:6]))
                v_in, w_in, wb_in = pv.copy(), pw.copy(), pwb.copy()
                I = s.contacts()
                geo = _contact_geometry(_craft_contacts(I, craft, ballast_tac), craft, x, M,
                                        craft_vertices(), piece_of)
                first = {"t": k * dt, "x": x.copy(), "geo": geo}
            if in_contact and k % 5 == 0:
```

## Touches (contact episodes)

**Definition.** How many times the craft goes from free to in contact during the drop.

`src/milani/detectors.py`, line 86:

```python
            if not s._in_contact:
                s.episodes += 1
            s.last_contact_t = s.t
```

## Last contact

**Definition.** The last step at which the craft was in contact before the drop ended.

**Why it can be trusted.** Within the simulated time this is exact (every step is tested). Whether a later touch would have come is what the end-of-drop test and the last-contact check below settle.

`src/milani/detectors.py`, line 88:

```python
            s.last_contact_t = s.t
```

## Exit state (exit speed, spin, elevation)

**Definition.** The craft's velocity and spin at the step its contact ended for the last time. Each separation overwrites it, so at the end it is the last one. Exit elevation is the angle of the exit velocity above the horizontal of the bed (not of the local surface).

`src/milani/impact.py`, line 281:

```python
            if was_in_contact and not in_contact:
                exit_state = {"t": k * dt, "x": x.copy(), "v": v.copy(), "w_g": w_g.copy(),
                              "w_b": v6[3:6].copy(), "M": M.copy()}
```

## Bounced off: when a drop is over

**Definition.** A drop ends as 'bounced off' when the craft has been free for 0.5 s, is rising, and its exact free flight (next entry) does not come back down on a grain the simulation can move, the free crop, within the time the run could still take (up to 300 s). Until that holds the simulation goes on and the solver decides, also when the craft's centre has crossed the crop's edge but a wing is still over it. The flight is checked against the whole bed: 'clear' if it touches nothing in that time, 'beyond' if its next touch is on the frozen ring or the bed outside the crop; the prediction is stored with the drop.

**Why it can be trusted.** Once the craft is free nothing acts on it but gravity, so its future is known exactly: nothing is left to simulate unless it comes back down on grains the simulation can move. The two tests this replaced both failed on bed 1. On 31 October a 3-30 s look-ahead of the tumbling craft missed a wing-tip graze 19 s after a drop was declared over. On 1 November the exact check showed 'flying out over the crop edge' had ended drop b955841420ca while a wing was still over the crop: it touches a free grain 0.58 m from the target 4 s later. The exact test also ends drops sooner: a 4 m-bed drop now stops 0.5 s after its last touch instead of 31 s later, with an identical result, at a seventh of the computing.

**Limits.** The free grains are taken at their pose when the test is made; a grain still creeping (mm/s) could move into the path afterwards. The replay in the last-contact check covers that wherever the flight passes within 5 cm of a free grain.

`src/milani/impact.py`, line 295:

```python
            def ahead():
                # how the drop ends, from the craft's exact free flight
                # (flight.scan): None while it would come back down on a free
                # grain, or pass within 1 cm + (fastest free grain's surface
                # speed) x (time ahead) of one, within the time the run could
                # still take; "clear" if it touches nothing, "beyond" if its
                # next touch is outside the free crop. A predicted free-grain
                # touch at t* is re-asked at t* - 0.5 s, at most 5 s on (the
                # flight is exact; only the grains' settling changes it)
                t = k * dt
                if predictor is None:
                    return "patch_edge" if left else None
                if t < ask_memo["next"]:
                    return ask_memo["how"]
                Vg = s.vectors("V____")[free]
                vg = (float((np.linalg.norm(Vg[:, :3], axis=1)
                             + np.linalg.norm(Vg[:, 3:6], axis=1) * radii[free]).max()) if len(free) else 0.0)
                how = predictor.decide(x, v, M, v6[3:6], s.vectors("Coor_")[:, :3],
                                       lambda i: s.axes(int(i) + 1), t, st.max_sim_time_s - t,
                                       grain_speed=vg)
                r = predictor.last
                wait = 0.5
                if how is None and r.get("touch_at_t") is not None:
```

`src/milani/detectors.py`, line 104:

```python
                how = ahead() if (s.free_time >= self.detach_min_time and rising) else None
                if how:
                    s._in_contact = False
                    s.detached_how = how
                    return "detached"
```

`src/milani/flight.py`, line None:

```python
(not found: def decide(self, x, v, M_rows, w_body, C, axes, t_now: float, horizon: float):)
```

## Exact free flight

**Definition.** From any free state the craft's motion is computed exactly: its centre of mass on the ballistic arc, its rotation from Euler's equations for a torque-free body (the spin axis wobbles because the craft's three moments of inertia differ), integrated by 4th-order Runge-Kutta in steps of at most 0.02 s. The craft is then moved along that flight over the bed in two stages. Points every 2 cm on its surface are compared with the bed's top surface (1 cm grid, each cell raised to its highest neighbour); wherever a point comes within 2 cm, the exact distance between the craft's pieces (three convex boxes when deployed) and every grain nearby (convex polyhedra) decides. A touch is an exact distance under 1 mm. Near a grain every step is the longest in which no point of the craft could close that distance (conservative advancement), so a touch cannot fall between two tests.

**Why it can be trusted.** It is the physics LMGC90 itself integrates for a free body, gyroscopic torque included, so it can be checked against the simulation: over the recorded free flights of bed 1 (10-70 s each) the predicted centre stays within 0.001 mm and the orientation within 0.06 degrees of the simulation's. The exact distance is the closest pair of points of two convex hulls, a small least-squares problem; on 1000 pairs of boxes with known distances its worst error is 2e-13 m.

**Limits.** The crop's grains are taken as they stand (the run's last frame for the check, the current pose in the end-of-drop test). Outside the crop the bed is as settled, which is what an undisturbed bed would be.

`src/milani/flight.py`, line 77:

```python
def _deriv(q, w, I):
    """d/dt of (q body->world, w body-frame) for a torque-free rigid body."""
    I1, I2, I3 = I
    dw = np.array([(I2 - I3) * w[1] * w[2] / I1,
                   (I3 - I1) * w[2] * w[0] / I2,
                   (I1 - I2) * w[0] * w[1] / I3])
    dq = 0.5 * _qmul(q, np.array([0.0, w[0], w[1], w[2]]))
    return dq, dw

```

`src/milani/flight.py`, line 159:

```python
def hull_distance(A, B, w: float = 1.0e3):
    """Distance between the convex hulls of point sets A and B (0 when they
    overlap) and the closest point of each: the smallest |p - q| over convex
    combinations p of A and q of B, a non-negative least-squares problem
    with the two sum-to-one constraints as heavily weighted rows, solved by
    bounded-variable least squares (scipy's nnls fails to converge on 10-15 %
    of touching boxes; BVLS: none of 1000, worst error 2e-13 m, 0.15 ms)."""
    from scipy.optimize import lsq_linear
    A = np.asarray(A, dtype=float)
    B = np.asarray(B, dtype=float)
    o = 0.5 * (A.mean(axis=0) + B.mean(axis=0))
    a, b = (A - o).T, (B - o).T
    n, m = a.shape[1], b.shape[1]
    M = np.zeros((5, n + m))
    M[:3, :n], M[:3, n:] = a, -b
    M[3, :n], M[4, n:] = w, w
    x = lsq_linear(M, np.array([0.0, 0.0, 0.0, w, w]), bounds=(0.0, np.inf), method="bvls", tol=1e-14).x
    p = a @ (x[:n] / x[:n].sum()) + o
    q = b @ (x[n:] / x[n:].sum()) + o
    return float(np.linalg.norm(p - q)), p, q


# ------------------------------------------------------------------ terrain
```

`src/milani/flight.py`, line 381:

```python
                h = (min(d, near_m) - touch_tol) / pt_speed      # conservative advancement
```

## Last-contact check

**Definition.** Every bounced-off drop is checked after it has finished, two ways. First, the exact free flight from its last touch over the whole bed: until the run's end against the grains outside the simulation (inside it the solver has already decided), after it against the free grains at their final pose, the frozen ring and the rest of the bed, until the craft has left the bed. Second, where that flight passes within 5 cm of a grain that can move, a replay of the drop with the end-of-drop test switched off, continued until the craft leaves the crop, stops, or 150 s past the original end. The replay is deterministic, so up to the original end it is the original run, and it includes the grains' own motion.

**Why it can be trusted.** Verdicts: last contact confirmed (no touch anywhere on the bed after the last touch); next touch off the patch (no later touch on the free crop; the next one, shown with when and where, is the landing of a new hop on the frozen ring or the bed beyond); lands back on the patch (a long hop back onto the crop after the run's 300 s limit); touched again (a touch the run should have simulated: it ended too early).

**Limits.** Drops checked before 1 November had only the replay, which cannot follow a craft out over the crop's edge; they show 'left over the crop edge' until re-checked.

`src/milani/checks.py`, line 169:

```python
def check_last_contact(run_dir: Path, extra_s: float = 150.0, keep: bool = False,
                       replay: str = "near") -> dict:
    from .flight import check_finished_run
    run_dir = Path(run_dir)
    rec = json.loads((run_dir / "outcome.json").read_text())
    ct = rec.get("contact") or {}
    out = {"run_id": rec["run_id"], "check_version": CHECK_VERSION,
           "t_last_contact": ct.get("last_t"), "t_detached": rec.get("sim_time_s"),
           "episodes": ct.get("episodes"), "detached_how": ct.get("detached_how")}
    if rec["outcome"] != "detached":
        out["verdict"] = "not applicable"
        (run_dir / "last_contact_check.json").write_text(json.dumps(out, indent=2))
        return out
    try:
        fl = check_finished_run(run_dir)
    except Exception as e:                                   # noqa: BLE001
        fl = {"verdict": f"error: {type(e).__name__}: {str(e)[:200]}"}
    out["flight"] = fl
    near = (fl.get("closest_by_layer_mm") or {}).get("free", 1e9) < 1000 * NEAR_FREE_M
    if str(ct.get("detached_how") or "") == "patch_edge" or _ended_beyond_crop(run_dir, rec):
        out["replay_verdict"] = "left the patch at detachment"
    elif replay == "all" or (replay == "near" and (near or not fl.get("verdict", "").startswith(
            ("clear", "next touch")))):
        if replay == "near" and near:
            # the closest approach is fl["from_t"] + t; the run ended at run_end_t
            t_close = float(fl["from_t"]) + float((fl.get("closest_by_layer_t_s") or {}).get("free", 0.0))
            extra_s = min(extra_s, max(10.0, t_close - float(fl["run_end_t"]) + 10.0))
        out["replay_extra_s"] = round(extra_s, 2)
        out.update(_replay(run_dir, rec, extra_s, keep))
    else:
        out["replay_verdict"] = (f"not needed (after the end the flight never comes within "
```

`src/milani/checks.py`, line 139:

```python
def _combine(rec: dict, fl: dict, rv: str | None, max_t: float) -> tuple[str, dict | None]:
    """The drop's verdict from the two checks, and its next touch."""
    ct = rec.get("contact") or {}
    t_last = float(ct.get("last_t") or 0.0)
    nxt = None
    fv = fl.get("verdict", "")
    if fl.get("touch"):
        cl = fl.get("closest") or {}
        nxt = {"where": {"free": "the free crop", "frozen": "the frozen ring",
                         "bed": "the bed beyond the simulation"}.get(fl.get("layer"), fl.get("layer")),
               "t": fl.get("touch_t"),
               "after_last_contact_s": round(float(fl["touch_t"]) - t_last, 2),
               "distance_from_target_m": cl.get("distance_from_target_m")}
    if rv == "touched again":
        return "touched again", nxt
    if rv == "touched the frozen ring as predicted":
        return "next touch off the patch", nxt
    if fv == "passed through unsimulated terrain":
        return "touched unsimulated terrain", nxt
    if fv == "next touch inside the patch":
        return ("touched again" if float(fl["touch_t"]) <= max_t else "lands back on the patch"), nxt
    if fv in ("next touch on the frozen ring", "next touch beyond the patch"):
        return "next touch off the patch", nxt
    if fv.startswith("clear"):
        return "confirmed", nxt
    if rv == "confirmed":
        return "confirmed", nxt
    return "unchecked", nxt

```

## Speed ratio e

**Definition.** e = |v_out| / |v_in|: exit speed over arrival speed.

`src/milani/outcome.py`, line 117:

```python
        rs["e_speed"] = _ratio(np.linalg.norm(v_out), np.linalg.norm(vi))
        rs["e_n_flat"] = _ratio(v_out[2], -vi[2])
        rs["e_t_flat"] = _ratio(np.linalg.norm(v_out[:2]), np.linalg.norm(vi[:2]))
```

## Normal and tangential restitution (flat frame)

**Definition.** Relative to the bed's horizontal: e_n = v_out,z / (-v_in,z) and e_t = |v_out,horizontal| / |v_in,horizontal|.

**Limits.** The bed's horizontal is not the surface the craft hit: on a 40-degree boulder face these mix normal and tangential.

`src/milani/outcome.py`, line 118:

```python
        rs["e_n_flat"] = _ratio(v_out[2], -vi[2])
        rs["e_t_flat"] = _ratio(np.linalg.norm(v_out[:2]), np.linalg.norm(vi[:2]))
```

## Normal restitution, contact frame

**Definition.** (v_out . n) / (-v_in . n), with n the mean contact normal at the FIRST touch.

**Limits.** Meaningful for single-touch drops only. After several touches v_out leaves a different surface than n describes, and the value can even be negative; the page shows it as 'not meaningful' then.

`src/milani/outcome.py`, line 120:

```python
        if rec["first_contact"] and rec["first_contact"]["normal"]:
            n = np.asarray(rec["first_contact"]["normal"])
            rs["e_n_contact"] = _ratio(v_out @ n, -(vi @ n))
```

## Energy kept

**Definition.** (kinetic + rotational energy at exit + m g (z_exit - z_first contact)) / (kinetic + rotational energy on arrival).

**Why it can be trusted.** Potential energy is counted from the first-contact height, so the number does not depend on where the box floor is. Until 2026-10-31 it was counted from the floor (about 6 % too high on a 1 m deep bed); older records are recomputed exactly from their stored terms.

`src/milani/outcome.py`, line 139:

```python
        e_in = ke_in + (rot_in or 0.0)
        e_out = ke_out + (rot_out or 0.0) + mass * g * (z_out - z_in)
        en = {"ke_in": ke_in, "ke_out": ke_out, "rot_in": rot_in, "rot_out": rot_out,
              "mech_in": e_in, "mech_out": e_out, "potential_reference": "first contact",
              "energy_ratio": round(e_out / e_in, 6) if e_in > 0 else None}
```

`src/milani/outcome.py`, line 226:

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

**Definition.** bounced off: see above. left the patch: the craft's centre left the free crop while on or near the ground, and its flight (if it was airborne) does not come back down on the crop (the result is cut off). came to rest: slower than 2 mm/s and 0.01 rad/s for 10 s in contact. captured: its fastest point stayed below 10 % of the arrival speed for 10 s (at micro-g friction takes minutes to stop a creep). time limit: 300 s simulated.

`src/milani/detectors.py`, line 46:

```python
    def update(self, in_contact: bool, speed: float, spin: float, rising: bool,
               ahead, left_patch: bool, point_speed: float | None = None,
               capture_speed: float | None = None, free_touch_ahead: bool = False) -> str | None:
        """Advance one step. `ahead` is a zero-argument callable, evaluated
        only when the craft has been free for detach_min_time and is rising
        (it is the expensive test): None while the craft's exact free flight
        comes back down on a free grain of the crop (keep simulating, the
        solver decides), otherwise how the drop ends: "clear" (no touch on
        the bed in the time the run could still take) or "beyond" (its next
        touch is outside the free crop) — flight.LivePredictor.
        `free_touch_ahead`: the last prediction is such a touch, still to come.

        "captured": after first contact, the box's fastest point
        (`point_speed`, e.g. v + spin x circumradius) stayed below
        `capture_speed` for rest_time without detaching. At asteroid gravity
        friction stops a creeping box only over minutes (mu g ~ 2.5e-5 m/s^2),
        so this ends runs that can no longer bounce away."""
        s = self.state
        s.t += self.dt
        if s.first_contact_t is not None and left_patch:
            if not in_contact and rising and s.free_time + self.dt >= self.detach_min_time:
                # flying out over the patch edge. Until 2026-10-31 this alone
                # ended the drop ("patch_edge"), but a wing still over the
                # crop can come down on it: b955841420ca (bed 1) touched a
                # free grain 0.58 m from the target 4 s after its run ended.
                # The exact flight decides, as anywhere else
                how = ahead()
                if how:
                    s.free_time += self.dt
                    s._in_contact = False
                    s.detached_how = how
                    return "detached"
            elif in_contact or not free_touch_ahead:
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
                how = ahead() if (s.free_time >= self.detach_min_time and rising) else None
                if how:
                    s._in_contact = False
```

`src/milani/impact.py`, line 292:

```python
            left = hd > st.crop_radius_m or (
                hd > st.crop_radius_m - 0.5 * reach and low and not in_contact)

```

## What touched first

**Definition.** The craft piece (bus, wing) and the kind of feature from the contact points of the first touch: 3 or more points a face, 2 an edge, 1 a corner (by where it lies on the box); a wing touch is 'wing'.

`src/milani/impact.py`, line 47:

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

`src/milani/impact.py`, line 376:

```python
        bed_gap0=bed_gap0, grains_moved=int(np.sum(disp > 1e-3)),
        max_grain_disp=float(disp.max()) if disp.size else 0.0,
```

## Crop-edge disturbance

**Definition.** The fastest free grain in the crop's outer band (width: half the crop radius or two largest grain radii, whichever is smaller) divided by the arrival speed.

**Why it can be trusted.** If the impact moves grains at the crop's edge, a larger crop could change the result. Measured on the grains' motion, not on forces: rigid grains wedged against the frozen ring carry arbitrary internal stress.

`src/milani/impact.py`, line 181:

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
