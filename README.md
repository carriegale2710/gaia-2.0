# GAIA 2.0

A refactored and enhanced version of the original GAIA Code system.

**Based on:** GAIA Code v3.4  
**Original repo:** [`alexey-max-fedorov/gaia-ai`](https://github.com/alexey-max-fedorov/gaia-ai)

> "GAIA Code is a Claude Code-inspired prompt system for Perplexity Spaces. It brings persistent memory, a plan engine, context-budgeted turns, GitHub MCP integration, and a slash-command skill engine into a Perplexity conversation."

---

## What is GAIA 2.0?

GAIA 2.0 modularizes the original monolithic prompt system into **focused, single-responsibility files**. This makes the system cheaper to run, safer to modify, and easier to test — while adding new capabilities like permission modes, plan lifecycles, and an expanded skill library.

---

## What Changed from Original GAIA?

| Area | Original GAIA | GAIA 2.0 |
|---|---|---|
| **Core files** | 4 files in `prompts/` folder (`SYSTEM_PROMPT.md`, `SYSTEM_INSTRUCTIONS.md`, `MEMORY_ENGINE.md`, `TURN_ENGINE.md`) | 5 engine files + 2 references (`SYSTEM_PROMPT.md`, `MEMORY_ENGINE.md`, `PLAN_ENGINE.md`, `TURN_ENGINE.md`, `BUILTINS.md`, `plan-reference.md`, `tasks-reference.md`) |
| **Skills** | 3 skills mixed in `prompts/` (`doctor.md`, `pr-review.md`, `update.md`) | 30+ skills in dedicated `skills/` folder (`/research`, `/to-prd`, `/bug-fix`, `/wizard`, etc.) |
| **Planning** | Embedded in `MEMORY_ENGINE.md` Part B | Standalone `PLAN_ENGINE.md` + comprehensive `plan-reference.md` + `tasks-reference.md` |
| **Permission modes** | None | 3 modes (Ask Permissions, Accept Edits, Bypass Permissions) |
| **Memory persistence** | Sandbox only | Sandbox + repo `.gaia/` sync + import/export for cross-session persistence |
| **Attribution rules** | None | Mandatory commit co-author + PR footer on every write |
| **Built-in commands** | Implicit | Explicit `/status`, `/help`, `/ask-permissions`, `/accept-edits`, `/dangerously-skip-permissions` |

---

## Architecture

```text
GAIA 2.0/
├── SYSTEM_PROMPT.md       # Identity, character, tool usage, coding philosophy, attribution rules
├── MEMORY_ENGINE.md       # Persistent memory (MEMORY.md), discovery pass, permission mode storage, repo sync
├── PLAN_ENGINE.md         # Plan trigger, PLAN.md/TASKS.md lifecycle, cross-turn execution
├── TURN_ENGINE.md         # Tool-call budgeting (15 calls/turn), context budget, commit batching, permission modes
├── BUILTINS.md            # /status and /help command behavior, permission mode commands
├── plan-reference.md      # Plan schema, lifecycle (draft → approved → on-hold → closed), task mapping
├── tasks-reference.md     # Task creation, execution, completion standards, Issue mapping
├── skills/                # 30+ slash-command skills (/research, /to-prd`, `/bug-fix`, `/wizard`, etc.)
└── README.md              # This file
```

### Key Design Principles

1. **Single responsibility** — each file has one clear purpose
2. **On-demand loading** — only `SYSTEM_PROMPT.md` loads every turn; engines load when their domain is active
3. **Explicit over implicit** — permission modes, built-ins, and attribution rules are documented, not hidden
4. **Durable state** — memory, plans, and tasks survive auto-compaction via sandbox + repo sync
5. **Testable components** — each engine can be validated independently

---

## New Features in GAIA 2.0

### 1. Permission Mode System

Control when GAIA asks for approval on writes:

- **Ask Permissions** (default) — approve every commit, PR, and file change
- **Accept Edits** — auto-approve routine writes; merges and default-branch changes still ask
- **Bypass Permissions** — no approval prompts at all

Commands: `/ask-permissions`, `/accept-edits`, `/dangerously-skip-permissions`

### 2. Plan Lifecycle Management

Formal plan states with explicit transitions:

```
draft → approved → on-hold → closed
```

- Plans live in `.gaia/plans/{status}/PLAN-YYYYMMDD-NNN.md`
- Active plan materialized as `.gaia/PLAN.md`
- Tasks tracked in `.gaia/TASKS.md` with GitHub Issue mapping
- Lifecycle log records every status change with reason

### 3. Repository Durable State

`.gaia/` folder in your repo stores:

- `MEMORY.md` — project structure, notes, permission mode, memories
- `PLAN.md` — active implementation plan
- `TASKS.md` — execution checklist with Issue refs

GAIA syncs sandbox state with repo state on session start.

### 4. Expanded Skill Library

30+ task-specific skills including:

- `/research` — market research and competitive analysis
- `/to-prd` — convert research to product requirements
- `/bug-fix` — diagnose and fix bugs
- `/wizard` — complex multi-step workflows
- `/brainstorming` — idea generation and exploration
- `/persist` — save important context to memory
- `/handoff` — prepare work for human review

Install more from GitHub: `install the skill from gh <owner>/<repo>`

### 5. Mandatory Attribution

Every GAIA-authored commit and PR includes:

**Commit trailer:**
```text
Co-Authored-By: GAIA Code <noreply@gaiacode.pro>
```

**PR footer:**
```text
-----

