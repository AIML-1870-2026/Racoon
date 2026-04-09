# LLM Switchboard — Product Specification

**Version:** 1.0  
**Status:** Draft  
**Last Updated:** 2026-04-07

---

## 1. Overview

The LLM Switchboard is a unified interface that allows users to send prompts to large language models across multiple providers through their respective APIs. Users can configure API credentials, select models, define output schemas, and inspect structured or unstructured responses — all from a single interface.

---

## 2. Goals

- Provide a single interface for interacting with multiple LLM providers
- Support structured (JSON schema-validated) and unstructured (plain text) response modes
- Allow users to store and manage API keys per provider
- Offer example prompts and reusable schema templates to accelerate workflows
- Surface clear, actionable error messages when requests fail

---

## 3. Supported LLM Providers & Models

Each provider section lists available models. The user selects one provider and one model per request. Model lists should be refreshable from the provider's API where supported.

### 3.1 Anthropic

| Model ID | Display Name | Context Window |
|---|---|---|
| `claude-opus-4-6` | Claude Opus 4.6 | 200k tokens |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 | 200k tokens |
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | 200k tokens |

### 3.2 OpenAI

| Model ID | Display Name | Context Window |
|---|---|---|
| `gpt-4o` | GPT-4o | 128k tokens |
| `gpt-4o-mini` | GPT-4o Mini | 128k tokens |
| `gpt-4-turbo` | GPT-4 Turbo | 128k tokens |
| `o3` | o3 | 200k tokens |
| `o4-mini` | o4 Mini | 200k tokens |

### 3.3 Google

| Model ID | Display Name | Context Window |
|---|---|---|
| `gemini-2.5-pro` | Gemini 2.5 Pro | 1M tokens |
| `gemini-2.5-flash` | Gemini 2.5 Flash | 1M tokens |
| `gemini-2.0-flash` | Gemini 2.0 Flash | 1M tokens |

### 3.4 Meta (via third-party hosts, e.g. Groq, Together AI, Fireworks)

| Model ID | Display Name | Context Window |
|---|---|---|
| `llama-4-scout-17b` | Llama 4 Scout 17B | 128k tokens |
| `llama-4-maverick-17b` | Llama 4 Maverick 17B | 128k tokens |
| `llama-3.3-70b` | Llama 3.3 70B | 128k tokens |

### 3.5 Mistral

| Model ID | Display Name | Context Window |
|---|---|---|
| `mistral-large-latest` | Mistral Large | 128k tokens |
| `mistral-small-latest` | Mistral Small | 128k tokens |
| `codestral-latest` | Codestral | 256k tokens |

> **Note:** Model lists are soft-coded and updatable via a `models.json` config file without requiring a code change. A "Refresh Models" button fetches the current model list from a provider's `/models` endpoint where the API supports it.

---

## 4. API Key Management

### 4.1 Key Entry Panel

Each provider has a dedicated API key input field. Keys are:

- Entered via a masked password-style input (`type="password"`)
- Stored in the browser's `sessionStorage` under a namespaced key (e.g., `llm_switchboard::key::anthropic`)
- Persisted only for the current browser session — keys are cleared when the tab is closed but survive page refreshes
- Never transmitted except directly to the respective provider's API endpoint
- Clearable individually per provider or all at once via a "Clear All Keys" action

### 4.2 Key Status Indicator

Each provider row shows a status badge next to the key field:

| Status | Badge | Meaning |
|---|---|---|
| Not set | `— No key` | Field is empty |
| Saved (unverified) | `● Saved` | Key is stored but not yet tested |
| Verified | `✓ Active` | Key passed a lightweight validation request |
| Invalid | `✗ Invalid` | Last request returned a 401 Unauthorized |

A "Test Key" button triggers a minimal API call (e.g., a single-token completion) to verify the key and update the status badge.

### 4.3 Security Notice

A persistent inline notice reads:

> "API keys are stored in your browser session and are automatically cleared when you close the tab. Keys are never sent to our servers."

---

## 5. Prompt Composer

### 5.1 Input Fields

| Field | Type | Description |
|---|---|---|
| System Prompt | Textarea (optional) | Instruction context sent as the `system` role |
| User Message | Textarea (required) | The primary prompt sent as the `user` role |
| Temperature | Slider (0.0 – 2.0) | Controls randomness; default `1.0` |
| Max Tokens | Number input | Upper bound on response length; default `1024` |

