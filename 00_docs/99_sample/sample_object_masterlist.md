# Object Masterlist — AthleteFlow v1.0

Last updated: 2026-08-19 (Stage 6 pass — all 14 files built; plan view/edit gap found and closed same day, incl. 2 new backend routes)

---

## Stage 1 — Project Setup & Infrastructure

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| Docker Compose stack | Infra | `docker-compose.yml` | ✅ Complete | 2026-05-14 |
| FastAPI skeleton + `GET /health` | Route | `backend/app/main.py` | ✅ Complete | 2026-05-14 |
| Alembic runner | Config | `backend/alembic/env.py`, `alembic.ini` | ✅ Complete | 2026-05-14 |
| Vite + React + Tailwind scaffold | Frontend | `frontend/` | ✅ Complete | 2026-05-14 |
| GitHub Actions CI | CI | `.github/workflows/ci.yml` | ✅ Complete | 2026-05-14 |

---

## Stage 2 — Authentication & Multi-Tenant Core

### Utilities

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `hash_password`, `verify_password`, `generate_token`, `hash_token` | Util | `backend/app/utils/security.py` | ✅ Complete (rewritten Jun 24: passlib removed, direct bcrypt 4.x) | 2026-05-15 |
| `derive_slug`, `slug_to_schema_name` | Util | `backend/app/utils/tenant.py` | ✅ Complete | 2026-05-15 |
| `provision(schema_name)` | Util | `backend/app/utils/schema_provisioner.py` | ✅ Complete | 2026-06-19 |

### Models (Public Schema)

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `Tenant` | Model | `backend/app/models/public/tenant.py` | ✅ Complete | 2026-05-15 |
| `UserIdentity` | Model | `backend/app/models/public/user_identity.py` | ✅ Complete | 2026-05-15 |
| `Session` | Model | `backend/app/models/public/session.py` | ✅ Complete | 2026-05-15 |
| `PasswordResetToken` | Model | `backend/app/models/public/password_reset_token.py` | ✅ Complete | 2026-05-15 |

### Models (Tenant Schema)

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `User` | Model | `backend/app/models/tenant/user.py` | ✅ Complete | 2026-05-15 |
| `AthleteProfile` | Model | `backend/app/models/tenant/athlete_profile.py` | ✅ Complete | 2026-05-15 |
| `Team` | Model | `backend/app/models/tenant/team.py` | ✅ Complete | 2026-05-15 |
| `Exercise` | Model | `backend/app/models/tenant/exercise.py` | ✅ Complete | 2026-05-15 |
| `TrainingPlan` | Model | `backend/app/models/tenant/training_plan.py` | ✅ Complete | 2026-05-15 |
| `PlanActivity` | Model | `backend/app/models/tenant/plan_activity.py` | ✅ Complete | 2026-05-15 |
| `ActivityLog` | Model | `backend/app/models/tenant/activity_log.py` | ✅ Complete | 2026-05-15 |

### Migrations

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| Public schema migration | Migration | `backend/alembic/versions/public/0001_create_public_tables.py` | ✅ Complete | 2026-05-22 |
| Tenant schema migration | Migration | `backend/alembic/versions/tenant/0001_create_tenant_tables.py` | ✅ Complete | 2026-05-22 |

### Services

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `AuthService` (login, logout, reset-password request + confirm, `provision_identity`) | Service | `backend/app/services/auth_service.py` | ✅ Complete (updated Aug 17: added `provision_identity` for Stage 3 user creation) | 2026-06-19 |
| `TenantService` (provision_tenant, list, get_by_slug, update_status) | Service | `backend/app/services/tenant_service.py` | ✅ Complete (updated Jun 24: SET LOCAL) | 2026-06-19 |

### Dependencies / Middleware

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `CurrentUser` dataclass | Dependency | `backend/app/dependencies.py` | ✅ Complete | 2026-06-19 |
| `get_current_user` | Dependency | `backend/app/dependencies.py` | ✅ Complete (updated Jun 24: SET LOCAL) | 2026-06-19 |
| `set_tenant_schema` | Dependency | `backend/app/dependencies.py` | ✅ Complete (updated Jun 24: SET LOCAL) | 2026-06-19 |
| `require_role(...)` | Dependency | `backend/app/dependencies.py` | ✅ Complete | 2026-06-19 |

