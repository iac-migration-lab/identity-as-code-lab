# Okta Seed Spec (Phase 1, step 1)

What to build in the Okta org `integrator-2631921` so it looks like a small enterprise "source" estate. Built outside Terraform on purpose: Phase 1 then imports it as brownfield. Built by `okta-inventory/seed/seed-okta-org.sh` (owner's choice, 2026-10-07) in run 1 with an SSWS API token (see docs/permission-register.md); anything the script cannot create is done by hand from this spec.

All people below are fictional. Passwords are set by the admin at creation and kept in the owner's password manager, never in this repo.

## Before seeding
- [ ] MFA (multi-factor authentication) enrolled on the Okta super admin (loose end from Phase 0).
- [ ] Note the start time; the findings entry needs total seeding time.

## Test targets (verified 2026-10-07)
- SAML (Security Assertion Markup Language) SP (service provider): IAM Showcase test SP. samltest.id is down.
  - ACS (Assertion Consumer Service) URL: `https://sptest.iamshowcase.com/acs`
  - Audience / SP entity ID: `IAMShowcase`
  - NameID format: `EmailAddress`. SP trusts every IdP (identity provider); no metadata upload needed. IdP-initiated sign-in only, unless set up through its AuthnRequest wizard.
  - Source: `https://sptest.iamshowcase.com/testsp_metadata.xml`
- OIDC (OpenID Connect) tester: `https://oidcdebugger.com/debug`. UNVERIFIED: still running; check before step 4.

Phase 2 warning: all SAML apps share the audience `IAMShowcase`. Okta allows that; Entra requires each app's identifier to be unique in the tenant (UNVERIFIED for SAML identifier URIs). Expect to sign in through Entra on only one of them at a time, or find a second tester. Log it as a finding when it bites.

## 1. Groups (3)
| Name | Description |
|---|---|
| `IT` | IT staff |
| `CustServ` | Customer service |
| `Finance` | Finance |

## 2. Users (8 active + 1 deactivated)
Free plan: 10 active users, admin counts as one. 8 active leaves one spare.

- Username (login): `first.last@iaclab.example`. `.example` is reserved (RFC 2606), so no real mailbox exists. UNVERIFIED: Okta accepts that TLD (top-level domain); if not, use `iaclab.example.com`.
- Primary email: `la-migration-lab+first.last@outlook.com` so any Okta email lands in the lab mailbox. UNVERIFIED: Okta accepts plus addresses.
- Create with **Activate now** and **I will set password**; untick "User must change password on first login".

| # | First | Last | Title | Department | employeeNumber | costCenter | Manager (login) | Groups |
|---|---|---|---|---|---|---|---|---|
| 1 | Ada | Reyes | IT Manager | IT | 1001 | CC-100 | (none) | IT |
| 2 | Ben | Okafor | Systems Engineer | IT | 1002 | CC-100 | ada.reyes | IT |
| 3 | Cara | Lind | Service Desk Analyst | IT | 1003 | CC-100 | ada.reyes | IT, CustServ |
| 4 | Dev | Patel | Customer Service Lead | CustServ | 2001 | CC-200 | (none) | CustServ |
| 5 | Eli | Moreno | Support Agent | CustServ | 2002 | CC-200 | dev.patel | CustServ |
| 6 | Fay | Novak | Support Agent | CustServ | 2003 | CC-200 | dev.patel | CustServ |
| 7 | Gus | Tanaka | Finance Manager | Finance | 3001 | CC-300 | (none) | Finance |
| 8 | Hana | Berg | Accountant | Finance | 3002 | CC-300 | gus.tanaka | Finance |
| 9 | Ivan | Cole | Former Accountant | Finance | 3003 | CC-300 | gus.tanaka | Finance, then **Deactivate** |

Why: user 3 is in two groups (overlapping access), user 9 is a leaver (must not migrate), managers exist for later SailPoint work. Fill manager by putting the manager's login in **managerId**.

## 3. Apps (6)
| Name | Type | Assigned to | Settings |
|---|---|---|---|
| `iac-saml-it-tools` | Custom SAML 2.0 | Group `IT` | ACS and audience from the test SP; NameID `EmailAddress`; app username `Okta username` |
| `iac-saml-custserv-desk` | Custom SAML 2.0 | Group `CustServ` | Same |
| `iac-saml-finance-ledger` | Custom SAML 2.0 | Group `Finance` | Same |
| `iac-saml-hr-portal` | Custom SAML 2.0 | Groups `IT`, `CustServ`, `Finance` | Same, plus attribute statements `department` = `user.department`, `title` = `user.title`, `employeeNumber` = `user.employeeNumber`; group attribute statement `groups`, filter Matches regex `.*` |
| `iac-oidc-portal` | OIDC Web app | Group `IT` | Authorization Code; sign-in redirect `https://oidcdebugger.com/debug` |
| Slack (OIN, Okta Integration Network) | Browse App Catalog, SAML 2.0 | Group `CustServ` | Workspace subdomain `iac-lab-placeholder`. Will not work; it exists to reproduce the `okta_app_saml` preconfigured-app drift noise |

## 4. Security objects
| Object | Name | Settings |
|---|---|---|
| Network zone (IP) | `iac-zone-office` | Gateway IPs `203.0.113.0/24` (documentation range, RFC 5737, no real traffic) |
| Password policy | `iac-pwd-staff` | Assigned to `IT`, `CustServ`, `Finance`; min length 12; one rule `iac-pwd-staff-rule` allowing self-service reset by email |
| Global session policy | `iac-gsp-staff` | Assigned to the 3 groups; rule `iac-gsp-staff-rule`: max session 8 hours, idle 2 hours |
| Authentication policy | `iac-authn-saml-apps` | Apply to the 4 custom SAML apps. Rule 1 `finance-2fa`: group `Finance`, any two factor types. Rule 2 `office-zone`: in zone `iac-zone-office`, Password. Catch-all: Password |
| Custom authorization server | `iac-authz-api` | Audience `api://iac-lab`; scope `iac.read`; claim `department` = `user.department` in access token; access policy `iac-authz-portal` for client `iac-oidc-portal`, rule allowing Authorization Code |

The authentication policy only exists in Identity Engine; if the Admin Console has **Security > Authentication Policies**, the org is Identity Engine.

## Expected counts after seeding
- Users: 9 (8 active, 1 deactivated) + admin
- Groups: 3 custom (+ built-in `Everyone`)
- Apps: 6 created (+ Okta built-in apps)
- Network zones: 1 custom; password policies: 1 custom; global session policies: 1 custom; authentication policies: 1 custom; authorization servers: 1 custom (+ `default`)

## Smoke test
1. Private window, sign in as `ada.reyes@iaclab.example`, open `iac-saml-it-tools` from the dashboard. The test SP page should show NameID `ada.reyes@iaclab.example`.
2. Same for `iac-saml-hr-portal`; check the `department`, `title`, `employeeNumber`, `groups` attributes.
3. Record results and the seeding time in docs/findings-log.md.
