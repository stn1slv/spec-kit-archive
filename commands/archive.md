---
description: "Archive a feature specification into main project memory after merge, resolving gaps and conflicts"
scripts:
  sh: ../../scripts/bash/check-prerequisites.sh --json --paths-only
  ps: ../../scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---
Act as the **Chief Software Architect** and **Documentation Maintainer**.
A feature has been merged into the `main` branch. Your goal is to **archive** the feature specification into the main project memory — ensuring completeness, resolving conflicts, closing gaps, and respecting the project constitution.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

### Input Parsing

Parse `$ARGUMENTS` as follows:
- **First token**: feature spec directory path (e.g., `specs/007-invoice-settings`)
- **Remaining tokens**: scope modifiers (optional, space-separated)

**Supported scope modifiers** (if none provided, update all artifacts):
- `--spec-only` — update only `.specify/memory/spec.md`
- `--plan-only` — update only `.specify/memory/plan.md`
- `--changelog-only` — update only `.specify/memory/changelog.md`
- `--agent-only` — update only the agent knowledge file (GEMINI.md / AGENTS.md / CLAUDE.md)

If `$ARGUMENTS` is empty, output `ERROR: No feature spec directory provided. Usage: /speckit.archive.run specs/###-feature-name [--scope-modifier]` and stop.

---

## Step 0: Setup & Validation (Gate)

### 0.1 Resolve Paths

Run `{SCRIPT}` to identify the active feature directory and its artifacts. This script is mandatory for path discovery. If the script is missing, stop and inform the user.

Derive absolute paths for:
- `REPO_ROOT` (from `{SCRIPT}` output)
- `FEATURE_DIR` (from `{SCRIPT}` output)
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

### 0.4 Validate or Bootstrap Memory Directory

Check if `MEMORY_DIR` exists:

**If `MEMORY_DIR` exists**: Read its contents. Note which files are present (`constitution.md`, `spec.md`, `plan.md`, `changelog.md`).

**If `MEMORY_DIR` does not exist**: Create it:
```
mkdir -p MEMORY_DIR
```

**If `MEMORY_DIR/spec.md` does not exist** (first archival):
- If `TEMPLATES_DIR/spec-template.md` exists, copy it as the seed and populate from the feature spec
- Otherwise, create `spec.md` with the feature's spec content as the initial main spec
- Note in the report: "Bootstrapped `.specify/memory/spec.md` from first feature"

**If `MEMORY_DIR/plan.md` does not exist** (first archival):
- If `TEMPLATES_DIR/plan-template.md` exists, copy it as the seed and populate from the feature plan
- Otherwise, create `plan.md` with the feature's plan content as the initial main plan
- Note in the report: "Bootstrapped `.specify/memory/plan.md` from first feature"

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

Read the feature specification and extract:

**From spec.md:**
- User Stories / Integration Scenarios (with priorities and acceptance criteria)
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

Identify entries in main memory that this feature **replaces** rather than extends. Look for:

- The same capability restated with different or incompatible behavior.
- An explicit statement in the feature spec that it replaces, deprecates, or removes prior behavior.
- A rule that narrows or widens an existing one such that both cannot hold at once.

Report each candidate with the evidence quoted:
```
🔄 SUPERSESSION CANDIDATES:
- FR-005 (main) ← superseded by FR-021 (feature)
  Main:    "[quote the existing requirement]"
  Feature: "[quote the replacing requirement]"
  Reason:  [why the new one replaces rather than complements the old one]
```

**This step is detection only — never remove anything here.** Every candidate must be confirmed by the user in Step 3 before 5.1.1 applies it. Overlap alone is not supersession: if both entries can hold at once, it is a consolidation case (5.1 step 9), not a supersession.

---

## Step 3: Clarify (exactly once; max 5 questions)

If conflicts or gaps require human judgment, ask **only questions that materially change scope or correctness**. Skip this step entirely if everything is unambiguous.

**Always ask** if any CRITICAL constitution conflicts were detected — these cannot be auto-resolved.

