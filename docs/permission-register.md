# Permission Register

Every grant made in Implementation Phase 0 (IP0), written at the time of the grant. Lab grants may be broad; this register shows what production must do differently.

Verdicts: **OK** (use as is in production), **Changes** (OK with the change noted), **Not OK** (do not use in production).

| # | Date | Run | Identity | Granted | Why | Verdict | Production should use |
|---|---|---|---|---|---|---|---|
| 1 | 2026-10-01 | 1 | `gh` CLI login (owner's GitHub account) | `delete_repo` OAuth scope | Delete and recreate the repo to purge old commit emails | Not OK | No standing `delete_repo` on any login; repo deletion through an org owner in the browser, with a change record. Remove after use (loose end). |
| 2 | 2026-10-01 | 1 | `la-migration-lab@outlook.com` via `az login` | Subscription Owner (sign-up default), Global Administrator (tenant creator default) | Budget alert, read-only tenant and subscription checks | Not OK | Named admin accounts with PIM (Privileged Identity Management) eligible roles; budget created by the pipeline identity with Cost Management Contributor on the subscription only. |
| 3 | 2026-10-01 | 1 | Okta super admin `mark@lencioni.io` | Super Administrator (org creator default) | Okta org owner | Not OK | Okta admin on a domain not tied to any production tenant; Super Admin only for break-glass; day-to-day admins on narrower roles. |
| 4 | 2026-10-07 | 1 | Okta SSWS API token `iac-run1-seed` (created by the super admin, file `~/.okta-lab/ssws-token`) | Everything the super admin can do (SSWS tokens inherit the creator's roles; no scopes) | Seed the org and bootstrap the Okta service apps fast in run 1 | Not OK | No SSWS tokens. Seeding is not a production step (the client org already exists); service apps are created by an admin, and automation uses OAuth service apps with private key JWT and narrow scopes. Revoke at end of run 1. |
