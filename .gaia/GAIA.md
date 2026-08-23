# GAIA Code — Repository Instructions

This file contains conventions specific to `carriegale2710/gaia-2.0`. General GAIA behavior, permissions, skills, built-ins, planning mechanics, memory mechanics, commit attribution, and pull-request attribution are defined by `SYSTEM_PROMPT.md`, `MEMORY_ENGINE.md`, `TURN_ENGINE.md`, and the Space instructions. Those files are authoritative and take precedence over this file.

## Loading rule

Read this file when working in `carriegale2710/gaia-2.0`, or when inspecting or changing `/.gaia/`, `/.agents/`, `AGENTS.md`, `CLAUDE.md`, plans, tasks, or repository workflow.

Load the core engine files first. Use this file only for repository-specific conventions and decisions. If this file conflicts with a core engine file, follow the core engine file and report the conflict.

## Repository identity

- Repository: `carriegale2710/gaia-2.0`.
- Purpose: GAIA Space configuration and architecture work.
- Product repository: not yet provided.
- Repository durable state: `/.gaia/`.
- Default integration branch: `dev`.
- Runtime state: sandbox `MEMORY.md`, `PLAN.md`, and `TASKS.md`.
- Skills: Space-file Markdown documents callable as slash commands.

Discovery is complete when the repository identity, default branch, integration branch, durable-state directory, and available agent instruction files have been recorded.

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

### Branch discovery

1. Read the repository branch list and identify the default branch.
2. Check whether `dev` exists.
3. If `dev` is absent, report that it is missing and use the repository default branch only for read-only discovery while waiting for approval to create `dev` from that branch.
4. Create `dev` only under the active permission mode and applicable approval gate. Do not perform repository changes or plan activation until the required base branch exists.
5. Create feature, fix, and chore branches from `dev`.

Branch discovery is complete when the selected base branch, default branch, and any required branch creation or approval state are known.

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

### First GitHub connection

1. Inspect `/.gaia/` on `dev`.
2. Read each existing durable file and record missing files.
3. Read the sandbox `MEMORY.md`, `PLAN.md`, and `TASKS.md` when present.
4. Compare repository and sandbox state without silently merging differences.
5. If only one side contains a file, report the difference and propose copying it to the other side.
6. If both sides differ, present the conflicting sections and ask which source is authoritative.
7. Apply synchronization only after the source, destination, files, and intended changes are clear.
8. Follow the active permission mode and engine approval rules for every repository write.

Synchronization is complete when the source of truth is identified for each differing file, the proposed or completed destinations are recorded, and no unresolved conflict remains hidden.

### Memory updates

- Append durable memory to the latest existing repository memory file; preserve prior entries.
- Do not overwrite repository memory unless the user explicitly requests an import or replacement.
- Follow `MEMORY_ENGINE.md` for sandbox memory structure, compaction, import, export, and permission-mode behavior.

## Plan workflow

Plan format, lifecycle, activation, task tracking, Issue mapping, and plan-history rules are defined in `.gaia/templates/plan-reference.md`. That reference is the single source for plan mechanics.

Repository durable plan state is stored in `/.gaia/PLAN.md` and `/.gaia/TASKS.md` while a plan is active. These files are execution state, not additional lifecycle plans.

## Informal work and stale plans

Use `BACKLOG.md` for informal tasks that are not ready for a formal plan. Keep it separate from formal plan execution and GitHub Issue records.

When a plan appears stale:

1. Identify the plan, status, age, and evidence of staleness.
2. Summarize the consequences of retaining, archiving, or discarding it.
3. Ask the user which action to take.
4. Follow the user's choice.
5. Preserve the plan unless the user explicitly chooses archival or deletion.

Stale-plan handling is complete when the user's decision and resulting file state are recorded.

## Repository decisions

The following older decisions are no longer active:

- `PLAN-001.md` naming is replaced by `PLAN-YYYYMMDD-NNN.md`.
- Plan archives use lifecycle folders rather than one undifferentiated archive.
- One shared plan template is used instead of separate templates for each lifecycle state.
- `.gaia/templates/plan-reference.md` is authoritative for plan mechanics; `/.gaia/GAIA.md` provides repository-specific workflow and state rules.
- The approved Tasks-to-Issues workflow replaces read-only, Issues-as-source, and bidirectional alternatives.
- Repository durable state uses `/.gaia/` alongside sandbox runtime state.

Do not revive a superseded convention unless the user explicitly requests a change to the current repository model.

## Completion standard

Repository-work instructions in this file are complete only when the agent can identify:

- the repository and branch to use;
- the applicable agent instruction files and their precedence;
- the location and state of durable files;
- the plan lifecycle and activation condition;
- the task-to-Issue mapping state;
- any unresolved drift, conflict, or approval requirement.
