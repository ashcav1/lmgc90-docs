## Container Objects

---

## `avatars()`

**What it does:** Creates an empty ordered container for all `avatar` (body)
objects.  Inherits from `list`.

**Arguments:** None

**Returns:** `avatars` instance

**As used in our code:**
```python
bodies = pre.avatars()
```

**Notes:**
- `bodies.addAvatar(av)` assigns `av.number = len(self)` (0-based internal index)
  then appends.
- `bodies += grain` calls `__iadd__`, which accepts a single avatar or a list
  of avatars.
- Bodies are numbered in creation order; chipy sees them 1-based.

---

## `materials()`

**What it does:** Creates an empty dictionary container for `material` objects.

**Arguments:** None

**Returns:** `materials` instance

**As used in our code:**
```python
mats = pre.materials()
mats.addMaterial(tdur, pdur)
```

**Notes:**
- `addMaterial(*mat)` validates each argument is a `bulk_behav.material` instance.
- Material names must be exactly 5 characters (pads/truncates internally).

---

## `models()`

**What it does:** Creates an empty dictionary container for `model` objects.

**Arguments:** None

**Returns:** `models` instance

**As used in our code:**
```python
mods = pre.models()
mods.addModel(mod)
```

**Notes:**
- `addModel(*mo)` validates each argument is a `model.model` instance and checks
  for duplicate names.

---

## `see_tables()`

**What it does:** Creates an empty list container for `see_table` (contact pair
definition) objects.

**Arguments:** None

**Returns:** `see_tables` instance

**As used in our code:**
```python
svs = pre.see_tables()
svs += pre.see_table(...)
```

**Notes:**
- `svs += see_table_obj` calls `__iadd__`, which wraps `addSeeTable()`.

---

## `tact_behavs()`

**What it does:** Creates an empty dictionary container for `tact_behav` (contact
law) objects.

**Arguments:** None

**Returns:** `tact_behavs` instance

**As used in our code:**
```python
tacts = pre.tact_behavs()
tacts += b
```

**Notes:**
- Uses `+= ` operator (`__add__` / `__iadd__`) to add behaviours — different from
  the `addBehav()` method, but functionally equivalent.

---

## `postpro_commands()`

**What it does:** Creates a container for post-processing command definitions.
Can be passed to `writeDatbox` to write `POSTPRO.DAT`.

**Arguments:** None

**Returns:** `postpro_commands` instance (empty by default)

**As used in our code:**
```python
post = pre.postpro_commands()
# ... (no commands added in current scripts)
pre.writeDatbox(..., post=post, ...)
```

**Notes:**
- Currently used as an empty placeholder.  Commands for extracting contact forces,
  energies, etc. would be added here.

---

