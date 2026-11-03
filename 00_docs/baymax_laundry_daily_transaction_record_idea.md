# SaaS Idea: Discovery Questions (Reusable Template)

A generic set of 20 questions to flesh out any new SaaS idea before building. Copy this file per idea, or just answer inline here. The goal is to force specificity — vague answers usually mean the idea isn't scoped enough yet to start building.

## Problem & Market

1. **Who exactly has this problem?**

Laundromat operators

2. **How do they solve this problem today**

currently using pen and paper to log transactions. move to realtime tracking. monitor transactions to prevent theft by staff. promote automation of branches by a single operator.

3. **How do you know this problem is real and painful**

currently reviewing transactions of a laundromat via logbook and google sheet.
prevalent theft by staff across the industry.

4. **Who are your first 3-5 potential users/customers you could actually get in front of?**
plan to bring in my own laundromat to monitor and record daily transactions realtime
my wife and her siblings.
other laundromats

## Scope & Core Workflow

5. **What's the single core workflow**

log records and transactions for the day.
log adjustments.
extract to daily reports.
roll-up and extract to monthly reports.
roll-up and extract to annual reports as needed by accountant.
track realtime transactions of a branch.

6. **What's the smallest version of this that's still useful?**

currently a manual process, transfer to excel or sheets.
no report generation. 

7. **What data does the system need to create, store, or calculate**, and where does that data come from (user entry, another system, a device, a third party)?

manual user entry of transactions.
calculate daily sales
calculate transactions
record transactions
calculate monthly
calculate bi-annual
calculate annual

v2: eventually a bookeeping app for a laundromat business
v3: extract cycle counts from laundry machine export and balance with the daily transcation records

8. **What's explicitly out of scope for v1?** 

individual transaction tracking
daily report
monthly reporting.
complete bookeeping can come from v2 or v3

## Users, Roles & Trust

9. **Who are the different types of users**

admin
owner
branch_login / laundry_tech

10. **What happens when something goes wrong or someone makes a mistake** — can entries/records be edited or deleted, and should there be a history of who changed what?

owner can make updates via a change request but still to be approved by the owner.
branch_login add change request on the record to be approved by the owner.

11. **Is there a trust or verification problem** (e.g. someone could lie, skip a step, or mis-report something) that the product needs to guard against?

missed transactions,
late transactions
incorrect reporting of transactions.

12. **What device/context is this used in** (phone, desktop, shared kiosk, in the field with no signal)? That shapes nearly every UI and architecture decision.

can be used in phone, tablet or browser.
probably a responsive layout would be best.

## Output & Value

13. **What does success look like to the user, concretely** — a report, a notification, a number going down, time saved? Name the 2-3 things that would make them open this daily or weekly.

daily transactions are logged correctly.
business owner / operator can see the transactions realtime
daily reports
no longer manual transfer from log book to excel
daily reports and monthly reports
extract to excel or csv

14. **Does this need to connect to anything else** (accounting software, another SaaS, a payment processor, an existing spreadsheet)? Is that integration required for v1 or a later nice-to-have?

none yet, will need to connect to machines on v3 for auditing of transactions vs cycle counts

15. **Are there compliance, legal, or industry-specific rules** this needs to respect (data retention, audit trail, accessibility, regulated terminology)?

none yet

## Business Model

16. **Who pays, and why would they pay rather than keep doing it for free/manually?** Be specific about the price point and what tier of value justifies it.

will be subscription based from a business owner / operator.
this will be multitenat db separated.

17. **What's the pricing model** — per user, per location, flat monthly, usage-based — and does that match how the value actually scales for the customer?

admin - backend or IT

per tenant, up to 5 users per tenant.
business owner / operator
staff

18. **What's the realistic path to your first paying customer**, and roughly how many customers you'd need to make this worth your time?

none, this can be my personal tracker for a business i want to launch
for this to be profitable, maybe 10 tenants?


## Risk & Validation

19. **What's the biggest assumption this idea depends on**, that if wrong, kills the whole thing? How could you test that cheaply, before investing in a full build?

will launch and be for own use, will need market verification.

20. **What's the smallest experiment you could run this week** — a conversation, a mockup, a manual/no-code version — to get real signal before writing product code?

i want to launch it now and make if for personal use for my own business.
monetization from other tenants use it be a secondary goal

---


## what i need to learn from this build

Agent Skills
Graphiify

Git Diagram
Git Ingest



Claude code
Supabase - backend
Vercel  - deployment
Namecheap - domain
Stripe - payments? other payment gateway?
Github - version control and CI/CD
Resend - emails
Clerk - authorization, other methods?
Cloudflare - DNS - What?
PostHog - Analytics?
Upstash - Redis?
Pinecone - vectorDB? postgresBD? cloud db?

CI/CD pipeline
Github - 
Grafana - 
Prometheus - 
Jenkins - 
Nexus - 

## build the following docs on ~/00_docs/

- `api_spec.md`
- `arch_overview.md`
- `critical_user_flow.md`
- `data_model.md`
- `prod_req_doc.md`
- `system_design.md`
- `test_strat.md`
- `file_system.md`

*Next step: fill in answers above for a specific idea, then turn it into a PRD / MVP feature list.*


