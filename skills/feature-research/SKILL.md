---
name: feature-research
description: "Research phase of a feature workflow. Explore the codebase in parallel, gather external best practices, and produce an approved Plan file at .claude/plans/<slug>.md. Usage: /feature-research <feature description> [--plan <path>]"
argument-hint: "<feature description> [--plan <path>]"
allowed-tools: Read, Write, Agent, AskUserQuestion, EnterPlanMode, ExitPlanMode, Glob, Grep
---

# feature-research

First stage of the 4-skill feature workflow (`feature-research` → `feature-work` → `feature-test` → `feature-review`). This skill defines **what to build** by exploring the codebase, consulting external sources, and writing an approved Plan file that later skills consume as a contract.

`$ARGUMENTS` is the feature description. If absent, collect the goal with AskUserQuestion before proceeding.

> **Language note:** This skill is authored in English per `.claude/rules/skill-authoring.md`. User-facing output (AskUserQuestion text, Plan file contents, mini-report) MUST follow the project's CLAUDE.md language preference. Example labels in references are English for authoring consistency only.

## Invariants

- User questions and choices use **AskUserQuestion** — never plain text prompts.
- Plan approval uses **EnterPlanMode → write Plan → ExitPlanMode**.
- External web research uses the **`external-research` agent** (never `general-purpose`; its WebSearch/WebFetch arrive deferred and the agent exits with zero tool uses).
- Every delegated agent prompt ends with the **fixed return schema** (see `references/plan-schema.md` §Agent Return Schema) so only summaries return to main context.

## References

- [references/plan-schema.md](references/plan-schema.md) — Plan file contract shared by all 4 feature-* skills (frontmatter, sections, status machine, agent return schema).
- [references/plan-template.md](references/plan-template.md) — Concrete template for writing the Plan body.
- [examples/good-plan.md](examples/good-plan.md) — Worked example demonstrating the expected specificity.

---

## Workflow

### 1. Parse arguments & scope

1. Extract the feature description from `$ARGUMENTS`. If missing or ambiguous, call AskUserQuestion for: goal, constraints, priority (must-have vs nice-to-have), expected outcome.
2. Derive a feature `<slug>` (kebab-case, ≤40 chars). Default plan path: `.claude/plans/<slug>.md`. Override if `--plan <path>` was supplied.
3. If the plan file already exists, ask via AskUserQuestion whether to overwrite, append, or pick a new slug.

### 2. Parallel exploration (4 agents, single message)

Launch all four in one Agent block. Each prompt includes the feature goal and project-language note, and ends with the fixed return schema from `references/plan-schema.md`.

- **Agent A-1 — Reusable code** (`subagent_type: Explore`): find existing utilities, helpers, and components the new feature should reuse. Return `path:line — purpose`.
- **Agent A-2 — Impact surface** (`subagent_type: Explore`): trace imports, call chains, and shared types touched by the change. Return affected files with reason.
- **Agent A-3 — Existing patterns** (`subagent_type: Explore`): locate analogous implementations. Return naming, directory, error-handling conventions to follow.
- **Agent B — External best practices** (`subagent_type: external-research`): the agent's own system prompt enforces sections, citations, and counts — only pass the goal, stack versions (detected from `package.json` / `build.gradle` / `pyproject.toml` / etc.), and architectural constraints from CLAUDE.md.

### 3. Synthesize

Compose findings into the Plan body. Before writing any new code idea, confirm it is not already covered by Agent A-1's reuse list. If critical information is missing, ask the user via AskUserQuestion before drafting the Plan.

### 4. Approve the plan

1. Call **EnterPlanMode**.
2. Write the Plan body following `references/plan-template.md`. Match the specificity of `examples/good-plan.md`.
3. Call **ExitPlanMode** for user approval.
4. On **approval**: write the Plan file to the resolved path with the frontmatter defined in `references/plan-schema.md` and `status.research: done`.
5. On **revision request**: update the Plan, optionally re-run step 2 exploration for gaps, then re-submit.

### 5. Mini-report (outline style)

Emit a final outline-form report as model text. Follow the project's language preference. Labels shown below are reference-only; translate at runtime.

```
──────────────────────────────────────────
  ✓ feature-research done   **<slug>**
──────────────────────────────────────────

**▸ Summary**
  - **Goal captured** — <one-phrase goal>
  - **Exploration** — <N agents>, <M reusable refs found>
  - **Plan approved** — **EnterPlanMode → ExitPlanMode**

**▸ Artifacts**
  - `.claude/plans/<slug>.md` — **Plan contract** (status.research: done)

**▸ Metrics**
  - Reuse hits **<n>**
  - External sources **<n>**
  - Risks flagged **<n>**

**▸ Next steps**
  - **Run `/feature-work <plan-path>`** to execute tasks
──────────────────────────────────────────
```

Outline rules: no complete sentences; bolden nouns, identifiers, numerics, and status words; symbols only from `✓ ⚠ ✗ ▸`; no full path dumps. Never emit the report via Bash — output as model text.

## Constraints

- SKILL.md body length ≤ 200 lines.
- Do not inline exploration content here; reference files hold the contract and templates.
- Never embed Plan body text into an agent prompt — pass the **path**, let the agent read it.
- Do not proceed to `feature-work` from inside this skill; that is the wrapper's responsibility.
