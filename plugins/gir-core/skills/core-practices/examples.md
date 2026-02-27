# CLAUDE-examples.md

> Code examples referenced by CLAUDE-core.md

## Session Initialization

### Drift Check Trigger
```bash
[ ! -f .gir/CLAUDE-activeContext.md ] || \
[ $(find .gir/CLAUDE-activeContext.md -mtime +0 2>/dev/null | wc -l) -gt 0 ] && \
echo "Running drift check..." || echo "Context fresh, skipping"
```

### Drift Check Commands
```bash
# List scripts/commands, count deps, find configs, list dirs, extract env vars
# Compare against CLAUDE-project.md baseline
```

## MCP Tool Patterns

### Gemini-CLI
```bash
gemini -p "explain the data flow" < largefile.js
cat src/components/*.tsx | gemini -p "find unused props"
gemini -p "review for performance issues" < hero-section.tsx
gemini -p "brainstorm approaches for [problem]"
```

### Sequential-Thinking
```typescript
// User: "Add authentication with JWT and refresh tokens"
// Correct: Use sequential-thinking FIRST
// Wrong: Jump to coding or TodoWrite
```

## Command Efficiency

### Banned vs Required
```bash
# BANNED               # USE INSTEAD
tree                   fd . -t f  # Files
find                   fd "name"  # Find by name
grep -r                rg "pattern"  # Search content
ls -R                  fd . -t d  # Dirs
cat | grep             rg pattern file
```

### Ripgrep Examples
```bash
rg "pattern"                    # Search all
rg -i "pattern"                 # Case insensitive
rg -t ts "pattern"              # Filter by type
rg -l "pattern"                 # Files only
rg -c "pattern"                 # Count matches
rg -A3 -B3 "pattern"            # With context
rg "(pat1|pat2)"                # Multiple patterns
```

### fd Examples
```bash
fd . -t f                       # All files
fd . -t d                       # All dirs
fd -e js                        # By extension
fd -e tsx -x wc -l {}           # With command
```

## GitHub CLI

### PRs
```bash
gh pr create --title "feat: JWT auth" --body "$(git log main..HEAD --format='- %s')"
git diff main...HEAD | claude -p "write PR description"
```

### Review
```bash
gh pr view 123 --comments
gh pr create --reviewer @security-team
```

### Issues
```bash
git commit -m "fix: resolve auth bug (fixes #123)"
```

## Git History
```bash
git log -p -- src/auth/ | head -500      # Why implemented this way?
git log --all --grep="auth" --oneline    # When added?
rg "implement auth" $(git log --all --format=%H -- src/auth/)
```

## Workflow Patterns

### Planning Phase
```
1. Invoke sequential-thinking (REQUIRED for complex tasks)
2. Use output to create TodoWrite plan
3. Present to user for approval
4. Adjust based on feedback
```

### Multi-Claude Quick Reference
```bash
# Same codebase, different concerns
Terminal 1: Main implementation
Terminal 2: /agent code-reviewer
Terminal 3: /agent debugger

# Isolated work (git worktrees)
git worktree add .worktrees/feature-a feature-a
git worktree add .worktrees/feature-b feature-b
```

### Git Strategy
```bash
main              # Production-ready
└── feat/xyz      # Features
└── fix/xyz       # Bug fixes
└── chore/xyz     # Maintenance
```

### Commit Format
```
feat: add contact form validation
fix: resolve mobile nav z-index
fix(hero): correct animation timing
chore: update dependencies
chore(config): adjust Tailwind theme
```
Prefixes: `feat` | `fix` | `chore` | `docs` | `style` | `refactor`

## Anti-Patterns

### Code Quality
```
Inline styles        → Use styling system
any type             → Properly type
console.log          → Remove before commit
Commented code       → Delete (git has history)
Magic numbers        → Use constants
Giant components     → Split at ~150 lines
Prop drilling >2     → Use context/composition
useEffect derived    → Use useMemo/inline
```

### UX Dark Patterns (Never)
```
Fake urgency         → Real scarcity only
Confirmshaming       → Neutral opt-out
Hidden costs         → Transparent pricing
Forced continuity    → Clear opt-out
Misleading buttons   → Honest CTAs
Privacy defaults     → Opt-in only
Roach motel          → Easy cancellation
Misdirection         → Clear info hierarchy
```

## Effective Prompting

**Bad**: "Add authentication"

**Good**: "Add JWT auth with httpOnly cookies, login/logout at /api/auth/*, middleware for /api/user/*"

## Project-Specific Examples

**Updated**: 2026-01-10
