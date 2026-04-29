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
- If no meaningful updates exist, respond exactly:

```text
No high-signal memory updates.
```

- If documentation changes are made, also refresh the per-project index when transcript or git context was processed.
- If documentation does not change but transcripts were processed, refresh the index only.

## Workflow

1. Read the current project entry points and docs before proposing changes. Use `resources/source-discovery.md`.
2. Load project docs notes and source configuration if present. Use `resources/project-notes-lookup.md`.
3. Load `.agents/state/agent-updated-docs-index.json` if present. Use `resources/index-format.md`.
4. Process only new or changed transcript sources plus bounded git context.
5. Extract only durable, reusable signals:
   - recurring user preferences or corrections
   - stable workspace facts
   - repeatable workflows
   - architecture, tooling, or process facts that affect future agent behavior
6. For useful but uncertain, ambiguous, sensitive, or contradictory candidates, ask a structured question before using them in documentation.
7. Apply the merge and placement rules in `resources/merge-policy.md`.
8. Validate the result:
   - `AGENTS.md` links still resolve
   - no duplicate or contradictory rules were introduced
   - every `SKILL.md` stays under 500 lines
   - resource links stay one level deep
   - `.cursor/rules` was not edited directly
9. Refresh the index atomically after transcript or git processing.

## Safety Rules

- Never store transcript content, transcript summaries, excerpts, absolute local paths, secrets, or private data in the index.
- Never copy raw transcript fragments containing private paths, names, tokens, `.env` values, customer data, or sensitive ticket details into docs.
- Ask before using a noticed element when its meaning, scope, durability, sensitivity, or placement is uncertain.
- If a new signal contradicts existing documentation, stop in dry-run, cite the conflict, and ask for confirmation before changing it unless the existing documentation is clearly broken.
- Remove or replace stale documentation when newer repeated evidence shows an old convention is obsolete; do not only append new rules.

## Resources

- `resources/source-discovery.md`: documentation, transcript, and git sources to inspect.
- `resources/project-notes-lookup.md`: project-local documentation notes and source configuration lookup.
- `resources/index-format.md`: per-project index schema, retention, git bounds, and atomic writes.
- `resources/merge-policy.md`: signal filtering, confidence, placement, conflicts, dry-run, and validation.
