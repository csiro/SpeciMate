# SpeciMate — User Guide: Choosing and Using Definitions
Create date: 2026-07-27
Last modified date: 2026-07-27

How to pick a definition for your project, and how to make your own when nothing
suitable exists. See the **Quick Start Guide** for a tour of the main window, and the
**Admin Guide** if you are setting up definitions for other people.

---

## 1. What a definition does

Before SpeciMate can extract anything, it needs to know **what to look for**. That is
what a definition holds:

- **Role** — the persona the AI adopts ("You're an insect collection curator.").
- **Prompt** — the instructions it follows.
- **Columns** — the fields you want back.

Alongside it sits a **display** — the on-screen layout used when you curate the
results. Every definition has at least one; some have several, so you can switch
between, say, a full layout and a quick-check layout without changing the data.

You do not have to build either from scratch. Most of the time you will pick a
ready-made definition from the library.

---

## 2. Three ways to get a definition

| Situation | Use |
|---|---|
| Someone has already set up a definition for this kind of material | **The library** (§3) |
| One-off batch, unfamiliar material, no existing definition | **Analyse Example Images** (§5) |
| You know exactly which fields you want and want to type them in | **Build Manually** (§6) |

Start at Step 1 of the main window: create or open your project and point the **Image
folder** at your images. Then go to Step 3, **Prompt Details**.

---

## 3. Using a definition from the library

This is the normal route.

