# MEMORY.md

## Project Structure

- Project: `carriegale2710/gaia` — GAIA Space configuration repository.
- Current work: GAIA architecture design; target product repository not yet provided.
- GAIA runtime state: sandbox `MEMORY.md`, `PLAN.md`, `TASKS.md`.
- Durable project state: repository `/.gaia/` on `dev`.
- Skills live in Space files and are callable as slash commands.

## Notes

### User preferences

- Solo founder, Melbourne, Australia; ADHD; short sprints; multiple projects.
- Prefer dense, concise bullets; avoid verbose prose.
- Load these skills on each new session /caveman, /ponytail.
- Stack for SaaS projects: Next.js, TypeScript, Tailwind, Supabase, Vercel, Stripe/Lemon Squeezy
- GAIA handles specification, tickets, and pull-request review.
- Remind user to export memory before ending sessions.
- New threads begin by importing attached memory.

### Branch policy

- GAIA always starts work from `dev`.
- If `dev` is missing, create it automatically from the repository default branch.
- Feature, fix, and chore branches branch from `dev`, never directly from `main`.
- Pull requests target `dev` by default.
- `main` receives release-ready changes from `dev`.
- Never work directly on `main`.

### Repository agent files

- Keep `/.gaia/`, `/.agents/`, root `AGENTS.md`, and root `CLAUDE.md` on `dev` and descendants, not `main`.
- Keep `/.gaia/` separate from `/.agents/`.
- Root `AGENTS.md` is canonical general agent guidance.
- Root `CLAUDE.md` is a compatibility shim pointing to `AGENTS.md`.
- `/.gaia/GAIA.md` contains GAIA-specific operational rules.
- `/.gaia/plans/README.md` provides navigation and format guidance only.

### Durable state

- When connected to remote Github repository, check for `./.gaia/` folder and existing durable files. If folder or files do not exist yet, create them inside the connected Github repository (confirm with user first).
- Durable files: `/.gaia/MEMORY.md`, `/.gaia/PLAN.md`, and `/.gaia/TASKS.md`.
- Plan lifecycle rules belong in `/.gaia/GAIA.md`, the canonical authority.
- Sandbox files (`MEMORY.md`, `PLAN.md`, `TASKS.md`) are active runtime state; synchronize them with repository state when first connected via Github Connector.
- Sandbox file updates, including `MEMORY.md`, must always append to the latest existing file; never overwrite prior content unless explicitly importing memory.
- When `/handoff` skill is called, ask user if they want the latest sandbox files appended to the repository durable files before executing the skill.

### Plan lifecycle

- One general template: `/.gaia/templates/PLAN.md`.
- Do not create separate templates for plan states.
- Plan folders: `draft/`, `approved/`, `on-hold/`, and `closed/`.
- Draft plans need review or approval; they create no Issues, branches, or product-code changes.
- Approved plans may coexist; GAIA executes only one plan at a time.
- Approved executable plans belong in `approved/`.
- Blocked approved plans belong in `on-hold/`.
- Active plan uses `/.gaia/PLAN.md`; active tasks use `/.gaia/TASKS.md`.
- Closed plans belong in `/.gaia/plans/closed/`.
- Closed plan filenames use `PLAN-YYYYMMDD-NNN.md`.
- Do not archive TASKS files; GitHub Issues provide durable task records.
- Plans use consistent YAML metadata: `id`, `status`, dates, `priority`, `blocked_by`, `github_issue_refs`.
- Plans include an append-only `## Lifecycle Log`.
- Every transition records date, previous status, new status, and reason.
- `on-hold` requires blocker IDs and explanation.
- Draft plans contain context, options, and open questions.
- Approved plans are implementation-ready.
- On-hold plans record blockers and unblocking conditions.
- Active plans record Issues and validation.
- Closed plans record outcome, Issue mapping, and decisions.

### Plan execution and GitHub Issues

- Approved plan tasks create GitHub Issues when the plan activates.
- `TASKS.md` is a temporary execution mirror.
- Before closing a plan, every task needs an Issue reference.
- Issue completion state must match task completion state.
- Record Issue mappings in the closed plan.
- Inspect existing repository labels first.
- Reuse clearly established labels.
- If labels are absent or unclear, create minimal labels: `plan`, `task`, `bug`, `research`, `design`, `marketing`.

### Stale plans and informal work

- Detect stale plans, summarize them, and ask the user before acting.
- After confirmation, distill decisions to memory.
- Follow the user’s explicit choice for retaining, archiving, or discarding full plans.
- Permanent deletion is never the default.
- Keep `BACKLOG.md` as a lightweight file for informal tasks not ready for formal plans.
- BACKLOG.md is separate from formal plan execution and GitHub Issue records.

### Superseded decisions

- `PLAN-001.md` naming is superseded by `PLAN-YYYYMMDD-NNN.md`.
- Archiving `TASKS-*.md` is superseded; archive plans only.
- Sandbox-only state is superseded by sandbox runtime state plus durable `/.gaia/` state.
- A single generic plans archive is superseded by lifecycle folders.
- Separate templates per lifecycle state are superseded by one shared template.
- Plan rules in `plans/README.md` as authority are superseded by `/.gaia/GAIA.md`.
- Asking before creating `dev` is superseded by automatic creation.
- Read-only, Issues-as-source, and bidirectional alternatives are superseded by approved Tasks-to-Issues workflow.

## Permissions

Mode: Accept Edits

## Memories

- 2026-08-23: Finalized branch model, durable state layout, agent-file ownership, plan lifecycle, archive policy, stale-plan handling, and Tasks-to-Issues workflow.
- 2026-08-23: Memory updates must append to the latest existing file; never overwrite prior content unless explicitly importing memory.
- 2026-08-23: Earlier memory writes overwrote prior content; consolidated artefacts restore known decisions and mark superseded choices.
- 2026-08-21: GAIA + Perplexity Pro is primary workflow; Claude.ai and Github Copilot targeted terminal support.
