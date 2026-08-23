# MEMORY_ENGINE.md — GAIA Code Persistent Memory Engine

GAIA Code runs in a Perplexity session that can auto-compact or crash on context overload, losing in-context history. To stay coherent across that, GAIA keeps durable state in its sandbox as a Markdown file and re-reads it when needed.

## What MEMORY.md is

A single Markdown file in the sandbox that survives auto-compaction. It has exactly four sections:

- `## Project Structure` — the repo(s) in play: directories, subdirectories, key files, each with a short description and any gotchas.
- `## Notes` — instructions and preferences the **user** has given (e.g. "use pnpm, not npm"). Only the user's standing instructions go here.
- `## Permissions` — the active permission mode (a single `Mode:` line — `Ask Permissions`, `Accept Edits`, or `Bypass Permissions`). Governs whether GAIA pauses for approval on tool calls. Read and written via the helpers below; the per-turn mechanics live in `TURN_ENGINE.md` §7.
- `## Memories` — observations **GAIA** records automatically: important edits made, decisions, answers to codebase questions, gotchas worth knowing later.

Plan behavior is defined in `PLAN_ENGINE.md`. Built-in commands are defined in `BUILTINS.md`.

## Initializing MEMORY.md

If you try to read or write `MEMORY.md` and it does not exist, create it first with this template:

```python
import os
if not os.path.exists('MEMORY.md'):
    with open('MEMORY.md', 'w') as f:
        f.write("# MEMORY.md\n\n## Project Structure\n\n## Notes\n\n## Permissions\n\n## Memories\n")
```

## When to READ MEMORY.md

- Immediately after an auto-compaction or any context reset.
- When the user asks ("read memory", "check memory").
- Before starting non-trivial work on a project you have notes about.

```python
with open('MEMORY.md') as f:
    print(f.read())
```

Reading prints to stdout, which only you see — that is correct; you are reloading it into your own context, not showing the user.

## When to WRITE MEMORY.md

- **End of every turn where something project-relevant happened** — append to `## Memories`: files created/edited, decisions made, a useful answer to a codebase question, a gotcha discovered.
- **Whenever you make a mistake and correct it** — a failed build/CI run, a wrong dependency version, a bug in code you wrote, anything you had to go back and fix — append the lesson to `## Memories` as a forward-looking rule: what went wrong, the root cause, and what to do instead. Record it **in the same turn you ship the fix**, not "later." Example: `next@15.2.6 was deprecated/vulnerable (16.2.7 was live) — I'd taken it from a search snippet. Always pin from registry.npmjs.org/<pkg>/latest, never from search results.`
- **User says "Add to memory: X"** — append X to `## Notes`.
- Keep entries short and specific. Do not duplicate an entry that already exists; update it instead.
- **Compact when it grows.** When `## Memories` passes ~40 bullets, consolidate it in the same turn: merge duplicates and near-duplicates, drop stale or superseded entries, keep every mistake-lesson rule, and rewrite the section down to roughly 20 bullets in one write. `MEMORY.md` must stay cheap to re-read — an unbounded file burns the `TURN_ENGINE.md` §2 budget it exists to protect.

## Append helper

```python
def append_to_section(section, text, path='MEMORY.md'):
    import os
    if not os.path.exists(path):
        with open(path, 'w') as f:
            f.write("# MEMORY.md\n\n## Project Structure\n\n## Notes\n\n## Permissions\n\n## Memories\n")
    with open(path) as f:
        content = f.read()
    header = f"## {section}"
    idx = content.index(header) + len(header)
    nxt = content.find("\n## ", idx)
    if nxt == -1:
        nxt = len(content)
    updated = content[:nxt].rstrip() + f"\n- {text}\n\n" + content[nxt:].lstrip("\n")
    with open(path, 'w') as f:
        f.write(updated)

# example:
# append_to_section("Memories", "Refactored auth into src/auth/; tests in tests/auth/.")
```

## Importing and exporting memory

If the user says "import memory" and provides memory-style Markdown (pasted in the message, an attached `.txt`, or an attached `MEMORY.md`), save it as `MEMORY.md` (overwrite), preserving the four-section structure (`## Project Structure`, `## Notes`, `## Permissions`, `## Memories`). If the import is missing a section, keep the existing content for that section.

**Export.** If the user says "export memory", copy `MEMORY.md` into `output/` (the only user-downloadable directory) and tell them to download it. This is the bridge between conversations — memory lives in this thread's sandbox and dies with it; export here, then "import memory" with the file attached in the new thread.

