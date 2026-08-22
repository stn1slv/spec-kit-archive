# Agent-context overlay

Exercises Step 5.3's context-file discovery (v1.3.0, change C) against the layout the first-party `agent-context` extension actually produces. A clean `project/` has none of this: it carries a single root `AGENTS.md` and no `agent-context` config, which is the **fallback** branch. This overlay is the **config** branch.

What it sets up, and the trap each piece carries:

| File | Purpose |
|------|---------|
| `.specify/extensions/agent-context/agent-context-config.yml` | Names **four** targets in `context_files` and **custom markers** (`<!-- TEAM CONTEXT START/END -->`, not the `<!-- SPECKIT ... -->` default) |
| `AGENTS.md` | Has a managed block **in the middle**, with writable prose above and below it. Names only `002-notifications`, in the legacy basename form, so archiving 001 is a genuine first write to this file |
| `docs/agent/CLAUDE.md` | A target at a **nested path**, proving entries are project-relative and need not sit at the root. Also names only `002-notifications` |
| `GEMINI.md` | **Entirely enclosed** in a marker block — no writable region at all |
| *(not created)* `QWEN.md` | Configured but **missing** from disk |

To use: copy a clean `project/`, then copy these files over it (creating `docs/agent/`), and archive `specs/001-task-manager`.

Expected, per Step 5.3:

- `AGENTS.md` and `docs/agent/CLAUDE.md` are both written, each getting the same section set, with nothing placed inside the `<!-- TEAM CONTEXT ... -->` block in `AGENTS.md`.
- Neither file names `001-task-manager` before the run. **This matters**: an earlier version of this overlay carried a `- specs/001-task-manager:` bullet in both files, which made the command's idempotency rule fire before either case could demonstrate a write, and round 6's Case R1 was invalidated by it. The pre-existing `002-notifications` bullet is in the **legacy basename form**, so it also has to survive the run untouched: only a bullet this run touches is upgraded.
- `GEMINI.md` and `QWEN.md` are **skipped and named in the report** — one for having no region outside its markers, one for not existing. Neither is created, and neither stops the run.
- The report's Path Resolution says discovery came from the config and names the target count; Changed Files carries one row per file written.

Note the deliberate trap in the fallback order: `GEMINI.md` exists at the root and is the *first* name the fallback probe would try, so a run that ignores the config and falls back would write to the one file that has no writable region. The two branches cannot be confused by accident.
