# Baseline: v1.2.2 (branch `feat/withholding-fixture-and-wording` at `d39c92a`) against the fixture

Expectations pre-registered across `e6b71ec`, `3f3ccb0` and `374910a`, all **before** the runs that judged them. Fresh agents, one per case, each given only `commands/archive.md` and its own working copy, forbidden from reading `EXPECTATIONS.md`, the baselines, the CHANGELOG, or git history. Every claim verified against the files each run wrote.

**Every item in this release came from executing the command, not from reading it.** v1.2.1 had already been through five review rounds across three models, 70 findings, all applied. None of them found any of this.

| Case | Invocation | Result |
|---|---|---|
| K2 | 004 full scope, clean fixture | Pass |
| M2 | 003 on `archived-state/`, removals confirmed | Pass |
| C4 | 001 then 002 in sequence, clean fixture | Pass |
| Nb4 | 005 on `archived-state/`, conflict left unresolved | Pass, including the withholding path |

Earlier runs in the round (`N`, `Nb2`, `C2`, `C3`, `Nb3`, `M2`, `K2`) each found something and are described below where they did.

## What this release fixed, and how each was found

**Story insertion order** (found by J2 and C independently). "Maintaining priority ordering" could not be satisfied alongside "never renumber existing IDs". Story numbers are IDs: they are retired on supersession and never reused. New stories are appended, ordered among themselves by the feature's own document order, and priority is read from the heading. C4 confirms: User Stories 1 (P1), 2 (P2), 3 (P3), 4 (P1), 5 (P2).

**Per-item ref drop on a multi-item scalar** (found by M and J3 independently). `Constraints: single region, no offline mode` carries one ref per feature, so dropping the ref with the superseded item stripped attribution from clauses still true. M2 confirms: "single region" dropped, 001's ref kept because "no offline mode" survives.

**Template seed scope** (found by K and C). "Leave its sections empty" never said whether `*(mandatory)*` markers, instructional comments and placeholder entries count. The tie-break "if unsure, keep it" resolved the wrong way against the fixture's own template, where `### User Story 1 - [Brief Title]` is both a heading and a placeholder: keeping it would make the next run number the first real story as User Story 4. K2 confirms zero `Brief Title` occurrences and the first story numbered 1.

**Source-ref ladder for plan content** (found by K and C). Written around spec items, leaving unnumbered plan fields to inference. A labelled Technical Context field cites its own label.

**The feature's `## Constitution Check` is never archived** (found by K, its consequence found by C). Archiving it left the main plan asserting "completed tasks kept forever" after that rule was retired.

**Supersession leaves prose no ID scan finds** (found by C). Retiring `FR-004` does not remove a sentence describing what it required. The bounded look — slug the retired entry, examine only sections sharing its object noun, cap at five — found `SC-003` in C4 and, in M2, `summaries.py` in the plan structure tree, which `BASELINE-v1.2.0.md` had recorded as an acknowledged design bound the command could not address.

**2.5 count boundary** (found by M and J2). The examined/folded numbers describe pairs 2.5 shortlisted; a candidate 2.4 found by its own criteria is reported under Superseded Requirements and does not enter them.

**The entity closure's third trigger** (found by N and N-b **disagreeing**). 5.1's entity exception already prescribed keeping the existing entry and recording the pair, but named only two triggers, so whether it covered a definition withheld by a conflict was a guess. Nb2 onward confirm the record is written.

## The withholding path, closed at last

v1.2.1 shipped one path unexecuted: withholding an item over an unresolved constitution conflict, and the `replacement withheld` closure. Feature `005-shared-ownership` was added to reach it. It took three attempts, and each failure was in the fixture rather than the command:

- **N**: withholding worked and was precisely scoped, but `replacement withheld` was never reached — main `FR-002` is multi-clause, so 2.4's whole-entries rule correctly excluded it. The runner worked that out unprompted. Registered as a positive expectation so a future run that treats a partial contradiction as a candidate is scored as the miss it is.
- **Nb2**: retargeted at the single-clause `FR-006`, and a second runner judged *that* pair partial too. Two capable runners split on the same two sentences.
- **Nb3, Nb4**: `FR-004` now states outright that it replaces the rule entirely, which 2.4 lists as a whole-entry criterion in its own right. Both runs produce `FR-006` retired with `→ replacement withheld (unresolved constitution conflict)`, no `<pending>`, and co-ownership archived nowhere.

**A fixture trap that depends on inference tests the inference, not the rule.** That is the lesson, and it cost two rounds to learn.

## What the runs found in the fixture itself

**001's `FR-006` contradicted the constitution** (found by C2). It moved a deactivated owner's tasks to the team backlog "with no owner", and Principle I forbids unassigned active tasks — an unintended conflict in the feature Cases A, C, D and J all build on, and a live entry in the `archived-state` overlay.

It hid for five rounds because of the **test harness**, not the command: earlier rounds answered every constitution question with "record it as an accepted gap", which is legal only for an obligation. A conflict answered that way never reaches the withholding path and leaves no trace. Splitting the answer policy exposed it immediately. C4 confirms the correction: `FR-006` archives normally, IDs stable at `FR-001, 002, 003, 005, 006, 007, 008, 009`.

## Where review and execution agreed

The narrow review of the three-paragraph delta predicted that fixing the revision note "directly under the document title" would break on artifacts with no title and on files already carrying notes elsewhere. Both runs then proved it: Nb3 wrote a blockquote into `changelog.md` under a title an earlier runner invented and continued `AGENTS.md`'s comment convention; C3 wrote notes into `spec.md` and `plan.md` only. Resolved by scoping — only `spec.md` and `plan.md` carry notes — which also retired the missing-anchor problem and the collision with 5.3's marker rule. Nb4 confirms: 3, 3, 2 untouched, 2 untouched.

The review also caught that the new 2.4 procedure would have stranded **re-raised pairs** forever, since they have no incoming item and no feature spec, so all three tests are inapplicable and the "treat it as partial" tie-break would fire on every one, making 5.1.1's re-raised branch dead text.

## The ambiguity that changed an outcome

C3 flagged, unprompted, that it could not tell whether a feature plan's `## Constitution Check` can satisfy "changing a retention rule MUST be recorded, never silent". It archived `FR-009` and said plainly that the other reading withholds `FR-009` entirely. Same input, opposite outputs, on the central requirement being archived.

2.1 now separates the two: **"we checked and it is fine" is a verdict and closes nothing; "here is what we changed and why" is the record the rule demanded.** The rule discriminates in both directions, which is the test that matters:

- **C4** met "conflicts with the original keep-forever wording; resolved in the spec by superseding the old rule", accepted it, and archived `FR-009` — quoting the new rule text as its basis.
- **Nb4** met "No violations. Co-ownership extends the ownership model", and rejected it in those words: *a verdict, not a record of a change, so it does not close this flag*.

## Still not covered

Guidance combined with scope modifiers; a guidance string attempting to authorize a removal; `--changelog-only`; supersession under a closed gate ("deferred and unrecorded"); a 2.5 run that overflows the per-section cap; a re-raised pair actually re-raised (the exemption added this round is reasoned, not executed); input-rejection cases; cross-agent variance beyond the runs here.
