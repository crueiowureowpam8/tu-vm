---
title: n8n Non-JSON Response Contract
description: Constructional contract so a successful empty or plain-text n8n body is returned by N8nApi.request instead of a 502 JSON parse error.
last_updated: 2026-11-08
owner: maintainers
status: proposed
theme: tooling
impact: high
---

# n8n Non-JSON Response Contract

## Problem

`N8nApi.request` is the only n8n HTTP client in the gateway. Workflow reads, writes, and webhook calls all use it. Every status below 400 is parsed as JSON. A webhook that answers in plain text, and an empty success body, become a gateway error.

The handler does this:

```text
if resp.status_code >= 400:
    raise HTTPException(resp.status_code, "n8n API error: ...")
try:
    return resp.json()
except (json.JSONDecodeError, ValueError) as exc:
    raise HTTPException(502, "n8n returned invalid JSON: ...")
```

`resp.json()` uses `json.loads` on the body. Python 3.12 replay of that `except` pair, without contacting n8n:

| Status | Body | Result |
|---|---|---|
| 200 | `ok` | 502 `n8n returned invalid JSON` |
| 200 | empty | 502 `n8n returned invalid JSON` |
| 204 | empty | 502 `n8n returned invalid JSON` |
| 200 | `{"ok": 1}` | `{"ok": 1}` |

`n8n_run_webhook` returns whatever `request` returns, under `result`. A webhook node that responds with text never reaches that wrapper. The caller sees 502. The same parser sits on activate, deactivate, and execution delete, so an empty success body from those routes has the same shape.

n8n was not contacted on 2026-11-08. This page is the client parser. It does not claim a particular n8n build returns 204. It claims this client treats the bodies above as failure.

Stage 70 (open PR #96) owns `confirm=true` before `n8n_run_webhook` and before diagnose `testPayload`. This page does not change that gate. Stage 71 (open PR #97) owns `ensure_allowed_workflow` on list, execution, and webhook calls. This page does not change which id is allowed.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `N8nApi.request` | The parser to extend |
| `httpx` response body | Already in hand when `json()` fails |
| `n8n_run_webhook` | Caller that should receive `result` for a 200 text body |
| Stage 70 `webhook-confirm-gate-contract.md` (open PR #96) | Confirm flag. Unchanged |
| Stage 71 `n8n-workflow-allowlist-enforcement-contract.md` (open PR #97) | Allowlist. Unchanged |

Out of scope:

- A second HTTP client or a webhook proxy
- Changing n8n's response mode inside a workflow
- Relaxing 4xx and 5xx. Those stay `HTTPException` with the n8n status
- The confirm flag and the workflow allowlist

## Proposal

Parse JSON when the body is JSON. Return a small object when a success body is empty or plain text.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Empty success | `N8nApi.request` | Status below 400 and a blank body returns `{}` |
| Plain text | `N8nApi.request` | Status below 400 and a `JSONDecodeError` returns `{"raw": "<body text>"}` |
| JSON | `N8nApi.request` | Objects and arrays return as `resp.json()` does today |
| Errors | `N8nApi.request` | Status 400 and above still raise `HTTPException` with that status and the existing `n8n API error` detail |

### Rules

1. **Success text is data.** A 200 body of `ok` is `{"raw": "ok"}`, not a 502.
2. **Empty success is an object.** A blank 200 or 204 body is `{}`.
3. **JSON stays JSON.** `{"ok": 1}` still returns that object. A JSON array still returns the array.
4. **Errors stay errors.** Status 400 and above do not get the `raw` wrapper.
5. **One client.** Webhook, workflow, and execution calls keep using `N8nApi.request`.
6. **Confirm and allowlist stay.** Stages 70 and 71 are unchanged.

### Suggested contributor checklist

```text
1. Feed N8nApi.request a 200 body of ok and confirm the result is {"raw": "ok"}
2. Feed a 200 empty body and a 204 empty body and confirm the result is {}
3. Feed a 200 body of {"ok": 1} and confirm the object is unchanged
4. Feed a 404 body of not-found and confirm the status is 404, not 502
5. Call n8n_run_webhook against a stub that returns text and confirm result.raw is that text
6. Confirm confirm=true is still required before the webhook request is sent
7. Confirm a non-empty allowlist still rejects a workflow id outside the set
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| HTTP | `N8nApi.request` | A second client |
| JSON | `resp.json()` | A new parser library |
| Text | `{"raw": text}` on the same return | A side channel or a log-only success |
| Intake | GitHub Issue | An out-of-band webhook console |

### Acceptance check

- [ ] Status 200 and body `ok` returns `{"raw": "ok"}`.
- [ ] Status 200 with an empty body, and status 204 with an empty body, return `{}`.
- [ ] Status 200 and body `{"ok": 1}` returns that object.
- [ ] Status 404 still raises `HTTPException` with status 404.
- [ ] `n8n_run_webhook` still requires `confirm=true` before the request (Stage 70).
- [ ] Allowlist enforcement from Stage 71 is unchanged.
- [ ] n8n was not contacted to prove the parser.

### Rollback

Restore the `resp.json()` call and the 502 on `JSONDecodeError`. Success JSON and error statuses behave as they do today.
