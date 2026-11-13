---
title: Queue Response Text Contract
description: Constructional contract so a completed queue-manager string reply is read with _first_text and the word result does not crash the bridge.
last_updated: 2026-11-10
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Queue Response Text Contract

## Problem

`_extract_response_text` assumes its argument is a dict. The first test is `"result" in payload`. For a string, Python treats `in` as a substring search. When the substring matches, `payload["result"]` indexes that string with a word and raises `TypeError`.

`_qm_submit_and_wait` returns `status_payload.get("result", {})` after status `completed`. A queue body whose `result` is a string is therefore the value this function receives. `/api/chat`, `/api/generate`, and `/v1/chat/completions` all pass that value through `_extract_response_text` before they build the Open WebUI payload with `_ollama_done_payload`.

Checked on 2026-11-10 with Python 3.12.3 by calling the same tests. The bridge process was not started. The queue manager was not contacted.

| Value returned as `result` | `"result" in value` | Outcome |
|---|---|---|
| `hello` | false | `_first_text` returns `hello` |
| `the result is 42` | true | `TypeError: string indices must be integers, not 'str'` |
| `{"text": "ok"}` | false | returns `ok` |
| `{"result": {"text": "nested"}}` | true | returns `nested` |

A reply that happens to contain the word `result` fails. A reply that does not contain that word is returned. Dict results keep the current key walk, including a nested `result` object.

`_first_text` already accepts `str`, `list`, `dict`, and everything else. The dict branch already looks for `content`, `text`, `response`, `caption`, `output`, and `result` before it falls through to `str(value)`.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `_first_text` | Reader for strings, lists, and nested dict fields |
| `_extract_response_text` | The one function the chat routes use to turn a queue result into text |
| `_qm_submit_and_wait` | Returns the completed `result` field to those routes |
| `_ollama_done_payload` | Builds the assistant message from the text that was extracted |
| `/api/chat`, `/api/generate`, `/v1/chat/completions` | Callers that already share this helper |

Out of scope:

- A new chat gateway or a second queue client
- Changing queue-manager request bodies or the `chat-vllm` service name
- Streaming behavior on these routes
- Embedding routes, which do not call `_extract_response_text`

## Proposal

Keep this bridge. If the completed result is not a dict, return `_first_text` of that value. Keep the dict branch that unwraps a `result` key.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Non-dict result | `_extract_response_text` | When `payload` is not a dict, return `_first_text(payload)` before any `in` test |
| Dict result | `_extract_response_text` | Keep `payload["result"]` only after confirming `payload` is a dict and the key exists |
| Missing key | `_extract_response_text` | Keep the fall-through to `_first_text(payload)` |
| Callers | the three chat routes | Keep passing the value from `_qm_submit_and_wait` into this helper |
| Wait loop | `_qm_submit_and_wait` | Keep returning the `result` field, including when that field is a string |

### Rules

1. **A string is text.** `the result is 42` is the assistant reply, not a mapping.
2. **The substring test is the bug.** `"result" in payload` runs only when `payload` is a dict.
3. **Dict replies stay nested.** A dict with `result.text` still returns that text through `_first_text`.
4. **One helper.** Chat, generate, and OpenAI-compatible completions keep calling `_extract_response_text`.
5. **No new parser.** `_first_text` remains the walker for strings, lists, and known keys.
6. **The queue stays remote.** This contract does not require a running queue manager to prove the branch.

### Suggested contributor checklist

```text
1. Pass the string hello and confirm the text is hello
2. Pass the string the result is 42 and confirm the text is that whole sentence
3. Pass {"text": "ok"} and confirm the text is ok
4. Pass {"result": {"text": "nested"}} and confirm the text is nested
5. Confirm /api/chat, /api/generate, and /v1/chat/completions still call this helper
6. Confirm the bridge process was not started
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Text from mixed JSON | `_first_text` | A new response schema or templating engine |
| Queue completion | `_qm_submit_and_wait` | A second poller |
| Assistant body | `_ollama_done_payload` | A custom Open WebUI plugin |
| Intake | GitHub Issue | A separate bridge tracker |

### Acceptance check

- [ ] `_extract_response_text("hello")` returns `hello`.
- [ ] `_extract_response_text("the result is 42")` returns `the result is 42` and does not raise.
- [ ] A dict with `text` still returns that text.
- [ ] A dict with a nested `result` object still returns the inner text.
- [ ] `/api/chat`, `/api/generate`, and `/v1/chat/completions` still obtain their assistant text from this function.
- [ ] The bridge process was not started to prove this page.

### Rollback

Restore the `"result" in payload` test with no dict check. A completed string that contains `result` raises `TypeError` again, which is today's behavior.
