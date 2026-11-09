# Product Requirements Document

## Table of Contents

1. [Summary](#summary)
2. [Problem Statement](#problem-statement)
3. [User Personas](#user-personas)
4. [Jobs To Be Done](#jobs-to-be-done)
5. [Features](#features)
6. [Success Metrics](#success-metrics)
7. [Non-Goals](#non-goals)

## Summary

Project Baymax is a web based transaction monitoring application.
It tracks transactions of the business in aid of bookkeeping.
- sales transactions
- expense transactions
- salary
- capital costs
- operating costs
- etc.

will eventually interface with a POS.
- take in sales, transmit transactions to baymax

will eventually interface with cycle counts of laundy machines
- take cycle counts from machines to balance with sales transactions 

will roll-up transactions into summaries
- daily
- weekly
- monthly
- half year

this is a database separated multi-tenant SaaS

---

## Problem Statement

a single one stop shop to monitor day to day operations and remotely manage a laundromat.
be able to track day to day transactions and roll-up the data into daily, weekly, and monthly charts and tables.
be able to audit transaction against cycle counts of machines in order to minimize employee pilferage.
be able to maximize the use of resources by properly tracking inventory supplies.
be able to project salary and expenses.

---

## User Personas

### Persona 1 - Alex, the SaaS Admin
- **Age:** 30 | **Role:** Admin
- **Goals:** Creates Tenant registry, tenant onboarding.
- **Frustrations:** Repetetive task of onboarding
- **Tech comfort:** High

### Persona 2 — Lindsay, the Franchise Owner
- **Age:** 34 | **Role:** Owner and Operator of the laundromat
- **Goals:** 1. Log Transactions on to the application
              - add sale transactions. add items eligble to account onto the transaction:
                 -- wash
                 -- dry
                 -- fold
                 -- soap
                 -- fabric conditioner
                 -- have option to add more via the franshise owner
                 -- pricing is maintained by franchise owner
                   -- fixed price already determined by owner
                   -- variable price only available to franchise owner
              - add expense transaction. add items eligble to account onto the transaction:
                 -- staff water
                 -- staff ice
                 -- staff snacks
                 -- utility: electric
                 -- utility: rent
                 -- utility: water
                 -- have option to add more via the franshise owner
                 -- pricing is maintained by franchise owner
                   --- fixed price already determined by owner
                   -- variable price only available to franchise owner
              - add adjusment transaction. add items eligble to account onto the transaction:
             2. See daily, weekly, and monthly transaction roll-up report
             3. Review and Approve Transaction balancing update
             4. Maintain transaction item lists for:
                 -- Sales Transaction Items
                 -- Expense Transaction Items
                 -- Adjustment Transaction Items
- **Frustrations:** need to properly monitor records and transactions remotely
- **Tech comfort:** Medium

### Persona 3 — Mathilda, the Laundromat Employee
- **Age:** 27 | **Role:** Laundromat Employee at POS
- **Goals:** 1. Log Transactions on to the application
               - add sale transactions. add items eligble to account onto the transaction:
                  -- wash
                  -- dry
                  -- fold
                  -- soap
                  -- fabric conditioner
                  -- have option to add more via the franshise owner
                  -- pricing is maintained by franchise owner
                    -- fixed price already determined by owner
                    -- variable price
               - add expense transaction. add items eligble to account onto the transaction:
                  -- staff water
                  -- staff ice
                  -- staff snacks
                  -- utility: electric
                  -- utility: rent
                  -- utility: water
                  -- have option to add more via the franshise owner
                  -- pricing is maintained by franchise owner
                    -- fixed price already determined by owner
                    -- variable price
               - add adjusment (expense or sales) transaction. add items eligble to account onto the transaction for approval of franchise owner.
             2. See daily, weekly, and monthly transaction roll-up report
             3. Maintain transaction item lists for:
                 -- Sales Transaction Items
                 -- Expense Transaction Items
                 -- Adjustment Transaction Items
- **Frustrations:** Must be intuitive to use and quick to be able to cater to customers.
- **Tech comfort:** Low

### Persona 4 - Iñigo, the Tenant Admin
- **Age:** 25 | **Role:** Tenant Admin
- **Goals:** 1. Log Transactions on to the application
              - add sale transactions. add items eligble to account onto the transaction:
                 -- wash
                 -- dry
                 -- fold
                 -- soap
                 -- fabric conditioner
                 -- have option to add more via the franshise owner
                 -- pricing is maintained by franchise owner
                   -- fixed price already determined by owner
                   -- variable price
              - add expense transaction. add items eligble to account onto the transaction:
                 -- staff water
                 -- staff ice
                 -- staff snacks
                 -- utility: electric
                 -- utility: rent
                 -- utility: water
                 -- have option to add more via the franshise owner
                 -- pricing is maintained by franchise owner
                   --- fixed price already determined by owner
                   --- variable priceKnow what everyone is working on; Identify athelete strength and weakneses, advantage and disadvantage; Provide Targets
              - add adjusment transaction. add items eligble to account onto the transaction:
             2. See daily, weekly, and monthly transaction roll-up report
             3. Review and Approve Transaction balancing update
             4. Create Users for the tenant
- **Frustrations:** Just a super user that has access to the tenant only
- **Tech comfort:** Medium


## Features
### Must Have
- This must be a multi-tenant application, database separated.
   -- Authentication will be on DB
   -- but must be planned to be migrated to Auth0

- be a point of sale
   -- record sales transactions
   -- record expense transactions
   -- record adjustment transactions
- create reports and dashboards
   -- create daily transaction roll-up
   -- create weekly transaction roll-up
   -- create monthly transaction roll-up
- monitor inventory and supplies
- monitor cash on hand

### Should Have
- Tenant onboarding process

- User Interface
   -- transaction entry screen
      --- fixed price
      --- variable price transaction
      --- transaction item maintenance screen
   -- view all transactions
   -- dashboard for daily, weekly, and mothly roll-ups
   -- dashboard for expenses
   -- Login screen
   -- User Maintenance

### Nice to Have
- ingest cycle data from LG washing machine
- Report of daily, weekly, and monthly transactions

## Success Metrics
- KPI 1: Multiple Tenants and Authentication is working.
- KPI 2: Transactions are recorded. 2 transaction types (fixed price, variable price)
- KPI 3: transaction items are added onto transactions
- KPI 4: transaction items are maintained - (fixed price)
- KPI 5: Dashboard is present


## Non-Goals
- Explicitly list what we are NOT building
- will be reevaluated in the future




