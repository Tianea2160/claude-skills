# Task Decomposition & Parallel Dispatch

Reference for Step 2 and Step 3 of `feature-work`. Use when translating `## Task Order` into TaskCreate items and deciding which may run in parallel.

## Decomposition shape

Each task must be:
- **Independently completable** — a single agent can finish it end-to-end.
- **Explicitly dependent** — predecessor tasks are named.
- **Verifiable** — a concrete completion signal exists (tests pass, a file compiles, a symbol is exported).

Example (from a Plan's `## Task Order`):

```
Task 1: [no deps]     Define the domain model
Task 2: [needs 1]     Implement the repository
Task 3: [needs 2]     Implement the service logic
Task 4: [needs 1]     Write the API endpoint
Task 5: [needs 3, 4]  Integration test
```

Each item maps 1:1 to TaskCreate:
- `subject` = task title (imperative)
- `description` = the Plan's `**Detail**` line
- `addBlockedBy` = tasks listed under `**Depends on**`

## Parallel dispatch

Two tasks may run in parallel **only when the file sets they touch are disjoint**. The Plan's `## Change Plan` table provides the file list for each task. Derive the overlap check from there.

```
Task 1: Define the domain model           → completes
  ├─ Task 2: Implement the repository  (Agent A)  ──┐
  ├─ Task 4: Write the API endpoint    (Agent B)  ──┤  file sets disjoint → parallel
  └─ Both complete → proceed to Task 3, Task 5
```

Dispatch pattern (single Agent block, multiple tool calls in one message). All instances use `subagent_type: implementer`:

```
Agent(subagent_type="implementer", description="Task 2 — repository", ...)
Agent(subagent_type="implementer", description="Task 4 — API endpoint", ...)
```

Each prompt supplies:
- The Plan path (not its body).
- The task's title and `**Detail**`.
- The specific files this task touches (from `## Change Plan`).
- Reuse hints relevant to this task (from `## Reuse`).
- The fixed return schema.

## Non-parallel signals

Serialize when any of these are true:
- File overlap between candidate tasks.
- One task introduces a symbol the other imports.
- The tasks share a migration/seed step.
- Total agent count would exceed 3 — prefer batches of 2–3 to keep the return-schema summaries legible.

## Test scoping per task

After an agent returns `status: ok` for a task, run tests scoped to the files it touched. Use the smallest command that covers those files:
- JS/TS: `vitest <file>`, `jest <file>`
- Python: `pytest <path>`
- Go: `go test ./<pkg>`
- Kotlin/Gradle: `./gradlew test --tests <class>`

Full-suite regression runs once at the end (Step 5), not per-task.
