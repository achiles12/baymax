---
name: build-object
description: Use when building any domain object for the Baymax project — models, services, schemas, migrations, or API routes. Trigger whenever the user says "create a [model/service/etc]", "add a new [entity]", or "build the [X] object".
---

# Building Objects

## Overview

Build the object based on the project design documentation. Test the created objects to make sure that the requirements are met. Create and write manual testing steps for testing on a different Windows machine on the same network. Use the brainstorm skill to create and write documentation for the specific objects. Cross reference the objects to be built between `file_system.md` and `deliverables.md`.

## When to Use

- when instructed to build object

## Prerequisites

Before writing any code, read:
**Project documents:** `~/00_docs/02_design/`
- `api_spec.md`
- `arch_overview.md`
- `critical_user_flow.md`
- `data_model.md`
- `prod_req_doc.md`
- `system_design.md`
- `test_strat.md`
- `file_system.md`

**Deliverables and timeline:** `~/00_docs/02_design/deliverables.md`

**Project notes:** `~/00_docs/05_stage**/`
Place documents and files generated (design, test script, manual test script, bash, python, sql, etc) on the correct stage directory as described by the `deliverables.md`
Files written in these directories must have a prefix of `yyyymmdd_` — use today's date. if there is an existing file with a similar filename, add a prefix of `_[number]`

**Objects**
Objects are to be written on the appropriate directory as described by `file_system.md`
---


## Object Build Sequence

### Analyze the Requirements
Look up the object on `deliverables.md` and `file_system.md`. Use the brainstorm skill to analyze requirements with the user. Review the specifications of the object based on the `Project documents` and `deliverables.md`

### Write the design document
Use the brainstorm skill to write the design with the user. Place the approved design on the appropriate directory on `Project notes`. Minimize writing code on the design document. The design document should be a high-level overview of the object to be built. It should include the following:
- Object name
- Object type (model, service, schema, migration, API route)
- Object description
- Object attributes (name, type, constraints, default values) 
- Function/method signatures (name, parameters, return type, description)
- Purpose
- Variables and constants
- general logic flow
- any references to other objects and scripts

### Build the Object
Build the object based on approved design. Ensure that objects have appropriate log message for debugging. Error trapping and handling should be incorporated. Add comments to explain complex logic and decisions. Follow the project coding standards and best practices. Add comments to explain variable uses, function logic, and any non-obvious decisions.

### Test
Create and execute to verify the build with the specifications. test scripts and results placed on the appropriate directory on `Project notes`.

### Manual Test
Prepare a test script to be executed on a separate windows machine on the same network. place the script on the appropriate directory on `Project notes`.

