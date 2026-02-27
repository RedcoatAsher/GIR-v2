# gir-automation

> **Requires [gir-core](../gir-core/).** Install gir-core first.

n8n workflow tooling for Claude Code. Adds a specialist agent that builds, edits, and debugs n8n automation workflows.

---

## Install

```bash
claude plugin install gir-core        # Required first
claude plugin install gir-automation
```

---

## Agents

| Agent | Role |
|-------|------|
| **n8n-builder** | Designs and implements n8n workflows from plain-language descriptions. Handles node selection, credential wiring, error branches, and iterative refinement. |

---

## MCP requirement

**n8n-builder** requires the `n8n-mcp` MCP server to interact with a running n8n instance. Configure it separately in your Claude Code MCP settings before using this agent.

The agent will load without it, but will not be able to create or modify live workflows.

---

## License

[MIT](LICENSE) — RedcoatAsher, 2026.
