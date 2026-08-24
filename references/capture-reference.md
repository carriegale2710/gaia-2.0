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

## Formal plans and tasks

Capture does not define the `PLAN.md`/`TASKS.md` templates or archive logic. Formal plans follow `plan-reference.md`; task state follows `tasks-reference.md`. When a captured item warrants a formal plan, route it into a new draft plan under `.gaia/plans/draft/` per `plan-reference.md` rather than writing an ad hoc plan file here.

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

Archive and lifecycle rules for the canonical `PLAN.md`/`TASKS.md` live in `plan-reference.md` and `tasks-reference.md` — this document does not duplicate them.
