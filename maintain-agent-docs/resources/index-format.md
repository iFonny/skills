# Index Format

The incremental index is per project:

```text
.agents/state/agent-updated-docs-index.json
```

It tracks processed transcript and git metadata only. It must not store transcript content, summaries, excerpts, diffs, absolute local paths, secrets, or private data.

Documentation candidates learned during a multi-batch run belong in the transient run-state file described in `run-state.md`, not in this index.

## Schema

```json
{
  "version": 2,
  "updatedAt": "2026-04-29T14:00:00.000Z",
  "git": {
    "baselineHead": "abc1234",
    "baselineMode": "fully-processed",
    "lastFullyProcessedHead": "abc1234",
    "lastProcessedAt": "2026-04-29T14:00:00.000Z",
    "batchCommitLimit": 50,
    "sweep": {
      "targetHead": "def5678",
      "rangeKind": "since-baseline",
      "baselineHead": "abc1234",
      "cursorCommit": "fedcba9",
      "order": "newest-first",
      "revListArgs": ["--topo-order"],
      "startedAt": "2026-04-29T14:00:00.000Z",
      "updatedAt": "2026-04-29T14:10:00.000Z"
    }
  },
  "transcriptBatchLimit": 50,
  "transcripts": {
    "cursor:transcript-id": {
      "source": "cursor",
      "displayPath": "agent-transcripts/transcript-id.jsonl",
      "mtimeMs": 1777471200000,
      "size": 12345,
      "processedAt": "2026-04-29T14:00:00.000Z",
      "signalCount": 2
    }
  }
}
```

## Field Rules

- `version`: schema version. For unknown versions, do not overwrite blindly; rebuild only if safe or ask the user.
- `updatedAt`: when the index was last written.
- `git.baselineHead`: git head used as the lower bound for future incremental runs.
- `git.baselineMode`: whether the baseline is fully processed or user-selected as an intentional cutoff. Valid values are `fully-processed` and `user-accepted-skip`.
- `git.lastFullyProcessedHead`: latest git head whose selected git scope was actually processed exhaustively. Do not set this for a user-selected skip.
- `git.lastProcessedAt`: when git context was processed.
- `git.batchCommitLimit`: maximum number of commits to inspect in one git batch. This is a batch size, not a fallback history cap.
- `git.sweep`: resumable git sweep state. It exists only while a sweep is incomplete.
- `git.sweep.targetHead`: head selected when the sweep started.
- `git.sweep.rangeKind`: selected sweep range. Use `since-baseline` for incremental or bounded sweeps and `full-history` for a user-confirmed full sweep.
- `git.sweep.baselineHead`: baseline used to build a `since-baseline` sweep range. Omit it for `full-history`.
- `git.sweep.cursorCommit`: most recent processed commit (resume point) in the deterministic commit list. Resume after this commit in the reconstructed list.
- `git.sweep.order`: processing order. Use `newest-first`.
- `git.sweep.revListArgs`: exact arguments needed to reconstruct the same commit list, such as `["--topo-order"]`.
- `transcriptBatchLimit`: maximum number of new or changed transcripts to process in one batch. Prefer 50 unless project docs notes configure a different value.
- `transcripts`: map of stable, sanitized transcript IDs to metadata.
- `source`: short label such as `cursor`, `codex`, `claude`, or `other`.
- `displayPath`: optional relative or best-effort path for debugging only.
- `mtimeMs` and `size`: basic incremental change detection.
- `processedAt`: when this transcript was processed.
- `signalCount`: number of durable items extracted during the last pass. It is diagnostic only and must not decide whether a file changed.

## Legacy Indexes

Version 1 indexes used `git.lastProcessedHead` and `git.commitLimit`. Resolve or migrate a version 1 index immediately after loading it, before processing any transcript or git batch. Do not treat a version 1 `lastProcessedHead` as proof that older history was fully processed. When upgrading safely:

- map `lastProcessedHead` to `git.baselineHead` only as a processing baseline
- set `git.baselineMode` to `user-accepted-skip` unless the user confirms a full historical sweep
- leave `git.lastFullyProcessedHead` unset unless the selected scope was actually processed exhaustively
- replace `commitLimit` with `batchCommitLimit`

If the migration choice is unclear, ask the user before processing any transcript or git batch.

## Stable Transcript IDs

Use keys shaped like:

```text
source:transcript-id
```

Never use absolute local paths, machine-local workspace slugs, usernames, or path-derived identifiers as keys. They are not portable across team members and can leak private machine details.

## Git Bounds

When git is available:

1. Inspect current working and staged changes.
2. If `git.sweep` exists, resume it before starting a new sweep.
3. If `git.baselineHead` is present and reachable, inspect commits in the selected scope since that baseline.
4. If `git.baselineHead` is missing, unavailable, or not an ancestor of the target head, resolve the scope with the user before processing (see `SKILL.md`). Never silently fall back to a fixed recent commit count.
5. Build a deterministic sweep list, for example with `git rev-list --topo-order <baselineHead>..targetHead` for `since-baseline`, or `git rev-list --topo-order <targetHead>` for a user-confirmed `full-history` sweep, newest first.
6. Process at most `git.batchCommitLimit` commits and the configured diff budget in one batch.
7. If the batch ends before the sweep is complete, persist `git.sweep` and run-state, then continue automatically when safe. The presence of `git.sweep` means only that the sweep is unfinished, not that a pause is required.
8. After a fully processed sweep, set `git.baselineHead` to the target head, set `git.baselineMode` to `fully-processed`, set `git.lastFullyProcessedHead` to the target head, remove `git.sweep`, and write the index atomically.
9. If the user selects a bounded first run or skip-git, set `git.baselineHead` and `git.baselineMode` to `user-accepted-skip` for the intentionally excluded older history. Do not set `git.lastFullyProcessedHead` for skipped history.

Do not store commit diffs, messages, patches, file contents, transcript summaries, or derived documentation candidates in the index.

Use `.agents/state/agent-updated-docs-run-state.json` for temporary candidate working state across batches. The index remains the only source of truth for progress.

## Cleanup And Retention

On each refresh:

- remove transcript entries whose source file no longer exists
- remove entries older than configured retention rules
- keep cleanup as bookkeeping, not as a documentation signal
- keep the index small enough to remain cheap to read

If project docs notes do not define retention, prefer conservative defaults such as 500 transcript entries or 180 days.

## Atomic Write

Write the index atomically after each transcript or git batch:

1. Ensure `.agents/state/` exists.
2. Serialize the new JSON to a temporary file in the same directory.
3. Rename the temporary file over `agent-updated-docs-index.json`.
4. If writing fails, leave the previous index untouched.

Atomic index writes are allowed during dry-run because they are bookkeeping. They must not contain documentation proposal content; proposal working state belongs in the transient run-state file.