### 5.2 Example Prompts

A collapsible "Example Prompts" panel provides pre-written prompts users can load in one click. Each example includes a title, a suggested model, and a description of what it demonstrates.

| Title | Suggested Model | Description |
|---|---|---|
| Summarize an article | Any | Paste article text and get a 3-sentence summary |
| Extract structured data | GPT-4o / Sonnet 4.6 | Pull fields from unstructured text into JSON |
| Code review | Codestral / Claude Sonnet | Review a code snippet for bugs and style issues |
| Classify sentiment | Gemini Flash / Haiku | Return a sentiment label with confidence score |
| Q&A over a document | Any large-context model | Answer a question grounded in provided text |
| Chain-of-thought math | o3 / o4-mini | Solve a multi-step word problem with reasoning |
| Roleplay persona | Any | Instruct the model to adopt a custom persona |
| Translate & localize | Mistral Large | Translate text and adapt idioms for a target culture |

Clicking an example populates the System Prompt and User Message fields and sets a recommended model, which the user can override.

---

## 6. Output Modes

The user selects an output mode before sending a request. The mode controls how the response is displayed and, when applicable, how the model is instructed to format its output.

### 6.1 Unstructured (Plain Text)

- No schema applied
- Model response rendered as plain text in a read-only code/text block
- Supports copy-to-clipboard and download as `.txt`
- Token usage displayed below the response (prompt tokens, completion tokens, total)

### 6.2 Structured (JSON)

- The user selects or defines a JSON schema (see Section 7)
- The schema is injected into the system prompt as a formatting instruction
- The response is parsed as JSON and displayed in a syntax-highlighted collapsible tree view
- If parsing fails, the raw response is shown alongside a parse error message (see Section 9)
- Supports copy-to-clipboard and download as `.json`

### 6.3 Structured (Table)

- A simplified structured mode that renders a JSON array of objects as an HTML table
- Column headers are inferred from object keys
- Supports copy as CSV and download as `.csv`

### 6.4 Structured (Markdown)

- Response is rendered as formatted Markdown (headings, lists, code blocks, tables)
- Useful for reports, documentation, or formatted summaries
- Supports download as `.md`

---

## 7. Schema Templates

Schema templates define the JSON structure the model is asked to return. They are used in Structured (JSON) output mode.

### 7.1 Built-in Templates

| Template Name | Description | Top-Level Fields |
|---|---|---|
| **Key-Value Pairs** | Simple flat object of string fields | `{ "key": "value", ... }` |
| **Sentiment Analysis** | Classify text with a score | `label`, `score`, `explanation` |
| **Entity Extraction** | Extract named entities from text | `entities[]` with `name`, `type`, `context` |
| **Structured Summary** | Summarize content into sections | `title`, `summary`, `key_points[]`, `takeaways[]` |
| **Classification** | Assign a category with confidence | `category`, `confidence`, `reasoning` |
| **Q&A Pair** | Generate question/answer pairs | `question`, `answer`, `source_quote` |
| **Action Items** | Extract tasks from meeting notes or text | `action_items[]` with `task`, `owner`, `due_date` |
| **Comparison Table** | Compare two or more items | `items[]` with `name` + dynamic attribute columns |

### 7.2 Custom Schema

Users can define their own schema by selecting the **Custom** option, which opens a schema editor:

- A JSON editor pane (syntax-highlighted, with bracket matching and inline error indicators)
- A "Validate Schema" button that checks for valid JSON syntax and basic JSON Schema compliance
- A "Preview Prompt Injection" button that shows exactly how the schema will be embedded in the system prompt
- Schema can be saved locally under a user-defined name and reused across sessions

**Custom schema editor placeholder text:**

```json
{
  "type": "object",
  "properties": {
    "field_one": { "type": "string" },
    "field_two": { "type": "number" }
  },
  "required": ["field_one"]
}
```

### 7.3 Schema Injection Format

When a schema is active, the following instruction is prepended to the system prompt:

```
Respond ONLY with a valid JSON object that strictly conforms to the following JSON Schema.
Do not include any explanation, commentary, or markdown formatting outside the JSON object.

Schema:
{schema}
```

---

