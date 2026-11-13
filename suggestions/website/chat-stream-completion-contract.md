---
title: Chat Stream Completion Contract
description: Constructional contract so stream true on the queue bridge finishes as one NDJSON line or one SSE chunk plus [DONE], using the text the bridge already has.
last_updated: 2026-11-11
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Chat Stream Completion Contract

## Problem

Open WebUI and other OpenAI-compatible clients set `stream` to true when the operator expects a live reply. The queue bridge never reads that field.

Checked on 2026-11-11 by parsing `qm-llm-bridge/app.py` with the Python 3.12.3 `ast` module. The constant `stream` does not appear in these functions:

| Function | Line | What it returns |
|---|---|---|
| `ollama_chat` | 162 | the dict from `_ollama_done_payload` |
| `ollama_generate` | 175 | that dict plus `response` |
| `openai_chat_completions` | 215 | one `chat.completion` object |

`stream: true`, `stream: false`, and a body that omits `stream` all take the same return. The bridge process was not started. The queue manager was not contacted.

`_qm_submit_and_wait` already blocks until the queue status is `completed`, then the route builds the full assistant text. There is no partial token to forward. The gap is the response shape after that wait.

An OpenAI client that sent `stream: true` reads `text/event-stream` lines that start with `data: ` and stops on `data: [DONE]`. The object this route returns, encoded as JSON, is one document. It does not start with `data:` and it does not contain `[DONE]`. The client waits until its own timeout and shows an empty or failed reply.

An Ollama client that sent `stream: true` reads `application/x-ndjson` and stops on an object whose `done` field is true. `_ollama_done_payload` already sets `done: true` and `done_reason: stop`. The route returns that object as a normal JSON document, with `Content-Type: application/json`, rather than as one NDJSON line.

Leaving `stream` unset must keep today's single JSON object. Callers that already work against this bridge did not send `stream: true`.

Stage 74, on branch `cursor/community-suggestions-system-1471`, owns a completed string that contains the word `result`. This page does not change `_extract_response_text`. It only changes how a finished string is written when `stream` is true.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `_qm_submit_and_wait` | Already waits for queue status `completed` |
| `_extract_response_text` | Already turns that result into the assistant string |
| `_ollama_done_payload` | The Ollama object with `done: true` |
| `/api/chat` and `/api/generate` | Ollama routes that ignore `stream` |
| `/v1/chat/completions` | OpenAI route that ignores `stream` |
| FastAPI response classes | Already available for a media type and a body |

Out of scope:

- A token-by-token queue protocol or a second model gateway
- Changing the wait loop, the model name, or the `chat-vllm` service
- Treating a missing `stream` field as a stream
- Embedding routes, which have no `stream` field

## Proposal

Keep the completed-text path. Read `stream` only when the JSON value is boolean true. Write the text the route already built in the shape that client asked for.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Missing or false | all three chat routes | Keep the current JSON object |
| Ollama true | `/api/chat`, `/api/generate` | One NDJSON line: the existing done payload. Generate also keeps `response` on that object. Media type `application/x-ndjson` |
| OpenAI true | `/v1/chat/completions` | One `data: ` line whose chunk contains the full assistant text, then a line `data: [DONE]`. Media type `text/event-stream` |
| Queue | `_qm_submit_and_wait` | Keep waiting for `completed` before any of those writes |
| Text | `_extract_response_text` | Keep the current call. Stage 74 owns the string-`result` fix |

### Rules

1. **Only an explicit true streams.** A missing field and `false` stay one JSON object.
2. **The text is already complete.** The stream body is that string once. Do not open a second poll.
3. **Ollama uses the done payload.** `done` is true on the single NDJSON line. Generate keeps the `response` field.
4. **OpenAI uses two lines.** One `data: ` chunk, then `data: [DONE]`.
5. **Chat parsing stays on Stage 74.** This page does not change how the string is extracted.
6. **The bridge stays stopped for the proof.** The AST check and a JSON encode of the current return are enough.

### Suggested contributor checklist

```text
1. Confirm the three chat functions contain no stream constant today
2. Omit stream and send stream false; confirm the current JSON object
3. Send stream true to /api/chat and confirm one NDJSON line with done true
4. Send stream true to /api/generate and confirm that line also has response
5. Send stream true to /v1/chat/completions and confirm a data line plus data: [DONE]
6. Confirm a stream body does not call the queue a second time
7. Confirm the bridge process was not started
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Finished Ollama body | `_ollama_done_payload` | A new chat schema |
| Finished OpenAI body | The `chat.completion` fields already built, written as one chunk | A token protocol |
| Bytes on the wire | FastAPI streaming response with the media type above | A sidecar stream proxy |
| Intake | GitHub Issue | A separate bridge tracker |

### Acceptance check

- [ ] The three chat functions, as checked out today, do not mention `stream`.
- [ ] A request that omits `stream`, and a request with `stream: false`, still return one JSON object.
- [ ] `/api/chat` with `stream: true` returns one `application/x-ndjson` line whose `done` value is true.
- [ ] `/api/generate` with `stream: true` returns that line and includes `response`.
- [ ] `/v1/chat/completions` with `stream: true` returns `text/event-stream`, one `data: ` chunk for the full text, and `data: [DONE]`.
- [ ] Each of those requests still performs one queue wait.
- [ ] The bridge process was not started to prove this page.

### Rollback

Return the dict from each route for every `stream` value. A client that sends `stream: true` again receives one JSON document and no `[DONE]` line, which is today's behavior.
