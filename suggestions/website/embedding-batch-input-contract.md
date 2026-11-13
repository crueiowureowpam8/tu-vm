---
title: Embedding Batch Input Contract
description: Constructional contract so each string in an OpenAI embedding input is embedded with the existing single-text queue call and returned at its own index.
last_updated: 2026-11-11
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Embedding Batch Input Contract

## Problem

`openai_embeddings` in `qm-llm-bridge/app.py` accepts the OpenAI field `input`, then sends one string to the queue.

```text
if isinstance(input_text, list):
    merged = "\n".join(str(v) for v in input_text)
else:
    merged = str(input_text or "")
POST /embeddings/{model}  {"text": merged}
return data: [{index: 0, embedding: <one vector>}]
```

A retrieval client sends a list of chunks and expects `len(data) == len(input)`, with `data[i].index == i`. This route always returns one item. The vector belongs to the joined string, so chunk 0 and chunk 1 cannot be stored as separate points.

Checked on 2026-11-11 with Python 3.12.3 by running those branches. The bridge process was not started. The queue manager was not contacted.

| `input` | Queue body | Response `data` |
|---|---|---|
| `["alpha", "beta"]` | `{"text": "alpha\nbeta"}` | length 1, index 0 |
| `"alpha"` | `{"text": "alpha"}` | length 1, index 0 |
| `["  ", "beta"]` | `{"text": "  \nbeta"}` | length 1, index 0 |
| `["alpha\nbeta"]` | `{"text": "alpha\nbeta"}` | length 1, index 0 |
| `[]` or `""` | not sent | HTTP 400, `Field 'input' must be non-empty` |

`["alpha", "beta"]` and `["alpha\nbeta"]` produce the same queue body. The queue cannot tell a two-chunk batch from one chunk that contains a newline. A later reader cannot split the single vector back into the original items.

`/api/embeddings` is a different route. It requires a non-empty string `prompt` and does not accept `input`. This page does not change that route.

Stage 74, on branch `cursor/community-suggestions-system-1471`, owns `_extract_response_text` when a completed chat string contains the word `result`. Embedding routes do not call that helper.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `POST /v1/embeddings` | The OpenAI-compatible route Open WebUI and other clients already call |
| `POST /embeddings/{model}` with `{"text": ...}` | The single-text queue call this route already makes |
| `data[].index` and `data[].embedding` | The response fields clients already read |
| `/api/embeddings` | Ollama-style prompt route; leave it |
| Stage 74 queue text page (other branch) | Chat string parsing; leave it |

Out of scope:

- A new embedding server, model host, or batch RPC
- The sentence-transformer cache on the Open WebUI container
- Changing `_extract_response_text` or the chat routes
- Starting the bridge or the queue to prove this page

## Proposal

Keep this bridge and the single-text queue call. When `input` is a list, call that endpoint once per item and place each vector at the item's index. A string stays one call and index 0.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| String input | `openai_embeddings` | Keep one `POST /embeddings/{model}` with `{"text": input}` and `data` length 1 at index 0 |
| List input | `openai_embeddings` | One existing POST per item. `data` length equals the list length. `index` is the position in `input` |
| Blank list | `openai_embeddings` | Keep HTTP 400 when there is no non-empty text to embed |
| Blank item inside a list | `openai_embeddings` | HTTP 400 that names the index. Do not join that item onto a neighbor with a newline |
| Queue | `/embeddings/{model}` | Keep the body `{"text": "<one string>"}` |
| Ollama route | `/api/embeddings` | Leave the `prompt` string check as it is |

### Rules

1. **One item, one vector.** `["alpha", "beta"]` returns two `data` objects, indexes 0 and 1.
2. **The newline is content.** A one-item list whose text is `alpha\nbeta` is one vector, and its queue body is that whole string.
3. **Indexes follow the input.** Position `i` in the list is `data` index `i`, including when calls run one after another.
4. **Reuse the queue body.** Each call still posts `{"text": ...}` to `/embeddings/{model}`.
5. **Empty input stays a 400.** `""` and `[]` do not reach the queue.
6. **Chat parsing stays on Stage 74.** This page does not change `_extract_response_text`.

### Suggested contributor checklist

```text
1. POST input "alpha" and confirm data length 1 at index 0
2. POST input ["alpha", "beta"] and confirm two queue texts, "alpha" and "beta", and data indexes 0 and 1
3. POST input ["alpha\nbeta"] and confirm one queue text equal to that whole string
4. POST input "" and [] and confirm HTTP 400 with no queue call
5. POST input ["  ", "beta"] and confirm the blank index is a 400, not a joined string
6. Confirm /api/embeddings still requires a string prompt
7. Confirm the bridge process was not started
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| One string in, one vector out | The existing `/embeddings/{model}` POST | A batch embedding product |
| Several chunks | A loop of that same POST | Joining chunks with newlines |
| Response shape | `object: list` and `data[].index` already returned | A custom retrieval schema |
| Intake | GitHub Issue | A separate embedding tracker |

### Acceptance check

- [ ] `input` `"alpha"` returns one embedding at index 0.
- [ ] `input` `["alpha", "beta"]` returns two embeddings, at indexes 0 and 1, from two single-text queue calls.
- [ ] `input` `["alpha\nbeta"]` is one queue text, `alpha\nbeta`, and one embedding.
- [ ] `""` and `[]` return HTTP 400 and do not call the queue.
- [ ] A blank item inside a non-empty list does not get joined onto another item.
- [ ] `/api/embeddings` still requires a non-empty string `prompt`.
- [ ] The bridge process was not started to prove this page.

### Rollback

Restore the newline join and the single `data` item at index 0. A list input again embeds one combined string, which is today's behavior.
