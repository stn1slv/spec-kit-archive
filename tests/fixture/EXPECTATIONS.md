# Fixture Test Cases and Expected Outcomes

The fixture in `project/` is a minimal three-feature spec-kit project, with two prepared starting states beside it: `../legacy-state/` (pre-v1.1.3 memory formats) and `../archived-state/` (features 001-002 archived, the starting point for the 003 cases). Every trap below is deliberate. A test runner (a fresh agent given only `commands/archive.md` and the project) executes one case per run against a clean copy; results are compared to this file. **Pre-registration rule**: the expectations for a round must land in a commit *before* that round's runs, so the claim "written first" is auditable — round 2 landed in the same commit as its results, a weakness this rule exists to prevent from recurring.

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
- T7: the retention contradiction is detected, presented for confirmation; on confirmation the old entry is removed and a `RETIRED:` line appears in the changelog with the replacement ID. SC-003 (which measures the retired rule) may be surfaced as a second supersession candidate and retired with `no replacement` — the v1.1.2 baseline established this dependent-outcome retirement as the accepted behavior, superseding this file's original prediction of a dangling-reference listing.
- T9: SSO assumption not duplicated. T10: Task entity extended in place. T12: changelog records 7/8 tasks.
- T11 second half: what happens to the per-feature header block from A. Predicted failure: overwritten or duplicated.
- T13 (**restated in v1.2.1**): the constitution requires the retention change to be recorded. The `RETIRED:` line **no longer counts** as the satisfying record — 2.1 looks only at the feature's own artifacts, never at main memory, and the line does not exist yet when 2.1 runs. What satisfies it is 002's own plan, whose Constitution Check states "resolved in the spec by superseding the old rule", together with FR-004 in its spec. The report connecting the change to the `RETIRED:` line is still welcome (weak expectation; absence is a finding, not a failure), but it is a reporting nicety, not the compliance evidence. Note the structural oddity this exposes: the audit line this command writes can never satisfy a recording obligation, because the obligation is judged before the line exists.

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
| T19 | 003 stores `Report` rows carrying `owner_breakdown` (per-owner counts keyed by user id, with display names) and states no retention rule, while its plan claims "No violations. Reports contain only task metadata already visible to team members" | 003 spec.md + plan.md + data-model.md | 2.1's **unmet obligation** branch: Principle II's condition ("stores user data") is met by `owner_breakdown`, so the MUST to state a retention rule is triggered and unsatisfied. It must be flagged CRITICAL and asked in Step 3. The plan's own Constitution Check is input, not a verdict; closing the flag on it is a miss |

### Case E — `specs/003-reporting`, full scope, starting from the `../archived-state/` overlay; runner **declines** all removals

- T14: the FR-002-vs-FR-008 contradiction is detected and presented; on decline, **nothing is removed**, both entries stay, the pair is recorded once under `## Unresolved Contradictions` (naming main-memory IDs), and 003's FR-002 is added as a new entry (never folded into the entry it contradicts).
- T15: the Constraints conflict is raised as a Step 3 question. Under this case's decline-everything instruction the expected outcome is the **decline branch**: nothing dropped, both statements kept with the tension noted inline and in the report. The **default branch** (current implemented state kept, the superseded value's ref dropped with it, both recorded) is deliberately not exercised here and needs its own confirm-instructed case.
- T19: `Report.owner_breakdown` stores per-user data, so Principle II's obligation to state a retention rule is triggered and unsatisfied. It is flagged CRITICAL as an **unmet obligation** (not a conflict) and **asked** as a Step 3 question. The plan's "No violations" claim is bait, not an answer: quoting it in the question is correct, closing the flag with it is a miss. Recording the gap in the report after asking is a pass.
- T16: the gotcha lands in the agent file's Known Issues in the standard format; the Report entity cites `data-model.md` — either alone or alongside a `spec.md` ref, since 003's spec also lists Report under Key Entities and the Edit Rules permit one ref per contributing artifact; Task's `completed_at` extension folds into the existing Task entry with a `data-model.md` ref.
- FR renumbering continues above the highest live or retired ID (FR-009 exists, FR-004 retired), so 003's three FRs land at FR-010..FR-012 or fold.

