# GIR v2 — Modularization Plan

> Split spokes into independent, marketplace-installable modules with a dynamic registry system.

**Status**: Implementation Complete (as of 2026-03-03)
**Date**: 2026-02-28
**Author**: rivit-studio + Claude

---

## Problem Statement

GIR v2 uses a hub-and-spoke architecture, but today all spokes ship in a single monorepo marketplace. When a user adds the marketplace and installs `gir-core`, the core-practices skill alone loads ~220 lines of context that includes MCP tool documentation for **every spoke** (n8n, Vercel, Figma, v0, Supabase, CodeRabbit, Jules). This burns tokens on every prompt regardless of whether the user has those spokes installed or even uses those tools.

**Specific waste in `core-practices/SKILL.md`:**

| Lines | Content | Belongs in |
|-------|---------|-----------|
| 46-52 | Context7, Ref, exa, fetch docs | Core (generic documentation tools) |
| 54-67 | Gemini-CLI delegation | `gir-ai` (new spoke) |
| 69-82 | Sequential-Thinking | Core (universal) |
| 87-92 | v0, Figma UI generation | `gir-web` |
| 94-95 | Vercel MCP tools | `gir-web` |
| 96-98 | Supabase MCP tools | `gir-database` (new spoke) |
| 99-103 | n8n MCP tools + skills | `gir-automation` |
| 105-107 | CodeRabbit, Jules, claude-mem | `claude-mem` stays in core; CodeRabbit + Jules → `gir-qa` (new spoke) |
| 176-179 | Agent listing (Web/Automation) | Respective spokes |

Additionally, `workflows/SKILL.md` references spoke-specific tools in its decision matrix (line 302: "Figma, v0" for design implementation) and commit phase (line 183: "deploy to Vercel").

---

## Goals

1. **Token efficiency** — Only installed module context loads per session
2. **True modularity** — Users install `gir-core`, then choose their spokes
3. **Dynamic registry** — A `GIR.modules` file that auto-updates when modules are installed/removed
4. **Extensibility** — Third-party / unofficial modules can self-register using the same system
5. **Monorepo development** — Keep all official plugins in the GIR repo for development convenience

---

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Repo structure | Stay monorepo | Easier to develop, test, and version together |
| Core scope | Keep all 5 agents + 6 skills as required | Core's value is the unified skill set; don't fragment it |
| Registry location | `.gir/GIR.modules` (project-local) | Consistent with existing `.gir/` memory bank pattern |
| Registration mechanism | Self-registering SessionStart hooks per spoke | Each module registers itself; no central scanner needed |
| Concurrent write safety | Each spoke hook reads-then-writes atomically (prompt-level lock via sequential Claude execution) | Claude processes prompts sequentially within a session; hooks don't execute in parallel threads, so simultaneous file corruption is not a concern in practice. If parallel hook execution is introduced in a future Claude version, migrate to a per-module file approach (`.gir/modules.d/gir-web.json`) with core aggregating them. |
| Manifest format | `gir-module.json` per plugin | Standard contract for official + unofficial modules |
| Stale entry cleanup | Spoke hooks self-manage their entries; core does NOT prune | GIR plugins are installed globally (not as project files), so filesystem checks would produce false positives. Stale entries from uninstalled spokes are harmless. Use `/gir-core:modules` to manually review. |

---

## Architecture

### Current (everything coupled)

```text
SessionStart hook fires
  └─ Core loads ALL context (including spoke MCP tool docs)
     └─ ~220 lines of core-practices, many irrelevant to user's project
```

### Target (modular, self-registering)

