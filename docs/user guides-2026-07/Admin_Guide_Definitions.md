# SpeciMate — Admin Guide: Prompt and Display Definitions


For the person who sets up definitions that other people will use. A separate
**User Guide** covers day-to-day curation.

---

## 1. The two definitions

SpeciMate separates **what** it extracts from **how** it is shown.

| | Prompt (data) definition | Display definition |
|---|---|---|
| Answers | What fields exist, and how the AI should extract them | How those fields are laid out on the curation screen |
| Identity | `definitionId` | `displayId` |
| Points at | — | its parent definition, via `definitionId` |
| Stored in | `<SpeciMate-Data>/definitions/` | `<SpeciMate-Data>/displays/` |
| Filename | `def_<timestamp>.json` (id-based) | sanitised display **name** `.json` |
| Edited in | **Data Editor** | **Display Editor** |

The two editors are separate cards. Move between them with the buttons along the
bottom of each: **Display Editor** in the Data Editor, and **Data Editor** in the
Display Editor.

One prompt definition can have **many** display definitions — e.g. a full curation
layout, a cut-down "quick check" layout, and a read-only review layout. A display
belongs to exactly one prompt definition.

> **Rule of thumb:** if a change affects the data you get back from the AI or the
> exported dataset, it belongs in the prompt definition. If it only affects what a
> curator sees on screen, it belongs in the display definition.

---

## 2. How they are linked

Three ids do all the work.

```
prompt definition                    display definition
  definitionId: def_1748…    <--------  definitionId: def_1748…   (parent link)
                                        displayId:    disp_1752…  (own identity)

  columns[]                            groups[] -> fields[]
    fieldId: "locality"      <--------    fieldId: "locality"     (field link)
```

- **`definitionId`** links a display to its parent definition. The **# Displays**
  column in the library and the display popup both filter on this.
- **`fieldId`** links a display field to a data column. It is derived from the field
  name when the column is created, then never changes — so a column can be **renamed**
  without breaking any display. Never link on `name`. `fieldId` is visible as the
  **Fieldid** column in the Data Editor and the **fieldId** column in the Display
  Editor.
- **`displayId`** is the display's own identity. Files are named after the display
  *name*, but everything internal resolves by `displayId`, so renaming a display is
  safe.

---

## 3. Folders and the library

On first run SpeciMate asks for a **SpeciMate-Data** folder. It contains:

```
SpeciMate-Data/
  definitions/     prompt definitions (shown as "Prompts Folder" in the library)
  displays/        display definitions
  services/        one JSON file per configured API service
```

The library window shows the active folder path at the top, a **Filter** box, and a
grid with these columns:

| Column | Meaning |
|---|---|
| **Locked** | Tick = protected from overwrite and deletion |
| **Created** | Creation date |
| **Name** | Definition name |
| **Type** | Specimen/collection type |
| **# Cols** | Number of columns |
| **# Displays** | How many display layouts point at this definition |
| **mode** | `Simple`, `Template` or `Auto` |
| **fileName** | File on disk |

Selecting a row shows its Name, Role, Prompt and Columns underneath.

The icon strip down the left side carries the row actions — information, new,
edit, duplicate, **Apply**, lock/unlock, and delete.

**Library vs project copy — important.**

- **Library** = the shared, reusable definition on disk. Written by
  **Save to Library…**.
- **Project copy** = a snapshot inside the `.specimate` project file, written by
  **Apply to Project**.

Editing a project copy does **not** change the library, and vice versa. When a project
is opened, SpeciMate re-resolves the project's display from the library by `displayId`
and adopts the current library version if found; if the display is missing from the
library (e.g. a project shared without the displays folder), the embedded copy is kept.

**Locking.** Locking a definition or display blocks overwrite and deletion. Users can
still duplicate it to make their own editable fork with a new id. This is the
recommended way to publish an institutional standard.

---

## 4. Creating a prompt definition

From the library, choose **New**. The **Create a Definition** screen offers three
routes:

| Button | Card it opens | Use for |
|---|---|---|
| **Analyse Example Images** | Auto Mode Setup (AI Analysis) | Drafting a definition from sample images |
| **Import a template** | Template Mode Setup | An institutional template spreadsheet |
| **Build Manually** | Simple Mode Setup | Typing Role, Prompt and column names directly |

