● Auth0 is a hosted identity and authentication service (now owned by Okta).
  Instead of building login yourself, you hand it off to Auth0, which handles:

  - Login and signup: username/password, social logins (Google, etc.),
    passwordless and magic links.
  - MFA: multi-factor authentication, plus password reset and breach detection.
  - Tokens and standards: it issues JWTs and implements OAuth 2.0 and OpenID
    Connect, so your backend just verifies tokens.
  - Roles and permissions: role-based access control (RBAC), and "Organizations"
    for multi-tenant B2B apps.
  - Enterprise SSO: SAML and similar, if a customer ever wants it.

  Your app redirects users to an Auth0-hosted login page, or embeds its widget.
  Auth0 authenticates them and returns a token. Your API validates that token
  and reads the user's identity and roles from it.

  For Baymax: your requirements say auth is stored in the database at launch and
  must be designed so it can migrate to Auth0 later. In practice that means:
  - Keep authentication behind an interface, such as AuthProvider. The rest of
    the code shouldn't touch password hashes directly.
  - Use a stable external user ID (a sub-style claim) as the user key, not just
    an email or an internal integer.
  - Keep roles (SaaS Admin, Tenant Admin, Franchise Owner, Employee) and the
    tenant mapping in your own database or claims, so they can later map to
    Auth0 roles or Organizations.
  - Issue JWTs yourself from day one, so switching to Auth0 tokens only changes
    who signs them.

  The trade-off is that Auth0 saves you from security-sensitive work, but it
  costs money per monthly active user and ties you to a vendor. That's why a
  database-backed start with a clean migration path is a reasonable choice for a
  small first tenant.


● Here are common SaaS building blocks by category, with notes on where each
  fits Baymax.

  # Authentication and identity

  - Auth0 (Okta): the full-featured option, with social login, MFA,
    Organizations for multi-tenancy and enterprise SSO.
  - Clerk: prebuilt login and user-management UI components. It's popular with
    React and Next.js and quick to set up.
  - Firebase Auth / Google Identity Platform: cheap and simple, with a generous
    free tier. Good for mobile and web.
  - AWS Cognito: cheap at scale, but fiddly to configure. Fits well if you're
    already on AWS.
  - Supabase Auth: bundled with their Postgres. Open source.
  - Keycloak (self-hosted): open source and free, but you run it yourself.
  - WorkOS: focused on enterprise SSO and SCIM for B2B.

  # Payments and billing

  - Stripe: the default for card payments, subscriptions, invoices and Stripe
    Billing. It supports per-seat or per-tenant pricing.
  - Paddle / Lemon Squeezy: merchant-of-record services. They handle sales tax
    and VAT for you.
  - Chargebee / Recurly: subscription management layered on top of a payment
    gateway.
  - Square: card payments and POS hardware. It's relevant for the v3 POS
    integration.

  # Email, SMS and notifications

  - SendGrid / Postmark / Amazon SES / Resend: transactional email such as
    password resets, daily summaries and approval alerts.
  - Twilio: SMS, WhatsApp and voice. Useful for alerts to the owner when an
    adjustment request is filed.
  - OneSignal / Firebase Cloud Messaging: push notifications.

  # Hosting and infrastructure

  - AWS / Google Cloud / Azure: full cloud platforms.
  - Render / Railway / Fly.io / Heroku: simpler app hosting.
  - Vercel / Netlify: frontend hosting.
  - Neon / Supabase / AWS RDS / Crunchy Bridge: managed PostgreSQL. This is
    relevant to your database-per-tenant model, and Neon in particular makes
    creating many databases cheap.
  - Cloudflare: DNS, CDN, DDoS protection and WAF.

  # Monitoring and error tracking

  - Sentry: error and exception tracking.
  - Datadog / New Relic / Grafana Cloud: metrics, logs and APM.
  - BetterStack / UptimeRobot: uptime checks and status pages.
  - PostHog / Mixpanel / Amplitude: product analytics, to see how staff actually
    use the entry screens.

  # File storage and exports

  - AWS S3 / Cloudflare R2 / Backblaze B2: object storage for CSV/Excel exports,
    receipts and backups.
  - Cloudinary: image storage and transformation.

  Support and communication

  - Intercom / Crisp / Zendesk / Freshdesk: in-app chat and ticketing.
  - Slack / Discord webhooks: internal alerts.

  # Dev workflow

  - GitHub / GitLab: code hosting and CI (GitHub Actions).
  - Docker Hub / AWS ECR: container registries.
  - Doppler / AWS Secrets Manager / HashiCorp Vault: secrets management.
  - Terraform / Pulumi: infrastructure as code. This helps automate provisioning
    a new database per tenant.

  # A practical starter set for Baymax

  This is a lean stack that suits a small first tenant:
  `Auth: database-backed now, with Auth0 or Clerk later.`
  `Hosting and database: Render, Railway or Fly.io, with managed PostgreSQL`
    (Neon is a good fit for database-per-tenant).
  `Billing: Stripe, once you have paying tenants.`
  `Email: Postmark or Resend`
  `Errors: Sentry`
  `Backups and exports: S3 or R2`

  The tech stack hasn't been decided yet, so none of this is a recommendation to
  adopt now. Pick providers when each need appears, and keep each one behind a
  thin interface so you can swap it later, as with the auth plan.

