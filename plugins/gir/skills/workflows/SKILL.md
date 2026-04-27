---
name: workflows
description: Advanced workflow patterns for Claude Code including Explore-Plan-Code-Commit (E-P-C-C), multi-Claude coordination, parallel development, and iterative refinement loops. Use when planning multi-step implementations or complex engineering tasks.
---
# CLAUDE-workflows.md

> Advanced workflow patterns and multi-Claude coordination (Anthropic best practices)

## Multi-Claude Coordination

### Parallel Review Pattern

**Writer Claude**: Implements features, writes tests, creates endpoints

**Reviewer Claude** (parallel): Reviews security, validates coverage, checks patterns

**When**: Large features, security-sensitive code, complex refactors, pre-production merges

### Subtask: Parallel Task Execution

**Subtask** automates parallel work via Git worktrees. Claude spawns subagents that work concurrently in isolated environments.

```bash
subtask draft fix-auth-bug        # Create task for subagent
subtask draft implement-api       # Another parallel task
subtask list                      # Monitor all tasks
subtask fix-auth-bug              # Open TUI for specific task
```

**Workflow**:
1. Identify parallelizable work during Plan phase
2. Draft subtasks for each independent piece
3. Subagents execute in separate worktrees
4. Review completed work in TUI
5. Merge or request modifications

**Ideal for**:
- Multiple independent features
- Parallel bug fixes
- Feature implementation + test writing
- Module-level refactors
- Concurrent code review + implementation

**Integration**: Use with Ralph loops for iterative parallel work. Deploy code-reviewer on merged results.

### Agent Teams (Built-in)

**Agent Teams** are a built-in Claude Code feature (experimental) that coordinate multiple Claude instances working together. One session acts as the team lead, spawning teammates that work independently and communicate directly.

> Enable with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in settings.json or environment.

```bash
# Tell Claude to create a team — it handles spawning and coordination
"Create an agent team to refactor the auth module. Spawn three teammates:
 one for the API layer, one for the database layer, one for tests."
```

**Workflow**:
1. Describe the task and team structure to Claude
2. Lead creates shared task list, spawns teammates
3. Teammates work independently, communicate via messages
4. Lead synthesizes results, coordinates merges
5. Clean up: ask the lead to shut down teammates and clean up

**Controls**:
- `Shift+Up/Down` — cycle between teammates (in-process mode)
- `Shift+Tab` — toggle delegate mode (lead coordinates only, no coding)
- `Ctrl+T` — toggle task list
- `Escape` — interrupt a teammate's current turn

**Best use cases**:
- Research and review (multiple reviewers, different lenses)
- New modules or features (each teammate owns a piece)
- Debugging with competing hypotheses (parallel investigation)
- Cross-layer coordination (frontend + backend + tests)

**Not ideal for**:
- Sequential tasks with dependencies
- Same-file edits (causes overwrites)
- Simple tasks (coordination overhead exceeds benefit)

**vs Subtask**: Agent teams are built-in and support inter-agent communication. Subtask uses isolated git worktrees with a CLI. Use agent teams when teammates need to discuss and coordinate; use Subtask when tasks are fully independent and you want file isolation.

**vs Subagents**: Subagents run within a single session and report back. Agent teams are separate sessions that communicate peer-to-peer. Use subagents for quick, focused research; use agent teams for complex collaborative work.

**Hooks for quality gates**:
- `TeammateIdle` — runs when a teammate finishes. Exit code 2 sends feedback and keeps them working.
- `TaskCompleted` — runs when a task is marked complete. Exit code 2 blocks completion with feedback.

### Git Worktrees (Manual)

For manual worktree management without Subtask:

```bash
git worktree add .worktrees/feature-xyz feature-xyz
git worktree add .worktrees/fix-auth-bug fix-auth-bug
git worktree list
git worktree remove .worktrees/feature-xyz
git worktree prune
```

**Use for**: Non-overlapping components, migrations, parallel experiments

### When Multiple Sessions

