Current phase: 0

# Findings Log

Append only. Newest entry at the bottom. Never rewrite an earlier entry; correct it with a new one.

## Entry template

```
## YYYY-MM-DD, Phase N, short title
- What the docs said:
- What actually happened:
- Time taken:
- Evidence:
- Change to the client plan, if any:
- Accepted risks added, if any:
```

---

## 2026-10-01, Phase 0, Phase 0 started
- What the docs said: Phase 0 installs Terraform CE (Community Edition) 1.11+, Azure CLI (command-line interface), SailPoint CLI, Conftest, PowerShell 7 + Microsoft365DSC (Desired State Configuration), and Node; exit requires `terraform -version`, `sail --version`, `conftest --version` recorded here.
- What actually happened: Tool versions detected on this machine (macOS 26.6.2, arm64, Homebrew 7.0.6) before any install:
  - terraform: not installed
  - az: not installed
  - sail: not installed
  - conftest: not installed
  - pwsh: 7.6.6 (Homebrew cask)
  - node: 26.8.1 (Homebrew; "Current" line, not LTS (Long-Term Support). Prism 5.16.0 needs Node >= 24.18.0; lab will use Node 24.21.0 LTS side by side, see docs/tool-install.md)
  - git: 2.55.0
  - Microsoft365DSC module: not installed
  - prism: not installed
- Time taken: under 1 hour (skeleton docs only).
- Evidence: shell output in the Phase 0 Claude Code session; docs/tool-install.md lists target versions verified 2026-10-01.
- Change to the client plan, if any: None yet. Observation: Microsoft365DSC docs now require PowerShell 7.6+ but do not mention macOS or Linux support; Phase 7 may need a Windows GitHub Actions runner.
- Accepted risks added, if any: Azure CLI and the PowerShell cask install through Homebrew, which only offers the current version; versions are pinned by assertion (check block) and `brew pin`, not by exact install.

## 2026-10-01, Phase 0, Install block 1 run (stopped at PowerShell step)
- What the docs said: docs/tool-install.md install block downloads and checksum-verifies Terraform, SailPoint CLI, Conftest, and Node; installs Prism, Azure CLI, PowerShell 7 (as a Homebrew cask), and Microsoft365DSC; then writes `~/lab-tools/env.sh`.
- What actually happened:
  - Checksums OK: terraform_1.16.4_darwin_arm64.zip, sail_Darwin_arm64.tar.gz, conftest_0.71.0_Darwin_arm64.tar.gz, node-v24.21.0-darwin-arm64.tar.gz.
  - SailPoint CLI tarball holds the binary at `sail_Darwin_arm64/bin/sail`, not at the root; extract line changed to `--strip-components 2` before this run.
  - Prism 5.16.0 installed (210 packages). npm deprecation warnings for transitive deps uuid@8.3.2, json-schema-ref-parser@6.1.0, @faker-js/faker@5.5.3. npm skipped the `@scarf/scarf` postinstall script (install telemetry) because it is not allow-listed; left blocked on purpose.
  - Azure CLI 2.90.0 installed via Homebrew and pinned (`brew list --pinned` shows azure-cli). Side effect: Homebrew also installed libsodium 1.0.22 and upgraded readline 8.3.6 and xz 5.8.4.
  - Script stopped at `brew install --cask powershell`: "Cask 'powershell' is unavailable: No Cask with this name exists." PowerShell is now a Homebrew formula; `brew info powershell` shows formula 7.6.6 installed. The 2026-10-01 "Phase 0 started" entry calling it a cask was wrong.
  - Not done because the script stopped: Microsoft365DSC install, `~/lab-tools/env.sh`.
  - Spot check after the run: terraform v1.16.4, sail 2.6.0, conftest 0.71.0, node v24.21.0, az 2.90.0.
- Time taken: about 10 minutes.
- Evidence: terminal output pasted into the Claude Code session 2026-10-01; `~/lab-tools/bin` contains terraform, sail, conftest.
- Change to the client plan, if any: Install scripts must check Homebrew formula vs cask at run time; package type changes without notice.
- Accepted risks added, if any: Prism carries deprecated transitive npm packages (mock server only, local, no real data).

