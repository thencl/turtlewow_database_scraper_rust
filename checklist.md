# Checklist

This document contains all checklists used in this project's quality gates. Every checklist is mandatory at its defined trigger point. A task, PR, or deployment is not complete until its corresponding checklist is fully checked. No exceptions.

---

## Pre-Commit Checklist

Run before every commit. This is the minimum bar for committing anything.

### Code Quality
- [ ] Code compiles cleanly with zero errors and zero warnings.
- [ ] No new lint errors or rule violations are introduced.
- [ ] No dead imports, unused variables, or commented-out code is committed.
- [ ] No magic numbers or magic strings — named constants are used.
- [ ] No placeholder, stub, or TODO implementation is present anywhere in committed code.
- [ ] All async operations handle loading, success, and error states.
- [ ] Error states are caught and handled explicitly — nothing is swallowed silently.

### Testing
- [ ] Unit tests pass for all directly affected modules.
- [ ] New tests were added for new functionality.
- [ ] A regression test was added if this commit fixes a bug.
- [ ] No tests were skipped or deleted to make the suite pass.
- [ ] Test coverage has not decreased from the previous commit.

### Safety
- [ ] A backup was created in `_backups/` before editing any file over 50 lines.
- [ ] No secrets, credentials, API keys, or `.env` values are present in committed files.
- [ ] Deprecated files were moved to `bin/` — not deleted.

### Documentation
- [ ] A changelog entry has been drafted for this change.
- [ ] Inline documentation (JSDoc / rustdoc) is present for new public functions.
- [ ] If architecture changed: `architecture.md` has been updated.
- [ ] If requirements changed: `requirements.md` has been updated.
- [ ] If a new feature was added: `feature_file_correlations.md` has been updated.

---

## Pre-PR Checklist

Run after all commits are staged and before opening a pull request.

### Completeness
- [ ] Every acceptance criterion in the linked requirement is fully implemented.
- [ ] No deferred or partially implemented items — everything in this PR is done.
- [ ] The definition of done (from `MASTER_PROMPT.md`) is satisfied for every changed item.

### Code Review Readiness
- [ ] PR title clearly describes what changed and why.
- [ ] PR description includes: problem being solved, solution approach, files changed, and how to test.
- [ ] Screenshots, recordings, or log output are included if UI or behavior changed visually.
- [ ] Related documentation links are listed in the PR description.
- [ ] The diff has been self-reviewed — no accidental debug code, temporary files, or unrelated changes included.

### Quality Gates
- [ ] All CI checks pass (lint, type check, tests, build).
- [ ] Accessibility checklist from `accessibility.md` is completed for any UI change.
- [ ] Security checklist from `security.md` pre-PR section is completed for any security-relevant change.
- [ ] `npm audit` / `cargo audit` shows no new high or critical vulnerabilities.

### Rust / Tauri Projects
- [ ] `cargo tauri dev` runs cleanly on the developer's machine.
- [ ] All new Tauri commands are registered in `main.rs` / `lib.rs` and documented.
- [ ] Tauri allowlist changes are minimal, documented, and reviewed.
- [ ] No Docker or hosted environment was required to test this change — everything works locally.

---

## Pre-Deploy Checklist

Run before deploying to any non-local environment. For Rust/Tauri projects, this applies when the project reaches confirmed shipping status and a distribution build is being prepared.

### Build
- [ ] A clean production build completes without errors.
- [ ] The build artifact has been smoke tested locally before distribution.
- [ ] Build output contains no development-only code, debug flags, or source maps (unless intentional).

### Environment
- [ ] All required environment variables are documented in `.env.example`.
- [ ] Environment variables are verified in the target environment.
- [ ] No hardcoded environment-specific values are present in the build.

### Data and Schema
- [ ] Database migrations are tested on a copy of real data.
- [ ] Migrations are reversible — a rollback path exists and is tested.
- [ ] No breaking schema changes are deployed without a migration.

### Security
- [ ] Full `npm audit` / `cargo audit` run — no high or critical issues.
- [ ] Secrets are confirmed rotated if any were ever accidentally exposed.
- [ ] All new permissions are documented and reviewed.

### Rollback
- [ ] A rollback procedure is written and verified before deploying.
- [ ] The previous stable build artifact is retained and deployable.
- [ ] Database migration rollback scripts are staged and tested.

### Post-Deploy Validation Steps
- [ ] Application starts and serves the homepage without errors.
- [ ] Critical P0 flows verified manually in the deployed environment.
- [ ] No new console errors or exceptions in the deployed environment.
- [ ] Analytics and logging are confirmed active and receiving events.
- [ ] The changelog has been updated to reflect the deployed version.

---

## Accessibility Checklist

Run for every PR that includes any UI change.

- [ ] Every new interactive element is reachable by keyboard in logical tab order.
- [ ] Every new interactive element is operable by keyboard (Enter / Space for buttons).
- [ ] Visible, high-contrast focus styles are present on every interactive element.
- [ ] No information is conveyed by color alone.
- [ ] Text contrast meets WCAG 2.1 AA (4.5:1 for normal text, 3:1 for large text and UI elements).
- [ ] All images have meaningful `alt` text or `alt=""` if decorative.
- [ ] All form inputs have persistent visible labels (not only placeholder text).
- [ ] Validation errors are text, adjacent to the field, and announced via `aria-live`.
- [ ] ARIA roles and labels are present and correct for all custom interactive elements.
- [ ] Modal dialogs trap focus while open and return focus to the trigger on close.
- [ ] All animations respect `prefers-reduced-motion`.
- [ ] Lighthouse accessibility score is 90 or above.
- [ ] No critical or serious axe-core violations.
- [ ] Manually tested with VoiceOver (macOS) or NVDA (Windows) if the change affects interactive elements.

---

## Security Checklist

Run for every PR that touches input handling, file system access, authentication, permissions, or external communication.

- [ ] All new user input is validated at the boundary (type, format, length, range).
- [ ] All new file system operations validate paths before execution.
- [ ] No secrets, credentials, or tokens are present in any committed file.
- [ ] New third-party dependencies have been reviewed for security and maintenance.
- [ ] Error messages do not expose stack traces, internal paths, or system details.
- [ ] Authentication and session logic, if touched, has been reviewed.
- [ ] Tauri IPC commands, if added, validate all arguments before acting.
- [ ] Tauri allowlist changes are minimal and documented.
- [ ] No new permissions are requested without documentation and justification.

---

## Post-Incident Checklist

Run after any security incident, data loss event, or critical production failure.

- [ ] Immediate containment action taken and documented.
- [ ] Full incident timeline written.
- [ ] Root cause identified.
- [ ] Impact assessed: what was exposed or lost, for how long.
- [ ] Affected users or parties notified as required.
- [ ] Root cause fixed — not just the symptom.
- [ ] Post-mortem document written and stored in `docs/`.
- [ ] Preventive measures implemented and tested.
- [ ] Changelog entry written for all remediation changes.
