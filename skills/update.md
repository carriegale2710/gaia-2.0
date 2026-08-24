---
name: update
description: >
  Update and improve the codebase when the user says "update this", "improve this code", 
  "refactor this", or asks to clean up, simplify, or fix architectural issues.
  Distinct from doctor: update = improvement; doctor = bug fix.
---

## Role

You improve existing code — refactoring, simplifying, fixing architectural issues, removing dead code.

## When to use

- User says "update this", "improve this code", "refactor this", "clean this up"
- User asks to simplify, remove duplication, or fix architecture
- You spot a code quality issue during another task

## Process

1. **Read the code first** — never propose changes to unread code (`SYSTEM_PROMPT.md` PART IV)

2. **Apply `/ponytail` principles**:
   - No unrequested abstractions
   - No avoidable dependencies
   - Prefer deletion over addition
   - Fewest files possible
   - Shortest working diff wins (after understanding the problem)

3. **Check blast radius** — grep callers, trace flows, map affected areas

4. **Change only what's necessary** — a refactor does not need surrounding cleanup

## Output

- Show the diff (or describe changes if no diff tool available)
- Explain why the change improves the code (one sentence)

## Boundaries

- Do not fix bugs — that's `doctor.md`'s job (bug = something broken; update = something improvable)
- Do not add features — that's planning territory (`prompts/PLAN_ENGINE.md`)
- Do not touch code you haven't read
