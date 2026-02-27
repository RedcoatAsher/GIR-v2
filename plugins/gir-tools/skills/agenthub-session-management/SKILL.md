---
name: agenthub-session-management
description: Patterns for managing multiple Claude Code sessions using AgentHub
---

# AgentHub Session Management

Skills for coordinating, monitoring, and optimizing multi-session Claude Code workflows.

## Session Lifecycle

### Session States
```text
NEW → THINKING → EXECUTING_TOOL → WAITING_FOR_USER → IDLE
         ↓              ↓                ↓
    AWAITING_APPROVAL   ↓                ↓
         ↓              ↓                ↓
         └──────────────┴────────────────┘
                        ↓
                    COMPLETED
```

### State Meanings

| State | Description | User Action |
|-------|-------------|-------------|
| Thinking | Processing request | Wait |
| Executing Tool | Running tool operation | Wait |
| Awaiting Approval | Needs confirmation | Approve/Reject |
| Waiting for User | Needs input | Provide input |
| Idle | Ready for new task | Send request |

## Session Data Structure

### Location
```text
~/.claude/projects/{encoded-path}/{sessionId}.jsonl
```

### Encoded Path Format
Project path is URL-encoded:
- `/Users/dev/myproject` → `%2FUsers%2Fdev%2Fmyproject`

### JSONL Entry Structure
Each line in the session file is a JSON object:
```json
{
  "type": "message|tool_use|tool_result",
  "timestamp": "ISO-8601",
  "content": {...}
}
```

## Multi-Session Patterns

### Pattern 1: Feature Parallel
Best for large features with independent components.

```text
┌─────────────────────────────────────────────────┐
│                  Main Feature                    │
├─────────────┬─────────────┬─────────────────────┤
│  Session 1  │  Session 2  │     Session 3       │
│   Backend   │   Frontend  │     Tests           │
│  (worktree) │  (worktree) │    (worktree)       │
├─────────────┴─────────────┴─────────────────────┤
│              Integration Branch                  │
└─────────────────────────────────────────────────┘
```

**Setup**:
```bash
# Create feature branches
git checkout -b feature/main
git push -u origin feature/main

# Create component branches
git checkout -b feature/backend feature/main
git checkout -b feature/frontend feature/main
git checkout -b feature/tests feature/main

# Create worktrees
git worktree add ../proj-backend feature/backend
git worktree add ../proj-frontend feature/frontend
git worktree add ../proj-tests feature/tests
```

### Pattern 2: Bug Swarm
Multiple sessions tackling different bugs simultaneously.

```text
┌───────────┬───────────┬───────────┐
│  Bug #1   │  Bug #2   │  Bug #3   │
│ Session A │ Session B │ Session C │
│ bugfix/1  │ bugfix/2  │ bugfix/3  │
└─────┬─────┴─────┬─────┴─────┬─────┘
      │           │           │
      └───────────┼───────────┘
                  ↓
              main branch
```

**Best for**: Sprint bug bashes, hotfix situations

### Pattern 3: Exploration Fan-Out
Multiple sessions exploring different solutions.

```text
                 Problem
                    │
      ┌─────────────┼─────────────┐
      ↓             ↓             ↓
  Approach A    Approach B    Approach C
  Session 1     Session 2     Session 3
      │             │             │
      └─────────────┼─────────────┘
                    ↓
            Best Solution Selected
                    ↓
              Main Session
```

**Best for**: Architecture decisions, algorithm selection

### Pattern 4: Pipeline
Sequential handoffs between specialized sessions.

```text
Session 1        Session 2        Session 3
[Architect] ──→  [Implement] ──→   [Review]
    │                 │                │
    ↓                 ↓                ↓
  Plan.md         Code + Tests    Feedback
```

**Handoff Document**:
```markdown
## Handoff: [From] → [To]
**Date**: YYYY-MM-DD
**Task**: [description]

### Completed
- [x] Item 1
- [x] Item 2

### In Progress
- [ ] Item 3 (started, needs completion)

### Pending
- [ ] Item 4

### Key Files
- `path/to/file.ts` - [description]

### Notes
- [important context]
- [decisions made]

### Blockers
- [any blockers for next session]
```

## Git Worktree Best Practices

### Creating Worktrees
```bash
# Always branch from a clean state
git fetch origin
git checkout main
git pull

# Create feature branch first
git checkout -b feature/name

# Then create worktree
git worktree add ../project-feature feature/name
```

### Worktree Hygiene
```bash
# List all worktrees
git worktree list

# Prune stale worktrees
git worktree prune

# Remove specific worktree
git worktree remove ../project-feature

# Force remove (with uncommitted changes)
git worktree remove --force ../project-feature
```

### Worktree + Session Mapping
Maintain a mapping document:
```markdown
## Active Worktrees

| Worktree Path | Branch | Session ID | Owner | Status |
|---------------|--------|------------|-------|--------|
| ../proj-auth | feature/auth | abc123 | @dev1 | Active |
| ../proj-api | feature/api | def456 | @dev2 | Idle |
```

## Session Monitoring

### Health Checks
```bash
# Find sessions active in last hour
find ~/.claude/projects -name "*.jsonl" -mmin -60

# Count active sessions per project
for dir in ~/.claude/projects/*/; do
  count=$(ls "$dir"*.jsonl 2>/dev/null | wc -l)
  echo "$(basename "$dir"): $count sessions"
done
```

### Search Across Sessions
```bash
# Find error patterns
grep -r "error" ~/.claude/projects/ --include="*.jsonl" | head -20

# Find tool usage
grep -r "tool_use" ~/.claude/projects/ --include="*.jsonl" | \
  jq -r '.content.name' 2>/dev/null | sort | uniq -c | sort -rn
```

## Coordination Protocols

### Starting Parallel Work
1. Create integration branch
2. Branch off for each session
3. Create worktrees
4. Document session assignments
5. Set merge order/dependencies

### During Parallel Work
1. Regular sync checks via AgentHub
2. Update shared context docs
3. Flag blockers immediately
4. No cross-session file edits

### Completing Parallel Work
1. Each session commits to own branch
2. PR to integration branch (in order)
3. Resolve conflicts per merge
4. Full test suite on integration
5. Merge to main

## Anti-Patterns

### Avoid
- **Cross-editing**: Multiple sessions editing same file
- **Orphan worktrees**: Not cleaning up after completion
- **Silent failures**: Not documenting session failures
- **Context loss**: No handoff documentation

### Prefer
- **Clear ownership**: One session per file/component
- **Clean cleanup**: Remove worktrees when done
- **Visible status**: Use AgentHub monitoring
- **Rich handoffs**: Detailed transition docs

## Token Optimization

### When to Split Sessions
- Task exceeds ~100K tokens of context
- Multiple independent components
- Different expertise needed (debug vs build)

### When to Keep Single Session
- Tightly coupled changes
- Complex state to maintain
- Quick iterations needed

## AgentHub-Specific Features

### Menu Bar Mode
- Quick status overview
- Session switching
- Minimal screen usage

### Popover Mode
- Detailed session info
- Multi-session comparison
- Diff previews

### Cross-Session Search
- Search by content
- Filter by state
- Filter by project
- Time-based filtering
