---
name: ai-delegation
description: "Route tasks to external AI tools (Gemini-CLI, Codex) to save Claude Code context tokens. Defines delegation rules: offload file analysis (>200 lines), code review, test generation, refactoring, regex, boilerplate, and data transforms to external tools. Keep quick edits (<50 lines), direct Q&A, architecture decisions, and final implementations in the main Claude session. Use when context is growing large, tasks involve bulk analysis or boilerplate generation, or when multiple AI tools are available for parallel work."
---

# AI Tool Delegation

Route tasks to external AI tools to save Claude Code context tokens.

## Workflow

1. **Assess task size** — Check if the task involves >200 lines of analysis, bulk generation, or repetitive transforms
2. **Choose tool** — Select Gemini-CLI for analysis/review tasks, Codex for code generation tasks
3. **Delegate** — Offload the task with clear instructions and file paths
4. **Integrate** — Review the external tool's output and apply it in the main session

## Delegation Rules

### Offload to External Tools
- File analysis (>200 lines)
- Code review
- Test generation
- Refactoring large files
- Regex pattern building
- Boilerplate generation
- Data transforms
- Explaining unfamiliar code

### Keep in Main Claude Session
- Quick edits (<50 lines)
- Direct Q&A with the user
- Architecture decisions
- Final implementations
- Tasks requiring conversation history

## Supported Tools

| Tool | Strengths | Best For |
|------|-----------|----------|
| Gemini-CLI | Large context window, fast analysis | File analysis, code review, explaining code |
| Codex | Code generation, refactoring | Test generation, boilerplate, data transforms |

## When to Delegate

Delegate when context exceeds ~50K tokens and the task is self-contained (can be described without the full conversation history). Do not delegate tasks that depend on prior conversation decisions or require user interaction mid-task.
