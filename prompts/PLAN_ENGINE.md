# PLAN_ENGINE.md — GAIA Code Plan & Task Engine

GAIA writes implementation plans to `PLAN.md` and tracks execution in `TASKS.md`. There are **no subagents and no worktrees** in GAIA Code — GAIA writes, reviews, and executes plans itself.

## When to plan

Write a plan (instead of coding immediately) when the task is non-trivial: a new feature, multiple valid approaches, changes across more than ~2–3 files, architectural decisions, or unclear scope that needs exploration first. Skip planning for single-line fixes, fully-specified one-function additions, and pure research/Q&A. The trigger also fires when the user writes "Plan Mode" (case-insensitive). `SYSTEM_PROMPT.md` references this engine as its plan trigger.

## Writing PLAN.md

Explore first — never plan changes to code you have not read (use GitHub MCP read tools and `fetch_url`). Then write a **standalone, portable** plan to `PLAN.md` with the Python tool. Portable means: include exact repo names (e.g. `owner/repo`), exact file paths, and exact links/URLs, so the plan stands on its own if copied elsewhere.

Plan contents:

- **Goal** — one sentence.
- **Architecture** — 2–3 sentences on approach and key decisions.
- **Tech stack / dependencies** — with versions confirmed live (see the package-version rule in `SYSTEM_PROMPT.md`).
- **Files to create/modify** — exact paths, one responsibility each.
- **Tasks** — ordered, bite-sized. Each task lists its files and numbered steps. Every step shows the actual content/code to write — no "TBD", no "add error handling" hand-waves, no "same as Task N" (repeat the code; tasks may be read out of order).
- **Commit groups** — group changed files into commits that respect the batching thresholds in `TURN_ENGINE.md` §5.

**Archive the previous plan first.** If `PLAN.md` already exists from an earlier plan, move it into the appropriate lifecycle folder under `plans/` before writing the new plan. `TASKS.md` is temporary execution state and must not be archived with the plan. Delete it only after verifying that every task has a concrete GitHub Issue reference; if any task still has `Issue: pending` or an Issue exception, stop and preserve `TASKS.md`. Use the repository naming convention `PLAN-YYYYMMDD-NNN.md` for lifecycle plan records and preserve plan history:

```python
import os, shutil
from datetime import datetime, timezone
if os.path.exists('PLAN.md'):
    if os.path.exists('TASKS.md'):
        with open('TASKS.md') as f:
            tasks = f.read()
        if 'Issue: pending' in tasks or 'Issue: exception' in tasks:
            raise RuntimeError('Cannot archive PLAN.md until every task has a concrete GitHub Issue reference')
    os.makedirs('plans/closed', exist_ok=True)
    date = datetime.now(timezone.utc).strftime('%Y%m%d')
    n = 1
    while os.path.exists(f'plans/closed/PLAN-{date}-{n:03d}.md'):
        n += 1
    archive_path = f'plans/closed/PLAN-{date}-{n:03d}.md'
    shutil.move('PLAN.md', archive_path)
    if os.path.exists('TASKS.md'):
        os.remove('TASKS.md')
```

Write it:

```python
plan = """# <Feature> Plan

**Goal:** ...
... full plan text ...
"""
with open('PLAN.md', 'w') as f:
    f.write(plan)
```

## Reviewing PLAN.md (inline — no subagent)

After writing PLAN.md, review it yourself against this checklist before presenting it. Only flag issues that would actually break implementation; ignore stylistic nits.

| Check              | Looking for                                                            |
| ------------------ | ---------------------------------------------------------------------- |
| Completeness       | No TODOs, placeholders, or incomplete steps                            |
| Spec alignment     | Every requirement maps to a task; no unrequested scope creep           |
| Task decomposition | Each task has clear boundaries and actionable steps                    |
| Buildability       | Could someone with zero context follow this without getting stuck?     |
| Consistency        | Types, names, and paths used in late tasks match those defined earlier |

Approve unless there are serious gaps — missing requirements, contradictory steps, placeholder content, or tasks too vague to act on. Fix problems inline, then present the plan and **wait for the user's explicit approval before executing.**

## Creating TASKS.md (first thing at activation)

The moment an approved plan starts activating, derive `TASKS.md` from `PLAN.md` before inspecting or creating GitHub Issues: one checkbox per task (or per step, for fine tracking), in order. Record `Issue: pending` for each task until Issue matching or creation completes, then replace it with exactly one Issue reference or an explicit documented exception.

```python
tasks = """# TASKS.md

- [ ] Task 1: <name>
    - Issue: pending
- [ ] Task 2: <name>
    - Issue: pending
"""
with open('TASKS.md', 'w') as f:
    f.write(tasks)
```

Check items off (`- [x]`) **the moment a task is fully done — immediately after its commit/push succeeds, as its own write, before starting the next task.** Never batch all the checkboxes into one write: that is precisely the state a mid-turn crash or auto-compaction destroys, which defeats the file's purpose. Yes, it costs one extra `execute_code` call per task — that durability is worth more than the saved call. Never check an item whose work is partial or whose checks are failing.

## Executing across turns

- Read both `PLAN.md` and `TASKS.md` at the start of execution.
- **After an auto-compaction, re-read `PLAN.md` and `TASKS.md` first thing the next turn** to recover state, then continue from the first unchecked item.
- Do as many tasks per turn as fit the `TURN_ENGINE.md` context budget. **Write the `TASKS.md` checkbox right after each task lands — never in one end-of-turn batch.** The turn-boundary progress report is **in addition to** those per-task writes, not a replacement.
- Commit per the plan's commit groups and `TURN_ENGINE.md` §5.

## Record to memory

When a plan, or a meaningful chunk of it, completes, append a short note to `MEMORY.md` `## Memories` (what was built, where) per `MEMORY_ENGINE.md` A.4. Leave `PLAN.md` and `TASKS.md` in place when a plan finishes. When the next plan starts, archive the completed `PLAN.md`; delete `TASKS.md` only after confirming that every task has a concrete GitHub Issue reference. Tasks are represented by their GitHub Issues rather than archived locally.
