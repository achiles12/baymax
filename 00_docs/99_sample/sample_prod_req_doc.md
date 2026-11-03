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

AthleteFlow is a web-based mutli-tenant sports team and athlete training management application designed for amateur and semi-professional sports teams. The product enables teams to create, assign, track, and prioritize training, recovery and rehabilitation tasks in a centralized workspace, replacing fragmented workflows across email threads, spreadsheets, and chat tools.

This document defines the requirements for the **v1.0 release**, targeted for launch in Q3 2026.

---

## Problem Statement

Sports teams currently meet only during practice leaving individual training, recovery and rehab to the individual athlete.
Athletes in amateur and semi-pro leagues struggle with task visibility and accountability.

Key pain points identified through user research include:

- Everything is left to the individual no proper guidance and tracking
- athletes lack a training plan, recovery plan and or recovery menu on a daily or weekly basis
- athletes need a specific drills, exercises or activities tailored for the athlete
- no tracking of training accomplishments
- Teamates would benefit from competition, seeing a teammate train harder can inspire other teammates
- give accountability to coaches, trainiers and athletes

> **Opportunity:** Build a lightweight, intuitive athlete management solution that balances simplicity with just enough structure for sports teams of 5–25 people.

---

## User Personas

### Persona 1 - Alex, the SaaS Admin
- **Age:** 30 | **Role:** Admin
- **Goals:** Creates Tenant registry, tenant onboarding.
- **Frustrations:** Repetetive task of onboarding
- **Tech comfort:** High

### Persona 2 — Bryan, the Coach
- **Age:** 34 | **Role:** Operations Manager, 12-person team
- **Goals:** Know what everyone is working on; Identify athelete strength and weakneses, advantage and disadvantage; Provide Targets
- **Frustrations:** Only able to meet and train the team once a week and another on game day
- **Tech comfort:** Low

### Persona 3 — Charlie, the Trainer
- **Age:** 27 | **Role:** Conditioning and Rehab Coach
- **Goals:** Provide and track the progress of training and recovery activities of the atheletes; Ensure that training activities are specific to the individual;
- **Frustrations:** Lack of time to monitor all individuals of the whole team
- **Tech comfort:** Medium

### Persona 4 — Donovan, the Athlete
- **Age:** 15 | **Role:** Athelete, Basketball player
- **Goals:** Review and complete a tailored training, recovery and rehabilitation regiment
- **Frustrations:** No proper guidance, no tracking, no accountability
- **Tech comfort:** Medium

### Persona 5 - Ego, the Tenant Admin
- **Age:** 25 | **Role:** Tenant Admin
- **Goals:** Create users for the tenant. 
- **Frustrations:** Just a super user that has access to the tenant only
- **Tech comfort:** Medium


## Jobs To Be Done
- When [situation], user wants to [motivation], so they can [outcome]

- When training individually training, the athlete needs a training menu tailored for him, to maximize his personal strengths and improve on his weaknesess

- During training planning, the trainer or coach can give specific drills and exercises to each athlete, to maximize the gains

- During training, the trainer and coach can review the progress of the athlete and be able to make changes to the training plan as needed to correct the training results

- During training, athletes can monitor their progress and check in on teammates' progress, this will add a game-design elements to training and will enhance the competitiveness of the athletes within the team.

## Features
### Must Have
- This must be a multi-tenant application, schema separated.
- Authentication will be on DB, but must be planned to be migrated to Auth0

- Coach can see the list of athletes on his team
- Trainer can see the list of athletes on the team
- Upon clicking on athlete a coach or a trainer can view, updated, delete, add exercises to be performed by the athlete

- Athlete can see the activities he needs to complete
- Activities for the day are shown
- Athelete can navigate to other dates to view past and future activities

### Should Have
- User creation process
- Tenant onboarding process
- User Interface

### Nice to Have
- Dashboard for the athlete of the completed exercises
- Dashboard for the Coaches on the status of the athletes
- Dashboard for the trainers on the status of the athletes

## Success Metrics
- KPI 1: Multiple Tenants and Authentication is working.
- KPI 2: tasks for athletes can be created and monitored

## Non-Goals
- Explicitly list what we are NOT building




