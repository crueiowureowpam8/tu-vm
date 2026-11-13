---
title: Node Operation Filter Contract
description: Constructional contract so n8n_get_node_types keeps properties by an exact operation name and treats a null displayOptions object as no condition.
last_updated: 2026-11-13
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Node Operation Filter Contract

## Problem

`n8n_get_node_types` in `mcp-gateway/app.py` filters parameter properties with substring membership:

```text
show_conditions = p.get("displayOptions", {}).get("show", {})
all_values = lowercased strings from each show list
keep the property when filter_operation in some value
```

`get` is a substring of `getall`. A caller who passes `operation: "get"` therefore receives properties that are shown only for `getAll`. The tool schema names `get` and `post` as the examples of this argument. Workflow authors use the filtered list to decide which fields to set. Extra `getAll` fields make a community workflow write the wrong parameters.

The same line raises when `displayOptions` is null. `dict.get` returns the stored null, and `None.get` raises `AttributeError`. One property of that shape stops the lookup before later properties are filtered.

Checked on 2026-11-13 with Python 3.12.3 by running that comparison. `mcp-gateway/n8n_node_types.json` is gitignored and was not in the tree. The gateway was not started. The replay used four properties and the same `in` test the function uses.

| `operation` | Properties kept today |
|---|---|
| `get` | the property with no `displayOptions`, the property shown for `get`, and the property shown for `getAll` |
| `post` | the property with no `displayOptions`, and the property shown for `post` |
| `getall` | the property with no `displayOptions`, and the property shown for `getAll` |

`get` keeps `getAll`. `getall` does not keep `get`, because `getall` is not a substring of `get`. The match is not symmetric, so the shorter name is the one that over-includes.

A separate property `{"displayOptions": null}` raises `AttributeError: 'NoneType' object has no attribute 'get'` on the show lookup.

Stage 71 (open PR #97) owns the empty catalog: when `_n8n_node_types` is empty, validation emits no `unknown type` warning. This page assumes a catalog entry is already loaded and only changes how its properties are selected. Stage 72 (open PR #98) owns the tool list that returns 503 when AFFiNE connect fails. This page does not change `list_tools`.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `n8n_get_node_types` | The lookup workflow authors already call before create and update |
| `operation` argument | The filter this route already accepts |
| `displayOptions.show` | The condition lists the catalog properties already carry |
| `includeParams: false` | Compact entries; leave that branch |
| Stage 71 empty-catalog page (PR #97) | Warning when the catalog file is missing; leave it |
| `scripts/extract-n8n-node-types.sh` | The generator for the gitignored catalog; leave it |

Out of scope:

- Shipping `n8n_node_types.json` in git
- A second node database or n8n-mcp search tool
- Changing create, update, or diagnose validation
- Starting the gateway or n8n to prove this page

## Proposal

Keep this tool and the show lists it already walks. Compare the requested operation to each show value as a whole lowercase string. Treat a missing, empty, or null `displayOptions` as "no condition" and keep that property.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Exact name | `n8n_get_node_types` | Keep a property when the lowercased operation equals one lowercased show value |
| Shorter name | same loop | `get` drops a property whose only show value is `getAll` |
| Longer name | same loop | `getall` keeps `getAll` and drops `get` |
| No condition | same loop | Missing `displayOptions`, `{}`, or `displayOptions: null` keeps the property and does not raise |
| Compact mode | `includeParams: false` | Leave the short entry that reports `paramCount` |
| Empty catalog | `_validate_workflow_nodes` | Leave the Stage 71 warning |

### Rules

1. **Whole values only.** `get` matches `get` and does not match `getAll`.
2. **Case stays folded.** `getAll` is compared as `getall`, which is what the loop already does before the test.
3. **`post` stays exact.** `post` keeps `post` and does not keep `get` or `getAll`.
4. **Null means no condition.** `displayOptions: null` keeps the property and does not raise `AttributeError`.
5. **Unconditional properties stay.** A property with no show object is still returned for every operation.
6. **The catalog file stays generated.** This page does not commit `n8n_node_types.json`.

### Suggested contributor checklist

```text
1. Filter operation get against show lists get, getAll, and post
2. Confirm get returns the unconditional property and the get property only
3. Confirm getall returns the unconditional property and the getAll property only
4. Confirm post returns the unconditional property and the post property only
5. Pass one property with displayOptions null and confirm the lookup returns the other properties
6. Confirm includeParams false still returns the compact entry
7. Confirm the gateway was not started and the catalog file was not loaded from disk
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Operation match | Equality on the show strings this loop already lowercases | `filter_operation in value` |
| Missing conditions | The existing "keep the property" branch | A new filter language |
| Null object | An `isinstance(..., dict)` test before `.get` | Catching `AttributeError` at the route |
| Intake | GitHub Issue | A separate node-schema tracker |

### Acceptance check

- [ ] `operation: "get"` returns properties shown for `get` and properties with no show object.
- [ ] `operation: "get"` omits a property whose show list is only `getAll`.
- [ ] `operation: "getall"` returns the `getAll` property and omits the `get` property.
- [ ] `operation: "post"` returns the `post` property and omits `get` and `getAll`.
- [ ] A property with `displayOptions: null` does not raise, and the rest of the node is still filtered.
- [ ] `includeParams: false` still returns the compact entry.
- [ ] The empty-catalog path from Stage 71 is unchanged.
- [ ] The gateway was not started to prove this page.

### Rollback

Restore the substring test and the unguarded `displayOptions` lookup. `get` again includes `getAll`, and a null `displayOptions` again raises `AttributeError`, which is today's behavior.
