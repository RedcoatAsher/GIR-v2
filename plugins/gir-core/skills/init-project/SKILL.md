# /init-project

Generate a `CLAUDE-project.md` for the current project by detecting its tech stack and prompting for key configuration values.

---

## Instructions

Follow these steps in order:

### Step 1: Detect the tech stack

Look for these files in the current directory and read them if they exist:

- `package.json` — Node.js/JavaScript/TypeScript project; extract `name`, `scripts`, `dependencies`, `devDependencies`
- `Cargo.toml` — Rust project; extract `[package]` name and dependencies
- `go.mod` — Go project; extract module name and Go version
- `pyproject.toml` — Python project; extract project name and dependencies
- `requirements.txt` — Python project (legacy); note Python is the language
- `pom.xml` — Java/Maven project; extract `<artifactId>` and `<dependencies>`
- `build.gradle` or `build.gradle.kts` — Java/Kotlin/Gradle project
- `mix.exs` — Elixir project
- `Gemfile` — Ruby project

From what you find, determine:
- Primary language and version (if detectable)
- Framework in use (Next.js, FastAPI, Express, etc.)
- Package manager (`npm`, `pnpm`, `yarn`, `cargo`, `pip`, `go`, etc.)
- Common dev/test/build/lint commands from `scripts` in `package.json` or equivalent

### Step 2: Ask the user for configuration values

Present your detected values and ask the user to confirm or override:

```
I detected the following for your project:

  Project name:   [detected or "unknown"]
  Language:       [detected or "unknown"]
  Framework:      [detected or "unknown"]
  Package mgr:    [detected or "unknown"]

  Dev command:    [detected, e.g. "npm run dev" or "unknown"]
  Build command:  [detected, e.g. "npm run build" or "unknown"]
  Test command:   [detected, e.g. "npm run test" or "unknown"]
  Lint command:   [detected, e.g. "npm run lint" or "unknown"]

Please confirm these or provide your preferred values.
(Press Enter to accept defaults, or type a replacement for any line.)
```

Wait for the user's response before proceeding.

### Step 3: Generate `CLAUDE-project.md`

Using the confirmed values, write `CLAUDE-project.md` in the current directory. Use the template structure below, substituting detected/confirmed values into the placeholder fields. Replace `[placeholder]` text with actual values where you have them, and leave the bracketed instructions for fields the user should complete manually.

---

## Output Template

The generated file must follow this exact structure:

````markdown
# CLAUDE-project.md

> **Project-specific configuration** — Customize for your project

<!-- GIR-SYNC: This file uses markers to enable smart merging during gir-sync updates.
     - Content between GIR:START/END markers may be updated by gir-sync
     - Content between PROJECT:START/END markers is preserved during sync
     - Edit PROJECT sections freely; avoid editing GIR sections directly -->

---

## Project Overview

<!-- PROJECT:START overview -->
**Name**: [PROJECT_NAME]

**Type**: [Choose one: web-app | mobile-app | api | library | cli-tool | desktop-app | other]

**Audience**: [Who uses this? Examples: SMBs, enterprises, developers, consumers, internal teams]

**Description**: [1-2 sentence description of what this project does]
<!-- PROJECT:END overview -->

---

## Tech Stack

<!-- GIR:START tech-stack-guidance -->
> **Guidance**: Be specific with versions when they matter for compatibility. Use "latest" only if you truly want bleeding edge updates.
>
> **Good examples**:
> - "Next.js 15.2.x (App Router)" — specific version, clear router choice
> - "React Native 0.73.x" — version matters for native modules
> - "FastAPI 0.104.x" — Python async framework
>
> **Avoid**: "Next.js", "latest React", "newest version"
<!-- GIR:END tech-stack-guidance -->

### Primary Stack
<!-- PROJECT:START tech-stack -->
```
Framework:    [DETECTED_FRAMEWORK]
UI Library:   [e.g., React 19.x, SwiftUI, Flutter 3.x, Vue 3.x]
Language:     [DETECTED_LANGUAGE]
Styling:      [e.g., Tailwind CSS v4, styled-components 6.x, CSS Modules]
Animations:   [e.g., Framer Motion, react-spring, Core Animation, Lottie]
Components:   [e.g., shadcn/ui, Material-UI, native components, custom design system]
```
<!-- PROJECT:END tech-stack -->

### Additional Tools
<!-- GIR:START tools-guidance -->
> **Guidance**: Only list tools you actively use and want Claude to know about
<!-- GIR:END tools-guidance -->

<!-- PROJECT:START additional-tools -->
```
State Mgmt:   [e.g., Zustand, Redux Toolkit, Context API, Combine, Riverpod, MobX]
Forms:        [e.g., React Hook Form, Formik, native forms, Zod validation]
Testing:      [e.g., Vitest, Jest, XCTest, pytest, Go test, Playwright]
API Client:   [e.g., React Query, SWR, Axios, Alamofire, Retrofit, fetch]
Analytics:    [e.g., Vercel Analytics, Firebase, Mixpanel, PostHog, Amplitude]
Deployment:   [e.g., Vercel, AWS, TestFlight, Google Play, Docker, Kubernetes]
Package Mgr:  [DETECTED_PACKAGE_MANAGER]
Database:     [e.g., PostgreSQL, MongoDB, SQLite, Supabase, Firebase, Redis]
ORM/Query:    [e.g., Prisma, Drizzle, SQLAlchemy, GORM, Sequelize]
```
<!-- PROJECT:END additional-tools -->

