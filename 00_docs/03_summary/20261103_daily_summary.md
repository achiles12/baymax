# Daily summary: 2026-11-03

Covers work since the last summary (2026-10-30). The project is in **M0 Design and decisions**, which the schedule starts on 2 Nov 2026. This session's work was all on the stack decision (deliverable 0.12). No code exists yet.

## Objects built

No code objects. No design docs were completed today.

| Item | Deliverable | What happened | Status |
|---|---|---|---|
| `01_research/stack_working.md` | 0.12 (input) | Filled the four empty rows (SaaS Admin console, Audit trail, Background jobs, Excel/CSV export and the rest of the Application stack and Running-it-locally tables) without touching existing rows. Then rated the stack and pasted the review into the file. | 🔨 Working draft, uncommitted |
| `01_research/tools.md` | 0.12 (input) | Read and used as the source for the self-hosted choices (Caddy, Cloudflare Tunnel, Tailscale, pgBackRest/restic, Uptime Kuma, Sentry/GlitchTip). Has uncommitted changes. | Reference |
| `01_research/stack_comparison.md` | 0.12 | Compared the current FastAPI stack with the tools.md suggestion (Django + HTMX). Not edited. The decision is not recorded here yet. | 🔨 Decision pending |

### Decisions and review results

- **Kept:** FastAPI + React/TypeScript/Vite/Tailwind + PostgreSQL (self-hosted). Recommended over Django + HTMX because offline-capable entry favors an SPA/PWA.
- **Auth:** you set the Auth row to Auth0, with a note that Supabase is weak for database-per-tenant.
- **Stack rating: 76/100.**
  - Strengths: real-Postgres testing, `Decimal`/`NUMERIC` money, a DB-enforced append-only audit log, and a workable access path for a home server.
  - Gaps: tenant provisioning and migrations aren't covered, offline entry has no plan, and the tooling rows are missing.
- **Your marks on the review:** Procrastinate over Celery; openpyxl over XlsxWriter; you flagged the missing items as `!!!` and asked what a prebuilt admin console is.

### Testing status

Nothing to test yet. `test_strat.md` (0.11) is still the AthleteFlow version.

### Where this fits

The stack decision (0.12) is the only M0 item that blocks M1. It is now partly made but not closed:

| Part of 0.12 | State |
|---|---|
| FastAPI vs Node | Leaning FastAPI; written into `stack_working.md`, not into `stack_comparison.md` |
| Auth0 vs Clerk | Auth0 written in the table. Conflicts with the PRD (DB auth at launch, Auth0 later) until the auth row says "behind an `AuthProvider` interface" |
| Hosting | Self-hosted (Docker Compose, Caddy, Cloudflare Tunnel, Tailscale). Hardware, UPS and uptime promise not stated |
| Email provider | Not chosen (Resend suggested) |
| Job queue / Excel library | Chosen in your marks (Procrastinate, openpyxl); not yet applied to the table |

I made no edits to `stack_working.md` after the review. The listed fixes are still pending.

### Housekeeping

- Docs were reorganised since 10-30 (`00_planning/`, `01_research/`, `02_design/`, `03_summary/`, `04_todos/`, `99_sample/`). `CLAUDE.md`, the masterlist and the skills still point at the old `00_docs/design/`, `00_docs/summary/` and `00_docs/todos/` paths. The stale `CLAUDE.md` says this isn't a git repo, but it is (branch `main`, one commit).
- Uncommitted: `stack_working.md` (new, staged-add with unstaged edits) and `tools.md`.
