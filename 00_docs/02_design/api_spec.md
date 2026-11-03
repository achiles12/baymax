# API Specification

REST API over JSON, served by the FastAPI backend and mounted at `/api/v1` (see `backend/app/api/v1/` in `file_system.md`). Table and field names match `data_model.md`. FastAPI generates the OpenAPI document at `/api/v1/openapi.json`; the frontend types are generated from it.

## Table of Contents

1. [Conventions](#conventions)
2. [Roles and access matrix](#roles-and-access-matrix)
3. [Auth](#auth)
4. [Admin: tenants](#admin-tenants)
5. [Users](#users)
6. [Items](#items)
7. [Transactions](#transactions)
8. [Change requests](#change-requests)
9. [Reports](#reports)
10. [Inventory](#inventory)
11. [Cash counts](#cash-counts)
12. [Error format](#error-format)
13. [Open points](#open-points)

---

## Conventions

| Topic | Rule |
|---|---|
| Base path | `/api/v1` |
| Format | `application/json`, UTF-8. CSV for exports |
| Auth | Opaque session token in an `HttpOnly; Secure; SameSite=Lax` cookie named `baymax_session`. A mutating request must also send the `X-CSRF-Token` header (value from `GET /auth/me`). A later Auth0 migration swaps the cookie for `Authorization: Bearer <JWT>` with no change to the routes |
| Tenant routing | The tenant is taken from the session, **never** from the URL or request body. A client can't ask for another tenant's data, so there is nothing to forge |
| IDs | UUID strings |
| Money | Decimal **strings** with 2 places: `"120.00"`. Never JSON numbers |
| Quantities | Decimal strings: `"2.5"` |
| Dates | `business_date` is `YYYY-MM-DD` in the tenant's timezone. Timestamps are ISO 8601 UTC |
| Pagination | `?limit=50&cursor=<opaque>`; `limit` max 200. Response: `{ "items": [...], "next_cursor": "..." \| null }` |
| Idempotency | `POST /transactions` requires `client_request_id` (UUID). Repeating it returns the original record with `200` instead of creating a duplicate |
| Request tracing | Every response carries `X-Request-Id`; the same id is written to `audit_log.request_id` and the logs |
| Rate limits | `/auth/login` and `/auth/reset-password/*`: 10 requests per minute per IP and per email. Others: 300 per minute per session. Exceeding returns `429` with `Retry-After` |

---

## Roles and access matrix

Roles: `saas_admin` (platform), `tenant_admin`, `owner`, `staff` (tenant). In the UI `staff` is the Employee / branch login.

| Capability | saas_admin | tenant_admin | owner | staff |
|---|:-:|:-:|:-:|:-:|
| Create / suspend tenants | ✅ | | | |
| Create / deactivate users | | ✅ | | |
| View item lists | | ✅ | ✅ | ✅ |
| Create / edit items and prices | | | ✅ | |
| Create sales and expense transactions | | ✅ | ✅ | ✅ |
| Enter a price on a *variable* item at entry | | ✅ | ✅ | ✅ |
| Create adjustment (posts immediately) | | ✅ | ✅ | |
| Create adjustment (goes to `pending`) | | | | ✅ |
| File a change request | | | | ✅ |
| Approve / reject adjustments and change requests | | ✅ | ✅ | |
| View all transactions | | ✅ | ✅ | ✅ |
| View roll-ups and expense reports | | ✅ | ✅ | ✅ |
| Export CSV | | ✅ | ✅ | |
| Record inventory movements | | ✅ | ✅ | ✅ |
| Create / edit inventory items | | | ✅ | |
| Record cash count | | ✅ | ✅ | ✅ |
| View audit log | | ✅ | ✅ | |

A `saas_admin` has no tenant and **cannot read tenant data** (transactions, items, reports). Calls to tenant routes from a `saas_admin` session return `403`.

A request that is authenticated but not allowed gets `403 forbidden`. A missing or expired session gets `401 unauthenticated`.

---

## Auth

### `POST /auth/login`

Public.

Request
```json
{ "email": "lindsay@example.com", "password": "..." }
```
Response `200`: sets the session cookie.
```json
{
  "user": { "id": "…", "display_name": "Lindsay", "email": "lindsay@example.com", "role": "owner" },
  "tenant": { "slug": "lindsay-laundry", "name": "Lindsay Laundry", "timezone": "Asia/Manila", "currency": "PHP" },
  "csrf_token": "…"
}
```
For a SaaS Admin, `tenant` is `null` and `role` is `"saas_admin"`.

Errors: `401 invalid_credentials` (same message whether the email exists or not), `423 account_locked` (after 5 failures, 15 minutes), `403 tenant_suspended`.

### `POST /auth/logout`
Revokes the session (`sessions.revoked_at`) and clears the cookie. `204`.

### `GET /auth/me`
Returns the same body as login for the current session. Used on page load to restore state. `401` if not logged in.

### `POST /auth/reset-password/request`
Public. Body `{ "email": "…" }`. **Always** returns `202`, whether or not the email exists. If it does, an email with a single-use link (1 hour) is sent.

### `POST /auth/reset-password/confirm`
Public. Body `{ "token": "…", "new_password": "…" }`.
`204` on success: the token's `used_at` is set, **all** existing sessions of that identity are revoked.
Errors: `400 invalid_or_expired_token` (also when already used), `422` for a weak password (min 10 characters).

---

## Admin: tenants

`saas_admin` only.

### `GET /admin/tenants`
List tenants: `id, slug, name, status, created_at, active_user_count`.

### `POST /admin/tenants`
Provision a tenant: creates the row, creates `baymax_t_{slug}`, runs tenant migrations, creates the first Tenant Admin identity and `users` row, and emails a set-password link.

Request
```json
{
  "name": "Lindsay Laundry",
  "slug": "lindsay-laundry",
  "timezone": "Asia/Manila",
  "currency": "PHP",
  "tenant_admin": { "email": "inigo@example.com", "display_name": "Iñigo" }
}
```
Response `202 Accepted` with `{ "id": "…", "status": "provisioning" }`. Provisioning takes a few seconds; the client polls `GET /admin/tenants/{id}` until `status` is `active` or `failed`.

Errors: `409 slug_taken`, `409 email_taken`, `422 invalid_slug`.

### `GET /admin/tenants/{id}`
Details including `status` and, if `failed`, `failure_reason`.

### `PATCH /admin/tenants/{id}`
Body: any of `{ "name", "status": "active" | "suspended", "user_limit" }`. Suspending revokes that tenant's sessions and blocks new logins. Slug and timezone can't be changed here.

### `POST /admin/tenants/{id}/retry-provisioning`
Re-runs provisioning for a tenant in `failed` state. Idempotent. `202`.

---

## Users

`tenant_admin` only (except `GET /users/me`).

### `GET /users`
List users of the tenant: `id, display_name, email, role, is_active`.

### `POST /users`
```json
{ "email": "mathilda@example.com", "display_name": "Mathilda", "role": "staff" }
```
Creates the identity (platform DB) and the `users` row (tenant DB), then emails a set-password link. `201`.
Errors: `409 user_limit_reached` (already 5 active users), `409 email_taken`, `422 invalid_role` (`saas_admin` can't be created here).

### `PATCH /users/{id}`
Body: any of `{ "display_name", "role", "is_active" }`. Deactivating revokes the user's sessions. A tenant must always keep at least one active `tenant_admin` (`409 last_tenant_admin`). A user can't change their own role or deactivate themselves (`403`).

---

## Items

### `GET /items?category=sales|expense|adjustment&include_inactive=false`
All roles. Sorted by `sort_order`. Response items:
```json
{ "id": "…", "category": "sales", "name": "Wash", "price_type": "fixed", "price": "60.00", "sort_order": 1, "is_active": true }
```

### `POST /items` · `owner`
```json
{ "category": "sales", "name": "Fold", "price_type": "fixed", "price": "25.00", "sort_order": 3 }
```
For `variable`, omit `price` (sending one is `422`). `201`. Errors: `409 duplicate_name`.

### `PATCH /items/{id}` · `owner`
Any of `name, price_type, price, sort_order, is_active`. A price change applies to future transactions only and is written to `audit_log` (`item.price_change`, before/after).

There is no `DELETE`. Retire an item with `is_active: false`.

---

## Transactions

### `POST /transactions`

All roles, with the rules in the access matrix.

```json
{
  "client_request_id": "0b9d…",
  "type": "sales",
  "business_date": "2026-10-29",
  "note": null,
  "lines": [
    { "item_id": "…wash", "quantity": "2",   "unit_price": null },
    { "item_id": "…soap", "quantity": "1",   "unit_price": null },
    { "item_id": "…fold", "quantity": "3.5", "unit_price": "18.00" }
  ]
}
```

Rules:
- `unit_price` is **ignored for fixed items**; the server uses `items.price`. For *variable* items it is required.
- For `type: "adjustment"`, `adjusts_type` (`sales` | `expense`), `direction` (`1` | `-1`) and `note` are required; `adjusts_transaction_id` is optional.
- Every `item_id` must belong to the list that matches `type` (a sales transaction can't use an expense item) and be active.
- `business_date` may not be in the future, and may not be more than 7 days in the past for `staff` (older entries need an owner).
- At least 1 line; at most 100.

Response `201` (or `200` for a repeated `client_request_id`):
```json
{
  "id": "…", "type": "sales", "status": "posted", "business_date": "2026-10-29",
  "total": "208.00", "created_by": { "id": "…", "display_name": "Mathilda" },
  "created_at": "2026-10-29T03:14:07Z",
  "lines": [
    { "line_no": 1, "item_id": "…", "item_name": "Wash", "quantity": "2", "unit_price": "60.00", "amount": "120.00" }
  ]
}
```
A `staff` adjustment returns `status: "pending"`.

Errors: `422 validation_error` (per-field detail), `422 price_required`, `422 wrong_item_category`, `403 forbidden`.

### `GET /transactions`

All roles. Filters: `from`, `to` (business dates), `type`, `status`, `created_by`, `item_id`, `limit`, `cursor`. Sorted by `business_date` desc, `created_at` desc. Lines are omitted in the list; `total` is included.

### `GET /transactions/{id}`
One transaction with its lines and, if any, its change requests and `decided_by`.

### `GET /transactions/{id}/lines`
The lines alone (for the table on the list page).

There is no `PUT`, `PATCH` or `DELETE` on transactions. Corrections go through [change requests](#change-requests).

---

## Change requests

### `POST /change-requests` · `staff`

```json
{
  "transaction_id": "…",
  "kind": "correct",
  "reason": "Customer had 3 washes, not 2",
  "proposed": { "lines": [ { "item_id": "…wash", "quantity": "3", "unit_price": null } ] }
}
```
`kind: "void"` takes no `proposed`. `reason` is required. Only a `posted` transaction can be targeted, and only one `pending` request per transaction is allowed (`409 request_already_pending`). `201`, status `pending`.

### `GET /change-requests?status=pending`
`staff` see their own; `owner` and `tenant_admin` see all.

### `GET /change-requests/{id}`
Includes the target transaction, the proposed lines, and a computed diff of old vs proposed total.

### `POST /change-requests/{id}/approve` · `owner`, `tenant_admin`
Optional body `{ "decision_note": "…" }`. In one DB transaction: applies the change (void, or void-and-replace), marks the request `approved`, writes `audit_log`. `200` with the updated request.
Errors: `403 cannot_approve_own_request`, `409 not_pending`, `409 target_changed` (the target is no longer `posted`).

### `POST /change-requests/{id}/reject` · `owner`, `tenant_admin`
Body `{ "decision_note": "…" }` (required). `200`.

### `POST /change-requests/{id}/cancel` · requester only
While `pending`. `200`.

### Pending staff adjustments
A staff-created adjustment (`status: "pending"`) is approved or rejected through the same pair of endpoints:

- `POST /transactions/{id}/approve`
- `POST /transactions/{id}/reject` (body `{ "decision_note": "…" }`)

Both `owner` and `tenant_admin`, and not the creator. `GET /transactions?status=pending` feeds the Approvals page together with `GET /change-requests?status=pending`.

---

## Reports

All roles can read roll-ups. Only `owner` and `tenant_admin` can export. All figures count `posted` transactions only.

### `GET /reports/rollup`

| Param | Values |
|---|---|
| `period` | `daily` \| `weekly` \| `monthly` \| `half-year` |
| `date` | Any `business_date` inside the wanted period. Default: today |
| `group_by` | `none` (default) \| `item` |

Response
```json
{
  "period": "weekly",
  "from": "2026-10-26", "to": "2026-11-01",
  "totals": { "sales": "18450.00", "expense": "6120.00", "adjustments": "-200.00", "net": "12130.00" },
  "series": [
    { "date": "2026-10-26", "sales": "2410.00", "expense": "830.00", "adjustments": "0.00", "net": "1580.00" }
  ],
  "by_item": [
    { "item_id": "…", "name": "Wash", "category": "sales", "quantity": "310", "amount": "18600.00" }
  ]
}
```
`series` is one row per day for daily/weekly/monthly and one row per month for half-year. `by_item` is present only when `group_by=item`.

### `GET /reports/expenses`
Params as above. Expense totals by item and period, for the expense dashboard.

### `GET /reports/export` · `owner`, `tenant_admin`
Params: `from`, `to`, `type` (optional), `format=csv` (default). Streams `text/csv` with one row per transaction line: `business_date, transaction_id, type, status, item, quantity, unit_price, amount, created_by, created_at`. Amounts are plain decimals, so the file opens correctly in Excel. `format=xlsx` is planned for v1.x. Max range 366 days (`422 range_too_large`).

### `GET /reports/audit-log` · `owner`, `tenant_admin`
Filters: `from`, `to`, `actor_id`, `entity`, `action`; paginated.

---

## Inventory

### `GET /inventory`
All roles. Each inventory item with derived `stock` and a `low_stock` boolean (`stock <= reorder_level`).

### `POST /inventory` · `owner`
`{ "name": "Detergent", "unit": "bottle", "reorder_level": "5" }`. `201`.

### `PATCH /inventory/{id}` · `owner`
`name, unit, reorder_level, is_active`.

### `POST /inventory/{id}/movements`
All roles.
```json
{ "kind": "used", "quantity": "1", "business_date": "2026-10-29", "note": null }
```
Send `quantity` as a positive number; the server applies the sign from `kind` (`stock_in` +, `used` −). For `adjustment`, send a signed quantity and a `note`.
Errors: `409 insufficient_stock` for a `used` movement that would take stock below zero.

### `GET /inventory/{id}/movements?from=&to=`
History.

---

## Cash counts

### `POST /cash-counts`
All roles.
```json
{ "business_date": "2026-10-29", "shift": "day", "opening_amount": "1000.00", "counted_amount": "3480.00", "note": null }
```
The server computes `expected_amount` from that date's posted transactions and stores `variance = counted − expected`. `201` returns all three. Errors: `409 count_exists` for the same `(business_date, shift)`.

### `GET /cash-counts?from=&to=`
All roles. A negative variance is highlighted in the UI.

---

## Error format

Every error uses the same body:

```json
{
  "error": {
    "code": "user_limit_reached",
    "message": "This tenant already has 5 active users.",
    "details": { },
    "request_id": "01J…"
  }
}
```

`code` is a stable machine-readable string; `message` is for people. For `422 validation_error`, `details` maps field paths to messages (`"lines.0.quantity": "must be greater than 0"`).

| Status | Used for |
|---|---|
| `400` | Malformed request, invalid or expired token |
| `401` | No session, expired session |
| `403` | Authenticated, not allowed (role, own-request approval, suspended tenant) |
| `404` | Not found **or belongs to another tenant** (never reveal which) |
| `409` | State conflict (limits, duplicates, not pending, idempotent mismatch) |
| `422` | Validation failure |
| `423` | Account locked |
| `429` | Rate limited |
| `500` | Unexpected; message is generic and `request_id` is the lookup key |

---

## Open points

1. **Pending staff adjustments** use `/transactions/{id}/approve|reject`, separate from `/change-requests/*`. If the Approvals page feels better with one queue, the API could instead auto-create a change request for each staff adjustment. `file_system.md` currently lists only `change_requests.py`, so the two approve/reject routes above would be added to `transactions.py`.
2. **`xlsx` export** is deferred; CSV opens in Excel.
3. **Webhook / bulk ingest endpoints** for POS (v3) and machine cycle counts (v3) are intentionally not specified yet.