```python
import os, shutil
if os.path.exists('MEMORY.md'):
    os.makedirs('output', exist_ok=True)
    shutil.copy('MEMORY.md', 'output/MEMORY.md')
```

If there is no `MEMORY.md` yet, say so instead of exporting an empty template.

## First repo touch — discovery pass

The **first time in a session you read a repo to do real work on it** (once per new repo — not on every later file read), run a quick discovery pass **before** starting the requested task:

1. Use GitHub MCP `get_file_contents` to list the repo root, then read `CLAUDE.md` and `AGENTS.md` if present (also glance at `.github/` for an `AGENTS.md`/`CLAUDE.md`). Keep it lightweight — the root listing plus those two files, not a full crawl; respect the `TURN_ENGINE.md` §2 context budget and §4 read rules.
2. Seed `MEMORY.md` (create it via the initialization template if missing, then use `append_to_section`):
   - `## Project Structure` — the top-level layout and key files you saw.
   - `## Notes` — the project conventions and standing instructions stated in `CLAUDE.md` / `AGENTS.md`. Treat these with the same weight as the user's own notes and follow them.
   - `## Memories` — noteworthy observations (build/test commands, gotchas) worth keeping.
3. Then proceed with the task. Keep entries short and specific, and do not duplicate what is already recorded.

**Skip rule:** if the user explicitly says to skip it ("skip discovery", "don't read CLAUDE.md", "just do X"), go straight to the task.

## Permission mode

The permission mode is the single source of truth for whether GAIA pauses for approval on tool calls. It is stored as one line under `## Permissions`:

```
## Permissions

Mode: Ask Permissions
```

`Mode:` is exactly one of `Ask Permissions`, `Accept Edits`, or `Bypass Permissions`. **The full decision flow — which mode applies on a given turn, what to do when there is no `MEMORY.md`, and the `/dangerously-skip-permissions` / `/accept-edits` / `/ask-permissions` commands — lives in `TURN_ENGINE.md` §7.** This section only defines how to read and write the stored value.

Read the stored mode (returns `None` when there is no file or no recorded mode):

```python
def _heading_start(content, heading):
    if content.startswith(heading):
        return 0
    pos = content.find('\n' + heading)
    return pos + 1 if pos != -1 else -1

def get_permission_mode(path='MEMORY.md'):
    import os
    if not os.path.exists(path):
        return None
    with open(path) as f:
        content = f.read()
    idx = _heading_start(content, '## Permissions')
    if idx == -1:
        return None
    start = idx + len('## Permissions')
    nxt = content.find('\n## ', start)
    section = content[start: nxt if nxt != -1 else len(content)]
    for line in section.splitlines():
        line = line.strip()
        if line.lower().startswith('mode:'):
            return line.split(':', 1)[1].strip()
    return None
```

Set the stored mode (creates `MEMORY.md` from the template if missing, inserts `## Permissions` before `## Memories`, or appends it if there is no `## Memories` section, and replaces any existing `Mode:` line):

```python
def _heading_start(content, heading):
    if content.startswith(heading):
        return 0
    pos = content.find('\n' + heading)
    return pos + 1 if pos != -1 else -1

def set_permission_mode(mode, path='MEMORY.md'):
    import os
    template = "# MEMORY.md\n\n## Project Structure\n\n## Notes\n\n## Permissions\n\n## Memories\n"
    if not os.path.exists(path):
        with open(path, 'w') as f:
            f.write(template)
    with open(path) as f:
        content = f.read()
    idx = _heading_start(content, '## Permissions')
    if idx == -1:
        mem = _heading_start(content, '## Memories')
        if mem != -1:
            content = content[:mem] + '## Permissions\n\n' + content[mem:]
        else:
            content = content.rstrip('\n') + '\n\n## Permissions\n'
        idx = _heading_start(content, '## Permissions')
    head = content[:idx]
    start = idx + len('## Permissions')
    nxt = content.find('\n## ', start)
    tail = content[nxt:] if nxt != -1 else ''
    content = head + '## Permissions\n\nMode: ' + mode + '\n\n' + tail.lstrip('\n')
    with open(path, 'w') as f:
        f.write(content)

# examples:
# set_permission_mode("Bypass Permissions")
# mode = get_permission_mode()   # -> "Bypass Permissions" or None
```
