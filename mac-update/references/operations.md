# Mac update operations

Read only the section requested by [the skill](./../SKILL.md).
These command details follow its audit and approval rules.

## Inventory

Check only the installed managers below; skip missing managers without installing or researching alternatives.
For inventory checks, use commands that list updates without installing them.
Before Homebrew maintenance, announce its possible migrations, installations, link changes, and cleanup.
For Homebrew, run `brew update` without confirmation before `brew outdated --greedy`.
Retain its output and report observed maintenance effects, including changes made before a failure.
Mark any unclear effects as unknown.
If the refresh fails, report the error and skip its outdated check.
Homebrew's [migration handling](https://github.com/Homebrew/brew/blob/main/Library/Homebrew/cmd/update_report/reporter.rb#L245-L324) includes installation and cleanup commands (checked 2026-11-18).

| Manager | Check |
|---|---|
| Homebrew packages and apps | `brew outdated --greedy` |
| pnpm globals | `pnpm outdated -g` |
| pnpm itself | Installed version and its manager's available version |
| npm globals | `npm outdated -g` |
| mise runtimes | `mise outdated --global`; `mise outdated --global --bump` for versions outside configured ranges |
| Rust toolchains | `rustup check` |
| macOS and Command Line Tools | `softwareupdate --list` |
| Mac App Store | `mas outdated` |
| uv and pipx tools | Installed tool lists and available package versions |
| Self-updating applications | Offer manual checks through their update interface |

Read application version metadata without launching applications.
For a missing npm global directory, inspect `npm prefix -g` and `npm root -g`.
Report the prefix, resolved npm executable, and error; keep its package inventory unknown until confirmed.
Offer further diagnosis without creating directories or changing configuration.

## Research

Open official release notes directly when their location is known.
In eco mode, start with consolidated notes or the target release notes.
If these are unavailable or incomplete, use at most one fallback search and one alternative official source per candidate.
Do not retry failed fallback requests in eco mode.
Stop at the limit and mark uncovered versions or missing information explicitly.

Filter manager metadata and request bounded source excerpts without truncating useful release notes.
Read security summaries in release notes or a directly linked vendor bulletin.
Make at most one additional security lookup per candidate when a warning needs clarification.
Count retries and alternative sources toward this limit.
Report unresolved points briefly when the limit is reached.
Do not search CVE databases or inspect exploit details during routine audits.
Deeper security analysis, including local exposure checks, requires an explicit user request.
Never execute vulnerability reproductions during the audit.
Run at most three independent release research tasks concurrently.
Reuse findings for the same tool and version interval within the audit.
Reduce concurrency when rate limits occur and respect the source's retry delay.
For non-security release research in full mode, after one unsuccessful retry, use another official source or report the gap.

## Report delivery

Choose the output format before drafting.
Prefer a normal response for text and source links.
Use a publication tool only when its attachments provide useful evidence or a requested deliverable.
Check its current length limits and required fields before submission.
Measure each prepared message against those limits.
Split long reports at manager or item boundaries without removing distinct useful changes or warnings.
Avoid duplicate summaries across messages and repeated entries during selection.

## Update commands

This execution procedure is a draft until its first approved run.
Use selected package names instead of blanket upgrades.
Prefer explicit versions when supported.
Confirm syntax with installed help before using version-dependent flags.

| Confirmed owner | Selected update command or constraint |
|---|---|
| Homebrew formulae | `brew upgrade --formula <selected-names>` |
| Homebrew casks | `brew upgrade --cask <selected-name>`, one cask per command |
| pnpm globals | `pnpm update -g --latest <selected-packages>` |
| pnpm owned by Homebrew or Corepack | Update through that owner |
| Standalone pnpm | `pnpm self-update <approved-version>` |
| npm-owned pnpm | Version-specific npm install with the confirmed global prefix |
| npm globals | Version-specific installs for selected packages under the confirmed Node installation |
| mise within configured ranges | `mise upgrade --global <selected-tools>` |
| mise outside configured ranges | `mise upgrade --global --bump <selected-tools>` after approval of global pin changes |
| rustup | Limit updates to selected toolchains or the separately selected manager |
| softwareupdate | Select exact labels for confirmed non-macOS updates; never use an unconditional install-all command |
| mas, uv, pipx | Confirm supported syntax before proposing commands |

Defer `softwareupdate` labels when their scope is unclear.
Approve casks as a batch when appropriate, but execute their commands sequentially.
Verify each result before starting the next cask; stop and ask after a failure.

For an out-of-range mise target, add `--dry-run` to the selected command before requesting approval.
Show affected configuration and lockfiles, target versions, and proposed pin changes.
Defer the target if the required pin changes are not approved.
The [mise upgrade documentation](https://mise.jdx.dev/cli/upgrade.html) describes how `--bump` changes configured ranges (checked 2026-11-18).

Offer `pnpm update -g --interactive --latest` only in a user-controlled terminal.
Otherwise, use structured choices.
The [pnpm self-update documentation](https://pnpm.io/cli/self-update) describes project pin changes when run in a project (checked 2026-11-18).
