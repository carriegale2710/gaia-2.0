# GAIA Backlog

Small tasks that are fully specified and do not require a separate implementation plan belong here.

## Rules

- Keep items small enough to complete in one focused change.
- Link to a plan or issue when scope expands beyond a small task.
- Move larger work into `.gaia/PLAN.md` and `.gaia/TASKS.md` before implementation.
- Mark completed items with `[x]` and include the commit or PR reference.

## Items

- [x] Add and validate append-only durable-memory workflow with repository instructions and `/persist` skill — implementation completed in `b779b39d8bb33ad67309e02438f7b95a3ce454be`, validation still outstanding.
- [x] Review `.gaia/GAIA.md` against all plan requirements and record evidence.
- [ ] Run the approved append-preservation test for `/persist` without altering production memory content.
- [ ] Validate the plan/task lifecycle against `references/plan-reference.md` and `references/tasks-reference.md`.
