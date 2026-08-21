# Spec-Kit Archive

A Spec-Kit extension to archive merged features into the main project memory.

## Overview

The `speckit.archive.run` command is a **Post-Merge Archival** tool designed to consolidate finalized feature specifications, plans, and technical debt into the project's canonical memory (`.specify/memory/`).

This extension acts as the "Outer Loop" of the Double-Loop Parity framework: it ensures that after a PR is merged, the project remembers it correctly.

## Features

- **Lifecycle Separation**: Operates purely on merging feature-level knowledge into project-level memory.
- **Layout tolerant**: Accepts every feature-directory layout spec-kit produces — sequential (`specs/007-invoice-settings`), timestamped (`specs/20260814-101500-invoice-settings`) and nested under a scope directory (`specs/billing/006-invoice-settings`). Nothing keys on a three-digit prefix, and a feature is always named by its full path relative to the repository root, so two features sharing a basename in different scopes are never confused. Passing a scope directory is refused rather than expanded into a batch run.
- **Ecosystem Consistency**: Uses the core Spec-Kit `check-prerequisites.sh` script to locate the repository root (handles monorepos and nested structures). The feature to archive always comes from the path you pass, never from the script's own feature state, which points at whatever you worked on last rather than what you are archiving.
- **Consolidation**: A detection pass keys every incoming item with a semantic slug, shortlists lookalike pairs against the existing entries, and issues an explicit fold / separate / contradiction verdict per pair — then folds exactly the fold verdicts, so the main spec stays a single consolidated specification instead of a per-feature digest. The report always states how many pairs were examined and folded, so "zero duplicates" means "examined and found distinct", not "did not look". Existing entries are never merged into each other, so an established requirement ID cannot disappear behind your back.
- **Traceability**: Adds item-level `[Source: specs/007-invoice/spec.md -> FR-012]` refs and revision notes in the main memory artifacts. A ref names the artifact the content actually came from (`spec.md`, `plan.md`, `data-model.md`), and an entry consolidated from several features carries one ref per feature.
- **Supersession**: Detects requirements a later feature wholly replaces and asks you to confirm before deleting anything. Confirmed removals are deleted from the main spec and recorded in `changelog.md`, and their IDs are never reissued. Anything you do not confirm stays put and is recorded as an unresolved contradiction, so the next archival raises it again instead of losing it.
- **Constitution compliance**: Checks each feature against the project constitution in three ways, because a MUST rule can fail in more than one. A **conflict** is feature content that contradicts a rule; you are asked, and an unresolved one withholds *that item only* while the rest of the feature archives normally. An **unmet obligation** is a rule requiring a statement the feature never makes ("every feature that stores user data MUST state its retention rule") — asked, but never a reason to withhold anything. An **action-requiring** rule ("all API routes MUST have automated tests") is reported as unverified and never flagged, because this command reads artifacts and cannot inspect a test run: a plan claiming coverage is a claim, not a verification. A feature's own `## Constitution Check` is read but never archived, and it cannot close a flag by asserting compliance — though a sentence in it recording what changed and why does satisfy a rule that asks for exactly that.
- **Bounded inputs**: Declares the complete list of files it may take content from. Git history, deleted files, ad-hoc notes and agent memory stores are not sources, and a missing artifact is never reconstructed. This is what keeps runs reproducible and keeps the `[Source: ...]` refs honest.
- **Bug awareness**: Works with both bug-report layouts — the feature-scoped `specs/###-feature/bugs/BUG-###.md` that extensions such as `spec-kit-bugfix` write, and the repo-level `.specify/bugs/<slug>/` that the first-party `bug` extension writes — without taking requirement text from either. Patched amendments arrive through the feature's own artifacts, where struck-through text is never archived as live. Each report is audited by status (treated as a claim, not a verification), addressed bug identifiers land in the changelog entry, and root-cause analyses feed the agent file's Known Issues. A repo-level report counts as this feature's only when an annotation names it or its own header names the feature; resemblance, dates and touched files never attribute one, and the report states how many were found against how many were attributed.
- **Agent context aware**: Current spec-kit does not manage agent context files itself; the opt-in `agent-context` extension does. This command reads that extension's config to find the files it manages, honours a custom marker pair, and updates **every** anchor a project keeps in sync rather than guessing a single filename. It never writes inside a tool-managed marker block, never creates a context file, and skips and reports any target that is missing or has no writable region. Without that config it falls back to probing `GEMINI.md`, `AGENTS.md`, `CLAUDE.md`.
- **Reporting**: Mandates absolute paths in the final Archival Report, ensuring logs are always useful regardless of your CWD.