1. In Step 3, click **Library**.
2. The list shows everything available. Each row gives the name, when it was created,
   the type, how many columns it returns (**# Cols**) and how many display layouts it
   has (**# Displays**). Use the **Filter** box at the top right to narrow a long list.
3. Click a row — its Role, Prompt and Columns appear underneath so you can check it is
   the right one.
4. Click **Apply** (the green tick in the icon strip down the left side).

The Role, Prompt and Columns fields on the main window fill in, and the definition
becomes part of your project. You are ready to process.

**Locked definitions.** A tick in the **Locked** column means the definition is
protected — usually an institutional standard. You can use one freely; you just cannot
overwrite it. If you need a variation, select it and use the duplicate icon, then edit
your copy.

### Choosing a display layout

If the definition has more than one layout, the **display selector** at Step 5 lists
them. Pick one and it is applied immediately; if the curation window is open it
redraws straight away.

Switching layout never changes your data — only what you see and which fields are
editable. You can switch back and forth mid-project as often as you like.

> The selector is greyed out for definitions built with **Build Manually**, which only
> ever have one layout. See §7 if you need more.

---

## 4. Ad-hoc projects

Two of the three routes on the **Create a Definition** screen exist for work that does
not fit an existing definition — a small batch of unusual material, a trial run, or a
collection nobody has set up yet:

- **Analyse Example Images** — *AI drafts the definition from sample images.*
- **Build Manually** — *Define columns and the extraction prompt yourself.*

(The third, **Import a template**, is for institutional template spreadsheets and is
covered in the Admin Guide.)

Both create a definition that belongs to **your project only**. Nothing is added to
the shared library unless you explicitly choose **Save to Library…**, so you cannot
disturb anyone else's setup by experimenting.

To reach them, click **New** in the library, then pick a route.

---

## 5. Analyse Example Images — let the AI propose the fields

Best when you are not sure what fields the material can support, or you want a
sensible starting point fast.

### How it works

You give SpeciMate a couple of example images and some context. It sends them to a
multimodal AI model and asks it to propose a set of fields, a Role and a Prompt — then
builds a definition from the answer.

### Steps

1. Choose **Analyse Example Images**. The **Auto Mode Setup (AI Analysis)** card opens.
2. Enter a **Definition Name**.
3. Click **Browse…** beside **Example Images** and choose **one to three** examples.
   - Pick images that are *typical* of the batch, not your best or worst.
   - If images are already listed, you are asked whether to **Add** or **Replace**.
4. Set **Images Type** from the popup — this tells the model what it is looking at.
5. Add **Hints (optional)**. This is the most useful thing you can do. Say what
   matters to you and what to call it:

   > *Extract data items into meaningful natural history collections related columns.
   > Mostly handwritten labels, some in French. I need collector, collection number,
   > date, locality and habitat. Use Darwin Core field names where they apply. Ignore
   > the barcode and accession stamps.*

6. Click **Analyse**. The status line at the bottom left reports progress; this takes
   a few moments while the images are sent away and analysed.

When it finishes, the status line reads something like *"Analysis complete - 12
columns found."* and the **Data Editor** opens so you can see what came back.

> If it says **0 columns found**, the model returned nothing usable. Check that a
> multimodal service is set up on the Services screen, then sharpen your hints and try
> again.

### Review before you use it

This produces a **draft**. Always check it in the Data Editor:

- **Field names** — the model tends to be verbose or inconsistent. Rename anything
  awkward now, before you process a whole batch.
- **Missing fields** — add anything it overlooked with the **+** icon.
- **Extra fields** — remove anything you will never fill in with **−**; each one costs
  you screen space and prompt length.
- **Column order** — use the **▲ ▼** arrows. This sets your exported column order.
- **Role and Prompt** — read them. They are what the AI will actually be told.

A default layout is generated automatically, so you can process and curate straight
away once you are happy. Click **Apply to Project** when you are.

### Tips

- Two or three images covering the variation in the batch work better than one.
- If the result is poor, adjust the hints and **Analyse** again rather than fixing
  dozens of fields by hand.

---

## 6. Build Manually — type it in yourself

Best when you already know what you want back and just want to get on with it. Opens
the **Simple Mode Setup** card.

### Steps

1. **Definition Name** — required.
2. **Role** — optional but worth setting. One line describing who the AI should be:

   > *You're an insect collection curator.*

3. **Prompt** — the instructions. Be specific about what to do with awkward cases:

   > *Extract the following columns from the image. Return JSON text only. Return an
   > empty string for anything you cannot read with confidence — do not guess or infer
   > values that are not written on the label. Give dates as YYYY-MM-DD.*

4. **Column Names** — one field name per line. These become the keys the AI returns
   and the column headings you export:

   ```
   Species name
   Collector names
   Collection date
   Locality
   State
   Country
   Habitat
   Accession number
   ```

   At least one column is required.

5. Then choose:
   - **Use in project** — applies the definition immediately and fills in the main
     window. Fastest route; you go straight to processing.
   - **Create Full Defn** — builds it and opens the **Data Editor**, where you can add
     types, formats and picklists, and save it to the library if you want to keep it.

### What this route does not give you

It is deliberately basic. Straight from **Use in project** there are no field types,
formats, picklists or groupings, and only one display layout. If you find yourself
wanting drop-down lists of accepted values, required-field checking, or several
layouts, use **Create Full Defn** instead — see §7.

### Tips

- Keep field names short and consistent; they appear as labels during curation.
- Column order here is your export order.
- Fields named things like *Notes*, *Habitat*, *Locality*, *Description* and
  *Remarks* are automatically given wider, multi-line boxes on screen.
- Already typed a prompt into the main window and want to keep it? Open this card and
  copy-paste it across.

---

## 7. Growing an ad-hoc definition

Two things you can do once a quick definition has proved its worth.

**Create Full Defn.** From the Simple Mode Setup card, this converts your basic
definition into a full one and opens the Data Editor, where you can add types,
formats, picklists and other per-field detail — and use multiple display layouts.

**Save it to the library.** From the Data Editor, click **Save to Library…**. It then
appears in the library for future projects, and for other people if you share the
SpeciMate-Data folder. Give it a name that will still make sense in six months.

---

## 8. Saving: project versus library

Two separate buttons, and the difference matters.

| Button | Writes to | Use when |
|---|---|---|
| **Apply to Project** | Your project only | Tweaking a definition for this batch |
| **Save to Library…** | The shared library | You want to reuse it, or share it |

Editing your project's copy does **not** change the library version, so you cannot
break a shared definition by adjusting it for your own work. Equally, changes you
make to your project will not reach anyone else until you save to the library.

If you close an editor with unsaved changes, you are asked whether to save, discard,
or stay.

---

## 9. Adjusting the layout

If you want to change what appears on the curation screen — not what is extracted —
click **Display Editor** in the Data Editor. A menu pops up listing the layouts
already attached to this definition, plus **Add New Display**.

In the Display Editor each row is one field, top to bottom in the order it appears on
screen. The controls you are most likely to want:

- **▲ Move Up** / **▼ Move Down** — reorder fields.
- **Toggle Visible** — hide a field you never fill in. It stays in the list here, so
  you can bring it back.
- **Toggle Span** — `1` is half width, `2` is full width across the form.
- **Render Hint…** — `single-line`, or `multi-line,N` for a taller box.
- **Toggle ReadOnly** — show a field but stop it being edited.
- **Preview** — see the form as curators will get it, before committing.
- **Refresh** — pull in changes made to the definition's columns.
- **Data Editor** — go back to the columns view.

Finish with **Apply to Project**, or **Save to Library…** to keep the layout for
future projects.

To base a new layout on an existing one without changing it, use **Duplicate** in the
displays library and edit the copy.

---

## 10. After the definition: the rest of the run

Briefly, so you know what comes next:

1. **Step 2** — tick OCR, Translate, Extract Entities as needed, and pick your model.
2. **Step 4** — click **Process images** and watch the progress bars.
3. **Step 5** — curate. The form is laid out by your display: required fields,
   drop-down lists and defaults all come from the definition.
4. **Step 6** — check and export. Columns come out in the definition's column order.

See the Quick Start Guide for more on these.

---

## 11. Troubleshooting

**The display selector at Step 5 is greyed out.**
Your definition was built with **Build Manually** and has only one layout. Use
**Create Full Defn** to get the full structure.

**A field I added to the definition isn't on the curation screen.**
Displays don't pick up new fields automatically. Open the display, click **Refresh** —
new fields are added but hidden — then **Toggle Visible** on the one you want and save.

**I reordered fields on screen but the export is in the old order.**
Export order comes from the definition's column order, not the layout. Change it in the
Data Editor with the **▲ ▼** arrows.

**It says the definition is locked.**
It is a protected, shared definition. Select it in the library and use the duplicate
icon to get an editable copy of your own.

**Analysis came back with 0 columns found.**
Check that a multimodal service is configured and working on the Services screen. If
the service is fine, sharpen your hints and try again with different example images.

**A field shows "(ungrouped)" in the Display Editor but looks grouped on the form.**
Known display quirk when a group has no heading. The layout itself is fine.

**I can't put a field back into the group I removed it from.**
Known limitation when the group has no heading. Give groups a label before moving
fields between them, or use **Set Group…** → new group and rebuild it.

**The # Displays count looks wrong.**
The count can lag just after you save a display. Refresh the list.
