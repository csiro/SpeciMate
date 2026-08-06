# SpeciMate — Conventions
Create date: 2026-07-20
Last modified date: 2026-07-20

House style and hard-won gotchas. Read this before writing or editing any handler.

## Handler prefixes

Each prefix marks which subsystem a handler belongs to, regardless of which stack it's
physically stored in.

| Prefix | Owner |
|---|---|
| `sm` | Shared/OCR utilities — services, prefs, data folders, logging, save/load |
| `ds` | Definition stack-level functions (data definitions) |
| `dp` | DefinitionPreview card handlers |
| `pm` | Prompts library functions |
| `de` | Display Editor commands |
| `cu` | Custom card (Metadata curation) handlers |
| `oc` | OCR-related utilities |

If you're not sure where a handler should live, match it to the subsystem it acts on, not
the card you happened to be looking at.

## Naming

- **Handler names**: dotted, readable as English — `smSaveData.OCR`, `smProject.open`,
  `deAssignMultiToGroup`. The part before the dot is the subsystem/object, after the dot
  is the action.
- **Script-local variables**: `s` prefix — `sImgDataA`, `sData.NER`, `sMultimodalConfig`.
- **Globals**: `g` prefix — `gPrefs`.
- **Constants**: `k` prefix — `kAPPNAME`, `kProcessType.OCR`, `kDataFolderPref`.
- **Parameters**: `p` prefix — `pFile`, `pForceSave`.
- **Temporary/local variables**: `t` prefix — `tFile`, `tJSON`, `tResult`.
- **Custom properties**: `c` prefix (e.g. `cFieldName`) to distinguish from built-in
  properties.

[Screenshot: example handler showing prefix conventions side by side]

## Logging and errors

- `smLog` — general trace logging, gated by a testing pref where noisy.
- `smError` — user-visible or logged error condition.
- `smTest` — verbose diagnostic output during development.
- `errorDialog` — formats and surfaces a LiveCode execution/parse error.
- Wrap risky operations (file I/O, JSON parsing, network calls) in `try`/`catch` and log
  the caught error rather than letting it fail silently.

## File I/O

- Read/write data files via `smReadDataFile` / `smWriteDataFile` — don't call `put ... into
  url` directly outside of these, so encoding and error handling stay consistent.
- JSON conversion uses PhotonJSON (`ArrayToJSON` / `JSONToArray`), not LiveCode's built-in
  array-to-JSON, due to past encoding issues.
- Binary array data (see `data-model.md`) uses `arrayEncode`/`arrayDecode`, not JSON.

## Comments

- Minimal. Code should read clearly on its own.
- Add a comment only where behaviour is non-obvious — e.g. why `break` is used instead of
  `exit repeat` inside a `switch`, or why a fallback path exists.
- Prefer inline notes on the subtle line over separate documentation blocks.

## Gotchas

A working list of behaviours that have already caused bugs. Check here before assuming.

1. **Array keys are not case-sensitive by default.** `caseSensitive` is local to the
   current handler and resets to `false` when it finishes — it does not propagate to
   handlers you call. Messages, object names, and LiveCode terms are never
   case-sensitive even when `caseSensitive` is `true`. A mismatch like `["Name"]` vs
   `["name"]` is not a bug unless `caseSensitive` is explicitly set `true` in that exact
   handler.
2. **`repeat for each key` order is non-deterministic.** Never rely on the "first" key
   from array iteration being meaningful.
3. **Dynamic control names must never be used as data keys.** Controls built at runtime
   (e.g. `fld_<fieldId>`) need a stable `cFieldName` custom property stashed at build
   time (see `cuCreateField` / `cuCreateCheckbox`). Save/close handlers must read the key
   from that property, not from the control's name.
4. **Two sources of truth on startup.** UI field content persists in the stack file
   across sessions; runtime state (e.g. `gPrefs`) does not. On launch, only UI fields
   survive, which can produce a state that looks valid but fails on save or action until
   runtime state is properly reloaded.
5. **`mouseLeave` in `cuFieldBehavior` must stay absent.** An earlier, apparently inert
   empty handler was load-bearing — restoring it causes popup menus to flicker. The
   picklist button handles its own dismissal.
6. **`visible = false` on a display field** controls rendering in the curation form
   (Metadata window) only — it does not affect visibility in the Display Editor grid.
7. **LLM payloads are built by string substitution, not serialisation.** Any unescaped
   character in a substituted value (prompt, role, entity names, OCR text) can break the
   request JSON. See `services.md` for the escaping rule.

## Adding to this list

When a bug traces back to a LiveCode or SpeciMate-specific behaviour that isn't obvious
from reading the code, add it here as a numbered gotcha — one line for the rule, one for
why it matters.