All three ask for a **Definition Name** first.

### 4.1 Build Manually (Simple mode)

Definition Name, **Role**, **Prompt**, and **Column Names** — one name per line.
Fast, but limited:

- No types, formats, picklists or categories.
- Only **one**, auto-generated display; the display selector on the main window is
  disabled for simple definitions.
- **Use in project** applies it straight away; **Create Full Defn** builds it and
  opens the Data Editor so you can add the richer structure.

Suitable for ad-hoc or exploratory work, not for a shared institutional setup.

### 4.2 Import a template (recommended for admins)

Two files, selected with the **Browse…** buttons beside **Template Fields TSV** and
**Picklist Values TSV**, then **Parse**.

The expected workflow is to maintain the definition as a spreadsheet and export two
sheets:

- **`Template notes`** → Template Fields TSV — the column definitions.
- **`PL values`** → Picklist Values TSV — picklist names and their values.

**TSV is strongly preferred** — the CSV parser does not handle quote characters inside
a field.

**Fields file** — headers may appear in any order and are matched case- and
whitespace-insensitively. Only `name` is mandatory; everything else degrades to a
sensible default.

| Header | Purpose |
|---|---|
| `name` | Field name (mandatory). Becomes the JSON key the AI is asked to return. |
| `category` | Grouping hint. Drives the **default display's** groups. |
| `required` | Accepts `x`, `(x)`, `y`, `(y)`, `yes`, `true` |
| `default` | Default value pre-filled during curation |
| `picklist` | `y` / `yes` / `true` — links to a picklist of the same name |
| `comments` | Free-text extraction guidance. Also used to infer type and format. |
| `format` | Explicit format; overrides anything inferred from `comments` |

> **There is no `type` column.** The **Type** shown against each field in the Data
> Editor is inferred from `comments`, with an explicit `format` taking precedence.
> Put your type guidance in the `comments` text. (Older builds list `type` in the
> on-screen instructions; a `type` column is ignored.)

Rows whose `name` is empty or `NaN` are skipped.

**Picklists file** — one picklist per **column**. The header row is the picklist name;
values run down the column. Empty cells and `NaN` are skipped, as are columns named
`Unnamed…`. Picklists are matched to fields by name, with a trailing `[PL]` stripped,
falling back to case-insensitive and partial matching. All imported picklists allow
custom values by default.

On **Parse**, SpeciMate validates the definition (duplicate names, duplicate or empty
`fieldId`s) and refuses the import if it fails — fix the TSV and parse again.

It also **generates the extraction prompt** from the fields. That generated prompt
carries the role, a general instruction, and then per-field guidance only for fields
that have a default, picklist, comment or format — fields with nothing extra to say
are omitted to keep the prompt small. Picklist *values* are not listed by default.
Category is deliberately excluded, since it is a grouping hint, not extraction guidance.

### 4.3 Analyse Example Images (Auto mode)

Sends sample images to a multimodal model and asks it to propose fields, prompt and
role.

| Control | Notes |
|---|---|
| **Definition Name** | Required |
| **Example Images** + **Browse…** | One to three images; you are asked whether to **Add** or **Replace** if the list is not empty |
| **Images Type** | Popup describing what the model is looking at (e.g. `general`) |
| **Hints (optional)** | Free text — the most useful control on the card |
| **Analyse** | Runs the analysis; the status line reports progress and the column count found |

The status line reads e.g. *"Analysis complete - 12 columns found."* A result of
**0 columns found** means the model returned nothing usable — check the multimodal
service, then sharpen the hints.

Treat the result as a **draft**: review every column in the Data Editor before saving
to the library.

---

## 5. Editing a prompt definition — the Data Editor

The Data Editor shows the definition **Name** and its **Mode** (Simple / Template /
Auto), then:

- **Role** — the persona/system text sent as `[ROLE]`.
- **Prompt** — the instruction text sent as `[PROMPT]`.
  The Role and Prompt boxes are resizable by dragging the divider bars.
- **Columns** — count, plus the grid: **Name**, **Type**, **Req**, **Format**,
  **Default**, **Picklist**, **Fieldid**. Use the **▲ ▼** arrows to reorder and the
  **+ −** icons to add or remove columns.
  **Column order here sets the exported dataset column order**, so get it right
  before users start exporting.
