# Implementation Backlog for Community-Based Website Suggestions

This backlog translates suggestions into implementation-ready work items with clear acceptance criteria.

## Completed / superseded (repository today)

These directions are satisfied without a custom suggestions stack:

- **Proposals & governance**: GitHub Issues (**Idea / suggestion** + **Bug report** templates), Discussions link, issue chooser security entry, [`CONTRIBUTING.md`](../CONTRIBUTING.md) (labels, `Fixes #` / Release publish notes), PR template with security/RFC checklist.
- **Security reporting**: [`SECURITY.md`](../SECURITY.md) (private reporting path + fallback).
- **Release ↔ issue linkage**: [Release Drafter](../.github/release-drafter.yml) + [workflow](../.github/workflows/release-drafter.yml) on `main`; draft releases grouped by PR labels (`skip-changelog` supported).
- **Triage hygiene**: [Stale automation](../.github/workflows/stale.yml) (`needs-info` cadence + idle issues/PRs); documented labels (`stale`, `pinned`, etc.).
- **Contributor diagnostics**: `./tu-vm.sh doctor`, `check-config`, `smoke-test`, `helper-contract-check`; [`scripts/pre-push-check.sh`](../scripts/pre-push-check.sh).
- **CI**: [`.github/workflows/ci.yml`](../.github/workflows/ci.yml) (`compose` render, `bash -n`, `check-config --ci`, smoke test, `/status/full` fixture validator; strict env gate on pull requests).
- **Docs & supply-chain hygiene (light)**: [docs link workflow](../.github/workflows/docs-links.yml), [Trivy config scan](../.github/workflows/trivy.yml) for `docker-compose.yml`, [Dependabot Actions](../.github/dependabot.yml).
- **CODEOWNERS template**: [`CODEOWNERS`](../CODEOWNERS) (replace placeholder team) + branch-protection notes in [`CONTRIBUTING.md`](../CONTRIBUTING.md).
- **Pre-commit (optional)**: [`.pre-commit-config.yaml`](../.pre-commit-config.yaml).
- **Release note helper**: [`scripts/release-note-helper.sh`](../scripts/release-note-helper.sh) and `./tu-vm.sh release-notes`.
- **Starter playbooks**: [`docs/playbooks/README.md`](../docs/playbooks/README.md) with stable anchor IDs for dashboard deep-links.
- **Landing dashboard**: Community strip **and** operator hub (per-playbook shortcuts + “What is new” → Releases / CHANGELOG) in [`nginx/html/index.html`](../nginx/html/index.html).
- **`/status/full` contract**: canonical shape in [`fixtures/status-full-contract.json`](../fixtures/status-full-contract.json), checked by [`scripts/validate_status_full_contract.py`](../scripts/validate_status_full_contract.py).

---

## Priority model

- **P0**: High impact, low complexity, immediate quality gain
- **P1**: Core community workflows
- **P2**: Scale and polish

---

## P1-1: Dynamic “What is new” content (optional polish)

### Scope

