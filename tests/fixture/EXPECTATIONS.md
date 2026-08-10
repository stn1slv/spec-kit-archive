# Fixture Test Cases and Expected Outcomes

The fixture in `project/` is a minimal two-feature spec-kit project. Every trap below is deliberate. A test runner (a fresh agent given only `commands/archive.md` and the project) executes one case per run against a clean copy; results are compared to this file, which is written before any run.

## Traps built into the fixture

| # | Trap | Location | What it tests |
|---|---|---|---|
| T1 | `## Clarifications` section with a session log | 001 spec.md | Whether the Q&A log is dropped, copied, or invents an empty heading in memory |
| T2 | User Story 3 has no Acceptance Scenarios | 001 spec.md | The carry rule: story must still be archived and named under Outstanding Items |
| T3 | Edge cases are unnumbered prose bullets | both specs | Source-ref ladder: quoted phrase rung, never a bare `-> Edge Cases` |
| T4 | `bugs/BUG-001.md` amending FR-003 | 001 bugs/ | Whether uncategorised in-FEATURE_DIR content leaks into memory (v1.1.2 has no rule for it) |
| T5 | `feature.json` points at 002 | .specify/ | Argument-vs-script precedence and its report line |
| T6 | 002 `FR-003` restates 001 `FR-002` in different words | 002 spec.md | Consolidation: fold, not append; ref ladder on the folded entry |
| T7 | 002 `FR-004` (delete after 90 days) contradicts 001 `FR-004` (keep forever) | 002 spec.md | Supersession candidate detection, confirmation gate, RETIRED line, SC-003 dangling-reference listing |
| T8 | 002 numbers its FRs locally from FR-001 | 002 spec.md | ID continuation: incoming items renumber above the highest main-memory ID |
| T9 | 002 repeats the SSO assumption verbatim | 002 spec.md | Assumption dedupe (5.1 step 8) |
| T10 | 002 extends the Task entity with one field | 002 spec.md | Entity extension, not restatement |
| T11 | Per-feature header block in spec-template | template + specs | What lands at the top of memory spec.md, and what happens to it on the second feature |
| T12 | 002 tasks.md has 7 of 8 complete | 002 tasks.md | Task counting |
| T13 | Constitution Principle II requires retention changes to be recorded | constitution.md | Compliance check interacts with T7: the 90-day rule change must be recorded, not silent |

## Case A — first archival, full scope: `specs/001-task-manager`

- Memory `spec.md`, `plan.md`, `changelog.md` created and populated; nothing else written except AGENTS.md may gain a Recent Changes entry.
- `spec.md` carries: 3 stories (Story 1 with 2 scenarios, Story 2 with 1), Story 3 archived without scenarios and named in Outstanding Items (T2); FR-001..006; 2 entities; 3 edge cases with quoted-phrase refs (T3); SC-001..003; 3 assumptions.
- Report names the argument-vs-script divergence (T5): script said 002, argument won.
- Changelog entry records 10/10 tasks.
- **T4, version-scoped.** Under v1.1.2 (no rule): predicted leak of BUG-001 content into FR entries — confirmed in `BASELINE-v1.1.2.md` (F1). From v1.1.3: `bugs/` is a named exclusion — BUG-001 is not read for content, FR-003 is archived unchanged, and the directory's presence is reported under Outstanding Items.
- **T1, version-scoped.** Under v1.1.2 (undefined): predicted empty `## Clarifications` heading — behavior varied by agent (F2). From v1.1.3: the log is deliberately not archived, no heading is carried, and the exclusion is stated in the report.
- **T11, version-scoped.** Under v1.1.2 (unspecified): predicted per-feature header block atop memory `spec.md` — agent-dependent (F4). From v1.1.3: seeds are titled `# Main Project Specification` / `# Main Implementation Plan` with the entire per-feature header block dropped.

## Case B — first archival, `--spec-only`: `specs/001-task-manager --spec-only`

- Only memory `spec.md` is written. **`plan.md` and `changelog.md` must not exist afterwards** (the v1.1.1-era silent-loss regression is an empty seeded `plan.md`).
- Report's Scoping section names what was skipped and states the re-run advice for this same feature.
- Supersession gate is closed (changelog not writable): irrelevant here (first run, no candidates), but the report must not claim supersession was evaluated against a gate that is open.

## Case C — second feature after A: `specs/002-notifications` (runner confirms proposed removals)

