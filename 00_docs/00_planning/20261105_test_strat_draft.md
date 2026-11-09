# Testing Strategy (draft rewrite for Baymax v1.0)

Draft of 2026-11-05 for deliverable 0.11. It replaces the AthleteFlow content in `test_strat.md`. Items marked **(my call)** are my suggestions and are not decided. Open points are listed at the end.

## Overview

Baymax holds money records for several businesses, and its main control is anti-pilferage: staff can't edit records, and every change goes through an approval with an audit trail. Testing is ordered by what costs most when wrong:

1. **Tenant isolation.** One tenant must never reach another's data (a database per tenant).
2. **Money correctness.** Totals, prices, roll-ups and exports must agree to the cent.
3. **The approval and audit control.** Staff can't edit; approvals can't be self-approved; the audit log can't be changed.
4. **Auth and roles.** Right screens and routes for each of the four roles.
5. **Speed and ease of entry** on a phone, for low-tech users.

Principles:
- Integration tests run against a **real PostgreSQL**. No mocks, and no SQLite stand-in.
- Rules the database enforces (CHECK constraints, grants, unique keys) are tested **at the database**, not only through the API, so a service bug can't hide a broken rule.
- Each acceptance line for F1–F13 in `critical_user_flow.md` maps to at least one automated test (see the matrix).

## Roles used in tests

| Role | Value | Can do (for test purposes) |
|---|---|---|
| SaaS Admin | `is_saas_admin` (no tenant) | Onboard, list, suspend and retry tenants. Cannot see tenant business data |
| Tenant Admin | `tenant_admin` | Create and deactivate users (5 active max). Can approve |
| Franchise Owner | `owner` | Items and prices, approvals, reports, export |
| Employee | `staff` | Enter sales, expenses, adjustments; file change requests. No edits, no approvals |

## Test levels

### 1. Unit tests (no database, no HTTP)

| Area | File (per `file_system.md`) | Examples |
|---|---|---|
| Security | `test_security.py` | argon2id hash and verify; token generation and hashing; common-password denylist; minimum length 10 |
| Tenant names | `test_tenant_utils.py` | Slug rules; `db_name` is `baymax_t_{slug}` with `-` replaced by `_`; reject unsafe slugs |
| Role checks | `test_role_checks.py` | Permission table for each role and action |
| Pricing | `test_pricing.py` | Fixed price comes from the item, never from the client; variable price needs an amount; line amount = quantity × unit price; total = sum of lines; `Decimal` rounding to 2 places |
| Periods | `test_rollup_periods.py` | Week start (default Monday), month ends including leap-year February, half-year boundaries (30 Jun / 1 Jul, 31 Dec / 1 Jan), business date vs `created_at` near midnight in the tenant timezone |

### 2. Integration tests (real PostgreSQL)

Run through the FastAPI app with `httpx` against a platform DB and two provisioned tenant DBs.

| File | What it proves |
|---|---|
| `test_auth.py` | Login, logout, `me`, lockout after 5 failures, session expiry and revocation, CSRF header required on mutating calls, set-password and reset links are single-use |
| `test_tenant_isolation.py` | See *Tenant isolation* below |
| `test_tenant_provisioning.py` | DB, role, migrations, settings and Tenant Admin are created; a failure at each step leaves status `failed`; retry resumes and completes |
| `test_users.py` | 5-active-user limit, including **concurrent** creates; the last Tenant Admin can't be deactivated; deactivation ends sessions; role assignment |
| `test_items.py` | Only `owner` creates or changes items and prices; a price change doesn't alter old transactions |
| `test_transactions.py` | Sales, expenses and adjustments; line totals; repeated `client_request_id` creates no duplicate; same ID with a different payload returns 409 |
| `test_change_requests.py` | Reason required; requester can't approve; approval voids the original and posts the replacement; rejection needs a note; every step writes `audit_log` |
| `test_reports.py` | Roll-up totals per period equal the underlying posted transactions; net formula; voided and pending rows excluded; CSV rows and totals equal the on-screen figures |
| `test_inventory.py`, `test_cash.py` | Stock movements update levels; cash variance = counted − expected; second count for the same date and shift refused |
| `test_migrations.py` **(my call, new)** | Platform and tenant migrations apply from empty to `head`; `migrate_all_tenants` continues past one failing tenant and exits non-zero |
| `test_auth_provider_contract.py` **(my call, new)** | The auth service meets one interface contract, so a Zitadel or Auth0 adapter can replace the DB adapter and pass the same tests |

### 3. Database-level tests

Run SQL directly as the **application role** and expect refusal (these are the `data_model.md` integrity rules):

