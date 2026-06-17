# Transient Run State

Use transient run state to carry documentation candidates and minimal batch
state across persistent batches.

```text
.agents/state/agent-updated-docs-run-state.json
```

This file is local, temporary, and not a source of truth. It exists because the
compact index tracks durable progress only and must not store candidate content.
Run state keeps temporary candidates and the last batch boundary state.

## Purpose

Use run state when transcript or git processing spans multiple batches:

- preserve candidates already learned in earlier batches
- keep enough context to merge, deduplicate, and present proposals after later batches
- avoid re-reading already processed sources only to recover candidate reasoning
- record that the current batch was processed and persisted before the next batch starts

The run state complements the index:

- the compact index and local registry store durable progress and cursors
- the run state stores temporary candidate working notes and batch boundary state

## Suggested Shape

```json
{
  "version": 1,
  "runId": "2026-06-17T09:20:00.000Z",
  "updatedAt": "2026-06-17T09:35:00.000Z",
  "status": "in-progress",
  "batch": {
    "kind": "transcripts",
    "source": "cursor",
    "processedInBatch": 50,
    "remainingAfterBatch": 12,
    "persistedAt": "2026-06-17T09:35:00.000Z",
    "status": "persisted"
  },
  "candidates": [
    {
      "id": "candidate-1",
      "kind": "workflow",
      "confidence": "medium",
      "target": ".agents/skills/example/SKILL.md",
      "summary": "Prefer structured questions over plain-text continue prompts for resumable batches.",
      "evidenceRefs": ["transcript:cursor:abc", "git:def123"],
      "status": "pending"
    }
  ]
}
```

## Field Rules

- `version`: run-state schema version.
- `runId`: stable ID for the current documentation maintenance run.
- `updatedAt`: last write time.
- `status`: `in-progress`, `proposed`, or `complete`.
- `batch`: latest multi-batch boundary state. Keep it minimal.
- `batch.kind`: `transcripts` or `git`.
- `batch.source`: transcript source for transcript batches; omit for git batches.
- `batch.processedInBatch`: number of files or commits processed in the batch.
- `batch.remainingAfterBatch`: number of files or commits known to remain after the batch.
- `batch.persistedAt`: when the compact index, local registry if needed, and run-state were written for the batch.
- `batch.status`: `persisted` when the batch was fully processed and persisted; use `partial` when persistence failed or scope remains unsafe.
- `candidates`: temporary documentation candidates accumulated across batches.
- `candidate.id`: stable within the run state.
- `candidate.kind`: preference, workflow, architecture, tooling, placement, stale-doc, or other short label.
- `candidate.confidence`: low, medium, or high.
- `candidate.target`: proposed target documentation path when known. Keep it relative.
- `candidate.summary`: working summary of the candidate.
- `candidate.evidenceRefs`: sanitized references only, such as stable transcript IDs or commit SHAs.
- `candidate.status`: pending, proposed, accepted, rejected, or discarded.

## Safety Rules

Run state is less strict than the index because it is local and temporary, but it is not unrestricted scratch space.

Do not intentionally store:

- obvious secrets, tokens, passwords, private keys, certificates, or `.env` values
- production dumps, raw private logs, crash dumps, or customer data
- large raw transcript chunks or large raw diffs
- absolute local paths when a relative path or stable source ID is enough

It may store richer candidate summaries and short rationale than the index when
needed to preserve reasoning across batches.

Batch state should stay small. Do not store file lists, transcript excerpts,
diffs, raw logs, or absolute paths in `batch`.

## Write And Cleanup

- Write run state atomically after every batch in a multi-batch run, even when no new candidate was found.
- Merge new candidates into existing run state; deduplicate by target, summary, and evidence references.
- For multi-batch runs, do not keep candidates or batch state only in model context. Persist them to run-state after each batch.
- Keep candidates until the run is complete and the user has accepted, rejected, or discarded them.
- Delete or clear the file after completion. If deletion fails, mark `status` as `complete` and remove candidate details.
- Never treat run state as the durable source of progress; use the compact index, local registry, and git sweep cursor for that.