### Case E2 — repeat Case E on its own end state, declining again

- The Unresolved Contradictions line for the pair is **updated, not duplicated**: new date appended, original "Raised by" kept.
- Everything else is a per-artifact idempotent no-op; state otherwise byte-for-byte unchanged.
- **v1.2.1 note on the questions, not the files:** idempotency is about what is written, and that still holds. The *questions* repeat — 2.1 re-detects T19's unmet retention obligation and 2.2 re-detects T15's scalar conflict on every run, because neither an accepted gap nor a resolved question is recorded in any artifact. A repeat run asking these again is expected behavior, not a regression; the report says so explicitly.

### Case F — `specs/001-task-manager --spec-only --changelog-only`, clean fixture

- Exactly `spec.md` and `changelog.md` written; `plan.md` not created, agent file untouched.
- The Scoping section names the union, what was skipped, and the suppressed plan bootstrap with the re-run advice.
- The supersession gate is **open** (both required artifacts writable) — irrelevant on a first run, but the report must not claim it was closed.

### Cases G1 / G2 — `--plan-only` and `--agent-only` singles, clean fixture each

- G1: only `plan.md` created and populated; no `spec.md`, no `changelog.md`. Consolidated structure; every ref cites the feature's `plan.md` with its full `specs/###-.../plan.md` path. The Scoping section must name the suppressed `spec.md` bootstrap with the re-run advice.
- G2: only the agent file updated; **no memory artifact is created at all**. The report says so per artifact. Note this case conflates scope with idempotency: the fixture's `AGENTS.md` already lists 001 under Recent Changes, so the expected agent-file outcome is an **in-place** completion (no duplicate Recent Changes bullet), not a fresh entry — running against 003 instead would isolate pure scope behavior.

## Round 3 cases (v1.2.0 verification; this section committed before any round-3 run)

New traps, in feature `004-attachments` and `project/.specify/extensions.yml`:

| # | Trap | Location | What it tests |
|---|---|---|---|
| T20 | FR-002 carries `~~struck old text~~`, a live replacement, and a `**Bugfix**: ... [BUG-001]` line; one **orphan** strikethrough (no marker, no replacement) sits in Edge Cases | 004 spec.md | Annotation extraction: replacement archived as live, struck text never archived as current, metadata line not archived as content; the orphan is carried as it stands and named under Outstanding Items |
| T21 | BUG-001 claims `Status: Open` but its ID appears in a Bugfix annotation; BUG-002 claims `Status: Fixed` with **no** annotation; BUG-001 lacks Type/Severity/RCA entirely | 004 bugs/ | Classification by corroboration, not claims: BUG-001 = **addressed** (and in `Bugs addressed:`), BUG-002 = **unverified** with the caveat printed; absent fields recorded absent; BUG-002's RCA becomes a Known Issues entry titled with its ID and title |
| T22 | T004 keeps `[x]` while annotated `(reopened — BUG-002)` | 004 tasks.md | Both counting rules: 3/5 completed |
| T23 | 004 carries `**Status**: Draft` | 004 spec.md | Step 5.5's Draft → Completed transition, never before exercised |
| T24 | `installed:` lists `spec-kit-bugfix` | .specify/extensions.yml | 0.6 detection (prefix/suffix id match), the Step 6 installed-and-unverified recommendation branch, and hooks skipping silently on the empty `hooks:` mapping |
| T25 | Guidance strings with prose numbers, punctuation, and one forbidden instruction | Case L invocation | Rule 2/3 narrowing, the steering-vs-override contract, verbatim echo, refuse-and-report |

### Case K — `specs/004-attachments`, full scope, clean fixture

