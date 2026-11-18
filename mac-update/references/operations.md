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
Reuse these templates without routine syntax searches.
Substitute verified names, paths, versions, and non-macOS labels, with shell quoting.
Keep the confirmed installation manager.
Consult local help only for unsupported syntax or version-dependent options.

The linked command references were checked on 2026-11-18.
Manual batches are for the user's terminal only; agent execution requires explicit selected items.

| Confirmed owner | Agent command for selected items | Manual batch command |
|---|---|---|
| [Homebrew](https://docs.brew.sh/Manpage) formulae | `brew upgrade --formula <names>` | `brew upgrade --greedy` |
| Homebrew casks | `brew upgrade --cask <name>` | Included in the Homebrew batch |
| [pnpm](https://pnpm.io/cli/update) globals | `pnpm update -g --latest <names>` | `pnpm update -g --interactive --latest` |
| pnpm owned by Homebrew | `brew upgrade --formula pnpm` | Same command |
| pnpm owned by [Corepack](https://github.com/nodejs/corepack#utility-commands) | `corepack install --global pnpm@<version>` | Same command |
| [Standalone pnpm](https://pnpm.io/cli/self-update) | `pnpm self-update <version>` | Same command |
| [npm](https://docs.npmjs.com/cli/v11/commands/npm-install) globals, including npm-owned pnpm | `npm install -g --prefix <prefix> <name>@<version>` | `npm install -g --prefix <prefix> <specs>` |
| [mise](https://mise.jdx.dev/cli/upgrade.html), within configured ranges | `mise upgrade --global <tools>` | `mise upgrade --global --interactive` |
| mise, outside configured ranges | `mise upgrade --global --bump <tools>` | `mise upgrade --global --bump --interactive <tools>` |
| [Rust toolchains](https://github.com/rust-lang/rustup/blob/master/src/cli/rustup_mode.rs) | `rustup update --no-self-update <toolchains>` | `rustup update --no-self-update` |
| Standalone rustup manager | `rustup self update` | Same command |
| [Mac App Store](https://github.com/mas-cli/mas#commands) | `mas upgrade <ids>` | `mas upgrade` |
| [uv tools](https://docs.astral.sh/uv/guides/tools/#upgrading-tools) | `uv tool upgrade <name>` | `uv tool upgrade --all` |
| [pipx tools](https://pipx.pypa.io/stable/how-to/manage-installed-apps.html) | `pipx upgrade <name>` | `pipx upgrade-all` |
| Command Line Tools | `softwareupdate --install <labels>` | Same explicit non-macOS labels; no unrestricted batch |

`<specs>` contains the reviewed npm `name@version` pairs.
Use the confirmed Node installation and global prefix.
Update managers installed through another owner using that owner's row, never their standalone updater.

For agent execution, run cask commands sequentially, verifying each before the next.
For mise outside configured ranges, preview the command with `--dry-run` before approval.
Show configuration and lockfile changes, targets, and required pin changes.
Defer when those changes are not approved.
The [mise documentation](https://mise.jdx.dev/cli/upgrade.html) explains `--bump` (checked 2026-11-18).

Manual batches must preserve native selection and confirmation prompts without `--yes`.
Do not describe commands without native prompts as interactive.
Omit any batch that would include blocked items, unmet prerequisites, or unintended updates.
For Homebrew, pnpm, and mise, append explicit item names when needed to limit the manual batch.
The [pnpm documentation](https://pnpm.io/cli/self-update) describes project pin changes when invoked inside projects (checked 2026-11-18).
