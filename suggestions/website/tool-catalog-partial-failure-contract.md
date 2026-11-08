---
title: Tool Catalog Partial Failure Contract
description: Constructional contract so GET /tools still returns n8n and knowledge-base schemas when the AFFiNE connection fails.
last_updated: 2026-11-08
owner: maintainers
status: proposed
theme: tooling
impact: high
---

# Tool Catalog Partial Failure Contract

## Problem

`GET /tools` is the gateway catalog. AFFiNE, n8n, the n8n MCP sidecar, and the knowledge base are separate servers. A failure to connect to AFFiNE currently drops the other schemas.

`env.example` ships:

```text
AFFINE_MCP_ENABLED=true
AFFINE_API_TOKEN=
AFFINE_COOKIE=
AFFINE_EMAIL=
AFFINE_PASSWORD=
MCP_ALLOWED_SERVERS=affine,n8n,n8n_mcp,kb
```

`_affine_enabled()` is true with those defaults. `AffineMcpConnection._connect` raises `RuntimeError("No AFFiNE auth configured.")` when the token, cookie, and email/password pair are all empty. It raises that before it starts a process.

`list_tools` does this:

```text
if affine is enabled:
    try:
        tools.extend(await affine_conn.list_tools())
    except Exception as exc:
        return 503 {"tools": [], "error": str(exc)}
if n8n is enabled:
    tools.extend(n8n schemas)
if n8n_mcp is enabled:
    tools.extend(n8n_mcp schemas)
if kb is enabled:
    tools.extend(kb schemas)
return {"tools": tools}
```

The `return` sits inside the AFFiNE `except`. The n8n and knowledge-base branches do not run.

Same-shape replay on 2026-11-08, using that control flow and the default auth error. The gateway process was not started. AFFiNE was not contacted.

| Case | Status | Tools |
|---|---|---|
| AFFiNE enabled, connect raises `No AFFiNE auth configured.` | 503 | `[]` |
| AFFiNE connect returns one tool, then the local n8n and kb names are appended | 200 | affine tool, `n8n_list_workflows`, `kb_search` |

Day to day, a portable install leaves AFFiNE stopped and without a token. An operator who starts the gateway to use n8n tools gets an empty catalog from `GET /tools`.

`GET /openapi-tools.json` does not call AFFiNE. It builds n8n, n8n MCP, and knowledge-base paths only. This page does not change that document. Open WebUI can keep using it. `GET /tools` is the catalog that should agree.

Stage 42 (open PR #68) owns `MCP_AFFINE_COMMAND` defaulting to `npx` in code while Compose sets `affine-mcp`. This page does not change the command or the args. The auth error above happens before either binary starts.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `list_tools` | The handler to change |
| `_affine_enabled` / `_n8n_enabled` / `_n8n_mcp_enabled` / `_kb_enabled` | The switches to keep |
| `_n8n_tool_schemas`, `_n8n_mcp_tool_schemas`, `_kb_tool_schemas` | Local schemas that do not need AFFiNE |
| `AffineMcpConnection._connect` | Stays the place that requires a token, cookie, or email/password |
| `GET /openapi-tools.json` | Unchanged Open WebUI document |
| Stage 42 `affine-mcp-command-fallback-contract.md` (open PR #68) | Which binary AFFiNE uses. Unchanged |

Out of scope:

- A new tool registry or marketplace
- Disabling AFFiNE by default
- Inventing an AFFiNE login inside the gateway
- Changing which n8n tools exist

## Proposal

Collect a per-server error and still return the servers that succeeded.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Catalog | `list_tools` | On AFFiNE failure, append `{"server": "affine", "error": ...}` and continue |
| Response | `list_tools` | Return `200` with `tools` and `errors` when at least one server produced tools or was skipped as disabled |
| All down | `list_tools` | Return `503` only when every enabled server failed and `tools` is empty |
| Connect | `_connect` | Keep the existing auth error. Do not start `affine-mcp` without credentials |

### Rules

1. **One server failing is not an empty catalog.** n8n, n8n MCP, and knowledge-base schemas are still listed.
2. **The error is visible.** The AFFiNE message `No AFFiNE auth configured.` is on the response, not only in the log.
3. **Disabled stays disabled.** A server switched off by `MCP_ALLOWED_SERVERS` or its `*_ENABLED` flag is omitted, with no error entry.
4. **503 means nothing listed.** If AFFiNE is the only enabled server and it fails, `503` and an empty `tools` array stay correct.
5. **OpenAPI stays as it is.** `/openapi-tools.json` does not gain an AFFiNE live call.
6. **Command stays Stage 42.** `MCP_AFFINE_COMMAND` and `MCP_AFFINE_ARGS` are unchanged.

### Suggested contributor checklist

```text
1. Set AFFINE_MCP_ENABLED=true and leave AFFINE_API_TOKEN, AFFINE_COOKIE, AFFINE_EMAIL, and AFFINE_PASSWORD empty
2. Call GET /tools with a bearer token and confirm n8n_list_workflows and kb_search are present
3. Confirm the body includes an affine error equal to "No AFFiNE auth configured."
4. Confirm the status is 200
5. Set MCP_ALLOWED_SERVERS=affine only and confirm the status is 503 with an empty tools array
6. Set AFFINE_MCP_ENABLED=false and confirm there is no affine error entry
7. Confirm GET /openapi-tools.json still omits a live AFFiNE connect
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Catalog | `GET /tools` | A new registry service |
| Enable flags | `MCP_ALLOWED_SERVERS` and `*_ENABLED` | A second config file |
| AFFiNE auth | The token, cookie, or email/password already on `_connect` | A gateway-owned AFFiNE account |
| Intake | GitHub Issue | An out-of-band catalog UI |

### Acceptance check

- [ ] With default AFFiNE flags and no AFFiNE credentials, `GET /tools` includes the n8n tool schemas and the knowledge-base tool schemas.
- [ ] That response includes the AFFiNE error string and is HTTP 200.
- [ ] With only `affine` allowed, a connect failure is HTTP 503 and `tools` is empty.
- [ ] `MCP_AFFINE_COMMAND` is unchanged.
- [ ] `/openapi-tools.json` does not call `affine_conn.list_tools`.
- [ ] The gateway process was not started, and AFFiNE was not contacted, to prove the missing `return`.

### Rollback

Restore the `return` of `503` with an empty tool list inside the AFFiNE `except`. The other branches behave as they do today.
