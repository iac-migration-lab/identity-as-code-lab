# Identity as Code Lab

Personal proof of concept of an Okta-to-Microsoft Entra ID migration run "as code", with SailPoint as the provisioning standard, on free-tier throwaway accounts. The owner is a buy-side IAM (Identity and Access Management) architect learning the method hands-on, not a developer.

## Read first, every session
- docs/poc-plan-identity-as-code-free-tier.md — the plan. Phases run in order. Current phase is tracked at the top of docs/findings-log.md.
- docs/terraform-provider-analysis-okta-entra-sailpoint.md — provider facts, limits, known bugs.
- docs/findings-log.md — what has actually happened so far. Append, never rewrite.

## Hard rules
- Throwaway accounts only. Never reference or configure the owner's company Microsoft 365 tenant, company SharePoint, production GitHub organization, or any client environment. If a step would, stop and say so.
- Never run `terraform apply` locally. Applies happen only through the GitHub Actions pipeline with environment gates. Local use is `init`, `validate`, `fmt`, `plan`, and `import`/`-generate-config-out`.
- Never write a secret, private key, client secret, or exported user JSON into the repo, into CLAUDE.md, or into a commit. Secrets live in GitHub environment secrets; state lives in Azure Storage; PII (personally identifiable information) exports go to Azure Blob only.
- Do not activate the Entra ID P2 trial, and do not create any Entra resource that requires P1/P2, until findings-log.md says Phase 2 has started.
- Ask before: creating or deleting any cloud resource, granting any permission or scope, running any command that calls a provider API with write intent, or force-pushing.

## Least-privilege gate
Before proposing any grant (scope, role, Graph permission, RBAC, secret), state in one short block: minimum needed, what broader would add, blast radius if leaked, cost of going narrow. Build the narrow one. Fixed decisions:
- Okta: OAuth 2.0 API service app, private key JWT (JSON Web Token), read-only scopes + Read-only Administrator role for inventory; separate `okta.apps.manage`-only identity for cutover, disabled between waves. No SSWS API tokens.
- Entra: app registrations `iac-reader`, `iac-planner`, `iac-applier` with federated credentials from GitHub Actions; no client secrets; Graph application permissions scoped to the current phase; admin consent only from the break-glass account.
- Break-glass account excluded from every Conditional Access policy, by module and by Conftest rule.

## How to answer
- Define every acronym in parentheses on first use, including HCL (HashiCorp Configuration Language).
- Verify provider arguments, CLI flags, and portal paths against current docs before using them; prefix anything unverified with `UNVERIFIED:`.
- Outline format. Short.
- When you need something from the owner, give one copy-pasteable command and ask for the output.
- One question per turn at most.
- Pin every provider and tool version. Record versions in findings-log.md.
- File names use dashes, not underscores.

## Repo layout
okta-inventory/   read-only Terraform for the Okta org, import blocks, generated + normalized HCL
entra/legacy/     brownfield-imported Entra objects, prevent_destroy on everything
entra/platform/   new shared Entra objects (groups, named locations, auth strengths, CA)
entra/waves/      per-wave app migrations
sailpoint/        SP-Config JSON and the mock-server pipeline
policy/           Conftest (OPA, Open Policy Agent) rules
.github/workflows/  snapshot, plan, apply, drift, change-gate
docs/             plan, analysis, findings-log, retro

## Commands
terraform fmt -recursive && terraform validate     # before every commit
conftest test --policy policy/ <plan.json>          # policy check on a plan
sail --version                                      # SailPoint CLI present

## Findings entry template
## YYYY-MM-DD, Phase N, short title
- What the docs said:
- What actually happened:
- Time taken:
- Evidence:
- Change to the client plan, if any:
- Accepted risks added, if any: