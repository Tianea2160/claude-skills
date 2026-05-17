# CLAUDE.md Best Practices

Core principles distilled from Anthropic's official docs, the Toss tech blog, and other industry case studies.

## Anthropic Official Recommendations

### Size and structure
- **Under 200 lines** per file — longer files reduce Claude's adherence to instructions
- Group content with markdown headers and bullets
- Use emphasis like "IMPORTANT" or "YOU MUST" for critical rules

### Include / Exclude criteria

| Include | Exclude |
|------|------|
| Build commands Claude cannot guess | Anything readable from the code |
| Code style rules that differ from defaults | Standard language conventions |
| Test commands and preferred runners | Detailed API docs (link instead) |
| Repository etiquette (branch naming, PR rules) | Information that changes frequently |
| Project-specific architectural decisions | Per-file code descriptions |
| Dev environment quirks (required env vars) | Self-evident things like "write clean code" |

### The core question
> For every line: "If I remove this, will Claude make a mistake?" → If not, delete it.

### Hierarchy
- Managed policy (organization) → Project CLAUDE.md → User CLAUDE.md
- `.claude/rules/` — path-scoped conditional rules (token savings)
- `.claude/skills/` — domain knowledge needed occasionally (use a skill, not CLAUDE.md)

## Key Takeaways from the Toss Tech Blog

### LLM literacy
- Even with the same model, results vary dramatically based on the level of context engineering
- Manage prompts/instructions like code (version control, code review, testing)

### 3-tier knowledge architecture
1. **Global** — company/team-wide standards (~/.claude/CLAUDE.md)
2. **Domain** — team/domain logic (.claude/rules/)
3. **Local** — per-repository implementation (CLAUDE.md)

### Core lessons
- Standardize before you scale — establish shared CLAUDE.md and skills first
- Executable SSOT — docs rot, but CLAUDE.md instructions are executable
- Distribute expert knowledge — package expert workflows into shared skills

## Practical Patterns

### What effective CLAUDE.md files have in common
1. **Command table** — build/test/deploy at a glance
2. **Architecture tree** — only the key directories and their purpose
3. **Gotchas section** — counterintuitive behavior, common mistakes
4. **Concise prose** — one line per concept

### Antipatterns
1. Verbosely describing structure that's readable from the code
2. Listing every file or function
3. Repeating generic coding principles
4. Including changelogs or TODOs
5. Exceeding 200 lines
