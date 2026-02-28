# GIR v2 — Modularization Plan

> Split spokes into independent, marketplace-installable modules with a dynamic registry system.

**Status**: Draft
**Date**: 2026-02-28
**Author**: RedcoatAsher + Claude

---

## Problem Statement

GIR v2 uses a hub-and-spoke architecture, but today all spokes ship in a single monorepo marketplace. When a user adds the marketplace and installs `gir-core`, the core-practices skill alone loads ~220 lines of context that includes MCP tool documentation for **every spoke** (n8n, Vercel, Figma, v0, Supabase, CodeRabbit, Jules). This burns tokens on every prompt regardless of whether the user has those spokes installed or even uses those tools.

**Specific waste in `core-practices/SKILL.md`:**

| Lines | Content | Belongs in |
|-------|---------|-----------|
| 46-52 | Context7, Ref, exa, fetch docs | Core (generic documentation tools) |
| 54-67 | Gemini-CLI delegation | Core (generic delegation) |
| 69-82 | Sequential-Thinking | Core (universal) |
| 87-92 | v0, Figma UI generation | `gir-web` |
| 94-95 | Vercel MCP tools | `gir-web` |
| 96-98 | Supabase MCP tools | New spoke or `gir-web` |
| 99-103 | n8n MCP tools + skills | `gir-automation` |
| 105-107 | CodeRabbit, Jules, claude-mem | New spoke(s) or user-config |
| 176-179 | Agent listing (Web/Automation) | Respective spokes |

Additionally, `workflows/SKILL.md` references spoke-specific tools in its decision matrix (line 302: "Figma, v0" for design implementation) and commit phase (line 183: "deploy to Vercel").

---

## Goals

1. **Token efficiency** — Only installed module context loads per session
2. **True modularity** — Users install `gir-core`, then choose their spokes
3. **Dynamic registry** — A `GIR.modules` file that auto-updates when modules are installed/removed
4. **Extensibility** — Third-party / unofficial modules can self-register using the same system
5. **Monorepo development** — Keep all official plugins in the GIR-v2 repo for development convenience

---

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Repo structure | Stay monorepo | Easier to develop, test, and version together |
| Core scope | Keep all 5 agents + 6 skills as required | Core's value is the unified skill set; don't fragment it |
| Registry location | `.gir/GIR.modules` (project-local) | Consistent with existing `.gir/` memory bank pattern |
| Registration mechanism | Self-registering SessionStart hooks per spoke | Each module registers itself; no central scanner needed |
| Manifest format | `gir-module.json` per plugin | Standard contract for official + unofficial modules |
| Stale entry cleanup | Core's SessionStart hook prunes missing modules | Self-healing when spokes are uninstalled |

---

## Architecture

### Current (everything coupled)

```
SessionStart hook fires
  └─ Core loads ALL context (including spoke MCP tool docs)
     └─ ~220 lines of core-practices, many irrelevant to user's project
```

### Target (modular, self-registering)

```
SessionStart hooks fire (one per installed module)
  ├─ gir-core hook:
  │   ├─ Restore .gir/CLAUDE-activeContext.md
  │   ├─ Read .gir/GIR.modules
  │   ├─ Prune entries for uninstalled modules
  │   └─ Load ONLY core context (~140 lines, no spoke tools)
  │
  ├─ gir-web hook (only if installed):
  │   ├─ Register in .gir/GIR.modules
  │   └─ Web-specific tool context loaded via its own skills
  │
  ├─ gir-automation hook (only if installed):
  │   ├─ Register in .gir/GIR.modules
  │   └─ n8n tool context loaded via its own skills
  │
  └─ gir-tools hook (only if installed):
      ├─ Register in .gir/GIR.modules
      └─ Team/subtask context loaded via its own skills
```

---

## Implementation Plan

### Phase 1: Module Manifest System

Create a standardized `gir-module.json` manifest that every GIR-compatible module includes. This is the contract for self-registration.

#### 1.1 Define the manifest schema

Create `gir-module.json` in each plugin root.

