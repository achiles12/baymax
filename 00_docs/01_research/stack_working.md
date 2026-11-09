### Application stack

| Layer | Pick 
|---|---|

| Frontend | React + TypeScript + Vite + Tailwind |
| API and Backend| Python — FastAPI |
| Database | **PostgreSQL** (self-hosted) |
| SaaS Admin console | Admin section inside the React app (SaaS Admin role only), backed by FastAPI routes on the platform DB. Retool or Appsmith if a faster back office is needed |
| Auth | Zitadel  | `not Supabase weak for database per tenant; Auth0 in a future state` |
| Audit trail | Append-only `audit_log` table in each tenant DB, written by the service layer in the same transaction as the change. Revoke `UPDATE`/`DELETE` on it for the app role |
| Background jobs | **Procrastinate** (Postgres-backed) `or **Celery + Redis**  Procrastinate avoids running Redis. Needed for rollups and exports `|
| Excel/CSV export | `openpyxl` / `XlsxWriter` | Standard |
| Money | Python `Decimal` + Pydantic `Decimal` + PostgreSQL `NUMERIC(12,2)`. Never floats |
| Tests | `pytest` + `httpx` (backend) against real PostgreSQL via Testcontainers, `Playwright` (E2E) |


### Running it locally

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
| Admin Console | Use DB Beaver for now `Maybe Directus in future. Appsmith or Retool` |


<!--
rate the stack out of 100. give reasons why. give suggestions how to improve

● Score: 76/100

  ┌────────────────────┬───────┬────────────────────────────────────────────┐
  │        Area        │ Score │                   Notes                    │
  ├────────────────────┼───────┼────────────────────────────────────────────┤
  │                    │       │ Postgres, FastAPI and SQLAlchemy suit      │
  │ Fit to             │ 17/20 │ database-per-tenant. The audit table       │
  │ requirements       │       │ matches the anti-pilferage goal. Money     │
  │                    │       │ handling is correct.                       │
  ├────────────────────┼───────┼────────────────────────────────────────────┤
  │                    │       │ Python + React + Postgres is a small,      │
  │ Simplicity and     │ 14/20 │ familiar set of parts. But it needs a      │
  │ maintainability    │       │ separate admin UI and a hand-built audit   │
  │                    │       │ trail.                                     │
  ├────────────────────┼───────┼────────────────────────────────────────────┤
  │                    │       │ Tenant provisioning and the per-tenant     │
  │ Risk coverage      │ 11/20 │ migration runner aren't covered yet. The   │
  │                    │       │ self-hosted risks are covered only by a    │
  │                    │       │ list of tools.                             │
  ├────────────────────┼───────┼────────────────────────────────────────────┤
  │                    │       │ Caddy, Cloudflare Tunnel, Tailscale and    │
  │ Ops and cost       │ 16/20 │ restic suit PH ISPs with CGNAT. Backups    │
  │ realism            │       │ are specified, and restores are to be      │
  │                    │       │ practiced.                                 │
  ├────────────────────┼───────┼────────────────────────────────────────────┤
  │ Completeness and   │ 12/20 │ Several gaps, listed below.                │
  │ consistency        │       │                                            │
  ├────────────────────┼───────┼────────────────────────────────────────────┤
  │                    │       │ The auth swap to Auth0 is planned. The v3  │
  │ Future-proofing    │ 6/10  │ machine-data import and offline entry are  │
  │                    │       │ open.                                      │
  └────────────────────┴───────┴────────────────────────────────────────────┘

  What's strong
  - Postgres on both the test and runtime side, with no mocks, which matches
    your test strategy.
  - Auth0 now chosen and the Supabase reasoning recorded.
  - A real access path for a home server (Tunnel for public access, Tailscale
    for admin) instead of open ports.
  - Money as NUMERIC and Decimal end to end.
  - The audit log is enforced at the database level by revoking UPDATE and
    DELETE, not just by convention.

  What lowers the score
  1. Auth0 as a hosted dependency conflicts with "auth in the DB at launch."
     prod_req_doc.md says database auth first, then Auth0. The table now reads
     as Auth0 from day one, and there's no AuthProvider interface row.
  2. Offline entry is unaddressed. Your own tools.md flags flaky PH connectivity
     and a single home server as risks. A React SPA with no PWA or
     queue-and-sync plan leaves transactions lost when Wi-Fi drops.
  3. Self-hosted plus the Auth0 dependency adds an outage path. The server being up doesn't
     help if the internet or Auth0 is down.
  4. The migration and provisioning story is missing. There's no row for the Alembic
     platform and tenant trees, or for the script that creates a tenant database. That is
     the riskiest piece, and tools.md says to prototype it first. **!!!**
  5. Gaps in the table:
     - No linting, type checking or package management (uv, Ruff, mypy). **!!!**
     - No frontend data layer (TanStack Query, React Hook Form + Zod). **!!!**
     - No charts for the roll-up screens. **!!!**
     - No openapi-typescript for type sharing. **!!!**
     - No email provider, though password resets and approval alerts need one. **!!!**
     - No SMS or Viber for owner alerts. **!!!**
  6. Admin console is hand-built. A custom admin section costs time for a back office with
     a handful of users. **!!What is a prebuit admin console!!**
  7. Leftover inconsistencies:
     - "API" and "Backend" are duplicate rows.
     - "Procrastinate or Celery + Redis" is still undecided. **!!Procrastinate!!**
     - Excel export lists openpyxl or XlsxWriter without choosing one. **!!openpyxl!!**
     - The Auth0 note is in a stray cell.
     - Table columns are uneven.

  How to improve
  1. Fix the auth line. Say "DB auth behind an AuthProvider interface at launch, Auth0 in
     v1.x", and keep the Supabase note.
  2. Add a PWA row. Use vite-plugin-pwa plus a local queue (IndexedDB) for entry, so the
     build supports offline from the start.
  3. Add rows for tenant provisioning and migrations. Alembic with a platform tree and a
     tenant tree, plus a provisioning script. Build this prototype first.
. uv, Ruff, mypy, structlog, TanStack Query, React Hook
     Form + Zod, Recharts and openapi-typescript.
  5. Decide the open choices. Pick Procrastinate (no Redis) and one Excel library
     (openpyxl, also used for reading cycle-count files in v3). Merge "API" and "Backend"
     into one row.
  6. Consider Appsmith for the admin console in v1. Hand-build a React admin only when the
     back office outgrows it.
  7. Add an email provider (Resend) and owner alerts (Semaphore or Viber). These are the
     outbound channels the approval flow depends on.
  8. Add a hardware and uptime row. UPS, SSD mirror, restore drills and a stated
     availability promise, since one machine holds other businesses' records and falls
     under the Data Privacy Act.

  I haven't changed any file for this review. I can apply the fixes in items 1 to 5 and 7
  to stack_working.md if you want.
-->