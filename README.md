## Skills

### `maintain-agent-docs`

Keeps agent-facing documentation current. The skill reads conversations, transcripts, git context, and existing docs to find durable reusable guidance. It then proposes documentation changes for you to accept or reject.

### `mac-update`

Use [`/mac-update`](mac-update/SKILL.md) to review useful changes and choose updates, starting with those that have limited impact.
Add `--deep` for project compatibility analysis.
Both modes show important changes and risks before you choose what to install.

## Install

```bash
gh skill install iFonny/skills maintain-agent-docs
```

After publication, install `mac-update`:

```bash
gh skill install iFonny/skills mac-update
```

To get later changes:

```bash
gh skill update
```