### Schemas

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `LoginRequest`, `LoginResponse`, `LogoutResponse` | Schema | `backend/app/schemas/auth.py` | ✅ Complete | 2026-06-20 |
| `ResetPasswordRequestBody`, `ResetPasswordRequestResponse` | Schema | `backend/app/schemas/auth.py` | ✅ Complete | 2026-06-20 |
| `ResetPasswordConfirmBody`, `ResetPasswordConfirmResponse` | Schema | `backend/app/schemas/auth.py` | ✅ Complete | 2026-06-20 |
| `ProvisionTenantRequest`, `TenantOut`, `ProvisionTenantResponse` | Schema | `backend/app/schemas/tenant.py` | ✅ Complete | 2026-06-24 |

### Routes

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `POST /api/v1/auth/login` | Route | `backend/app/api/v1/auth.py` | ✅ Complete | 2026-06-20 |
| `POST /api/v1/auth/logout` | Route | `backend/app/api/v1/auth.py` | ✅ Complete | 2026-06-20 |
| `POST /api/v1/auth/reset-password/request` | Route | `backend/app/api/v1/auth.py` | ✅ Complete | 2026-06-20 |
| `POST /api/v1/auth/reset-password/confirm` | Route | `backend/app/api/v1/auth.py` | ✅ Complete | 2026-06-20 |
| `v1_router` aggregator | Router | `backend/app/api/v1/router.py` | ✅ Complete (updated Jun 24: admin router included) | 2026-06-20 |
| `main.py` router registration + HTTPException handler | Infra | `backend/app/main.py` | ✅ Complete | 2026-06-20 |
| `POST /api/v1/admin/tenants` | Route | `backend/app/api/v1/admin.py` | ✅ Complete | 2026-06-24 |

### Dev Utilities

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `dev_get_reset_token.py` | Script | `backend/scripts/dev_get_reset_token.py` | ✅ Complete | 2026-06-20 |

