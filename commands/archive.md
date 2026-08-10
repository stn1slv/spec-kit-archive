---
description: "Archive a feature specification into main project memory after merge, resolving gaps and conflicts"
argument-hint: "specs/###-feature-name [--spec-only|--plan-only|--changelog-only|--agent-only]"
scripts:
  sh: ../../scripts/bash/check-prerequisites.sh --json --paths-only
  ps: ../../scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---
Act as the **Chief Software Architect** and **Documentation Maintainer**.
A feature has been merged into the `main` branch. Your goal is to **archive** the feature specification into the main project memory — ensuring completeness, resolving conflicts, closing gaps, and respecting the project constitution.

**Completeness means nothing is lost from the feature's own artifacts.** It does **not** mean filling gaps from elsewhere. See **Allowed Sources** below, which bounds every step of this command.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

### Input Parsing

**This command archives exactly one feature per run.** There is no batch or range mode. To archive several features, run the command once per feature in ascending feature order, so each run sees the result of the one before it.

Parse `$ARGUMENTS` as follows:
- **First token**: feature spec directory path (e.g., `specs/007-invoice-settings`)
- **Remaining tokens**: scope modifiers (optional, space-separated)

**Supported scope modifiers** (if none provided, update all artifacts):
- `--spec-only` — update only `.specify/memory/spec.md`
- `--plan-only` — update only `.specify/memory/plan.md`
- `--changelog-only` — update only `.specify/memory/changelog.md`
- `--agent-only` — update only the agent knowledge file (GEMINI.md / AGENTS.md / CLAUDE.md)

If **several** scope modifiers are supplied, the scope is their **union** — `--spec-only --changelog-only` updates both `spec.md` and `changelog.md` and nothing else. "Only" bounds the whole set, not each flag individually.

**Reject anything else.** The first three checks are textual and run before Step 0; the fourth needs `REPO_ROOT` and so runs as soon as 0.1 has resolved it, still ahead of every write. **No file is written when any of them fails** — a rejected invocation must leave the repository exactly as it found it. Do not guess at the intent of input you cannot parse.

1. **Empty input, or no feature at all.** If `$ARGUMENTS` is empty, **or the first token starts with `--`** (the feature path must come first, before any modifier), output `ERROR: No feature spec directory provided. Usage: /speckit.archive.run specs/###-feature-name [--scope-modifier]` and stop.
2. **More than one feature.** This check comes **before** the unrecognized-token check, so a range or a second path gets the guidance below rather than a generic parse error. Reject when the input covers more than one feature:
   - two or more tokens that reference a feature — a path containing `/`, or a bare feature number or name such as `007` or `007-invoice-settings`
   - a glob character (`*` or `?`) in any token
   - a **word** range marker — `thru`, `through`, or `to` — appearing as a whole token between two feature references (`specs/001 thru specs/008`, `specs/001 thru 008`)
   - a `..` **separating two feature references inside a single token** (`specs/001..specs/008`, `001..008`)

   A word marker only counts as a whole token, never as part of a directory name, so `specs/003-import-to-csv` and `specs/012-through-put` are legitimate single features. A `..` only counts when it sits between two feature references, so the leading `../` of a relative path such as `../specs/001-foo` is not a range. On a match, output:
   ```
   ERROR: This command archives one feature per run — no ranges or globs.
   Run it once per feature, in ascending order:
     /speckit.archive.run specs/001-first-feature
     /speckit.archive.run specs/002-second-feature
   ```
   and stop.
3. **Unrecognized token.** If any remaining token after the first is not one of the four modifiers above, output `ERROR: Unrecognized argument '[token]'. Supported: --spec-only, --plan-only, --changelog-only, --agent-only.` and stop.
4. **Ambiguous first token.** The first token must resolve to **exactly one** existing directory under `REPO_ROOT`. A numeric prefix such as `specs/001` may expand to `specs/001-name-of-feature` only when exactly one directory matches. If nothing matches, or more than one does, output `ERROR: '[token]' does not resolve to exactly one feature directory` — listing the matches when there are several — and stop.

---

## Allowed Sources (hard boundary)

Everything you write into main project memory must come from the files below. **This list is complete.**

- The artifacts inside `FEATURE_DIR`, **except the `bugs/` subdirectory** (see the exclusions below). 0.3 inventories these artifacts and Step 1 reads the main ones; `contracts/` also feeds the steps that ask about it (2.3's Integration gap row, 5.2's Routing), while `checklists/` and `quickstart.md` are inventoried and allowed should a step ask, though today none takes content from them
- The existing files in `.specify/memory/`, `constitution.md` among them (0.4, 0.5, Step 2, Step 5)
- `.specify/templates/` — any template a step calls for; today the seed templates in Step 0.4, plus an agent-file template where one exists (recent spec-kit versions ship none — Step 5.3 does not depend on it)
- `.specify/extensions.yml` (Steps 0.6 and 7.1)
- The agent knowledge file in `REPO_ROOT` (GEMINI.md / AGENTS.md / CLAUDE.md), for Step 5.3
- The output of `{SCRIPT}`

The step numbers above are **descriptive, not restrictive**. This list bounds *which files* you may take content from, never *which step* may read one. If a step needs a file on this list, it may read it.

**Take content from nowhere else.** Not from git history, `git log`, `git show`, stashes, other branches, or any file that was deleted or renamed. Not from ad-hoc notes files. Not from an agent memory or session store. Not from another feature's spec directory: other features reach main memory only by being archived in their own run.

**Two named exclusions inside otherwise-allowed locations.** First, a `bugs/` subdirectory of `FEATURE_DIR` (bugfix extensions write one) is **not a content source**: never merge a bug report or its amendments into main memory, and never let one alter the text of an item being archived — in test runs this produced requirements silently rewritten from bug files, with the outcome depending on which agent ran the command. Note the directory's presence under `## Outstanding Items` in the Step 6 report instead. Second, the feature spec's `## Clarifications` section (the Q&A session log `/speckit.clarify` maintains) is **deliberately not archived**: clarify already integrates every accepted answer into the sections this command does archive, so its decisions arrive through them. Do not copy the log, and do not carry an empty `## Clarifications` heading into main memory.

