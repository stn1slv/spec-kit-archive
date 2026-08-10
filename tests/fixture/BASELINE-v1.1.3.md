# Baseline: v1.1.3 (tag `v1.1.3`) against the extended fixture

Round-2 cases, 2026-08-10, executed by fresh agents given only `commands/archive.md` (the released v1.1.3 text) and prepared project states. Every claim verified against the files each run wrote. The four round-1 after-tests (full scope, `--spec-only`, second feature, repeat) had already passed against this command text; see `BASELINE-v1.1.2.md` for the v1.1.2 comparison.

| Case | Invocation | Result |
|---|---|---|
| E | 003 full scope, removals declined | Pass |
| E2 | repeat of E, declined again | Pass, verified against a pre-run snapshot |
| F | `--spec-only --changelog-only` | Pass |
| G1 | `--plan-only` | Pass |
| G2 | `--agent-only` | Pass |
| J | 002 on the legacy-state overlay | Pass, one deviation noted below |

## Verified working

- **Declined supersession (E, T14)**: three contradiction pairs detected (the planted FR pair plus the story and SC consequences, unprompted), nothing removed, all three recorded under `## Unresolved Contradictions` with main-memory IDs and attribution; the contradicting FR added as a new entry beside its counterpart, never folded.
- **One line per pair (E2)**: re-raised, declined again; each line updated in place ("declined again" + date), original *Raised by* kept, no duplicates; all other artifacts byte-identical to the snapshot.
- **Scalar conflict (E, T15)**: raised as a Step 3 question as required. Under the harness's decline-everything instruction the runner kept both statements with an inline note (a user-directed custom outcome); the default drop-with-ref path remains unexercised.
- **Scope union (F, T18)**: exactly `spec.md` + `changelog.md`; plan bootstrap suppressed with the exact advice; gate correctly reported **open** under the union.
- **Single modifiers (G1/G2, T18)**: `--plan-only` created only `plan.md` (21 refs, all citing `plan.md`); `--agent-only` touched only `AGENTS.md` and created **no** memory artifact, with both suppressed bootstraps reported.
- **research.md / data-model.md (E, T16)**: gotcha merged into Known Issues in the standard format; entity refs cite `data-model.md` precisely (`-> "Report"`, `-> "Task (extension)"`); Task entity carries three refs across three features.
- **Legacy migration (J, T17)**: per-feature header untouched; `AS-001..003` backfilled **without** ref upgrades (backfill-is-not-touching held); old `### 001` plan blocks intact beside newly created shared sections, mixed layout reported; changelog gained the new-format entry above the untouched old-format one; supersession worked across formats.
- Unscripted: E flagged the missing Report retention rule against constitution Principle II as a `NEEDS CLARIFICATION` outstanding item.

## Deviations and open observations

- **Fold judgment is agent-dependent (J vs run C2)**: C2 folded 002's "owner deactivated between scheduling and sending" edge case into 001's owner-deactivation entry; J judged them distinct failure modes and added it separately. Both defensible under "fold cases describing the same failure mode". Consequences: the touched-entry legacy-ref upgrade path is **still unexercised** (J's non-fold meant the entry was never touched), and this is direct evidence for the v1.2.0 consolidation detection pass — equivalence needs criteria, not judgment.
- The scalar-conflict **default** path (drop the superseded value with its ref) and a **confirmed** supersession under a closed gate remain unexercised.

## Still not covered

Touched-entry legacy-ref upgrade; scalar-conflict default resolution; `--changelog-only` single; supersession candidates under a closed gate ("deferred and unrecorded" reporting); cross-agent variance (all runs used the same agent family).
