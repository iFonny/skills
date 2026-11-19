## Skills

### `maintain-agent-docs`

Keeps agent-facing documentation current. The skill reads conversations, transcripts, git context, and existing docs to find durable reusable guidance. It then proposes documentation changes for you to accept or reject.

### `mac-update`

Run [`/mac-update`](mac-update/SKILL.md), or ask which tools on your Mac have updates, to get a report with useful changes, security fixes, and copyable commands. 
The skill installs nothing until you ask in a later message. Add `--deep` to also check the compatibility of your projects.
The report is in French unless you ask for another language.


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
