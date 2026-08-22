# Agent-context defaults overlay

Exercises Step 5.3 **branch (b)**: the defaults lookup that `agent-context` itself performs when its config names nothing.

This is the state a fresh `specify extension add agent-context` leaves behind — `context_file: ""` and `context_files: []` — so it is the **normal** case for an installed extension, not an edge case. In it, that extension does not guess filenames: it reads the integration key from `.specify/init-options.json` and looks it up in its own `agent-context-defaults.json`.

| File | Purpose |
|------|---------|
| `.specify/init-options.json` | Records `"integration": "copilot"` |
| `.specify/extensions/agent-context/agent-context-config.yml` | Present but naming nothing, ending branch (a) |
| `.specify/extensions/agent-context/agent-context-defaults.json` | Trimmed copy of the real map; `copilot` maps to `.github/copilot-instructions.md` |
| `.github/copilot-instructions.md` | The project's real anchor, which the last-resort probe never looks for |

To use: copy a clean `project/`, copy these files over it, **delete the root `AGENTS.md`** that `project/` ships, and archive `specs/001-task-manager`.

Deleting `AGENTS.md` is the point of the case. With it present, a run that skips the defaults lookup still finds *a* file and looks like it worked; without it, skipping the lookup finds nothing at all and the miss is unambiguous.

Expected:

- Discovery reports **branch (b)**, the defaults lookup, and resolves exactly one target: `.github/copilot-instructions.md`.
- That file is updated. Its `Recent Changes` bullet is in the legacy basename form, so it must be matched and upgraded in place to `specs/001-task-manager`, never duplicated.
- Nothing is created. No `AGENTS.md`, `CLAUDE.md` or `GEMINI.md` appears.
- A report claiming the last-resort probe, or one that skips 5.3 for want of a file, is a miss: both mean the defaults lookup did not happen.
