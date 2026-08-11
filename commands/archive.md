---
description: "Archive a feature specification into main project memory after merge, resolving gaps and conflicts"
argument-hint: "specs/###-feature-name [--spec-only|--plan-only|--changelog-only|--agent-only] [guidance text]"
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
- **Then**: scope modifiers (optional, `--` flags immediately after the path), followed by optional free-form guidance text

**Supported scope modifiers** (if none provided, update all artifacts):
- `--spec-only` — update only `.specify/memory/spec.md`
- `--plan-only` — update only `.specify/memory/plan.md`
- `--changelog-only` — update only `.specify/memory/changelog.md`
- `--agent-only` — update only the agent knowledge file (GEMINI.md / AGENTS.md / CLAUDE.md)

If **several** scope modifiers are supplied, the scope is their **union** — `--spec-only --changelog-only` updates both `spec.md` and `changelog.md` and nothing else. "Only" bounds the whole set, not each flag individually.

**Validate everything else.** The first three checks are textual and run before Step 0; the fourth needs `REPO_ROOT` and so runs as soon as 0.1 has resolved it, still ahead of every write. **No file is written when any of them fails** — a rejected invocation must leave the repository exactly as it found it. What survives the four checks is a scope modifier or guidance text; there is no third category.

1. **Empty input, or no feature at all.** If `$ARGUMENTS` is empty, **or the first token starts with `--`** (the feature path must come first, before any modifier), output `ERROR: No feature spec directory provided. Usage: /speckit.archive.run specs/###-feature-name [--scope-modifier] [guidance]` and stop.
2. **More than one feature.** This check comes **before** the flag check, so a range or a second path gets the guidance below rather than a generic parse error. Reject when the input covers more than one feature. The checks key on **feature-shaped tokens** — a token whose path contains a `specs/###` segment in any form, with or without the trailing name (`specs/002-x`, `specs/001`, `../specs/002-x`, `/repo/specs/002-x`; a glob character may stand in for digits, as in `specs/00*`), or a bare feature number or name such as `007` or `007-invoice-settings`. A `specs/` path counts **anywhere** in the input; a bare number or name counts only **in the leading region** (before the first prose token), or **beside a range marker whose other side is a feature reference** (the `008` in `specs/001 thru 008`) — digits inside later prose (`handle 404 errors`, `max 500 items`, `attention to 3 edge cases`) are guidance, and so is ordinary punctuation (`double-checked?`, `billing/invoicing`, `*emphasis*`):
   - two or more feature-shaped tokens
   - a glob character (`*` or `?`) **inside a feature-shaped token** (`specs/00*`, `0??-export`)
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
3. **Unrecognized flag.** Flags are recognized **only until the first non-flag token**: a `--` token in that leading position that is not one of the four modifiers gets `ERROR: Unrecognized flag '[token]'. Supported: --spec-only, --plan-only, --changelog-only, --agent-only.` and stops the run. From the first non-flag token onward, everything — including words that start with `--`, such as guidance about a `--force` flag — is **guidance text** (see Guidance Text below), taken verbatim. Rule 2 has already run, so a second feature reference, range, or glob written in any form rule 2 recognizes has already been rejected before guidance exists.
4. **Ambiguous first token.** The first token must resolve to **exactly one** existing directory under `REPO_ROOT`. A numeric prefix such as `specs/001` may expand to `specs/001-name-of-feature` only when exactly one directory matches. If nothing matches, or more than one does, output `ERROR: '[token]' does not resolve to exactly one feature directory` — listing the matches when there are several — and stop.

### Guidance Text

