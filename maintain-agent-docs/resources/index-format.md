# Index Format

The compact incremental index is per project:

```text
.agents/state/agent-updated-docs-index.json
```

It tracks progress metadata only. It must not store transcript content, transcript
summaries, excerpts, diffs, absolute local paths, secrets, private data, or
documentation candidates.

Documentation candidates and multi-batch batch state belong in
`agent-updated-docs-run-state.json`; see `run-state.md`.

If the loaded index is not current, or if it still contains the old
per-transcript `transcripts` map, load `index-migrations.md` before processing
any transcript or git batch.

## Current Schema

```json
{
  "version": 3,
  "updatedAt": "2026-06-17T12:00:00.000Z",
  "transcriptBatchLimit": 50,
  "transcriptSources": {
    "cursor": {
      "lastScanAt": "2026-06-17T12:00:00.000Z",
      "watermarkMtimeMs": 1770000000000,
      "watermarkId": "cursor:abc",
      "recentWindowCount": 100,
      "processedCount": 114,
      "changedCount": 3,
      "skippedCount": 1,
      "olderChangeDetection": "local-registry"
    },
    "codex": {
      "lastScanAt": "2026-06-17T12:00:00.000Z",
      "watermarkMtimeMs": 1770000000000,
      "watermarkId": "codex:abc",
      "recentWindowCount": 100,
      "processedCount": 18,
      "changedCount": 1,
      "skippedCount": 4,
      "olderChangeDetection": "recent-window-only"
    }
  },
  "git": {
    "baselineHead": "abc1234",
    "baselineMode": "fully-processed",
    "lastFullyProcessedHead": "abc1234",
    "lastProcessedAt": "2026-06-17T12:00:00.000Z",
    "batchCommitLimit": 50,
    "sweep": {
      "targetHead": "def5678",
      "rangeKind": "since-baseline",
      "baselineHead": "abc1234",
      "cursorCommit": "fedcba9",
      "order": "newest-first",
      "revListArgs": ["--topo-order"],
      "pathspec": "front/",
      "startedAt": "2026-06-17T12:00:00.000Z",
      "updatedAt": "2026-06-17T12:10:00.000Z"
    }
  }
}
```

## Field Rules

- `version`: current schema version. Unknown or older versions require
  `index-migrations.md` before processing.
- `updatedAt`: when the compact index was last written.
- `transcriptBatchLimit`: maximum number of transcript files to inspect in one
  batch. Prefer `50` unless project docs notes configure a different value.
- `transcriptSources`: source-level transcript progress. It stores watermarks
  and counts, not per-file transcript entries.
- `transcriptSources.<source>.lastScanAt`: last completed scan time for that
  source.
- `transcriptSources.<source>.watermarkMtimeMs`: newest processed transcript
  modification time for that source.
- `transcriptSources.<source>.watermarkId`: stable transcript id paired with the
  watermark for deterministic tie-breaking.
- `transcriptSources.<source>.recentWindowCount`: safety window size that is
  always rechecked for the source.
- `transcriptSources.<source>.processedCount`, `changedCount`, `skippedCount`:
  diagnostic counts from the last completed scan.
- `transcriptSources.<source>.olderChangeDetection`: whether older modified
  transcripts can be detected from the local registry. Valid values are
  `local-registry`, `recent-window-only`, and `unavailable`.
- `git.baselineHead`: git head used as the lower bound for future incremental
  runs.
- `git.baselineMode`: whether the baseline is fully processed or user-selected
  as an intentional cutoff. Valid values are `fully-processed` and
  `user-accepted-skip`.
- `git.lastFullyProcessedHead`: latest git head whose selected git scope was
  processed exhaustively. Do not set this for a user-selected skip.
- `git.lastProcessedAt`: when git context was processed.
- `git.batchCommitLimit`: maximum number of commits to inspect in one batch.
  This is a batch size, not a fallback history cap.
- `git.sweep`: resumable git sweep state. It exists only while a sweep is
  incomplete.
- `git.sweep.targetHead`: head selected when the sweep started.
- `git.sweep.rangeKind`: selected sweep range. Use `since-baseline` for
  incremental or bounded sweeps and `full-history` for a user-confirmed full
  sweep.
- `git.sweep.baselineHead`: baseline used to build a `since-baseline` sweep
  range. Omit it for `full-history`.
- `git.sweep.cursorCommit`: most recent processed commit in the deterministic
  commit list. Resume after this commit in the reconstructed list.
- `git.sweep.order`: processing order. Use `newest-first`.
- `git.sweep.revListArgs`: exact arguments needed to reconstruct the same
  commit list, such as `["--topo-order"]`.
- `git.sweep.pathspec`: git pathspec relative to the repository root for the
  selected project scope.

## Local Transcript Registry

Precise per-transcript bookkeeping belongs in a local-only file:

```text
.agents/state/agent-updated-docs-transcripts.local.json
```

Projects should ignore this file, as they do for run-state. It is optional
bookkeeping: if it is missing, the compact index still supports watermark plus
recent-window scanning.

Suggested shape:

```json
{
  "version": 1,
  "updatedAt": "2026-06-17T12:00:00.000Z",
  "sources": {
    "cursor": {
      "cursor:abc": {
        "displayPath": "agent-transcripts/abc/abc.jsonl",
        "mtimeMs": 1770000000000,
        "size": 12345,
        "processedAt": "2026-06-17T12:00:00.000Z"
      }
    }
  }
}
```

Registry rules:

- Use stable ids shaped like `source:transcript-id`.
- Keep `displayPath` relative or best-effort only for debugging.
- Never store absolute local paths, transcript content, summaries, excerpts,
  secrets, or private data.
- If the registry write fails and older-file precision is needed for the current
  batch, stop with a partial run. If the compact index still represents the
  processed batch safely, record `olderChangeDetection` as `recent-window-only`.

## Normal Atomic Write

For non-migration batches:

1. Ensure `.agents/state/` exists.
2. Write the local transcript registry to a temporary file when it changed.
3. Write the compact index to a temporary file.
4. Rename temporary files over their targets.
5. If a required write fails, leave the previous files untouched and report a
   partial run.

Index and registry writes are allowed during dry-run because they are
bookkeeping. They must not contain documentation proposal content.
