# Backend Processes

This document lists every process the Baymax backend runs. For each one it says what starts it, how it gets started, where the code will live, and what it does. It pulls together material from `system_design.md`, `api_spec.md`, `data_model.md`, `arch_overview.md` and `object_masterlist.md`. If this document and a source document disagree, the source document wins and this one should be updated.

Companion document: `ui_screens.md` (which screen triggers which request).

## Table of Contents

- [Trigger types](#trigger-types)
- [Process summary](#process-summary)
- [A. Per-request pipeline](#a-per-request-pipeline)
- [B. Authentication](#b-authentication)
- [C. Tenants and platform](#c-tenants-and-platform)
- [D. Users](#d-users)
- [E. Items](#e-items)
- [F. Transactions](#f-transactions)
- [G. Change requests](#g-change-requests)
- [H. Reports](#h-reports)
- [I. Inventory and cash](#i-inventory-and-cash)
- [J. Internal shared processes](#j-internal-shared-processes)
- [K. Operator, deploy and scheduled processes](#k-operator-deploy-and-scheduled-processes)
- [L. Not in v1](#l-not-in-v1)
- [Open points](#open-points)

---

## Trigger types

v1 has no queue and no worker process (`system_design.md` § Scope and sizing). Almost everything runs inside an HTTP request. The trigger types are:

| Code | Trigger type | How it's triggered |
|---|---|---|
| **REQ** | User request | A screen action sends an HTTP call to `/api/v1/...`. The process runs synchronously in the API process, in one DB transaction, before the response returns |
| **POLL** | Client polling | The frontend repeats a read request on a timer (dashboard every 30 s, approvals badge every 60 s, tenant status while provisioning). The backend just sees ordinary REQ calls |
| **MW** | Per-request middleware | Runs automatically on every request (or every request of a certain kind) before the route handler |
| **INT** | Internal | Called by another process, never directly by a client |
| **CLI** | Operator command | A person with server access runs a script in `scripts/` |
| **DEPLOY** | Deploy pipeline | A step in the release sequence |
| **SCHED** | Scheduled | Runs on a timer (cron or the hosting provider's scheduler) |
| **INFRA** | Infrastructure | Called by the load balancer or hosting platform |

---

## Process summary

| ID | Process | Trigger | Started by | Entry point | Service / code |
|---|---|---|---|---|---|
| A1 | Request ID and logging | MW | Every request | middleware | `main.py` |
| A2 | Rate limiting | MW | Every request | middleware | `main.py` |
| A3 | Session authentication | MW | Every authenticated request | `get_current_user` | `dependencies.py`, `auth_service` |
| A4 | CSRF check | MW | Every `POST`/`PATCH`/`DELETE` | dependency | `dependencies.py` |
| A5 | Role check | MW | Every protected route | `require_role(...)` | `dependencies.py` |
| A6 | Tenant DB session | MW | Every tenant-data route | `get_tenant_db` | `tenant_db.py` |
| A7 | Error handling | MW | Any unhandled exception | exception handler | `main.py` |
| B1 | Login | REQ | S1 Login | `POST /auth/login` | `auth_service.login` |
| B2 | Logout | REQ | Log out button | `POST /auth/logout` | `auth_service.revoke` |
| B3 | Restore session | REQ | Page load | `GET /auth/me` | `auth_service` |
| B4 | Request password reset | REQ | S1 "Forgot password" | `POST /auth/reset-password/request` | `auth_service` |
| B5 | Set or reset password | REQ | S2, from emailed link | `POST /auth/reset-password/confirm` | `auth_service` |
| C1 | Provision tenant | REQ / CLI | S3 New tenant, or operator | `POST /admin/tenants`, `scripts/provision_tenant.py` | `tenant_service.provision_tenant` |
| C2 | Retry provisioning | REQ | S3 Retry on a failed tenant | `POST /admin/tenants/{id}/retry-provisioning` | `tenant_service` |
| C3 | Suspend or reactivate tenant | REQ | S3 status change | `PATCH /admin/tenants/{id}` | `tenant_service` |
| C4 | Tenant engine load and eviction | INT | First request for a tenant / LRU cap | `get_tenant_session` | `tenant_db.py` |
| D1 | Create user | REQ | S4 Add user | `POST /users` | `user_service` |
| D2 | Update or deactivate user | REQ | S4 edit | `PATCH /users/{id}` | `user_service` |
| E1 | Create item | REQ | S5 Add item | `POST /items` | `item_service` |
| E2 | Update item / price change | REQ | S5 edit | `PATCH /items/{id}` | `item_service` |
| F1 | Create transaction | REQ | S6 Save | `POST /transactions` | `transaction_service.create_transaction` |
| F2 | Approve or reject staff adjustment | REQ | S8 Approve/Reject | `POST /transactions/{id}/approve`, `/reject` | `transaction_service` |
| G1 | File change request | REQ | S7 Request correction | `POST /change-requests` | `change_request_service` |
| G2 | Approve change request | REQ | S8 Approve | `POST /change-requests/{id}/approve` | `change_request_service.approve` |
| G3 | Reject change request | REQ | S8 Reject | `POST /change-requests/{id}/reject` | `change_request_service` |
| G4 | Cancel change request | REQ | S7 Cancel (requester) | `POST /change-requests/{id}/cancel` | `change_request_service` |
| H1 | Roll-up | REQ / POLL | S9 Dashboard (30 s poll) | `GET /reports/rollup` | `report_service` |
| H2 | Expense summary | REQ | S10 | `GET /reports/expenses` | `report_service` |
| H3 | CSV export | REQ | S11 Export | `GET /reports/export` | `report_service` |
| H4 | Audit log query | REQ | S12 | `GET /reports/audit-log` | `report_service` / `audit` |
| I1 | Record inventory movement | REQ | S14 | `POST /inventory/{id}/movements` | `inventory_service` |
| I2 | Stock level and low-stock flag | INT | Every inventory read | `GET /inventory` | `inventory_service` |
| I3 | Record cash count | REQ | S13 Save count | `POST /cash-counts` | `cash_service` |
| J1 | Write audit entry | INT | Every write process | — | `services/audit.py` |
| J2 | Line pricing and totals | INT | F1, G2 | — | `transaction_service` |
| J3 | Send email | INT | B4, C1, D1 | — | `email_service` |
| J4 | Seed starter items | INT | C1 | — | `tenant_service` |
| J5 | Revoke sessions | INT | B2, B5, C3, D2 | — | `auth_service.revoke` |
| K1 | Migrate platform DB | DEPLOY / CLI | Release step 1 | `alembic -x target=platform upgrade head` | `backend/alembic/` |
| K2 | Migrate all tenant DBs | DEPLOY / CLI | Release step 2 | `scripts/migrate_all_tenants.py` | `alembic` tenant tree |
| K3 | Tenant schema version check | INT | Tenant engine first load (C4) | `get_tenant_session` | `tenant_db.py` |
| K4 | Nightly backup | SCHED | Nightly timer | `scripts/backup_all.py` | `pg_dump` |
| K5 | Restore one tenant | CLI | Operator, from a runbook | `scripts/restore_tenant.py` | `pg_restore` |
| K6 | Orphan identity check | SCHED | Nightly timer | `scripts/check_orphan_identities.py` | — |
| K7 | Health and readiness | INFRA | Load balancer / platform | `GET /healthz`, `GET /readyz` | `main.py` |

---

## A. Per-request pipeline

These run in order on every request before the route handler. Each request gets one tenant DB session, which is committed or rolled back when the request ends.

### A1. Request ID and logging
- **Trigger:** every request.
- **Does:** assigns a `request_id`, returns it as `X-Request-Id`, and writes one JSON log line per request with `tenant`, `user_id`, route, status and duration. The same ID is written to `audit_log.request_id`. Passwords, tokens and CSRF values are never logged.

### A2. Rate limiting
- **Trigger:** every request.
- **Does:** `/auth/login` and `/auth/reset-password/*` allow 10 requests per minute per IP and per email. All other routes allow 300 per minute per session. Going over the limit returns `429` with `Retry-After`.

### A3. Session authentication (`get_current_user`)
- **Trigger:** every request that isn't public.
- **Does:**
  1. Hashes the `baymax_session` cookie and loads the matching `sessions` row.
  2. Requires the session to be unrevoked, unexpired (12 h) and idle for less than 8 h. `last_seen_at` is updated at most once a minute.
  3. Loads the identity, and for tenant users the `users` row from the tenant DB.
- **Role:** read again on every request, so deactivating or demoting someone takes effect immediately.
- **Errors:** `401`.

### A4. CSRF check
- **Trigger:** every `POST`, `PATCH` and `DELETE`.
- **Does:** compares `X-CSRF-Token` against the session's token in constant time. `SameSite=Lax` cookies are a second layer.

### A5. Role check (`require_role`)
- **Trigger:** every protected route, using the access matrix in `api_spec.md`.
- **Sensitive actions:** setting prices and approving are checked here and checked again inside the service, so a new route can't skip the check.

### A6. Tenant DB session (`get_tenant_db`)
- **Trigger:** every route that touches tenant data.
- **Does:** takes the tenant slug from the authenticated session, never from the request. Gets a session from the tenant engine registry (C4), refuses tenants that aren't `active`, and commits or rolls back at the end of the request.
- **Rule:** this is the only way to get a tenant session.

### A7. Error handling
- **Trigger:** any unhandled exception.
- **Does:** rolls back the DB transaction and returns `500` with a generic message and the `request_id`. Full details go to the error tracker (Sentry, still open).
- **Other errors:**
  - Tenant DB unreachable: `503` with `Retry-After`.
  - Tenant mid-migration: `503 tenant_migrating`.

---

## B. Authentication

Auth lives in the platform DB, behind the `auth_service` interface, so it can be replaced by Auth0 later (`system_design.md` § Authentication internals).

### B1. Login
- **Trigger:** REQ `POST /auth/login` from S1. Rate-limited.
- **Steps:**
  1. Normalizes the email and looks up `user_identities`. If no identity is found, it still runs a dummy argon2 check so response timing doesn't reveal whether the email exists.
  2. Rejects with `423` if the account is locked. If the password is wrong, increments `failed_login_count`. Five failures lock the account for 15 minutes.
  3. Checks that the identity is active and, for tenant users, that the tenant is `active` (`403 tenant_suspended` otherwise).
  4. Checks that the tenant `users` row exists and is active.
  5. Creates a 32-byte token, stores its SHA-256 hash in `sessions` with a 12 h expiry, and sets the cookie. Returns the user, the tenant and the `csrf_token`.
- **On success:** `failed_login_count` is reset.

### B2. Logout
- **Trigger:** REQ `POST /auth/logout`.
- **Does:** sets `sessions.revoked_at` and clears the cookie. Returns `204`.

### B3. Restore session
- **Trigger:** REQ `GET /auth/me`, on every SPA page load.
- **Does:** returns the same body as login for the current session, including the CSRF token. Returns `401` if the user isn't logged in.

### B4. Request password reset
- **Trigger:** REQ `POST /auth/reset-password/request` from S1. Rate-limited.
- **Does:** always returns `202`. If the email exists, it creates a single-use `password_reset_tokens` row with a 1 h expiry, storing only the hash, and sends the link through J3.

### B5. Set or reset password
- **Trigger:** REQ `POST /auth/reset-password/confirm` from S2. The user gets there from a reset email (B4) or a set-password email for a new account (C1, D1).
- **Does:** checks that the token is unexpired and unused, checks the password (at least 10 characters, not on the denylist), and stores it as an argon2id hash. Sets `used_at` on the token and revokes all sessions for that identity (J5).
- **Errors:** `400 invalid_or_expired_token`, `422` for a weak password.

---

## C. Tenants and platform

### C1. Provision tenant
- **Trigger:**
  - REQ `POST /admin/tenants` from S3. `saas_admin` only. Returns `202` straight away, and the client polls `GET /admin/tenants/{id}` until the status is `active` or `failed`.
  - CLI `scripts/provision_tenant.py`.
- **Resumable:** each step records its progress on the tenant row and skips work that's already done (`system_design.md` § Tenant provisioning).

| Step | Action |
|---|---|
| 1 | Validates the slug and email. Inserts the `tenants` row with `status = provisioning` |
| 2 | `CREATE ROLE baymax_t_{slug}_app` and `CREATE DATABASE baymax_t_{slug}`, using an autocommit admin connection |
| 3 | Runs the tenant Alembic migrations to `head` |
| 4 | Inserts `tenant_settings` (timezone and currency copied over) and the first `users` row (`tenant_admin`). Seeds starter items (J4) |
| 5 | Inserts the platform `user_identities` row and the set-password token |
| 6 | Sends the set-password email (J3) |
| 7 | Sets `status = active` |

- **On failure:** any exception sets `status = failed` with a `failure_reason`. Because the identity (step 5) is created after the tenant DB (step 4), a login can never point at a database that doesn't exist.
- **Async:** how the work continues after the `202` is not yet defined. See [Open points](#open-points) 1.

### C2. Retry provisioning
- **Trigger:** REQ `POST /admin/tenants/{id}/retry-provisioning` from S3, on a `failed` tenant.
- **Does:** re-runs C1 from step 1 and skips the steps that are already complete. Idempotent. Returns `202`.

### C3. Suspend or reactivate tenant
- **Trigger:** REQ `PATCH /admin/tenants/{id}` with `status`. The same route also changes `name` and `user_limit`.
- **Does:** suspending revokes all of the tenant's sessions (J5) and blocks new logins (B1) and new tenant DB sessions (A6). The data is kept. v1 has no way to drop a tenant.

### C4. Tenant engine load and eviction
- **Trigger:** INT. Runs on the first request for a tenant since the process started or since that tenant's engine was evicted.
- **Does:**
  1. Reads the tenant row, which must be `active`.
  2. Creates an engine with its own pool (3 connections plus 2 overflow), using that tenant's own DB role.
  3. Runs the schema version check (K3).
  4. Caches the engine in an LRU registry of up to 50 engines. When the cap is hit, the least recently used engine is disposed.

---

## D. Users

### D1. Create user
- **Trigger:** REQ `POST /users` from S4. `tenant_admin` only.
- **Does:**
  1. Locks the `tenant_settings` row and checks the active-user limit (`409 user_limit_reached`).
  2. Creates the platform `user_identities` row (`409 email_taken`) and the tenant `users` row.
  3. Creates a set-password token and sends the email (J3). Writes an audit entry (J1).
- **Two databases:** this spans the platform DB and the tenant DB, so it can't be one DB transaction. See [Open points](#open-points) 2.

### D2. Update or deactivate user
- **Trigger:** REQ `PATCH /users/{id}` from S4.
- **Does:** changes `display_name`, `role` or `is_active`. Deactivating someone revokes their sessions (J5). A role change takes effect on that user's next request (A3).
- **Guards:**
  - The tenant must keep at least one active `tenant_admin` (`409 last_tenant_admin`).
  - A user can't change their own role or deactivate themselves (`403`).

---

## E. Items

### E1. Create item
- **Trigger:** REQ `POST /items` from S5. `owner` only.
- **Does:** validates the category and price type. A `fixed` item needs a price and a `variable` item must not have one (CHECK). Names must be unique (`409 duplicate_name`). Writes an audit entry.

### E2. Update item / price change
- **Trigger:** REQ `PATCH /items/{id}` from S5. `owner` only.
- **Does:** a price change applies only to future transactions and is audited as `item.price_change` with the before and after values. Items are retired with `is_active: false`. There is no delete.

---

## F. Transactions

### F1. Create transaction
- **Trigger:** REQ `POST /transactions` from S6 Save. Any role, within the access matrix. The client retries with the same `client_request_id` after a network failure.
- **Does:** one DB transaction in the tenant DB (`system_design.md` § Writing a transaction).
  1. **Idempotency.** If the `client_request_id` already exists with the same actor and payload, returns the original with `200`. If the payload is different, returns `409`.
  2. **Authorize.** Checks the role against the transaction type.
  3. **Validate dates.** The date can't be in the future, and staff can't go back more than 7 days.
  4. **Price lines and total.** Runs J2.
  5. **Status.** Sales and expense are `posted`. An adjustment by an owner or admin is `posted`; an adjustment by staff is `pending`.
  6. **Save.** Inserts the transaction, its lines and an audit entry (J1), then commits.
- **Concurrency:** there's no running balance, so concurrent saves don't block each other. The unique index on `client_request_id` closes the retry race.

### F2. Approve or reject staff adjustment
- **Trigger:** REQ `POST /transactions/{id}/approve` or `/reject` from S8. `owner` or `tenant_admin`, and not the person who created it.
- **Does:** locks the transaction row, requires `pending`, and sets `posted` or `rejected` along with `decided_by`. Writes an audit entry. A reject needs a `decision_note`.

---

## G. Change requests

Saved transactions can't be edited. Fixes go through a change request that someone else has to approve.

### G1. File change request
- **Trigger:** REQ `POST /change-requests` from S7. `staff`.
- **Does:**
  - Requires a `reason` and a `posted` target transaction.
  - Only one `pending` request is allowed per transaction (`409 request_already_pending`).
  - A `kind: void` request has no `proposed` lines. A `kind: correct` request stores the proposed replacement lines as JSONB.
  - Saves with status `pending`. Its count shows in the approvals badge.

### G2. Approve change request
- **Trigger:** REQ `POST /change-requests/{id}/approve` from S8. `owner` or `tenant_admin`. One DB transaction (`system_design.md` § Change-request approval).
  1. `SELECT ... FOR UPDATE` on the request, so two approvers are handled one at a time. Requires `pending` and that the approver isn't the requester (also a DB CHECK).
  2. Locks the target transaction and requires `posted` (`409 target_changed` otherwise).
  3. For `void`, sets the target to `voided`. For `correct`, voids the target and inserts a replacement that is repriced through J2. The replacement links back with `adjusts_transaction_id`, keeps the original creator and `business_date`, and records the approver in `decided_by`.
  4. Marks the request `approved` and writes an audit entry with before and after snapshots.
- **Second approver:** gets `409 not_pending`.

### G3. Reject change request
- **Trigger:** REQ `POST /change-requests/{id}/reject` from S8. Requires a `decision_note`.
- **Does:** the same steps as G2 except step 3. The target transaction doesn't change.

### G4. Cancel change request
- **Trigger:** REQ `POST /change-requests/{id}/cancel` by the requester, while the request is `pending`.
- **Does:** sets the status to `cancelled` and writes an audit entry.

---

## H. Reports

Reports are read-only and computed on demand from `posted` transactions. Nothing is stored in advance in v1.

### H1. Roll-up
- **Trigger:** REQ / POLL `GET /reports/rollup` from S9. The dashboard repeats it every 30 s.
- **Does:**
  1. `period_bounds(period, date, week_starts_on)` works out the date range (daily, weekly, monthly or half-year).
  2. One SQL query adds up the totals by `business_date`.
  3. Missing days are filled with zeros, and the daily rows are summed up to week or month. Half-year groups by month.
  4. `group_by=item` adds a breakdown by item.
- **Targets:** under 500 ms p95, and under 1 s for half-year.

### H2. Expense summary
- **Trigger:** REQ `GET /reports/expenses` from S10.
- **Does:** expense totals by item and period, using the same period logic as H1.

### H3. CSV export
- **Trigger:** REQ `GET /reports/export` from S11. `owner` or `tenant_admin`.
- **Does:**
  - Streams one CSV row per transaction line through a server-side cursor, so memory use stays flat.
  - Writes UTF-8 with a BOM and plain decimal numbers, so the file opens correctly in Excel.
  - Prefixes any cell starting with `=`, `+`, `-` or `@` with `'`, so the spreadsheet won't treat it as a formula.
  - Allows at most 366 days (`422 range_too_large`).
- **Target:** one month in under 5 s.

### H4. Audit log query
- **Trigger:** REQ `GET /reports/audit-log` from S12. `owner` or `tenant_admin`.
- **Does:** a paginated read of `audit_log`, filtered by date, actor, entity and action.

---

## I. Inventory and cash

### I1. Record inventory movement
- **Trigger:** REQ `POST /inventory/{id}/movements` from S14. Any role.
- **Does:** the server sets the sign from `kind`: `stock_in` adds and `used` subtracts. An `adjustment` is sent signed and needs a note. A `used` movement that would take stock below zero is refused (`409 insufficient_stock`). Writes an audit entry.
- **Related routes:** creating and editing inventory items (`POST/PATCH /inventory`) are owner-only CRUD with an audit entry, and follow the same pattern as E1 and E2.

### I2. Stock level and low-stock flag
- **Trigger:** INT, on every `GET /inventory`.
- **Does:** stock is `SUM(quantity)` of the movements, worked out each time it's read. No balance is stored. `low_stock` is `stock <= reorder_level`.

### I3. Record cash count
- **Trigger:** REQ `POST /cash-counts` from S13. Any role.
- **Does:** the server works out `expected_amount`. It never takes it from the client:
  - Starts from the opening amount.
  - Adds the posted sales for the `business_date` and subtracts the posted expenses.
  - Adds sales-side adjustments and subtracts expense-side adjustments.
- **Second shift on the same date:** only counts transactions created after the first count. The first count's counted amount becomes the default opening amount.
- **Saved values:** `expected_amount` and `variance = counted − expected` are stored on the row, so approving a correction later doesn't rewrite history. Cash counts are append-only.
- **Errors:** `409 count_exists` for a `(business_date, shift)` that already has a count.

---

## J. Internal shared processes

### J1. Write audit entry
- **Called by:** every process that writes, in the same DB transaction, so the audit entry commits or rolls back with the change.
- **Does:** records the actor, action, entity, before and after values, and the `request_id`. The app's DB role can only insert into `audit_log`. It has no `UPDATE` or `DELETE` grant.

### J2. Line pricing and totals
- **Called by:** F1 and G2 (repricing a correction).
- **Rules:**
  - Each item must be active and in the list that matches the transaction type.
  - A `fixed` item uses `items.price`, and any price the client sends is ignored. A `variable` item needs a `unit_price` of zero or more.
  - `amount = round_half_up(qty × unit_price, 2)`.
  - `total` is the sum of the rounded line amounts, using Decimal arithmetic throughout.
- **Unit test:** `test_pricing`.

### J3. Send email
- **Called by:** B4 (reset link), C1 step 6 and D1 (set-password link).
- **Does:** prints to the console in dev and sends through the email provider in prod (Resend, still open). See [Open points](#open-points) 3.

### J4. Seed starter items
- **Called by:** C1 step 4, for deliverable 3.2.
- **Does:** inserts a default sales and expense item list so a new tenant can start entering sales straight away.

### J5. Revoke sessions
- **Called by:** B2 (one session), B5 (every session of the identity), C3 (every session of the tenant) and D2 (every session of the user).
- **Does:** sets `revoked_at`, and A3 then rejects the session on its next request.

---

## K. Operator, deploy and scheduled processes

### K1. Migrate platform DB
- **Trigger:** DEPLOY, release step 1. Can also be run by hand.
- **Command:** `alembic -x target=platform upgrade head`.

### K2. Migrate all tenant DBs
- **Trigger:** DEPLOY, release step 2, as `scripts/migrate_all_tenants.py`.
- **Does:**
  - Runs the tenant Alembic tree on every tenant whose status is `active` or `suspended`.
  - Keeps going if one tenant fails, and records the result in `tenant_settings.schema_version`.
  - Exits non-zero with a summary if any tenant failed.
- **Release step 3** starts the new API. Migrations must work with the previous API version: only add columns, and rename or drop in a later release.

### K3. Tenant schema version check
- **Trigger:** INT, when C4 loads a tenant engine.
- **Does:** if the tenant's `schema_version` is older than the API expects, requests for that tenant get `503 tenant_migrating` instead of failing partway through.

### K4. Nightly backup
- **Trigger:** SCHED, nightly, as `scripts/backup_all.py` (deliverable 5.5).
- **Does:** runs `pg_dump` on the platform DB and on each tenant DB, and keeps 30 days of dumps. Point-in-time recovery is added if the host offers it.
- **Restore drill:** must be done before the first external tenant.

### K5. Restore one tenant
- **Trigger:** CLI, an operator following a runbook. `scripts/restore_tenant.py`.
- **Does:** restores one tenant DB from its dump without touching the other tenants.

### K6. Orphan identity check
- **Trigger:** SCHED, nightly (`data_model.md` § Integrity rules).
- **Does:** reports any tenant `users.identity_id` that has no matching platform `user_identities` row. It only reports. It doesn't fix anything.
- **Script:** `scripts/check_orphan_identities.py` (deliverable 5.5).

### K7. Health and readiness
- **Trigger:** INFRA, called by the load balancer or hosting platform.
- **Routes:**
  - `GET /healthz` checks that the process is up.
  - `GET /readyz` checks that the platform DB is reachable. Tenant DBs are left out on purpose, so one broken tenant doesn't take the whole API out of rotation.

---

## L. Not in v1

Listed so later work fits the same model:

| Version | Process | Trigger |
|---|---|---|
| v1.x | Auth0 login: `get_identity` validates a JWT through JWKS, and login, logout and reset become redirects | REQ |
| v1.x | `xlsx` export | REQ |
| v2 | Posted transactions generate journal entries | INT, in the same DB transaction as F1 and G2 |
| v3 | POS ingest: `POST /ingest/pos/transactions` with a per-device API key, going through `transaction_service` with `source = pos` | REQ from a device |
| v3 | Cycle-count import and reconciliation report. This is the first process where a background worker is justified | REQ upload, then probably a worker job |
| Scaling | A `daily_totals` table kept up to date in the same DB transaction as writes, if roll-ups get slow | INT |

---

## Open points

1. **How provisioning runs in the background is not specified.** `POST /admin/tenants` returns `202` and the client polls, but no source doc says how C1 keeps running after the response. Candidates are FastAPI `BackgroundTasks` (in-process, lost if the API restarts mid-run, then recovered with Retry) and a Procrastinate job. This is tied to `arch_overview.md` Open decision 3. Running provisioning from inside the API also means the API needs credentials that can `CREATE DATABASE` and `CREATE ROLE`.
2. **Creating a user spans two databases** (platform identity plus tenant `users` row) with no defined order or rollback. Provisioning handles this with step order and resumability. D1 needs a matching rule, such as tenant row first and then identity, with clean-up on failure. K6 would catch whatever is left over.
3. **Email failure handling.** If J3 fails during D1 or C1 step 6, it isn't defined whether the request fails or the admin gets a "resend link" action. No resend route exists in `api_spec.md`.
4. **Expired sessions and reset tokens are never deleted.** No cleanup job is defined. This is low volume, but a nightly SCHED purge next to K6 would keep the tables small.
5. **Who runs the nightly jobs?** K4 and K6 need a scheduler (host cron, the provider's scheduled jobs, or Procrastinate's periodic tasks). The choice depends on the hosting decision, which is still open.
6. **Rate limiter storage.** An in-memory limiter only works with one API container. Scaling path step 4 (several containers) would need a shared store, either Postgres or Redis.
7. ~~K6 has no script.~~ **Resolved 2026-10-30:** `scripts/check_orphan_identities.py` added to `object_masterlist.md` (5.5).
