# TASKS.md

- [ ] Task 1: Add repository instructions — status: implemented, unvalidated
  - [x] `.gaia/GAIA.md` exists and defines the `dev` branch convention.
  - [x] `.gaia/GAIA.md` defines `.gaia/MEMORY.md` as append-only.
  - [x] `.gaia/GAIA.md` requires SHA-based read–modify–write and post-write verification.
  - [ ] Review the complete file against every plan requirement and record evidence.

- [ ] Task 2: Add the persist skill — status: implemented, unvalidated
  - [x] `skills/persist.md` exists and defines `/persist`.
  - [x] The skill specifies complete-read, SHA capture, append-only update, confirmation, and post-write verification.
  - [ ] Exercise the workflow with an approved harmless append-preservation test.

- [ ] Task 3: Add planning and backlog state — status: implemented, unvalidated
  - [x] `.gaia/PLAN.md`, `.gaia/TASKS.md`, and `.gaia/BACKLOG.md` exist.
  - [x] Lifecycle references exist under `references/`.
  - [ ] Validate the full lifecycle against the repository references and record evidence.

## Progress

Implementation commit: `b779b39d8bb33ad67309e02438f7b95a3ce454be`.
Reference commit: `460b3abe0ccfd6c0e08d94bd891f5df78079da85`.

Current status: implemented and partially reviewed, but not fully validated. No task should be marked `[x]` until its remaining validation criteria pass.
