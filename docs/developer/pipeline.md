# SpeciMate — Pipeline
Create date: 2026-07-20
Last modified date: 2026-07-20

How a batch of images actually gets processed, from kick-off to export. Can be run
during Setup or Curation (see `architecture.md`) — the mechanics here are the same
either way.

## Stages

Image load → OCR → Translate (optional) → NER/entity extraction → curation (manual) →
export.

Each stage is optional except image load — a run might be OCR-only, OCR+NER with no
translation, plain-LLM with no OCR at all, and so on, controlled by `sRunProcess`
(`sRunProcess["Translate"]`, `sRunProcess["NER"]`, etc.).

## Kicking off a run

- `smProcessAsync` is the entry point — validates there are files to process and an API
  key where required, then calls `initializeProcessing`.
- `initializeProcessing` refuses to start if any of `sActiveRequests.OCR` /
  `.Trans` / `.NER` is already `> 0` — you can't start a second run while one is active.
  It resets per-stage counters and indexes (`sCurrentIndex.OCR/.Trans/.NER`), sets tsNet
  timeouts, and builds the API/service details for the run.

[Screenshot: Process step showing progress bars for OCR/Translate/Extract]

## Async model

Requests are fired per-image, tracked per stage:

- `sActiveRequests.OCR` / `.Trans` / `.NER` — count of in-flight requests for that stage.
- `sCurrentIndex.OCR` / `.Trans` / `.NER` — position in the file list for that stage.
- `sImagesToProcess` — the file list for the run; `sNER.images.to.process` — a
  secondary queue specifically for images waiting on NER after OCR/translate finished
  (used because rate limiting or ordering can mean NER trails behind the other stages).

`processImage` is the driver — called repeatedly (after each response, and on resume
from a pause) to push the next file(s) through whichever stage is next for it, up to
each stage's configured `maxRequests` concurrency limit.

**Known fragile spot**: the flag that marks "processing" isn't reliably cleared when a
pipeline branch is skipped entirely — e.g. a plain-LLM run with no OCR, or OCR+NER with
no translation. If you're chasing a stuck-processing bug, trace the completion check
across each branch combination rather than assuming one central "done" handler covers
all of them (see `specimate_prioritised_todo.md`, item 1.2).

## Rate limiting and retries

- `smRateLimitPause pDelay` — pauses all sending. Uses exponential backoff
  (`sBackoffSeconds`, doubling, capped at 300s) combined with any `Retry-After` value
  from the response. Only schedules one resume even if called multiple times while
  already paused.
- `smResumeAfterRateLimit` — clears the pause flag and calls `processImage` to continue.
- `smServerErrorPause` — shorter, fixed backoff for 5xx errors.
- Response handling by status:
  - **429** → re-queue the image (`smRequeueImage`, pushed to front of the NER queue) and
    pause.
  - **5xx** → re-queue and pause (shorter backoff).
  - **Other 4xx** → logged only, *not* re-queued — these need user intervention (bad
    key, bad request) and won't self-resolve.
  - **200 with a rate-limit error in the JSON body** — some APIs report this way even on
    success status; checked explicitly (`error.code`/`error.type` containing
    `rate_limit` or `resource_exhausted`) and treated the same as a 429.
- `sRateLimitPaused` is checked at the top of `processImage` before sending NER
  requests — a paused run does nothing until resumed, so if a run looks stuck, check
  this flag before assuming a code bug.

## Reading processed data mid-pipeline

`smGetImageFullText` / `.Translated` (see `data-model.md`) are how later stages read the
output of earlier ones — e.g. NER reads OCR text via these rather than touching
`sImgDataA` directly. `smTranslationNeeded` decides whether an image's OCR text actually
needs translating before it's queued.

## Export

Once curation is done, data is exported from the Tabular stack.

- `smArrayToTable` builds the export table from `sData.NER`, given an ordered column
  spec, exclusion list, and extra columns.
- **Output order spec** (`outputOrder` field) is a comma-delimited list of column specs,
  each normally just a column name, but two special forms are supported:
  - `#foo` — **fixed string marker**: output column header is `foo`, and every row gets
    the literal value `foo`.
  - `@yearExtracted=2026` — **constant column marker**: output column header is
    `yearExtracted`, every row gets the fixed value `2026`. This is the preferred way to
    add a fixed-value column (the `#` form only supports the header text itself as the
    value). The first `=` splits name from value; further `=` characters in the value
    are preserved as-is.
- Duplicate output headers are caught before export (`smValidateOutputColumnSpecs`); a
  validation error stops the export rather than silently producing a malformed file.
- Excluded columns: certain fields (`prompt`, `prompt.header`, `results`,
  `result-fulltext`, `role`) are excluded by default from tabular export since they can
  contain multi-line text that would break TSV without further escaping.
- The `outputOrder` field's content is saved back to the project definition
  (`smProject.definition.set.prop "outputOrder"` on `closeField`, or explicitly via the
  Save button) — so it persists with the project, not just the open Tabular window.

[Screenshot: Tabular export screen with outputOrder field]

See `services.md` for what happens inside a single OCR/NER request, and `data-model.md`
for where the results end up.
