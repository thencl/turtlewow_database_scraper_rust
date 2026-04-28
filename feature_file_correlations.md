# Feature File Correlations

This document maps every major feature area to the source files that implement it. It exists so that any developer or agent working on a feature immediately knows which files are relevant, what they own, and what other files must be checked when making changes.

**Keep this file updated.** Every time a new feature is added or a file's responsibility changes, this document is updated in the same commit.

---

## How to Use This Document

1. Find the feature group relevant to your change.
2. Read the file list and the edit guidance before touching any file.
3. After making changes, verify all correlated files are still consistent.
4. If your change introduces a new file or a new responsibility, add it here.

---

## Template Groups

> Replace these with the actual feature groups for your project. Every significant feature area should have a group. Groups should be cohesive — files in the same group are likely to need editing together.

---

## Group 1 — App Shell and Startup

**Purpose**: Bootstraps the application, sets up routing, registers global providers, and initializes top-level state.

**Files**:
- `src/main.tsx` — Entry point. Mounts the root React component.
- `src/App.tsx` — Root component. Defines routing, global providers, and app-level layout.
- `src/store/` — Global state initialization.

**Why correlated**: Changes to routing, global context, or app-level layout all pass through these files. A change to any one of them often requires checking the others.

**Edit guidance**:
- When adding a new route: update `App.tsx` and verify all navigation links.
- When adding a new global provider: add it in `App.tsx` and document it in `architecture.md`.
- When changing startup logic: verify the application still boots cleanly from a cold start.

---

## Group 2 — Layout and Window Management

**Purpose**: Defines the top-level UI shell, panels, window frames, and layout regions.

**Files**:
- `src/components/layout/` — All layout components (panels, frames, rails, regions).
- `src/store/workspaceContext.tsx` — Layout state (collapsed panels, active views, panel sizing).
- `src/pages/Dashboard.tsx` — Primary layout host.

**Why correlated**: Layout components consume workspace state. Changes to state shape affect all layout consumers. Changes to layout components may require state shape changes.

**Edit guidance**:
- When modifying panel collapse/expand logic: verify all panel consumers, not just the one being edited.
- When changing workspace state shape: audit every file that reads or writes that state.
- When adding a new panel type: register it in `Dashboard.tsx` and in `workspaceContext.tsx`.

---

## Group 3 — Theme and Design System

**Purpose**: Manages the visual theme, color system, typography, and design tokens.

**Files**:
- `src/components/ThemeStudio.tsx` (or equivalent) — Theme picker and preview UI.
- `src/styles/tokens.css` (or equivalent) — CSS custom properties / design tokens.
- `src/styles/global.css` — Base styles and resets.

**Why correlated**: Design token changes cascade across every component. Theme switching logic depends on the token structure.

**Edit guidance**:
- When adding a new color token: add it to `tokens.css`, document it in the UI design doc, and verify contrast ratios.
- When changing the theme switching mechanism: verify all theme-dependent components visually.

---

## Group 4 — Data / Persistence Layer

**Purpose**: Manages reading and writing persistent data — database, local storage, or file system.

**Files**:
- `src/services/` — Data access services and API clients.
- `src-tauri/src/commands/` (Tauri) — Rust IPC command handlers.
- `src-tauri/src/db/` (Tauri) — Database access layer.
- `migrations/` — Database schema migrations.

**Why correlated**: Frontend services call Tauri commands. Command handlers access the database. Schema changes require migration files. All four layers move together.

**Edit guidance**:
- When changing a data model: update the migration, the Rust handler, the TypeScript service, and all consumers.
- When adding a new Tauri command: register it in `main.rs`, add the TypeScript invoke call, and test IPC end to end.
- When modifying a migration: verify it is reversible and test it on a copy of real data.

---

## Group 5 — Authentication and Permissions

**Purpose**: Handles user identity, session management, and permission gating.

**Files**:
- `src/services/auth.ts` (or equivalent) — Authentication logic.
- `src/store/authContext.tsx` (or equivalent) — Auth state.
- `src/components/ProtectedRoute.tsx` (or equivalent) — Route-level permission guards.
- `src-tauri/src/commands/auth.rs` (Tauri) — Native-side auth logic.

**Why correlated**: Auth state flows from the backend through the service layer into context and then into every protected route and component.

**Edit guidance**:
- When changing session logic: verify that logout correctly clears all state.
- When adding a new protected route: add the guard and verify unauthenticated access is correctly blocked.
- When changing permission checks: audit all consumers of the permission state.

---

## Group 6 — Testing Infrastructure

**Purpose**: Test utilities, fixtures, mocks, and shared test helpers.

**Files**:
- `tests/` — All test files, mirroring `src/` structure.
- `tests/fixtures/` — Shared test data and factory functions.
- `tests/mocks/` — Shared mock implementations.
- `vitest.config.ts` / `jest.config.ts` — Test runner configuration.

**Why correlated**: Changes to test utilities or mocks can affect many tests across the suite. Configuration changes affect the entire test run.

**Edit guidance**:
- When modifying a shared mock: run the full test suite to catch unintended breakage.
- When adding a new fixture: check whether existing tests could benefit from using it.

---

## Adding a New Group

Copy the template below and fill it in when adding a new feature area:

```markdown
## Group N — Feature Name

**Purpose**: One sentence describing what this group does.

**Files**:
- `path/to/file.ts` — What this file owns.
- `path/to/another.ts` — What this file owns.

**Why correlated**: Why these files need to be considered together.

**Edit guidance**:
- When doing X: also check Y.
- When changing Z: verify A and B.
```