## 8. Response Panel

The response panel occupies the right half of the interface (or bottom half on mobile). It contains:

- **Model badge** — shows which provider and model generated the response
- **Latency** — time from request send to full response received (ms)
- **Token usage** — prompt / completion / total token counts
- **Output display** — formatted per the active output mode (Section 6)
- **Request metadata toggle** — expandable section showing the full request payload sent to the API (headers redacted)
- **Re-run button** — resends the exact same request
- **Compare mode** — sends the same prompt to up to 3 models simultaneously and renders responses side-by-side

---

## 9. Error Messages

All API errors are caught and surfaced in a dismissible error banner above the response panel. Each error includes a human-readable explanation and a suggested resolution.

### 9.1 Authentication Errors

| HTTP Code | Error ID | User-Facing Message |
|---|---|---|
| 401 | `AUTH_INVALID_KEY` | **Invalid API Key** — The key provided for {Provider} was rejected. Please check that the key is correct and has not been revoked. |
| 403 | `AUTH_INSUFFICIENT_PERMISSIONS` | **Access Denied** — Your API key does not have permission to use this model or endpoint. Check your account's tier or feature access at the provider's dashboard. |

### 9.2 Rate Limiting & Quota Errors

| HTTP Code | Error ID | User-Facing Message |
|---|---|---|
| 429 | `RATE_LIMIT_REQUESTS` | **Too Many Requests** — You've exceeded the request rate limit for {Provider}. Wait a moment before retrying, or reduce request frequency. |
| 429 | `RATE_LIMIT_TOKENS` | **Token Quota Exceeded** — You've hit the per-minute token limit. Try reducing Max Tokens or simplify your prompt. |
| 402 | `QUOTA_EXHAUSTED` | **Account Quota Exhausted** — Your {Provider} account has no remaining credits or has hit its monthly usage cap. Add credits or upgrade your plan. |

### 9.3 Request Errors

| HTTP Code | Error ID | User-Facing Message |
|---|---|---|
| 400 | `INVALID_REQUEST_BODY` | **Bad Request** — The request was malformed. This may be caused by an invalid schema, an unsupported parameter for this model, or a missing required field. Details: `{error.message}` |
| 400 | `CONTEXT_LENGTH_EXCEEDED` | **Prompt Too Long** — Your prompt exceeds the context window for {Model} ({limit} tokens). Shorten your input or switch to a model with a larger context window. |
| 400 | `INVALID_MODEL` | **Unknown Model** — The selected model ID `{model_id}` is not recognized by {Provider}. Refresh the model list and try again. |
| 422 | `SCHEMA_PARSE_FAILURE` | **Response Parse Error** — The model returned a response that could not be parsed as valid JSON. The raw response is shown below. Try adjusting your schema or adding stricter formatting instructions. |

### 9.4 Server & Network Errors

| HTTP Code | Error ID | User-Facing Message |
|---|---|---|
| 500 | `PROVIDER_SERVER_ERROR` | **Provider Error** — {Provider}'s API returned an internal server error. This is not caused by your request. Please retry in a few moments. |
| 503 | `PROVIDER_UNAVAILABLE` | **Service Unavailable** — {Provider} is temporarily unavailable or undergoing maintenance. Check the provider's status page and retry shortly. |
| — | `NETWORK_TIMEOUT` | **Request Timed Out** — No response was received within {timeout}s. Check your internet connection or try again. You can adjust the timeout in Settings. |
| — | `NETWORK_OFFLINE` | **No Internet Connection** — Your device appears to be offline. Restore your connection and retry. |
| — | `CORS_BLOCKED` | **Request Blocked** — The request was blocked by the browser's CORS policy. See Section 9.6 for a full explanation. |

### 9.5 Application Errors

| Error ID | User-Facing Message |
|---|---|
| `NO_API_KEY` | **No API Key Set** — You haven't entered an API key for {Provider}. Add your key in the API Keys panel to continue. |
| `NO_MODEL_SELECTED` | **No Model Selected** — Please select a model before sending your request. |
| `EMPTY_PROMPT` | **Prompt is Empty** — The User Message field cannot be blank. |
| `INVALID_JSON_SCHEMA` | **Invalid Schema** — Your custom schema contains a JSON syntax error. Check the schema editor for highlighted errors before retrying. |

