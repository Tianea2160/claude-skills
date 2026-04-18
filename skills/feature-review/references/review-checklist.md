# Code Quality Review Checklist

Reference for Agent C-2 in Step 3 of `feature-review`. Apply each category against the diff of every changed file. For each violation, emit a finding: `severity`, `path:line`, `one_line_desc`.

## Correctness

- Does the happy path return the expected result for the inputs in the Plan's `## Goal`?
- Are error paths handled? If an error is caught, is the action (log, rethrow, fallback) deliberate?
- Are promises/futures awaited? No lost rejections, no unhandled rejections.
- Are loop and recursion bounds safe? No infinite loops, no unbounded recursion.

## Edge cases

- Null / undefined / empty inputs (empty array, empty string, zero-length buffer).
- Boundary values: `0`, `1`, `Number.MAX_SAFE_INTEGER`, integer overflow, Unicode edge cases.
- Concurrency: shared mutable state, race conditions on startup/shutdown, double-initialization.
- Missing auth / missing permission — does the code fail-closed?

## Security

- Input validation at every trust boundary (API entry, message consumers, file uploads).
- Injection vectors: SQL, NoSQL, command, LDAP, template, XXE.
- Secret handling: no tokens/keys in logs, no secrets committed, no secrets echoed to responses.
- Sensitive data exposure: PII, auth tokens, internal URLs.
- AuthN/AuthZ: every new endpoint checks both; roles and scopes enforced server-side.

## API & contract

- Public function signatures match what the Plan promised.
- Breaking changes flagged? Versioning decisions documented?
- Error payloads consistent with the project's conventions.

## Reuse & simplicity

- Does the change introduce utilities that already exist? (Cross-check `## Reuse` in the Plan.)
- Are there three or more near-identical snippets screaming for a helper? (vs. premature abstraction.)
- Dead code, commented-out blocks, `TODO`/`FIXME` without issue link.

## Tests

- Every new public path has a test in the Plan's `## Test Scenarios`.
- Tests assert on observable behavior, not private implementation details.
- Flaky patterns absent: real timers when time matters, no uncontrolled sleeps, no network reliance.

## Style & conventions

- Matches the project's existing naming (camelCase vs snake_case, file suffixes, directory placement).
- Comments explain *why* at non-obvious spots only; no restating what the code does.
- Imports ordered per project convention; no unused imports.

## Severity guide

- **blocker**: correctness, security, or contract breakage — must fix.
- **major**: edge-case gap, missing auth check, or spec deviation — strongly advise fix.
- **minor**: style, naming, reuse opportunity — fix if easy, otherwise note.

## Finding format

```
- severity: blocker|major|minor
  path:line
  one_line_desc — <≤ 12 words>
```

Cap at **10 findings per agent return**. If more exist, include the top 10 by severity and note the remainder in `summary`.
