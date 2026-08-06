# SpeciMate — Getting Started

Created: 2 July 2026
Last modified: 7 July 2026

Welcome to SpeciMate testing! This guide walks you through the main window, step by step, to get your first project running.

## Step 1: Project Setup

- **New Project** — start a fresh project.
- **Open Project** — load an existing `.specimate` project file.
- **Save Project** / **Close Project** — save your work or close the current project.
- **Select definition** — choose what type of data you're extracting. This determines which fields SpeciMate will look to extract.

## Step 2: Select Processes

Choose which processing steps to run on your images:

- **OCR** — extract text from the image.
- **Translate** — translate extracted text.
- **Extract Entities** — pull structured data (species name, date, location, etc.) into fields.
- **Use translation** — when extracting entities, use the translated text rather than the original.
- **Select model** – Select LLM to use when extracting fields using Prompt instructions.

You can select any combination depending on your images and goals.

## Step 3: Prompt Details

Fine-tune the instructions sent to the AI model:

- **Library** — pick a saved prompt from your prompt library.
- **New** — create a new prompt from scratch. Opens the Definition Wizard.
- **Edit Prompt** — review or adjust the current prompt before processing.

Most users can skip this step and use the default prompt for their chosen definition.

## Step 4: Process Images

- Click **Process images** to start. Progress bars show status for OCR, Translation, and Extraction.
- Use **Filter** and the **Show** dropdown to narrow down which images are displayed.
- **Sample** lets you select a small random batch to test settings on before running the full folder.
- **OCR result** expands to show the raw extracted text for the selected image.

## Step 5: Data Curation

Once processing is complete, use this step to review and correct extracted data image-by-image. Use the dropdown to switch between saved display layouts.

## Step 6: Data Check and Export

- The grid icon opens a table view of all your data for a final check.
- From the table view you can export your results to file.
- Use the dropdown to choose alternative methods of export (e.g. **LLM data**).

## Quick Start Checklist

1. New Project → select an image folder and a definition.
2. Select the processes you want (Step 2).
3. (Optional) Review the prompt (Step 3).
4. Click **Process images** (Step 4).
5. Curate results (Step 5).
6. Check and export your data (Step 6).

---

*Questions or issues while testing? Note down what you were doing and any error messages, and pass them along — this helps a lot.*
