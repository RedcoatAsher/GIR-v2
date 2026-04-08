---
name: agenthub-session-management
description: "Coordinate multiple Claude Code sessions using AgentHub with git worktrees. Covers session lifecycle states (NEW through COMPLETED), JSONL session data at ~/.claude/projects/, four multi-session patterns (Feature Parallel, Bug Swarm, Exploration Fan-Out, Pipeline), worktree creation and cleanup, session monitoring via health checks, and coordination protocols for parallel work. Use when running parallel Claude Code sessions, setting up git worktrees for multi-agent work, debugging session state, or planning multi-session feature development."
---

# AgentHub Session Management

Coordinate, monitor, and optimize multi-session Claude Code workflows using AgentHub and git worktrees.

## Workflow

1. **Assess scope** — Determine if the task benefits from multiple sessions (independent components, >100K token context, different expertise needed)
2. **Choose pattern** — Select from Feature Parallel, Bug Swarm, Exploration Fan-Out, or Pipeline based on task structure
3. **Set up worktrees** — Create integration branch, component branches, and git worktrees
4. **Monitor sessions** — Use AgentHub health checks and session state tracking
5. **Coordinate merges** — Follow the completing protocol: commit per branch → PR to integration → resolve conflicts → test → merge

## Session Lifecycle

Sessions progress through these states:

| State | Meaning | Action |
|-------|---------|--------|
| NEW | Just created | Assign task |
| THINKING | Processing request | Wait |
| EXECUTING_TOOL | Running tool operation | Wait |
| AWAITING_APPROVAL | Needs confirmation | Approve/Reject |
| WAITING_FOR_USER | Needs input | Provide input |
| IDLE | Ready for new task | Send request |
| COMPLETED | Finished | Review output |

Session data is stored at `~/.claude/projects/{url-encoded-path}/{sessionId}.jsonl` as JSON objects with `type`, `timestamp`, and `content` fields.

## Multi-Session Patterns

### Feature Parallel
Split a large feature into independent components (backend, frontend, tests), each in its own worktree and session.

```bash
# Set up integration + component branches
git checkout -b feature/main && git push -u origin feature/main
git checkout -b feature/backend feature/main
git checkout -b feature/frontend feature/main
# Create worktrees
git worktree add ../proj-backend feature/backend
git worktree add ../proj-frontend feature/frontend
```

### Bug Swarm
Multiple sessions tackling different bugs simultaneously — each on its own `bugfix/N` branch merging back to main. Best for sprint bug bashes and hotfix situations.

### Exploration Fan-Out
Three sessions explore different approaches to the same problem. Evaluate results, select the best solution, then continue in a single main session. Best for architecture decisions and algorithm selection.

### Pipeline
Sequential handoffs between specialized sessions (Architect → Implement → Review). Each session produces a handoff document with completed items, in-progress work, key files, and blockers.

## Git Worktree Best Practices

```bash
# Create: always branch from clean state
git fetch origin && git checkout main && git pull
git checkout -b feature/name
git worktree add ../project-feature feature/name

# Cleanup
git worktree list          # see all worktrees
git worktree prune         # remove stale entries
git worktree remove ../project-feature  # remove specific
```

Track active worktrees with a mapping table: Worktree Path | Branch | Session ID | Owner | Status.

## Coordination Protocol

**Starting**: Create integration branch → branch per session → create worktrees → document assignments → set merge order.

**During**: Regular AgentHub sync checks → update shared context → flag blockers immediately → enforce one-session-per-file ownership.

**Completing**: Commit per branch → PR to integration branch (in order) → resolve conflicts → full test suite → merge to main.

## Anti-Patterns to Avoid

- **Cross-editing**: Multiple sessions editing the same file — assign clear file ownership
- **Orphan worktrees**: Always clean up with `git worktree remove` when done
- **Context loss**: Always create handoff documents between pipeline stages
- **Silent failures**: Document session failures for the next session to pick up

## When to Split vs Keep Single

**Split**: Task exceeds ~100K tokens, multiple independent components, different expertise needed (debug vs build).

**Keep single**: Tightly coupled changes, complex state to maintain, quick iterations needed.
