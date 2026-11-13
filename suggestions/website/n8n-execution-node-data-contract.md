---
title: Execution Node Data Contract
description: Constructional contract so n8n_get_execution summarizes a node when data or json is null and that parse does not trip the circuit breaker.
last_updated: 2026-11-10
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Execution Node Data Contract

## Problem

`n8n_get_execution` builds a per-node summary from the last run of each entry in `runData`. Two nested fields are read as dicts even when the JSON value is null.

The sample block does this:

```text
out_main = last_run.get("data", {}).get("main", [])
...
sample = first_branch[0].get("json", {}) if isinstance(first_branch[0], dict) else {}
node_info["sampleOutput"] = {k: v for i, (k, v) in enumerate(sample.items()) if i < 5}
```

`dict.get` returns the stored null when the key exists. The default `{}` applies only when the key is missing. A present `"data": null` makes the next `.get` raise `AttributeError`. A present `"json": null` makes `.items()` raise `AttributeError`.

Checked on 2026-11-10 by replaying those expressions in Python 3.12.3. The gateway was not started. n8n was not contacted.

| Last-run shape | Result |
|---|---|
| `data` key missing, `main` holds one JSON object `{a: 1}` | Summary includes `outputCount` 1 and `sampleOutput` `{a: 1}` |
| `data` is null and `error.message` is `boom` | `AttributeError: 'NoneType' object has no attribute 'get'` |
| `data.main[0][0].json` is null | `AttributeError: 'NoneType' object has no attribute 'items'` |

`n8n_get_execution` is an n8n tool, so `invoke` sets `is_n8n_tool`. The `AttributeError` is not an `HTTPException`. The generic handler calls `CircuitBreaker.mark_failure` and returns `_as_tool_response` of an error envelope. A one-line replay of that handler moved `fail_count` from 0 to 1. The tool wrapper still sets `"error": None`. The operator loses the execution id, status, and error text that were already copied onto the summary before the node loop.

`n8n_diagnose_workflow` already avoids the top-level form of this bug: it reads `resultData` only when `detail.get("data")` is a dict. The get-execution node loop does not use that check. `n8n_get_execution` is not in `_is_write_tool`, so this page does not change write approval or dedupe.

Stage 73 (open PR #99) owns `CircuitBreaker.allow` leaving `fail_count` at the threshold after the quiet period. This page leaves that class as it is. The summarizer should not raise, so this parse does not call `mark_failure`.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `n8n_get_execution` | The summary operators already request by execution id |
| `isinstance(detail.get("data"), dict)` in `n8n_diagnose_workflow` | The guard to copy onto the node loop |
| `invoke` exception handler | Already counts real n8n `HTTPException` values through `mark_failure` |
| `CircuitBreaker` | Stays on the Stage 73 contract |
| `_as_tool_response` | Stays the envelope wrapper for a summary that returned normally |

Out of scope:

- A second execution database or a hosted n8n log viewer
- Changing `N8N_CIRCUIT_FAIL_THRESHOLD` or the quiet-period reset
- Webhook `confirm`, workflow allowlists, or list cursors
- Treating `n8n_get_execution` as a write tool

## Proposal

Keep this summary. Read `data` and `json` only when they are dicts. A null field omits the sample and still returns the execution fields that were already collected.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Node payload | `n8n_get_execution` | Bind `data = last_run.get("data")` and read `main` only when `data` is a dict |
| Sample JSON | `n8n_get_execution` | Build `sampleOutput` only when the item `json` value is a dict |
| Non-dict run | `n8n_get_execution` | When the last run entry is not a dict, skip that node the way a non-dict workflow node is already skipped |
| Error text | `n8n_get_execution` | Keep copying `error.message`, `errorNode`, and `errorDescription` before the sample block |
| Diagnose | `n8n_diagnose_workflow` | Keep the existing `isinstance(..., dict)` check on execution `data` |
| Circuit | `invoke` | Leave `mark_failure` on the `HTTPException` path. A local summary must return normally |

### Rules

1. **Null is an empty sample.** `"data": null` still returns id, status, workflow id, and the top-level error message.
2. **Null item JSON is an empty sample.** `"json": null` sets `outputCount` and omits `sampleOutput`.
3. **Missing keys stay on the current path.** A missing `data` key still uses an empty object, which is what `.get("data", {})` already did for that case.
4. **The first five keys stay.** When `json` is a dict, `sampleOutput` still keeps the first five fields.
5. **The breaker counts n8n failures.** It does not count this `AttributeError`, because the summarizer no longer raises it.
6. **Stage 73 stays the breaker contract.** `allow` and `mark_failure` keep their current source.
7. **No new component.** The summary remains inside `n8n_get_execution`.

### Suggested contributor checklist

```text
1. Summarize a run whose data key is missing and confirm sampleOutput still appears
2. Summarize a run whose data value is null and confirm no exception and no sampleOutput
3. Confirm that null-data run still includes the top-level error message
4. Summarize a branch item whose json value is null and confirm outputCount is set and sampleOutput is absent
5. Replay the invoke exception path and confirm mark_failure is not called for these shapes
6. Confirm an HTTPException from n8n_api.request still reaches mark_failure
7. Confirm CircuitBreaker.allow is unchanged from Stage 73
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Null guard | `isinstance(value, dict)` already used in `n8n_diagnose_workflow` | A schema compiler or a second parser |
| Execution body | `GET /api/v1/executions/{id}` already called by this tool | A stored copy of n8n history |
| Failure pause | Existing `mark_failure` for HTTP errors from n8n | A new breaker for parse errors |
| Intake | GitHub Issue | An out-of-band debug form |

### Acceptance check

- [ ] A last run with `"data": null` returns the execution summary and does not raise.
- [ ] That summary still includes the top-level error message when `error` is an object.
- [ ] A last run with a missing `data` key still fills `sampleOutput` from `main`.
- [ ] A branch item with `"json": null` sets `outputCount` and does not call `.items()` on null.
- [ ] These shapes do not call `CircuitBreaker.mark_failure`.
- [ ] An `HTTPException` from the n8n request still calls `mark_failure`.
- [ ] `CircuitBreaker.allow` is unchanged.
- [ ] The gateway process was not started to prove this page.

### Rollback

Restore `.get("data", {}).get("main", [])` and `sample.items()` without a dict test. A null `data` or `json` field raises `AttributeError` again and increments the circuit counter, which is today's behavior.
