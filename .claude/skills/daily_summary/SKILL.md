---
name: daily-summary
description: use this skill when summarizing the tasks for the day
---

## Prerequisites

Before writing any code, read:
**Project documents:** `~/00_docs/design/`
- `api_spec.md`
- `arch_overview.md`
- `critical_user_flow.md`
- `data_model.md`
- `prod_req_doc.md`
- `system_design.md`
- `test_strat.md`
- `file_system.md`
- `object_masterlist.md`

**Deliverables and timeline:** `~/00_docs/design/deliverables.md`

**Project notes** `~/00_docs/stage**/`
Place documents and files generated (design, test script, manual test script, bash, python, sql, etc) on the correct stage directory as described by the `deliverables.md`
Files written in these directories must have a prefix of `yyyymmdd_` — use today's date. if there is an existing file with a similar filename, add a prefix of `_[number]`

**Project Summary** `~/00_docs/summary/`
Files written in these directories must have a prefix of `yyyymmdd_` — use today's date. if there is an existing file with a similar filename, add a prefix of `_[number]`

**todo** `~/00_docs/todos/`
Files written in these directories must have a prefix of `yyyymmdd_` — use today's date. if there is an existing file with a similar filename, add a prefix of `_[number]`

**Objects**
Objects are to be written on the appropriate directory as described by `file_system.md`
---

# Overview
Summarize the accomplishments since last execution of the daily-summary. Place the created summary on **Project notes**

## Summary
Summarize todays accomplishements.
- objects built
- status of testing
- where the objects built fit in the project

## ToDo
Create a list of next actions to take.
This should be the next 3 steps to be taken.
Give the current and the immediate succeding stage status and schedule, pull it from `gantt_sched.md` and `deliverables.md`. Place on **todo**.

## Update schedule
Project schedule and timeline on `gantt_sched.md`. back-up the schedule by adding a prefix of `yyyymmdd_` to a copy of the `gantt_sched.md`. Create or update the masterlist of all objects with their current build status and date completed.

## When to Use
- when instructed to summarize todays activities