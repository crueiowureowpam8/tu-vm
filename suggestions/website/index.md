---
title: Community Suggestions Website
description: Publishable Stage 73 website markdown for gzip tar archives, n8n list cursors, and the n8n circuit breaker after its quiet period.
last_updated: 2026-11-09
owner: maintainers
status: proposed
---

# Community Suggestions Website

This directory holds **publishable website markdown**: curated pages a future static docs site can ship without a second editable copy.

Participation stays on GitHub. These pages say which existing TU-VM tools to extend so day-to-day document processing, workflow listing, and n8n calls stay reliable for a community of operators and contributors.

## Why this exists

TU-VM already has the community path and the operator tools: GitHub Issues, `CONTRIBUTING.md`, `tu-vm.sh`, the helper API, the Nginx landing page, the MCP gateway, the LangGraph supervisor, playbooks, and CI. Historical suggestion files in `suggestions/` and open website PRs #27–#98 already specify the docs framework, governance, and the product contracts through Stage 72.

**Stage 73** (this run) records three defects checked on 2026-11-09. They are new findings. They do not restate Stages 1–72.

- `_get_file_type` classifies `bundle.tar.gz` as an archive. `_process_archive` then tests `Path.suffix`, which is `.gz`, so the tar branch does not run. The function still returns `status: success` and `Extracted 0 files`. A gzip tar built in memory opens with `tarfile` and contains `notes.txt`.
- `n8n_list_workflows` accepts `cursor` and then returns only `result` and `total`. A body with `nextCursor: page-2` loses that field. One archived row in a two-row page is removed, and the caller still has no cursor for the next page. `n8n_list_executions` drops `nextCursor` the same way, and its schema has no `cursor` property.
- `CircuitBreaker.allow` blocks while `open_until` is in the future. After the window, `fail_count` is still at the threshold, so the next failure opens the circuit again. A replay with threshold 3 showed that shape. The source default is 8.

The contracts:

- Open `.tar.gz` with the `tarfile` branch that already exists.
- Return the list cursor the n8n response already carries, and accept `cursor` on execution lists.
- Require a full threshold of new failures after the quiet period ends.

No second archive product, workflow engine, or circuit-breaker service.

## Boundaries (do not reinvent)

| Already exists | Website role |
|---|---|
| GitHub Issues + suggestion form | Sole proposal intake |
| [`CONTRIBUTING.md`](../../CONTRIBUTING.md) | Contributor path and labels |
| [`SECURITY.md`](../../SECURITY.md) | Vulnerability reporting |
| [`docs/playbooks/`](../../docs/playbooks/README.md) | Operator recipes |
| Nginx landing page | Local control plane |
| Historical files in [`suggestions/`](../) | Archive and planning source |
| Stage 1–72 website pages (open PRs #27–#98) | Implement those contracts in code; leave their pages as they are |
| `UniversalMinIOProcessor._process_archive` | The archive reader to extend |
| `n8n_list_workflows` and `n8n_list_executions` | The list handlers to extend |
| `CircuitBreaker` | The breaker to extend |

Stage 41 (open PR #67) owns MinIO object-list pagination. Stage 52 (open PR #78) owns `notes.pdf` and `notes.md` sharing `notes.txt`. Stage 69 (open PR #95) owns source `.txt` being skipped. Stage 70 (open PR #96) owns webhook `confirm=true`. Stage 71 (open PR #97) owns the non-empty workflow allowlist. Stage 72 (open PR #98) owns plain-text n8n success bodies. This stage leaves those contracts as they are.

## Page map (Stage 73)

| Page | Purpose |
|---|---|
| [Gzip tar suffix](./tar-gz-archive-suffix-contract.md) | A `.tar.gz` upload must be opened as a tar archive |
| [n8n list cursor](./n8n-list-cursor-contract.md) | Workflow and execution lists must keep the next page cursor |
| [Circuit breaker threshold](./circuit-breaker-threshold-contract.md) | The quiet period must restore the configured failure threshold |

## Expected sibling pages

Stages 1–72 are not files on this branch. When those PRs merge, keep their pages as siblings in this folder. Leave those files uncopied here.

Representative filenames only: `processor-object-list-pagination-contract.md`, `notes-output-key-contract.md`, `source-text-file-contract.md`, `webhook-confirm-gate-contract.md`, `n8n-workflow-allowlist-enforcement-contract.md`, `n8n-non-json-response-contract.md`.

Extend `stage-merge-playbook.md` through **Stage 73** when reconciling hubs so earlier pages are not dropped.

## Framework reuse (summary)

| Need | Prefer | Avoid |
|---|---|---|
| Proposal intake | GitHub Issues | A suggestion database |
| Docs site | Static framework already proposed in Stage 3 (Starlight remains the default) | A custom community portal |
| Archive reading | `tarfile` inside `_process_archive` | A new extraction service |
| List paging | `cursor` and `nextCursor` on the existing n8n tools | A second n8n client |
| Failure pause | `CircuitBreaker` in `mcp-gateway/app.py` | A hosted circuit-breaker product |

## Navigation for a future docs site

Map this folder to a `Community` section:

1. How to submit, status, and decisions (Stages 1–4, when those PRs are merged)
2. Operator day-to-day: a `.tar.gz` object in a watched bucket yields the text inside the archive
3. Operator day-to-day: workflow and execution lists can be continued from the cursor n8n returned
4. Operator day-to-day: after the n8n circuit quiets, the configured failure count applies again

When a static site is adopted, move or mount these pages into the docs content root. Keep a single editable copy.

## Related archive and planning docs

- [`../README.md`](../README.md) — suggestions hub
- [`../index.md`](../index.md) — community suggestions map
- [`../implementation-backlog.md`](../implementation-backlog.md) — trimmed execution backlog
- [`../historical-suggestions.md`](../historical-suggestions.md) — changelog ideas
- [`../website-roadmap-from-historical-suggestions.md`](../website-roadmap-from-historical-suggestions.md) — historical product direction