```text
SessionStart hooks fire (one per installed module)
  ├─ gir-core hook:
  │   ├─ Restore .gir/CLAUDE-activeContext.md
  │   ├─ Ensure .gir/GIR.modules exists with gir-core entry
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
  ├─ gir-tools hook (only if installed):
  │   ├─ Register in .gir/GIR.modules
  │   └─ Team/subtask context loaded via its own skills
  │
  ├─ gir-database hook (only if installed):
  │   ├─ Register in .gir/GIR.modules
  │   └─ Supabase tool context loaded via its own skills
  │
  ├─ gir-ai hook (only if installed):
  │   ├─ Register in .gir/GIR.modules
  │   └─ Gemini-CLI, Codex, and other AI delegation tools
  │
  └─ gir-qa hook (only if installed):
      ├─ Register in .gir/GIR.modules
      └─ CodeRabbit, Jules, and QA/review tool context
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
      "team-lead",
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
    "skills": ["design-principles", "frontend-design", "web-tools"],
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
    "skills": ["n8n-tools"],
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
    "agents": ["subtask-manager", "agenthub"],
    "skills": ["subtask", "agenthub-session-management"],
    "commands": [],
    "mcp": {}
  }
}
```

**`plugins/gir-database/gir-module.json`** (new spoke):
```json
{
  "name": "gir-database",
  "version": "2.0.0",
  "type": "spoke",
  "description": "Database management agents and tools — Supabase",
  "requires": ["gir-core"],
  "provides": {
    "agents": [],
    "skills": ["database-tools"],
    "commands": [],
    "mcp": {
      "supabase": {
        "description": "Supabase project and database management",
        "commands": ["search_docs", "list_projects", "list_tables", "execute_sql", "apply_migration", "deploy_edge_function", "get_logs", "get_advisors", "create_branch", "merge_branch"]
      }
    }
  }
}
```

**`plugins/gir-ai/gir-module.json`** (new spoke):
```json
{
  "name": "gir-ai",
  "version": "2.0.0",
  "type": "spoke",
  "description": "AI tool integrations — Gemini-CLI, Codex, and other AI delegation targets",
  "requires": ["gir-core"],
  "provides": {
    "agents": [],
    "skills": ["ai-delegation"],
    "commands": [],
    "mcp": {}
  }
}
```

**`plugins/gir-qa/gir-module.json`** (new spoke):
```json
{
  "name": "gir-qa",
  "version": "2.0.0",
  "type": "spoke",
  "description": "QA and review tools — CodeRabbit PR reviews, Jules AI delegation",
  "requires": ["gir-core"],
  "provides": {
    "agents": [],
    "skills": ["qa-tools"],
    "commands": [],
    "mcp": {
      "coderabbit": {
        "description": "Automated PR reviews",
        "commands": ["get_coderabbit_reviews", "get_review_details", "resolve_comment"]
      },
      "jules": {
        "description": "AI task delegation",
        "commands": ["create_session", "get_session", "send_session_message"]
      }
    }
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

- **Lines 54-67** — Gemini-CLI delegation → move to `gir-ai/skills/ai-delegation/SKILL.md`
- **Lines 87-92** — UI Generation (v0, Figma) → move to `gir-web/skills/web-tools/SKILL.md`
- **Lines 94-95** — Vercel MCP → move to `gir-web/skills/web-tools/SKILL.md`
- **Lines 96-98** — Supabase MCP → move to `gir-database/skills/database-tools/SKILL.md`
- **Lines 99-103** — n8n MCP + skills → move to `gir-automation/skills/n8n-tools/SKILL.md`
- **Lines 105-106** — CodeRabbit, Jules → move to `gir-qa/skills/qa-tools/SKILL.md`
- **Lines 176-179** — Agent listing for Web/Automation categories → remove from core

**Keep in core** (universal tools):

- **Lines 46-52** — Context7, Ref, exa, fetch (generic documentation lookup)
- **Lines 69-82** — Sequential-Thinking (core MCP dependency)
- **Line 107** — claude-mem (memory tool stays in core)

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
description: MCP tool reference for web development — v0, Figma, Vercel.
  Apply when doing UI generation or deployment for frontend work.
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

**`plugins/gir-database/skills/database-tools/SKILL.md`** (new):

```markdown
---
name: database-tools
description: MCP tool reference for database management — Supabase.
  Apply when managing database schemas, migrations, edge functions, or querying data.
---
# Database Tools

