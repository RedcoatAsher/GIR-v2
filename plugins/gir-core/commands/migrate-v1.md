# /migrate-v1

Migrate an existing GIR-v1 project to the GIR-v2 modular plugin system. Preserves all memory bank data, converts configuration files, removes v1 artifacts, and guides through v2 plugin installation.

---

## Instructions

Follow these steps in order. **Do not skip steps.** Ask for user confirmation before any destructive action.

### Step 1: Detect GIR-v1 installation

Scan the current project directory for GIR-v1 artifacts. Check for the existence of each of these files and directories:

**Root-level files:**
- `CLAUDE.md` — The monolithic v1 instructions file
- `CLAUDE-core.md` — Alternative name some v1 installs used
- `GIR-sync.sh` — The v1 sync script
- `CLAUDE-project.md` — May already exist if user partially migrated

**`.gir/` memory bank directory:**
- `.gir/CLAUDE-activeContext.md`
- `.gir/CLAUDE-decisions.md`
- `.gir/CLAUDE-patterns.md`
- `.gir/CLAUDE-resources.md`
- `.gir/CLAUDE-troubleshooting.md`

**`.claude/` directory (v1-synced agents and config):**
- `.claude/agents/` — Check for any `.md` agent files
- `.claude/settings.json` — Local Claude Code settings
- `.mcp.json` — MCP tool configuration (root level)

**Other v1 artifacts:**
- `CLAUDE-examples.md` — v1 examples file sometimes synced alongside CLAUDE.md
- `CLAUDE-workflows.md` — v1 workflows file sometimes synced alongside CLAUDE.md
- `CLAUDE-specgates.md` — v1 specgates file sometimes synced alongside CLAUDE.md

Read each file that exists (for `CLAUDE.md` / `CLAUDE-core.md`, read the full contents so you can extract project-specific configuration later).

Present the findings:

```
GIR-v1 Migration Scan
=====================

Root files:
  CLAUDE.md              [found | not found]
  CLAUDE-core.md         [found | not found]
  GIR-sync.sh            [found | not found]
  CLAUDE-project.md      [found | not found]
  CLAUDE-examples.md     [found | not found]
  CLAUDE-workflows.md    [found | not found]
  CLAUDE-specgates.md    [found | not found]

Memory bank (.gir/):
  CLAUDE-activeContext.md    [found (has content) | found (template only) | not found]
  CLAUDE-decisions.md        [found (has content) | found (template only) | not found]
  CLAUDE-patterns.md         [found (has content) | found (template only) | not found]
  CLAUDE-resources.md        [found (has content) | found (template only) | not found]
  CLAUDE-troubleshooting.md  [found (has content) | found (template only) | not found]

Claude config (.claude/):
  agents/                [found: list agent files | not found]
  settings.json          [found | not found]

MCP config:
  .mcp.json              [found | not found]
```

**If no v1 artifacts are found** (no `CLAUDE.md`, no `CLAUDE-core.md`, no `.gir/`, no GIR-sync.sh), inform the user:

```
No GIR-v1 installation detected in this directory.

If this project uses GIR-v1, make sure you're running this command from the project root
where CLAUDE.md and .gir/ are located.

To set up a fresh GIR-v2 project instead, run:
  /gir-core:init-project
  /gir-core:init-memory-bank
```

Then stop — do not proceed further.

**If v1 artifacts are found**, continue to Step 2.

---

### Step 2: Back up existing files

Before making any changes, create a timestamped backup directory.

```bash
mkdir -p .gir-v1-backup-$(date +%Y%m%d-%H%M%S)
```

Copy ALL detected v1 files into the backup directory, preserving structure:

- `CLAUDE.md` and/or `CLAUDE-core.md` → backup root
- `CLAUDE-examples.md`, `CLAUDE-workflows.md`, `CLAUDE-specgates.md` → backup root (if they exist)
- `GIR-sync.sh` → backup root
- `.gir/` → backup as `.gir/` subdirectory (full copy)
- `.claude/agents/` → backup as `.claude/agents/` subdirectory
- `.mcp.json` → backup root
- `CLAUDE-project.md` → backup root (if it exists)

