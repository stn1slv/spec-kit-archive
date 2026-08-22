# Baseline: v1.3.0 (branch `feat/1.3.0-upstream-compatibility` at `43d165b`) against the fixture

Expectations pre-registered in `3fe4c91`, **before** any run in this round. Fresh agents, one per case, each given only `commands/archive.md` and its own working copy, forbidden from reading `EXPECTATIONS.md`, the baselines, the CHANGELOG, or git history. Sixteen invocations across thirteen cases. Every claim below was checked against the files each run wrote, not against what the run said it did.

| Case | Invocation | Result |
|---|---|---|
| Q1 | timestamped feature, full scope, clean | Pass |
| Q2 | nested feature, full scope, clean | Pass |
| Q3a | `specs/billing/006` prefix expansion | Pass |
| Q3b | `specs/billing` scope directory | Pass |
| Q3c | `specs/006` must not reach into a scope directory | Pass |
| Q4a-d | four input rejections | Pass |
| Q5 | `to` carve-out against a timestamped name | Pass |
| R1 | agent context, config branch | Pass on T35-T38; invalidated by a fixture defect, repaired and re-run as R1b below |
| R2 | `--agent-only` on the same overlay | Pass |
| P | both bug layouts in one run | Pass |
| L6 | legacy Recent Changes bullet upgraded in place | Pass on T45; **found an unrelated defect**, see below |
| R3 | agent context via the defaults lookup | Pass |
| Ctl | control, `004` on clean `project/` | Pass |

The three changes this round exists to verify are now confirmed by execution: **B** (feature-directory identification) by Q1, Q2, Q3a-c and Q4a-d; **C** (agent context discovery) by R2 and R3; **D** (the repo-level bug layout) by P. Case Ctl confirms the shared-text edits did not perturb the v1.2.2 result.

## What the round proved

**Case P is the strongest result.** Every canary held. The three phrases in files the command never opens (`normalize EXIF orientation`, `enforce the per-team storage quota at commit time`, `reject any image whose orientation tag cannot be parsed`) are absent from memory and from the agent file, and so are the two sharper ones inside `assessment.md`'s own off-limits sections (`canonical imaging pipeline`, `sole gatekeeper`). The Prevention Rule in Known Issues was written from the root-cause hypothesis rather than lifted from Proposed Remediation, which is the leak the round-2 review predicted and this fixture was rebuilt to catch. `report-timezone` appears nowhere: not opened, not classified, not listed, and the report states the 3-slugs-2-attributed count so the exclusion is visible. Both slugs reach `**Bugs addressed:** BUG-001, thumbnail-orientation, attachment-quota-drift` verbatim, with no invented `BUG-003`, and `attachment-quota-drift`'s `not-applied` status is named as a discrepancy under `## Outstanding Items` while the annotation still wins. T43's bare-id `bug` match fired.

**R3 is the branch a real project most often takes**, and it works. The config parses, ships the fresh-install empty state, and ends branch (a) without resolving anything; branch (b) reads `integration: copilot` and lands on `.github/copilot-instructions.md`. No `AGENTS.md`, `CLAUDE.md` or `GEMINI.md` was created, and the legacy basename bullet was upgraded in place.

**T45 is closed.** L6 ends with exactly one `- specs/001-task-manager:` bullet, upgraded in place from the legacy form, and 002's bullet is left in the legacy form because this run did not touch it. The duplicate-bullet regression that the `FEATURE_ID`-strict rule would have introduced does not occur.

**T46 reproduced.** The fixture's own script exits 0 and reports `FEATURE_DIR = specs/002-notifications` whatever feature is archived. Ctl names both features and correctly claims no walk-up fallback, because the decision keys on `REPO_ROOT`.

**The rejection paths ran for the first time.** All six write-nothing cases left their working copies byte-identical, verified by SHA-256 over every file before and after.

**Cross-run determinism held** where it was testable: Q1 and Q5 archive the same feature from the same clean state and produce identical IDs and the same ref count.

## Case R1 was invalidated by a fixture defect, since repaired

