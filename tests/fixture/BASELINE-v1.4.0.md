# Baseline: v1.4.0 (branch `feat/v1.4.0`) against the fixture

Expectations pre-registered in `43e65ee` (round 7) and `8c185ae` (Case C3), **before** the runs they cover. The command under test is `commands/archive.md` with SHA-256 prefix `e007f105efd902b0`. Fresh agents, one per case, each given only that file and its own working copy, and forbidden from reading `EXPECTATIONS.md`, the baselines, the CHANGELOG or git history. Every claim below was checked against the files each run wrote, not only against what the run reported.

| Case | Covers | Result |
|---|---|---|
| H1 | hook without `enabled`; disabled after-hook; new changelog title; skipped 2.5 prints no numbers | Pass |
| H2 | unreadable `extensions.yml` | Pass, one sub-check unverifiable (below) |
| L6b | retired items stay retired on re-archival | Pass |
| M2 | item-level refs in new `RETIRED:` lines | Not exercised (below) |
| C3 | item-level refs in new `RETIRED:` lines, explicit replacement | Pass |
| R4 | already-merged agent files are still completed | Pass |
| Q7 | existing directory outside `specs/` | Pass |
| Q6 | token naming no directory keeps rule 4's error | Pass |
| Ctl | control, `004` on clean `project/` | Pass |

## What the round confirmed

**Hooks follow core.** H1 printed the before-hook with no `enabled` field as an Optional Pre-Hook block carrying `Command:`, `Description:`, `Prompt:` and `To execute:`, showed `speckit.demo.notify` as an Optional Hook after archival, and left out the `enabled: false` hook. H2 told the user that `.specify/extensions.yml` could not be read, quoted the parser error, said no hooks (mandatory ones included) were checked, and archived 001 normally (FR-001 to FR-006 on disk).

**Retired items stay retired.** L6b is the L6 run that found the defect in round 6. This time 001's `FR-004` and `SC-003` were not re-added: the live IDs are `FR-001..003, FR-005..009` and `SC-001, 002, 004, 005`, with no `FR-010` or `SC-006`, and no new Unresolved Contradictions line. Both legacy file-only `RETIRED:` lines matched: `FR-004` because the incoming text contradicts the named replacement `FR-009`, and `SC-003` because its line says `no replacement`. Both are listed under Superseded Requirements as already retired, and no question was asked. T45 held: 001's legacy agent bullet was upgraded in place.

**New `RETIRED:` lines carry item-level refs.** C3 archived 002 on top of a Case A end state and confirmed removals. `changelog.md` gained `RETIRED: FR-004 (from specs/001-task-manager/spec.md -> FR-004) → replaced by FR-009` and `RETIRED: SC-003 (from specs/001-task-manager/spec.md -> SC-003) → no replacement`, with no `<pending>`.

**Idempotency per artifact.** In R4 both writable agent files already named `specs/001-task-manager`. Neither gained a second bullet, and both were still completed: each gained 001's stack under Active Technologies, and `docs/agent/CLAUDE.md` gained the missing Project Structure section. `GEMINI.md` was unchanged and `QWEN.md` was not created.

**Outside `specs/`.** Q7 stopped with `ERROR: 'features/001-outside' lies outside specs/. Only feature directories under REPO_ROOT/specs can be archived.` Q6 still stopped with rule 4's `does not resolve to exactly one feature directory`. Both working copies were byte-identical before and after (SHA-256 over every file).

**Ctl unchanged in content.** `FR-001..003`, `3/5` tasks, `Bugs addressed: BUG-001`, Status flipped to Completed, the struck edge case carried with its `~~` markup, no `**Bugfix**:` line archived, BUG-002's root cause in Known Issues, 001's legacy bullet untouched, the last-resort probe reported, and the script's 002 named beside the archived 004 with no walk-up fallback. The two expected report differences appeared: `changelog.md` opens with `# Changelog`, and Consolidation gives only the skip reason.

## Deviations and limits

- **M2 did not exercise the new format.** Its runner judged main `FR-008` against 003's `FR-002` partial under 2.4's whole-or-partial procedure, asked no supersession question, and wrote no `RETIRED:` line. Case M's expectation (a confirmed FR-008 supersession) dates from v1.2.1, before v1.2.2 added that procedure, and M had not been re-run since. The judgment is defensible ("at most one daily summary" and "one reminder per overdue task per day" can both hold) and the procedure is unchanged in this release, so this is recorded as a stale expectation rather than a regression. C3 was added to cover the format.
- **H2's final report was not delivered.** A safety classifier stopped the runner's reply before the Archival Report. The parse-error notice and the archived files were verified from the transcript and the disk; the report's "bugfix extension: unknown" line could not be checked.
- **L6b rewrote one bullet of 001's existing changelog entry**: "owner-deactivation move to the team backlog" became "reassignment of a deactivated owner's tasks to the team lead". The `archived-state` overlay predates the v1.2.2 fixture correction to 001's FR-006, so the update brings the entry in line with the feature. This is the "an entry the feature changed is updated" half of the new idempotency rule.
- **Em dashes.** A local write hook in the test environment rejects the em dash character, so runners wrote a spaced hyphen where templates show an em dash (revision notes, changelog headers, some copied bullets). This is a harness artefact, not command behaviour.

## Install smoke test

`specify extension add --dev` against spec-kit 1.0.13 (installed via `uvx` from the `v1.0.13` tag):

- `claude`, `--script sh`: registered as `.claude/skills/speckit-archive-run/SKILL.md`; `{SCRIPT}` resolved to `.specify/scripts/bash/check-prerequisites.sh --json --paths-only`; no `__SPECKIT_COMMAND_*__` token left; references render as `/speckit-archive-run`.
- `codex`, `--script py` (skills): registered as `.agents/skills/speckit-archive-run/SKILL.md`; `{SCRIPT}` resolved to `python3 .specify/scripts/python/check_prerequisites.py --json --paths-only`; references render as `$speckit-archive-run`.
- `generic`: not registered on 1.0.13, which is core behaviour fixed upstream by #4785 after that release. On upstream `main` it registers as `.myagent/commands/speckit.archive.run.md` with no token left.
