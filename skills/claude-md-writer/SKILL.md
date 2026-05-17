---
name: claude-md-writer
description: "Author and refactor CLAUDE.md and manage .claude/rules/. Analyzes the codebase to produce a best-practice-aligned, efficient CLAUDE.md plus path-scoped rules. Usage: /claude-md-writer [create|refactor|rules]"
argument-hint: "[create|refactor|rules]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion
---

# CLAUDE.md Writer

Analyze the codebase to author or refactor an efficient, best-practice-aligned CLAUDE.md, and manage `.claude/rules/` path-scoped rule files.

## Mode Selection

Determine the mode based on `$ARGUMENTS`:
- `create`, or no argument + no existing CLAUDE.md → **Create mode**
- `refactor`, or no argument + existing CLAUDE.md → **Refactor mode**
- `rules` → **Rules management mode**

## Core Principles

**Apply these principles to every task without exception:**

1. **Stay under 200 lines** — Never exceed 200 lines per file. Longer files make Claude more likely to ignore the instructions.
2. **Only what code cannot reveal** — For every line, ask "If I remove this, will Claude make a mistake?"
3. **Executable commands** — Every command must be copy-paste runnable.
4. **Project-specific information only** — Don't include general best practices or language defaults.
5. **Hierarchical separation** — Root CLAUDE.md holds project-wide context, nested CLAUDE.md files hold domain context, rules hold conditional guidance.

### What to include

- Build/test/deploy commands (Claude cannot guess these)
- Code style rules that differ from defaults
- Architectural decisions and their rationale (code shows WHAT, CLAUDE.md explains WHY)
- Environment variables and setup requirements
- Gotchas (counterintuitive behavior, common mistakes)
- Workflows (branch naming, PR conventions)

### What never to include

- Per-file descriptions (readable from the code itself)
- Standard language conventions (Claude already knows these)
- Verbose explanations or tutorials
- API documentation (link to it instead)
- Information that changes frequently
- Self-evident directives like "write clean code"

## Phase 1: Discovery

### 1.1 Locate existing files

```bash
find . -name "CLAUDE.md" -o -name ".claude.local.md" 2>/dev/null | head -50
```

```bash
ls -la .claude/rules/ 2>/dev/null
```

### 1.2 Detect the project type

Inspect the following to determine the project type:

| File/Directory | Project Type |
|-------------|-------------|
| `package.json` | Node.js/Frontend |
| `build.gradle.kts`, `pom.xml` | JVM (Kotlin/Java) |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `pyproject.toml`, `requirements.txt` | Python |
| `Dockerfile`, `docker-compose.yml` | Container |
| `kustomization.yaml`, `Chart.yaml` | Kubernetes/Helm |
| `terraform/`, `*.tf` | IaC |

### 1.3 Pick the language

- If a CLAUDE.md already exists → match its language
- Otherwise → ask the user with AskUserQuestion

## Phase 2: Analysis

Use Explore agents to analyze the codebase in parallel.

### Analysis targets

1. **Build system** — build/test/lint commands and scripts
2. **Architecture** — directory layout, core patterns, module relationships
3. **Environment** — required env vars, config files, dependencies
4. **Workflow** — CI/CD, branching strategy, deployment process
5. **Existing docs** — README, prior CLAUDE.md, comments

## Phase 3: Interview

Use AskUserQuestion to collect information that cannot be inferred from the code.

**Must ask:**
- Team/personal workflow rules (branch naming, PR conventions, etc.)
- Gotchas not visible in the code
- The reasoning (WHY) behind architectural decisions
- Any special coding conventions

**No need to ask:**
- Structure that can be read from the code
- Standard configuration

## Phase 4: Generate / Refactor

### Create mode

Use [references/section-guide.md](references/section-guide.md) as a reference to author the CLAUDE.md.

**Structure template:**

```markdown
# project-name

One-line description.

## Commands

| Command | Description |
|-------|------|
| `command` | description |

## Architecture

\```
dir/    # purpose
\```

## Code Style

- Only rules that differ from defaults

## Gotchas

- gotcha 1
- gotcha 2
```

### Refactor mode

1. Read the existing CLAUDE.md
2. Evaluate it against [references/best-practices.md](references/best-practices.md)
3. Present the issues to the user:
   - Unnecessary content (anything readable from the code)
   - Missing content (anything code cannot reveal)
   - Whether the file exceeds 200 lines
4. Apply changes after user approval

### Nested CLAUDE.md

For monorepos or large projects, split CLAUDE.md across subdirectories:
- root: overall project context, common commands
- nested: domain-specific rules, module-local information

## Phase 5: Rules Management

`.claude/rules/*.md` files are conditional rules that load only when working on files matching specific paths.

### When rules are a good fit

- Rules that apply only to a specific file type (e.g., API endpoint conventions)
- Patterns that apply only to a specific directory (e.g., test authoring rules)
- Rules whose scope is too narrow to belong in CLAUDE.md

### Rules file format

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
---

# API conventions

- Validate input on every endpoint
- Use the standard error response format
```

### Rules creation workflow

1. Identify the project's primary domains/directories
2. Identify patterns that recur within each domain
3. Extract rules from CLAUDE.md that can be scoped to a path
4. Propose to the user and create after approval

## Final Verification

After authoring or editing, always confirm:

- [ ] Is each CLAUDE.md under 200 lines?
- [ ] Are all commands runnable?
- [ ] Have you avoided anything readable from the code itself?
- [ ] Does it contain only project-specific information?
- [ ] Do the `paths` in rules files match real, existing paths?