*One narrow exception:* when the **Legacy refs** edit rule asks you to upgrade an existing `[Source: specs/###-feature-name]` ref, you may open that feature's **corresponding artifacts** — `spec.md` for an entry in the main spec (plus `data-model.md` for an entity, which is often defined there), `plan.md` for an entry in the main plan — **solely to identify which item the ref points at**, and may copy that item's ID, or its heading or opening phrase, into the ref itself. Take nothing else from those files, and never into the entry's own text. If the item cannot be identified in any of them, **leave the directory-level ref as it is** and note it in the Step 6 report: a coarse ref that is true beats a file-level ref that guesses.

**Never recover a missing artifact's previous content.** This forbids *recovering old content*, not *creating files*: Step 0.4 creating an **empty seed** for a missing memory artifact is required and unaffected. If a file above is absent, treat it as absent — Step 0.2 stops when a required feature file is missing, and a missing memory artifact counts as empty. What you must not do is go looking for that file's earlier contents in git history or a backup and continue from them. That turns a first archival into something neither you nor the user can reproduce.

**Why this is strict.** An item-level `[Source: specs/###-feature/spec.md -> FR-012]` ref asserts that an entry came from a specific item in a specific feature spec. Content pulled from anywhere else still gets a ref, so the ref becomes false. This boundary is what makes the traceability mean anything.

**This bounds content, not tooling.** Running `git status` or `git diff --check` to verify what you just wrote is fine. Reading git to *obtain* requirements, plans, or prior memory to archive is not.

Report compliance under `## Sources` in Step 6.

---

## Step 0: Setup & Validation (Gate)

### 0.1 Resolve Paths

Resolve paths **in this order** — each step depends on the one before it, so do not reorder them.

**1. `REPO_ROOT`.** Run `{SCRIPT}` and take `REPO_ROOT` from its output.

- **If `{SCRIPT}` is missing**, stop and inform the user. The script ships with Spec-Kit, so its absence means this is not an initialized Spec-Kit project and nothing else in this command can be relied on.
- **If `{SCRIPT}` runs but exits non-zero** — commonly `Feature directory not found` on a clean `main` checkout with no `.specify/feature.json` — this is **not** fatal. Its feature directory is not used anyway (see step 2). Recover `REPO_ROOT` by resolving the first token of `$ARGUMENTS` against the **current working directory** and walking up to the nearest ancestor containing `.specify/`. Note the fallback in the Step 6 report. Stop only if no such ancestor exists.

**2. `FEATURE_DIR` — the argument always wins.** Resolve the first token of `$ARGUMENTS` **under `REPO_ROOT`**, not under the current working directory, even when step 1's fallback started from cwd — the walk-up has already established `REPO_ROOT` by then, and a run invoked from a subdirectory would otherwise reject a perfectly valid `specs/001-x`. Apply the **ambiguous first token** check from Input Parsing at this point: it must match exactly one existing directory, and a numeric prefix such as `specs/001` may expand only when the match is unique. That directory is `FEATURE_DIR`.

Ignore whatever feature directory `{SCRIPT}` reports. The script resolves it from the project's own state (`SPECIFY_FEATURE_DIRECTORY`, then `.specify/feature.json`), which is whichever feature was last worked on, **not** the one being archived; archival runs after a merge, so the two routinely differ. When they differ, report both in Step 6 so a user who passed the wrong path can see it.

**3. Remaining paths.**
- `MEMORY_DIR` (`REPO_ROOT / .specify/memory`)
- `TEMPLATES_DIR` (`REPO_ROOT / .specify/templates`)

**Path convention**: Feature specs live in `specs/{###-feature-name}/` at repo root. Use absolute paths for all file operations.

### 0.2 Validate Feature Directory

Verify `FEATURE_DIR` exists and contains:
- `spec.md` (required)
- `plan.md` (required)

If any required file is missing:
> ⚠️ Invalid feature spec: Missing required files in `FEATURE_DIR`. Expected:
> - spec.md
> - plan.md
>
> Run `/speckit.specify` and `/speckit.plan` first.

**Then stop. Do not modify any files.**

### 0.3 Inventory Optional Artifacts

Note which of these exist in `FEATURE_DIR` (for use in later steps):
- `tasks.md` — archival and task counting
- `research.md` — knowledge capture, known issues & gotchas
- `data-model.md` — entity merging
- `contracts/` — API documentation (non-empty directory)
- `checklists/` — quality tracking
- `quickstart.md` — integration scenarios

A `bugs/` directory may also exist (bugfix extensions write one). Note whether it does, but it is **not** an archival artifact: no step takes content from it (see Allowed Sources), and its presence is only reported under `## Outstanding Items` in Step 6.

### 0.4 Validate or Bootstrap Memory Directory

Check if `MEMORY_DIR` exists:

**If `MEMORY_DIR` exists**: Read its contents. Note which files are present (`constitution.md`, `spec.md`, `plan.md`, `changelog.md`), **and for `spec.md` and `plan.md` whether each is empty or populated**. Step 2 and the 5.1 ID rules key on that fact, so record it once here rather than re-deriving it.

**What "empty" means for a memory artifact** — here, and wherever this command asks whether one is empty: it carries **no content entries**, meaning no requirements, stories, entities, edge cases, outcomes, assumptions, dependencies or modules. A Step 0.4 seed is **always** empty in this sense, even though it has section headings and is not a zero-byte file. Headings, template boilerplate and revision notes do not count as content, and a missing file counts as empty. This says nothing about other uses of the word, such as an empty `$ARGUMENTS`.

**If `MEMORY_DIR` does not exist**: Create it:
```
mkdir -p MEMORY_DIR
```

**Seed only what this run will populate.** Bootstrap `spec.md` only when 5.1 is in scope, and `plan.md` only when 5.2 is in scope. Seeding a file the run then skips is worse than not seeding it: the empty file exists, so no later run bootstraps it, no later run recognises it as unfilled, and this feature's content is lost for good.

When a scope modifier suppresses a bootstrap, skip it silently and record it under `## Scoping` in the Step 6 report, using this wording (substituting the artifact and feature):

> `plan.md` was not seeded because it is out of scope for this run. To get **this** feature's plan content into it, re-run `/speckit.archive.run specs/###-feature-name` at full scope. A full-scope run for a *different* feature will create the file but fill it with that feature's content, not this one's.

