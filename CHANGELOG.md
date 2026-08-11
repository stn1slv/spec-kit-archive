# Changelog

All notable changes to the Archive extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.1] - 2026-08-11

### Fixed

- **Constitution obligations left unmet are now flagged.** Step 2.1 only ever tested for
  *conflicts*, so a MUST principle that requires a statement from any feature meeting a
  condition ("every feature that stores user data MUST state its retention rule") could be
  violated by omission without anything to flag: nothing in the feature contradicted the text,
  the required content was simply absent. 2.1 now checks both kinds and has a second CRITICAL
  block for the obligation form. The check is bounded — an obligation is examined only when the
  feature actually does the thing the rule conditions on; it is met by a statement in the feature's
  own artifacts, anywhere among the ones Step 1 reads unless the rule names a location, and it
  **never withholds content**, since nothing in the feature contradicts
  anything and the triggering content is the feature's ordinary work. The statement need only be
  this feature's own — whether one statement covers every kind of data the feature stores is a
  judgment for the user, not a flag from this step. Main memory never satisfies
  an obligation: rules of this kind bind each feature, so an earlier feature's compliance is not
  this one's. Both branches now cover Core Principles, Architecture Standards, and Quality Gates
  alike. Step 3 states the resolution options explicitly (amend the feature spec and re-run, accept
  the gap, or declare the obligation not triggered), notes that the run archives either way since
  there is no abort path from an answer, and forbids this command from writing the missing
  statement into **any** artifact — memory or feature spec — because text placed in the feature
  spec would be read back as a source on the next run, producing a ref that looks honest and is not.
  The "materially changes scope or correctness" filter is also scoped explicitly to discretionary
  questions, so an Always-ask category can no longer be dropped on the grounds that no answer would
  change what gets written. Conflicts and obligations are asked as separate questions, since an
  obligation may be closed as an accepted gap and a conflict may not.
- **What an unresolved constitution conflict does is now stated.** The command said in three places
  that a conflict must be resolved "before archival can proceed" and nowhere said what happens when
  the user does not resolve one, leaving an agent free to halt the run or to continue. It now says
  once: the conflicting item alone is withheld, everything else in the feature archives normally,
  the run completes, and the withheld item is named in the report with a re-archive recommendation.
  Nothing in Steps 3 to 7 aborts a run. Withholding is named as the one exception to the file's
  completeness absolutes, and its three consequences are stated: the item's 2.5 fold does not
  happen, a `RETIRED:` line whose replacement was withheld closes as `replacement withheld`, and
  the conflict question's legal options exclude "archive it anyway", which no answer can authorize.
- **Constitution rules requiring an action are no longer flagged.** A rule like "All API routes MUST
  have automated tests before merge" requires something to be *done*, not stated, and this command
  reads artifacts — it cannot inspect a codebase or a CI run. Treating silence in a plan as an
  unmet obligation would have raised a mandatory question on almost every feature. Such rules are
  now reported under Outstanding Items as **unverified**, and only an artifact plainly stating the
  action was skipped is a conflict.
- **A feature's own Constitution Check no longer settles the question.** A `plan.md` verdict of
  "No violations" records what the author believed at planning time, before the work was done.
  It is quoted in the Step 3 question when relevant, but it can no longer close a 2.1 flag on
  its own, and neither can the command's own judgment that a violation looks minor: that call
  belongs to the user in Step 3.
