---
paths:
  - "skills/*/SKILL.md"
---

# Skill Authoring Rules

> Reference: [Extend Claude with skills — official docs](https://code.claude.com/docs/en/skills)

## Frontmatter fields

| Field                      | Required        | Description                                                                                                |
|----------------------------|-----------------|------------------------------------------------------------------------------------------------------------|
| `name`                     | Optional        | Defaults to the directory name. Specify only when it differs from the directory name                       |
| `description`              | **Recommended** | Used by Claude to decide automatic invocation. Combined with `when_to_use`, capped at **1,536 characters** |
| `when_to_use`              | Optional        | Additional trigger conditions. Combined with `description`, capped at 1,536 characters                     |
| `argument-hint`            | Optional        | Autocomplete hint for `/skill <args>`. Example: `"<feature description>"`                                  |
| `allowed-tools`            | Optional        | Tools usable without approval (**allowlist** — does not block other tools)                                 |
| `disable-model-invocation` | Optional        | Recommended `true` for skills with side effects (deploy, commit, etc.)                                     |
| `user-invocable`           | Optional        | When `false`, hides the skill from the `/` menu; only Claude can invoke it                                 |
| `context`                  | Optional        | When set to `fork`, runs in an isolated subagent                                                           |

## Structural rules

- For any user question or selection, always use the **AskUserQuestion tool** (plain-text questions are forbidden)
- Implementation approval flows in this order: **EnterPlanMode → write plan → ExitPlanMode**
- Phase 4 reports must be **emitted as model text directly** — do not use Bash printf (the Claude Code UI collapses it)
- External web research must use the **`external-research` agent**, not `general-purpose`
  (`general-purpose` has been observed exiting with 0 tool uses because WebSearch/WebFetch start in a deferred state)

## Authoring language

- New SKILL.md bodies, the frontmatter `description`, and documents under `references/` must be **written in English**
- Runtime output shown to the user (AskUserQuestion text, report output, etc.) follows the language setting in the project CLAUDE.md
- Rationale: Claude's description-based automatic invocation performs reliably in English, and it ensures compatibility with the global skill ecosystem
- This rule is not retroactive for existing Korean skills — switch to English at rewrite time

## Report format

- Every skill's final / mini-report must be rendered in **outline form**
- No complete sentences — bullets built from noun phrases and verb phrases
- Hierarchy is capped at two levels (`-`, `  -`)
- Fixed symbol palette: `✓` success/done, `⚠` warning/partial, `✗` failure/blocker, `▸` section header
- Emphasize numbers, status words, and unique identifiers with `**bold**` or inline code
- Do not output via Bash printf/echo — always emit as model text

## Reference document pattern

- Link documents under `references/` from SKILL.md using relative paths
- Design them so Claude reads them only when needed at runtime (do not copy their contents into the SKILL.md body)

## Caveat

Path-scoped rules are **loaded only on Read operations** and do not apply to Write (file creation).
The authoritative spec for new SKILL.md files is the "Skill file conventions" section in the root `CLAUDE.md`.
(See [GitHub Issue #23478](https://github.com/anthropics/claude-code/issues/23478))
