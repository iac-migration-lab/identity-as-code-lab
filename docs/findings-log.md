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