**`plugins/gir-core/gir-module.json`:**
```json
{
  "name": "gir-core",
  "version": "2.0.0",
  "type": "core",
  "description": "Core agents, workflows, best practices, and auto-delegation",
  "requires": [],
  "provides": {
    "agents": [
      "feature-architect",
      "code-reviewer",
      "debugger",
      "subtask-manager",
      "spec-analyst"
    ],
    "skills": [
      "auto-delegation",
      "core-practices",
      "workflows",
      "ralph-loops",
      "specgates",
      "state-machines"
    ],
    "commands": [
      "init-project",
      "init-memory-bank",
      "drift-check",
      "status"
    ],
    "mcp": {
      "sequential-thinking": {
        "description": "Extended reasoning for complex planning and debugging",
        "commands": ["sequentialthinking"]
      }
    }
  }
}
```

**`plugins/gir-web/gir-module.json`:**
```json
{
  "name": "gir-web",
  "version": "2.0.0",
  "type": "spoke",
  "description": "Frontend/fullstack agents and design skills",
  "requires": ["gir-core"],
  "provides": {
    "agents": ["docs-fetcher", "deploy-manager", "ui-generator"],
    "skills": ["design-principles", "frontend-design"],
    "commands": [],
    "mcp": {
      "v0": {
        "description": "React scaffolding and UI mockups",
        "commands": ["createChat"]
      },
      "figma": {
        "description": "UI generation from Figma designs",
        "commands": ["get_design_context", "get_screenshot", "generate_diagram"]
      },
      "vercel": {
        "description": "Deployment management",
        "commands": ["list_projects", "get_project", "list_deployments", "get_deployment", "get_deployment_build_logs", "deploy_to_vercel", "search_vercel_documentation"]
      },
      "context7": {
        "description": "Library and API documentation lookup",
        "commands": ["resolve-library-id", "get-library-docs"]
      },
      "ref": {
        "description": "Broader documentation search and URL content",
        "commands": ["search", "fetch"]
      }
    }
  }
}
```

**`plugins/gir-automation/gir-module.json`:**
```json
{
  "name": "gir-automation",
  "version": "2.0.0",
  "type": "spoke",
  "description": "n8n workflow builder agent",
  "requires": ["gir-core"],
  "provides": {
    "agents": ["n8n-builder"],
    "skills": [],
    "commands": [],
    "mcp": {
      "n8n-mcp": {
        "description": "n8n workflow design, validation, and deployment",
        "commands": ["search_nodes", "get_node", "validate_node", "validate_workflow", "search_templates", "get_template", "n8n_create_workflow", "n8n_get_workflow", "n8n_list_workflows", "n8n_test_workflow", "n8n_deploy_template"]
      }
    }
  }
}
```

**`plugins/gir-tools/gir-module.json`:**
```json
{
  "name": "gir-tools",
  "version": "2.0.0",
  "type": "spoke",
  "description": "Power user tools — AgentHub, team coordination, subtask parallel execution",
  "requires": ["gir-core"],
  "provides": {
    "agents": ["team-lead", "agenthub"],
    "skills": ["subtask", "agenthub-session-management"],
    "commands": [],
    "mcp": {}
  }
}
```

#### 1.2 Add `dependencies` to plugin.json files

Update each spoke's `.claude-plugin/plugin.json` to include a formal `dependencies` field:

```json
{
  "name": "gir-web",
  "version": "2.0.0",
  "dependencies": ["gir-core"],
  ...
}
```

---

### Phase 2: Clean Up Core

This is where the actual token savings happen. Remove all spoke-specific content from core skills.

#### 2.1 Refactor `core-practices/SKILL.md`

**Remove entirely** (move to spoke skills):

- **Lines 87-92** — UI Generation (v0, Figma) → move to `gir-web/skills/web-tools/SKILL.md`
- **Lines 94-95** — Vercel MCP → move to `gir-web/skills/web-tools/SKILL.md`
- **Lines 96-98** — Supabase MCP → move to `gir-web/skills/web-tools/SKILL.md` (or new spoke)
- **Lines 99-103** — n8n MCP + skills → move to `gir-automation/skills/n8n-tools/SKILL.md`
- **Lines 105-107** — CodeRabbit, Jules, claude-mem → move to spoke or remove (user-configured)
- **Lines 176-179** — Agent listing for Web/Automation categories → remove from core

