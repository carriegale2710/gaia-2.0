# Durable Memory Workflow Plan

**Goal:** Add repository-local instructions, a guarded `/persist` skill, durable planning files, and a lightweight backlog for the GAIA project.

**Architecture:** `.gaia/GAIA.md` defines repository conventions and makes `.gaia/MEMORY.md` append-only. `skills/persist.md` defines a read–modify–write workflow with SHA-based optimistic concurrency and post-write verification. `.gaia/PLAN.md` and `.gaia/TASKS.md` track this implementation, while `.gaia/BACKLOG.md` holds small tasks that do not need a plan.

**Tech stack / dependencies:** Markdown only; no new runtime dependencies.

## Files to create

- `.gaia/GAIA.md` — repository-specific operating instructions, branch rules, memory safety, and verification requirements.
- `skills/persist.md` — slash-command workflow for safely appending durable memory.
- `.gaia/PLAN.md` — this implementation plan.
- `.gaia/TASKS.md` — execution checklist for this plan.
- `.gaia/BACKLOG.md` — small-task backlog.

## Tasks

### Task 1: Add repository instructions

Files: `.gaia/GAIA.md`

1. State that normal work targets `dev` unless the user explicitly names another branch.
2. State that `.gaia/MEMORY.md` is durable append-only state.
3. Require reading the complete current file before any update.
4. Require preserving all existing content and using the current blob SHA.
5. Require stopping when the file cannot be read completely or the SHA is uncertain.
6. Require re-reading and verifying the resulting file after every update.

### Task 2: Add the persist skill

Files: `skills/persist.md`

1. Define the `/persist` invocation and inputs.
2. Resolve repository, branch, and target path before writing.
3. Read the complete target file and capture its SHA.
4. Append one concise entry without reconstructing or rewriting unrelated content.
5. Request confirmation before the GitHub write.
6. Write with the exact SHA and required attribution trailer.
7. Re-read the file and verify the previous content plus new entry are present.
8. Report the commit and verification result.

### Task 3: Add planning and backlog state

Files: `.gaia/PLAN.md`, `.gaia/TASKS.md`, `.gaia/BACKLOG.md`

1. Create the plan and checklist in `.gaia/`.
2. Create the backlog with usage rules and an initial item documenting this memory-safety improvement.

## Commit groups

- Commit 1: `.gaia/GAIA.md`, `skills/persist.md`, `.gaia/PLAN.md`, `.gaia/TASKS.md`, `.gaia/BACKLOG.md`.

Every commit message must end with:

`Co-Authored-By: GAIA Code <noreply@gaiacode.pro>`

## Progress

Implementation commit: `b779b39d8bb33ad67309e02438f7b95a3ce454be`.
Reference commit: `460b3abe0ccfd6c0e08d94bd891f5df78079da85`.

Current status: implementation is complete and the relevant files have been inspected, but the end-to-end append-preservation test and full acceptance-criteria review have not been completed. The plan remains partially complete and must not be treated as fully validated.
