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

## Other AI Tools

Additional AI delegation targets can be configured here as they become available
(e.g., Codex, other CLI-based AI tools).
