# Index Migrations

Load this resource only when the compact index is not current, when the index
still contains the old per-transcript `transcripts` map, when a previous
migration is partial or failed, or when the user explicitly asks for migration
handling.

Do not run migration logic during normal up-to-date scans. The current schema
and normal write rules live in `index-format.md`.

When this resource applies, complete and persist the migration to current version
before any transcript or git processing. The only exception is a `When To Ask`
condition below: pause and ask the user first, then continue.

## Safety Rules

- Never store transcript content, transcript summaries, excerpts, diffs,
  absolute local paths, secrets, or private data.
- Never overwrite an unknown index version without asking the user.
- Treat a successful migration as bookkeeping only, not as evidence that older
  git history or transcript content was reviewed for documentation signals.
- If any required write fails, leave the previous files untouched and report a
  partial run.

## When To Ask

Ask the user before continuing when:

- the index version is unknown;
- the existing baseline is missing, unreachable, or not an ancestor of the
  target head;
- migration would intentionally skip older transcript or git scope;
- a previous migration left ambiguous partial files;
- the local registry cannot be written and precise older transcript detection is
  required for the selected run.

## v1 To v3

Version 1 indexes used legacy git fields such as `git.lastProcessedHead` and
`git.commitLimit`.

Migration steps:

1. Map `git.lastProcessedHead` to `git.baselineHead` only as a processing
   baseline.
2. Set `git.baselineMode` to `user-accepted-skip` unless the user confirms that
   the selected historical scope was fully processed.
3. Leave `git.lastFullyProcessedHead` unset unless the selected scope was
   actually processed exhaustively.
4. Replace `git.commitLimit` with `git.batchCommitLimit`.
5. Initialize `transcriptSources` from discovered sources with conservative
   watermarks. If source history was not exhaustively reviewed, record that as
   an intentional cutoff through counts and `olderChangeDetection`.
6. Create or rebuild the local transcript registry when available.
7. Remove legacy v1-only fields from the compact index.

## v2 To v3

Version 2 indexes stored one entry per transcript under `transcripts`.

Migration steps:

1. Group `transcripts` entries by `source`.
2. For each source, compute the newest `(mtimeMs, id)` pair as the source
   watermark.
3. Convert per-file counts into source-level diagnostics.
4. Move sanitized per-transcript metadata into
   `.agents/state/agent-updated-docs-transcripts.local.json`.
5. Set `olderChangeDetection` to `local-registry` when the registry write
   succeeds; otherwise use `recent-window-only` if the compact index still
   safely represents the selected batch.
6. Remove the `transcripts` map from the compact index.
7. Preserve valid git baseline and sweep fields, upgrading field names only
   when required by `index-format.md`.

## Atomic Migration Write

Migration writes affect both the compact index and the local registry. Use this
order:

1. Build the migrated compact index and local registry in memory.
2. Write the local registry to a temporary file if it is needed.
3. Write the compact index to a temporary file.
4. Rename the local registry temporary file into place.
5. Rename the compact index temporary file into place.
6. If any required step fails, keep the previous files and report a partial run.

Do not delete the old per-transcript information from the in-memory migration
result until the local registry write is confirmed or the user accepts the loss
of older precise transcript detection.

## Post-Migration Validation

After migration:

- the compact index has `version: 3`;
- the compact index has `transcriptSources` and no top-level `transcripts` map;
- the local registry exists when older transcript precision is required;
- `git.batchCommitLimit` is present when git metadata exists;
- skipped historical scope is represented as `user-accepted-skip`, not
  `fully-processed`;
- the next normal scan can proceed using `index-format.md` and
  `source-discovery.md` without loading this migration resource again.
