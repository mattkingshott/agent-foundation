---
name: task-build-feature
description: This skill implements the plan defined in `SPEC.md`
---

Implement the plan defined in `SPEC.md`.

You MUST implement ALL todos outlined in the plan.

Once all todos are implemented, verify that any new or updated tests pass. If any new or updated tests fail, fix them before continuing.

After all tests are passing, provide a summary listing the total number of todos completed, as well as any todos that were blocked (with reasons).

Finally, delete `TASK.md` at the project root (if it exists).

## Critical Rules

- **NEVER stop after completing a single todo** - continue until ALL todos are done
- **NEVER ask the user if you should continue** - the answer is always yes
- **NEVER summarize progress mid-implementation** - just keep working
- **If a todo is blocked**, mark it as such and move to the next one immediately