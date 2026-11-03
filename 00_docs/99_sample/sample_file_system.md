# Initial Filesystem Set-up
# Updated as needed

prj_root/
├── backend/
│   ├── app/
│   │   ├── main.py                        # FastAPI app instance, middleware registration
│   │   ├── config.py                      # Settings via pydantic-settings (.env)
│   │   ├── database.py                    # SQLAlchemy engine, session factory
│   │   ├── dependencies.py                # get_current_user, set_tenant_schema, require_role
│   │   │
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── router.py              # Aggregates all v1 routers
│   │   │       ├── auth.py                # /auth/login, /logout, /reset-password/*
│   │   │       ├── admin.py               # /admin/tenants  (SaaS Admin only)
│   │   │       ├── users.py               # /users
│   │   │       ├── athletes.py            # /athletes, /athletes/{id}
│   │   │       ├── teams.py               # /teams, /teams/{id}/members
│   │   │       ├── exercises.py           # /exercises, /exercises/{id}
│   │   │       ├── plans.py               # /plans, /plans/{id}, /plans/{id}/activities
│   │   │       └── activity_logs.py       # /activities, /activity-logs, /activity-logs/{id}
│   │   │
│   │   ├── models/                        # SQLAlchemy ORM models
│   │   │   ├── public/                    # public schema — global tables
│   │   │   │   ├── tenant.py              # tenants
│   │   │   │   ├── user_identity.py       # user_identities
│   │   │   │   ├── session.py             # sessions
│   │   │   │   └── password_reset_token.py
│   │   │   └── tenant/                    # per-tenant schema tables
│   │   │       ├── user.py                # users
│   │   │       ├── athlete_profile.py
│   │   │       ├── team.py
│   │   │       ├── team_member.py
│   │   │       ├── exercise.py
│   │   │       ├── training_plan.py
│   │   │       ├── plan_activity.py
│   │   │       └── activity_log.py
│   │   │
│   │   ├── schemas/                       # Pydantic request/response models
│   │   │   ├── auth.py
│   │   │   ├── user.py
│   │   │   ├── athlete.py
│   │   │   ├── team.py
│   │   │   ├── exercise.py
│   │   │   ├── plan.py
│   │   │   └── activity_log.py
│   │   │
│   │   ├── services/                      # Business logic — no HTTP, no DB sessions
│   │   │   ├── auth_service.py            # user identity creation is an auth concern; TenantService only handles tenant-level provisioning (schema creation, migrations, initial admin user)
│   │   │   ├── tenant_service.py          # TenantService handles all provisioning (tenant creation + user creation within a tenant)
│   │   │   ├── user_service.py            # handles ongoing user creation, day-to-day user management
│   │   │   ├── team_service.py
│   │   │   ├── exercise_service.py
│   │   │   ├── plan_service.py
│   │   │   ├── plan_activity_service.py
│   │   │   └── activity_log_service.py
│   │   │
│   │   └── utils/
│   │       ├── security.py                # password hashing, token generation
│   │       └── tenant.py                  # slug generation, schema name derivation
│   │       └── schema_provisioner.py      # CREATE SCHEMA + run Alembic tenant migrations
│   │
│   ├── alembic/
│   │   ├── env.py
│   │   ├── alembic.ini
│   │   └── versions/
│   │       ├── public/                    # public schema migrations (run once)
│   │       └── tenant/                    # tenant schema migration template (run per tenant)
│   │
│   ├── tests/
│   │   ├── conftest.py                    # fixtures: test DB, tenant provisioning, seeded users
│   │   ├── unit/
│   │   │   ├── test_security.py           # password hashing, token generation
│   │   │   ├── test_tenant_utils.py       # slug generation, schema derivation
│   │   │   └── test_role_checks.py        # permission logic
│   │   └── integration/
│   │       ├── test_auth.py               # login, logout, reset, session expiry
│   │       ├── test_tenant_isolation.py   # tenant A cannot read tenant B
│   │       ├── test_users.py
│   │       ├── test_exercises.py
│   │       ├── test_plans.py
│   │       └── test_activity_logs.py
│   │
│   ├── pyproject.toml
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   │   ├── main.tsx                       # React entry point
│   │   ├── App.tsx                        # Router setup (React Router)
│   │   │
│   │   ├── api/                           # Typed API client functions
│   │   │   ├── client.ts                  # Base fetch wrapper — attaches auth headers
│   │   │   ├── auth.ts
│   │   │   ├── athletes.ts
│   │   │   ├── exercises.ts
│   │   │   ├── plans.ts
│   │   │   └── activityLogs.ts
│   │   │
│   │   ├── components/                    # Reusable UI components
│   │   │   ├── ActivityCard.tsx
│   │   │   ├── ActivityStatusBadge.tsx
│   │   │   ├── DateNavigator.tsx
│   │   │   └── EmptyState.tsx
│   │   │
│   │   ├── contexts/
│   │   │   └── AuthContext.tsx            # Session state, login/logout actions
│   │   │
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   │
│   │   ├── layouts/
│   │   │   └── AppShell.tsx               # Nav, auth guard, role-based route protection
│   │   │
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx
│   │   │   ├── PasswordResetPage.tsx
│   │   │   ├── AthletesListPage.tsx       # Coach / Trainer view
│   │   │   ├── AthleteProfilePage.tsx     # Coach / Trainer view
│   │   │   ├── PlanEditorPage.tsx         # Coach / Trainer view
│   │   │   ├── ExerciseLibraryPage.tsx    # Coach / Trainer view
│   │   │   └── AthleteDashboardPage.tsx   # Athlete view
│   │   │
│   │   └── types/
│   │       └── index.ts                   # Shared TypeScript interfaces (User, Plan, etc.)
│   │
│   ├── e2e/                               # Playwright E2E tests
│   │   ├── auth.spec.ts
│   │   ├── tenant-onboarding.spec.ts
│   │   ├── user-creation.spec.ts
│   │   ├── coach-flow.spec.ts
│   │   ├── athlete-flow.spec.ts
│   │   ├── tenant-isolation.spec.ts
│   │   └── fixtures/
│   │       └── seed.py                    # Seeds 2 tenants, all 4 roles, 1 active plan
│   │
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   ├── tsconfig.json
│   └── package.json
│
├── .github/
│   └── workflows/
│       └── ci.yml                         # lint, typecheck, pytest, playwright on PR
│
├── docker-compose.yml                     # Postgres + API + frontend for local dev
├── .env.example
└── README.md