**Keep in core** (universal tools):

- **Lines 46-52** — Context7, Ref, exa, fetch (generic documentation lookup)
- **Lines 54-67** — Gemini-CLI delegation
- **Lines 69-82** — Sequential-Thinking (core MCP dependency)

**Replace removed sections with** a single dynamic reference:

```markdown
## Installed Module Tools

Active modules and their MCP tools are listed in `.gir/GIR.modules`.
Run `/gir-core:modules` to see installed modules and available tools.
New modules register themselves automatically at session start.
```

This replaces ~40 lines of spoke-specific tool documentation with 4 lines.

#### 2.2 Create new spoke-owned tool skills

**`plugins/gir-web/skills/web-tools/SKILL.md`** (new):

```markdown
---
name: web-tools
description: MCP tool reference for web development — v0, Figma, Vercel, Context7, Ref.
  Apply when doing UI generation, deployment, or documentation lookup for frontend work.
---
# Web Development Tools

## UI Generation

| Tool | Use |
|------|-----|
| **v0** createChat | React scaffolding, UI mockups |
| **Figma** get_design_context | UI from Figma nodes |
| **Figma** get_screenshot | Visual reference |
| **Figma** generate_diagram | Flowcharts, diagrams in FigJam |

## Deployment

### Vercel MCP
`list_projects`, `get_project`, `list_deployments`, `get_deployment`,
`get_deployment_build_logs`, `deploy_to_vercel`, `search_vercel_documentation`

## Documentation

| Need | Tool | When |
|------|------|------|
| Library/API docs | **Context7** | Code gen, setup, config, library questions |
| General docs | **Ref** | Broader search, URL content |
| Code examples | **exa** | Real-world patterns, SDKs |
| Web + images | **fetch:imageFetch** | URLs with image extraction |

**Auto-trigger Context7**: When generating code, configuring libraries, or API questions.

## Supabase MCP
`search_docs`, `list_projects`, `list_tables`, `execute_sql`, `apply_migration`,
`deploy_edge_function`, `get_logs`, `get_advisors`, `create_branch`, `merge_branch`
```

**`plugins/gir-automation/skills/n8n-tools/SKILL.md`** (new):

```markdown
---
name: n8n-tools
description: MCP tool reference for n8n workflow automation. Apply when building,
  validating, or debugging n8n workflows.
---
# n8n Automation Tools

## n8n MCP
`search_nodes`, `get_node`, `validate_node`, `validate_workflow`,
`search_templates`, `get_template`, `n8n_create_workflow`, `n8n_get_workflow`,
`n8n_list_workflows`, `n8n_test_workflow`, `n8n_deploy_template`

**Skills**: n8n-node-configuration, n8n-code-javascript/python,
n8n-workflow-patterns, n8n-expression-syntax, n8n-validation-expert
```

#### 2.3 Clean up `workflows/SKILL.md`

- **Line 183** — Remove "deploy to Vercel" from Commit phase. Replace with: "deploy if configured"
- **Line 219** — "Visual Iteration Workflow" section references v0/Figma tools. Add a guard: "Requires gir-web module"
- **Lines 302-303** — Decision matrix references "Figma, v0". Add note: "(requires gir-web)"

These are lightweight annotations, not full content extraction. The detailed tool docs are already in the spoke skills.

---

### Phase 3: Self-Registering Module System

#### 3.1 Add SessionStart hooks to each spoke

Each spoke gets a `hooks/hooks.json` that registers the module in `.gir/GIR.modules` at session start.

**`plugins/gir-web/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-web is listed as an installed module with version 2.0.0, type spoke, agents [docs-fetcher, deploy-manager, ui-generator], skills [design-principles, frontend-design, web-tools], and mcp tools [v0, figma, vercel, context7, ref]. If it doesn't exist or gir-web is missing, add it. Write the updated file. Do this silently without telling the user."
          }
        ]
      }
    ]
  }
}
```

