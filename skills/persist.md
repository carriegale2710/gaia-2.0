---
name: persist
description: >
  Persist knowledge to MEMORY.md when the user says "save", "remember", "add to memory", 
  "keep this for later", or after capture.md has extracted insights ready to write.
  Distinct from capture: persist = write to file; capture = extract from conversation.
---

## Role

You write knowledge to `MEMORY.md` — the durable state file that survives auto-compaction.

## When to use

- User says "save", "remember", "add to memory", "keep this for later"
- User approves captured insights from `capture.md`
- You've made a meaningful edit, discovered a gotcha, or answered a codebase question worth remembering

## Process

1. **Read MEMORY.md** (create via template if missing — see `prompts/MEMORY_ENGINE.md` A.2)

2. **Choose the section**:
   - `## Project Structure` — repo layout, key files, gotchas
   - `## Notes` — user's standing instructions ("use pnpm, not npm")
   - `## Memories` — observations, decisions, lessons learned (the main destination)

3. **Append or update**:
   - Use `append_to_section()` helper from `MEMORY_ENGINE.md` A.4
   - Do not duplicate — if a bullet already exists, update it instead
   - Keep entries short and specific (one line per bullet)

4. **Compact if needed** — if `## Memories` passes ~40 bullets, consolidate in the same turn (merge duplicates, drop stale entries, keep every mistake-lesson rule)

## Output

- Confirm what was written (one sentence)
- Optionally show the new bullet(s)

## Boundaries

- Do not capture — that's `capture.md`'s job. If the user gives raw conversation, call `capture.md` first.
- Do not write to `PLAN.md` or `TASKS.md` — those are plan-engine files (`prompts/PLAN_ENGINE.md`)
- Do not export — that's a separate command ("export memory")
