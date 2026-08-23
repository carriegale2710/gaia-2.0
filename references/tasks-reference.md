# TASKS.md Reference

This reference describes how GAIA creates and manages the repository task checklist at `.gaia/TASKS.md`, following `prompts/MEMORY_ENGINE.md` Part B.

## Creation

Create `.gaia/TASKS.md` when execution of an approved `.gaia/PLAN.md` begins. Derive one checkbox per plan task, or one checkbox per step when finer-grained durability is needed. Keep the order identical to the plan.

Example:

```markdown
# TASKS.md

- [ ] Task 1: Add repository instructions
- [ ] Task 2: Add the persist skill
- [ ] Task 3: Add validation evidence
```

Do not create or check off execution tasks merely because a plan was drafted. A draft plan remains separate from active execution.

## Completion standard

A task is complete only when:

- Its implementation is finished.
- Its acceptance criteria are satisfied.
- Relevant files have been reviewed after the change.
- Required tests or validation checks pass, or an explicit reason is recorded when validation cannot run.
- The commit or write has succeeded.
- The completion evidence is recorded in the task or adjacent progress section.

File existence alone is not completion evidence.

## Update timing

Check an item off immediately after that task’s commit or push succeeds and validation is complete, before beginning the next task. Never defer all checkbox updates until the end of a turn. Never check an item whose work is partial or whose validation is failing.

## Progress across turns

At the beginning of execution, read both `.gaia/PLAN.md` and `.gaia/TASKS.md`. After compaction or context reset, read them again before continuing. Resume at the first unchecked task.

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

Say **Continue** to keep going.
***
```

## State discipline

Use standard checkboxes for active task state unless the repository adopts an explicit richer state model. If a task is implemented but not validated, leave it unchecked and record `status: implemented, unvalidated`. If validation fails, leave it unchecked and record the failure. Do not mark tasks complete based on an optimistic interpretation of the plan.

## Safe updates

Preserve unrelated checklist content during updates. Read the complete current file, capture its current SHA, perform a read–modify–write, and re-read the result. Verify that the old content remains and that each changed checkbox appears exactly as intended.
