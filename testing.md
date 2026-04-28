# Testing

This document defines the testing strategy, standards, tooling, and acceptance criteria for this project. Every developer and agent working on this project must follow these standards. Testing is not optional — untested code is unfinished code.

---

## Testing Philosophy

- Tests are written alongside features, not after.
- A bug fix is not complete until a test exists that would have caught that bug.
- Tests must be deterministic — the same test run must always produce the same result.
- Tests must be independent — no test should depend on the side effects of another.
- A failing test is a blocker. Nothing ships with failing tests.

---

## Test Types and Scope

### Unit Tests
- **Scope**: A single function, hook, or module in isolation.
- **Dependencies**: All external dependencies are mocked.
- **Goal**: Verify the logic of a single unit is correct for all relevant inputs and edge cases.
- **When required**: Every exported function and every stateful hook must have unit tests.
- **Coverage target**: 70% line coverage minimum across the codebase.

### Integration Tests
- **Scope**: Multiple modules or layers working together.
- **Dependencies**: Real implementations used where possible; external services mocked at the network boundary.
- **Goal**: Verify that the wiring between modules is correct and that data flows as expected.
- **When required**: Every major feature workflow must have at least one integration test.
- **Coverage target**: 50% of primary user flows covered.

### End-to-End (E2E) Tests
- **Scope**: Full user flows from the UI through all layers to persistence.
- **Dependencies**: Real application running against a test environment.
- **Goal**: Verify that the system works correctly as a whole from the user's perspective.
- **When required**: All P0 and P1 features must have E2E coverage.

### Smoke Tests
- **Scope**: The most critical paths in the application.
- **Goal**: Quickly confirm that the application is alive and functional after a deploy or major change.
- **When required**: Run after every deploy and after every major code change before marking a task complete.
- **Duration**: Must complete in under 5 minutes.

---

## Test Tooling

> Fill in for your project stack. Examples below.

| Tool | Purpose |
|---|---|
| _(e.g. Vitest / Jest)_ | Unit and integration test runner |
| _(e.g. Testing Library)_ | Component and hook testing utilities |
| _(e.g. Playwright / Cypress)_ | End-to-end testing |
| _(e.g. cargo test)_ | Rust unit and integration tests |
| _(e.g. axe-core)_ | Automated accessibility testing |
| _(e.g. Lighthouse CI)_ | Performance and accessibility auditing |

---

## Running Tests

### Type Check
```bash
# Frontend (adjust for your project)
cd frontend && npm run typecheck
```

### Unit and Integration Tests
```bash
# Frontend
cd frontend && npm test

# Backend / Rust
cd backend && cargo test
```

### Full Build Verification
```bash
# Frontend
cd frontend && npm run verify

# Full project
npm run build
```

### End-to-End Tests
```bash
# Adjust for your E2E tool
npx playwright test
```

### Accessibility Audit
```bash
# Run Lighthouse against your local build
npx lighthouse http://localhost:3000 --output html --output-path ./reports/accessibility.html
```

---

## Writing Tests — Standards

### Naming Convention
```
describe('ComponentOrModule', () => {
  describe('methodOrBehavior', () => {
    it('does X when Y', () => { ... });
    it('returns Z when input is empty', () => { ... });
    it('throws an error when required field is missing', () => { ... });
  });
});
```

### Test Structure — Arrange / Act / Assert
Every test follows this structure explicitly:
```typescript
it('updates the workspace name when a valid name is provided', () => {
  // Arrange
  const initialState = createInitialState({ name: 'Old Name' });

  // Act
  const result = updateWorkspaceName(initialState, 'New Name');

  // Assert
  expect(result.name).toBe('New Name');
});
```

### What to Test
- Happy path: the expected behavior when inputs are valid.
- Edge cases: empty inputs, null, undefined, zero, max values.
- Error paths: what happens when something goes wrong.
- Boundary conditions: values at the exact edge of valid/invalid ranges.

### What Not to Mock
- Do not mock the unit under test.
- Do not mock the language's built-in behavior.
- Do not mock stable, well-tested utilities that have no side effects.

