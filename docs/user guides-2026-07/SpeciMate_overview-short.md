# SpeciMate: A Collaborative Metadata Extraction Tool for Biological Specimen Collections
Create date: 2026-07-27
Last modified date: 2026-07-30

**Abstract**

The digitisation of biological specimen collections is crucial for preserving and providing access to invaluable scientific data. This paper describes SpeciMate, an easy-to-use application designed to improve metadata extraction from digitised specimen images through **human-AI collaboration**. SpeciMate is service-agnostic: it calls external AI services that the user configures, typically Google Cloud Vision for Optical Character Recognition (OCR), Google Cloud Translation for translation, and any OpenAI-compatible Large Language Model (LLM) endpoint — commercial, institutional or locally hosted — for metadata entity extraction. Extraction targets are described by user-editable **definitions**, and the curation interface is described by separate **display** layouts, so the application can be adapted to new material without code changes. The application facilitates an iterative workflow involving **AI-assisted processing** and **interactive human curation**, enhancing the efficiency and accuracy of digitisation efforts for biological collections.

**1. Introduction**

Natural history collections worldwide represent a vital record of biodiversity. The digitisation of these collections is increasingly important for research, education, and public engagement. Despite the significance of these collections, a substantial portion remains underutilised due to challenges in accessing and interpreting physical specimens and their associated records. The digitisation process typically involves imaging specimens followed by the extraction of metadata from specimen labels. Traditional methods of metadata extraction are often labour-intensive, time-consuming, and prone to error.

Artificial intelligence (AI) offers significant potential to enhance metadata extraction through technologies such as Optical Character Recognition (OCR), translation, and Named Entity Recognition (NER). SpeciMate is an application developed to leverage these AI capabilities within a **human-AI collaborative framework** to improve the efficiency and accuracy of metadata extraction from biological specimen images.

A design goal throughout has been to keep domain expertise, not code, in control of the result. What is extracted, how it is asked for, and how it is presented for checking are all data that a collection manager can edit, share and version — not compiled-in assumptions about a handful of specimen types.

**2. Application Description and Workflow**

SpeciMate is a multi-platform application developed using LiveCode. It is designed to be user-friendly and accessible on both Apple and Windows computers. The application facilitates a workflow divided into key stages: **project setup, definition selection or authoring, batch processing of images, and interactive curation of the resulting metadata**.

**2.1. Projects and shared configuration**

Work is organised into **projects**. A project ties together an image folder, the definition in use, the active display layout, and the extracted data produced so far. Project data is written to a `specimate` sub-folder inside the image folder, so a project travels with its images and can be handed to a colleague as a single folder.

Shared configuration lives separately, in a **SpeciMate-Data** folder holding three sub-folders: `definitions/`, `displays/` and `services/`. An institution can maintain one such folder so that all its users inherit the same agreed definitions and the same configured services. SpeciMate prompts for the location of this folder on first run.

**2.2. Definitions and displays**

Earlier versions of SpeciMate offered a fixed menu of specimen types, each wired to a hard-coded curation screen. This has been replaced by two separable, user-editable objects:

*   A **prompt (data) definition** describes *what* to extract: a **Role** (the persona the model adopts), a **Prompt** (the instructions it follows), and an ordered list of **Columns** — the metadata fields required, optionally with types, formats, default values, required flags, and picklists of accepted values.
*   A **display definition** describes *how* those fields are laid out for checking: order, grouping under coloured headings, half- or full-width fields, single- or multi-line boxes, visibility, and read-only status.

One definition may have several displays — for example a full curation layout and a cut-down "quick check" layout. Fields are linked between the two by a stable internal `fieldId`, so a column can be renamed without breaking any layout. Switching display never alters the underlying data, only what the curator sees and can edit.

