# Workflows (planned)

GitHub Actions only runs `.yml`/`.yaml` files here; this file only explains what is coming.

- `snapshot` — nightly and on demand: read-only Okta `terraform plan -detailed-exitcode`, counts to `docs/inventory-counts.md`, user/assignment JSON to Azure Blob only (Phase 1).
- `plan` — on pull request: `terraform plan` as `iac-planner` via OIDC (OpenID Connect), Conftest policy check on the plan JSON, plan posted to the PR (Phase 3).
- `apply` — on merge: apply as `iac-applier` to `nonprod`, then to `prod` after required-reviewer approval (Phase 3).
- `drift` — nightly `terraform plan -detailed-exitcode` on prod; exit code 2 opens or updates a GitHub issue, nothing else (Phase 3).
- `change-gate` — reads `Change: <system>:<id>` from the PR body, looks it up through that ITSM's adapter (lab: GitHub Issues), and fails unless the change is scheduled or implement and its window covers now (Phase 3). See docs/change-gate-design.md.
