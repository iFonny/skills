# Index Format

The incremental index is per project:

```text
.agents/state/agent-updated-docs-index.json
```

It tracks processed transcript and git metadata only. It must not store transcript content, summaries, excerpts, diffs, absolute local paths, secrets, or private data.

## Schema

```json
{
  "version": 1,
  "updatedAt": "2026-04-29T14:00:00.000Z",
  "git": {
    "lastProcessedHead": "abc1234",
    "lastProcessedAt": "2026-04-29T14:00:00.000Z",
    "commitLimit": 30
  },
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
- `git.lastProcessedHead`: latest git head whose recent history was considered.
- `git.lastProcessedAt`: when git context was processed.
- `git.commitLimit`: fallback recent commit window when the previous head is unavailable.
- `transcripts`: map of stable, sanitized transcript IDs to metadata.
- `source`: short label such as `cursor`, `codex`, `claude`, or `other`.
- `displayPath`: optional relative or best-effort path for debugging only.
- `mtimeMs` and `size`: basic incremental change detection.
- `processedAt`: when this transcript was processed.
- `signalCount`: number of durable items extracted during the last pass. It is diagnostic only and must not decide whether a file changed.

## Stable Transcript IDs

Use keys shaped like:

```text
source:transcript-id
```

Never use absolute local paths, machine-local workspace slugs, usernames, or path-derived identifiers as keys. They are not portable across team members and can leak private machine details.

## Git Bounds

When git is available:

1. Inspect current working changes and staged changes.
2. Inspect commits since `git.lastProcessedHead`.
3. If `git.lastProcessedHead` is missing or unreachable, inspect only `git.commitLimit` recent commits.
4. After processing, update `git.lastProcessedHead` to the current head.

Do not store commit diffs, messages, or derived documentation candidates in the index.

## Cleanup And Retention

On each refresh:

- remove transcript entries whose source file no longer exists
- remove entries older than configured retention rules
- keep cleanup as bookkeeping, not as a documentation signal
- keep the index small enough to remain cheap to read

If project docs notes do not define retention, prefer conservative defaults such as 500 transcript entries or 180 days.

## Atomic Write

Write the index atomically:

1. Ensure `.agents/state/` exists.
2. Serialize the new JSON to a temporary file in the same directory.
3. Rename the temporary file over `agent-updated-docs-index.json`.
4. If writing fails, leave the previous index untouched.
