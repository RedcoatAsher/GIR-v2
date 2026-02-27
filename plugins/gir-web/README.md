# gir-web

> **Requires [gir-core](../gir-core/).** Install gir-core first.

Frontend and fullstack tooling for Claude Code. Adds specialist agents for documentation fetching, deployment management, and UI generation, plus skills for design fundamentals and frontend patterns.

---

## Install

```bash
claude plugin install gir-core   # Required first
claude plugin install gir-web
```

---

## Agents

| Agent | Role |
|-------|------|
| **docs-fetcher** | Fetches and summarizes external documentation, library references, and API specs on demand |
| **deploy-manager** | Manages Vercel deployments — triggers deploys, inspects build logs, and surfaces runtime errors |
| **ui-generator** | Generates UI components and layouts from descriptions or design references |

### MCP requirements

These agents integrate with external MCP servers that must be configured separately in your Claude Code settings:

- **docs-fetcher** — requires `context7` or `Ref` MCP server
- **deploy-manager** — requires `vercel` MCP server
- **ui-generator** — requires `v0` or `figma` MCP server

The agents will still load without these MCP servers, but their primary capabilities will be unavailable.

---

## Skills

| Skill | What it activates |
|-------|------------------|
| **frontend-design** | Component architecture patterns, state management conventions, accessibility defaults, responsive layout guidance |
| **design-principles** | Visual hierarchy, spacing systems, typography choices, and color usage — applied during UI generation and review |

---

## License

[MIT](LICENSE) — RedcoatAsher, 2026.
