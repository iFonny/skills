# Project Docs Notes Lookup

The skill is global, but each project can define local documentation context and transcript source hints.

## Lookup Order

Look for project-specific documentation notes in this order:

1. `.agents/project-docs-notes.md`
2. `.agents/maintain-agent-docs/project-docs-notes.md`
3. a clearly named documentation notes section in `AGENTS.md`

If none exist, continue with defaults. Do not create project docs notes unless the user asks or the active documentation update requires them.

## What Project Docs Notes May Contain

Project docs notes are lightweight durable memory for one project's documentation. They may contain plain markdown bullets such as:

- documentation ownership and placement preferences
- project-specific docs that must be checked
- project-specific docs that must be ignored
- source-of-truth reminders
- stable workflow notes
- known transcript source labels or local transcript discovery hints
- retention preferences for `.agents/state/agent-updated-docs-index.json`

Example:

```markdown
# Project Docs Notes

Project: Investhub Back-Office

- New API patterns: keep `src/api/README.md` as the API source of truth.
- Design system decisions belong in `.agents/skills/design-system/` resources, not `AGENTS.md`.
- Do not mine `.env*` files or private logs for memory updates.
```

## Optional Structured Config

Use YAML only when the project needs machine-readable configuration. Plain markdown is preferred for normal notes.

```yaml
maintainAgentDocs:
  transcriptSources:
    - source: cursor
      description: Cursor agent transcripts for this workspace
    - source: codex
      description: Codex session transcripts when configured locally
    - source: github-copilot
      description: GitHub Copilot Chat exports or configured local transcript source
    - source: antigravity
      description: Antigravity workspace or agent transcripts when configured locally
  retention:
    maxTranscriptEntries: 500
    maxAgeDays: 180
  docs:
    include:
      - AGENTS.md
      - .agents/**
      - README*
      - docs/**
    exclude:
      - .env*
      - "**/secrets/**"
```

Keep user-specific absolute paths out of versioned notes. If a machine-local path is unavoidable, the user should provide it during the session or keep it in an untracked local file.

## Scope Rules

- Project facts belong in project documentation.
- Global user preferences belong in global memory or a global skill only when they apply across projects.
- Tool-specific details belong in focused resources or project docs notes, not in always-on entry files.
