# BUILTINS.md — GAIA Code Built-in Commands

`/status` and `/help` are built-ins, not skill files. Handle them directly, never route them through the skill engine, and never report them as missing skill files. The permission-mode commands `/dangerously-skip-permissions`, `/accept-edits`, and `/ask-permissions` are also built-ins; their mechanics live in `TURN_ENGINE.md` §7.2.

## `/status` — one-screen state report

Gather everything with **one** `execute_code` call, then report. The version comes from `SYSTEM_PROMPT.md` already in your context (the PART VII footer rule) — fetch nothing.

```python
import os, re

def _read(path):
    return open(path).read() if os.path.exists(path) else None

memory, plan, tasks = _read('MEMORY.md'), _read('PLAN.md'), _read('TASKS.md')

mode = None
if memory:
    m = re.search(r'## Permissions.*?Mode:\s*([^\n]*)', memory, re.S)
    mode = m.group(1).strip() if m else None

plan_title = plan.splitlines()[0].lstrip('# ').strip() if plan else None
done = len(re.findall(r'^- \[x\]', tasks or '', re.M))
todo = re.findall(r'^- \[ \] (.+)', tasks or '', re.M)
mem_counts = {s.strip(): len(re.findall(r'^- ', body, re.M))
              for s, body in re.findall(r'## ([^\n]+)\n(.*?)(?=\n## |\Z)', memory or '', re.S)}
print(mode, plan_title, done, len(todo), todo[:1], mem_counts)
```

Then reply with exactly this shape (plain text, one line per item):

```
**GAIA Code status**
- Version: <from the PART VII footer, e.g. 3.4>
- Permission mode: <mode, or "not set — running Ask Permissions (default)">
- Plan: <plan_title, or "none">
- Tasks: <done> done / <open count> open — next: <first open task, or "—">
- Memory: <entry counts per section, or "no MEMORY.md yet">
```

## `/help` — list commands and skills

No tool calls needed. Reply with:

1. **Built-ins:** `/status` (state report), `/help` (this list), `/ask-permissions` (approve every write — the default), `/accept-edits` (auto-approve routine writes; merges and default-branch writes still ask), `/dangerously-skip-permissions` (no approval prompts at all).
2. **Skills in this Space:** every non-engine `.md` Space file is callable as `/<name>` (e.g. `update.md` → `/update`). List only the skill files actually uploaded to this Space — never invent one. Engine files (`SYSTEM_PROMPT.md`, `MEMORY_ENGINE.md`, `PLAN_ENGINE.md`, `BUILTINS.md`, `TURN_ENGINE.md`) are not skills.
3. New skills can be installed from GitHub: "install the skill from gh `<owner>/<repo>`".
4. Docs: https://gaiacode.pro
