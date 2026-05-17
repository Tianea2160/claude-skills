---
name: feature-review
description: "Review phase of a feature workflow. Runs two parallel verification agents (goal alignment, code quality) against the implemented change, self-heals up to 2 retries per finding, and captures lessons into CLAUDE.md. Usage: /feature-review <plan-path>"
argument-hint: "<plan-path>"
allowed-tools: Read, Write, Edit, Bash, Agent, AskUserQuestion, Glob, Grep
---

# feature-review

Fourth stage of the 4-skill feature workflow. Verifies that the implementation matches the Plan's goals and passes quality checks; captures durable lessons into CLAUDE.md / `.claude/rules/`.

`$ARGUMENTS` must be the path to the Plan file.

> **Language note:** This skill is authored in English per `.claude/rules/skill-authoring.md`. User-facing output follows the project's CLAUDE.md language preference.

## Invariants

- Agent prompts carry **only the Plan path and `git diff --stat` file list**. Agents load diffs themselves via `git diff <file>`.
- Every delegated agent prompt ends with the fixed return schema from `feature-research/references/plan-schema.md`.
- Self-healing: the same agent that flagged an issue attempts the fix and re-reports. **Two retries maximum**; on the third failure, surface via AskUserQuestion.
- Delta-only re-verification: on the second review round, only the files flagged in findings are reviewed again.

## References

- [references/review-checklist.md](references/review-checklist.md) — Code-quality checklist used by Agent C-2.
- `../feature-research/references/plan-schema.md` — Plan contract, agent return schema, status machine.

---

## Workflow

### 1. Resolve the Plan

1. Take the Plan path from `$ARGUMENTS`. If empty, prompt via AskUserQuestion.
2. Read the frontmatter. If `status.work` is `pending`, ask how to proceed. If `status.test` is `pending`, ask whether to continue without tests.

### 2. Collect the changed-file list

Run `git diff --stat` and capture the list of changed paths. **Do not** embed diff bodies here; agents will fetch them individually.

### 3. Parallel verification (2 agents, single message)

Dispatch both in one Agent block. Each prompt includes: the Plan path, the changed-file list, and the fixed return schema.

- **Agent C-1 — Goal alignment**: read the Plan's `## Goal`, `## Change Plan`, and `## Test Scenarios`; for each requirement, confirm coverage in the diff. Return `findings` with `severity`, `path:line`, `one_line_desc` for each gap.
- **Agent C-2 — Code quality**: apply `references/review-checklist.md` against each changed file (loaded via `git diff <file>`). Return `findings` per the same shape.

### 4. Resolve findings with self-healing

For each `issues` / `blocked` return:
1. Dispatch the **`implementer` agent** (`subagent_type: implementer`) with the finding(s), the target files, and instructions to apply the fix then re-report `status: ok`.
2. On retry failure, the agent exhausts 2 attempts internally, then returns `status: blocked` with its reasoning.
3. On `blocked`, surface to the user via AskUserQuestion: continue with caveat, request manual fix, or abort.

Rationale: C-1 and C-2 are verification-only; actual edits go through `implementer` so main context never sees diff bodies.

### 5. Delta-only second round

After fixes land, re-run Step 3 but narrow the scope to the files from `findings` only. Loop Steps 3–5 until both agents return `status: ok` or the user accepts the remaining caveats.

### 6. Append findings to the Plan

Write the final finding set (including resolutions) to the Plan's `## Review Results` section. Do not rewrite other sections.

### 7. Capture durable lessons (optional)

If new architectural rules or conventions emerged that code alone cannot teach, update the relevant `CLAUDE.md` or `.claude/rules/*.md`. Targets:
- New architecture decisions (e.g., "notifications go over WebSocket").
- Conventions solidified during implementation (e.g., "event types live in `src/types/`").
- Non-obvious gotchas (e.g., "socket.io must initialize after HTTP server creation").

Skip this step if nothing was learned that is not already visible in the code.

### 8. Update Plan status

Set `status.review: done` in the Plan's frontmatter.

### 9. Mini-report (outline style)

Emit a final outline-form report as model text, in the project's preferred language. Labels below are reference-only.

```
──────────────────────────────────────────
  ✓ feature-review done   **<slug>**
──────────────────────────────────────────

**▸ Summary**
  - **Goal alignment** — **<n/n requirements>**
  - **Quality checks** — **<pass|issues resolved>**
  - **Self-heals applied** — **<m>**

**▸ Artifacts**
  - `.claude/plans/<slug>.md` — **Review Results appended**
  - `CLAUDE.md` / `.claude/rules/*.md` — **updated (if applicable)**

**▸ Metrics**
  - Findings **<total>** / resolved **<n>** / caveats **<k>**
  - Files reviewed **<n>**

**▸ Next steps**
  - **Merge-ready** / **User action** — <specific item if any>
──────────────────────────────────────────
```

Outline rules: no complete sentences; bold nouns/identifiers/numerics/status; symbols only from `✓ ⚠ ✗ ▸`; never emit via Bash.

## Constraints

- Do not modify source files from main; source edits flow only through `implementer`. Main-thread Edits are limited to the Plan's `## Review Results` / `status.review` and CLAUDE.md-style rule files.
- Do not rewrite any other Plan section.
- No third retry without user approval.