- **Picklists** — a summary line showing each picklist and its value count, e.g.
  *(1 defined): State (5)*.

### Buttons

| Button | Effect |
|---|---|
| **Edit Picklists…** | Opens the picklist editor |
| **Display Editor** | Pops up a menu of this definition's displays, plus **Add New Display** — see §6.2 |
| **Save to Library…** | Writes to the shared library |
| **Apply to Project** | Writes to the current project only |
| **Cancel** | Discards |

**Save to Library…** resolves by `definitionId`. Renaming a definition and saving to
library **overwrites in place**, so displays stay linked. A locked entry is refused —
duplicate it instead.

### The picklist editor

Left pane lists the picklists, with **+ −** to add and remove. For the selected
picklist:

- **Name**
- **Allow custom values (unbounded)** — ticked by default. Untick for a strict list,
  where curators may only choose an existing value.
- **Items** — the values, with **+ −** and **Sort**.

Finish with **Save Changes** then **Done**, or **Cancel**.

---

## 6. Creating and editing a display definition

### 6.1 The default display

Whenever a definition is created, a default display is generated automatically:

- One group per `category`, in the order categories first appear. Fields with no
  category go into a group called "Custom".
- Alternating group background colours.
- Group headings are only written when there is more than one group (and the
  category isn't "General") — a heading over a single-group form is just clutter.
- `span` and `renderHint` are inferred from the field name: wide, multi-line
  treatment for notes, habitat, locality, description and remarks; single-line
  otherwise.

For many setups the default is good enough. Customise when you need a specific
field order, hidden fields, or read-only fields.

### 6.2 Getting to the Display Editor

In the Data Editor, click **Display Editor**. A popup menu appears listing every
display already attached to this definition, plus **Add New Display** at the bottom.

- Choose an existing display to edit it.
- Choose **Add New Display** to generate a fresh default layout from the current
  columns.

Displays can also be opened from the displays library.

### 6.3 The Display Editor

The information line under the name reports the layout and field counts, e.g.
*Grid: 2 cols | Gap: 8px | Row gap: 6px | Label: 120px | Fields: 31 (4 hidden)*.

The grid is a flat, ordered list of every field — **Number**, **Field Name**,
**fieldId**, **Vis**, **Span**, **Render Hint**, **R/O**, **Group**, **bgColor**.
Row order **is** the on-screen order. Operations apply to all selected rows.

| Control | Effect |
|---|---|
| **▲ Move Up** / **▼ Move Down** | Reorder fields |
| **Toggle Span** | `1` = half width, `2` = full width |
| **Render Hint…** | `single-line` or `multi-line,N` (N = rows) |
| **Toggle Visible** | Show/hide on the curation form. Hidden fields stay in this grid |
| **Toggle ReadOnly** | Read-only on the curation form |
| **Set Group…** | Assign to an existing group, create a new one, or remove from group |
| **Delete Group** | Remove a group (its fields become ungrouped) |
| **Edit Group** | Rename the group's heading |
| **Apply Group** | Apply the group label/colour fields to the selection |
| **Pick Colour…** | Set the group's background colour |
| **↑ ↓ + −** (below the buttons) | Move, add and delete rows |

Bottom row:

| Button | Effect |
|---|---|
| **Preview** | Render the form live, as curators will see it |
| **Data Editor** | Go to the columns view for this definition |
| **Refresh** | Re-sync with the data definition — see §9 |
| **Save to Library…** | Write to the displays library |
| **Apply to Project** | Write to the current project only |
| **Cancel** | Discard |

**Layout defaults** (per display): 2 grid columns, 8 px field gap, 6 px row gap,
16 px section gap, 120 px label width.

A new display is assigned a fresh `displayId` on first save to the library. Because
the file is named after the display, renaming and saving moves the file and removes
the old one; a name clash with a *different* display prompts before overwriting.

**To fork a display** rather than overwrite it, use **Duplicate** in the displays
library. Duplicating gives the copy its own `displayId` and appends a suffix to the
name, so repeated duplication doesn't overwrite an earlier copy. There is no
Save As in the editor.

---

## 7. Applying a display to a project

Two routes:

1. **The display selector** on the main window (Step 5). It lists every display whose
   `definitionId` matches the project's current definition, plus the currently active
   display (which may have been designed for a different definition). Disabled for
   simple-mode definitions.
2. **Apply to Project** from the Display Editor.

Applying a display swaps the project's active display and, if the curation window is
open, rebuilds the form immediately.

---

## 8. What happens at run time

### Extraction (NER)

The service payload is a **template string with placeholders**, not a serialised
structure. Before each call:

| Placeholder | Filled from |
|---|---|
| `[ROLE]` | definition's Role |
| `[PROMPT]` | definition's Prompt |
| `[ENTITIES]` | pipe-delimited list of column names |
| `[TEXT_TO_BE_PARSED]` | the OCR'd (and optionally translated) text |
| `[MODEL]`, `[MAXTOKENS]`, `[BASE64_ENCODED_IMAGE]` | service record |

> **Admin caution:** because this is string substitution, an unescaped `"` or
> backslash in a Role or Prompt can break the JSON payload and cause the call to
> fail. Avoid raw double quotes in prompt text until per-value escaping
> (`smJSONEscape`) is applied everywhere; use single quotes for emphasis instead.

### Curation

The Metadata form is built from the data definition (field names, picklists,
defaults, required flags) **plus** the display definition (order, groups, spans,
render hints, visibility, read-only). A `span 2` field landing in the right-hand
column is pushed to a new row.

### Export

Export column order comes from the **data definition's column order**, not the
display. Reordering fields in the Display Editor does not change the exported file.

---

## 9. Common admin tasks

**Publish a standard definition for a team**
1. Build it with **Import a template** from reviewed TSVs.
2. Refine columns, picklists, Role and Prompt in the Data Editor.
3. Build and check the display(s) via **Display Editor**; use **Preview** before
   saving.
4. **Save to Library…** for both.
5. Lock the definition so users must duplicate to modify.
6. Distribute the `definitions/` and `displays/` folders together — a project shared
   without its displays falls back to its embedded snapshot.

**Add a field after users have started work**
1. Add the column in the Data Editor, **Save to Library…**.
2. Open each affected display and click **Refresh**. New columns are appended as
   **hidden**; removed columns are dropped; renamed columns keep their place with the
   new label. All other per-field settings are preserved.
3. **Toggle Visible** on the new field, position it, and save.

**Rename a field**
Rename the column only. `fieldId` does not change, so displays stay linked; run
**Refresh** on each display to pick up the new label.

**Fork a locked definition**
Duplicate it in the library. The copy gets a new `definitionId` — note that its
existing displays will **not** follow, since they point at the original's id. Duplicate
the displays too and re-point them if needed.

---

## 10. Known limitations

Worth knowing before they surprise you.

- **Removing a field from an unlabelled group is effectively one-way.** The
  **Set Group…** picker lists groups by label and skips unlabelled ones, so a field
  removed from an unlabelled group cannot be put back through the UI — only into a
  *new* group. Default displays commonly have unlabelled groups.
- **Two groups sharing a label collide.** The picker resolves a choice back to the
  first matching label, making the second group unreachable. Keep group labels unique.
- **The Group column can read "(ungrouped)" for fields that are in fact grouped**,
  where the group has no heading.
- **The # Displays count can be stale** immediately after saving a display. Refresh
  the list to correct it.
- **Simple-mode definitions support only one display.** Use **Create Full Defn** to
  get the full structure.
- **Sort order in the library grids is not preserved** across refreshes.
- **Export order follows the data definition**, not the display — a frequent source of
  "why didn't my reordering work?" questions.

---

## 11. Quick reference

| Thing | Value |
|---|---|
| Definitions folder | `<SpeciMate-Data>/definitions/` |
| Displays folder | `<SpeciMate-Data>/displays/` |
| Definition file name | `def_<timestamp>.json` |
| Display file name | `<sanitised display name>.json` |
| Parent link | display `definitionId` → definition `definitionId` |
| Field link | display field `fieldId` → column `Fieldid` |
| Save to shared library | **Save to Library…** |
| Save to this project only | **Apply to Project** |
| Fields TSV headers | `name` (required), `category`, `required`, `default`, `picklist`, `comments`, `format` |
| Picklists TSV | one picklist per column; header = picklist name |
| Span values | `1` (half width), `2` (full width) |
| Render hints | `single-line`, `multi-line,N` |
