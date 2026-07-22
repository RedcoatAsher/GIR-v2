---
name: core-practices
description: Universal Claude Code best practices for any coding task. Covers tool usage patterns, anti-patterns, code review standards, commit conventions, and workflow quality standards. Apply to ALL software engineering tasks.
---
# CLAUDE-core.md

> Universal Claude Code best practices — All project types

## Core Directives

### Execution Philosophy
- **Plan before executing** — Explore → Plan → Code → Commit (CLAUDE-workflows.md)
- **Do exactly what's asked** — nothing more, nothing less
- **Edit over create** — prefer modifying existing files
- **Use subagents early** — preserve main context
- **Clear context between tasks** — `/clear` to maintain focus
- **Verify before finishing** — confirm solution works
- **Collaborate actively** — iterate, present options, ask questions
- **Parallel execution** — invoke independent ops simultaneously

> **Critical**: "Steps #1-#2 (Explore-Plan) are crucial—without them, Claude tends to jump straight to coding" — Anthropic

### Spec-Driven Conventions
See `/gir:load-specgates` for full conventions. Key rules: max 3 `[NEEDS CLARIFICATION]` markers per feature (resolve before coding), mark parallel tasks with `[P]` + file paths, use phased task structure (Setup→Foundational→Stories→Polish), document state machines for features with >3 states.

### File Rules
- Never create files unless necessary
- Never create docs (*.md, README) unless requested
- Always prefer editing over creating

### Git Commits
- Format: `feat|fix|chore(scope): description`
- Commit incrementally, atomic changes
- Verify builds pass before committing
- Never commit without testing

## MCP Tool Strategy

### Documentation (Use First)

| Need | Tool | When |
|------|------|------|
| Library/API docs | **Context7** | Code gen, setup, config, library questions |
| General docs | **Ref** | Broader search, URL content |
| Code examples | **exa** | Real-world patterns, SDKs |
| Web + images | **fetch:imageFetch** | URLs with image extraction |

**Auto-trigger Context7**: When generating code, configuring libraries, or API questions.

### Sequential-Thinking

See routing-stub Tier 1 — deploy for complex tasks (>3 steps), architecture, debugging, refactors (3+ files). Skip for single-file edits and trivial fixes.

## Command Efficiency

**Banned**: `tree`, `find`, `grep -r`, `ls -R`, `cat | grep`

**Use Instead**:
- File listing: `fd . -t f`, `rg --files`, `ls -la`
- Content search: `rg "pattern"`, `rg -i`, `rg -t ts`
- File finding: `fd "name"`, `fd -e js`
- JSON: `jq`

## Memory Bank

| File | Purpose |
|------|---------|
| `.gir/CLAUDE-activeContext.md` | Session state, goals, progress |
| `.gir/CLAUDE-patterns.md` | Code patterns, conventions |
| `.gir/CLAUDE-decisions.md` | Architecture decisions |
| `.gir/CLAUDE-troubleshooting.md` | Issues and solutions |
| `.gir/MISSION.md` | Active priorities, unattended operation state |
| `.gir/POLICY.md` | Repo policy, conventions |
| `.gir/ESCALATION.md` | Must-escalate conditions |
| `.gir/DOD.md` | Definition of done checklist |
| `.gir/REVIEW-LOG.md` | Review outcomes, audit trail |
| `.gir/ESCALATION-LOG.md` | Escalation events with full context + timestamps |

**Memory-first lookup**: before re-deriving anything, check the bank — errors → troubleshooting, conventions → patterns, "why is it like this" → decisions. Entries are advisory, not authoritative: reconcile against the current task and any higher-priority policy before reusing. Found it and still applies → reuse; solved something new → append it.

**Workflow**: See routing-stub Session Start Protocol — authoritative for session init, escalation checks, and completion gates.

## Anti-Patterns

**Code**: Inline styles, `any` type, `console.log` in commits, commented code, magic numbers, giant components (>150 lines), prop drilling >2 levels, useEffect for derived state

**UX (Never)**: Fake urgency, confirmshaming, hidden costs, forced continuity, misleading buttons, privacy-invasive defaults, roach motel, misdirection

## Skills & Agents

### Custom Agents (.claude/agents/)

**Core**: feature-architect, code-reviewer, debugger

**Usage**: `/agent <name>` or Task tool

### Recommended Skills

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

Provides: plan-implementer, requesting-code-review, subagent-driven-development, finishing-a-development-branch, test-driven-development, systematic-debugging

## Interactive Collaboration

**Controls**: Escape (interrupt), Double-Escape (jump back), Request undo, Ask questions

**Rules**:
- Plan before coding, course correct as needed
- Ask clarifying questions when ambiguous
- Present options for decisions
- Checkpoint at milestones
- Never execute large changes fully autonomously
- Never assume user intent without confirmation

**Effective prompts**: Be specific. "Add JWT auth with httpOnly cookies, login/logout at /api/auth/*, middleware for /api/user/*" vs "Add authentication"

