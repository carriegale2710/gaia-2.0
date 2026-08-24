---
name: capture
description: >
  Capture knowledge from conversations when the user says "capture this", "extract insights", 
  "summarize for later", or asks to pull structured knowledge from a discussion.
  Distinct from persist: capture = extract from conversation; persist = save to MEMORY.md.
---

## Role

You extract structured knowledge from conversation — insights, decisions, gotchas, answers to codebase questions — and format them for persistence.

## When to use

- User says "capture this", "extract insights", "summarize for later"
- User asks "what did we learn?" or "what should we remember?"
- End of a debugging session, planning session, or research deep-dive

## Process

1. **Identify knowledge types** in the conversation:
   - Decisions made (and why)
   - Gotchas discovered
   - Answers to codebase questions
   - Patterns or conventions observed
   - Mistakes and lessons learned

2. **Format as MEMORY.md bullets** — each insight as a single `- ` line, ready to append to `## Memories`

3. **Distinguish from persist**:
   - Capture = extract and format (this skill)
   - Persist = actually write to MEMORY.md (`skills/persist.md`)

## Output

Return the captured knowledge as a bulleted list. Do not write to files — that's `persist.md`'s job.

## Boundaries

- Do not capture everything — only what earns its place (non-obvious, reusable, decision-shaping)
- Do not duplicate what's already in MEMORY.md (check first if unsure)
- Do not write — output the bullets for the user to review, then call `persist.md` if they approve
