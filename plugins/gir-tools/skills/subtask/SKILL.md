---
name: subtask
description: Parallel task execution using Git worktrees. Use this skill when the user wants to work on multiple features simultaneously, spawn parallel subagents, or distribute work across isolated environments. Enables faster development by running tasks concurrently while preventing code conflicts.
---

# Subtask Skill

Distribute work across multiple parallel tasks using Git worktrees. Each task runs in isolation, enabling concurrent development without conflicts.

## Prerequisites

**The Subtask CLI must be installed manually.** GIR will initialize and configure Subtask, but cannot install the CLI itself.

### Install Subtask CLI

```bash
# Option 1: Install script (recommended)
curl -fsSL https://subtask.dev/install.sh | bash

# Option 2: Homebrew
brew install zippoxer/tap/subtask

# Option 3: Go
go install github.com/zippoxer/subtask/cmd/subtask@latest
```

### After CLI Installation

Run these once per project:
```bash
subtask init              # Initialize in repository
subtask install --project # Install Claude Code skill
```

More info: https://github.com/zippoxer/subtask

## When to Use

- **Multiple independent features**: "Build login and dashboard simultaneously"
- **Parallel bug fixes**: "Fix these 3 bugs in parallel"
- **Concurrent reviews**: "Review and refactor different modules"
- **Feature + tests**: "Implement feature while writing tests"
- **Exploration + implementation**: "Research approach while prototyping"

## Core Commands

### Task Management

```bash
subtask draft <task-name>     # Create new task for subagent
subtask list                  # Show all tasks with status
subtask <task-name>           # Open TUI for specific task
```

### System

```bash
subtask install               # Install Claude Code skill
subtask init                  # Initialize in repository
subtask update --check        # Check for updates
subtask update                # Install latest version
```

## Task Lifecycle

1. **Draft** → Task created, awaiting subagent
2. **Working** → Subagent executing in worktree
3. **Replied** → Subagent complete, awaiting review

## Invocation Patterns

### Natural Language Triggers

- "fix the login bug with Subtask"
- "lets do these 3 features with Subtask"
- "implement X, Y, Z in parallel"
- "spawn agents for each module"

### Process Flow

1. User requests parallel work
2. Claude invokes Subtask skill
3. Create tasks via `subtask draft`
4. Subagents spawn in isolated worktrees
5. Monitor progress in TUI (`subtask <name>`)
6. Review completed work
7. Merge or request modifications

## Best Practices

### Task Naming

```bash
subtask draft fix-auth-redirect      # Clear, specific
subtask draft implement-user-api     # Action-oriented
subtask draft refactor-db-queries    # Describes scope
```

### Ideal Task Size

- **Good**: Single feature, specific bug, module refactor
- **Too small**: Typo fix, single-line change
- **Too large**: Full system rewrite, multi-module feature

### Parallelization Strategy

| Scenario | Approach |
|----------|----------|
| 2-3 independent tasks | Draft all, monitor in TUI |
| Dependent tasks | Complete prerequisite first |
| Large refactor | Split by module/domain |
| Feature + tests | Parallel if test interface known |

## Integration with GIR

### With Explore-Plan-Code-Commit

**Plan phase**: Identify parallelizable work → Draft subtasks

**Code phase**: Subagents execute in worktrees → Monitor and review

**Commit phase**: Merge completed subtasks → Single or separate commits

### With Ralph Loops

Combine for iterative parallel work:
- Each subtask can run its own Ralph loop
- Main Claude coordinates and reviews

### With Code-Reviewer

After subtask completion:
1. Review each task's diff in TUI
2. Deploy code-reviewer for merged changes
3. Address issues before final commit

## TUI Features

- **Progress monitoring**: Real-time status updates
- **Diff viewing**: See changes per task
- **Conversation history**: Review subagent reasoning
- **Merge decisions**: Accept/reject/modify

## Limitations

- Tasks must be truly independent (no shared file conflicts)
- Each task operates on separate branch/worktree
- Merge conflicts require manual resolution
- Network-dependent for subagent communication

## Quick Reference

| Action | Command |
|--------|---------|
| Create task | `subtask draft <name>` |
| List all | `subtask list` |
| Monitor | `subtask <name>` |
| Install skill | `subtask install` |
| Initialize repo | `subtask init` |
