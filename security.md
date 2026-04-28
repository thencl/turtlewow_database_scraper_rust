# Security

This document defines the security requirements, standards, and review process for this project. Security is a first-class requirement — it is not reviewed at the end. Every feature that touches user data, the file system, authentication, or external communication must be reviewed against this document before shipping.

---

## Security Philosophy

- Principle of least privilege: request only the permissions the feature actually needs.
- Defense in depth: never rely on a single layer of protection.
- Fail securely: when something goes wrong, the system must fail in a state that does not expose data or grant unintended access.
- Explicit over implicit: security decisions are documented and deliberate, never accidental.

---

## Input Validation

All external input is untrusted until validated. This applies to:
- User-provided form values.
- URL parameters and query strings.
- File contents read from disk.
- Data received from external APIs or services.
- IPC messages in Tauri applications.
- WebSocket messages.
- CLI arguments.

### Validation Rules
- Validate at the boundary — the first point the data enters the system.
- Validate type, format, length, and range.
- Reject invalid input explicitly — do not silently coerce or ignore it.
- Never trust client-side validation alone — always validate on the server or in the Rust backend.
- Use an allowlist (permitted values) rather than a denylist (blocked values) wherever possible.

---

## File System Operations

- **Path validation is mandatory** before every file system read or write.
- Resolve all paths to their absolute form and verify they are within the intended directory before use.
- Never execute a path that was constructed from user input without sanitization.
- Never allow directory traversal: reject paths containing `..`, symbolic link chains, or absolute paths that escape the expected root.
- File operations that could overwrite existing data must confirm the target before writing.
- Sensitive file operations must be logged.

### Tauri-Specific
- Use Tauri's `allowlist` in `tauri.conf.json` to explicitly permit only the file system paths and capabilities the application needs.
- Do not use `all: true` in the Tauri allowlist in production builds.
- IPC command handlers must validate all arguments before acting on them.
- Do not expose Rust functions to the frontend that have broader access than needed.

---

## Authentication and Sessions

- Passwords are never stored in plain text — use a strong, modern hashing algorithm (e.g. Argon2, bcrypt).
- Session tokens are cryptographically random, sufficiently long (minimum 128 bits), and stored securely.
- Tokens are invalidated on logout and on password change.
- Authentication failures return generic error messages — do not reveal whether the username or password was wrong.
- Implement rate limiting on authentication endpoints.
- Multi-factor authentication must be supported if the application handles sensitive or personal data.

---

## Secrets Management

- Secrets (API keys, tokens, passwords, connection strings) are **never** committed to source control.
- All secrets are stored in environment variables or a dedicated secrets manager.
- `.env` files are listed in `.gitignore`.
- `.env.example` documents every variable name with a description but no real values.
- Secrets are never logged, printed to console, or included in error messages.
- Secrets are rotated regularly and immediately upon any suspected exposure.

---

## Dependency Management

Before adopting any new dependency:
- Check for known vulnerabilities: `npm audit` / `cargo audit`.
- Review the dependency's maintenance status — is it actively maintained?
- Review the dependency's permissions and access surface.
- Prefer well-established packages with large install bases and active security disclosure processes.
- Prefer fewer dependencies over more — every dependency is an attack surface.

Run dependency audits:
```bash
# Node / npm
npm audit

# Rust / Cargo
cargo audit

# Review outdated packages
npm outdated
cargo outdated
```

Pin dependency versions in production. Do not use unbounded version ranges (`*`, `^latest`).

---

## Data Protection

- Do not collect data that is not needed.
- Do not retain data longer than necessary.
- Personal or sensitive data at rest must be encrypted.
- Personal or sensitive data in transit must use TLS 1.2 or higher.
- Data access must be logged for sensitive operations.
- Users must be able to delete their own data on request.

---

## Error Handling

- Error messages shown to users must be generic and never expose stack traces, internal paths, database details, or system information.
- Detailed error information is logged internally and never sent to the client.
- All unhandled errors are caught at a top-level boundary and logged.
- Errors in security-sensitive operations (authentication, file access, data write) are always logged with sufficient context to investigate.

---

## Permissions

- Request only the permissions the feature actually requires.
- Document every permission the application requests and the reason for it.
- Never silently escalate permissions — always prompt the user and explain why.
- Sensitive operations (delete, overwrite, export, share) require explicit user confirmation.

### Tauri Permissions
- Tauri capabilities are configured in `tauri.conf.json` and must be kept minimal.
- Review the Tauri allowlist on every new feature that adds file, network, or OS-level access.
- Document every capability and why it is needed in `architecture.md`.

---

## Security Review Process

### Pre-Commit
- [ ] No secrets, credentials, or API keys are present in any committed file.
- [ ] All new input handling validates at the boundary.
- [ ] All new file system operations validate paths.
- [ ] No new broad permissions were added without documentation.

### Pre-PR
- [ ] `npm audit` or `cargo audit` passes with no high or critical vulnerabilities.
- [ ] New dependencies have been reviewed for security and maintenance status.
- [ ] Error messages do not expose internal details.
- [ ] Authentication and session logic has been reviewed if touched.
- [ ] Tauri allowlist changes have been reviewed and documented.

### Pre-Deploy
- [ ] Full dependency audit completed.
- [ ] Environment variables are verified in the target environment.
- [ ] Secrets are rotated if they were ever exposed or committed accidentally.
- [ ] Security-sensitive flows have been manually tested.

---

## Incident Response

If a security incident occurs (credential exposure, data breach, unauthorized access):

1. **Contain immediately** — revoke exposed credentials, disable affected functionality.
2. **Document** — record exactly what happened, what was exposed, and when it was discovered.
3. **Assess impact** — determine what data or systems were affected and for how long.
4. **Notify** — notify affected users and relevant parties as required by law and policy.
5. **Remediate** — fix the root cause, not just the symptom.
6. **Post-mortem** — document the full incident and the changes made to prevent recurrence.

---

## Security Debt Policy

Security vulnerabilities are treated as bugs, not feature work. Severity:

- **Critical**: Immediate fix required. Blocks all other work. Must be resolved before any new feature work.
- **High**: Must be fixed within the current sprint.
- **Medium**: Must be triaged and scheduled within two sprints.
- **Low**: Tracked in the backlog and addressed in regular maintenance.

No critical or high vulnerabilities may be present at shipping time.
