---
title: Gzip Tar Suffix Contract
description: Constructional contract so _process_archive opens a .tar.gz object with the tarfile branch it already uses for .tar and .tgz.
last_updated: 2026-11-09
owner: maintainers
status: proposed
theme: documents
impact: high
---

# Gzip Tar Suffix Contract

## Problem

`SUPPORTED_EXTENSIONS` maps `.tar.gz` to `archive`. The watcher queues that object because `_get_file_type` matches the longest suffix with `str.endswith`. `_process_archive` then decides the format with `Path.suffix`.

Checked on 2026-11-09 with Python 3.12.3. A gzip tar was built in memory. MinIO and Tika were not contacted. The processor container was not started.

| Name | `Path.suffix` | `Path.suffixes` | Outer tar test (`suffix in ['.tar', '.tar.gz', '.tgz']`) | `endswith('.tar.gz')` |
|---|---|---|---|---|
| `bundle.tar.gz` | `.gz` | `.tar`, `.gz` | False | True |
| `bundle.tgz` | `.tgz` | `.tgz` | True | False |
| `bundle.tar` | `.tar` | `.tar` | True | False |

`.tar` and `.tgz` enter the existing `tarfile` branch. `.tar.gz` enters neither the zip branch nor the tar branch. `extracted_files` stays empty, and the function still returns `status: success` with this text:

```text
Archive: bundle.tar.gz
Extracted 0 files:
```

The same bytes open with `tarfile.open(..., mode="r:gz")`. The member list is `notes.txt`.

`_get_file_type("bundle.tar.gz")` follows the supported-extension loop, so the watcher does treat the object as work. `process_file` then stores the empty success text. The output key uses `Path.stem`, which for this name is `bundle.tar`, so the stored object is `bundle.tar.txt`. The next pass sees that object and skips the archive. The member text is never written.

Stage 41 (open PR #67) owns paging `list_objects_v2`. Stage 52 (open PR #78) owns `notes.pdf` and `notes.md` sharing one `.txt` key. Stage 69 (open PR #95) owns a source `.txt` being classified as skip. Those contracts stay. This page is the `.tar.gz` branch test.

## Existing scan (reuse first)

| Asset | Role |
|---|---|
| `SUPPORTED_EXTENSIONS['.tar.gz']` | Already `archive` |
| `_get_file_type` | Already matches `.tar.gz` by longest `endswith` |
| `_process_archive` zip and tar branches | The readers to keep |
| `tarfile` | The library that already opens `.tar` and `.tgz` |
| Stage 41 `processor-object-list-pagination-contract.md` (open PR #67) | Object listing. Unchanged |
| Stage 52 output-key page (open PR #78) | Shared stem for pdf and markdown. Unchanged |
| Stage 69 `source-text-file-contract.md` (open PR #95) | Source `.txt`. Unchanged |

Out of scope:

- A new archive service or a second extractor binary
- Changing the byte and member caps already specified for this processor
- Renaming `.tgz` or `.tar` handling
- Collapsing every output stem onto one key

## Proposal

Use the same filename test `_get_file_type` already uses, then keep the `tarfile` branch.

### Contribution lanes

| Lane | Where | Suggested change |
|---|---|---|
| Select | `_process_archive` | Treat a name that `endswith('.tar.gz')` like `.tgz` and `.tar` |
| Open | The existing `tarfile.open` call | Read gzip for `.tar.gz` and `.tgz`. Plain `.tar` stays a tar |
| Report | The success return | `files_count` counts members actually read |
| Queue | Watcher and `_get_file_type` | Leave the existing `archive` classification |

### Rules

1. **One suffix test.** `.tar.gz` is an archive when the supported-extension map says it is.
2. **`Path.suffix` is the last dot.** It is `.gz` for `bundle.tar.gz`. The branch test has to see the whole `.tar.gz` ending, which `endswith` or `Path.suffixes` already can.
3. **Empty is not success.** If the tar branch does not run, the function must not return `status: success` with zero members for a `.tar.gz` payload.
4. **`.tar` and `.tgz` stay on the current branch.** Both already match `Path.suffix`.
5. **Members stay in the existing text block.** `notes.txt` inside the sample archive appears in the extracted text.
6. **Caps and output keys stay with their owners.** Stage 41, Stage 52, and Stage 69 pages stay as written.
7. **No new package.** `tarfile` is already imported.

### Suggested contributor checklist

```text
1. Build a gzip tar in memory with one member, notes.txt, whose body is a short sentence
2. Call _process_archive("bundle.tar.gz") and confirm status is success and files_count is 1
3. Confirm the returned text contains notes.txt and the sentence
4. Call _process_archive("bundle.tgz") and _process_archive("bundle.tar") and confirm each still opens
5. Call _process_archive("bundle.zip") and confirm the zip branch still lists its members
6. Confirm a second pass sees the stored output and does not claim the archive had zero files
7. Confirm .txt source handling and pdf/md stem handling are still the Stage 69 and Stage 52 contracts
```

### Framework reuse

| Need | Prefer | Avoid |
|---|---|---|
| Format choice | The `endswith` loop already in `_get_file_type` | A MIME database service |
| Reader | `tarfile` and `zipfile` in this module | A new unpacking container |
| Intake | GitHub Issue | An out-of-band upload form |

### Acceptance check

- [ ] `_process_archive` on `bundle.tar.gz` returns `files_count` 1 for a one-member gzip tar.
- [ ] The returned text includes the member name `notes.txt`.
- [ ] `.tgz` and `.tar` still open through `tarfile`.
- [ ] `.zip` still opens through `zipfile`.
- [ ] A `.tar.gz` object is not stored as a success record that says `Extracted 0 files`.
- [ ] Stage 41, Stage 52, and Stage 69 contracts are unchanged.
- [ ] MinIO was not contacted to prove this page.

### Rollback

Restore the `Path.suffix` tests in `_process_archive`. `.tar` and `.tgz` behave as they do today. `.tar.gz` again returns a success body with zero members.
