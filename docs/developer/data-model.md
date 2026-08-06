# SpeciMate — Data Model
Create date: 2026-07-20
Last modified date: 2026-07-20

Every structure that persists or is passed between subsystems. Read `architecture.md`
first for where these fit into Setup vs Curation.

## Data definition (`sDefinition` / `sDataDef`)

Describes *what fields exist* for a specimen type and how NER should extract them.

- **Key fields**: `definitionId`, `name`, `mode` (`"simple"` or full/template mode),
  `columns[]` — each with `name`, `fieldId`, `category`, `required`, `default`,
  `picklist`, `multiline`, `comments`.
- **`fieldId`**: derived from the field name (`dsNameToFieldId`) if not already set;
  `dsEnsureFieldIds` backfills any missing ones. This is the stable key used to link a
  display definition's fields back to the data definition — never rely on `name` for
  linking, since it can change independent of `fieldId`.
- **Validation**: `dsValidateDefinition` checks for duplicate `fieldId`s, duplicate
  names, and empty `fieldId`s before downstream use.
- **Simple mode**: a flattened, ad-hoc definition built directly from the OCR window's
  Step 3 fields (Role/Prompt/Columns). Only field names survive — richer structure
  (picklists, column types) from a previously loaded full definition is not preserved
  when flattening.
- **Storage**: JSON, filename based on `definitionId`, in `<dataFolder>/definitions/`.

[Screenshot: Project Definition editor showing a definition's field list]

## Display definition (`sDisplayDef` / `sDisplaysA`)

Describes *how fields are laid out* in the curation form — independent of the data
definition, but linked to one.

- **Top-level keys**: `displayId`, `name`, `definitionId` (link back to the data
  definition), `layout`, `date`, `modified`.
- **`groups[]`**: each group has `groupId`, `groupLabel`, `bgColor`, and a `fields[]`
  list. Ungrouped fields get a synthetic `groupId` (`ungrouped_<n>`) so each visual block
  still has a unique id — this avoids `rect_<groupId>` collisions when the same real
  `groupId` appears in more than one non-contiguous block.
- **Per-field props**: `fieldId` (matches the data definition's `fieldId`), `span`,
  `renderHint` (defaults to `"single-line"`), `visible` (defaults to `true` if absent —
  only stored when `false`), `readOnly` (defaults to `false` — only stored when `true`).
- **`visible = false`** hides the field from the curation form only; it stays visible in
  the Display Editor grid (`sGridData`) so it can still be edited there.
- **Array key order is unreliable after a JSON round-trip.** Rebuilding the grid
  (`dePopulateGrid`) sorts by stored `displayOrder`/sequence rather than trusting key
  order — keep this pattern for anything else that flattens groups/fields.
- **Storage**: JSON, filename is the sanitised **display name** (not an id) in
  `<dataFolder>/displays/`. Because the filename is name-based, renaming a display
  effectively creates a duplicate file rather than renaming in place — this differs from
  data definitions, which rename cleanly since their filename is `definitionId`-based.

[Screenshot: Display Editor grid with a group and a couple of fields]

## Project file (`.specimate`)

The saved state of one project.

- Built from `smGetPref("project")` (the full `project.*` prefs sub-tree) and written as
  JSON via `smWriteDataFile`.
- On open (`smProject.open`), every key in the file is fed back through `smProject.set`,
  repopulating the same `project.*` prefs tree.
- Contains an **embedded display snapshot** — enough of the current display definition to
  restore curation without a separate lookup — plus general project keys such as
  `project.name`, `project.folder`, `project.filename`.
- **Does not contain** the OCR/NER processing data itself. Those are saved as separate
  binary files alongside the project (see below); the project file only records which
  files/paths belong to it.
- A `/specimate` subfolder is created under the project's image folder to hold these
  project-local files.

## Processing data (OCR / Translation / NER)

The actual extracted content, as distinct from the definitions that describe it.

- **In memory**:
  - `sImgDataA` — keyed by image name; holds the raw OCR response (Google's JSON
    structure, several response-shape variants have existed over time — see
    `smGetImageFullText` for the fallback chain) and, once translated, a
    `["translated"]["data"]["translations"]["1"]["translatedText"]` entry.
  - `sData.NER` — keyed by specimen; holds the extracted entity fields per specimen
    after NER.
  - `sData.Changed` (`smDataChanged`) — per-datatype dirty flag (`"OCR"`, `"NER"`) that
    gates whether a save is needed.
- **Accessors**: `getImgDataA()` / `saveImgData` for OCR; `getData.NER()` / `saveData.NER`
  for NER. Always go through these rather than touching the script-locals directly, so
  the dirty flag stays accurate.
- **On disk**: saved as **binary `arrayEncode` files** (`.bin`), not JSON — a deliberate
  choice (`smSaveData.OCR`, `smSaveData.NER`) after past problems decoding JSON text
  containing irregular characters from OCR'd text. TSV export (`smExport.OCR`,
  `smExportRawData.NER`) is a separate, one-way output path for sharing/inspection, not
  used for reload.
- **Location**: alongside the project, referenced by path in the project prefs
  (`project.file.OCR` etc. — see todo notes on this key sometimes being cleared/reset).

## Preferences (`gPrefs`)

- Loaded/saved as JSON via `smPrefsFilePath` (fixed filename, not per-project).
- Holds: chosen SpeciMate-Data folder path, window positions (per-stack rect, validated
  against `the screenRects` on restore via `smRectOnScreen`), the `project.*` sub-tree
  used to build `.specimate` files, service/testing flags.
- **Known issue**: a `Lang` vs `Language` key mismatch between the prefs reader and an
  older setter — not yet fixed. Check both keys if language prefs seem to silently reset.
- Corrupted prefs files are renamed to `.bad` on load failure rather than overwritten,
  so a bad file can still be inspected afterwards.

## Folder layout summary

| Location | Contents |
|---|---|
| `<SpeciMate-Data folder>/definitions/` | Data definitions, JSON, filename = `definitionId` |
| `<SpeciMate-Data folder>/displays/` | Display definitions, JSON, filename = sanitised display name |
| `<SpeciMate-Data folder>/services/` | Service configs |
| `<project image folder>/specimate/` | `.specimate` project file, OCR/NER `.bin` data files |

See `services.md` for service record structure, and `pipeline.md` for how processing
data moves from capture through to export.