Everything after the feature path **and its leading scope modifiers** is **guidance**: free-form instructions from the user, the same channel every core spec-kit command offers. (A malformed `--` token in the leading flag position is rule 3's fatal error, never guidance.) Example: `/speckit.archive.run specs/007-invoice Pay extra attention to the entity model and call out anything touching billing in the report.`

Guidance **steers, never overrides**. It may direct your attention and emphasis, request extra detail or specific call-outs in the Step 6 report, and name areas to double-check. It may **not**:

- add content sources — Allowed Sources bounds every read, guidance included
- skip, reorder, or weaken any step or check
- change the scope — only the four modifiers do that
- change ID assignment or renumber anything
- authorize a removal — only the Step 3 confirmation does

When guidance asks for something on this list, do not comply and do not stop: run the command normally and state in the report which part of the guidance was set aside and why. Guidance makes runs harder to reproduce, so the Step 6 report echoes it verbatim under `## Guidance` — a reviewer can always see what shaped the run, and a run with no guidance is the reproducible baseline.

One consequence of the ordering: rule 2 runs before guidance is classified, so a `specs/...` path anywhere in the input — or a bare feature number in the leading region or paired with a range word and another feature reference — is rejected as a second feature even when it was meant as guidance. Ordinary prose, punctuation, and numbers inside sentences are safe; refer to other features by name in prose ("the invoice feature") when guidance must mention one.

---

## Allowed Sources (hard boundary)

Everything you write into main project memory must come from the files below. **This list is complete.**

- The artifacts inside `FEATURE_DIR` — the `bugs/` subdirectory only through the **two bounded reads** defined in the exclusions below, everything else fully. 0.3 inventories these artifacts and Step 1 reads the main ones; `contracts/` also feeds the steps that ask about it (2.3's Integration gap row, 5.2's Routing), while `checklists/` and `quickstart.md` are inventoried and allowed should a step ask, though today none takes content from them
- The existing files in `.specify/memory/`, `constitution.md` among them (0.4, 0.5, Step 2, Step 5)
- `.specify/templates/` — any template a step calls for; today the seed templates in Step 0.4, plus an agent-file template where one exists (recent spec-kit versions ship none — Step 5.3 does not depend on it)
- `.specify/extensions.yml` (Steps 0.6 and 7.1)
- The agent knowledge file in `REPO_ROOT` (GEMINI.md / AGENTS.md / CLAUDE.md), for Step 5.3
- The output of `{SCRIPT}`

The step numbers above are **descriptive, not restrictive**. This list bounds *which files* you may take content from, never *which step* may read one. If a step needs a file on this list, it may read it.

**Take content from nowhere else.** Not from git history, `git log`, `git show`, stashes, other branches, or any file that was deleted or renamed. Not from ad-hoc notes files. Not from an agent memory or session store. Not from another feature's spec directory: other features reach main memory only by being archived in their own run.

**Two bounded exclusions inside otherwise-allowed locations.** First, a `bugs/` subdirectory of `FEATURE_DIR` (bugfix extensions write `BUG-###.md` reports into it) is **not a source of requirement text**: never merge a bug report's amendments into main memory, and never let one alter the text of an item being archived — in test runs this produced requirements silently rewritten from bug files, with the outcome depending on which agent ran the command. The sanctioned channel already exists: a bugfix extension's patch step writes its amendments into the feature's own `spec.md`/`plan.md`/`tasks.md`, which Step 1 reads normally (see **Bugfix annotations** there). Exactly **two bounded reads** of the report files are allowed: each report's header fields (bug ID and title — they share the report's first line — plus Type, Severity, `**Status**`) for the Step 1 audit, the Step 6 status listing, and the 5.3 entry titles, and each report's `## Root Cause Analysis` for the agent file's Known Issues in 5.3. Nothing else — and the Step 6 `## Sources` section declares both reads whenever they happened. Second, the feature spec's `## Clarifications` section (the Q&A session log `/speckit.clarify` maintains) is **deliberately not archived**: clarify already integrates every accepted answer into the sections this command does archive, so its decisions arrive through them. Do not copy the log, and do not carry an empty `## Clarifications` heading into main memory.

*One narrow exception:* when the **Legacy refs** edit rule asks you to upgrade an existing `[Source: specs/###-feature-name]` ref — or when 5.2 seeds a shared scalar field from a legacy per-feature block and needs a ref for that block's feature, which runs the same ladder on the same directory-level ref — you may open that feature's **corresponding artifacts** — `spec.md` for an entry in the main spec (plus `data-model.md` for an entity, which is often defined there), `plan.md` for an entry in the main plan — **solely to identify which item the ref points at**, and may copy that item's ID, or its heading or opening phrase, into the ref itself. Take nothing else from those files, and never into the entry's own text. If the item cannot be identified in any of them, **leave the directory-level ref as it is** and note it in the Step 6 report: a coarse ref that is true beats a file-level ref that guesses.

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

A `bugs/` directory may also exist (bugfix extensions such as `spec-kit-bugfix` write `BUG-###.md` reports into it). Inventory the report files if present. Only the two bounded reads defined in Allowed Sources apply to them — header fields for the Step 1 audit, Root Cause Analysis for 5.3 — and requirement text is never taken from them. (Step 0.6, which runs later, records whether a bugfix extension is installed; Step 6 combines that with this inventory and the Step 1 audit.)

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

**Constitution is non-negotiable.** Any feature content that conflicts with a constitution MUST rule — or leaves one of its obligations unmet, per 2.1 — is flagged as CRITICAL and raised with the user. Non-negotiable means it is never resolved silently. The two branches then differ, and 2.1 and Step 3 state each: a **conflict** must be resolved before the conflicting item is archived, and an unresolved one withholds that item while the rest of the feature archives normally; an **unmet obligation** the user knowingly accepts is recorded as an accepted gap and nothing is withheld. A conflict is never closed as an accepted gap. Do not silently override or reinterpret constitution rules.

While extracting, note for each MUST rule — Core Principle, Architecture Standard, or Quality Gate alike — whether it forbids something, **requires a statement** from features meeting a condition, or does **both** (one sentence often does: "MUST state its retention rule … changing it MUST be recorded, never silent"). 2.1 checks both kinds, and the second is invisible unless you record it here. Record also any location a rule names for its required statement, since 2.1 keys on it.

**A third shape: rules requiring an action.** Some MUST rules require something to be *done* rather than *stated* ("All API routes MUST have automated tests before merge"). This command reads artifacts; it cannot inspect a codebase, a test run, or a CI result, so it can never establish whether the action happened. Mark such a rule as **action-requiring** and note it for 2.1, which handles it differently from the other two shapes and never turns it into a CRITICAL finding.

### 0.6 Check Extension Hooks (before archival)

Check if `REPO_ROOT/.specify/extensions.yml` exists:
- If it exists, also note (for Step 6) whether the top-level `installed` list names a bugfix extension — an entry whose id is, starts with, or ends with `bug` or `bugfix` — `bug`, `bugfix`, `spec-kit-bugfix` all count; an id merely containing `bug` elsewhere, like `debug-tools`, does not. When the file or its `installed` list is absent, or the file cannot be parsed, record "unknown" and move on. This is context only, never a gate: the `bugs/` handling keys on the files existing, not on the installer.
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
- Count completed tasks: lines matching `- [X]` or `- [x]` — except a task carrying a `(reopened — BUG-NNN)` note, which counts as incomplete even when its checkbox still shows `[x]`
- Count total tasks: lines matching `- [ ]` or `- [X]` or `- [x]`

**From `bugs/` (if it exists) — a bounded audit, never content:**
- Per report file, the header fields only: bug ID and title (the report's first line), Type, Severity, and `**Status**`, whatever its value. Real reports vary: a field may be missing (record it as absent, never infer it), a status may say `Fixed` or anything else, and the root-cause section may not exist.
- Per report file, the `## Root Cause Analysis` section (or none) — solely for the agent file's Known Issues merge in 5.3
- **Classify each report**, and use this classification everywhere downstream: **addressed** — the report's ID appears in a `**Bugfix**:` annotation inside the feature artifacts (see below), so the patch demonstrably happened; **unverified** — everything else, *regardless of what its Status claims*. Field experience is that bugs get documented well but rarely marked patched or verified, so a status value is always a claim, never a corroboration.

**Bugfix annotations inside the feature artifacts.** A bugfix extension's patch step amends the feature's own `spec.md`/`plan.md`/`tasks.md` and leaves markers. When extracting:
- Text struck through with `~~...~~` counts as **superseded by a patch** only when a `**Bugfix**:` marker or a live replacement wording sits in or beside the same entry: then extract the replacement only, and never archive the struck text as current content. Struck text with neither marker nor replacement is not a patch artifact you can interpret — carry it as it stands and name it under `## Outstanding Items`; dropping it silently would violate the completeness rule.
- `**Bugfix**: [DATE] — [BUG-NNN] ...` lines are patch metadata, not requirements: do not archive them as content; collect the bug IDs they name for the changelog entry (5.4).
- A task annotated `(reopened — BUG-NNN)` is an incomplete task; count it as such.

---

## Step 2: Conflict Detection & Gap Analysis

Before merging, systematically check for issues.

**Empty comparison target (applies to 2.2, 2.3, 2.4, and 2.5).** A check that compares this feature against a main-memory artifact means nothing when that artifact is empty (defined in Step 0.4): there is no prior content to collide with, nothing that could be superseded, nothing to fold into, and every item is trivially "missing". **Skip each check whose comparison target is empty.**

Judge the two artifacts **separately**, because a run can have one populated and the other not:
- Spec-side — 2.2 requirement ID collisions and entity redefinitions, the Requirements and Data Model rows of 2.3, and the whole of 2.4 and 2.5 — keys on `.specify/memory/spec.md`.
- Plan-side — 2.2 dependency conflicts, 2.2 Technical Context scalar conflicts, and the Architecture, Integration and Testing rows of 2.3 — keys on `.specify/memory/plan.md`. (The scalar check is skipped when that file is empty or out of scope, like the rest of the plan side: an empty main plan has neither shared fields nor legacy blocks to disagree with.)

2.1 always runs: the constitution is independent of both.

### 2.1 Constitution Compliance (CRITICAL)

This step checks **two** things, and a MUST rule can fail either way. Both branches cover everything 0.5 extracted: Core Principles, Architecture Standards, and Quality Gates.

1. **Conflicts.** For each extracted requirement, user story, and architecture decision, verify it does not conflict with any constitution MUST rule.
2. **Unmet obligations.** Some MUST rules do not forbid a behavior, they *require a statement* from any feature that does a certain thing ("every feature that stores user data MUST state its retention rule"). A feature that does the thing and never makes the statement violates the rule by omission, and this is not a "conflict" — nothing in the feature contradicts the text, the required content is simply absent. Flag it here anyway; it is exactly as binding. Obligation-shaped wording appears in Core Principles, Architecture Standards, and Quality Gates alike.

   **Bounded so this stays a check, not a survey.** Three bounds:

   - **Triggered only by what this feature does.** The obligation applies **only when this feature actually does the thing the rule conditions on**, evidenced in the feature's own artifacts (its spec, plan, or data model shows it storing user data, adding an API route, and so on). A rule whose condition this feature never meets is not checked. A rule stated **without** a condition applies to every feature, so its condition is always met; it is checked the same way, and it is satisfied or not on the same evidence — an unconditional rule is not a licence to flag, only a rule that is never skipped.
   - **Where the statement may live.** Look **only in this feature's own artifacts**, and only in the ones **Step 1 reads** — never in main memory, and never in the two places Allowed Sources excludes (a `bugs/` report beyond its two bounded reads, and the spec's `## Clarifications` log). An obligation is met by a statement anywhere among those artifacts, **unless the rule names a location** ("MUST state its retention rule *in the spec*"), in which case it must appear there; a rule's own wording is never reinterpreted (0.5). The statement must be **this feature's own** — that is the whole of the topicality test. Do not go further and ask whether one statement covers every kind of data the feature stores: where a rule is worded per feature ("every feature … MUST state *its* retention rule"), a retention statement this feature makes satisfies it, and whether its coverage is complete is a judgment for the user, not a flag from this step. Main memory does not count even when it holds a matching statement, for two reasons — rules of this kind bind **each feature**, so an earlier feature's compliance is not this one's, and an entry there can be removed by a confirmed supersession later in this very run (5.1.1). The consequence is deliberate: every feature meeting the condition raises its own obligation, and answering for one feature does not answer for the next.
   - **Action-requiring rules are never flagged here.** A rule 0.5 marked action-requiring is checked by looking at what the feature's artifacts say about that action, and the outcome is **never** a CRITICAL finding and never a Step 3 question. If an artifact plainly states the action was not done, that is a **conflict** and goes through branch 1 on that statement. If the artifacts say nothing either way — the ordinary case, because a plan is not a test report — the rule is simply **unverifiable from the allowed sources**: record it under `## Outstanding Items` as unverified, naming the rule and what would settle it, and move on. Silence in a plan is not evidence the action was skipped, and a mandatory question built on that silence would fire on almost every feature. This is the one shape where "not evidenced" and "not satisfied" are different things.
   - **What it never does.** An unmet obligation **never withholds content.** Unlike a conflict, nothing in the feature contradicts the rule — the triggering content is the feature's ordinary work, and it is archived normally. The Edit Rules' "do not merge content that violates them" governs conflicts, not this branch. The obligation is asked in Step 3 and reported; it is not a reason to drop an entity, a requirement, or a story.

**If a conflict exists**, flag it as CRITICAL:
```
🔴 CONSTITUTION CONFLICT:
- Feature FR-XXX: "[requirement]" conflicts with Principle [N]: "[principle text]"
  → This MUST be resolved in Step 3 before this item can be archived.
```

**If an obligation is unmet**, flag it as CRITICAL in its own form:
```
🔴 CONSTITUTION OBLIGATION UNMET:
- [Principle N / Architecture Standard / Quality Gate]: "[rule text]"
  Triggered by: [what this feature does that meets the rule's condition, and where it is stated]
  Missing:      [the statement the rule requires and no artifact makes]
  → This must be raised in Step 3 before archival completes; the user decides how it is resolved.
```

**The feature's own Constitution Check is input, not a verdict.** A feature's `plan.md` usually carries a `## Constitution Check` section. Read it and quote it in the Step 3 question when it bears on the answer, but **never close a flag on it alone**: it records what the author believed at planning time, before the work was done, and re-checking that belief against the finished artifacts is the reason this step exists. "No violations" in a plan is evidence about intent, not a finding.

**Where your own judgment belongs, and where it does not.** Deciding whether a rule's condition is met — does this feature store user data, does it add an API route — is this step's work and must be done here; that is what keeps the check bounded, and a condition genuinely not met means no flag. Deciding that a *met and unsatisfied* obligation is too minor to raise is not: once a flag qualifies, it goes to Step 3 and the user decides. If you find yourself arguing why an unmet obligation need not be asked about, that argument is the answer to the Step 3 question, not a reason to skip it.

### 2.2 Conflicts

1. **Requirement ID Collisions:** If the feature has an ID that already exists in main spec, flag it.
2. **Entity Redefinitions:** If an entity is being modified (not just added), highlight the delta.
3. **Dependency Conflicts:** If a new dependency version conflicts with existing ones, note it.
4. **Technical Context Scalar Conflicts:** compare **every pair of values that 5.2 will have to reconcile** in a shared Technical Context field, and note each direct disagreement. That is three pairings, not one: this feature's value against the main plan's shared field; this feature's value against any legacy per-feature block's; and — because 5.2 composes legacy values into shared fields whether or not this feature touches them — each legacy block's value against the shared field and against the other legacy blocks', **including for fields this feature says nothing about**. 5.2 resolves these while writing, but its rule asks for a Step 3 question on the consequential ones, and Step 3 runs exactly once, before Step 5, so a conflict found only at writing time can no longer be asked about. Detect them here.

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

### 2.5 Consolidation Candidates (detection pass)

The step that makes folding real. Without a detection pass, "zero duplicates" is a statement about not having looked — a field run against a 612-entry main spec folded nothing from 30 incoming items and reported zero duplicates, truthfully, because no step ever compared them. **Skip this step when the comparison target is empty or when `spec.md` is not in scope** (matching 2.4's guard — a contradiction verdict must never route work into a skipped step).

1. **Key the incoming items.** For each story, requirement, **entity**, edge case, outcome, and assumption extracted in Step 1, derive an **imperative-phrase slug**: lowercase actor-action-object, stripped of modality and filler — "System MUST send the task owner a notification 24 hours before the deadline" becomes `owner-deadline-notification`; "Users MUST be able to assign a task to exactly one owner" becomes `assign-task-single-owner`; an entity's slug is its name plus its distinguishing fields. Slugs are search keys for this step only; they are never written into any artifact. (The approach follows core `/speckit.analyze`'s token-efficient duplication pass; the procedure here is self-contained and does not depend on that command.)
2. **Index the target section.** Derive the same slugs for the **existing** entries of each section that will receive incoming items — section by section, never the whole document at once. This is what keeps detection workable at several hundred entries.
3. **Shortlist by slug overlap.** An incoming item and an existing entry form a candidate pair when their slugs share the **object noun and the action verb stem** (for entities: the same name, or the same object noun in their field lists). Rank candidates by the number of shared slug tokens, breaking ties by the incoming item's document order, then by the existing entry's document order. Cap the shortlist at **20 pairs per target section**, with two kinds of pairs **never dropped by the cap** — they are admitted *in addition to* the 20 ranked slots, so the cap's arithmetic never forces one out: entity pairs sharing the same name (entities are identity-keyed — a second same-named entity is exactly the redefinition 2.2 flags) and pairs whose texts match exactly. When more pairs qualify than the cap allows, keep the strongest by the ranking and name each dropped pair under `## Outstanding Items` with a recommendation to re-run after resolving the backlog — a dropped pair is unexamined, and an incoming item **none of whose retained pairs folds** enters as a new entry, which is the accumulation this step exists to prevent.
4. **Judge each shortlisted pair by prose**, with exactly three verdicts:
   - **fold** — same actor, capability, and object, and the merged text can keep **every** constraint from both sides — because one side's conditions contain the other's, or because the merged wording states both (the Edit Rules' preservation test, verbatim)
   - **separate** — same territory but genuinely different conditions, limits, or qualifiers that one merged statement cannot carry without losing a side; both entries stand
   - **contradiction** — the two cannot both hold; add the pair to 2.4's candidates and let the supersession flow handle it
   For **user stories**, judge by the same-user-goal test of 5.1 step 2 instead: two stories covering the same goal are **fold** even when their fields differ, because the block merge keeps both statements of a differing field; **separate** applies only when the goals differ. For **same-named entity pairs**, only two verdicts exist — **fold** (the incoming definition matches or extends the existing entity) or **contradiction** (it redefines the entity incompatibly; route to 2.4, and 2.2 flags it) — never `separate`: entities are identity-keyed, and two entries under one name are the restatement 5.1 step 4 forbids.
5. **Record the verdict table** — one line per pair: incoming item → existing entry → verdict — and include it in the Step 4 impact preview so the user sees the warrant before any edit. 5.1 folds exactly the pairs marked fold and nothing else. The keying, ranking, and tie-break rules above are fixed so that rerunning without changes reproduces the same shortlist; the verdicts themselves remain prose judgment, and the table's job is to **record** that judgment where it can be audited, instead of leaving it implicit in the writing step.

---

## Step 3: Clarify (exactly once; max 5 questions)

If conflicts or gaps require human judgment, ask **only questions that materially change scope or correctness**. Skip this step entirely if everything is unambiguous.

**Both sentences above govern discretionary questions only** — the materiality filter and the skip-the-step-entirely clause alike. The **Always ask** questions below are exempt from both: none of them may be dropped for failing the materiality test, and their presence means the step is not skipped, however unambiguous the rest of the run looks. Their value is the record of the user's decision, not the effect on this run's output, so "the answer would not change what gets written" is never a reason to drop one.

**Always ask** if any CRITICAL constitution findings were detected in 2.1 — conflicts and unmet obligations alike — because these cannot be auto-resolved. For an unmet obligation, quote the rule, what triggered it, and the feature's own Constitution Check verdict if it made one, then ask the user how to resolve it. The options are typically: **amend the feature spec and re-run this command**, **record it as an accepted gap** and archive as-is, or **declare the obligation not triggered** because the condition is not actually met.

- **For an unmet obligation, this run does not stop, whichever option is chosen.** There is no abort path from an obligation answer: the feature's content is archived now (2.1's third bound), and on the amend-and-re-run option the added statement arrives through an Allowed Source on the next pass and is merged by 5.1 like any other incoming item — the Edit Rules' idempotency rule keeps the re-run from adding a second copy of what is already there. Say so when offering that option, so the user is not left expecting the run to halt.
- **Never write the missing statement yourself, into any artifact** — not `.specify/memory/`, and not the feature's own `spec.md` or `plan.md` either. A Step 3 answer is not an Allowed Source, and text you place in the feature spec would be read back as a source on the next run, producing a `[Source: ...]` ref that looks honest and is not. Amending the feature spec is the **user's** action, taken outside this command.

**Ask conflicts separately from obligations.** The two branches take different answers, so one question cannot serve both: an obligation may be closed as an accepted gap, and a conflict never may (0.5). Ask one question covering all conflicts and a second covering all unmet obligations. The conflict question's legal options are: **amend the feature and re-archive**, **declare it not actually a conflict** (the rule does not say what it appeared to say, or the item does not do what it appeared to do), or **leave it unresolved**, which withholds that item. "Archive it anyway" is **not** an option — the Edit Rules forbid merging conflicting content and no Step 3 answer overrides them, unlike a confirmed removal, which they explicitly key on.

**What an unresolved conflict does.** A conflict the user does not resolve **withholds that item and only that item**: the conflicting requirement, story, or decision is not merged (Edit Rules), everything else in the feature is archived normally, and the run completes. It is reported under `## Constitution Compliance` as unresolved and under `## Outstanding Items` with the recommendation to resolve it and re-archive. 2.1's conflict template says the conflict must be resolved "before this item can be archived" — *that item*, not the run; nothing anywhere in Steps 3 to 7 aborts a run.

**Withholding is the one exception to this command's completeness promise**, and it is bounded to exactly the flagged item. Three rules elsewhere read as absolutes and must be understood with it:

- The opening promise that nothing is lost from the feature's own artifacts, and 5.1's **Empty seed** rule that nothing extracted in Step 1 may be left out, both mean *nothing except an item withheld by an unresolved 2.1 conflict*. That item is named in the report instead, so it is visible rather than lost, and re-archiving after the conflict is resolved brings it in.
- A withheld item's **2.5 fold verdict is not applied.** The pair simply does not fold, exactly as a fold pair whose target was removed by a supersession does not fold; count it as not folded, and name the reversion in the Step 6 report. "Folding matches the verdict table exactly" means every fold it marks *whose incoming item was archived*.
- When a withheld item is the **replacement for a confirmed supersession**, the removal has already happened and its `RETIRED:` line cannot name a replacement that was never written. Close the line as `→ replacement withheld (unresolved constitution conflict)` and name the pair under `## Outstanding Items`. This satisfies the no-`<pending>` rule with a true statement rather than a false `no replacement`.

**Always ask** if any supersession candidates were detected in Step 2.4, **provided the supersession gate is open** (defined once below). Removal is destructive and requires explicit confirmation. Ask this question first if the budget is tight.

**Bundle by category.** Four **mandatory** question categories can fire in one run — constitution conflicts, unmet obligations, supersession candidates, and 5.2 scalar conflicts. Ask **one** question per category covering all of that category's items, exactly as the supersession question already does, rather than one question per finding. That leaves one slot of the five for everything discretionary (2.2 ID collisions and entity redefinitions, 2.3 gaps, anything else needing judgment), so bundle those into a single question too, and when they will not fit, report them under `## Outstanding Items` rather than dropping a mandatory question to make room.

**The supersession gate.** Supersession requires **both** `.specify/memory/spec.md` (where the entry is removed from) and `.specify/memory/changelog.md` (where the audit line goes) to be **writable under the current scope**. Compute this from the scope modifiers actually supplied rather than assuming any particular one; with no modifiers, everything is in scope and the gate is open.

When the gate is closed: do not ask the question, remove nothing, write no `RETIRED:` lines, and report the candidates as deferred, naming the scope that closed the gate. This gate governs the whole supersession flow — Step 3 and 5.1 alike.

**A closed gate also blocks the contradiction record.** The `## Unresolved Contradictions` list lives in `changelog.md`, so when that file is out of scope the deferred candidates cannot be written down anywhere durable. Meanwhile 5.1 still adds the feature's conflicting item as a new entry (except a same-named entity, which is never duplicated — its incoming definition is simply not archived, so what is lost there is the record, not spec hygiene), so the main spec ends the run holding both sides of a contradiction that nothing will re-raise. This is the one case where a scope modifier leaves the spec in a worse state than a full run.

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

This gives the user a preview before edits are applied. Include every confirmed supersession target as a `Remove` row, and append the 2.5 verdict table below the map — the folds about to happen and the pairs judged separate are part of the preview.

---

## Step 5: Archival (Apply Edits)

### Edit Rules
- Use absolute paths for all file references.
- Preserve existing section layout and ordering. Consolidate *within* a section; do not reorganize the document.
- **Consolidate, do not accumulate.** Merge each incoming item into the existing entry that already covers the same ground. Append a new entry only when no equivalent exists. The main spec is one consolidated specification, not a per-feature digest. **For the categories 2.5 keys** (stories, requirements, entities, edge cases, outcomes, assumptions), which items fold is decided by the 2.5 verdict table, not ad hoc while writing: fold the pairs it marks fold, keep its separate pairs separate, and never fold a pair it did not examine. The plan (5.2) and the agent file (5.3) are outside 2.5's scope and follow their own steps' merge rules.
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
- **Constitution constraints must be respected** — do not merge content that violates them. This governs 2.1's **conflict** branch, where specific feature content contradicts a rule. It does **not** govern 2.1's **unmet obligation** branch, where nothing contradicts anything and the triggering content is the feature's ordinary work: that content is archived normally and the gap is reported. Never withhold a story, requirement, or entity over an unmet obligation.

### 5.1 Update Main Specification (`.specify/memory/spec.md`)

Each step below **consolidates** into the existing section rather than appending a new per-feature block. For the categories 2.5 keys, folding is governed by the **2.5 verdict table**: fold exactly the pairs it marks fold, keep its separate pairs as separate entries, and leave its contradiction pairs to the supersession flow. A fold pair whose existing entry was just removed by a confirmed supersession (step 1 below) has lost its target: the incoming item enters as a **new** entry instead, and the reversion is noted in the Step 6 report.

**Removals come first.** Step 1 applies the confirmed supersessions, before any merging. Nothing can then be folded into an entry that is about to be deleted.

**Empty seed.** If `spec.md` is empty, the numbered steps below run **normally** and populate the empty sections; there is simply nothing to fold into. Nothing extracted in **Step 1 (Feature Analysis)** may be left out — that is the whole-command Step 1, not step 1 of the list below.

1. **Apply confirmed supersessions** — see 5.1.1 below. This happens before everything else.
2. **Merge User Stories / Integration Scenarios** — fold into an existing story when it covers the same user goal; otherwise add, maintaining priority ordering. **Carry each story's entire block across with it**: description, `Why this priority`, `Independent Test`, `Acceptance Scenarios`, and any other labelled field the story carries. The **heading line is reconstructed, not copied**: its number comes from the ID rules, and its priority is part of the block like any other field — a **new** story's heading takes the incoming priority verbatim, while a **folded** story keeps the existing entry's number and priority; when the incoming story states a different priority, keep the existing one and record the disagreement under `## Outstanding Items` in the Step 6 report — the incoming `Why this priority` rationale is still carried per the whole-block rule, but **label it** with its source feature and the priority it argued for (e.g. `(specs/002-notifications argued P1)`), so the story text does not contradict its own heading. Never carry a subset of the fields — a field left off an enumerated list is exactly how Acceptance Scenarios and priority rationales were silently lost in earlier versions. When folding, merge the blocks field by field: combine list-valued fields (such as scenario lists) dropping only exact duplicates, and where a prose field genuinely differs, keep both statements inside the one story — that is how the Edit Rules' constraint-preservation requirement is met for stories: nothing is lost, and the story is not split in two. Never drop a story because a field is missing: if the feature spec states no Acceptance Scenarios for it, carry the story across anyway and name it under `## Outstanding Items` in the Step 6 report. Report it there rather than as a 2.3 gap: 2.3 ran back in Step 2, and on a first archival its Requirements row was skipped outright because the main spec was empty. Inventing missing fields is not an option; Allowed Sources forbids it.
3. **Merge Functional Requirements** — fold into the existing requirement when it states the same capability; otherwise add, continuing from the highest existing ID. Group by domain/module if the spec is large.
4. **Merge Key Entities** — add new entities; extend existing ones with new fields rather than restating the entity.
5. **Merge Edge Cases and Error Handling** — fold cases describing the same failure mode into one entry.
6. **Update Data Flow / Architecture** if the feature changed system data flows.
7. **Merge Success Criteria / Measurable Outcomes** if present. Fold outcomes measuring the same thing into one entry; otherwise continue from the highest existing ID (e.g., SC-XXX).
8. **Merge Assumptions**: add new assumptions under the `## Assumptions` section (if the main spec lacks one, create it after Success Criteria to match the template's section order); skip any whose content is already recorded in main memory — an exact-content match is always in the 2.5 shortlist (the cap never drops it), so every such skip is backed by a fold verdict. A 2.5 **fold** verdict on an assumption pair means exactly this: keep the existing entry, attach the incoming feature's source ref per the Edit Rules, add no second copy — and count it as folded in the Consolidation numbers. **Number them**: assumptions carry `AS-###` IDs in the main spec, so they can be cited, deduplicated, and superseded like requirements. Do this in two passes, in this order. *First, backfill:* if the section holds assumptions with **no ID at all** (archived by an older version), number those, in their current order, continuing above the highest `AS` ID found in **either** the section or the changelog's retired list — the **Retired IDs are off-limits** rule applies to the `AS` sequence like any other. An assumption that already carries an ID, under `AS` or any other project convention, keeps it; never renumber. Backfilling adds a label and changes nothing else — it is **not** "touching" the entry for the Legacy refs rule, so it never by itself triggers a ref upgrade. *Then, new entries:* each assumption added this run continues above the highest `AS` ID after the backfill, again skipping retired IDs. For the source ref: most feature specs leave assumptions unnumbered, so the ref takes the quoted-phrase rung of the ladder; if the feature spec numbers its assumptions, cite the feature's own ID, as the ladder's first rung requires.
9. **Close out the `RETIRED:` lines** opened in step 1. Every replacement ID is now settled, so fill in each line's replacement reference (see 5.1.1 step 3). Do not finish 5.1 with a `RETIRED:` line left incomplete.

**Do not fold an incoming item into an entry you flagged in 2.4 as contradicting it.** Add it as a new entry instead, so the contradiction stays visible for the user to resolve rather than being silently merged away. **Entities are the exception**: never materialize a second entry under the same entity name. When a same-named entity contradiction is declined (or the supersession gate is closed), keep the existing entity entry unchanged and **do not archive the incoming definition**; record the conflict in the changelog's `## Unresolved Contradictions` — naming the entity and citing the incoming definition's source ref, since it has no main-memory entry of its own — and under `## Outstanding Items`, so the next archival of a feature touching that entity raises it again.

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

**Seeding a shared Technical Context field beside legacy blocks.** When you write a shared Technical Context field in a file that also holds legacy per-feature blocks, that field's contributing values are not only this run's: the legacy blocks hold earlier features' values for the same field. Compose **all** of them into the shared field — the legacy values and this run's — one source ref per contributing feature. Writing only this run's value publishes it as *the* current implemented state while the rest of that state sits in a block the shared section supersedes in the reader's eyes, and every later run folds into the understated field.

**This is a condition, not a one-time event.** It applies whenever a legacy block still holds an uncomposed value for a shared Technical Context field you are writing — whether this run creates that field or an earlier run did and left the legacy value behind. Check the legacy blocks for the field each time; a version before this rule existed could have created the shared field already. **One exception, and it matters**: a legacy value that any **earlier run's revision note** in this file records as the **losing side of a resolved scalar conflict** is not "uncomposed" — it was considered and deliberately dropped, so leave it alone. Read **all** the revision notes `plan.md` carries, not only the one you are writing this run; the record you are looking for is the one the conflict branch requires (the old value, its ref, and the replacement), and it was written by a previous run. Apply the exception **per item, not per line**: a legacy line often carries several items (`**Constraints**: single region, no offline mode`), and only the item named in the note is settled — the rest of that line still composes. Without this, a settled conflict would be re-composed or re-asked on every future run, quietly undoing the user's decision; with it applied too broadly, live values would be dropped alongside the settled one.

Four rules keep it narrow:

- **Technical Context fields only, judged by structure.** This rule covers exactly the **labelled fields** of a legacy block's Technical Context — a `**Field name**: value` line, one label carrying one value, appearing once (`**Language/Version**`, `**Primary Dependencies**`, `**Storage**`, `**Constraints**`, `**Scale/Scope**`, `**Project Type**`, `**Performance Goals**`). Compose every such field into its shared counterpart, whether its value is a single phrase or several items in one line: the label appears once, so the shared line reads as the whole value for that label and a legacy line left outside it is invisible to the reader. Everything else in a legacy block — a **section** of enumerated entries, whatever its shape (`### 001 Routing`'s bullets, `### 001 Structure`'s tree, a table of test cases) — **stays where it is**, because a reader unions a shared section with a legacy one and loses nothing. **Do not decide this by counting commas in the value**: `**Constraints**: single region, no offline mode` and `**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, Jinja2 3.1` are both labelled fields and both compose, even though the second reads as a list. Field versus section is the test. A labelled field **outside** Technical Context (a `**Structure Decision**:` line inside a legacy structure block, say) is out of this rule's scope and stays where it is; when its shared counterpart reads as incomplete without it, name that under `## Outstanding Items` rather than moving it.
- **How the values combine, and what counts as a conflict.** A field whose value **enumerates items** (`Primary Dependencies`, `Storage`) composes by taking the union of the items, which is also what step 1 below asks for; a field holding **one statement** composes by merging the statements, per the parent rule. Either way, a **conflict** is a direct disagreement about the same thing — `FastAPI 0.115` against `FastAPI 0.118`, or "single region" against "multi-region" — not the mere presence of items one side does not list. Only a direct disagreement takes the conflict branch, and it takes it for **that item alone**: the rest of the field still composes. When both sides state the **same** value, the field carries it once and both features are contributors, so it takes **one ref each** — a restated value is still that feature's value, and dropping its ref would understate who the field describes.
- **Which ref the legacy contributor gets.** The seeded field is a new entry, but its legacy contributor's provenance is only as good as the legacy block's own ref, so take that ref as the starting point and apply the Edit Rules' **Legacy refs** ladder to it: identify the field in the originating feature's plan if you can (that identification read is the sanctioned Allowed Sources exception), otherwise keep the directory-level form. Never invent an artifact name for a value you did not find in that artifact. When the legacy line carries **no ref at all** (a hand-written block), there is nothing to upgrade and nothing to guess: compose the value, attach no ref for that contributor, and say so in the Step 6 report — an unattributed value is honest, an invented ref is not. This ladder is applied to the **new** ref on the shared field only; the legacy block's own ref is left exactly as it is, because that entry was not touched.
- **The legacy blocks are still not modified**, in either branch. When the values **compose**, the composed shared field carries both, and the legacy line stays as it is, so that value now appears twice. When they **conflict**, the conflict branch above applies **in full and unchanged** — including its "when it is consequential" condition on asking, and its requirement to record the old value, its ref, and the replacement in the plan's revision note as well as the report — with one clarification: it governs what you *write*, not what already exists. So seed the shared field with the value describing the current implemented state and simply **do not carry the losing value or its ref into the shared field**. Do not move, delete, or strike the legacy line to enforce it. Two branches follow from that: when you are **creating** the field, nothing needs dropping, because neither the losing value nor its ref was ever written there; when you are **completing a field an earlier run already created** and the value already in it is the losing side, the parent rule applies without this clarification — drop that value and its ref from the shared field as it says, since both are already written. Name every field seeded this way in the Step 6 report alongside the mixed-layout note, saying for each whether the legacy line is now a **duplicate** (composed) or a **superseded statement still carrying its own ref** (conflict), and recommend pruning it by hand — the second kind matters more, because a reader who finds it will be reading a value the shared field contradicts.

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

   **"Known Issues & Gotchas"** — if `research.md` exists in the feature, extract any gotchas/issues; if `bugs/` reports carry a `## Root Cause Analysis` (the one content read Allowed Sources permits from them), turn each into an entry titled with the bug ID and short title. Merge both kinds using the standard format:
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
**Bugs addressed:** [the bug IDs collected from `**Bugfix**:` annotations in the feature artifacts, e.g. BUG-001, BUG-003 — the annotation *is* the corroboration, so this line works even when the `bugs/` directory is absent; these include every report the Step 1 audit classifies **addressed** (an annotation may also name a bug that has no report file). Never list an ID on the strength of a report's Status claim alone. Omit this line when none]
```

Count tasks using the checkbox format: `- [X]` or `- [x]` = completed; `- [ ]` = incomplete; a task with a `(reopened — BUG-NNN)` note = incomplete regardless of its checkbox (same rule as Step 1). If `tasks.md` does not exist, omit the "Tasks Completed" line.

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
[Confirm every change came only from the Allowed Sources. Name anything you needed but could not find, and state that you did not reconstruct it. If you consulted git or any other tool to verify your own writes rather than to obtain content, say so here. When the `bugs/` bounded reads happened, declare both: the header-field audit and any Root Cause Analysis taken for Known Issues. When another feature's artifacts were opened under the ref-identification exception (a Legacy refs upgrade, or a 5.2 scalar seeding), name the files and confirm nothing but ref text was taken from them.]

## Path Resolution
[`FEATURE_DIR` and how it was resolved. Note it when `{SCRIPT}` reported a different feature directory, or when the script failed and `REPO_ROOT` was derived by walking up from the argument. Otherwise "Resolved from argument".]

## Feature Status
[List spec/plan files whose status was updated from Draft to Completed, or "No status fields found"]

## Bootstrapped
[List any files that were created for the first time, or "None". For a bootstrapped `spec.md`, confirm every category extracted in Step 1 is present in the file, or name the ones that are not and why.]

## Constitution Compliance
[Confirm all merged content respects constitution constraints. List **every** 2.1 finding — conflicts and unmet obligations both — with its disposition: the user's Step 3 answer, or why it is still unresolved. A finding the user closed ("not actually triggered", "accepted gap") belongs here too; a closed finding that appears nowhere is indistinguishable from one that was never raised. For an accepted gap, state plainly that this report is its only record: nothing records the acceptance in any artifact, so **re-archiving this same feature will detect the gap and ask again**, and a future feature meeting the same condition will raise its own. If the acceptance should survive, the user has to record it themselves.]

## Edits Applied
[Brief summary of each artifact update]

## Conflicts Resolved
[List any conflicts that were resolved and how, or "None"]

## Consolidation
[Always give the 2.5 numbers first: `incoming items: M; candidate pairs examined: K (dropped by the shortlist cap: D); folded: N` — a zero must be legible as "examined and found distinct", never as "did not look". Then each fold, e.g. "this feature's equivalent requirement folded into FR-012, which now carries 2 source refs", and any fold verdict that reverted to a new entry because its target was retired. Or "None (target was empty; 2.5 skipped)" / "None (spec.md out of scope; 2.5 skipped)"]

## Superseded Requirements
[Confirmed removals as `OLD-ID (retired) → replaced by NEW-ID` or `OLD-ID (retired, no replacement)`. Also list:
- candidates left unresolved, and the contradiction each leaves in the spec (these are also written to changelog.md and re-raised next run)
- **deferred and unrecorded** — candidates deferred because the supersession gate was closed *and* the contradiction could not be written to changelog.md. Name the scope responsible and state plainly that these will **not** be raised again automatically; recommend a re-run at full scope
- dangling references to retired IDs found in spec.md, plan.md, constitution.md, or the agent file
Or "None"]

## Outstanding Items
[Any remaining `NEEDS CLARIFICATION` markers. Also name any user story carried across with no Acceptance Scenarios, per 5.1 step 2 — the story is archived, but nothing states how to verify it. Any story fold where the incoming priority differed from the existing entry's (the existing priority was kept). Any legacy directory-level ref left unchanged because its item could not be identified. Any 2.5 candidate pairs dropped by the per-section shortlist cap (named individually, with a re-run recommendation). Any struck-through text carried as-is because it had neither a Bugfix marker nor a replacement. Name any item withheld because a 2.1 conflict went unresolved, with the rule it conflicts with and the recommendation to resolve it and re-archive this feature; name any 2.5 fold that did not happen because its incoming item was withheld, and any `RETIRED:` line closed as `replacement withheld`. Name each **action-requiring** constitution rule 2.1 could not verify, with what would settle it — these are unverified, not violated. When `plan.md` has a mixed layout, name it here and name every Technical Context field seeded from legacy per-feature blocks (5.2), marking each legacy line as a duplicate or as a superseded statement per 5.2, and recommend pruning it by hand. If `FEATURE_DIR` contains a `bugs/` directory, list each report with its Step 1 classification (**addressed** / **unverified**) and its claimed Status, and state plainly: every **unverified** report may not be reflected in the archived spec — *regardless of what its Status claims* — and no requirement text was taken from the reports (see Allowed Sources). When Step 0.6 found a bugfix extension installed and unverified reports exist, recommend running its patch and verify steps before re-archiving; when reports exist but no bugfix extension is installed, note they may be stale; when the installed state is "unknown", say so and make neither recommendation. Or "None"]

## Defaults Applied
[Any decisions made with reasonable defaults instead of asking, or "None"]

## Scoping
[Which artifacts were updated, and which were skipped due to scope modifiers. Name any artifact whose **bootstrap** was suppressed by scope in Step 0.4, and state that recovering this feature's content into it needs a re-run of this same feature at full scope.]

## Guidance
[The guidance text verbatim, one line on how it shaped the run, and any part set aside because it asked for something the Guidance Text rules forbid. Or "None provided."]
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

- All content taken only from the Allowed Sources. Nothing reconstructed from git history, deleted files, notes, or an agent memory store. No requirement text merged from `bugs/` (only the bounded header audit and Root Cause intake) nor from the `## Clarifications` log; every bug report listed with its audited status when a `bugs/` directory exists; struck-through patched text never archived as live.
- Every archived story carries its entire block — all labelled fields, not an enumerated subset.
- Guidance text, when provided, applied within its limits: echoed verbatim in the report with any refused part named; scope, sources, steps, IDs, and removals unaffected by it.
- Folding matches the 2.5 verdict table exactly, and the report's Consolidation section carries the examined/folded counts when 2.5 ran (a skipped 2.5 reports its skip reason instead).
- All non-conflicting feature content merged into main memory artifacts.
- Feature content folded into existing entries where the 2.5 verdict table judged them equivalent, each carrying item-level source refs; any pairs the shortlist cap dropped are named in the report. No pre-existing entry merged into another.
- Confirmed supersessions applied, their IDs retired, and one `RETIRED:` line opened at removal and closed out by 5.1 step 9 — none left `<pending>`. Unresolved contradictions recorded in the top-level changelog section so the next run re-raises them, or reported as "deferred and unrecorded" when scope prevented that. Nothing removed without explicit confirmation.
- Constitution compliance verified for all merged content, and every 2.1 finding reported with its disposition — conflicts resolved in Step 3 or the conflicting item withheld and reported, unmet obligations asked and recorded with no content withheld over one, action-requiring rules reported as unverified rather than flagged, and no missing statement written by this command into any artifact.
- Legacy per-feature blocks left unmodified, and every shared Technical Context field written beside them composed from each contributing feature whose value survived, with one source ref per surviving contributor (a value dropped as the losing side of a scalar conflict is named in the report instead, and a legacy line carrying no ref contributes its value without one).
- Memory directory bootstrapped for every artifact this run's scope will populate, and any artifact whose bootstrap was suppressed by scope named under `## Scoping`.
- Feature spec `**Status**: Draft` updated to `Completed` (if applicable).
- Conflicts either resolved (with user input) or marked with `NEEDS CLARIFICATION` (max 3). This covers the 2.2 and 2.3 kinds; an unresolved **constitution** conflict takes the third disposition instead — the item is withheld and named in the report, since an item that was never written has nowhere to carry a marker.
- Archival Report printed with absolute paths for all changed files, constitution status, and next steps.
- Scoping hints respected — skipped artifacts explicitly noted.
