# Legacy-state overlay

Memory files simulating **pre-v1.1.3 output**, holding feature 001's content, for testing the migration rules added in v1.1.3. It is a **synthetic composite**, not a faithful copy of any single version's output: v1.1.2's own baseline runs emitted item-level refs and clean headers, while the per-feature header and per-feature plan blocks come from field-observed codex runs (BASELINE-v1.1.2.md, F4/F6). The composite deliberately stacks every known legacy artifact into one state so a single run exercises every migration rule.

To use: copy a clean `project/`, replace its `.specify/memory/` content with these three files (keep `constitution.md`), then archive `specs/002-notifications` at full scope.

Deliberate legacy artifacts:

| Artifact | Where | Migration rule under test |
|---|---|---|
| `## Feature 001:` per-feature header block (carried verbatim from 001's spec header) | spec.md | Never rewritten retroactively (0.4: "new seeds only") |
| Unnumbered assumptions with directory-level refs | spec.md | AS-### backfill in current order (5.1 step 8), which must not by itself trigger ref upgrades |
| Directory-level `[Source: specs/001-task-manager]` refs on **FR-002**, all three edge cases, and all assumptions | spec.md | Legacy-ref upgrade fires only on touched entries. FR-002 is the **guaranteed** fold target (002's FR-003 folds into it per T6, confirmed in both prior baselines), so its ref upgrade is deterministic; the edge-case fold is agent-judgment (J declined it) and the other legacy refs must stay unchanged |
| Per-feature blocks (`### 001 Technical Context` with scalar fields, `### 001 Dependencies` etc.), refs in mixed forms | plan.md | Not reorganized wholesale; 002 content goes into newly created shared sections; scalar merging across the legacy layout; mixed layout reported |
| Old entry header `### Task Manager — 2026-08-09` and bare `**Spec:**` path | changelog.md | Untouched entries keep their format; the new 002 entry is prepended in the new format above it |

**Version note.** Case J in `BASELINE-v1.1.3.md` ran against the **original** overlay (FR-002 carried an item-level ref, plan.md had no Technical Context or refs, spec header said `Status at archive`). The overlay was revised afterward per the PR #9 review to make the legacy-ref trap deterministic and add plan-side scalar coverage; the next J-style run exercises the revised state.
