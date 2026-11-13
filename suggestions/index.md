# Community Suggestions Hub

This folder contains practical, implementation-ready suggestions for improving the TechUties VM platform with a **community-first approach**.

The recommendations focus on:
- Reusing mature frameworks instead of rebuilding core capabilities
- Improving daily contributor workflows and operator experience
- Creating transparent governance so community proposals can move from idea to production

## Why this exists

To avoid reinventing the wheel, we should standardize on proven open-source patterns, then customize only where the project has unique needs.

## Suggestion map

1. [Community System Framework](./community-system-framework.md)  
   Defines how suggestions are proposed, reviewed, accepted, and implemented.

2. [Website and Documentation Framework](./website-and-docs-framework.md)  
   Recommends a docs website stack and contribution model for clear public communication.

3. [Day-to-Day Tooling Framework](./day-to-day-tooling.md)  
   Covers operational tooling, automation, quality gates, and contributor productivity.

## Working principles

- **Community-first**: proposals are public, discussable, and traceable
- **Low-friction contribution**: templates, examples, and automation for common tasks
- **Operational reliability**: every accepted idea includes rollout and rollback guidance
- **Security and privacy by default**: preserve TU-VM's private-AI posture while expanding ecosystem value

## Suggested execution sequence

### Phase 1 (Foundation)

Done on the GitHub-native path: suggestion + PR templates, [`CONTRIBUTING.md`](../CONTRIBUTING.md), CI smoke/check-config, starter [`docs/playbooks/`](../docs/playbooks/README.md), landing links.

Still open:

- Dedicated docs site structure and navigation (optional static site later)
- Explicit maintainer label/ownership conventions documented beside Issues

### Phase 2 (Acceleration)

- Automation for triage and stale-issue workflows (labels, bots)
- Playbook surfacing from the dashboard with clear anchors
- Lightweight adoption metrics (release cadence, time-to-close by label)

### Phase 3 (Scale)

- Open community working groups
- Create plugin/integration curation process
- Publish quarterly roadmap and retrospective summaries

See [`implementation-backlog.md`](./implementation-backlog.md) for the trimmed backlog and the next ten prioritized recommendations.

## Consolidated entries from parallel suggestion branches

[`README.md`](./README.md) as the primary folder entry point and canonical
5. [Extensions and Integration Framework](./extensions-and-integration-framework.md)
| Historical reuse | [`historical-patterns-from-project.md`](./historical-patterns-from-project.md) | Existing TU-VM patterns to extend before adding new systems. |
1. [Historical Suggestions](./historical-suggestions.md) and [Website Historical Baseline](./website-historical-baseline.md)
4. [Website Community Framework](./website-community-framework.md)
4. [Community governance](./website-community-governance.md) — defines
3. [Website community pages](./website-community-pages.md) — defines the page
3. [Website Community Platform](./website-community-platform.md)
0. [Website Community System Blueprint](./website-community-system-blueprint.md)
0. [Canonical Website Community System](./website-community-system-canonical.md): Start here for the consolidated framework, lifecycle statuses, suggestion file contract, and tooling roadmap.
4. [Contributor Tooling and Day-to-Day Operations](./website-contributor-tooling.md)
5. [Day-to-day tooling](./website-day-to-day-tooling.md) — specifies established
1. [Historical Suggestions Baseline](./website-historical-baseline.md)
1. [Historical suggestions](./website-historical-suggestions.md) — checks
2. [Website Information Architecture](./website-information-architecture.md)
3. [Website Markdown Publishing System](./website-markdown-publishing-system.md)
7. [Historical Product Roadmap](./website-roadmap-from-historical-suggestions.md)
4. [Website Suggestion Dedupe Map](./website-suggestion-dedupe-map.md)
4. [Website Tools and Automation](./website-tools-and-automation.md)
4. [Day-to-Day Community Tools](./website/day-to-day-community-tools.md)
- Publish quarterly roadmap and retrospective summaries via [`website/decision-log.md`](./website/decision-log.md) + [`website/implemented-showcase.md`](./website/implemented-showcase.md)
- Create plugin/integration curation process (start from [`website/extension-pilot-contract.md`](./website/extension-pilot-contract.md))
2. [How to Submit](./website/how-to-submit.md)
1. [Publishable website pages](./website/index.md)
5. [MCP Tools Community Catalog](./website/mcp-tools-catalog.md) — distinct constructional focus
3. [Status Board](./website/status-board.md)
