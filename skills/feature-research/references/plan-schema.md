# Plan File Schema — Shared Contract

The Plan file at `.claude/plans/<slug>.md` is the single source of truth shared by the 4 feature-* skills (`feature-research`, `feature-work`, `feature-test`, `feature-review`) and the `new-feature` wrapper.

Each skill **reads the Plan by path** and never embeds its body into an agent prompt. Every skill updates only its own status field and its designated sections.

## File Structure

```markdown
---
slug: <feature-slug>
status:
  research: done|pending
  work: done|pending
  test: done|pending
  review: done|pending
---

## Goal
<1–2 sentences stating what is built and why>

## Strategy
<approach and rationale; note any alternatives considered>

## Change Plan
| File | Change Type | Concrete Change |
|------|-------------|-----------------|
| `path/file.ts` | modify | <what/how> |
| `path/new.ts` | create  | <role>          |

## Reuse
- `path:line` — <what to reuse, how>

## Task Order
### 1. <Task title — maps to TaskCreate.subject>
- **Detail**: <maps to TaskCreate.description>
- **Depends on**: none | Task N

### 2. <Task title>
- **Detail**: <…>
- **Depends on**: Task 1 (parallel with Task 3 ok)

## External Notes
- <best practices, pitfalls, caveats from Agent B>

## Test Scenarios       <!-- appended by feature-test -->
- <scenario title> — <target path:line, inputs, expected outcome>

## Review Results       <!-- appended by feature-review -->
- <finding severity, path:line, fix decision>
```

## Status Machine

Each skill must:
1. On start, read the frontmatter `status`.
2. If a required predecessor is `pending`, present AskUserQuestion offering to (a) abort, (b) continue and skip that stage, or (c) run the predecessor first.
3. On successful completion, update only its own stage to `done`.

| Skill | Reads | Writes |
|-------|-------|--------|
| `feature-research` | — | whole file + `status.research: done` |
| `feature-work` | `status.research` | `status.work: done` |
| `feature-test` | `status.work` | append `## Test Scenarios`, `status.test: done` |
| `feature-review` | `status.work`, `status.test` | append `## Review Results`, `status.review: done` |

## Agent Return Schema (Fixed)

Every delegated agent prompt across all feature-* skills **must end** with this block so only summaries return to the main context:

```
RETURN ONLY this schema (no diff bodies, no file contents):
- status: ok|issues|blocked
- summary: ≤3 lines
- findings: [{severity, path:line, one_line_desc}]  # max 10
- artifacts: [relative paths touched]
Do NOT echo plan text, diff bodies, or code blocks.
```

Pointers only — agents read the Plan and `git diff <file>` themselves from the paths passed in the prompt.

## Naming & Path

- Slug: kebab-case, ≤40 chars, stable across all 4 stages.
- Path: `.claude/plans/<slug>.md` unless the user overrides via `--plan <path>`.
- If the file already exists, the orchestrating skill (usually `feature-research`) resolves the collision via AskUserQuestion.
