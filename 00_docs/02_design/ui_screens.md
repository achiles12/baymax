# UI Screens

Every screen in Baymax v1: what it's for, who can use it, what's on it, and the rules it must follow. This doc gathers what is spread across the other design docs. Flows (F1–F13) are from `critical_user_flow.md`, permissions from the access matrix in `api_spec.md`, page components from `object_masterlist.md`, and acceptance criteria from `deliverables.md`. If this doc disagrees with one of those, the source doc wins and this doc should be corrected.

Roles: `saas_admin` (SaaS Admin), `tenant_admin` (Tenant Admin), `owner` (Franchise Owner), `staff` (Employee / branch login).

## Table of Contents

1. [Screen summary](#screen-summary)
2. [Navigation and home screens](#navigation-and-home-screens)
3. [Design rules for all screens](#design-rules-for-all-screens)
4. [S1. Login](#s1-login)
5. [S2. Set password / reset password](#s2-set-password--reset-password)
6. [S3. Tenants (SaaS Admin)](#s3-tenants-saas-admin)
7. [S4. Users](#s4-users)
8. [S5. Items](#s5-items)
9. [S6. Transaction entry](#s6-transaction-entry)
10. [S7. Transactions list](#s7-transactions-list)
11. [S8. Approvals](#s8-approvals)
12. [S9. Dashboard](#s9-dashboard)
13. [S10. Expenses dashboard](#s10-expenses-dashboard)
14. [S11. Reports and export](#s11-reports-and-export)
15. [S12. Audit log](#s12-audit-log)
16. [S13. Cash on hand](#s13-cash-on-hand)
17. [S14. Inventory](#s14-inventory)
18. [Shared components](#shared-components)
19. [Open points](#open-points)

---

## Screen summary

| # | Screen | Page component | Roles | Flows | Deliv. |
|---|---|---|---|---|---|
| S1 | Login | `LoginPage` | Public | F2, F13 | 2.3 |
| S2 | Set password / reset password | `PasswordResetPage` | Public (token link) | F2, F13 | 2.4 |
| S3 | Tenants | `TenantsAdminPage` | `saas_admin` | F1 | 2.8 |
| S4 | Users | `UsersPage` | `tenant_admin` | F3 | 2.10 |
| S5 | Items | `ItemMaintenancePage` | `owner` edits; `tenant_admin`, `staff` read-only | F4 | 3.1 |
| S6 | Transaction entry (Sales / Expense / Adjustment) | `TransactionEntryPage` | `staff`, `owner`, `tenant_admin` | F5, F6, F8 | 3.6 |
| S7 | Transactions list | `TransactionsListPage` | `staff`, `owner`, `tenant_admin` | F7 | 3.7, 4.1 |
| S8 | Approvals | `ApprovalsPage` | `owner`, `tenant_admin` | F9 | 4.3 |
| S9 | Dashboard | `DashboardPage` | `owner`, `tenant_admin`, `staff` | F11 | 4.5 |
| S10 | Expenses dashboard | `ExpensesDashboardPage` | `owner`, `tenant_admin`, `staff` | F11 | 4.6 |
| S11 | Reports and export | `ReportsPage` | `owner`, `tenant_admin` | F12 | 4.7 |
| S12 | Audit log | `AuditLogPage` | `owner`, `tenant_admin` | F7, F9 | 4.8 |
| S13 | Cash on hand | `CashOnHandPage` | `staff`, `owner`, `tenant_admin` | F10 | 5.2 |
| S14 | Inventory | `InventoryPage` | `owner` edits items; all tenant roles record movements | F10 | 5.1 |

---

## Navigation and home screens

`AppShell` wraps every signed-in screen. It reads the role from `AuthContext`, sends unauthenticated users to Login, and hides screens the role can't use. Hiding is a convenience only, because the server enforces the same rules (`system_design.md` § Frontend design).

After login, each role lands on its home screen (F2):

| Role | Home screen | Navigation items |
|---|---|---|
| `saas_admin` | Tenants | Tenants |
| `tenant_admin` | Users | Users, Entry, Transactions, Approvals, Dashboard, Expenses, Reports, Cash, Inventory, Items (read-only) |
| `owner` | Dashboard | Dashboard, Entry, Transactions, Approvals, Expenses, Reports, Cash, Inventory, Items |
| `staff` | Transaction entry (Sales) | Entry, Transactions, Cash, Inventory, Dashboard, Expenses, Items (read-only) |

- **Phone:** bottom navigation. Owners and Tenant Admins have more items than fit, so the bottom bar shows the four most used and puts the rest under **More** (see [Open points](#open-points)).
- **Tablet and desktop:** side navigation with all items.
- **Approvals badge:** shows the count of waiting items and refreshes every 60 seconds.
- A `saas_admin` never sees tenant screens. Their sessions get `403` from tenant routes.

---

## Design rules for all screens

From `arch_overview.md`, `system_design.md` and the cross-flow rules in `critical_user_flow.md`:

- **Responsive.** One set of components serves phone, tablet and desktop.
- **Built for low tech comfort.** Large touch targets (minimum 48 px) and plain wording ("Add wash", not "Create line item").
- **Little or no typing on entry screens.** Fixed-price items need no keyboard. A number pad appears for quantities and variable amounts.
- **Saved records are read-only.** No screen offers edit or delete on a saved transaction, cash count or audit entry. Corrections go through a change request (S7 → S8).
- **The server's figures win.** Totals shown while entering are previews. The confirmation shows the total the server calculated.
- **Money** is shown with the tenant's currency symbol and 2 decimals (e.g. ₱208.00).
- **Dates** use the tenant's timezone. "Today" means today's business date for the tenant.
- **Empty states** use `EmptyState` with a short sentence and, where it makes sense, the next action.
- **Errors** are plain sentences. Login failures always show the same generic message.

---

## S1. Login

**Purpose:** sign in. **Who:** everyone. **Flows:** F2, F13.

**On screen**
- Email and password fields, and a **Log in** button
- **Forgot password** link

**Behavior**
- On success, opens the home screen for the user's role (see [Navigation](#navigation-and-home-screens)).
- A wrong email or password shows one generic message. After five failures the account is locked for 15 minutes.
- **Forgot password** asks for an email and always replies "If that email is registered, we've sent a link", whether or not the email exists.

---

## S2. Set password / reset password

**Purpose:** set a first password from the welcome email, or a new one after a reset. **Who:** anyone holding a valid link. **Flows:** F2, F13.

One page serves both cases (`PasswordResetPage`).

**On screen**
- New password and confirm password fields (at least 10 characters)
- **Set password** button

**Behavior**
- The link's token is single use. Reset links last 1 hour.
- An expired or used link shows "This link has expired" with a button to request a new one.
- On success the user goes to Login. After a reset, all of the user's existing sessions are ended.

---

## S3. Tenants (SaaS Admin)

**Purpose:** onboard and manage laundromat tenants. **Who:** `saas_admin`. **Flow:** F1.

**On screen**
- A list of tenants showing name, slug, status (`provisioning`, `active`, `failed`, `suspended`) and created date
- **New tenant** button

**New tenant form:** business name, slug, timezone, currency, Tenant Admin's name and email.

**Behavior**
- After submit, the row shows progress and then **Active**.
- A `failed` tenant shows the failure reason and a **Retry** button. Retrying is safe to repeat.
- A duplicate slug or email is rejected before anything is created.
- **Suspend** is available on active tenants.
- The SaaS Admin can't open any tenant's data from here.

---

## S4. Users

**Purpose:** add, change and deactivate users within the tenant. **Who:** `tenant_admin`. **Flow:** F3.

**On screen**
- User list with name, email, role and active/inactive status
- A counter such as "3 of 5 users"
- **Add user** button

**Add user form:** name, email and role (`owner`, `staff` or `tenant_admin`). Saving emails a set-password link (S2).

**Behavior**
- At 5 active users, **Add user** is disabled with the text "User limit reached. Deactivate someone or contact support."
- **Deactivate** ends the user's sessions at once and keeps their history.
- The last active Tenant Admin can't be deactivated.
- Users can't change their own role.

---

## S5. Items

**Purpose:** maintain the item lists that drive the entry screen. **Who:** `owner` edits; `tenant_admin` and `staff` see the lists read-only. **Flow:** F4.

**On screen**
- Tabs for the three lists: **Sales**, **Expense** and **Adjustment**
- Each item shows its name, its price type (fixed with an amount, or variable) and whether it's active
- Owner only: **Add item**, edit, reorder, and an on/off switch

**Add or edit item:** name, then **Fixed price** (enter the amount) or **Variable price** (no amount).

**Behavior**
- Display order here is the order of the buttons on the entry screen, so owners put the most used items first.
- A price change applies to new transactions only and is written to the audit log.
- Switching an item off removes it from the entry screen but keeps it in history.
- A new tenant starts with the starter items (Wash, Dry, Fold, Soap, Fabric conditioner; Staff water, Staff ice, Staff snacks, Utility: electric, Utility: rent, Utility: water).
- Staff and Tenant Admin see no edit controls.

---

## S6. Transaction entry

**Purpose:** record sales, expenses and adjustments at the counter. **This is the most used screen and the one to optimize.** **Who:** `staff`, `owner`, `tenant_admin`. **Flows:** F5, F6, F8.

**Target:** a typical sale (a few fixed-price items) in under 15 seconds, with three taps plus **Save** and no typing.

**On screen**
- A type switch: **Sales** (the default), **Expense** and **Adjustment**
- The business date, defaulting to today. Staff can choose up to 7 days back.
- Item buttons for the selected type (`ItemPicker`), in the order set on S5
- The running list (`TransactionLineList`): each line with quantity, amount, **−** and **✕**, plus a running total
- An optional short note field (Expense, Adjustment). For an adjustment the note holds the reason.
- A **Save sale** / **Save expense** / **Save adjustment** button

**Behavior**
- Tapping a fixed-price item adds it. Tapping it again raises the quantity.
- Tapping a variable-price item opens a number pad (`PriceInput`) for the quantity or amount.
- **−** and **✕** fix slips before saving. Nothing can be edited after saving.
- On save: a short confirmation with the server's total ("Saved ₱208.00"), then an empty screen ready for the next customer.
- **Adjustment mode:** choose whether it affects Sales or Expenses and whether it adds or subtracts, then choose adjustment items and amounts and give a reason. A staff adjustment saves as **Pending approval** and doesn't count in totals until approved. An owner or Tenant Admin adjustment posts immediately.
- **Lost connection:** the entry stays on screen with "Not saved yet, retrying…" and retries with the same `client_request_id`. The screen isn't cleared until the server confirms, and a double tap saves once.
- Fixed prices come from the server's item list, so a price changed in the browser has no effect.

**Layout:** single column on a phone. Two columns on tablet and desktop: item buttons on the left and the current sale on the right.

---

## S7. Transactions list

**Purpose:** find saved records and request corrections. **Who:** all tenant roles can view; `staff` can file corrections. **Flow:** F7.

**On screen**
- Filters: date (default today), type, status and user
- Rows showing time, type, user, total and a status (`StatusBadge`): posted, pending approval, correction pending, voided
- Tapping a row opens its lines
- Staff: a **Request correction** action on each posted record

**Request correction:** choose **Cancel this transaction** or **Change it** (and edit the lines). A reason is required.

**Behavior**
- A record with a pending request shows **Correction pending** and counts as originally entered until someone decides.
- There can be only one pending request per transaction. The requester can withdraw it while it's pending.
- After approval the original shows as **voided** and the corrected record appears next to it. Both stay visible.
- After rejection the record is unchanged and the requester sees the owner's note.

---

## S8. Approvals

**Purpose:** decide change requests and staff adjustments. This is the anti-pilferage control. **Who:** `owner`, `tenant_admin`. **Flow:** F9.

**On screen**
- One queue, oldest first, holding change requests and pending staff adjustments
- Each card shows who asked, when, the reason, and what changes (old total → new total)
- **Approve** and **Reject** buttons

**Behavior**
- Reject requires a short note.
- Nobody can approve their own request.
- The result appears at once on the transactions list and in the roll-ups.
- The navigation badge shows how many items are waiting.

---

## S9. Dashboard

**Purpose:** see how the business is doing, including remotely on a phone. **Who:** `owner` (home screen), `tenant_admin`, `staff`. **Flow:** F11.

**On screen**
- Period selector (`PeriodSelector`): **Day / Week / Month / Half-year**, with back and forward arrows
- Totals for the period: sales, expenses and net
- A chart of sales vs expenses (`RollupChart`)
- A table of totals by item
- The latest transactions
- Pointers to waiting approvals and to a non-zero cash variance

**Behavior**
- Refreshes automatically about every 30 seconds, so a sale saved at the counter appears without a manual reload.
- Only posted, approved records count.
- Must be usable on a phone.

---

## S10. Expenses dashboard

**Purpose:** see where money goes (utilities, staff snacks and so on). **Who:** `owner`, `tenant_admin`, `staff`. **Flow:** F11.

**On screen**
- The same period selector as S9
- Expense totals by item for the period, with a chart

---

## S11. Reports and export

**Purpose:** give the accountant a file instead of a hand-copied spreadsheet. **Who:** `owner`, `tenant_admin`. **Flow:** F12.

**On screen**
- A date range picker (at most 366 days)
- An **Export CSV** button

**Behavior**
- Downloads one row per transaction line. The file opens directly in Excel or Google Sheets.
- The exported totals must equal the totals shown on screen for the same range.

---

## S12. Audit log

**Purpose:** see who changed what and when. **Who:** `owner`, `tenant_admin`. **Deliverable:** 4.8.

**On screen**
- Filters: actor, entity, action and date
- A read-only list of entries: time, actor, action, entity, and the reason where there is one (price changes, approvals, rejections, voids, user changes)

---

## S13. Cash on hand

**Purpose:** close the day by counting cash against what the system expects. **Who:** `staff`, `owner`, `tenant_admin`. **Flow:** F10.

**On screen**
- Business date and shift
- Opening float and counted amount, entered with a number pad
- **Expected** amount (opening + the day's sales − expenses ± approved adjustments), calculated by the server
- **Variance** (counted − expected)
- **Save** button and a list of past counts

**Behavior**
- A non-zero variance is flagged and visible to the owner, including on the Dashboard.
- Only one count is allowed per date and shift. Counts can't be edited, so a wrong count is fixed by recording another one with a note.

---

## S14. Inventory

**Purpose:** track supplies such as detergent and fabric conditioner. **Who:** all tenant roles record usage and deliveries; only `owner` creates or edits inventory items. **Flow:** F10.

**On screen**
- A list of supplies with current stock and reorder level
- A low-stock warning on items at or below their reorder level
- Record **Used** or **Received** with a quantity and an optional note
- Owner only: add and edit inventory items

**Behavior**
- Stock is worked out from movements and can't go below zero.

---

## Shared components

| Component | Used on | Purpose |
|---|---|---|
| `AppShell` | All signed-in screens | Navigation, role guards, home screen per role |
| `ItemPicker` | S6 | Large item buttons for the selected list |
| `TransactionLineList` | S6, S7 (correction) | Lines with quantity, amount, **−**/**✕**, running total |
| `PriceInput` | S6, S13 | Number pad for quantities and amounts |
| `StatusBadge` | S7, S8 | Posted, pending, correction pending, voided |
| `EmptyState` | All lists | Friendly message when there's nothing to show |
| `PeriodSelector` | S9, S10 | Day / Week / Month / Half-year with back and forward |
| `RollupChart` | S9, S10 | Sales vs expenses chart |

---

## Open points

1. ~~No page component for Reports and export (S11).~~ **Resolved 2026-10-30:** `ReportsPage` added to `object_masterlist.md` (4.7).
2. ~~No page component for the Audit log (S12).~~ **Resolved 2026-10-30:** `AuditLogPage` added to `object_masterlist.md` (4.8).
3. **Phone navigation for owners and Tenant Admins.** They have 9 or 10 items, which don't fit in a bottom bar. A "More" menu is proposed above. Confirm which four items stay in the bar.
4. **Staff access to the dashboards.** The access matrix in `api_spec.md` lets `staff` view roll-ups and expense reports. Confirm the owner wants staff to see sales totals; if not, change the matrix first.
5. **Shift on Cash on hand.** "One count per date and shift" implies a shift selector. Shifts aren't defined anywhere yet (names, how many per day, or whether a tenant can configure them).
6. **URL routes** for each screen aren't defined. Add them here when the frontend router is set up.
7. **Wireframes.** This doc describes content and behavior only. Add wireframes (Figma or sketches), starting with S6, before building M3.
