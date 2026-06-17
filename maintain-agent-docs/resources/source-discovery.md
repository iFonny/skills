# Source Discovery

Use this guide to find documentation, transcripts, and git context.

## Documentation Sources

Read relevant existing documentation before proposing updates:

- `AGENTS.md`
- `CLAUDE.md`
- `GEMINI.md`
- `CONTRIBUTING.md`
- `README*`
- `.agents/rules/*`
- `.agents/skills/*`
- `.agents/workflows/*`
- `skills-lock.json` when present, to identify upstream-owned skills that must not be edited directly
- `docs/**`
- `.github/**`
- other project-owned `*.md` files when they are clearly part of developer or agent guidance

Do not read both `.cursor/rules` and `.agents/rules` when they are the same symlinked/generated content. Prefer `.agents/rules`.

## Files To Avoid

Do not mine these for memory updates:

- `.env*`
- credentials, secrets, tokens, certificates, private keys
- database dumps, private logs, crash dumps, local caches
- generated build output
- dependency directories

If these files appear in transcripts, do not copy their contents. Extract only a safe, durable rule if one exists.

## Transcript Sources

Always consider the current conversation first.

Then process transcript files that are new or changed since the index, newest first, before git context:

- Cursor transcripts when available in the current environment.
- Codex session transcripts when the local Codex session store exists and sessions can be matched to the current project.
- Claude or other agent transcripts only when their source locations are known, configured, or provided by the user.
- Project-specific transcript sources from project docs notes or a dedicated config section.

Known source patterns:

- `cursor`: current conversation context plus Cursor workspace transcript files, commonly under a Cursor project-state directory shaped like `<cursor-project-state>/<workspace-slug>/agent-transcripts/*.jsonl`.
- `codex`: auto-discover Codex CLI session transcripts when present, commonly under `~/.codex/sessions/**/*.jsonl`.
- `claude`: Claude Code project transcripts when present, commonly under `~/.claude/projects/**/*.jsonl`.
- `antigravity`: Google Antigravity workspace or agent transcripts when available through exports or project/user configuration.
- `other`: any user-provided or project-configured transcript source.

These patterns are discovery hints, not shared documentation paths. Validate that a source exists before using it.

Batch processing:

- use `transcriptBatchLimit` as the per-batch count
- process the whole batch, persist it, then continue automatically while more remain and the next batch fits safely
- do not start git processing until transcript processing is up to date

## Project Matching For Global Transcript Stores

Global transcript stores can contain sessions from unrelated projects. Before processing an auto-discovered transcript from a global store, match it to the current project.

For Codex transcripts:

- Discover the local Codex session store automatically when it exists.
- Read only the bounded metadata or early JSONL records needed to identify the project.
- Include a session only when metadata clearly points to the current project, such as `cwd`, `workdir`, workspace root, git repository root, or another explicit project identifier.
- Skip sessions when no clear project match is found.
- Do not infer project membership from generic conversation text alone.

Treat transcript paths as machine-local. Never store absolute local paths in shared documentation or as index keys.

Use stable transcript IDs shaped like:

```text
source:transcript-id
```

Keep `displayPath` relative or best-effort only for debugging.

## Git Context

Use git to detect documentation impact from repository changes, not only the active session.

Inspect, in order:

- current working and staged changes
- an in-progress `git.sweep`, before starting a new sweep
- commits in the selected scope since `git.baselineHead`

Cover this scope before concluding no update is needed. For baseline resolution, sweep ranges, batching,
and batch persistence, follow `index-format.md`; for scope decisions and the selected-scope guarantee, follow `SKILL.md`.

Do not store diffs, commit messages, patches, or file contents in the index.
