# System Design

The detailed mechanics behind `arch_overview.md`: how tenants are routed and provisioned, how a transaction is written, how roll-ups are calculated, and how the pieces behave under failure. Tables are defined in `data_model.md`, endpoints in `api_spec.md`.

Stack assumption: the baseline in `file_system.md` (Python 3.12, FastAPI, SQLAlchemy 2.x, Alembic, PostgreSQL 16, React + TypeScript). The stack is not yet confirmed, so framework-specific details are marked as such.

## Table of Contents

1. [Scope and sizing](#scope-and-sizing)
2. [Tenant DB registry and connection management](#tenant-db-registry-and-connection-management)
3. [Tenant provisioning](#tenant-provisioning)
4. [Migrations across tenants](#migrations-across-tenants)
5. [Authentication internals](#authentication-internals)
6. [Writing a transaction](#writing-a-transaction)
7. [Change-request approval](#change-request-approval)
8. [Roll-up queries](#roll-up-queries)
9. [Cash expected amount](#cash-expected-amount)
10. [Export](#export)
11. [Frontend design](#frontend-design)
12. [Errors, retries and idempotency](#errors-retries-and-idempotency)
13. [Security design](#security-design)
14. [Observability and operations](#observability-and-operations)
15. [Performance budget](#performance-budget)
16. [Scaling path](#scaling-path)
17. [v3 sketch: POS and cycle-count reconciliation](#v3-sketch-pos-and-cycle-count-reconciliation)
18. [Risks](#risks)

---

## Scope and sizing

| Quantity | Assumption |
|---|---|
| Tenants | ~10 within the first year (owner's own laundromat first) |
| Users per tenant | ≤ 5 |
| Transactions per tenant per day | 50–300, each with 1–6 lines |
| Rows per tenant per year | ~100k transactions, ~300k lines |
| Concurrent requests | Single digits. Peaks at opening and closing |
| Availability target | Business hours matter most; 99.5% monthly is enough for v1 |

These numbers are why the design is a single API process and a single Postgres server. Nothing here needs queues, caches, or read replicas.

---

## Tenant DB registry and connection management

`backend/app/tenant_db.py` maps a tenant slug to a SQLAlchemy engine.

```
get_tenant_session(slug) -> Session
    engine = registry.get(slug)            # cached
    if engine is None:
        row = platform.tenants[slug]        # must be status = active
        engine = create_engine(url_for(row.db_name), pool_size=3, max_overflow=2, pool_pre_ping=True)
        registry.put(slug, engine)          # LRU, max 50 engines
    return Session(engine)
```

- **Credentials.** Each tenant database has its own role (`baymax_t_{slug}_app`) that has rights on that database only. The URL template and the role's secret come from configuration, not from the registry table. A bug that opens the wrong engine still can't read a different tenant's data without that tenant's credentials.
- **Pool size.** Small per-tenant pools (3 + 2 overflow) with an LRU cap of 50 engines. Worst case 250 connections, far below what the tenant count needs; with 10 tenants it is about 50. `max_connections` on Postgres is set with headroom; PgBouncer is the fallback if tenant count grows.
- **Eviction.** The least recently used engine is disposed when the cap is hit. A tenant that goes idle holds no connections after eviction.
- **Suspended tenants.** `get_tenant_session` refuses any tenant whose registry status isn't `active`; existing sessions are revoked on suspension.
- **Dependency.** `get_tenant_db` is the **only** public way to get a tenant session. It reads the slug from the authenticated session, never from the request. A unit test greps that no route takes a tenant parameter.
- **Request scope.** One session per request, committed or rolled back at the end of the request by the dependency, and closed in `finally`.

---

## Tenant provisioning

Run by `tenant_service.provision_tenant` (API: `POST /admin/tenants`, CLI: `scripts/provision_tenant.py`). It is **resumable**: each step records progress on the tenant row and checks whether it is already done.

| Step | Action | Idempotent check |
|---|---|---|
| 1 | Validate slug (`^[a-z][a-z0-9-]{2,30}$`, not reserved) and email; insert `tenants` row with `status = provisioning` | Unique slug |
| 2 | `CREATE ROLE baymax_t_{slug}_app …` and `CREATE DATABASE baymax_t_{slug} OWNER …` using an admin connection to the Postgres server (autocommit, since `CREATE DATABASE` can't run in a transaction) | `IF NOT EXISTS` via catalog lookup |
| 3 | Run Alembic tenant migrations to `head` against the new DB | Alembic is idempotent |
| 4 | Insert `tenant_settings` and the first `users` row (role `tenant_admin`) in the tenant DB | Unique `identity_id` |
| 5 | Insert the `user_identities` row in the platform DB and create the set-password token | Unique `email` |
| 6 | Send the set-password email | Token reuse on retry |
| 7 | Set `tenants.status = active` | |

Any exception sets `status = failed` with `failure_reason`; **Retry** re-runs from step 1 and skips completed steps. Because the identity (step 5) is created after the tenant DB (step 4), a failure never leaves a login that points at a non-existent database.

A deprovision path (drop database) is **not** in v1. Offboarding is `suspended`, with data kept.

---

## Migrations across tenants

- Two Alembic trees: `alembic/versions/platform/` (run once) and `alembic/versions/tenant/` (run against every tenant database). `env.py` selects the target with `-x tenant=<slug>` or `-x target=platform`.
- `scripts/migrate_all_tenants.py` loops over `tenants` where `status IN ('active','suspended')`, runs `upgrade head` on each, **continues on failure**, records the result in `tenant_settings.schema_version`, and exits non-zero with a summary if any failed.
- **Deploy order:** (1) migrate platform, (2) migrate all tenants, (3) start the new API. Because step 3 comes after, all migrations must be **backward compatible** with the previous API version for the short window (add columns nullable or with defaults; do not rename or drop in the same release; drop in a later release).
- At startup the API checks the schema version of a tenant when its engine is first loaded and returns `503 tenant_migrating` if it is older than expected, rather than failing mid-request.

---

## Authentication internals

**Login** (`auth_service.login`)
1. Normalize email (lower case, trimmed). Look up `user_identities`. If not found, still run a dummy argon2 verify to keep timing similar.
2. If `locked_until` is in the future → `423`. Verify the password; on failure increment `failed_login_count` and lock at 5 for 15 minutes.
3. Check `is_active`, and for tenant users that the tenant is `active`.
4. Look up the tenant `users` row; if the profile is missing or inactive → `403`.
5. Create a token with 32 random bytes (URL-safe base64). Store its SHA-256 in `sessions` with `expires_at = now + 12h`. Set the cookie.

**Per request** (`get_current_user`)
- Hash the cookie token, load the `sessions` row, require `revoked_at IS NULL` and `expires_at > now()` and idle time under 8 hours (`last_seen_at`, updated at most once per minute to avoid a write per request).
- Load the identity and (for tenant users) the `users` row from the tenant DB to get `role` and `is_active`. Role is read every request, so deactivating or demoting someone takes effect immediately.

**CSRF.** The session carries a CSRF token returned by `/auth/me`. Mutating requests (`POST`, `PATCH`, `DELETE`) must send it in `X-CSRF-Token`; the check is a constant-time comparison. Cookies are `SameSite=Lax` as a second layer.

**Passwords.** argon2id with library defaults; minimum 10 characters; no composition rules. A short common-password denylist is checked.

**Auth0 swap.** Replace `get_identity` so it validates a JWT (`iss`, `aud`, `exp`, signature via JWKS, cached) and maps `sub` → `user_identities.external_subject`. The tenant is resolved from the identity row exactly as now. Login, logout and reset routes then delegate to Auth0 (redirect flow) and are removed or become thin redirects. Nothing in services or tenant DBs changes.

---

## Writing a transaction

`transaction_service.create_transaction(db, actor, payload)`; one database transaction in the tenant DB.

```
1. Idempotency: SELECT by client_request_id.
     found + same actor + same payload hash  -> return existing (200)
     found + different payload               -> 409 idempotency_mismatch
2. Authorize: actor.role vs payload.type (see api_spec access matrix).
3. Load all referenced items in one query. For each line:
     - item must be active, in the category that matches payload.type
     - fixed  -> unit_price = item.price      (client value ignored)
     - variable -> unit_price required, >= 0
     - amount = round_half_up(quantity * unit_price, 2)
4. total = sum(line.amount)  (Decimal arithmetic only)
5. status:
     type in (sales, expense)                     -> posted
     type = adjustment and role in (owner, admin) -> posted
     type = adjustment and role = staff           -> pending
6. INSERT transactions, transaction_lines, audit_log (action transaction.create).
7. COMMIT.
```

Concurrency: there is no shared counter or running balance, so concurrent entries don't contend. The unique index on `client_request_id` closes the race in step 1 (the loser gets a unique violation and re-reads).

Rounding rule: line amounts are rounded per line (half-up), and the total is the sum of rounded lines, so the screen, the stored total and the CSV always agree to the cent.

---

## Change-request approval

`change_request_service.approve(db, actor, request_id)`, one database transaction:

```
1. SELECT change_request ... FOR UPDATE      (serializes two approvers)
2. require status = 'pending'; actor.role in (owner, tenant_admin); actor.id != requested_by
3. SELECT target transaction FOR UPDATE; require status = 'posted'   else 409 target_changed
4. kind = void:    UPDATE transactions SET status = 'voided'
   kind = correct: UPDATE target SET status = 'voided';
                   INSERT replacement transaction (+ lines, repriced as in create_transaction,
                   adjusts_transaction_id = target.id, created_by = original creator,
                   decided_by = actor)
5. UPDATE change_request SET status = 'approved', decided_by, decided_at
6. INSERT audit_log (before = target snapshot, after = result)
7. COMMIT
```

Rejection and cancellation are the same shape without step 4. The pending staff adjustment path (`transactions/{id}/approve`) sets `status = posted` or `rejected` the same way, with a lock on the transaction row.

The replacement transaction keeps the original `business_date` unless the proposal changes it, so a correction doesn't move money between days by accident.

---

## Roll-up queries

Roll-ups are computed on demand from `transactions` and `transaction_lines` where `status = 'posted'`. Nothing is pre-aggregated in v1.

Daily series for a range (the base for all periods):

```sql
SELECT t.business_date,
       SUM(t.total) FILTER (WHERE t.type = 'sales')                              AS sales,
       SUM(t.total) FILTER (WHERE t.type = 'expense')                            AS expense,
       SUM(t.total * t.direction) FILTER (WHERE t.type = 'adjustment'
                                          AND t.adjusts_type = 'sales')          AS sales_adj,
       SUM(t.total * t.direction) FILTER (WHERE t.type = 'adjustment'
                                          AND t.adjusts_type = 'expense')        AS expense_adj
FROM transactions t
WHERE t.status = 'posted'
  AND t.business_date BETWEEN :from AND :to
GROUP BY t.business_date
ORDER BY t.business_date;
```

Net per day = `sales + sales_adj − (expense + expense_adj)`. The service fills missing days with zeros so charts have no gaps.

Period handling lives in one pure function (`report_service.period_bounds(period, date, week_starts_on)`), unit tested for boundaries:
- weekly: starts on `week_starts_on`, 7 days;
- monthly: first to last day, including February and leap years;
- half-year: Jan 1–Jun 30 or Jul 1–Dec 31;
- the weekly range may straddle two months (the weekly view is by week, not clipped to a month).

Weekly and monthly views reuse the daily series and sum in Python; half-year groups by month (`date_trunc('month', business_date)`). By-item breakdown joins `transaction_lines` to `transactions` and groups by `item_id`.

**Indexes:** `transactions (business_date, type, status)` serves all of the above; `transaction_lines (transaction_id)` serves the join. With ~100k transactions a year per tenant, the half-year query scans about 50k rows and finishes in tens of milliseconds, so no caching or materialized views are needed. If that stops being true, add a `daily_totals` table maintained in the same DB transaction as writes (not a cache), without changing the API.

---

## Cash expected amount

Computed server-side when a count is recorded (never trusted from the client) and stored on the row so history doesn't change when later corrections are approved:

```
expected = opening_amount
         + sum(posted sales totals for business_date)
         − sum(posted expense totals for business_date)
         + sum(posted sales-side adjustments)  − sum(posted expense-side adjustments)
```

Only `posted` transactions count. If two shifts are counted in a day, the second shift's expected amount includes only transactions after the first count's `created_at`; the first shift's counted amount becomes the second's default opening amount. (For v1 with one `day` shift per date this is the whole day.)

---

## Export

`GET /reports/export` streams CSV so memory stays flat: a server-side cursor over the line query, `StreamingResponse`, UTF-8 with a BOM so Excel reads accented characters, decimals as plain `123.45`. A cell that starts with `=`, `+`, `-` or `@` is prefixed with `'` to prevent spreadsheet formula injection from item names or notes. Range capped at 366 days.

---

## Frontend design

- **Routing and guards.** `AppShell` wraps all routes, reads the role from `AuthContext`, redirects unauthenticated users to login, and hides pages the role can't use. The server enforces the same rules; the UI hiding is convenience only.
- **Data fetching.** Typed client in `src/api/`; TanStack Query **[open, recommended in stack_comparison.md]** for caching and refetch. The dashboard polls every 30 seconds; the Approvals badge every 60.
- **Entry screen.** Item buttons are rendered from `GET /items?category=…` (cached, refreshed on focus). Taps update local state; the total shown is a convenience; the server's total is authoritative and is displayed in the confirmation. Minimum touch target 48 px; no keyboard for fixed-price items; a number pad for quantities.
- **Offline behavior.** v1 requires a connection. A failed save is retried with the same `client_request_id` and the screen is not cleared until success. A fuller offline queue (PWA) is out of scope for v1.
- **Responsive layout.** Bottom navigation on phones, side navigation on tablet and desktop. The same components serve all three; the entry screen is single-column on phone and two-column (items | current sale) on tablet/desktop.
- **State.** Server state in the query cache; the in-progress sale is local component state (lost on reload, which is acceptable because nothing was saved).
- **Types.** Generated from the OpenAPI document **[open: openapi-typescript]**, so a change to a schema breaks the build instead of production.

---

## Errors, retries and idempotency

| Situation | Behavior |
|---|---|
| Network failure on save | Client retries with the same `client_request_id`; server returns the original record if the first attempt got through |
| Two approvers click at once | Row lock on the request; the second gets `409 not_pending` |
| Tenant DB unreachable | `503` with `Retry-After`; the platform DB and other tenants are unaffected because pools are per tenant |
| Tenant mid-migration | `503 tenant_migrating` |
| Provisioning step fails | Tenant marked `failed`; retry resumes |
| Unhandled exception | Rolled back; `500` with generic message and `request_id`; error tracker gets full detail |

All service methods that write open exactly one DB transaction and either commit all changes (including the audit entry) or none.

---

## Security design

| Threat | Control |
|---|---|
| Reading another tenant's data | Physical DB separation, per-tenant DB role, tenant from session only, isolation tests at 100% |
| Employee alters or deletes a record to hide theft | No edit/delete routes; DB role for audit log can only insert; change requests need a different approver; voids keep the original |
| Employee under-rings a sale | Fixed prices enforced server-side; cash count variance; owner dashboard; (v3) cycle counts vs sales |
| Owner account compromise | Lockout, rate limits, session revocation on password reset, Auth0 and MFA planned |
| Credential stuffing | Rate limit per IP and per email, lockout, generic error messages |
| Session theft | `HttpOnly`, `Secure`, `SameSite=Lax` cookie; hashed tokens at rest; short idle timeout |
| CSRF | Header token plus `SameSite` |
| SQL injection | ORM with bound parameters; no string-built SQL (the tenant DB name comes from the registry and is validated against `^baymax_t_[a-z0-9_]+$` before use in `CREATE DATABASE`) |
| Spreadsheet formula injection | Prefix on export cells |
| Secrets | Environment variables; `.env.example` has placeholders only; no secrets in the repo or logs |
| Stale permissions | Role re-read from the tenant DB on every request |

---

## Observability and operations

- **Logs:** JSON, one line per request, with `request_id`, `tenant`, `user_id`, route, status, duration. Passwords, tokens and CSRF values are never logged.
- **Health:** `GET /healthz` (process up) and `GET /readyz` (platform DB reachable). A tenant check is not part of readiness so one bad tenant doesn't take the API out of rotation.
- **Errors:** an error tracker for API and frontend **[open: Sentry]**.
- **Metrics (optional for v1):** request rate/latency, pool usage per tenant, login failures, provisioning failures.
- **Backups:** nightly `pg_dump` of the platform DB and each tenant DB, retained 30 days, with a **restore drill** before the first external tenant. Point-in-time recovery if the hosting provider offers it.
- **Runbooks to write:** restore one tenant; re-run failed migration for one tenant; revoke all sessions; rotate the database role secret.

---

## Performance budget

| Operation | Target (p95) |
|---|---|
| Save a transaction | < 300 ms |
| Load entry screen (items cached) | < 1 s on mobile data |
| Daily / weekly / monthly roll-up | < 500 ms |
| Half-year roll-up | < 1 s |
| CSV export of one month | < 5 s |
| Login | < 800 ms (argon2 dominates) |

---

## Scaling path

Not needed for v1; recorded so later choices don't paint the project into a corner.

1. More tenants → PgBouncer in front of Postgres; smaller per-tenant pools.
2. One heavy tenant → move its database to another server; change the connection URL stored for it.
3. Slow roll-ups → `daily_totals` table maintained transactionally, then (if needed) background jobs.
4. More API load → run several API containers; they are stateless (sessions are in the DB).
5. Many tenants (hundreds) → revisit database-per-tenant versus a pooled model for small tenants.

---

## v3 sketch: POS and cycle-count reconciliation

**POS.** A POS device authenticates with a per-device API key scoped to one tenant and calls `POST /ingest/pos/transactions`. The handler calls the same `transaction_service` with `source = 'pos'` and the device's `external_ref` as the idempotency key. Staff entry stays available for walk-in corrections. Device keys are stored hashed in the tenant DB.

**Cycle counts.** Import a machine export (CSV first) into `machine_cycle_counts (machine_id, business_date, cycles, import_batch_id)`; re-importing a batch is idempotent on `(machine_id, business_date)`. A reconciliation report compares, per day, cycles on washers against the quantity of `Wash` lines sold and dryer cycles against `Dry` lines, shows the difference, and flags days beyond a tolerance the owner sets. Mapping items to machine types is a setting. Large imports are the first case where a background worker is justified.

---

## Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Stack not decided | Rework if changed after coding starts | Decide FastAPI vs Node before the first object is built; this document is framework-light except where marked |
| Tenant migrations fail halfway across tenants | Mixed schema versions | Backward-compatible migrations, per-tenant version check, `503 tenant_migrating`, a runbook |
| Staff avoid the system (don't ring up sales) | The tool can't detect unrecorded sales | Cash count variance now; machine cycle counts in v3 |
| Slow or missing connectivity at the shop | Lost or duplicated entries | Idempotent retry; PWA/offline queue considered after v1 |
| Single Postgres server | One failure affects all tenants | Backups with restore drills; managed Postgres with failover once there are paying tenants |
| 5-user limit vs shared logins | Staff share one login, weakening the audit trail | Make adding staff easy; show who is logged in; discourage shared accounts in onboarding |
| Pricing and approval rules differ per business | Owner wants different rules | Keep rules in services behind clear functions; settings table available for later options |