🌱 Generated with [GAIA Code](https://gaiacode.pro)
```

### 6. Enhanced Tool Discipline

- **15 tool calls per turn** (Perplexity Space override from default 3)
- **~40K token context budget** per turn (~160 KB)
- **Commit batching by file size** — small (8/files), medium (4/files), large (1/file)
- **Registry-first dependency resolution** — never pin versions from search snippets

---

## Quick Start

### 1. Set Up Your Space

1. Fork or clone GAIA 2.0 into your Perplexity workspace
2. Upload all engine files (`SYSTEM_PROMPT.md`, `MEMORY_ENGINE.md`, `PLAN_ENGINE.md`, `TURN_ENGINE.md`, `BUILTINS.md`, `plan-reference.md`, `tasks-reference.md`)
3. Upload the `skills/` folder
4. Connect GitHub MCP for repository access

### 2. Configure Permission Mode

Default is **Ask Permissions** (safest). Change with:

```
/ask-permissions        # approve every write (default)
/accept-edits          # auto-approve routine writes
/dangerously-skip-permissions  # no approval prompts
```

### 3. Start Working

- **Simple tasks** — just ask (GAIA skips planning for single-line fixes)
- **Complex work** — GAIA will automatically plan first (or say "Plan Mode")
- **Use skills** — `/research`, `/to-prd`, `/bug-fix`, etc. for task-specific workflows

### 4. Check Status

```
/status   # version, permission mode, active plan, task progress, memory sections
/help     # list all built-ins and available skills
```

---

## Migration from Original GAIA

If you're upgrading from the original GAIA (`alexey-max-fedorov/gaia-ai`):

### Breaking Changes

- `SYSTEM_INSTRUCTIONS.md` removed — identity merged into `SYSTEM_PROMPT.md` Part I
- Planning moved from `MEMORY_ENGINE.md` Part B to standalone `PLAN_ENGINE.md`
- Skills moved from `prompts/` to `skills/` folder

### What to Do

1. Replace your old `prompts/` folder with the 5 engine files + 2 references
2. Move or re-upload skills into a `skills/` subfolder
3. Update any custom skills to match the new skill template format (frontmatter with `name:` and `description:`)
4. Test with `/status` and `/doctor` to verify deployment

---

## Running a Test

To validate improvements, use the included **`gaia-vs-gaia-2-mvp-test-plan.md`**:

- **Metrics**: response length (proxy for token use) and first-pass task completion rate
- **Setup**: two Perplexity Spaces (original vs. GAIA 2.0), same GitHub MCP repo, same model
- **Time**: ~3–4 hours for 16 paired tasks
- **Output**: defensible, measurable results for résumé or documentation

---

## File Reference

| File | Purpose | Size |
|---|---|---|
| `SYSTEM_PROMPT.md` | Identity, tools, coding practices, attribution | ~17 KB |
| `MEMORY_ENGINE.md` | Persistent memory, discovery, permission storage, repo sync | ~11 KB |
| `PLAN_ENGINE.md` | Plan trigger, lifecycle, cross-turn execution | ~3 KB |
| `TURN_ENGINE.md` | Tool budgeting, context limits, commit batching, permission modes | ~9 KB |
| `BUILTINS.md` | `/status`, `/help`, permission commands | ~3 KB |
| `plan-reference.md` | Plan schema, lifecycle states, task mapping | ~12 KB |
| `tasks-reference.md` | Task creation, execution, completion standards | ~6 KB |
| `skills/*.md` | 30+ task-specific slash commands | varies |

---

## Limitations & Notes

- **Perplexity-only** — designed for Perplexity Spaces with GitHub MCP; won't work standalone
- **Token estimates** — "40–60% reduction" is architectural; run the MVP test for your actual numbers
- **No build step** — sandbox has no internet; CI (e.g., Vercel) is where code actually compiles
- **Attribution mandatory** — every commit/PR must include GAIA Code trailer/footer

---

## Resources

- **Original GAIA:** https://github.com/alexey-max-fedorov/gaia-ai
- **GAIA Code docs:** https://gaiacode.pro
- **Install skills:** `install the skill from gh <owner>/<repo>`

---

**Built with GAIA Code 3.4 in Perplexity**
