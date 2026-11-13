---
title: Knowledge Search Scan Contract
description: Constructional contract so kb_search walks the existing MinIO index iterator until it fills the requested match limit or the listing ends.
last_updated: 2026-11-11
owner: maintainers
status: proposed
theme: knowledge
impact: high
---

# Knowledge Search Scan Contract

## Problem

`kb_search` asks `KnowledgeBaseApi.list_objects` for 500 keys and then searches only that list. The tool schema's `limit` is the number of matches to return (1 to 100, default 20). It is not a cursor, and the handler never reads a start key.

`_list_objects_sync` appends names from the MinIO iterator and breaks when `len(rows) >= limit`. `kb_list_objects` uses that helper with a schema maximum of 200. Its result object has `bucket`, `prefix`, `count`, and `keys`. There is no field for the next key.

Checked on 2026-11-11 with Python 3.12.3 by replaying that break on 600 synthetic index rows. MinIO was not contacted. The gateway was not started.

| Call | What the replay kept | What it dropped |
|---|---|---|
| Search `welcome`, match limit 20 | `.index/0000.json`, whose excerpt is `welcome packet` | keys 500 through 599 were never scanned |
| Search `community-minutes`, match limit 20 | no matches | `.index/0500.json` and `.index/0599.json`, both of which contain that phrase |
| `kb_list_objects` limit 200 | the first 200 names | the other 400 names, and no continuation key |

A shared knowledge bucket that grows past one page looks searchable. Queries that only occur in later index objects return `count: 0`. The match limit of 20 never runs, because the scan stopped first.

`kb_index_document` already writes one JSON object per source key under `KB_INDEX_PREFIX` (default `.index/`). `kb_search` already builds a haystack from `title`, `sourceKey`, `excerpt`, and `tags`, and keeps a row when the query is a substring. Those parts are the search. The cap is the listing.

Stage 41 (open PR #67) owns `list_objects_v2` pagination in the document watcher. This page is the gateway helper. Stage 70 (open PR #96) owns `KB_MAX_OBJECT_BYTES` on reads. This page does not change the byte cap. A search that opens an index object still goes through `get_object_bytes`.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `KnowledgeBaseApi._list_objects_sync` | The iterator walk that breaks at `limit` |
| `kb_search` | Substring search over index JSON |
| `kb_list_objects` | The paged name list operators and tools already call |
| `kb_index_document` | Writes the JSON rows this search reads |
| `KB_INDEX_PREFIX` | The prefix both tools already pass |
| Stage 41 watcher pagination (open PR #67) | Processor listing; leave it |
| Stage 70 read cap (open PR #96) | Object byte ceiling; leave it |

Out of scope:

- A second search engine, vector index, or ranking service
- Changing the haystack fields or the substring test
- The document watcher's `ContinuationToken` loop
- Raising `KB_MAX_OBJECT_BYTES`

## Proposal

Keep the iterator and the substring test. Search until `limit` matches are collected or the iterator ends. When a caller still wants a bounded page, return the last key visited so the next call can skip names at or before that key.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Search scan | `kb_search` | Walk the existing iterator until `limit` matches are kept or the listing ends. Do not pass a fixed 500 |
| Search result | `kb_search` | Add `truncated: false` when the iterator ended. When a scan stops early, `truncated: true` and `nextKey` set to the last name visited |
| List page | `kb_list_objects` | Keep the schema maximum of 200 as the page size. When the page is full and more names exist, return `nextKey` as that last name |
| List helper | `_list_objects_sync` | Accept the previous `nextKey` and skip names up through that key before counting the page |
| Index writer | `kb_index_document` | Keep one JSON object per source key under the current prefix |
| Read size | `get_object_bytes` | Leave the Stage 70 cap in place |

### Rules

1. **The match limit is the result size.** A search for 20 matches may read more than 20 index objects. It may not stop at 500 objects while matches remain later in the listing.
2. **The end of the iterator is the end of the scan.** `truncated` is false only when every later name was considered.
3. **A stopped page names the last key.** The next call passes that key and skips names up through it. Listing order stays the order the iterator already yields.
4. **List and search share the helper.** `kb_list_objects` does not grow a private walker.
5. **The index document stays the same.** Title, source key, excerpt, and tags remain the haystack.
6. **The watcher stays on Stage 41.** This page does not edit `universal_auto_processor.py`.

### Suggested contributor checklist

```text
1. Build 600 index rows and put the only match at key 501
2. Confirm today's scan of 500 returns count 0 for that match
3. Walk until the match limit or the iterator ends and confirm the row is returned
4. Confirm a full list page of 200 includes nextKey and the following page starts after it
5. Confirm a scan that reaches the last name sets truncated false and omits nextKey
6. Confirm get_object_bytes is still the reader, with the existing byte cap
7. Confirm MinIO and the gateway were not started
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Object names | The MinIO iterator `list_objects` already uses | A new inventory service |
| Match text | The JSON `kb_index_document` already stores | A second index format |
| Next page | The last key already in that listing | A cursor database |
| Intake | GitHub Issue | A search-service tracker |

### Acceptance check

- [ ] A match whose index key sorts after the first 500 names is returned by `kb_search` when no earlier row matches.
- [ ] `kb_search` stops at the schema `limit` of matches, and sets `truncated` false when the iterator ended.
- [ ] A search that stops before the iterator ends returns `truncated: true` and `nextKey` equal to the last name visited.
- [ ] `kb_list_objects` still returns at most 200 keys and includes `nextKey` when further names exist.
- [ ] A follow-up list or search that passes that `nextKey` does not return names at or before it.
- [ ] `kb_index_document` still writes one JSON object per source key, and reads still use `get_object_bytes`.
- [ ] MinIO and the gateway were not started to prove this page.

### Rollback

Pass 500 into `list_objects` from `kb_search`, and return only `bucket`, `prefix`, `count`, and `keys` from the list tool. Matches past the first 500 index objects disappear again, which is today's behavior.