**Memory artifacts are project-level documents.** They outlive every feature, so they must never open with one feature's metadata. When seeding, **drop the entire per-feature header block**: the title line (`# Feature Specification: ...` / `# Implementation Plan: ...`) and **every** metadata line before the first section heading, whatever it is called — `**Feature Branch**`, `**Created**`, `**Status**`, `**Branch**`, `**Date**`, `**Input**`, and any other bold metadata line are examples, not the complete set. Title the seeds `# Main Project Specification` and `# Main Implementation Plan`. When mirroring the feature's own files instead (no template), the same rule applies, and the feature spec's `## Clarifications` heading is also left out (see Allowed Sources). These rules govern **new seeds only**: a memory file written by an older version keeps whatever header or archived sections it has — do not rewrite it retroactively; the Edit Rules' "preserve existing layout" applies.

**If `MEMORY_DIR/spec.md` does not exist and `spec.md` is in scope**:
- If `TEMPLATES_DIR/spec-template.md` exists, copy it as the seed and **leave its sections empty**, removing template placeholder text and the per-feature header block per the rule above
- Otherwise, create `spec.md` containing the section headings the feature spec uses, all empty
- **Do not populate it here** — Step 1 has not run yet, so nothing has been extracted; 5.1 fills it
- Note in the report: "Bootstrapped empty `.specify/memory/spec.md`; populated by 5.1"

**If `MEMORY_DIR/plan.md` does not exist and `plan.md` is in scope**:
- If `TEMPLATES_DIR/plan-template.md` exists, copy it as the seed and **leave its sections empty**, removing template placeholder text and the per-feature header block per the rule above
- Otherwise, create `plan.md` containing the section headings the feature plan uses, all empty
- **Do not populate it here** — 5.2 fills it
- Note in the report: "Bootstrapped empty `.specify/memory/plan.md`; populated by 5.2"

### 0.5 Load Constitution (Guardrails)

Read `MEMORY_DIR/constitution.md` if it exists. Extract:
- Core Principles (numbered roman numerals or named sections)
- Architecture Standards
- Quality Gates

**Constitution is non-negotiable.** Any feature content that conflicts with a constitution MUST principle is flagged as CRITICAL and must be resolved before merging. Do not silently override or reinterpret constitution rules.

### 0.6 Check Extension Hooks (before archival)

