# GAIA Code — Repository Instructions

This file holds only repository-specific conventions. General GAIA behaviour (permissions, skills, built-ins, commit/PR attribution rules) lives in the core engine files and should not be duplicated here.

## Project Structure

- Project: `carriegale2710/gaia-2.0` — GAIA Space configuration repository.
- Current work: GAIA architecture design; target product repository not yet provided.
- GAIA runtime state: sandbox `MEMORY.md`, `PLAN.md`, `TASKS.md`.
- Durable project state: repository `/.gaia/` on `dev`.
- Skills live in Space files and are callable as slash commands.

## Branch model

- GAIA always starts work from `dev`.
- If `dev` is missing, create it automatically from the repository default branch.
- Feature, fix, and chore branches must branch from `dev`.
- Never work directly on `main`.
- Pull requests target `dev` by default.
- `main` receives release-ready changes from `dev`.

## Repository agent files

- Keep `/.gaia/`, `/.agents/`, root `AGENTS.md`, and root `CLAUDE.md` on `dev` and its descendants, not `main`.
- Keep `/.gaia/` separate from `/.agents/`.
- Root `AGENTS.md` is the canonical general agent guidance.
- Root `CLAUDE.md` is a compatibility shim pointing to `AGENTS.md`.
- This file (`/.gaia/GAIA.md`) contains GAIA-specific operational rules for this repository.
- `/.gaia/plans/README.md` provides navigation and format guidance only.

## Durable state

- Durable project state lives in `/.gaia/` on the `dev` branch.
- Required durable files:
  - `/.gaia/MEMORY.md`
  - `/.gaia/PLAN.md`
  - `/.gaia/TASKS.md`
- When connected to a GitHub repository:
  - Check for the `/.gaia/` folder and existing durable files on `dev`.
  - If the folder or files do not exist, confirm with the user, then create them.
- Synchronize sandbox state (`MEMORY.md`, `PLAN.md`, `TASKS.md` in the chat) with durable `/.gaia/` state when first connected via the GitHub connector.
- Memory updates must always append to the latest existing file; never overwrite prior content unless explicitly importing memory.
- When `/handoff` is called, ask the user if they want the latest sandbox files appended to the repository durable files before executing the skill.

## Plan lifecycle

- One general template: `/.gaia/templates/PLAN.md`.
- Do not create separate templates for plan states.
- Plan folders:
  - `draft/`
  - `approved/`
  - `on-hold/`
  - `closed/`
- Draft plans need review or approval; they create no Issues, branches, or product-code changes.
- Approved plans may coexist; GAIA executes only one plan at a time.
- Approved executable plans belong in `approved/`.
- Blocked approved plans belong in `on-hold/`.
- Active plan uses `/.gaia/PLAN.md`; active tasks use `/.gaia/TASKS.md`.
- Closed plans belong in `/.gaia/plans/closed/` with filenames `PLAN-YYYYMMDD-NNN.md`.
- Do not archive `TASKS-*.md` files; GitHub Issues provide durable task records.
- Plans use consistent YAML metadata:
  - `id`
  - `status`
  - dates
  - `priority`
  - `blocked_by`
  - `github_issue_refs`
- Plans include an append-only `## Lifecycle Log`.
  - Every transition records: date, previous status, new status, and reason.
  - `on-hold` requires blocker IDs and explanation.
- Plan state expectations:
  - Draft: context, options, and open questions.
  - Approved: implementation-ready.
  - On-hold: blockers and unblocking conditions.
  - Active: Issues and validation.
  - Closed: outcome, Issue mapping, and decisions.

## Plan execution and GitHub Issues

- Approved plan tasks create GitHub Issues when the plan activates.
- `TASKS.md` is a temporary execution mirror.
- Before closing a plan, every task needs an Issue reference.
- Issue completion state must match task completion state.
- Record Issue mappings in the closed plan.
- Inspect existing repository labels first; reuse clearly established labels.
- If labels are absent or unclear, create minimal labels:
  - `plan`
  - `task`
  - `bug`
  - `research`
  - `design`
  - `marketing`

## Stale plans and informal work

- Detect stale plans, summarize them, and ask the user before acting.
- Follow the user's explicit choice for retaining, archiving, or discarding full plans.
- Permanent deletion is never the default.
- Keep `BACKLOG.md` as a lightweight file for informal tasks not ready for formal plans.
- `BACKLOG.md` is separate from formal plan execution and GitHub Issue records.

## Superseded decisions

- `PLAN-001.md` naming is superseded by `PLAN-YYYYMMDD-NNN.md`.
- Archiving `TASKS-*.md` is superseded; archive plans only.
- Sandbox-only state is superseded by sandbox runtime state plus durable `/.gaia/` state.
- A single generic plans archive is superseded by lifecycle folders.
- Separate templates per lifecycle state are superseded by one shared template.
- Plan rules in `plans/README.md` as authority are superseded by `/.gaia/GAIA.md`.
- Asking before creating `dev` is superseded by automatic creation.
- Read-only, Issues-as-source, and bidirectional alternatives are superseded by the approved Tasks-to-Issues workflow.

## User preferences (context)

- Solo founder, Melbourne, Australia; ADHD; short sprints; multiple projects.
- Prefers dense, concise bullets; avoids verbose prose.
- Default skills to load on each new session: `/caveman`, `/ponytail`.
- Typical SaaS stack: Next.js, TypeScript, Tailwind, Supabase, Vercel, Stripe/Lemon Squeezy.
- GAIA handles specification, tickets, and pull-request review.
- Remind the user to export memory before ending sessions.
- New threads begin by importing attached memory.
