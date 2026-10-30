# BULK_BEHAV.DAT Structure

Written by `pre.writeDatbox()` when `gravy=` is supplied.  Patched by
`command_impact.py` before `ReadDatbox` to switch gravity.

```
$gravy  
                   grv1= 0.0000000e+00  grv2= 0.0000000e+00  grv3=-9.8100000e+01
$behav  lawty
 MOUxx  RIGID                           Umas= 1.0000000e+02
$behav  lawty
 TDURx  RIGID                           Umas= 1.0000000e+03
```

- `grv1`, `grv2`, `grv3` — gravity acceleration components in x, y, z  
  (all three fields exist; patch any with `re.sub(r"grv3=\s*[-+0-9.eE]+", ...)`)
- `Umas` — mass density (set via `material(density=...)` in pre)
- `RIGID` — material law type

**Gravity patching pattern** (from `command_impact.py`):
```python
text = behav_file.read_text()
text_new = re.sub(r"grv3=\s*[-+0-9.eE]+", f"grv3={new_value: .7e}", text)
behav_file.write_text(text_new)
```
Apply **before** `ReadDatbox()`.  Safe to call only on `grv3`; `grv1`/`grv2` can
be patched with identical regexes if non-vertical gravity is needed.

---
