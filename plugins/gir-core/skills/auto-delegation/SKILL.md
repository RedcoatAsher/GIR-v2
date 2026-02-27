---
name: auto-delegation
description: Auto-delegation and task prioritization rules for Claude Code. Determines when to use subagents, which agent to delegate to, and how to break down complex tasks. Apply when planning work, starting new tasks, or when a task involves multiple concerns.
---
# CLAUDE-delegation.md

> Auto-Delegation & Prioritization Rules — Optimal subagent usage

## Philosophy

**Goal**: Maximize token efficiency via predictable agent delegation.

**Benefits**: Consistent usage, 60-80% token savings, reduced cognitive load, predictable behavior

## 3-Tier Auto-Delegation

### Tier 1: MANDATORY (Auto-Deploy)

#### Sequential-Thinking
**When**: Complex tasks (>3 steps), architecture, debugging, refactors (3+ files), feature design, optimization, security, migrations

**Why**: 60-80% token savings, prevents re-planning

```
"Using sequential-thinking to analyze approach..."
[Reasoning]
"Complete. Key insights: [summary]"
```

#### Code-Reviewer
**When**: Before `git commit`, completing major tasks, finishing features

**Why**: Catches issues before git history

```
"Deploying code-reviewer before commit..."
[Review]
"Complete. 2 suggestions, 0 critical issues."
```

#### Explore Agent
**When**: Task involves >5 files, "where is..." / "how does..." questions, mapping codebase

**Skip**: <5 files, user provided paths, simple codebase (<20 files)

```
"Task requires 12 files. Deploying Explore agent..."
[Exploration]
"Found 3 relevant modules: auth/, api/, db/"
```

### Tier 2: RECOMMENDED (Usually Auto-Deploy)

#### Subtask (Parallel Worktrees)
**When**: >2 independent tasks, multiple unrelated features, non-overlapping modules, feature + tests simultaneously

**Why**: Isolated execution, concurrent development, no merge conflicts during work

**Limit**: Practical limit ~5 parallel tasks (depends on system resources)

**Override**: "do sequentially", "use single worktree"

```
"Identified 3 independent tasks. Using Subtask for parallel execution..."
[Draft subtasks]
"Subagents spawned. Monitor with: subtask list"
```

#### Plan Agent
**When**: >8 TodoWrite items, multiple approaches, cross-system integration

**Override**: "skip plan agent"

#### Parallel Agents (In-Context)
**When**: >2 independent tasks within same worktree, quick parallel investigations

**Limit**: Max 3 parallel agents

**Override**: "do sequentially"

**Note**: Prefer Subtask over Parallel Agents when tasks modify files (isolation prevents conflicts)

#### Debugger Agent
**When**: No obvious cause, multi-system issue, specific reproduction conditions

**Override**: "let me debug"

#### Ralph Loop Pattern
**When**: Clear completion criteria (tests pass, build succeeds), iterative work, TDD

**Skip**: Needs human judgment, ambiguous requirements, unpredictable iterations

**Override**: "I'll handle iterations"

```
"Using Ralph loop with criteria:
- Tests pass, build succeeds, integration verified
[Iterations]
Ralph loop complete. Task verified."
```

### Tier 3: NEVER Auto-Delegate

Keep in main context:
- Final implementation (after planning)
- Direct user Q&A
- Architecture decisions
- Single-file quick edits (<50 lines)
- Trivial bug fixes
- User preference questions

## Prioritization Rules

### Default Order (When Not Specified)

**CRITICAL** (Blockers): DB schema, auth setup, type defs, env/config, deps

**HIGH** (Core): API endpoints, DB queries, business logic, integrations, error handling

**MEDIUM** (Features): UI components, forms, loading states, feedback, responsive design

**LOW** (Polish): Animations, optimization, cleanup, extra error messages, docs

### User Override

- "Do these in order: X, Y, Z" → Follow exactly
- "X is priority" → Do X first
- "Focus on performance" → Reprioritize optimization to HIGH
- Dependency conflict → Ask: "Z must be done first. Proceed with Z→X→Y?"

## Communication Patterns

**Mandatory (Tier 1)**: Brief announcement, proceed
```
"Deploying [agent] for [reason]..."
[Work]
"Complete. [Key findings]."
```

**Recommended (Tier 2)**: Explain why, announce, report
```
"Feature has 12 steps. Deploying Plan agent..."
[Work]
"Complete. Recommended approach: [summary]"
```

## Configuration

Add to `CLAUDE-project.md`:

```yaml
delegation:
  mode: "balanced"  # aggressive | balanced | minimal
  auto_explore_threshold: 5
  auto_plan_threshold: 8
  auto_parallel_threshold: 2
  max_parallel_agents: 3
  always_code_review: true
  always_sequential_thinking: true
```

**Modes**:
- Aggressive: thresholds 3/5/2, max 5 parallel
- Balanced (default): 5/8/2, max 3 parallel
- Minimal: 10/12/2, max 2 parallel

## Interrupt Controls

**During execution**: Escape (stop), Type message (read after), Double-Escape (halt/discard)

**Before deployment**: "skip that", "I'll handle it", Escape

**After completion**: "ignore that", "try different approach"

## Integration with Workflow

### Explore-Plan-Code-Commit

**Explore**: IF >5 files → AUTO Explore agent, ELSE manual read

**Plan**: ALWAYS sequential-thinking, IF >8 items → RECOMMENDED Plan agent, IF >2 independent file-modifying tasks → CONSIDER Subtask

**Code**: IF >2 independent file-modifying → RECOMMENDED Subtask (parallel worktrees), IF read-only/quick → parallel agents, Gemini-CLI for boilerplate

**Commit**: ALWAYS code-reviewer, then commit in main (or per-subtask commits)

### TodoWrite Integration

Auto-analyze: task dependencies, independent tasks (parallel-eligible), complexity, priority order

## Quick Reference

| Agent | Trigger | Tier | Skip? |
|-------|---------|------|-------|
| Sequential-Thinking | Complex planning | Mandatory | Trivial only |
| Code-Reviewer | Before commit | Mandatory | Emergency only |
| Explore | >5 files | Mandatory | <5 files |
| Subtask | >2 independent, file-modifying | Recommended | "do sequentially" |
| Plan | >8 items | Recommended | "skip plan agent" |
| Parallel (in-context) | >2 independent, read-only | Recommended | "do sequentially" |
| Debugger | Complex bug | Recommended | "let me debug" |
| Ralph Loop | Clear criteria | Recommended | "I'll iterate" |

### Priority Order
```
CRITICAL (Dependencies) → HIGH (Core) → MEDIUM (Features) → LOW (Polish)
```

## Project-Specific Delegation Rules

**Version**: 1.1 (2026-01-23)
