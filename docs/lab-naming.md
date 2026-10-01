# Lab Naming

Prefix `iac` (Infrastructure as Code) everywhere. Dashes where the platform allows them; where it does not, the same words with the dashes removed.

| Object | Name | Notes |
|---|---|---|
| GitHub org | `iac-migration-lab` | Must be globally unique; if taken, append `-01`. Free plan, public. |
| GitHub repo | `identity-as-code-lab` | Fixed by the plan. Public (needed for free environment gates). |
| Lab email | `la-migration-lab@outlook.com` | Created 2026-10-01. Personal Microsoft account (Outlook.com), used for every lab sign-up. Not on `lencioni.io`: that domain belongs to the owner's company Microsoft 365 tenant. Does not follow the `iac` prefix; kept as created. |
| GitHub second reviewer account | `iac-lab-reviewer` | Personal GitHub account for the `prod` required-reviewer gate (Phase 3). |
| Entra tenant display name | `IaC Migration Lab` | Display only; changeable later. |
| Entra initial domain | `iacmigrationlab.onmicrosoft.com` | No dashes. UNVERIFIED: whether the initial domain prefix accepts hyphens; letters and numbers only is the safe choice. Permanent once created. |
| Okta org | `IaC Migration Lab` (org name) | UNVERIFIED: Integrator Free Plan assigns the subdomain (e.g. `integrator-NNNNNNN.okta.com`); record the actual URL here after sign-up. |
| Azure resource group | `rg-iac-lab` | One region for everything; record it here after creation. |
| Azure storage account | `stiaclabtfstate` | 3 to 24 lowercase letters and digits, no dashes, globally unique. If taken, append two digits (`stiaclabtfstate01`). Containers: `tfstate-nonprod`, `tfstate-prod`. |
| Entra app registration (read) | `iac-reader` | Fixed by CLAUDE.md. |
| Entra app registration (plan) | `iac-planner` | Fixed by CLAUDE.md. |
| Entra app registration (apply) | `iac-applier` | Fixed by CLAUDE.md. |
| Okta service app (inventory) | `iac-okta-reader` | Read-only scopes + Read-only Administrator role. |
| Okta service app (cutover) | `iac-okta-cutover` | `okta.apps.manage` only; deactivated between waves. |
| Break-glass account | `bg-admin-01@iacmigrationlab.onmicrosoft.com` | Display name `Break Glass 01`. Cloud-only Global Administrator, excluded from every CA (Conditional Access) policy. Group: `bg-ca-exclusion`. |
