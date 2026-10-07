# Migration Execution Runbook (Okta to Entra, as code)

Built from what each Implementation Phase 0 (IP0) run teaches. Run 1 fills the steps in as they work; runs 2 and 3 tighten them. Each step points to the evidence in docs/findings-log.md.

Status legend: **Draft** (not run yet), **Run 1** (worked once), **Proven** (worked in two or more runs).

## 1. Prerequisites
| Step | Status | Notes |
|---|---|---|
| Source org reachable, admin with MFA (multi-factor authentication) | Draft | |
| Target tenant with break-glass account and `bg-ca-exclusion` group | Draft | |
| Entra ID P1/P2 licensing for CA (Conditional Access) | Draft | Lab uses the P2 trial |
| GitHub repo, environments `nonprod` and `prod`, reviewers | Draft | Public repo needed on the free plan |
| Tool versions pinned | Run 1 | docs/tool-install.md |

## 2. Identities and permissions
See docs/permission-register.md. Production column is the target.

## 3. Terraform state
| State key | Location | Holds | Who can read / write |
|---|---|---|---|
| `okta-inventory` | TBD | Okta users, groups, apps, policies (contains PII (personally identifiable information)) | TBD |
| `legacy` | TBD | Brownfield Entra objects | TBD |
| `platform` | TBD | Shared Entra objects | TBD |
| `waves/<wave>` | TBD | One wave's apps | TBD |

## 4. Inventory (source)
| Step | Status | Notes |
|---|---|---|
| Import blocks and `-generate-config-out` | Draft | |
| Normalize generated HCL (HashiCorp Configuration Language); time per app | Draft | |
| Nightly snapshot and drift | Draft | |

## 5. Per-app migration
### SAML (Security Assertion Markup Language) app
| Step | Status | Notes |
|---|---|---|
| Read Okta app settings and attribute statements from inventory | Draft | |
| Create Entra enterprise app (gallery template or custom) in a wave | Draft | |
| Claims: mapping policy vs custom claims policy | Draft | |
| Assign groups | Draft | |
| Exchange metadata with the SP (service provider) | Draft | |
| Test sign-in | Draft | |

### OIDC (OpenID Connect) app
| Step | Status | Notes |
|---|---|---|
| Read Okta client settings (redirect URIs, grant types, scopes, claims) | Draft | |
| Create Entra app registration + service principal | Draft | |
| Recreate custom claims and scopes | Draft | |
| Issue new client credentials to the app owner | Draft | |
| Test sign-in | Draft | |

## 6. Cutover wave
| Step | Status | Notes |
|---|---|---|
| Change record and PR (pull request) | Draft | |
| Plan, policy check, approval, apply | Draft | |
| Validation window | Draft | |
| Remove Okta assignments | Draft | |

## 7. Rollback
| Step | Status | Notes |
|---|---|---|
| Re-create Okta assignment; re-point SP to Okta | Draft | Time it |

## 8. Decommission
| Step | Status | Notes |
|---|---|---|
| Remove Okta app after the final validation window | Draft | |
