# Stack Comparison

Compares the **current stack** (baseline, as used in `file_system.md`) with a **suggested alternative** stack, then lists **add-ons** that fit the current stack.

The current stack remains the baseline. Nothing here replaces it unless decided.

Constraints that drive the comparison (from `prod_req_doc.md`):
- Multi-tenant, **database separated**: one platform DB + one DB per tenant
- Auth on the DB at launch, planned migration to **Auth0**
- Responsive UI (phone / tablet / browser), fast entry for low-tech-comfort staff
- Roll-ups: daily, weekly, monthly, half-year; export to CSV / Excel
- v3: ingest laundry machine cycle counts and reconcile against sales


---

## 1. Current Stack (baseline)

| Layer | Choice |
|---|---|
| Frontend | React + TypeScript + Vite + Tailwind |
| API | Python — FastAPI |
| DB access | SQLAlchemy |
| Validation | Pydantic |
| Migrations | Alembic (platform tree + tenant tree) |
| Database | PostgreSQL |
| Tests | pytest + httpx (backend), Playwright (E2E) |
| CI | GitHub Actions |
| Local dev | Docker Compose |

Node.js is used for frontend tooling only (Vite, npm).

<!--


-->

---

## 2. Suggested Alternative — Node.js API + Python worker

| Layer | Choice | Why |
|---|---|---|
| Frontend | React + TypeScript + Vite + Tailwind | Unchanged |
| API | **Node.js + TypeScript (Fastify)** | Same language as the frontend, so types and validation are shared |
| DB access | **Kysely** + `pg` driver | Easy to keep one connection pool per tenant DB. Prisma assumes one database fixed when the app is built, which doesn't fit database-per-tenant |
| Validation | **Zod** | Define once, use in both API and frontend forms |
| Migrations | Kysely migrator | Platform set + tenant set, run against every tenant DB |
| Python worker | **Python** (pandas / openpyxl) | Roll-ups, CSV / Excel export, v3 cycle-count import |
| Job queue | **pg-boss** | Jobs stored in Postgres, no Redis needed |
| Database | PostgreSQL 16 | Platform DB + one DB per tenant |
| Tests | Vitest (API + UI), pytest (worker), Playwright (E2E) | |
| Repo | pnpm workspaces: `apps/web`, `apps/api`, `packages/shared`, `worker/` | `packages/shared` holds the Zod schemas and types |
| Hosting | Long-running containers (Docker on a VPS, Railway, Fly or Render) | See hosting note below |

---

## 3. Side-by-side

| Concern | Current (FastAPI) | Suggested (Node API + Python worker) |
|---|---|---|
| Languages on the backend | 1 (Python) | 2 (TypeScript + Python) |
| Type sharing with frontend | Manual, or generated from FastAPI's OpenAPI schema | Direct, through `packages/shared` |
| Validation | Pydantic on the API; frontend rules written separately | Zod in one place for both |
| Per-tenant DB connections | SQLAlchemy engine cache per tenant (`tenant_db.py`) | `pg` pool cache per tenant |
| Migrations per tenant | Alembic with `-x tenant=<slug>` | Kysely migrator looped over tenant DBs |
| Reports / Excel / data work | Same codebase (pandas, openpyxl) | Separate Python worker |
| v3 machine data import | Same codebase | Python worker |
| Number of things to deploy | API + frontend | API + worker + frontend |
| Familiarity | Already used on the previous project | New for the API layer |
| Effort to start v1 | Lower — sample layout already exists | Higher — new layout and toolchain |

**Summary:** the current stack is simpler and already meets the Node + Python + Postgres preference. The alternative is worth it mainly for one language across frontend and API. Its Python worker only pays off at v3.

---

## 4. Add-ons for the Current Stack

These add to the current stack. They don't replace any part of it.

### Authentication

| Add-on | Use | When | Notes |
|---|---|---|---|
| **Auth0** | Hosted login, password reset, MFA | v1.x (planned migration) | Auth0 **Organizations** can map one-to-one to tenants. Keep `user_identities.tenant_id` in the platform DB so routing to the tenant DB doesn't change. |
| Auth provider interface | `auth_service.py` behind a small interface (`verify_credentials`, `issue_session`, `get_identity`) | v1 | DB auth at launch, then swap to Auth0 by implementing the same interface. Keeps the migration to a single module. |
| PyJWT / Authlib | Verify Auth0-issued JWTs in FastAPI dependencies | With Auth0 | Replaces DB session lookup in `get_current_user` |
| Clerk (alternative) | Hosted auth with prebuilt React components | Instead of Auth0 | Listed in the idea doc. Pick one; the PRD names Auth0. |

### Backend (Python)

| Add-on | Use | When |
|---|---|---|
| **uv** | Fast Python package and virtualenv management | v1 |
| **Ruff** | Linting + formatting | v1 |
| **mypy** or **pyright** | Type checking | v1 |
| **Procrastinate** or **arq** | Background jobs: scheduled roll-ups, exports. Procrastinate uses Postgres; arq needs Redis | v1 if roll-ups get slow, otherwise v2 |
| **pandas + openpyxl** | CSV / Excel export, roll-up calculations | v1 (export is wanted) |
| **structlog** | Structured logs, with tenant slug and user on every line | v1 |

### Frontend