**`plugins/gir-automation/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-automation is listed as an installed module with version 2.0.0, type spoke, agents [n8n-builder], skills [n8n-tools], and mcp tools [n8n-mcp]. If it doesn't exist or gir-automation is missing, add it. Write the updated file. Do this silently without telling the user."
          }
        ]
      }
    ]
  }
}
```

**`plugins/gir-tools/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-tools is listed as an installed module with version 2.0.0, type spoke, agents [team-lead, agenthub], skills [subtask, agenthub-session-management], and mcp tools []. If it doesn't exist or gir-tools is missing, add it. Write the updated file. Do this silently without telling the user."
          }
        ]
      }
    ]
  }
}
```

#### 3.2 Update core's SessionStart hook

Enhance `plugins/gir-core/hooks/hooks.json` to also handle the registry:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if .gir/CLAUDE-activeContext.md exists in the current project. If it does, read it to restore session context. If it doesn't, briefly mention that the user can run /gir-core:init-memory-bank to set up a memory bank. Then, if .gir/GIR.modules exists, read it and verify all listed modules are actually installed (their agents should be available). Remove any entries for modules that are no longer installed. Ensure gir-core is listed with version 2.0.0, type core, agents [feature-architect, code-reviewer, debugger, subtask-manager, spec-analyst], skills [auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines], commands [init-project, init-memory-bank, drift-check, status], and mcp [sequential-thinking]. If .gir/GIR.modules doesn't exist and .gir/ directory exists, create it with just the gir-core entry. Do the module registry work silently."
          }
        ]
      }
    ]
  }
}
```

#### 3.3 Define the `GIR.modules` file format

The generated `.gir/GIR.modules` file will look like this (Markdown for human readability):

```markdown
# GIR Modules Registry
> Auto-generated. Updated each session by installed GIR module hooks.

## Installed Modules

### gir-core (v2.0.0) — core
- **Agents**: feature-architect, code-reviewer, debugger, subtask-manager, spec-analyst
- **Skills**: auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines
- **Commands**: init-project, init-memory-bank, drift-check, status
- **MCP**: sequential-thinking

### gir-web (v2.0.0) — spoke
- **Agents**: docs-fetcher, deploy-manager, ui-generator
- **Skills**: design-principles, frontend-design, web-tools
- **MCP**: v0, figma, vercel, context7, ref

### gir-tools (v2.0.0) — spoke
- **Agents**: team-lead, agenthub
- **Skills**: subtask, agenthub-session-management
- **MCP**: (none)
```

Using Markdown (not JSON) because:
- Human-readable when users open it
- Consistent with other `.gir/` files (all Markdown)
- Still parseable by Claude for programmatic lookups
- Lower token count than equivalent JSON

---

### Phase 4: Module Discovery Command

#### 4.1 Create `/gir-core:modules` command

**`plugins/gir-core/commands/modules.md`:**

```markdown
---
name: modules
description: List installed GIR modules and discover available ones
---

Read `.gir/GIR.modules` and display the currently installed modules in a clean table format.

Then inform the user about available official modules they haven't installed:

**Official GIR Modules:**

| Module | Description | Install |
|--------|-------------|---------|
| gir-core | Core agents, workflows, best practices (REQUIRED) | `claude plugin install gir-core` |
| gir-web | Frontend/fullstack agents and design skills | `claude plugin install gir-web` |
| gir-automation | n8n workflow builder agent | `claude plugin install gir-automation` |
| gir-tools | AgentHub + team coordination + subtask parallel execution | `claude plugin install gir-tools` |

Mark installed modules with a checkmark. Show their version and what they provide.

