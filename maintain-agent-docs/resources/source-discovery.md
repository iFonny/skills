# Source Discovery

Use this guide to find documentation, transcripts, and git context without making the global skill project-specific.

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

Then process transcript files that are new or changed since the index:

- Cursor transcripts when available in the current environment.
- Codex, Claude, or other agent transcripts only when their source locations are known, configured, or provided by the user.
- Project-specific transcript sources from project docs notes or a dedicated config section.

Known source patterns:

- `cursor`: current conversation context plus Cursor workspace transcript files, commonly under a Cursor project-state directory shaped like `<cursor-project-state>/<workspace-slug>/agent-transcripts/*.jsonl`.
- `codex`: Codex CLI session transcripts when present, commonly under `~/.codex/sessions/**/*.jsonl`.
- `claude`: Claude Code project transcripts when present, commonly under `~/.claude/projects/**/*.jsonl`.
- `antigravity`: Google Antigravity workspace or agent transcripts when available through exports or project/user configuration.
- `other`: any user-provided or project-configured transcript source.

These patterns are discovery hints, not shared documentation paths. Validate that a source exists before using it.

Treat transcript paths as machine-local. Never store absolute local paths in shared documentation or as index keys.

Use stable transcript IDs shaped like:

```text
source:project-or-workspace-slug:transcript-id
```

Keep `displayPath` relative or best-effort only for debugging.

## Git Context

Use git context to confirm stable changes, not to infer broad rules from weak evidence.

Default bounds:

- current working changes
- staged changes
- new commits since `git.lastProcessedHead`
- if the previous head is unavailable or unreachable, only a small recent window such as the latest 30 commits

Do not store diffs, commit messages, patches, or file contents in the index. Store only metadata described in `index-format.md`.
