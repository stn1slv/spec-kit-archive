# Changelog

All notable changes to the Archive extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
