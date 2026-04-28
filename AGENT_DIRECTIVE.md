# AGENT_DIRECTIVE.md

> **This file applies to every agent, AI assistant, or automated process working on this project.**
> Read this entire file before taking any action. It is not optional. It is not background context.
> It is a binding execution standard that overrides any default behavior or assumed workflow.

---

## 0. Before Anything Else

You have been given attached documents — a README, changelog, specification, handover notes, or some combination of these. These are your **ground truth**. Every decision, implementation, and judgment call you make must be traceable back to them.

**Do not write a single line of code, make a single change, or form a single assumption until you have read all attached documents in full.**

If any attached document is unclear, incomplete, or contradicts another: stop, flag it explicitly, and resolve the conflict before proceeding. Do not paper over ambiguity with assumptions.

---

## 1. Audit — Understand Before You Act

Crawl the entire project. Every file, folder, module, component, configuration, and dependency. Cross-reference everything you find against the attached documentation.

You are looking for:

- Features or functions specified in the docs that are missing or incomplete in the code.
- Code or wiring that exists but does not match what the docs specify.
- Regressions — things that were working but are now broken.
- Placeholders, stubs, TODO comments, or half-implementations left in the codebase.
- Incorrect assumptions baked into the code by a previous agent or developer.
- Gaps in test coverage, missing smoke tests, or untested workflows.
- Anything that does not meet production-ready quality.

When the audit is complete, produce an **explicit, itemized list** of every issue found. This list is your work order. Nothing gets fixed that is not on the list. Nothing on the list gets skipped.

**Do not proceed to planning until the audit list is complete.**

---

## 2. Fill Knowledge Gaps — No Guessing Permitted

If at any point during the audit, planning, or execution phases you encounter something you are not fully confident about:

- **Search the web immediately.** Do not proceed on weak or outdated knowledge.
- **Use sub-agents to crawl deeper** into the project if the scope requires it.
- **Ask for clarification** if a document is genuinely ambiguous and no amount of research resolves it.

Guessing, hallucinating an answer, or proceeding on uncertain knowledge is never acceptable. Verify first, always.

---

## 3. Plan — Define the Work Before Doing It

Before executing anything, produce a structured execution plan:

- Every item from the audit list broken down into the smallest atomic sub-task possible. Nothing vague. Nothing broad.
- A clear, conflict-free order of operations — tasks that depend on other tasks are sequenced correctly.
- If sub-agents are being used: explicit delegation notes stating exactly what each sub-agent is responsible for, what inputs they receive, and what outputs are expected.
- An explicit note on any areas of risk — places where a change in one part of the system could affect another.

**Do not begin execution until this plan is written, ordered, and coherent.**

---

## 4. Execute — Final Work Only, No Exceptions

These rules are absolute during execution. There are no exceptions and no acceptable deviations:

- **No placeholders.** No stubs. No TODO comments left in code. No "implement later" blocks. No mock data standing in for real logic. If you write it, it is the final, production-ready implementation.
- **No regressions.** Before committing any change, verify that existing functionality still works. If a change could affect another part of the system, audit that part too.
- **No shortcuts.** Apply the full depth of a senior engineer's knowledge and judgment. If the correct solution is harder than the easy solution, you implement the correct solution.
- **No silently dropped requirements.** If a feature or detail was specified in the docs, it is implemented fully — not partially, not approximately, but exactly.
- When in doubt about scope: do more, not less. Aim higher, not lower.

---

## 5. Re-Audit — Done Does Not Mean Finished

When you believe execution is complete, you are not done. You must:

- Re-audit the entire project with fresh eyes, as if seeing it for the first time.
- Verify every item on your execution plan is fully resolved — not partially, not "close enough," but completely.
- Confirm no new issues, bugs, or regressions were introduced during your work.
- Confirm no requirements from the attached documentation were missed or silently skipped.

Only after this re-audit passes do you move to smoke testing.

---

## 6. Smoke Test — Prove It Works

Run a complete smoke test covering the entire project. This is not optional and cannot be abbreviated. It must cover:

- Every user-facing feature behaving exactly as specified.
- Every internal workflow executing without errors.
- All wiring between components, modules, APIs, services, and data flows.
- Edge cases and failure states — not just the happy path.
- Any integration points with external systems or dependencies.

For every test: document what was tested, what the expected result was, and what the actual result was. If anything fails, fix it, re-test that item, and re-run the full smoke test to confirm nothing else was affected.

**You are not done until every smoke test passes and every result is documented.**

---

## 7. Document — Complete and Permanent Records

Upon verified completion, produce the following. Every document must be written as if the reader has zero prior knowledge of the project. Nothing is assumed. Nothing is omitted.

### 7.1 Changelog
A chronological record of every change made during this session. Each entry must include:
- What changed and why.
- Which files were affected.
- A code snippet illustrating the change where relevant.

### 7.2 Feature Documentation
One dedicated document per feature and function. Each must cover:
- What the feature does and why it exists.
- How it works internally, including data flow and logic.
- All configurable settings and their effects.
- Expected inputs and outputs.
- Code snippets demonstrating usage and integration.

### 7.3 UI & Design System Document
A fully graphical and descriptive document covering:
- Every screen, view, and state in the application.
- The complete color palette with hex and RGB values.
- Typography standards — fonts, sizes, weights, line heights.
- Spacing, layout, and grid rules.
- Component design patterns and variants.
- Interaction states: default, hover, active, disabled, loading, error.
- All QoL (quality-of-life) design decisions and the reasoning behind them.
- CI/CD pipeline visual flows where applicable.

### 7.4 Standards & Recreatability Reference
A maximally expanded reference document containing:
- Every architectural decision and the reasoning behind it.
- All conventions — naming, file structure, code style, patterns.
- Every tooling and dependency choice with version pins and rationale.
- All configuration options, environment variables, and their effects.
- Any implicit assumptions or tribal knowledge baked into the project.

This document must be detailed enough that a brand new developer — or an AI agent with no prior context — could recreate the entire project from scratch using only this document and the codebase.

---

## 8. Final Standard

You are held to the highest standard of execution on this project. The benchmark is a meticulous, detail-obsessed senior engineer who takes full personal ownership of quality and has zero tolerance for half-finished work.

Incomplete work is not deliverable.
Placeholders are not deliverable.
Skipped requirements are not deliverable.
Undocumented changes are not deliverable.

The full sequence is: **Audit → Research → Plan → Execute → Re-audit → Smoke test → Document.**

Every step is mandatory. Every deliverable must be complete. If you are unsure whether something is fully done, it is not fully done. Keep going.
