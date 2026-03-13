---
name: qa-auth
description: Audit authentication and authorization - role guards, route protection, permission gaps, token handling, permission check consistency, visibility matrix, rate limiter infrastructure verification, OTP brute-force protection
user-invocable: true
argument-hint: "[module]"
---

# QA Auth - Authentication & Authorization Auditor

## Purpose
Verify all endpoints and frontend routes have proper authentication and authorization. Detect unprotected endpoints, missing role guards, frontend routes accessible without login, permission escalation risks, inconsistent permission check patterns, permission name mismatches, and missing role-based UI visibility.

## Usage
```
/qa-auth                      # Full audit
/qa-auth exercises            # Single module only
```

## 15 Checks

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

## Execution Algorithm

You MUST follow these steps in order. Do NOT skip steps. Do NOT modify any files. This is a READ-ONLY diagnostic.

### Step 0: Context & Requirements Gathering (MANDATORY)

Before auditing, you MUST understand what features are actually in scope for this project. Auditing code that exists but is intentionally unused/deprecated leads to false positives.

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
Glob: .claude-project/docs/PROJECT_KNOWLEDGE.md
```

Extract:
- Which features are actively used (e.g., "supports email login only" means Apple/Google login code is out of scope)
- Which modules are in production vs planned/deprecated
- User roles and their intended access levels
- Authentication method (JWT, session, OAuth providers, etc.)

**0b. Read sprint/feature requirements if available:**
```
Glob: .claude-project/requirements/features/*.md
```

**0c. Ask clarifying questions BEFORE proceeding:**

If you find code for features not mentioned in requirements (e.g., OTP verification, social login, FCM push tokens), you MUST ask the user:

> "I found code for [feature X] but it's not mentioned in the project requirements. Should I include it in the audit or skip it as unused/planned code?"

Wait for the user's response before proceeding. Do NOT assume unused code is a bug.

**0d. Build the In-Scope Feature List:**

Create a clear list of what IS and IS NOT in scope based on requirements + user answers:
```
IN SCOPE: [feature1, feature2, ...]
OUT OF SCOPE: [feature3 (reason), feature4 (reason), ...]
```

Only audit in-scope features in subsequent steps. Out-of-scope items may be mentioned in an "Info" section but should NOT be flagged as bugs.

**0e. Read role hierarchy from project CLAUDE.md or equivalent configuration.**

Extract defined roles, their hierarchy, and intended access levels. If no role hierarchy is documented, ask the user to clarify roles before proceeding.

---

### Step 1: Project & Framework Detection

**1a. Detect backend framework:**

| Signal | Framework |
|--------|-----------|
| `@nestjs/common` in package.json | NestJS |
| `express` in package.json (no nest) | Express |
| `manage.py` in root | Django |
| `pom.xml` or `build.gradle` with spring-boot | Spring Boot |
| `composer.json` with laravel | Laravel |
| `next` in package.json + API routes | Next.js |

**1b. Detect frontend framework:**

| Signal | Framework |
|--------|-----------|
| `react` in package.json or JSX/TSX files | React |
| `.vue` files + `vue` in package.json | Vue |
| `@angular/core` in package.json | Angular |
| `.svelte` files | Svelte |
| `next` in package.json | Next.js |

**1c. Detect auth library:**

| Signal | Library |
|--------|---------|
| `next-auth` / `@auth/core` in package.json | NextAuth / Auth.js |
| `@casl/ability` / `@casl/react` / `@casl/vue` | CASL |
| `firebase/auth` in package.json | Firebase Auth |
| `@clerk/nextjs` / `@clerk/clerk-react` | Clerk |
| `@supabase/supabase-js` + `.auth` usage | Supabase Auth |
| `@auth0/auth0-react` / `@auth0/nextjs-auth0` | Auth0 |
| Custom: `useAuth()`, `usePermission()`, `AuthContext` | Custom auth hook/context |

**1d. Determine auth patterns based on detected framework:**

Use the framework-specific pattern reference table below:

| Framework | Auth Guard | Public Route | Role Decorator | Current User |
|-----------|-----------|--------------|----------------|-------------|
| NestJS | APP_GUARD/JwtAuthGuard | @Public() | @Roles() | @CurrentUser() |
| Express | passport.authenticate | - | custom middleware | req.user |
| Spring Boot | @PreAuthorize | @PermitAll | @Secured/@RolesAllowed | @AuthenticationPrincipal |
| Django | @login_required | @permission_classes([AllowAny]) | @permission_required | request.user |
| Laravel | ->middleware('auth') | - | ->middleware('role:admin') | auth()->user() |
| Next.js | middleware.ts | matcher config | custom | getServerSession() |

Record the detected framework and its auth patterns for all subsequent steps.

**1e. Determine global vs per-route auth configuration:**

Search for the framework-appropriate global auth guard pattern. This fundamentally changes how you interpret every other check.

If global guard found:
- Auth is GLOBAL. All endpoints are protected by default.
- Only endpoints marked as public opt out of auth.
- The primary risk becomes accidental public marking on sensitive endpoints.

If NOT found:
- Auth is PER-CONTROLLER/PER-ROUTE. Each controller or method must explicitly use auth guard.
- The primary risk becomes forgetting to add the guard on new endpoints.
- This is a much weaker posture and should be flagged as a systemic concern.

Record the global auth mode for use in all subsequent checks.

**1f. Auto-detect project paths:**

Scan for frontend and backend source directories:
```
Glob: **/package.json
Glob: **/manage.py
Glob: **/pom.xml
Glob: **/composer.json
```

Record discovered paths (e.g., `frontend/`, `backend/src/`, `apps/web/`, `server/`) for all subsequent file searches. Do NOT assume hardcoded paths.

**1g. Check CORS configuration:**

Search for CORS configuration in the backend entry point.

Classify CORS posture:
- Allows ALL origins (dangerous in production)
- Properly restricted to specific origins
- No CORS call at all (browser default blocks cross-origin)

**1h. Check rate limiting infrastructure:**

Search for rate limiting patterns appropriate to the detected framework. Record whether rate limiting exists at all and where it is applied.

**1i. Check API prefix:**

Search for global API prefix configuration. Record it for route matching.

### Step 2: Backend Endpoint Auth Mapping

For each controller/route handler, build a complete auth profile of every endpoint method.

Glob for controller files based on the detected framework's conventions. If the user specified a module argument, filter to only controllers in that module directory.

For each controller/handler file, read the entire file and extract:

1. **Class-level / route-level decorators/middleware**: Auth guards or role restrictions applied at the class or route group level.

2. **Method-level decorators/middleware**: For each HTTP method handler, extract:
   - The HTTP method and route path
   - Whether it is marked as public
   - Whether it has an auth guard (relevant only if no global guard)
   - Whether it has role restrictions and which roles are listed
   - Whether the current user is injected/used
   - The method name

3. **Classify each endpoint** into one of four categories:
   - **Public**: Intentionally no auth required.
   - **Auth + Role**: Protected by auth guard AND has role restrictions.
   - **Auth only**: Protected by auth guard but NO role restriction. Any authenticated user can access.
   - **Unprotected**: No guard, not public, AND no global guard. This is a BUG.

Build an auth matrix as a data structure:
```
{
  endpoint: 'POST /exercises',
  method: 'create',
  controller: 'ExerciseController',
  file: '<auto-detected-path>/controllers/exercise.controller.ts',
  isPublic: false,
  roles: ['COACH', 'ADMIN'],
  hasCurrentUser: true,
  ownershipCheck: false,
  classification: 'Auth + Role'
}
```

### Step 3: Frontend Route Auth Mapping

Scan the frontend router configuration to determine which routes are protected.

First, find the router configuration files by globbing for common patterns in the auto-detected frontend directories.

Also search for the auth protection component used in routing:
```
Grep: ProtectedRoute|PrivateRoute|RequireAuth|AuthGuard|isAuthenticated|authRequired|RequireLogin
```

For each route definition found, determine:
- The route path
- Which app it belongs to (if multiple frontend apps exist)
- Whether it is wrapped in an auth-protecting component
- Whether the page component itself checks auth state on mount and redirects
- Whether there is a role-based guard on the route

Also scan for role-based route protection:
```
Grep: role.*===|hasRole|checkRole|RoleGuard|allowedRoles|requiredRole
```

Build a route protection matrix:
```
{
  path: '/exercises',
  app: 'frontend',
  isProtected: true,
  roleCheck: null,
  file: '<auto-detected-path>/routes.tsx'
}
```

### Step 4: Token and Session Audit

Check how authentication tokens are stored and transmitted on the frontend.

```
Grep: localStorage\.(set|get)Item.*token|localStorage\.(set|get)Item.*jwt|localStorage\.(set|get)Item.*auth|sessionStorage\.(set|get)Item.*token|document\.cookie|httpOnly|withCredentials|Authorization.*Bearer
```

Classify token storage mechanism:
- **HTTP-only cookie** (set by backend, sent automatically by browser with `withCredentials: true`): SECURE. Token is not accessible to JavaScript, immune to XSS theft.
- **localStorage**: INSECURE. Any XSS vulnerability allows token theft. Flag as CRITICAL.
- **sessionStorage**: SLIGHTLY BETTER than localStorage (cleared on tab close) but still XSS vulnerable. Flag as HIGH.
- **In-memory only** (React state/context, no persistence): SECURE against XSS but token lost on page refresh. Flag as informational.

Also check for token leakage vectors:
```
Grep: console\.(log|debug|info|warn).*token|console\.(log|debug|info|warn).*jwt|console\.(log|debug|info|warn).*auth
```

Check for tokens in URL parameters:
```
Grep: \?token=|\&token=|queryParam.*token|searchParams.*token
```

### Step 5: Sensitive Data Audit

Check if backend properly excludes sensitive fields from API responses.

Search for field exclusion patterns appropriate to the detected framework:
- NestJS/TypeORM: `@Exclude()`, `@Expose()`, `class-transformer`, `ClassSerializerInterceptor`
- Django: `serializer` fields/exclude, `defer()`, `only()`
- Spring Boot: `@JsonIgnore`, `@JsonProperty(access = WRITE_ONLY)`
- Laravel: `$hidden` array in model
- Express/Prisma: `select` in queries, response mapping

Look for sensitive field names in entity/model files:
```
Grep: password|passwordHash|salt|resetToken|verificationToken|refreshToken|apiKey|secret|privateKey
```

For each sensitive field found, verify it is excluded from responses via the framework-appropriate mechanism.

### Step 6: Permission Check Utility Analysis (NEW)

Analyze how permission checks are implemented across the frontend codebase for consistency.

**6a. Detect permission check patterns used:**

| Pattern Type | Examples | Assessment |
|-------------|---------|------------|
| Custom hook | `useAuth()`, `usePermission()`, `useRole()`, `useAbility()` | Good - centralized |
| Context provider | `AuthContext`, `PermissionContext`, `AbilityContext` | Good - shared state |
| HOC | `withAuth(Component)`, `withRole('admin')(Component)` | Good - reusable |
| Vue directive | `v-permission`, `v-role`, `v-can` | Good - declarative |
| Angular directive | `*appHasRole`, `*appCan` | Good - declarative |
| Inline check | `user.role === 'admin'` directly in JSX/template | Poor - not reusable |
| CASL ability | `useAbility()` + `can('action', 'subject')` | Good - fine-grained |

**6b. Check consistency across components:**

Scan all components that perform role/permission checks. Flag if:
- Multiple different patterns are used (e.g., some components use `useAuth()` hook, others use inline `user.role` checks, others use HOCs)
- Mixed casing in role names (e.g., `'admin'` vs `'Admin'` vs `'ADMIN'`)
- Some use enums/constants while others use string literals (`Role.ADMIN` vs `'admin'`)
- Different hooks used for the same purpose (`useAuth()` vs `usePermission()`)

**6c. Trace permission data source:**

| Source | Pattern | Freshness Risk |
|--------|---------|----------------|
| JWT decode | `jwtDecode(token).role` | Stale until token refreshed |
| API call | `GET /me` -> user object with roles | Fresh on each call |
| Session | `useSession()` (NextAuth), `getServerSession()` | Fresh per session |
| Context | `AuthContext.user.role` | Fresh if context updates on auth change |
| Store | `useAuthStore().user.role` | Fresh if store syncs with auth state |
| Cookie | `document.cookie` parse | Stale, security risk if not httpOnly |

**6d. Staleness check:**

| Check | Description | Severity |
|-------|-------------|----------|
| Token refresh on role change | If admin changes a user's role, does the user's token update? | CRITICAL if no refresh |
| Session revalidation | How often is the session/token revalidated? | WARNING if never |
| Permission cache duration | Are permissions cached? For how long? | INFO |

### Step 7: Permission Definition Sync Analysis (NEW)

Compare role/permission names and structures across the full stack.

**7a. Extract role definitions from each layer:**

| Layer | Where to Look |
|-------|--------------|
| Frontend constants | Role enums, constants, types (e.g., `Role.ADMIN`, `ROLES.EDITOR`) |
| Frontend inline | String literals in conditional rendering (e.g., `'admin'`, `'editor'`) |
| JWT payload | Token decode logic, auth context type definitions |
| Backend decorators | Role guard values (e.g., `@Roles('admin')`, `@Secured("ROLE_ADMIN")`) |
| Backend enums | Role enum definitions in entities/models |
| Database | Column type, ENUM values, migration files |

**7b. Cross-layer comparison:**

Verify:
- Same role names across all layers (case-sensitive!)
- Same permission granularity (role-based vs permission-based)
- Single role vs multi-role model consistent
- Inherited permissions handled (admin inherits editor permissions?)
- Role format consistency (e.g., `ROLE_ADMIN` in Spring vs `admin` in frontend)

Flag any mismatches as Check 12 findings.

### Step 8: UI Element Visibility Matrix (NEW)

Build a per-role visibility matrix for all role-gated UI elements.

**8a. Find all conditionally rendered elements based on role/permission:**

Search patterns by framework:
- **React**: `{user.role === '...' && <Component />}`, `{hasPermission('...') && ...}`, `{can('action', 'subject') && ...}`
- **Vue**: `v-if="user.role === '...'"`, `v-permission="'...'"`, `v-role="'...'"`, `v-if="can('...', '...')"`
- **Angular**: `*ngIf="authService.hasRole('...')"`, `*appHasRole="'...'"`, `*appCan="'...'"`

**8b. Build the visibility matrix:**

For each conditionally rendered element:

| Property | Description |
|----------|-------------|
| `element` | Component/button/link being conditionally rendered |
| `action` | What the element does (delete, edit, create, view, export) |
| `condition` | The role/permission check expression |
| `visibleTo` | Roles that can see it |
| `hiddenFrom` | Roles that cannot see it |
| `apiEndpoint` | The API call this element triggers (if traceable) |
| `backendGuard` | Whether the corresponding backend endpoint has a matching role guard |
| `sourceLocation` | `file:line` reference |

**8c. Anti-pattern detection:**

| Pattern | Example | Severity | Why Bad |
|---------|---------|----------|---------|
| Hardcoded role string | `role === 'admin'` | WARNING | Typo-prone, not refactor-safe |
| Negative role check | `role !== 'viewer'` | WARNING | Breaks when new roles are added |
| Multiple inconsistent checks | `role === 'admin'` in one place, `hasRole('ADMIN')` in another | WARNING | Casing/naming inconsistency |
| No centralized permission util | Ad-hoc `user.role` checks scattered | INFO | Hard to maintain and audit |

**8d. Cross-reference with backend:**

For each element in the visibility matrix, verify the corresponding backend endpoint has a matching role restriction. If the frontend hides an element from certain roles but the backend allows those roles to call the API, flag as Check 13.

### Step 9: Run All 15 Checks

Now execute each check using the data gathered in Steps 1-8.

#### Check 1: Unprotected Endpoint (CRITICAL)

If global guard exists:
- Review all endpoints marked as public. For each one ask: should this really be public?
- Flag any data-modifying endpoint (POST, PUT, PATCH, DELETE) that is public unless it is clearly an auth endpoint (login, register, forgot-password).
- Flag any endpoint returning sensitive data that is public.

If no global guard (per-controller mode):
- Flag every endpoint method that does NOT have an auth guard and is NOT marked public.
- These are genuinely unprotected and accessible to anyone.

#### Check 2: Missing Role Guard (HIGH)

Apply heuristic role expectations based on controller path and operation type:

| Controller/Path Pattern | Expected Roles | Write Operations |
|------------------------|----------------|------------------|
| `/admin/*` | Admin-level roles | All |
| `/users` (write ops) | Admin-level roles | POST, PUT, PATCH, DELETE |
| `/users` (read own) | Any auth | GET own profile |
| Resource write endpoints | Content-creator roles | POST, PUT, PATCH, DELETE |
| Resource read endpoints | Any auth | GET |
| DELETE endpoints | Restricted roles | DELETE |

Use the roles defined in CLAUDE.md to determine expected role restrictions. For each endpoint, compare actual role guards against expected roles. Flag mismatches.

Also flag any DELETE endpoint that has no role restriction (any authenticated user can delete).

#### Check 3: Unprotected Frontend Route (HIGH)

For each frontend route that is NOT in the known public routes list (login, register, forgot-password, public landing):
- Check if it is wrapped in the auth protection component
- If not wrapped and not public, flag as unprotected

Known public routes (should NOT require auth):
- `/login`, `/signin`
- `/register`, `/signup`
- `/forgot-password`, `/reset-password`
- `/` (landing page, if it exists as public)

Everything else should require authentication.

#### Check 4: Role Mismatch Frontend vs Backend (HIGH)

Cross-reference the backend auth matrix with frontend UI:

For each backend endpoint that has role restrictions:
1. Identify the corresponding frontend UI element (button, form, page section, menu item)
2. Check if the frontend conditionally renders that element based on user role
3. If the frontend shows the UI to all authenticated users but the backend restricts to specific roles, flag it

A mismatch means: user sees a button, clicks it, gets a 403 error. Bad UX and potential information leak.

#### Check 5: Missing Ownership Check (MEDIUM)

For endpoints that operate on user-specific resources, verify ownership validation:

Resources that MUST have ownership checks:
- Any user-scoped resource (e.g., logs, profiles, messages, surveys, notifications)
- Resource updates: user should only update own data (unless admin role)

For each such endpoint:
1. Check if the current user is injected in the method signature
2. Check if the service method compares `resource.userId === currentUser.id` or similar
3. If no ownership check exists, flag it

#### Check 6: Token Exposure (CRITICAL)

From Step 4 data, flag any instance of:
- `localStorage.setItem` with token/jwt/auth key names
- `sessionStorage.setItem` with token/jwt/auth key names
- Token values logged to console
- Token passed in URL query parameters
- Token stored in Redux/state that persists to localStorage

Expected secure pattern:
- Backend sets HTTP-only cookie (project-configured cookie name)
- Frontend sends `withCredentials: true` on API calls
- No JavaScript access to the token at all

#### Check 7: Missing CORS Config (MEDIUM)

From Step 1 data, evaluate CORS posture.

Flag if:
- CORS allows all origins
- `credentials: true` with wildcard origin (browser will block this anyway, but it indicates misconfiguration)
- No CORS configuration at all (may break legitimate cross-origin requests)

Expected secure pattern:
- `origin` set to specific frontend URLs (auto-detected from project config)
- `credentials: true` to allow cookie transmission

#### Check 8: Sensitive Data in Response (MEDIUM)

From Step 5 data, flag any sensitive field in an entity/model that:
- Does NOT have the framework-appropriate exclusion mechanism
- Is NOT excluded via query selection
- Is NOT filtered out in a response DTO/serializer

Common sensitive fields to check:
- `password`, `passwordHash`, `hash`
- `salt`
- `resetToken`, `resetPasswordToken`
- `verificationToken`, `emailVerificationToken`
- `refreshToken`
- `apiKey`, `secretKey`
- `internalNotes`, `adminComments`

Also verify that the framework's serialization/exclusion mechanism is properly activated (e.g., serializer interceptor, hidden attributes, JSON ignore).

#### Check 9: Missing Rate Limiting (MEDIUM)

From Step 1 data, check rate limiting on critical auth endpoints:

Endpoints that MUST have rate limiting:
- Login endpoint - prevents brute force password attacks
- Registration endpoint - prevents mass account creation
- Forgot-password endpoint - prevents email/SMS bombing
- Reset-password endpoint - prevents brute force token guessing
- Email verification endpoint - prevents verification abuse
- OTP endpoint - prevents OTP bombing

If global rate limiting exists, check if auth endpoints have stricter limits (lower threshold).
If no rate limiting at all, flag ALL auth endpoints as missing rate limiting.

**Critical Infrastructure Check:**

Beyond checking for rate limit decorators on endpoints, verify the rate limiting infrastructure is actually active:

**NestJS:**
```
Grep: "APP_GUARD.*ThrottlerGuard|useClass.*ThrottlerGuard" in app.module.ts or core module
```
If `ThrottlerModule.forRoot()` is imported but `ThrottlerGuard` is NOT registered as `APP_GUARD`, flag as **CRITICAL** — all `@Throttle()` decorators are effectively dead code.

**Express:**
```
Grep: "rateLimit|express-rate-limit|rate-limiter" in app setup
```
Verify the rate limiter middleware is actually `app.use()`-d, not just imported.

**Django:**
```
Grep: "DEFAULT_THROTTLE_CLASSES" in settings
```
Verify throttle classes are configured in REST_FRAMEWORK settings.

If rate limiting decorators/annotations exist but the infrastructure guard is not active, escalate severity from MEDIUM to **CRITICAL** — this is worse than having no rate limiting at all, because it creates a false sense of security.

#### Check 10: Frontend Role-Based UI Gap (MEDIUM)

For each backend endpoint that has role restrictions, trace the full flow:

1. Find the API call in frontend service files
2. Find where that service function is called in components
3. Check if the component conditionally renders based on role

If the component renders the UI element for ALL authenticated users but the backend restricts access:
- The user will see the option but get a 403 when they try to use it
- Flag as a role-based UI gap

#### Check 11: Permission Check Inconsistency (MEDIUM)

From Step 6 data, flag if:
- Multiple different permission check utilities are used inconsistently across components (e.g., some use hooks, others use HOCs, others use inline checks)
- The same permission is checked differently in different places
- No centralized permission check utility exists (all inline ad-hoc checks)

Include specifics: which files use which pattern, and recommend consolidating to a single pattern.

#### Check 12: Permission Name Mismatch (HIGH)

From Step 7 data, flag if:
- Frontend role/permission names don't match backend definitions (e.g., frontend uses `'Admin'` but backend expects `'admin'` or `'ROLE_ADMIN'`)
- Role enum values in frontend don't align with backend enum/string values
- JWT payload role format differs from what frontend code checks against
- Database role values differ from application-level role names

This is HIGH severity because mismatched names mean permission checks silently fail (always deny or always allow).

#### Check 13: Missing Visibility Matrix (MEDIUM)

From Step 8 data, flag UI elements that:
- Are NOT conditionally rendered based on role when the corresponding backend endpoint restricts access
- Are visible to all authenticated users but trigger API calls that return 403 for certain roles
- Have no role-based conditional rendering at all despite backend role restrictions

For each finding, specify: which element, which roles should NOT see it, and which backend endpoint restricts access.

#### Check 14: Rate Limiter Infrastructure Inactive (CRITICAL)

This check is derived from the infrastructure verification in Check 9 but is reported separately due to its critical nature.

From Step 1h data, determine if rate limiting infrastructure is actually functional:

**Verification steps:**

1. Check if rate limiting module/package is imported in the application configuration
2. Check if the corresponding guard/middleware is registered globally (not just imported)
3. Check if individual endpoint decorators reference the rate limiter

**Flag as CRITICAL if:**
- Rate limiting module is imported (e.g., `ThrottlerModule.forRoot()`) AND
- Rate limiting decorators exist on endpoints (e.g., `@Throttle()`) AND
- But the guard is NOT registered as a global guard (`APP_GUARD`)

This means developers believed rate limiting was active (they added decorators) but the infrastructure was never wired. All auth endpoints are unprotected from brute-force attacks.

**NestJS specific:** Check `providers` array in `AppModule` or core module for:
```typescript
{ provide: APP_GUARD, useClass: ThrottlerGuard }
```

**Express specific:** Check that `app.use(rateLimit(...))` is called, not just imported.

**Django specific:** Check `DEFAULT_THROTTLE_CLASSES` in `REST_FRAMEWORK` settings.

#### Check 15: Missing Brute-Force Protection (HIGH)

Verify that OTP verification, token validation, and password reset flows have attempt counters and lockout mechanisms.

**15a. Find OTP/token verification endpoints:**
```
Grep: "verifyOtp|verify-otp|validateOtp|validate-otp|resetPassword|reset-password|verifyToken|verify-token|verifyCode|verify-code" in service files
```

**15b. For each verification endpoint, check for attempt tracking:**
```
Grep: "attempts|attemptCount|tryCount|failedAttempts|retryCount|maxAttempts|MAX_ATTEMPTS" in the service method
```

**15c. Verification checklist for each OTP/token verification flow:**

| Check | Description | Severity if Missing |
|-------|-------------|---------------------|
| Attempt counter exists | `attempts` field incremented on each verification attempt | HIGH |
| Maximum attempt limit | Check against max (e.g., 5 attempts) | HIGH |
| Lockout on max attempts | OTP/token deleted or account locked after max attempts | HIGH |
| Counter applies to ALL verification paths | Same OTP may be verified via different endpoints (e.g., `verifyOtp` AND `resetPassword`) — both must check attempts | CRITICAL |

**Flag if:**
- OTP/token verification exists but has no attempt counter
- Attempt counter exists in one verification path but not another (e.g., `verifyPhoneOtp` checks attempts but `resetPassword` bypasses it)
- OTP is not deleted after successful use (allows replay)
- OTP expiry is checked but attempt limit is not

**15d. Also check for password-related brute-force protection:**
- Login attempt tracking (separate from rate limiting — account-level lockout after N failed attempts)
- Password change endpoint accepting old password without attempt limit

### Step 10: Compile Report

Compile all findings into the output format below. Group by severity. Include file paths and line numbers for every finding.

Count totals per check and per severity level.

## Output Format

```markdown
# QA Auth Audit Report

**Date**: YYYY-MM-DD
**Scope**: All modules | {specified module}
**Backend Framework**: {auto-detected}
**Frontend Framework**: {auto-detected}
**Auth Library**: {auto-detected}
**Global Auth**: Yes ({framework-specific guard}) | No (per-controller)
**Token Storage**: HTTP-only cookie | localStorage | sessionStorage | Mixed
**CORS Mode**: Restricted (specific origins) | Open (all origins) | Not configured
**Rate Limiting**: Global | Partial | None

## Summary

| Severity | Check | Issues Found |
|----------|-------|-------------|
| CRITICAL | 1. Unprotected endpoint | X |
| CRITICAL | 6. Token exposure | X |
| HIGH | 2. Missing role guard | X |
| HIGH | 3. Unprotected frontend route | X |
| HIGH | 4. Role mismatch frontend<>backend | X |
| HIGH | 12. Permission name mismatch | X |
| MEDIUM | 5. Missing current-user validation | X |
| MEDIUM | 7. Missing CORS config | X |
| MEDIUM | 8. Sensitive data in response | X |
| MEDIUM | 9. Missing rate limiting | X |
| MEDIUM | 10. Frontend role-based UI gap | X |
| MEDIUM | 11. Permission check inconsistency | X |
| MEDIUM | 13. Missing visibility matrix | X |
| CRITICAL | 14. Rate limiter infrastructure inactive | X |
| HIGH | 15. Missing brute-force protection | X |
| | **Total** | **X** |

## Backend Auth Coverage Matrix

| Endpoint | Method | Auth | Roles | Public | Current User | Status |
|----------|--------|------|-------|--------|--------------|--------|
| POST /auth/login | login | N/A | - | Yes | No | OK (public) |
| GET /users | findAll | Yes | ADMIN | No | No | OK |
| POST /exercises | create | Yes | - | No | Yes | WARNING: No role |
| GET /admin/stats | getStats | No | - | No | No | CRITICAL: UNPROTECTED |

## Frontend Route Protection Matrix

| Route | App | Protected | Role Check | Status |
|-------|-----|-----------|------------|--------|
| /login | frontend | Public | - | OK |
| /exercises | frontend | Yes | - | OK |
| /admin/users | dashboard | Yes | ADMIN | OK |
| /admin/settings | dashboard | No | - | CRITICAL: UNPROTECTED |

## Permission Check Consistency Matrix

| File | Pattern Used | Role Format | Centralized | Status |
|------|-------------|-------------|-------------|--------|
| UserList.tsx | useAuth() hook | Role.ADMIN | Yes | OK |
| Settings.tsx | inline user.role === 'admin' | string literal | No | WARNING: inconsistent |
| Dashboard.tsx | withRole HOC | Role.ADMIN | Yes | WARNING: different pattern |

## Permission Name Sync Matrix

| Layer | Role Name | Format | Match Status |
|-------|-----------|--------|-------------|
| Frontend enum | Role.ADMIN | PascalCase constant | - |
| Frontend inline | 'admin' | lowercase string | MISMATCH with enum |
| JWT payload | 'ADMIN' | uppercase string | - |
| Backend guard | 'ADMIN' | uppercase string | OK |
| Database | 'ADMIN' | uppercase string | OK |

## UI Element Visibility Matrix

| Element | Action | Visible To | Hidden From | API Endpoint | Backend Guard | Match |
|---------|--------|-----------|-------------|-------------|---------------|-------|
| Delete Button | delete | admin | all others | DELETE /users/:id | ADMIN | OK |
| Edit Button | update | admin, editor | viewer | PATCH /items/:id | No guard | LEAK |
| Export Button | export | all | - | GET /export | No guard | OK (no restriction) |

---

## CRITICAL Issues

### Check 1: Unprotected Endpoints

| # | Endpoint | Method | Controller | File | Line |
|---|----------|--------|------------|------|------|
| 1 | GET /admin/stats | getStats | AdminController | path/to/admin.controller.ts | 42 |

**Risk**: Anyone on the internet can call this endpoint without any authentication.
**Fix**: Add auth guard or ensure global guard covers this controller.

### Check 6: Token Exposure

| # | Storage | Key | File | Line | Risk |
|---|---------|-----|------|------|------|
| 1 | localStorage | access_token | path/to/auth.ts | 15 | XSS can steal token |

**Risk**: Cross-site scripting (XSS) attacks can read localStorage and steal the auth token.
**Fix**: Move to HTTP-only cookie set by the backend. Remove all localStorage token operations.

---

## HIGH Issues

### Check 2: Missing Role Guards

| # | Endpoint | Expected Role | Current | File | Line |
|---|----------|---------------|---------|------|------|
| 1 | DELETE /users/:id | ADMIN | Any auth | path/to/user.controller.ts | 88 |

**Risk**: Any authenticated user (including low-privilege roles) can perform this action.
**Fix**: Add role restriction decorator to the method.

### Check 3: Unprotected Frontend Routes

| # | Route | App | Should Require | File |
|---|-------|-----|----------------|------|
| 1 | /admin/settings | dashboard | Auth + ADMIN | path/to/routes.tsx |

**Risk**: Users can navigate directly to this page without logging in.
**Fix**: Wrap route in auth protection component with role check.

### Check 4: Role Mismatches (Frontend vs Backend)

| # | Endpoint | Backend Roles | Frontend Check | Gap |
|---|----------|---------------|----------------|-----|
| 1 | DELETE /exercises/:id | ADMIN, COACH | None (visible to all) | Low-privilege user sees delete button, gets 403 |

**Risk**: User sees functionality they cannot use. Potential information disclosure.
**Fix**: Conditionally render based on user role in the component.

### Check 12: Permission Name Mismatches

| # | Frontend Name | Backend Name | Layer | File | Line |
|---|--------------|-------------|-------|------|------|
| 1 | 'Admin' | 'ADMIN' | Role check | path/to/component.tsx | 42 |

**Risk**: Case-sensitive comparison fails silently. Permission check always denies or always allows.
**Fix**: Use consistent role name format across all layers. Prefer enum/constants over string literals.

---

## MEDIUM Issues

### Check 5: Missing Ownership Checks

| # | Endpoint | Resource | Current User | Ownership Validated | File | Line |
|---|----------|----------|--------------|---------------------|------|------|
| 1 | PATCH /exercise-logs/:id | ExerciseLog | Yes | No | path/to/service.ts | 34 |

**Risk**: User A can modify User B's data by guessing/knowing the ID.
**Fix**: Add ownership validation in service layer.

### Check 7: CORS Configuration

| # | Issue | Current Setting | File | Line |
|---|-------|-----------------|------|------|
| 1 | Wildcard origin | `origin: '*'` | path/to/main.ts | 12 |

**Risk**: Any website can make authenticated requests to the API if credentials are included.
**Fix**: Set `origin` to specific frontend URLs (auto-detected from project config).

### Check 8: Sensitive Data in Response

| # | Entity | Field | Excluded | Serializer Active | File | Line |
|---|--------|-------|---------|-------------------|------|------|
| 1 | User | password | No | No | path/to/user.entity.ts | 22 |

**Risk**: Password hash returned in API responses. Aids offline brute force attacks.
**Fix**: Apply framework-appropriate field exclusion mechanism.

### Check 9: Missing Rate Limiting

| # | Endpoint | Has Rate Limit | File | Line |
|---|----------|---------------|------|------|
| 1 | POST /auth/login | No | path/to/auth.controller.ts | 18 |

**Risk**: Attacker can attempt unlimited password guesses.
**Fix**: Add rate limiting to auth endpoints.

### Check 10: Frontend Role-Based UI Gaps

| # | Backend Endpoint | Backend Roles | Frontend Component | Role Check Present | File |
|---|-----------------|---------------|--------------------|--------------------|------|
| 1 | POST /exercises | COACH, ADMIN | ExerciseList.tsx | No | path/to/ExerciseList.tsx |

**Risk**: Low-privilege users see action buttons but get 403 when clicking.
**Fix**: Conditionally render based on `user.role`.

### Check 11: Permission Check Inconsistencies

| # | Pattern A | File A | Pattern B | File B | Issue |
|---|-----------|--------|-----------|--------|-------|
| 1 | useAuth() hook | UserList.tsx | inline user.role | Settings.tsx | Different patterns for same purpose |

**Risk**: Inconsistent patterns make auditing difficult and increase chance of bugs.
**Fix**: Consolidate all permission checks to use a single centralized utility (preferably a hook or directive).

### Check 13: Missing Visibility Matrix Entries

| # | UI Element | Backend Endpoint | Backend Roles | Role Rendering | File |
|---|-----------|-----------------|---------------|----------------|------|
| 1 | Create button | POST /items | ADMIN, EDITOR | Visible to all auth users | path/to/ItemList.tsx |

**Risk**: Users see actions they cannot perform. Results in 403 errors and poor UX.
**Fix**: Add conditional rendering based on user role to match backend restrictions.

---

## Recommendations

1. **[Priority 1]** Fix all CRITICAL issues first — unprotected endpoints and token exposure are immediate security risks.
2. **[Priority 2]** Fix permission name mismatches (Check 12) — these cause silent permission failures.
3. **[Priority 3]** Add missing role guards to sensitive endpoints (user management, admin operations).
4. **[Priority 4]** Protect all frontend routes with auth guard components.
5. **[Priority 5]** Add ownership validation to user-specific resource endpoints.
6. **[Priority 6]** Consolidate permission check patterns to a single utility (Check 11).
7. **[Priority 7]** Build complete UI visibility matrix matching backend role restrictions (Check 13).
8. **[Priority 8]** Configure rate limiting on auth endpoints to prevent brute force.
9. **[Priority 9]** Ensure all sensitive fields are excluded from API responses.
10. **[Priority 10]** Align frontend role-based UI with backend role restrictions.
```

## Scanning Patterns Reference

Quick reference for grep/glob patterns used throughout this audit. Adapt file globs to auto-detected project paths.

| What to Find | Regex Pattern | File Glob |
|------|---------|-------|
| Global guard registration (NestJS) | `APP_GUARD\|useGlobalGuards` | backend/src/**/*.module.ts |
| Global guard registration (Express) | `passport\.authenticate\|app\.use.*auth` | server/**/*.ts, server/**/*.js |
| Global guard registration (Spring) | `@EnableGlobalMethodSecurity\|SecurityFilterChain` | src/**/*.java |
| Global guard registration (Django) | `REST_FRAMEWORK.*DEFAULT_PERMISSION` | settings.py, settings/**/*.py |
| Auth guard on controller/method | Framework-dependent (see detection table) | Controller/handler files |
| Public endpoint marker | Framework-dependent (see detection table) | Controller/handler files |
| Role restriction | Framework-dependent (see detection table) | Controller/handler files |
| Current user injection | Framework-dependent (see detection table) | Controller/handler files |
| HTTP method decorators | `@(Get\|Post\|Put\|Patch\|Delete)\(` or framework equivalent | Controller/handler files |
| Token in localStorage | `localStorage\.(set\|get)Item` | frontend/**/*.ts, frontend/**/*.tsx |
| Token in sessionStorage | `sessionStorage\.(set\|get)Item` | frontend/**/*.ts, frontend/**/*.tsx |
| Cookie handling | `document\.cookie\|httpOnly\|withCredentials` | frontend/**/*.ts, backend/**/*.ts |
| CORS configuration | `enableCors\|CorsOptions\|ALLOW_ORIGINS\|cors\(` | Backend entry point |
| Rate limiting | Framework-dependent rate limit patterns | Backend source files |
| Field exclusion | Framework-dependent exclusion patterns | Entity/model files |
| Serializer/interceptor | Framework-dependent serialization config | Backend source files |
| Protected route component | `ProtectedRoute\|PrivateRoute\|RequireAuth\|AuthGuard` | Frontend route files |
| Frontend role checks | `role.*===\|hasRole\|checkRole\|isAdmin\|hasPermission\|can\(` | Frontend component files |
| Ownership validation | `userId.*===\|belongsTo\|checkOwnership\|isOwner` | Backend service files |
| Sensitive fields | `password\|passwordHash\|salt\|resetToken\|secretKey\|apiKey` | Entity/model files |
| Permission hooks | `useAuth\|usePermission\|useRole\|useAbility` | Frontend component files |
| Permission HOCs | `withAuth\|withRole\|withPermission` | Frontend component files |
| Permission directives | `v-permission\|v-role\|v-can\|\*appHasRole\|\*appCan` | Frontend template files |
| Role enums/constants | `enum.*Role\|ROLES\.\|Role\.\|ROLE_` | Frontend and backend source files |

