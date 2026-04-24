---
name: core-practices
description: Universal Claude Code best practices for any coding task. Covers tool usage patterns, anti-patterns, code review standards, commit conventions, and workflow quality standards. Apply to ALL software engineering tasks.
---
# CLAUDE-core.md

> Universal Claude Code best practices — All project types

## Session Initialization

Run drift check on session start if `.gir/CLAUDE-activeContext.md` is stale (>24h) or missing. Compare scripts, deps, configs, dirs, env vars against CLAUDE-project.md baseline. On drift: update activeContext, ask user to update project config.

See [CLAUDE-examples.md](CLAUDE-examples.md#session-initialization-examples) for implementation.

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
See [CLAUDE-specgates.md](CLAUDE-specgates.md) for full conventions. Key rules: max 3 `[NEEDS CLARIFICATION]` markers per feature (resolve before coding), mark parallel tasks with `[P]` + file paths, use phased task structure (Setup→Foundational→Stories→Polish), document state machines for features with >3 states.

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

### Sequential-Thinking (Required)

**Use `sequential-thinking:sequentialthinking` BEFORE**:
- Multi-step architectural decisions
- Planning phases (before TodoWrite)
- Complex debugging
- Large refactors (3+ files)
- Feature design with options
- Performance/security analysis
- Migration planning

**Benefits**: 60-80% context savings, prevents re-planning, documents rationale

**Skip for**: Single-file edits, obvious answers, trivial fixes (<10 lines)

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

**Workflow**: Read activeContext + MISSION on session start → check ESCALATION before risky actions → verify DOD before reporting complete

## Workflow Patterns

> See CLAUDE-workflows.md for comprehensive docs

### Explore-Plan-Code-Commit (with Auto-Delegation)

**1. Explore** — Read files, deploy Explore agent if >5 files, DON'T CODE YET

**2. Plan** — Use sequential-thinking (mandatory), create TodoWrite, get approval
- Deploy Plan agent if >8 items

**3. Code** — Implement systematically, verify each step
- Deploy parallel agents if >2 independent tasks
- Use Gemini-CLI for boilerplate

**4. Commit** — Deploy code-reviewer (mandatory), then commit

### Subagent Strategy

**Deploy for**: Exploration, architecture planning, parallel investigations, verification

**Keep in main**: Tasks needing history, final implementations, architecture decisions

### Multi-Claude Coordination

Use for: Parallel features (git worktrees), implementation + review, research + dev, complex debugging

See [CLAUDE-examples.md](CLAUDE-examples.md#multi-claude-coordination-quick-reference)

### Context Management

**Use `/clear`**: After features, switching tasks, before refactors

**Don't clear**: Mid-task, when user might reference history

## Anti-Patterns

**Code**: Inline styles, `any` type, `console.log` in commits, commented code, magic numbers, giant components (>150 lines), prop drilling >2 levels, useEffect for derived state

**UX (Never)**: Fake urgency, confirmshaming, hidden costs, forced continuity, misleading buttons, privacy-invasive defaults, roach motel, misdirection

## Skills & Agents

### Custom Agents (.claude/agents/)

**Core**: feature-architect, code-reviewer, debugger

**Web**: docs-fetcher, deploy-manager, ui-generator

**Automation**: n8n-builder

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

## Pre-Completion Checklist

- [ ] Explored before coding
- [ ] Used sequential-thinking for complex tasks
- [ ] Planned and got approval
- [ ] Solution verified (builds/runs)
- [ ] No unnecessary files created
- [ ] Edited existing files where possible
- [ ] No console.log in code
- [ ] Commit format: `feat|fix|chore(scope): description`
- [ ] Memory bank updated if needed
- [ ] Deployment successful (if applicable)
- [ ] Context cleared with `/clear` if switching tasks

## Project Extensions