### Tests

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `test_security` | Unit | `backend/tests/unit/test_security.py` | ✅ Complete | 2026-05-15 |
| `test_tenant_utils` | Unit | `backend/tests/unit/test_tenant_utils.py` | ✅ Complete | 2026-05-15 |
| `test_public_models` | Unit | `backend/tests/unit/test_public_models.py` | ✅ Complete | 2026-05-15 |
| `test_tenant_models` | Unit | `backend/tests/unit/test_tenant_models.py` | ✅ Complete | 2026-05-15 |
| `test_auth_service` | Unit | `backend/tests/unit/test_auth_service.py` | ✅ Complete | 2026-06-19 |
| `test_tenant_service` | Unit | `backend/tests/unit/test_tenant_service.py` | ✅ Complete | 2026-06-19 |
| `test_role_checks` | Unit | `backend/tests/unit/test_role_checks.py` | ✅ Complete | 2026-06-19 |
| `integration/conftest.py` (module fixtures) | Test Fixture | `backend/tests/integration/conftest.py` | ✅ Complete (updated Aug 17: `test_tenant` teardown scoped to tenant_id, not just seeded admin's identity) | 2026-06-19 |
| `test_auth` (integration) | Integration | `backend/tests/integration/test_auth.py` | ✅ Complete | 2026-06-19 |
| `test_tenant_isolation` (integration) | Integration | `backend/tests/integration/test_tenant_isolation.py` | ✅ Complete (updated Jun 24: FK-safe _drop_tenant helper) | 2026-06-19 |
| `test_auth_api` (HTTP integration) | Integration | `backend/tests/integration/test_auth_api.py` | ✅ Complete | 2026-06-20 |
| `test_admin_api` (HTTP integration) | Integration | `backend/tests/integration/test_admin_api.py` | ✅ Complete | 2026-06-24 |

---

## Stage 3 — User & Team Management

### Migrations

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| Tenant migration — `users`, `athlete_profiles`, `teams`, `team_members` | Migration | `backend/alembic/versions/tenant/0001_create_tenant_tables.py` | ✅ Complete (trimmed from bundled 8-table migration; added missing `UniqueConstraint(team_id, user_id)` on `team_members`) | 2026-07-30 |

### Services

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `UserService` (`create_user`, `list_users`, `get_user`) | Service | `backend/app/services/user_service.py` | ✅ Complete | 2026-08-17 |
| `TeamService` (`create_team`, `add_member`, `list_athletes_for_user`, `get_team_athlete`) | Service | `backend/app/services/team_service.py` | ✅ Complete (updated Aug 17: athlete-scoping functions for Step 3) | 2026-08-17 |

### Schemas

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `CreateUserRequest`, `UserOut`, `CreateUserResponse` | Schema | `backend/app/schemas/user.py` | ✅ Complete | 2026-08-17 |
| `CreateTeamRequest`, `TeamOut`, `AddTeamMemberRequest`, `TeamMemberOut` | Schema | `backend/app/schemas/team.py` | ✅ Complete | 2026-08-17 |
| `AthleteOut`, `PlanSummaryOut`, `AthleteDetailOut` | Schema | `backend/app/schemas/athlete.py` | ✅ Complete | 2026-08-17 |

### Routes

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `POST /api/v1/users`, `GET /api/v1/users` | Route | `backend/app/api/v1/users.py` | ✅ Complete | 2026-08-17 |
| `POST /api/v1/teams`, `POST /api/v1/teams/{id}/members` | Route | `backend/app/api/v1/teams.py` | ✅ Complete | 2026-08-17 |
| `GET /api/v1/athletes`, `GET /api/v1/athletes/{id}` | Route | `backend/app/api/v1/athletes.py` | ✅ Complete — closes Stage 3 exit criteria | 2026-08-17 |
| `v1_router` — users/teams/athletes wired in | Router | `backend/app/api/v1/router.py` | ✅ Complete | 2026-08-17 |

### Dev Utilities

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `dev_manual_test_user_team.py` | Script | `backend/scripts/dev_manual_test_user_team.py` | ✅ Complete | 2026-08-17 |

### Infra fix

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `expire_on_commit=False` on `SessionLocal` | Fix | `backend/app/database.py` | ✅ Complete — first tenant-schema-writing routes exposed a post-commit `SET LOCAL search_path` reversion bug; see `20260817_stage3_routes_test_results.md` | 2026-08-17 |

### Tests

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `test_tenant_schema_migration` | Integration | `backend/tests/integration/test_tenant_schema_migration.py` | ✅ Complete — 7/7 passed | 2026-07-30 |
| `test_user_service` (unit) | Unit | `backend/tests/unit/test_user_service.py` | ✅ Complete — 8/8 passed | 2026-08-17 |
| `test_team_service` (unit) | Unit | `backend/tests/unit/test_team_service.py` | ✅ Complete — 5/5 passed | 2026-08-17 |
| `test_user_service` (integration) | Integration | `backend/tests/integration/test_user_service.py` | ✅ Complete — 6/6 passed | 2026-08-17 |
| `test_team_service` (integration) | Integration | `backend/tests/integration/test_team_service.py` | ✅ Complete — 3/3 passed | 2026-08-17 |
| `test_auth_service` `provision_identity` additions (unit) | Unit | `backend/tests/unit/test_auth_service.py` | ✅ Complete — 2 new, 22/22 passed | 2026-08-17 |
| `test_users_api` (HTTP integration) | Integration | `backend/tests/integration/test_users_api.py` | ✅ Complete — 10/10 passed | 2026-08-17 |
| `test_teams_api` (HTTP integration) | Integration | `backend/tests/integration/test_teams_api.py` | ✅ Complete — 9/9 passed | 2026-08-17 |
| `test_team_service` athlete-scoping additions (unit) | Unit | `backend/tests/unit/test_team_service.py` | ✅ Complete — 5 new, 10/10 passed | 2026-08-17 |
| `test_athletes_api` (HTTP integration) | Integration | `backend/tests/integration/test_athletes_api.py` | ✅ Complete — 10/10 passed | 2026-08-17 |

**Stage 3 status: ✅ Complete** — all deliverables, endpoints, and exit
criteria from `deliverables.md` §Stage 3 built and tested. Next: Stage 4.

---

## Stage 4 — Exercise Library & Training Plans

### Migrations

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| Tenant migration — `exercises`, `training_plans`, `plan_activities` | Migration | `backend/alembic/versions/tenant/0002_create_exercise_plan_tables.py` | ✅ Complete (split from bundled migration; added missing `CheckConstraint("end_date >= start_date")` on `training_plans`; also contains Stage 5's `activity_logs` table, see below) | 2026-07-30 |

### Services

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `ExerciseService` (`create_exercise`, `list_exercises`, `get_exercise`, `update_exercise`, `delete_exercise`) | Service | `backend/app/services/exercise_service.py` | ✅ Complete — delete raises `ExerciseError("in_use")` on FK violation (`plan_activities` reference) | 2026-08-18 |
| `TrainingPlanService` (`create_plan`, `get_plan`, `update_plan`, `update_status`) | Service | `backend/app/services/plan_service.py` | ✅ Complete — `update_status` rejects any transition once `status == completed` (terminal) | 2026-08-18 |
| `PlanActivityService` (`add_activity`, `get_activity`, `update_activity`, `delete_activity`) | Service | `backend/app/services/plan_activity_service.py` | ✅ Complete | 2026-08-18 |

### Schemas

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `CreateExerciseRequest`, `UpdateExerciseRequest`, `ExerciseOut` | Schema | `backend/app/schemas/exercise.py` | ✅ Complete | 2026-08-18 |
| `CreatePlanRequest`, `UpdatePlanRequest`, `UpdatePlanStatusRequest`, `PlanOut`, `CreateActivityRequest`, `UpdateActivityRequest`, `PlanActivityOut` | Schema | `backend/app/schemas/plan.py` | ✅ Complete | 2026-08-18 |

### Routes

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `GET/POST /api/v1/exercises`, `PUT/DELETE /api/v1/exercises/{id}` | Route | `backend/app/api/v1/exercises.py` | ✅ Complete — `GET` open to all 4 tenant roles; write ops `coach`/`trainer` only | 2026-08-18 |
| `POST/PUT /api/v1/plans`, `PATCH /api/v1/plans/{id}/status`, `POST/PUT/DELETE /api/v1/plans/{id}/activities[/{aid}]` | Route | `backend/app/api/v1/plans.py` | ✅ Complete — closes Stage 4 exit criteria | 2026-08-18 |
| `GET /api/v1/plans/{id}`, `GET /api/v1/plans/{id}/activities` | Route | `backend/app/api/v1/plans.py` | ✅ Complete — **added post-hoc**, not in the original Stage 4 endpoint table (see note below) | 2026-08-19 |
| `v1_router` — Stage 4 block uncommented | Router | `backend/app/api/v1/router.py` | ✅ Complete | 2026-08-18 |

### Services (addendum)

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `list_activities(db, plan_id)` | Service | `backend/app/services/plan_activity_service.py` | ✅ Complete — orders by `sort_order` | 2026-08-19 |

### Tests

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `test_exercise_plan_migration` | Integration | `backend/tests/integration/test_exercise_plan_migration.py` | ✅ Complete — 7/7 passed | 2026-07-30 |
| `test_exercise_service`, `test_plan_service`, `test_plan_activity_service` (unit) | Unit | `backend/tests/unit/test_{exercise,plan,plan_activity}_service.py` | ✅ Complete — 9/9, 8/8, 8/8 passed (`test_plan_activity_service` +1 for `list_activities`, Aug 19) | 2026-08-18 |
| `test_exercises_api`, `test_plans_api` (HTTP integration) | Integration | `backend/tests/integration/test_{exercises,plans}_api.py` | ✅ Complete — 12/12, 21/21 passed (`test_plans_api` +5 for the new `GET` routes, Aug 19), incl. Stage 4 exit-criterion test (5 activities / 3 dates / activate) | 2026-08-18 |

**Stage 4 status: ✅ Complete** — all deliverables, endpoints, and exit
criteria from `deliverables.md` §Stage 4 built and tested (318/318 full
suite passing as of Aug 19). `GET /plans`/`GET /plans/{id}` were
**originally** intentionally out of scope — not in the Stage 4 endpoint
table; plan data was meant to be reached via `GET /api/v1/athletes/{id}`
(Stage 3) only. That held until Stage 6 needed to reopen an already-
created plan for viewing/editing, which has no other data path (the
athlete-detail plan summary has no activities). Added 2026-08-19 — see
`stage06/20260819_2_plan_view_edit_design.md`. Next: Stage 5.

---

## Stage 5 — Athlete Activity View & Logging

### Migrations

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| Tenant migration — `activity_logs` | Migration | `backend/alembic/versions/tenant/0002_create_exercise_plan_tables.py` (bundled with Stage 4, see above) | ✅ Complete — done ~2 weeks ahead of its Sprint 3 schedule slot | 2026-07-30 |

### Services

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `ActivityLogService` (`list_activities_for_date`, `get_plan_activity_for_athlete`, `create_log`, `get_log`, `update_log`) | Service | `backend/app/services/activity_log_service.py` | ✅ Complete — `list_activities_for_date` joins `plan_activities`→`training_plans` filtered to `status=active`, which is what makes draft plans invisible to the athlete; `create_log` rejects duplicates (`ActivityLogError("already_logged")`, 409) | 2026-08-18 |

### Schemas

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `ExerciseSummaryOut`, `ActivityLogSummaryOut`, `ActivityViewOut`, `CreateActivityLogRequest`, `UpdateActivityLogRequest`, `ActivityLogOut` | Schema | `backend/app/schemas/activity_log.py` | ✅ Complete | 2026-08-18 |

### Routes

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `GET /api/v1/activities?date=`, `POST /api/v1/activity-logs`, `PATCH /api/v1/activity-logs/{id}` | Route | `backend/app/api/v1/activity_logs.py` | ✅ Complete — all three `require_role("athlete")` only; closes Stage 5 exit criteria | 2026-08-18 |
| `v1_router` — Stage 5 block uncommented | Router | `backend/app/api/v1/router.py` | ✅ Complete | 2026-08-18 |

### Tests

Migration coverage: `test_exercise_plan_migration.py` (see Stage 4) — `activity_logs` enum
rejection and FK-to-`plan_activities` violation cases included.

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `test_activity_log_service` (unit) | Unit | `backend/tests/unit/test_activity_log_service.py` | ✅ Complete — 12/12 passed | 2026-08-18 |
| `test_activity_logs_api` (HTTP integration) | Integration | `backend/tests/integration/test_activity_logs_api.py` | ✅ Complete — 15/15 passed, incl. Stage 5 exit-criterion test (view/log/update across 3 dates) and the deferred Stage 4 "draft plan invisible to athlete" case | 2026-08-18 |

**Stage 5 status: ✅ Complete** — all deliverables, endpoints, and exit
criteria from `deliverables.md` §Stage 5 built and tested (312/312 full
suite passing). This also closes the Stage 4 exit-criteria item that
depended on this route ("draft plan is invisible to athlete" — verified
by `test_list_activities_draft_plan_invisible`). Next: Stage 6 (frontend).

---

## Stage 6 — Frontend: Auth & Coach/Trainer Views

### Types

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| Shared interfaces (`User`, `Athlete`, `Plan`, etc.) | Types | `frontend/src/types/index.ts` | ✅ Complete | 2026-08-18 |

### API client

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `apiFetch` / `ApiError` typed fetch wrapper | API | `frontend/src/api/client.ts` | ✅ Complete — attaches `Authorization: Bearer`, parses `{"error":{"code","message"}}` shape, handles 401 | 2026-08-18 |
| `login()`, `logout()`, `requestReset()`, `confirmReset()` | API | `frontend/src/api/auth.ts` | ✅ Complete | 2026-08-18 |
| `listAthletes()`, `getAthlete(id)` | API | `frontend/src/api/athletes.ts` | ✅ Complete | 2026-08-18 |
| `listExercises()`, `createExercise()`, `updateExercise()`, `deleteExercise()` | API | `frontend/src/api/exercises.ts` | ✅ Complete | 2026-08-19 |
| `createPlan()`, `patchStatus()`, `addActivity()`, `updateActivity()`, `deleteActivity()` | API | `frontend/src/api/plans.ts` | ✅ Complete — `updatePlan()` route exists server-side but intentionally not called from any UI this pass (no post-create rename/redate in the editor) | 2026-08-19 |

### Contexts / Hooks / Layout

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `AuthContext` (session state, `login`/`logout`, `localStorage` persistence) | Context | `frontend/src/contexts/AuthContext.tsx` | ✅ Complete | 2026-08-18 |
| `useAuth` (`user`, `isAuthenticated`, `role`) | Hook | `frontend/src/hooks/useAuth.ts` | ✅ Complete | 2026-08-18 |
| `AppShell` (top nav, auth guard, role-based route protection) | Layout | `frontend/src/layouts/AppShell.tsx` | ✅ Complete | 2026-08-18 |
| Routing wired into `App.tsx` | Infra | `frontend/src/App.tsx` | ✅ Complete | 2026-08-18 |

### Pages

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `LoginPage` | Page | `frontend/src/pages/LoginPage.tsx` | ✅ Complete | 2026-08-18 |
| `PasswordResetPage` (request + confirm, two-step) | Page | `frontend/src/pages/PasswordResetPage.tsx` | ✅ Complete | 2026-08-18 |
| `AthletesListPage` (roster filtered to coach's team) | Page | `frontend/src/pages/AthletesListPage.tsx` | ✅ Complete | 2026-08-18 |
| `AthleteProfilePage` (detail + assigned plans) | Page | `frontend/src/pages/AthleteProfilePage.tsx` | ✅ Complete | 2026-08-18 |
| `ExerciseLibraryPage` (list + create/edit/delete, role-gated) | Page | `frontend/src/pages/ExerciseLibraryPage.tsx` | ✅ Complete | 2026-08-19 |
| `PlanEditorPage` (create **or** view/edit an existing plan, add/reorder/activate activities) | Page | `frontend/src/pages/PlanEditorPage.tsx` | ✅ Complete — routes `athletes/:id/plans/new` and `athletes/:id/plans/:planId` share one component; view/edit added 2026-08-19 after user-reported gap (see `stage06/20260819_2_plan_view_edit_design.md`); ↑/↓ buttons for reorder, no drag-and-drop library added | 2026-08-19 |

### Verification

| Check | Status | Date |
|-------|--------|------|
| `npx tsc --noEmit` | ✅ Pass — 0 errors | 2026-08-18 |
| `npm run build` (production) | ✅ Pass — 44 modules, no warnings | 2026-08-18 |
| `npm run lint` | 🚫 Blocked — pre-existing gap, not caused by this build. ESLint 9 needs `eslint.config.js` (flat config); repo only has the legacy script, no config file, no `@typescript-eslint`/`eslint-plugin-react-hooks` installed. Stage 1 scaffolding debt, flagged for a future stage. | 2026-08-18 |
| Backend contract verification (live API, real provisioned tenant, no mocking) | ✅ Pass — `login`, `GET /athletes`, `GET /athletes/{id}`, reset-password request, 401/404 error shapes all match byte-for-byte | 2026-08-18 |
| Browser verification (rendering, click-through, responsive 1280px/375px, console cleanliness) | ✅ Pass — all 9 steps of `stage06/20260818_manual_test_frontend_auth_coach.md` executed on a physical Windows machine over the LAN, full pass | 2026-08-19 |
| `api/exercises.ts` / `api/plans.ts` backend-contract verification (live API, no mocking) | ✅ Pass — full flow (3 exercises, plan, 3 activities, reorder, remove, activate, 409 in-use delete) exercised against the real running stack, response shapes match exactly | 2026-08-19 |
| Plan view/edit backend-contract verification (`GET /plans/{id}`, `GET /plans/{id}/activities`, `PUT /plans/{id}`) | ✅ Pass — get, list (empty), 404 on both new routes, rename/redate, re-get confirms persistence | 2026-08-19 |
| `ExerciseLibraryPage` / `PlanEditorPage` browser verification | 🟡 Not yet executed — manual test script written (`stage06/20260819_manual_test_plan_exercise_editor.md`), stack rebuilt/redeployed with the new pages live at `192.168.50.113:20305`, pass not yet run | 2026-08-19 |

Manual test scripts: `stage06/20260818_manual_test_frontend_auth_coach.md` (PASS, Aug 19),
`stage06/20260819_manual_test_plan_exercise_editor.md` (not yet executed).
Design docs: `stage06/20260818_frontend_auth_coach_design.md`,
`stage06/20260819_plan_exercise_editor_design.md`.
Test-results docs: `stage06/20260818_frontend_auth_coach_test_results.md`,
`stage06/20260819_plan_exercise_editor_test_results.md`.

### Infra Fixes (Aug 19 — unblocking Stage 6 manual verification)

| Object | Type | File | Status | Date |
|--------|------|------|--------|------|
| `scripts/deploy.sh` (rebuild + redeploy + migrate + verify, one command) | Script | `scripts/deploy.sh` | ✅ Complete | 2026-08-19 |
| Alembic public/tenant revision-id collision fix (scope `Config.version_locations` per chain, matching `schema_provisioner.py`'s existing pattern) | Fix | `scripts/deploy.sh` | ✅ Complete — public schema stamped at `0001` (was un-stamped, bootstrapped outside Alembic) | 2026-08-19 |
| `VITE_API_URL` / `CORS_ORIGINS` LAN reachability fix (Dockerized frontend was baked with `localhost:20310`, unreachable from any other machine) | Fix | `.env` | ✅ Complete — verified via live CORS preflight (200) | 2026-08-19 |
| `stage06/20260819_docker_rebuild_redeploy.md` (rebuild/redeploy runbook) | Doc | `stage06/20260819_docker_rebuild_redeploy.md` | ✅ Complete | 2026-08-19 |

**Stage 6 status: 🟡 In progress — all 14 deliverable files built, one manual pass
remaining.** Auth + coach roster foundation (10 files) is fully complete and
browser-verified (Aug 19). `api/exercises.ts`, `api/plans.ts`,
`ExerciseLibraryPage`, `PlanEditorPage` (the remaining 4 files) are built,
type-checked, and backend-contract-verified against the live API (Aug 19) —
the full create-plan/add-3-activities/reorder/activate flow was exercised
end-to-end through the real HTTP API. User testing the same day surfaced a
real gap — no way to view or edit an already-created plan — traced to the
backend having no `GET /plans/{id}` route at all; closed same day by adding
that route plus `GET /plans/{id}/activities` and extending `PlanEditorPage`
to handle both creating and reopening a plan (see
`stage06/20260819_2_plan_view_edit_design.md`). What's left before Stage 6's
exit criterion is fully met: the browser pass
(`stage06/20260819_manual_test_plan_exercise_editor.md`, extended with
view/edit steps, not yet executed).

---

## Stage 7 — Frontend: Athlete Views

| Object | Status |
|--------|--------|
| All objects | ⬜ Not started |

---

## Stage 8 — E2E Automated Test Suite

| Object | Status |
|--------|--------|
| All objects | ⬜ Not started |

---

## Stage 9 — Manual QA Pass & Launch Prep

| Object | Status |
|--------|--------|
| All objects | ⬜ Not started |