## Edge Cases to Watch For

1. **Base class / inherited methods**: If a controller extends a base class, inherited CRUD methods may not have decorators from the child class. Verify that class-level decorators on the child controller apply to inherited methods.

2. **WebSocket / real-time auth**: WebSocket gateways (Socket.IO, WS, SignalR) should validate auth tokens during the handshake phase. Check for connection middleware that verifies tokens.

3. **File upload endpoints**: Endpoints that accept file uploads (multipart/form-data) sometimes bypass auth middleware due to middleware ordering. Verify file upload endpoints are still protected.

4. **API documentation endpoint**: Swagger/OpenAPI docs endpoints should be protected or disabled in production. Check if documentation is conditionally enabled.

5. **Health check endpoint**: Typically at `/health` or `/`. Should be public but should not leak system info.

6. **Redirect-based auth bypass**: Check if any endpoint reads `returnUrl` or `redirect` parameters that could be manipulated for open redirect attacks.

7. **Separate frontend deployments**: If multiple frontend apps exist (e.g., user-facing + admin dashboard), verify each has its own auth checks and does not rely solely on another app's auth state.

8. **Middleware chain ordering**: Auth middleware placed after route handler creates a security gap. Verify ordering is correct for the detected framework.

9. **Token expiry during form fill**: Submit fails due to expired token. Check for graceful handling.

