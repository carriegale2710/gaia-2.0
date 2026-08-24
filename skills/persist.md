---
name: persist
description: Safely append a concise entry to the repository's durable memory file.
disable-model-invocation: true
---

# Persist

Use this skill only when the user asks to persist, append, or save project memory in the repository.

> Note: this skill is a work in progress and needs testing before use; track validation work in `.gaia/BACKLOG.md`.

## Inputs

Resolve these before any write:

- Repository owner and name.
- Branch, defaulting to `dev` for this project unless the user explicitly names another branch.
- Target path, defaulting to `.gaia/MEMORY.md`.
- The exact new memory entry.

## Safe procedure

1. Read the complete target file from the resolved branch with GitHub MCP.
2. Capture the exact current blob SHA returned for that file.
3. Confirm the target is the expected file and that its contents are complete.
4. Insert one concise entry into the appropriate existing section, preserving all other content byte-for-byte.
5. Do not reconstruct the file from summaries, excerpts, prior context, or a remembered template.
6. If the file is missing, truncated, ambiguous, or its SHA is unavailable, stop and ask the user.
7. Before writing, request confirmation showing the exact repository, branch, path, entry, and resulting complete content or a clear diff.
8. Update the file using the captured SHA and a commit message ending with the exact trailer:

   `Co-Authored-By: GAIA Code <noreply@gaiacode.pro>`

9. Re-read the same file from the same branch after the write.
10. Verify that the previous complete content remains present and the new entry appears exactly once.
11. Report the commit SHA, file SHA, branch, and verification result.

## Failure handling

- If the update reports a SHA conflict, re-read the file and start again with the new SHA.
- If post-write verification fails, do not overwrite or retry blindly; report the discrepancy and ask for direction.
- Never delete or replace existing memory to make an append easier.