- **Shared plan scalars seeded beside legacy blocks were understated.** When 5.2 created a
  shared Technical Context in a `plan.md` still holding pre-v1.1.3 per-feature blocks, scalar
  fields (Constraints, Scale/Scope, and the like) were seeded with only the current run's value
  while earlier features' values stayed quarantined in the legacy blocks. A scalar has one slot,
  so the shared field read as *the* current implemented state while describing part of it, and
  every later run folded into the understated field. Seeding now composes the legacy values
  together with this run's, one source ref per contributing feature, with the legacy contributor's
  ref taken through the existing Legacy refs ladder. The rule is a condition rather than a one-time
  event, so a shared field an earlier version already created is still completed — with one
  exception, so that a legacy value the revision note records as the losing side of a resolved
  conflict is not re-composed on every later run, quietly undoing the user's decision. Step 2.2
  gained a detection pass for Technical Context scalar conflicts, because 5.2 asks for a Step 3
  question about them and Step 3 runs once, before the writing step that used to find them. Bounded
  to the
  labelled `**Field**: value` lines of a legacy block's Technical Context, judged by structure
  rather than by the shape of the value: a section of enumerated entries stays put, because the
  reader unions a shared section with a legacy one and loses nothing, while a labelled field
  appears once and so reads as the whole value for that label. Legacy blocks are still
  never modified in either branch — when values conflict, the losing value is simply not carried
  into the shared field — and the report says for each legacy line whether it is now a duplicate or
  a superseded statement, with a recommendation to prune by hand.

## [1.2.0] - 2026-08-10

### Added

