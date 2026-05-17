# Section-by-Section Authoring Guide

Guidance on what to include and exclude in each section of CLAUDE.md.
Not every section is required — use only the ones relevant to your project.

## Project Description

**Keep it to one line.** Give Claude context about what the project is.

```markdown
# my-project

Spring Boot social-auth API server. Uses PostgreSQL + Redis.
```

- A brief tech stack mention helps Claude apply appropriate patterns
- No need to explain business logic

## Commands

**The most important section.** If Claude can't build or test, productivity plummets.

```markdown
## Commands

| Command | Description |
|-------|------|
| `./gradlew build` | Full build |
| `./gradlew test` | Run tests |
| `./gradlew :module:test --tests "*.ClassName"` | Run a single test |
| `npm run dev` | Dev server (port 3000) |
```

- Include project-specific flags even for standard commands
- Always include how to run a single test

## Architecture

**Just the key directories and their purpose.** Don't go down to the file level.

```markdown
## Architecture

\```
src/
├── api/          # REST controllers
├── domain/       # Business logic (DDD)
├── infra/        # External system integrations
└── config/       # Configuration classes
\```
```

- Limit to 3-5 depth
- Omit the comment when the purpose is obvious from the name

## Code Style

**Only rules that differ from defaults.** Don't write language defaults like "use camelCase".

```markdown
## Code Style

- Always use the `ErrorResponse` sealed class for error responses
- Repository method names: `findByXxx` (follows Spring Data convention)
- Per-environment settings only in `application-{profile}.yml`
```

## Environment

**Only what setup requires.**

```markdown
## Environment

Required:
- Java 21+
- Docker (for running PostgreSQL, Redis locally)
- `KAKAO_REST_API_KEY` — for Kakao OAuth

Setup:
- `cp .env.example .env` then fill in the values
```

## Gotchas

**The highest-value section.** Information that isn't visible in the code and prevents mistakes.

```markdown
## Gotchas

- Do not delete PVCs — risk of data loss
- The `staging` namespace is shared by multiple apps
- Always pin versions for external Helm charts
- Changes to `base/` affect every environment
```

## Workflow

**Only when team rules exist.**

```markdown
## Workflow

- Branches: `feature/ISSUE-123-description`
- PR merges: squash merge only
- Do not push directly to main
```

## Secrets Management

**Only the location and management approach. Do not include actual values (except in private repos).**

```markdown
## Secrets

- `postgresql-secret` (namespace: prod) — DB credentials
- `.env` — for local development (gitignored)
```

## Rules Files (.claude/rules/)

Path-scoped rules that can be extracted from CLAUDE.md.
Best for rules that apply only to specific file types or directories.

```yaml
# .claude/rules/api-design.md
---
paths:
  - "src/api/**/*.kt"
---

# API conventions

- Validate input on every endpoint
- Error responses use the ErrorResponse sealed class
- No business logic in controllers
```

### Rules vs CLAUDE.md criteria

| Criterion | CLAUDE.md | Rules |
|------|-----------|-------|
| Scope | Entire project | Specific paths only |
| Load timing | Always | When working on matching files |
| Token cost | Always consumed | Only when needed |
| Examples | Build commands, overall architecture | API conventions, test patterns |
