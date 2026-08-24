---
name: capture
description: Captures session state before context resets. Use when ending a sprint, switching projects, or pausing mid-session to persist decisions, progress, and unfinished requests.
---

# Capture

Synthesise the current session into six lists, confirm with the user, then write to state files. Bullets only — never prose.

## Process

### 1. Read existing memory

Read `MEMORY.md`. Create it if missing — four sections: `## Project Structure`, `## Notes`, `## Permissions`, `## Memories`.

### 2. Extract six lists

Pull each list from explicit conversation content only. 15-word cap per bullet; split rather than expand.

**Decisions made** — settled choices. Prefix `[project]` when multiple projects active.
**Pending decisions** — raised but unresolved; a future choice must be made before work proceeds. Prefix `[ ]`.
**Follow-up questions** — unknowns to research or validate. Prefix `[ ]`.
**Unfinished requests** — things the user explicitly asked for that were not completed. Prefix `[ ]`.
**Progress** — artifacts built, written, or completed.
**Blockers** — hard blockers on next steps. Prefix `[BLOCKED]`. Write "None" if none.

Write every section even if empty — use "None this session."

### 3. Confirm

Show all six lists. Ask: "Anything missing or wrong before I save?"

Complete when the user replies `yes`, `save`, or `looks good`. Any other reply → revise and re-show.

### 4. Write MEMORY.md — Sessions section

Append this dated capture block to `## Memories`:

```markdown
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

Append only — never overwrite.

### 5. Write PLAN.md and TASKS.md (only if a formal plan was discussed)

If a feature, strategy, or multi-step work was explicitly planned and approved → write both files using the templates in `plan-reference.md` and `tasks-reference.md`. Archive any existing files first according to those references.

Complete when both files exist on disk and match the discussed plan. Skip entirely if no formal plan — note "No plan this session."

### 6. Update BACKLOG.md (if informal tasks exist but no formal plan)

If tasks or to-dos were discussed without a formal plan → append them to `BACKLOG.md` under `## Backlog` using this template. Create the file if missing.

```markdown
# BACKLOG.md

## Backlog

- [ ] task — [project] — YYYY-MM-DD

## Done

- [x] completed task — [project] — YYYY-MM-DD
```

### 7. Surface all open items

Scan ALL capture blocks in `## Memories` — not just this session. Present three lists, each item labelled with its capture date:

1. **Unfinished requests** — every `[ ]` unfinished request (address these first next session)
2. **Pending decisions** — every unresolved `[ ]` decision
3. **Follow-up questions** — every unanswered `[ ]` question

Complete when all three lists are shown and every item has a date label.

## Rules

- Infer only from explicit conversation text
- Write every section even if empty
- Confirm before every file write
- 15-word cap per bullet — split rather than expand
- Unfinished requests are listed first in Step 7
