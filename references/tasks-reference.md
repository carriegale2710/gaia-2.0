# TASKS.md Reference

This reference is the single source for task creation and execution. It defines how GAIA creates and manages the repository task checklist at `.gaia/TASKS.md`, following `prompts/PLAN_ENGINE.md`. Plan authoring and lifecycle rules are defined in `plan-reference.md`.

## Creation

Create `.gaia/TASKS.md` before GitHub Issue creation: only after the plan is approved and materialized as `.gaia/PLAN.md`. Derive one checkbox per plan task, or one checkbox per step when finer-grained durability is needed. Keep the order identical to the plan. Set `Issue: pending` for each task until Issue matching or creation completes; then replace it with exactly one Issue reference or an explicit documented exception. Use `Issue: exception - <reason>; approved by <authority> on <ISO-8601 date>` for an exception. Activation is complete only when `.gaia/PLAN.md` and `.gaia/TASKS.md` both reflect the approved plan and every task has its Issue mapping or documented exception; do not begin execution before activation is complete.

Example:

```python
tasks = """# TASKS.md

# TASKS.md

- [ ] Task 1: Add repository instructions
  - Issue: pending
  - status: planned
  - validation: pending
- [ ] Task 2: Add the persist skill
  - Issue: pending
  - status: planned
  - validation: pending
- [ ] Task 3: Add validation evidence
  - Issue: pending
  - status: planned
  - validation: pending

"""
with open('TASKS.md', 'w') as f:
    f.write(tasks)
```

After one GitHub Issue has been matched or created for every task, replace each `Issue: pending` value with its Issue reference:

```markdown
# TASKS.md

- [ ] Task 1: Add repository instructions
  - Issue: #123
  - status: ready
  - validation: pending
- [ ] Task 2: Add the persist skill
  - Issue: #124
  - status: ready
  - validation: pending
- [ ] Task 3: Add validation evidence
  - Issue: #125
  - status: ready
  - validation: pending
```

Do not create or check off execution tasks merely because a plan was drafted. A draft plan remains separate from active execution.

## Execution

At the beginning of execution, read both `.gaia/PLAN.md` and `.gaia/TASKS.md`. Verify that activation is complete and that the first unchecked task has an Issue reference or documented exception before changing repository files. After compaction or context reset, read them again before continuing. Resume at the first unchecked task.

Execute only the first unchecked task that fits the turn budget. Complete its implementation and validation before beginning another task. If the plan assigns the task to a commit group, wait for that group's commit to succeed; multiple tasks may share one commit reference. Record the shared commit reference for each task in the group. Use the plan's commit groups and append this exact trailer to every authored commit message:

```text
Co-Authored-By: GAIA Code <noreply@gaiacode.pro>
```

## Completion standard

A task is complete only when:

- Its implementation is finished.
- Its acceptance criteria are satisfied.
- Relevant files have been reviewed after the change.
- Required tests or validation checks pass, or an explicit reason is recorded when validation cannot run.
- The commit or write has succeeded.
- Completion evidence is recorded in the task entry or adjacent progress section using `status`, `validation`, `commit`, and `issue` fields.

File existence alone is not completion evidence.

When all tasks are complete, check every item and record the validation result, commit, and closed Issue:

```markdown
# TASKS.md

- [x] Task 1: Add repository instructions
  - Issue: #123 (closed)
  - status: complete
  - validation: `markdownlint .gaia/GAIA.md` passed
  - commit: `abc1234`
- [x] Task 2: Add the persist skill
  - Issue: #124 (closed)
  - status: complete
  - validation: `python -m pytest tests/persist` passed
  - commit: `def5678`
- [x] Task 3: Add validation evidence
  - Issue: #125 (closed)
  - status: complete
  - validation: `git diff --check` passed
  - commit: `ghi9012`
```

## Update timing

Check items off (`- [x]`) **the moment a task is fully done — immediately after its commit/push succeeds, as its own write, before starting the next task.** Never batch all the checkboxes into one write: that is precisely the state a mid-turn crash or auto-compaction destroys, which defeats the file's purpose. Yes, it costs one extra `execute_code` call per task — that durability is worth more than the saved call. Never check an item whose work is partial or whose checks are failing.

## Progress across turns

At a turn boundary, report:

```text
***
**PROGRESS REPORT**

**Completed:**
- [Specific completed tasks and evidence]

**Current State:**
- [Current plan and task state]

**Remaining:**
- [Unchecked tasks]

**NEXT STEP:** [Immediate next action]

***
```

## State discipline

Use standard checkboxes for active task state. Task `status` must be one of `planned`, `ready`, `in_progress`, `implemented, unvalidated`, or `complete`, with transitions in that order except that `in_progress` may return to `ready` when work is paused. Set `validation` to `pending`, `passed: <evidence>`, or `failed: <reason>`. Set `Issue` to `pending`, `#<number>`, or the exact documented exception format above. If validation fails, leave the task unchecked and record the failure. Do not mark tasks complete until implementation, validation, and the applicable commit or commit group have succeeded.

## Safe updates

Preserve unrelated checklist content during updates. Read the complete current file, capture its current SHA, perform a read–modify–write, and re-read the result. Verify that the old content remains and that each changed checkbox appears exactly as intended.
