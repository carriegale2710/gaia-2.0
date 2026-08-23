# Completion Verification Gate — Draft Plan

**Status:** Draft — saved for later implementation.

**Goal:** Prevent GAIA from marking tasks complete based only on file creation by requiring explicit acceptance criteria, validation evidence, and a verification gate.

## Proposed architecture

Keep implementation state in `.gaia/` and separate three states: created, reviewed, and validated. `.gaia/PLAN.md` defines requirements and acceptance criteria; `.gaia/TASKS.md` records execution and validation evidence; `.gaia/BACKLOG.md` tracks small tasks. Repository-wide rules belong in `.gaia/GAIA.md`. The existing `/persist` skill remains focused on safe durable-memory updates, while a future `/verify` skill audits task completion against the plan.

## Proposed changes

### 1. Add acceptance criteria to plans

Every plan task must include objective checks. Examples:

- Required files exist.
- Required rules or behavior are present.
- Relevant files have been re-read and reviewed.
- Critical workflows have an end-to-end validation result.
- Commit, file SHA, and verification evidence are recorded.

A parent task cannot be complete while any child criterion remains incomplete.

### 2. Strengthen task states

Preferred state model:

```markdown
- [ ] Not started
- [~] Implemented, not yet validated
- [x] Implemented and validated
- [!] Blocked or failed validation
```

If standard checkboxes must be retained, use explicit status text such as:

```markdown
- [ ] Add persist skill — status: implemented, unvalidated
- [x] Add persist skill — validated against append-preservation test
```

### 3. Add repository-wide completion rules

Extend `.gaia/GAIA.md` with rules that:

- Never check off a task based only on file existence.
- Treat implemented and validated as separate states.
- Require evidence for every completed task.
- Leave tasks open when validation cannot be run.
- Record validation results, commit references, and file SHAs.
- Do not mark a task complete merely because a progress update was requested.

### 4. Add a verification workflow

A future `/verify` skill should:

1. Read `.gaia/PLAN.md` and `.gaia/TASKS.md`.
2. Read every file named by the relevant task.
3. Map each acceptance criterion to concrete evidence.
4. Run or inspect the specified validation procedure.
5. Report unmet criteria without changing task state.
6. Require explicit user approval before changing a task from implemented to validated, unless repository policy later authorizes automatic status updates.

### 5. Validate the durable-memory workflow

Use a harmless, explicitly approved test entry or fixture:

1. Read the complete original memory file.
2. Capture its current SHA.
3. Append one test entry.
4. Write using the original SHA.
5. Re-read the resulting file.
6. Confirm all original content remains and the new entry appears exactly once.
7. Record the commit, file SHA, and result.

Do not use a destructive replacement or reconstruct the memory file from summaries.

## Acceptance criteria for this draft

- [ ] Draft is saved under `.gaia/plans/draft/`.
- [ ] No active implementation files are changed.
- [ ] The draft explicitly separates implementation from validation.
- [ ] The draft defines evidence required before checking tasks complete.
- [ ] The draft proposes, but does not implement, `/verify`.

## Later implementation tasks

- [ ] Update `.gaia/GAIA.md` with completion-evidence rules.
- [ ] Update `.gaia/PLAN.md` template or planning convention with acceptance criteria.
- [ ] Update `.gaia/TASKS.md` convention with implementation and validation states.
- [ ] Create and test `/verify`.
- [ ] Run the append-preservation validation for `/persist`.
- [ ] Update `.gaia/BACKLOG.md` with follow-up items.

## Non-goals

- Do not implement `/verify` in this draft.
- Do not alter `.gaia/MEMORY.md`.
- Do not change the current task status files as part of saving this draft.
