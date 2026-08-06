# SpeciMate — User Guide
Create date: 2026-07-27
Last modified date: 2026-07-27

The complete guide to using SpeciMate, from first run to exported dataset.

Related guides:

- **Quick Start Guide** — a one-page tour of the main window.
- **Setting Up Services** — obtaining API keys and configuring providers.
- **User Guide: Choosing and Using Definitions** — picking and building definitions.
- **Admin Guide: Prompt and Display Definitions** — setting up definitions for others.

---

## 1. What SpeciMate does

SpeciMate turns a folder of specimen label images into a checked, structured dataset.

It does this in three machine steps and one human step:

| Step | What happens |
|---|---|
| **OCR** | Text is read off each image |
| **Translation** *(optional)* | Non-English text is translated to English |
| **Entity extraction (NER)** | An AI model pulls named fields out of that text |
| **Curation** | You review, correct and approve each record |

The result is exported as a tab-separated (TSV) dataset.

SpeciMate does **not** perform OCR, translation or extraction itself. It sends your images and text to external web services that you supply and pay for. See §3.

---

## 2. Concepts worth knowing before you start

**Project.** A project ties together an image folder, a definition, and the data extracted so far. Project data lives in a `specimate` sub-folder inside your image folder, so a project travels with its images.

**Definition (prompt definition).** Tells the AI *what to extract*: a **Role**, a **Prompt**, and a list of **Columns**. Usually you pick one from the library rather than writing one.

**Display.** Tells SpeciMate *how to lay the fields out* on the curation screen. One definition can have several displays — a full layout, a quick-check layout, and so on. Switching display never changes your data.

**Service.** A record describing one external API — its type, URL, model, key and payload template. You need at least an OCR service and an LLM service.

**SpeciMate-Data folder.** The shared folder holding definitions, displays and services. SpeciMate asks you to choose or locate it on first run.

---

## 3. First run

1. **Choose your SpeciMate-Data folder.** SpeciMate prompts for it at startup. If your institution already has one, point at it — you then inherit its definitions and services. Otherwise accept the default location and start empty.

   ```
   SpeciMate-Data/
     definitions/    prompt definitions
     displays/       display layouts
     services/       one file per configured API service
   ```

2. **Set up your services.** You need accounts and API keys from the providers themselves — typically Google Cloud (Vision for OCR, Translation) and an LLM provider (OpenAI, Azure OpenAI, Gemini). Enter the endpoint and key on the **Services** screen. Full instructions are in the *Setting Up Services* guide.

   [Screenshot: Services screen with a configured OCR and LLM service]

3. **Check your images.** Very large images cost more and can be rejected by some APIs. The **resize** button on the main window makes a resized copy of a whole folder — see §5.1.

To change the data folder later, use the **Choose SpeciMate-Data folder** option. If the folder ever goes missing at startup, SpeciMate asks you to locate it again.

---

## 4. The main window

The main window is a numbered sequence. Work down it.

| Step | Purpose |
|---|---|
| **1. Project Setup** | Create or open a project; point at the image folder |
| **2. Select Processes** | Choose which steps to run, and which model |
| **3. Prompt Details** | Choose or build the definition |
| **4. Process Images** | Run the batch and watch progress |
| **5. Data Curation** | Open the curation windows and correct the results |
| **6. Data Check and Export** | Verify and write out the dataset |

[Screenshot: main window, whole, with all six steps visible]

### 4.1 Step 1 — Project Setup

- **New Project** — name it, then choose the folder holding your images. SpeciMate
  creates a `specimate` sub-folder there for project data. Names must be unique within
  the folder.
- **Open Project** — continue an existing project.
- **Image folder** — the source folder. Everything else is derived from it.
- **Save Project** / **Close Project** — as expected. Closing clears the window.

The project remembers its definition, its display, and where its OCR and extraction
data files are. It does **not** contain the extracted data itself — that sits in
separate files alongside, so you can keep more than one set of results for the same
images.

[Screenshot: Step 1 row with a project open — project name, image folder, definition type]

### 4.2 Step 2 — Select Processes

[Screenshot: Step 2 with OCR and Extract Entities ticked and a model selected]

Tick the steps you want:

- **OCR?** — read the text from the images.
- **Translate?** — translate the OCR text.
- **Extract Entities?** — pull the defined fields out of the text.

You can run these separately. A common pattern is OCR the whole batch first, check the
text quality, then run extraction.

Then set:

- **Select Model** — which configured LLM does the extraction.
- **Multimodal?** — send the *image* to the model rather than the OCR text. Useful for
  layouts OCR handles poorly; needs a multimodal service configured, and costs more.
- **Use translation?** — extract from the translated text rather than the original.
- **Exclude some text from entity extraction?** — leave out text you know is noise
  (barcodes, accession stamps).

### 4.3 Step 3 — Prompt Details

