# Baseline: v1.1.2 (`be10922`) against the fixture

Four runs, 2026-08-10, executed by fresh Claude agents given only `commands/archive.md` and a clean copy of `project/`. Every claim below was verified against the files each run wrote, not the runner's report. Run outputs live outside the repo (session scratchpad `fxruns/`).

| Case | Invocation | Result |
|---|---|---|
| A | `specs/001-task-manager` (full) | Pass with 2 findings (F1, F3) |
| B | `specs/001-task-manager --spec-only` | Pass with 1 finding (F1); **no empty `plan.md`/`changelog.md` created** |
| C | `specs/002-notifications` (full, removals confirmed) | Pass; inherits F3 via A's state |
| D | `specs/001-task-manager` repeat after A | **Pass, byte-for-byte identical state** |

## Verified working in v1.1.2

- Scope-gated bootstrap (T-B): `--spec-only` wrote exactly one file; Scoping carried the re-run advice verbatim.
- Idempotency (D): zero changes on repeat, correct per-artifact reasoning, no duplicate refs or changelog entries.
- Supersession end-to-end (C, T7): both candidates confirmed, entries removed, two `RETIRED:` lines with reasons and replacement refs, retired IDs kept off-limits, no dangling references, no leftover `<pending>`.
- Consolidation (C, T6/T9/T10): reworded FR folded into FR-002 preserving both constraints with two source refs; Task entity extended in place; edge case folded; duplicate assumption skipped.
- ID continuation (C, T8): local FR-001..004 entered as FR-007..009 (one folded); SC as SC-004..005.
- Ref ladder (T3): IDs, quoted phrases, zero bare section names, in all runs.
- Argument-vs-script precedence (T5) reported in every applicable run.
- Story without scenarios (T2) carried and named in Outstanding Items.
- Allowed Sources: no run read `specs/002-notifications` while archiving 001, no git content, tooling carve-out exercised gracefully on a non-git fixture.

Deviation, judged better than the expectation: EXPECTATIONS predicted SC-003 surviving as a dangling reference; run C instead surfaced it as a second supersession candidate and retired it with `no replacement`. The expectation is updated by this baseline: dependent-outcome retirement is acceptable and desirable.

## Failures and undefined behavior (v1.1.3 targets)

- **F1 — `bugs/` is nondeterministic (T4, confirmed in the strongest form).** Run B treated `bugs/BUG-001.md` as an allowed FEATURE_DIR artifact, rewrote FR-003's archived text with the bug's amendment, attached a `bugs/BUG-001.md -> "Amendment"` ref, and self-answered a confirmation question. Runs A and D concluded the opposite ("not an artifact any step reads") and archived FR-003 unchanged. Same command, same input, opposite main-memory content.
- **F2 — Clarifications undefined (T1).** Both runners silently excluded the session log (one documented the choice, one never mentioned it); Joel Young's codex run emitted an empty heading. Three runs, two behaviors, no rule.
- **F3 — NEW: "Why this priority" dropped.** Run A archived stories without their "Why this priority" lines; run B kept them. Fourth instance of the enumeration-loss class (after assumptions/outcomes in 1.1.0, the 5.1 gap in 1.1.1, acceptance scenarios in 1.1.2): any story sub-field not named in 5.1 step 2 survives only by agent whim. Fix the class: carry the story's entire block, not an enumerated subset.
- **F4 — Memory doc header agent-dependent (T11).** Neither Claude runner reproduced the `## Feature 001:` block Joel's codex wrote; both invented a clean `# Main Specification` title. The command specifies no header, so every agent invents one.
- **F5 — Changelog appended chronologically**, confirming the field report; newest entry sits last.
- **F6 — plan structure agent-dependent.** These runs consolidated `plan.md` into shared sections; Joel's codex built per-feature blocks. No structure is stated, so both are "compliant".

## Not yet covered

Declined supersession, scope unions (`--spec-only --changelog-only`), `--plan-only`/`--changelog-only`/`--agent-only` singles, a run with `research.md`/`data-model.md` present, and cross-agent variance (all four runs used the same agent family; Joel's codex runs are the only other data point).