- FR renumbering (T8): 002's four FRs enter as FR-007..FR-010 or fold; local numbers never collide with FR-001..006.
- T6: 002's single-owner FR folds into main FR-002, gaining a second source ref via the ladder; not appended as a near-duplicate.
- T7: the retention contradiction is detected, presented for confirmation; on confirmation the old entry is removed, a `RETIRED:` line appears in the changelog with the replacement ID, and the report lists SC-003 as a dangling reference (it depends on keep-forever).
- T9: SSO assumption not duplicated. T10: Task entity extended in place. T12: changelog records 7/8 tasks.
- T11 second half: what happens to the per-feature header block from A. Predicted failure: overwritten or duplicated.
- T13: the constitution requires the retention change to be recorded; the RETIRED line satisfies it, and the report should connect the two (weak expectation; absence is a finding, not a failure).

## Case D — repeat of A after A: `specs/001-task-manager`

- No file changes at all. Report says the feature is already archived, per artifact.
- No second changelog entry, no second Recent Changes bullet, no duplicated source refs.

## Round 2 cases (v1.1.3 verification; written before any round-2 run)

New traps, in feature `003-reporting` and the `../legacy-state/` overlay:

| # | Trap | Location | What it tests |
|---|---|---|---|
| T14 | 003 `FR-002` (one reminder **per overdue task** daily) contradicts main `FR-008` (**at most one** daily summary) | 003 spec.md | Declined supersession: recording, re-raise, one-line-per-pair |
| T15 | 003 Constraints ("multi-region read replica") conflicts with the archived "single region" | 003 plan.md | Scalar-field narrow override: Step 3 question, superseded value's ref dropped with it, both recorded |
| T16 | `research.md` with one gotcha; `data-model.md` defining Report and extending Task | 003 | Known Issues merge in 5.3; entity refs citing `data-model.md` |
| T17 | Legacy-state overlay: per-feature header, unnumbered assumptions, directory-level refs, per-feature-block plan, old-format chronological changelog | legacy-state/ | Every migration rule: no retroactive rewrite, AS backfill without ref cascade, touched-only legacy-ref upgrade, shared-sections creation, prepend-above-old-format |
| T18 | Scope union and single modifiers | invocations | Union semantics; each single modifier writes exactly its artifact |

### Case E — `specs/003-reporting`, full scope, on the C2 end state; runner **declines** all removals

- T14: the FR-002-vs-FR-008 contradiction is detected and presented; on decline, **nothing is removed**, both entries stay, the pair is recorded once under `## Unresolved Contradictions` (naming main-memory IDs), and 003's FR-002 is added as a new entry (never folded into the entry it contradicts).
- T15: the Constraints conflict is raised as a Step 3 question; on the default, the field holds the current implemented state, the superseded value's ref is dropped with the value, and both are recorded in the revision note and report.
- T16: the gotcha lands in the agent file's Known Issues in the standard format; the Report entity's ref cites `data-model.md`; Task's `completed_at` extension folds into the existing Task entry with a `data-model.md` ref.
- FR renumbering continues above the highest live or retired ID (FR-009 exists, FR-004 retired), so 003's three FRs land at FR-010..FR-012 or fold.

### Case E2 — repeat Case E on its own end state, declining again

- The Unresolved Contradictions line for the pair is **updated, not duplicated**: new date appended, original "Raised by" kept.
- Everything else is a per-artifact idempotent no-op; state otherwise byte-for-byte unchanged.

### Case F — `specs/001-task-manager --spec-only --changelog-only`, clean fixture

- Exactly `spec.md` and `changelog.md` written; `plan.md` not created, agent file untouched.
- The Scoping section names the union, what was skipped, and the suppressed plan bootstrap with the re-run advice.
- The supersession gate is **open** (both required artifacts writable) — irrelevant on a first run, but the report must not claim it was closed.

### Cases G1 / G2 — `--plan-only` and `--agent-only` singles, clean fixture each

- G1: only `plan.md` created and populated; no `spec.md`, no `changelog.md`. Consolidated structure, refs cite `plan.md`.
- G2: only the agent file updated; **no memory artifact is created at all**. The report says so per artifact.

### Case J — `specs/002-notifications`, full scope, clean fixture with the legacy-state overlay

- T17, per the overlay README's table: the `## Feature 001:` header block survives untouched; unnumbered assumptions get `AS-001..003` backfilled in order plus `AS-004` for 002's new one, with **no** ref upgrades triggered by numbering alone; the owner-deactivation edge case (the one entry 002 folds into) has its directory-level ref upgraded, while the other legacy refs stay; 002's plan content creates shared sections next to the untouched `### 001 ...` blocks and the mixed layout is reported; the changelog gains the 002 entry at the top in the new format while the 001 entry keeps `— 2026-08-09` and its bare path.
- The supersession machinery still works across formats: keep-forever vs 90-day retention is detected against the legacy-format spec (runner confirms in this case), FR-004 retired with a `RETIRED:` line.
