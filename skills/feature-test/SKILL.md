---
name: feature-test
description: "Test authoring phase of a feature workflow. Derives scenarios from an approved Plan, writes tests per file via delegated agents, runs them, and self-heals implementation up to 2 retries. Usage: /feature-test <plan-path>"
argument-hint: "<plan-path>"
allowed-tools: Read, Write, Edit, Bash, Agent, AskUserQuestion, Glob, Grep
---

# feature-test

Third stage of the 4-skill feature workflow. Consumes the Plan (now with `status.work: done`) and produces the test suite for the implemented feature.

`$ARGUMENTS` must be the path to the Plan file.

> **Language note:** This skill is authored in English per `.claude/rules/skill-authoring.md`. User-facing output follows the project's CLAUDE.md language preference.

## Invariants

- Pass the Plan **by path** to agents; never inline its body into prompts.
- Run the project's existing test framework; do not introduce a new one without AskUserQuestion.
- Every delegated agent prompt ends with the fixed return schema (see `feature-research/references/plan-schema.md` § Agent Return Schema).
- Self-healing loop: on test failure the owning agent attempts a fix and re-reports `status: ok`. **Maximum two retries per scenario**; on the third failure, surface to the user.

## References

- [references/test-patterns.md](references/test-patterns.md) — How to detect the project's test framework, convention, and command.
- `../feature-research/references/plan-schema.md` — Plan contract and agent return schema.

---

## Workflow

### 1. Resolve the Plan

1. Take the Plan path from `$ARGUMENTS`. If empty, prompt via AskUserQuestion (same resolution as `feature-work`).
2. Read the frontmatter. If `status.work` is `pending`, ask whether to (a) abort, (b) continue anyway, or (c) run `/feature-work` first.

### 2. Detect the test framework

Consult `references/test-patterns.md` to determine:
- Test command (from `package.json` scripts, `Makefile`, `build.gradle`, `pyproject.toml`, `CLAUDE.md`).
- Test file location convention (colocated `*.test.ts`, sibling `tests/`, `__tests__/`, `test/<mirror>`).
- Assertion style and helpers in use.

If detection is ambiguous, ask via AskUserQuestion to choose among detected candidates.

### 3. Derive test scenarios (delegated)

Dispatch one `Explore` agent (read-only scenario derivation) to read the Plan's `## Goal`, `## Change Plan`, `## Reuse`, and `## External Notes`, then return a list of scenarios. Each scenario: `title`, `target path:line`, `inputs`, `expected outcome`. Prompt ends with the fixed return schema.

### 4. Persist scenarios to the Plan

Open the Plan file and **append** the returned scenarios to the `## Test Scenarios` section. Do not rewrite other sections.

### 5. Write tests (delegated, per file)

For each test file to be created (grouped by target module):
1. Dispatch the **`implementer` agent** (`subagent_type: implementer`) with: the Plan path, the scenarios for this module, the target file conventions, and the fixed return schema.
2. The agent writes the test file and returns only the created path plus a one-line summary.

Parallelization: test files for different modules may be dispatched in one Agent block as parallel `implementer` instances; collate the returns before proceeding.

### 6. Run tests and self-heal

1. Run the full test command once.
2. For each failing test: dispatch **`implementer`** with the failure output and ask it to **fix either the test or the implementation** and re-report. The agent retries up to 2 times internally, then surfaces `status: blocked`.
3. On `blocked` status, stop self-healing and ask the user via AskUserQuestion how to proceed.

### 7. Update Plan status

Set `status.test: done` in the Plan's frontmatter. Do not modify other fields.

### 8. Mini-report (outline style)

Emit a final outline-form report as model text, in the project's preferred language. Labels below are reference-only.

```
──────────────────────────────────────────
  ✓ feature-test done   **<slug>**
──────────────────────────────────────────

**▸ Summary**
  - **Scenarios added** — **<n>**
  - **Test files created** — **<k>**
  - **Self-heals applied** — **<m>**

**▸ Artifacts**
  - `<test-path>` — **<covers>**
  - …

**▸ Metrics**
  - Tests **<pass/total>**
  - Framework — `<vitest|pytest|go test|...>`

**▸ Next steps**
  - **Run `/feature-review <plan-path>`** to validate
──────────────────────────────────────────
```

Outline rules: no complete sentences; bold nouns/identifiers/numerics/status; symbols only from `✓ ⚠ ✗ ▸`; never emit via Bash.

## Constraints

- SKILL.md body length ≤ 200 lines.
- Do not create new test frameworks or alternative runners.
- Do not touch Plan sections other than `## Test Scenarios` and `status.test`.
- Two-retry ceiling on self-healing; beyond that, the user decides.
