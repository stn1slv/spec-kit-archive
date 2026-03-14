# Spec-Kit Archive

A Spec-Kit extension to archive merged features into the main project memory.

## Overview

The `speckit.archive.run` command is a **Post-Merge Archival** tool designed to consolidate finalized feature specifications, plans, and technical debt into the project's canonical memory (`.specify/memory/`).

This extension acts as the "Outer Loop" of the Double-Loop Parity framework: it ensures that after a PR is merged, the project remembers it correctly.

## Features

- **Lifecycle Separation**: Operates purely on merging feature-level knowledge into project-level memory.
- **Ecosystem Consistency**: Uses the core Spec-Kit `check-prerequisites.sh` script for reliable path resolution (handles monorepos and nested structures).
- **Traceability**: Preserves `[Source: specs/###-feature-name]` tags and revision notes in the main memory artifacts.
- **Reporting**: Mandates absolute paths in the final Archival Report, ensuring logs are always useful regardless of your CWD.

## Installation

You can install this extension via the Spec-Kit CLI:

```bash
specify extension add archive --from https://github.com/stn1slv/spec-kit-archive/archive/refs/tags/v1.0.0.zip
```
*(Note: Replace `v1.0.0` with the latest release version)*

## Usage

```bash
/speckit.archive.run specs/###-feature-name
```

You can optionally restrict the scope of the updates:
- `--spec-only` — update only `.specify/memory/spec.md`
- `--plan-only` — update only `.specify/memory/plan.md`
- `--changelog-only` — update only `.specify/memory/changelog.md`
- `--agent-only` — update only the agent knowledge file

## Workflow

1.  **Run `check-prerequisites.sh`** to find the active feature artifacts.
2.  **Verify Constitution Compliance**: Check that feature implementations don't violate project "MUSTs".
3.  **Perform Impact Map**: Ask up to 5 clarifying questions before proceeding.
4.  **Archive Data**: Append entities, requirements, dependencies, and architecture notes into the main memory.
5.  **Output Report**: Provide a comprehensive status report indicating changed files and what you should do next.
