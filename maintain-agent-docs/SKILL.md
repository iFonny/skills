---
name: maintain-agent-docs
description: Maintains agent-facing documentation by mining current and recent conversations, transcripts, git context, and existing docs for durable reusable guidance. Use when updating AGENTS.md, agent rules, skills, workflows, project docs, or when the user asks to preserve recurring preferences, stable workspace facts, or documentation learnings.
---

# Maintain Agent Docs

## When To Use

Use this skill when:

- The user asks to update, add, audit, or refresh agent documentation.
- A session reveals durable preferences, repeated corrections, stable project facts, or reusable workflows.
- Documentation may be stale, duplicated, contradictory, or missing links.
- A project needs its incremental transcript index refreshed.

Do not use it to preserve one-off task details, secrets, private data, transient debugging context, ticket-specific facts, or speculative conclusions.

## Operating Mode

- Default to dry-run unless the user explicitly asks to update documentation or the active workflow clearly authorizes edits.
- In dry-run, provide proposed diffs grouped by target file, confidence, and evidence. Do not write documentation.
- Index updates are bookkeeping, not documentation edits. During dry-run, you may write the per-project index atomically to preserve transcript and git progress.
- When batching, keep candidates learned so far in transient run state
  (`.agents/state/agent-updated-docs-run-state.json`) so a later batch continues the same run.
  It is local and not a source of truth. See `resources/run-state.md`.
- After presenting the dry-run, end with a structured question (via the available question tool) listing each proposal as an option. Apply only the proposals the user selects. Skip the question entirely when there are no proposals — the "No high-signal memory updates." response is sufficient. The index refresh is bookkeeping, not a proposal: never put it in the question.
- Treat each invocation as a fresh complete pass for the current request. Do not reuse conclusions, proposals, or "no update" results from a previous `maintain-agent-docs` run unless the user explicitly asks for continuity; index cursors and sweeps are only processing bounds and bookkeeping aids.
- If you delegate work to subagents and the task requires reliability, judgment, or tradeoff analysis, use the same model as the parent agent.
- Persist after each batch (canonical rule referenced by the resources):
  - write the index and run-state atomically after each batch
  - before starting the next batch or presenting proposals, confirm that all items
    in the current batch were processed and persisted, and that any remaining
    discovered source is either queued for a later batch or explicitly skipped
    with a reason
  - continue automatically with the next batch while it fits safely in context
  - never present an incomplete sweep as complete
  - pause only when: a user decision is required (`baselineHead` absent or unreachable,
    full sweep vs bounded vs skip, ambiguous candidate); working-tree plus staged changes
    exceed the batch budget; context is too full for the next batch; an index or run-state
    write fails; writes are blocked by runtime mode or tool policy; or the run is
    complete and proposals need validation
  - when writes are blocked, ask what to do or abort
  - when a pause needs user input, use the question tool if available; otherwise tell the
    user they can reply `continue`
- If no meaningful updates exist, respond exactly:

```text
No high-signal memory updates.
```

- If documentation changes are made, also refresh the per-project index when transcript or git context was processed.
- If documentation does not change but transcripts were processed, refresh the index only.

## Workflow

1. Read the current project entry points and docs before proposing changes. Use `resources/source-discovery.md`.
2. Load project docs notes and source configuration if present. Use `resources/project-notes-lookup.md`.
3. Load `.agents/state/agent-updated-docs-index.json` if present. Use `resources/index-format.md`; for legacy v1 indexes, follow its `Legacy Indexes` section before processing any transcript or git batch.
3a. Load `.agents/state/agent-updated-docs-run-state.json` if present. Use `resources/run-state.md`. Merge new candidates into this run state and write it after each batch that changes candidates.
4. Process transcript sources before git:
   - Process transcripts new or changed since the index, newest first.
   - Process up to `transcriptBatchLimit` per batch.
   - Write each processed transcript entry incrementally.
   - At the batch limit, persist the batch and continue automatically if more remain and the next batch fits safely. Do not start or resume git until transcripts are up to date.
