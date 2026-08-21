# Bug-extension overlay

Exercises the **repo-level** bug layout that the first-party `bug` extension writes: `.specify/bugs/<slug>/{assessment,fix,test}.md`, rather than the feature-scoped `specs/###/bugs/BUG-###.md` a clean `project/` already carries.

**These report files mirror the real templates**, field for field and heading for heading, as emitted by `extensions/bug/commands/speckit.bug.{assess,fix,test}.md` upstream: `assessment.md` carries `**Slug**`, `**Created**`, `**Source**`, `**Verdict**`, `**Severity**` and a `## Root Cause Hypothesis`; `fix.md` carries `**Status**: applied | partial | not-applied`; `test.md` carries `**Result**`. This matters more than it looks: an earlier draft of this overlay invented `**Type**`, `**Status**` and `## Root Cause Analysis` headings in `assessment.md`, which made the command's rules pass against a format no tool emits. A fixture that invents its own input cannot detect that the rules read the wrong thing.

To use: copy a clean `project/`, then copy these files over it, and archive `specs/004-attachments`. Both layouts are then present in the same run, which is the point — 004 keeps its own `bugs/BUG-001.md` and `bugs/BUG-002.md`.

## What each piece traps

| Path | Attribution | Expected classification |
|------|-------------|------------------------|
| `.specify/bugs/thumbnail-orientation/` | The `**Bugfix**:` annotation on FR-004 in the patched `specs/004-attachments/spec.md` names the slug | **addressed**; slug appears verbatim in `**Bugs addressed:**` |
| `.specify/bugs/attachment-quota-drift/` | The annotation on FR-005 names it, but its `fix.md` says `**Status**: not-applied` | **addressed** on the annotation's authority, **and** the status discrepancy named under `## Outstanding Items` |
| `.specify/bugs/report-timezone/` | No annotation names it | Not this feature's: **never opened**, not classified, not listed — only counted |
| `specs/004-attachments/spec.md` | Patched to carry both annotations as FR-004 and FR-005 | Annotations are metadata, never archived as requirement text |
| `.specify/extensions.yml` | `installed` names the bare id `bug` | Exercises 0.6's "is" branch, which no earlier case has run |

Attribution is annotation-only. The real `assessment.md` template carries **no field naming a feature**, so there is no honest way to attribute a report by reading it — which is also why attribution must cost no file read: a slug *is* its directory name.

## What must not happen

Beyond the header regions and `## Root Cause Hypothesis`, nothing is read. `fix.md`'s body is never read past its header fields, and `test.md` is never opened. Each carries a deliberately requirement-shaped `MUST` sentence so a leak is greppable in the archived memory:

- `thumbnail-orientation/fix.md` — "normalize EXIF orientation"
- `thumbnail-orientation/test.md` — "reject any image whose orientation tag cannot be parsed"
- `attachment-quota-drift/fix.md` — "enforce the per-team storage quota at commit time"

None of those phrases may appear anywhere in `.specify/memory/` after the run. The root-cause hypotheses **may** appear, but only under the agent context file's Known Issues, titled by slug.

`report-timezone` must not be opened at all, so its root-cause hypothesis must not appear either — a Known Issues entry mentioning report timezones is proof the attribution rule was skipped.

Counts matter too: the report must state that `.specify/bugs/` holds three slugs and that two were attributed.
