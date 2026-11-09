# Testing Strategy

## Overview

This document defines the testing approach for Baymax v1.0. Given the multi-tenant, db-separated architecture and role-based access model, test coverage must prioritize tenant isolation, authentication, and the training plan lifecycle.

---

## Test Levels

### 1. Unit Tests
Test pure business logic in isolation — no database, no HTTP.

**Targets:**
- Password hashing and token generation utilities
- Role permission checks (who can create/edit/delete exercises, plans, logs)
- Training plan date range validation (start_date < end_date)
- Activity log status transition rules
- Tenant slug generation and db name derivation

### 2. Integration Tests
Test the interaction between application code and a real database. Must use a real PostgreSQL instance with isolated tenant DBs — no mocks.

**Targets:**
- Auth: login resolves `user_identities`, sets `search_path`, returns session token
- Auth: expired sessions are rejected; `used_at` is set on consumed reset tokens
- Tenant isolation: queries in `tenant_a` db cannot read `tenant_b` data
- Role enforcement at the repository/service layer
- Training plan CRUD with correct `athlete_id` and `assigned_by` scoping
- `plan_activities` filtered by `scheduled_date` (daily activity view)
- `activity_logs` creation and status updates
- Exercise library scoped to tenant (coach/trainer create, athlete read-only)

### 3. End-to-End (E2E) Tests
Test full user flows through the UI against a running application with a seeded database.

**Targets:**

| Flow                           | Persona         |
|--------------------------------|-----------------|
| Tenant onboarding              | SaaS Admin      |
| User creation within a tenant  | Tenant Admin    |
| Login and session persistence  | All roles       |
| View athlete list, open profile| Coach, Trainer  |
| Assign exercise to athlete     | Coach, Trainer  |
| Create and activate a plan     | Coach, Trainer  |
| View today's activities        | Athlete         |
| Navigate to past/future dates  | Athlete         |
| Mark activity as completed     | Athlete         |

### 4. Manual Testing
For flows that are difficult to automate or require visual verification.

**Targets:**
- UI layout and responsiveness across supported browsers (Chrome, Firefox, Safari, Edge — latest 2 versions)
- Onboarding prompts and empty state messaging
- Dark/light mode rendering
- Activity date navigation UX

---

## Coverage Targets

| Layer          | Target | Notes                                      |
|----------------|--------|--------------------------------------------|
| Business logic | ≥ 80%  | Unit tests — auth utilities, role checks   |
| API / services | ≥ 75%  | Integration tests against real DB          |
| Critical paths | 100%   | Auth, tenant isolation, plan/log lifecycle |

---

## Critical Test Scenarios

### Tenant Isolation
- A user in `tenant_a` cannot read users, plans, or logs from `tenant_b`
- `search_path` is always set before any tenant-db query
- A `user_identities` row with a mismatched `tenant_id` cannot authenticate into the wrong tenant

### Authentication
- Valid credentials issue a session; token is stored hashed
- Expired session is rejected with 401
- Password reset token is single-use (`used_at` is set on first use)
- Invalid or expired reset token is rejected

### Role-Based Access Control
- Athlete cannot create or edit exercises or training plans
- Coach and Trainer can view all athletes on their team
- Tenant Admin cannot access another tenant's data
- Unauthenticated requests are rejected for all protected routes

### Training Plan Lifecycle
- Plan with `status = draft` is not visible to athlete until activated
- `plan_activities` for a given `scheduled_date` are returned in `sort_order`
- Coach/Trainer can add, edit, and remove activities from an active plan
- Activity dates outside plan `start_date`–`end_date` are rejected

### Activity Logs
- Athlete can log completion only for their own `plan_activities`
- `completed_at` is set on `status = completed`; null otherwise
- Dashboard queries use `athlete_id` denormalization correctly

---

## Tools

| Purpose          | Tool                              |
|------------------|-----------------------------------|
| Unit & integration tests (backend) | pytest          |
| E2E tests        | Playwright                        |
| Test database    | PostgreSQL (local or Docker)      |
| API testing      | pytest + httpx (ASGI test client) |
| Coverage reports | pytest-cov (lcov)                 |
| CI               | GitHub Actions                    |

---

## Test Data Strategy

- Each integration test suite provisions its own tenant db and tears it down after the run.
- E2E tests use a shared seeded database reset between test runs.
- No production data is used in any test environment.
- Seed data covers all four roles (tenant_admin, coach, trainer, athlete) and a complete training plan with past, present, and future `scheduled_date` entries.
