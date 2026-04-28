# Changelog

All changes to this project are documented here. This file is append-only — do not edit or delete existing entries. Add new entries at the top, inside the `[Unreleased]` section. When a version is released, the `[Unreleased]` section is moved into a versioned block.

Follow the format strictly. Every entry must be traceable to a requirement, task, or issue.

---

## Format Reference

```
## [VERSION] — YYYY-MM-DD — Release Title (optional)

### Added
- `path/to/file.tsx`: Description of what was added and why.

### Changed
- `path/to/file.tsx`: Description of what changed, what it was before, and why it changed.

### Fixed
- `path/to/file.tsx`: Description of the bug, what caused it, and how it was fixed.

### Removed
- `path/to/file.tsx`: What was removed and why. (Moved to `bin/` — not deleted.)

### Security
- Description of any security-relevant change.

### AI-Assisted
- Tool and model used, what it produced, and what human review / modification was applied.
```

**Category rules**:
- `Added` — New features or files.
- `Changed` — Changes to existing behavior, API, or structure.
- `Fixed` — Bug fixes.
- `Removed` — Deprecated or removed functionality (files moved to `bin/`).
- `Security` — Any change with a security implication.
- `AI-Assisted` — AI-generated or AI-assisted changes, per `ai_guidelines.md`.

---

## [Unreleased]

> Add all in-progress changes here as they are made. Do not leave this section empty — if you made a change, it belongs here.

---

## [0.1.0] — YYYY-MM-DD — Initial project setup

### Added
- Project scaffolded with base documentation suite:
  - `AGENT_DIRECTIVE.md` — Agent execution standard.
  - `MASTER_PROMPT.md` — Universal planning protocol.
  - `docs/architecture.md` — Architecture reference.
  - `docs/requirements.md` — Requirements tracking.
  - `docs/testing.md` — Testing standards and commands.
  - `docs/accessibility.md` — Accessibility standards.
  - `docs/ai_guidelines.md` — AI usage policy.
  - `docs/security.md` — Security requirements.
  - `docs/checklist.md` — Quality gate checklists.
  - `docs/feature_file_correlations.md` — Feature-to-file mapping.
  - `docs/system_prompt.md` — Agent system prompt.
  - `CHANGELOG.md` — This file.
  - `README.md` — Project overview and quick-start.
- `.github/prompts/` — Reusable prompt files for common agent tasks.
- `bin/` and `_backups/` directories established.
- `.env.example` created with all required variable names documented.
