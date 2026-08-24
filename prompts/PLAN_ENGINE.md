# Plan Engine

Use this document for non-trivial work: new features, changes spanning more than two or three files, architectural decisions, or unclear scope. Skip planning for a one-line fix, a fully specified single-function change, or pure research.

## Planning workflow

1. Explore first; read every relevant file before planning changes.
2. If `PLAN.md` already exists, archive it with its matching `TASKS.md` under `plans/` using the next free number.
3. Write a standalone `PLAN.md` with the goal, architecture, confirmed dependencies, exact files, actionable ordered tasks, and commit groups.
4. Review for completeness, scope alignment, buildability, and consistency.
5. Present the plan and wait for explicit user approval before execution.
6. At execution start, derive `TASKS.md`; check each task immediately after its commit succeeds.
7. Record meaningful completed work in `MEMORY.md`.

## Plan references

Use `references/plan-reference.md` for detailed plan templates and archive rules. Use `references/tasks-reference.md` for task tracking details.
