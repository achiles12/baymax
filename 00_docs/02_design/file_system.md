# Initial Filesystem Set-up
# Updated as needed

Last updated: 2026-10-30 (synced with `object_masterlist.md`: tenant settings model, email and audit services, staff-adjustment approve/reject routes, Reports and Audit log pages, operator scripts, `00_docs/` layout).

Stack carried over from the sample (not yet confirmed for Baymax): FastAPI + SQLAlchemy + Alembic, PostgreSQL, React (TypeScript) + Vite + Tailwind, Playwright.

Multi-tenancy is **database separated** (see `prod_req_doc.md`):
- `baymax_platform` — one shared database: tenant registry + authentication (users log in here, then are routed to their tenant DB)
- `baymax_t_{slug}` — one database per tenant: all laundromat data (items, transactions, approvals, inventory, cash)

baymax/                                    # Project root
├── backend/                               # Python API (FastAPI) — runs as one container
│   ├── app/                               # Application package
│   │   ├── main.py                        # FastAPI app instance, middleware registration
│   │   ├── config.py                      # Settings via pydantic-settings (.env) — platform DB URL, tenant DB URL template
│   │   ├── database.py                    # Platform engine + session factory
│   │   ├── tenant_db.py                   # Tenant engine registry — one cached engine per tenant DB, get_tenant_session(slug)
│   │   ├── dependencies.py                # get_current_user, get_tenant_db, require_role, CSRF check
│   │   │
│   │   ├── api/                           # HTTP layer — route handlers only, delegate to services
│   │   │   └── v1/                        # Version 1 of the REST API, mounted at /api/v1
│   │   │       ├── router.py              # Aggregates all v1 routers
│   │   │       ├── auth.py                # /auth/login, /logout, /me, /reset-password/request|confirm
│   │   │       ├── admin.py               # /admin/tenants, /admin/tenants/{id}, /{id}/retry-provisioning  (SaaS Admin only — onboarding, status)
│   │   │       ├── users.py               # /users  (Tenant Admin — max 5 users per tenant)
│   │   │       ├── items.py               # /items, /items/{id}  (sales / expense / adjustment item lists + pricing)
│   │   │       ├── transactions.py        # /transactions, /transactions/{id}, /transactions/{id}/lines, /transactions/{id}/approve|reject (pending staff adjustments)
│   │   │       ├── change_requests.py     # /change-requests, /change-requests/{id}/approve|reject|cancel  (Owner / Tenant Admin approve)
│   │   │       ├── reports.py             # /reports/rollup?period=daily|weekly|monthly|half-year, /reports/expenses, /reports/export (CSV), /reports/audit-log
│   │   │       ├── inventory.py           # /inventory, /inventory/{id}/movements
│   │   │       └── cash.py                # /cash-counts  (cash on hand)
│   │   │
│   │   ├── models/                        # SQLAlchemy ORM models
│   │   │   ├── platform/                  # baymax_platform DB — global tables
│   │   │   │   ├── tenant.py              # tenants (slug, db_name, status)
│   │   │   │   ├── user_identity.py       # user_identities (email, password hash, tenant_id) — Auth0 migration point
│   │   │   │   ├── session.py             # sessions (token hash, user_identity_id, expires_at)
│   │   │   │   └── password_reset_token.py  # password_reset_tokens (token hash, expires_at, used_at — single use)
│   │   │   └── tenant/                    # baymax_t_{slug} DB — per-tenant tables
│   │   │       ├── tenant_settings.py     # tenant_settings (timezone, currency, week_starts_on, schema_version) — one row
│   │   │       ├── user.py                # users (role: tenant_admin, owner, staff)
│   │   │       ├── item.py                # items (category: sales|expense|adjustment, price_type: fixed|variable, price)
│   │   │       ├── transaction.py         # transactions (type, business_date, status, created_by)
│   │   │       ├── transaction_line.py    # transaction_lines (item_id, qty, unit_price, amount)
│   │   │       ├── change_request.py      # change_requests (target transaction, proposed change, status, decided_by)
│   │   │       ├── audit_log.py           # audit_log (who changed what, before/after)
│   │   │       ├── inventory_item.py      # inventory_items (supplies — soap, conditioner, etc.)
│   │   │       ├── inventory_movement.py  # inventory_movements (stock in / used / adjusted)
│   │   │       └── cash_count.py          # cash_counts (cash on hand per day / shift)
│   │   │
│   │   ├── schemas/                       # Pydantic request/response models
│   │   │   ├── auth.py                    # LoginRequest / LoginResponse, reset-password request and confirm bodies
│   │   │   ├── tenant.py                  # ProvisionTenantRequest, TenantOut — SaaS Admin tenant registry
│   │   │   ├── user.py                    # UserCreate, UserUpdate, UserOut — includes role
│   │   │   ├── item.py                    # ItemCreate, ItemUpdate, ItemOut — category, price_type, price
│   │   │   ├── transaction.py             # TransactionCreate (with lines), TransactionOut, TransactionLineOut
│   │   │   ├── change_request.py          # ChangeRequestCreate, ChangeRequestOut, ApproveRejectBody
│   │   │   ├── report.py                  # RollupQuery (period, date range), RollupRow, ExpenseSummary
│   │   │   ├── inventory.py               # InventoryItemOut, InventoryMovementCreate, stock level
│   │   │   └── cash.py                    # CashCountCreate, CashCountOut — counted vs expected cash
│   │   │
│   │   ├── services/                      # Business logic — no HTTP, no DB sessions
│   │   │   ├── auth_service.py            # login, logout, reset-password, provision_identity
│   │   │   ├── tenant_service.py          # tenant provisioning (create DB, run migrations, initial Tenant Admin)
│   │   │   ├── user_service.py            # day-to-day user management, 5-user limit
│   │   │   ├── item_service.py            # item list maintenance; variable pricing restricted by role
│   │   │   ├── transaction_service.py     # record sales / expense / adjustment transactions + lines, totals
│   │   │   ├── change_request_service.py  # staff submit, owner approve/reject, applies change + writes audit_log
│   │   │   ├── report_service.py          # daily / weekly / monthly / half-year roll-ups, expense summary, CSV export
│   │   │   ├── inventory_service.py       # supply stock levels, record stock in / usage / adjustments
│   │   │   ├── cash_service.py            # record cash counts, compare against expected cash from the day's transactions
│   │   │   ├── email_service.py           # set-password and reset emails — console sender in dev, provider in prod
│   │   │   └── audit.py                   # write_audit helper — called inside every write service's DB transaction
│   │   │
│   │   └── utils/                         # Pure helpers — no DB sessions, no HTTP
│   │       ├── security.py                # password hashing, token generation
│   │       ├── tenant.py                  # slug generation, tenant DB name derivation
│   │       └── db_provisioner.py          # CREATE DATABASE baymax_t_{slug} + run Alembic tenant migrations
│   │
│   ├── alembic/                           # Database migrations
│   │   ├── env.py                         # Targets platform DB, or a given tenant DB via -x tenant=<slug>
│   │   ├── alembic.ini                    # Alembic config — script location, logging
│   │   └── versions/                      # Migration scripts, split by database type
│   │       ├── platform/                  # platform DB migrations (run once)
│   │       └── tenant/                    # tenant DB migrations (run against every tenant DB)
│   │
│   ├── tests/                             # Backend tests (pytest) — run against a real PostgreSQL, no mocks
│   │   ├── conftest.py                    # fixtures: platform DB, 2 provisioned tenant DBs, seeded users per role
│   │   ├── unit/                          # Pure logic tests — no DB, no HTTP
│   │   │   ├── test_security.py           # password hashing, token generation
│   │   │   ├── test_tenant_utils.py       # slug generation, DB name derivation
│   │   │   ├── test_role_checks.py        # permission logic
│   │   │   ├── test_pricing.py            # fixed vs variable price rules
│   │   │   └── test_rollup_periods.py     # week / month / half-year boundaries
│   │   └── integration/                   # API + service tests against real platform and tenant DBs
│   │       ├── test_auth.py               # login, logout, reset, session expiry
│   │       ├── test_tenant_isolation.py   # tenant A session can never reach tenant B DB
│   │       ├── test_tenant_provisioning.py  # tenant DB is created, migrated, and gets its Tenant Admin
│   │       ├── test_users.py              # user CRUD, role assignment, 5-user limit
│   │       ├── test_items.py              # item list CRUD per category, only Owner can set prices
│   │       ├── test_transactions.py       # create sales / expense / adjustment transactions, line totals
│   │       ├── test_change_requests.py    # approval flow + audit_log entries
│   │       ├── test_reports.py            # roll-up totals per period, expense summary, CSV export
│   │       ├── test_inventory.py          # stock movements update stock levels
│   │       └── test_cash.py               # cash counts and variance against expected cash
│   │
│   ├── pyproject.toml                     # Python dependencies, pytest / ruff / mypy config
│   └── Dockerfile                         # Backend container image
│
├── frontend/                              # React (TypeScript) single-page app, built with Vite
│   ├── src/                               # Frontend source
│   │   ├── main.tsx                       # React entry point
│   │   ├── App.tsx                        # Router setup (React Router)
│   │   │
│   │   ├── api/                           # Typed API client functions
│   │   │   ├── client.ts                  # Base fetch wrapper — attaches auth headers
│   │   │   ├── auth.ts                    # login, logout, reset-password calls
│   │   │   ├── admin.ts                   # tenant registry and onboarding calls (SaaS Admin)
│   │   │   ├── users.ts                   # user maintenance calls (Tenant Admin)
│   │   │   ├── items.ts                   # item list and pricing calls
│   │   │   ├── transactions.ts            # create and list transactions and lines
│   │   │   ├── changeRequests.ts          # submit, approve, reject, cancel change requests
│   │   │   ├── reports.ts                 # roll-up, expense summary, CSV export download, audit log
│   │   │   ├── inventory.ts               # inventory items and stock movements
│   │   │   └── cash.ts                    # cash on hand counts
│   │   │
│   │   ├── components/                    # Reusable UI components
│   │   │   ├── ItemPicker.tsx             # Large touch-friendly item buttons for quick entry
│   │   │   ├── TransactionLineList.tsx    # Lines on the current transaction — qty, price, remove, running total
│   │   │   ├── PriceInput.tsx             # Fixed price display / variable price entry
│   │   │   ├── PeriodSelector.tsx         # daily / weekly / monthly / half-year
│   │   │   ├── RollupChart.tsx            # Chart of roll-up totals (sales vs expenses) for the selected period
│   │   │   ├── StatusBadge.tsx            # transaction / change request status
│   │   │   └── EmptyState.tsx             # Placeholder shown when a list has no data yet
│   │   │
│   │   ├── contexts/                      # React context providers
│   │   │   └── AuthContext.tsx            # Session state, login/logout actions
│   │   │
│   │   ├── hooks/                         # Reusable React hooks
│   │   │   └── useAuth.ts                 # Access AuthContext — current user, role, login/logout
│   │   │
│   │   ├── layouts/                       # Page wrappers shared across routes
│   │   │   └── AppShell.tsx               # Responsive nav (phone / tablet / desktop), auth guard, role-based routes
│   │   │
│   │   ├── pages/                         # One component per route
│   │   │   ├── LoginPage.tsx              # Email + password login
│   │   │   ├── PasswordResetPage.tsx      # Request a reset link and set a new password
│   │   │   ├── TransactionEntryPage.tsx   # Staff / Owner — sales, expense, adjustment entry
│   │   │   ├── TransactionsListPage.tsx   # All roles in tenant — view all transactions
│   │   │   ├── ItemMaintenancePage.tsx    # Owner — item lists and pricing
│   │   │   ├── ApprovalsPage.tsx          # Owner — review change requests / adjustments
│   │   │   ├── DashboardPage.tsx          # Roll-up dashboard
│   │   │   ├── ExpensesDashboardPage.tsx  # Expense totals by item and period
│   │   │   ├── InventoryPage.tsx          # Supply stock levels and stock movements
│   │   │   ├── CashOnHandPage.tsx         # Record cash counts and see variance
│   │   │   ├── ReportsPage.tsx            # Owner / Tenant Admin — date range and CSV export
│   │   │   ├── AuditLogPage.tsx           # Owner / Tenant Admin — filterable, read-only audit log
│   │   │   ├── UsersPage.tsx              # Tenant Admin — user maintenance
│   │   │   └── TenantsAdminPage.tsx       # SaaS Admin — tenant registry / onboarding
│   │   │
│   │   └── types/                         # Shared TypeScript types
│   │       └── index.ts                   # Shared TypeScript interfaces (User, Item, Transaction, etc.)
│   │
│   ├── e2e/                               # Playwright E2E tests
│   │   ├── auth.spec.ts                   # login, logout, session persistence, password reset
│   │   ├── tenant-onboarding.spec.ts      # SaaS Admin creates a tenant and its Tenant Admin
│   │   ├── user-creation.spec.ts          # Tenant Admin creates users, 5-user limit
│   │   ├── transaction-entry.spec.ts      # Staff records sales and expense transactions with fixed and variable prices
│   │   ├── item-maintenance.spec.ts       # Owner adds items and sets prices
│   │   ├── approval-flow.spec.ts          # Staff submits a change request, Owner approves / rejects
│   │   ├── dashboard.spec.ts              # Roll-up dashboard shows correct totals per period
│   │   ├── tenant-isolation.spec.ts       # A user of one tenant cannot see another tenant's data
│   │   └── fixtures/                      # E2E test data
│   │       └── seed.py                    # Seeds 3 tenants, all roles, item lists, a week of transactions
│   │
│   ├── public/                            # Static assets served as-is (icons, favicon)
│   ├── index.html                         # Vite HTML entry point
│   ├── vite.config.ts                     # Vite build and dev server config — proxies /api to the backend
│   ├── tailwind.config.ts                 # Tailwind theme and content paths
│   ├── tsconfig.json                      # TypeScript compiler options
│   └── package.json                       # Frontend dependencies and npm scripts (dev, build, lint, test:e2e)
│
├── scripts/                               # Operational CLI scripts
│   ├── create_saas_admin.py               # Create the first SaaS Admin identity in a new environment + set-password link
│   ├── provision_tenant.py                # CLI wrapper around TenantService — create tenant DB + Tenant Admin
│   ├── seed_dev.py                        # Dev only: first SaaS Admin, a dev tenant and one user per role; idempotent; refuses to run in production
│   ├── migrate_all_tenants.py             # Run Alembic tenant migrations against every tenant DB
│   ├── backup_all.py                      # Nightly pg_dump of the platform DB and every tenant DB (30-day retention)
│   ├── restore_tenant.py                  # Restore one tenant DB from a dump without touching other tenants
│   └── check_orphan_identities.py         # Nightly report of tenant users whose platform identity is missing
│
├── 00_docs/                               # Project documentation (not shipped)
│   ├── design/                            # Authoritative design docs (PRD, API spec, data model, flows, screens, processes, …)
│   ├── stage{N}/                          # Per-stage notes, object design docs, manual test scripts, security review
│   ├── summary/                           # Daily summaries (yyyymmdd_*.md)
│   ├── todos/                             # Daily todo lists (yyyymmdd_*.md)
│   ├── runbooks/                          # Restore a tenant, re-run a failed migration, revoke sessions, rotate DB secret
│   └── sample/                            # Templates from an earlier project — format reference only
│
├── .github/                               # GitHub configuration
│   └── workflows/                         # GitHub Actions workflows
│       └── ci.yml                         # lint, typecheck, pytest, playwright on PR
│
├── docker-compose.yml                     # Postgres + API + frontend for local dev
├── .env.example                           # Template for required environment variables — copy to .env
└── README.md                              # Project overview and local setup steps
