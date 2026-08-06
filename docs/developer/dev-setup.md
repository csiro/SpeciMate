# SpeciMate — Dev Setup
Create date: 2026-07-20
Last modified date: 2026-07-20

Getting a working dev environment. Short by design — see the other docs for anything
behavioural.

## LiveCode

- Use the same LiveCode version the project stack files were last saved with (check the
  `.livecode` file's version, or ask if unsure — mixing versions on save can churn the
  file unnecessarily).
- The app expects to run as a standalone-style stack with substacks (`Projects`,
  `Services`, `Prompts`) contained in the main stack file, plus external script libraries
  loaded separately (see below). Opening the main stack in the IDE is the normal way to
  develop.

## Script library location

`smUseStacks` resolves the `Script Libraries` folder differently depending on
environment:

| Environment | Path used |
|---|---|
| Mobile | `specialFolderPath("engine") & "/Script Libraries"` |
| Standalone / other | `specialFolderPath("resources") & "/Script Libraries"` |

It loads `Utilities` and `PhotonJSON` from there (as `.livecode` or falling back to
`.livecodescript`), resolving aliases if needed. If either fails to load,
`smUseStacks` logs "Could not find utility stack" — that's the first thing to check if
`sm*` handlers seem to be missing at startup.

For IDE development, make sure the `Script Libraries` folder sits under the resolved
`specialFolderPath("resources")` for your setup, or adjust locally as needed — this path
resolution hasn't yet been made dev-environment-aware (see `architecture.md` for the
full startup sequence).

## SpeciMate-Data folder

- On first run, `smInitDataFolder` prompts for a folder containing `definitions/`,
  `displays/`, `services/`. For dev work, point this at a working copy you're happy to
  have modified — not a shared/production copy — since editing definitions, displays, or
  services from the app writes back to this folder.
- The chosen path is remembered in prefs (`kDataFolderPref`), so it persists across
  restarts until changed.

## API keys

- Each service (OCR, Translation, LLM, Multimodal) carries its own `APIKey` field,
  configured via the Services screen and stored in that service's JSON file under
  `<SpeciMate-Data folder>/services/`. See `services.md` for the full record shape.
- **Never commit real API keys.** If you export services for sharing (e.g. into version
  control or documentation), export without including the key — the export path already
  supports stripping it (`pIncludeKey` false).
- An older `readAPIConfig "metadata.ini"` path exists in the code but is commented out —
  treat the Services screen / per-service JSON as the current source of truth for keys,
  not an `.ini` file.

## Running from the IDE vs standalone build

- `smGetStackPath` / `smStackPath` branch on `the environment` — mobile uses the engine
  folder, everything else derives the path from the running stack's file location. Don't
  hardcode absolute paths in new code; use these helpers.
- Standalone builds package the `Script Libraries` folder into the app's `Resources`
  location — if a library loads fine in the IDE but not in a built standalone, check that
  the build's copy-files settings actually include that folder.

## Quick checklist for a new dev machine

1. Install the correct LiveCode version.
2. Clone/copy the stack files and the `Script Libraries` folder into a layout matching
   the path resolution above.
3. Choose (or create) a working `SpeciMate-Data` folder with `definitions/`, `displays/`,
   `services/` subfolders when first prompted.
4. Configure at least one OCR/LLM service with a personal API key via the Services
   screen — don't reuse a shared key committed anywhere.
5. Open the main stack and confirm `smInitStacks` completes without a
   "Could not find utility stack" or data-folder error in the Log window.
