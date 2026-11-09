# Daily summary: 2026-11-05

Covers work since the last summary (2026-11-03). The project is in **M0 Design and decisions** (2–13 Nov 2026). No code exists yet.

## Objects built

No code objects. No design doc was completed today.

| Item | Deliverable | What happened | Status |
|---|---|---|---|
| `01_research/stack_comparison.md` | 0.12 | The "Stack Decision" tables were pasted at the bottom, copied from `stack_working.md`. The review fixes were not applied. | 🔨 Draft recorded, not signed off |
| `01_research/stack_working.md` | 0.12 | Auth now reads **Zitadel** (was Auth0). Added an "Admin Console" row: DBeaver for now, maybe Directus later. | 🔨 Working draft, uncommitted |
| `00_planning/test_strat.md` | 0.11 | Title and a few lines changed earlier; the body is still AthleteFlow. | Superseded by the draft below |
| `00_planning/20261105_test_strat_draft.md` | 0.11 | Full Baymax rewrite: priorities, four roles, unit/integration/database-level/E2E/manual levels, tenant isolation checklist, F1–F13 test matrix, critical scenarios, performance targets, coverage targets, fixtures, CI timing. Not yet swapped into `test_strat.md`. | 🔨 Draft, awaiting review |
| `00_planning/prod_req_doc.md` | 0.1 | Removed the empty "Jobs To Be Done" heading and added one line under Non-Goals. Neither section is written. | 🔨 Partly edited |

| `02_design/deliverables.md` | M2 | Added deliverable **2.15 Dev seed script** (`scripts/seed_dev.py`): first SaaS Admin, a dev tenant via `TenantService`, one user per role; re-runnable; refuses to run in production. 3.9 (E2E seed) now builds on it. Also listed in `file_system.md` and `object_masterlist.md`. | ✅ Design recorded; script not built |

### Decisions and discussion

- **Admin console.** I compared prebuilt options that run in Docker. Directus is my pick for later (light, works on the existing Postgres). Appsmith is the heavier fallback. NocoDB is the simplest and has weaker roles. You chose **DBeaver for now**.
  - I advised using DBeaver only as the owner's own tool, with tenant onboarding as a provisioning script or CLI. DBeaver bypasses FastAPI validation and the audit log. This is not yet written into the stack docs.
  - **Update (later on 5 Nov):** you edited `stack_comparison.md` so the two rows no longer conflict. DBeaver is the developer's own tool. The in-app SaaS Admin console (React section on FastAPI routes, with Retool or Appsmith as options) is "provisioned to prod admin and not during dev". This is the Admin Console decision. Still open: how tenants get onboarded during dev (a provisioning script is my suggestion, not yet recorded), and Directus/Appsmith are only noted as "maybe in future".
- **Dev onboarding.** Provisioning logic is already planned in M2 (2.6, `provision_tenant.py`). The new seed script (2.15) is a thin wrapper over it plus 2.14, so the M2 dates don't move. A low-likelihood risk row (+1 day) was added to the schedule. The design rule: services hold the logic; the CLI scripts and the future admin endpoint are thin callers.
- **Auth.** Zitadel replaces Auth0 in the table. The PRD says DB auth at launch with Auth0 later, so the row still needs an `AuthProvider` interface note and a check against the PRD.

- **Stack accepted as is (5 Nov).** You decided the stack proceeds without the review fixes. 0.12 is closed; the deferred items are listed in `stack_comparison.md`. This means Zitadel stays as the Auth pick for now, and the Auth0/`AuthProvider` conflict with the PRD is unresolved.

### Testing status

Nothing to test yet. The draft test strategy adds suggestions not in the stack decision (Vitest, template-database fixtures, two extra integration test files), marked "(my call)" for review. Test tooling in the stack draft is pytest + httpx + Testcontainers + Playwright.

### Where this fits

| Deliverable | State |
|---|---|
| 0.12 Stack decision | ✅ Accepted as is on 5 Nov (due 6 Nov). Review fixes deferred |
| 0.11 Test strategy | Full draft written; needs your review, then replaces `test_strat.md` |
| 0.1 PRD gaps | Partly edited |
| 0.13 Owner assumptions | Not sent |

### Housekeeping

- `CLAUDE.md`, the skills and the masterlist paths still use the old `00_docs/design/`, `summary/` and `todos/` folders. The skills' prerequisites list files that now live in other folders.
- Uncommitted: `stack_working.md`, `tools.md`, `stack_comparison.md`, `prod_req_doc.md`, `test_strat.md`, and the schedule and masterlist updates.
- The skill told me to place the summary in `stage*/`. I put it in `03_summary/`, where earlier summaries live.
