# Test Pattern Detection

Reference for Step 2 of `feature-test`. Determine the project's test framework, file layout, and run command before writing tests.

## Framework detection (quick checks)

| Signal | Framework |
|--------|-----------|
| `package.json` has `"vitest"` in deps or `"test"` script invoking `vitest` | Vitest |
| `package.json` has `"jest"` or `jest.config.*` exists | Jest |
| `pyproject.toml` declares `pytest` / `pytest.ini` / `conftest.py` present | Pytest |
| `pyproject.toml` declares `unittest` only, or `test_*.py` with `unittest.TestCase` | Python `unittest` |
| `go.mod` present and files `_test.go` | Go `testing` |
| `build.gradle` with `dependencies { testImplementation 'org.junit...' }` | JUnit 5 |
| `Cargo.toml` present | `cargo test` |
| `CLAUDE.md` explicitly names the command | Use that |

When more than one candidate matches, prefer what `CLAUDE.md` names; otherwise surface an AskUserQuestion with the candidates.

## Command detection

Pull the run command directly from the authoritative source, in this order:
1. `CLAUDE.md` — if it names a test command, use it verbatim.
2. `package.json` — `scripts.test` / `scripts.test:unit`.
3. `Makefile` — `make test` target.
4. `build.gradle` / `build.gradle.kts` — `./gradlew test`.
5. `pyproject.toml` — `[tool.pytest.ini_options]` present → `pytest`.
6. Fall back to the framework's default runner.

Scoped runs for a single file or package (used after each `feature-work` task):
- Vitest: `pnpm vitest run <file>` / `npx vitest run <file>`
- Jest: `npx jest <file>`
- Pytest: `pytest <path>`
- Go: `go test ./<pkg>`
- Gradle: `./gradlew test --tests <FullyQualifiedClass>`

## Location conventions

| Convention | Where tests go |
|-----------|----------------|
| Colocated | `src/foo.ts` → `src/foo.test.ts` |
| `__tests__` | `src/foo.ts` → `src/__tests__/foo.test.ts` |
| Mirror tree | `src/foo.ts` → `test/foo.test.ts` |
| Go convention | `pkg/foo.go` → `pkg/foo_test.go` |
| Maven/Gradle | `src/main/.../Foo.kt` → `src/test/.../FooTest.kt` |

Detect by scanning for existing `*.test.*` / `*_test.*` files and mirroring their placement relative to the source files in the Plan's `## Change Plan`.

## Assertion & helper discovery

Before writing any test:
- Grep for a `test/helpers/` or `tests/conftest.py` to find existing builders, fixtures, fakes.
- Grep for shared matchers (`expect.extend`, custom jest/vitest matchers).
- Reuse `createTestToken`-style helpers rather than fabricating new fixtures.

## Scenario coverage rubric

From the Plan's `## Goal` and `## Change Plan`, each scenario should cover at least one of:
- Happy path per public entry point.
- Each explicit constraint from `## Goal`.
- Each risk from `## External Notes`.
- Edge cases: null/empty inputs, boundary values, concurrent writes (when relevant).
- Failure paths: invalid inputs, missing auth, downstream errors.

When a scenario maps directly to a line in `## Change Plan` (e.g., "broadcast is emitted after `updateOrderStatus`"), cite that `path:line` in the appended `## Test Scenarios` entry.