[Screenshot: Step 3 with Role, Prompt and Columns filled from a library definition]

Three fields — **Role**, **Prompt**, **Columns** — plus a button to fill them:

- **Edit Prompt** — adjust the definition currently loaded into this project.

Definitions are covered properly in the *User Guide: Choosing and Using Definitions*.
Two points worth repeating here:

- **Apply to Project** changes only your project. **Save to Library…** changes the
  shared copy that everyone sees.
- **Locked** definitions cannot be overwritten. Duplicate one if you need a variant.

[Screenshot: definition library, a row selected, showing Role/Prompt/Columns beneath]

### 4.4 Step 4 — Process Images

Click **Process images**.

[Screenshot: Step 4 mid-run — file list on the left, progress bars part-filled, OCR result below]

- The **file list** on the left shows the images. Use **Filter** to narrow it and
  **Show** to switch between all files, processed files and unprocessed files — the
  quickest way to find what a failed run missed.
- The **Progress** bars on the right track OCR, Translation and Extract independently.
- The **OCR result** panel below shows the text read from the selected image.
- The status line at bottom left reports what is happening.

**Interruptions and errors.** If a provider rate-limits SpeciMate, processing pauses
and resumes automatically after a backoff, and the affected image is re-queued. Server
errors are treated the same way with a shorter wait. Other errors — a bad key, a
malformed request — are logged and *not* retried, because they will not fix
themselves. If a run appears to have stalled, check the **Log** window first.

**Re-running.** Processing an image again overwrites its previous result for that
stage. Use the **Show unprocessed** filter to run only what is missing rather than
paying to redo the whole batch.

---

## 5. Working with images

### 5.1 Resizing a folder

The **resize** button on the main window makes reduced copies of every image in a
folder.

1. Click it and enter a maximum dimension in pixels (2048 is the default and is
   usually plenty for label text).
2. Choose the source folder, then a **different** target folder.
3. Optionally choose a resize quality — *normal* is the default and is fast; *good*
   and *best* are slower but sharper.

Images already smaller than the limit are copied unchanged. The run reports how many
were processed, skipped and failed. **Click the button a second time to stop** — the
current image finishes, then the run halts cleanly.

Supported types: JPG, JPEG, PNG, GIF, BMP, TIF, TIFF.

[Screenshot: maximum-dimension prompt, and the status line during a resize run]

### 5.2 The image window

During curation the image window shows the current specimen. Clicking into a metadata
field tries to locate that text in the OCR result and highlight it on the image, so you
can check a value against the label without hunting for it.

[Screenshot: image window showing a specimen label, text highlighted]

---

## 6. Step 5 — Data Curation

This is where most of your time goes. Click **Curation** to open the image and
metadata windows.

### 6.1 The metadata form

The form is built from the project's display definition, so its layout depends on which
display is active. Fields may be grouped under headings, coloured, full or half width,
single or multi-line, and some may be read-only.

[Screenshot: curation form with grouped fields, a coloured section heading, and the image window alongside]

**Moving between records.** Up and down arrow keys step to the previous and next
specimen. Tab moves between fields.

**Picklists.** Fields with a defined list of accepted values show a popup. You can
type into the field and pick from the keyboard; choosing a value moves on to the next
field automatically.

[Screenshot: picklist popup open on a field]

**Validation colours.** A field's background tells you about its value:

| Colour | Meaning |
|---|---|
| Pink/red tint | Required field empty, wrong type, or a value not allowed by a closed picklist |
| Amber tint | Value not in the picklist, but the picklist allows custom entries |

[Screenshot: form showing one required-empty field and one out-of-picklist field, colours visible]

**Map.** If the record has usable decimal latitude and longitude, a map appears showing
the point. Coordinates that are not numeric are flagged rather than plotted.

### 6.2 Marking records

- **Checked** — the record has been reviewed and approved. This is the flag that
  matters most: it is included in the export, and it prevents accidental
  re-processing.

SpeciMate records **who** edited a record and **when**, automatically, if set on the main screen (click the info icon at top right to see settings).

[Screenshot: Checked / Problems / Remarks controls, with EditedBy and EditDate shown]


### 6.4 Switching display layout

The display selector at Step 5 lists every layout attached to the current definition.
Pick one and the curation form redraws immediately. Your data is untouched; only the
arrangement, visibility and editability of fields change.

The selector is disabled for simple, manually built definitions, which only ever have one layout.

[Screenshot: display selector popup listing the layouts for the current definition]

You can also change layouts directly in the curation window from a popup menu at the top, but only if more than one display definition exists for this data.

---

## 7. Step 6 — Data Check and Export

### 7.1 The table view

The **Data Table** window shows all extracted records as a grid, so you can scan for gaps and outliers before exporting.

[Screenshot: Data Table window with several records loaded]

### 7.2 Output order

