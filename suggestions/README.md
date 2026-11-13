# Suggestions Hub

This folder contains detailed, implementation-oriented suggestions for building a community-driven website and contributor system without reinventing existing work.

## Historical baseline used

These suggestions are consolidated from previous `community-suggestions-*` branches so repeated ideas are reused instead of reauthored from scratch.

Recurring themes identified across historical branches:

1. Website information architecture for docs + operations + community
2. Community governance and contribution workflow
3. Practical contributor tooling for day-to-day operations
4. A phased roadmap built from already proposed feature directions

## Files in this folder

- `website-historical-baseline.md`  
  Historical suggestion patterns and how they were merged into a single framework.

- `website-information-architecture.md`  
  Detailed website structure, content model, and docs framework recommendation.

- `website-community-framework.md`  
  Community operating model, governance, ownership, and review standards.

- `website-contributor-tooling.md`  
  Concrete tooling proposals that improve day-to-day contributor productivity.

- `website-roadmap-from-historical-suggestions.md`  
  Sequenced roadmap that maps historical suggestions to implementation milestones.

## Design principles for all suggestions

1. Reuse existing project surfaces first (`README.md`, `CHANGELOG.md`, `tu-vm.sh`, helper API, nginx landing page).
2. Add modular improvements over deep rewrites.
3. Keep secure defaults and LAN-first behavior as non-negotiable.
4. Prioritize contribution quality, reproducibility, and maintainability.

To submit an idea via GitHub only, see [CONTRIBUTING.md](../CONTRIBUTING.md) at the repository root (Issues + PR templates).

For the **trimmed backlog** (implemented items removed) and **ten prioritized next recommendations**, see [`implementation-backlog.md`](./implementation-backlog.md).

## Consolidated entries from parallel suggestion branches

5. Link accepted or implemented suggestions back to issues, PRs, and [`CHANGELOG.md`](../CHANGELOG.md).
1. Reuse existing project surfaces first: [`README.md`](../README.md), [`CHANGELOG.md`](../CHANGELOG.md), [`tu-vm.sh`](../tu-vm.sh), [`helper/uploader.py`](../helper/uploader.py), and [`nginx/html/index.html`](../nginx/html/index.html).
2. [`community-system-framework.md`](./community-system-framework.md) — proposal review and decision model.
5. [`day-to-day-tooling.md`](./day-to-day-tooling.md) — contributor and maintainer tooling.
6. [`extensions-and-integration-framework.md`](./extensions-and-integration-framework.md) — safe community extension contract.
2. Check [`historical-suggestions.md`](./historical-suggestions.md) and [`historical-patterns-from-project.md`](./historical-patterns-from-project.md) before proposing a new idea.
2. [`historical-suggestions.md`](./historical-suggestions.md) and [`website-historical-baseline.md`](./website-historical-baseline.md) - prior ideas and recurring patterns that should be checked before proposing new work.
[`index.md`](./index.md) is an execution overview and should link back here
2. [`website-and-docs-framework.md`](./website-and-docs-framework.md) — static
- [`website-community-framework.md`](./website-community-framework.md)
4. [`website-community-governance.md`](./website-community-governance.md) —
3. [`website-community-pages.md`](./website-community-pages.md) — detailed
- Start with [`website-community-system-blueprint.md`](./website-community-system-blueprint.md). It is the current canonical synthesis of the historical suggestion files, recommended framework path, community lifecycle, governance model, and day-to-day tooling ideas.
- [`website-contributor-tooling.md`](./website-contributor-tooling.md)
5. [`website-day-to-day-tooling.md`](./website-day-to-day-tooling.md) —
1. [`website-historical-baseline.md`](./website-historical-baseline.md) -
1. [`website-historical-suggestions.md`](./website-historical-suggestions.md) —
- [`website-information-architecture.md`](./website-information-architecture.md)
- [`website-markdown-publishing-system.md`](./website-markdown-publishing-system.md)
8. [`website-roadmap-from-historical-suggestions.md`](./website-roadmap-from-historical-suggestions.md) — historical product direction; the implementation backlog controls current execution order.
- For the **trimmed backlog** (implemented items removed) and **ten prioritized next recommendations**, see [`implementation-backlog.md`](./implementation-backlog.md). Before adding new website/community suggestion files, check [`website-suggestion-dedupe-map.md`](./website-suggestion-dedupe-map.md).
| [`website-tools-and-automation.md`](./website-tools-and-automation.md) | Day-to-day tooling, quality checks, duplicate detection, dashboard metrics, and maintainer automation. |
- [`website/`](./website/)
| [`website/day-to-day-community-tools.md`](./website/day-to-day-community-tools.md) | Frameworks and daily commands |
| [`website/hardware-compatibility-matrix.md`](./website/hardware-compatibility-matrix.md) | Living host/RAM/GPU/storage fit artifact |
| [`website/how-to-submit.md`](./website/how-to-submit.md) | Dedupe-first submission guide |
- Canonical constructional contracts for a future static docs site live in [`website/`](./website/index.md) (Stage 33 on this branch). Stages 1–32 remain on open sibling PRs (#27–#58) and should be merged as siblings, not rewritten.
| [`website/mcp-tools-catalog.md`](./website/mcp-tools-catalog.md) | Distinct next construction: MCP tools community contract |
| [`website/operator-service-profiles.md`](./website/operator-service-profiles.md) | Named Tier 1/Tier 2 profiles for day-to-day ops |
| [`website/persona-entry-paths.md`](./website/persona-entry-paths.md) | First-hour routes that only link existing commands/docs |
| [`website/status-board.md`](./website/status-board.md) | Curated lifecycle projection (GitHub remains SoR) |