● Several tools from my last list don't fit the Philippines well, so I've
  revised it. This is from my own knowledge, not live research, and provider
  availability and fees change. Check each provider's current PH support before
  committing.

  What changes in the Philippines

  # Payments and billing
  ## Stripe
   - I don't believe it's open to PH-registered businesses, so it's
    probably not your billing option. Verify this before designing around it.
  ## PayMongo:
   - a PH-built gateway for cards, GCash, Maya, GrabPay and bank
    transfer. It's the likely default for subscriptions and one-off payments.
  ## Xendit:
   - a gateway with a strong PH presence. It supports e-wallets, bank
    transfer, over-the-counter payment and recurring payments.
  ## Dragonpay:
   - bank transfer and over-the-counter options (7-Eleven, Bayad
    Center). Useful for tenants without cards.
  ## GCash / Maya business accounts:
   - direct e-wallet acceptance. Many owners will
    prefer paying this way.
  ## Paddle / Lemon Squeezy:
   - merchant-of-record services that handle foreign tax.
    They suit selling to overseas customers, not local ones.
  ## Billing logic:
   - with only 5 users per tenant and flat per-tenant pricing, you
    could invoice monthly and collect via GCash or bank transfer. That avoids a
    subscription engine at first.

  # SMS and notifications
  ## Semaphore, Movider, Globe Labs:
   - PH SMS providers, usually much cheaper
    locally than Twilio.
  ## Viber Business / Facebook Messenger:
   - most owners and staff already use
    these, so they may work better than email for alerts.
  ## Firebase Cloud Messaging:
   - free push notifications for a phone PWA or app.
  ## Email:
   - Postmark, Resend and Amazon SES all work from the PH. Staff rarely
    check email, so make the owner the main recipient.

  # Hosting and data
  - No major hyperscaler region in the Philippines that I know of. The nearest
    are Singapore (AWS, GCP, Azure) and sometimes Jakarta or Hong Kong. Latency
    is fine, around 30–60 ms.
  - Cloudflare: has a Manila point of presence for CDN and DNS.
  - Local hosting (PLDT Enterprise, Globe Business, Converge ICT): an option if
    a client demands PH-resident data, but costs more and offers less tooling.
  - Neon, Supabase, Render, Railway, Fly.io: these work fine from the PH, with
    Singapore regions.

  # Authentication
  ## Auth0, Clerk, Firebase Auth and Cognito:
    all work, but pricing is in USD.
    Check their fees against low PH margins.
  ## Phone-number login via SMS OTP:
    worth supporting, since many PH staff use a
    phone number more than an email address. Firebase Auth supports this, but
    SMS costs add up.

  Philippine rules that affect the design

  1. Data Privacy Act of 2012 (RA 10173): you'll hold employee names, contact
     info and sales data. You need privacy notices, consent, access controls,
     breach procedures and likely NPC registration if you cross certain
     thresholds. Your audit trail and per-tenant databases help here.
  2. BIR (tax) requirements: Baymax records sales and could look like a
     receipting system. Official receipts and invoices have to come from
     BIR-registered, accredited systems. Under the EOPT Act, sales invoices
     replaced official receipts for many cases. If you only record the owner's
     internal logbook, say so clearly in the product. If Baymax ever issues
     receipts or does bookkeeping (v2), check BIR rules on Computerized
     Accounting Systems (CAS) and POS registration.
  3. VAT and billing: if you charge tenants, you'll invoice with 12% VAT or the
     percentage-tax regime, depending on how you're registered. Ask an
     accountant.
  4. Cash-heavy operations: laundromats are mostly cash and GCash. Your
     reconciliation story (sales vs. machine cycle counts) matters more than
     card processing.

  Practical product implications

  - Connectivity is unreliable. Consider an offline-first or queue-and-sync
    entry screen, since an employee can't lose a transaction when the Wi-Fi
    drops.
  - Low-end Android phones are the common device, so keep the app light (a PWA
    works well).
  - Price in PHP, and prefer local payment rails to avoid foreign-exchange fees.
  - Language: consider Filipino/Taglish labels for employee-facing screens.

  Revised starter set

  - Hosting and database: Render, Railway or Fly.io plus managed PostgreSQL
    (Neon) in Singapore.
  - Billing: manual GCash or bank-transfer invoicing first, then PayMongo or
    Xendit.
  - SMS and alerts: Semaphore or Viber for owner alerts, with Resend or Postmark
    for email.
  - Errors: Sentry.
  - Backups and exports: Cloudflare R2 or S3 (Singapore).
  - Auth: database-backed now, with Auth0 or Clerk later as planned.

  A PH accountant or lawyer should confirm the BIR and Data Privacy Act points
  before you launch to paying tenants. I can also add a short "PH compliance and
  providers" note to 00_docs/design/ if you'd like it kept with the design
  docs.
