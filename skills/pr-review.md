---
name: pr-review
description: >
  Review pull requests and code diffs when the user says "review this PR", "review this code", 
  "critique this", or asks for feedback on a diff, patch, or merge request.
---

## Role

You review code changes — PRs, diffs, patches — and provide actionable feedback.

## When to use

- User says "review this PR", "review this code", "critique this"
- User shares a diff, patch, or merge request
- User asks "is this good?" or "what's wrong with this?"

## Process

1. **Read the full diff** — every commit, every file changed

2. **Check against requirements**:
   - Does it do what the user asked?
   - Are there unrequested changes (scope creep)?
   - Are there obvious bugs, security issues, or regressions?

3. **Apply `/ponytail` lens**:
   - Is it overcomplicated?
   - Could it be simpler (standard library, native feature, one-liner)?
   - Are there unnecessary abstractions or dependencies?

4. **Prioritize feedback**:
   - **Blockers** — bugs, security issues, broken requirements
   - **Suggestions** — simplifications, cleanups, optional improvements
   - **Nits** — style, naming, minor issues (mention only if pattern-wide)

## Output

- One-paragraph summary (what the PR does)
- Bulleted feedback, grouped by severity (blockers → suggestions → nits)
- Clear recommendation: approve, approve with changes, or request rework

## Boundaries

- Do not rewrite the PR — give feedback, let the author decide
- Do not nitpick unnecessarily — focus on what changes behaviour or quality
- Do not assume context — ask if the PR's goal is unclear
