---
paths:
  - "agents/*.md"
---

# Agent Authoring Rules

> Reference: [Agent tool — official docs](https://code.claude.com/docs/en/agents)
> Reference: [Equipping agents for the real world with Agent Skills — Anthropic Engineering](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

## Frontmatter fields

| Field         | Required        | Description                                                                                                                             |
|---------------|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `name`        | Recommended     | Agent identifier (used as the `subagent_type` value)                                                                                    |
| `description` | **Required**    | Used by Claude to choose a delegation target. The more specific it is, the more accurate the automatic delegation                       |
| `tools`       | **Recommended** | Declare the tools the agent uses — if omitted, some environments start them in a deferred state, risking an early exit with 0 tool uses |
| `model`       | Optional        | Defaults to inheriting the caller's model. Set explicitly to tune cost or speed                                                         |

## Required sections

Every agent file must include all the following sections:

1. **Strategy** — the order and method of work (how to compose queries, how to cross-check, etc.)
2. **Output format** — a code block showing the markdown shape the caller expects
3. **Failure handling** — behavior when tools are unavailable or results are sparse (**never exit with 0 tool uses**)
4. **Forbidden** — actions the agent must not take (e.g., unsourced claims, code modifications)

## Authoring language

- The agent file's **frontmatter `description` and the entire body must be written in English** (no exceptions)
- Runtime output shown to the user (summaries, reports the agent returns) follows **the language requested by the caller, or the project CLAUDE.md language setting**
- Rationale:
    - Claude's `description`-based automatic delegation performs consistently in English
    - Maintains compatibility with the global subagent ecosystem
- Existing Korean agents must **switch to English on rewrite** (e.g., `external-research`, `implementer`)

## Caveat

Path-scoped rules are **loaded only on Read operations** and do not apply to Write (file creation).
(See [GitHub Issue #23478](https://github.com/anthropics/claude-code/issues/23478))