10. **Role changes mid-session**: Admin changes a user's role while user is logged in. Check if token/session is refreshed.

11. **Negative permission patterns**: `role !== 'viewer'` breaks silently when new roles are added. Use positive inclusion.

12. **Enum vs string serialization**: Frontend uses `Role.ADMIN` but JWT contains `'admin'`. Check if serialization preserves the expected format.

13. **Rate limiter decorator without global guard**: The most dangerous false positive — developers see `@Throttle()` decorators and believe endpoints are protected, but without the global guard registration, decorators are inert. Always verify the infrastructure layer, not just the application layer.

14. **Multiple OTP verification paths**: The same OTP may be verifiable through different code paths (e.g., a dedicated verify endpoint AND a reset-password endpoint that verifies inline). Brute-force protection must cover ALL paths, not just the primary one.

## Related Skills

- `qa-inputs` — Input fields (form validation per role)
- `qa-back-nav` — Back navigation (auth redirect uses replace)
- `qa-states` — Loading/error/empty states (403 error state)
- `qa-modal` — Modals (create/edit modals may be role-gated)
- `qa-list` — Tables/lists (row actions may be role-gated, data filtered by role)
- `qa-buttons` — Buttons (role-gated action buttons)
- `qa-api-sync` — API sync (auth header verification)

## READ-ONLY Diagnostic

This skill is a READ-ONLY diagnostic. It does NOT modify any files. It scans, analyzes, and reports findings. All recommended fixes are presented as suggestions in the report for the developer to implement.
