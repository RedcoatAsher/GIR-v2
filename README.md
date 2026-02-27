# GIR v2

> Drop-in Claude Code plugin ecosystem. Curated agents, auto-delegation, memory bank, and design/deployment tooling — structured as a hub-and-spoke set of plugins.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Architecture

GIR uses a **hub-and-spoke** model. `gir-core` is required and provides the foundation. Spoke plugins are optional and extend it for specific domains.

```
                        ┌─────────────────┐
                        │    gir-core     │  ← REQUIRED
                        │                 │
                        │  agents:        │
                        │  • feature-architect
                        │  • code-reviewer│
                        │  • debugger     │
                        │  • subtask-mgr  │
                        │  • spec-analyst │
                        │                 │
                        │  skills + cmds  │
                        │  hooks + MCP    │
                        └────────┬────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
   ┌──────▼──────┐        ┌──────▼──────┐       ┌──────▼──────┐
   │   gir-web   │        │gir-automation│       │  gir-tools  │
   │             │        │             │       │             │
   │ • docs-     │        │ • n8n-      │       │ • agenthub  │
   │   fetcher   │        │   builder   │       │ • team-lead │
   │ • deploy-   │        │             │       │ • subtask   │
   │   manager   │        └─────────────┘       │   skills    │
   │ • ui-       │                              └─────────────┘
   │   generator │
   └─────────────┘
```

---

## Install

```bash
claude plugin marketplace add RedcoatAsher/GIR-v2

claude plugin install gir-core          # Required — install first
claude plugin install gir-web           # Optional — frontend/fullstack
claude plugin install gir-automation    # Optional — n8n workflows
claude plugin install gir-tools         # Optional — AgentHub + team tooling
```

---

## Plugins

| Plugin | Description | Includes | Who needs it |
|--------|-------------|----------|--------------|
| [gir-core](plugins/gir-core/) | Core hub. Agents, delegation, workflows, memory bank, slash commands | 5 agents, 6 skills, 4 commands, SessionStart hook, sequential-thinking MCP | Everyone |
| [gir-web](plugins/gir-web/) | Frontend and fullstack tooling | 3 agents, 2 skills | Frontend/fullstack devs |
| [gir-automation](plugins/gir-automation/) | n8n workflow building | 1 agent | Teams using n8n |
| [gir-tools](plugins/gir-tools/) | AgentHub integration and agent team coordination | 2 agents, 2 skills | Power users running parallel agent workflows |

---

## What stays in your project

GIR plugins install globally. Your project keeps its own configuration:

- **`CLAUDE-project.md`** — Project-specific tech stack, commands, and conventions. GIR reads this automatically.
- **`.gir/`** — Memory bank directory. GIR writes session context, patterns, decisions, and troubleshooting notes here across sessions.

These files are yours. GIR does not overwrite them.

---

## License

[MIT](LICENSE) — RedcoatAsher, 2026.