[Screenshot: output order field containing a mix of ordinary and @constant column specs]

The **output order** field defines exactly which columns are exported and in what
order — a comma-separated list of column names. Two special forms are available:

| Spec | Effect |
|---|---|
| `locality` | Ordinary column |
| `#verified` | Column headed `verified`, with the literal value `verified` in every row |
| `@yearExtracted=2026` | Column headed `yearExtracted`, with `2026` in every row |

`@name=value` is the preferred way to add a fixed column — an institution code, a
project identifier, a processing year. The first `=` separates name from value; any
further `=` characters stay in the value.

Your output order is saved with the project.

### 7.3 Exporting

Click **Export data** and choose a filename. SpeciMate writes tab-separated text
(UTF-8).

Notes on what you get:

- **imageName** is always included, whether or not you listed it.
- **Checked** is added as an extra column.
- A few fields are excluded automatically because they contain line breaks that would
  break a TSV file: `prompt`, `prompt.header`, `results`, `result-fulltext` and `role`.
- **Duplicate column headers stop the export.** If two specs resolve to the same
  header name, SpeciMate reports the clash rather than writing a malformed file. Fix
  the output order and try again.

[Screenshot: duplicate-header validation message]

There is a separate **export OCR text** action if you want the raw read text rather
than the extracted fields.

---

## 8. Other windows

| Window | Use |
|---|---|
| **Services** | Configure OCR, Translation, LLM and Multimodal services |
| **Prompts / Library** | Browse, filter, duplicate, lock and delete definitions and displays |
| **Data Editor** | Edit a definition's columns, types, formats and picklists |
| **Display Editor** | Edit a layout — order, visibility, width, grouping, colour |
| **Data Table** | Grid view of extracted data, output order, export |
| **Map** | Plot all geolocated specimens in the project |
| **Log** | Running record of what SpeciMate did — the first place to look when something goes wrong |
| **Control Panel** | Jump to any window; double-click an entry in the list |

There is no general preferences screen. The only setting you can change directly is the
**SpeciMate-Data folder** — use the choose-data-folder option when you need to point at
a different one. Everything else that persists (window positions, the last project, and
so on) is remembered automatically.

[Screenshot: Log window during a run]

---

## 9. Where your data lives

| Location | Contents |
|---|---|
| `<SpeciMate-Data>/definitions/` | Prompt definitions |
| `<SpeciMate-Data>/displays/` | Display layouts |
| `<SpeciMate-Data>/services/` | Service configurations, including API keys |
| `<image folder>/specimate/` | The project file, plus OCR and extraction data files |

Two consequences worth knowing:

- **Back up your image folder and your SpeciMate-Data folder separately.** Neither
  contains the other.
- **Sharing a project means sharing the images folder.** If the recipient does not
  have the matching display in their SpeciMate-Data folder, SpeciMate falls back to the
  copy stored inside the project file, so the layout still works.

API keys are stored in the services folder in plain form. Treat that folder as
sensitive and do not put it in a shared drive that others should not read.

---

## 10. Troubleshooting

**Nothing happens when I click Process images.**
Check that a project is open, an image folder is set, at least one process is ticked,
and a definition is loaded. The Log window will usually say which is missing.

**Processing stops partway through.**
Most often a rate limit — SpeciMate pauses and resumes on its own, so give it a minute.
If the Log shows repeated 4xx errors, the key, URL or model name is wrong; those are
not retried.

**All the extracted values are empty or nonsense.**
Look at the OCR result panel first. If the OCR text is poor, no amount of prompt
tuning will help — try better images, or Multimodal extraction. If the OCR text is
fine, the Prompt and Columns are the thing to adjust.

**A field I expect is not on the curation screen.**
It is probably hidden in the active display. Switch display, or use Toggle Visible in
the Display Editor. Hidden fields still hold data and still export.

**My column order is wrong in the export.**
The output order field at Step 6 controls export order, independently of the order in
the definition.

**The export failed with a duplicate-header message.**
Two entries in the output order produce the same header — often an ordinary column and
a `#` or `@` spec with the same name. Rename one.

**A definition will not save.**
It is probably locked. Duplicate it and edit your copy.

**My changes did not reach my colleagues.**
You used **Apply to Project** instead of **Save to Library…**.

---

## 11. A typical session, end to end

1. Resize the image folder if the originals are large.
2. New Project → name it → point at the images folder.
3. Step 3 → **Library** → select the definition for this material → **Apply**.
4. Step 2 → tick **OCR?** → **Process images**. Check a few OCR results.
5. Step 2 → untick OCR, tick **Extract Entities?** → **Process images**.
6. Step 5 → **Curation**. Work through the records, correcting values and ticking
   **Checked** as you go. Use **Problems** for anything you cannot resolve.
7. Step 6 → set the output order, add any constant columns → **Export data**.
8. Save the project.
