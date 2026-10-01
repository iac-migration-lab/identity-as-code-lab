# Tool Install (Apple Silicon Mac)

Versions and download URLs checked 2026-10-01 against the vendor release pages listed under Sources.

## Design
- Pinned binaries go in `~/Projects/okta-to-entra-migration/lab-tools/` (inside the project, git-ignored) and are only on PATH after `source ~/Projects/okta-to-entra-migration/lab-tools/env.sh`. Nothing else on the machine changes (Homebrew Node 26 stays as is).
- Every download is checksum-verified before it is unpacked.
- Azure CLI and PowerShell come from Homebrew (both as formulae), which installs only the current version. They are pinned by `brew pin` (formula) and by the version check below, not by exact install.

## Pinned versions

| Tool | Version | Method | Verified |
|---|---|---|---|
| Terraform CE (Community Edition) | 1.16.4 | releases.hashicorp.com zip + SHA256SUMS | Yes |
| Azure CLI | 2.90.0 | `brew install azure-cli` (Microsoft's documented macOS method) | Yes, method and current version |
| SailPoint CLI (`sail`) | 2.6.0 | GitHub release tarball + checksums file | Yes, release assets, checksum line, and tarball layout (confirmed on install 2026-10-01) |
| Conftest | 0.71.0 | GitHub release tarball + checksums.txt | Yes |
| PowerShell 7 | 7.6.6 | `brew install powershell` (Homebrew formula, not a cask; already installed) | Yes, latest release and formula type (2026-10-01) |
| Microsoft365DSC | 1.26.909.1 | PowerShell Gallery `Install-Module` | Yes, version and PowerShell 7.6+ requirement. UNVERIFIED: macOS support (docs mention Windows only) |
| Node LTS (Long-Term Support) | 24.21.0 | nodejs.org tarball + SHASUMS256.txt | Yes |
| Stoplight Prism | 5.16.0 | `npm install -g @stoplight/prism-cli@5.16.0` (needs Node >= 24.18.0) | Yes |

## Install block

Paste the whole block into a terminal. It runs in a child bash shell, so a failure stops the script without closing your terminal.

```bash
bash -euo pipefail <<'EOF'
TF=1.16.4; SAIL=2.6.0; CONFTEST=0.71.0; NODE=24.21.0; PRISM=5.16.0; M365DSC=1.26.909.1
T="$HOME/Projects/okta-to-entra-migration/lab-tools"; mkdir -p "$T/bin" "$T/dl"; cd "$T/dl"

# Terraform CE
curl -fsSLO "https://releases.hashicorp.com/terraform/$TF/terraform_${TF}_darwin_arm64.zip"
curl -fsSLO "https://releases.hashicorp.com/terraform/$TF/terraform_${TF}_SHA256SUMS"
grep " terraform_${TF}_darwin_arm64.zip$" "terraform_${TF}_SHA256SUMS" | shasum -a 256 -c -
unzip -o -q "terraform_${TF}_darwin_arm64.zip" terraform -d "$T/bin"

# SailPoint CLI (binary sits at sail_Darwin_arm64/bin/sail inside the tarball)
curl -fsSLO "https://github.com/sailpoint-oss/sailpoint-cli/releases/download/$SAIL/sail_Darwin_arm64.tar.gz"
curl -fsSLO "https://github.com/sailpoint-oss/sailpoint-cli/releases/download/$SAIL/sail_${SAIL}_checksums.txt"
grep " sail_Darwin_arm64.tar.gz$" "sail_${SAIL}_checksums.txt" | shasum -a 256 -c -
tar -xzf sail_Darwin_arm64.tar.gz -C "$T/bin" --strip-components 2 sail_Darwin_arm64/bin/sail

# Conftest
curl -fsSLO "https://github.com/open-policy-agent/conftest/releases/download/v$CONFTEST/conftest_${CONFTEST}_Darwin_arm64.tar.gz"
curl -fsSL -o conftest_checksums.txt "https://github.com/open-policy-agent/conftest/releases/download/v$CONFTEST/checksums.txt"
grep " conftest_${CONFTEST}_Darwin_arm64.tar.gz$" conftest_checksums.txt | shasum -a 256 -c -
tar -xzf "conftest_${CONFTEST}_Darwin_arm64.tar.gz" -C "$T/bin" conftest

# Node LTS, side by side with Homebrew Node
curl -fsSLO "https://nodejs.org/dist/v$NODE/node-v$NODE-darwin-arm64.tar.gz"
curl -fsSL -o node_SHASUMS256.txt "https://nodejs.org/dist/v$NODE/SHASUMS256.txt"
grep " node-v$NODE-darwin-arm64.tar.gz$" node_SHASUMS256.txt | shasum -a 256 -c -
rm -rf "$T/node" && mkdir -p "$T/node"
tar -xzf "node-v$NODE-darwin-arm64.tar.gz" -C "$T/node" --strip-components 1

# Stoplight Prism, installed into the pinned Node
PATH="$T/node/bin:$PATH" npm install -g "@stoplight/prism-cli@$PRISM"

# Azure CLI (Homebrew installs current; pin stops upgrades)
brew update
brew install azure-cli
brew pin azure-cli

# PowerShell 7 (already present at 7.6.6; installs current if missing)
brew list --formula powershell >/dev/null 2>&1 || brew install powershell

# Microsoft365DSC (UNVERIFIED on macOS)
pwsh -NoProfile -Command "Install-Module Microsoft365DSC -RequiredVersion $M365DSC -Scope CurrentUser -Force"

# Lab PATH, opt-in per terminal
printf 'export PATH="%s/bin:%s/node/bin:$PATH"\n' "$T" "$T" > "$T/env.sh"
echo "Done. Run: source ~/Projects/okta-to-entra-migration/lab-tools/env.sh"
EOF
```

## Version check block

Run this, then paste the full output back.

```bash
source ~/Projects/okta-to-entra-migration/lab-tools/env.sh
echo "terraform: $(terraform -version | head -1)"
echo "az: $(az version --query '"azure-cli"' -o tsv)"
echo "sail: $(sail --version 2>&1 | head -1)"
echo "conftest: $(conftest --version 2>&1 | head -1)"
echo "pwsh: $(pwsh --version)"
echo "Microsoft365DSC: $(pwsh -NoProfile -Command '(Get-Module -ListAvailable Microsoft365DSC | Sort-Object Version -Descending | Select-Object -First 1).Version.ToString()')"
echo "node: $(node --version)"
echo "prism: $(prism --version)"
echo "git: $(git --version)"
```

## Notes
- Microsoft365DSC docs state "Microsoft365DSC requires PowerShell 7.6 or higher" and every reference is Windows. If `Export-M365DSCConfiguration` fails on macOS in Phase 7, run it on a Windows GitHub Actions runner instead. `Update-M365DSCDependencies` is deferred to Phase 7.
- `brew unpin azure-cli` before any deliberate upgrade; record the new version in docs/findings-log.md.

## Sources
- Terraform releases: https://releases.hashicorp.com/terraform/1.16.4/
- Azure CLI on macOS: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-macos
- SailPoint CLI install: https://developer.sailpoint.com/docs/tools/cli/
- SailPoint CLI releases: https://github.com/sailpoint-oss/sailpoint-cli/releases/tag/2.6.0
- Conftest releases: https://github.com/open-policy-agent/conftest/releases/tag/v0.71.0
- PowerShell releases: https://github.com/PowerShell/PowerShell/releases/tag/v7.6.6
- Microsoft365DSC prerequisites: https://microsoft365dsc.com/user-guide/get-started/prerequisites/
- Microsoft365DSC on PowerShell Gallery: https://www.powershellgallery.com/packages/Microsoft365DSC
- Node.js release index: https://nodejs.org/dist/index.json
- Prism on npm: https://www.npmjs.com/package/@stoplight/prism-cli
