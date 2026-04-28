# Architecture

This document defines the system architecture for this project. It is a living document — update it whenever a significant architectural decision is made. Every change must also produce a changelog entry.

---

## Rust / Tauri Projects — Local-First Architecture Rule

> **This section is mandatory for all Rust-based and Tauri-based projects.**

- The Tauri shell is the **only** application container until shipping status is reached. Do not architect around Docker, a web server, or any hosted runtime during pre-ship development.
- The IPC bridge between the Tauri frontend (WebView) and the Rust backend is the **only** communication channel between UI and native logic. Do not bypass it.
- Native OS APIs (file system, notifications, tray, window management) are accessed exclusively through Tauri commands — never through browser APIs or polyfills.
- Architecture decisions that would require a hosted environment to test are deferred until after shipping status is confirmed and documented.
- When Docker or hosting is eventually introduced, it must be additive — the local Tauri build must remain fully functional and remain the primary development and testing environment.

---

## System Overview

> **Fill in when starting the project.**

| Property | Value |
|---|---|
| Project name | — |
| Primary language(s) | — |
| Frontend framework | — |
| Backend framework / language | — |
| Database / persistence layer | — |
| Build tool | — |
| Deployment target | — |
| Minimum supported runtime / browser | — |

---

## High-Level Architecture Diagram

> Replace this section with a diagram (Mermaid, ASCII, or linked image) once the architecture is defined.

```
[ Client / Frontend ]
        |
        v
[ API / Backend Layer ]
        |
        v
[ Data / Persistence Layer ]
        |
        v
[ External Services / Integrations ]
```

---

## Layer Definitions

### Frontend
- **Responsibility**: All user interface rendering, user input handling, and client-side state management.
- **Framework**: _(fill in)_
- **State management**: _(fill in — e.g. React Context, Zustand, Redux)_
- **Styling approach**: _(fill in — e.g. CSS Modules, Tailwind, styled-components)_
- **Key conventions**:
  - Components are small and single-responsibility.
  - All shared UI elements live in `src/components/`.
  - Page-level components live in `src/pages/`.
  - Hooks live in `src/hooks/`.
  - Utility functions live in `src/utils/`.
  - Types and interfaces live in `src/types/`.

### Backend / API Layer
- **Responsibility**: Business logic, data validation, authentication, and persistence orchestration.
- **Framework**: _(fill in)_
- **Authentication method**: _(fill in)_
- **API style**: _(fill in — REST / GraphQL / RPC / tRPC)_
- **Key conventions**:
  - Controllers handle only routing and request/response shaping.
  - Business logic lives in service modules, never in controllers.
  - All inputs are validated at the boundary before entering service logic.
  - Errors are always returned in a consistent, structured format.

### Data / Persistence Layer
- **Responsibility**: Durable storage, schema management, and data access.
- **Technology**: _(fill in — SQLite / PostgreSQL / Redis / etc.)_
- **ORM / query layer**: _(fill in)_
- **Schema migration tool**: _(fill in)_
- **Key conventions**:
  - Migrations are forwards-compatible and reversible.
  - Raw queries are only used when the ORM cannot express the operation.
  - No business logic lives in the database layer.

### External Services / Integrations
> List every external dependency the system relies on.

| Service | Purpose | Owned by | Failure mode |
|---|---|---|---|
| _(name)_ | _(what it does)_ | _(team/vendor)_ | _(what happens if it is down)_ |

---

## Module and Component Map

> Define each major module, what it owns, and what it does not own.

### Module: _(name)_
- **Owns**: _(list of responsibilities)_
- **Does not own**: _(list of things explicitly out of scope for this module)_
- **Communicates with**: _(other modules and how — direct import / event / API call)_
- **Key files**: _(list of primary files in this module)_

---

## State Management

- **Where global state lives**: _(fill in)_
- **Who can mutate global state**: _(fill in — only reducers / only services / etc.)_
- **Where local component state is appropriate**: _(fill in)_
- **Caching strategy**: _(fill in — none / React Query / SWR / manual)_

---

## Data Flow — Primary Use Cases

> For each primary use case, trace the full data flow from user action to persistence and back.

### Use Case: _(name)_
1. User performs _(action)_.
2. _(Component)_ calls _(function / API endpoint)_.
3. _(Layer)_ validates and processes the input.
4. _(Layer)_ persists the result to _(store)_.
5. _(Component)_ receives the updated state and re-renders.
6. Error handling: _(what happens if any step fails)_.

---

## Folder Structure

```
project-root/
├── src/                        # Application source
│   ├── components/             # Shared UI components
│   ├── pages/                  # Page-level components / route views
│   ├── hooks/                  # Custom React hooks (or equivalent)
│   ├── services/               # Business logic and API clients
│   ├── store/                  # Global state management
│   ├── types/                  # TypeScript types and interfaces
│   └── utils/                  # Pure utility functions
├── tests/                      # Test files mirroring src structure
├── docs/                       # Project documentation
│   ├── MASTER_PROMPT.md
│   ├── architecture.md         ← this file
│   ├── requirements.md
│   ├── testing.md
│   ├── accessibility.md
│   ├── ai_guidelines.md
│   ├── security.md
│   ├── checklist.md
│   └── feature_file_correlations.md
├── bin/                        # Deprecated files (never deleted, moved here)
├── _backups/                   # Pre-edit backups of modified files
├── .github/
│   ├── copilot-instructions.md
│   └── prompts/                # Reusable .prompt.md files
├── AGENT_DIRECTIVE.md
├── CHANGELOG.md
├── README.md
└── .env.example
```

---

## Naming Conventions

| Context | Convention | Example |
|---|---|---|
| React components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase, `use` prefix | `useWorkspaceState.ts` |
| Utility functions | camelCase | `formatDate.ts` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| CSS classes | kebab-case | `panel-header` |
| API endpoints | kebab-case | `/api/user-profile` |
| Database tables | snake_case | `user_sessions` |
| Environment variables | SCREAMING_SNAKE_CASE | `DATABASE_URL` |

---

## Architectural Decision Log

> Record every significant architectural decision here. Never delete entries — append only.

### ADR-001 — _(title)_
- **Date**: _(date)_
- **Status**: Accepted / Superseded / Deprecated
- **Context**: _(what situation forced this decision)_
- **Decision**: _(what was decided)_
- **Alternatives considered**: _(what else was evaluated)_
- **Consequences**: _(what this decision makes easier and harder)_

---

## Change Management

- Architecture changes must be documented here before implementation begins.
- Every architecture change requires a corresponding changelog entry.
- Deprecated components or modules are moved to `bin/` — never deleted.
- See `feature_file_correlations.md` for a mapping of features to files.
