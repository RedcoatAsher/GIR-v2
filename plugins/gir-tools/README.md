# gir-tools

> **Requires [gir-core](../gir-core/).** Install gir-core first.

Power user tooling for parallel agent workflows. Adds AgentHub integration, agent team coordination, and enhanced subtask skills.

---

## Install

```bash
claude plugin install gir-core    # Required first
claude plugin install gir-tools
```

---

## Agents

| Agent | Role |
|-------|------|
| **agenthub** | Connects Claude Code to the AgentHub macOS app. Manages agent sessions, routes tasks across running agents, and surfaces session state in the AgentHub UI. Requires the [AgentHub macOS app](https://agenthub.app) installed and running. |
| **team-lead** | Coordinates multi-agent teams within a single Claude Code session. Assigns work to teammates, tracks progress, and synthesizes results. Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` set in `settings.json`. |

---

## Skills

| Skill | What it activates |
|-------|------------------|
| **subtask** | Extended subtask patterns — branching parallel workstreams, dependency ordering, and progress reporting via the `subtask` CLI |
| **agenthub-session-management** | Protocols for registering, suspending, and resuming agent sessions through AgentHub. Used by the agenthub agent and available for manual invocation. |

---

## Setup notes

**agenthub agent**: The AgentHub macOS app must be installed and running. No MCP configuration is required — AgentHub communicates directly with Claude Code.

**team-lead agent**: Add the following to your Claude Code `settings.json` to enable agent teams:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

---

## License

[MIT](LICENSE) — RedcoatAsher, 2026.