5. Run the git documentation impact check only after transcript processing is complete:
   - Always inspect the current working tree and staged changes before any commit sweep.
   - If working-tree plus staged changes exceed the batch budget, stop and ask the user to commit or otherwise shrink the changes. Large uncommitted changes are re-read on every invocation and block reliable progress.
   - If `git.sweep` exists, resume it before starting a new sweep.
   - If no `git.baselineHead` exists, or the stored baseline is unavailable or not an ancestor of the current target, ask a structured question before git processing. Offer full sweep, bounded recent window, or transcripts-only/skip-git. Never start a full history sweep without explicit user confirmation.
   - Guarantee exhaustive processing only for the selected scope; excluded older history is intentionally out of scope, not processed.
5a. Before concluding the impact check, enumerate the file list changed in the selected scope, plus working-tree and staged changes. Use the deterministic sweep range: `<baselineHead>..targetHead` for `since-baseline` sweeps, or the full reachable history for `full-history` sweeps. Map each path against the impact zones in `resources/merge-policy.md`. Commit messages and summaries are not a substitute for the file list.
5b. Read the actual diff for changed files before deciding whether documentation must change:
   - Use a deterministic commit list for sweeps, such as `git rev-list --topo-order <baselineHead>..targetHead` for `since-baseline` or `git rev-list --topo-order <targetHead>` for `full-history`, newest first.
   - Store sweep state with `targetHead`, `rangeKind`, optional `baselineHead`, `cursorCommit`, `order`, and `revListArgs`, then reconstruct the same list when resuming.
   - Run diff stats for the current batch range, plus working-tree and staged stats when those changes fit the budget.
   - For each changed file in the current batch, read the diff with `git diff <range> -- <path>` or the equivalent staged/working-tree command.
   - Skip files that clearly cannot affect any documented convention (lockfiles, generated output, asset bumps).
   - At ~300 KB or ~50 files of diff read, persist the batch and continue automatically when safe. Never fall back to file-name-only judgment for unread diffs.
6. Extract only durable, reusable signals from each completed transcript or git batch:
   - recurring user preferences or corrections
   - stable workspace facts
   - repeatable workflows
   - architecture, tooling, or process facts that affect future agent behavior
7. For useful but uncertain, ambiguous, sensitive, or contradictory candidates, ask a structured question before using them in documentation.
8. Apply the merge and placement rules in `resources/merge-policy.md`.
9. Validate the result:
   - `AGENTS.md` links still resolve
   - no duplicate or contradictory rules were introduced
   - every `SKILL.md` stays under 500 lines
   - resource links stay one level deep
   - `.cursor/rules` was not edited directly
10. Refresh the index atomically after each transcript or git batch, and after final transcript or git completion. A failed write must leave the previous index untouched.
11. Clear the transient run state only after the run is complete and its candidates have been proposed, accepted, rejected, or discarded as not meaningful.

## Safety Rules

- Never store transcript content, transcript summaries, excerpts, absolute local paths, secrets, or private data in the index.
- Keep run-state pragmatic but not reckless: it may contain richer temporary candidates than the index, but do not intentionally store obvious secrets, tokens, private keys, `.env` values, production dumps, or raw private logs.
- Never copy raw transcript fragments containing private paths, names, tokens, `.env` values, customer data, or sensitive ticket details into docs.
- If `skills-lock.json` marks a skill with `sourceType: "github"`, treat that skill as upstream-owned. Do not modify that skill or its resources directly; place project-specific overrides in project docs, rules, notes, or a separate project-owned skill instead.
- Ask before using a noticed element when its meaning, scope, durability, sensitivity, or placement is uncertain.
- If a new signal contradicts existing documentation, stop in dry-run, cite the conflict, and ask for confirmation before changing it unless the existing documentation is clearly broken.
- Remove or replace stale documentation when newer repeated evidence shows an old convention is obsolete; do not only append new rules.

## Resources

- `resources/source-discovery.md`: documentation, transcript, and git sources to inspect.
- `resources/project-notes-lookup.md`: project-local documentation notes and source configuration lookup.
- `resources/index-format.md`: per-project index schema, v1-to-v2 migration, retention, git bounds, and atomic writes.
- `resources/run-state.md`: transient local candidate state across transcript and git batches.
- `resources/merge-policy.md`: signal filtering, confidence, placement, conflicts, dry-run, and validation.
