# PLAN.md Reference

This reference describes how GAIA creates and manages the repository plan at `.gaia/PLAN.md`, following `prompts/MEMORY_ENGINE.md` Part B.

## When to create a plan

Create or replace a plan for non-trivial work: a new feature, changes across more than roughly two or three files, multiple viable approaches, architectural decisions, or unclear scope. Skip planning for a single-line fix, a fully specified one-function change, or pure research and Q&A.

Explore before planning. Read the relevant repository files first; do not plan changes to code that has not been inspected.

## Archive before replacement

If an active `.gaia/PLAN.md` exists and a new plan is needed, archive it with its matching `.gaia/TASKS.md` under `.gaia/plans/` using the next free numbered pair:

```text
.gaia/plans/PLAN-001.md
.gaia/plans/TASKS-001.md
```

Increment the number until both destination names are free. Never overwrite plan history. For a draft that is not replacing the active plan, save it under `.gaia/plans/draft/` and leave the active plan and tasks unchanged.

## Required plan contents

A standalone plan should include:

- Goal — one sentence.
- Architecture — the approach and important decisions.
- Tech stack and dependencies — versions confirmed using the required live registry or release sources.
- Files to create or modify — exact repository paths and one responsibility per file.
- Ordered, bite-sized tasks.
- Concrete implementation steps for every task; no placeholders or hand-waves.
- Commit groups that respect repository commit batching rules.
- Acceptance criteria and validation evidence for each task when completion claims matter.

The plan must use exact repository names, paths, branches, and relevant URLs so it remains portable.

## Review before execution

After writing the plan, review it against:

- Completeness — no TODOs, placeholders, or incomplete steps.
- Spec alignment — every requirement maps to a task and no unrequested scope is added.
- Task decomposition — each task has a clear boundary and actionable steps.
- Buildability — another agent can follow it without guessing.
- Consistency — names and paths remain consistent across tasks.

Fix serious gaps before presenting the plan. Wait for explicit user approval before executing it.

## Execution lifecycle

When the user approves execution:

1. Read `.gaia/PLAN.md` and `.gaia/TASKS.md` first.
2. Derive or refresh `.gaia/TASKS.md` from the approved plan.
3. Execute only the first unchecked task that fits the turn budget.
4. Commit the completed task using the required attribution trailer.
5. Mark that task complete immediately after its commit succeeds, before starting the next task.
6. Record validation evidence rather than treating file existence as proof.
7. Continue from the first unchecked task on the next turn.

## Durable-state safety

Use read–modify–write for plan and task updates. Preserve unrelated content. Re-read the updated files after every write. If the complete current content or SHA is unavailable, stop rather than reconstructing the file.