Confirm the backup:

```
Backup created: .gir-v1-backup-YYYYMMDD-HHMMSS/

  [list all files copied]

All existing files have been backed up. If anything goes wrong during
migration, you can restore from this directory.

Continue with migration? (yes/no)
```

Wait for user confirmation. If they say no, stop.

---

### Step 3: Preserve the memory bank

The `.gir/` memory bank is **fully compatible** between v1 and v2. No conversion needed.

**If `.gir/` exists with content:**
- Leave all 5 memory bank files exactly as they are
- Do NOT overwrite them with templates
- Report: "Memory bank preserved — all existing context, decisions, patterns, resources, and troubleshooting data retained."

**If `.gir/` exists but files are template-only:**
- Leave them as-is (v2 templates are the same format)
- Report: "Memory bank files found but contain only templates. They'll work as-is with v2."

**If `.gir/` does not exist:**
- Do NOT create it yet — Step 7 will handle this
- Report: "No memory bank found. Will be created during v2 setup."

**New file — create `.gir/GIR.modules`:**

If `.gir/` exists, create the initial module registry. This will be populated fully after v2 plugins are installed:

```markdown
# GIR Modules Registry
> Auto-generated. Updated each session by installed GIR module hooks.
> Migrated from GIR-v1 on [TODAY'S DATE]

## Installed Modules

### gir-core (v2.0.0) — core
- **Agents**: feature-architect, code-reviewer, debugger, subtask-manager, spec-analyst
- **Skills**: auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines
- **Commands**: init-project, init-memory-bank, drift-check, status, migrate-v1
- **MCP**: sequential-thinking
```

---

### Step 4: Convert CLAUDE.md to CLAUDE-project.md

This is the most important step. The monolithic `CLAUDE.md` (or `CLAUDE-core.md`) from v1 needs to be converted into the structured `CLAUDE-project.md` format with proper markers.

**If `CLAUDE-project.md` already exists:**
- Read it and check if it already has `GIR:START` / `PROJECT:START` markers
- If it has markers: skip conversion, report "CLAUDE-project.md already in v2 format."
- If it exists but has no markers: treat it as a v1-style file that needs conversion (proceed below)

**If only `CLAUDE.md` or `CLAUDE-core.md` exists (or `CLAUDE-project.md` without markers):**

Read the entire file and extract project-specific information. Look for and extract:

1. **Project name** — Look for headings, title references, or project identifiers
2. **Tech stack** — Look for mentions of frameworks (Next.js, React, Python, etc.), languages, and versions
3. **Commands** — Look for `npm run`, `yarn`, `pnpm`, `cargo`, `go`, `pip`, `python`, or other dev/build/test/lint commands
4. **Project structure** — Look for directory trees or file organization descriptions
5. **Key patterns** — Look for coding conventions, naming rules, architecture patterns
6. **Path aliases** — Look for `@/` imports, tsconfig paths, etc.
7. **Environment variables** — Look for `.env` references, API keys, configuration variables
8. **Git workflow** — Look for branch naming, commit conventions, PR processes
9. **MCP tools** — Look for MCP tool references (which ones the project actually uses)
10. **Custom rules** — Any project-specific rules, constraints, or guidelines

Present what you extracted to the user:

```
Extracted from your v1 configuration:

  Project name:    [extracted or "not found"]
  Framework:       [extracted or "not found"]
  Language:        [extracted or "not found"]
  Dev command:     [extracted or "not found"]
  Build command:   [extracted or "not found"]
  Test command:    [extracted or "not found"]
  Lint command:    [extracted or "not found"]
  MCP tools used:  [list of tools referenced]

  Additional content detected:
  - [list any other project-specific sections found]

Please confirm these values or provide corrections before I generate
the new CLAUDE-project.md.
```

Wait for user confirmation or corrections.