---

## Commands

<!-- GIR:START commands-guidance -->
> **Guidance**: List ALL commands developers need to know. Include descriptions.
<!-- GIR:END commands-guidance -->

<!-- PROJECT:START commands -->
```bash
[DEV_COMMAND]      # Start development server
[BUILD_COMMAND]    # Production build
[TEST_COMMAND]     # Run test suite
[LINT_COMMAND]     # Run linter
[deploy-command]   # [e.g., "Deploy to production"]
[db-command]       # [e.g., "Run database migrations"]
```
<!-- PROJECT:END commands -->

<!-- GIR:START commands-example -->
**Example**:
```bash
npm run dev        # Start Next.js development server
npm run build      # Production build
npm run test       # Run Vitest tests
npm run lint       # ESLint + Prettier
npm run db:push    # Push Prisma schema changes
```
<!-- GIR:END commands-example -->

---

## Project Structure

<!-- GIR:START structure-guidance -->
> **Guidance**: Show your ACTUAL folder structure. Use comments to explain non-obvious directories.
> Include only top-level and important subdirectories. Don't list every file.
<!-- GIR:END structure-guidance -->

<!-- PROJECT:START project-structure -->
```
[project-root]/
├── [dir1]/          # [Purpose, e.g., "React components"]
│   ├── [subdir]/    # [Purpose, e.g., "Reusable UI primitives"]
│   └── [subdir]/    # [Purpose, e.g., "Feature-specific components"]
├── [dir2]/          # [Purpose, e.g., "API routes and endpoints"]
├── [dir3]/          # [Purpose, e.g., "Database schema and migrations"]
├── [dir4]/          # [Purpose, e.g., "Shared utilities and helpers"]
└── [dir5]/          # [Purpose, e.g., "Static assets"]
```
<!-- PROJECT:END project-structure -->

<!-- GIR:START structure-example -->
**Example (Next.js)**:
```
my-app/
├── app/             # Next.js App Router pages
│   ├── api/         # API routes
│   └── (auth)/      # Route groups
├── components/      # React components
│   ├── ui/          # shadcn/ui primitives
│   └── features/    # Feature-specific components
├── lib/             # Utilities, helpers, config
├── public/          # Static assets
└── prisma/          # Database schema
```
<!-- GIR:END structure-example -->

---

## Key Patterns

<!-- GIR:START patterns-guidance -->
> **Guidance**: Document project-specific conventions that differ from standard practices.
> These help Claude understand your codebase's unique patterns.
<!-- GIR:END patterns-guidance -->

<!-- PROJECT:START key-patterns -->
- **[Pattern name]**: [Description]
  - Example: "Color system: oklch with CSS variables (--primary, --background, --foreground)"

- **[Pattern name]**: [Description]
  - Example: "Client components: Always use 'use client' directive at top of file"

- **[Pattern name]**: [Description]
  - Example: "API routes: Use server actions in app/actions/ instead of app/api/"
<!-- PROJECT:END key-patterns -->

<!-- GIR:START patterns-examples -->
**Common patterns to document**:
- Color/theme system
- Component architecture (server vs client, composition patterns)
- Data fetching strategy
- Error handling approach
- Authentication flow
- File naming conventions
<!-- GIR:END patterns-examples -->

---

## Path Aliases

<!-- GIR:START aliases-guidance -->
> **Guidance**: Copy from your tsconfig.json, jsconfig.json, or build config
<!-- GIR:END aliases-guidance -->

<!-- PROJECT:START path-aliases -->
```typescript
"@/*" → [maps to, e.g., "./src/*", "./*", "./app/*"]
"@components/*" → [maps to, e.g., "./src/components/*"]
"@lib/*" → [maps to, e.g., "./src/lib/*"]
"@/utils" → [maps to, e.g., "./src/utils"]
```
<!-- PROJECT:END path-aliases -->

---

## Environment Variables

<!-- GIR:START env-guidance -->
> **Guidance**: List ALL environment variables. Mark which are required vs optional.
> Include brief description of what each is used for and where to get values.
<!-- GIR:END env-guidance -->

<!-- PROJECT:START env-vars -->
```bash
# === Required ===
[VAR_NAME]=              # [Description: what it's for, where to get it]

# === Optional ===
[VAR_NAME]=              # [Description and when you'd use it]

# === Development Only ===
[VAR_NAME]=              # [Description]
```
<!-- PROJECT:END env-vars -->

---

## Session Initialization Baseline

