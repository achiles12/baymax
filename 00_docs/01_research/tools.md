# SaaS Tools Research for Baymax

Third-party tools to consider instead of building, adjusted for a Philippine setting.

> **Caveat:** Compiled from general knowledge, not live research. Provider availability, PH support and pricing change. Verify each provider before committing. The tech stack is not yet decided, so nothing here is a firm choice. Pick providers when each need appears, and keep each behind a thin interface so it can be swapped (as with auth).

## Contents

1. [Auth0 and the auth migration plan](#1-auth0-and-the-auth-migration-plan)
2. [Tools by category](#2-tools-by-category)
3. [Philippine rules that affect the design](#3-philippine-rules-that-affect-the-design)
4. [Product implications](#4-product-implications)
5. [Recommended starter set](#5-recommended-starter-set)
6. [Self-hosted (local) stack](#6-self-hosted-local-stack)

---

## 1. Auth0 and the auth migration plan

Auth0 (owned by Okta) is a hosted identity and authentication service. Instead of building login yourself, you hand it off. It handles:

- **Login and signup:** username/password, social logins, passwordless and magic links.
- **MFA:** plus password reset and breach detection.
- **Tokens and standards:** issues JWTs; implements OAuth 2.0 and OpenID Connect, so the backend only verifies tokens.
- **Roles and permissions:** RBAC, and "Organizations" for multi-tenant B2B apps.
- **Enterprise SSO:** SAML and similar.

The app redirects users to an Auth0-hosted login page (or embeds its widget). Auth0 authenticates them and returns a token. The API validates it and reads identity and roles from it.

**Baymax requirement:** auth is stored in the database at launch and must be able to migrate to Auth0 later. To keep that cheap:

- Keep authentication behind an interface such as `AuthProvider`. Nothing else touches password hashes.
- Use a stable external user ID (a `sub`-style claim) as the user key, not just an email or internal integer.
- Keep roles (SaaS Admin, Tenant Admin, Franchise Owner, Employee) and the tenant mapping in your own database or claims, so they map later to Auth0 roles or Organizations.
- Issue JWTs yourself from day one, so switching only changes who signs them.

**Trade-off:** Auth0 removes security-sensitive work but costs money per monthly active user and creates vendor lock-in. A database-backed start with a clean migration path suits a small first tenant.

---

## 2. Tools by category

PH notes are marked **PH:**.

### Authentication and identity

| Tool | Notes |
|---|---|
| Auth0 (Okta) | Full-featured: social login, MFA, Organizations for multi-tenancy, enterprise SSO |
| Clerk | Prebuilt login and user-management UI; popular with React/Next.js; quick setup |
| Firebase Auth / Google Identity Platform | Cheap, generous free tier; web and mobile; supports phone-number OTP |
| AWS Cognito | Cheap at scale but fiddly; fits if already on AWS |
| Supabase Auth | Bundled with their Postgres; open source |
| Keycloak | Open source and free; self-hosted |
| WorkOS | Enterprise SSO and SCIM for B2B |

**PH:** All of these work, but pricing is in USD, so check against low PH margins. Phone-number login via SMS OTP is worth supporting, since many staff use a phone number more than email. SMS costs add up.

### Payments and billing

| Tool | Notes |
|---|---|
| PayMongo                       |   PH-built gateway: cards, GCash, Maya, GrabPay, bank transfer. Likely default for subscriptions and one-off payments |
| Xendit                         |   Strong PH presence: e-wallets, bank transfer, over-the-counter, recurring payments |
| Dragonpay                      |   Bank transfer and over-the-counter (7-Eleven, Bayad Center); useful for tenants without cards |
| GCash / Maya business accounts |   Direct e-wallet acceptance; many owners will prefer it |
| Stripe                         |   Default elsewhere. **PH:** believed not open to PH-registered businesses; verify before designing around it |
| Paddle / Lemon Squeezy         |   Merchant-of-record (handle foreign tax). Suited to overseas customers, not local ones |
| Chargebee / Recurly            |   Subscription management on top of a gateway |
| Square                         |   Card payments and POS hardware; relevant to v3 POS integration |
| Invoice Ninja                  |   Open-source invoicing |

**PH:** With 5 users per tenant and flat per-tenant pricing, invoice monthly and collect via GCash or bank transfer at first. That avoids a subscription engine.

### Email, SMS and notifications

| Tool | Notes |
|---|---|
| Postmark           | Transactional email: password resets, daily summaries, approval alerts. All work from the PH |
| Resend             | Transactional email: password resets, daily summaries, approval alerts. All work from the PH |
| Amazon SES         | Transactional email: password resets, daily summaries, approval alerts. All work from the PH |
| SendGrid           | Transactional email: password resets, daily summaries, approval alerts. All work from the PH |
| Semaphore          | PH SMS providers; usually much cheaper locally than Twilio |
| Movider            | PH SMS providers; usually much cheaper locally than Twilio |
| Globe Labs         | PH SMS providers; usually much cheaper locally than Twilio |
| Twilio             | SMS, WhatsApp, voice; pricier in the PH |
| Viber Business     | SMS, WhatsApp, voice; pricier in the PH |
| Facebook Messenger         | Most owners and staff already use them; may beat email for alerts |
| Firebase Cloud Messaging   | Most owners and staff already use them; may beat email for alerts |
| OneSignal                  | Push notifications for a PWA or app (FCM is free) |
| Slack / Discord webhooks   | Internal alerts |

**PH:** Staff rarely check email, so make the owner the main recipient. Use Viber or SMS for owner alerts on adjustment requests.

### Hosting and infrastructure

| Tool | Notes |
|---|---|
| AWS / Google Cloud / Azure | Full cloud platforms. **PH:** no major region in the Philippines that I know of; nearest is Singapore (sometimes Jakarta or Hong Kong), about 30-60 ms |
| Render, Railway, Fly.io, Heroku | Simpler app hosting; work from the PH with Singapore regions |
| Vercel, Netlify | Frontend hosting |
| Neon, Supabase, AWS RDS, Crunchy Bridge | Managed PostgreSQL. Neon makes creating many databases cheap, which suits database-per-tenant |
| Cloudflare | DNS, CDN, DDoS protection, WAF. **PH:** has a Manila point of presence |
| PLDT Enterprise, Globe Business, Converge ICT | Local hosting if a client demands PH-resident data; costs more, less tooling |

### Storage and exports

- **AWS S3, Cloudflare R2, Backblaze B2:** object storage for CSV/Excel exports, receipts and backups.
- **Cloudinary, Uploadthing:** image storage, transformation and uploads.
- **XlsxWriter, openpyxl, SheetJS:** Excel generation.
- **WeasyPrint, Puppeteer:** PDF generation. Don't hand-roll either.

### Monitoring, errors and analytics

- **Sentry:** error tracking.
- **Datadog, New Relic, Grafana Cloud:** metrics, logs, APM. **Grafana Loki, BetterStack:** logs.
- **BetterStack, UptimeRobot:** uptime checks. **Instatus, Statuspage:** status pages.
- **PostHog, Mixpanel, Amplitude:** product analytics. Session replay (PostHog, Hotjar) shows where low-tech-comfort employees get stuck. PostHog also covers feature flags.

### Product and engineering libraries

- **Admin panels:** Retool, Appsmith (open source), Django Admin, Filament, React-Admin. Gives a SaaS Admin back office without building a UI.
- **Background jobs:** Celery, Arq, Sidekiq, BullMQ; hosted Inngest, Trigger.dev.
- **Feature flags:** PostHog, Unleash (open source), LaunchDarkly (pricey).
- **Search:** Postgres full-text first, then Meilisearch, Typesense, Algolia.
- **Database migrations:** Alembic, Flyway, Liquibase.
- **API docs:** FastAPI's built-in OpenAPI, Swagger UI, Redoc, Postman.
- **UI kits:** shadcn/ui, Material UI, Tailwind UI.
- **Charts and reporting:** Chart.js, Recharts, Apache ECharts; Metabase or Superset for embedded reporting.
- **Date, time and money:** `date-fns` or `luxon`; decimal/money types, never floats.

### Developer workflow

- **Code hosting and CI/CD:** GitHub (Actions), GitLab CI.
- **Containers:** Docker, Docker Compose (local); Docker Hub, AWS ECR (registries).
- **Security scanning:** Dependabot, Snyk, GitHub secret scanning, Trivy.
- **Secrets:** Doppler, Infisical, AWS Secrets Manager, HashiCorp Vault.
- **Infrastructure as code:** Terraform, Pulumi. Helps automate provisioning a database per tenant.
- **Testing:** pytest, Playwright, Testcontainers (real PostgreSQL), k6 (load).

### Customer support and success

- **Chat and helpdesk:** Crisp, Tawk.to (free), Intercom, Freshdesk, Zendesk.
- **Messenger/Viber integration:** Chatfuel, Meta Business Suite. Matters in the PH.
- **Docs and knowledge base:** Docusaurus, GitBook, Notion, Helpjuice.
- **In-app onboarding tours:** Userflow, Appcues, Shepherd.js (open source).

### Business operations

- **Accounting:** Xero, QuickBooks, Sprout, Zoho Books. BIR compliance varies; ask an accountant.
- **Legal templates:** Termly, Iubenda, or a lawyer. Check their Data Privacy Act coverage.
- **E-signature:** DocuSign, PandaDoc, Dropbox Sign.
- **CRM and forms:** HubSpot (free tier), Pipedrive, Notion or a spreadsheet; Tally, Typeform, Google Forms.
- **Marketing site:** Webflow, Framer, WordPress, or a static site on Cloudflare Pages.
- **Team tools:** Slack or Discord, Notion, Linear or GitHub Projects, Google Workspace.

### Data and integrations

- **Automations and webhooks:** Zapier, Make, n8n (self-hosted).
- **Customer-facing integrations:** Merge, Nango, Paragon. Only if v3 becomes integration-heavy.
- **Data sync and analytics:** Airbyte, dbt, if reporting outgrows the main database.

---

## 3. Philippine rules that affect the design

1. **Data Privacy Act of 2012 (RA 10173):** you will hold employee names, contact info and sales data. Needs privacy notices, consent, access controls, breach procedures, and likely NPC registration past certain thresholds. The audit trail and per-tenant databases help.
2. **BIR (tax) requirements:** Baymax records sales and could resemble a receipting system. Official receipts and invoices must come from BIR-registered, accredited systems. Under the EOPT Act, sales invoices replaced official receipts in many cases. If Baymax only records the owner's internal logbook, state that clearly in the product. If it ever issues receipts or does bookkeeping (v2), check BIR rules on Computerized Accounting Systems (CAS) and POS registration.
3. **VAT and billing:** invoicing tenants means 12% VAT or the percentage-tax regime, depending on your registration. Ask an accountant.
4. **Cash-heavy operations:** laundromats are mostly cash and GCash. The reconciliation story (sales vs. machine cycle counts) matters more than card processing.

> A PH accountant or lawyer should confirm the BIR and Data Privacy Act points before launching to paying tenants.

---

## 4. Product implications

- **Unreliable connectivity:** consider an offline-first or queue-and-sync entry screen, so no transaction is lost when Wi-Fi drops.
- **Low-end Android phones:** keep the app light; a PWA works well.
- **Pricing:** price in PHP and prefer local payment rails to avoid foreign-exchange fees.
- **Language:** consider Filipino/Taglish labels on employee-facing screens.

---

## 5. Recommended starter set

For a small team with one tenant. Skip everything else until a real need appears, since each tool adds cost, a login and a dependency.

| Need | Choice |
|---|---|
| Auth | Database-backed now; Auth0 or Clerk later |
| Hosting and database | Render, Railway or Fly.io + managed PostgreSQL (Neon) in Singapore |
| Billing | Manual GCash or bank-transfer invoicing first; then PayMongo or Xendit |
| Email | Postmark or Resend |
| SMS and alerts | Semaphore or Viber for owner alerts |
| Errors | Sentry |
| CI and analytics | GitHub Actions, PostHog (analytics and feature flags) |
| Admin console | Django Admin, Retool or Appsmith |
| UI | shadcn/ui or similar |
| Excel export | XlsxWriter or openpyxl |
| Support | Crisp or Tawk.to (works with Messenger/Viber) |
| Backups and exports | Cloudflare R2 or S3 (Singapore) |

The framework choice will decide some of these. Django, for example, brings its own admin panel, ORM and migrations.

This table assumes cloud hosting. If you host on your own hardware, see [section 6](#6-self-hosted-local-stack).

---

## 6. Self-hosted (local) stack

Goal: host on your own server (for example a mini PC) while using existing tools, so effort goes into business logic and product features. This is a recommendation, not a final decision.

### Application stack

| Layer | Pick | Why |
|---|---|---|
| Backend framework | **Django** (+ Django REST Framework or Django Ninja) | Batteries included: ORM, migrations, admin, auth, forms, permissions |
| Database | **PostgreSQL** (self-hosted) | Matches the test strategy (real Postgres, no mocks) |
| Frontend | **Django templates + HTMX + Alpine.js + Tailwind** to start | No separate SPA, responsive, light on low-end Android. Move to a React PWA only if offline entry requires it |
| SaaS Admin console | **Django Admin** | Free tenant-management UI |
| Auth | Django user model + `django-allauth`, behind the `AuthProvider` interface | Database-backed at launch. Self-hosted **Authentik** or **Keycloak** are alternatives to a paid Auth0 migration |
| Audit trail | `django-simple-history` or `django-auditlog` | "Who changed what" nearly for free. Append-only guarantees still need to be added for the anti-pilferage goal |
| Background jobs | **Procrastinate** (Postgres-backed) or **Celery + Redis** | Procrastinate avoids running Redis. Needed for rollups and exports |
| Excel/CSV export | `openpyxl` / `XlsxWriter` | Standard |
| Money | `Decimal` / `django-money` | Never floats |
| Tests | `pytest-django`, Testcontainers, Playwright | Fits `test_strat.md` |

> **Decision to make early:** the popular multi-tenant package `django-tenants` is schema-per-tenant, but Baymax uses database-per-tenant. Django supports multiple databases, but adding tenant databases at runtime needs a custom database router and connection registry. This is the main piece to build by hand. FastAPI + SQLAlchemy gives more control but fewer built-ins. **Prototype tenant database provisioning plus a per-tenant migration runner first**, since it is the riskiest part.

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

### Keep outsourced (do not self-host)

- **Email:** Resend or Postmark. Self-hosted mail is often blocked as spam.
- **SMS/alerts:** Semaphore or Viber.
- **Payments:** PayMongo or Xendit once billing starts.
- **Off-site backup storage:** Cloudflare R2 or Backblaze B2.

### Local-hosting risks to plan for

- **Power outages:** use a UPS and configure a clean shutdown when it runs low.
- **Internet outages:** employees cannot reach the server during an outage. This strengthens the case for offline queueing in the entry screen, or for hosting inside the laundromat on the same LAN.
- **Disk failure:** use an SSD mirror (RAID 1) or at minimum tested off-site backups.
- **Single point of failure:** one machine means downtime during repairs. Acceptable for tenant one, but it should shape pricing and promises for later tenants.
- **Security:** you own patching, firewall and disk encryption. This matters because the server holds other businesses' financial records and falls under the Data Privacy Act.
