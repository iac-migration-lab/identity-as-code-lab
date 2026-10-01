# Policy (planned, Phase 3)

Conftest (OPA, Open Policy Agent) rules run against `terraform show -json` plan output:

- Every CA (Conditional Access) policy excludes the break-glass group.
- Every new CA policy starts in report-only.
- No resource in `entra/legacy/` may be destroyed.

Run: `conftest test --policy policy/ <plan.json>`
