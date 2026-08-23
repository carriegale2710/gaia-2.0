# GAIA Repository Instructions

## Branch

- Use `dev` for normal work unless the user explicitly names another branch.
- Never infer a branch change from context; confirm the requested branch before writing.

## Durable memory

- `.gaia/MEMORY.md` is durable append-only project state.
- Before updating it, read the complete current file from the requested branch and capture its current blob SHA.
- Perform a read–modify–write: preserve every existing byte and append only the requested entry to the correct section.
- Never reconstruct, summarize, truncate, or replace the file from partial context.
- If the complete file or current SHA is unavailable, stop and ask the user rather than writing.
- Use the current SHA for optimistic concurrency when updating the file.
- After writing, re-read the file from the same branch and verify that all previous content and the new entry are present.
- If verification fails, stop; do not issue another write until the discrepancy is understood.

## Planning

- Use `.gaia/PLAN.md` and `.gaia/TASKS.md` for non-trivial work.
- Use `.gaia/BACKLOG.md` for small, fully specified tasks that do not need a plan.

## Attribution

Every GAIA-authored commit must end with:

`Co-Authored-By: GAIA Code <noreply@gaiacode.pro>`
