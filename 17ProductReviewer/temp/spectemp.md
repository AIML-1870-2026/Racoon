# LLM Switchboard — Product Specification

**Version:** 1.1
**Status:** Implemented
**Last Updated:** 2026-04-09

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
- Enable side-by-side model comparison

---

## 3. Supported LLM Providers & Models

Each provider section lists available models. The user selects one provider and one model per request.

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

### 3.4 Meta (via Groq)

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

### 3.6 Custom Provider

Users can configure a custom provider with:

| Field | Description |
|---|---|
| Provider Name | Display name for the custom provider |
| API Endpoint URL | Full URL to the chat completions endpoint |
| API Format | OpenAI-Compatible, Anthropic Format, or Google Gemini Format |
| Model Name / ID | The model identifier expected by the API |
| API Key | Authentication key for the custom provider |
| Custom Headers | Optional JSON object of additional HTTP headers |

> **Tip:** Many providers use OpenAI-compatible APIs including: Groq, Together AI, Fireworks, Anyscale, Perplexity, DeepSeek, OpenRouter, and local servers like Ollama, LM Studio, and vLLM.

---

## 4. API Key Management

### 4.1 Key Entry Panel

Each provider has a dedicated API key input field accessed via provider tabs (Anthropic, OpenAI, Google, Meta, Mistral, Custom). Keys are:

- Entered via a masked input field with toggle visibility button
- Stored in the browser's `sessionStorage` under a namespaced key (e.g., `llm_switchboard::key::anthropic`)
- Persisted only for the current browser session — keys survive page refreshes but are cleared when the tab is closed
- Never transmitted except directly to the respective provider's API endpoint
- Clearable individually per provider or all at once via a "Clear All Keys" action

### 4.2 Key Status Indicator

Each provider row shows a status badge next to the key field:

| Status | Badge | Meaning |
|---|---|---|
| Not set | `— No key` | Field is empty |
| Saved (unverified) | `● Saved` | Key is stored but not yet tested |
| Verified | `✓ Active` | Key passed a test request |
| Invalid | `✗ Invalid` | Last request returned a 401 Unauthorized |

A "Test Key" button triggers a validation request to verify the key and update the status badge.

### 4.3 Security Notice

A persistent inline notice reads:

> "API keys are stored locally in your browser and are never sent to our servers. Clear your keys when using shared or public devices."

---

## 5. Prompt Composer

### 5.1 Input Fields

| Field | Type | Description |
|---|---|---|
| System Prompt | Textarea (optional) | Instruction context sent as the `system` role |
| User Message | Textarea (required) | The primary prompt sent as the `user` role |
| Temperature | Slider (0.0 – 2.0) | Controls randomness; default `1.0` |
| Max Tokens | Number input | Upper bound on response length; default `1024`, max `100000` |

### 5.2 Example Prompts

A collapsible "Example Prompts" panel provides pre-written prompts users can load in one click. Each example includes a title, a suggested model, system prompt, user message, and a description.

| Title | Suggested Model | Description |
|---|---|---|
| Summarize an article | Any | Get a 3-sentence summary |
| Extract structured data | GPT-4o | Pull fields into JSON |
| Code review | Claude Sonnet 4.6 | Review for bugs and style |
| Classify sentiment | Gemini 2.5 Flash | Sentiment with confidence |
| Q&A over document | Any | Answer grounded in text |
| Chain-of-thought math | o3 | Multi-step reasoning |
| Roleplay persona | Any | Custom persona response |
| Translate & localize | Mistral Large | Translation with adaptation |

Clicking an example populates the System Prompt and User Message fields and sets a recommended model, which the user can override.

---

## 6. Output Modes

The user selects an output mode before sending a request. The mode controls how the response is displayed and, when applicable, how the model is instructed to format its output.

### 6.1 Plain Text

- No schema applied
- Model response rendered as plain text in a read-only code block
- Supports copy-to-clipboard and download as `.txt`
- Token usage displayed below the response (prompt tokens, completion tokens, total)

### 6.2 JSON

- The user selects or defines a JSON schema (see Section 7)
- The schema is injected into the system prompt as a formatting instruction
- The response is parsed as JSON and displayed in a syntax-highlighted collapsible tree view
- If parsing fails, the raw response is shown alongside a parse error message
- Supports copy-to-clipboard and download as `.json`

### 6.3 Table

- A structured mode that renders a JSON array of objects as an HTML table
- Column headers are inferred from object keys
- Supports copy as CSV and download as `.csv`

### 6.4 Markdown

