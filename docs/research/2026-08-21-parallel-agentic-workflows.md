# GAIA Code — Parallel Agentic Workflows: Research & Improvement Opportunities

**Date:** 2026-08-21  
**Scope:** How GAIA Code's current single-agent, sequential architecture could be improved to support parallel agentic workflows.  
**Repo referenced:** [alexey-max-fedorov/gaia-ai](https://github.com/alexey-max-fedorov/gaia-ai/tree/master)

---

## 1. Why Parallelism Matters

Current industry research (2025–2026) identifies four concrete benefits of parallel agentic execution:

| Benefit                         | Finding                                                                                                     | Source                                                                                                                                                             |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Latency reduction**           | Total latency approaches the longest single step, not the sum of all steps                                  | [gurusup.com — Multi-Agent Orchestration Guide](https://gurusup.com/blog/multi-agent-orchestration-guide)                                                          |
| **Quality improvement**         | Specialized parallel agent teams outperform single monolithic models by 30–60% on complex multi-step tasks  | [dev.to — Multi-Agent AI Systems Practical Guide](https://dev.to/aiwave/multi-agent-ai-systems-a-practical-guide-to-orchestrating-llms-for-complex-workflows-3geh) |
| **Context window preservation** | Accuracy degrades when context utilization exceeds 60–70%; splitting work across agents keeps windows fresh | [gurusup.com — Multi-Agent Orchestration Guide](https://gurusup.com/blog/multi-agent-orchestration-guide)                                                          |
| **Error isolation**             | Errors in one sub-task don't compound into unrelated steps — a leading failure mode in sequential pipelines | [redis.io — Why Multi-Agent LLM Systems Fail](https://redis.io/blog/why-multi-agent-llm-systems-fail/)                                                             |

> **Anti-pattern to avoid:** most production systems need 3–5 specialized agents, not 10+. Over-engineering topology is the most common early mistake. — [dev.to multi-agent guide](https://dev.to/aiwave/multi-agent-ai-systems-a-practical-guide-to-orchestrating-llms-for-complex-workflows-3geh)

---

## 2. Parallelism Opportunities Within GAIA's Existing Tool Budget

GAIA already has a 15-tool-call budget per turn. This is enough to run a _simulated_ fan-out pattern within a single turn, without needing subagents.

### 2.1 Fan-out reads (partially supported today)

`TURN_ENGINE §1` says "batch independent calls in one message." This is parallel _reads_ — e.g. reading `CLAUDE.md`, `README.md`, and `package.json` in the same tool batch rather than sequentially.

**Gap:** the guidance is informal. There is no explicit "read batch phase" concept in the turn structure, so GAIA frequently issues reads one at a time when it could batch them.

### 2.2 Fan-out writes (already governed)

Multiple independent file writes _could_ be fanned out, but `TURN_ENGINE §5` governs this via `push_files` batching. The constraint here is commit atomicity and error recovery — already well-handled.

### 2.3 Fan-out skill execution (not supported — key gap)

When the user chains skills (e.g. `/research-to-issues`), each stage runs sequentially with a gate between them. Independent research sub-tasks — e.g. "research pattern A" and "research pattern B" — cannot run in parallel within a single skill invocation.

---

## 3. Improvement Opportunities (Prompt-Only — No Platform Changes Required)

### 3.1 Parallel Task Lanes in `TASKS.md`

**Current state:** `TASKS.md` is a flat ordered checklist — purely sequential.

**Proposed change:** Add a `[PARALLEL START/END]` block syntax for tasks with no dependency on each other:

```markdown
- [ ] Task 1: Research auth patterns
- [PARALLEL START]
  - [ ] Task 2a: Read /src/auth/
  - [ ] Task 2b: Read /src/api/
  - [ ] Task 2c: Search commits for "auth" changes
- [PARALLEL END — merge before Task 3]
- [ ] Task 3: Write auth refactor plan
```

GAIA would issue all reads in the PARALLEL block as a single batched tool-call phase, respect the TURN_ENGINE context budget across the batch, then collect outputs before proceeding. No subagent needed — just explicit batching discipline.

**Files to edit:** `MEMORY_ENGINE.md §B.2` (plan writing rules), `§B.5` (execution rules).  
**Effort:** Low — prose addition only.

---

### 3.2 Codified Read-Phase / Write-Phase Turn Structure

**Current state:** `TURN_ENGINE` says batch when possible, but the guidance is informal.

**Proposed change:** Formalize a two-phase structure for non-trivial turns:

1. **Read phase** — all `get_file_contents`, `search_web`, `fetch_url`, `list_commits` calls issued concurrently as one batch.
2. **Write phase** — commits and file writes after all reads resolve.

This prevents the common failure mode of interspersed reads and writes that each consume context budget inefficiently. Analogous to the "sectioning" parallelization pattern described in agentic workflow literature — breaking a task into independent subtasks processed concurrently before merging outputs. — [spring.io — Building Effective Agents, Part 1](https://spring.io/blog/2025/01/21/spring-ai-agentic-patterns/)

**Files to edit:** `TURN_ENGINE.md §4`.  
**Effort:** Low — prose addition.

---

### 3.3 Skill Fan-Out Directive

**Current state:** Skills are strictly sequential — one instruction set, one action stream.

**Proposed change:** Add an optional `parallel_subtasks:` frontmatter field to the skill file spec. When present, GAIA issues all listed sub-task tool calls as a single batch at the start of that skill's execution step, collects outputs, then continues with merged results:

```yaml
parallel_subtasks:
  - search_web: "query A"
  - search_web: "query B"
  - get_file_contents: "path/to/X"
```

**Files to edit:** `skills/write-a-skill.md`, `SYSTEM_INSTRUCTIONS.md` skill engine block.  
**Effort:** Medium — new spec field + skill engine handling logic.

---

### 3.4 Evaluator-Optimizer Loop for Plans

**Current state:** `MEMORY_ENGINE §B.3` has a single-pass self-review checklist for `PLAN.md`.

**Proposed change:** Replace with a three-pass evaluator-optimizer loop — a high-value agentic pattern documented across multiple frameworks:

1. **Generator pass** — write `PLAN.md` as today.
2. **Evaluator pass** — re-read `PLAN.md` as a second engineer. Identify: tasks that could run in parallel, context-budget risks, missing dependencies. Produce a scored gap list.
3. **Optimizer pass** — rewrite only the flagged sections.

This is a prompt-only change. The pattern consistently improves output quality in LLM pipeline research. — [spring.io agentic patterns](https://spring.io/blog/2025/01/21/spring-ai-agentic-patterns/), [MDPI LLM Multi-Agent Survey 2026](https://www.mdpi.com/1999-5903/18/6/326)

**Files to edit:** `MEMORY_ENGINE.md §B.3`.  
**Effort:** Low — prose revision.

---

### 3.5 Context-Budget-Aware Task Scheduling

**Current state:** `TURN_ENGINE §2` defines a soft 40K token ceiling but leaves budget estimation to GAIA's in-turn judgment.

**Proposed change:** Add a `weight: S/M/L` hint field to `TASKS.md` entries, filled in during plan creation. Before executing a turn, GAIA sums weights of candidate tasks and selects the largest set that fits the ceiling — moving budget estimation from implicit reasoning to explicit pre-computation.

**Files to edit:** `MEMORY_ENGINE.md §B.2, §B.4`, `TURN_ENGINE.md §2`.  
**Effort:** Medium — adds new field syntax and scheduling logic.

---

## 4. What Requires Platform Changes (Out of Scope)

The following improvements would require Perplexity to expose new capabilities not available in Spaces as of mid-2026:

- **True concurrent tool execution** — multiple tool calls running in parallel at the infrastructure layer (not just batched in one message).
- **Subagent spawning** — a second isolated context window for a background task.
- **Shared state bus** — a durable key-value store accessible across multiple Space conversations simultaneously.
- **Streaming partial results** — receiving incremental output from a long tool call while issuing other calls.

For multi-agent orchestration at this level, frameworks such as LangGraph, AutoGen, and OpenAI Agents SDK are the current state of the art. — [MDPI LLM Multi-Agent Orchestration Survey, June 2026](https://www.mdpi.com/1999-5903/18/6/326)

---

## 5. Priority Order

| Priority | Change                                  | Files to Edit                                       | Effort |
| -------- | --------------------------------------- | --------------------------------------------------- | ------ |
| 1        | Parallel task lanes in TASKS.md         | `MEMORY_ENGINE.md §B.2, §B.5`                       | Low    |
| 2        | Read-phase / write-phase turn structure | `TURN_ENGINE.md §4`                                 | Low    |
| 3        | Evaluator-optimizer loop for plans      | `MEMORY_ENGINE.md §B.3`                             | Low    |
| 4        | Skill fan-out directive                 | `skills/write-a-skill.md`, `SYSTEM_INSTRUCTIONS.md` | Medium |
| 5        | Context-budget-aware task scheduling    | `MEMORY_ENGINE.md §B.2, §B.4`, `TURN_ENGINE.md §2`  | Medium |

---

## 6. Open Questions

- Would parallel task lanes in `TASKS.md` complicate the auto-compaction recovery logic in `§B.5`? The re-read-and-resume pattern needs to handle partial `[PARALLEL]` blocks gracefully.
- Is the 15-tool-call cap a hard API limit or a soft prompt instruction? If hard, a 12-call read-phase fan-out leaves almost no budget for writes in the same turn — may require read and write phases to always span separate turns.
- Is there appetite to re-introduce a lightweight worktrees or subagent concept, given Claude Code now ships this natively? Could be modeled as a skill that opens a second Space conversation via a shareable link and passes a `MEMORY.md` export as context.

---

## References

1. **gurusup.com — Multi-Agent Orchestration: How to Coordinate AI Agents at Scale** (May 2026)  
   https://gurusup.com/blog/multi-agent-orchestration-guide

2. **dev.to — Multi-Agent AI Systems: A Practical Guide to Orchestrating LLMs for Complex Workflows** (June 2026)  
   https://dev.to/aiwave/multi-agent-ai-systems-a-practical-guide-to-orchestrating-llms-for-complex-workflows-3geh

3. **redis.io — Why Multi-Agent LLM Systems Fail & How to Fix Them** (April 2026)  
   https://redis.io/blog/why-multi-agent-llm-systems-fail/

4. **dev.to — Multi-Agent Orchestration: Patterns That Work** (March 2026)  
   https://dev.to/thedailyagent/multi-agent-orchestration-a-guide-to-patterns-that-work-1h81

5. **MDPI Future Internet — LLM-Based Multi-Agent Orchestration: A Survey of Frameworks, Communication Protocols, and Emerging Patterns** (June 2026)  
   https://www.mdpi.com/1999-5903/18/6/326

6. **spring.io — Building Effective Agents with Spring AI, Part 1: Agentic Patterns** (January 2025)  
   https://spring.io/blog/2025/01/21/spring-ai-agentic-patterns/

7. **medium.com — Routing & Parallelization: Agentic AI Workflow Patterns (2/4)** (July 2025)  
   https://medium.com/the-advanced-school-of-ai/routing-parallelization-agentic-workflow-patterns-part-2-2bc1b6c46113

8. **youmind.com — Cockpit Architecture: Adaptive Agent Orchestration System** (June 2026)  
   https://youmind.com/landing/x-viral-articles/cockpit-adaptive-agent-orchestration-system

9. **alexey-max-fedorov/gaia-ai — CLAUDE.md** (source repo)  
   https://github.com/alexey-max-fedorov/gaia-ai/blob/master/CLAUDE.md

10. **alexey-max-fedorov/gaia-ai — prompts/TURN_ENGINE.md** (source repo)  
    https://github.com/alexey-max-fedorov/gaia-ai/blob/master/prompts/TURN_ENGINE.md

11. **alexey-max-fedorov/gaia-ai — prompts/MEMORY_ENGINE.md** (source repo)  
    https://github.com/alexey-max-fedorov/gaia-ai/blob/master/prompts/MEMORY_ENGINE.md

12. **alexey-max-fedorov/gaia-ai — research/OVERHAUL.md** (source repo)  
    https://github.com/alexey-max-fedorov/gaia-ai/blob/master/research/OVERHAUL.md
