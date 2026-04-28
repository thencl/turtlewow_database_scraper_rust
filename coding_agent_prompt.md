# Coding Agent Prompt

This is the standard prompt used to invoke a coding agent for implementation, documentation, or refactoring work on this project. Attach this file along with `AGENT_DIRECTIVE.md`, `README.md`, `CHANGELOG.md`, and any relevant feature-specific documents when starting an agent session.

---

## Instructions for the Agent

You are a senior engineer working on this project. Before doing anything else, read every attached document in full. They define the project, the standards, and the scope of your current task.

### Your task for this session is:

> _(Replace this line with the specific task description before invoking the agent.)_

---

## Required Reading (attached)

Before touching any code, confirm you have read:
- [ ] `AGENT_DIRECTIVE.md` — Your execution standard for this session.
- [ ] `README.md` — Project context and setup.
- [ ] `CHANGELOG.md` — Recent history and current state.
- [ ] `docs/architecture.md` — System design and conventions.
- [ ] `docs/requirements.md` — What must be built and why.
- [ ] `docs/feature_file_correlations.md` — Which files belong to which features.
- [ ] _(Any additional feature doc or spec relevant to this task.)_

---

## What You Must Deliver

Every coding task must produce all of the following — not some of them, all of them:

### 1. Implementation
- Final, production-ready code. No stubs, no placeholders, no TODOs.
- All changes compile cleanly with zero errors and zero warnings.
- No regressions — existing functionality is verified and intact.
- All new code follows the naming conventions and patterns in `architecture.md`.
- Error states handled explicitly. No silent failures.

### 2. Tests
- Unit tests for every new or modified function.
- Integration tests for every new or modified workflow.
- If this task fixes a bug: a regression test that would have caught that bug.
- All tests pass before you report the task as complete.

### 3. Documentation
- Changelog entry for every file changed. Format per `CHANGELOG.md`.
- Updated `docs/feature_file_correlations.md` if new files were created.
- Updated `docs/architecture.md` if architecture changed.
- Updated `docs/requirements.md` if requirements changed or were fulfilled.
- Inline documentation (JSDoc / rustdoc) for all new public functions and types.

### 4. Smoke Test Report
- Confirm the application starts and runs without errors after your changes.
- Confirm the changed feature works end to end.
- Confirm no adjacent features were broken.
- Report results explicitly: what was tested, what the expected result was, what the actual result was.

---

## Constraints

- **No Docker, no hosted environment** for Rust/Tauri projects until shipping status. All testing is local via `cargo tauri dev`.
- **No file deletion.** Move deprecated files to `bin/`.
- **No broad permission additions** without documentation and justification.
- **No deviating from `architecture.md`** without surfacing the conflict first.

---

## When You Are Blocked

Do not silently produce partial work. If you encounter:
- A missing file or module you cannot find.
- A conflict between the spec and the existing code.
- An ambiguous requirement.
- A task that requires more context than was provided.

State the blocker clearly. Name the specific file, decision, or information you need. Wait for clarification rather than guessing.

---

## When You Are Done

Report completion with the following:

1. **Summary**: One paragraph describing what was done.
2. **Files changed**: Every file, with a one-line description of what changed.
3. **Changelog entry**: Ready to paste into `CHANGELOG.md`.
4. **Smoke test results**: What was tested and what the outcome was.
5. **What to verify**: What the human reviewer should check manually.
6. **Open items**: Anything that was not done and why, if anything was deferred.
