---
name: qa-security
description: "Audit backend security vulnerabilities - SQL injection, cryptographic randomness, file upload safety, sensitive logging, production exposure, XSS, infrastructure config"
user-invocable: true
argument-hint: "[module]"
---

# QA Security — Backend Security Deep Audit

Detect code-level security vulnerabilities beyond authentication: injection attacks, cryptographic weaknesses, unsafe file handling, sensitive data leakage, production configuration gaps, and client-side security.

## Execution Mode

- **Standalone** (`/qa-security [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check auth`): qa-fix uses the checks below as its checklist for the auth layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | SQL/NoSQL injection | **Critical** | Raw user input interpolated into queries without parameterization |
| 2 | Insecure randomness | **High** | `Math.random()`, `random.random()`, or similar non-cryptographic RNG used for security-sensitive values (OTP, tokens, secrets) |
| 3 | Unsanitized file upload | **Medium** | User-controlled filename used in storage path without sanitization (path traversal risk) |
| 4 | Sensitive data in logs | **High** | `console.log`, `logger.debug`, or equivalent outputs JWT tokens, passwords, OTP codes, cookies, or API keys |
| 5 | Debug/test routes in production | **Medium** | Static file serving, debug endpoints, or test fixtures accessible without environment guard |
| 6 | XSS via unsafe URL handling | **High** | `window.open()`, `href`, `src`, or `location` set from user-controlled data without scheme validation (`javascript:` protocol injection) |
| 7 | Infrastructure secrets | **Medium** | Hardcoded secrets, missing auth on Redis/DB, default credentials, `process.env` used at module evaluation time instead of ConfigService |
| 8 | Missing input sanitization at boundaries | **Medium** | User input from request body/params/query not validated before use in business logic (beyond DTO validation — raw SQL, file ops, external API calls) |
| 9 | Unsafe deserialization | **High** | `JSON.parse()` on untrusted input without try-catch, `eval()`, `new Function()`, or `child_process.exec` with user input |
| 10 | Side-effect logic bugs in auth flows | **Medium** | Password change/reset setting unintended flags (e.g., `isVerified: false`), token refresh not invalidating old token, session not cleared on password change |
| 11 | IME/i18n input compatibility | **Medium** | `onKeyPress` used instead of `onKeyDown` with `isComposing` check (breaks CJK input), or Enter key handling without IME composition guard |
| 12 | Missing request size limits | **Low** | No body size limit on file upload or JSON endpoints, no array length limit on batch endpoints, enabling resource exhaustion |
