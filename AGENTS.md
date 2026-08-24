# AGENTS.md — Context Pointers for gaia-2.0

This file tells the agent which documents to reach and when. Each pointer names a document and lists the **branches** (distinct cases) that should trigger reaching it. Read `/writing-for-agents` for the full discipline.

---

## Memory operations

**When the user says** "save", "remember", "add to memory", "keep this for later", or asks to capture knowledge from a conversation → read `skills/persist.md`

**When the user says** "update this", "improve this code", or asks to refactor or clean up existing code → read `skills/update.md`

---

## Debugging

**When the user says** "debug", "fix this bug", "why is this broken", "troubleshoot", or reports an error → read `skills/doctor.md`

---

## Code review

**When the user says** "review this PR", "review this code", "critique this", or asks for feedback on a diff → read `skills/pr-review.md`

---

## Knowledge capture

**When the user says** "capture this", "extract insights", "summarize for later", or asks to pull knowledge from a conversation → read `skills/capture.md`

---

## Engine docs (reference only)

These are not skills — they are mechanics docs the agent reaches when a skill or task requires them:

- `prompts/MEMORY_ENGINE.md` — how MEMORY.md works, when to read/write
- `prompts/TURN_ENGINE.md` — turn budgeting, tool-call limits, commit batching
- `prompts/PLAN_ENGINE.md` — when and how to write plans

---

## Core identity

Always loaded (do not reach via pointer):

- `prompts/SYSTEM_PROMPT.md` — agent character, tone, mandatory rules
- `.gaia/GAIA.md` — project-specific identity and conventions
