# Documentation Update Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Fix all stale and missing documentation across gir-core/README.md, gir-web/README.md, root README.md, and PLAN-modularization.md.

**Architecture:** Pure documentation edits — no code changes. Four files, four independent tasks. No dependencies between tasks.

**Tech Stack:** Markdown

---

## Context

Four files need updating based on an audit of the repo:

1. `plugins/gir-core/README.md` — missing 2 commands (`/modules`, `/migrate-v1`); header says "Four slash commands"
2. `plugins/gir-web/README.md` — claims 3 skills, documents only 2; `web-tools` skill is missing entirely
3. `README.md` (root) — module table says "5 commands" for gir-core, should be 6
4. `PLAN-modularization.md` — marked "Draft" but implementation is complete; contains stale agent placement examples

No tests needed — these are documentation-only changes. Each task ends with a commit.

---

### Task 1: Update gir-core/README.md — Add missing commands

**Files:**
- Modify: `plugins/gir-core/README.md:44-52`

**What to change:**

The Slash Commands section currently says "Four Slash Commands" (implied by listing 4) and is missing `/modules` and `/migrate-v1`.

**Step 1: Add `/modules` and `/migrate-v1` rows to the commands table**

Replace the current commands table (lines 47-51):

```markdown
| Command | Usage |
|---------|-------|
| `/init-project` | Scaffolds `CLAUDE-project.md` for a new or existing project |
| `/init-memory-bank` | Creates the `.gir/` memory bank directory with starter files |
| `/drift-check` | Compares current code state against the active spec; surfaces divergence |
| `/status` | Prints active context, current session goals, and memory bank summary |
```

With:

```markdown
| Command | Usage |
|---------|-------|
| `/init-project` | Scaffolds `CLAUDE-project.md` for a new or existing project |
| `/init-memory-bank` | Creates the `.gir/` memory bank directory with starter files |
| `/drift-check` | Compares current code state against the active spec; surfaces divergence |
| `/status` | Prints active context, current session goals, and memory bank summary |
| `/modules` | Lists all installed GIR modules and their registered tools |
| `/migrate-v1` | Migrates a GIR v1 project to v2 — backs up, converts `CLAUDE.md`, installs right modules |
```

**Step 2: Verify the file looks correct**

Open `plugins/gir-core/README.md` and confirm the table has 6 rows.

**Step 3: Commit**

```bash
git add plugins/gir-core/README.md
git commit -m "docs(gir-core): add /modules and /migrate-v1 to slash commands table"
```

---

### Task 2: Update gir-web/README.md — Add missing web-tools skill

**Files:**
- Modify: `plugins/gir-web/README.md:39-44`

**What to change:**

The Skills section currently lists 2 skills (`frontend-design`, `design-principles`) but the plugin has a third: `web-tools` (MCP tool reference for v0, Figma, Vercel).

**Step 1: Add `web-tools` row to the skills table**

Replace the current skills table:

```markdown
| Skill | What it activates |
|-------|------------------|
| **frontend-design** | Component architecture patterns, state management conventions, accessibility defaults, responsive layout guidance |
| **design-principles** | Visual hierarchy, spacing systems, typography choices, and color usage — applied during UI generation and review |
```

With:

```markdown
| Skill | What it activates |
|-------|------------------|
| **frontend-design** | Component architecture patterns, state management conventions, accessibility defaults, responsive layout guidance |
| **design-principles** | Visual hierarchy, spacing systems, typography choices, and color usage — applied during UI generation and review |
| **web-tools** | MCP tool reference for UI generation (v0, Figma) and deployment (Vercel) — quick reference for available tool names and actions |
```

**Step 2: Verify the file looks correct**

Open `plugins/gir-web/README.md` and confirm the skills table has 3 rows.

**Step 3: Commit**

```bash
git add plugins/gir-web/README.md
git commit -m "docs(gir-web): add missing web-tools skill to skills table"
```

---

### Task 3: Update root README.md — Fix gir-core command count

**Files:**
- Modify: `README.md:163`

**What to change:**

The Modules table describes gir-core as having "5 commands" but there are 6.

**Step 1: Find the gir-core row in the modules table**

It's around line 163 and currently reads:

```markdown
| [gir-core](plugins/gir-core/) | Core hub. Agents, delegation, workflows, memory bank, slash commands | 5 agents, 6 skills, 5 commands, SessionStart hook, sequential-thinking MCP | Everyone |
```

**Step 2: Update "5 commands" to "6 commands"**

Replace that row with:

```markdown
| [gir-core](plugins/gir-core/) | Core hub. Agents, delegation, workflows, memory bank, slash commands | 5 agents, 6 skills, 6 commands, SessionStart hook, sequential-thinking MCP | Everyone |
```

**Step 3: Verify the file looks correct**

Confirm the gir-core row now says "6 commands".

**Step 4: Commit**

```bash
git add README.md
git commit -m "docs: fix gir-core command count in modules table (5 → 6)"
```

---

### Task 4: Update PLAN-modularization.md — Mark complete, fix stale examples

**Files:**
- Modify: `PLAN-modularization.md`

**What to change:**

Four things:
1. Change `Status: Draft` → `Status: Implementation Complete`
2. Fix gir-core example that wrongly includes `subtask-manager` (it's in gir-tools)
3. Fix gir-tools example that wrongly includes `team-lead` (it's in gir-core)
4. Fix the gir-core hook prompt example that lists stale agents and commands

**Step 1: Update status header**

Replace:
```markdown
**Status**: Draft
```

With:
```markdown
**Status**: Implementation Complete (as of 2026-03-03)
```

**Step 2: Fix Phase 1 gir-core agents example (around line 126)**

Find the block that lists gir-core agents as:
```json
"agents": [
  "feature-architect",
  "code-reviewer",
  "debugger",
  "subtask-manager",
  "spec-analyst"
]
```

Replace `"subtask-manager"` with `"team-lead"`:
```json
"agents": [
  "feature-architect",
  "code-reviewer",
  "debugger",
  "team-lead",
  "spec-analyst"
]
```

**Step 3: Fix Phase 3 gir-tools agents example (around line 222)**

Find the block that lists gir-tools agents as:
```json
"agents": ["team-lead", "agenthub"],
```

Replace `"team-lead"` with `"subtask-manager"`:
```json
"agents": ["subtask-manager", "agenthub"],
```

**Step 4: Fix gir-tools hook prompt (around line 520)**

Find the string in the gir-tools hook prompt:
```
- **Agents**: team-lead, agenthub
```

Replace with:
```
- **Agents**: subtask-manager, agenthub
```

**Step 5: Fix gir-core hook prompt agents list (around line 595 and 615)**

Find the gir-core hook prompt block listing:
```
- **Agents**: feature-architect, code-reviewer, debugger, subtask-manager, spec-analyst
- **Skills**: auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines
- **Commands**: init-project, init-memory-bank, drift-check, status
```

Replace with:
```
- **Agents**: feature-architect, code-reviewer, debugger, team-lead, spec-analyst
- **Skills**: auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines
- **Commands**: init-project, init-memory-bank, drift-check, status, modules, migrate-v1
```

**Step 6: Verify all replacements look correct**

Search for `subtask-manager` and `team-lead` in the file to confirm they're in the right places.

**Step 7: Commit**

```bash
git add PLAN-modularization.md
git commit -m "docs: mark modularization plan complete, fix stale agent placement examples"
```

---

## Execution Order

Tasks 1–4 are fully independent. Execute in any order. No shared state.

Recommended order: 1 → 2 → 3 → 4 (smallest to most involved).
