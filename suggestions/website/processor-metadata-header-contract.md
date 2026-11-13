---
title: Processor Metadata Header Contract
description: Constructional contract so a long archive file listing stays valid JSON, fits the object metadata budget, and never blocks the processed text upload.
last_updated: 2026-11-13
owner: maintainers
status: proposed
theme: reliability
impact: high
---

# Processor Metadata Header Contract

## Problem

`store_processed_content` in `tika-minio-processor/universal_processor.py` copies processing metadata onto the MinIO object:

```text
metadata_json = json.dumps(metadata, ensure_ascii=True)
if len(metadata_json) > 1500:
    metadata_json = metadata_json[:1500] + "...(truncated)"
s3_metadata["extracted-metadata"] = quote(metadata_json, safe="")
put_object(..., Metadata=s3_metadata)
```

The slice is in the middle of a JSON string. The stored header is not JSON, and the suffix makes the value longer than the 1500-character cap. `quote()` then percent-encodes braces, quotes, and commas, so the header grows again. The text body and this header are one `put_object`. A header the store rejects takes the extracted text with it.

`_process_archive` already builds this metadata. It keeps up to 50 extracted file rows (`name` and `size`) on the result that `process_file_from_minio` passes into `store_processed_content`.

Checked on 2026-11-13 with Python 3.12.3 by running that slice and `quote()` on the archive shape. Tika was not contacted. MinIO was not contacted. The watcher was not started.

| Step | Result for 50 rows named `community/minutes-000.txt` through `community/minutes-049.txt` |
|---|---|
| `json.dumps` | 2714 characters |
| Slice at 1500 plus `...(truncated)` | 1514 characters |
| `json.loads` of the sliced value | `JSONDecodeError: Unterminated string starting at: line 1 column 1497 (char 1496)` |
| `quote(sliced, safe="")` | 2480 bytes |

The function comment says the 1500-character cap is there so the value stays under the header limit after quoting. The quoted value alone is 2480 bytes. That is already over a 2048-byte user-metadata budget, before the other headers this put sends (`Content-Type`, `processed-by`, `original-filename`, `processing-timestamp`, `processing-date`, `processing-method`).

A community archive of meeting notes therefore stores either an unreadable metadata header or no text object at all. The text body was already extracted.

Stage 52 (open PR #78) owns the processed object key, including keeping parent directories and avoiding a shared `notes.txt` for `notes.pdf` and `notes.docx`. Stage 73 (open PR #99) owns selecting `.tar.gz` the way `_get_file_type` already does. Stage 41 (open PR #67) owns watcher pagination and the archive expansion budget. This page leaves those as they are.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `store_processed_content` | The put that already writes the `.txt` body |
| `json.dumps(..., ensure_ascii=True)` | The serializer this header already uses |
| `quote(..., safe="")` | The encoder S3 metadata already requires |
| `_process_archive` metadata | The in-tree producer of a 50-row file list |
| `extracted_files[:50]` | The cap on that list; leave the cap |
| Stage 52 object-key page (PR #78) | Where the `.txt` object is named; leave it |
| Stage 73 `.tar.gz` page (PR #99) | Which archive branch runs; leave it |

Out of scope:

- A metadata database or a sidecar search index
- Raising the 50-file listing cap
- Changing the object key or the `.tar.gz` reader
- Starting MinIO, Tika, or the watcher to prove this page

## Proposal

Keep the text put and the existing metadata keys. Shrink the object by dropping list rows or keys until `json.dumps` is still valid JSON and `quote()` of that JSON fits beside the other headers. When it still does not fit, omit `extracted-metadata` and put the text body anyway.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Valid JSON | `store_processed_content` | Drop `extracted_files` rows from the end until the dumps is at or under the budget. The value must `json.loads` |
| Quoted size | same function | Call `quote()` before accepting the header. Keep the quoted value small enough that this header plus the other keys on the same put stay within 2048 bytes |
| Overflow | same function | Omit `extracted-metadata` and still `put_object` the text body |
| Archive listing | `_process_archive` | Leave `extracted_files[:50]` as the source list |
| Object key | `store_processed_content` | Leave the Stage 52 key |
| Archive bytes | processor caps | Leave the Stage 41 budget and the Stage 73 suffix branch |

### Rules

1. **Do not slice a JSON string.** A shortened header still parses with `json.loads`.
2. **Quote before the size check.** The budget applies to the percent-encoded value, which is what MinIO receives.
3. **2048 bytes is the user-metadata budget.** The quoted `extracted-metadata` value plus the other header keys and values on this put stay within 2048 bytes.
4. **The text body wins.** If the metadata cannot fit, the `.txt` object is still stored and the header is absent.
5. **Fifty names stay the producer cap.** This page does not raise `extracted_files[:50]`.
6. **Keys and archive type stay on their pages.** Stage 52 and Stage 73 keep object names and `.tar.gz` selection.

### Suggested contributor checklist

```text
1. Build the 50-row archive metadata used above and confirm today's slice does not json.loads
2. Confirm today's quote() of that slice is 2480 bytes
3. After the change, confirm the stored header json.loads to an object
4. Confirm quote() of the stored header, plus the other put headers, is within 2048 bytes
5. Force a metadata object that cannot fit and confirm put_object still receives the text body and omits extracted-metadata
6. Confirm a small metadata object is stored unchanged
7. Confirm MinIO, Tika, and the watcher were not started
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Header value | `json.dumps` and `quote` already imported here | A metadata service |
| Too many file rows | Drop rows from `extracted_files` until the quoted value fits | Cutting the JSON text at character 1500 |
| Header rejected by size | Omit that one key and put the text | Failing the whole object |
| Intake | GitHub Issue | A separate document-pipeline tracker |

### Acceptance check

- [ ] The 50-row archive metadata is stored as JSON that `json.loads` accepts, or the header is omitted.
- [ ] The stored header is not produced by slicing at character 1500.
- [ ] `quote()` of the stored header, together with the other metadata keys on the same put, is within 2048 bytes.
- [ ] A metadata object that cannot fit does not remove the text body from `put_object`.
- [ ] A metadata object under the budget is stored as the full JSON object.
- [ ] `extracted_files[:50]`, the Stage 52 object key, and the Stage 73 `.tar.gz` branch stay as they are.
- [ ] MinIO, Tika, and the watcher were not started to prove this page.

### Rollback

Restore the 1500-character slice and the `...(truncated)` suffix. A 50-row archive listing again becomes invalid JSON whose quoted form is 2480 bytes, which is today's behavior.