| Add-on | Use | When |
|---|---|---|
| **TanStack Query** | API data fetching, caching, refetch | v1 |
| **React Hook Form + Zod** | Transaction entry and item maintenance forms | v1 |
| **Recharts** | Roll-up and expense dashboard charts | v1 |
| **openapi-typescript** | Generate frontend types from FastAPI's OpenAPI schema | v1 — gives most of the type-sharing benefit of the Node alternative |
| **PWA (vite-plugin-pwa)** | Installable on the branch tablet / phone | v1.x |

### Platform and operations (from the idea doc's learning list)

| Add-on | Use | When | Fit |
|---|---|---|---|
| **GitHub Actions** | CI: lint, type check, pytest, Playwright | v1 | Already planned |
| **Resend** | Transactional email (password reset, approval notifications) | v1 | Good fit |
| **Namecheap** | Domain | v1 | Good fit |
| **Cloudflare** | DNS, TLS, proxy / caching in front of the app | v1 | Good fit |
| **Sentry** | Error tracking for API and frontend | v1 | Good fit |
| **PostHog** | Product analytics (which screens staff use, entry speed) | v1.x | Optional |
| **Stripe** | Per-tenant subscription billing | When onboarding paying tenants | Good fit |
| **Grafana + Prometheus** | Metrics dashboards and alerts | v2 | Optional at this scale |
| **Upstash (Redis)** | Cache / job queue | Only if arq is chosen | Optional |
| **Supabase** | Hosted Postgres | — | Weak fit: database-per-tenant means one Supabase project per tenant |
| **Vercel** | Hosting | Frontend only | Fine for the static React build; not for the API, which needs long-lived per-tenant connection pools |
| **Pinecone / vector DB** | — | — | Not needed; no search or AI features in the PRD |
| **Jenkins / Nexus** | — | — | Not needed; GitHub Actions covers CI/CD |

---

## Hosting note

Database-per-tenant needs a long-running API process that keeps one connection pool per tenant. Either stack should run as containers (Docker on a VPS, Railway, Fly or Render) with managed or self-hosted Postgres. The React build can be served from Cloudflare Pages or Vercel.

---

## Open decisions

1. Keep FastAPI (current) or move the API to Node.js (alternative)
2. Auth0 vs Clerk, and when to migrate off DB auth
3. Background jobs in v1 (Procrastinate) or SQL-only roll-ups until v2
4. Hosting provider for the API and Postgres

# Stack Decision

**Accepted 2026-11-05: the stack proceeds as written below.** The 3 Nov review fixes are not applied. Not decided, and carried as open items: Zitadel vs Auth0 and the `AuthProvider` interface note; PWA/offline queue; Alembic platform/tenant trees and provisioning (covered by deliverables 2.6, 2.7, 2.15); Python and frontend tooling rows; email and alert providers; hardware, UPS and uptime promise. Decide each when its milestone needs it.
## Application stack

| Layer | Pick 
|---|---|

| Frontend | React + TypeScript + Vite + Tailwind |
| API and Backend| Python — FastAPI |
| Database | **PostgreSQL** (self-hosted) |
| SaaS Admin console | Admin section inside the React app (SaaS Admin role only), backed by FastAPI routes on the platform DB. Retool or Appsmith if a faster back office is needed. this will be provisioned to prod admin and not during dev |
| Auth | Zitadel  | `not Supabase weak for database per tenant; Auth0 in a future state` |
| Audit trail | Append-only `audit_log` table in each tenant DB, written by the service layer in the same transaction as the change. Revoke `UPDATE`/`DELETE` on it for the app role |
| Background jobs | **Procrastinate** (Postgres-backed) `or **Celery + Redis**  Procrastinate avoids running Redis. Needed for rollups and exports `|
| Excel/CSV export | openpyxl `openpyxl XlsxWriter` | Standard |
| Money | Python `Decimal` + Pydantic `Decimal` + PostgreSQL `NUMERIC(12,2)`. Never floats |
| Tests | `pytest` + `httpx` (backend) against real PostgreSQL via Testcontainers, `Playwright` (E2E) |


## Running it locally

| Concern | Pick |
|---|---|
| Packaging and deploy | **Docker Compose**: app, Postgres, job worker, reverse proxy |
| Reverse proxy / HTTPS | **Caddy** (automatic HTTPS) |
| Public access without a static IP | **Cloudflare Tunnel**: works behind dynamic IPs and CGNAT (common with PH ISPs) and hides the home IP |
| Remote admin / SSH | **Tailscale**: private access, no open ports |
| Backups | `pg_dump` or **pgBackRest** on a schedule, copied off-site with **restic** or **rclone** to Cloudflare R2 or Backblaze B2. Practice restores |
| Uptime alerts | **Uptime Kuma** (self-hosted) plus an external check from outside your network |
| Errors | **Sentry** (cloud free tier) or **GlitchTip** (self-hosted) |
| Metrics/logs (optional) | Grafana + Prometheus + Loki, only when needed |
| CI/CD | **GitHub Actions** runs tests, then deploys over Tailscale/SSH |
| Secrets | `.env` kept out of git; Infisical or Doppler later |
| Payments | Keep this open for now `PayMongo or Xendit once billing starts. `|
| Email | Keep Open for now `Resend`|
| Alerts | Keep Open for now `Semaphore or Viber`|
| Admin Console | Use DB Beaver for Me as Developer `Maybe Directus in future. Appsmith or Retool` |