Static links to [latest release](https://github.com/techuties/tu-vm/releases/latest) and [`CHANGELOG.md`](../CHANGELOG.md) are on the landing page. **Optional next step:** fetch the latest GitHub Release title/body or parse the top of `CHANGELOG.md` and show **three** short bullets without leaving the LAN dashboard (requires a same-origin proxy, build-time injection, or cached JSON — avoid leaking operator traffic to third parties).

### Acceptance criteria

- At least three human-readable highlights visible on the dashboard when data exists.
- Graceful fallback to today’s static links when API data is unavailable.

---

## P2-1: Frontend modularization

### Scope

Refactor monolithic `nginx/html/index.html` into maintainable assets:

- `assets/js/*`
- `assets/css/*`
- optional component abstraction

### Acceptance criteria

- Existing UX is behaviorally equivalent after refactor.
- Linting is active for extracted JS/CSS.
- Build/deploy path remains compatible with current Docker/Nginx setup.

---

## P2-2: Automated browser smoke tests

### Scope

Add Playwright checks for core flows.

### Acceptance criteria

- CI executes smoke tests on key website interactions.
- Failing tests block regressions on critical flows.
- Test docs describe local run procedure for community contributors.

---

## P2-3: Feature-flagged rollout strategy

### Scope

Roll out major dashboard or experimental UI behavior behind flags (example: optional panels, beta integrations).

### Acceptance criteria

- Flags can be toggled via config/env without code edits.
- Rollback path documented and tested.
- Observability includes basic visibility into flag-dependent code paths where relevant.

---

## Suggested implementation order

1. **Next high-value recommendations** — supply-chain depth, frontend modularization, browser smoke tests, richer dashboard content.
2. **P1-1** — only if operators want inline release bullets without clicking GitHub.
3. **P2-1**, **P2-2**, **P2-3**

---

## Next high-value recommendations (10)

_Shipped from the prior round: playbook shortcuts + operator hub, static “What is new” links, pre-commit config, Dependabot, CODEOWNERS template, docs-links + Trivy config workflows, release-note-helper, `/status/full` fixture validator._

1. **Trivy (or Grype) image CVE scans** — Iterate pinned Compose images with actionable severity thresholds (separate from today’s config-only scan).
2. **Incremental dashboard asset extraction** — Break out CSS/JS from [`nginx/html/index.html`](../nginx/html/index.html); introduce ESLint/stylelint on extracted files (**P2-1**).
3. **Playwright smoke tests** — Tier-1 flows against `tu.lan` or headless nginx fixture (**P2-2**).
4. **Compose profile for CI integration** — Minimal service set (or mocks) to curl `/status/full` against a live helper response shape, complementing the static fixture.
5. **Playbook version notes** — Short matrix in [`docs/playbooks/README.md`](../docs/playbooks/README.md): TU-VM major tag / compose behaviours that change commands.
6. **Tighten Trivy gate** — Switch from `exit-code: 0` to failing on HIGH/CRITICAL once noise is triaged.
7. **Markdown style lint** — markdownlint on `docs/` + root policy files with a narrow rule set.
8. **SBOM export (optional)** — CycloneDX/SPDX artifact on release for regulated operators.
9. **Feature-flag pattern for dashboard experiments** — Env-driven toggles before large UI changes (**P2-3**).
10. **n8n / AFFiNE maintainer workflows** — Lightweight triage reminders (behind Tier-2 services) per day-to-day-tooling docs, if the team adopts them.

## Consolidated entries from parallel suggestion branches

5. **Pull request template cleanup** — Remove the duplicated Security and Verification sections from [`.github/pull_request_template.md`](../.github/pull_request_template.md).
- execute [`scripts/helper-contract-check.sh`](../scripts/helper-contract-check.sh),
- Declare the canonical reading path in [`README.md`](./README.md).
- Implement the smallest useful slice of [`extensions-and-integration-framework.md`](./extensions-and-integration-framework.md):
- Adopt one Markdown-first website framework using the guidance in [`website-and-docs-framework.md`](./website-and-docs-framework.md).
- [`website-community-framework.md`](./website-community-framework.md)
   [`website-community-pages.md`](./website-community-pages.md).
- Implement the static website foundation described in [`website-community-system-blueprint.md`](./website-community-system-blueprint.md):
- [`website-contributor-tooling.md`](./website-contributor-tooling.md)
   [`website-day-to-day-tooling.md`](./website-day-to-day-tooling.md).
- [`website-historical-baseline.md`](./website-historical-baseline.md)
- [`website-information-architecture.md`](./website-information-architecture.md)
- Adopt [`website-markdown-publishing-system.md`](./website-markdown-publishing-system.md) as the canonical path for turning existing suggestion markdown into website-ready community pages. Start with frontmatter, generated indexes, and validation around the current `suggestions/` files before adding any new website service.
- [`website-roadmap-from-historical-suggestions.md`](./website-roadmap-from-historical-suggestions.md)
- **Suggestion documentation consolidation**: [`website-markdown-publishing-system.md`](./website-markdown-publishing-system.md) defines the website markdown publishing path, while [`website-suggestion-dedupe-map.md`](./website-suggestion-dedupe-map.md) maps historical duplicate clusters.
- _Constructional website contracts for Stages 1–13 live under [`website/`](./website/) (Stage 13: Open WebUI audio/STT, web-search, PDF day-2, IP allowlist day-2, fixture/contract-test, Compose healthchecks, host cleanup/prune). Prefer implementing those contracts over authoring parallel suggestion markdown._
7. [AFFiNE MCP command fallback](./website/affine-mcp-command-fallback-contract.md) — code default `affine-mcp`, matching the npm bin.
- _Stage 3 website drafts (docs only): [`website/decision-log.md`](./website/decision-log.md), [`website/implemented-showcase.md`](./website/implemented-showcase.md), [`website/docs-framework-adoption.md`](./website/docs-framework-adoption.md), [`website/community-quality-gates.md`](./website/community-quality-gates.md), [`website/extension-pilot-contract.md`](./website/extension-pilot-contract.md), [`website/hardware-class-intake.md`](./website/hardware-class-intake.md)._
3. **P2-1**, **P2-2**, **P2-3** — modularization/smoke plus Stage 9 [`website/dashboard-feature-flag-experiments.md`](./website/dashboard-feature-flag-experiments.md) when available.
- _Stage 4 website drafts (docs only): [`website/mcp-catalog-ci-contract.md`](./website/mcp-catalog-ci-contract.md), [`website/operator-profile-cli.md`](./website/operator-profile-cli.md), [`website/extension-pilot-scaffold.md`](./website/extension-pilot-scaffold.md), [`website/supply-chain-community-gates.md`](./website/supply-chain-community-gates.md), [`website/dashboard-modularization-smoke.md`](./website/dashboard-modularization-smoke.md), [`website/stage-merge-playbook.md`](./website/stage-merge-playbook.md)._
2. **P1-1** — only if operators want inline release bullets without clicking GitHub; see Stage 9 [`website/dashboard-release-highlights-contract.md`](./website/dashboard-release-highlights-contract.md) when that sibling merges.
2. **Dashboard text insertion** — [`dashboard-untrusted-text-contract.md`](./website/dashboard-untrusted-text-contract.md). Stop assigning announcement text, object keys, and control error bodies with `innerHTML`.
1. **Hardware matrix adoption** — Link [`website/hardware-compatibility-matrix.md`](./website/hardware-compatibility-matrix.md) from playbooks; optionally ask Issues for `class_id` + redacted `doctor` evidence.
1. **Client IP trust for `/whitelist/auto`** — [`helper-client-ip-trust-contract.md`](./website/helper-client-ip-trust-contract.md). Prefer `X-Real-IP`. Do not return `ips` to anonymous callers.
1. [Helper control allowlist](./website/helper-control-service-allowlist-contract.md) — unknown `/control` names return 400 and never reach the Docker socket.
- Contracts live in [`website/`](./website/index.md). Implement these instead of writing another parallel framework file. Do not repeat Stage 11 logging, Stage 18 `no-new-privileges` (code only), Stage 20 IPv6, Stage 22 journald, Stage 23 GUC `-c` / timezone, Stage 24 unused `HEALTH_CHECK_*` / `NGINX_WORKER_*`, Stage 26 Chromium shm, Stage 29 Tika DNS / shm / REDISCLI_AUTH / n8n binary volume, Stage 30 (PR #56), Stage 31 (PR #57), or Stage 32 (PR #58).
5. [MCP build pins](./website/mcp-tool-build-pin-contract.md) — replace `@latest` with the 2026-09-26 registry versions.
6. [MCP listener honesty](./website/mcp-tool-listener-auth-contract.md) — no host ports, and supergateway `--oauth2Bearer` is not inbound auth.
1. **MCP tools catalog metadata** — Add `mcp-tools/catalog.yaml` (or equivalent) for the four in-repo tools and CI schema validation per [`website/mcp-tools-catalog.md`](./website/mcp-tools-catalog.md).
2. [Nginx header inheritance](./website/nginx-add-header-inheritance-contract.md) — locations keep the security headers their server block already sets.
3. **Operator service profiles (docs → CLI)** — Stabilize membership in [`website/operator-service-profiles.md`](./website/operator-service-profiles.md), then add `./tu-vm.sh profile list|show|apply --plan` as a thin wrapper (no security-posture changes).
2. **Persona entry paths** — Keep [`website/persona-entry-paths.md`](./website/persona-entry-paths.md) as the first-hour switchboard; add a one-line pointer from root README Contributing when ready.
4. [Archive expansion budget](./website/processor-archive-expansion-budget-contract.md) — member, byte, and ratio caps before zip/tar reads.
3. [Watcher object budget](./website/processor-object-byte-budget-contract.md) — `PROCESSOR_MAX_OBJECT_BYTES=67108864` before `Body.read()`.
