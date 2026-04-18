# Final Report — Outline-form Template

The wrapper `new-feature` renders this template at the end of the chain as model text. All four feature-* skills already emit compatible mini-reports; this template composes them into a single view and adds the cross-stage `▸ Stage results` block.

## Outline-form rules (shared across all feature-* mini-reports)

- **No full sentences.** Use noun phrases and verb phrases. Strip subjects and particles when natural.
- **Two bullet levels maximum** (`-`, `  -`).
- **Bold** nouns/identifiers/numerics/status words so the bold-only pass conveys the skeleton.
- **Inline code** for symbol names and literal paths: `` `src/foo.ts` ``.
- **Symbol palette** (fixed meanings): `✓` success/done, `⚠` partial/warning, `✗` failure/blocker, `▸` section marker.
- **No full-path dumps** — summarize by role or module.
- **Render as model text.** Never via Bash (Claude Code UI collapses Bash output).

## Wrapper final report template

```
──────────────────────────────────────────
  <overall symbol> feature delivered   **<slug>**
──────────────────────────────────────────

<One-line TL;DR — bold the essential noun phrase(s)>

**▸ Stage results**
  ✓ research — **Plan approved** (`.claude/plans/<slug>.md`)
  ✓ work — **<n/n tasks>**, tests **<pass/total>**
  ✓ test — **<k scenarios>**, self-heals **<m>**
  ✓ review — **<findings resolved>** / caveats **<c>**

**▸ Key changes** (meaningful units, not file lists)
  - **<change 1 essence>** — <supporting detail>
  - **<change 2 essence>** — <supporting detail>

**▸ Verification**
  ✓ Requirements **<n/n met>**
  ✓ Code quality **passed**
  ⚠ Warnings **<n addressed>**

**▸ Follow-up**
  - **<actor + action>** — <why>
  - …

**▸ How to test**
  `$ <project test command>`
──────────────────────────────────────────
```

## State-based variants

- **Complete**: `✓ feature delivered` (default).
- **Partial**: `⚠ partial delivery` — put the partial-cause under TL;DR first.
- **Blocked / failed**: `✗ delivery halted` — move a `▸ Blocker` block **above** `▸ Stage results`.
- **Warnings only**: keep `✓ feature delivered`; mark warnings via `⚠` inside `▸ Verification`.

## Composition rules

1. Read each stage's mini-report the wrapper received during the chain.
2. For `▸ Stage results`, emit one bullet per skill with the symbol, skill name, and the most load-bearing numeric or status from that skill's report.
3. Pull `▸ Key changes` from the research Plan's `## Change Plan` + the work mini-report's artifacts. Aggregate to 3–5 meaningful units, never a raw file list.
4. `▸ Verification` combines review's goal-alignment count and quality pass/fail with any outstanding caveats.
5. `▸ Follow-up` is for items the user must act on (env vars, migrations, follow-up sprint work) — skip if none.
6. `▸ How to test` names one reproducible command the user can run to see the feature live.

## Forbidden

- Full-path listings (use role or module names).
- Box-drawing characters beyond `─` — CJK width alignment breaks with `│ ╭ ╮`.
- Emitting the report from a Bash `printf` / `echo` — always model text.
- Rewriting the per-stage mini-reports; they remain the source of truth.
