## Skills

### `maintain-agent-docs`

Keeps agent-facing documentation current. The skill reads conversations, transcripts, git context, and existing docs to find durable reusable guidance. It then proposes documentation changes for you to accept or reject.

### `mac-update`

Use [`/mac-update`](mac-update/SKILL.md) to review available macOS and global tool updates.
See important changes and risks, including their potential impact on your projects, before choosing what to install.
Homebrew maintenance runs first and can install or clean up packages during automatic migrations.

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
