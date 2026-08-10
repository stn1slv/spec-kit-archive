# Baseline: v1.1.3 against the extended fixture

Runs from 2026-08-10, executed by fresh agents given only `commands/archive.md` and prepared project states. Every claim verified against the files each run wrote. Case letters: A-D are the v1.1.2 baseline runs (see `BASELINE-v1.1.2.md`); A2-D2 are the same four cases re-run against the v1.1.3 text; E-J are the round-2 cases.

## Round-1 after-tests (A2-D2)

Run against the v1.1.3 command text at `dc2287a` — **before** the round-2/3 review fixes (`06ccc4f`, `4e529f2`), which changed 12 lines on paths these cases do not exercise. The released tag differs from the tested text by those fixes; they were execution-tested only via the round-2 cases below where their paths fire.

| Case | Invocation | Result (verified on disk) |
|---|---|---|
| A2 | 001 full scope | Pass: 3/3 `Why this priority` carried (v1.1.2 dropped them); project-level titles; all 19 plan refs cite `plan.md`; changelog `archived` header + relative link; BUG-001 never read |
| B2 | 001 `--spec-only` | Pass: only `spec.md` written; FR-003 unmodified by the bug file; `AS-001..003`; Clarifications excluded with stated reason |
| C2 | 002 after A2 | Pass: newest-first changelog; FR-004 → FR-009 supersession; FR fold with both constraints; AS-004 added, SSO deduplicated; consolidated plan; missing Independent Test carried as-is |
| D2 | 001 repeat after A2 | Pass: byte-for-byte identical state |

C2's end state is committed as `../archived-state/` and is the pre-registered starting point for Cases E/E2.

## Round-2 cases

| Case | Invocation | Result |
|---|---|---|
| E | 003 full scope, removals declined | **Pass with one pre-registered outcome not produced** (T15 default branch — see Deviations) |
| E2 | repeat of E, declined again | Pass, verified against a pre-run snapshot |
| F | `--spec-only --changelog-only` | Pass |
| G1 | `--plan-only` | Pass |
| G2 | `--agent-only` | Pass (an idempotency-conflated case: 001 was already in AGENTS.md, so the verified outcome is in-place completion without duplication) |
| J | 002 on the legacy-state overlay | **Pass with one pre-registered outcome not produced** (T17 edge-case ref upgrade — see Deviations) |

## Verified working

- **Declined supersession (E, T14)**: three contradiction pairs detected (the planted FR pair plus the story and SC consequences, unprompted), nothing removed, all three recorded under `## Unresolved Contradictions` with main-memory IDs and attribution; the contradicting FR added as a new entry beside its counterpart, never folded.
- **One line per pair (E2)**: re-raised, declined again; each line updated in place ("declined again" + date), original *Raised by* kept, no duplicates; all other artifacts byte-identical to the snapshot.
- **Scope union (F, T18)**: exactly `spec.md` + `changelog.md`; plan bootstrap suppressed with the exact advice; gate correctly reported **open** under the union (the gate needs only `spec.md` + `changelog.md` per the command).
- **Single modifiers (G1/G2, T18)**: `--plan-only` created only `plan.md` (21 refs, every one the full `[Source: specs/001-task-manager/plan.md -> ...]` form); `--agent-only` touched only `AGENTS.md` and created **no** memory artifact, with both suppressed bootstraps reported.
- **research.md / data-model.md (E, T16)**: gotcha merged into Known Issues in the standard format; entity refs cite `specs/003-reporting/data-model.md` precisely (`-> "Report"`, `-> "Task (extension)"`); Task entity carries three refs across three features.
- **Legacy migration (J, T17, partial)**: per-feature header untouched; `AS-001..003` backfilled **without** ref upgrades (backfill-is-not-touching held); old `### 001` plan blocks intact beside newly created shared sections, mixed layout reported; changelog gained the new-format entry above the untouched old-format one; supersession worked across formats.
- **Constitution interplay (E, now registered as T19)**: the runner asked a Step 3 question about the missing Report retention rule rather than silently accepting the plan's "No violations" claim, and recorded the gap. The T19 expectation formalizes this: asking is required, silence is a miss.

## Deviations

- **T15 default branch unexercised (E)**: the scalar Constraints conflict was raised as a Step 3 question as required, but the case's blanket decline instruction routed it to the keep-both branch (both statements kept with an inline note). The pre-registered default outcome (drop the superseded value with its ref) did not occur and still needs its own confirm-instructed case. The original expectation predicted the wrong branch for this case; corrected in EXPECTATIONS.md.
- **Fold judgment is agent-dependent (J vs C2)**: C2 folded 002's "owner deactivated between scheduling and sending" edge case into 001's owner-deactivation entry; J judged them distinct failure modes and added it separately. Both defensible under "fold cases describing the same failure mode". Consequences: J left the pre-registered edge-case legacy-ref upgrade unexercised, and this divergence is direct evidence for the v1.2.0 consolidation detection pass — equivalence needs criteria, not judgment. The overlay was revised after J (see its README's version note) to pin the legacy-ref trap to the guaranteed FR-002 fold instead.

## Still not covered

- Touched-entry legacy-ref upgrade (deterministic from the revised overlay's FR-002; needs a re-run of J).
- Scalar-conflict **default** resolution (drop superseded value + ref, confirm-instructed).
- `--changelog-only` single modifier.
- Supersession candidates under a **closed** gate ("deferred and unrecorded" reporting).
- Step 5.5 `Draft` → `Completed` (every fixture spec says `Implemented`; a `Draft` fixture spec is needed).
- The entire Input Parsing rejection block (empty input, ranges, globs, unrecognized token, ambiguous first token) — field-tested by Joel Young on v1.1.1, never fixture-tested.
- Extension hooks (0.6/7.1) — the fixture ships no `.specify/extensions.yml`.
- `contracts/`, `quickstart.md`, `checklists/` present in a feature.
- Cross-agent variance (all runs used the same agent family).