If `.gir/GIR.modules` doesn't exist, say the memory bank hasn't been initialized and suggest running `/gir-core:init-memory-bank` first.
```

---

### Phase 5: Marketplace & Documentation Updates

#### 5.1 Update `marketplace.json`

Add `dependencies` and `required` fields:

```json
{
  "name": "gir-plugins",
  "owner": { "name": "RedcoatAsher", "url": "https://github.com/RedcoatAsher" },
  "metadata": {
    "description": "GIR — Modular Claude Code productivity ecosystem",
    "version": "2.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "gir-core",
      "source": "./gir-core",
      "description": "Core agents, workflows, best practices, and auto-delegation for any Claude Code project",
      "version": "2.0.0",
      "category": "productivity",
      "required": true,
      "keywords": ["agents", "workflows", "best-practices", "delegation", "code-review"]
    },
    {
      "name": "gir-web",
      "source": "./gir-web",
      "description": "Frontend/fullstack agents and design skills",
      "version": "2.0.0",
      "category": "web-development",
      "dependencies": ["gir-core"],
      "keywords": ["frontend", "design", "deployment", "ui-generator"]
    },
    {
      "name": "gir-automation",
      "source": "./gir-automation",
      "description": "n8n workflow builder agent",
      "version": "2.0.0",
      "category": "automation",
      "dependencies": ["gir-core"],
      "keywords": ["n8n", "workflows", "automation"]
    },
    {
      "name": "gir-tools",
      "source": "./gir-tools",
      "description": "Power user tools — AgentHub, subtask parallel execution, team coordination",
      "version": "2.0.0",
      "category": "productivity",
      "dependencies": ["gir-core"],
      "keywords": ["agenthub", "subtask", "parallel", "multi-session"]
    }
  ]
}
```

#### 5.2 Update `README.md`

Rewrite the Install section to emphasize the modular experience:

```markdown
## Install

### Step 1: Add the marketplace
```bash
claude plugin marketplace add RedcoatAsher/GIR-v2
```

### Step 2: Install core (required)
```bash
claude plugin install gir-core
```

### Step 3: Install modules you need
```bash
# Frontend/fullstack development
claude plugin install gir-web

# n8n workflow automation
claude plugin install gir-automation

# AgentHub + team coordination
claude plugin install gir-tools
```

