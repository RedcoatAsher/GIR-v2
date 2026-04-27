# gir-database

> **Requires [gir](../gir/).** Install gir first.

Database management tooling for Claude Code. Adds Supabase MCP integration for schema management, migrations, edge functions, and query execution.

---

## Install

```bash
claude plugin install gir   # Required first
claude plugin install gir-database
```

---

## Skills

| Skill | What it activates |
|-------|------------------|
| **database-tools** | Supabase MCP tool reference — schema inspection, SQL execution, migrations, edge functions, branch management |

---

## MCP requirement

**database-tools** requires the `supabase` MCP server configured in your Claude Code MCP settings before using this plugin.

The skill will still load without it, but Supabase operations will be unavailable.

---

## License

[MIT](LICENSE) — rivit-studio, 2026.
