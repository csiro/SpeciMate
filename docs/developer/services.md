# SpeciMate — Services
Create date: 2026-07-20
Last modified date: 2026-07-20

Everything about talking to external APIs: OCR, Translation, LLM, and Multimodal.

## Service record

Each configured service is an array with (at least):

| Key | Meaning |
|---|---|
| `Name` | Display name, also used to derive the saved filename (slugified) |
| `ServiceType` / process type | `OCR`, `Translation`, `LLM`, `Multimodal` (`kProcessType.*`) |
| `URL` | Endpoint. `[XXXX]` in the URL is replaced with the API key if the key belongs in the URL rather than a header |
| `Header` | Raw header text; `[XXXX]` replaced with the API key the same way |
| `Model` | Model name/identifier |
| `APIKey` | Stripped out before export unless explicitly included (see below) |
| `maxTokens` | Defaults to `256` if empty when building a test payload |
| `Template` | The payload template string — see below |

Services are stored as individual JSON files, one per service, filename = slugified
service `Name`, via `saveServicesToIndividualFiles` / `loadServicesFromIndividualFiles`.
Exporting without `pIncludeKey` strips `APIKey` from the saved file — don't assume an
exported service file is safe to share as-is if a key was included.

[Screenshot: Services configuration screen for one service]

## Payload templates

LLM/Multimodal payloads are **built by string substitution into a template, not by
serialising an array**. The template is a JSON-shaped string with placeholders:

| Placeholder | Replaced with |
|---|---|
| `[MODEL]` | Service's `Model` |
| `[PROMPT]` | The instruction prompt |
| `[ROLE]` | The role/system text |
| `[TEXT_TO_BE_PARSED]` | The OCR'd (or plain) text to process |
| `[ENTITIES]` | Entity/column list description |
| `[MAXTOKENS]` | Max tokens value |
| `[BASE64_ENCODED_IMAGE]` | Base64 image data, CR and linefeed stripped, for multimodal calls |

This is checked by looking for a template whose first non-space character is `{`; if the
template is empty or doesn't look like JSON, no fallback payload is currently built —
the call fails with a "no template data supplied" error rather than guessing a shape.

### Escaping — read this before touching a substitution site

Because the payload is a template string rather than a serialised structure, **any
unescaped character in a substituted value can break the JSON** — quotes, backslashes,
newlines, control characters. This is a real risk because the values being substituted
include:

- Raw OCR text (`[TEXT_TO_BE_PARSED]`) — **highest risk**, since it's unpredictable,
  user-uploaded-image content, not something typed carefully into a UI.
- Prompt, role, and entity list text — lower risk, but still free text.

**The fix belongs at each substitution site**, not in the prompt editor UI — sanitising
input in the editor would miss the OCR text entirely, since it never passes through the
editor.

- **`smJSONEscape`** is the intended helper: apply it to each value immediately before
  substitution into the template.
- **`smEscapeCRsInJSON` is not a substitute.** It operates on the whole payload after
  assembly and uses quote-counting logic that breaks under the conditions it's meant to
  handle. Don't reach for it as a quick fix.
- Status: agreed as the solution, not yet applied everywhere. The three
  `[TEXT_TO_BE_PARSED]` branches in particular have been flagged as inconsistent —
  confirm current state in the todo list before assuming this is finished.

## Provider differences

- **Routing**: `smConfigureMultimodal` inspects the service URL — if it contains
  `"google"`, it's treated as Gemini; if it contains `"openai"`, OpenAI-compatible;
  otherwise it defaults to OpenAI-compatible (works for most self-hosted/compatible
  endpoints).
- **OpenAI-compatible**: standard chat-completions-shaped payload via the template
  mechanism above.
- **Gemini**: separate call/parse path (`smCallGeminiMultimodal` /
  `smParseGeminiResponse`). Response shape differs — content comes from
  `candidates[1]["content"]["parts"][1]["text"]`; errors come from `["error"]["message"]`.
  Don't assume the OpenAI response parser works for Gemini or vice versa.
- **Transport**: `smPostJSON` prefers `tsNet` (async-capable) when available, falling
  back to `libURL`. New service-calling code should go through `smPostJSON` rather than
  building its own request.

## Testing a service

`Services_TestCurrentService` (or equivalent) builds a minimal test payload — a canned
prompt/role/text and a 1×1 placeholder PNG for multimodal — using the same template
substitution path as real calls. Useful as the reference example when adding a new
provider: it exercises URL, header, and template substitution end to end without needing
a real image.

## Adding a new service

1. Add a service record via the Services screen: `Name`, `URL`, `Header`, `Model`,
   `APIKey`, `Template`.
2. Write the `Template` for the provider's expected request shape, using the placeholders
   above wherever a substituted value is needed.
3. If the provider needs response parsing different from the existing OpenAI-compatible
   path, add a dedicated parse function (following the Gemini pattern) rather than
   overloading the existing one with conditionals.
4. Run the built-in service test before wiring it into a real process step.
5. Confirm `smJSONEscape` (or its current equivalent) is applied to every value you
   substitute into the template — especially if the new service also accepts OCR text.