`agent-context-state/AGENTS.md` and `docs/agent/CLAUDE.md` both ship a `- specs/001-task-manager: ...` Recent Changes bullet already in `FEATURE_ID` form, and R1 archives `specs/001-task-manager`. So the idempotency rule fires on both targets before the case can test what it was written to test: that a config naming several files causes the same section set to be written to each.

The run still passed every marker trap. `GEMINI.md` and `QWEN.md` were skipped and named, `QWEN.md` was not created, the `<!-- TEAM CONTEXT ... -->` block came through byte-for-byte, and the nested `docs/agent/CLAUDE.md` was written. But `AGENTS.md` was left untouched, so the pre-registered "both files are written" was not observed here.

**R2 covered it at the time.** Same overlay, same feature, `--agent-only`, and that run wrote both targets, so the multi-target write was confirmed by the case that was not meant to confirm it. The overlay has since been repaired to name only `002-notifications`, and both cases were re-run: see round 6b below.

## What the round found in the command

**Retired items resurrect on re-archival** (found by L6, unprompted). Re-archiving `001-task-manager` onto the `archived-state` overlay added `FR-010` and `SC-006`, carrying that feature's `FR-004` and `SC-003` whose main-spec entries `002-notifications` had already retired. The main spec now asserts both "kept forever, never deleted" and "deleted after 90 days".

The mechanism: the retired-ID list governs **ID assignment** only. Nothing compares an incoming item's *content* against the `RETIRED:` audit trail, and the completeness promise forces the item in when it has no counterpart to fold into. The changelog line `RETIRED: FR-004 (from specs/001-task-manager/spec.md) → replaced by FR-009` states exactly what would suppress this, and no rule reads it that way.

This is not a v1.3.0 regression. Source refs already carried the `specs/`-prefixed path before this release, so v1.2.x behaves identically. The run recorded both pairs under `## Unresolved Contradictions`, so nothing is silent. It is a genuine gap, newly found, and it belongs to a later release.

**Idempotency's scope is undefined at the artifact level** (found by R1 and R2 disagreeing on identical input). The Edit Rules say a feature "has already been merged into an artifact" when that artifact carries source refs naming it, and then give only two consequences: do not append a second copy, do not attach a duplicate ref. Whether an already-merged artifact should still be completed section by section is never stated. R1 read it as "skip the artifact" and left `AGENTS.md` alone; R2 read it as "do not duplicate the entry" and extended Active Technologies and added the missing section. Two capable runners, same state, opposite files.

**Three report-only under-specifications**, each flagged independently by six or more runners:

- The `## Consolidation` template says "Always give the 2.5 numbers first" and also offers "None (target was empty; 2.5 skipped)". When 2.5 never ran there is no honest value for K and N, and printing zero is the reading the same sentence forbids. The Done Criteria settle it in favour of the skip reason; the template's "Always" is what a runner reads first. Most runs printed both.
- 5.3 step 2 says to update the section set "creating any that are missing", but `Known Issues & Gotchas` and `Commands` have strictly conditional bodies. With no `research.md` and no attributed root-cause section, literal compliance creates an empty heading in a file this command does not own. Every runner declined and said so.
- 5.4 prescribes no document title for a newly created `changelog.md`. Three distinct first lines appeared across the round: `# Main Project Changelog`, `# Changelog`, and a bare `## Merged Features Log`.

None of these changes archived content.

**Two seeding ambiguities that do change the file.** 0.4's placeholder-removal rule does not clearly cover the plan template's `**Label**: [bracketed stand-in]` Technical Context lines: they are labelled fields, not headings, so neither the removal bullet nor the keep-the-heading tie-break plainly reaches them. Runners split between deleting the lines and keeping the labels with empty values. 5.2 then rewrites them either way on a full-scope run, so the divergence is invisible here, but a `--plan-only` run on a feature stating fewer fields would differ. Separately, `## Complexity Tracking` is a per-feature planning artifact like `## Constitution Check`, which 0.4 names explicitly for removal, and is itself named nowhere; the keep-if-unsure tie-break carried it into main memory as a permanently empty heading.

