# Deliverables

What Baymax v1 delivers, grouped into milestones, with the acceptance criteria that say each one is done. Dates are not set here; they belong in `gantt_sched.md`. Scope comes from `prod_req_doc.md`, structure from `file_system.md`, behavior from `api_spec.md`, `data_model.md`, `critical_user_flow.md` and `app_flows.md`, screens from `ui_screens.md`, and backend processes from `backend_processes.md`.

Last updated: 2026-10-30.

## Table of Contents

01. [Definition of done](#definition-of-done)
02. [Milestone overview](#milestone-overview)
03. [M0. Design and decisions](#m0-design-and-decisions)
04. [M1. Foundation](#m1-foundation)
05. [M2. Authentication and tenants](#m2-authentication-and-tenants)
06. [M3. Core transactions](#m3-core-transactions)
07. [M4. Approvals, reports and dashboard](#m4-approvals-reports-and-dashboard)
08. [M5. Inventory, cash and release](#m5-inventory-cash-and-release)
09. [Later versions](#later-versions)
10. [Out of scope for v1](#out-of-scope-for-v1)
11. [Traceability to success metrics](#traceability-to-success-metrics)
12. [Dependencies and open decisions](#dependencies-and-open-decisions)

---

## Definition of done

A deliverable is done only when all of these hold (from the project workflow in `CLAUDE.md`):

01. It has a **design doc approved** through brainstorming before it was built (objects only).
02. The code is in the location given by `file_system.md`.
03. **Automated tests** pass: pytest against a real PostgreSQL (no mocks) and, for user-facing flows, Playwright.
04. A **manual test script** exists and has been run from a separate Windows machine on the same network.
05. Lint and type checks pass in CI.
06. Tenant-isolation tests covering the deliverable pass (target 100% for isolation paths).
07. Notes are saved under `00_docs/stage*/`, and the day's summary under `00_docs/summary/`.

---

## Milestone overview

| ID | Milestone | Outcome |
|---|---|---|
| M0 | Design and decisions | All design docs written; stack decided; test strategy rewritten |
| M1 | Foundation | Repo, tooling, CI, local environment, empty app skeleton running |
| M2 | Authentication and tenants | A SaaS Admin can onboard a tenant and its users can log in |
| M3 | Core transactions | Item lists and sales/expense/adjustment entry work end to end |
| M4 | Approvals, reports and dashboard | Change requests, roll-ups, dashboards and CSV export work |
| M5 | Inventory, cash and release | Supplies, cash on hand, hardening, and the first production tenant live |

Each milestone depends on the one before it. M3 is the first point at which the owner's own laundromat could start recording real transactions.

---

## M0. Design and decisions

| # | Deliverable | Location | Acceptance criteria |
|---|---|---|---|
| 0.1 | Product requirements | `00_docs/design/prod_req_doc.md` | Exists. **To finish:** Jobs To Be Done and Non-Goals sections filled; stray AthleteFlow text removed |
| 0.2 | API specification | `api_spec.md` | Written; reviewed against the data model |
| 0.3 | Architecture overview | `arch_overview.md` | Written; open decisions listed |
| 0.4 | Critical user flows | `critical_user_flow.md` | Written; acceptance checks usable as E2E cases |
| 0.5 | Data model | `data_model.md` | Written; assumptions confirmed by the owner |
| 0.6 | System design | `system_design.md` | Written |
| 0.7 | File structure | `file_system.md` | Exists; synced with `object_masterlist.md` (2026-10-30) |
| 0.8 | Deliverables list | `deliverables.md` | This document |
| 0.9 | Schedule | `gantt_sched.md` | Written from this document |
| 0.10 | Object master list | `object_masterlist.md` | Every model, service, schema, route and page named in `file_system.md` listed with owner, dependencies and status |
| 0.11 | Test strategy for Baymax | `test_strat.md` | Rewritten: tenant-per-database fixtures, Baymax roles and flows, coverage targets |
| 0.12 | **Stack decision** | `stack_comparison.md` (decision recorded) | FastAPI vs Node API decided; Auth0 vs Clerk decided; hosting and email provider chosen |
| 0.13 | Assumptions confirmed | `data_model.md` § Assumptions | Owner has confirmed or changed each of the six listed assumptions |
| 0.14 | UI screens | `ui_screens.md` | Written; every screen mapped to a page component, roles, flows and deliverable; open points resolved before M3 wireframes |
| 0.15 | Backend processes | `backend_processes.md` | Written; every process listed with its trigger and entry point; open points resolved before M2 |
| 0.16 | App flows | `app_flows.md` | Written; every user, operator, system and error flow listed; open points resolved or moved to a deliverable |

**Exit criteria:** no "open" decision remains that blocks M1.

---

## M1. Foundation

| # | Deliverable | Location | Acceptance criteria |
|---|---|---|---|
| 1.1 | Git repository | project root | Initialized with a `.gitignore`, a default branch and a first commit. (The folder isn't a repo yet.) |
| 1.2 | Backend skeleton | `backend/` | App starts; `/healthz` and `/readyz` respond; settings load from `.env` |
| 1.3 | Frontend skeleton | `frontend/` | Vite app builds and serves; proxies `/api` to the backend |
| 1.4 | Local environment | `docker-compose.yml`, `.env.example` | One command brings up Postgres, API and frontend; `.env.example` lists every variable |
| 1.5 | Migration setup | `backend/alembic/` | Platform and tenant trees exist; `-x tenant=<slug>` works |
| 1.6 | Test harness | `backend/tests/conftest.py` | Fixtures create the platform DB and two tenant DBs on a real Postgres and tear them down |
| 1.7 | CI pipeline | `.github/workflows/ci.yml` | Lint, type check, pytest, Playwright run on every pull request |
| 1.8 | Tooling configuration | `pyproject.toml`, `package.json` | Ruff, type checker, pytest, ESLint configured |
| 1.9 | README | `README.md` | A new developer can set up and run the app from it |
| 1.10 | Add commands to `CLAUDE.md` | `CLAUDE.md` | Build, lint, test and run commands recorded; "design phase" status updated |

---

## M2. Authentication and tenants

| # | Deliverable | Backend | Frontend | Acceptance criteria (flows) |
|---|---|---|---|---|
| 2.1 | Platform models and migrations | `models/platform/*`, `versions/platform/` | | Tables per `data_model.md`; CHECK constraints present |
| 2.2 | Security utilities | `utils/security.py` | | argon2id hashing, token generation and hashing; unit tested |
| 2.3 | Auth service (behind an interface) | `services/auth_service.py`, `api/v1/auth.py`, `schemas/auth.py` | `api/auth.ts`, `AuthContext`, `LoginPage` | Login, logout, `me`; lockout after 5 failures; sessions expire and are revocable (F2) |
| 2.4 | Password reset | same | `PasswordResetPage` | Single-use token; unknown email gets the same response; sessions ended on reset (F13) |
| 2.5 | Tenant DB registry | `tenant_db.py`, `dependencies.py` | | One cached engine per tenant; `get_tenant_db` reads tenant from session only |
| 2.6 | Tenant provisioning | `tenant_service.py`, `utils/db_provisioner.py`, `scripts/provision_tenant.py` | | Creates DB, role, migrations, settings, Tenant Admin; resumable after a failure (F1) |
| 2.7 | Tenant migrations runner | `scripts/migrate_all_tenants.py` | | Runs on every tenant; continues on failure; reports a summary |
| 2.8 | Tenant admin API and page | `api/v1/admin.py`, `schemas/tenant.py` | `TenantsAdminPage`, `api/admin.ts` | SaaS Admin creates, lists, suspends tenants; sees `failed` and retries (F1) |
| 2.9 | Tenant models: users, settings | `models/tenant/user.py` | | Created by tenant migration |
| 2.10 | User management | `user_service.py`, `api/v1/users.py`, `schemas/user.py` | `UsersPage`, `api/users.ts` | 5-active-user limit under concurrency; last Tenant Admin protected; deactivation ends sessions (F3) |
| 2.11 | App shell and role routing | | `AppShell`, `useAuth` | Correct home screen per role; guards on every route; responsive navigation |
| 2.12 | Email sending | | | Set-password and reset emails delivered by the chosen provider (a console/file sender in development) |
| 2.13 | **Tenant isolation test suite** | `tests/integration/test_tenant_isolation.py`, `e2e/tenant-isolation.spec.ts` | | A tenant A session cannot read or write tenant B data by any route; 100% of isolation paths covered |
| 2.14 | First SaaS Admin bootstrap | `scripts/create_saas_admin.py` | | An operator creates the first `saas_admin` identity in a new environment and receives a set-password link; refuses to run twice for the same email (O7) |

**Milestone test:** a SaaS Admin creates a tenant; its Tenant Admin sets a password, logs in, adds an owner and a staff user; a second tenant exists and cannot be reached from the first.

---

## M3. Core transactions

| # | Deliverable | Backend | Frontend | Acceptance criteria (flows) |
|---|---|---|---|---|
| 3.1 | Items and pricing | `models/tenant/item.py`, `item_service.py`, `api/v1/items.py`, `schemas/item.py` | `ItemMaintenancePage`, `api/items.ts` | Three lists; fixed and variable items; only the owner can change items or prices; price change is audited and doesn't alter past transactions (F4) |
| 3.2 | Starter item lists | provisioning seed | | A new tenant starts with the default sales and expense items from the PRD (owner can change them) |
| 3.3 | Audit log | `models/tenant/audit_log.py` | | Append-only; application role cannot update or delete it |
| 3.4 | Transactions | `models/tenant/transaction.py`, `transaction_line.py`, `transaction_service.py`, `api/v1/transactions.py`, `schemas/transaction.py` | `api/transactions.ts` | Sales, expense, adjustment; server prices fixed items; exact decimal totals; idempotent retries; no edit or delete route (F5, F6, F8) |
| 3.5 | Pricing and rounding rules | `transaction_service` | | Unit tests for fixed vs variable, rounding half-up, category mismatch, date limits |
| 3.6 | Entry screen | | `TransactionEntryPage`, `ItemPicker`, `TransactionLineList`, `PriceInput` | A typical fixed-price sale is saved in three taps plus Save with no typing; works on phone, tablet and desktop; unsaved state survives a failed request (F5) |
| 3.7 | Transactions list | | `TransactionsListPage`, `StatusBadge`, `EmptyState` | Filter by date, type, status, user; all roles can view |
| 3.8 | E2E tests | | `item-maintenance.spec.ts`, `transaction-entry.spec.ts` | Pass in CI |
| 3.9 | Seed data for E2E | | `e2e/fixtures/seed.py` | Seeds three tenants, all roles, item lists and a week of transactions |

**Milestone test:** the owner sets up items; an employee records a day of sales and expenses on a phone; totals match a hand calculation. This is the point at which the owner's laundromat can begin a trial alongside the paper logbook.

---

## M4. Approvals, reports and dashboard

| # | Deliverable | Backend | Frontend | Acceptance criteria (flows) |
|---|---|---|---|---|
| 4.1 | Change requests | `models/tenant/change_request.py`, `change_request_service.py`, `api/v1/change_requests.py`, `schemas/change_request.py` | `api/changeRequests.ts` | Staff file void/correct requests with a reason; one pending per transaction; requester can't approve; approve voids the original and creates the replacement atomically (F7) |
| 4.2 | Pending staff adjustments | `transaction_service`, `api/v1/transactions.py` | | Excluded from totals until approved; approve and reject routes (F8) |
| 4.3 | Approvals page | | `ApprovalsPage` | Single queue showing requester, reason and old vs new totals; reject requires a note (F9) |
| 4.4 | Roll-up reports | `report_service.py`, `api/v1/reports.py`, `schemas/report.py`, `tests/unit/test_rollup_periods.py` | `api/reports.ts` | Daily, weekly, monthly and half-year roll-ups match the underlying transactions; period boundaries tested including leap years and week straddling months |
| 4.5 | Dashboard | | `DashboardPage`, `PeriodSelector`, `RollupChart` | Sales, expenses, net for the chosen period; refreshes automatically; usable on a phone (F11) |
| 4.6 | Expense dashboard | | `ExpensesDashboardPage` | Expense totals by item and period |
| 4.7 | CSV export | `report_service` export, `GET /reports/export` | `ReportsPage` | Row-per-line file opens in Excel; totals equal on-screen figures; formula-injection protection; 366-day cap (F12) |
| 4.8 | Audit log view | `GET /reports/audit-log` | `AuditLogPage` | Owner and Tenant Admin filter by actor, entity, action, date; entries show before/after values; read-only (F36) |
| 4.9 | E2E tests | | `approval-flow.spec.ts`, `dashboard.spec.ts` | Pass in CI |

**Milestone test:** an employee files a correction; the owner approves it on a phone; the daily, weekly and monthly dashboards change accordingly; the month exports to CSV and its total equals the dashboard.

---

## M5. Inventory, cash and release

| # | Deliverable | Backend | Frontend | Acceptance criteria (flows) |
|---|---|---|---|---|
| 5.1 | Inventory | `models/tenant/inventory_item.py`, `inventory_movement.py`, `inventory_service.py`, `api/v1/inventory.py`, `schemas/inventory.py` | `InventoryPage`, `api/inventory.ts` | Stock derived from movements; low-stock warning; cannot use more than is in stock (F10) |
| 5.2 | Cash on hand | `models/tenant/cash_count.py`, `cash_service.py`, `api/v1/cash.py`, `schemas/cash.py` | `CashOnHandPage`, `api/cash.ts` | Expected cash computed server-side; variance shown; append-only; one count per date and shift (F10) |
| 5.3 | Security review | | | Items in `system_design.md` § Security design verified; rate limits, CSRF, cookie flags, secrets handling checked |
| 5.4 | Performance check | | | Targets in `system_design.md` § Performance budget met on a year of synthetic data per tenant |
| 5.5 | Backups, restore drill and nightly checks | `scripts/backup_all.py`, `scripts/restore_tenant.py`, `scripts/check_orphan_identities.py` | | Nightly dump of platform and each tenant DB; a tenant restored from backup into a clean server; nightly orphan-identity report runs (K6) |
| 5.6 | Production environment | | | Hosting, database, domain, TLS, email provider, error tracking configured; `staging` seeded |
| 5.7 | Observability | | | Structured logs with request id, tenant and user; health checks; error tracker receiving events |
| 5.8 | Runbooks | `00_docs/` | | Restore one tenant, re-run a failed tenant migration, revoke all sessions, rotate the DB role secret |
| 5.9 | Manual test pass | `00_docs/` | | All manual test scripts executed on a Windows machine on the same network; results recorded |
| 5.10 | **First tenant live** | | | The owner's laundromat is onboarded in production and has recorded real transactions for a full week alongside the logbook with matching totals |

**Release criteria:** all success metrics in `prod_req_doc.md` are met (see the table below) and there are no open critical or high defects.

---

## Later versions

Listed so v1 does not block them. Not delivered in v1.

| Version | Deliverable |
|---|---|
| v1.x | Auth0 migration; PWA (installable on the branch tablet); `.xlsx` export; email notifications for pending approvals; Stripe billing for paying tenants |
| v2 | Full bookkeeping: chart of accounts, journal entries, salary, capital and operating costs, fiscal-year option |
| v3 | POS integration (device keys, ingest endpoint); import of machine cycle counts from LG washers and a reconciliation report against sales |

---

## Out of scope for v1

- Full bookkeeping or accounting-software integration
- POS integration and machine cycle-count ingestion
- Card, e-wallet or other non-cash payment tracking
- Salary projection and expense forecasting
- Offline mode with a local queue
- Multiple branches inside one tenant (one tenant is one laundromat for now)
- Billing and subscription management
- Native mobile apps (the responsive web app is the mobile experience)
- Tenant deletion and data offboarding (tenants are suspended, not dropped)

---

## Traceability to success metrics

From `prod_req_doc.md`:

| Success metric | Delivered by |
|---|---|
| KPI 1: multiple tenants and authentication working | 2.3, 2.6, 2.8, 2.10, 2.13 |
| KPI 2: transactions recorded, fixed and variable price | 3.4, 3.5, 3.6 |
| KPI 3: items added onto transactions | 3.4, 3.6 |
| KPI 4: transaction items maintained (fixed price) | 3.1, 3.2 |
| KPI 5: dashboard present | 4.5, 4.6 |

PRD "Must have" items not covered by a KPI: reports (4.4, 4.7), inventory and supplies (5.1), cash on hand (5.2).

---

## Dependencies and open decisions

| Item | Needed by | Notes |
|---|---|---|
| Stack decision (FastAPI vs Node, Auth0 vs Clerk) | Start of M1 | Deliverable paths above assume the FastAPI layout in `file_system.md` |
| Hosting provider for API and Postgres | M5 (start of 5.6) | Must support long-running containers |
| Email provider | M2 (2.12) | A development sender is enough until M5 |
| Owner's confirmation of the six data-model assumptions | M3 | Especially variable pricing roles and adjustment approval |
| Starter item lists and prices for the first tenant | M3 | Needed to seed and to run the trial |
| A second device (Windows machine on the same network) for manual tests | Every milestone | Required by the project workflow |
| `test_strat.md` rewrite | Before M2 tests | Still describes AthleteFlow |
| How provisioning runs after `202` (in-process task or job queue) | M2 (2.6) | `backend_processes.md` open point 1; tied to the background-jobs decision |
| Rollback rule for user creation across platform and tenant DBs | M2 (2.10) | `backend_processes.md` open point 2 |
| Email failure handling and a "resend set-password link" action | M2 (2.10, 2.12) | `backend_processes.md` open point 3, `app_flows.md` open point 3 |
| Scheduler for nightly jobs (backups, orphan check, expired session cleanup) | M5 (5.5) | Depends on the hosting provider |
| Whether owners and Tenant Admins can void or correct their own entries | M4 (4.1) | `app_flows.md` open point 2; today only staff can file change requests |
| Corrected cash count vs the one-count-per-date-and-shift rule | M5 (5.2) | `app_flows.md` open point 6 |
| Phone navigation for owner and Tenant Admin (9–10 items) | M2 (2.11) | `ui_screens.md` open point 3 |