- Response is rendered as formatted Markdown (headings, lists, code blocks, tables)
- Useful for reports, documentation, or formatted summaries
- Supports download as `.md`

---

## 7. Schema Templates

Schema templates define the JSON structure the model is asked to return. They are used in JSON and Table output modes.

### 7.1 Built-in Templates

| Template Name | Description |
|---|---|
| **Key-Value Pairs** | Simple flat object of string fields |
| **Sentiment Analysis** | Classify text with `label`, `score`, `explanation` |
| **Entity Extraction** | Extract `entities[]` with `name`, `type`, `context` |
| **Structured Summary** | Summarize into `title`, `summary`, `key_points[]`, `takeaways[]` |
| **Classification** | Assign `category`, `confidence`, `reasoning` |
| **Q&A Pair** | Generate `question`, `answer`, `source_quote` |
| **Action Items** | Extract `action_items[]` with `task`, `owner`, `due_date` |

### 7.2 Custom Schema

Users can define their own schema by selecting the **Custom Schema** option, which opens a schema editor:

- A JSON editor pane with monospace font
- A "Validate Schema" button that checks for valid JSON syntax
- A "Preview Injection" button that shows exactly how the schema will be embedded in the system prompt

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

- **Empty state** — Shows a placeholder message before any request is sent
- **Model badge** — Shows which provider and model generated the response (color-coded by provider)
- **Latency** — Time from request send to full response received (ms)
- **Token usage** — Prompt / completion / total token counts
- **Output display** — Formatted per the active output mode (Section 6)
- **Response actions:**
  - Copy — Copy response to clipboard
  - Download — Download response as file (format depends on output mode)
  - Re-run — Resends the exact same request
  - View Request — Toggles display of the full request payload sent to the API (headers redacted)

---

## 9. Compare Mode

The Compare Models section allows users to send the same prompt to multiple models simultaneously and view responses side by side.

### 9.1 Usage

1. Expand the "Compare Models" section
2. Select 2-4 models from any configured providers
3. Click "Send Request" to query all selected models simultaneously
4. View responses in a grid layout with each model's output displayed in its own card

### 9.2 Compare Display

Each comparison card shows:
- Model badge with provider color
- Response latency and token metrics
- Full response text (scrollable if long)
- Loading state while waiting for response
- Error state if a specific model fails

> **Note:** Compare mode requires API keys configured for each provider whose models you select.

---

## 10. Error Handling

All API errors are caught and surfaced in a dismissible error banner above the response panel. Each error includes a human-readable explanation and a suggested resolution.

### 10.1 Error Display

- **Error banner** — Appears above the response panel with error title and message
- **Error popup** — A toast notification in the bottom-right corner with a link to the Error Guide
- **Error Guide** — A dedicated page (`errors.html`) with detailed explanations and resolutions for all error types

### 10.2 Error Categories

#### Authentication Errors

| HTTP Code | Error ID | Title |
|---|---|---|
| 401 | `AUTH_INVALID_KEY` | Invalid API Key |
| 403 | `AUTH_INSUFFICIENT_PERMISSIONS` | Access Denied |

#### Rate Limiting & Quota Errors

| HTTP Code | Error ID | Title |
|---|---|---|
| 429 | `RATE_LIMIT_REQUESTS` | Too Many Requests |
| 429 | `RATE_LIMIT_TOKENS` | Token Quota Exceeded |
| 402 | `QUOTA_EXHAUSTED` | Account Quota Exhausted |

#### Request Errors

| HTTP Code | Error ID | Title |
|---|---|---|
| 400 | `INVALID_REQUEST_BODY` | Bad Request |
| 400 | `CONTEXT_LENGTH_EXCEEDED` | Prompt Too Long |
| 400 | `INVALID_MODEL` | Unknown Model |
| 422 | `SCHEMA_PARSE_FAILURE` | Response Parse Error |

#### Server & Network Errors

| HTTP Code | Error ID | Title |
|---|---|---|
| 500 | `PROVIDER_SERVER_ERROR` | Provider Error |
| 503 | `PROVIDER_UNAVAILABLE` | Service Unavailable |
| — | `NETWORK_TIMEOUT` | Request Timed Out |
| — | `NETWORK_OFFLINE` | No Internet Connection |
| — | `CORS_BLOCKED` | Request Blocked (CORS) |

#### Application Errors

| Error ID | Title |
|---|---|
| `NO_API_KEY` | No API Key Set |
| `NO_MODEL_SELECTED` | No Model Selected |
| `EMPTY_PROMPT` | Prompt is Empty |
| `INVALID_JSON_SCHEMA` | Invalid Schema |