Definitions and displays are held in a **library**, browsable and filterable, showing for each entry its name, type, number of columns and number of attached layouts. Institutional standards can be **locked** so that users may apply them freely but not overwrite them; a user needing a variant duplicates the locked definition and edits the copy. A clear distinction is drawn between **Apply to Project**, which affects only the current project, and **Save to Library**, which changes the shared copy that everyone sees.

**2.3. Three routes to a definition**

Where no suitable definition exists, three routes are offered:

*   **Use the library.** The normal route: select an existing definition, review its Role, Prompt and Columns, and apply it.
*   **Analyse Example Images.** The user supplies one to three representative images, an image type, and free-text hints describing what matters ("mostly handwritten labels, some in French; I need collector, collection number, date, locality and habitat; use Darwin Core field names where they apply; ignore the barcode"). A multimodal model is asked to propose a field set, Role and Prompt, from which SpeciMate builds a draft definition and a default layout. The draft is deliberately opened for review before use.
*   **Build Manually.** The user types a Role, a Prompt and one column name per line — the fastest route when the required fields are already known. Such a definition can later be promoted to a full definition with types, picklists and multiple layouts.

The AI-drafted route is itself a collaboration: the model proposes a vocabulary for unfamiliar material, and the curator prunes, renames and reorders it before a single specimen is processed at scale.

**2.4. Services**

Each external API is described by a **service record** created on the Services screen. There are four types — OCR, Translation, LLM (text) and Multimodal (image) — and any number of each may be defined, with one nominated as the default for its type.

A service record holds a name, type, endpoint URL, API key, HTTP headers, a request **template**, a model identifier, a token ceiling, and a maximum number of concurrent requests. The URL, headers and template are plain text containing placeholders — `[BASE64_ENCODED_IMAGE]`, `[TEXT]`, `[TEXT_TO_BE_PARSED]`, `[MODEL]`, `[ROLE]`, `[PROMPT]`, `[ENTITIES]`, `[MAXTOKENS]` and `[XXXX]` for the key — which are substituted immediately before each call. This template approach makes the application provider-agnostic: OpenAI, Azure OpenAI, OpenRouter, Google Gemini and locally hosted servers such as LM Studio or Ollama can all be used, and a new endpoint is usually a configuration change rather than a code change. **Check** validates a template as JSON, and **Test** issues a real request through the same substitution path used in production, reporting the response.

Services can be exported and imported for sharing, with API keys optionally stripped. Keys are otherwise stored with the service record, and the services folder should be treated as sensitive.

**2.5. Batch processing**

Three processes may be selected — **OCR**, **Translation** and **Extract Entities** — individually or together. When run together the correct order is enforced automatically; OCR must precede the others. Translation is invoked only where the OCR result reports a non-English, non-Latin language with reasonable confidence, so it costs nothing on English-only batches. A **Multimodal** option instead sends the image directly to a multimodal model, performing reading and extraction in a single call and bypassing OCR and translation.

Requests are issued asynchronously, several at a time, up to the per-service concurrency limit. Provider rate limits and transient server errors are handled by pausing, backing off and re-queuing the affected image; errors that will not resolve themselves — a rejected key, a malformed request, an unknown model — are logged and not retried. Progress is shown per process, the image list can be filtered to show processed or unprocessed files, and a Log window records the detail.

A built-in **resize** utility makes reduced copies of an entire image folder to a chosen maximum dimension and quality, since large images cost more, process more slowly, and are rejected outright by some APIs.

**2.6. Interactive curation**

Selecting **Curation** opens the image window and the metadata form. The form is generated from the active display definition, so its arrangement, grouping and colouring reflect the layout in use rather than any built-in screen.

Curation features include:

