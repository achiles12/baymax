# App Flows

This is the full catalog of Baymax flows. It covers what each role does in the app, what operators do on the server, the automatic system flows, and how the app recovers when something goes wrong. Each flow lists the actor, the trigger, the steps, the screens (from `ui_screens.md`) and the backend processes (from `backend_processes.md`).

The 13 must-work flows (F1–F13) are covered in detail in `critical_user_flow.md`, along with personas, failure cases and acceptance checks. They're summarized here so the catalog is complete. If this document and `critical_user_flow.md` disagree, `critical_user_flow.md` wins. Flows numbered F14 and up appear only here.

Roles: `saas_admin`, `tenant_admin`, `owner`, `staff`. Access rules come from `api_spec.md` § Roles and access matrix.

## Table of Contents

- [Flow summary](#flow-summary)
- [1. Access and accounts](#1-access-and-accounts)
- [2. Tenant administration](#2-tenant-administration)
- [3. Users](#3-users)
- [4. Items and prices](#4-items-and-prices)
- [5. Daily entry](#5-daily-entry)
- [6. Corrections and approvals](#6-corrections-and-approvals)
- [7. End of day: cash and inventory](#7-end-of-day-cash-and-inventory)
- [8. Reporting and oversight](#8-reporting-and-oversight)
- [9. Operator flows](#9-operator-flows)
- [10. Automatic system flows](#10-automatic-system-flows)
- [11. Error and recovery flows](#11-error-and-recovery-flows)
- [12. Later versions](#12-later-versions)
- [Open points](#open-points)

---

## Flow summary

**Critical** means the flow is one of the 13 detailed in `critical_user_flow.md`.

| ID | Flow | Actor | Critical | Screens | Processes |
|---|---|---|:-:|---|---|
| F1 | Tenant onboarding | `saas_admin` | ✅ | S3 | C1, J3, J4 |
| F2 | First login and set password | any new user | ✅ | S2, S1 | B5, B1 |
| F3 | Create users | `tenant_admin` | ✅ | S4 | D1, J3 |
| F4 | Set up item lists and prices | `owner` | ✅ | S5 | E1, E2 |
| F5 | Record a sale | `staff` (any tenant role) | ✅ | S6 | F1 |
| F6 | Record an expense | `staff`, `owner` | ✅ | S6 | F1 |
| F7 | Correct a mistake (change request) | `staff` → approver | ✅ | S7 | G1 |
| F8 | File an adjustment | `staff` → approver | ✅ | S6 | F1 (pending) |
| F9 | Approve or reject | `owner`, `tenant_admin` | ✅ | S8 | G2, G3, F2 |
| F10 | End-of-day close | `staff`, `owner` | ✅ | S13, S14 | I3, I1 |
| F11 | Check the business remotely | `owner` | ✅ | S9, S10, S8, S13 | H1, H2 |
| F12 | Export for the accountant | `owner` | ✅ | S11 | H3 |
| F13 | Forgot password | any | ✅ | S1, S2 | B4, B5 |
| F14 | Log in (returning user) | any | | S1 | B1 |
| F15 | Log out | any | | AppShell | B2 |
| F16 | Session restore and expiry | any | | all | B3, A3 |
| F17 | Account lockout | any | | S1 | B1 |
| F18 | Retry failed provisioning | `saas_admin` | | S3 | C2 |
| F19 | Suspend or reactivate a tenant | `saas_admin` | | S3 | C3, J5 |
| F20 | Change tenant name or user limit | `saas_admin` | | S3 | C3 |
| F21 | Edit a user or change their role | `tenant_admin` | | S4 | D2 |
| F22 | Deactivate or reactivate a user | `tenant_admin` | | S4 | D2, J5 |
| F23 | Change a price | `owner` | | S5 | E2 |
| F24 | Retire or restore an item | `owner` | | S5 | E2 |
| F25 | Record a variable-price sale | any tenant role | | S6 | F1, J2 |
| F26 | Back-dated entry | any tenant role | | S6 | F1 |
| F27 | Owner or admin posts an adjustment directly | `owner`, `tenant_admin` | | S6 | F1 |
| F28 | Void a transaction (change request) | `staff` → approver | | S7, S8 | G1, G2 |
| F29 | Cancel own change request | `staff` | | S7 | G4 |
| F30 | Track own requests | `staff` | | S7 | G1 reads |
| F31 | Browse and filter transactions | any tenant role | | S7 | `GET /transactions` |
| F32 | Second shift or corrected cash count | `staff`, `owner` | | S13 | I3 |
| F33 | Set up inventory items | `owner` | | S14 | `POST/PATCH /inventory` |
| F34 | Receive stock or adjust a count | any tenant role | | S14 | I1 |
| F35 | Respond to a low-stock warning | `owner` | | S14, S9 | I2 |
| F36 | Review the audit log | `owner`, `tenant_admin` | | S12 | H4 |
| F37 | Review cash variances | `owner` | | S13 | `GET /cash-counts` |
| O1 | Release and deploy | operator | | — | K1, K2, K3 |
| O2 | Recover a failed tenant migration | operator | | — | K2 |
| O3 | Provision a tenant from the command line | operator | | — | C1 |
| O4 | Restore one tenant | operator | | — | K5 |
| O5 | Revoke all sessions | operator | | — | J5 |
| O6 | Rotate a tenant DB role secret | operator | | — | C4 |
| O7 | Create the first SaaS Admin | operator | | — | `scripts/create_saas_admin.py` |
| Y1 | Nightly backup | scheduler | | — | K4 |
| Y2 | Orphan identity check | scheduler | | — | K6 |
| Y3 | Health and readiness checks | load balancer | | — | K7 |
| Y4 | Dashboard and badge auto-refresh | frontend timer | | S9, AppShell | H1, approvals count |
| E1 | Save fails on a bad connection | `staff` | | S6 | F1 idempotency |
| E2 | Two approvers act at once | approvers | | S8 | G2 lock |
| E3 | Target changed before approval | approver | | S8 | G2 |
| E4 | Tenant is migrating or unreachable | any | | all | K3, A7 |
| E5 | User limit reached | `tenant_admin` | | S4 | D1 |
| E6 | Rate limited | any | | S1, S2 | A2 |
| E7 | Permission denied or wrong tenant | any | | all | A5, A6 |

---

## 1. Access and accounts

### F2. First login and set password *(critical)*
- **Trigger:** a new user receives a set-password email, sent by F1 or F3.
- **Steps:**
  1. The user opens the link and lands on S2.
  2. They choose a password (at least 10 characters).
  3. They're sent to S1, log in, and land on their role's home screen.
- **Note:** the link expires after 1 hour and works once. See [Open points](#open-points) 3 for what happens when it expires.

### F13. Forgot password *(critical)*
- **Trigger:** the user taps "Forgot password" on S1.
- **Steps:**
  1. The user enters their email. The app always says a link was sent, whether or not the email exists.
  2. They follow the emailed link and set a new password on S2.
  3. Every existing session for that user ends, and they log in again.

### F14. Log in (returning user)
- **Trigger:** the user opens the app and has no valid session.
- **Steps:**
  1. The user enters their email and password on S1.
  2. On success, they land on their home screen: staff → Transaction entry, owner → Dashboard, tenant_admin → Users, saas_admin → Tenants.
- **Failures:**
  - Wrong password: "Email or password is incorrect" (`401`), whether or not the email exists.
  - Suspended tenant: `403 tenant_suspended`.

### F15. Log out
- **Trigger:** the user taps Log out in the AppShell menu.
- **Steps:** the session is revoked, the cookie is cleared, and the user goes back to S1.
- **Shared devices:** this matters on a shared branch tablet. The entry screen should show who is logged in (`system_design.md` § Risks).

### F16. Session restore and expiry
- **Trigger:** a page load or browser refresh.
- **Steps:**
  1. The SPA calls `GET /auth/me`.
  2. If the session is valid, the user's state is restored and they stay on the same page.
  3. If not (12 h absolute or 8 h idle has passed, or the session was revoked), the user is sent to S1.
- **Unsaved sale:** an in-progress sale isn't saved anywhere, so it's lost. That's acceptable because nothing was recorded.

### F17. Account lockout
- **Trigger:** five wrong passwords for one account.
- **Steps:**
  1. The account is locked for 15 minutes (`423`), and S1 says so.
  2. The user waits it out, or uses F13. A reset doesn't clear the lock early. See [Open points](#open-points) 4.

---

## 2. Tenant administration

### F1. Tenant onboarding *(critical)*
- **Trigger:** a new laundromat signs up, and the SaaS Admin taps "New tenant" on S3.
- **Steps:**
  1. The SaaS Admin enters the name, slug, timezone, currency and the Tenant Admin's email and name.
  2. The API returns `202`, and S3 shows "Provisioning…" while it polls the tenant's status.
  3. The tenant DB is created and migrated, starter items are seeded, and the Tenant Admin is created and emailed.
  4. The status becomes `active`, and the Tenant Admin continues with F2.

### F18. Retry failed provisioning
- **Trigger:** S3 shows a tenant as `failed`, with the reason.
- **Steps:** the SaaS Admin fixes the cause if needed (for example email settings) and taps Retry. Provisioning resumes, skipping the steps that already finished, and S3 polls until the tenant is `active`.

### F19. Suspend or reactivate a tenant
- **Trigger:** non-payment, offboarding or a security concern.
- **Steps:**
  1. The SaaS Admin sets the status to `suspended` on S3.
  2. Every session in that tenant ends at once. Logins fail with "tenant suspended", and the data is kept.
  3. Reactivating sets the status back to `active`, and users log in again as normal.

### F20. Change tenant name or user limit
- **Trigger:** a rename, or a pricing change that allows more users.
- **Steps:** the SaaS Admin edits the name or `user_limit` on S3. The slug and timezone can't be changed.

---

## 3. Users

### F3. Create users *(critical)*
- **Trigger:** a new employee or owner joins.
- **Steps:**
  1. The Tenant Admin taps "Add user" on S4 and enters the email, name and role (`owner`, `staff` or `tenant_admin`).
  2. The user is created and emailed a set-password link, and continues with F2.
- **Limit:** a 6th active user is blocked (see E5).

### F21. Edit a user or change their role
- **Trigger:** a correction or a promotion.
- **Steps:**
  1. The Tenant Admin edits the name or role on S4.
  2. The new role applies on that user's next request, so screens they can no longer use stop working for them straight away.
- **Guards:**
  - The Tenant Admin can't change their own role.
  - The tenant must keep at least one active `tenant_admin`.

### F22. Deactivate or reactivate a user
- **Trigger:** an employee leaves.
- **Steps:**
  1. The Tenant Admin sets the user to inactive on S4.
  2. The user's sessions end at once. Their past transactions and audit entries keep their name.
  3. Deactivating frees a seat under the 5-user limit. Reactivating needs a free seat.
- **Guard:** a Tenant Admin can't deactivate themselves.

---

## 4. Items and prices

### F4. Set up item lists and prices *(critical)*
- **Trigger:** a new tenant, or a change to the services offered.
- **Steps:**
  1. The owner opens S5 and picks a list: Sales, Expense or Adjustment.
  2. They add items with a name, a price type (fixed with a price, or variable) and a sort order.
  3. The entry screen (S6) shows the items in that order.

### F23. Change a price
- **Trigger:** the owner changes what something costs.
- **Steps:**
  1. The owner edits the price on S5.
  2. The new price applies to new transactions only. Past transactions keep the price they were saved with.
  3. The change is audited with the old and new price, and appears in F36.

### F24. Retire or restore an item
- **Trigger:** a service is discontinued.
- **Steps:** the owner sets the item to inactive on S5. It disappears from S6 but stays on past transactions and reports. Turning it active again brings it back. Items can't be deleted.

---

## 5. Daily entry

### F5. Record a sale *(critical, most frequent)*
- **Trigger:** a customer pays at the counter.
- **Steps:**
  1. The staff member opens S6 on the Sales tab.
  2. They tap item buttons (Wash, Dry, Soap…). Each tap adds one, and quantities change with a number pad.
  3. The running total shows on screen.
  4. They tap Save. The server prices the lines and saves them, and the confirmation shows the server's total.
  5. The screen clears for the next customer.
- **Target:** three taps for a typical sale, and a save in under 300 ms.

### F6. Record an expense *(critical)*
- **Trigger:** money is paid out (supplies, utilities, snacks).
- **Steps:** same as F5, on the Expense tab with expense items.

### F25. Record a variable-price sale
- **Trigger:** a service priced per job, such as fold by the kilo.
- **Steps:**
  1. The staff member taps a variable item, and S6 asks for a unit price through PriceInput.
  2. They enter the quantity and price and save.
- **Price rules:** the server accepts the entered price for variable items only. Fixed-item prices always come from the item list.

### F26. Back-dated entry
- **Trigger:** a sale or expense from an earlier day wasn't entered at the time, such as just after midnight or a missed entry.
- **Steps:**
  1. On S6 the user changes the business date (the default is today in the tenant's timezone) and saves.
  2. The entry counts toward that day's roll-up.
- **Limits:** future dates are refused. Staff can go back at most 7 days. Anything older has to be entered by an owner.

### F27. Owner or admin posts an adjustment directly
- **Trigger:** the owner needs to correct a total (for example a refund, or a fix to their own mistake).
- **Steps:**
  1. On the S6 Adjustment tab, the owner picks the side (sales or expense) and the direction (+ or −), adds lines and a required note, and saves.
  2. The adjustment posts immediately, with no approval step, and appears in F36.

---

## 6. Corrections and approvals

### F7. Correct a mistake (change request) *(critical)*
- **Trigger:** a staff member notices a saved transaction is wrong.
- **Steps:**
  1. On S7 they open the transaction and tap "Request correction".
  2. They enter the corrected lines and a reason, and submit.
  3. The original stays as it is, and the request waits in S8 for an approver.

### F8. File an adjustment *(critical)*
- **Trigger:** staff need to record a change that isn't a correction of one transaction (for example a refund).
- **Steps:** the staff member uses the S6 Adjustment tab with a note. The adjustment saves as `pending`, doesn't count in totals, and goes to S8.

### F9. Approve or reject *(critical)*
- **Trigger:** the approvals badge shows waiting items.
- **Steps:**
  1. The owner or admin opens S8. Each card shows who asked, when, the reason, and the old total next to the new total.
  2. They tap Approve, or Reject with a required note.
- **Approve:** a correction voids the original and adds a replacement; a void request voids the transaction; a pending adjustment becomes posted.
- **After:** reports reflect the result straight away.
- **Rule:** nobody can approve their own request.

### F28. Void a transaction (change request)
- **Trigger:** a transaction shouldn't exist at all (a duplicate or a test entry).
- **Steps:**
  1. On S7 the staff member taps "Request correction", chooses Void, and gives a reason.
  2. An approver approves it in F9.
  3. The transaction shows as `voided` on S7 and drops out of all totals. It's never deleted.

### F29. Cancel own change request
- **Trigger:** a staff member filed a request by mistake.
- **Steps:** while the request is still `pending`, the requester taps Cancel on S7. The request becomes `cancelled`, and the transaction is untouched.

### F30. Track own requests
- **Trigger:** a staff member wants to know whether a request was approved.
- **Steps:** S7 shows the status of the staff member's own requests (pending, approved, rejected or cancelled), with the approver's note when there is one.

---

## 7. End of day: cash and inventory

### F10. End-of-day close *(critical)*
- **Trigger:** the shop closes.
- **Steps:**
  1. The staff member counts the cash, opens S13, and enters the opening float and the counted amount.
  2. The server shows the expected amount and the variance. A variance other than zero is flagged.
  3. Optionally, they record supplies used or received on S14.

### F32. Second shift or corrected cash count
- **Trigger:** a shop runs two shifts, or a count was entered wrongly.
- **Second shift:**
  1. The staff member saves a count with another shift label.
  2. Its expected amount covers only transactions saved after the previous count. The previous counted amount becomes the default opening float.
- **Corrected count:** counts can't be edited. The staff member records a new count with a note. See [Open points](#open-points) 6 for a conflict with the `(business_date, shift)` unique rule.

### F33. Set up inventory items
- **Trigger:** the owner starts tracking supplies.
- **Steps:** on S14 the owner adds an item with a name, a unit and a reorder level, and can edit it or set it inactive later.

### F34. Receive stock or adjust a count
- **Trigger:** a delivery arrives, or a shelf count doesn't match the system.
- **Steps:**
  1. On S14 the user records `stock_in` with a quantity, or an `adjustment` with a signed quantity and a note.
  2. The stock level updates.
- **Limit:** a `used` movement can't take stock below zero.

### F35. Respond to a low-stock warning
- **Trigger:** stock falls to or below the reorder level.
- **Steps:**
  1. S14 shows a low-stock warning (and S9 too, if the dashboard shows it).
  2. The owner buys more. The purchase is recorded as an expense (F6) and the delivery as `stock_in` (F34).

---

## 8. Reporting and oversight

### F11. Check the business remotely *(critical)*
- **Trigger:** the owner wants to see how the shop is doing.
- **Steps:**
  1. The owner opens S9 and switches between Day, Week, Month and Half-year, moving back and forward in time.
  2. They look at the chart and the by-item breakdown.
  3. They check S10 for expenses, S8 for approvals waiting, and S13 for cash variances.
- **Refresh:** the dashboard refreshes about every 30 s.

### F12. Export for the accountant *(critical)*
- **Trigger:** month end or tax time.
- **Steps:** on S11 the owner picks a date range (up to 366 days) and taps Export CSV. The file opens directly in Excel or Google Sheets.

### F31. Browse and filter transactions
- **Trigger:** looking up a specific sale, or checking one person's entries.
- **Steps:**
  1. On S7 the user filters by date, type, status, who entered it, or item.
  2. They open one transaction to see its lines and any change requests.
  3. The list loads in pages.

### F36. Review the audit log
- **Trigger:** a suspected problem, or a routine check.
- **Steps:** on S12 the owner or admin filters by date, person, entity and action. Each entry shows who did what and when, with the old and new values. The audit log can't be edited.

### F37. Review cash variances
- **Trigger:** part of F11, or a pattern of shortfalls.
- **Steps:**
  1. On S13 the owner views the count history for a date range. Negative variances are highlighted.
  2. They compare against the day's transactions (F31) and audit entries (F36), and follow up with staff.

---

## 9. Operator flows

These are run by whoever operates the servers, using `scripts/` and the runbooks in `00_docs/runbooks/` (deliverable 5.8).

### O1. Release and deploy
1. Migrate the platform DB.
2. Migrate all tenant DBs (`migrate_all_tenants.py`). This continues past failures and gives a summary at the end.
3. Start the new API.
- **Compatibility:** migrations must work with the previous API version.
- **Lagging tenant:** a tenant whose migration failed gets `503 tenant_migrating` until O2 is done.

### O2. Recover a failed tenant migration
1. Read the summary from O1.
2. Fix the cause.
3. Re-run the migration for that one tenant.
4. Check `tenant_settings.schema_version`. The tenant comes back automatically.

### O3. Provision a tenant from the command line
- **Steps:** run `scripts/provision_tenant.py` to do the same thing as F1 without the UI. It's useful for the first tenant and for testing.

### O4. Restore one tenant
1. Pick a backup from Y1.
2. Restore it into a clean database (`restore_tenant.py`).
3. Point the tenant at it.
- **Other tenants:** unaffected.
- **Drill:** must be run before the first external tenant.

### O5. Revoke all sessions
- **Trigger:** a suspected breach.
- **Steps:** revoke every session (or every session in one tenant). Everyone has to log in again.

### O6. Rotate a tenant DB role secret
1. Set a new secret for `baymax_t_{slug}_app` in Postgres and in the configuration.
2. Restart the API, or evict the cached engine, so new connections use the new secret.

### O7. Create the first SaaS Admin
- **Trigger:** first setup of an environment.
- **Steps:** run `scripts/create_saas_admin.py` with an email. It creates the `saas_admin` identity and sends a set-password link, and the admin continues with F2 (deliverable 2.14).

---

## 10. Automatic system flows

### Y1. Nightly backup
- **Trigger:** scheduler.
- **Steps:** `pg_dump` runs for the platform DB and each tenant DB, and 30 days of dumps are kept.

### Y2. Orphan identity check
- **Trigger:** scheduler, nightly.
- **Steps:** reports tenant users whose platform identity is missing. It only reports, and doesn't fix anything.

### Y3. Health and readiness checks
- **Trigger:** the load balancer.
- **Steps:**
  - `/healthz` checks that the process is up.
  - `/readyz` checks that the platform DB is reachable. Tenant DBs are left out, so one broken tenant doesn't take the API down.

### Y4. Dashboard and badge auto-refresh
- **Trigger:** frontend timers.
- **Steps:** S9 re-fetches the roll-up every 30 s. The approvals badge re-fetches the pending counts every 60 s. S3 polls a tenant that is provisioning.

---

## 11. Error and recovery flows

### E1. Save fails on a bad connection
1. The save doesn't get a response.
2. S6 keeps the sale on screen and retries with the same `client_request_id`.
3. If the first attempt actually reached the server, the original record comes back. No duplicate is created.
4. The screen clears only after a successful save.

### E2. Two approvers act at once
- **What happens:** the first approval wins. The second approver gets "already decided" (`409 not_pending`), and S8 refreshes.

### E3. Target changed before approval
- **What happens:** the target transaction is no longer `posted`, for example because an earlier request already voided it. Approval fails with `409 target_changed`. The approver rejects the stale request, or the requester files a new one.

### E4. Tenant is migrating or unreachable
- **What happens:** the user sees "Baymax is updating, try again shortly" (`503`). Other tenants aren't affected.

### E5. User limit reached
- **What happens:** S4 blocks the 6th active user and says why. The Tenant Admin deactivates someone (F22), or asks for a higher limit (F20).

### E6. Rate limited
- **What happens:** too many login or reset attempts. The user sees a "try again in N seconds" message, based on `Retry-After`.

### E7. Permission denied or wrong tenant
- **What happens:**
  - A role that isn't allowed gets `403`. The UI hides those actions, but the server is what enforces the rule.
  - A record from another tenant returns `404`, so it can't be told apart from a record that doesn't exist.

---

## 12. Later versions

| Version | Flow |
|---|---|
| v1.x | Log in through Auth0, optionally with MFA. Existing users are migrated in batches or on first login |
| v1.x | Install the app on the branch tablet (PWA), possibly with an offline entry queue |
| v1.x | Export to `.xlsx` |
| v2 | Bookkeeping: posted transactions create journal entries, and the owner views ledgers and statements |
| v3 | POS sync: a POS device sends sales automatically, and staff entry stays available for corrections |
| v3 | Machine reconciliation: the owner uploads a cycle-count export, and the report compares washer and dryer cycles with Wash and Dry sold, flagging days beyond a tolerance |

---

## Open points

1. ~~No flow creates the first SaaS Admin (O7).~~ **Resolved 2026-10-30:** `scripts/create_saas_admin.py` added as deliverable 2.14.
2. **Owners and admins can't file change requests.** The access matrix allows `POST /change-requests` for `staff` only. An owner who mis-enters a sale can only fix it with an adjustment (F27), and can't void the original. Decide whether owners can void directly (audited) or file a request for another approver.
3. **No "resend set-password link" flow.** If the F2 link expires before first use, the user has to go through F13 themselves. A Tenant Admin or SaaS Admin can't resend the link from S4 or S3.
4. **Does a password reset clear a lockout?** F17 doesn't say. Clearing the lock on a successful reset would help low-tech users.
5. **Entries older than 7 days.** Staff can't back-date further than 7 days, and no flow says how they ask an owner to enter one.
6. **Corrected cash count conflicts with the unique rule.** `cash_counts` is unique on `(business_date, shift)`, so "record a new count" (F32) fails with `409 count_exists` for the same shift. Either allow a replacement count that links to the one it supersedes, or relax the constraint.
7. **Who resolves a stale request (E3)?** Rejecting it by hand works, but the server could auto-reject pending requests whose target is no longer `posted`.
8. **Shared branch logins.** F15 depends on staff logging out on a shared tablet. No auto-logout or quick user switch is defined.
