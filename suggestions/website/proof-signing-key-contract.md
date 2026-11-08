---
title: Proof Signing Key Contract
description: Constructional contract so generate_secrets replaces CHANGE_ME_PROOF_HMAC and the existing rollout gate checks the HMAC on proof rows.
last_updated: 2026-11-08
owner: maintainers
status: proposed
theme: security
impact: high
---

# Proof Signing Key Contract

## Problem

Proof-of-execution rows are signed with `MCP_PROOF_SIGNING_KEY`. A fresh copy of `env.example` sets that key to the public placeholder `CHANGE_ME_PROOF_HMAC`. `generate_secrets` never replaces it.

Checked on 2026-11-08. The same `sed` substitutions `generate_secrets` runs were applied to a copy of `env.example` under `/tmp`, with markers in place of `openssl` output. `.env` was not created in the repo. No secret value was printed.

| After those substitutions | Result |
|---|---|
| `MCP_GATEWAY_TOKEN` | Replaced (`CHANGE_ME_MCP_TOKEN` is in the sed list) |
| `MCP_PROOF_SIGNING_KEY` | Still `CHANGE_ME_PROOF_HMAC` |
| Other live `CHANGE_ME` assignments | None. Two `CHANGE_ME_LONG_RANDOM` hits are comments on the browserless lines |

`tu-vm.sh` has zero mentions of `CHANGE_ME_PROOF`. Compose still passes the variable through:

```text
MCP_PROOF_SIGNING_KEY: ${MCP_PROOF_SIGNING_KEY:-}
```

`_sign_proof` in `mcp-gateway/app.py` does this:

```text
secret = os.getenv("MCP_PROOF_SIGNING_KEY", os.getenv("MCP_GATEWAY_TOKEN", ""))
if not secret:
    return ""
return hmac.new(secret, canonical_json, sha256).hexdigest()
```

Python 3.12 replay of that function:

| `MCP_PROOF_SIGNING_KEY` | Signature for a dummy `{"claimed": true, "tool": "demo"}` row |
|---|---|
| `CHANGE_ME_PROOF_HMAC` | A 64-character hex digest. The placeholder is a working HMAC key |
| `""` (variable set, empty) | `""`. `os.getenv` returns the empty string and does not use `MCP_GATEWAY_TOKEN` |
| Unset, `MCP_GATEWAY_TOKEN` set | A digest. The existing fallback applies only when the variable is unset |

Anyone who can read `env.example` can compute the same digest the gateway writes for a copied `.env`. `scripts/rollout-gates.sh` loads the proof JSON and scores `claimed` against `verificationEvidence.verified`. A search of that script finds no `signature` field. A row can pass the rate gate with any signature, including a digest made from the placeholder.

