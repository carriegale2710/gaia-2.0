# PLAN.md Reference

This reference is the single source for plan authoring and lifecycle mechanics. Task creation and execution are defined in `tasks-reference.md`; repository-specific workflow rules are in `.gaia/GAIA.md` and runtime rules are in `PLAN_ENGINE.md`.

## When to create a plan

Create a new plan for non-trivial work: a new feature, changes across more than roughly two or three files, multiple viable approaches, architectural decisions, or unclear scope. Skip planning for a single-line fix, a fully specified one-function change, or pure research and Q&A. Never overwrite an existing plan; archive or transition it according to its lifecycle status.

Explore before planning. Read the relevant repository files first; do not plan changes to code that has not been inspected.

## Plan lifecycle

Plans use these statuses:

- `draft` — context, options, and open questions; not executable.
- `approved` — implementation-ready and eligible for activation.
- `on-hold` — approved but blocked; record blocker IDs and unblocking conditions.
- `closed` — completed or deliberately ended, with outcome and task mapping.

`active` is an execution condition, not a status. The active approved plan is `.gaia/PLAN.md`; active execution state is `.gaia/TASKS.md`.

Plans live under `.gaia/plans/` in `draft/`, `approved/`, `on-hold/`, or `closed/`, using filenames in the format `PLAN-YYYYMMDD-NNN.md`. An approved plan activates only after explicit user approval and placement in `approved/`.

Example lifecycle locations for `PLAN-20260824-001.md`:

```text
.gaia/plans/draft/PLAN-20260824-001.md       # being prepared
.gaia/plans/approved/PLAN-20260824-001.md    # approved, not active
.gaia/PLAN.md                                # active execution copy
.gaia/plans/on-hold/PLAN-20260824-001.md     # approved but blocked
.gaia/plans/closed/PLAN-20260824-001.md      # completed or ended
```

Only the active plan is copied to `.gaia/PLAN.md`; do not create a second lifecycle record with an `active` status.

## Starting a new plan

### 1. Archive the old plan

**Archive the previous plan first.** If `PLAN.md` already exists from an earlier plan, move it into the appropriate lifecycle folder under `plans/` before writing the new plan. `TASKS.md` is temporary execution state and must not be archived with the plan. Delete it only after verifying that every task has a concrete GitHub Issue reference. If any task still has `Issue: pending` or an Issue exception, stop and preserve `TASKS.md`.

```python
import os, shutil
from datetime import datetime, timezone

def archive_active_plan():
    if not os.path.exists('.gaia/PLAN.md'):
        return
    if os.path.exists('.gaia/TASKS.md'):
        with open('.gaia/TASKS.md') as f:
            tasks = f.read()
        if 'Issue: pending' in tasks or 'Issue: exception' in tasks:
            raise RuntimeError('Cannot archive PLAN.md until every task has a concrete GitHub Issue reference')
    os.makedirs('.gaia/plans/closed', exist_ok=True)
    date = datetime.now(timezone.utc).strftime('%Y%m%d')
    n = 1
    while os.path.exists(f'.gaia/plans/closed/PLAN-{date}-{n:03d}.md'):
        n += 1
    shutil.move('.gaia/PLAN.md', f'.gaia/plans/closed/PLAN-{date}-{n:03d}.md')
    if os.path.exists('.gaia/TASKS.md'):
        os.remove('.gaia/TASKS.md')
```

### 2. Write the new plan

````python
plan = """# <Feature> Plan

**Goal:** ...
... full plan text ...
"""
with open('PLAN.md', 'w') as f:
    f.write(plan)

## Required plan contents

A standalone plan should include:

- YAML front matter with these required fields at minimum: `id`, `status`, `created_at`, `updated_at`, `priority`. Optionally add: `blocked_by`, and `github_issue_refs`.
- Use ISO 8601 UTC timestamps for dates and one of `draft`, `approved`, `on-hold`, or `closed` for `status`. Use an empty list for `blocked_by` and `github_issue_refs` when there are no entries.
- Example front matter:

  ```yaml
  ---
  id: PLAN-20260824-001
  status: draft
  created_at: 2026-08-24T00:00:00Z
  updated_at: 2026-08-24T00:00:00Z
  priority: medium
  blocked_by: []
  github_issue_refs: []
  ---
````

- Goal — one sentence.
- Architecture — the approach and important decisions.
- Tech stack and dependencies — versions confirmed using the required live registry or release sources (see the package-version rule in `SYSTEM_PROMPT.md`).
- Files to create or modify — exact repository paths and one responsibility per file.
- An ordered task list with bite-sized task boundaries.
- Concrete implementation steps for every task; no placeholders or hand-waves. Write each task's steps in full instead of pointing at another task (e.g. "same as Task 2"), since tasks may be read out of order. Task checklist creation and execution follow `tasks-reference.md`.
- Acceptance criteria and validation evidence for each task when completion claims matter.

Example task block:

```markdown
### Task 1: Add repository instructions

**Files:**

- Create: `.agents/repository.md`
- Modify: `AGENTS.md`
- Test: `tests/repository-instructions.test.md`

**Interfaces:**

- Consumes: existing repository conventions in `AGENTS.md`
- Produces: documented instructions for future agents

- [ ] **Step 1: Write the failing test**
- [ ] **Step 2: Add the instructions**
- [ ] **Step 3: Run the validation command**

