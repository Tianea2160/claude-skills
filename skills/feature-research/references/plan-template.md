# Plan Body Template

Use this template inside EnterPlanMode to draft the Plan body. The frontmatter and section list follow `plan-schema.md`; this file specifies **how to write** each section concretely.

```markdown
## Goal

<1–2 sentences: what is built and why. State the problem it solves.>

## Strategy

<Approach + rationale. If alternatives existed, name the chosen one and why.
Call out compatibility, performance, or security trade-offs.>

## Change Plan

| File | Change Type | Concrete Change |
|------|-------------|-----------------|
| `path/file.ts` | modify | Replace function X with Y to support Z |
| `path/new.ts` | create  | Role: … |

## Reuse

- `path:line` — <what is reused, how it is adapted>

## Task Order

### 1. <Task title — imperative, becomes TaskCreate.subject>
- **Detail**: <concrete steps — becomes TaskCreate.description>
- **Depends on**: none

### 2. <Task title>
- **Detail**: <…>
- **Depends on**: Task 1 must complete

### 3. <Task title>
- **Detail**: <…>
- **Depends on**: none (parallel with Task 2)

## External Notes

- <External best practice, pitfall, version-specific gotcha from Agent B>
```

## Authoring Rules

- **Specificity over brevity.** Name the file, function, and line for every change. Avoid vague verbs ("improve", "clean up"); use concrete actions ("extract `validateJWT` as a pure function", "add `broadcastToUser` call after line 45").
- **Direct mapping to TaskCreate.** The subtitle of each "Task Order" entry becomes `TaskCreate.subject`, the `Detail` line becomes `TaskCreate.description`. Authors should read each entry aloud as if dictating a ticket.
- **Reuse first.** Populate the `## Reuse` section before the `## Change Plan`. Every net-new file should be justified against the reuse list.
- **Explicit dependencies.** Mark tasks that can run in parallel as "parallel with Task N". This is what `feature-work` uses to dispatch parallel agents.
- **External Notes are actionable.** Do not paste entire docs; summarize the specific instruction ("socket.io v4 requires `createServer` before `listen`").

## What Not to Include

- Implementation code — the Plan directs the change, it does not pre-compute it.
- Step-by-step rationale essays — keep Strategy to a paragraph.
- Agent transcripts — Agent B's external research is filtered into the `## External Notes` bullets only.
