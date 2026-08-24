# PLAN.md Reference

This reference is the single source for plan authoring, plan schema, and plan lifecycle mechanics. Task schema and execution bookkeeping are defined in `tasks-reference.md`; repository-specific workflow rules are in `.gaia/GAIA.md`; runtime triggers for _when_ GAIA reads or writes these files are in `PLAN_ENGINE.md`. Nothing in this document is restated elsewhere — if another file needs it, it points here.

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

Plans live under `.gaia/plans/` in `draft/`, `approved/`, `on-hold/`, or `closed/`, using filenames in the format `PLAN-YYYYMMDD-NNN.md`, matching the plan's `id` field. An approved plan activates only after explicit user approval and placement in `approved/`.

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

**Archive the previous plan first.** If `.gaia/PLAN.md` already exists from an earlier plan, resolve it by `id` before writing the new plan:

- If a doc matching the active plan's `id` already exists anywhere under `.gaia/plans/` (any of `draft/`, `approved/`, `on-hold/`, `closed/`), that doc is the record — just sync it: overwrite its content with the current `.gaia/PLAN.md` so it's up to date. Leave it in its existing folder; do not move it between lifecycle folders unless the caller passes an explicit target status (see "Closing a plan").
- If no matching doc exists, archive `.gaia/PLAN.md` into `.gaia/plans/draft/` by default. This is for human categorisation only — never default an uncategorised plan into `approved/`, `on-hold/`, or `closed/` without the user's explicit consent.

`TASKS.md` is temporary execution state and must not be archived with the plan. Delete it only after verifying that every task has a concrete GitHub Issue reference. If any task still has `Issue: pending` or an Issue exception, stop and preserve `TASKS.md`.

```python
import os, re, shutil
from datetime import datetime, timezone

PLAN_STATUSES = ('draft', 'approved', 'on-hold', 'closed')

def extract_plan_id(plan_content):
    match = re.search(r'^id:\s*(\S+)', plan_content, re.MULTILINE)
    if not match:
        raise RuntimeError('PLAN.md front matter is missing an `id` field')
    return match.group(1)

def find_matching_plan_doc(plan_id):
    for status in PLAN_STATUSES:
        path = f'.gaia/plans/{status}/{plan_id}.md'
        if os.path.exists(path):
            return path
    return None

def archive_active_plan(target_status=None):
    if not os.path.exists('.gaia/PLAN.md'):
        return
    if os.path.exists('.gaia/TASKS.md'):
        with open('.gaia/TASKS.md') as f:
            tasks = f.read()
        if 'Issue: pending' in tasks or 'Issue: exception' in tasks:
            raise RuntimeError('Cannot archive PLAN.md until every task has a concrete GitHub Issue reference')

    with open('.gaia/PLAN.md') as f:
        plan_content = f.read()
    plan_id = extract_plan_id(plan_content)
    existing = find_matching_plan_doc(plan_id)

    if existing:
        if target_status and f'/{target_status}/' not in existing:
            # explicit status change requested — move it there
            os.makedirs(f'.gaia/plans/{target_status}', exist_ok=True)
            os.remove(existing)
            with open(f'.gaia/plans/{target_status}/{plan_id}.md', 'w') as f:
                f.write(plan_content)
        else:
            # matching doc already lives in the right place — just sync it
            with open(existing, 'w') as f:
                f.write(plan_content)
    else:
        status = target_status or 'draft'  # never default to approved/on-hold/closed
        os.makedirs(f'.gaia/plans/{status}', exist_ok=True)
        with open(f'.gaia/plans/{status}/{plan_id}.md', 'w') as f:
            f.write(plan_content)

    os.remove('.gaia/PLAN.md')
    if os.path.exists('.gaia/TASKS.md'):
        os.remove('.gaia/TASKS.md')
```

### 2. Write the new plan

A standalone plan must include:

- YAML front matter with these required fields at minimum: `id`, `status`, `created_at`, `updated_at`, `priority`. Optionally add `blocked_by` and `github_issue_refs`.
- Use ISO 8601 UTC timestamps for dates and one of `draft`, `approved`, `on-hold`, or `closed` for `status`. Use an empty list for `blocked_by` and `github_issue_refs` when there are no entries.
- Goal — one sentence.
- Architecture — the approach and important decisions.
- Tech stack and dependencies — versions confirmed using the required live registry or release sources (see the package-version rule in `SYSTEM_PROMPT.md`).
- Files to create or modify — exact repository paths and one responsibility per file.
- An ordered task list with bite-sized task boundaries and full implementation steps per task (no pointers like "same as Task 2"; tasks may be read out of order). Task checklist creation and execution follow `tasks-reference.md`.
- Acceptance criteria and validation evidence for each task when completion claims matter.
- Commit groups that respect repository commit batching rules. A group may contain multiple tasks; every task in a group records the same commit reference, and no task in the group is complete until the group commit succeeds.
- An append-only `## Lifecycle Log`: each transition records the date, previous status, new status, and reason. `on-hold` transitions also record blocker IDs and unblocking conditions.

### Canonical example

The fields above stay the same object across a plan's whole life — only the values change as the plan moves through its lifecycle. One example, annotated for the points where later steps update it, replaces separate front-matter, task-block, and lifecycle-log examples:

```markdown
---
id: PLAN-20260824-001
status: draft # → approved → on-hold → closed as the plan transitions
created_at: 2026-08-24T00:00:00Z
updated_at: 2026-08-24T00:00:00Z
priority: medium
blocked_by: []
github_issue_refs: [] # activation fills this: [{task: 1, issue: 123}, ...]
# approved_at / closed_at are added only once the plan reaches that status
---

# <Feature> Plan

**Goal:** ...

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

**Issue:** pending # → issue: 123 once mapped during activation

## Lifecycle Log

- 2026-08-24T00:00:00Z | previous: draft | new: approved | reason: User approved implementation.
- 2026-08-25T00:00:00Z | previous: approved | new: on-hold | reason: Waiting for repository access. blockers: github-access; unblocks_when: GitHub MCP connection is available.
- 2026-08-26T00:00:00Z | previous: on-hold | new: approved | reason: GitHub MCP connection restored.
- 2026-08-27T00:00:00Z | previous: approved | new: closed | reason: All tasks and mapped Issues completed.
```

### Review before execution

After writing a new plan, review it against:

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
6. Record exactly one Issue reference per task, or an explicit documented exception, in both the plan's `github_issue_refs` and the corresponding task entry (see the canonical example above). Use `Issue: exception - <reason>; approved by <authority> on <ISO-8601 date>` for an exception.
7. Activation is complete only after `.gaia/PLAN.md` and `.gaia/TASKS.md` both reflect the approved plan and every task has its Issue mapping or documented exception. Do not begin execution before this condition is satisfied.

## Durable-state boundary during plan execution

Repository durable plan records live under `.gaia/`. Sandbox `PLAN.md` and `TASKS.md` are runtime state and are not automatically interchangeable. See `tasks-reference.md` for task-state update and preservation rules.

Check off each task's checkbox as its own write immediately after that task completes — never batch checkbox updates into one write — because a mid-turn crash or auto-compaction destroys unwritten state, which defeats the checklist's purpose as a durability record.

Completed tasks are represented by their GitHub Issues rather than archived locally: `.gaia/TASKS.md` is deleted once every task has a concrete Issue reference, not preserved as a historical record.

## Closing a plan

To close an active plan, run the archive procedure in "Starting a new plan" §1 with `target_status='closed'`. Closing is an explicit status request, not the no-match default, so a matching doc moves straight into `closed/` and an unmatched one is created there directly — neither ever lands in `draft/` first. Before archiving, confirm with the user that all tasks are complete, mapped to GitHub Issues, and those Issues are closed; stop and resolve with the user if any check fails.
