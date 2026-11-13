---
title: Embedding Result Reader Contract
description: Constructional contract so an empty embedding list is returned as-is and a null or non-dict result does not raise while the bridge reads the queue response.
last_updated: 2026-11-13
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Embedding Result Reader Contract

## Problem

`ollama_embeddings` and `openai_embeddings` in `qm-llm-bridge/app.py` read the queue body with one `or` chain:

```text
embedding = qm_result.get("embedding") or qm_result.get("result", {}).get("embedding") or []
```

An empty list is falsy in Python, so a present `embedding: []` is discarded and the chain evaluates `result`. When `result` is null, `.get` raises `AttributeError`. When `result` is a string, `.get` raises the same error. When `result` is a dict that holds a different vector, the empty list is replaced by that other vector.

Checked on 2026-11-13 with Python 3.12.3 by running that expression. The bridge process was not started. The queue manager was not contacted.

| Queue body | What the expression returns |
|---|---|
| `{"embedding": [0.1, 0.2]}` | `[0.1, 0.2]` |
| `{"embedding": []}` | `[]` (the `result` key is absent, so the default `{}` supplies no vector) |
| `{"embedding": [], "result": null}` | `AttributeError: 'NoneType' object has no attribute 'get'` |
| `{"result": null}` | the same `AttributeError` |
| `{"embedding": null, "result": "not-a-dict"}` | `AttributeError: 'str' object has no attribute 'get'` |
| `{"result": {"embedding": [0.5]}}` | `[0.5]` |
| `{"embedding": [], "result": {"embedding": [0.9]}}` | `[0.9]` |

The last row is a successful empty vector overwritten by a nested vector. A retrieval client that stores `data[].embedding` cannot tell "the model returned no dimensions" from "the reader jumped to another field".

`/api/embeddings` requires a non-empty string `prompt`. `/v1/embeddings` accepts `input`. Both routes use this reader after the queue call. This page does not change how `input` is turned into queue text.

Stage 75, on branch `cursor/suggestions-folder-structure-6eff`, owns the newline join that sends one `text` field for a list `input`. Stage 74, on branch `cursor/community-suggestions-system-1471`, owns `_extract_response_text` when a completed chat string contains the word `result`. Embedding routes do not call that helper.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `POST /api/embeddings` | Ollama-style route; keeps the `prompt` string check |
| `POST /v1/embeddings` | OpenAI-style route; keeps the `input` handling from Stage 75 |
| `POST /embeddings/{model}` | The single-text queue call both routes already make |
| `embedding` and `result.embedding` | The two fields this reader already understands |
| Stage 75 batch page (other branch) | One vector per input item; leave that join |
| Stage 74 queue text page (other branch) | Chat string parsing; leave it |

Out of scope:

- A new embedding server or batch RPC
- Changing `_extract_response_text` or the chat routes
- Starting the bridge or the queue to prove this page

## Proposal

Keep both routes and the same queue call. Read the list that is already on the body. Call `.get` on `result` only after confirming it is a dict.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Present list | `ollama_embeddings` and `openai_embeddings` | When `embedding` is a list, return that list, including `[]` |
| Nested list | same reader | When `embedding` is missing or not a list, and `result` is a dict, return `result["embedding"]` when that value is a list, including `[]` |
| Other shapes | same reader | Return `[]` when neither field is a list. A null `result` and a string `result` take this path |
| Ollama route | `/api/embeddings` | Leave the non-empty string `prompt` check |
| OpenAI route | `/v1/embeddings` | Leave the Stage 75 input join and the `data[].index` response |

### Rules

1. **An empty list is a value.** `{"embedding": []}` returns `[]` and does not read `result`.
2. **Null is not a dict.** `{"result": null}` and `{"embedding": [], "result": null}` return `[]` and do not raise.
3. **A string result is not a dict.** `{"result": "not-a-dict"}` returns `[]` and does not raise.
4. **A nested list still works.** `{"result": {"embedding": [0.5]}}` returns `[0.5]` when the top-level `embedding` key is absent.
5. **The batch join stays on Stage 75.** This page does not split list `input` and does not add queue calls.
6. **Chat parsing stays on Stage 74.** This page does not change `_extract_response_text`.

### Suggested contributor checklist

```text
1. Body {"embedding": [0.1, 0.2]} returns that list on both embedding routes
2. Body {"embedding": []} returns [] and does not read result
3. Body {"embedding": [], "result": null} returns [] and does not raise
4. Body {"result": null} returns [] and does not raise
5. Body {"embedding": null, "result": "not-a-dict"} returns [] and does not raise
6. Body {"result": {"embedding": [0.5]}} returns [0.5]
7. Confirm /api/embeddings still requires a non-empty string prompt
8. Confirm the bridge process was not started
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| One vector field | The `embedding` list the queue body already has | A new response schema |
| Wrapped vector | `result` only when `isinstance(result, dict)` | `or` on a list |
| Empty vector | Return `[]` | Treating `[]` as missing |
| Intake | GitHub Issue | A separate embedding tracker |

### Acceptance check

- [ ] `{"embedding": [0.1, 0.2]}` is returned unchanged from both embedding routes.
- [ ] `{"embedding": []}` returns `[]` and does not consult `result`.
- [ ] `{"embedding": [], "result": null}` returns `[]` and does not raise `AttributeError`.
- [ ] `{"result": null}` and `{"result": "not-a-dict"}` return `[]` and do not raise.
- [ ] `{"result": {"embedding": [0.5]}}` returns `[0.5]` when the top-level embedding is absent.
- [ ] `/api/embeddings` still rejects a missing or blank `prompt`.
- [ ] `/v1/embeddings` still uses the Stage 75 input join.
- [ ] The bridge process was not started to prove this page.

### Rollback

Restore the `or` chain on both routes. An empty list again falls through to `result`, and a null `result` again raises `AttributeError`, which is today's behavior.
