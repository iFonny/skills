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
