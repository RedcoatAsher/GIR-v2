---
name: ai-delegation
description: AI tool delegation patterns — Gemini-CLI, Codex, and other external AI tools.
  Apply when delegating tasks to external AI tools to save tokens.
---
# AI Tool Delegation

## Gemini-CLI Delegation

Offload to save tokens:

DELEGATE: File analysis (>200 lines), code review, test gen, refactoring,
regex, boilerplate, data transforms, explaining code

KEEP IN MAIN: Quick edits (<50 lines), direct Q&A, architecture decisions,
final implementations

Batch related analyses into a single delegation call — one call analyzing five
files beats five calls.

## First-Pass Code Review

External models are enabled at will — a tool participates only if configured in
`.gir/ai-integrations.md`. When one or more review-capable tools are enabled
(Gemini-CLI, Codex, ...), they tackle code review as the **first pass**:

1. Delegate the diff to the configured tool(s) — collect findings cheaply
2. Feed the findings into the `code-reviewer` agent as input, not verdict
3. `code-reviewer` runs the final DOD gate — external findings never approve or
   reject on their own

No tools configured → skip silently; `code-reviewer` reviews from scratch.

## Other AI Tools

Additional AI delegation targets can be configured here as they become available
(e.g., Codex, other CLI-based AI tools).
