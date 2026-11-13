---
title: Community Suggestions Website
description: Publishable Stage 30 community website markdown for Pi-hole v6 FTLCONF env, MinIO root user interpolation, leftover MCP/processor service DNS, n8n proxy hops and secure cookies, Qdrant/MinIO healthcheck binaries, Compose project name, and n8n auth env honesty—extending Stages 1–29 without reinventing frameworks or community intake.
last_updated: 2026-08-31
owner: maintainers
status: proposed
---

# Community Suggestions Website

This directory holds **publishable website markdown**: curated, read-only pages that a future static docs site can ship without creating a second editable tree.

Participation stays on GitHub. These pages explain how the community system works, which frameworks to reuse, and which living contracts keep day-to-day work easy.

## Why this exists

TU-VM already has strong operational surfaces (`tu-vm.sh`, helper API, Nginx dashboard, playbooks, CI, smoke/pre-push gates, Compose IPAM, local tar backup, and digest-pinned images). Historical suggestion branches repeatedly asked for the same foundations:

1. Clear website information architecture for docs, operations, and community
2. Lightweight governance and contribution workflow
3. Practical tooling that reduces daily friction
4. Reuse of mature frameworks instead of custom platforms

**Stage 30** (this run) adds **Pi-hole v6, MinIO identity, leftover MCP/processor DNS, n8n reverse-proxy, healthcheck-binary, Compose project-name, and n8n auth-honesty constructional contracts** that remained open after Stage 1–29 drafts and consolidation PRs (#23–#55):

- Official Pi-hole v6 `FTLCONF_*` keys instead of v5 `WEBPASSWORD` / `PIHOLE_DNS_` / `ServerIP`
- `MINIO_ROOT_USER` from `.env` instead of a hardcoded `admin` in Compose and the processor
- Leftover `ai_n8n` / `ai_affine` / `ai_n8n_mcp` / processor `ai_tika` URL defaults after Stage 28/29 service-DNS pages
- Official n8n `N8N_PROXY_HOPS` and `N8N_SECURE_COOKIE` behind the existing Nginx vhost
- Image-honest Qdrant/MinIO probes (`wget` / `mc ready`) instead of assuming `curl`
- Compose top-level `name:` so project labels stay stable across clone paths
- Honest `N8N_USER` / `N8N_PASSWORD` ownership (gateway only, not n8n login)

These are **constructional**: they tell contributors how to extend official Pi-hole/MinIO/n8n/Qdrant/Compose keys and the GitHub-native community path without inventing a Pi-hole config sidecar, a second identity store, a service-discovery microservice, a cookie-rewrite proxy, a probe sidecar, a directory-name lock script, or a parallel n8n auth product.

## Boundaries (do not reinvent)

| Already exists | Website role |
|---|---|
| GitHub Issues + suggestion form | Sole proposal intake and discussion |
| [`CONTRIBUTING.md`](../../CONTRIBUTING.md) | Contributor path and labels |
| [`CHANGELOG.md`](../../CHANGELOG.md) / Releases | Shipped-work record |
| [`SECURITY.md`](../../SECURITY.md) | Vulnerability reporting path |
| [`docs/playbooks/`](../../docs/playbooks/README.md) | Operator recipes |
| Nginx landing dashboard | Local control plane (not a social platform) |
| Historical files in [`suggestions/`](../) | Archive and planning source |
| Stage 1–22 website pages (open PRs #27–#48) | Contracts and living artifacts—implement, do not rewrite |
| Stage 23 website pages (open PR #49) | Snippets, ILM, read-only rootfs, GUCs, Qdrant memory, timezone, role networks |
| Stage 24 website pages (open PR #50) | Nginx worker env, Redis ACL, MinIO SSE, host sysctl, depends_on health, backup wrap, HEALTH_CHECK_* |
| Stage 25 website pages (open PR #51) | Stop grace, tini, non-root user, CHECK_INTERVAL, pids_limit, mem_swappiness, Tika /tika probe |
| Stage 26 website pages (open PR #52) | Chromium shm, n8n-mcp pin, MCP restart honesty, Tika/MinIO limits, processor tmpfs, datastore nofile, OOM order |
| Stage 27 website pages (open PR #53) | Helper pin, helper `/health`, CPU/IO shares, host `/tmp` isolation, docker GID, helper limits, socket-proxy |
| Stage 28 website pages (open PR #54) | pull_policy, leftover Tier 2 budgets, n8n healthz, x-* DRY, AFFiNE Redis auth, n8n Postgres DNS, include |
| Stage 29 website pages (open PR #55) | start_interval, Nginx tmpfs, n8n binary prune, labels, Open WebUI Tika DNS, Postgres shm, REDISCLI_AUTH |
| Consolidation / knowledge packs / support / change-aware PRs (#23–#26) | Adjacent proposals—extend, do not duplicate |

Do **not** add a local voting database, suggestion queue API, or dashboard authentication for public participation. The website publishes curated views and links back to GitHub.

Do **not** republish Stage 11 `container-log-retention-contract.md` (Compose `logging` max-size) or Stage 22 `logging-driver-journald-contract.md` as a new logging product. Stage 18 `container-hardening-baseline-contract.md` already covers `security_opt: no-new-privileges` — implement that in **code**, do not write another page. Do **not** rewrite Stage 23 read-only policy, Stage 24 interval env keys, Stage 26 Chromium shm, Stage 26 MCP restart honesty, Stage 28 n8n Postgres DNS, or Stage 29 start_interval / Nginx tmpfs / n8n binary / labels / Open WebUI Tika DNS / Postgres shm / REDISCLI_AUTH.

## Page map (Stage 30)

| Page | Purpose |
|---|---|
| [Pi-hole v6 FTLCONF env](./pihole-v6-ftlconf-env-contract.md) | Official `FTLCONF_*` instead of v5 names |
| [MinIO root user env](./minio-root-user-env-contract.md) | Interpolate `MINIO_ROOT_USER` everywhere `admin` is hardcoded |
| [Leftover MCP service DNS](./leftover-mcp-service-dns-contract.md) | `n8n` / `affine` / `n8n_mcp` after Stage 28/29 |
| [n8n proxy hops and cookies](./n8n-proxy-hops-cookie-contract.md) | Official `N8N_PROXY_HOPS` + `N8N_SECURE_COOKIE` |
| [Qdrant/MinIO healthcheck binaries](./qdrant-minio-healthcheck-binary-contract.md) | Probes that exist in the pinned images |
| [Compose project name](./compose-project-name-contract.md) | Top-level `name:` for stable project labels |
| [n8n auth env honesty](./n8n-auth-env-honesty-contract.md) | `N8N_USER` / `N8N_PASSWORD` are gateway-only |

## Expected sibling Stage 1 pages

When merged from the Stage 1 website proposal, keep these as siblings (same folder, one content root later):

| Page | Purpose |
|---|---|
| `how-to-submit.md` | High-signal suggestion writing and dedupe checks |
| `status-board.md` | Curated projection of suggestion lifecycle |
| `day-to-day-community-tools.md` | Frameworks and commands that ease daily work |
| `mcp-tools-catalog.md` | Human contribution contract for optional MCP tool images |

## Expected sibling Stage 2 pages

| Page | Purpose |
|---|---|
| `hardware-compatibility-matrix.md` | Living host/RAM/GPU/storage fit artifact |
| `persona-entry-paths.md` | First-hour routes for operators and contributors |
| `operator-service-profiles.md` | Named Tier 1/Tier 2 profiles for resource control |

## Expected sibling Stage 3 pages

| Page | Purpose |
|---|---|
| `decision-log.md` | Accepted / deferred / declined rationale |
| `implemented-showcase.md` | Changelog-adjacent proof that community ideas shipped |
| `docs-framework-adoption.md` | Starlight-first adoption gates |
| `community-quality-gates.md` | Day-to-day evidence map |
| `extension-pilot-contract.md` | Compose-backed extension package shape |
| `hardware-class-intake.md` | Optional Issue-form `class_id` |

## Expected sibling Stage 4 pages

| Page | Purpose |
|---|---|
| `mcp-catalog-ci-contract.md` | `mcp-tools/catalog.yaml` schema + CI validation |
| `operator-profile-cli.md` | `tu-vm.sh profile` list/show/apply `--plan` |
| `extension-pilot-scaffold.md` | First `extensions/<id>/` package + validator |
| `supply-chain-community-gates.md` | Image CVE scans, severity gates, optional SBOM |
| `dashboard-modularization-smoke.md` | Extract Nginx assets + Playwright Tier-1 smoke |
| `stage-merge-playbook.md` | Reconcile Stage hubs without losing siblings |

## Expected sibling Stage 5 pages

| Page | Purpose |
|---|---|
| `devcontainer-contributor-environment.md` | Repeatable Codespaces/Dev Containers setup |
| `compose-ci-live-profile.md` | Minimal Compose profile for live `/status/full` |
| `playbook-version-matrix.md` | Release-aware notes for operator playbooks |
| `community-label-and-board-contract.md` | GitHub labels + Projects as the OS |
| `community-health-digest.md` | Privacy-safe metrics as static markdown |
| `task-runner-wrapper.md` | `just`/`make` thin facade over existing scripts |
| `control-plane-contribution-contract.md` | Safe nginx allowlist and control-plane rules |

## Expected sibling Stage 6 pages

| Page | Purpose |
|---|---|
| `suggestion-corpus-registry.md` | YAML registry + CI for historical suggestions |
| `website-frontmatter-ci-contract.md` | Schema and link checks for publishable pages |
| `n8n-workflow-catalog.md` | Community contribution path for n8n recipes |
| `observability-contribution-contract.md` | Grafana/Prometheus dashboard and alert contribution rules |
| `backup-restore-community-drill.md` | Recurring reliability drill using existing backup/restore commands |
| `airgap-docs-mirror.md` | Offline/static docs pack for LAN and disconnected operators |
| `deprecation-notice-framework.md` | Breaking-change and deprecation communication pattern |

## Expected sibling Stage 7 pages

| Page | Purpose |
|---|---|
| `helper-api-contribution-contract.md` | Evolve helper/`/status/full` without breaking the dashboard |
| `tier2-idle-autostop-policy.md` | Opt-in idle timeout for heavy on-demand services |
| `battery-power-operator-signals.md` | Laptop battery widgets and energy-aware recommendations |
| `local-resource-history.md` | Local-only CPU/memory trend snapshots for operators |
| `service-dependency-map.md` | Declared dependencies for safe Tier 2 start ordering |
| `dashboard-accessibility-mobile.md` | A11y and touch-first acceptance baseline for the control plane |
| `ai-pipeline-contribution-contract.md` | Community path for MCP gateway and LangGraph supervisor **code** changes |

## Expected sibling Stage 8 pages

| Page | Purpose |
|---|---|
| `smart-startup-optimization.md` | Tier-aware quick-start planner without a new orchestrator |
| `secret-rotation-credential-hygiene.md` | Day-to-day secret lifecycle using `generate-secrets` and playbooks |
| `compose-override-contribution-contract.md` | Safe community path for local Compose customizations |
| `image-update-channel-policy.md` | Pin, check, update, and rollback with clear operator channels |
| `rag-knowledge-contribution-contract.md` | Community docs and corpora via MinIO/Tika/Qdrant/Open WebUI |
| `release-canary-community-program.md` | Lightweight pre-release testing using GitHub + existing smoke tools |
| `cross-service-e2e-scenario-catalog.md` | Named scenarios that wrap `chain-smoke`, helper checks, and playbooks |

## Expected sibling Stage 9 pages

| Page | Purpose |
|---|---|
| `privacy-preserving-usage-analytics.md` | Local-only usage patterns and recommendations |
| `ollama-model-catalog-contribution.md` | Community path for recommending local models |
| `affine-collaboration-contribution-contract.md` | Safe community recipes around AFFiNE |
| `pihole-dns-hygiene-contribution.md` | LAN DNS / blocklist hygiene (distinct from Tailscale bridge) |
| `offsite-backup-rclone-contract.md` | Optional remote backup lane over existing `backup` |
| `dashboard-feature-flag-experiments.md` | Env-driven UI experiments |
| `dashboard-release-highlights-contract.md` | Optional “What is new” bullets on the LAN dashboard |

## Expected sibling Stage 10 pages

| Page | Purpose |
|---|---|
| `open-webui-contribution-contract.md` | Safe community path for Open WebUI config, init, and tool wiring |
| `document-pipeline-contribution-contract.md` | Tika + MinIO + processor contribution rules |
| `data-plane-hygiene-contract.md` | Volume and corpus hygiene distinct from ILM |
| `tls-certificate-lifecycle-contract.md` | Local TLS renew/replace without a public CA product |
| `disk-pressure-volume-ops-contract.md` | Host disk pressure distinct from Qdrant RAM |
| `gpu-acceleration-contribution-contract.md` | Optional GPU lanes for Ollama / Open WebUI |
| `qm-llm-bridge-contribution-contract.md` | QM / local LLM bridge rules |

## Expected sibling Stage 11 pages

| Page | Purpose |
|---|---|
| `tailscale-lan-bridge-contract.md` | Optional Tailscale path distinct from Pi-hole DNS |
| `host-cron-maintenance-contract.md` | Cron install/remove without a second supervisor |
| `browserless-automation-contribution-contract.md` | Playwright/browserless contribution rules |
| `workflow-operator-contribution-contract.md` | Workflow Operator product lane |
| `autonomous-write-guard-policy.md` | Operator write-guard postures and env flags |
| `container-log-retention-contract.md` | Compose logging max-size / retention (do not rewrite) |
| `privileged-host-ops-contract.md` | Narrow sudoers / privileged host ops |

## Expected sibling Stage 12 pages

| Page | Purpose |
|---|---|
| `access-mode-and-firewall-contract.md` | Secure / public / lock access modes |
| `nginx-edge-routing-contract.md` | Edge vhost and proxy contribution rules |
| `env-schema-contribution-contract.md` | `.env` / `env.example` contribution rules |
| `diagnostics-and-triage-contract.md` | Doctor / diagnose evidence paths |
| `dashboard-announcements-contract.md` | Announcement surface contribution rules |
| `lan-dns-client-onboarding-contract.md` | Client/router DNS onboarding |
| `host-port-binding-contract.md` | Host port conflict and bind rules |

## Expected sibling Stage 13 pages

| Page | Purpose |
|---|---|
| `open-webui-audio-stt-contract.md` | Speech-to-text config and repair helpers |
| `open-webui-websearch-contract.md` | Optional web-search loader privacy rules |
| `pdf-operator-day2-contract.md` | PDF status / test / logs / reset toolkit |
| `ip-allowlist-day2-contract.md` | Whitelist list/add/remove day-to-day ops |
| `fixture-and-contract-test-contribution.md` | Fixture and schema validator contribution |
| `compose-healthcheck-contribution-contract.md` | Honest Compose healthcheck design |
| `host-cleanup-and-prune-contract.md` | Safe cleanup without surprise data loss |

## Expected sibling Stage 14 pages

| Page | Purpose |
|---|---|
| `cli-subcommand-contribution-contract.md` | Safe community path for extending `tu-vm.sh` |
| `changelog-release-notes-contribution-contract.md` | Human + automation path for release communication |
| `rollout-gates-proof-store-contract.md` | Pre/post-release proof-store quality gates |
| `n8n-mcp-sidecar-contribution-contract.md` | Enrichment sidecar rules distinct from workflow catalogs |
| `openwebui-minio-sync-day2-contract.md` | Operator contract for upload/TXT sync hygiene |
| `seed-chat-context-contribution-contract.md` | Community path for assistant context bootstraps |
| `pdf-loader-switch-day2-contract.md` | Tika / PyMuPDF toggle rules for day-to-day ops |

## Expected sibling Stage 15 pages

| Page | Purpose |
|---|---|
| `safe-update-contribution-contract.md` | Extend pin maps and check/apply/rollback without a second updater |
| `n8n-node-types-extraction-contract.md` | Regenerate gateway node-types JSON from local n8n |
| `open-webui-init-firstboot-contract.md` | Idempotent first-boot patching distinct from day-2 loader toggles |
| `langgraph-supervisor-smoke-contract.md` | Day-2 trust-path smoke without a parallel QA platform |
| `mcp-gateway-ops-contribution-contract.md` | Auth-preserving gateway ops and tool routing contributions |
| `docs-link-ci-contribution-contract.md` | Lychee workflow and exclude hygiene for community docs |
| `minio-bucket-lifecycle-contract.md` | Bucket naming, retention, and corpus hygiene |

## Expected sibling Stage 16 pages

| Page | Purpose |
|---|---|
| `daily-checkup-contribution-contract.md` | Scheduled Tier-aware health/update status without a monitoring SaaS |
| `pre-push-check-contribution-contract.md` | Fast local contributor gate wrapping shared scripts |
| `smoke-test-contribution-contract.md` | Static default + optional Tier-1 `--live` probes |
| `ci-workflow-contribution-contract.md` | Primary Actions gate that calls `scripts/` |
| `github-community-automation-contract.md` | Stale, issue forms, and triage without a local queue |
| `codeowners-review-routing-contract.md` | Path ownership and review routing |
| `dependabot-actions-maintenance-contract.md` | Weekly Actions bumps distinct from image CVE lanes |

## Expected sibling Stage 17 pages

| Page | Purpose |
|---|---|
| `weekly-stack-update-contribution-contract.md` | Cron-friendly full-stack update lane distinct from daily checkup |
| `check-config-contribution-contract.md` | Secret-safe preflight for compose/env/allowlist |
| `helper-contract-check-contribution-contract.md` | Live helper JSON probes when `ai_helper_index` is up |
| `pre-commit-hooks-contribution-contract.md` | Optional local hooks wrapping shared syntax gates |
| `release-drafter-contribution-contract.md` | Label-driven draft releases without a second changelog bot |
| `security-reporting-contribution-contract.md` | Private advisory-first vulnerability path |
| `changelog-refresh-contribution-contract.md` | Local Unreleased section helper distinct from Release Drafter |

## Expected sibling Stage 18 pages

| Page | Purpose |
|---|---|
| `compose-ipam-address-registry.md` | Unique static IPv4 allocation on `172.20.0.0/16` |
| `compose-resource-budget-contract.md` | Energy-aware CPU/memory limits for community Compose changes |
| `mcp-tool-sandbox-contribution-contract.md` | Safe community path for Playwright, filesystem, fetch, and memory |
| `beginner-quickstart-contribution-contract.md` | Portable-default onboarding without silent full-stack starts |
| `local-backup-restore-contribution-contract.md` | `tu-vm.sh backup` / `restore` format, rotation, and secret-safe archives |
| `helper-control-auth-contract.md` | `CONTROL_TOKEN`, header-first auth, and unauthenticated 401 behavior |
| `container-hardening-baseline-contract.md` | cap_drop / no-new-privileges / docker.sock review for Compose PRs |

## Expected sibling Stage 19 pages

| Page | Purpose |
|---|---|
| `tika-xml-config-contribution-contract.md` | Parsers, OCR, and timeouts in bind-mounted Tika XML |
| `prometheus-exporter-opt-in-contract.md` | Exporters stay off default start / energy budget |
| `compose-native-profiles-contract.md` | Align Compose `profiles:` with `tu-vm.sh` tier arrays |
| `grafana-dashboard-as-code-contract.md` | Provisioned JSON dashboards beside the datasource file |
| `nginx-login-rate-limit-contract.md` | Apply unused `login` zone to verified auth locations |
| `affine-backup-volume-coverage-contract.md` | Include AFFiNE volumes in the existing backup loop |
| `mcp-fetch-cidr-deny-contract.md` | Block Fetch from `172.20.0.0/16` and RFC1918/link-local |

## Expected sibling Stage 20 pages

| Page | Purpose |
|---|---|
| `tika-minio-processor-python-contribution-contract.md` | Watch/retry/status changes stay in the existing Alpine worker |
| `nginx-stub-status-exporter-contract.md` | Internal stub_status only, paired with opt-in nginx-exporter |
| `affine-redis-durability-contract.md` | Named volume + explicit AOF/RDB; do not share platform Redis |
| `ipv6-dual-stack-policy-contract.md` | IPv4-first; dual-stack is all-or-nothing |
| `systemd-user-unit-install-contract.md` | Optional oneshot `--user` unit; not a second supervisor |
| `wsl2-contributor-path-contract.md` | Windows contributors use WSL2 + the same bash scripts |
| `n8n-webhook-login-zone-split-contract.md` | Split locations so login can use `zone=login` without breaking webhooks |

## Expected sibling Stage 21 pages

| Page | Purpose |
|---|---|
| `nginx-upstream-dns-contract.md` | Prefer Compose service names with an explicit resolver |
| `platform-redis-durability-contract.md` | Make `redis_data` + AOF/RDB an explicit policy for `ai_redis` |
| `helper-uploader-module-contribution-contract.md` | Split `uploader.py` into importable modules |
| `multi-arch-image-policy-contract.md` | Digest pins that work on amd64 and arm64 |
| `podman-rootless-contributor-path-contract.md` | Optional Docker-compatible runtime lane |
| `apparmor-seccomp-profile-contract.md` | Optional LSM profiles on top of existing limits |
| `restic-borg-backup-backend-contract.md` | Optional backend behind `tu-vm.sh backup` / `restore` |

## Expected sibling Stage 22 pages

| Page | Purpose |
|---|---|
| `helper-index-image-bake-contract.md` | Bake apk/pip into `helper/Dockerfile`; keep the bind-mount |
| `postgres-wal-pitr-policy-contract.md` | Name dump-plus-volume as default; opt-in WAL behind `tu-vm.sh` |
| `image-provenance-cosign-contract.md` | Verify digest pins without replacing Trivy or safe-update |
| `compose-secrets-contribution-contract.md` | Optional `secrets:` / `*_FILE`; `.env` stays the clone path |
| `qdrant-snapshot-compaction-contract.md` | Official snapshot API plus volume tar; no second vector DB |
| `logging-driver-journald-contract.md` | json-file default; opt-in journald without Loki |
| `helper-compose-watch-contract.md` | Bind-mount first; optional `develop.watch` |

## Expected sibling Stage 23 pages

| Page | Purpose |
|---|---|
| `nginx-compose-configs-contract.md` | Shared TLS/header snippets; optional Compose `configs:` |
| `minio-object-versioning-ilm-contract.md` | Opt-in object history via existing `mc` |
| `read-only-rootfs-proxy-contract.md` | `read_only` + tmpfs for Nginx, not LSM |
| `postgres-statement-timeout-pooling-contract.md` | Real `-c` flags; optional PgBouncer later |
| `qdrant-hnsw-quantization-contract.md` | Fit RAG in the 1G limit without a second vector DB |
| `host-timezone-ntp-contract.md` | One `TU_VM_TZ`; host clock; no time container |
| `inter-service-network-isolation-contract.md` | Role bridges beyond a single `ai_network` |

## Expected sibling Stage 24 pages

| Page | Purpose |
|---|---|
| `nginx-worker-env-wiring-contract.md` | Wire or delete unused `NGINX_WORKER_*` keys |
| `redis-acl-users-contract.md` | Optional ACL file on top of `requirepass` |
| `minio-sse-kms-contract.md` | Opt-in SSE-S3 or local KMS secret |
| `host-sysctl-tuning-contract.md` | Optional inotify / swappiness drop-in |
| `compose-depends-on-health-contract.md` | `service_healthy` when a probe exists |
| `backup-archive-encryption-contract.md` | Optional wrap so README matches `create_backup` |
| `healthcheck-interval-env-contract.md` | Wire or delete unused `HEALTH_CHECK_*` keys |

## Expected sibling Stage 25 pages

| Page | Purpose |
|---|---|
| `compose-stop-grace-period-contract.md` | SIGTERM drain on postgres/qdrant/minio/tika |
| `compose-init-tini-contract.md` | `init: true` on helper and processor |
| `compose-nonroot-user-contract.md` | Numeric `user:` on stateless first-party services |
| `processor-check-interval-energy-contract.md` | Honest `CHECK_INTERVAL` vs 10s poll |
| `mcp-pids-limit-sandbox-contract.md` | Fork caps on MCP tools and browserless |
| `compose-mem-swappiness-contract.md` | Per-container swap policy vs host sysctl |
| `tika-healthcheck-honesty-contract.md` | Real `/tika` probe so Stage 24 can use `service_healthy` |

## Expected sibling Stage 26 pages

| Page | Purpose |
|---|---|
| `chromium-shm-size-contract.md` | 1G `/dev/shm` on browserless and mcp-playwright |
| `n8n-mcp-digest-pin-contract.md` | Pin the last floating official sidecar |
| `mcp-tier2-restart-honesty-contract.md` | `restart: "no"` on filesystem/fetch/memory |
| `tika-minio-resource-limits-contract.md` | Cgroup CPU/RAM on the unbounded Tier 1 pair |
| `processor-tmpfs-status-share-contract.md` | Scratch tmpfs without breaking PDF progress |
| `ulimits-nofile-datastores-contract.md` | Copy Nginx `ulimits.nofile` to postgres/minio |
| `oom-score-adj-priority-contract.md` | Kill Ollama before Pi-hole and Postgres |

## Expected sibling Stage 27 pages

| Page | Purpose |
|---|---|
| `helper-python-image-pin-contract.md` | Digest-pin `python:3-alpine` until Stage 22 bake |
| `helper-status-healthcheck-honesty-contract.md` | Probe existing `/health` so Nginx can wait |
| `cpu-shares-blkio-weight-contract.md` | Relative scheduler weights vs hard cgroup caps |
| `helper-host-tmp-isolation-contract.md` | Cron JSON off host `/tmp` (not the Tika status file) |
| `helper-docker-gid-group-add-contract.md` | Non-root helper leftover from Stage 25 |
| `helper-resource-limits-contract.md` | `deploy.resources` on `helper_index` |
| `docker-socket-proxy-contract.md` | Tecnativa proxy instead of a raw socket bind |

## Expected sibling Stage 28 pages

| Page | Purpose |
|---|---|
| `pull-policy-missing-digest-contract.md` | `pull_policy: missing` so day-to-day `up` does not re-pull |
| `unbounded-tier2-resource-limits-contract.md` | `deploy.resources` leftover after Stages 26–27 |
| `n8n-healthz-when-started-contract.md` | Enable the commented `/healthz` probe |
| `compose-extension-fields-dry-contract.md` | `x-*` anchors for repeated DNS (and later logging) |
| `affine-redis-requirepass-contract.md` | Auth parity with platform Redis |
| `n8n-postgres-service-dns-contract.md` | `DB_POSTGRESDB_HOST: postgres` not a static IP |
| `compose-include-split-contract.md` | Official `include:` instead of a custom splitter |

## Expected sibling Stage 29 pages

| Page | Purpose |
|---|---|
| `healthcheck-start-interval-contract.md` | Compose `start_interval` for fast first ready |
| `nginx-tmpfs-runtime-contract.md` | Official writable paths so `read_only` can land |
| `n8n-binary-data-volume-contract.md` | Filesystem mode path + prune honesty |
| `compose-service-label-catalog-contract.md` | `tu-vm.tier` / `tu-vm.role` for generators |
| `openwebui-tika-service-dns-contract.md` | Open WebUI `tika` / `minio` instead of `ai_*` URLs |
| `postgres-shm-size-contract.md` | `/dev/shm` for official shared_buffers |
| `redis-healthcheck-auth-env-contract.md` | `REDISCLI_AUTH` instead of `redis-cli -a` |

## Framework reuse (summary)

| Need | Prefer | Avoid |
|---|---|---|
| Proposal intake | GitHub Issues (suggestion form) | Local voting DB / community API |
| Docs website | Static framework after Stage 3 gates (Starlight default) | Custom SSR community portal |
| Control plane | Existing Nginx dashboard + helper | Rebuilding ops UI as a SPA prerequisite |
| First-boot readiness | Compose `start_interval` + `start_period` | Helper scrape or sleep loops |
| Read-only Nginx | Official tmpfs on `/run` and `/var/cache/nginx` | Writable root or a pid sidecar |
| n8n execution blobs | Official filesystem directory + prune | A second MinIO execution bucket |
| Service catalog | Compose `tu-vm.*` labels | A YAML registry beside Compose |
| Tika / MinIO URLs | Compose service DNS | `container_name` or IPAM literals |
| Postgres `/dev/shm` | Compose `shm_size` matching `shared_buffers` | `ipc: host` or Chromium's 1G value |
| Redis probe auth | Official `REDISCLI_AUTH` | `redis-cli -a` on argv |
| Pi-hole v6 env | Official `FTLCONF_*` | v5 `WEBPASSWORD` / `PIHOLE_DNS_` / `ServerIP` |
| MinIO identity | `${MINIO_ROOT_USER}` everywhere | Hardcoded `admin` in Compose / processor / `mc` |
| Leftover MCP URLs | Compose service DNS (`n8n`, `affine`) | `http://ai_n8n:5678` defaults |
| n8n behind Nginx | Official `N8N_PROXY_HOPS` + `N8N_SECURE_COOKIE` | Cookie-rewrite sidecars |
| Qdrant / MinIO probes | Binaries the image ships (`wget` / `mc ready`) | Assumed `curl` |
| Compose project id | Top-level `name:` | Directory-name lock scripts |
| n8n login vs gateway | Document gateway-only keys; official n8n auth | A second n8n auth product |
| Image pull on daily `up` | Stage 28 `pull_policy: missing` | A pull daemon or Watchtower-on-start |
| Container log retention | Stage 11 `logging:` max-size | A new logging product or Loki default |
| Privilege drop | Stage 18 `no-new-privileges` **code** | Another hardening essay |
| Image CVEs / pins | Stage 4 Trivy + Stage 15 safe-update | Treating labels as supply-chain |

## Navigation for a future docs site

Map this folder to a docs section such as `Community`:

1. How to submit → status board → decision log → implemented showcase
2. Day-to-day tools → Pi-hole FTLCONF / MinIO root user / leftover MCP DNS
3. n8n proxy hops → healthcheck binaries → Compose `name:` → n8n auth honesty
4. Quality gates → merge playbook

When a static framework is adopted, **move or mount** these pages into the docs content root; do not maintain two editable copies.

Extend `stage-merge-playbook.md` through **Stage 30** when reconciling hubs so sibling pages from PRs #27–#55 are not lost.

## Related archive and planning docs

- [`../README.md`](../README.md) — suggestions hub
- [`../index.md`](../index.md) — community suggestions map
- [`../website-community-pages.md`](../website-community-pages.md) — lifecycle vocabulary
- [`../implementation-backlog.md`](../implementation-backlog.md) — trimmed execution backlog
- [`../website-roadmap-from-historical-suggestions.md`](../website-roadmap-from-historical-suggestions.md) — historical product direction

## Consolidated entries from parallel suggestion branches

| [`.github/dependabot.yml`](../../.github/dependabot.yml) | Already updates GitHub Actions; the docker ecosystem is the matching knob for image bumps |
| [`.github/pull_request_template.md`](../../.github/pull_request_template.md) | The checklist to deduplicate |
| [`tika-minio-processor/Dockerfile`](../../tika-minio-processor/Dockerfile) | Pattern for a pinned helper image |
- [`../historical-suggestions.md`](../historical-suggestions.md) — changelog ideas
- When a static docs framework is adopted (reuse-first default: Astro Starlight; see [`../website-and-docs-framework.md`](../website-and-docs-framework.md)), map this folder to `docs/community/` (or the configured content root) **without** maintaining two editable copies:
| [Access mode and firewall contract](./access-mode-and-firewall-contract.md) | Safe community path for `secure` / `public` / `locked` and UFW |
| [AFFiNE backup volume coverage](./affine-backup-volume-coverage-contract.md) | Include AFFiNE volumes in the existing backup loop |
| [AFFiNE collaboration contribution contract](./affine-collaboration-contribution-contract.md) | Safe community recipes around the existing AFFiNE workspace |
| [AFFiNE Compose start](./affine-compose-start-contract.md) | Dashboard Start must bring up the dependency group |
| [AFFiNE MCP command fallback](./affine-mcp-command-fallback-contract.md) | Code default matches bin `affine-mcp` |
| [AFFiNE Redis durability](./affine-redis-durability-contract.md) | Named volume + explicit AOF/RDB; do not share platform Redis |
| [AFFiNE Redis requirepass](./affine-redis-requirepass-contract.md) | Auth parity with platform Redis |
| [AI pipeline contribution contract](./ai-pipeline-contribution-contract.md) | Community path for MCP gateway and LangGraph supervisor changes |
| [Air-gapped docs mirror](./airgap-docs-mirror.md) | Offline/static docs pack for LAN and disconnected operators |
| [Allowlist network rewrite](./allowlist-network-rewrite-contract.md) | Shell rewrites must keep the networks Nginx and the helper already accept |
| [Announcement firewall status](./announcement-firewall-status-contract.md) | The public-access announcement reads `ufw status`, not the text of the `ufw allow` command |
| [Announcement health query](./announcement-health-query-contract.md) | A failed Docker or feed read must not look like a healthy stack |
| [Announcement service roster](./announcement-service-roster-contract.md) | The banner must cover the containers Compose actually runs |
| [MinIO API certificate name](./api-minio-certificate-name-contract.md) | The self-signed certificate must name `api.minio.tu.lan` |
| [AppArmor / seccomp profiles](./apparmor-seccomp-profile-contract.md) | Optional LSM profiles on top of existing resource limits |
| [Autonomous write-guard policy](./autonomous-write-guard-policy.md) | Day-to-day confirm, rate-limit, and circuit-breaker operator policy |
| [Backup archive encryption](./backup-archive-encryption-contract.md) | Optional wrap so README matches `create_backup` |
| [Backup archive mode](./backup-archive-mode-contract.md) | Backup tarballs are owner-readable, matching `.env` |
| [Backup restore apply](./backup-restore-apply-contract.md) | Apply the archive in place, then start Tier 1 only |
| [Backup/restore community drill](./backup-restore-community-drill.md) | Recurring reliability drill using existing backup/restore commands |
| [Backup rotation empty dump](./backup-rotation-empty-dump-contract.md) | A failed database dump does not become the only archive |
| [Backup running gate](./backup-running-gate-contract.md) | Volume archives do not depend on `grep -q` of `docker compose ps` |
| [Backup volume image pin](./backup-volume-image-pin-contract.md) | Volume archives are packed with the pinned Postgres image |
| [Backup volume prefix](./backup-volume-prefix-contract.md) | Archive the suffix `verify_data_retention` already accepts |
| [Battery and power operator signals](./battery-power-operator-signals.md) | Laptop battery widgets and energy-aware recommendations |
| [Beginner quickstart contribution contract](./beginner-quickstart-contribution-contract.md) | Portable-default onboarding without flipping silent full-stack starts |
| [Browserless automation contribution](./browserless-automation-contribution-contract.md) | Browserless + Playwright MCP contribution and ops rules |
| [Browserless token required](./browserless-token-required-contract.md) | Empty `TOKEN` must not leave Chromium anonymous |
| [Changelog-refresh contribution contract](./changelog-refresh-contribution-contract.md) | Local Unreleased section helper distinct from Release Drafter |
| [Changelog / release-notes contribution](./changelog-release-notes-contribution-contract.md) | Human + automation path for release communication |
| [Chat stream completion](./chat-stream-completion-contract.md) | `stream: true` must finish in the shape that client already expects |
| [Check-config contribution contract](./check-config-contribution-contract.md) | Secret-safe preflight for compose/env/allowlist without a second config product |
| [Check-config secret coverage](./check-config-secret-coverage-contract.md) | `--strict` flags published defaults `generate-secrets` already knows |
| [Checkup RAG config write](./checkup-rag-config-write-contract.md) | The daily checkup does not `UPDATE` the Open WebUI config row |
| [Chromium shm_size](./chromium-shm-size-contract.md) | 1G `/dev/shm` on browserless and mcp-playwright |
| [CI workflow contribution contract](./ci-workflow-contribution-contract.md) | Primary Actions gate that calls `scripts/` |
| [Circuit breaker threshold](./circuit-breaker-threshold-contract.md) | The quiet period must restore the configured failure threshold |
| [CLI subcommand contribution contract](./cli-subcommand-contribution-contract.md) | Safe community path for extending `tu-vm.sh` |
| [CODEOWNERS review routing contract](./codeowners-review-routing-contract.md) | Path ownership and review routing |
| [Community health digest](./community-health-digest.md) | Privacy-safe metrics published as static markdown |
| [Label and board contract](./community-label-and-board-contract.md) | GitHub labels + Projects as the community operating system |
| [Community quality gates](./community-quality-gates.md) | Day-to-day evidence map for contributors and maintainers |
| [Compose CI live profile](./compose-ci-live-profile.md) | Minimal Compose profile to exercise live `/status/full` |
| [Compose CLI selection](./compose-cli-selection-contract.md) | `tu-vm.sh` must pick the same Compose CLI the other scripts already pick |
| [Compose depends_on health](./compose-depends-on-health-contract.md) | `service_healthy` when a probe exists |
| [Compose extension fields DRY](./compose-extension-fields-dry-contract.md) | `x-*` anchors for repeated DNS (and later logging) |
| [Compose healthcheck contribution contract](./compose-healthcheck-contribution-contract.md) | Healthcheck design rules that keep `docker compose ps` honest |
| [Compose include split](./compose-include-split-contract.md) | Official `include:` instead of a custom splitter |
| [Compose init / tini](./compose-init-tini-contract.md) | `init: true` on helper and processor |
| [Compose IPAM address registry](./compose-ipam-address-registry.md) | Unique static IPv4 allocation on `172.20.0.0/16` so new services cannot collide |
| [Compose mem_swappiness](./compose-mem-swappiness-contract.md) | Per-container swap policy vs host sysctl |
| [Native Compose profiles](./compose-native-profiles-contract.md) | Align Compose `profiles:` with `tu-vm.sh` Tier arrays instead of a second start model |
| [Compose non-root user](./compose-nonroot-user-contract.md) | Numeric `user:` on stateless first-party services |
| [Compose override contribution contract](./compose-override-contribution-contract.md) | Safe community path for local Compose customizations |
| [Compose resource-budget contract](./compose-resource-budget-contract.md) | Energy-aware CPU/memory limits for community Compose changes |
| [Compose secrets](./compose-secrets-contribution-contract.md) | Optional `secrets:` / `*_FILE`; `.env` stays the clone path |
| [Compose service label catalog](./compose-service-label-catalog-contract.md) | `tu-vm.tier` / `tu-vm.role` for generators |
| [Compose stop grace period](./compose-stop-grace-period-contract.md) | SIGTERM drain on postgres/qdrant/minio/tika |
| [Container hardening baseline](./container-hardening-baseline-contract.md) | cap_drop / no-new-privileges / docker.sock review for Compose PRs |
| [Container log retention contract](./container-log-retention-contract.md) | Local log growth hygiene without a logging SaaS |
| [Control-plane contribution contract](./control-plane-contribution-contract.md) | Safe nginx allowlist and control-plane change rules |
| [CPU shares and blkio weight](./cpu-shares-blkio-weight-contract.md) | Relative scheduler weights vs hard cgroup caps |
| [Cross-service e2e scenario catalog](./cross-service-e2e-scenario-catalog.md) | Named scenarios that wrap `chain-smoke`, helper checks, and playbooks |
| [Checkup container state](./daily-checkup-container-state-contract.md) | Exact inspect status, and an empty JSON array |
| [Daily checkup contribution contract](./daily-checkup-contribution-contract.md) | Scheduled Tier-aware health/update status without a monitoring SaaS |
| [Checkup stays read-only](./daily-checkup-readonly-contract.md) | Do not UPDATE Open WebUI config or reload node types from the daily pulse |
| [Dashboard accessibility and mobile](./dashboard-accessibility-mobile.md) | A11y and touch-first acceptance baseline for the control plane |
| [Dashboard announcements contract](./dashboard-announcements-contract.md) | Priority taxonomy for helper-driven operator alerts |
| [Dashboard feature-flag experiments](./dashboard-feature-flag-experiments.md) | Env-driven UI experiments without a feature-flag platform |
| [Dashboard modularization and smoke](./dashboard-modularization-smoke.md) | Extract Nginx assets + Playwright Tier-1 smoke |
| [Dashboard release highlights](./dashboard-release-highlights-contract.md) | Optional “What is new” bullets from Releases/CHANGELOG on the LAN dashboard |
| [Dashboard untrusted text](./dashboard-untrusted-text-contract.md) | Log samples, filenames, and error bodies render as text |
| [Dashboard VM IP source](./dashboard-vm-ip-source-contract.md) | The KPI shows host `HOST_IP` or an empty value |
| [Data-plane hygiene contract](./data-plane-hygiene-contract.md) | Day-to-day Postgres / Redis / Qdrant hygiene without a DBA platform |
| [Day-to-day community tools](./day-to-day-community-tools.md) | Frameworks and commands that ease daily work |
| [Dead Postgres env honesty](./dead-postgres-env-honesty-contract.md) | Remove ignored `POSTGRES_*` GUC-shaped env; Stage 23 owns `-c` |
| [Debug loopback publish opt-in](./debug-loopback-publish-opt-in-contract.md) | Optional 127.0.0.1:8080/9000/9001/8081 |
| [Decision log](./decision-log.md) | Accepted / deferred / declined rationale without a second tracker |
| [Dependabot Actions maintenance contract](./dependabot-actions-maintenance-contract.md) | Weekly Actions bumps distinct from image CVE lanes |
| [Deprecation notice framework](./deprecation-notice-framework.md) | Breaking-change and deprecation communication pattern |
| [Dev Container contributor environment](./devcontainer-contributor-environment.md) | Repeatable Codespaces/Dev Containers setup for contributors |
| [Diagnostics and triage contract](./diagnostics-and-triage-contract.md) | `doctor` / `diagnose` / smoke as day-to-day triage tools |
| [Disk pressure and volume ops](./disk-pressure-volume-ops-contract.md) | Host disk pressure, LVM extend, and volume cleanup recipes |
| [Docker daemon DNS merge](./docker-daemon-dns-merge-contract.md) | A DNS override keeps the storage driver and is restored if the check fails |
| [Docker ps status column](./docker-ps-status-column-contract.md) | A running container must count as running |
| [Docker socket proxy](./docker-socket-proxy-contract.md) | Tecnativa proxy instead of a raw socket bind |
| [Docs framework adoption](./docs-framework-adoption.md) | Constructional runbook to adopt a static docs framework safely |
| [Docs link CI contribution contract](./docs-link-ci-contribution-contract.md) | Lychee workflow and exclude hygiene for community docs |
| [Document pipeline contribution contract](./document-pipeline-contribution-contract.md) | Tika + MinIO + `tika_minio_processor` contribution and ops rules |
| [Embedding batch input](./embedding-batch-input-contract.md) | Each input string gets its own vector and index |
| [Embedding result reader](./embedding-result-reader-contract.md) | An empty vector stays empty, and a null `result` does not crash the route |
| [Endpoint probe SNI](./endpoint-probe-sni-contract.md) | Named vhosts are only reached when SNI matches `server_name` |
| [Env example dead keys](./env-example-dead-keys-contract.md) | Delete or mark knobs nothing consumes |
| [Env schema contribution contract](./env-schema-contribution-contract.md) | Evolving `env.example` without secret leakage |
| [Disk extend device discovery](./extend-disk-device-contract.md) | Growth targets the root filesystem's volume, or exits when there is no LVM |
| [Extension pilot contract](./extension-pilot-contract.md) | First community extension package shape using Compose reuse |
| [Extension pilot scaffold](./extension-pilot-scaffold.md) | First `extensions/<id>/` package + `validate-extension.sh` |
| [External IP tile](./external-ip-kpi-contract.md) | Remove the KPI no payload fills |
| [Firewall command order](./firewall-command-order-contract.md) | `secure`, `public`, and `lock` follow UFW first-match order |
| [Fixture and contract-test contribution](./fixture-and-contract-test-contribution.md) | How community evolves fixtures and schema validators |
| [Gateway public docs](./gateway-public-docs-contract.md) | Anonymous routes on `oweb.tu.lan` stay liveness-only |
| [Gateway sidecar start](./gateway-sidecar-start-contract.md) | Starting the MCP Gateway also starts the n8n-mcp container Compose already names |
| [GitHub community automation contract](./github-community-automation-contract.md) | Stale, issue forms, and triage without a local queue |
| [GPU acceleration contribution](./gpu-acceleration-contribution-contract.md) | Optional GPU enablement tied to the hardware matrix |
| [Grafana dashboard-as-code](./grafana-dashboard-as-code-contract.md) | Provision JSON dashboards beside the existing datasource file |
| [Hardware-class intake](./hardware-class-intake.md) | Optional Issue-form `class_id` tied to the hardware matrix |
| [Hardware compatibility matrix](./hardware-compatibility-matrix.md) | Living community artifact for host/RAM/GPU/storage fit |
| [Healthcheck interval env](./healthcheck-interval-env-contract.md) | Wire or delete unused `HEALTH_CHECK_*` keys |
| [Healthcheck start interval](./healthcheck-start-interval-contract.md) | Compose `start_interval` for fast first ready |
| [Helper API contribution contract](./helper-api-contribution-contract.md) | Evolve `helper/uploader.py` and `/status/full` without breaking the dashboard |
| [Helper client IP trust](./helper-client-ip-trust-contract.md) | Bootstrap enrolls the peer and does not list the allowlist anonymously |
| [Helper Compose watch](./helper-compose-watch-contract.md) | Bind-mount first; optional `develop.watch` |
| [Helper-contract-check contribution contract](./helper-contract-check-contribution-contract.md) | Live helper JSON probes when `ai_helper_index` is up |
| [Helper control-auth contract](./helper-control-auth-contract.md) | `CONTROL_TOKEN`, header-first auth, and unauthenticated 401 behavior |
| [Helper control allowlist](./helper-control-service-allowlist-contract.md) | Unknown names never reach the Docker API |
| [Helper Docker API timeout](./helper-docker-api-timeout-contract.md) | Stop must return before the helper client gives up |
| [Helper docker GID `group_add`](./helper-docker-gid-group-add-contract.md) | Non-root helper leftover from Stage 25 |
| [Helper DOMAIN wiring](./helper-domain-env-wiring-contract.md) | Pass `DOMAIN` into `helper_index` |
| [Helper host `/tmp` isolation](./helper-host-tmp-isolation-contract.md) | Cron JSON off host `/tmp` (not the Tika status file) |
| [Helper image build](./helper-image-build-contract.md) | Build the helper from a lockfile instead of installing packages on every start |
| [Helper index image bake](./helper-index-image-bake-contract.md) | Bake apk/pip into `helper/Dockerfile`; keep the bind-mount |
| [Helper project mount](./helper-project-mount-contract.md) | Remove the unused repository bind so `.env` and TLS keys are not in the helper container |
| [Helper Python image pin](./helper-python-image-pin-contract.md) | Digest-pin `python:3-alpine` until Stage 22 bake |
| [Helper resource limits](./helper-resource-limits-contract.md) | `deploy.resources` on `helper_index` |
| [Helper healthcheck honesty](./helper-status-healthcheck-honesty-contract.md) | Probe existing `/health` so Nginx can wait |
| [Helper status probe DNS](./helper-status-probe-dns-contract.md) | TCP probes use service names; Docker API keeps `container_name` |
| [Helper uploader modules](./helper-uploader-module-contribution-contract.md) | Split `uploader.py` into importable modules without a second Flask app |
| [Host cleanup and prune contract](./host-cleanup-and-prune-contract.md) | Safe backup/log/image cleanup without surprise data loss |
| [Host cron maintenance contract](./host-cron-maintenance-contract.md) | Safe community path for checkup / update / MinIO sync crons |
| [Host metrics source](./host-metrics-source-contract.md) | Dashboard load, memory, disk, and battery read a declared host source |
| [Host port binding contract](./host-port-binding-contract.md) | Port 53 / 80 / 443 conflict detection and recovery |
| [Host sysctl tuning](./host-sysctl-tuning-contract.md) | Optional inotify / swappiness drop-in |
| [Host timezone / NTP](./host-timezone-ntp-contract.md) | One `TU_VM_TZ`; host clock; no time container |
| [How to submit](./how-to-submit.md) | High-signal suggestion writing and dedupe checks |
| [Image provenance / cosign](./image-provenance-cosign-contract.md) | Verify digest pins without replacing Trivy or safe-update |
| [Image update channel policy](./image-update-channel-policy.md) | Pin, check, update, and roll back with clear operator channels |
| [Image update compare](./image-update-compare-contract.md) | Compare manifest digests for the Compose pins operators actually run |
| [Implemented showcase](./implemented-showcase.md) | Changelog-adjacent proof that community ideas shipped |
| [Inter-service isolation](./inter-service-network-isolation-contract.md) | Role bridges beyond a single `ai_network` |
| [IP allowlist day-2 contract](./ip-allowlist-day2-contract.md) | Day-to-day whitelist ops distinct from control-plane PR security |
| [IPv6 dual-stack policy](./ipv6-dual-stack-policy-contract.md) | IPv4-first; dual-stack is all-or-nothing |
| [Knowledge-base public read](./kb-public-read-default-contract.md) | Anonymous reads off unless opted in |
| [Knowledge-base read byte cap](./kb-read-byte-cap-contract.md) | A get, extract, or index must stop at the same byte ceiling as put |
| [Knowledge search scan](./kb-search-scan-cap-contract.md) | A match past the first 500 index keys must still be findable |
| [LAN DNS client onboarding contract](./lan-dns-client-onboarding-contract.md) | Router and client first-hour DNS recipes |
| [Landing font privacy](./landing-font-privacy-contract.md) | The dashboard does not load typefaces from Google |
| [LangGraph supervisor smoke contract](./langgraph-supervisor-smoke-contract.md) | Day-2 trust-path smoke without a parallel QA platform |
| [Leftover timezone literals](./leftover-timezone-literal-contract.md) | `TU_VM_TZ` for n8n and Pi-hole after Stage 23 |
| [Local backup/restore contribution contract](./local-backup-restore-contribution-contract.md) | `tu-vm.sh backup` / `restore` format, rotation, and secret-safe archives |
| [Local resource usage history](./local-resource-history.md) | Local-only CPU/memory trend snapshots for operators |
| [Logging driver journald](./logging-driver-journald-contract.md) | json-file default; opt-in journald without Loki |
| [MCP catalog CI contract](./mcp-catalog-ci-contract.md) | `mcp-tools/catalog.yaml` schema + CI validation without a marketplace |
| [MCP catalog counts](./mcp-catalog-count-contract.md) | Landing copy must follow the schema functions and `nodeTypesCount` |
| [MCP Fetch CIDR deny](./mcp-fetch-cidr-deny-contract.md) | Block Fetch from reaching `172.20.0.0/16` and other link-local targets |
| [MCP gateway ops contribution contract](./mcp-gateway-ops-contribution-contract.md) | Auth-preserving gateway ops and tool routing contributions |
| [MCP pids_limit sandbox](./mcp-pids-limit-sandbox-contract.md) | Fork caps on MCP tools and browserless |
| [MCP Tier-2 restart honesty](./mcp-tier2-restart-honesty-contract.md) | `restart: "no"` on filesystem/fetch/memory |
| [MCP build pins](./mcp-tool-build-pin-contract.md) | Replace `@latest` with recorded versions |
| [MCP listener honesty](./mcp-tool-listener-auth-contract.md) | No host ports, no fake inbound bearer |
| [MCP tool restart policy](./mcp-tool-restart-policy-contract.md) | Filesystem, fetch, and memory stay stopped until an operator starts them |
| [MCP tool sandbox contribution contract](./mcp-tool-sandbox-contribution-contract.md) | Safe community path for Playwright, filesystem, fetch, and memory MCP images |
| [MCP tools community catalog](./mcp-tools-catalog.md) | Contribution contract for optional MCP tool images |
| [MinIO bucket lifecycle contract](./minio-bucket-lifecycle-contract.md) | Bucket naming, retention, and corpus hygiene |
| [MinIO mc client transport](./minio-mc-client-transport-contract.md) | Compose network and `mc alias set` arguments, not host networking |
| [MinIO versioning / ILM](./minio-object-versioning-ilm-contract.md) | Opt-in object history via existing `mc` |
| [MinIO public URL env](./minio-public-url-env-contract.md) | Interpolate console URL keys already in `env.example` |
| [MinIO SSE / KMS](./minio-sse-kms-contract.md) | Opt-in SSE-S3 or local KMS secret |
| [MinIO sync cron secret](./minio-sync-cron-secret-contract.md) | Start must not write the root password into crontab |
| [Multi-arch image policy](./multi-arch-image-policy-contract.md) | Digest pins that work on amd64 and arm64 without a second compose file |
| [n8n binary data volume](./n8n-binary-data-volume-contract.md) | Filesystem mode path + prune honesty |
| [n8n block env in Code nodes](./n8n-block-env-access-in-node-contract.md) | Official `N8N_BLOCK_ENV_ACCESS_IN_NODE` |
| [n8n community packages](./n8n-community-packages-contract.md) | Official package flags / optional env-managed allowlist |
| [n8n concurrency production limit](./n8n-concurrency-production-limit-contract.md) | Official `N8N_CONCURRENCY_PRODUCTION_LIMIT` (default 5) |
| [n8n encryption key honesty](./n8n-encryption-key-honesty-contract.md) | Require a unique key; drop the published Compose hex default |
| [Execution node data](./n8n-execution-node-data-contract.md) | A null node payload must still return the execution summary |
| [n8n execution save mode](./n8n-execution-save-mode-contract.md) | Official save keys; progress stays false |
| [n8n execution timeout](./n8n-execution-timeout-contract.md) | Official `EXECUTIONS_TIMEOUT` / `EXECUTIONS_TIMEOUT_MAX` |
| [n8n executions prune](./n8n-executions-data-prune-contract.md) | Official `EXECUTIONS_DATA_PRUNE*` keys |
| [n8n form-data file size](./n8n-formdata-file-size-contract.md) | Official `N8N_FORMDATA_FILE_SIZE_MAX=200` |
| [n8n Git bare-repo disable](./n8n-git-node-bare-repos-contract.md) | Official `N8N_GIT_NODE_DISABLE_BARE_REPOS` |
| [n8n healthz when started](./n8n-healthz-when-started-contract.md) | Enable the commented `/healthz` probe |
| [n8n hide usage page](./n8n-hide-usage-page-contract.md) | Official `N8N_HIDE_USAGE_PAGE=true` |
| [n8n hiring banner](./n8n-hiring-banner-contract.md) | Official `N8N_HIRING_BANNER_ENABLED=false` |
| [n8n Assistant module](./n8n-instance-ai-disabled-contract.md) | Official `N8N_DISABLED_MODULES=instance-ai` |
| [n8n list cursor](./n8n-list-cursor-contract.md) | Workflow and execution lists must keep the next page cursor |
| [n8n-mcp digest pin](./n8n-mcp-digest-pin-contract.md) | Pin the last floating official sidecar |
| [n8n-mcp sidecar contribution contract](./n8n-mcp-sidecar-contribution-contract.md) | Enrichment sidecar rules distinct from workflow catalogs |
| [n8n metrics](./n8n-metrics-disabled-contract.md) | Official `N8N_METRICS=false` |
| [n8n MFA enforced](./n8n-mfa-enforced-contract.md) | Official `N8N_MFA_ENFORCED_ENABLED` on the shared security-policy lock |
| [Node-type catalog](./n8n-node-type-catalog-contract.md) | Create and update must not pass while the node catalog is empty |
| [n8n node-types extraction contract](./n8n-node-types-extraction-contract.md) | Regenerate gateway node-types JSON from local n8n |
| [n8n node exclude list](./n8n-nodes-exclude-contract.md) | Official `NODES_EXCLUDE` default array |
| [n8n response body](./n8n-non-json-response-contract.md) | A successful plain-text or empty n8n body must not become a 502 |
| [n8n onboarding flow](./n8n-onboarding-flow-disabled-contract.md) | Official `N8N_ONBOARDING_FLOW_DISABLED=true` |
| [n8n payload size](./n8n-payload-size-max-contract.md) | Official `N8N_PAYLOAD_SIZE_MAX=16` |
| [n8n personal-space sharing](./n8n-personal-space-sharing-contract.md) | Official personal-space publish/share off |
| [n8n personalization](./n8n-personalization-contract.md) | Official `N8N_PERSONALIZATION_ENABLED=false` |
| [n8n Postgres schema](./n8n-postgres-schema-contract.md) | Schema `n8n` exists before n8n migrates |
| [n8n Postgres service DNS](./n8n-postgres-service-dns-contract.md) | `DB_POSTGRESDB_HOST: postgres` not a static IP |
| [n8n public API Swagger disabled](./n8n-public-api-swagger-disabled-contract.md) | Official `N8N_PUBLIC_API_SWAGGERUI_DISABLED=true`; API stays on for MCP |
| [n8n restrict file access](./n8n-restrict-file-access-contract.md) | Official `N8N_RESTRICT_FILE_ACCESS_TO` + dedicated volume |
| [n8n official task runners](./n8n-runners-enabled-contract.md) | `N8N_RUNNERS_ENABLED` — not a Justfile control plane |
| [n8n runner payload](./n8n-runners-max-payload-contract.md) | Official `N8N_RUNNERS_MAX_PAYLOAD` at 64 MiB |
| [n8n SameSite cookie](./n8n-samesite-cookie-contract.md) | Official `N8N_SAMESITE_COOKIE=strict` (Stage 30 owns Secure + hops) |
| [n8n settings-file permissions](./n8n-settings-file-permissions-contract.md) | Official `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS` |
| [n8n SSRF mesh allow](./n8n-ssrf-mesh-allow-contract.md) | Official SSRF protection plus `172.20.0.0/16` |
| [n8n templates disabled](./n8n-templates-disabled-contract.md) | Official `N8N_TEMPLATES_ENABLED=false` (+ optional `N8N_TEMPLATES_HOST`) |
| [n8n update deactivate](./n8n-update-deactivate-contract.md) | A failed update must not leave the workflow stopped |
| [n8n version notifications](./n8n-version-notifications-contract.md) | Official `N8N_VERSION_NOTIFICATIONS_ENABLED=false` |
| [n8n webhook vs login zone](./n8n-webhook-login-zone-split-contract.md) | Split locations so login can use `zone=login` without breaking webhooks |
| [n8n workflow allowlist](./n8n-workflow-allowlist-enforcement-contract.md) | A non-empty allowlist must cover list, execution, and webhook calls |
| [n8n workflow catalog](./n8n-workflow-catalog.md) | Community contribution path for reusable n8n recipes |
| [n8n workflow file export](./n8n-workflow-file-export-contract.md) | `n8n-workflows.json` is not a concatenation of every JSON file |
| [Nginx header inheritance](./nginx-add-header-inheritance-contract.md) | Locations keep the headers their server block already sets |
| [Nginx Compose configs](./nginx-compose-configs-contract.md) | Shared TLS/header snippets; optional Compose `configs:` |
| [Nginx edge routing contract](./nginx-edge-routing-contract.md) | Vhost, proxy, and conf contribution rules at the edge |
| [Nginx login rate-limit](./nginx-login-rate-limit-contract.md) | Apply the unused `login` zone to auth surfaces without a WAF product |
| [Nginx DOMAIN server_name](./nginx-server-name-domain-contract.md) | Rewrite static `*.tu.lan` vhosts when `DOMAIN` changes |
| [Nginx static health prefix](./nginx-static-health-prefix-contract.md) | Exact `location = /health` so `/health/db` and `/healthz` reach the app |
| [Nginx stub_status exporter pairing](./nginx-stub-status-exporter-contract.md) | Internal stub_status only, paired with opt-in nginx-exporter |
| [Nginx TLS cipher names](./nginx-tls-cipher-suite-contract.md) | Drop suite names OpenSSL does not implement |
| [Nginx tmpfs runtime](./nginx-tmpfs-runtime-contract.md) | Official writable paths so `read_only` can land |
| [Nginx Docker-DNS upstreams](./nginx-upstream-dns-contract.md) | Prefer Compose service names with an explicit resolver; keep static IPs as a fallback |
| [Nginx worker env wiring](./nginx-worker-env-wiring-contract.md) | Wire or delete unused `NGINX_WORKER_*` keys |
| [Node operation filter](./node-operation-filter-contract.md) | `get` must not keep parameters that are shown only for `getAll` |
| [Observability contribution contract](./observability-contribution-contract.md) | Grafana/Prometheus dashboard and alert contribution rules |
| [Offsite backup rclone contract](./offsite-backup-rclone-contract.md) | Optional remote backup lane reusing rclone + existing backup commands |
| [Ollama keep-alive unload](./ollama-keep-alive-unload-contract.md) | Official `OLLAMA_KEEP_ALIVE` / max loaded |
| [Ollama model catalog contribution](./ollama-model-catalog-contribution.md) | Community path for recommending and documenting local models |
| [OOM score priority](./oom-score-adj-priority-contract.md) | Kill Ollama before Pi-hole and Postgres |
| [Open WebUI audio / STT contract](./open-webui-audio-stt-contract.md) | Safe community path for speech-to-text config and repair helpers |
| [Open WebUI contribution contract](./open-webui-contribution-contract.md) | Safe community path for Open WebUI config, init, and tool wiring |
| [Open WebUI init / first-boot contract](./open-webui-init-firstboot-contract.md) | Idempotent first-boot patching distinct from day-2 loader toggles |
| [Open WebUI web-search contract](./open-webui-websearch-contract.md) | Loader and privacy rules for optional web-search features |
| [Open WebUI admin analytics](./openwebui-admin-analytics-contract.md) | Official `ENABLE_ADMIN_ANALYTICS=false` (UI tab, not Scarf) |
| [Admin chat access](./openwebui-admin-chat-access-contract.md) | Official `ENABLE_ADMIN_CHAT_ACCESS=false` |
| [Admin export](./openwebui-admin-export-contract.md) | Official `ENABLE_ADMIN_EXPORT=false` |
| [HTTP client timeout](./openwebui-aiohttp-client-timeout-contract.md) | Official `AIOHTTP_CLIENT_TIMEOUT=600` |
| [Outbound redirect following](./openwebui-aiohttp-redirects-contract.md) | Official `AIOHTTP_CLIENT_ALLOW_REDIRECTS=false` |
| [Open WebUI API key endpoints](./openwebui-api-keys-endpoint-contract.md) | Official `ENABLE_API_KEYS` + endpoint restrictions |
| [Open WebUI audio model parameter](./openwebui-audio-model-parameter-contract.md) | The repair must pass the model as data, and the checker must read it back as data |
| [Chunk env names](./openwebui-chunk-env-names-contract.md) | Official `CHUNK_SIZE` and `CHUNK_OVERLAP` |
| [Open WebUI code execution](./openwebui-code-execution-contract.md) | Official `ENABLE_CODE_EXECUTION=false` |
| [Open WebUI code interpreter](./openwebui-code-interpreter-contract.md) | Official `ENABLE_CODE_INTERPRETER=false` |
| [Open WebUI community sharing](./openwebui-community-sharing-contract.md) | Official `ENABLE_COMMUNITY_SHARING=false` |
| [Open WebUI Compose env honesty](./openwebui-compose-env-honesty-contract.md) | Drop unofficial `WEBUI_*` keys the image may ignore |
| [Open WebUI CORS origin](./openwebui-cors-origin-contract.md) | Official `CORS_ALLOW_ORIGIN` from `DOMAIN` |
| [Database pool](./openwebui-db-pool-contract.md) | Official SQLAlchemy pool size and overflow |
| [Default user role](./openwebui-default-user-role-contract.md) | Official `DEFAULT_USER_ROLE=pending` |
| [Open WebUI direct connections](./openwebui-direct-connections-contract.md) | Official `ENABLE_DIRECT_CONNECTIONS=false` |
| [Forward user info headers](./openwebui-forward-user-info-headers-contract.md) | Official `ENABLE_FORWARD_USER_INFO_HEADERS=false` |
| [Artifact iframe CSP](./openwebui-iframe-csp-contract.md) | Official `IFRAME_CSP` example policy |
| [Open WebUI init hook](./openwebui-init-hook-unwired-contract.md) | The mounted script has to be the entrypoint |
| [JWT expiry](./openwebui-jwt-expiry-contract.md) | Official `JWT_EXPIRES_IN=4w`; never `0` or `-1` |
| [Open WebUI ↔ MinIO sync day-2](./openwebui-minio-sync-day2-contract.md) | Operator contract for upload/TXT sync hygiene |
| [Open WebUI official env honesty](./openwebui-official-env-honesty-contract.md) | Wire or delete unused `WEBUI_*` keys |
| [Open WebUI offline embeddings](./openwebui-offline-embeddings-contract.md) | HF cache volume + official offline flags |
| [OpenAI API toggle](./openwebui-openai-api-disabled-contract.md) | Official `ENABLE_OPENAI_API=false` |
| [OpenAI API passthrough](./openwebui-openai-api-passthrough-contract.md) | Official `ENABLE_OPENAI_API_PASSTHROUGH=false` |
| [Open WebUI password validation](./openwebui-password-validation-contract.md) | Official `ENABLE_PASSWORD_VALIDATION=true` plus regex/hint |
| [Open WebUI PDF engine keys](./openwebui-pdf-engine-keys-contract.md) | The loader switch writes the config paths this image reads |
| [Open WebUI PersistentConfig honesty](./openwebui-persistent-config-honesty-contract.md) | Official `ENABLE_PERSISTENT_CONFIG=false` for compose-as-source-of-truth |
| [Open WebUI pip frontmatter](./openwebui-pip-install-frontmatter-contract.md) | Official `ENABLE_PIP_INSTALL_FRONTMATTER_REQUIREMENTS=false` |
| [Profile image data URI size](./openwebui-profile-image-data-uri-size-contract.md) | Official `PROFILE_IMAGE_MAX_DATA_URI_SIZE` |
| [Profile image forwarding](./openwebui-profile-image-url-forwarding-contract.md) | Official `ENABLE_PROFILE_IMAGE_URL_FORWARDING=false` |
| [Active user count](./openwebui-public-active-users-contract.md) | Official `ENABLE_PUBLIC_ACTIVE_USERS_COUNT=false` |
| [Open WebUI public URL](./openwebui-public-url-contract.md) | Official `WEBUI_URL` = Nginx oweb vhost |
| [Open WebUI Qdrant client budget](./openwebui-qdrant-client-budget-contract.md) | New collections and client calls follow the flags the pinned image reads |
| [RAG upload limits](./openwebui-rag-upload-limits-contract.md) | Official size, count, and extension keys |
| [Open WebUI recovery command](./openwebui-recovery-command-contract.md) | The health hint runs a command that exists, and that command does not restart Docker |
| [Open WebUI Scarf analytics](./openwebui-scarf-analytics-contract.md) | Official `SCARF_NO_ANALYTICS=true` next to existing DNT |
| [Open WebUI session cookies](./openwebui-session-cookie-contract.md) | Official `WEBUI_SESSION_COOKIE_*` / `WEBUI_AUTH_COOKIE_*` |
| [Open WebUI signup / first user](./openwebui-signup-first-user-contract.md) | Official `ENABLE_SIGNUP` after first admin |
| [Open WebUI Tika metadata flag](./openwebui-tika-metadata-flag-contract.md) | The Compose flag is unread; the loader keeps text, not a metadata record |
| [Open WebUI Tika service DNS](./openwebui-tika-service-dns-contract.md) | `tika` / `minio` instead of `ai_*` URLs |
| [Tool-call iterations](./openwebui-tool-call-iteration-contract.md) | Official cap of 8; never `-1` |
| [Open WebUI version update check](./openwebui-version-update-check-contract.md) | Official `ENABLE_VERSION_UPDATE_CHECK=false` |
| [Fetch URL length](./openwebui-web-fetch-length-contract.md) | Official `WEB_FETCH_MAX_CONTENT_LENGTH=50000` |
| [Operator profile CLI](./operator-profile-cli.md) | `tu-vm.sh profile` list/show/apply `--plan` thin wrapper |
| [Operator service profiles](./operator-service-profiles.md) | Named Tier 1/Tier 2 profiles for day-to-day resource control |
| [Optional command arguments](./optional-command-argument-contract.md) | Help text that marks an argument optional must survive `set -u` |
| [PDF command probes](./pdf-cli-probe-contract.md) | Pipeline checks reach Tika on the Docker network and MinIO with the password from `.env` |
| [PDF loader switch day-2](./pdf-loader-switch-day2-contract.md) | Tika / PyMuPDF toggle rules for day-to-day ops |
| [PDF operator day-2 contract](./pdf-operator-day2-contract.md) | Operator toolkit for PDF pipeline status, test, logs, and reset |
| [PDF test working directory](./pdf-test-workdir-contract.md) | `pdf-test` must upload the file it generated and log in the project |
| [Persona entry paths](./persona-entry-paths.md) | First-hour routes for operators, docs, and integration contributors |
| [Pi-hole backup gravity](./pihole-backup-gravity-contract.md) | Backup must not download blocklists into the live resolver |
| [Pi-hole DNS hygiene contribution](./pihole-dns-hygiene-contribution.md) | Day-to-day LAN DNS / allowlist hygiene without a second DNS product |
| [Pi-hole DNS restore order](./pihole-dns-restore-order-contract.md) | Host DNS switches to Pi-hole after the container is up |
| [Pi-hole domain suffix](./pihole-domain-suffix-contract.md) | Local DNS names and the URLs the control script prints follow `DOMAIN` |
| [Dual-stack DNS records](./pihole-dual-stack-dns-record-contract.md) | LAN and Tailscale names must land in the mounted dnsmasq file |
| [Pi-hole dual-stack hosts](./pihole-dual-stack-hosts-contract.md) | The mounted snippet must load the hosts file sync-dns writes |
| [Pi-hole published password honesty](./pihole-published-password-honesty-contract.md) | Required `PIHOLE_PASSWORD`; no published `SwissPiHole2024!` fallback |
| [Pi-hole admin password quotes](./pihole-webpassword-quote-contract.md) | The Compose default must not include the quote characters |
| [Pipeline worker on the dashboard](./pipeline-worker-dashboard-contract.md) | Poll `/status/tika-processor` from the Tier 1 grid |
| [Platform Redis durability](./platform-redis-durability-contract.md) | Make `redis_data` + AOF/RDB an explicit policy for `ai_redis` |
| [Playbook version matrix](./playbook-version-matrix.md) | Release-aware notes for operator playbooks |
| [Podman / rootless path](./podman-rootless-contributor-path-contract.md) | Optional Docker-compatible runtime lane; keep Compose as source of truth |
| [Postgres runtime GUCs](./postgres-runtime-guc-contract.md) | Tuning values on the Postgres service are server settings, not unused environment names |
| [Postgres shm size](./postgres-shm-size-contract.md) | `/dev/shm` for official shared_buffers |
| [Postgres GUCs / pooling](./postgres-statement-timeout-pooling-contract.md) | Real `-c` flags; optional PgBouncer later |
| [Platform Postgres WAL/PITR](./postgres-wal-pitr-policy-contract.md) | Name dump-plus-volume as default; opt-in WAL behind `tu-vm.sh` |
| [Pull request checklist](./pr-template-checklist-contract.md) | One security section and one verification section |
| [Pre-commit hooks contribution contract](./pre-commit-hooks-contribution-contract.md) | Optional local hooks wrapping shared syntax gates |
| [Pre-push check contribution contract](./pre-push-check-contribution-contract.md) | Fast local contributor gate wrapping shared scripts |
| [Privacy-preserving usage analytics](./privacy-preserving-usage-analytics.md) | Local-only usage patterns and recommendations without SaaS telemetry |
| [Privileged host-ops contract](./privileged-host-ops-contract.md) | Sudoers, disk extend, and host privilege contribution rules |
| [Archive expansion budget](./processor-archive-expansion-budget-contract.md) | Cap members, uncompressed bytes, and ratio |
| [Processor archive suffix and object key](./processor-archive-identity-contract.md) | `.tar.gz` must extract, and two sources must not share one `.txt` |
| [Processor bucket context](./processor-bucket-context-contract.md) | A failed file does not leave the processor pointed at that watch bucket |
| [Processor check interval energy](./processor-check-interval-energy-contract.md) | Honest `CHECK_INTERVAL` vs 10s poll |
| [Watcher attempt cap](./processor-file-attempt-cap-contract.md) | Finite attempts, including exceptions |
| [Processor meta Accept](./processor-meta-accept-contract.md) | `/meta` is asked for JSON, so the CSV default is not parsed as an object |
| [Processor metadata header](./processor-metadata-header-contract.md) | Extracted text is stored when Tika metadata does not fit in the object header |
| [Processor MinIO endpoint](./processor-minio-endpoint-fallback-contract.md) | A failed DNS lookup does not pin the watcher to a copied MinIO address |
| [Watcher object budget](./processor-object-byte-budget-contract.md) | Stop `Body.read()` before the 512M limit |
| [Watcher page size](./processor-object-list-pagination-contract.md) | Reuse the existing ContinuationToken loop |
| [Placeholder objects](./processor-placeholder-object-contract.md) | `.gitkeep` and other unknown keys are not sent to Tika |
| [Processor tmpfs and status share](./processor-tmpfs-status-share-contract.md) | Scratch tmpfs without breaking PDF progress |
| [Prometheus exporter opt-in](./prometheus-exporter-opt-in-contract.md) | Keep exporters off the always-on energy budget unless an operator opts in |
| [Proof signing key](./proof-signing-key-contract.md) | The HMAC key must be generated, and the rollout gate must check it |
| [Public origin DOMAIN](./public-origin-domain-contract.md) | Interpolate `DOMAIN` into n8n / AFFiNE / MinIO public URLs |
| [Pull policy on digest pins](./pull-policy-missing-digest-contract.md) | `pull_policy: missing` so day-to-day `up` does not re-pull |
| [Qdrant API key](./qdrant-api-key-contract.md) | Official Qdrant + Open WebUI API key pair |
| [Qdrant health probe](./qdrant-healthcheck-probe-contract.md) | The pinned image has no `curl` |
| [Qdrant HNSW / quantization](./qdrant-hnsw-quantization-contract.md) | Fit RAG in the 1G limit without a second vector DB |
| [Qdrant service API key](./qdrant-service-api-key-contract.md) | Official Qdrant + Open WebUI API key |
| [Qdrant snapshot / compaction](./qdrant-snapshot-compaction-contract.md) | Official snapshot API plus volume tar; no second vector DB |
| [QM LLM bridge contribution](./qm-llm-bridge-contribution-contract.md) | Community contract for the queue-manager LLM bridge service |
| [Queue response text](./qm-response-text-contract.md) | A completed text reply must survive the word `result` |
| [Quickstart flag combination](./quickstart-flag-combination-contract.md) | `--server` and `--no-secure` must work together |
| [RAG knowledge contribution contract](./rag-knowledge-contribution-contract.md) | Community docs and corpora via MinIO/Tika/Qdrant/Open WebUI |
| [rclone MinIO config](./rclone-minio-config-contract.md) | Host mounts follow `MINIO_ROOT_PASSWORD` after the first write |
| [Read-only rootfs proxies](./read-only-rootfs-proxy-contract.md) | `read_only` + tmpfs for Nginx, not LSM |
| [Redis ACL users](./redis-acl-users-contract.md) | Optional ACL file on top of `requirepass` |
| [Redis healthcheck auth env](./redis-healthcheck-auth-env-contract.md) | `REDISCLI_AUTH` instead of `redis-cli -a` |
| [Redis memory headroom](./redis-maxmemory-headroom-contract.md) | `maxmemory` and the client query cap sit below the 256M cgroup |
| [Release canary community program](./release-canary-community-program.md) | Lightweight pre-release testing using GitHub + existing smoke tools |
| [Release Drafter contribution contract](./release-drafter-contribution-contract.md) | Label-driven draft releases without a second changelog bot |
| [Restart preserves the running set](./restart-preserves-running-set-contract.md) | `./tu-vm.sh restart` must bring back the services that were running |
| [restic / borg backup backend](./restic-borg-backup-backend-contract.md) | Optional backend behind `tu-vm.sh backup` / `restore` |
| [Rollout gates / proof-store contract](./rollout-gates-proof-store-contract.md) | Pre/post-release proof-store quality gates |
| [Safe-update contribution contract](./safe-update-contribution-contract.md) | Extend pin maps and check/apply/rollback without a second updater |
| [Safe-update wrapper](./safe-update-wrapper-contract.md) | One update implementation, with the old script as a flag-compatible wrapper |
| [Secret generation identity](./secret-generation-identity-contract.md) | One generated value per service, and a proof key that is not a public placeholder |
| [Secret rotation and credential hygiene](./secret-rotation-credential-hygiene.md) | Day-to-day secret lifecycle using `generate-secrets` and playbooks |
| [Security reporting contribution contract](./security-reporting-contribution-contract.md) | Private advisory-first vulnerability path for community reporters |
| [Seed chat context contribution](./seed-chat-context-contribution-contract.md) | Community path for assistant context bootstraps |
| [Seed context image pin](./seed-context-image-pin-contract.md) | The context pack is copied without pulling `alpine:latest` |
| [Service dependency map](./service-dependency-map.md) | Declared dependencies for safe Tier 2 start ordering |
| [Service probe credentials](./service-probe-credentials-contract.md) | Retention, checkup, and the PDF loader switch authenticate the way the audio tools already do |
| [Smart startup optimization](./smart-startup-optimization.md) | Tier-aware quick-start planner without a new orchestrator |
| [Smoke test contribution contract](./smoke-test-contribution-contract.md) | Static default + optional Tier-1 `--live` probes |
| [Source text files](./source-text-file-contract.md) | A source `.txt` must reach the text processor that already exists |
| [Stage merge playbook](./stage-merge-playbook.md) | Reconcile Stage 1–3 `website/` hubs without losing siblings |
| [Startup secret gate](./startup-secret-gate-contract.md) | `./tu-vm.sh start` flags every published default `generate_secrets` can replace |
| [Status access curl](./status-access-curl-contract.md) | "Services are accessible" follows Tier 1 health, not a static curl |
| [Status board](./status-board.md) | Curated projection of suggestion lifecycle |
| [Stop preserves containers](./stop-preserves-containers-contract.md) | `./tu-vm.sh stop` must leave containers for the dashboard Start button |
| [Stopped container prune](./stopped-container-prune-contract.md) | Cleanup must leave the Tier 2 containers the dashboard starts |
| [Suggestion corpus registry](./suggestion-corpus-registry.md) | YAML registry + CI so historical suggestions are discoverable, not rewritten |
| [Supervisor callback credential](./supervisor-callback-credential-contract.md) | Only the supervisor token may skip a second delegation |
| [Supervisor write-tool set](./supervisor-write-tool-set-contract.md) | Deletes join the same strict-verify and dedupe set as other n8n writes |
| [Supply-chain community gates](./supply-chain-community-gates.md) | Image CVE scans, severity gates, optional SBOM on release |
| [Sync client image pin](./sync-client-image-pin-contract.md) | The upload sync does not pull `alpine:latest` or untagged `minio/mc` |
| [systemd user-unit install](./systemd-user-unit-install-contract.md) | Optional oneshot `--user` unit; not a second supervisor |
| [Tailscale LAN bridge contract](./tailscale-lan-bridge-contract.md) | Optional Tailscale dual-stack access using existing `sync-dns` |
| [Gzip tar suffix](./tar-gz-archive-suffix-contract.md) | A `.tar.gz` upload must be opened as a tar archive |
| [Task runner wrapper](./task-runner-wrapper.md) | `just`/`make` thin facade over `tu-vm.sh` and scripts |
| [Tier 1 startup gate](./tier1-startup-gate-contract.md) | A health status of `starting` must not skip bucket setup and cron |
| [Tier 2 idle auto-stop policy](./tier2-idle-autostop-policy.md) | Opt-in idle timeout for heavy on-demand services |
| [Tika client and server timeout](./tika-client-server-timeout-contract.md) | The processor must not wait longer than Tika's task cap |
| [Tika healthcheck honesty](./tika-healthcheck-honesty-contract.md) | Real `/tika` probe so Stage 24 can use `service_healthy` |
| [Tika-MinIO processor Python](./tika-minio-processor-python-contribution-contract.md) | Watch/retry/status changes stay in the existing Alpine worker |
| [Tika and MinIO resource limits](./tika-minio-resource-limits-contract.md) | Cgroup CPU/RAM on the unbounded Tier 1 pair |
| [Tika OCR runtime](./tika-ocr-runtime-contract.md) | OCR uses the official full image and `timeoutSeconds`, not a no-op XML element on the minimal image |
| [Tika pipeline readiness](./tika-pipeline-readiness-contract.md) | Wait until Tika listens, with a probe the image can run |
| [Tika update track](./tika-update-track-contract.md) | `./tu-vm.sh update` follows the 3.3.1 full tag, not Tika 4.1.0 `latest` |
| [Tika XML config contribution](./tika-xml-config-contribution-contract.md) | Safe community path for `tika-config/tika-config.xml` without a second extractor |
| [TLS certificate lifecycle contract](./tls-certificate-lifecycle-contract.md) | Self-signed and CA-ready cert lifecycle using existing SSL helpers |
| [TLS certificate SAN refresh](./tls-certificate-san-refresh-contract.md) | The self-signed certificate includes the IPv4 names the default vhost already allows |
| [Tool catalog partial failure](./tool-catalog-partial-failure-contract.md) | A missing AFFiNE login must not hide n8n and knowledge-base tools |
| [Datastore nofile ulimits](./ulimits-nofile-datastores-contract.md) | Copy Nginx `ulimits.nofile` to postgres/minio |
| [Unbounded Tier 2 resource limits](./unbounded-tier2-resource-limits-contract.md) | `deploy.resources` leftover after Stages 26–27 |
| [Unverified write dedupe](./unverified-write-dedupe-contract.md) | A failed verification must not block a retry of the same intent |
| [Update Check Now button](./update-check-button-contract.md) | The button must not report a placeholder as up to date |
| [Update rollback snapshot](./update-rollback-compose-snapshot-contract.md) | Rollback restores the compose file from this update and does not start every service |
| [Update status file](./update-status-file-contract.md) | One status directory; the helper must not clobber a timezone-aware checkup |
| [Webhook confirm gate](./webhook-confirm-gate-contract.md) | A webhook dispatch must require the confirm flag the other writes use |
| [Website frontmatter CI contract](./website-frontmatter-ci-contract.md) | Schema and link checks for publishable website markdown |
| [Weekly cron schedule clock](./weekly-cron-schedule-timezone-contract.md) | Sunday 03:30 Europe/Zurich must be the time the job actually runs |
| [Weekly stack update contribution contract](./weekly-stack-update-contribution-contract.md) | Cron-friendly full-stack update lane distinct from daily checkup and safe-update digests |
| [Weekly apt prompt](./weekly-update-apt-prompt-contract.md) | `apt-get upgrade` in the weekly job must not wait on a config-file question |
| [Allowlist public-IP fallback](./whitelist-add-public-ip-contract.md) | Stop calling a public IP service when the host route is missing |
| [Allowlist writer syntax](./whitelist-writer-syntax-contract.md) | `whitelist-add` and `whitelist-remove` must run the helper's writer |
| [Workflow Operator contribution](./workflow-operator-contribution-contract.md) | Open WebUI Workflow Operator / autonomous n8n engineering lane |
| [Write approval fail-closed](./write-approval-fail-closed-contract.md) | An empty approval-token list must deny writes |
| [WSL2 contributor path](./wsl2-contributor-path-contract.md) | Windows contributors use WSL2 + the same bash scripts |
