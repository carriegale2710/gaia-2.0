---
name: capture-reference
description: Templates and code for the /capture skill. Loaded on demand — do not invoke directly.
disable-model-invocation: true
---

# Capture Reference

## Sessions capture block template

Append to `## Memories` in MEMORY.md:

```
### Session — YYYY-MM-DD HH:MM

**Decisions made**
- [project] decision text

**Pending decisions**
- [ ] unresolved choice

**Follow-up questions**
- [ ] question to answer

**Unfinished requests**
- [ ] explicit request not completed

**Progress**
- artifact or milestone

**Blockers**
- [BLOCKED] blocker / None
```

## PLAN.md universal template

Works for any plan type: coding feature, design sprint, marketing strategy, research plan.

```markdown
# [Goal Title] — YYYY-MM-DD

**Type:** Code | Design | Research | Marketing | Other
**Goal:** one sentence
**Status:** Draft | Approved | In Progress | Complete

## Context

1–2 sentences on why this plan exists and what success looks like.

## Approach

2–3 sentences on the chosen direction and key decisions made.

## Steps

Ordered, bite-sized. Each step is independently completable and verifiable.

- [ ] Step 1
- [ ] Step 2

## Optional fields (fill when relevant)

**Tech stack:** key technologies with versions (Code plans)
**Constraints:** budget, timeline, scope limits (any plan type)
**Open questions:** things to resolve before or during execution
```

## TASKS.md template

Derived from an approved PLAN.md. One checkbox per step.

```markdown
# TASKS.md — [Goal Title]

## In Progress

- [ ] task being worked on

## Backlog

- [ ] task not yet started — [project]

## Done

- [x] completed task
```

## BACKLOG.md template

For informal tasks captured without a formal plan.

```markdown
# BACKLOG.md

## Backlog

- [ ] task — [project] — YYYY-MM-DD

## Done

- [x] completed task — [project] — YYYY-MM-DD
```

Append new tasks under `## Backlog`. Move to `## Done` when complete.

## Archive existing PLAN.md before writing a new one

```python
import os, shutil
if os.path.exists('PLAN.md'):
    if os.path.exists('TASKS.md'):
        with open('TASKS.md') as f:
            tasks = f.read()
        if 'Issue: pending' in tasks or 'Issue: exception' in tasks:
            raise RuntimeError('Cannot archive PLAN.md until every task has a concrete GitHub Issue reference')
    os.makedirs('plans/closed', exist_ok=True)
    n = 1
    while os.path.exists(f'plans/closed/PLAN-{n:03d}.md'): n += 1
    shutil.move('PLAN.md', f'plans/closed/PLAN-{n:03d}.md')
    if os.path.exists('TASKS.md'):
        os.remove('TASKS.md')
```
