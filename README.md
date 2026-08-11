# Spec-Kit Archive

A Spec-Kit extension to archive merged features into the main project memory.

## Overview

The `speckit.archive.run` command is a **Post-Merge Archival** tool designed to consolidate finalized feature specifications, plans, and technical debt into the project's canonical memory (`.specify/memory/`).

This extension acts as the "Outer Loop" of the Double-Loop Parity framework: it ensures that after a PR is merged, the project remembers it correctly.

## Features

- **Lifecycle Separation**: Operates purely on merging feature-level knowledge into project-level memory.
- **Ecosystem Consistency**: Uses the core Spec-Kit `check-prerequisites.sh` script to locate the repository root (handles monorepos and nested structures). The feature to archive always comes from the path you pass, never from the script's own feature state, which points at whatever you worked on last rather than what you are archiving.
- **Consolidation**: A detection pass keys every incoming item with a semantic slug, shortlists lookalike pairs against the existing entries, and issues an explicit fold / separate / contradiction verdict per pair — then folds exactly the fold verdicts, so the main spec stays a single consolidated specification instead of a per-feature digest. The report always states how many pairs were examined and folded, so "zero duplicates" means "examined and found distinct", not "did not look". Existing entries are never merged into each other, so an established requirement ID cannot disappear behind your back.
- **Traceability**: Adds item-level `[Source: specs/###-feature-name/spec.md -> FR-012]` refs and revision notes in the main memory artifacts. A ref names the artifact the content actually came from (`spec.md`, `plan.md`, `data-model.md`), and an entry consolidated from several features carries one ref per feature.
- **Supersession**: Detects requirements a later feature wholly replaces and asks you to confirm before deleting anything. Confirmed removals are deleted from the main spec and recorded in `changelog.md`, and their IDs are never reissued. Anything you do not confirm stays put and is recorded as an unresolved contradiction, so the next archival raises it again instead of losing it.
- **Bounded inputs**: Declares the complete list of files it may take content from. Git history, deleted files, ad-hoc notes and agent memory stores are not sources, and a missing artifact is never reconstructed. This is what keeps runs reproducible and keeps the `[Source: ...]` refs honest.
- **Bug awareness**: Works with bugfix extensions (such as `spec-kit-bugfix`) without taking requirement text from bug reports — patched amendments arrive through the feature's own artifacts, where struck-through text is never archived as live. Each `bugs/` report is audited by status (treated as a claim, not a verification), addressed bug IDs land in the changelog entry, and root-cause analyses feed the agent file's Known Issues.
- **Reporting**: Mandates absolute paths in the final Archival Report, ensuring logs are always useful regardless of your CWD.

## Hooks

The command checks `.specify/extensions.yml` for `before_archive` and `after_archive` hooks. **These are extension-defined events, not core Spec-Kit ones.** Core fires `before_`/`after_` hooks for its own commands (`specify`, `plan`, `tasks`, `implement`, and so on); archival runs after the cycle, so this command reads and reports the hooks itself. Another extension can register on them, but they only fire when `speckit.archive.run` is invoked.

## Installation

You can install this extension via the Spec-Kit CLI:

```bash
specify extension add archive --from https://github.com/stn1slv/spec-kit-archive/archive/refs/tags/v1.2.2.zip
```
*(Note: Replace `v1.2.2` with the latest release version)*

To upgrade an existing installation, add `--force` — without it the CLI refuses to overwrite the installed version:

```bash
specify extension add archive --from https://github.com/stn1slv/spec-kit-archive/archive/refs/tags/v1.2.2.zip --force
```

## Usage

```bash
/speckit.archive.run specs/###-feature-name
```

**One feature per run.** There is no batch or range mode: `specs/001 thru specs/008` and `specs/00*` are rejected. Archive several features by running the command once per feature, in ascending order, so each run builds on the previous one.

You can optionally restrict the scope of the updates:
- `--spec-only` — update only `.specify/memory/spec.md`
- `--plan-only` — update only `.specify/memory/plan.md`
- `--changelog-only` — update only `.specify/memory/changelog.md`
- `--agent-only` — update only the agent knowledge file

Free-form text after the feature path is **guidance**, like in the core spec-kit commands:

```bash
/speckit.archive.run specs/007-invoice Pay extra attention to the entity model.
```

Guidance steers attention, emphasis, and report detail. It cannot add content sources, skip steps, change scope or IDs, or approve removals, and the report echoes it verbatim so every run stays auditable. Do not put feature paths (in any form) or leading bare feature numbers into guidance — those are rejected as a second feature. Ordinary prose, punctuation, and numbers inside sentences are fine.

## Workflow

1.  **Resolve paths**: run `check-prerequisites.sh` for the repository root, then take the feature directory from the path you passed.
2.  **Verify Constitution Compliance**: Check that feature implementations don't violate project "MUSTs".
3.  **Perform Impact Map**: Ask up to 5 clarifying questions before proceeding, including confirmation of any superseded requirements.
4.  **Archive Data**: Consolidate entities, requirements, dependencies, and architecture notes into the main memory, and apply confirmed supersessions.
5.  **Output Report**: Provide a comprehensive status report indicating changed files and what you should do next.
