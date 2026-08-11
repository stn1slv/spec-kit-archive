# Baseline: v1.2.0 (branch `feat/guidance-bugs-consolidation` at `bda2a79`) against the extended fixture

Round-3 cases, 2026-08-10, expectations pre-registered in commit `bda2a79` **before** any run (the pre-registration rule is satisfied and auditable for the first time). Fresh agents, one per case; every claim verified against the files each run wrote.

| Case | Invocation | Result |
|---|---|---|
| K | 004 full scope, clean fixture | Pass — every T20-T24 outcome |
| L | 001 full scope with the three-part guidance string | Pass — every T25 outcome |
| M | 003 on `archived-state/`, removals confirmed | Pass with one T19 miss (below) |
| J2 | 002 on the revised legacy overlay | Pass with one scalar-migration miss (below) |

## Verified working (all confirmed on disk)

- **Bugfix annotations (K, T20)**: FR-002 archived in its live replacement wording only; the struck text and the `**Bugfix**:` line never entered as content; the orphan strikethrough carried as-is with a quoted-phrase ref and named under Outstanding Items.
- **Status-claim inversion, both directions (K, T21)**: BUG-001 `addressed` despite claiming `Open` (annotation corroborates); BUG-002 `unverified` despite claiming `Fixed` (no annotation), caveat printed; absent Type/Severity/RCA recorded absent, never inferred; BUG-002's RCA became a Known Issues entry titled with ID and title; `Bugs addressed: BUG-001` in the changelog; both bounded reads declared under Sources.
- **Counting (K, T22)**: 3/5 tasks — the `[x] ... (reopened — BUG-002)` task counted incomplete in both counting rules.
- **First `Draft → Completed` (K, T23)**: written to the feature spec; every earlier fixture spec said `Implemented`, so 5.5 had never fired before.
- **Installed-extension detection (K, L, T24)**: `spec-kit-bugfix` matched; the installed-plus-unverified recommendation branch fired in K; `hooks: {}` skipped silently everywhere.
- **Guidance (L, T25)**: the run was not rejected ("3 bullet points" and sentence punctuation passed the narrowed rule 2); steering honored (entity double-check surfaced the planted constitution conflict, deadline call-outs delivered, the changelog summary is exactly 3 bullets); "skip the constitution check" refused with the reason stated, and 2.1 ran and caught the planted conflict; the full text echoed verbatim under `## Guidance`.
- **Consolidation counts (M, J2)**: both populated-memory runs opened Consolidation with real numbers (M: 9 incoming, 4 examined, 1 folded; J2: 14/5/3) and verdict variety — folds, separates, and contradictions routed to supersession. "Zero folded" is now always accompanied by "examined".
- **Scalar default branch (M, T15's missing half)**: the Constraints conflict was raised as a Step 3 question; on the default, "single region" was dropped and the change recorded. Nuance the rule did not anticipate, resolved sensibly: the 001 ref was **kept** because it still vouches for the surviving "no offline mode" clause of the same multi-clause scalar — only a ref citing nothing surviving would be dropped.
- **Deterministic legacy-ref upgrade (J2)**: the FR-002 fold upgraded the directory-level ref to item level (via the sanctioned exception read of 001's spec) and attached 002's ref; the SSO assumption fold did the same (the registered either-acceptable exposure resolved to upgrade); all five untouched legacy refs stayed; the AS backfill triggered no upgrades by itself.
- **Three-way supersession with dependents (M)**: FR-008, User Story 5, and SC-005 retired together with closed `RETIRED:` lines and reasons; the 001-era audit lines below them untouched (append-only held); no dangling references.

## Misses (both judgment variance, neither data loss)

- **T19 (M)**: the missing Report retention rule was judged non-critical (aggregate counts, not user data; the plan's own Constitution Check) instead of being asked as a Step 3 question. Runs E (v1.1.3) and K both asked in equivalent situations; M reasoned its way past the ask. The pre-registered expectation says asking is required. Wording candidate for v1.2.x: 2.1's CRITICAL classification should not be soluble by the feature's own self-assessment.

  **Reclassified during v1.2.1 planning (2026-08-11): part fixture defect, part wording gap, and the recorded candidate would not by itself have fixed it.** M gave two reasons, and the second stood on its own: `003-reporting/data-model.md` defined `Report` as a team id, a period, three integer counts and a timestamp, so "aggregate counts, not user data" was a defensible reading of Principle II's condition, reachable without ever leaning on the plan's claim. T19 was also retro-registered from run E's behavior (`BASELINE-v1.1.3.md:37`), so it encoded one observed run rather than an unambiguous reading of the fixture. The real gap was underneath: 2.1 tested only for *conflicts*, and an unstated retention rule is an unmet obligation, not a conflict — a category 2.1 did not have. v1.2.1 adds that category, adds the self-assessment clause as its companion, and sharpens the fixture (`Report.owner_breakdown`) so T19 is falsifiable. Run M against the sharpened fixture is the pass/fail test.
- **Legacy scalar migration (J2)**: 001's `Constraints`/`Scale/Scope` values stayed inside their legacy `### 001` blocks; the new shared Technical Context holds only 002's values with 002 refs. The migration rule ("old blocks stay; fold **this run's** content into shared sections") and the scalar-compose rule genuinely pull against each other on legacy layouts, and the runner took the conservative reading. Wording candidate for v1.2.x: when a shared section is created beside legacy blocks, seed its scalar fields by composing the legacy values, one ref per contributor.

  **Confirmed and implemented in v1.2.1 (2026-08-11).** The Case J/J2 expectations already required composed values, so the expectation was right and the prompt was ambiguous. 5.2 now states the seeding rule explicitly, bounded to scalar fields, with the legacy contributor's ref routed through the Legacy refs ladder and the legacy blocks still untouched.

## Observations

- L used file-level refs for all plan entries where other runs used the quoted-field rung; permitted by the ladder's last rung, but the field-label rung was available. Fidelity variance, not a defect.
- M volunteered a true plan-side insight the command cannot resolve: the retired daily-summary behavior still has its worker (`summaries.py`) listed in the main plan, and no allowed source states whether the code was decommissioned. Correctly routed to Manual Review — plan-side supersession remains an acknowledged design bound.

## Still not covered

- Guidance combined with scope modifiers; a guidance string attempting to authorize a removal; `--changelog-only`; supersession under a closed gate ("deferred and unrecorded"); a 2.5 run that actually overflows the per-section cap; input-rejection block (fixture-level); cross-agent variance.
