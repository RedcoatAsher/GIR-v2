# gir-tools

> **Requires [gir](../gir/).** Install gir first.

Power user tooling for parallel agent workflows. Adds AgentHub integration, subtask parallel execution, and enhanced subtask skills.

---

## Install

```bash
claude plugin install gir    # Required first
claude plugin install gir-tools
```

---

## Agents

| Agent | Role |
|-------|------|
| **agenthub** | Connects Claude Code to the AgentHub macOS app. Manages agent sessions, routes tasks across running agents, and surfaces session state in the AgentHub UI. Requires the [AgentHub macOS app](https://agenthub.app) installed and running. |
| **subtask-manager** | Creates and coordinates parallel subtasks via the `subtask` CLI. Spawns isolated subagents in Git worktrees for concurrent, conflict-free development. |

---

## Skills

| Skill | What it activates |
|-------|------------------|
| **subtask** | Extended subtask patterns — branching parallel workstreams, dependency ordering, and progress reporting via the `subtask` CLI |
| **agenthub-session-management** | Protocols for registering, suspending, and resuming agent sessions through AgentHub. Used by the agenthub agent and available for manual invocation. |

---

## Setup notes

**agenthub agent**: The AgentHub macOS app must be installed and running. No MCP configuration is required — AgentHub communicates directly with Claude Code.

**subtask-manager agent**: The `subtask` CLI must be installed:

```bash
# Install (pick one)
curl -fsSL https://subtask.dev/install.sh | bash
brew install zippoxer/tap/subtask
go install github.com/zippoxer/subtask/cmd/subtask@latest

# Then initialize
subtask init && subtask install
```

---

## License

[MIT](LICENSE) — rivit-studio, 2026.