| Scenario | Single | Subagents | Subtask | Agent Teams |
|----------|--------|-----------|---------|-------------|
| Small bug fix | Yes | — | — | — |
| Feature + tests | Yes | Optional | — | — |
| Focused research tasks | — | Yes | — | — |
| Multiple independent features | — | — | Yes | Optional |
| Large refactor + new feature | — | — | Yes | Yes |
| Code gen + review (parallel) | — | — | Yes | Yes |
| Cross-layer coordination | — | — | — | Yes |
| Debugging competing hypotheses | — | — | — | Yes |
| Multi-perspective review | — | — | — | Yes |

## Subagent Strategy

> "Preserve context without losing efficiency" — Anthropic

Deploy subagents **early and often** to preserve main context.

**Use for**: Exploration, pattern discovery, verification, parallel investigations, detail-gathering

**Keep in main**: Tasks needing history, final implementations, direct Q&A, architecture decisions

### Available Types

| Agent | Purpose | When |
|-------|---------|------|
| **Explore** | Fast codebase nav | Finding patterns, understanding structure |
| **Plan** | Architecture | Before implementation |
| **General-purpose** | Complex tasks | Research, verification |
| **claude-code-guide** | Doc lookup | Claude Code features |

### Patterns

**Explore → Plan → Implement**:
1. Deploy Explore agent → Find patterns, integration points
2. Deploy Plan agent → Generate architecture plan
3. Review with user → Get approval
4. Implement in main → Execute plan

**Parallel Investigation**:
Deploy 3+ agents simultaneously (check commits, analyze logs, search issues), synthesize in main

**Context Preservation**:
Don't pollute main with tangential investigation. Deploy agent, get findings, continue main work.

### Best Practices
- Deploy BEFORE consuming context with exploration
- Launch multiple agents in parallel (single tool call)
- Resume agents for follow-ups (preserves context)
- Use `run_in_background: true` for long tasks
- Specify thoroughness: quick/medium/very thorough

## Primary Workflow: Explore-Plan-Code-Commit

> "Steps #1-#2 are crucial—without them, Claude jumps straight to coding" — Anthropic

### Phase 1: Explore (DON'T CODE)

**Actions**: Read files, deploy Explore subagent, search with `rg`, check CLAUDE-patterns.md, review activeContext

**Exit**: Understand existing code, patterns, changes needed, dependencies

### Phase 2: Plan (THINK FIRST)

**Actions**: Use `sequential-thinking` for complex decisions, create TodoWrite plan, deploy Plan agent if needed, present for approval

**Exit**: Plan documented, user approved, files identified, dependencies clear, clarification gate passed (see CLAUDE-specgates.md)

### Phase 3: Code

**Actions**: Implement per plan, track with TodoWrite, use Gemini-CLI for boilerplate, Context7 for docs, verify with build

**Exit**: All todos complete, build passes, solution verified

### Phase 4: Commit

**Actions**: Pre-commit review (Gemini-CLI/CodeRabbit), conventional commit format, update memory bank, deploy to Vercel, document solutions

**Exit**: Code reviewed, committed, deployed, documented

## TDD Workflow

1. Write tests (expected inputs/outputs)
2. Confirm tests fail (red)
3. Commit tests
4. Implement until tests pass (green)
5. Refactor and optimize
6. Verify all tests pass

## Ralph Loop Workflow

Continuous iteration until completion criteria met. See [RALPH-LOOPS.md](RALPH-LOOPS.md) for full guide.

**When**: TDD, build fixes, API implementations, long-running features, migrations, quality improvements

**Not for**: Frequent design decisions, ambiguous requirements, production debugging

**Process**:
1. Define clear completion criteria (tests pass, build succeeds)
2. Set safety parameters (max iterations)
3. Iterate: implement → verify → analyze → iterate
4. Signal completion when criteria met

**Max iterations**: Simple 10-15, Medium 15-25, Complex 25-40

**Integration**: Use in Code phase of Explore-Plan-Code-Commit

## Visual Iteration Workflow

1. Receive design mock/screenshot
2. Generate (v0 or Figma tools)
3. Screenshot result
4. Compare and iterate
5. Refine until match

