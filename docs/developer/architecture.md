# SpeciMate — Architecture
Create date: 2026-07-20
Last modified date: 2026-07-20

The map. Read this first if you're new to the codebase.

## Two main processes

Every SpeciMate project moves through two broad phases:

1. **Setup** — creating/opening a project, choosing a data definition, building or
   selecting a display definition, configuring services and prompts. Mostly application
   admin: getting the project and its definitions into a state ready to work with.
2. **Curation** — reviewing and correcting the extracted metadata for each specimen,
   using the Metadata custom card, checking off fields, and eventually exporting.

**Processing (OCR / Translation / NER)** can be triggered from either phase — it isn't a
separate third stage. A user might process a batch during setup before curating, or
process individual images on demand while curating. See `pipeline.md` for how processing
itself works.

[Screenshot: main OCR window showing Setup controls]
[Screenshot: Metadata curation window]

## Stack inventory

| Stack | Role |
|---|---|
| OCR | Main window — project setup, process steps, image list, OCR window fields (`baseFolder` etc.) |
| Project Definition | Data definition editor (`sDataDef`) — fields, NER config, DefinitionPreview card (`dp*`) |
| Column Mapping | Reusable mapping UI — opens after caller sets `uColumnsRequired` / `uColumnsPossible` stack properties |
| Prompts | Prompt library (`pm*`) — save/load/list prompt definitions |
| Services | Service configuration (OCR, Translation, LLM, Multimodal) |
| Metadata | Curation custom card (`cu*`) — dynamically built fields/checkboxes per specimen |
| Tabular | Table view/export of NER data, output column ordering |
| Preferences | Preferences UI |
| Control Panel | — |
| Find | Search/filter within curation |
| Map | Map display for geolocated specimens |
| images | Image list/thumbnail handling |
| ColumnEditor | Display Editor (`de*`) — column/field layout for a display definition |
| Service Selection | Picking a configured service for a process step |
| Log | Debug/trace log window |

Utility code (`sm*`) is not tied to one window — it's loaded as external script libraries
(see below) and callable from anywhere.

## Startup sequence (`smInitStacks`)

Roughly in order:

1. `smUseStacks` — loads external script libraries (`Utilities`, `PhotonJSON`) from the
   `Script Libraries` folder (path differs for mobile vs standalone vs dev).
2. `open stack "Log"`.
3. `start using stack "Projects" / "Services" / "Prompts"` — these are substacks
   contained in the main stack file, not external.
4. `smSetPrefs.local`.
5. `smLoadPrefs kAPPNAME` — reads the prefs JSON file; on failure/corruption, renames the
   bad file and falls back to defaults.
6. `smInitDataFolder` — ensures the SpeciMate-Data folder (definitions/displays/services)
   exists; prompts the user to locate/choose one if missing.
7. `smServicesInit`.
8. `smDataChanged.init`.
9. `tsNetInit` — initialises the networking library used for API calls.
10. Misc: current username, country array setup.

Known fragility: prefs-loading was temporarily disabled at one point to unblock a
startup issue, which broke Services loading as a side effect. If Services aren't
appearing on launch, check this path first (see `specimate_prioritised_todo.md`, item
S.1).

## Message path / `start using`

Because `Projects`, `Services`, and `Prompts` are `start using`d at startup, unqualified
handler calls from anywhere in the app can resolve into any of them. If a handler name
seems to "just work" without an explicit stack reference, this is usually why — check
these three stacks before assuming a handler is missing.

External libraries (`Utilities`, `PhotonJSON`) are loaded the same way, so `sm*` handlers
are globally callable regardless of which card is active.

## Folder locations at a glance

- **SpeciMate-Data folder** (user-chosen, persisted in prefs): `definitions/`,
  `displays/`, `services/` — the shared library content, independent of any one project.
- **Project folder**: wherever the user's images live; a `/specimate` subfolder is
  created there to hold the project's own saved data (OCR/NER binary files, etc.).

See `data-model.md` for what's actually stored in each location.
