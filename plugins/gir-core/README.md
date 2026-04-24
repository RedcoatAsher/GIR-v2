# gir-core

> The required hub of the GIR plugin ecosystem. Provides agents, routing-stub orchestration, workflow skills, slash commands, a SessionStart hook, and the sequential-thinking MCP server.

---

## Install

```bash
claude plugin install gir-core
```

---

## Agents

Five specialist agents are registered and available for delegation or direct invocation.

| Agent | Role |
|-------|------|
| **feature-architect** | Breaks down features into tasks, writes specs, plans implementation sequences |
| **code-reviewer** | Reviews diffs and files for correctness, clarity, security, and test coverage |
| **debugger** | Investigates errors, traces root causes, proposes and verifies fixes |
| **team-lead** | Orchestrates agent teams for complex, multi-part work. Delegates tasks, enforces quality gates, and coordinates teammates |
| **spec-analyst** | Reads spec documents and translates them into actionable implementation plans |

---

## Skills

Three skills are always loaded. Three are available on demand via `/load-*` commands.

**Always loaded:**

| Skill | What it activates |
|-------|------------------|
| **routing-stub** | Orchestration core — routing rules, delegation tiers, Graphify integration, escalation trigger |
| **core-practices** | Universal coding standards, commit hygiene, error handling patterns |
| **state-machines** | Structured state/transition modeling for complex feature logic |

**On demand:**

| Skill | Load command | What it activates |
|-------|-------------|------------------|
| **workflows** | `/gir-core:load-workflows` | Multi-step coordination patterns for complex features, reviews, and releases |
| **ralph-loops** | `/gir-core:load-ralph` | Continuous iteration protocol — plan, execute, verify, repeat until done |
| **specgates** | `/gir-core:load-specgates` | Spec-driven development gates; blocks implementation until spec is confirmed |

---

## Slash Commands

| Command | Usage |
|---------|-------|
| `/init-project` | Scaffolds `CLAUDE-project.md` for a new or existing project |
| `/init-memory-bank` | Creates the `.gir/` memory bank directory with starter files |
| `/drift-check` | Compares current code state against the active spec; surfaces divergence |
| `/status` | Prints active context, current session goals, and memory bank summary |
| `/modules` | Lists all installed GIR modules and their registered tools |
| `/load-workflows` | Loads the workflows skill on demand for multi-step or delegated work |
| `/load-specgates` | Loads the specgates skill on demand for new features or scope changes |
| `/load-ralph` | Loads the ralph-loops skill on demand for iterative refinement work |

---

## SessionStart Hook

gir-core installs a `SessionStart` hook that runs automatically when a Claude Code session begins. It reads `.gir/MISSION.md` (active priorities), `.gir/CLAUDE-activeContext.md` (session state), and `.gir/ESCALATION.md` (must-escalate conditions) if present — restoring full operational context without manual prompting.

---

## sequential-thinking MCP

gir-core bundles the `sequential-thinking` MCP server. This gives Claude a structured tool for breaking down multi-step problems before executing them, reducing planning errors on complex tasks.

No additional configuration is required — it runs as part of the plugin.

---

## Setup notes

**team-lead agent**: Add the following to your Claude Code `settings.json` to enable agent teams:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

---

## Works best with

- [gir-web](../gir-web/) — adds frontend/fullstack agents and design skills
- [gir-automation](../gir-automation/) — adds n8n workflow building
- [gir-tools](../gir-tools/) — adds AgentHub integration and agent team coordination
- [gir-database](../gir-database/) — adds Supabase database management tools
- [gir-ai](../gir-ai/) — adds AI tool delegation patterns (Gemini-CLI, Codex)
- [gir-qa](../gir-qa/) — adds QA and review tools (CodeRabbit, Jules)

---

## License

[MIT](LICENSE) — rivit-studio, 2026.
