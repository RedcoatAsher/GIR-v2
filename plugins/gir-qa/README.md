# gir-qa

> **Requires [gir-core](../gir-core/).** Install gir-core first.

QA and code review tooling for Claude Code. Integrates CodeRabbit for automated PR reviews and Jules for AI task delegation.

---

## Install

```bash
claude plugin install gir-core   # Required first
claude plugin install gir-qa
```

---

## Skills

| Skill | What it activates |
|-------|------------------|
| **qa-tools** | CodeRabbit and Jules MCP tool reference — automated PR reviews, AI task delegation |

---

## MCP requirements

- **qa-tools (CodeRabbit)** — requires the `coderabbit` MCP server
- **qa-tools (Jules)** — requires the `jules` MCP server

Configure these separately in your Claude Code MCP settings. The skill loads without them, but QA operations will be unavailable.

---

## License

[MIT](LICENSE) — rivit-studio, 2026.
