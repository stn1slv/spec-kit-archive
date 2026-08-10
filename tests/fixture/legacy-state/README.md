# Legacy-state overlay

Memory files in the **v1.1.2 output formats**, holding feature 001's content, for testing the migration rules added in v1.1.3. To use: copy a clean `project/`, replace its `.specify/memory/` content with these three files (keep `constitution.md`), then archive `specs/002-notifications` at full scope.

Deliberate legacy artifacts:

| Artifact | Where | Migration rule under test |
|---|---|---|
| `## Feature 001:` per-feature header block | spec.md | Never rewritten retroactively (0.4: "new seeds only") |
| Unnumbered assumptions | spec.md | AS-### backfill in current order (5.1 step 8), which must not by itself trigger ref upgrades |
| Directory-level `[Source: specs/001-task-manager]` refs on all three edge cases and all assumptions | spec.md | Legacy-ref upgrade only on touched entries: 002 folds into the owner-deactivation edge case (upgrade expected there), the other legacy refs must stay unchanged |
| Per-feature blocks (`### 001 Dependencies`) | plan.md | Not reorganized wholesale; 002 content goes into newly created shared sections; mixed layout reported |
| Old entry header `### Task Manager — 2026-08-09` and bare `**Spec:**` path, chronological position | changelog.md | Untouched entries keep their format; the new 002 entry is prepended in the new format above it |