## Supabase MCP
`search_docs`, `list_projects`, `list_tables`, `execute_sql`, `apply_migration`,
`deploy_edge_function`, `get_logs`, `get_advisors`, `create_branch`, `merge_branch`
```

**`plugins/gir-ai/skills/ai-delegation/SKILL.md`** (new):

```markdown
---
name: ai-delegation
description: AI tool delegation patterns — Gemini-CLI, Codex, and other external AI tools.
  Apply when delegating tasks to external AI tools to save tokens.
---
# AI Tool Delegation

## Gemini-CLI Delegation

Offload to save tokens:

DELEGATE: File analysis (>200 lines), code review, test gen, refactoring,
regex, boilerplate, data transforms, explaining code

KEEP IN MAIN: Quick edits (<50 lines), direct Q&A, architecture decisions,
final implementations

## Other AI Tools

Additional AI delegation targets can be configured here as they become available
(e.g., Codex, other CLI-based AI tools).
```

**`plugins/gir-qa/skills/qa-tools/SKILL.md`** (new):

```markdown
---
name: qa-tools
description: QA and code review tool integrations — CodeRabbit PR reviews, Jules AI delegation.
  Apply when doing code reviews, PR analysis, or delegating QA tasks.
---
# QA & Review Tools

## CodeRabbit
Automated PR reviews: `get_coderabbit_reviews`, `get_review_details`, `resolve_comment`