<!-- GIR:START baseline-guidance -->
> **Guidance**: This baseline is used for drift detection at session start.
> Update these values whenever your project structure changes significantly.
>
> **How to get these values**:
> ```bash
> # Scripts
> jq '.scripts | keys' package.json
>
> # Dependency count
> jq '((.dependencies // {}) + (.devDependencies // {})) | length' package.json
>
> # Top directories
> fd -t d -d 1 --exclude node_modules --exclude .git
>
> # Config files
> fd -t f -d 1 -e json -e js -e ts --exclude package.json --exclude node_modules
>
> # Environment variables
> rg "^[A-Z_]+=" .env.example | cut -d= -f1
> ```
<!-- GIR:END baseline-guidance -->

<!-- PROJECT:START session-baseline -->
```yaml
scripts: [list your npm/yarn scripts, e.g., "dev", "build", "test", "lint"]
dep_count: ~[number]                    # Total dependencies + devDependencies (flag if ±10)
top_dirs: [list top-level folders, e.g., "app", "components", "lib", "public"]
env_vars: [list required env var names, e.g., "DATABASE_URL", "API_KEY"]
config_files: [list config files in root, e.g., "tsconfig.json", "next.config.js"]
last_verified: [YYYY-MM-DD]
```
<!-- PROJECT:END session-baseline -->

---

## Build Configuration

<!-- GIR:START build-guidance -->
> **Guidance**: Note any special build settings, ignored errors, or optimizations.
<!-- GIR:END build-guidance -->

<!-- PROJECT:START build-config -->
- **[Setting name]**: [Why it's configured this way]
<!-- PROJECT:END build-config -->

---

## Git Workflow (Project-Specific)

<!-- GIR:START git-guidance -->
> **Guidance**: Document YOUR git workflow and branch strategy.
<!-- GIR:END git-guidance -->

### Branch Strategy
<!-- PROJECT:START git-branch-strategy -->
```bash
[main-branch]      # [e.g., "main" - production-ready code]
└── [dev-branch]   # [e.g., "dev" - integration branch (optional)]
    └── [feature-prefix]/xyz   # [e.g., "feat/", "fix/", "chore/"]
```
<!-- PROJECT:END git-branch-strategy -->

### Custom Git Rules

<!-- PROJECT:START git-rules -->
- **[Rule]**: [Description]
<!-- PROJECT:END git-rules -->

---

## Installed MCP Tools

<!-- GIR:START mcp-guidance -->
> **Guidance**: List MCP servers you've installed BEYOND the core set.
<!-- GIR:END mcp-guidance -->

<!-- PROJECT:START mcp-tools -->
```
Installed:
- [tool-name]     # [Why you need it in this project]

Not Installed:
- [tool-name]     # [Why not needed]
```
<!-- PROJECT:END mcp-tools -->

---

## Active Agent Categories

<!-- GIR:START agents-guidance -->
> **Guidance**: List which agent categories from `.claude/agents/` you're using.
<!-- GIR:END agents-guidance -->

<!-- PROJECT:START agent-categories -->
```
Active Categories:
- core/           # Always include (feature-architect, code-reviewer, debugger)
- [category]/     # [e.g., "web/" for Next.js/React/Vercel projects]

Not Using:
- [category]/     # [Why not applicable]
```
<!-- PROJECT:END agent-categories -->

---

## Delegation Configuration (Optional)

<!-- PROJECT:START delegation-config -->
**Current Mode**: `balanced` (default)

### Uncomment to Customize

```yaml
# delegation:
#   mode: "balanced"  # Options: aggressive | balanced | minimal
#   auto_explore_threshold: 5
#   auto_plan_threshold: 8
#   auto_parallel_threshold: 2
#   max_parallel_agents: 3
#   always_code_review: true
#   always_sequential_thinking: true
```
<!-- PROJECT:END delegation-config -->

---

## Project-Specific Checklist Items

<!-- PROJECT:START project-checklist -->
Before completing tasks:
- [ ] [Project check, e.g., "Passes accessibility audit (WCAG 2.1 AA)"]
- [ ] [Project check, e.g., "Lighthouse performance score >90"]
- [ ] [Project check, e.g., "Database migrations applied successfully"]
<!-- PROJECT:END project-checklist -->

---

## Additional Notes

<!-- PROJECT:START additional-notes -->
[Free-form notes, links to external docs, team conventions, etc.]
<!-- PROJECT:END additional-notes -->
````

---

### Step 4: Confirm and guide next steps

After writing the file, tell the user:

```
Created CLAUDE-project.md in the current directory.

Sections that need your attention:
- Project Overview: Add your project name, type, audience, and description
- Tech Stack: Verify/complete the detected values and add missing tools
- Project Structure: Replace the placeholder structure with your actual directory layout
- Path Aliases: Copy from your tsconfig.json or jsconfig.json if applicable
- Environment Variables: List all required and optional env vars
- Session Baseline: Run the suggested commands to fill in dep_count, scripts, top_dirs
- Git Workflow: Document your branch strategy and any custom rules

Tip: Run /gir-core:init-memory-bank to also scaffold the .gir/ memory bank directory.
```