**New shared plan sections have no specified position.** 5.2 names them and the Edit Rules say to preserve existing ordering, but nothing says where a section that did not exist before goes. Every runner placed Routing, Configuration and Testing Strategy after Project Structure, so it did not bite, but two compliant runs could order the file differently.

**"Nothing else is ever read" is not literally executable** with whole-file reads (found by three runners independently). Opening `BUG-001.md` at all surfaces its `## Amendment` section. Every run complied with the rule's purpose, and the FR-003 amendment reached no memory file, but the rule is phrased as a constraint on reading when it is a constraint on merging.

## Ctl against the v1.2.2 Case K result

Archived content matches on every scored item: `3/5` tasks, `Bugs addressed: BUG-001`, `Status` Draft to Completed, `FR-001..FR-003`, the orphan struck edge case carried with its `~~` markup and named under Outstanding Items, no `**Bugfix**:` line archived as content, BUG-002's root cause in Known Issues, and 001's legacy bullet untouched because it is a different feature.

One difference, in the report only and outside the pre-registration: the run states `0 repo-level slugs, 0 attributed` where the expectation said the report would say nothing about attribution counts on a project with no `.specify/bugs/`. Stating the zero is consistent with the rule that a silent non-attribution must stay visible, so this is recorded as an acceptable difference rather than a miss.

## Round 6b: re-runs after the meta-review

A four-model review of the release branch (`opus`, `fable`, `pro`, `flash`) returned sixteen findings. Nine were applied. Four of them changed behaviour that round 6 had already exercised, and one was the R1 fixture defect recorded above, so those cases were re-run against the amended command and the repaired overlay. Expectations for all five were pre-registered before the runs.

| Case | Covers | Result |
|---|---|---|
| R1b | the repaired `agent-context-state/` overlay | Pass |
| R2b | same overlay, `--agent-only` | Pass |
| Q6 | branch (b) must not expand outside `SPECS_DIR` | Pass |
| Q4d-b | rule 2 still rejects after the three-digit narrowing | Pass |
| Q5b | guidance still accepted after the same narrowing | Pass |

**Case R1 is now genuinely verified.** With the overlay naming only `002-notifications`, both writable targets received a real write: `AGENTS.md` and `docs/agent/CLAUDE.md` each gained exactly one `- specs/001-task-manager:` bullet, each kept its legacy `- 002-notifications:` bullet untouched, the `<!-- TEAM CONTEXT ... -->` block came through byte-for-byte, and `GEMINI.md` and `QWEN.md` were skipped and named without `QWEN.md` being created. R2b repeats it under `--agent-only` with `.specify/memory/` holding only `constitution.md` afterwards. The multi-target write is now proved by the case written to prove it, not only by its sibling.

**Q6 confirms the new branch (b) guard.** `sr` no longer expands to `src/` at the repository root: the ladder rejects the match for lying outside `SPECS_DIR` and the run stops with rule 4's resolution error rather than 0.2's misleading "Missing required files", writing nothing.

**The rule 2 narrowing is behaviour-preserving where it must be.** `specs/billing/006 thru 008` is still rejected, because `008` clears the new three-digit floor, and `specs/20260814-101500-timestamped-export` with guidance containing `3`, `404` and `migrate to postgres` is still accepted and archived with the guidance echoed verbatim. Both working copies for the rejection cases are byte-identical before and after.

### Still open after this round

Two review findings were left for a decision rather than an edit, and both are recorded here so they are not rediscovered as new:

- **Idempotency's artifact-level scope.** Whether an artifact that already names the feature should still receive its missing or updated sections is undefined, and R1 and R2 demonstrated two runners reading it in opposite ways. The repaired overlay removes the collision from these cases but does not settle the rule.
- **Exact-heading coupling.** Requiring `## Root Cause Hypothesis` and `## Root Cause Analysis` verbatim, with no tolerance for a variant, is what stopped this release reading a heading no tool emits. It also means an upstream rename breaks extraction silently. The trade-off is deliberate and unresolved.

The re-archival defect found by L6, where a feature's retired items return under fresh IDs, also remains open and belongs to a later release.
