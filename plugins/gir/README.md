# gir

> The required hub of the GIR plugin ecosystem. Provides agents, routing-stub orchestration, workflow skills, slash commands, a SessionStart hook, and the sequential-thinking MCP server.

---

## Install

```bash
claude plugin install gir
```

---

## Agents

Seven specialist agents are registered and available for delegation or direct invocation.

| Agent | Role |
|-------|------|
| **feature-architect** | Breaks down features into tasks, writes specs, plans implementation sequences |
| **code-reviewer** | Reviews diffs and files for correctness, clarity, security, and test coverage |
| **debugger** | Investigates errors, traces root causes, proposes and verifies fixes |
| **team-lead** | Orchestrates agent teams for complex, multi-part work. Delegates tasks, enforces quality gates, and coordinates teammates |
| **spec-analyst** | Reads spec documents and translates them into actionable implementation plans |
| **parallel-orchestrator** | Decomposes tasks into independent workstreams and dispatches concurrent agents with worktree isolation |
| **autonomous-executor** | Executes MISSION.md plans autonomously — checkpoints state, enforces DOD/ESCALATION gates, resumes after usage-cap interruptions |

---

## Skills

Three skills are always loaded. Five are available on demand.

**Always loaded:**

| Skill | What it activates |
|-------|------------------|
| **routing-stub** | Orchestration core — routing rules, delegation tiers, Graphify integration, escalation trigger, parallel trigger phrases |
| **core-practices** | Universal coding standards, commit hygiene, error handling patterns |
| **state-machines** | Structured state/transition modeling for complex feature logic |

**On demand:**

| Skill | Load command | What it activates |
|-------|-------------|------------------|
| **workflows** | `/gir:load-workflows` | Multi-step coordination patterns for complex features, reviews, and releases |
| **ralph-loops** | `/gir:load-ralph` | Continuous iteration protocol — plan, execute, verify, repeat until done |
| **specgates** | `/gir:load-specgates` | Spec-driven development gates; blocks implementation until spec is confirmed |
| **parallel-agents** | `/gir:parallel` (auto-loaded) | Native parallel dispatch protocol using worktree-isolated agents |
| **autonomous-mode** | `/gir:run` (auto-loaded) | Autonomous execution schema, checkpoint format, cap fail-safe, unattended rules |

---

## Slash Commands

All commands use the `/gir:` prefix regardless of which module provides them.

| Command | What it does |
|---------|-------------|
| `/gir:init-setup` | Full project setup — runs init-project then init-memory-bank in one step |
| `/gir:init-project` | Scaffolds `CLAUDE-project.md` for a new or existing project |
| `/gir:init-memory-bank` | Creates the `.gir/` memory bank directory with 11 starter files |
| `/gir:parallel` | Decomposes the current task into independent workstreams and executes concurrently |
| `/gir:run` | Executes the active plan in `MISSION.md` autonomously with checkpoint/resume |
| `/gir:update` | Checks for and applies updates to installed GIR plugins |
| `/gir:status` | Prints active context, current session goals, and memory bank summary |
| `/gir:modules` | Lists all installed GIR modules and their registered tools |
| `/gir:drift-check` | Compares current code state against the active spec; surfaces divergence |
| `/gir:load-workflows` | Loads the workflows skill on demand for multi-step or delegated work |
| `/gir:load-specgates` | Loads the specgates skill on demand for new features or scope changes |
| `/gir:load-ralph` | Loads the ralph-loops skill on demand for iterative refinement work |

---

## Native Parallel Agents

`/gir:parallel` uses Claude Code's built-in `Agent` tool with `isolation: "worktree"` — each agent works in its own isolated git worktree, preventing file conflicts between concurrent workstreams.

**Auto-triggered by natural language.** The routing-stub detects these phrases and invokes `/gir:parallel` automatically:

> "parallelize this", "dispatch agents", "run these in parallel", "spawn agents", "do these simultaneously", "concurrent tasks"

No external CLI required.

---

## Autonomous Execution

`/gir:run` reads `.gir/MISSION.md` and executes the plan without human confirmation at each step. Key properties:

- **Checkpoints** before and after every significant action (written to `CLAUDE-activeContext.md`)
- **DOD gate** enforced on every task completion — never reports success prematurely
- **ESCALATION gate** always active — escalation is never suppressed in unattended mode
- **BLOCKED recovery**: retry → pivot (log to REVIEW-LOG.md) → escalate to human
- **Usage-cap fail-safe**: schedules a resume agent at run start (T+70min). If the session dies at a cap, the next window auto-resumes from the last checkpoint

---

## SessionStart Hook

gir installs a `SessionStart` hook that runs automatically when a Claude Code session begins:

1. Reads `.gir/MISSION.md` → loads active priorities
2. Reads `.gir/CLAUDE-activeContext.md` → restores session state
3. Reads `.gir/ESCALATION.md` → holds must-escalate conditions in context
4. Detects `autonomous_run.status == "in-progress"` → auto-resumes if >65 min elapsed and `resume_after_cap: true`

---

## sequential-thinking MCP

gir bundles the `sequential-thinking` MCP server. This gives Claude a structured tool for breaking down multi-step problems before executing them, reducing planning errors on complex tasks.

No additional configuration required — it runs as part of the plugin.

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

## Operating Model & Setup

- **[Operating Model](../../docs/OPERATING-MODEL.md)** — session-start protocol, Graphify gate, routing rules, delegation tiers, completion gate, and rule authority map. Start here to understand how gir works day-to-day.
- **[Setup Story](../../docs/SETUP-STORY.md)** — Phase 0–7 bootstrapping arc with reusable prompt templates. Use when setting up a new GIR-style repo or hardening an existing one.

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
