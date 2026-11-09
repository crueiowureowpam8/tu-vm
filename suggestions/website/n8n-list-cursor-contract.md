---
title: n8n List Cursor Contract
description: Constructional contract so workflow and execution list tools return the nextCursor n8n already sends and executions accept a cursor.
last_updated: 2026-11-09
owner: maintainers
status: proposed
theme: operations
impact: high
---

# n8n List Cursor Contract

## Problem

`n8n_list_workflows` tells the model it can pass `cursor`. The handler copies that value onto the n8n query, then builds a new object with only `result` and `total`. `n8n_list_executions` does the same with the response, and its input schema has no `cursor` property.

Checked on 2026-11-09 by replaying the shaping loops in `mcp-gateway/app.py`. n8n was not contacted.

Workflow body used in the replay:

```json
{
  "data": [
    {"id": "1", "name": "keep", "isArchived": false},
    {"id": "2", "name": "old", "isArchived": true}
  ],
  "nextCursor": "page-2"
}
```

| Shaper | Returned keys | `total` | `nextCursor` |
|---|---|---|---|
| `n8n_list_workflows` with `includeArchived` false | `result`, `total` | 1 | Absent |
| `n8n_list_executions` | `result`, `total` | 2 for a two-row body | Absent |

The archived row is removed after the page is fetched. The cursor that would fetch the following page is removed in the same return. A caller that received one visible workflow cannot ask for `page-2`.

Execution lists have the same return shape. The schema properties are `limit` and `workflowId` only, so a later page cannot be requested even after the response keeps a cursor.

Stage 70 (open PR #96) owns `confirm=true` before a webhook dispatch. Stage 71 (open PR #97) owns which workflow id a non-empty allowlist may touch. Stage 72 (open PR #98) owns a plain-text or empty n8n success body. Those contracts stay. This page is the list cursor.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `n8n_list_workflows` input `cursor` | Already forwarded on the query |
| `includeArchived` | Already applied to each row |
| `n8n_list_executions` | The execution list to extend |
| `N8nApi.request` | The client that already returns the parsed object |
| Stage 70 `webhook-confirm-gate-contract.md` (open PR #96) | Webhook confirm. Unchanged |
| Stage 71 `n8n-workflow-allowlist-enforcement-contract.md` (open PR #97) | Allowlist holes. Unchanged |
| Stage 72 `n8n-non-json-response-contract.md` (open PR #98) | Non-JSON success bodies. Unchanged |

Out of scope:

- A second n8n client or a cached workflow index
- Changing which workflows an allowlist may touch
- Changing the webhook confirm check
- Parsing rules for a non-JSON success body

## Proposal

Keep the list tools. Copy `nextCursor` when the response has it, and let execution lists take the same `cursor` input workflows already take.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Workflow response | `n8n_list_workflows` | When the parsed body has `nextCursor`, include that string on the tool result |
| Archived rows | The same handler | Filtering `isArchived` must not drop `nextCursor` |
| Execution request | `n8n_list_executions` schema and query | Add `cursor`, and forward it the way the workflow list already does |
| Execution response | `n8n_list_executions` | Copy `nextCursor` the same way |

### Rules

1. **The cursor is part of the page.** A tool result that came from a body containing `nextCursor` includes that value.
2. **Absent stays absent.** A body with no `nextCursor` does not gain an empty string.
3. **Archived filtering keeps the page token.** Hiding archived rows changes `result` and `total`. It leaves `nextCursor` in place.
4. **Executions can continue.** The execution schema accepts `cursor`, and the handler sends it on the n8n query.
5. **Names stay.** The tools remain `n8n_list_workflows` and `n8n_list_executions`.
6. **Allowlist, confirm, and JSON parsing stay with their owners.** Stage 70, Stage 71, and Stage 72 pages stay as written.
7. **No new service.** `N8nApi.request` remains the HTTP client.

### Suggested contributor checklist

```text
1. Replay a workflow body with two rows, one archived, and nextCursor page-2
2. Confirm the tool result total is 1 and nextCursor is still page-2
3. Replay a workflow body with no nextCursor and confirm the tool result has no nextCursor key
4. Pass cursor page-2 into n8n_list_workflows and confirm the outgoing query contains that cursor
5. Pass cursor exec-2 into n8n_list_executions and confirm the outgoing query contains that cursor
6. Replay an execution body with nextCursor exec-2 and confirm the tool result keeps it
7. Confirm webhook confirm, the allowlist, and non-JSON success bodies still follow Stages 70–72
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Page token | `nextCursor` on the body `N8nApi.request` already parsed | A private offset index |
| Request field | The `cursor` property already on `n8n_list_workflows` | A new list tool |
| Intake | GitHub Issue | A workflow-catalog product |

### Acceptance check

- [ ] `n8n_list_workflows` returns `nextCursor` when the n8n body has it.
- [ ] An archived row removed from `result` does not remove `nextCursor`.
- [ ] A body without `nextCursor` produces a tool result without that key.
- [ ] `n8n_list_executions` accepts `cursor` and places it on the n8n query.
- [ ] `n8n_list_executions` returns `nextCursor` when the n8n body has it.
- [ ] Stage 70, Stage 71, and Stage 72 contracts are unchanged.
- [ ] n8n was not contacted to prove this page.

### Rollback

Return only `result` and `total` from both list handlers, and remove `cursor` from the execution schema if it was added. Listing again matches today's first page only.
