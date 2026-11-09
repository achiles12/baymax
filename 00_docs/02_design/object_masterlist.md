# Object Masterlist: Baymax v1.0

Last updated: 2026-11-05 (0.11 test strategy draft written; 0.12 stack accepted as is; added `seed_dev.py`, 2.15; 0.1, 0.11 and 0.12 status refreshed; no objects built). Previous: 2026-11-03 (0.12 status refreshed). 2026-10-30 (added design docs 0.14–0.16, the first SaaS Admin script, `ReportsPage`, `AuditLogPage` and the nightly orphan check. Nothing built yet)

Every object that `file_system.md` plans, grouped by stage. Stages match the milestones in `deliverables.md` (Stage N = M·N), and the **Deliv.** column gives the deliverable number there. Update the Status and Date columns as each object is built, following the rule that an object gets an approved design doc before it is built.

**Status key:** ⬜ Not started · 🟡 Design approved · 🔨 In progress · ✅ Complete · ⛔ Blocked

**Build order inside a stage:** utilities → models → migrations → services → dependencies → schemas → routes → frontend → tests.

Stack-specific files (Python/FastAPI/React) follow the baseline in `file_system.md`. If the stack decision (see `stack_comparison.md`) changes, update the File column; the objects themselves stay the same.

## Contents

