# GIR v2

> Modular Claude Code plugin ecosystem. Install only what you need — curated agents, auto-delegation, memory bank, and domain-specific tooling.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Why Modular?

GIR is designed so you **only pay the token cost for what you use**. Install `gir-core`, then add only the spokes your project needs. No wasted context, no irrelevant tools loaded into every session.

| Setup | Token Savings vs. Monolithic |
|-------|------------------------------|
| Core only | **~45% reduction** |
| Core + 1 spoke | ~35–40% reduction |
| Core + 2–3 spokes | ~25–35% reduction |
| All 7 modules | Comparable total, but fully modular |

Each module self-registers at session start. Installed modules list their tools in `.gir/GIR.modules`. Uninstalled modules are invisible — zero token overhead.

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
    ┌────────────┬───────────────┼───────────────┬────────────┬────────────┐
    │            │               │               │            │            │
┌───▼───┐  ┌────▼────┐  ┌───────▼──────┐  ┌────▼────┐  ┌────▼───┐  ┌────▼───┐
│gir-web│  │gir-auto-│  │  gir-tools   │  │gir-data-│  │ gir-ai │  │ gir-qa │
│       │  │ mation  │  │              │  │  base   │  │        │  │        │
│• docs │  │         │  │ • agenthub   │  │         │  │• Gemini│  │• Code- │
│• deploy│ │ • n8n-  │  │ • team-lead  │  │• Supa-  │  │  -CLI  │  │  Rabbit│
│• ui-  │  │   builder│ │ • subtask    │  │  base   │  │• Codex │  │• Jules │
│  gen  │  │         │  │   skills     │  │  tools  │  │        │  │        │
└───────┘  └─────────┘  └──────────────┘  └─────────┘  └────────┘  └────────┘
```

---

## Install

### Step 1: Add the marketplace

```bash
claude plugin marketplace add RedcoatAsher/GIR-v2
```

### Step 2: Install core (required)

```bash
claude plugin install gir-core
```

### Step 3: Install the modules you need

```bash
claude plugin install gir-web           # Frontend/fullstack (v0, Figma, Vercel)
claude plugin install gir-automation    # n8n workflow automation
claude plugin install gir-tools         # AgentHub + team coordination
claude plugin install gir-database      # Database management (Supabase)
claude plugin install gir-ai            # AI delegation (Gemini-CLI, Codex)
claude plugin install gir-qa            # QA & review (CodeRabbit, Jules)
```

### Step 4: Discover modules

```
/gir-core:modules
```

---

## Modules

| Module | Description | Includes | Who needs it |
|--------|-------------|----------|--------------|
| [gir-core](plugins/gir-core/) | Core hub. Agents, delegation, workflows, memory bank, slash commands | 5 agents, 6 skills, 4 commands, SessionStart hook, sequential-thinking MCP | Everyone |
| [gir-web](plugins/gir-web/) | Frontend and fullstack tooling — v0, Figma, Vercel | 3 agents, 3 skills | Frontend/fullstack devs |
| [gir-automation](plugins/gir-automation/) | n8n workflow building | 1 agent, 1 skill | Teams using n8n |
| [gir-tools](plugins/gir-tools/) | AgentHub integration and agent team coordination | 2 agents, 2 skills | Power users running parallel agent workflows |
| [gir-database](plugins/gir-database/) | Database management — Supabase | 1 skill | Projects using Supabase |
| [gir-ai](plugins/gir-ai/) | AI tool delegation — Gemini-CLI, Codex | 1 skill | Users with external AI tools |
| [gir-qa](plugins/gir-qa/) | QA & review tools — CodeRabbit, Jules | 1 skill | Teams using automated code review |

---

## What stays in your project

GIR plugins install globally. Your project keeps its own configuration:

- **`CLAUDE-project.md`** — Project-specific tech stack, commands, and conventions. GIR reads this automatically.
- **`.gir/`** — Memory bank directory. GIR writes session context, patterns, decisions, and troubleshooting notes here across sessions.
- **`.gir/GIR.modules`** — Auto-generated registry of installed modules. Updated each session by module hooks.

These files are yours. GIR does not overwrite them.

---

## Creating Custom Modules

Any Claude Code plugin can become a GIR module by including a `gir-module.json` manifest, a SessionStart hook for self-registration, and declaring `"requires": ["gir-core"]`. See [PLAN-modularization.md](PLAN-modularization.md#phase-6-third-party-module-support) for the full module contract.

---

## License

[MIT](LICENSE) — RedcoatAsher, 2026.
