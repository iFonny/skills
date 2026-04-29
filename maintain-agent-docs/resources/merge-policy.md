# Merge Policy

Use this policy to decide what becomes documentation, where it belongs, and when to ask before changing anything.

## Signal Filter

Keep only durable, reusable items:

- recurring user preferences or corrections
- stable workspace facts
- repeatable workflows
- architecture, tooling, or process facts that affect future agent behavior
- documentation structure or routing rules that prevent future mistakes

Reject:

- secrets or private data
- one-off instructions
- transient details
- temporary debugging context
- ticket-specific or customer-specific details
- speculative guesses
- raw transcript fragments
- content already documented in the right place

## Confidence Rules

Write directly only when all are true:

- the user explicitly requested documentation edits or the active workflow authorizes them
- the signal is repeated, explicitly corrected by the user, or confirmed by docs, code, or git
- the target location is clear
- the update does not contradict existing documentation
- the wording can be generalized without preserving private details

Otherwise, dry-run only.

## Code Change Documentation Impact

When code changes are part of the active session or recent git scope, check whether related documentation must change before concluding there are no high-signal updates.

Update documentation when code changes affect:

- public APIs, service patterns, query keys, mutations, or data contracts
- architecture boundaries, folder conventions, imports, routing, or server/client rules
- commands, setup, environment variables, build, test, deploy, or release workflows
- design system usage, reusable UI patterns, or component conventions
- error handling, logging, telemetry, debug flags, security behavior, or permissions
- docs explicitly marked as source of truth for the touched area

Do not update docs for purely internal implementation changes unless they alter a reusable convention, public behavior, documented workflow, or source-of-truth guidance.

When the impact is strong, non-contradictory, and the target docs are clear, write the related documentation updates in the same pass if edits are authorized. Otherwise, use dry-run or a structured question.

## Multi-File Updates

Documentation updates are not limited to one file. A single durable signal may require coordinated changes across several docs.

When a signal affects multiple documentation surfaces:

- identify every impacted target file before writing
- group dry-run diffs by file
- update all affected docs in the same pass when edits are authorized
- keep one source of truth and link to it instead of duplicating detailed guidance
- validate every changed doc after the update

Examples:

- a new API convention may update `src/api/README.md` and a routing note in `.agents/rules/api.md`
- a rule placement change may update `AGENTS.md`, a specialized rule, and a skill resource
- stale guidance may require deleting one rule and adding a replacement elsewhere

## Candidate Questions

Ask a structured question before using a noticed element in documentation when the candidate is useful but:

- uncertain or weakly evidenced
- ambiguous in scope or wording
- possibly sensitive or private after redaction
- contradicted by existing documentation
- unclear in placement between global preference, project fact, workflow, rule, skill, or human docs

Prefer options shaped like:

- use as documentation
- ignore
- reformulate
- provide more context

Do not ask about candidates that should be rejected outright, such as secrets, private data, one-off task details, or raw transcript fragments.

## Dry-Run Format

Dry-run output should include proposed diffs grouped by target file:

~~~markdown
## Proposed Documentation Updates

### `path/to/file.md`
Confidence: high
Reason: repeated user correction and existing docs lack the rule.

```diff
+ New proposed line.
```

### `path/to/file2.md`
....
~~~

Do not write files during dry-run.

## Placement Rules

- `AGENTS.md`: entry point, always-on routing, concise source-of-truth statements.
- `.agents/rules/*`: stable mandatory project rules.
- `.agents/skills/*`: procedural guidance and task-specific playbooks.
- `.agents/workflows/*`: repeatable project workflows.
- `resources/*`: detailed examples, templates, and longer procedures.
- README, `docs/**`, `.github/**`: human-facing project documentation when the signal is not agent-specific.
- Global skills or memory: preferences that apply across projects.

Do not duplicate detailed sub-rules in `AGENTS.md`. Link to the specialized rule or skill instead.

## Conflict Handling

If a new signal contradicts existing documentation:

1. Stop in dry-run.
2. Cite the existing documentation and the new evidence at a high level.
3. Ask for confirmation before replacing the rule.

The only exception is clearly broken documentation, such as a dead link or a file reference that no longer exists.

Use a structured question tool when available.

## Staleness And Removal

Documentation maintenance includes deletion and replacement.

Remove or replace stale guidance when:

- newer repeated evidence shows an old convention is obsolete
- code or project structure proves a documented path no longer exists
- a rule duplicates a more authoritative source
- two docs contradict each other and the user confirms the correct version

Do not keep compatibility text for unshipped branch experiments unless the user asks.

## Post-Update Validation

After writing documentation:

- verify `AGENTS.md` links still resolve
- check for duplicate or contradictory rule statements
- keep every `SKILL.md` under 500 lines
- keep resource references one level deep
- confirm `.cursor/rules` was not edited directly

## Final Output

If updates were made, summarize:

- changed documentation files
- whether the index was refreshed
- skipped or rejected candidates, if relevant

If no meaningful documentation update exists, respond exactly:

```text
No high-signal memory updates.
```
