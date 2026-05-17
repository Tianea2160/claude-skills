---
name: feature-work
description: "Implementation phase of a feature workflow. Consumes an approved Plan file, decomposes its Task Order into TaskCreate items, dispatches parallel agents for non-overlapping files, then runs tests and /simplify. Usage: /feature-work <plan-path>"
argument-hint: "<plan-path>"
allowed-tools: Read, Write, Edit, Bash, Agent, AskUserQuestion, TaskCreate, TaskUpdate, Skill, Glob, Grep
---

# feature-work

Second stage of the 4-skill feature workflow. Consumes a Plan approved by `feature-research` and produces working code, guided by the Plan's `## Task Order` and `## Change Plan` sections.

`$ARGUMENTS` must be the path to an existing Plan file. If missing, resolve via AskUserQuestion.

> **Language note:** This skill is authored in English per `.claude/rules/skill-authoring.md`. User-facing output follows the project's CLAUDE.md language preference.

## Invariants

- **Plan body is read by path**, never embedded into agent prompts. Each dispatched agent opens the Plan via Read itself.
- User questions use **AskUserQuestion**.
- Parallel dispatch only when the target files of two tasks **do not overlap** (checked against the Plan's `## Change Plan` table).
- Every delegated agent prompt ends with the fixed return schema from `feature-research/references/plan-schema.md` (§ Agent Return Schema).

## References

- [references/task-breakdown.md](references/task-breakdown.md) — Task decomposition and parallel dispatch examples.
- `../feature-research/references/plan-schema.md` — Plan file contract and agent return schema.

---

## Workflow

### 1. Resolve the Plan

1. Take the Plan path from `$ARGUMENTS`. If empty, list files under `.claude/plans/` and ask the user which to use via AskUserQuestion. If none exist, stop and recommend running `/feature-research` first.
2. Read the Plan's frontmatter. If `status.research` is `pending`, ask (AskUserQuestion) whether to (a) abort, (b) continue anyway, or (c) run `/feature-research` first.

### 2. Decompose into tasks

1. Parse the Plan's `## Task Order` section. For each entry:
   - `TaskCreate.subject` = the entry's title
   - `TaskCreate.description` = the entry's `**Detail**` line
2. Establish dependencies from the `**Depends on**` lines using `TaskUpdate.addBlockedBy`.

### 3. Execute tasks

Loop: pick ready tasks (no unfinished blockers), dispatch them, advance.

- **Single task ready**: `TaskUpdate → in_progress`, delegate to the **`implementer` agent** (`subagent_type: implementer`) with:
  - A pointer to the Plan path and the task's title/detail.
  - The specific files this task touches (from `## Change Plan`).
  - The reuse hints relevant to this task (from `## Reuse`).
  - The scoped test command for the touched paths (if available).
  - The fixed return schema (so only a summary comes back).
- **Multiple ready tasks with disjoint file sets**: dispatch them in a single Agent block as parallel `implementer` instances. See `references/task-breakdown.md` for shape.
- **File overlap detected**: serialize; do not dispatch in parallel.

The `implementer` agent (see `agents/implementer.md`) loads the Plan by path, performs the edits, runs scoped tests internally with up to 2 self-heal retries, and returns only the fixed return schema.

After each task's agent returns `status: ok`, call `TaskUpdate → completed`.

### 4. Targeted tests after each task

Run the scoped test command for the touched paths after each task (e.g., `vitest <file>`, `pytest <path>`, `go test <pkg>`). Long test output should be captured by the agent that owns the task and only summarized to main. Detect the command from `package.json` / `Makefile` / `build.gradle` / `pyproject.toml` / `CLAUDE.md`.

### 5. Full test sweep + simplify

1. Once all tasks are `completed`, run the project's full test command once to catch regressions.
2. Invoke `/simplify` to let the review pass refine reuse/quality/efficiency on the changed code.
3. If `/simplify` makes edits, re-run the test sweep.

### 6. Update Plan status

Edit the Plan file's frontmatter: set `status.work: done`. Do not touch other fields.

### 7. Mini-report (outline style)

Emit a final outline-form report as model text, in the project's preferred language. Labels below are reference-only.

```
──────────────────────────────────────────
  ✓ feature-work done   **<slug>**
──────────────────────────────────────────

**▸ Summary**
  - **Tasks executed** — **<n/n completed>**
  - **Parallel dispatches** — **<k>**
  - **simplify** — ✓ applied

**▸ Artifacts**
  - `<path>` — **<role>**
  - …

**▸ Metrics**
  - Tests **<pass/total>**
  - Files changed **<n>**

**▸ Next steps**
  - **Run `/feature-test <plan-path>`** to add and run tests
──────────────────────────────────────────
```

Outline rules: no complete sentences; bold nouns/identifiers/numerics/status; symbols only from `✓ ⚠ ✗ ▸`; no full path dumps. Never emit via Bash — model text only.

## Constraints

- SKILL.md body length ≤ 200 lines.
- Do not delete or re-order Plan sections; touch only `status.work`.
- Do not start `feature-test` or `feature-review` from here; the wrapper orchestrates transitions.
- Two retries maximum for agent self-healing on a single task; on a third failure surface to the user.