Stage 54 (open PR #80) owns the six keys that share `CHANGE_ME_SECURE_PASSWORD`, and the credential card that prints a different value than the file. This page does not change those keys or that card. Stage 53 (open PR #79) owns the start-up detectors versus the defaults `generate_secrets` already knows. Stage 14 (open PR #40) owns the claimed-verify rate and the mismatch rate. Those thresholds stay.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `env.example` `MCP_PROOF_SIGNING_KEY` | The placeholder to replace |
| `generate_secrets` sed for `CHANGE_ME_MCP_TOKEN` | The pattern to copy for this one key |
| `set_env_var_if_default` | Already used for the other generated keys |
| `_sign_proof` | The HMAC to keep |
| `scripts/rollout-gates.sh` | The reader that should check `signature` |
| Stage 54 `secret-generation-identity-contract.md` (open PR #80) | Shared password placeholder. Unchanged |
| Stage 14 `rollout-gates-proof-store-contract.md` (open PR #40) | Rate gate. Unchanged |

Out of scope:

- A key-management service, HSM, or public transparency log
- Printing the proof key on the operator credential card
- Changing `MCP_GATEWAY_TOKEN` generation
- Shipping proof-store contents to GitHub Issues

## Proposal

One generated key, one HMAC function, and a check in the gate script that already reads the store.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Generate | `generate_secrets` | `sed` replace of `CHANGE_ME_PROOF_HMAC` with its own `openssl rand -hex 32` value, the same shape as `CHANGE_ME_MCP_TOKEN` |
| Second run | `set_env_var_if_default` | Key `MCP_PROOF_SIGNING_KEY`, default `CHANGE_ME_PROOF_HMAC`, so a later run still replaces the placeholder |
| Sign | `_sign_proof` | If the value is empty or equals `CHANGE_ME_PROOF_HMAC`, do not sign with it. If the variable is unset, keep the `MCP_GATEWAY_TOKEN` fallback |
| Gate | `scripts/rollout-gates.sh` | When `MCP_PROOF_SIGNING_KEY` is set and is not the placeholder, recompute the HMAC over the row without its `signature` field. A mismatch fails the gate. Keep the rate math |

### Rules

1. **One placeholder, one key.** `CHANGE_ME_PROOF_HMAC` is not a valid signing secret after `generate_secrets`.
2. **Do not share the password sed.** This key is not part of the `CHANGE_ME_SECURE_PASSWORD` global replace.
3. **Placeholder does not sign.** `_sign_proof` returns an empty signature for the placeholder and for an empty value.
4. **Unset still falls back.** A missing variable may still use `MCP_GATEWAY_TOKEN`, which is what the function does today.
5. **The gate checks the digest.** `rollout-gates.sh` fails when a row in the window has a signature that does not match the configured key.
6. **Rates stay.** `claimed_verify_rate` and `mismatch_rate` stay as Stage 14 defined them.
7. **The key stays off the card and out of logs.**

### Suggested contributor checklist

```text
1. Copy env.example to a temp file and run the generate_secrets substitutions
2. Confirm MCP_PROOF_SIGNING_KEY is no longer CHANGE_ME_PROOF_HMAC
3. Confirm POSTGRES_PASSWORD, REDIS_PASSWORD, MINIO_ROOT_PASSWORD, N8N_PASSWORD, KB_MINIO_SECRET_KEY, and PIHOLE_PASSWORD are untouched by this change
4. Call _sign_proof with the placeholder and confirm the signature is empty
5. Sign a dummy row with a generated key and confirm rollout-gates accepts that signature
6. Change one byte of the stored signature and confirm the gate exits non-zero
7. Confirm the rate lines still print claimed_verify_rate and mismatch_rate
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Key material | `openssl rand -hex 32` inside `generate_secrets` | A new secrets service |
| Algorithm | The HMAC-SHA256 already in `_sign_proof` | A second signature scheme |
| Check | `scripts/rollout-gates.sh` | A hosted audit product |
| Intake | GitHub Issue | An out-of-band key form |

### Acceptance check

- [ ] After `generate_secrets`, `MCP_PROOF_SIGNING_KEY` is not `CHANGE_ME_PROOF_HMAC`.
- [ ] The six `CHANGE_ME_SECURE_PASSWORD` keys are still owned by Stage 54.
- [ ] `_sign_proof` returns `""` when the key is the placeholder or empty.
- [ ] `_sign_proof` still signs with `MCP_GATEWAY_TOKEN` when `MCP_PROOF_SIGNING_KEY` is unset.
- [ ] `rollout-gates.sh` fails a windowed row whose `signature` does not match `MCP_PROOF_SIGNING_KEY`.
- [ ] `rollout-gates.sh` still enforces `MIN_CLAIMED_VERIFY_RATE` and `MAX_MISMATCH_RATE`.
- [ ] The proof key is not printed and `.env` was not created in the repo to prove this page.

### Rollback

Remove the new sed, the `set_env_var_if_default` line, the placeholder rejection, and the HMAC compare in the gate script. Signing and the rate gate behave as they do today.