### 9.6 CORS Limitations — Why Browser-Based API Requests Can Be Blocked

**What is CORS?**

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that restricts web pages from making HTTP requests to a domain different from the one that served the page. If this app is served from `https://llm-switchboard.app` and it attempts to call `https://api.openai.com`, the browser treats that as a *cross-origin* request and enforces CORS rules before allowing it to proceed.

**Why does it block the request?**

Before sending the actual API call, the browser first sends a preflight `OPTIONS` request to the API server, asking: *"Do you allow requests from this origin?"* The API server responds with headers like:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type
```

If the server's response does not explicitly permit the requesting origin, the browser refuses to forward the request — even if the API key itself is perfectly valid. The block happens entirely inside the browser, before the request ever reaches the LLM provider.

**Why does this affect API keys specifically?**

LLM provider APIs require an `Authorization` header (e.g., `Authorization: Bearer sk-...`) to authenticate requests. Browsers treat requests containing custom headers like `Authorization` as non-simple requests, which always trigger a CORS preflight. If the provider's API does not include permissive CORS headers in its preflight response, the browser blocks the call regardless of whether the key is correct.

This is a deliberate design choice by some providers: their APIs are intended to be called from a backend server, not directly from a browser, in part to reduce the risk of API key exposure in client-side code.

**Which providers are affected?**

| Provider | Browser-Direct Calls | Notes |
|---|---|---|
| Anthropic | ❌ Blocked | Does not include CORS headers; requires a backend proxy |
| OpenAI | ❌ Blocked | No CORS support for direct browser requests |
| Google Gemini | ⚠️ Partial | Some endpoints permit browser access; behavior varies |
| Mistral | ❌ Blocked | Backend-only API |
| Meta (via Groq/Together/Fireworks) | Varies | Depends on the hosting provider's CORS policy |

**How the Switchboard handles this**

When a `CORS_BLOCKED` error is detected, the Switchboard offers two resolutions:

1. **Proxy Mode (recommended):** Route all API requests through a lightweight backend proxy server that adds the appropriate CORS headers. Configure the proxy URL in Settings → Proxy URL. The proxy forwards the request to the provider and returns the response to the browser. The API key is sent in the `Authorization` header from the proxy, not the browser, which avoids the CORS restriction entirely.

2. **Local Proxy / Desktop Mode:** If running the Switchboard locally (e.g., via `localhost`), some providers permit requests from `localhost` origins. A local proxy such as a simple Express or FastAPI server running on the same machine can forward requests without triggering cross-origin restrictions.

> ⚠️ **Security note:** When using a proxy, ensure it is a server you control. Never route your API keys through a third-party proxy service you do not trust, as the proxy can read your keys and request payloads in transit.

---

## 10. Settings

| Setting | Type | Default | Description |
|---|---|---|---|
| Request Timeout | Number (seconds) | `30` | How long to wait before surfacing a timeout error |
| Proxy URL | Text input | *(empty)* | Optional HTTP proxy for routing API requests |
| Save Prompt History | Toggle | On | Whether to persist sent prompts in local history |
| Max History Items | Number | `50` | Maximum number of past prompts to store |
| Theme | Select | System | Light / Dark / System |
| Default Output Mode | Select | Plain Text | Pre-select an output mode on load |

---

## 11. Browser Storage Schema

Data is stored under the `llm_switchboard::` namespace using two storage mechanisms:

### 11.1 Session Storage (cleared when tab closes)

| Key | Type | Description |
|---|---|---|
| `llm_switchboard::key::{provider}` | String | API key per provider |

### 11.2 Local Storage (persists across sessions)

| Key | Type | Description |
|---|---|---|
| `llm_switchboard::settings` | JSON object | User settings (Section 10) |
| `llm_switchboard::history` | JSON array | Array of past request/response pairs |
| `llm_switchboard::schemas` | JSON array | User-saved custom schemas |
| `llm_switchboard::last_model` | String | Last selected `provider::model_id` |

---

## 12. Non-Goals (v1.0)

- Multi-turn conversation / chat history threading
- Image, audio, or file input (vision/multimodal support deferred to v1.1)
- Server-side key storage or user accounts
- Streaming responses (planned for v1.1)
- Fine-tuned or custom model endpoints
- Automatic model benchmarking or cost comparison
