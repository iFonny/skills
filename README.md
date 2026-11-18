## Skills

### `maintain-agent-docs`

Keeps agent-facing documentation current. The skill reads conversations, transcripts, git context, and existing docs to find durable reusable guidance. It then proposes documentation changes for you to accept or reject.

### `mac-update`

Lists macOS and global tool updates through the manual `/mac-update` command.
The skill compares release notes with active project requirements and asks which updates to apply.
See [the skill](mac-update/SKILL.md) for the procedure.

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