Check if `REPO_ROOT/.specify/extensions.yml` exists:
- If it exists, read it and look for entries under `hooks.before_archive`
- If the YAML cannot be parsed or is invalid, skip hook checking silently
- Filter to only hooks where `enabled: true`
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook
- For each executable hook, output based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks
    **Optional Pre-Hook**: {extension}
    Command: `/{command}` — {description}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks
    **Automatic Pre-Hook**: {extension}
    EXECUTE_COMMAND: /{command}
    Wait for the result before proceeding.
    ```
- If no hooks are registered or the file does not exist, skip silently

---

## Step 1: Feature Analysis

Read the feature specification and extract the following. These files, in `FEATURE_DIR`, are the **only** source of feature content (see **Allowed Sources**). If something you expect is not in them, it is not available: record the gap in Step 2.3 rather than looking for it elsewhere.

**From spec.md:**
- User Stories / Integration Scenarios — each story's **entire block**: its priority, its description, and **every labelled field it carries** (the template's names are `Why this priority`, `Independent Test`, `Acceptance Scenarios`; a spec may use other labels, or omit some). Extract the block whole, never an enumerated subset of fields — every category this command has ever silently lost (assumptions, acceptance scenarios, priority rationale) was lost because it was not on someone's list
- Do **not** extract the `## Clarifications` session log — its decisions already live in the sections above (see Allowed Sources)
- Functional Requirements (detect the project's ID convention — e.g., FR-XXX, REQ-XXX, or unnumbered)
- Non-Functional Requirements (if any)
- Key Entities and their fields
- Edge cases and error handling
- Success Criteria / Measurable Outcomes (detect the ID convention, e.g., SC-XXX)
- Assumptions (target users, scope boundaries, data/environment)

**From plan.md:**
- New dependencies introduced (with versions)
- New modules/services created
- Architecture changes (project structure, routing)
- Configuration changes (env vars, properties)
- Branch name (from metadata)

**From data-model.md (if exists):**
- New models and their definitions
- Relationships between entities
- Validation rules

**From research.md (if exists):**
- Key technical decisions and trade-offs
- External API integrations
- Known issues and gotchas (for agent file merging)

**From tasks.md (if exists):**
- Count completed tasks: lines matching `- [X]` or `- [x]`
- Count total tasks: lines matching `- [ ]` or `- [X]` or `- [x]`

---

## Step 2: Conflict Detection & Gap Analysis

Before merging, systematically check for issues.

**Empty comparison target (applies to 2.2, 2.3, and 2.4).** A check that compares this feature against a main-memory artifact means nothing when that artifact is empty (defined in Step 0.4): there is no prior content to collide with, nothing that could be superseded, and every item is trivially "missing". **Skip each check whose comparison target is empty.**

Judge the two artifacts **separately**, because a run can have one populated and the other not:
- Spec-side — 2.2 requirement ID collisions and entity redefinitions, the Requirements and Data Model rows of 2.3, and the whole of 2.4 — keys on `.specify/memory/spec.md`.
- Plan-side — 2.2 dependency conflicts, and the Architecture, Integration and Testing rows of 2.3 — keys on `.specify/memory/plan.md`.

2.1 always runs: the constitution is independent of both.

### 2.1 Constitution Compliance (CRITICAL)

For each extracted requirement, user story, and architecture decision, verify it does not conflict with any constitution MUST principle or Architecture Standard.

**If a constitution conflict exists**, flag it as CRITICAL:
```
🔴 CONSTITUTION CONFLICT:
- Feature FR-XXX: "[requirement]" conflicts with Principle [N]: "[principle text]"
  → This MUST be resolved before archival can proceed.
```

### 2.2 Conflicts

1. **Requirement ID Collisions:** If the feature has an ID that already exists in main spec, flag it.
2. **Entity Redefinitions:** If an entity is being modified (not just added), highlight the delta.
3. **Dependency Conflicts:** If a new dependency version conflicts with existing ones, note it.

### 2.3 Gaps

Categorize discrepancies between the feature spec and main memory:

| Category | What to look for |
|----------|-----------------|
| **Requirements** | Missing IDs, unmatched acceptance criteria |
| **Architecture** | Undocumented modules, missing routing/wiring |
| **Integration** | New contracts not reflected in main plan |
| **Data Model** | Entity changes without migration notes |
| **Testing** | New components without test strategy |

**If conflicts or significant gaps exist**, list them:
```
⚠️ ISSUES DETECTED:
- FR-005: Main says "X", Feature says "Y" → Recommend: [resolution]
- Entity `User`: Added field `role` → Verify backward compatibility
- Gap: New `/api/settings` route not in main plan routing section
```

### 2.4 Supersession Candidates

**Skip this step** if `.specify/memory/spec.md` is empty (see the Step 2 preamble) or if `spec.md` is not in scope.

Otherwise, identify entries in `.specify/memory/spec.md` that this feature **wholly replaces**. Look for:

- The same capability restated with different or incompatible behavior.
- An explicit statement in the feature spec that it replaces, deprecates, or removes prior behavior.
- A rule that narrows or widens an existing one such that both cannot hold at once.

Two rules bound what counts:

- **Whole entries only.** If only *part* of an existing entry is obsolete, it is **not** a supersession candidate. Report the contradiction and leave the entry untouched. Partial rewrites are the user's call, not this command's.
- **Overlap is not supersession.** If both entries can hold at once, this is ordinary consolidation (5.1), not a supersession.

Also read the `## Unresolved Contradictions` section of `.specify/memory/changelog.md`, if that file exists, and re-raise each pair listed there as a candidate while both entries are still present and still contradictory. That is how a contradiction the user declined on an earlier run gets another chance to be resolved. Present a re-raised pair as `FR-012 (main) vs FR-023 (main)`, since both sides already carry main-memory IDs.

Report each candidate with the evidence quoted:
```
🔄 SUPERSESSION CANDIDATES:
- FR-005 (main) ← superseded by FR-021 (feature)
  Main:    "[quote the existing requirement]"
  Feature: "[quote the replacing requirement]"
  Reason:  [why the new one replaces rather than complements the old one]

- FR-008 (main) ← removed, no replacement
  Main:    "[quote the existing requirement]"
  Feature: "[quote the statement that removes this behavior]"
  Reason:  [why the behavior is being retired outright]
```

**This step is detection only — never remove anything here.** Every candidate must be confirmed by the user in Step 3 before 5.1 applies it.

---

## Step 3: Clarify (exactly once; max 5 questions)

If conflicts or gaps require human judgment, ask **only questions that materially change scope or correctness**. Skip this step entirely if everything is unambiguous.

**Always ask** if any CRITICAL constitution conflicts were detected — these cannot be auto-resolved.

**Always ask** if any supersession candidates were detected in Step 2.4, **provided the supersession gate is open** (defined once below). Removal is destructive and requires explicit confirmation. Ask this question first if the budget is tight, and bundle constitution conflicts into a single combined question if needed.

**The supersession gate.** Supersession requires **both** `.specify/memory/spec.md` (where the entry is removed from) and `.specify/memory/changelog.md` (where the audit line goes) to be **writable under the current scope**. Compute this from the scope modifiers actually supplied rather than assuming any particular one; with no modifiers, everything is in scope and the gate is open.

When the gate is closed: do not ask the question, remove nothing, write no `RETIRED:` lines, and report the candidates as deferred, naming the scope that closed the gate. This gate governs the whole supersession flow — Step 3 and 5.1 alike.

**A closed gate also blocks the contradiction record.** The `## Unresolved Contradictions` list lives in `changelog.md`, so when that file is out of scope the deferred candidates cannot be written down anywhere durable. Meanwhile 5.1 still adds the feature's conflicting item as a new entry, so the main spec ends the run holding both sides of a contradiction that nothing will re-raise. This is the one case where a scope modifier leaves the spec in a worse state than a full run.

Do not paper over it. Report those candidates under a distinct **"deferred and unrecorded"** heading in Step 6, state plainly that they will **not** be raised again automatically, and recommend re-running the command at full scope to resolve them. If the run can be made at full scope instead, that is always the better option.

Use this format and **wait for answers**:

```markdown
## Question [N]: [Topic]
**Context**: [Quote the relevant spec/plan/constitution section]
**Decision Needed**: [1 sentence]
**Suggested Answers**:
| Option | Answer | Implications |
|--------|--------|--------------|
| A | [Option A] | [Impact] |
| B | [Option B] | [Impact] |
| C | [Option C] | [Impact] |
| Custom | Provide your own | [How it affects scope] |

**Your choice**: _[Wait for user response]_
```

**For supersession candidates**, ask **one** question covering all of them rather than one question per candidate, which would exhaust the question budget:

```markdown
## Question [N]: Confirm supersessions
**Context**: [List each candidate as `OLD-ID ← NEW-ID`, quoting both entries]
**Decision Needed**: Which of these should be removed from `.specify/memory/spec.md`?
**Suggested Answers**:
| Option | Answer | Implications |
|--------|--------|--------------|
| A | Remove all listed | Each entry is deleted, its ID retired, and one `RETIRED:` line written to changelog.md |
| B | Remove none | Main spec keeps both entries; each contradiction is recorded and re-raised next run |
| C | Remove only [IDs] | Confirm a subset; the rest are kept and recorded as unresolved |
| Custom | Provide your own | [How it affects which entries survive] |

**Your choice**: _[Wait for user response]_
```

Treat anything the user does not explicitly confirm as **not** superseded.

Every candidate the user does not confirm leaves two conflicting entries in the main spec. Record each one in the top-level `## Unresolved Contradictions` section of `changelog.md` (see 5.4) as well as in the Step 6 report, so 2.4 re-raises it on the next run instead of it becoming invisible.

**Rules:**
- Max 5 questions total.
- Max 3 unresolved `NEEDS CLARIFICATION` markers in output — beyond that, make reasonable defaults and note them in the report.
- If no questions are needed, proceed directly to Step 4.

---

## Step 4: Impact Mapping

Before making any edits, produce a brief impact map:

```markdown
### Impact Map
| Artifact | Sections Affected | Change Type |
|----------|------------------|-------------|
| `.specify/memory/spec.md` | User Stories, FR-012–FR-015, Entities | Consolidate + Add |
| `.specify/memory/spec.md` | FR-005 | Remove (superseded by feature FR-021) |
| `.specify/memory/plan.md` | Dependencies, Project Structure | Append |
| `.specify/memory/changelog.md` | Merged Features Log | New entry |
| `GEMINI.md` | Recent Changes, Known Issues | Append |
```

This gives the user a preview before edits are applied. Include every confirmed supersession target as a `Remove` row.

---

## Step 5: Archival (Apply Edits)

### Edit Rules
- Use absolute paths for all file references.
- Preserve existing section layout and ordering. Consolidate *within* a section; do not reorganize the document.
- **Consolidate, do not accumulate.** Merge each incoming item into the existing entry that already covers the same ground. Append a new entry only when no equivalent exists. The main spec is one consolidated specification, not a per-feature digest.
- **Only ever fold an incoming feature item into an existing entry.** Never merge two entries that both already exist in main memory. Accumulation came from appending incoming items, so this is enough to fix it, and it guarantees an existing main-memory ID can never disappear through consolidation.
- **The surviving text of a merge must preserve every constraint** from all contributing entries. If one entry's wording would lose a condition, limit, or qualifier stated by the other, the two are **not** equivalent — keep them separate. A source ref must never point at an entry whose constraint was dropped.
- Add an **item-level** `[Source: specs/###-feature-name/<file> -> ID]` traceability ref to each merged entry, where `<file>` is the feature artifact the content actually came from: `spec.md` for spec items (e.g. `[Source: specs/007-invoice/spec.md -> FR-012]`), `plan.md` for plan-derived entries, `data-model.md` for entities it defines. A ref must never name `spec.md` for content that came from another artifact — that would assert a provenance that is not true. An entry consolidated from several features carries one ref per contributing feature; an entry drawing on two artifacts of the same feature (say `spec.md` and `data-model.md`) may carry one ref per artifact. Never attach a duplicate ref to a source the entry already cites.
  - **How to read the arrow.** `file -> ID` means "this entry came **from** the item `ID`, which lives in that file". It points from a file to an item **inside** it. It never means "the source item became this ID": a source `User Story 1` folded into main memory's `User Story 6` is still cited as `-> User Story 1`, because the ref names where the content came from, not where it landed.
  - **When the source item carries no ID.** A feature spec may number its requirements but leave edge cases or stories unnumbered, so there is no ID to cite. Quote the item's own heading or opening phrase instead: `[Source: specs/002-billing/spec.md -> "Card declined mid-checkout"]`. A bare section name such as `-> Edge Cases` is **not** acceptable — it names a section, not an item, so it identifies nothing. If no stable phrase exists either, use the file-level form `[Source: specs/002-billing/spec.md]` — safe here because for a **new** ref you know exactly which artifact the content came from. (Legacy-ref upgrades have no such knowledge, which is why their unidentifiable case keeps the directory-level ref instead.)
- **Legacy refs**: entries written in the older directory-level form (`[Source: specs/###-feature-name]`) carry no item ID. When you touch such an entry, upgrade the ref using **the same ladder as above** — the originating item's ID, else its heading or opening phrase in quotes, citing the artifact where you found it (the Allowed Sources exception names which artifacts you may search). When the item cannot be identified in any searchable artifact, **keep the directory-level ref unchanged** and note it under `## Outstanding Items` in the Step 6 report — never upgrade it to a file-level form naming an artifact the item was not found in, because the provenance rule above applies to upgraded refs too. An identifiable but unnumbered item takes the middle rung. Do not modify legacy refs on entries this feature does not touch.
- Add a **Revision note** (date + reason) to each modified artifact.
- Respect scoping hints — skip artifacts not in scope and explicitly note them. **Out of scope means not written, never not read**: artifacts outside the scope are still read when a rule requires it (for example collecting retired IDs or checking for a prior run in `changelog.md`). A rule that reads a missing artifact treats it as empty rather than stopping, so no rule below needs its own existence check.
- **Idempotency is judged per artifact, not per run.** This feature has already been merged into an artifact if that artifact carries source refs naming it, **or an entry naming it** — the changelog's Merged Features Log entry, or the agent file's Recent Changes bullet, neither of which carries source refs. Check the artifact you are about to write, not `changelog.md` on its behalf, because scope modifiers mean a feature can be present in `spec.md` while no changelog entry exists. When it is already present, update in place: never append a second copy, and never attach a source ref an entry already cites.
- **Detect and follow the project's existing ID convention** (FR-XXX, REQ-XXX, Flow1, US-XX, etc.). Continue the sequence from the highest existing ID in main memory. Never reuse or renumber existing IDs. **When `.specify/memory/spec.md` is empty** there is no convention to detect and no highest ID: adopt the **feature spec's own** convention and carry its IDs across unchanged, so `FR-007` in the feature stays `FR-007` in main memory and its source ref matches. Do not renumber, and never inherit example IDs left behind by template placeholder text. Assumptions are the one exception: they take `AS-###` IDs assigned by 5.1 step 8 even on an empty spec, because feature specs usually leave them unnumbered and the main spec needs a stable key. Key this on the spec being empty, **not** on the run being a first archival: scope modifiers make runs possible where `plan.md` is being seeded while `spec.md` is already populated, and carrying the feature's IDs into a populated spec would duplicate existing ones.
  - **Retired IDs still win.** Carry the feature's IDs across unchanged **except** any that appear in the retired list (next rule). Renumber a colliding item above the highest ID in **both** the retired list and the feature's own carried IDs, per the next rule — renumbering above the retired list alone would land on an ID the feature already uses.
- **Retired IDs are off-limits.** Before assigning any new ID, read `.specify/memory/changelog.md` and collect the ID immediately following each `RETIRED:` marker. **Collect only that ID** — the rest of the line names the live replacement and must be ignored. Continue numbering above the highest ID found in **either** the main spec or that retired list, so a retired ID is never reissued even when it was the highest-numbered entry.
- **When consolidating equivalent items, keep the earliest existing ID** and attach the later features' source refs to it. Never renumber the surviving entry.
- **Constitution constraints must be respected** — do not merge content that violates them.

### 5.1 Update Main Specification (`.specify/memory/spec.md`)

Each step below **consolidates** into the existing section rather than appending a new per-feature block.

**Removals come first.** Step 1 applies the confirmed supersessions, before any merging. Nothing can then be folded into an entry that is about to be deleted.

**Empty seed.** If `spec.md` is empty, the numbered steps below run **normally** and populate the empty sections; there is simply nothing to fold into. Nothing extracted in **Step 1 (Feature Analysis)** may be left out — that is the whole-command Step 1, not step 1 of the list below.

1. **Apply confirmed supersessions** — see 5.1.1 below. This happens before everything else.
2. **Merge User Stories / Integration Scenarios** — fold into an existing story when it covers the same user goal; otherwise add, maintaining priority ordering. **Carry each story's entire block across with it**: description, `Why this priority`, `Independent Test`, `Acceptance Scenarios`, and any other labelled field the story carries. The **heading line is reconstructed, not copied**: its number comes from the ID rules, and its priority is part of the block like any other field — a **new** story's heading takes the incoming priority verbatim, while a **folded** story keeps the existing entry's number and priority; when the incoming story states a different priority, keep the existing one and record the disagreement under `## Outstanding Items` in the Step 6 report — the incoming `Why this priority` rationale is still carried per the whole-block rule, but **label it** with its source feature and the priority it argued for (e.g. `(specs/002-notifications argued P1)`), so the story text does not contradict its own heading. Never carry a subset of the fields — a field left off an enumerated list is exactly how Acceptance Scenarios and priority rationales were silently lost in earlier versions. When folding, merge the blocks field by field: combine list-valued fields (such as scenario lists) dropping only exact duplicates, and where a prose field genuinely differs, keep both statements inside the one story — that is how the Edit Rules' constraint-preservation requirement is met for stories: nothing is lost, and the story is not split in two. Never drop a story because a field is missing: if the feature spec states no Acceptance Scenarios for it, carry the story across anyway and name it under `## Outstanding Items` in the Step 6 report. Report it there rather than as a 2.3 gap: 2.3 ran back in Step 2, and on a first archival its Requirements row was skipped outright because the main spec was empty. Inventing missing fields is not an option; Allowed Sources forbids it.
3. **Merge Functional Requirements** — fold into the existing requirement when it states the same capability; otherwise add, continuing from the highest existing ID. Group by domain/module if the spec is large.
4. **Merge Key Entities** — add new entities; extend existing ones with new fields rather than restating the entity.
5. **Merge Edge Cases and Error Handling** — fold cases describing the same failure mode into one entry.
6. **Update Data Flow / Architecture** if the feature changed system data flows.
7. **Merge Success Criteria / Measurable Outcomes** if present. Fold outcomes measuring the same thing into one entry; otherwise continue from the highest existing ID (e.g., SC-XXX).
8. **Merge Assumptions**: add new assumptions under the `## Assumptions` section (if the main spec lacks one, create it after Success Criteria to match the template's section order); skip any whose content is already recorded in main memory. **Number them**: assumptions carry `AS-###` IDs in the main spec, so they can be cited, deduplicated, and superseded like requirements. Do this in two passes, in this order. *First, backfill:* if the section holds assumptions with **no ID at all** (archived by an older version), number those, in their current order, continuing above the highest `AS` ID found in **either** the section or the changelog's retired list — the **Retired IDs are off-limits** rule applies to the `AS` sequence like any other. An assumption that already carries an ID, under `AS` or any other project convention, keeps it; never renumber. Backfilling adds a label and changes nothing else — it is **not** "touching" the entry for the Legacy refs rule, so it never by itself triggers a ref upgrade. *Then, new entries:* each assumption added this run continues above the highest `AS` ID after the backfill, again skipping retired IDs. For the source ref: most feature specs leave assumptions unnumbered, so the ref takes the quoted-phrase rung of the ladder; if the feature spec numbers its assumptions, cite the feature's own ID, as the ladder's first rung requires.
9. **Close out the `RETIRED:` lines** opened in step 1. Every replacement ID is now settled, so fill in each line's replacement reference (see 5.1.1 step 3). Do not finish 5.1 with a `RETIRED:` line left incomplete.

**Do not fold an incoming item into an entry you flagged in 2.4 as contradicting it.** Add it as a new entry instead, so the contradiction stays visible for the user to resolve rather than being silently merged away.

#### 5.1.1 Apply Confirmed Supersessions

Apply only if the **supersession gate** (defined in Step 3) is open. If it is closed, remove nothing and report the candidates as deferred.

For each supersession candidate **confirmed by the user in Step 3**:

1. Remove the entry from `.specify/memory/spec.md`. Do not leave a placeholder, strikethrough, or `[Superseded by: ...]` note — the point is that no stale requirement text remains in the file agents load as context.
2. **Retire its ID.** It must never be reused or reassigned, even though its number is now unused.
3. **Open** a line in the feature's changelog entry, immediately, before moving to the next candidate:
   ```
   - RETIRED: FR-005 (from specs/003-billing/spec.md) → replaced by <pending>. Reason: [one line]
   ```
   The retired ID and reason are written **now**, so no entry is ever removed without a record existing. Only the replacement reference is left open, because it is not known yet.

   **Which ID the replacement reference takes.** Whichever main-memory ID the feature's replacing item ends up under once steps 2–8 finish — a **new** ID if it was added as a new entry, or the **existing** entry's ID if it folded into one (earliest ID wins, so that entry keeps its original number). The IDs quoted in Step 2.4 are the *feature's* local numbering and must never appear here — with one exception: a **re-raised pair** already carries main-memory IDs on both sides (2.4 presents it as `FR-012 (main) vs FR-023 (main)`), so when the user confirms removing one side, the surviving main-memory ID **is** the replacement reference, exactly as quoted; the never-quote rule targets feature-local numbering only. Write `→ no replacement` straight away when the feature retires the behavior outright; that case has nothing to wait for.

   **5.1 step 9 closes these lines.** Completing a line you opened during this run is part of writing it, not a rewrite; the append-only rule in 5.4 governs lines from *previous* runs. No `<pending>` marker may survive the end of 5.1.

   If `changelog.md` has no entry for this feature yet, create it now using the 5.4 template, at the position 5.4 specifies (directly under the `## Merged Features Log` heading); 5.4 will then update that same entry rather than adding a second one.
4. Scan for references to the retired ID in `.specify/memory/spec.md` itself (cross-references such as "as specified in FR-005" survive the deletion of their target), and in `plan.md`, `constitution.md`, and the agent knowledge file. Do not rewrite them — list any dangling references in the Step 6 report.

Candidates the user did not confirm are left untouched, recorded in the top-level `## Unresolved Contradictions` section of the changelog, and reported in Step 6. Never remove an entry without explicit confirmation.

### 5.2 Update Main Plan (`.specify/memory/plan.md`)

The main plan is **one consolidated document**, exactly like the main spec: it mirrors the plan template's own sections (Summary, Technical Context, Project Structure, Routing, Configuration, Testing Strategy, and so on), and each feature's plan content folds **into those shared sections**. Never add per-feature blocks or headers (`## 003 Workstreams` is wrong) — the reader must see the current implemented state, not a log of how it accumulated; the changelog is the log. A `plan.md` an older version already filled with per-feature blocks is **not** reorganized wholesale (the Edit Rules' "preserve existing layout" applies): fold this run's content into the shared sections, creating them if the file has none, leave the old blocks as they are, and note the mixed layout in the Step 6 report. Plan content that fits no existing section gets a new **shared** section named for its topic, never for its feature, and carries a source ref like any other entry. **Scalar Technical Context fields** (Project Type, Performance Goals, Constraints, Scale/Scope, and the like) hold one value describing the **current implemented state**: when two features' values compose, merge them (`web-service` becomes `web-service with background worker`), keeping one source ref per contributing feature. When they genuinely conflict rather than compose, a scalar cannot hold both values, so this is a **deliberate, narrow override** of the Edit Rules' constraint-preservation test, which has no separate-entries option here: raise the conflict as a Step 3 question when it is consequential (a performance target, a hard constraint) rather than deciding silently; then keep the value describing the current implemented state, **drop the superseded value's source ref together with the value it cited** — a ref must never stay attached to a value that no longer reflects it — and record the old value, its ref, and the replacement in the revision note and the Step 6 report.

1. **Dependencies:** Add new packages (with versions) to "Primary Dependencies" or equivalent section.
2. **Project Structure:** Add new modules/services to the structure tree.
3. **Configuration:** Note new environment variables or config additions.
4. **Routing & Navigation:** Add new routes, endpoints, or wiring.
5. **Testing Strategy:** Add test coverage notes for new components.
6. **Remove from "Future Work"** anything that was just implemented.
7. Ensure plan reflects the *implemented* state.

### 5.3 Update Agent Knowledge File (GEMINI.md / AGENTS.md / CLAUDE.md)

1. Find the project's agent knowledge file (check, in order: `GEMINI.md`, `AGENTS.md`, `CLAUDE.md` in REPO_ROOT).
2. If found, update the sections below, creating any that are missing. The section set is defined **here**, not by a template: recent spec-kit versions ship no agent-file template (an older project may still carry one in `.specify/templates/`, in which case follow its layout for these sections). If the file contains a tool-managed marker block (such as `<!-- SPECKIT ... -->` markers), **never write inside the markers** — the CLI overwrites that region; place these sections outside it.

   **"Active Technologies"** — add any new languages/frameworks/versions from the feature plan.

   **"Project Structure"** — update if modules were added.

   **"Commands"** — add new build/run commands if the tech stack changed.

   **"Recent Changes"** — prepend a new entry:
   ```markdown
   - ###-feature-name: [Brief description of what was added]
   ```

   **"Known Issues & Gotchas"** — if `research.md` exists in the feature, extract any gotchas/issues and merge them using the standard format:
   ```markdown
   ### ⚠️ [Issue Title]
   **Issue:** [What went wrong]
   **Root Cause:** [Why it happened]
   **Prevention Rule:** [Actionable rule]
   ```
   Deduplicate against existing entries.

3. If no agent file exists, skip this step and note it in the report.

### 5.4 Archive to Changelog

Create or update `.specify/memory/changelog.md`. **One entry per feature**: if this feature already has an entry in the Merged Features Log, update that entry in place rather than appending a second one.

**Newest first.** The Merged Features Log is reverse-chronological: insert a new feature entry **directly under the `## Merged Features Log` heading**, above all earlier entries. Updating an existing entry keeps it where it is, and entries inherited from older versions are never reordered. When you update an entry an older version wrote, bring **that entry's own** header and `**Spec:**` line to the current format (the `archived` label and the file link) — the same touch-to-upgrade principle as legacy refs; entries this run does not touch keep whatever format they have. The `## Unresolved Contradictions` section stays at the end of the file regardless.

The entry's date is the **archival date**, which is why the header says `archived` — a feature is often archived well after its branch merged, and pretending otherwise would misdate the log. The `**Spec:**` line is a relative Markdown link from `.specify/memory/` to the feature's spec file; a file-level link has no heading anchor that can rot.

```markdown
## Merged Features Log

### [FEATURE NAME] — archived YYYY-MM-DD
**Branch:** [branch-name from plan.md]
**Spec:** [specs/###-feature-name/spec.md](../../specs/###-feature-name/spec.md)

**What was added:**
- [Summary of user stories/scenarios implemented]

**New Components:**
- [Modules/services added]

**Superseded:**
- RETIRED: FR-005 (from specs/003-billing/spec.md) → replaced by FR-022. Reason: [one line]
- RETIRED: FR-008 (from specs/004-export/spec.md) → no replacement. Reason: [one line]

**Tasks Completed:** [completed]/[total] tasks
```

Count tasks using the checkbox format: `- [X]` or `- [x]` = completed; `- [ ]` = incomplete. If `tasks.md` does not exist, omit the "Tasks Completed" line.

The **Superseded** block is a permanent audit trail of IDs removed from the main spec in 5.1.1, and belongs to the feature entry that removed them. It is **append-only across runs**: once a run has finished, its lines are immutable — never edit, reorder, or prune them. (Completing a line you opened earlier in the *current* run, per 5.1.1 step 3, is part of writing it, not a rewrite.) Omit the block when the feature retired nothing, and never add a line for a removal that did not happen.

Every line starts with the literal marker `RETIRED:` followed by the retired ID, because 5.1's ID rules scan for exactly that marker when collecting IDs that must never be reissued. The rest of the line names a **live** replacement and is deliberately ignored by that scan. If the retired entry carried several source refs, list them all; if it carried a legacy ref or none, say so.

#### Unresolved Contradictions (top-level, not per-feature)

Maintain a single `## Unresolved Contradictions` section at the **end of `changelog.md`**, outside the Merged Features Log:

```markdown
## Unresolved Contradictions

- FR-012 vs FR-023 — [one line on how they conflict]. Raised by specs/007-invoice on YYYY-MM-DD; user declined removal.
```

This is a **working list, not an audit trail**, which is why it is deliberately kept out of the per-feature entries: it is meant to shrink, and resolving an item should never mean editing a past feature's record. Step 2.4 reads this one section on later runs and re-raises each pair while both entries are still present and still conflicting, so a declined contradiction gets another chance instead of becoming invisible. **One line per pair**: when a pair the user declines again is already listed, update its existing line — append the new decline date, keep the original *Raised by* attribution — never add a second line for the same pair.

**Delete a line once its contradiction is resolved** — because one side was removed, because the entries no longer conflict, or because the user confirmed the removal on a later run. A resolved pair left here would be re-raised forever. Omit the whole section when the list is empty.

### 5.5 Update Feature Spec Status

In the feature's `spec.md` and `plan.md` files (inside `FEATURE_DIR`, **not** in memory), check for a `**Status**:` metadata field in the document header (typically in the first 10 lines, e.g., `**Status**: Draft`).

If found and the value is `Draft`, update it to `Completed`:
- `**Status**: Draft` → `**Status**: Completed`

This marks the feature specification as finalized after merge. Do not change other status values (e.g., `In Progress`, `Blocked`) — only `Draft` → `Completed`.

---

## Step 6: Archival Report

Output the following structured report. Use **absolute paths** for all file references.

```markdown
# Archival Report

## Changed Files
| File (absolute path) | Change Summary |
|----------------------|----------------|
| `/absolute/path/to/spec.md` | Added [IDs], [N] user stories, [N] entities |
| `/absolute/path/to/plan.md` | Updated dependencies, project structure |
| `/absolute/path/to/changelog.md` | New entry for [feature name] |
| `/absolute/path/to/GEMINI.md` | Recent Changes, Known Issues |

## Sources
[Confirm every change came only from the Allowed Sources. Name anything you needed but could not find, and state that you did not reconstruct it. If you consulted git or any other tool to verify your own writes rather than to obtain content, say so here.]

## Path Resolution
[`FEATURE_DIR` and how it was resolved. Note it when `{SCRIPT}` reported a different feature directory, or when the script failed and `REPO_ROOT` was derived by walking up from the argument. Otherwise "Resolved from argument".]

## Feature Status
[List spec/plan files whose status was updated from Draft to Completed, or "No status fields found"]

## Bootstrapped
[List any files that were created for the first time, or "None". For a bootstrapped `spec.md`, confirm every category extracted in Step 1 is present in the file, or name the ones that are not and why.]

## Constitution Compliance
[Confirm all merged content respects constitution constraints, or list any unresolved CRITICAL conflicts]

## Edits Applied
[Brief summary of each artifact update]

## Conflicts Resolved
[List any conflicts that were resolved and how, or "None"]

## Consolidation
[Feature items folded into existing entries, e.g. "this feature's equivalent requirement folded into FR-012, which now carries 2 source refs". Or "None"]

## Superseded Requirements
[Confirmed removals as `OLD-ID (retired) → replaced by NEW-ID` or `OLD-ID (retired, no replacement)`. Also list:
- candidates left unresolved, and the contradiction each leaves in the spec (these are also written to changelog.md and re-raised next run)
- **deferred and unrecorded** — candidates deferred because the supersession gate was closed *and* the contradiction could not be written to changelog.md. Name the scope responsible and state plainly that these will **not** be raised again automatically; recommend a re-run at full scope
- dangling references to retired IDs found in spec.md, plan.md, constitution.md, or the agent file
Or "None"]

## Outstanding Items
[Any remaining `NEEDS CLARIFICATION` markers. Also name any user story carried across with no Acceptance Scenarios, per 5.1 step 2 — the story is archived, but nothing states how to verify it. Any story fold where the incoming priority differed from the existing entry's (the existing priority was kept). Any legacy directory-level ref left unchanged because its item could not be identified. If `FEATURE_DIR` contains a `bugs/` directory, state that it exists and that its content was not archived (see Allowed Sources). Or "None"]

## Defaults Applied
[Any decisions made with reasonable defaults instead of asking, or "None"]

## Scoping
[Which artifacts were updated, and which were skipped due to scope modifiers. Name any artifact whose **bootstrap** was suppressed by scope in Step 0.4, and state that recovering this feature's content into it needs a re-run of this same feature at full scope.]
```

**Important:** Do NOT delete the input feature spec files.

---

## Step 7: Post-Archival Hooks & Recommendations

### 7.1 Check Extension Hooks (after archival)

Check if `REPO_ROOT/.specify/extensions.yml` exists:
- Look for entries under `hooks.after_archive`
- Apply the same filtering and output logic as Step 0.6
- If no hooks are registered or the file does not exist, skip silently

### 7.2 Recommendations

Provide actionable next steps:

1. **Manual Review Items:** Anything flagged during conflict detection or constitution compliance check.
   - If any supersessions were reported as **deferred and unrecorded**, recommend re-running `/speckit.archive.run` at full scope (no modifiers) so they can be raised, decided, and recorded.
2. **Cleanup Suggestions:**
   - Can the feature spec folder be archived? (e.g., `mv specs/###-feature-name .specify/archive/`)
   - Are there orphaned files to remove?
3. **Verification:**
   - Run `make test` (or the project's equivalent) to verify nothing broke.
   - Review the archival report for accuracy.
4. **Follow-up:**
   - Update `README.md` if CLI commands or user-facing APIs changed.
   - Capture architectural insights from `research.md` into project memory if applicable.

---

## Done Criteria

- All content taken only from the Allowed Sources. Nothing reconstructed from git history, deleted files, notes, or an agent memory store. Nothing merged from `bugs/` or the `## Clarifications` log, with a `bugs/` directory's presence reported when one exists.
- Every archived story carries its entire block — all labelled fields, not an enumerated subset.
- All non-conflicting feature content merged into main memory artifacts.
- Feature content folded into existing entries where equivalent, each carrying item-level source refs. No pre-existing entry merged into another.
- Confirmed supersessions applied, their IDs retired, and one `RETIRED:` line opened at removal and closed out by 5.1 step 9 — none left `<pending>`. Unresolved contradictions recorded in the top-level changelog section so the next run re-raises them, or reported as "deferred and unrecorded" when scope prevented that. Nothing removed without explicit confirmation.
- Constitution compliance verified for all merged content.
- Memory directory bootstrapped for every artifact this run's scope will populate, and any artifact whose bootstrap was suppressed by scope named under `## Scoping`.
- Feature spec `**Status**: Draft` updated to `Completed` (if applicable).
- Conflicts either resolved (with user input) or marked with `NEEDS CLARIFICATION` (max 3).
- Archival Report printed with absolute paths for all changed files, constitution status, and next steps.
- Scoping hints respected — skipped artifacts explicitly noted.
