# Turn Engine

This document governs tool-call budgets, context limits, efficient repository reads, commit batching, and permission mechanics.

## Tool-call budget

The Space allows up to 15 tool calls per message. Use fewer when the work is complete. Batch independent calls and use sequential calls only when one result feeds the next.

## Context budget

Keep fetched and generated content under roughly 40,000 tokens per turn. Split work when approaching the limit and report progress at a clean boundary.

## Repository reads

Use repository tools to read only what the task requires. Resolve names and IDs before writes. Paginate with medium page sizes and continue only as far as needed.

## Commit batching

Prefer `push_files` for multi-file changes. Batch small files in groups of up to eight, medium files up to four, and large files individually. Always include the exact attribution trailer required by `prompts/SYSTEM_INSTRUCTIONS.md` in every commit message.

## Permission modes

Every external write is governed by the active permission mode recorded in sandbox `MEMORY.md`:

- **Ask Permissions:** require approval for every write.
- **Accept Edits:** routine writes may proceed without approval; merges, repository creation or forking, and default-branch writes require approval.
- **Bypass Permissions:** no approval is required.

If the mode is missing or malformed, use Ask Permissions.

## GitHub write metadata

Every write must include the tool-specific approval metadata required by the connector. For GitHub writes, use `_tool_input_summary` in the form `[{Tool Name}] {short action}` and set `_requires_user_approval` according to the active permission mode.
