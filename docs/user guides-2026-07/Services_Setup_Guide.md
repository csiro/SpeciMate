# SpeciMate — Setting Up Services (OCR, Translation and LLM)

This guide explains how to obtain accounts and API keys from service providers, and how to enter those details on the **Services** screen so SpeciMate can extract text from specimen images, translate it, and pull structured metadata out of it.

---

## 1. What a "service" is in SpeciMate

SpeciMate does not do OCR, translation or entity extraction itself. It sends your images and text to external web APIs that you supply. Each of those APIs is described by a **service record** that you create on the Services screen.

There are four service types:

| Type | What it does | When you need it |
|---|---|---|
| **OCR** | Reads text from a specimen label image | Required for the OCR step |
| **Translation** | Translates non-English OCR text into English | Only if you process foreign-language labels |
| **LLM** | Extracts metadata fields from OCR **text** | Required for the metadata extraction step |
| **Multimodal** | Extracts metadata directly from the **image** (no OCR step) | Only if you use "Include image" processing for entity extraction |

You can define as many services of each type as you like — for example several LLM models from different providers — and choose which one is active and which is the default.

---

## 2. Before you start: accounts you will need

You must set these up **with the provider**, outside SpeciMate. SpeciMate only stores the resulting endpoint URL and API key.

### 2.1 Google Cloud (for OCR and Translation)

Both Google Cloud Vision (OCR) and Google Cloud Translation are billable Google Cloud services. You need to:

1. Create a Google account, then a **Google Cloud project** at
   `console.cloud.google.com`.
2. **Enable billing** on that project. A free trial credit is usually available, but the APIs will not respond without billing enabled.
3. **Enable the APIs** you intend to use — *Cloud Vision API* for OCR, *Cloud Translation API* for translation. Each must be enabled separately.
4. Create an **API key** (APIs & Services → Credentials → Create credentials → API key).
5. **Restrict the key** — limit it to the Vision and Translation APIs, and if possible to your IP address. An unrestricted key that leaks can be used by anyone at your expense.
6. **Copy** the key somewhere **safe**. Google will not show it again in full.

> Check the current pricing pages before running a large batch. Both services are charged per image / per character, and a folder of several thousand specimen images is not free. Note that up to 1000 images may be processed for free each month (at time of writing).

### 2.2 An LLM provider (for metadata extraction)

You need at least one. Common choices:

- **OpenAI** — account at `platform.openai.com`, add a payment method, create an API key.
- **Azure OpenAI** — your organisation deploys a model and gives you a deployment endpoint plus a key.
- **Google Gemini** — API key from Google AI Studio, or via Google Cloud.
- **OpenRouter** — a single account and key that proxies to many models.
- **A local model** (LM Studio, Ollama, llama.cpp) — no account and no cost, but you must have the server running on your own machine. See §6.4.

Whichever you choose, you need three things before you open the Services screen:

- the **endpoint URL** to POST to,
- the **API key** (unless local),
- the **model name** exactly as the provider spells it (e.g. `gpt-4.1-mini`,
  `gemini-2.5-flash`).

---

## 3. The Services screen

Open it from **Service Selection → Configure services**. The screen has three parts:

- **Services Folder** (top) — the folder on disk where service definitions are stored, one JSON file per service. Click the field to choose or change it; SpeciMate reloads the services from that folder when you do.
- **The service list** (left) — every service you have defined, with its Active tick, type, name and model. Click a row to edit it. `New`, `Duplicate` and `Delete` act on this list.
- **The editor** (right) — the details of the selected service.

Buttons at the bottom: **Export**, **Import**, **Test**, and **Save** / **Cancel**.

### 3.1 The editor fields

| Field | Meaning |
|---|---|
| **Service Name** | Your own label for this service. It must be unique — it is how the service is identified everywhere else, and it becomes the filename on disk. |
| **Active?** | Tick to make this service selectable. Unticked services are kept but hidden from the Service Selection screen. |
| **Service Type** | `OCR`, `Translation`, `LLM` or `Multimodal`. This controls which other fields are enabled. |
| **Model** | The provider's model identifier. Disabled for OCR and Translation. |
| **URL** | The full endpoint the request is POSTed to. |
| **API Key** | Your key. Stored with the service. See §8 for handling. |
| **Header text** | The HTTP headers to send, one per line, `Name: value`. |
| **Template** | The JSON request body, with placeholders. See §4. |
| **max. Tokens** | Ceiling on the response length for LLM/Multimodal. Disabled for OCR and Translation. |
| **max. Concurrent Requests** | How many requests SpeciMate will have in flight at once for this service. |

**max. Concurrent Requests** is the setting to reduce first if you start seeing rate-limit errors during a batch. `20` is a reasonable starting point for a commercial API; drop it to `1`–`4` for a local model or a free tier. If in doubt then check the Service provider's documentation.

