---
name: qa-auth
description: "Audit authentication and authorization - role guards, route protection, permission gaps, token handling, permission check consistency, visibility matrix, rate limiter infrastructure verification, OTP brute-force protection"
user-invocable: true
argument-hint: "[module]"
---

# QA Auth — Authentication & Authorization Audit

Verify all endpoints and frontend routes have proper authentication and authorization, detecting unprotected endpoints, missing role guards, permission escalation risks, and token handling issues.

## Execution Mode

- **Standalone** (`/qa-auth [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check auth`): qa-fix uses the checks below as its checklist for the auth layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Unprotected endpoint | **Critical** | Controller method has no auth guard and is not marked as public |
| 2 | Missing role guard | **High** | Endpoint should restrict by role but has no role decorator |
| 3 | Unprotected frontend route | **High** | Frontend page accessible without auth redirect |
| 4 | Role mismatch frontend<>backend | **High** | Frontend shows action for role that backend doesn't allow |
| 5 | Missing current-user validation | **Medium** | Endpoint uses user data but doesn't verify ownership |
| 6 | Token exposure | **Critical** | JWT stored in localStorage instead of HTTP-only cookie |
| 7 | Missing CORS config | **Medium** | Backend allows all origins or missing CORS |
| 8 | Sensitive data in response | **Medium** | API returns password hash, tokens, or secrets |
| 9 | Missing rate limiting | **Medium** | Login/register without rate limiter |
| 10 | Frontend role-based UI gap | **Medium** | Backend has role restriction but frontend doesn't hide UI |
| 11 | Permission check inconsistency | **Medium** | Different permission check utilities used inconsistently across components (hooks vs HOCs vs inline) |
| 12 | Permission name mismatch | **High** | Frontend role/permission names don't match backend definitions (casing, format, enum vs string) |
| 13 | Missing visibility matrix | **Medium** | UI elements not conditionally rendered based on role when backend restricts access |
| 14 | Rate limiter infrastructure inactive | **Critical** | Rate limiting decorators exist but the guard/middleware is not registered globally — all rate limits are dead code |
| 15 | Missing brute-force protection | **High** | OTP/token verification endpoint has no attempt counter or lockout mechanism |

## Skill-Specific Patterns

### Auth Library Detection

| Signal | Library |
|--------|---------|
| `next-auth` / `@auth/core` in package.json | NextAuth / Auth.js |
| `@casl/ability` / `@casl/react` / `@casl/vue` | CASL |
| `firebase/auth` in package.json | Firebase Auth |
| `@clerk/nextjs` / `@clerk/clerk-react` | Clerk |
| `@supabase/supabase-js` + `.auth` usage | Supabase Auth |
| `@auth0/auth0-react` / `@auth0/nextjs-auth0` | Auth0 |
| Custom: `useAuth()`, `usePermission()`, `AuthContext` | Custom auth hook/context |

### Framework Auth Patterns

| Framework | Auth Guard | Public Route | Role Decorator | Current User |
|-----------|-----------|--------------|----------------|-------------|
| NestJS | APP_GUARD/JwtAuthGuard | @Public() | @Roles() | @CurrentUser() |
| Express | passport.authenticate | - | custom middleware | req.user |
| Spring Boot | @PreAuthorize | @PermitAll | @Secured/@RolesAllowed | @AuthenticationPrincipal |
| Django | @login_required | @permission_classes([AllowAny]) | @permission_required | request.user |
| Laravel | ->middleware('auth') | - | ->middleware('role:admin') | auth()->user() |
| Next.js | middleware.ts | matcher config | custom | getServerSession() |