### Step 4: Discover modules
```
/gir-core:modules
```
```

Add a "Creating Custom Modules" section explaining the `gir-module.json` manifest format for third-party developers.

#### 5.3 Update `init-memory-bank` command

Ensure it creates the initial `.gir/GIR.modules` file with just the `gir-core` entry when initializing the memory bank.

---

## Phase 6: Third-Party Module Support

#### 6.1 Document the module contract

Any Claude Code plugin can become a GIR module by:

1. Including a `gir-module.json` at its plugin root
2. Adding a SessionStart hook that registers itself in `.gir/GIR.modules`
3. Declaring `"requires": ["gir-core"]` in its manifest

**Example** for a hypothetical `gir-supabase` third-party module:

```json
{
  "name": "gir-supabase",
  "version": "1.0.0",
  "type": "spoke",
  "description": "Supabase database management agent",
  "requires": ["gir-core"],
  "provides": {
    "agents": ["supabase-manager"],
    "skills": ["supabase-tools"],
    "commands": [],
    "mcp": {
      "supabase": {
        "description": "Supabase project and database management",
        "commands": ["search_docs", "list_projects", "list_tables", "execute_sql", "apply_migration", "deploy_edge_function"]
      }
    }
  }
}
```

The module's SessionStart hook registers itself using the same pattern as official spokes.

---

## File Change Summary

| File | Action | Description |
|------|--------|-------------|
| `plugins/gir-core/gir-module.json` | **Create** | Core module manifest |
| `plugins/gir-web/gir-module.json` | **Create** | Web module manifest |
| `plugins/gir-automation/gir-module.json` | **Create** | Automation module manifest |
| `plugins/gir-tools/gir-module.json` | **Create** | Tools module manifest |
| `plugins/gir-core/skills/core-practices/SKILL.md` | **Edit** | Remove spoke-specific MCP tool docs (~40 lines), add GIR.modules reference (~4 lines) |
| `plugins/gir-core/skills/workflows/SKILL.md` | **Edit** | Add "(requires gir-web)" guards to spoke-specific workflow references |
| `plugins/gir-web/skills/web-tools/SKILL.md` | **Create** | New skill: web MCP tool documentation (extracted from core) |
| `plugins/gir-automation/skills/n8n-tools/SKILL.md` | **Create** | New skill: n8n MCP tool documentation (extracted from core) |
| `plugins/gir-web/hooks/hooks.json` | **Create** | SessionStart hook for self-registration |
| `plugins/gir-automation/hooks/hooks.json` | **Create** | SessionStart hook for self-registration |
| `plugins/gir-tools/hooks/hooks.json` | **Create** | SessionStart hook for self-registration |
| `plugins/gir-core/hooks/hooks.json` | **Edit** | Enhance to manage GIR.modules registry |
| `plugins/gir-core/commands/modules.md` | **Create** | `/gir-core:modules` discovery command |
| `.claude-plugin/marketplace.json` | **Edit** | Add `dependencies` and `required` fields |
| `plugins/*/`.claude-plugin/plugin.json` | **Edit** | Add `dependencies` field to spoke plugin.json files |
| `README.md` | **Edit** | Update install instructions for modular experience |

**New files**: 9
**Modified files**: 7
**Deleted files**: 0

---

## Token Impact Estimate

| Scenario | Before (all bundled) | After (core only) | Savings |
|----------|---------------------|-------------------|---------|
| User with core only | ~220 lines core-practices + full workflows | ~140 lines core-practices + cleaned workflows | ~35% reduction per session |
| User with core + web | Same ~220 lines (already includes web tools) | ~140 core + ~30 web-tools skill | Similar total, but modular |
| User with core + automation | Same ~220 lines (already includes n8n) | ~140 core + ~10 n8n-tools skill | ~30% reduction |
| User with everything | Same ~220 lines | ~140 + ~30 + ~10 + spoke skills | Slight increase (skill overhead), but better organized |

The key win is for **core-only users** and **users with 1-2 spokes** — they no longer pay the token cost for modules they don't use.

---

## Migration Path

### For existing users

1. Update the marketplace: `claude plugin marketplace update RedcoatAsher/GIR-v2`
2. Core update brings the new hook + cleaned skills automatically
3. Installed spokes update and gain their own hooks
4. First session after update: hooks create `.gir/GIR.modules`
5. No action required from the user

### Breaking changes

- None for plugin behavior
- The `core-practices` skill no longer documents spoke-specific tools (this is intentional — they now live in spoke skills)
- Users who relied on core-practices listing all MCP tools as a reference should use `/gir-core:modules` instead

---

## Open Questions

1. **Supabase** — Should it stay in `gir-web` or become its own spoke (`gir-database`)? It's backend-focused but often paired with web projects.

2. **Documentation tools** (Context7, Ref, exa) — Currently in core-practices. These are generic enough for core but primarily used during web development. Keep in core or move to gir-web?

3. **CodeRabbit, Jules, claude-mem** — These are external tools, not GIR-specific. Remove from core entirely and let users configure them independently? Or create a `gir-integrations` spoke?

4. **Gemini-CLI** — Currently documented in core as a delegation target. Keep in core? It's a general-purpose tool but not universally available.

5. **Hook execution order** — Claude Code fires SessionStart hooks in plugin install order. If a spoke hook fires before core's hook, `.gir/GIR.modules` might not exist yet. The spoke hook prompt says "if .gir/ directory exists" which handles this — but should we guarantee core fires first?

6. **Module versioning** — Should the GIR.modules file track version compatibility? E.g., "gir-web 2.0.0 requires gir-core >=2.0.0"?

---

## Implementation Order

```
Phase 1 (Foundation)     → gir-module.json manifests + plugin.json dependencies
Phase 2 (Core cleanup)   → Extract spoke content from core skills
Phase 3 (Registration)   → SessionStart hooks + GIR.modules generation
Phase 4 (Discovery)      → /gir-core:modules command
Phase 5 (Documentation)  → README, marketplace.json, migration guide
Phase 6 (Extensibility)  → Third-party module documentation
```

Phases 1-2 can be done in parallel. Phase 3 depends on Phase 1. Phases 4-6 can be done in any order after Phase 3.