**Acceptance:** The instructions file exists, the test passes, and the validation command is recorded in `.gaia/TASKS.md`.

**Issue:** pending
```

- Commit groups that respect repository commit batching rules.
- An append-only `## Lifecycle Log`; each transition records the date, previous status, new status, and reason. `on-hold` transitions also record blocker IDs and unblocking conditions. Use entries such as: `- 2026-08-24T00:00:00Z | previous: draft | new: approved | reason: User approved implementation.`
- Commit groups may contain multiple tasks. Every task in a group records the same commit reference, and no task in the group is complete until the group commit succeeds.

Example lifecycle entries:

```markdown
## Lifecycle Log

- 2026-08-24T00:00:00Z | previous: draft | new: approved | reason: User approved implementation.
- 2026-08-25T00:00:00Z | previous: approved | new: on-hold | reason: Waiting for repository access. blockers: github-access; unblocks_when: GitHub MCP connection is available.
- 2026-08-26T00:00:00Z | previous: on-hold | new: approved | reason: GitHub MCP connection restored.
- 2026-08-27T00:00:00Z | previous: approved | new: closed | reason: All tasks and mapped Issues completed.
```

### Review before execution

After writing the a new plan, review it against:

- Completeness — no TODOs, placeholders, or incomplete steps.
- Spec alignment — every requirement maps to a task and no unrequested scope is added.
- Task decomposition — each task has a clear boundary and actionable steps.
- Buildability — another agent can follow it without guessing.
- Consistency — names and paths remain consistent across tasks.

Fix serious gaps — missing requirements, contradictory steps, placeholder content, or tasks too vague to act on — before presenting the plan. Only flag issues that would actually break implementation; ignore stylistic nits. Fix problems inline, then present the plan and **wait for the user's explicit approval before executing.**

## Activating an approved plan

When the user explicitly approves a plan for execution:

1. Place the approved plan in `.gaia/plans/approved/`.
2. Materialize the approved plan as `.gaia/PLAN.md`.
3. Follow `tasks-reference.md` to derive `.gaia/TASKS.md` with one task entry per plan task and `Issue: pending` for each unmapped task.
4. Inspect existing GitHub Issues and match each task to one equivalent open Issue where possible.
5. Create one Issue for each task without an equivalent Issue, reusing established labels or only the minimal allowed labels: `plan`, `task`, `bug`, `research`, `design`, and `marketing`.
6. Record exactly one Issue reference per task, or an explicit documented exception, in both the plan and corresponding task entry. Use `Issue: exception - <reason>; approved by <authority> on <ISO-8601 date>` for an exception.
7. Activation is complete only after `.gaia/PLAN.md` and `.gaia/TASKS.md` both reflect the approved plan and every task has its Issue mapping or documented exception. Do not begin execution before this condition is satisfied.

Example plan-level Issue mapping after activation:

```yaml
github_issue_refs:
  - task: 1
    issue: 123
  - task: 2
    issue: 124
  - task: 3
    issue: 125
```

## Durable-state boundary during plan execution

Repository durable plan records live under `.gaia/`. Sandbox `PLAN.md` and `TASKS.md` are runtime state and are not automatically interchangeable. See `tasks-reference.md` for task-state update and preservation rules.

Check off each task's checkbox as its own write immediately after that task completes — never batch checkbox updates into one write — because a mid-turn crash or auto-compaction destroys unwritten state, which defeats the checklist's purpose as a durability record. The turn-boundary progress report required by `tasks-reference.md` supplements these per-task writes; it does not replace them.

Completed tasks are represented by their GitHub Issues rather than archived locally: `.gaia/TASKS.md` is deleted once every task has a concrete Issue reference, not preserved as a historical record.

## Closing an plan

1. Check if a plan is active or inactive. An active plan will have been loaded into `./.gaia/PLAN.md` from a matching file in the `./.gaia/plans/approved` folder. If active, complete all steps below. If plan is inactive (plan not loaded at `./.gaia/PLAN.md`), skip to step 4.

2. **Plan is active**: Perform these 3 checks:

- All tasks have been completed in `TASKS.md`.
- All tasks have been mapped to github issues.
- All corresponding Github Issues have been closed. (Check with user)

If any checks fail, stop and ask the user resolve these. Only proceed if the user gives explicit consent to close the plan despite failed checks.

3. Once user confirms to close the plan, summarise any progress reports or commit history from `./gaia/TASKS.md` and append this to the bottom of `/.gaia/PLAN.md`. Optionally add a tiny completion note next to any task block based on task history. Once done, clear the the content from `/.gaia/TASKS.md`.

4. Update the closed plan's metadata with the final dates and outcome, for example:

```yaml
---
id: PLAN-20260824-001
status: closed
created_at: 2026-08-24T00:00:00Z
updated_at: 2026-08-27T00:00:00Z
approved_at: 2026-08-24T00:00:00Z
closed_at: 2026-08-27T00:00:00Z
priority: medium
blocked_by: []
github_issue_refs: [123, 124, 125]
---
```

The plan must use exact repository names, paths, branches, and relevant URLs so it remains portable. If the plan was closed with incomplete tasks, record the reason why.

5. Move the now edited closed plan file to `.gaia/plans/closed/` and, if necessary, rename it using Lifecycle naming conventions.