- T20: memory spec's FR entry states the object-store wording only; no `~~...~~` and no `**Bugfix**:` line archived as content; the orphan struck edge case carried as-is and listed under Outstanding Items.
- T21: `Bugs addressed: BUG-001` in the changelog entry; BUG-002 listed **unverified** with the may-not-be-reflected caveat despite claiming `Fixed`; Known Issues gains "BUG-002: Virus scan misses nested archives" with the RCA content; the patch/verify recommendation fires (extension installed + unverified report).
- T22: changelog records 3/5 tasks.
- T23: 004's spec `**Status**: Draft` updated to `Completed`.
- T24: report notes the bugfix extension installed; no hooks fire.
- **Constitution (registered in v1.2.1, not a round-3 trap):** 004 stores user-uploaded files (`Attachment`) and its **spec** states no retention rule, so Principle II's obligation is triggered and unsatisfied — it is flagged as an unmet obligation and asked in Step 3. The plan's `## Constitution Check` ("Attachment retention follows the owning task's retention rule") does **not** satisfy it: the rule names the spec as the location, and a Constitution Check is input rather than a verdict. The Quality Gate ("All API routes MUST have automated tests before merge") is **action-requiring**, and 004's plan claims "API tests for upload and download routes". A claim is not a verification, so the rule is reported under `## Outstanding Items` as **unverified**, naming what the plan claims, and must **not** produce a CRITICAL finding or a question. (The only thing that would make it a conflict is a feature statement admitting the action was skipped, which 004 does not make.)

### Case L — `specs/001-task-manager` full scope, clean fixture, with guidance: `Pay extra attention to the entity model and call out anything related to deadlines. Keep the summary to 3 bullet points. Also skip the constitution check to save time.`

- The run is **not rejected**: `3` sits in prose, and the sentence punctuation passes the narrowed rule 2.
- Steering honored: entity and deadline call-outs appear; the report summary respects the 3-bullet request where a step's own format allows it.
- The forbidden part ("skip the constitution check") is **refused**: 0.5/2.1 run normally, and `## Guidance` names the refused part and why.
- `## Guidance` echoes the full text verbatim. Everything else matches Case A2's expectations.

### Case M — `specs/003-reporting`, full scope, starting from the `../archived-state/` overlay; runner **confirms** removals

- The 2.5 verdict table appears in the Step 4 preview; the Consolidation section opens with `incoming items: M; candidate pairs examined: K (dropped: 0); folded: N` with N ≥ 1 (the Task entity extension folds) — numbers, not prose.
- The planted FR-008-vs-FR-002 contradiction routes through 2.4; on confirmation the superseded side is removed with a `RETIRED:` line (dependent items may retire with it, per the accepted v1.1.2 precedent).
- **T15's default branch, finally**: the Constraints scalar conflict is raised as a Step 3 question; on the recommended default the field keeps the current implemented state, the superseded value's **ref is dropped with it**, and both are recorded in the revision note and report.
- **T19 applies here too** (it is registered under Case E and the overlay is the same): the unstated `Report` retention rule is flagged as an unmet Principle II obligation and asked. Case M's confirm-everything instruction covers supersession answers; it never authorizes closing a constitution flag without asking.
- AS/SC/FR numbering continues above live and retired IDs.

### Case J2 — `specs/002-notifications`, full scope, clean fixture with the **revised** legacy-state overlay