| # | Rule | Test |
|---|---|---|
| 1 | `is_saas_admin = (tenant_id IS NULL)` | Insert violating rows |
| 2 | Fixed ⇔ price present | Fixed item without price; variable item with price |
| 3 | `quantity > 0`, `unit_price >= 0` | Zero, negative |
| 4 | `decided_by <> requested_by` | Self-approval insert |
| 5 | `audit_log` has no `UPDATE`/`DELETE` grant | Try both as the app role |
| 6 | `client_request_id` unique | Duplicate insert |
| 7 | Movement sign matches kind | Wrong sign |

### 4. End-to-end tests (Playwright)

Run a real browser against the running stack (Docker Compose) with the E2E seed (3.9, built on the dev seed 2.15). Use a **phone viewport** for staff flows and a desktop viewport for owner and admin flows.

### 5. Manual tests

Scripts per stage, run from the separate Windows machine on the same network (project rule). Cover:
- Real phone and tablet use of the entry screen by someone unfamiliar with the app.
- Visual checks that don't automate well: layout, touch target size, empty states, readability.
- Email delivery once a real provider is set up (M5).
- Restore drill from backup (M5).

## Tenant isolation (100% of paths)

Fixtures: tenant A and tenant B, each with data in every table.

- A tenant A session gets 404 or 403 for every tenant B record on **every route** (generated from the route list, so a new route can't skip the check).
- The tenant is taken from the session only. A client-sent tenant ID, header or query parameter is ignored.
- Each tenant's database role cannot connect to the other tenant's database.
- A tenant A user cannot use a tenant B record's ID in a body field (for example an item ID on a transaction line).
- SaaS Admin sessions cannot read tenant business tables through the tenant routes.
- Suspended tenant: users are refused; data intact.
- Tenant engine cache: after a request for B, the next request for A still uses A's database (guards against a cached-connection mix-up).
- E2E repeat of the above through the UI (`tenant-isolation.spec.ts`).

## Flow coverage matrix (F1–F13)

Acceptance lines come from `critical_user_flow.md`.

| Flow | Persona | Integration | E2E |
|---|---|---|---|
| F1 Tenant onboarding | SaaS Admin | `test_tenant_provisioning` (reaches `active`; failed step then Retry) | Create tenant, see status, retry a forced failure |
| F2 First login, set password | New user | `test_auth` (used or expired link refused; valid link sets password) | Link → set password → correct home screen per role |
| F3 Create users | Tenant Admin | `test_users` (6th user refused; free a slot; last admin protected) | Add user, hit the limit |
| F4 Item lists and prices | Owner | `test_items` (staff blocked; old transactions unchanged) | Owner edits price; staff sees no edit controls |
| F5 Record a sale | Staff | `test_transactions` (total; duplicate request; fixed price not overridable) | Three taps + Save on a phone; measure taps |
| F6 Record an expense | Staff / Owner | `test_transactions` (variable amount required) | Variable expense entry |
| F7 Correct a mistake | Staff → Owner | `test_change_requests` (no direct edit; reason; void and replace; no self-approval) | Staff files, owner approves on a phone |
| F8 File an adjustment | Staff → Owner | `test_transactions`, `test_change_requests` (staff adjustment excluded until approved; owner adjustment posts at once) | Both paths |
| F9 Owner approves | Owner | `test_change_requests` (reject needs a note; totals change only on approval) | Approve and reject |
| F10 End-of-day close | Staff | `test_cash` (variance; duplicate count refused) | Close the day |
| F11 Check remotely | Owner | `test_reports` (totals per period match transactions) | Staff enters; owner dashboard shows it within the refresh interval |
| F12 Export | Owner | `test_reports` (CSV equals on-screen) | Download and compare |
| F13 Forgot password | Any | `test_auth` (same message for unknown email; reused link refused; old sessions end) | Full reset flow |

## Critical scenarios (100% coverage)

### Authentication
- Passwords stored as argon2id; tokens stored hashed; the plain token is never logged.
- Lockout after 5 failed logins; unknown email takes similar time to a known one.
- Expired or revoked session returns 401.
- Reset and set-password tokens are single-use and expire.
- Every mutating request without a valid `X-CSRF-Token` is refused.

### Role-based access
- Table-driven: every route × every role (plus no session) has an expected allow or deny, so an unlisted route fails the test.
- `staff` cannot create items, change prices, approve, view the audit log or export **(confirm against `api_spec.md`)**.
- `owner` and `tenant_admin` can approve; nobody approves their own request.

### Money and records
- All amounts are `Decimal`/`NUMERIC(12,2)`; a test fails on any float in the money path.
- Posted transactions are never edited or deleted; only voided through an approved change request.
- Roll-up net = sales + sales-side adjustments − expenses − expense-side adjustments.
- Business date near midnight lands in the right period for the tenant's timezone.

### Audit trail
- Every create, void, approval and rejection writes an `audit_log` row in the same database transaction; if the change rolls back, so does the log row.
- The app role cannot update or delete `audit_log`.

### Concurrency and retries
- Two simultaneous user creates at 4 active users: exactly one succeeds.
- A double-tapped Save sends the same `client_request_id` and creates one transaction.

## Non-functional checks

| Check | Target | How |
|---|---|---|
| Save a transaction | p95 < 300 ms | Timed in integration/performance test on seeded data |
| Entry screen load | < 1 s on mobile data | Playwright with network throttling |
| Roll-ups | daily/weekly/monthly p95 < 500 ms; half-year < 1 s | Seed a year of transactions, then query |
| CSV export, one month | < 5 s | Same data set |
| Login | p95 < 800 ms | Integration |
| Responsive layout | Phone, tablet, desktop | Playwright viewports (phone 390 px, tablet 768 px, desktop 1280 px) **(my call)**, then manual |
| Browsers | Latest Chrome on Android, Safari on iOS, Chrome/Edge on desktop **(my call; the old list was desktop-only)** | Playwright chromium and webkit; manual on real devices |
| Offline entry | Not tested in v1 | Re-add when the PWA queue is built |

Targets are from `system_design.md` § Performance budget.

## Coverage targets

| Layer | Target | Notes |
|---|---|---|
| Business logic (unit) | ≥ 90% **(my call; the old target was 80%)** | Pricing, periods, security, roles |
| API and services (integration) | ≥ 80% **(my call; old target 75%)** | Real databases |
| Tenant isolation paths | **100%** | Route-generated, see above |
| Auth, money and audit paths | **100%** | See critical scenarios |
| Frontend components | Key flows covered by E2E; unit tests for entry-screen logic | |

CI fails the build if a 100% area drops, or the overall figure drops below target.

## Tools

| Purpose | Tool |
|---|---|
| Backend unit and integration | `pytest`, `pytest-asyncio` if the API is async **(my call)** |
| API calls | `httpx` against the ASGI app |
| Real databases | PostgreSQL via Testcontainers (as in the stack decision) |
| Concurrency tests | `asyncio.gather` or threads against the same tenant DB |
| Coverage | `pytest-cov` |
| E2E | Playwright (chromium, webkit) |
| Frontend unit tests | Vitest + React Testing Library **(my call; not in the stack decision)** |
| Static checks | Ruff and mypy in CI (listed in `file_system.md`; the stack decision didn't confirm them) |
| CI | GitHub Actions |

## Test data and fixtures

- **Session fixtures** (`conftest.py`): one Postgres container; one platform DB with migrations applied; two tenant DBs, `tenant_a` and `tenant_b`, provisioned through `TenantService` so provisioning is exercised too; one user per role in each.
- **Speed (my call):** build a migrated template database once per session and create each test DB from it with `CREATE DATABASE ... TEMPLATE`. Tests that need a clean tenant create one from the template; most tests roll back a transaction instead.
- **Dev seed (2.15)** creates the first SaaS Admin, a dev tenant and one user per role. The **E2E seed (3.9)** builds on it: three tenants, all roles, item lists and a week of transactions. The seed scripts are not production code paths and refuse to run in production.
- **Known state:** E2E resets the database between runs.
- **Time:** tests inject a clock so period boundaries and session expiry are deterministic.
- **Email:** development uses the console/file sender; tests read the file to get set-password and reset links.
- No production data in any test environment.

## When tests run

| When | What |
|---|---|
| Every commit (local) | Unit tests |
| Every push / PR (CI) | Ruff, mypy, unit, integration, database-level tests, isolation suite |
| Nightly or before merge to `main` | E2E, performance checks |
| End of each stage | Manual test script on the Windows machine; stage sign-off |
| M5 | Security review, full manual regression, restore drill |

## Out of scope for v1

- Load testing beyond the performance budget (single-digit concurrent users).
- Automated tests of the v2 bookkeeping and v3 POS/cycle-count features.
- Offline queue (PWA) tests, until it is built.
- Third-party auth (Zitadel/Auth0) end-to-end, beyond the contract test.

## Open points

1. **Staff permissions for the audit log, reports and export:** confirm against `api_spec.md` before the role matrix is finalised.
2. **Frontend unit test tool:** Vitest + React Testing Library is my suggestion; it isn't in the stack decision.
3. **Browser and device list:** confirm which phones your staff actually use.
4. **Coverage numbers** above the old 80/75 are my suggestion; keep the old ones if you prefer.
5. **Auth provider:** the contract test assumes an `AuthProvider` interface. This stays open while Zitadel vs Auth0 is undecided.