- [Stage 0: Design documents](#stage-0--design-documents)
- [Stage 1: Project setup and infrastructure](#stage-1--project-setup-and-infrastructure)
- [Stage 2: Authentication and tenants](#stage-2--authentication-and-tenants)
- [Stage 3: Core transactions](#stage-3--core-transactions)
- [Stage 4: Approvals, reports and dashboard](#stage-4--approvals-reports-and-dashboard)
- [Stage 5: Inventory, cash and release](#stage-5--inventory-cash-and-release)
- [Counts](#counts)

---

## Stage 0: Design documents

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| Product requirements | Doc | `00_docs/design/prod_req_doc.md` | 0.1 | 🔨 Exists; Jobs To Be Done heading removed (not written), Non-Goals has one placeholder line, stray AthleteFlow text not yet checked | |
| API specification | Doc | `00_docs/design/api_spec.md` | 0.2 | ✅ Complete | 2026-10-29 |
| Architecture overview | Doc | `00_docs/design/arch_overview.md` | 0.3 | ✅ Complete | 2026-10-29 |
| Critical user flows | Doc | `00_docs/design/critical_user_flow.md` | 0.4 | ✅ Complete | 2026-10-29 |
| Data model | Doc | `00_docs/design/data_model.md` | 0.5 | ✅ Complete | 2026-10-29 |
| System design | Doc | `00_docs/design/system_design.md` | 0.6 | ✅ Complete | 2026-10-29 |
| File structure | Doc | `00_docs/design/file_system.md` | 0.7 | ✅ Complete (synced with this list) | 2026-10-30 |
| Deliverables | Doc | `00_docs/design/deliverables.md` | 0.8 | ✅ Complete | 2026-10-29 |
| Schedule | Doc | `00_docs/design/gantt_sched.md` | 0.9 | ✅ Complete (progress tracked from 2026-10-30) | 2026-10-29 |
| Object masterlist | Doc | `00_docs/design/object_masterlist.md` | 0.10 | ✅ Complete | 2026-10-29 |
| Test strategy (rewrite for Baymax) | Doc | `00_docs/design/test_strat.md` | 0.11 | 🔨 Full draft written 2026-11-05 (`00_planning/20261105_test_strat_draft.md`); awaiting review, then replaces `test_strat.md` | |
| Stack comparison and decision | Doc | `00_docs/design/stack_comparison.md` (working draft: `00_docs/01_research/stack_working.md`) | 0.12 | ✅ Accepted as is (2026-11-05); review fixes deferred, see `stack_comparison.md` | 2026-11-05 |
| Data-model assumptions confirmed | Doc | `00_docs/design/data_model.md` § Assumptions | 0.13 | ⬜ Not started (needs the owner) | |
| UI screens | Doc | `00_docs/design/ui_screens.md` | 0.14 | 🔨 Written; open points pending | 2026-10-30 |
| Backend processes | Doc | `00_docs/design/backend_processes.md` | 0.15 | 🔨 Written; open points pending | 2026-10-30 |
| App flows | Doc | `00_docs/design/app_flows.md` | 0.16 | 🔨 Written; open points pending | 2026-10-30 |

---

## Stage 1: Project setup and infrastructure

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| Git repository, `.gitignore` | Infra | project root | 1.1 | ⬜ Not started | |
| Docker Compose stack (Postgres + API + frontend) | Infra | `docker-compose.yml` | 1.4 | ⬜ Not started | |
| Environment template | Config | `.env.example` | 1.4 | ⬜ Not started | |
| FastAPI skeleton + `GET /healthz`, `GET /readyz` | Route | `backend/app/main.py` | 1.2 | ⬜ Not started | |
| Settings (platform DB URL, tenant DB URL template) | Config | `backend/app/config.py` | 1.2 | ⬜ Not started | |
| Platform engine + session factory | Infra | `backend/app/database.py` | 1.2 | ⬜ Not started | |
| Backend project config (deps, ruff, type checker, pytest) | Config | `backend/pyproject.toml` | 1.8 | ⬜ Not started | |
| Backend Dockerfile | Infra | `backend/Dockerfile` | 1.4 | ⬜ Not started | |
| Alembic runner (platform + tenant targets) | Config | `backend/alembic/env.py`, `alembic.ini` | 1.5 | ⬜ Not started | |
| Test fixtures: platform DB, two tenant DBs, seeded users per role | Test Fixture | `backend/tests/conftest.py` | 1.6 | ⬜ Not started | |
| Vite + React + Tailwind scaffold | Frontend | `frontend/` (`index.html`, `vite.config.ts`, `tailwind.config.ts`, `tsconfig.json`, `package.json`, `src/main.tsx`, `src/App.tsx`) | 1.3 | ⬜ Not started | |
| Base API client (attaches CSRF header, handles 401) | Frontend | `frontend/src/api/client.ts` | 1.3 | ⬜ Not started | |
| Shared TypeScript types | Frontend | `frontend/src/types/index.ts` | 1.3 | ⬜ Not started | |
| GitHub Actions CI | CI | `.github/workflows/ci.yml` | 1.7 | ⬜ Not started | |
| README | Doc | `README.md` | 1.9 | ⬜ Not started | |
| Update `CLAUDE.md` with build/lint/test commands | Doc | `CLAUDE.md` | 1.10 | ⬜ Not started | |

---

## Stage 2: Authentication and tenants

### Utilities

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `hash_password`, `verify_password`, `generate_token`, `hash_token` | Util | `backend/app/utils/security.py` | 2.2 | ⬜ Not started | |
| `derive_slug`, `slug_to_db_name`, reserved-slug check | Util | `backend/app/utils/tenant.py` | 2.6 | ⬜ Not started | |
| `create_database`, `create_role`, `run_tenant_migrations` | Util | `backend/app/utils/db_provisioner.py` | 2.6 | ⬜ Not started | |

### Models (platform database)

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `Tenant` | Model | `backend/app/models/platform/tenant.py` | 2.1 | ⬜ Not started | |
| `UserIdentity` | Model | `backend/app/models/platform/user_identity.py` | 2.1 | ⬜ Not started | |
| `Session` | Model | `backend/app/models/platform/session.py` | 2.1 | ⬜ Not started | |
| `PasswordResetToken` | Model | `backend/app/models/platform/password_reset_token.py` | 2.1 | ⬜ Not started | |

### Models (tenant database)

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `TenantSettings` | Model | `backend/app/models/tenant/tenant_settings.py` | 2.9 | ⬜ Not started | |
| `User` | Model | `backend/app/models/tenant/user.py` | 2.9 | ⬜ Not started | |

### Migrations

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| Platform migration `0001` | Migration | `backend/alembic/versions/platform/0001_create_platform_tables.py` | 2.1 | ⬜ Not started | |
| Tenant migration `0001` (settings, users) | Migration | `backend/alembic/versions/tenant/0001_create_users_settings.py` | 2.9 | ⬜ Not started | |

### Services

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `AuthService` behind an auth-provider interface (login, logout, get_identity, reset request + confirm, `provision_identity`) | Service | `backend/app/services/auth_service.py` | 2.3, 2.4 | ⬜ Not started | |
| `TenantService` (provision resumable, list, get, retry, update status) | Service | `backend/app/services/tenant_service.py` | 2.6, 2.8 | ⬜ Not started | |
| `UserService` (create with 5-user limit, update, deactivate, last-admin guard) | Service | `backend/app/services/user_service.py` | 2.10 | ⬜ Not started | |
| Email sender (console in dev, provider in prod) | Service | `backend/app/services/email_service.py` | 2.12 | ⬜ Not started | |

### Dependencies / middleware

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| Tenant engine registry (LRU, `get_tenant_session`) | Infra | `backend/app/tenant_db.py` | 2.5 | ⬜ Not started | |
| `get_current_user` (session + role read from tenant DB) | Dependency | `backend/app/dependencies.py` | 2.3 | ⬜ Not started | |
| `get_tenant_db` | Dependency | `backend/app/dependencies.py` | 2.5 | ⬜ Not started | |
| `require_role(...)` | Dependency | `backend/app/dependencies.py` | 2.3 | ⬜ Not started | |
| CSRF check | Dependency | `backend/app/dependencies.py` | 2.3 | ⬜ Not started | |
| Rate limiter (login, reset) | Middleware | `backend/app/main.py` | 2.3 | ⬜ Not started | |

### Schemas

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `LoginRequest`, `LoginResponse` | Schema | `backend/app/schemas/auth.py` | 2.3 | ⬜ Not started | |
| Reset password request and confirm bodies | Schema | `backend/app/schemas/auth.py` | 2.4 | ⬜ Not started | |
| `ProvisionTenantRequest`, `TenantOut` | Schema | `backend/app/schemas/tenant.py` | 2.8 | ⬜ Not started | |
| `UserCreate`, `UserUpdate`, `UserOut` | Schema | `backend/app/schemas/user.py` | 2.10 | ⬜ Not started | |

### Routes

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `POST /auth/login`, `POST /auth/logout`, `GET /auth/me` | Route | `backend/app/api/v1/auth.py` | 2.3 | ⬜ Not started | |
| `POST /auth/reset-password/request`, `/confirm` | Route | `backend/app/api/v1/auth.py` | 2.4 | ⬜ Not started | |
| `GET/POST /admin/tenants`, `GET/PATCH /admin/tenants/{id}`, `POST …/retry-provisioning` | Route | `backend/app/api/v1/admin.py` | 2.8 | ⬜ Not started | |
| `GET/POST /users`, `PATCH /users/{id}` | Route | `backend/app/api/v1/users.py` | 2.10 | ⬜ Not started | |
| `v1_router` aggregator | Router | `backend/app/api/v1/router.py` | 2.3 | ⬜ Not started | |

### Scripts

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `provision_tenant.py` | Script | `scripts/provision_tenant.py` | 2.6 | ⬜ Not started | |
| `migrate_all_tenants.py` | Script | `scripts/migrate_all_tenants.py` | 2.7 | ⬜ Not started | |
| `create_saas_admin.py` (first SaaS Admin) | Script | `scripts/create_saas_admin.py` | 2.14 | ⬜ Not started | |
| `seed_dev.py` (dev seed: SaaS Admin, dev tenant, one user per role) | Script | `scripts/seed_dev.py` | 2.15 | ⬜ Not started | |

### Frontend

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| Auth API calls | API client | `frontend/src/api/auth.ts` | 2.3 | ⬜ Not started | |
| Tenant admin API calls | API client | `frontend/src/api/admin.ts` | 2.8 | ⬜ Not started | |
| User API calls | API client | `frontend/src/api/users.ts` | 2.10 | ⬜ Not started | |
| `AuthContext`, `useAuth` | Context / Hook | `frontend/src/contexts/AuthContext.tsx`, `frontend/src/hooks/useAuth.ts` | 2.3 | ⬜ Not started | |
| `AppShell` (responsive nav, guards, role routes) | Layout | `frontend/src/layouts/AppShell.tsx` | 2.11 | ⬜ Not started | |
| `LoginPage` | Page | `frontend/src/pages/LoginPage.tsx` | 2.3 | ⬜ Not started | |
| `PasswordResetPage` (also used for set-password) | Page | `frontend/src/pages/PasswordResetPage.tsx` | 2.4 | ⬜ Not started | |
| `TenantsAdminPage` | Page | `frontend/src/pages/TenantsAdminPage.tsx` | 2.8 | ⬜ Not started | |
| `UsersPage` | Page | `frontend/src/pages/UsersPage.tsx` | 2.10 | ⬜ Not started | |

### Tests

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `test_security` | Unit | `backend/tests/unit/test_security.py` | 2.2 | ⬜ Not started | |
| `test_tenant_utils` | Unit | `backend/tests/unit/test_tenant_utils.py` | 2.6 | ⬜ Not started | |
| `test_role_checks` | Unit | `backend/tests/unit/test_role_checks.py` | 2.3 | ⬜ Not started | |
| `test_auth` | Integration | `backend/tests/integration/test_auth.py` | 2.3, 2.4 | ⬜ Not started | |
| `test_tenant_isolation` | Integration | `backend/tests/integration/test_tenant_isolation.py` | 2.13 | ⬜ Not started | |
| `test_tenant_provisioning` | Integration | `backend/tests/integration/test_tenant_provisioning.py` | 2.6 | ⬜ Not started | |
| `test_users` | Integration | `backend/tests/integration/test_users.py` | 2.10 | ⬜ Not started | |
| `auth.spec.ts` | E2E | `frontend/e2e/auth.spec.ts` | 2.3, 2.4 | ⬜ Not started | |
| `tenant-onboarding.spec.ts` | E2E | `frontend/e2e/tenant-onboarding.spec.ts` | 2.8 | ⬜ Not started | |
| `user-creation.spec.ts` | E2E | `frontend/e2e/user-creation.spec.ts` | 2.10 | ⬜ Not started | |
| `tenant-isolation.spec.ts` | E2E | `frontend/e2e/tenant-isolation.spec.ts` | 2.13 | ⬜ Not started | |
| Manual test script, Stage 2 | Manual | `00_docs/stage2/manual_test_stage2.md` | all | ⬜ Not started | |

---

## Stage 3: Core transactions

### Models and migration

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `Item` | Model | `backend/app/models/tenant/item.py` | 3.1 | ⬜ Not started | |
| `AuditLog` | Model | `backend/app/models/tenant/audit_log.py` | 3.3 | ⬜ Not started | |
| `Transaction` | Model | `backend/app/models/tenant/transaction.py` | 3.4 | ⬜ Not started | |
| `TransactionLine` | Model | `backend/app/models/tenant/transaction_line.py` | 3.4 | ⬜ Not started | |
| Tenant migration `0002` (items, audit_log with restricted grants, transactions, lines) | Migration | `backend/alembic/versions/tenant/0002_items_transactions_audit.py` | 3.1–3.4 | ⬜ Not started | |

### Services

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `ItemService` (CRUD per category, owner-only prices, audited price change) | Service | `backend/app/services/item_service.py` | 3.1 | ⬜ Not started | |
| Starter item seed on provisioning | Service | `backend/app/services/tenant_service.py` (extend) | 3.2 | ⬜ Not started | |
| `AuditService` / audit helper | Service | `backend/app/services/audit.py` | 3.3 | ⬜ Not started | |
| `TransactionService` (create with pricing, rounding, idempotency, status rules) | Service | `backend/app/services/transaction_service.py` | 3.4, 3.5 | ⬜ Not started | |

### Schemas and routes

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `ItemCreate`, `ItemUpdate`, `ItemOut` | Schema | `backend/app/schemas/item.py` | 3.1 | ⬜ Not started | |
| `TransactionCreate`, `TransactionOut`, `TransactionLineOut` | Schema | `backend/app/schemas/transaction.py` | 3.4 | ⬜ Not started | |
| `GET/POST /items`, `PATCH /items/{id}` | Route | `backend/app/api/v1/items.py` | 3.1 | ⬜ Not started | |
| `POST /transactions`, `GET /transactions`, `GET /transactions/{id}`, `GET /transactions/{id}/lines` | Route | `backend/app/api/v1/transactions.py` | 3.4 | ⬜ Not started | |

### Frontend

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| Item API calls | API client | `frontend/src/api/items.ts` | 3.1 | ⬜ Not started | |
| Transaction API calls | API client | `frontend/src/api/transactions.ts` | 3.4 | ⬜ Not started | |
| `ItemPicker` | Component | `frontend/src/components/ItemPicker.tsx` | 3.6 | ⬜ Not started | |
| `TransactionLineList` | Component | `frontend/src/components/TransactionLineList.tsx` | 3.6 | ⬜ Not started | |
| `PriceInput` | Component | `frontend/src/components/PriceInput.tsx` | 3.6 | ⬜ Not started | |
| `StatusBadge` | Component | `frontend/src/components/StatusBadge.tsx` | 3.7 | ⬜ Not started | |
| `EmptyState` | Component | `frontend/src/components/EmptyState.tsx` | 3.7 | ⬜ Not started | |
| `ItemMaintenancePage` | Page | `frontend/src/pages/ItemMaintenancePage.tsx` | 3.1 | ⬜ Not started | |
| `TransactionEntryPage` | Page | `frontend/src/pages/TransactionEntryPage.tsx` | 3.6 | ⬜ Not started | |
| `TransactionsListPage` | Page | `frontend/src/pages/TransactionsListPage.tsx` | 3.7 | ⬜ Not started | |

### Tests

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `test_pricing` | Unit | `backend/tests/unit/test_pricing.py` | 3.5 | ⬜ Not started | |
| `test_items` | Integration | `backend/tests/integration/test_items.py` | 3.1 | ⬜ Not started | |
| `test_transactions` | Integration | `backend/tests/integration/test_transactions.py` | 3.4 | ⬜ Not started | |
| `item-maintenance.spec.ts` | E2E | `frontend/e2e/item-maintenance.spec.ts` | 3.8 | ⬜ Not started | |
| `transaction-entry.spec.ts` | E2E | `frontend/e2e/transaction-entry.spec.ts` | 3.8 | ⬜ Not started | |
| E2E seed | Fixture | `frontend/e2e/fixtures/seed.py` | 3.9 | ⬜ Not started | |
| Manual test script, Stage 3 | Manual | `00_docs/stage3/manual_test_stage3.md` | all | ⬜ Not started | |

---

## Stage 4: Approvals, reports and dashboard

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `ChangeRequest` | Model | `backend/app/models/tenant/change_request.py` | 4.1 | ⬜ Not started | |
| Tenant migration `0003` (change_requests, report indexes) | Migration | `backend/alembic/versions/tenant/0003_change_requests.py` | 4.1 | ⬜ Not started | |
| `ChangeRequestService` (file, approve, reject, cancel; atomic apply + audit) | Service | `backend/app/services/change_request_service.py` | 4.1 | ⬜ Not started | |
| Staff adjustment approve/reject | Service | `backend/app/services/transaction_service.py` (extend) | 4.2 | ⬜ Not started | |
| `ReportService` (`period_bounds`, roll-ups, expense summary, CSV export stream) | Service | `backend/app/services/report_service.py` | 4.4, 4.7 | ⬜ Not started | |
| `ChangeRequestCreate`, `ChangeRequestOut`, `ApproveRejectBody` | Schema | `backend/app/schemas/change_request.py` | 4.1 | ⬜ Not started | |
| `RollupQuery`, `RollupRow`, `ExpenseSummary` | Schema | `backend/app/schemas/report.py` | 4.4 | ⬜ Not started | |
| `GET/POST /change-requests`, `…/{id}/approve`, `/reject`, `/cancel` | Route | `backend/app/api/v1/change_requests.py` | 4.1 | ⬜ Not started | |
| `POST /transactions/{id}/approve`, `/reject` | Route | `backend/app/api/v1/transactions.py` (extend) | 4.2 | ⬜ Not started | |
| `GET /reports/rollup`, `/expenses`, `/export`, `/audit-log` | Route | `backend/app/api/v1/reports.py` | 4.4, 4.6–4.8 | ⬜ Not started | |
| Change request API calls | API client | `frontend/src/api/changeRequests.ts` | 4.1 | ⬜ Not started | |
| Report API calls (incl. CSV download) | API client | `frontend/src/api/reports.ts` | 4.4, 4.7 | ⬜ Not started | |
| `PeriodSelector` | Component | `frontend/src/components/PeriodSelector.tsx` | 4.5 | ⬜ Not started | |
| `RollupChart` | Component | `frontend/src/components/RollupChart.tsx` | 4.5 | ⬜ Not started | |
| `ApprovalsPage` | Page | `frontend/src/pages/ApprovalsPage.tsx` | 4.3 | ⬜ Not started | |
| `DashboardPage` | Page | `frontend/src/pages/DashboardPage.tsx` | 4.5 | ⬜ Not started | |
| `ExpensesDashboardPage` | Page | `frontend/src/pages/ExpensesDashboardPage.tsx` | 4.6 | ⬜ Not started | |
| `ReportsPage` (date range, CSV export) | Page | `frontend/src/pages/ReportsPage.tsx` | 4.7 | ⬜ Not started | |
| `AuditLogPage` | Page | `frontend/src/pages/AuditLogPage.tsx` | 4.8 | ⬜ Not started | |
| Request-correction action on transactions list | UI | `frontend/src/pages/TransactionsListPage.tsx` (extend) | 4.1 | ⬜ Not started | |
| `test_rollup_periods` | Unit | `backend/tests/unit/test_rollup_periods.py` | 4.4 | ⬜ Not started | |
| `test_change_requests` | Integration | `backend/tests/integration/test_change_requests.py` | 4.1, 4.2 | ⬜ Not started | |
| `test_reports` | Integration | `backend/tests/integration/test_reports.py` | 4.4, 4.7, 4.8 | ⬜ Not started | |
| `approval-flow.spec.ts` | E2E | `frontend/e2e/approval-flow.spec.ts` | 4.9 | ⬜ Not started | |
| `dashboard.spec.ts` | E2E | `frontend/e2e/dashboard.spec.ts` | 4.9 | ⬜ Not started | |
| Manual test script, Stage 4 | Manual | `00_docs/stage4/manual_test_stage4.md` | all | ⬜ Not started | |

---

## Stage 5: Inventory, cash and release

| Object | Type | File | Deliv. | Status | Date |
|--------|------|------|--------|--------|------|
| `InventoryItem` | Model | `backend/app/models/tenant/inventory_item.py` | 5.1 | ⬜ Not started | |
| `InventoryMovement` | Model | `backend/app/models/tenant/inventory_movement.py` | 5.1 | ⬜ Not started | |
| `CashCount` | Model | `backend/app/models/tenant/cash_count.py` | 5.2 | ⬜ Not started | |
| Tenant migration `0004` (inventory, cash_counts) | Migration | `backend/alembic/versions/tenant/0004_inventory_cash.py` | 5.1, 5.2 | ⬜ Not started | |
| `InventoryService` (derived stock, movements, no negative stock) | Service | `backend/app/services/inventory_service.py` | 5.1 | ⬜ Not started | |
| `CashService` (server-computed expected amount, variance) | Service | `backend/app/services/cash_service.py` | 5.2 | ⬜ Not started | |
| `InventoryItemOut`, `InventoryMovementCreate`, stock level | Schema | `backend/app/schemas/inventory.py` | 5.1 | ⬜ Not started | |
| `CashCountCreate`, `CashCountOut` | Schema | `backend/app/schemas/cash.py` | 5.2 | ⬜ Not started | |
| `GET/POST /inventory`, `PATCH /inventory/{id}`, `GET/POST /inventory/{id}/movements` | Route | `backend/app/api/v1/inventory.py` | 5.1 | ⬜ Not started | |
| `GET/POST /cash-counts` | Route | `backend/app/api/v1/cash.py` | 5.2 | ⬜ Not started | |
| Inventory API calls | API client | `frontend/src/api/inventory.ts` | 5.1 | ⬜ Not started | |
| Cash API calls | API client | `frontend/src/api/cash.ts` | 5.2 | ⬜ Not started | |
| `InventoryPage` | Page | `frontend/src/pages/InventoryPage.tsx` | 5.1 | ⬜ Not started | |
| `CashOnHandPage` | Page | `frontend/src/pages/CashOnHandPage.tsx` | 5.2 | ⬜ Not started | |
| `test_inventory` | Integration | `backend/tests/integration/test_inventory.py` | 5.1 | ⬜ Not started | |
| `test_cash` | Integration | `backend/tests/integration/test_cash.py` | 5.2 | ⬜ Not started | |
| Backup and restore scripts | Script | `scripts/backup_all.py`, `scripts/restore_tenant.py` | 5.5 | ⬜ Not started | |
| Nightly orphan-identity check | Script | `scripts/check_orphan_identities.py` | 5.5 | ⬜ Not started | |
| Production configuration (hosting, TLS, email, error tracking) | Infra | deployment config | 5.6, 5.7 | ⬜ Not started | |
| Runbooks | Doc | `00_docs/runbooks/` | 5.8 | ⬜ Not started | |
| Security review notes | Doc | `00_docs/stage5/security_review.md` | 5.3 | ⬜ Not started | |
| Manual test script, Stage 5 + full regression | Manual | `00_docs/stage5/manual_test_stage5.md` | 5.9 | ⬜ Not started | |

---

## Counts

Planned objects per stage (rows above, excluding Stage 0 docs). Update as rows are added.

| Stage | Rows | Not started | Complete |
|---|---|---|---|
| 1 Setup | 16 | 16 | 0 |
| 2 Auth and tenants | 54 | 54 | 0 |
| 3 Core transactions | 30 | 30 | 0 |
| 4 Approvals and reports | 26 | 26 | 0 |
| 5 Inventory, cash, release | 22 | 22 | 0 |

## Additions to `file_system.md`

None outstanding. `file_system.md` was synced with this list on 2026-10-30. When a row adds a file the structure doesn't show, mark it *(not yet in `file_system.md`)* and list it here.