Then generate `CLAUDE-project.md` using the v2 template structure with proper markers, populating the `PROJECT:START/END` sections with the extracted values. Use the exact template from `/gir-core:init-project` but fill in the extracted values instead of placeholders.

**Critical rules for conversion:**
- ALL project-specific content goes inside `<!-- PROJECT:START -->` / `<!-- PROJECT:END -->` markers
- ALL GIR guidance/examples go inside `<!-- GIR:START -->` / `<!-- GIR:END -->` markers
- Include the `GIR-SYNC` header comment at the top
- If the v1 file had custom sections that don't map to the v2 template, preserve them in the "Additional Notes" section at the bottom
- **Never discard project-specific content** — if you're unsure where something goes, put it in "Additional Notes"

---

### Step 5: Handle v1-synced agents

**If `.claude/agents/` contains agent files:**

Compare the agent files against the v2 agent set:

**v2 core agents** (provided by gir-core plugin):
- `feature-architect.md`
- `code-reviewer.md`
- `debugger.md`
- `team-lead.md`
- `spec-analyst.md`

**v2 web agents** (provided by gir-web plugin):
- `docs-fetcher.md`
- `deploy-manager.md`
- `ui-generator.md`

**v2 automation agents** (provided by gir-automation plugin):
- `n8n-builder.md`

**v2 tools agents** (provided by gir-tools plugin):
- `subtask-manager.md`
- `agenthub.md`

For each agent file found in `.claude/agents/`:
- If it matches a v2 agent name: mark it for removal (v2 plugin will provide it)
- If it does NOT match any v2 agent: it's a **custom agent** — preserve it

Report:

```
Agent migration plan:

  Will be replaced by v2 plugins:
    [list agents that match v2 names → which plugin provides them]

  Custom agents (will be preserved):
    [list any agents that don't match v2 names]

  Recommended action:
    Remove v1-synced agents that v2 plugins will provide.
    Keep custom agents in .claude/agents/.

Proceed with removing v1-synced agents? (yes/no)
```

Wait for confirmation. If yes, remove only the v1-synced agents (ones that match v2 names). Leave custom agents untouched.

**If `.claude/agents/` does not exist or is empty:**
- Report: "No v1 agents found. v2 plugins will provide agents automatically."

---

### Step 6: Clean up v1 artifacts

After conversion is complete, offer to remove v1-specific files that are no longer needed:

```
The following v1 files are no longer needed (backed up in .gir-v1-backup-*/):

  [list files to remove, e.g.:]
  - CLAUDE.md (replaced by CLAUDE-project.md)
  - CLAUDE-core.md (replaced by CLAUDE-project.md)
  - GIR-sync.sh (v2 uses plugin marketplace instead)
  - CLAUDE-examples.md (now bundled in gir-core plugin)
  - CLAUDE-workflows.md (now bundled in gir-core plugin)
  - CLAUDE-specgates.md (now bundled in gir-core plugin)

Remove these files? (yes/no)
```

