# Baseline: v1.2.1 (branch `fix/constitution-obligations-legacy-scalars` at `ba2fb42`) against the fixture

Round-4 cases, 2026-08-11. Expectations pre-registered across commits `4dae708`, `9371329`, `f52faee`, and `ba2fb42`, all **before** any run. Four fresh agents, one per case, each given only `commands/archive.md` and its own working copy, and explicitly forbidden from reading `EXPECTATIONS.md`, the baselines, the CHANGELOG, or git history. Every claim below was verified against the files each run wrote, not against the runner's report.

| Case | Invocation | Result |
|---|---|---|
| M | 003 on `archived-state/`, removals confirmed | **Pass** — including T19, the release's headline fix |
| J2 | 002 on the revised legacy overlay, removals confirmed | **Pass** — every composition outcome |
| K | 004 full scope, clean fixture | **Pass** — v1.2.0 bug awareness intact, both new constitution expectations met |
| C | 001 then 002 in sequence, clean fixture | **Pass** — including the restated T13 |
| J3 | 003 on J2's end state, removals confirmed | **Pass** — first run to exercise the conflict branch of legacy seeding |
| J4 | 003 repeated on J3's end state | **Pass** — byte-identical, and the revision-note exception fired for the stated reason |

## The two fixes this release exists for

**T19 (M) — the miss that opened v1.2.1 is closed.** The runner flagged Principle II as an unmet obligation, triggered by `Report.owner_breakdown` (per-owner counts keyed by user id, with display names), and asked it as a Step 3 question. It rejected the plan's "No violations. Reports contain only task metadata already visible to team members" on the stated grounds that it is a claim about *visibility*, not about *retention* — the precise reasoning run M failed to reach under v1.2.0. Both halves of the fix contributed: the obligation branch gave the finding a category, and the sharpened fixture removed the "aggregate counts, not user data" escape.

**Legacy scalar composition (J2) — the shared field is no longer understated.** The new shared Technical Context composes all five labelled fields of the `### 001` block. `Constraints` reads "single region, no offline mode, email only, no push or SMS" and `Scale/Scope` carries both features' values, each with one ref per contributor. The `### 001 Technical Context` block is **byte-identical** to the overlay's original, confirmed by diff: the do-not-modify bound held. Identical values took one ref each (`Language/Version: Python 3.12` carries both 001's and 002's refs), which is the round-5 clarification landing correctly.

**The re-ask loop is closed (J3 → J4), and this was the release's least certain fix.** J3 resolved a scalar conflict *against a legacy value*: the shared `Constraints` dropped "single region", kept its three other clauses, and the `### 001` block kept the dropped value where it was. That left precisely the trap state — a legacy block holding a value the shared field deliberately does not — which without the round-5 exception reads as "uncomposed" and gets composed back in, silently reversing the user's Step 3 decision. J4 re-ran the same feature on that state and wrote **nothing**: all three memory checksums unchanged, and a full-tree `diff` between J3 and J4 empty.

It passed for the stated reason, not by accident. Asked for a per-field disposition, the J4 runner reported that it considered `Constraints`, applied 5.2's instruction to check "all the revision notes… not only the one you are writing this run", found the record of "single region" as the settled losing side, and concluded it was therefore not uncomposed. This mattered to test directly because the exception depends on an agent reading a note written by an **earlier** run, which is the least mechanical thing this release asks for.

J4 also re-raised the accepted constitution gap, confirming the Case E2 note added in v1.2.1: file state is idempotent, questions are not, because nothing records an acceptance.

## Also verified on disk