## Jules
AI task delegation: `create_session`, `get_session`, `send_session_message`
```

#### 2.3 Clean up `workflows/SKILL.md`

- **Line 183** — Remove "deploy to Vercel" from Commit phase. Replace with: "deploy if configured"
- **Line 219** — "Visual Iteration Workflow" section references v0/Figma tools. Add a guard: "Requires gir-web module"
- **Lines 302-303** — Decision matrix references "Figma, v0". Add note: "(requires gir-web)"

These are lightweight annotations, not full content extraction. The detailed tool docs are already in the spoke skills.

---

### Phase 3: Self-Registering Module System

#### 3.1 Add SessionStart hooks to each spoke

Each spoke gets a `hooks/hooks.json` that registers the module in `.gir/GIR.modules` at session start. All hooks use the same Markdown entry format (defined in §3.3):

```
### <module-name> (v<version>) — <type>
- **Agents**: <comma-separated agents>
- **Skills**: <comma-separated skills>
- **Commands**: <comma-separated commands or "(none)">
- **MCP**: <comma-separated mcp keys or "(none)">
```

Hook prompts reference this format explicitly to prevent inconsistencies across entries.

**`plugins/gir-web/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-web is listed. If gir-web is missing, append the following entry exactly as shown (using this Markdown format):\n\n### gir-web (v2.0.0) — spoke\n- **Agents**: docs-fetcher, deploy-manager, ui-generator\n- **Skills**: design-principles, frontend-design, web-tools\n- **Commands**: (none)\n- **MCP**: v0, figma, vercel, context7, ref\n\nWrite the updated file. Do this silently without telling the user."
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
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-automation is listed. If gir-automation is missing, append the following entry exactly as shown:\n\n### gir-automation (v2.0.0) — spoke\n- **Agents**: n8n-builder\n- **Skills**: n8n-tools\n- **Commands**: (none)\n- **MCP**: n8n-mcp\n\nWrite the updated file. Do this silently without telling the user."
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
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-tools is listed. If gir-tools is missing, append the following entry exactly as shown:\n\n### gir-tools (v2.0.0) — spoke\n- **Agents**: subtask-manager, agenthub\n- **Skills**: subtask, agenthub-session-management\n- **Commands**: (none)\n- **MCP**: (none)\n\nWrite the updated file. Do this silently without telling the user."
          }
        ]
      }
    ]
  }
}
```

**`plugins/gir-database/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-database is listed. If gir-database is missing, append the following entry exactly as shown:\n\n### gir-database (v2.0.0) — spoke\n- **Agents**: (none)\n- **Skills**: database-tools\n- **Commands**: (none)\n- **MCP**: supabase\n\nWrite the updated file. Do this silently without telling the user."
          }
        ]
      }
    ]
  }
}
```

**`plugins/gir-ai/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-ai is listed. If gir-ai is missing, append the following entry exactly as shown:\n\n### gir-ai (v2.0.0) — spoke\n- **Agents**: (none)\n- **Skills**: ai-delegation\n- **Commands**: (none)\n- **MCP**: (none)\n\nWrite the updated file. Do this silently without telling the user."
          }
        ]
      }
    ]
  }
}
```

**`plugins/gir-qa/hooks/hooks.json`:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If .gir/ directory exists in the current project, check if .gir/GIR.modules exists. If it does, read it and ensure gir-qa is listed. If gir-qa is missing, append the following entry exactly as shown:\n\n### gir-qa (v2.0.0) — spoke\n- **Agents**: (none)\n- **Skills**: qa-tools\n- **Commands**: (none)\n- **MCP**: coderabbit, jules\n\nWrite the updated file. Do this silently without telling the user."
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
            "prompt": "Check if .gir/CLAUDE-activeContext.md exists in the current project. If it does, read it to restore session context. If it doesn't, briefly mention that the user can run /gir-core:init-memory-bank to set up a memory bank. Then, if .gir/GIR.modules exists, read it and ensure gir-core is listed using this exact format:\n\n### gir-core (v2.0.0) — core\n- **Agents**: feature-architect, code-reviewer, debugger, team-lead, spec-analyst\n- **Skills**: auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines\n- **Commands**: init-project, init-memory-bank, drift-check, status, modules, migrate-v1\n- **MCP**: sequential-thinking\n\nDo not remove spoke entries — an uninstalled spoke's hook no longer runs, so its entry will simply persist as stale. Stale entries are harmless; users can remove them manually or via /gir-core:modules. If .gir/GIR.modules doesn't exist and .gir/ directory exists, create it with just the gir-core entry above. Do the module registry work silently."
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
- **Agents**: feature-architect, code-reviewer, debugger, team-lead, spec-analyst
- **Skills**: auto-delegation, core-practices, workflows, ralph-loops, specgates, state-machines
- **Commands**: init-project, init-memory-bank, drift-check, status, modules, migrate-v1
- **MCP**: sequential-thinking

### gir-web (v2.0.0) — spoke
- **Agents**: docs-fetcher, deploy-manager, ui-generator
- **Skills**: design-principles, frontend-design, web-tools
- **Commands**: (none)
- **MCP**: v0, figma, vercel, context7, ref

### gir-tools (v2.0.0) — spoke
- **Agents**: subtask-manager, agenthub
- **Skills**: subtask, agenthub-session-management
- **Commands**: (none)
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
| gir-database | Database management — Supabase | `claude plugin install gir-database` |
| gir-ai | AI tool delegation — Gemini-CLI, Codex | `claude plugin install gir-ai` |
| gir-qa | QA & review tools — CodeRabbit, Jules | `claude plugin install gir-qa` |

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
  "owner": { "name": "rivit-studio", "url": "https://github.com/rivit-studio" },
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
    },
    {
      "name": "gir-database",
      "source": "./gir-database",
      "description": "Database management — Supabase MCP tools",
      "version": "2.0.0",
      "category": "database",
      "dependencies": ["gir-core"],
      "keywords": ["supabase", "database", "sql", "migrations"]
    },
    {
      "name": "gir-ai",
      "source": "./gir-ai",
      "description": "AI tool delegation — Gemini-CLI, Codex, and other AI integrations",
      "version": "2.0.0",
      "category": "ai-tools",
      "dependencies": ["gir-core"],
      "keywords": ["gemini", "codex", "ai", "delegation"]
    },
    {
      "name": "gir-qa",
      "source": "./gir-qa",
      "description": "QA & review tools — CodeRabbit PR reviews, Jules AI delegation",
      "version": "2.0.0",
      "category": "quality",
      "dependencies": ["gir-core"],
      "keywords": ["coderabbit", "jules", "code-review", "qa"]
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
claude plugin marketplace add rivit-studio/GIR
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

# Database management (Supabase)
claude plugin install gir-database

# AI tool delegation (Gemini-CLI, Codex)
claude plugin install gir-ai

# QA & review tools (CodeRabbit, Jules)
claude plugin install gir-qa
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

**Example** for a hypothetical `gir-terraform` third-party module:

```json
{
  "name": "gir-terraform",
  "version": "1.0.0",
  "type": "spoke",
  "description": "Terraform infrastructure-as-code management",
  "requires": ["gir-core"],
  "provides": {
    "agents": ["terraform-planner"],
    "skills": ["terraform-patterns"],
    "commands": [],
    "mcp": {
      "terraform": {
        "description": "Terraform plan, apply, and state management",
        "commands": ["plan", "apply", "show", "state_list"]
      }
    }
  }
}
```

The module's SessionStart hook registers itself using the same pattern as official spokes.

---

## File Change Summary

### New plugin directories (3 new spokes)

| Directory | Description |
|-----------|-------------|
| `plugins/gir-database/` | New spoke: Supabase database management |
| `plugins/gir-ai/` | New spoke: Gemini-CLI, Codex, AI delegation |
| `plugins/gir-qa/` | New spoke: CodeRabbit, Jules, QA/review tools |

### New files

| File | Description |
|------|-------------|
| `plugins/gir-core/gir-module.json` | Core module manifest |
| `plugins/gir-web/gir-module.json` | Web module manifest |
| `plugins/gir-automation/gir-module.json` | Automation module manifest |
| `plugins/gir-tools/gir-module.json` | Tools module manifest |
| `plugins/gir-database/gir-module.json` | Database module manifest |
| `plugins/gir-ai/gir-module.json` | AI module manifest |
| `plugins/gir-qa/gir-module.json` | QA module manifest |
| `plugins/gir-web/skills/web-tools/SKILL.md` | Web MCP tool docs (extracted from core) |
| `plugins/gir-automation/skills/n8n-tools/SKILL.md` | n8n MCP tool docs (extracted from core) |
| `plugins/gir-database/skills/database-tools/SKILL.md` | Supabase tool docs (extracted from core) |
| `plugins/gir-ai/skills/ai-delegation/SKILL.md` | AI delegation patterns (extracted from core) |
| `plugins/gir-qa/skills/qa-tools/SKILL.md` | QA tool docs (extracted from core) |
| `plugins/gir-web/hooks/hooks.json` | SessionStart hook for self-registration |
| `plugins/gir-automation/hooks/hooks.json` | SessionStart hook for self-registration |
| `plugins/gir-tools/hooks/hooks.json` | SessionStart hook for self-registration |
| `plugins/gir-database/hooks/hooks.json` | SessionStart hook for self-registration |
| `plugins/gir-ai/hooks/hooks.json` | SessionStart hook for self-registration |
| `plugins/gir-qa/hooks/hooks.json` | SessionStart hook for self-registration |
| `plugins/gir-core/commands/modules.md` | `/gir-core:modules` discovery command |
| `plugins/gir-database/.claude-plugin/plugin.json` | Database plugin metadata |
| `plugins/gir-ai/.claude-plugin/plugin.json` | AI plugin metadata |
| `plugins/gir-qa/.claude-plugin/plugin.json` | QA plugin metadata |

### Modified files

| File | Description |
|------|-------------|
| `plugins/gir-core/skills/core-practices/SKILL.md` | Remove spoke-specific MCP tool docs (~50 lines), add GIR.modules reference (~4 lines) |
| `plugins/gir-core/skills/workflows/SKILL.md` | Add "(requires gir-web)" guards to spoke-specific workflow references |
| `plugins/gir-core/hooks/hooks.json` | Enhance to manage GIR.modules registry |
| `.claude-plugin/marketplace.json` | Add 3 new spokes + `dependencies` and `required` fields |
| `plugins/gir-web/.claude-plugin/plugin.json` | Add `dependencies` field |
| `plugins/gir-automation/.claude-plugin/plugin.json` | Add `dependencies` field |
| `plugins/gir-tools/.claude-plugin/plugin.json` | Add `dependencies` field |
| `README.md` | Update install instructions + add module table for 7 spokes |

**New files**: 22
**Modified files**: 8
**Deleted files**: 0

---

## Token Impact Estimate

| Scenario | Before (all bundled) | After (modular) | Savings |
|----------|---------------------|-----------------|---------|
| Core only | ~220 lines core-practices + full workflows | ~120 lines core-practices + cleaned workflows | **~45% reduction** |
| Core + web | Same ~220 lines | ~120 core + ~20 web-tools | ~35% reduction |
| Core + automation | Same ~220 lines | ~120 core + ~10 n8n-tools | ~40% reduction |
| Core + web + database | Same ~220 lines | ~120 core + ~20 web + ~10 database | ~30% reduction |
| All 7 modules | Same ~220 lines | ~120 + all spoke skills (~80 lines total) | ~10-16% token overhead (~19-32K tokens); fully modular |

The key wins:
- **Core-only users** save ~45% — Gemini-CLI delegation, all MCP tool docs, and all spoke agent listings are gone
- **Users with 1-3 spokes** save 25-40% — only the tools they use are loaded
- **claude-mem stays in core** — memory is a universal concern, no overhead for most users
- **Context7/Ref/exa stay in core** — documentation lookup is universal, saves users from needing gir-web just for docs

---

## Migration Path

### For existing users

1. Update the marketplace: `claude plugin marketplace update rivit-studio/GIR`
2. Core update brings the new hook + cleaned skills automatically
3. Installed spokes update and gain their own hooks
4. First session after update: hooks create `.gir/GIR.modules`
5. No action required from the user

### Breaking changes

- None for plugin behavior
- The `core-practices` skill no longer documents spoke-specific tools (this is intentional — they now live in spoke skills)
- Users who relied on core-practices listing all MCP tools as a reference should use `/gir-core:modules` instead

---

## Resolved Questions

| # | Question | Decision |
|---|----------|----------|
| 1 | **Supabase** — Stay in `gir-web` or own spoke? | **New spoke: `gir-database`** — Supabase is backend/database focused; give it its own module. |
| 2 | **Documentation tools** (Context7, Ref, exa) — Core or gir-web? | **Keep in core** — These are generic and useful across all project types (Python, Go, Rust, etc.). |
| 3 | **CodeRabbit, Jules, claude-mem** — Where do they go? | **`claude-mem` stays in core.** CodeRabbit + Jules → **new spoke: `gir-qa`** (QA/review tooling). |
| 4 | **Gemini-CLI** — Keep in core or make optional? | **New spoke: `gir-ai`** — Contains Gemini-CLI, Codex, and other AI tool integrations users may want. |
| 5 | **Hook execution order** — What if a spoke fires before core? | **Guard clause** — Each spoke hook checks `if .gir/ exists` before writing. If core hasn't fired yet, the spoke silently skips. Catches up next session. |
| 6 | **Module versioning** — Track compatibility in GIR.modules? | **Track versions, don't enforce** — Informational only. No runtime semver checking for now. |

---

## Implementation Order

```
Phase 1 (Foundation)     → gir-module.json manifests + plugin.json dependencies
                           Create 3 new spoke plugin directories (gir-database, gir-ai, gir-qa)
Phase 2 (Core cleanup)   → Extract spoke content from core skills into spoke-owned skills
                           Move Gemini-CLI → gir-ai, Supabase → gir-database,
                           CodeRabbit/Jules → gir-qa, v0/Figma/Vercel → gir-web
Phase 3 (Registration)   → SessionStart hooks for all 6 spokes + enhanced core hook
Phase 4 (Discovery)      → /gir-core:modules command
Phase 5 (Documentation)  → README, marketplace.json (7 plugins total), migration guide
Phase 6 (Extensibility)  → Third-party module documentation + gir-module.json schema docs
```

Phases 1-2 can be done in parallel. Phase 3 depends on Phase 1. Phases 4-6 can be done in any order after Phase 3.

**Total plugins after implementation: 7** (1 core + 6 spokes)

```
gir-core (required) ─┬─ gir-web (frontend/fullstack)
                      ├─ gir-automation (n8n)
                      ├─ gir-tools (AgentHub/teams)
                      ├─ gir-database (Supabase)
                      ├─ gir-ai (Gemini-CLI/Codex)
                      └─ gir-qa (CodeRabbit/Jules)
```
