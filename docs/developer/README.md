# SpeciMate — Developer Documentation
Create date: 2026-07-20
Last modified date: 2026-07-20

Reference documentation for developing SpeciMate: a LiveCode application for specimen
image curation and metadata extraction (OCR, translation, and NER via configurable
LLM/Multimodal services). This set is for developers — end users should start with
`Quick_Start_Guide.md` instead.

## Contents

1. **[Architecture](architecture.md)** — the map. Stack inventory, startup sequence, and
   the two main processes (Setup, Curation) that every project moves through.
2. **[Data Model](data-model.md)** — every structure that persists: data definitions,
   display definitions, the project file, prefs, and the OCR/Translation/NER processing
   data itself.
3. **[Pipeline](pipeline.md)** — how a batch of images is actually processed: the async
   request model, rate limiting and retries, and export/output-order rules.
4. **[Services](services.md)** — configuring and calling external APIs: service records,
   payload templates, the JSON-escaping rule, and provider differences.
5. **[Conventions](conventions.md)** — handler prefixes, naming, logging, and the
   numbered list of LiveCode/SpeciMate gotchas worth knowing before you edit anything.
6. **[Dev Setup](dev-setup.md)** — getting a working environment: LiveCode version,
   script library paths, the SpeciMate-Data folder, and API key handling.

## Reading order for someone new

Architecture → Data Model → Pipeline → Services → Conventions → Dev Setup. Conventions
is short and can be read any time; Dev Setup only matters once you're actually setting up
a machine.
