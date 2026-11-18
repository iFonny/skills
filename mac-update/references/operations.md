# Mac update operations

Use these commands under [the skill's approval rules](./../SKILL.md).

## Inventory

Run `brew update` before Homebrew's inventory without an extra confirmation.
Retain its output and report maintenance effects, including partial changes on failure.
If it fails, skip Homebrew's outdated check.
The [Homebrew migration implementation](https://github.com/Homebrew/brew/blob/main/Library/Homebrew/cmd/update_report/reporter.rb#L245-L324) includes installation and cleanup (checked 2026-11-18).

| Manager | Check |
|---|---|
| Homebrew packages and apps | `brew outdated --greedy` |
| pnpm globals | `pnpm outdated -g` |
| pnpm itself | Installed version and confirmed owner's available version |
| npm globals | `npm outdated -g` |
| mise runtimes | `mise outdated --global`; add `--bump` to check outside configured ranges |
| Rust toolchains | `rustup check` |
| macOS and Command Line Tools | `softwareupdate --list` |
| Mac App Store | `mas outdated` |
| uv and pipx tools | Installed tool lists and available package versions |
| Self-updating applications | Offer manual checks in their update interface |

Interpret documented nonzero exit statuses for outdated packages.
For a missing npm global directory, report `npm prefix -g`, `npm root -g`, the resolved executable, and the error.
Leave the inventory unknown; do not create directories or repair configuration.

## Update commands

Update execution remains unverified until an approved live run.
Confirm command syntax against installed help and ownership against inventories and executable paths.
Use selected names and explicit versions where supported; never blanket upgrades.

| Confirmed owner | Selected update command or constraint |
|---|---|
| Homebrew formulae | `brew upgrade --formula <selected-names>` |
| Homebrew casks | `brew upgrade --cask <selected-name>`, one cask per command |
| pnpm globals | `pnpm update -g --latest <selected-packages>` |
| pnpm owned by Homebrew or Corepack | Update through that owner |
| Standalone pnpm | `pnpm self-update <approved-version>` |
| npm globals, including npm-owned pnpm | Version-specific installs under the confirmed global prefix and Node installation |
| mise within configured ranges | `mise upgrade --global <selected-tools>` |
| mise outside configured ranges | `mise upgrade --global --bump <selected-tools>` after approval of pin changes |
| rustup | Selected toolchains or the separately selected manager |
| softwareupdate | Exact confirmed non-macOS labels only; defer unclear labels |
| mas, uv, pipx | Confirm supported selected-package syntax |

Execute cask commands sequentially, verifying each before the next.
For mise outside configured ranges, preview the command with `--dry-run` before approval.
Show configuration and lockfile changes, targets, and required pin changes.
Defer when those changes are not approved.
The [mise documentation](https://mise.jdx.dev/cli/upgrade.html) explains `--bump` (checked 2026-11-18).

Use interactive manager commands only in a user-controlled terminal.
The [pnpm documentation](https://pnpm.io/cli/self-update) describes project pin changes when invoked inside projects (checked 2026-11-18).