> "Claude's outputs improve significantly with iteration" — Anthropic

## Checklist-Driven Migrations

For large-scale changes (deps, APIs, refactors):

1. Generate checklist of affected areas
2. Use TodoWrite as scratchpad
3. Address systematically
4. Mark completion after verification

## Context Management

### When to /clear
- After completing features
- Switching to unrelated tasks
- Before large refactors
- Unfocused conversation

### Don't Clear
- Mid-task
- User might reference history
- Architecture decisions need context

**Alternative**: Deploy subagent for tangential tasks instead of polluting main

## Interactive Collaboration

**Controls**: Escape (interrupt), Double-Escape (alternatives), Request undo, Ask questions

**Rules**:
- Plan first, course correct as needed
- Ask when ambiguous
- Present options for decisions
- Checkpoint at milestones
- Never fully autonomous large changes
- Never assume intent without confirmation

**Specificity improves results**: Detailed prompts → better first attempts

## Data Input Strategies

| Method | Best For |
|--------|----------|
| Direct copy | Small snippets, errors |
| Pipe to Claude | Command output, logs |
| Tool retrieval | Codebase files |
| URL fetching | External docs |

## Git Integration

### History Analysis
```bash
git log -p -- src/auth/ | head -500    # Why?
git log --all --grep="auth" --oneline   # When?
```

### Complex Operations
- Interactive rebases: Provide list, get plan
- Merge conflicts: Share markers, get resolution
- Commit messages: Generate from `git diff --staged`
- Cherry-picking: Identify and apply

### GitHub CLI
```bash
gh pr create --title "feat: JWT" --body "$(git log main..HEAD --format='- %s')"
gh pr view 123 --comments
gh pr create --reviewer @security-team
git commit -m "fix: auth bug (fixes #123)"
```

## Workflow Decision Matrix

| Task | Workflow | Tools |
|------|----------|-------|
| Small bug | Explore-Plan-Code-Commit | rg, Read, Edit |
| New feature | E-P-C-C + TDD | Subagents, TodoWrite |
| Long-running feature | E-P + Ralph Loop | Ralph, TodoWrite |
| Build/test fixes | Ralph Loop | npm scripts |
| API implementation | E-P + Ralph + TDD | Tests, TodoWrite |
| Large refactor | Checklist + Multi-session | Worktrees |
| Design implementation | Visual Iteration | Figma, v0 |
| Migration | Checklist + Ralph | TodoWrite, Subagents |
| Bug investigation | Subagent exploration | Explore agent, rg |
| Architecture decision | Sequential thinking | Plan agent |
| Multiple independent tasks | Subtask | Git worktrees, TUI |
| Parallel features | Subtask + E-P-C-C | Subagents per task |
| Feature + tests | Subtask (parallel) | Isolated worktrees |
| Cross-layer changes | Agent Teams | Lead + teammates |
| Parallel code review | Agent Teams | Multiple reviewers |
| Competing debug hypotheses | Agent Teams | Parallel investigation |
| Complex collaborative work | Agent Teams + E-P-C-C | Shared task list |

## Key Takeaways

1. **Plan before coding** — Exploration prevents wasted effort
2. **Subagents early** — Preserve main context
3. **Clear context frequently** — Maintain focus
4. **Parallel sessions for quality** — Writer + Reviewer
5. **Iterate with user** — Collaboration over autonomy (except Ralph)
6. **Ralph loops for iteration** — Autonomous on well-defined tasks
7. **Specific prompts** — Detail improves first-attempt success
8. **Mix input methods** — Right tool for each data type
9. **Leverage git history** — Answer architectural questions
10. **Subtask for parallelism** — Multiple independent tasks in isolated worktrees
11. **Agent teams for collaboration** — When teammates need to communicate, not just report back

## Project-Specific Workflows

**Updated**: 2026-02-10 | **Source**: [Anthropic Best Practices](https://anthropic.com/engineering/claude-code-best-practices), [Agent Teams Docs](https://code.claude.com/docs/en/agent-teams)
