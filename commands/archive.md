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

If **several** scope modifiers are supplied, the scope is their **union** — `--spec-only --changelog-only` updates both `spec.md` and `changelog.md` and nothing else. "Only" bounds the whole set, not each flag individually.

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

**Bootstrapped spec (applies to 2.2 and 2.4).** If `.specify/memory/spec.md` was bootstrapped from this same feature in Step 0.4, the main spec *is* the feature's content. Comparing the feature against its own copy would flag every requirement as a collision and every restatement as a supersession, so **skip 2.2 and 2.4 entirely** in that case. A feature cannot collide with, or supersede, itself.

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

**Skip this step** if the spec was bootstrapped from this feature (see the Step 2 preamble) or if `spec.md` is not in scope.

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
- Add an **item-level** `[Source: specs/###-feature-name/spec.md -> ID]` traceability ref to each merged entry (e.g. `[Source: specs/007-invoice/spec.md -> FR-012]`). An entry consolidated from several features carries one ref per contributing feature. Never attach a second ref for a feature the entry already cites.
- **Legacy refs**: entries written in the older directory-level form (`[Source: specs/###-feature-name]`) carry no item ID. When you touch such an entry, upgrade the ref to `[Source: specs/###-feature-name/spec.md -> ID]` if the originating item can be identified, or to `[Source: specs/###-feature-name/spec.md]` if it cannot. Do not modify legacy refs on entries this feature does not touch.
- Add a **Revision note** (date + reason) to each modified artifact.
- Respect scoping hints — skip artifacts not in scope and explicitly note them. **Out of scope means not written, never not read**: artifacts outside the scope are still read when a rule requires it (for example collecting retired IDs or checking for a prior run in `changelog.md`).
- **Detect and follow the project's existing ID convention** (FR-XXX, REQ-XXX, Flow1, US-XX, etc.). Continue the sequence from the highest existing ID in main memory. Never reuse or renumber existing IDs.
- **Retired IDs are off-limits.** Before assigning any new ID, read `.specify/memory/changelog.md` **if that file exists** (on a first archival it does not yet) and collect the ID immediately following each `RETIRED:` marker. **Collect only that ID** — the rest of the line names the live replacement and must be ignored. Continue numbering above the highest ID found in **either** the main spec or that retired list, so a retired ID is never reissued even when it was the highest-numbered entry.
- **When consolidating equivalent items, keep the earliest existing ID** and attach the later features' source refs to it. Never renumber the surviving entry.
- **Constitution constraints must be respected** — do not merge content that violates them.

### 5.1 Update Main Specification (`.specify/memory/spec.md`)

Each step below **consolidates** into the existing section rather than appending a new per-feature block.

**Removals come first.** Step 1 applies the confirmed supersessions, before any merging. Nothing can then be folded into an entry that is about to be deleted.

**First run.** If `spec.md` was bootstrapped from this same feature in Step 0.4, its content is already the feature's content. Do not merge the feature into its own copy: skip the merging in steps 2–8 and only attach source refs to the bootstrapped entries. (Step 2.4 has already been skipped for the same reason, so step 1 has nothing to apply.)

**Idempotency.** If this feature already has an entry in the Merged Features Log (`changelog.md`), this is a re-run. Update that entry in place rather than appending a second one, and never attach a source ref an entry already cites.

1. **Apply confirmed supersessions** — see 5.1.1 below. This happens before everything else.
2. **Merge User Stories / Integration Scenarios** — fold into an existing story when it covers the same user goal; otherwise add, maintaining priority ordering.
3. **Merge Functional Requirements** — fold into the existing requirement when it states the same capability; otherwise add, continuing from the highest existing ID. Group by domain/module if the spec is large.
4. **Merge Key Entities** — add new entities; extend existing ones with new fields rather than restating the entity.
5. **Merge Edge Cases and Error Handling** — fold cases describing the same failure mode into one entry.
6. **Update Data Flow / Architecture** if the feature changed system data flows.
7. **Merge Success Criteria / Measurable Outcomes** if present. Fold outcomes measuring the same thing into one entry; otherwise continue from the highest existing ID (e.g., SC-XXX).
8. **Merge Assumptions**: add new assumptions under the `## Assumptions` section (if the main spec lacks one, create it after Success Criteria to match the template's section order); skip any already recorded in main memory.
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

   **Which ID the replacement reference takes.** Whichever main-memory ID the feature's replacing item ends up under once steps 2–8 finish — a **new** ID if it was added as a new entry, or the **existing** entry's ID if it folded into one (earliest ID wins, so that entry keeps its original number). The IDs quoted in Step 2.4 are the *feature's* local numbering and must never appear here. Write `→ no replacement` straight away when the feature retires the behavior outright; that case has nothing to wait for.

   **5.1 step 9 closes these lines.** Completing a line you opened during this run is part of writing it, not a rewrite; the append-only rule in 5.4 governs lines from *previous* runs. No `<pending>` marker may survive the end of 5.1.

   If `changelog.md` has no entry for this feature yet, create it now using the 5.4 template; 5.4 will then update that same entry rather than adding a second one.
4. Scan for references to the retired ID in `.specify/memory/spec.md` itself (cross-references such as "as specified in FR-005" survive the deletion of their target), `plan.md`, `constitution.md`, and the agent knowledge file. Do not rewrite them — list any dangling references in the Step 6 report.

Candidates the user did not confirm are left untouched, recorded in the top-level `## Unresolved Contradictions` section of the changelog, and reported in Step 6. Never remove an entry without explicit confirmation.

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

This is a **working list, not an audit trail**, which is why it is deliberately kept out of the per-feature entries: it is meant to shrink, and resolving an item should never mean editing a past feature's record. Step 2.4 reads this one section on later runs and re-raises each pair while both entries are still present and still conflicting, so a declined contradiction gets another chance instead of becoming invisible.

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
[Feature items folded into existing entries, e.g. "this feature's equivalent requirement folded into FR-012, which now carries 2 source refs". Or "None"]

## Superseded Requirements
[Confirmed removals as `OLD-ID (retired) → replaced by NEW-ID` or `OLD-ID (retired, no replacement)`. Also list:
- candidates left unresolved, and the contradiction each leaves in the spec (these are also written to changelog.md and re-raised next run)
- **deferred and unrecorded** — candidates deferred because the supersession gate was closed *and* the contradiction could not be written to changelog.md. Name the scope responsible and state plainly that these will **not** be raised again automatically; recommend a re-run at full scope
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

- All non-conflicting feature content merged into main memory artifacts.
- Feature content folded into existing entries where equivalent, each carrying item-level source refs. No pre-existing entry merged into another.
- Confirmed supersessions applied, their IDs retired, and one `RETIRED:` line opened at removal and closed out by 5.1 step 9 — none left `<pending>`. Unresolved contradictions recorded in the top-level changelog section so the next run re-raises them, or reported as "deferred and unrecorded" when scope prevented that. Nothing removed without explicit confirmation.
- Constitution compliance verified for all merged content.
- Memory directory bootstrapped if this was the first archival.
- Feature spec `**Status**: Draft` updated to `Completed` (if applicable).
- Conflicts either resolved (with user input) or marked with `NEEDS CLARIFICATION` (max 3).
- Archival Report printed with absolute paths for all changed files, constitution status, and next steps.
- Scoping hints respected — skipped artifacts explicitly noted.