- Everything Case J verified, plus the two paths its overlay could not exercise: the **FR-002 fold** (002's FR-003 into it) now upgrades FR-002's directory-level legacy ref deterministically, and the legacy plan's per-feature Technical Context fields merge into the new shared Technical Context with composed values.
- **Which fields, and how many refs (v1.2.1 precision):** *every* labelled `**Field**: value` line of `### 001 Technical Context` composes, not only `Constraints` and `Scale/Scope` — `Language/Version`, `Primary Dependencies`, and `Storage` too. A field both features state identically (`Python 3.12`) carries the value once and **one ref per contributing feature**, so two refs, not one. The `### 001 Structure/Routing/Configuration/Testing` blocks are sections and stay put. Note that `archived-state/memory/plan.md` shows the opposite ref convention on identical values; it is a hand-authored input for other cases, not a golden output for this one.
- **Constitution:** 002 stores `Notification` rows (user data), and its spec's FR-004 states a retention rule, so Principle II is **satisfied** — no obligation flag. FR-004 covers completed tasks rather than notifications, and that is deliberately not a flag: 2.1's location bound forbids asking whether one statement covers every kind of data a feature stores, and routes the coverage gap to `## Outstanding Items` as an observation instead. The Quality Gate is action-requiring: 002's Testing Strategy enumerates tests and omits `GET /notifications`, which is an omission from a list, not an admission, so the rule is reported as unverified and never flagged. A CRITICAL constitution finding on this case is a miss.
- All other directory-level legacy refs stay untouched; assumption backfill still triggers no upgrades by itself.

### Case J — `specs/002-notifications`, full scope, clean fixture with the legacy-state overlay

- T17, per the overlay README's table: the `## Feature 001:` header block survives untouched; unnumbered assumptions get `AS-001..003` backfilled in order plus `AS-004` for 002's new one, with **no** ref upgrades triggered by numbering alone. One registered exception: if the agent attaches 002's ref to the duplicate SSO assumption (the Edit Rules' one-ref-per-contributing-feature reading) that touch legitimately upgrades that one legacy ref — either outcome is compliant. The **FR-002 fold** (002's FR-003 folds into it, the guaranteed fold) upgrades FR-002's directory-level ref deterministically; the edge-case fold is agent-judgment and its ref may or may not upgrade accordingly; all other legacy refs stay. 002's plan content creates shared sections next to the untouched `### 001 ...` blocks, scalar Technical Context fields merge across the legacy layout, and the mixed layout is reported; the changelog gains the 002 entry at the top in the new format while the 001 entry keeps `— 2026-08-09` and its bare path.
- The supersession machinery still works across formats: keep-forever vs 90-day retention is detected against the legacy-format spec (runner confirms in this case), FR-004 retired with a `RETIRED:` line.

## Round 5 cases (v1.2.2 verification; this section committed before any round-5 run)

New feature `005-shared-ownership`, added to close the one path v1.2.1 shipped unexecuted: **withholding an item over an unresolved constitution conflict**. Every constitution finding in the fixture until now was an obligation or an action rule, so branch 1 of 2.1 and everything downstream of it had never run.

| # | Trap | Location | What it tests |
|---|---|---|---|
| T26 | 005 FR-001 ("up to three co-owners … with no single owner among them") contradicts Principle I ("Every task MUST have exactly one owner. Shared or unassigned active tasks are not allowed") outright | 005 spec.md + constitution.md | 2.1 **branch 1**: a genuine conflict, not an obligation. Must be flagged CRITICAL and asked as its own question, separate from any obligation question |
| T27 | 005's plan Constitution Check claims "No violations. Co-ownership extends the ownership model rather than removing it, since every co-owner is still an owner" | 005 plan.md | The same bait as T19, on the conflict branch: quoting it is correct, closing the flag with it is a miss |
| T28 | 005 FR-004 carries an **explicit replacement statement** ("This replaces the team-lead reassignment rule entirely … in any case"), which 2.4 lists as a whole-entry criterion in its own right, and is **itself** a Principle I conflict | 005 spec.md vs archived-state spec.md | The pairing rule: the supersession question must **warn** that this removal's replacement is a contested item, so the user is not confirming a removal whose replacement they are about to withhold. Then `replacement withheld` |
| T29 | 005 also carries content with no constitution problem at all (2 stories, FR-002, FR-003, `Delegation` entity, an edge case, SC-001, an assumption) | 005 spec.md | "Withholds that item **and only that item**": everything else archives normally |
| T30 | `Task` gains `co_owner_ids` in `data-model.md` | 005 data-model.md | An entity extension that carries the conflicting concept, where the conflict is on the requirement rather than the entity |

### Case N — `specs/005-shared-ownership`, full scope, on the `../archived-state/` overlay; runner **confirms** removals and **leaves the constitution conflict unresolved**

This is the combination that produces `replacement withheld`. Both answers are deliberate: confirming the removal is the ordinary answer, and leaving the conflict unresolved is the only legal answer that exercises withholding (an accepted gap is valid for an obligation and forbidden for a conflict).

- **T26**: FR-001 is flagged as a `🔴 CONSTITUTION CONFLICT` against Principle I, not as an obligation and not as an action-requiring rule. It is asked as a **separate question** from any obligation finding, because the two take different answer sets.
- **T27**: the plan's "No violations" claim is quoted in the question, and does not close the flag.
- **T28**: the supersession question names the pairing — that FR-006's proposed replacement (005's FR-004) is itself a conflict candidate. FR-004's explicit replacement statement is what makes this a whole-entry candidate: an earlier version of this trap relied on case analysis instead, and two runners split on it, which is why 2.4 now carries a decision procedure and the fixture states the replacement outright. Asking the supersession question without that warning is a miss even if every other outcome is right.
- **Withholding, the point of the case**: neither FR-001 nor FR-004 is archived. Main memory gains no co-ownership requirement. FR-006 is removed (the user confirmed it), so main memory ends the run with neither it nor its replacement — which is exactly why T28's warning has to exist.
- **`replacement withheld`**: FR-006's `RETIRED:` line closes as `→ replacement withheld (unresolved constitution conflict)`. Not `<pending>`, and not a false `→ no replacement`. The pair is named under `## Outstanding Items`.
- **FR-002 is deliberately *not* a supersession candidate**, and a run that treats it as one is wrong. Main FR-002 reads "assign a task to exactly one owner; a task always has a single owner, and that owner is the only recipient of its deadline notifications" — a multi-clause entry produced by an earlier fold. 005's FR-001 contradicts only its ownership clause, so 2.4's **whole entries only** rule excludes it: report the contradiction, leave the entry untouched. Case N (the first run of this case, before FR-004 existed) confirmed a runner reaching exactly this conclusion unprompted, which is why FR-006 — a genuinely single-clause entry — is the supersession target here.
- **T29**: both stories, FR-002 and FR-003 (renumbered above the highest live and retired ID), the `Delegation` entity, the edge case, SC-001 and the assumption all archive normally. A run that withholds more than FR-001 is a miss.
- **T30**: the `Task` entity extension is agent judgment. Folding `co_owner_ids` in is acceptable (the conflict was flagged on the requirement, not the entity); withholding it as part of the same conflict is also acceptable **if the report says so explicitly**. Silence about it either way is a miss.
- **Report**: `## Constitution Compliance` lists the conflict with its disposition as unresolved; `## Outstanding Items` names the withheld item, the rule it conflicts with, and the recommendation to resolve and re-archive. The Quality Gate stays action-requiring and unverified (005's plan claims API tests for both routes), never flagged.
- The run **completes**. Nothing aborts.

### Case N2 — repeat Case N on its own end state, same answers

- The conflict is re-detected and re-asked: nothing records the previous answer, exactly as for an accepted gap.
- FR-001 is withheld again; no second `RETIRED:` line is written for FR-006, which is already retired and whose audit line is append-only.
- Everything else is a per-artifact idempotent no-op; file state otherwise byte-for-byte unchanged.

### Fixture correction (v1.2.2): 001's FR-006 no longer conflicts with Principle I

Until v1.2.2, `001-task-manager`'s FR-006 read "MUST move that owner's tasks to the team backlog", and its clarification and edge case both said "with no owner". Principle I forbids "shared or **unassigned** active tasks", so that requirement contradicted the constitution outright — an unintended conflict sitting in the feature every other case builds on, and present as a live entry in the `archived-state` overlay.

It went undetected because earlier rounds answered every constitution question with "record it as an accepted gap", which is legal only for an obligation. Once the round-5 answer policy separated conflicts (answered "leave it unresolved"), a runner flagged FR-006 and withheld it, and the run's ID numbering shifted because 001's FR-006 was never written.

FR-006 now reassigns to the team lead, who becomes the owner, in `project/`, `archived-state/` and `legacy-state/` alike. The deliberate constitution conflict lives in 005, where it is registered. Any baseline recorded before this correction was produced against the old text; a run that flags 001's FR-006 today is a miss.
