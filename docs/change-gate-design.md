# Change Gate Design (ITSM-agnostic)

Decided 2026-10-01 after the ServiceNow Personal Developer Instance (PDI) sign-up had no instances available. The gate must work with any ITSM (IT Service Management) tool; the lab uses GitHub Issues as a stand-in.

## What the gate enforces
A pull request may apply to `prod` only if it names a change record that is:
1. In an approved-to-implement state (normalized `scheduled` or `implement`), and
2. Inside its planned window (`window_start <= now <= window_end`, UTC (Coordinated Universal Time)).

Otherwise the gate fails the job. A PR with no change reference also fails.

## Shape
- **Change reference in the PR body**: one line, `Change: <system>:<id>`. Examples: `Change: github:12`, `Change: servicenow:CHG0030001`, `Change: jira:ITSM-42`.
- **`change-gate` workflow step**: parses the reference, calls the adapter for `<system>`, gets back a normalized record, and checks state and window. The check logic is the same for every ITSM; only the adapter differs.
- **Normalized record** every adapter returns:

  | Field | Type | Meaning |
  |---|---|---|
  | `id` | string | Change ID in the source system |
  | `state` | `draft` / `assess` / `scheduled` / `implement` / `closed` / `cancelled` | Mapped from the tool's own states |
  | `window_start`, `window_end` | ISO 8601 UTC | Planned implementation window |
  | `url` | string | Link posted back to the PR |

- **Adapters**: one small script per ITSM, read-only, credentials from GitHub environment secrets or OIDC (OpenID Connect) where the tool supports it.

## Lab adapter: GitHub Issues
- Change record = an issue in this repo with label `change`.
- State = exactly one label: `change:draft`, `change:assess`, `change:scheduled`, `change:implement`, `change:closed`, `change:cancelled`.
- Window = two lines in the issue body: `window_start: 2026-10-20T01:00:00Z` and `window_end: 2026-10-20T03:00:00Z`.
- Access: the workflow's built-in `GITHUB_TOKEN` with `issues: read`. No new account, secret, or cost.
- Limit: anyone with write access can relabel an issue, so this proves the pipeline mechanics, not segregation of duties. A real ITSM supplies the approval workflow.

## Client adapters (not built in the lab)
| ITSM | Read call | State mapping to confirm with the client |
|---|---|---|
| ServiceNow | `GET /api/now/table/change_request?number=<id>` | `state` values for Scheduled and Implement; `start_date`, `end_date` |
| Jira Service Management | `GET /rest/api/3/issue/<id>` | Workflow status names; window custom fields |
| Others | Read-only REST call by ID | Same normalized record |

## Tests the lab must pass (Phase 3 exit)
- PR without `Change:` line fails.
- PR naming a `change:draft` issue fails.
- PR naming a `change:scheduled` issue outside its window fails.
- PR naming a `change:implement` issue inside its window passes.