---

## 4. Placeholders

The URL, headers and template are plain text. SpeciMate fills in the variable parts by substituting placeholders immediately before sending the request. **Spelling and brackets must match exactly.**

### Used in the URL and Header text

| Placeholder | Replaced with |
|---|---|
| `[XXXX]` | The API Key field |

Put `[XXXX]` wherever the key belongs — in the URL query string (Google style) *or* in an `Authorization` header (OpenAI style), whichever your provider requires.

### Used in the Template

| Placeholder | Replaced with | Applies to |
|---|---|---|
| `[BASE64_ENCODED_IMAGE]` | The image, base64-encoded | OCR, Multimodal |
| `[TEXT]` | The text to be translated | Translation |
| `[TEXT_TO_BE_PARSED]` | The OCR'd (or translated) text | LLM |
| `[MODEL]` | The Model field | LLM, Multimodal |
| `[ROLE]` | The role text from the current prompt | LLM, Multimodal |
| `[PROMPT]` | The instruction text from the current prompt | LLM, Multimodal |
| `[ENTITIES]` | The list of fields to extract, from the data definition | LLM, Multimodal |
| `[MAXTOKENS]` | The max. Tokens field | LLM, Multimodal |

Note the two different text placeholders: translation uses `[TEXT]`, the LLM uses
`[TEXT_TO_BE_PARSED]`. They are not interchangeable.

> **Don't press Return inside a quoted string in the template.** A literal line break inside a JSON string is invalid JSON. If you want a line break in the prompt text, type `\n` instead. You can lay the template out over several lines freely — just not in the middle of a `"..."` value.

---

## 5. Setting up Google OCR

1. Click **New**, and set **Service Type** to `OCR`.
2. **Service Name**: something recognisable, e.g. `Google OCR - MyOrg`.
3. **URL**:
   ```
   https://vision.googleapis.com/v1/images:annotate?key=[XXXX]
   ```
4. **API Key**: paste your Google Cloud key.
5. **Header text**:
   ```
   Content-Type: application/json
   ```
6. **Template**:
   ```json
   { "requests": [ { "image": { "content": "[BASE64_ENCODED_IMAGE]" },
     "features": [ { "type": "TEXT_DETECTION" } ] } ] }
   ```
7. Tick **Active?**, set **max. Concurrent Requests** (start with `10`–`20`), and **Save**.

> SpeciMate reads the OCR result from the Google Vision response structure, so an OCR service must return Vision-shaped JSON. A different OCR provider will need code changes, not just a new service record.

---

## 6. Setting up translation and LLM services

### 6.1 Google Translate

1. **New** → **Service Type** = `Translation`, name it e.g. `Google Translate - MyOrg`.
2. **URL**:
   ```
   https://translation.googleapis.com/language/translate/v2?key=[XXXX]
   ```
3. **API Key**: your Google Cloud key (the same key works if you enabled both APIs).
4. **Header text**:
   ```
   Content-Type: application/json
   ```
5. **Template**:
   ```
   { "q": "[TEXT]", "target": "en" }
   ```
6. Tick **Active?** and **Save**.

Translation is only invoked when the OCR result reports a detected language that isn't English or Latin with reasonable confidence, so it costs nothing on English-only batches.

### 6.2 An OpenAI-compatible LLM (OpenAI, Azure OpenAI, OpenRouter)

1. **New** → **Service Type** = `LLM`.
2. **Model**: the provider's exact model string, e.g. `gpt-4.1-mini`.
3. **URL**: the chat-completions endpoint. For example:
   - OpenAI: `https://api.openai.com/v1/chat/completions`
   - Azure OpenAI: `https://<your-resource>.cognitiveservices.azure.com/openai/deployments/<deployment>/chat/completions?api-version=<version>`
   - OpenRouter: `https://openrouter.ai/api/v1/chat/completions`
