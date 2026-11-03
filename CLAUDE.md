# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

Project Baymax is in the **design phase**. There's no code yet: `backend/`, `frontend/` and `scripts/` are empty, and there's no build, lint or test tooling. This isn't a git repository yet either. Add commands to this file once tooling exists.

## What Baymax is

Baymax is a multi-tenant SaaS for laundromat operators. It records daily sales, expense and adjustment transactions, then rolls them up into daily, weekly, monthly and half-year summaries. Its goals are to replace pen-and-paper logbooks and spreadsheets and to deter staff pilferage. The first tenant is the owner's own laundromat.

Key requirements (from `00_docs/design/prod_req_doc.md` and `00_docs/baymax_laundry_daily_transaction_record_idea.md`):
- **Multi-tenancy:** each tenant gets its own database, not just its own schema. Pricing is per tenant, with up to 5 users each.
- **Auth:** authentication is stored in the database at launch and must be designed so it can migrate to Auth0 later.
- **Roles:** SaaS Admin (onboards tenants), Tenant Admin (creates users within a tenant), Franchise Owner (maintains item lists and prices, approves adjustments and change requests) and Employee/branch login (enters transactions at the POS).
- **Transactions:** there are three types: sales, expense and adjustment. Each one holds line items from lists the owner maintains. Items have either a fixed price set by the owner or a variable price. Staff can't edit records directly: they file a change request or adjustment, and the owner approves it, so an audit trail is needed.
- **UI:** must be responsive across phone, tablet and desktop. Employees have low tech comfort, so entry must be quick and intuitive.
- **Later versions:** v2 adds full bookkeeping. v3 adds POS integration and ingesting cycle counts from laundry machines (LG washers) to reconcile against sales. CSV/Excel export is wanted.

## Documentation layout

- `00_docs/design/` holds the authoritative Baymax design docs. Today it contains `prod_req_doc.md`, `test_strat.md` and `file_system.md`, which is the planned directory layout and the place new code goes.
- The remaining design docs still need to be written into `00_docs/design/`: `api_spec.md`, `arch_overview.md`, `critical_user_flow.md`, `data_model.md`, `system_design.md`, `deliverables.md`, `gantt_sched.md` and `object_masterlist.md`.
- `00_docs/sample/` holds templates carried over from an earlier project (AthleteFlow). Use them as format references only, not as Baymax requirements.
- `trash/` holds discarded docs from AthleteFlow (`atl2/`). Ignore them unless the user points you there.

**Caveat:** `00_docs/design/test_strat.md` is still the AthleteFlow testing strategy (athletes, training plans) and hasn't been rewritten for Baymax. Its general approach does carry over: pytest for unit and integration tests against a real PostgreSQL with no mocks, Playwright for E2E, and tenant isolation tests at 100% coverage. Its domain-specific content and its schema-per-tenant assumption don't carry over, since Baymax gives each tenant its own database. The tech stack hasn't been decided for Baymax.

## Project skills (`.claude/skills/`)

Three project skills are defined: `build-object`, `start-day` and `daily-summary`. They expect this workflow:
- Generated notes go in `00_docs/stage*/`, summaries in `00_docs/summary/` and todos in `00_docs/todos/`. Filenames are prefixed with `yyyymmdd_`, using today's date, with `_[number]` added to avoid collisions.
- Code goes where `file_system.md` says.
- Every object gets a design doc approved through brainstorming before it's built, plus automated tests and a manual test script to run from a separate Windows machine on the same network.

In the skills, `~/00_docs/...` paths mean this repo's `00_docs/`, not the home directory.