Wait for confirmation. If yes, remove only the listed files. Do NOT remove:
- `CLAUDE-project.md` (just created/converted)
- `.gir/` directory (preserved memory bank)
- `.claude/settings.json` (user settings, not GIR-related)
- `.mcp.json` (user's MCP config, may have non-GIR tools)
- Custom agents in `.claude/agents/`

---

### Step 7: Guide v2 plugin installation

Based on what MCP tools and agents were detected in the v1 setup, recommend which v2 plugins to install:

```
GIR-v2 Plugin Installation
==========================

Based on your v1 configuration, here are the recommended plugins:

  Required:
    claude plugin marketplace add RedcoatAsher/GIR-v2
    claude plugin install gir-core

  Recommended (based on your v1 setup):
    [If v0/Figma/Vercel tools were referenced]:
      claude plugin install gir-web          # Frontend: v0, Figma, Vercel

    [If n8n tools were referenced]:
      claude plugin install gir-automation   # Automation: n8n workflows

    [If Supabase tools were referenced]:
      claude plugin install gir-database     # Database: Supabase

    [If Gemini-CLI/Codex were referenced]:
      claude plugin install gir-ai           # AI delegation: Gemini-CLI, Codex

    [If CodeRabbit/Jules were referenced]:
      claude plugin install gir-qa           # QA/review: CodeRabbit, Jules

    [If team-lead/agenthub/subtask were referenced]:
      claude plugin install gir-tools        # Power tools: AgentHub, subtask

  Optional (not detected in your v1 setup but available):
    [list any plugins NOT recommended above]

Run these commands in your terminal to install the plugins. The plugins
will automatically register themselves in .gir/GIR.modules at the start
of your next Claude Code session.
```

**Important**: Do NOT attempt to run `claude plugin install` commands — these must be run by the user in their terminal outside of this session.

---

### Step 8: Update .gitignore

Check `.gitignore` for GIR-related entries:

- Ensure `.gir/` is listed (memory bank should not be committed)
- If `.gir-v1-backup-*/` was created, add it to `.gitignore` as well
- Do NOT remove any existing `.gitignore` entries

If changes are needed, show what will be added and ask for confirmation.

---

### Step 9: Final verification and summary

Read back the key files to verify everything is correct:

1. Read `CLAUDE-project.md` — verify it has proper markers and project content
2. Read `.gir/GIR.modules` — verify it exists
3. Check `.gir/` memory bank files — verify they're still intact
4. Check that v1 artifacts were removed (if user confirmed)
5. Check `.gitignore` — verify entries are present

Present the final summary:

```
GIR v1 → v2 Migration Complete
===============================

What was preserved:
  [checkmark] Memory bank (.gir/) — all existing context retained
    - CLAUDE-activeContext.md    [preserved with content | template]
    - CLAUDE-decisions.md        [preserved with content | template]
    - CLAUDE-patterns.md         [preserved with content | template]
    - CLAUDE-resources.md        [preserved with content | template]
    - CLAUDE-troubleshooting.md  [preserved with content | template]

What was converted:
  [checkmark] CLAUDE.md → CLAUDE-project.md (with v2 markers)

What was created:
  [checkmark] .gir/GIR.modules (module registry)

What was removed:
  [list removed files, or "nothing — user chose to keep v1 files"]

What was backed up:
  [checkmark] .gir-v1-backup-YYYYMMDD-HHMMSS/ (full backup of all v1 files)

Custom agents preserved:
  [list or "none"]

Next steps:
  1. Install v2 plugins (commands listed above)
  2. Start a new Claude Code session — plugins will auto-register
  3. Run /gir-core:status to verify everything is working
  4. Run /gir-core:drift-check to assess memory bank freshness
  5. Delete .gir-v1-backup-*/ once you've confirmed everything works

If anything went wrong, restore from backup:
  cp -r .gir-v1-backup-YYYYMMDD-HHMMSS/* .
```

---

## Edge Cases

### User has both CLAUDE.md and CLAUDE-project.md
- Read both files
- If `CLAUDE-project.md` already has v2 markers, keep it and extract any unique content from `CLAUDE.md` into its "Additional Notes" section
- If neither has markers, merge content from both into a new `CLAUDE-project.md`
- Always confirm with user before overwriting

### User has custom sections in CLAUDE.md
- Any section that doesn't map to the v2 template structure goes into "Additional Notes" at the bottom of `CLAUDE-project.md`
- Never silently discard content

### User has custom MCP tools in .mcp.json
- Do NOT modify `.mcp.json` — v2 plugins bring their own MCP configs via the plugin system
- Inform user: "Your .mcp.json was left unchanged. v2 plugins manage their own MCP configurations separately."

### User has custom hooks in .claude/hooks/
- Do NOT modify any existing hooks
- Inform user: "Your existing hooks were left unchanged. v2 plugins will add their own SessionStart hooks via the plugin system."

### Memory bank has a different structure than expected
- If `.gir/` contains files not in the standard 5-file set, preserve them
- Report any unexpected files found
- Never delete files from `.gir/` that you don't recognize
