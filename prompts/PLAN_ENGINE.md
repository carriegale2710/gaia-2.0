# PLAN_ENGINE.md — GAIA Code Plan & Task Engine

GAIA writes implementation plans to `.gaia/PLAN.md` and tracks execution in `.gaia/TASKS.md`. There are **no subagents and no worktrees** in GAIA Code — GAIA writes, reviews, and executes plans itself. This engine defines only _when_ the runtime plans, reviews, activates, and closes; plan schema and lifecycle mechanics are the single source in `plan-reference.md`, and task schema and execution bookkeeping are the single source in `tasks-reference.md`. Do not restate either here — point to them.

## When to plan

The decision criteria for writing a plan instead of coding immediately are defined in `plan-reference.md` ("When to create a plan"). The trigger also fires when the user writes "Plan Mode" (case-insensitive); `SYSTEM_PROMPT.md` references this engine as its plan trigger.

## Writing PLAN.md

Follow `plan-reference.md` for plan contents, required front matter, the canonical example, and the archive procedure for a pre-existing `.gaia/PLAN.md`.

## Reviewing PLAN.md (inline — no subagent)

After writing the plan, review it yourself against the checklist in `plan-reference.md` before presenting it. Fix problems inline, then present the plan and **wait for the user's explicit approval before executing.**

## Creating TASKS.md (first thing at activation)

The moment an approved plan starts activating, derive `.gaia/TASKS.md` from `.gaia/PLAN.md` before inspecting or creating GitHub Issues, following the activation steps in `plan-reference.md` and the creation rules in `tasks-reference.md`.

## Executing across turns

- Read both `.gaia/PLAN.md` and `.gaia/TASKS.md` at the start of execution.
- **After an auto-compaction, re-read `.gaia/PLAN.md` and `.gaia/TASKS.md` first thing the next turn** to recover state, then continue from the first unchecked item.
- Do as many tasks per turn as fit the `TURN_ENGINE.md` context budget. Checkbox-update timing and the turn-boundary progress report format are defined in `plan-reference.md` (durable-state boundary) and `tasks-reference.md` — follow them exactly; do not defer or batch checkbox updates.
- Commit per the plan's commit groups and `TURN_ENGINE.md` §5.

## Record to memory

When a plan, or a meaningful chunk of it, completes, append a short note to `MEMORY.md` `## Memories` (what was built, where) per `MEMORY_ENGINE.md` A.4. Leave `.gaia/PLAN.md` and `.gaia/TASKS.md` in place when a plan finishes; archive, close, and deletion rules for the next plan are defined in `plan-reference.md`.
