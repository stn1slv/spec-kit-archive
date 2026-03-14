# Changelog

All notable changes to the Archive extension will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-14

### Added

- Initial release of the Archive extension
- Command: `/speckit.archive.run` — post-merge archival of feature specs into project memory
- Merges user stories, functional requirements, entities, and architecture into `.specify/memory/spec.md`
- Updates dependencies, project structure, and routing in `.specify/memory/plan.md`
- Updates agent knowledge files (GEMINI.md / AGENTS.md / CLAUDE.md)
- Appends to `.specify/memory/changelog.md` with task completion counts
- Constitution compliance enforcement before merging
- Memory directory bootstrapping on first archival
- Feature spec status update (`Draft` → `Completed`)
- Scope modifiers (`--spec-only`, `--plan-only`, `--changelog-only`, `--agent-only`)
- Extension hook support (`before_archive`, `after_archive`)
- Archival Report with absolute paths and traceability tags
