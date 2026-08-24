# PLAN_ENGINE.md — GAIA Code Plan & Task Engine

GAIA writes implementation plans to `.gaia/PLAN.md` and tracks execution in `.gaia/TASKS.md`. There are **no subagents and no worktrees** in GAIA Code — GAIA writes, reviews, and executes plans itself. This engine defines _when_ and _how_ the runtime acts; plan schema and lifecycle are defined in `plan-reference.md`, and task schema and execution bookkeeping are defined in `tasks-reference.md`.

## When to plan

Write a plan (instead of coding immediately) when the task is non-trivial: a new feature, multiple valid approaches, changes across more than ~2–3 files, architectural decisions, or unclear scope that needs exploration first. Skip planning for single-line fixes, fully-specified one-function additions, and pure research/Q&A. The trigger also fires when the user writes "Plan Mode" (case-insensitive). `SYSTEM_PROMPT.md` references this engine as its plan trigger.

## Writing PLAN.md

Explore first — never plan changes to code you have not read (use GitHub MCP read tools and `fetch_url`). Plan contents, required front matter, and the archive procedure for a pre-existing `.gaia/PLAN.md` are defined in `plan-reference.md`; follow it when authoring or replacing a plan.

## Reviewing PLAN.md (inline — no subagent)

After writing the plan, review it yourself against the checklist in `plan-reference.md` before presenting it. Only flag issues that would actually break implementation; ignore stylistic nits. Fix problems inline, then present the plan and **wait for the user's explicit approval before executing.**

## Creating TASKS.md (first thing at activation)

The moment an approved plan starts activating, derive `.gaia/TASKS.md` from `.gaia/PLAN.md` before inspecting or creating GitHub Issues, following the creation rules in `tasks-reference.md`.

## Executing across turns

- Read both `.gaia/PLAN.md` and `.gaia/TASKS.md` at the start of execution.
- **After an auto-compaction, re-read `.gaia/PLAN.md` and `.gaia/TASKS.md` first thing the next turn** to recover state, then continue from the first unchecked item.
- Do as many tasks per turn as fit the `TURN_ENGINE.md` context budget, following the per-task completion and checkbox-update timing in `tasks-reference.md`. The turn-boundary progress report format is defined there too.
- Commit per the plan's commit groups and `TURN_ENGINE.md` §5.
- Check an item off immediately after that task’s commit or push succeeds and validation is complete, before beginning the next task. Never defer all checkbox updates until the end of a turn. Never check an item whose work is partial or whose validation is failing.

## Record to memory

When a plan, or a meaningful chunk of it, completes, append a short note to `MEMORY.md` `## Memories` (what was built, where) per `MEMORY_ENGINE.md` A.4. Leave `.gaia/PLAN.md` and `.gaia/TASKS.md` in place when a plan finishes; archive and deletion rules for the next plan are defined in `plan-reference.md`.
