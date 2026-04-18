---
name: new-feature
description: "End-to-end feature workflow wrapper. Chains feature-research → feature-work → feature-test → feature-review and emits the final outline-form report. Usage: /new-feature <feature description>"
argument-hint: "<feature description>"
allowed-tools: Read, Write, Edit, Skill, AskUserQuestion
---

# new-feature

Wrapper skill that runs the full feature workflow end-to-end by chaining four independent skills:

1. `/feature-research` — produces an approved Plan at `.claude/plans/<slug>.md`.
2. `/feature-work` — implements the Plan.
3. `/feature-test` — writes and runs tests.
4. `/feature-review` — verifies goal alignment and code quality.

Each stage is a self-contained skill that can also be invoked directly. This wrapper adds orchestration and a **final outline-form report** that stitches the four mini-reports together.

`$ARGUMENTS` is the feature description.

> **Language note:** This skill is authored in English per `.claude/rules/skill-authoring.md`. User-facing output follows the project's CLAUDE.md language preference.

## References

- [references/report-template.md](references/report-template.md) — Final outline-form report template for Step 5.

## Workflow

### 1. Run research

Invoke `Skill: feature-research` with `$ARGUMENTS`. On completion, capture the returned Plan path.

If the user declines the Plan during ExitPlanMode inside `feature-research`, the wrapper stops and surfaces the decline.

### 2. Run implementation

Invoke `Skill: feature-work` with the Plan path.

### 3. Run tests

Invoke `Skill: feature-test` with the Plan path.

### 4. Run review

Invoke `Skill: feature-review` with the Plan path.

### 5. Emit the final outline-form report

Render the combined outline-form report following `references/report-template.md`, as model text in the project's preferred language. Never emit via Bash.

The report stitches each stage's mini-report `▸ Summary`/`▸ Artifacts`/`▸ Metrics` into a single view, adds a `▸ Stage results` block, and surfaces any `▸ Next steps` that require user action.

## Invariants

- Do not re-implement the stage logic inline; always delegate to the dedicated skills.
- Do not embed Plan body text into the wrapper's prompts or report — reference by path.
- Preserve each stage's mini-report as-is; the final report composes them, does not replace them.
- If any stage stops with `status: blocked` or an unresolved AskUserQuestion, halt the chain and surface the condition.

## Constraints

- SKILL.md body length ≤ 80 lines.
- Allowed tools are intentionally narrow: orchestration only.
