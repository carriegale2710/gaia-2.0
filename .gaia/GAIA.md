# GAIA Code — Repository Instructions

Repository-specific conventions for `carriegale2710/gaia-2.0`. Core engine files remain authoritative.

## Repository identity

- Repository: `carriegale2710/gaia-2.0`.
- Purpose: GAIA Space configuration and architecture work.
- Product repository: not yet provided.
- Repository durable state: `/.gaia/`.
- Default integration branch: `dev`.
- Runtime state: sandbox `MEMORY.md`, `PLAN.md`, and `TASKS.md`.
- Skills: Space-file Markdown documents callable as slash commands.

## Branch convention

For repository changes, use `dev` as the default base branch and target pull requests to `dev`.

| Change type | Base branch                               | Pull-request target |
| ----------- | ----------------------------------------- | ------------------- |
| Feature     | `dev`                                     | `dev`               |
| Fix         | `dev`                                     | `dev`               |
| Chore       | `dev`                                     | `dev`               |
| Release     | Ask before departing from this convention | Confirm explicitly  |
| Hotfix      | Ask before departing from this convention | Confirm explicitly  |

Read-only inspection may use the branch required to answer the user's request. If the user specifies another base branch, follow that instruction unless it conflicts with a higher-priority engine rule.

## Agent-file layout

The repository uses the following ownership model:

| Path                     | Role                                           | Expected branch scope |
| ------------------------ | ---------------------------------------------- | --------------------- |
| `/.gaia/`                | GAIA-specific durable state and workflow files | `dev` and descendants |
| `/.agents/`              | Repository agent guidance                      | `dev` and descendants |
| `/AGENTS.md`             | Canonical general agent guidance               | `dev` and descendants |
| `/CLAUDE.md`             | Compatibility pointer to `AGENTS.md`           | `dev` and descendants |
| `/.gaia/GAIA.md`         | GAIA-specific repository conventions           | `dev` and descendants |
| `/.gaia/plans/README.md` | Plan-folder navigation and format guidance     | `dev` and descendants |

`AGENTS.md` is the canonical general repository guidance. `CLAUDE.md` points to it for compatibility. `GAIA.md` supplies only GAIA-specific operational conventions.

Treat copies of these files on `main` as repository drift. Report the drift during discovery. Do not remove or rewrite files on `main` unless the user explicitly requests that cleanup.

The agent-file layout is verified when each listed path has been classified as present, absent, or unexpected, with the relevant branch recorded.

## Durable state

Required repository files:

- `/.gaia/MEMORY.md` — durable project knowledge and decisions.
- `/.gaia/PLAN.md` — active approved plan, when one exists.
- `/.gaia/TASKS.md` — execution mirror for the active plan, when one exists.

Sandbox files with the same names are runtime state. They are not automatically interchangeable with repository files.

## Informal work and stale plans

Use `BACKLOG.md` for informal tasks that are not ready for a formal plan. Keep it separate from formal plan execution and GitHub Issue records.