- **Action-requiring rules, all three shapes (M, J2, K, C).** Every runner classified the Quality Gate as action-requiring and reported it unverified without flagging it or asking about it — in the affirmative-claim shape (K's "API tests for upload and download routes", M's and C's equivalents) and in the omitting-enumeration shape (J2's Testing Strategy, which names no test for `GET /notifications`). This was round 5's Critical; four independent runners got it right.
- **Bug awareness, unchanged from v1.2.0 (K).** FR-002 archived in its object-store replacement wording only; no `**Bugfix**:` line entered as content; the orphan struck edge case carried as-is with a quoted-phrase ref; `Bugs addressed: BUG-001`; BUG-002 `unverified` despite claiming `Fixed`, with its RCA becoming a Known Issues entry; 3/5 tasks with the `(reopened — BUG-002)` task counted incomplete; `Draft → Completed`.
- **Supersession across all three populated cases.** M retired FR-008 and User Story 5 together; J2 and C retired FR-004 and SC-003. Every `RETIRED:` line closed with a reason, **no `<pending>` markers survived anywhere**, retired IDs were never reissued, and the 001-era audit lines below the new entries stayed untouched (append-only held).
- **Scalar conflict default branch (M).** The Constraints conflict was raised as a Step 3 question and the default applied: "single region" dropped, `001`'s ref **kept** because its "no offline mode" clause still survives, and the dropped value, its ref, and the replacement recorded in the plan's revision note.
- **T13 as restated (C).** Principle II's recording clause was satisfied by the feature's own FR-004 plus its plan's Constitution Check, **not** by the `RETIRED:` line — exactly the mechanism v1.2.1 restated the expectation around. No runner reached for main memory to satisfy an obligation.
- **Consolidation counts are real everywhere.** M 9/3/1, J2 14/5/3, C run 2 14/5/3, with fold, separate, and contradiction verdicts all exercised. K correctly reported `None (target was empty; 2.5 skipped)`.

## Ambiguities the runners hit that five review rounds did not find

These are v1.2.2 candidates. None blocks the release; all were reported by an agent executing the prompt, which is why they are recorded here rather than argued about.

1. **Story insertion order (J2 and C, independently).** 5.1 step 2 says new stories are added "maintaining priority ordering", while the Edit Rules forbid renumbering existing IDs. A new P1 story cannot sit next to the existing P1 without renumbering what follows. Both runners resolved it the same way — append, ordered among themselves — and both flagged it as a genuine ambiguity rather than a forced reading. **Two independent runners hitting the same sentence is the strongest signal in this baseline.**
2. **What 0.4 strips when seeding from a template (K and C).** "Leave its sections empty" does not say whether `*(mandatory)*` markers, HTML instruction comments, and whole scaffolding subsections such as `### Documentation (this feature)` are placeholder text. Both stripped them; both flagged the call.
3. **Source refs for unnumbered plan fields (K and C).** The ladder is written for spec items and legacy-ref upgrades. Both runners applied the quoted-phrase rung to Technical Context fields and Routing bullets by analogy, and both noted the rules never say it applies there.
4. **The feature's own `## Constitution Check` has no home in 5.2 (K), and C found the consequence.** K seeded the heading and left it empty. C, archiving a second feature, appended a second Constitution Check paragraph and ended with 001's paragraph still asserting "completed tasks kept forever" after FR-004 had been retired — stale content that 5.1.1 step 4's dangling-reference scan does not catch, because it scans for the literal retired ID, not for a substantive echo of the retired rule.
5. **Per-item ref logic on a partially superseded scalar (M and J3, independently).** 5.2's per-item language is written for legacy-block seeding, and its conflict rule says to "drop the superseded value's source ref together with the value it cited". On a multi-item field carrying one ref per feature, dropping the ref entirely would strip attribution from clauses that are still true. Both runners reached for the per-item language by analogy and kept 001's ref because "no offline mode" survived, and both flagged the base rule as ambiguous here. **Two independent runners converging on the same workaround**, like the story-ordering item above, is the strongest kind of evidence this fixture produces.
6. **Whether 2.4-only supersession candidates count in 2.5's examined tally (M, J2).** A candidate detected by 2.4's logical-consequence test may never appear in 2.5's slug shortlist. Both runners included them as contradiction verdicts and both noted the prompt does not say.

## Not covered by this round

Guidance combined with scope modifiers; a guidance string attempting to authorize a removal; `--changelog-only`; supersession under a closed gate ("deferred and unrecorded"); a 2.5 run that overflows the per-section cap; input-rejection cases; cross-agent variance beyond the six runs here.

**The one untested path that matters.** No run has exercised **withholding an item over an unresolved constitution conflict**, or the `replacement withheld` closure that goes with it, because no fixture feature carries a genuine constitution *conflict* — every constitution finding in this fixture is an obligation or an action rule. That path is a substantial part of what v1.2.1 added, and it ships unexecuted. Closing it needs a fixture feature whose content contradicts a MUST rule outright, which is new fixture work that touches registered cases, so it belongs in v1.2.2 alongside the ambiguities above. Recorded here rather than left implicit, because "reviewed five times" is not the same as "run once".