- **Guidance text.** Free-form text after the feature path is accepted again, restoring the
  channel every core spec-kit command offers (v1.1.1's strict rejection broke a working guided
  workflow, reported in #3). Guidance steers attention, emphasis, and report detail; it cannot
  add sources, skip steps, change scope or IDs, or authorize removals, and it is echoed
  verbatim in the report under a new `## Guidance` section so runs stay auditable. Flags are
  still validated strictly; ranges, feature paths, and globs inside feature references are
  still rejected before guidance is classified.
- **Bounded `bugs/` support**, designed around spec-kit-bugfix's actual behavior: its patch
  step already writes amendments into the feature's own artifacts, so requirement text is
  still never taken from bug reports. What is new: a per-report status audit (ID, Type,
  Severity, Status — with Status explicitly treated as unreliable, per field experience),
  a `Bugs addressed:` line in the changelog entry, Root Cause Analysis feeding the agent
  file's Known Issues, awareness of whether a bugfix extension is installed, and extraction
  rules for bugfix annotations inside feature artifacts: struck-through text is never
  archived as live, `**Bugfix**:` lines are metadata not content, reopened tasks count as
  incomplete (#3).
- **Consolidation detection pass (Step 2.5)**, modelled on core `/speckit.analyze`:
  imperative-phrase slugs key every incoming item, existing entries are indexed per section,
  candidate pairs are shortlisted by slug overlap (capped at 20 with an overflow count), and
  each pair gets one of three verdicts — fold, separate, or contradiction (routed to
  supersession). The report now always states `incoming / examined / folded` counts, so
  "zero duplicates" is legible as "examined and found distinct" rather than "did not look"
  (#3, the original report's core complaint).

### Changed

- 5.1 folding of the spec-side categories is gated on the 2.5 verdict table instead of ad-hoc
  judgment while writing — two baseline runs folded the same pair differently; the table
  records that judgment where it can be audited and previewed before any edit.
- The unrecognized-argument rejection narrowed to `--flags`; bare text is guidance.

## [1.1.3] - 2026-08-10

### Fixed

- **User stories are archived as whole blocks.** 5.1 listed which story sub-fields to carry, and
  any field not on the list survived only by agent whim: baseline testing showed one run keeping
  the `Why this priority` lines and another silently dropping them — the fourth instance of the
  same failure class (assumptions in 1.1.0, the merge-path gap in 1.1.1, acceptance scenarios in
  1.1.2). The extraction and merge rules now carry each story's entire block, every labelled
  field included, instead of an enumerated subset (#3).
- **`bugs/` files can no longer rewrite archived requirements.** The Allowed Sources boundary was
  location-based, so files a bugfix extension writes inside the feature directory were formally
  permitted content: in baseline runs one agent rewrote FR text from a bug file's amendment and
  cited it, while another refused to read the same file. `bugs/` is now a named exclusion — its
  presence is reported, its content never merged. Native bug fold-in is planned as a separate
  feature (#3).
- **The `## Clarifications` session log has a defined fate: deliberately not archived.** Core
  `/speckit.clarify` already integrates every accepted answer into the sections this command does
  archive, so the log would duplicate them. Previously undefined, which produced an empty
  `## Clarifications` heading in one field run and silent exclusion in others (#3).
- **Memory artifacts no longer open with one feature's metadata.** Seeding copied the template's
  per-feature header (`# Feature Specification: ...`, `**Feature Branch**`, `**Created**`), which
  a second feature must then overwrite or duplicate. Seeds are now titled as project-level
  documents and the per-feature header block is dropped (#3).
- **Source refs name the artifact the content actually came from.** The ref template hard-coded
  `spec.md`, so a plan-derived entry would carry a ref asserting a provenance that is not true.
  Refs now cite `plan.md`, `data-model.md`, or whichever artifact contributed the content.
- Step 5.3 no longer depends on the agent-file template, which recent spec-kit versions removed
  (spec-kit #2259 replaced it with a CLI-managed marker block). The section set is defined in the
  command itself, and writing inside a tool-managed marker block is forbidden.

### Added

- Assumptions get `AS-###` IDs in the main spec, continuing above the highest existing ID, so
  they can be cited, deduplicated, and superseded like requirements (#3).
- A test fixture (`tests/fixture/`): a minimal two-feature project with thirteen deliberate
  traps, pre-registered expectations, and a recorded v1.1.2 baseline of four executed runs. The
  baseline found the `Why this priority` loss and proved the `bugs/` nondeterminism.

### Changed

- The changelog's Merged Features Log is now newest-first, entry headers say
  `archived YYYY-MM-DD` (the date was always the archival date; now the label says so), and the
  `**Spec:**` line is a relative link to the feature's spec file (#3).
- The main `plan.md` structure is now stated: one consolidated document mirroring the plan
  template's sections, never per-feature blocks — previously unstated, and one field run
  produced per-feature headers (#3).

## [1.1.2] - 2026-08-09

### Fixed

- The command now states the complete list of files it may take content from, and forbids
  everything else. It previously said what to read but never that this was the only permitted
  source, so an agent asked to "ensure completeness" would consult git history, `git log`,
  notes files, or its own memory store, and in one reported run recovered a deleted
  `.specify/memory/spec.md` and continued from it. That makes runs non-reproducible, bypasses
  the first-archival bootstrap entirely, and — because content from outside still receives an
  item-level `[Source: ...]` ref — makes the traceability refs assert a provenance that is not
  true. Missing artifacts are now explicitly never reconstructed, and the Step 6 report has a
  `## Sources` section confirming where content came from. Verifying your own writes with git
  is still allowed; reading git for content is not (#3).
- **Acceptance Scenarios are no longer dropped on archival.** Step 1 extracted user stories
  "with priorities and acceptance criteria", but 5.1 only told the agent to preserve priority
  ordering and never mentioned the scenarios, so they had no defined path into `spec.md` and
  were silently lost. This is the same failure mode fixed in 1.1.1 for other categories,
  surviving in the one category that was never listed in 5.1. The extraction wording now also
  matches the template's own `Acceptance Scenarios` heading (#3).
- A source item that carries no ID now has a defined citation form. The traceability rule
  assumed every source item had an ID, so against a feature spec with unnumbered edge cases an
  agent produced `-> Edge Cases`, naming a section rather than an item. The rule now falls back
  to a quoted heading or opening phrase, then to the file-level form, and rejects bare section
  names outright. Upgrading an older directory-level ref uses the same ladder, so an
  identifiable but unnumbered item no longer drops straight to the file-level form (#3).
- The `->` in a source ref is now defined. It means "came **from** this item in that file", and
  never "this source item became that ID", which is how it had been misread (#3).
- The allowlist entries name the steps that use each file descriptively rather than
  restrictively. An earlier draft scoped `.specify/templates/` to "seed templates only, for
  Step 0.4", which would have forbidden Step 5.3 from reading the agent-file template that
  lives in the same directory.
- The first token now resolves under `REPO_ROOT` rather than the current working directory.
  Invoked from a subdirectory, a valid `specs/###-feature-name` could previously be rejected
  as "does not resolve to exactly one feature directory" (#3).

### Added

- `argument-hint` frontmatter, so the expected argument shape is visible where the command is
  invoked rather than only in the rejection message. Spec-Kit preserves it into the generated
  Claude `SKILL.md` for extension commands.

### Changed

- Dropped `requires.scripts` from `extension.yml`. It is not part of the manifest schema and
  the validator ignores unknown keys under `requires`, so it never had any effect. The command
  already handles a missing `check-prerequisites.sh` itself in Step 0.1.
- README now states that `before_archive` / `after_archive` are extension-defined events, not
  core Spec-Kit ones.

## [1.1.1] - 2026-08-09

### Fixed

- First archival no longer produces a near-empty `.specify/memory/spec.md`. Bootstrapping in
  Step 0.4 ran before the extraction step and was told to "populate from the feature spec",
  while the first-run rule in 5.1 then skipped the merge steps that fill the file. Extracted
  content therefore had no defined path into `spec.md`. Bootstrapping now creates an empty
  seed and 5.1 populates it like any other run (#3).
- Unsupported invocations are rejected instead of being improvised. Ranges (`specs/001 thru
  specs/008`), globs, unrecognized flags, and first tokens matching zero or several
  directories now stop the command before any file is written. The command archives exactly
  one feature per run, which is now stated in both the command and the README (#3).
- `FEATURE_DIR` precedence is defined. Step 0.1 derived it from `check-prerequisites.sh` while
  Input Parsing derived it from the first argument, with no stated winner. The script resolves
  the last feature worked on rather than the one being archived, so the argument now wins, and
  a disagreement is reported. A non-zero exit from the script is also handled instead of being
  undefined (#3).

## [1.1.0] - 2026-08-04

### Added

- Capture `Assumptions` and `Measurable Outcomes` (SC-XXX) when archiving a feature
  spec into `.specify/memory/spec.md`. Previously these sections were dropped, which
  meant assumptions recorded in feature specs were lost on archival (#3).
- Item-level traceability refs (`[Source: specs/###-feature/spec.md -> FR-012]`), so a
  consolidated entry carries one ref per contributing feature (#3).
- Supersession pass: detects requirements a later feature wholly replaces, confirms removals
  with the user before deleting anything, removes the retired entry from `spec.md`, and
  records it as a `RETIRED:` line in `changelog.md`. Retired IDs are read back on later runs
  so they are never reissued. Removal is skipped entirely unless both `spec.md` and
  `changelog.md` are writable, so a deletion can never happen without an audit trail (#3).
- Unresolved contradictions are recorded in `changelog.md` when a supersession is not
  confirmed, and re-raised on the next archival rather than being reported once and lost (#3).

### Changed

- Main spec merging now **consolidates** into existing entries instead of appending
  per-feature extractions. Replaces the previous "prefer appending over restructuring"
  edit rule, which caused the main spec to accumulate near-duplicate items (#3).
  Only incoming feature items are folded into existing entries; two entries that both
  already exist in main memory are never merged, so an established ID cannot vanish.

## [1.0.0] - 2026-03-14

### Added

- Initial release of the Archive extension
- Command: `/speckit.archive.run` — post-merge archival of feature specs into project memory
- Merges user stories, functional requirements, entities, and architecture into `.specify/memory/spec.md`
- Updates dependencies, project structure, and routing in `.specify/memory/plan.md`
- Updates agent knowledge files (GEMINI.md / AGENTS.md / CLAUDE.md)
- Appends to `.specify/memory/changelog.md` with task completion counts
- Constitution compliance enforcement before merging
- Memory directory bootstrapping on first archival
- Feature spec status update (`Draft` → `Completed`)
- Scope modifiers (`--spec-only`, `--plan-only`, `--changelog-only`, `--agent-only`)
- Extension hook support (`before_archive`, `after_archive`)
- Archival Report with absolute paths and traceability tags
