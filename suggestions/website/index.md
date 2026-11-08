---
title: Community Suggestions Website
description: Publishable Stage 72 website markdown for the proof signing key, the MCP tool catalog when AFFiNE is unconfigured, and n8n responses that are not JSON.
last_updated: 2026-11-08
owner: maintainers
status: proposed
---

# Community Suggestions Website

This directory holds **publishable website markdown**: curated pages a future static docs site can ship without a second editable copy.

Participation stays on GitHub. These pages say which existing TU-VM tools to extend so day-to-day proof checks, tool discovery, and n8n calls stay reliable for a community of operators and contributors.

## Why this exists

TU-VM already has the community path and the operator tools: GitHub Issues, `CONTRIBUTING.md`, `tu-vm.sh`, the helper API, the Nginx landing page, the MCP gateway, the LangGraph supervisor, playbooks, and CI. Historical suggestion files in `suggestions/` and open website PRs #27–#97 already specify the docs framework, governance, and the product contracts through Stage 71.

**Stage 72** (this run) records three defects checked on 2026-11-08. They are not restatements of Stages 1–71.

- `generate_secrets` replaces every `CHANGE_ME_*` assignment it knows about. `MCP_PROOF_SIGNING_KEY=CHANGE_ME_PROOF_HMAC` is still that placeholder afterward. `_sign_proof` HMACs with that public string. `scripts/rollout-gates.sh` never reads `signature`.
- `GET /tools` asks AFFiNE for tools first. With the defaults in `env.example` (`AFFINE_MCP_ENABLED=true` and no AFFiNE token, cookie, or email/password), connect raises `No AFFiNE auth configured.` The handler answers `503` and an empty tool list, so the n8n and knowledge-base schemas are not returned.
- `N8nApi.request` parses every HTTP status below 400 with `resp.json()`. A body of `ok`, an empty body, and a `204` all become `502` `n8n returned invalid JSON`. A JSON object still parses.

The contracts:

- Give the proof key its own generated value, and make the existing rollout gate check the HMAC.
- Keep listing n8n and knowledge-base tools when the AFFiNE connection fails.
- Accept an empty or plain-text n8n success body on the client that already exists.

No second audit product, tool registry, or workflow engine.

## Boundaries (do not reinvent)

| Already exists | Website role |
|---|---|
| GitHub Issues + suggestion form | Sole proposal intake |
| [`CONTRIBUTING.md`](../../CONTRIBUTING.md) | Contributor path and labels |
| [`SECURITY.md`](../../SECURITY.md) | Vulnerability reporting |
| [`docs/playbooks/`](../../docs/playbooks/README.md) | Operator recipes |
| Nginx landing page | Local control plane |
| Historical files in [`suggestions/`](../) | Archive and planning source |
| Stage 1–71 website pages (open PRs #27–#97) | Implement those contracts in code; leave their pages as they are |
| `generate_secrets` | The secret writer to extend |
| `GET /tools` | The catalog handler to keep serving the other servers |
| `N8nApi.request` | The n8n client to extend |
| `scripts/rollout-gates.sh` | The proof reader to extend |

Stage 54 (open PR #80) owns the shared `CHANGE_ME_SECURE_PASSWORD` collision and the printed credential card. This stage leaves those six keys alone. Stage 53 (open PR #79) owns aligning the start-up detectors with the defaults `generate_secrets` already replaces. Stage 14 (open PR #40) owns the claimed-verify rate math in the rollout gate. Stage 42 (open PR #68) owns the AFFiNE command default (`affine-mcp` versus `npx`). Stage 70 (open PR #96) owns webhook `confirm=true`. Stage 71 (open PR #97) owns which workflow id a non-empty allowlist may touch.

## Page map (Stage 72)

| Page | Purpose |
|---|---|
| [Proof signing key](./proof-signing-key-contract.md) | The HMAC key must be generated, and the rollout gate must check it |
| [Tool catalog partial failure](./tool-catalog-partial-failure-contract.md) | A missing AFFiNE login must not hide n8n and knowledge-base tools |
| [n8n response body](./n8n-non-json-response-contract.md) | A successful plain-text or empty n8n body must not become a 502 |

## Expected sibling pages

Stages 1–71 are not files on this branch. When those PRs merge, keep their pages as siblings in this folder. Leave those files uncopied here.

Representative filenames only: `secret-generation-identity-contract.md`, `startup-secret-gate-contract.md`, `rollout-gates-proof-store-contract.md`, `affine-mcp-command-fallback-contract.md`, `webhook-confirm-gate-contract.md`, `n8n-workflow-allowlist-enforcement-contract.md`.

Extend `stage-merge-playbook.md` through **Stage 72** when reconciling hubs so earlier pages are not dropped.

## Framework reuse (summary)

| Need | Prefer | Avoid |
|---|---|---|
| Proposal intake | GitHub Issues | A suggestion database |
| Docs site | Static framework already proposed in Stage 3 (Starlight remains the default) | A custom community portal |
| Proof key | `generate_secrets` and `MCP_PROOF_SIGNING_KEY` | A new key-management service |
| Proof check | `scripts/rollout-gates.sh` | A second audit product |
| Tool catalog | `GET /tools` | A new registry |
| n8n body | `N8nApi.request` | A second HTTP client |

## Navigation for a future docs site

Map this folder to a `Community` section:

1. How to submit, status, and decisions (Stages 1–4, when those PRs are merged)
2. Operator day-to-day: proof rows are signed with a generated key and the rollout gate checks the signature
3. Operator day-to-day: the tool catalog still lists n8n and knowledge-base tools when AFFiNE has no login
4. Operator day-to-day: a webhook that returns plain text is a success body, not a JSON error

When a static site is adopted, move or mount these pages into the docs content root. Keep a single editable copy.

## Related archive and planning docs

- [`../README.md`](../README.md) — suggestions hub
- [`../index.md`](../index.md) — community suggestions map
- [`../implementation-backlog.md`](../implementation-backlog.md) — trimmed execution backlog
- [`../historical-suggestions.md`](../historical-suggestions.md) — changelog ideas
- [`../website-roadmap-from-historical-suggestions.md`](../website-roadmap-from-historical-suggestions.md) — historical product direction