**Always ask** if any supersession candidates were detected in Step 2.4 — removal is destructive and requires explicit confirmation.

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
| A | Remove all listed | Each removal is recorded in changelog.md; its ID is retired |
| B | Remove none | Main spec keeps both entries; the contradiction is reported, not resolved |
| C | Remove only [IDs] | Confirm a subset; the rest are kept as-is |
| Custom | Provide your own | [How it affects which entries survive] |

**Your choice**: _[Wait for user response]_
```

Treat anything the user does not explicitly confirm as **not** superseded.

If the 5-question budget is exhausted before this question can be asked (for example by CRITICAL constitution conflicts), **do not remove anything**. List every candidate under "Superseded Requirements" in the Step 6 report as "not asked — question budget exhausted" so the user can re-run and decide.

When the user declines a removal, the main spec knowingly retains two conflicting entries. Report each retained contradiction in Step 6 so it is visible rather than silently absorbed later.

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
| `.specify/memory/spec.md` | FR-005 | Remove (superseded by FR-021) |
| `.specify/memory/plan.md` | Dependencies, Project Structure | Append |
| `.specify/memory/changelog.md` | Merged Features Log | New entry |
| `GEMINI.md` | Recent Changes, Known Issues | Append |
```

This gives the user a preview before edits are applied.

---

## Step 5: Archival (Apply Edits)

### Edit Rules
- Use absolute paths for all file references.
- Preserve existing section layout and ordering. Consolidate *within* a section; do not reorganize the document.
- **Consolidate, do not accumulate.** Merge each incoming item into the existing entry that already covers the same ground. Append a new entry only when no equivalent exists. The main spec is one consolidated specification, not a per-feature digest.
- **The surviving text of a merge must preserve every constraint** from all contributing entries. If one entry's wording would lose a condition, limit, or qualifier stated by the other, the two are **not** equivalent — keep them separate. A source ref must never point at an entry whose constraint was dropped.
- Add an **item-level** `[Source: specs/###-feature-name/spec.md -> ID]` traceability ref to each merged entry (e.g. `[Source: specs/007-invoice/spec.md -> FR-012]`). An entry consolidated from several features carries one ref per contributing feature. Never attach a second ref for a feature the entry already cites.
- **Legacy refs**: entries written in the older directory-level form (`[Source: specs/###-feature-name]`) carry no item ID. Upgrade a legacy ref to the item-level form when you touch its entry and the originating item can be identified; otherwise leave it as-is and record it in the item-level form without the `-> ID` suffix.
- Add a **Revision note** (date + reason) to each modified artifact.
- Respect scoping hints — skip artifacts not in scope and explicitly note them.
- **Detect and follow the project's existing ID convention** (FR-XXX, REQ-XXX, Flow1, US-XX, etc.). Continue the sequence from the highest existing ID in main memory. Never reuse or renumber existing IDs.
- **Retired IDs are off-limits.** Before assigning any new ID, read the `**Superseded:**` blocks in `.specify/memory/changelog.md` and collect every ID recorded there. Continue numbering above the highest ID found in **either** the main spec or that retired list, so a retired ID is never reissued even when it was the highest-numbered entry.
- **When consolidating equivalent items, keep the earliest existing ID** and attach the later features' source refs to it. Never renumber the surviving entry.
- **Constitution constraints must be respected** — do not merge content that violates them.

### 5.1 Update Main Specification (`.specify/memory/spec.md`)

Each step below **consolidates** into the existing section rather than appending a new per-feature block.

**Supersession exception (applies to steps 1–7).** An incoming item confirmed in Step 3 as superseding an existing entry is **always added as a new entry with a new ID**. Never fold it into the entry it supersedes: that entry is about to be removed in step 8, and folding would destroy the replacement along with it. This exception overrides the "fold when it states the same capability" rule for these items only.

**Idempotency.** If this feature already has an entry in the Merged Features Log (`changelog.md`), this is a re-run. Update the existing records rather than adding duplicates, and never attach a second source ref for a feature an entry already cites.

