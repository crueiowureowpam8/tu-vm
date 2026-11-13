---
title: Circuit Breaker Threshold Contract
description: Constructional contract so CircuitBreaker clears its failure count when the open window ends and the configured threshold applies again.
last_updated: 2026-11-09
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Circuit Breaker Threshold Contract

## Problem

`CircuitBreaker` opens after `N8N_CIRCUIT_FAIL_THRESHOLD` failures (default 8) and stays open for `N8N_CIRCUIT_OPEN_SECONDS` (default 60). `allow` refuses calls while `open_until` is in the future. It does not change `fail_count` when that time has passed. The next failure increments a counter that is already at the threshold, so the circuit opens again on that single call.

Checked on 2026-11-09 by replaying the class rules in Python 3.12.3. The gateway was not started. n8n was not contacted. The replay used threshold 3 and a 60 second window so the table stays short. The source defaults are 8 and 60. The shape is the same: the counter is left at the threshold.

| Step | `fail_count` | `allow` |
|---|---|---|
| Three failures at t=1000 | 3 | The third failure sets `open_until` to 1060 |
| `allow` at t=1001 | 3 | `CIRCUIT_OPEN` |
| `allow` at t=1061 | 3 | Permitted. The counter is still 3 |
| One failure at t=1061 | 4 | `open_until` moves forward again |
| `allow` at t=1062 | 4 | `CIRCUIT_OPEN` |

A success still clears the counter. A second replay opened the circuit, waited past the window, called `mark_success`, and observed `fail_count` 0 and `open_until` 0.

Clearing the counter on every `allow` would be wrong. While failures are still accumulating, `open_until` is 0. That in-progress count has to survive until it reaches the threshold.

Stage 72 (open PR #98) owns the tool catalog when AFFiNE auth is missing, and the plain-text n8n success body. This page leaves both of those handlers as they are. The breaker is the pause around n8n tool calls.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `CircuitBreaker` | The counter and the open window |
| `N8N_CIRCUIT_FAIL_THRESHOLD` | The count that should apply after every quiet period |
| `N8N_CIRCUIT_OPEN_SECONDS` | The quiet period to keep |
| `mark_success` | Already resets `fail_count` and `open_until` |
| `invoke` | Already calls `allow` before an n8n tool and `mark_failure` on a raised error |
| Stage 72 website pages (open PR #98) | Catalog and response-body contracts. Unchanged |

Out of scope:

- A hosted status page or a second breaker implementation
- Changing the default threshold or the default quiet period
- Counting failures for tools that are not n8n tools
- Retrying the failed n8n call inside the breaker

## Proposal

Keep this class. When the quiet period has elapsed, start the failure count from zero before the next trial.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Half-open entry | `CircuitBreaker.allow` | If `open_until` is greater than 0 and no longer in the future, set `fail_count` to 0 and `open_until` to 0, then permit the call |
| Still open | `CircuitBreaker.allow` | If `open_until` is in the future, keep raising `CIRCUIT_OPEN` |
| Accumulating | `CircuitBreaker.allow` | If `open_until` is 0, leave `fail_count` unchanged |
| Success | `mark_success` | Keep the reset that already sets both fields to 0 |
| Trip | `mark_failure` | Keep opening when `fail_count` reaches the threshold |

### Rules

1. **The threshold is the threshold.** After a quiet period, `N8N_CIRCUIT_FAIL_THRESHOLD` new failures are required to open the circuit again.
2. **One expired window clears the old count.** `allow` does this only when `open_until` was set and that time has passed.
3. **An open window still refuses.** Calls during `open_until` raise `CIRCUIT_OPEN` and do not increment `fail_count`.
4. **In-progress failures stay.** A counter of 2 with `open_until` 0 is still 2 on the next `allow`.
5. **Success still heals.** `mark_success` continues to zero the counter and the window.
6. **Defaults stay.** 8 failures and 60 seconds remain the defaults in code.
7. **No new component.** The class in `mcp-gateway/app.py` remains the breaker.

### Suggested contributor checklist

```text
1. Open the breaker with threshold 3 and confirm allow refuses during the window
2. Advance past open_until, call allow, and confirm fail_count is 0
3. Record two failures and confirm the circuit stays closed
4. Record the third failure and confirm the circuit opens
5. With open_until still 0 and fail_count 2, call allow and confirm fail_count is still 2
6. Call mark_success after the window and confirm fail_count and open_until are 0
7. Confirm the AFFiNE catalog failure and the non-JSON n8n body still follow Stage 72
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Pause | `CircuitBreaker.allow` | A proxy product that tracks n8n errors |
| Limits | `N8N_CIRCUIT_FAIL_THRESHOLD` and `N8N_CIRCUIT_OPEN_SECONDS` | Hard-coded replacement numbers |
| Intake | GitHub Issue | An out-of-band on-call form |

### Acceptance check

- [ ] After `open_until` passes, the next `allow` sets `fail_count` to 0.
- [ ] That same call is permitted.
- [ ] The following failures must reach `N8N_CIRCUIT_FAIL_THRESHOLD` before `CIRCUIT_OPEN` returns.
- [ ] `allow` during an open window still raises `CIRCUIT_OPEN`.
- [ ] `allow` while `open_until` is 0 does not clear an in-progress `fail_count`.
- [ ] `mark_success` still sets `fail_count` and `open_until` to 0.
- [ ] The gateway process was not started to prove this page.

### Rollback

Remove the expired-window reset from `allow`. The next failure after a quiet period opens the circuit again, which is today's behavior.