### What Must Be Mocked
- Network requests.
- File system operations.
- Timers and dates (use a deterministic fake).
- External services and third-party APIs.
- Randomness.

---

## Acceptance Criteria for UI Features

Every UI feature passes the following before it is considered done:

- [ ] Renders without console errors or warnings.
- [ ] All interactive elements are reachable and operable by keyboard.
- [ ] Focus management is correct (focus goes to the right place after an action).
- [ ] Visible focus styles are present.
- [ ] ARIA roles and labels are correct and meaningful.
- [ ] All defined interaction states (default, hover, active, disabled, loading, error) are implemented and tested.
- [ ] Behavior is correct at multiple viewport widths.
- [ ] Accessibility audit passes with no critical or serious violations.

---

## Acceptance Criteria for API / Service Features

Every API or service feature passes the following before it is considered done:

- [ ] Returns the correct response for all valid inputs.
- [ ] Returns structured, consistent error responses for all invalid inputs.
- [ ] Handles network timeouts and upstream failures gracefully.
- [ ] Does not leak implementation details or sensitive data in error messages.
- [ ] Is covered by at least one integration test that exercises the full request/response cycle.

---

## Smoke Test Protocol

Run the following after every major change or deployment. Document results.

| Test | Expected Result | Actual Result | Pass/Fail |
|---|---|---|---|
| Application starts without errors | No console errors on load | — | — |
| Primary navigation works | All routes resolve | — | — |
| Core P0 feature 1 works | _(define expected behavior)_ | — | — |
| Core P0 feature 2 works | _(define expected behavior)_ | — | — |
| Data persists after page reload | _(define expected behavior)_ | — | — |
| Error states surface correctly | Error messages are visible | — | — |
| No broken console errors | Zero unhandled errors | — | — |

---

## Rust / Tauri Projects — Local-First Testing Rule

> **This section is mandatory for all Rust-based and Tauri-based projects.**

Until the project has reached confirmed shipping status, the following rules apply without exception:

- **No Docker.** Do not containerize the application, its dependencies, or its test environment during development. Docker adds environment abstraction that hides Tauri and system-level integration issues. Problems that only appear on the real OS must be caught early, not hidden.
- **No remote hosting.** Do not deploy to any hosted environment (VPS, cloud function, container registry, staging server) during pre-ship development. All development, testing, and validation runs locally on the developer's machine.
- **Tauri is the runtime.** All testing happens inside the Tauri shell. Do not test UI in a bare browser or a Node server — Tauri's WebView, IPC bridge, and native API surface must be exercised from day one.
- **Local build is the only build.** The only valid test environment is `cargo tauri dev` running on the target OS. If it does not work there, it is not done.
- **OS-native behavior must be verified locally.** File system access, permissions, window management, tray icons, system notifications, and all other native integrations must be manually verified on the target platform before any task is marked complete.
- **When shipping status is reached**, Docker and hosted environments may be introduced — but only after the local Tauri build is stable, all smoke tests pass locally, and the decision to deploy is explicitly documented in the changelog.

### Rust / Tauri Test Commands
```bash
# Run Rust unit tests
cargo test

# Run Tauri app in dev mode (primary test environment)
cargo tauri dev

# Build release binary for local verification
cargo tauri build

# Run frontend type checks before tauri dev
cd src-tauri/../frontend && npm run typecheck && cd ..

# Run frontend unit tests
cd frontend && npm test
```

---

## Continuous Integration

All tests run automatically on every pull request and on every merge to main. A PR cannot be merged if:
- Any test is failing.
- Type checks fail.
- Lint errors are present.
- Build does not produce a clean artifact.

---

## Test Debt Policy

- Test debt is tracked as issues in the project backlog, not as TODO comments in test files.
- No feature may lower the overall test coverage percentage.
- Skipped tests must have a comment explaining why and a linked issue for resolution.
- Tests must not be permanently skipped — they must either be fixed or deleted with a documented reason.