---

## 11. CORS Limitations

### 11.1 What is CORS?

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that restricts web pages from making HTTP requests to a domain different from the one that served the page. If the app attempts to call an API on a different domain, the browser enforces CORS rules before allowing it to proceed.

### 11.2 Provider CORS Support

| Provider | Browser-Direct Calls | Notes |
|---|---|---|
| Anthropic | Blocked | Requires a backend proxy |
| OpenAI | Blocked | No CORS support for browser requests |
| Google Gemini | Partial | Some endpoints may work; behavior varies |
| Mistral | Blocked | Backend-only API |
| Meta (Groq/Together) | Varies | Depends on hosting provider's CORS policy |

### 11.3 Resolution Options

**Option 1: Proxy Mode (Recommended)**
Route all API requests through a lightweight backend proxy server. Configure the proxy URL in Settings. The proxy forwards requests to the provider and returns responses to the browser.

**Option 2: Local Proxy / Desktop Mode**
If running locally, some providers permit requests from localhost origins. A local proxy (Express, FastAPI, etc.) on the same machine can forward requests without triggering CORS.

> **Security Warning:** When using a proxy, ensure it is a server you control. Never route your API keys through a third-party proxy service you do not trust.

---

## 12. Settings

| Setting | Type | Default | Description |
|---|---|---|---|
| Request Timeout | Number (5-120 seconds) | `30` | How long to wait before surfacing a timeout error |
| Proxy URL | Text input | *(empty)* | Optional HTTP proxy for routing API requests |
| Theme | Select | System | Light / Dark / System |
| Save Prompt History | Toggle | On | Whether to persist sent prompts in local history |
| Max History Items | Number (1-200) | `50` | Maximum number of past prompts to store |
| Default Output Mode | Select | Plain Text | Pre-select an output mode on load |

---

## 13. History

The History panel shows previously sent prompts with:

- Model used
- Truncated user message preview
- Timestamp

Clicking a history item loads the prompt configuration back into the composer.

---

## 14. Browser Storage Schema

Data is stored under the `llm_switchboard::` namespace using two storage mechanisms:

### 14.1 Session Storage (cleared when tab closes)

| Key | Type | Description |
|---|---|---|
| `llm_switchboard::key::{provider}` | String | API key per provider (anthropic, openai, google, meta, mistral, custom) |

### 14.2 Local Storage (persists across sessions)

| Key | Type | Description |
|---|---|---|
| `llm_switchboard::settings` | JSON object | User settings (Section 12) |
| `llm_switchboard::history` | JSON array | Array of past request/response pairs |
| `llm_switchboard::schemas` | JSON array | User-saved custom schemas |
| `llm_switchboard::last_model` | String | Last selected `provider::model_id` |
| `llm_switchboard::custom_provider` | JSON object | Custom provider configuration |

---

## 15. UI Layout

### 15.1 Header

- Logo with app name "LLM Switchboard"
- Navigation links: Error Guide, History, Settings

### 15.2 Main Container

Two-panel layout (side-by-side on desktop, stacked on mobile):

**Left Panel (Input):**
- API Keys section (collapsible, with provider tabs)
- Model Selection section (collapsible)
- Prompt Composer section (collapsible)
- Example Prompts section (collapsible, collapsed by default)
- Output Mode section (collapsible)
- Compare Models section (collapsible, collapsed by default)
- Send Request button

**Right Panel (Response):**
- Response actions bar
- Error banner (when applicable)
- Response metadata (model, latency, tokens)
- Response content area
- Request metadata (expandable)
- Compare grid (when in compare mode)

### 15.3 Modals

- **Settings Modal** — Configure app settings
- **History Modal** — Browse and reload past prompts

---

## 16. Theming

The app supports three theme modes:

- **System** — Follows OS preference (light/dark)
- **Dark** — Dark background with light text (default appearance)
- **Light** — Light background with dark text

Provider-specific accent colors:
- Anthropic: `#d4a574` (tan)
- OpenAI: `#10a37f` (green)
- Google: `#4285f4` (blue)
- Meta: `#0668E1` (blue)
- Mistral: `#ff7000` (orange)

---

## 17. Non-Goals (v1.x)

- Multi-turn conversation / chat history threading
- Image, audio, or file input (vision/multimodal support)
- Server-side key storage or user accounts
- Streaming responses
- Fine-tuned or custom model endpoints
- Automatic model benchmarking or cost comparison