*   **Navigation:** up and down arrow keys move between specimens; tab moves between fields. Changes are saved automatically.
*   **Contextual highlighting:** clicking into a metadata field locates the corresponding text in the OCR result and highlights it on the specimen image, so a value can be checked against the label directly.
*   **Picklists:** fields with a defined list of accepted values present a popup; selecting a value advances to the next field.
*   **Validation feedback:** field background colour indicates a required field left empty, a value of the wrong type, a value outside a closed picklist, or a custom value permitted by an open picklist.
*   **Geographic feedback:** where a record carries usable decimal coordinates, its position is shown on a map. Non-numeric coordinates are flagged rather than plotted.
*   **Record status:** **Checked**, **Problems** and free-text **Remarks** flags record the outcome of review. SpeciMate stores who edited each record and when.
*   **Targeted re-processing:** a single poorly extracted record can be sent back to the model from the curation window. Records already marked **Checked** are protected from accidental re-processing.
*   **Layout switching:** the active display can be changed mid-project, redrawing the form immediately without touching the data.

**2.7. Whole dataset curation and export**

The **Data Table** window presents every extracted record as a grid, for scanning gaps and outliers before export. It supports sorting by column, filtering by text string, searching within a chosen column, running automated checks (for example ID/filename mismatch, which signals a misread accession number), and re-processing selected rows.

A **Map and Timeline** window plots selected records geographically and temporally, with linked selection between the two views, after the user maps dataset columns to the required fields.

Export writes tab-separated (TSV) text in UTF-8, ready for Excel, OpenRefine, the Specify workbench or a collection database importer. An **output order** field controls exactly which columns are written and in what sequence, and supports fixed-value columns — for example `@yearExtracted=2026` adds a column of that name with that value in every row — which is a convenient way to stamp an institution code, project identifier or processing year onto a dataset. Duplicate header names are detected and reported rather than written out. The raw OCR text can be exported separately.

**2.8. Version control and troubleshooting**

SpeciMate implements semi-automatic **version control** by writing OCR and extraction data to date-time-stamped files within the project's `specimate` folder. More than one set of results can therefore be kept for the same images, allowing the outputs of different models or different prompts to be compared directly.

For **troubleshooting**, a **Log** window displays messages and errors during execution, and can be filtered, cleared, saved, or emailed to the developer for support.

**3. Human-AI Collaboration in SpeciMate**

SpeciMate is designed to foster **effective human-AI collaboration** at three distinct points.

First, in **defining the task**. Where a definition already exists, human judgement is expressed in choosing and reviewing it; where none does, the AI proposes a field set from example images and the curator refines it. In both cases the machine's brief is inspected before it is issued at scale.

Second, in **prompt engineering**. Role, Prompt and Columns remain editable throughout, and because processing writes versioned output files, the effect of a prompt change on a subset of images can be measured rather than guessed at.

Third, in **curation**. Human knowledge of the collection, of handwriting, of historical place names and of taxonomic history is essential for verifying and correcting AI-extracted metadata. The interface is built to support that judgement — highlighting source text, validating against controlled vocabularies, plotting coordinates, and recording provenance — rather than to replace it. This approach leverages the speed of AI for initial extraction while relying on human expertise for accuracy and contextual understanding.

**4. Application to Biological Specimens**

Because definitions and displays are data rather than code, SpeciMate is not limited to a predetermined set of specimen types. It has been used for herbarium sheets, insect slides, insect types and tabular data such as survey forms, and new material is accommodated by authoring or adapting a definition. Institutions can publish agreed, locked definitions for their standard workflows while individual users retain the freedom to build ad-hoc definitions for one-off batches without disturbing the shared library.

**5. Conclusion**

SpeciMate provides a user-friendly platform for **AI-assisted metadata extraction** from digitised biological specimen images, underpinned by **effective human-AI collaboration**. By combining user-editable extraction definitions and display layouts, configurable service records that make the application independent of any single AI provider, a resilient asynchronous processing pipeline, and interactive curation tools, SpeciMate offers a practical solution for accelerating the digitisation of natural history collections and improving access to crucial biodiversity data. Its features for definition authoring, prompt engineering, batch processing, whole-dataset checking and flexible export empower collection managers and researchers to enhance both the efficiency and the accuracy of their digitisation efforts.

**References**
