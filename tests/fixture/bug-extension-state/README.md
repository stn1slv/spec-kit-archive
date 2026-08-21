# Bug-extension overlay

Exercises the **repo-level** bug layout (v1.3.0, change D) that the first-party `bug` extension writes: `.specify/bugs/<slug>/{assessment,fix,test}.md`, rather than the feature-scoped `specs/###/bugs/BUG-###.md` a clean `project/` already carries.

To use: copy a clean `project/`, then copy these files over it, and archive `specs/004-attachments`. Both layouts are then present in the same run, which is the point — 004 keeps its own `bugs/BUG-001.md` and `bugs/BUG-002.md`.

## What each piece traps

| Path | Attribution | Expected classification |
|------|-------------|------------------------|
| `.specify/bugs/thumbnail-orientation/` | **Channel (a)** — the `**Bugfix**:` annotation on FR-004 in the patched `specs/004-attachments/spec.md` names the slug | **addressed**; its slug appears in the changelog's `**Bugs addressed:**` line |
| `.specify/bugs/attachment-quota-drift/` | **Channel (b)** — its own `**Feature**:` header names `specs/004-attachments`; no annotation anywhere | **unverified**, *despite `Status: Fixed`*; must **not** reach `**Bugs addressed:**` |
| `.specify/bugs/report-timezone/` | **Neither** — no annotation, no feature header; it is about the reporting feature | Not this feature's: not read, not classified, not listed |
| `specs/004-attachments/spec.md` | Patched to carry the channel-(a) annotation as a new FR-004 | The annotation is metadata, never archived as requirement text |
| `.specify/extensions.yml` | `installed` names the bare id `bug` | Exercises 0.6's "is" branch, which no earlier case has run |

## What must not happen

`fix.md` and `test.md` are **never read** beyond a `## Root Cause Analysis`. Each carries a deliberately requirement-shaped `MUST` sentence so a leak is greppable in the archived memory:

- `thumbnail-orientation/fix.md` — "normalize EXIF orientation"
- `thumbnail-orientation/test.md` — "reject any image whose orientation tag cannot be parsed"
- `attachment-quota-drift/fix.md` — "enforce the per-team storage quota at commit time"

None of those phrases may appear anywhere in `.specify/memory/` after the run. The root-cause analyses **may** appear, but only under the agent context file's Known Issues, titled by slug.

Counts matter too: the report must state that `.specify/bugs/` holds three reports and that two were attributed, so `report-timezone` is visibly excluded rather than silently absent.