1. **Merge User Stories / Integration Scenarios** — fold into an existing story when it covers the same user goal; otherwise add, maintaining priority ordering.
2. **Merge Functional Requirements** — fold into the existing requirement when it states the same capability; otherwise add, continuing from the highest existing ID. Group by domain/module if the spec is large.
3. **Merge Key Entities** — add new entities; extend existing ones with new fields rather than restating the entity.
4. **Merge Edge Cases and Error Handling** — fold cases describing the same failure mode into one entry.
5. **Update Data Flow / Architecture** if the feature changed system data flows.
6. **Merge Success Criteria / Measurable Outcomes** if present. Fold outcomes measuring the same thing into one entry; otherwise continue from the highest existing ID (e.g., SC-XXX).
7. **Merge Assumptions**: add new assumptions under the `## Assumptions` section (if the main spec lacks one, create it after Success Criteria to match the template's section order); skip any already recorded in main memory.
8. **Apply confirmed supersessions** — remove the items confirmed in Step 3 (see 5.1.1 below).
9. **Consolidation pass** — review the entries **this feature added or modified** in the sections above (stories, requirements, entities, edge cases, measurable outcomes, assumptions), together with the existing entries they duplicate. Where two entries state the same thing in different words, merge them into a single entry that keeps the earliest ID and carries a source ref per contributing feature. Leave genuinely distinct entries separate; do not over-merge items that only look similar.

   **Do not sweep the whole document.** Entry pairs unrelated to this feature are out of scope: merging them would change content the Step 4 impact map never previewed.

   **Declined supersessions are exempt.** If the user declined a supersession candidate in Step 3, the entries involved must be left as two separate entries. Merging them would carry out the removal the user just refused. Record the retained contradiction in the Step 6 report so it stays visible.

   **An absorbed ID is a removal.** When a merge makes an ID disappear, retire that ID exactly as 5.1.1 step 3 requires, record it in the changelog `**Superseded:**` block noting it was absorbed by consolidation rather than superseded, and run the same dangling-reference scan (5.1.1 step 4).

#### 5.1.1 Apply Confirmed Supersessions

**Audit gate (check first).** Removal is only permitted when the audit trail can actually be written. If `.specify/memory/changelog.md` is out of scope (for example under `--spec-only`), **skip supersessions entirely**, leave every entry in place, and report them under "Superseded Requirements" as deferred with the reason. Deleting a requirement with no record anywhere is never acceptable.

For each supersession candidate **confirmed by the user in Step 3**:

1. The **replacing** entry has already been merged by steps 1–7 above, under a new main-memory ID — whichever section it belongs to (user story, requirement, measurable outcome, and so on). **Do not merge it again here.** Confirm it is present and note its ID for the changelog entry. The replacement is a new entry in main memory, not an edit of the superseded one.
   - **If the feature removes behavior without replacing it**, there is no replacing entry. That is valid: proceed with the removal and record it as a retirement with no replacement.
   - **ID namespaces differ.** The IDs quoted in Step 2.4 are the *feature's* numbering; the replacement's main-memory ID is usually different. Always record the **main-memory** ID in the changelog and the Step 6 report, never the feature-local one.
2. Remove the superseded entry from `.specify/memory/spec.md`. Do not leave a placeholder, strikethrough, or `[Superseded by: ...]` note — the point is that no stale requirement text remains in the file agents load as context.
   - **If the superseded entry carries source refs from several features** (it is itself the product of an earlier consolidation), do **not** delete the whole entry. Only part of it is superseded. Rewrite it to drop the superseded clause and that feature's ref, keeping the entry, its ID, and the refs of every feature still represented. Record this as a partial supersession.
3. **Retire the removed ID** when the whole entry was removed. It must never be reused or reassigned to a future entry, even though its number is now unused. A partially superseded entry keeps its ID and is not retired.
4. Scan the other memory artifacts for references to the retired ID: `.specify/memory/spec.md` itself (cross-references such as "as specified in FR-005" survive the deletion of their target), `plan.md`, `constitution.md`, and the agent knowledge file. Do not rewrite them automatically — list any dangling references in the Step 6 report so the user can resolve them.
5. Record the removal in the changelog (see 5.4), naming the retired ID and, when there is one, the replacing main-memory ID.

Candidates that were **not** confirmed are left untouched and reported in Step 6. Never remove an entry that the user did not explicitly confirm.

### 5.2 Update Main Plan (`.specify/memory/plan.md`)

1. **Dependencies:** Add new packages (with versions) to "Primary Dependencies" or equivalent section.
2. **Project Structure:** Add new modules/services to the structure tree.
3. **Configuration:** Note new environment variables or config additions.
4. **Routing & Navigation:** Add new routes, endpoints, or wiring.
5. **Testing Strategy:** Add test coverage notes for new components.
6. **Remove from "Future Work"** anything that was just implemented.
7. Ensure plan reflects the *implemented* state.

### 5.3 Update Agent Knowledge File (GEMINI.md / AGENTS.md / CLAUDE.md)

1. Find the project's agent knowledge file (check, in order: `GEMINI.md`, `AGENTS.md`, `CLAUDE.md` in REPO_ROOT).
2. If found, follow the agent-file-template structure and update these sections:

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

Create or update `.specify/memory/changelog.md`:

```markdown
## Merged Features Log

### [FEATURE NAME] — YYYY-MM-DD
**Branch:** [branch-name from plan.md]
**Spec:** specs/###-feature-name

**What was added:**
- [Summary of user stories/scenarios implemented]

**New Components:**
- [Modules/services added]

**Superseded:**
- FR-005 (from specs/003-billing/spec.md) → replaced by FR-022. Reason: [one line]
- FR-008 (from specs/004-export/spec.md) → retired, no replacement. Reason: [one line]
- FR-011 (from specs/002-auth/spec.md) → absorbed into FR-009 by consolidation. Reason: [one line]

**Tasks Completed:** [completed]/[total] tasks
```

Count tasks using the checkbox format: `- [X]` or `- [x]` = completed; `- [ ]` = incomplete. If `tasks.md` does not exist, omit the "Tasks Completed" line.

The **Superseded** block is the audit trail for every ID that left the main spec, whether removed in 5.1.1 or absorbed by the consolidation pass (5.1 step 9). Record one line per removal: the retired ID, the source ref it came from, what replaced it (a **main-memory** ID, "retired, no replacement", or "absorbed into ID"), and a one-line reason. This keeps the removal traceable in project memory without leaving stale requirement text in `spec.md`. Omit the block entirely if nothing was superseded or absorbed.

This block is also read back on later runs: 5.1's ID rules require collecting these retired IDs before assigning any new one, so never rewrite or prune it.

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

## Feature Status
[List spec/plan files whose status was updated from Draft to Completed, or "No status fields found"]

## Bootstrapped
[List any files that were created for the first time, or "None"]

## Constitution Compliance
[Confirm all merged content respects constitution constraints, or list any unresolved CRITICAL conflicts]

## Edits Applied
[Brief summary of each artifact update]

## Conflicts Resolved
[List any conflicts that were resolved and how, or "None"]

## Consolidation
[Entries merged into existing ones, e.g. "FR-012 absorbed the feature's equivalent requirement; now carries 2 source refs". Name any ID that disappeared through absorption. Or "None"]

## Superseded Requirements
[Confirmed removals as `OLD-ID (retired) → replaced by NEW-ID (main-memory ID)`, `OLD-ID (retired, no replacement)`, or `OLD-ID (partially superseded, entry retained)`. Also list:
- candidates detected but declined, and the contradiction each one leaves in the spec
- candidates skipped because the changelog was out of scope (deferred)
- candidates never asked because the question budget was exhausted
- dangling references to retired IDs found in spec.md, plan.md, constitution.md, or the agent file
Or "None"]

## Outstanding Items
[Any remaining `NEEDS CLARIFICATION` markers, or "None"]

## Defaults Applied
[Any decisions made with reasonable defaults instead of asking, or "None"]

## Scoping
[Which artifacts were updated, and which were skipped due to scope modifiers]
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

- All non-conflicting feature content merged into main memory artifacts.
- Merged content consolidated into existing entries where equivalent, each carrying item-level source refs.
- Supersession candidates detected, confirmed by the user, applied, and recorded in the changelog. Nothing removed without explicit confirmation.
- Constitution compliance verified for all merged content.
- Memory directory bootstrapped if this was the first archival.
- Feature spec `**Status**: Draft` updated to `Completed` (if applicable).
- Conflicts either resolved (with user input) or marked with `NEEDS CLARIFICATION` (max 3).
- Archival Report printed with absolute paths for all changed files, constitution status, and next steps.
- Scoping hints respected — skipped artifacts explicitly noted.