## Hooks

The command checks `.specify/extensions.yml` for `before_archive` and `after_archive` hooks. **These are extension-defined events, not core Spec-Kit ones.** Core fires `before_`/`after_` hooks for its own commands (`specify`, `plan`, `tasks`, `implement`, and so on); archival runs after the cycle, so this command reads and reports the hooks itself. Another extension can register on them, but they only fire when `speckit.archive.run` is invoked.

## Requirements

Spec-Kit **0.14.0 or later**. That floor is set by the last core feature this extension depends on: `scripts/python` shipping in the wheel, so a project initialised with `--script py` has a working script path. The other dependencies land earlier — validated `category`/`effect` manifest fields in 0.10.2, the `agent-context` extension owning context files in 0.12.0, and `py:` command frontmatter in 0.12.10. There is no upper bound: every lookup this command added degrades to previous behaviour when its input is absent.

## Installation

You can install this extension via the Spec-Kit CLI:

```bash
specify extension add archive --from https://github.com/stn1slv/spec-kit-archive/archive/refs/tags/v1.3.0.zip
```
*(Note: Replace `v1.3.0` with the latest release version)*

To upgrade an existing installation, add `--force` — without it the CLI refuses to overwrite the installed version:

```bash
specify extension add archive --from https://github.com/stn1slv/spec-kit-archive/archive/refs/tags/v1.3.0.zip --force
```

## Usage

```bash
/speckit.archive.run <feature-dir>
```

**One feature per run.** There is no batch or range mode: `specs/001 thru specs/008` and `specs/00*` are rejected. Archive several features by running the command once per feature, in ascending order, so each run builds on the previous one.

You can optionally restrict the scope of the updates:
- `--spec-only` — update only `.specify/memory/spec.md`
- `--plan-only` — update only `.specify/memory/plan.md`
- `--changelog-only` — update only `.specify/memory/changelog.md`
- `--agent-only` — update only the agent context file(s)

Free-form text after the feature path is **guidance**, like in the core spec-kit commands:

```bash
/speckit.archive.run specs/007-invoice Pay extra attention to the entity model.
```

Guidance steers attention, emphasis, and report detail. It cannot add content sources, skip steps, change scope or IDs, or approve removals, and the report echoes it verbatim so every run stays auditable. Do not put feature paths (in any form) or leading bare feature references into guidance — those are rejected as a second feature. A feature path is recognised by its shape, so **any** `specs/...` token in guidance is rejected, not only a numbered one; name a path in prose without the `specs/` prefix when guidance must mention it. Ordinary prose, punctuation, and numbers inside sentences are fine.

## Workflow

1.  **Resolve paths**: run `check-prerequisites.sh` for the repository root, then take the feature directory from the path you passed.
2.  **Verify Constitution Compliance**: Check that feature implementations don't violate project "MUSTs".
3.  **Perform Impact Map**: Ask up to 5 clarifying questions before proceeding, including confirmation of any superseded requirements.
4.  **Archive Data**: Consolidate entities, requirements, dependencies, and architecture notes into the main memory, and apply confirmed supersessions.
5.  **Output Report**: Provide a comprehensive status report indicating changed files and what you should do next.
