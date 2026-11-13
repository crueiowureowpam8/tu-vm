---
title: Announcement Health Query Contract
description: Constructional contract so a failed Docker or announcement fetch is shown as a warning, using the checkup file the helper already mounts.
last_updated: 2026-11-10
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Announcement Health Query Contract

## Problem

The landing page can tell operators that the stack is healthy when the health query did not succeed.

`_live_container_health` starts from four empty lists. If `docker_get("/containers/json?all=true")` raises, the exception is swallowed and those empty lists are returned. If the response status is not 200, the function returns the same empty lists before it looks at any container. `announcements()` treats empty health lists as "nothing to report". With no log or update item either, it appends one item titled **All Systems Operational**.

Nginx already proxies `GET /status/announcements` to the helper route `GET /announcements`. The dashboard `refreshAnnouncements` function renders that JSON. Its `catch` path does not say the fetch failed. It replaces the feed with one item titled **System Running** and the message `Welcome to TechUties VM.` A non-OK HTTP response becomes `{}`, so the list is cleared.

Checked on 2026-11-10 by replaying those branches in Python 3.12.3. The helper process was not started. The Docker daemon was not started. The landing page was not opened.

| Query shape | Lists returned | Title the current branches produce |
|---|---|---|
| Exception from the socket call | empty | All Systems Operational |
| HTTP status 500 | empty | All Systems Operational |
| HTTP 200 and no known container down | empty | All Systems Operational |
| HTTP 200 and `ai_postgres` not running | PostgreSQL in `down` | Core Services Down |
| Browser `fetch` throws | feed replaced in the catch | System Running |

`scripts/daily-checkup.sh` already writes `/tmp/tu-vm-health-status.json` with `down_services`, `unhealthy_services`, `ondemand_stopped`, `total_restarts`, and `last_check`. A repository search finds that path only at the writer. The helper service already bind-mounts `/tmp`, which is how `/tmp/tu-vm-log-status.json` and `/tmp/tu-vm-update-status.json` reach `announcements()` today. The health file is the unread sibling of those two files.

A successful live query that finds nothing down should keep the operational item. Tier 2 containers that are absent stay off the down list, which is the current on-demand rule. `ondemand_stopped` in the checkup file follows the same rule.

Stage 69 (open PR #95) owns the `docker ps` status column used by shell checks. This page leaves `scripts/daily-checkup.sh` as it is, including the way that script selects container names.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `announcements()` | Builds the feed the landing page already renders |
| `_live_container_health` | Live Docker list for Tier 1 and Tier 2 labels |
| `TIER1_CONTAINERS` | Container name to label map already used by the live check |
| `/tmp/tu-vm-log-status.json` | Cached log counts the feed already reads |
| `/tmp/tu-vm-update-status.json` | Cached update flag the feed already reads |
| `/tmp/tu-vm-health-status.json` | Cached container lists the feed does not read yet |
| helper `/tmp` mount in `docker-compose.yml` | Makes the checkup files visible inside `ai_helper_index` |
| `.ann-critical` and `.ann-warning` | Styles already applied from `type` |
| `refreshAnnouncements` | The only browser reader of this feed |

Out of scope:

- A new monitoring database, status SaaS, or second dashboard
- Changing the helper `/tmp` mount
- Editing the checkup writer or its container-name filter
- Treating a stopped Tier 2 service as a core outage
- Replacing the landing page

## Proposal

Keep this feed. A health query that does not succeed must produce a critical or warning item. A query that succeeds and finds no Tier 1 outage may still say the stack is operational.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Live query result | `_live_container_health` | Return a distinct failure when the socket call raises or the status is not 200, instead of four empty lists |
| Cached fallback | `announcements()` | On that failure, read `/tmp/tu-vm-health-status.json` the same way the log and update files are read |
| Name labels | `announcements()` | Map `down_services` and `unhealthy_services` through `TIER1_CONTAINERS` and the Tier 2 labels |
| On-demand rows | `announcements()` | Ignore `ondemand_stopped` for the critical down item |
| No cache either | `announcements()` | Append one `critical` item whose title says the Docker health check failed |
| Success | `announcements()` | Keep **All Systems Operational** only after a successful live query with empty outage lists |
| Browser throw or non-OK | `refreshAnnouncements` | Use one `warning` item that says the announcement feed could not be loaded |
| Styles | existing `ann-*` classes | Keep `type` values `critical`, `warning`, and `info` |

### Rules

1. **Empty lists mean healthy only after a successful list.** A socket exception or a non-200 Docker response is not an empty stack.
2. **Use the checkup file that is already on the shared `/tmp`.** `down_services` becomes the existing core-down item. `unhealthy_services` becomes the existing unhealthy item. `total_restarts` can feed the existing excessive-restart item.
3. **On-demand stays quiet.** Names in `ondemand_stopped` do not become "Core Services Down".
4. **A missing cache is still a failure.** If the live query failed and the JSON file is missing or unreadable, the feed contains one critical item and does not contain **All Systems Operational**.
5. **The browser catch is the same rule.** `System Running` is reserved for a feed that loaded. A thrown fetch or a non-OK status uses `type: warning`.
6. **The writer stays put.** `check_container_health` in `scripts/daily-checkup.sh` is not edited by this contract.
7. **No new component.** The helper route and the landing-page function remain the feed.

### Suggested contributor checklist

```text
1. Replay a socket exception and confirm the feed title is not All Systems Operational
2. Replay HTTP 500 from the container list and confirm the same
3. Place a health file with down_services ["ai_postgres"] and confirm the core-down item names PostgreSQL
4. Place ondemand_stopped ["ai_ollama"] and confirm it does not create a core-down item
5. Remove the health file and confirm one critical Docker-check item remains
6. Replay a successful empty list and confirm All Systems Operational remains
7. Replay a thrown fetch and a non-OK response and confirm the warning title System Running is gone
8. Confirm the checkup script and the helper /tmp mount are unchanged
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Live containers | Docker socket call already in `_live_container_health` | A second agent or metrics stack |
| Last known containers | `/tmp/tu-vm-health-status.json` | A new status database |
| Browser feed | `refreshAnnouncements` and `ann-warning` | A new notification library |
| Intake | GitHub Issue | An out-of-band status page |

### Acceptance check

- [ ] A Docker socket exception does not produce **All Systems Operational**.
- [ ] A non-200 container-list response does not produce **All Systems Operational**.
- [ ] On that failure, `down_services` and `unhealthy_services` from `/tmp/tu-vm-health-status.json` become the existing critical and warning items.
- [ ] `ondemand_stopped` does not mark a core service down.
- [ ] A failed live query with no readable health file produces one critical item.
- [ ] A successful live query with no Tier 1 outage still produces **All Systems Operational**.
- [ ] A thrown `fetch` and a non-OK `/status/announcements` response render a warning, not **System Running**.
- [ ] `scripts/daily-checkup.sh` and the helper volume list are unchanged.
- [ ] The helper process and the Docker daemon were not started to prove this page.

### Rollback

Return four empty lists on a Docker failure and keep the **System Running** catch item. The landing page will again describe a failed health query as a healthy stack, which is today's behavior.