## 2026-10-01, Phase 0, Version check run before install finished
- What the docs said: docs/tool-install.md version check block sources `~/lab-tools/env.sh`, then prints each tool version for this log.
- What actually happened: The check ran before the two remaining install steps (Microsoft365DSC install, writing `~/lab-tools/env.sh`). Output:
  - `source`: no such file `~/lab-tools/env.sh`
  - terraform, sail, conftest, prism: "command not found" (binaries exist in `~/lab-tools/bin`, but that folder is not on PATH (the shell's search list) without env.sh)
  - az: 2.90.0
  - pwsh: PowerShell 7.6.6
  - Microsoft365DSC: "InvalidOperation: You cannot call a method on a null-valued expression." (module not installed)
  - node: v26.8.1 (Homebrew Node, not the lab's 24.21.0, because env.sh is missing)
  - git: 2.55.0
- Time taken: under 5 minutes.
- Evidence: terminal output pasted into the Claude Code session 2026-10-01; same result when Claude ran the check in its own shell.
- Change to the client plan, if any: The version check should test that env.sh exists first and print one clear "install incomplete" line, instead of a cascade of "command not found" errors.
- Accepted risks added, if any: None.

## 2026-10-01, Phase 0, Lab tools moved into the project folder
- What the docs said: docs/tool-install.md put pinned binaries in `~/lab-tools/`.
- What actually happened: Owner asked for the tools to live inside the project. Moved `~/lab-tools` (587 MB: bin, dl, node) to `~/Projects/okta-to-entra-migration/lab-tools/` with `mv`; nothing re-downloaded. Prism's link is relative, so it survived the move. Added `lab-tools/` to .gitignore; `git status` confirms it is not tracked. Updated every path in docs/tool-install.md. Entries above that say `~/lab-tools` describe the old location.
- Time taken: under 5 minutes.
- Evidence: `ls lab-tools` shows bin, dl, node; `git status --short` shows no lab-tools files.
- Change to the client plan, if any: None.
- Accepted risks added, if any: Tool binaries sit inside the repo folder; protected from commit only by .gitignore.

## 2026-10-01, Phase 0, Install finished, version check passed
- What the docs said: Phase 0 exit requires `terraform -version`, `sail --version`, `conftest --version` recorded here. docs/tool-install.md pins the versions below.
- What actually happened: Claude ran the two remaining steps locally (no cloud or provider API calls): wrote `lab-tools/env.sh`, and ran `Install-Module Microsoft365DSC -RequiredVersion 1.26.909.1 -Scope CurrentUser -Force` (15 seconds, no errors, no output). Version check after `source lab-tools/env.sh`:
  - terraform: Terraform v1.16.4
  - az: 2.90.0 (Homebrew, pinned)
  - sail: sail version 2.6.0
  - conftest: Conftest: 0.71.0
  - pwsh: PowerShell 7.6.6 (Homebrew formula)
  - Microsoft365DSC: 1.26.909.1
  - node: v24.21.0 (lab copy; `which node` resolves to lab-tools/node/bin)
  - prism: 5.16.0
  - git: 2.55.0
  All match the pinned versions in docs/tool-install.md. The three Phase 0 exit versions (terraform, sail, conftest) are recorded.
- Time taken: under 5 minutes.
- Evidence: version check output in the Claude Code session 2026-10-01; `brew list --pinned` shows azure-cli.
- Change to the client plan, if any: None. Microsoft365DSC installs on macOS; whether `Export-M365DSCConfiguration` runs on macOS is still UNVERIFIED until Phase 7.
- Accepted risks added, if any: None.

## 2026-10-01, Phase 0, Private GitHub repo created; environment gates not available on it
- What the docs said: CLAUDE.md says applies run only through GitHub Actions with environment gates, and secrets live in GitHub environment secrets. Before checking, Claude said (UNVERIFIED) that free-plan private repos lack required reviewers but GitHub Pro would fix it.
- What actually happened:
  - Owner approved; Claude ran `gh repo create mlencioni/okta-to-entra-migration --private --source . --remote origin --push`. Repo: https://github.com/mlencioni/okta-to-entra-migration, visibility PRIVATE, personal account (not a company org). Pushed `main` (commits 3c682fa, 61b87d6).
  - GitHub docs (deployments-and-environments reference, read 2026-10-01) say, for Free, Pro, and Team plans on private repos: required reviewers, wait timers, custom protection rules, and admin bypass are public-repo only. Free private repos also get no environment secrets, no environment variables, and no deployment branch restrictions; Pro or Team adds those three, but not reviewers. Claude's earlier "Pro fixes it" was wrong: reviewers on a private repo need GitHub Enterprise.
  - Account plan not confirmed (`gh api user --jq .plan.name` returned nothing with the current token scopes); assumed Free.
- Time taken: under 10 minutes.
- Evidence: `gh repo view --json visibility,url`; https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
- Change to the client plan, if any: Decide before Phase 1 workflows: (a) make the lab repo public (nothing secret is committed; federated credentials need no stored secret), or (b) stay private and replace environment gates with branch protection plus a manual `workflow_dispatch` apply, with secrets at repo level. Client plan should state that environment-gated applies on private repos need GitHub Enterprise.
- Accepted risks added, if any: Until decided, the repo has no enforceable apply gate; no apply workflow exists yet, so no live exposure.

## 2026-10-01, Phase 0, Repo moved to lab org per docs/lab-naming.md
- What the docs said: docs/lab-naming.md and the plan put the repo at `iac-migration-lab/identity-as-code-lab`, public, in a free lab org, not under the owner's personal account.
- What actually happened:
  - Claude had created `mlencioni/okta-to-entra-migration` without reading docs/lab-naming.md (process miss; caught during the pre-public scan, before anything was public).
  - Pre-public scan of all tracked files and full history: no secrets, keys, or client names. Only personal data: owner's name and personal email as commit author on all commits.
  - Owner created org `iac-migration-lab` (Free plan) in the browser; owner is admin.
  - Claude transferred and renamed the repo with `gh api -X POST repos/mlencioni/okta-to-entra-migration/transfer -f new_owner=iac-migration-lab -f new_name=identity-as-code-lab`, and set `origin` to https://github.com/iac-migration-lab/identity-as-code-lab.git. Still PRIVATE.
  - Local folder name stays `okta-to-entra-migration`; only the GitHub name changed.
- Time taken: about 15 minutes.
- Evidence: `gh api repos/iac-migration-lab/identity-as-code-lab --jq '.full_name + " " + .visibility'` returned `iac-migration-lab/identity-as-code-lab private`.
- Change to the client plan, if any: Read the naming doc before creating any named resource.
- Accepted risks added, if any: None yet; author email decision pending before the repo goes public.

## 2026-10-01, Phase 0, Commit email rewritten, repo recreated, made public
- What the docs said: Plan and docs/lab-naming.md: repo is public (needed for free environment gates and environment secrets). No personal data in the repo.
- What actually happened:
  - Rewrote author and committer email on all commits to `mark@lencioni.io` with `git filter-branch --env-filter` (first to the GitHub noreply address, then to mark@lencioni.io at the owner's request). Dates and file contents unchanged (`git diff` against the pre-rewrite branch was empty). Repo-local `user.email` set to mark@lencioni.io. Force-pushed with `--force-with-lease`.
  - Finding: after the force-push, GitHub still served the old commits by SHA (the original commits returned the old personal email). A force-push does not remove commits from GitHub. Commit SHAs quoted in earlier entries (3c682fa, 61b87d6) were among them.
  - Fix: confirmed the repo held only `main` (0 forks, issues, releases, webhooks), owner added the `delete_repo` scope to the `gh` login, Claude ran `gh repo delete` then `gh repo create iac-migration-lab/identity-as-code-lab --private` and pushed `main`. All six old SHAs now return "No commit found". New history: ba98562, 2fb6215, d05e508, 53ef7ff, plus this entry.
  - Then set visibility to public.
  - Old SHAs in earlier entries of this log are now dead references; this entry maps them.
- Time taken: about 30 minutes, including a failed first `gh auth refresh`.
- Evidence: `gh api repos/iac-migration-lab/identity-as-code-lab/commits/<old sha>` returns 404 for 3c682fa, 61b87d6, 88d2358, d6eca41, 4ec1f3e.
- Change to the client plan, if any: To scrub data from a GitHub repo's history, rewriting plus force-pushing is not enough; delete and recreate the repo (or ask GitHub Support to purge) and check old SHAs before making it public. Set the commit email before the first commit.
- Accepted risks added, if any: Owner's name and mark@lencioni.io are public in commit metadata (accepted by owner).