4. **API Key**: your key.
5. **Header text**:
```
   Content-Type: application/json
   Authorization: Bearer [XXXX]
```
   (Azure OpenAI may instead expect `api-key: [XXXX]` — follow your provider's docs.)
6. **Template**: copy the text-only template for your model family from
   **§9 Reference: template variants**. The two families need different parameters, and using the wrong one produces an immediate `400` error.
7. **max. Tokens**: `8192` is a sensible default. **max. Concurrent Requests**: `20`.
8. Tick **Active?** and **Save**.

**NOTE** Asking for `json_object` output matters — SpeciMate expects the model to return JSON it can map onto your metadata fields.

### 6.3 A multimodal service (image straight to metadata)

Same as above, but set **Service Type** to `Multimodal` and use one of the multimodal templates in **§9**. The difference is in the user message: instead of a plain string, its `content` is an array holding a text part and an image part, with `[BASE64_ENCODED_IMAGE]` embedded in a `data:` URL.

If the service URL contains `google`, SpeciMate uses its Gemini request/response path; otherwise it assumes an OpenAI-compatible shape. Bear that in mind when naming endpoints.

### 6.4 A local model (LM Studio, Ollama)

1. Start the local server and note the port it is listening on.
2. **URL**: e.g. `http://localhost:1234/v1/chat/completions` (LM Studio) or
   `http://localhost:11434/v1/chat/completions` (Ollama).
3. **Model**: whatever the local server reports, e.g. `qwen/qwen3-8b`.
4. **API Key**: local servers usually ignore it, but **enter a dummy value such as `local` anyway** — leaving it blank produces a warning, and some servers reject an empty `Authorization` header.
5. **Header text** and **Template**: as for the OpenAI-compatible service above.
6. Set **max. Concurrent Requests** low — `1` to `4`. A local model will queue or fall over under twenty simultaneous requests.

---

## 7. Checking, testing and activating

### 7.1 Check the template

Click **Check** next to the Template field. It parses the template as JSON and reports whether it is valid. Common causes of failure are a missing brace or bracket, a stray comma before a closing brace, or a `"` inside a string that hasn't been escaped.

`Check` only validates the JSON *shape*. It does not verify the placeholders, so also read through the template and confirm every placeholder you need is present and correctly spelt.

### 7.2 Test the service

Select the service and click **Test**. SpeciMate builds a small test request using the same substitution path as real processing — a canned prompt and text, and a tiny placeholder image for multimodal services — and shows you the response. This is the fastest way to confirm that the endpoint, key, headers and template all work together.

To test with a real image, **hold Shift while clicking Test** and choose an image file (pick a small one). That image is remembered and used for subsequent multimodal tests.

If the test fails, the response usually tells you which part is wrong:

| Symptom | Likely cause |
|---|---|
| `401` / `403` / "invalid API key" | Key wrong, expired, or missing `[XXXX]` in URL or header |
| `404` | URL wrong — check the path, deployment name and API version |
| `400` with a JSON parse complaint | Template malformed, or a placeholder left unsubstituted |
| "Unsupported parameter: `max_tokens` … use `max_completion_tokens`" | Template is the older variant on a newer model — see §9 |
| "Unsupported value: `temperature`… only the default (1) is supported" | Remove `temperature` from the template — see §9 |
| "model not found" | Model field doesn't match the provider's exact string |
| `429` | Rate limited — reduce max. Concurrent Requests |
| Nothing at all / connection refused | Local server not running, or wrong port |
| "No template data supplied" | Template is empty, or doesn't start with `{` |
| Model replies but the label text is misread | Small label text — raise `detail`, or use a higher-resolution image |

### 7.3 Make it available for use

1. Tick **Active?** on the service and **Save**. Only active services appear as choices.
2. Go to **Service Selection** and pick the default service for each type — OCR, Translation, LLM, Multimodal.
3. If no default has been chosen for a type, SpeciMate falls back to the first active service of that type and logs that it has done so.

Processing will stop with an error if the OCR or Translation service you're using has no API key. A missing LLM key only produces a log warning, because local models don't need one.

---

## 8. Storing, sharing and backing up services

- Each service is saved as its own JSON file in the Services Folder. The filename is derived from the service name (lowercased, spaces and punctuation replaced with hyphens).
- **Renaming a service changes its filename**, so keep names stable once you're relying on them.
- **Delete** doesn't destroy the file — it moves it to a `deleted` subfolder of the Services Folder.

### Export

**Export** writes the currently selected service to the Services Folder. Hold **Shift** to
export all of them; hold **Control** to choose a different destination folder.

You are asked whether to **include the API key**. Answer **No** for anything you intend to share, commit to a repository, or email — an exported file with a key in it is a plaintext credential. Answer **Yes** only for your own backup, kept somewhere protected.

**NOTE** If you export without including the API key and have not used the control Key to select a different folder then you may overwrite an existing service (and lose your API key). 

### Import

**Import** reads every `.json` file in the Services Folder and **replaces your whole service list** with them. To update only certain services instead, select those rows first and hold **Shift** while clicking Import — matching is by service name, and everything else is left alone.

Because exported files usually have the key stripped, expect to re-enter API keys after importing a colleague's service definitions.

---

## 9. Reference: template variants

Copy these into the **Template** field and change nothing except where noted. All four use the OpenAI **Chat Completions** shape, which is what OpenAI, Azure OpenAI, OpenRouter, LM Studio, Ollama and most self-hosted servers accept.

### 9.1 Which variant do I need?

OpenAI split its request parameters partway through the GPT-5 generation, and the older parameters are now rejected outright rather than ignored. Pick by model family:

| Your model | Token limit parameter | `temperature` |
|---|---|---|
| `gpt-4o`, `gpt-4.1`, `gpt-4.1-mini`, `gpt-4.1-nano`, and most local/open models | `max_tokens` | Allowed — use `0` |
| `gpt-5*`, `o1`, `o3`, `o4-mini` and other reasoning models | `max_completion_tokens` | **Not allowed** — omit it entirely |

Reasoning models also reject `top_p`, `presence_penalty`, `frequency_penalty`,
`logprobs` and `logit_bias`. If in doubt, start with the newer variant: it is accepted by almost everything, whereas `max_tokens` is a hard failure on GPT-5-class models.

Gemini services do **not** use these templates — SpeciMate builds Gemini requests on a separate code path.

### 9.2 Text only — GPT-4.x era and most local models

```json
{ "model": "[MODEL]",
  "messages": [
    {"role": "developer", "content": "[ROLE]"},
    {"role": "user", "content": "[PROMPT] [TEXT_TO_BE_PARSED] [ENTITIES]"}
  ],
  "response_format": { "type": "json_object" },
  "temperature": 0,
  "max_tokens": [MAXTOKENS] }
```

### 9.3 Text only — GPT-5.x and other reasoning models

```json
{ "model": "[MODEL]",
  "messages": [
    {"role": "developer", "content": "[ROLE]"},
    {"role": "user", "content": "[PROMPT] [TEXT_TO_BE_PARSED] [ENTITIES]"}
  ],
  "response_format": { "type": "json_object" },
  "max_completion_tokens": [MAXTOKENS] }
```

### 9.4 Multimodal — GPT-4.x era

```json
{ "model": "[MODEL]",
  "messages": [
    {"role": "developer", "content": "[ROLE]"},
    {"role": "user", "content": [
      {"type": "text", "text": "[PROMPT] [ENTITIES]"},
      {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,[BASE64_ENCODED_IMAGE]", "detail": "high"}}
    ]}
  ],
  "response_format": { "type": "json_object" },
  "temperature": 0,
  "max_tokens": [MAXTOKENS] }
```

### 9.5 Multimodal — GPT-5.x and other reasoning models

```json
{ "model": "[MODEL]",
  "messages": [
    {"role": "developer", "content": "[ROLE]"},
    {"role": "user", "content": [
      {"type": "text", "text": "[PROMPT] [ENTITIES]"},
      {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,[BASE64_ENCODED_IMAGE]", "detail": "high"}}
    ]}
  ],
  "response_format": { "type": "json_object" },
  "max_completion_tokens": [MAXTOKENS] }
```

### 9.6 Notes on the multimodal templates

- **Image format.** `data:image/jpeg;base64,` assumes JPEG. Change it to
  `data:image/png;base64,` if your specimen images are PNG. Supported types are PNG, JPEG, WEBP and non-animated GIF.
- **`detail`.** Controls how much resolution the model gets. `high` is the safe portable choice and is what you want for label text. `low` is cheaper but downsamples to 512×512, which will lose small handwriting. Newer models (gpt-5.4 and later) also accept `original`, which preserves the input dimensions — worth trying if labels are being misread, though it costs more tokens. Omitting `detail` is fine; the model chooses.
- **No `[TEXT_TO_BE_PARSED]`.** Multimodal calls send the image instead of OCR text, so that placeholder isn't used and won't be substituted.
- **Cost.** Images are billed as tokens, and a full-resolution specimen scan can be expensive. Test on a handful of images and check your provider's usage dashboard before committing to a large batch.

### 9.7 A note on `json_object`

`"response_format": { "type": "json_object" }` asks the model for valid JSON but does not constrain its shape. OpenAI now also offers `json_schema`, which guarantees the response matches a schema you supply. SpeciMate doesn't generate schemas yet, so `json_object` remains the right setting — but if a model keeps returning JSON with the wrong field names, that's the reason, and tightening the prompt is currently the fix.

---

## 10. Setup checklist

- [ ] Google Cloud project created, billing enabled, Vision and/or Translation API enabled
- [ ] Google API key created and restricted
- [ ] LLM provider account created and API key obtained (or local server running)
- [ ] Services Folder set on the Services screen
- [ ] OCR service: URL with `[XXXX]`, key, `Content-Type` header, template with `[BASE64_ENCODED_IMAGE]`
- [ ] Translation service (if needed): URL with `[XXXX]`, key, template with `[TEXT]`
- [ ] LLM service: URL, key, `Authorization` header, model, and the **correct template variant for the model family** (§9)
- [ ] max. Tokens and max. Concurrent Requests set sensibly
- [ ] **Check** passes on every template
- [ ] **Test** succeeds for every service
- [ ] **Active?** ticked, and defaults chosen on the Service Selection screen
- [ ] Services exported as a backup (with keys) and/or for sharing (without keys)
