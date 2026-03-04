# /status

Show an overview of the user's GIR configuration — what's installed, what's configured, and what's missing.

---

## Instructions

Follow these steps in order. Gather all information before presenting the report.

### Step 1: Check installed GIR plugins

Look for plugin directories under `~/.claude/plugins/`. Check whether these directories exist:

- `~/.claude/plugins/gir-core/`
- `~/.claude/plugins/gir-web/`
- `~/.claude/plugins/gir-automation/`
- `~/.claude/plugins/gir-tools/`
- `~/.claude/plugins/gir-database/`
- `~/.claude/plugins/gir-ai/`
- `~/.claude/plugins/gir-qa/`

Note which ones are present and which are absent. If `~/.claude/plugins/` does not exist at all, note that no plugins directory was found.

### Step 2: Check for `CLAUDE-project.md` in the current directory

Check whether `CLAUDE-project.md` exists in the current working directory.

- If it exists: note that it's present. Optionally read the first 20 lines to extract the project name and type from the overview section.
- If it does not exist: note that it's missing.

### Step 3: Check for `.gir/` memory bank

Check whether `.gir/` exists in the current working directory.

- If `.gir/` exists: list which of the 5 expected files are present:
  - `CLAUDE-activeContext.md`
  - `CLAUDE-decisions.md`
  - `CLAUDE-patterns.md`
  - `CLAUDE-resources.md`
  - `CLAUDE-troubleshooting.md`
  - Also check if `GIR.modules` is present.
- If `.gir/` does not exist: note that the memory bank is not set up.

### Step 4: Check for available agents

Look for agent files in `~/.claude/agents/`. List any subdirectories or `.md` files found there. If the directory does not exist, note that no agents are installed.

### Step 5: Present the status report

Format the report cleanly. Use plain text — no excessive decorations.

```
GIR Status
==========

Project
-------
  Directory:      [current working directory path]
  CLAUDE-project: [present | MISSING — run /gir-core:init-project]
  [If present, show]:  Name: [detected name]  Type: [detected type]

Memory Bank (.gir/)
-------------------
  [If .gir/ exists]:
    CLAUDE-activeContext.md    [present | missing]
    CLAUDE-decisions.md        [present | missing]
    CLAUDE-patterns.md         [present | missing]
    CLAUDE-resources.md        [present | missing]
    CLAUDE-troubleshooting.md  [present | missing]
    GIR.modules                [present | missing]

  [If .gir/ does not exist]:
    Not initialized — run /gir-core:init-memory-bank

Plugins (~/.claude/plugins/)
----------------------------
  gir-core        [installed | not found]  ← required
  gir-web         [installed | not found]
  gir-automation  [installed | not found]
  gir-tools       [installed | not found]
  gir-database    [installed | not found]
  gir-ai          [installed | not found]
  gir-qa          [installed | not found]
  [any other directories found under ~/.claude/plugins/]

Agents (~/.claude/agents/)
--------------------------
  [list subdirectories and/or agent files found]
  [or: No agents directory found at ~/.claude/agents/]

Recommendations
---------------
  [List only what's missing or needs attention, e.g.:]
  - CLAUDE-project.md not found — run /gir-core:init-project to generate one
  - Memory bank not initialized — run /gir-core:init-memory-bank to scaffold .gir/
  - [specific missing memory bank files if .gir/ exists but files are absent]
  [If everything is configured]:
  - Configuration looks complete. Run /gir-core:drift-check to assess memory bank freshness.
```

Keep the report factual and concise. Do not speculate about plugins or agents that aren't found — only report what is present.
