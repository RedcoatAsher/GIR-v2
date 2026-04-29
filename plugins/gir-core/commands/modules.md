# /modules

List installed GIR modules and discover available ones.

---

## Instructions

Read `.gir/GIR.modules` and display the currently installed modules in a clean table format.

Then inform the user about available official modules they haven't installed:

**Official GIR Modules:**

| Module | Description | Install |
|--------|-------------|---------|
| gir | Core agents, workflows, best practices (REQUIRED) | `claude plugin install gir` |
| gir-web | Frontend/fullstack agents and design skills | `claude plugin install gir-web` |
| gir-automation | n8n workflow builder agent | `claude plugin install gir-automation` |
| gir:atc | AgentHub + team coordination + subtask parallel execution | `claude plugin install gir:atc` |
| gir-database | Database management — Supabase | `claude plugin install gir-database` |
| gir-ai | AI tool delegation — Gemini-CLI, Codex | `claude plugin install gir-ai` |
| gir-qa | QA & review tools — CodeRabbit, Jules | `claude plugin install gir-qa` |

Mark installed modules with a checkmark. Show their version and what they provide.

If `.gir/GIR.modules` doesn't exist, say the memory bank hasn't been initialized and suggest running `/gir:init-memory-bank` first.
