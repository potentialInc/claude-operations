---
description: Analyze permission and role-based access control in a page/component and generate a comprehensive QA report covering route protection, UI element visibility, API authorization matching, and permission check patterns
argument-hint: "<page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow]"
---

# QA Permission / Role

Analyze permission and role-based access control in a frontend page or component, trace authorization rules across the full stack (Frontend UI → Route Guard → Backend API Guard), and generate a comprehensive QA report with test cases.

---

## Purpose

This command helps you:
1. **Detect route protection** — Find auth guards, role guards, and redirect behavior for the page
2. **Audit UI element visibility** — Identify buttons, links, and sections conditionally rendered by role
3. **Match frontend-backend authorization** — Verify every hidden UI action also has a backend guard
4. **Analyze permission check patterns** — Hooks, HOCs, directives, and their consistency
5. **Detect permission leaks** — Frontend hides an element but backend allows the API call
6. **Verify data-level filtering** — List APIs filter by user's scope/role/ownership
7. **Check permission definition sync** — Same role/permission names across frontend and backend
8. **Generate test cases** — Produce actionable QA test checklists per role

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack authorization tracing
- (Optional) Project `CLAUDE.md` with project-specific role definitions

---

## Usage

```bash
/qa-permission-role <page-or-component-file-path>
```

**Examples:**
```bash
# React page with role-based UI
/qa-permission-role src/pages/admin/UserManagement.tsx

# With backend tracing
/qa-permission-role frontend/app/pages/settings.tsx --backend backend/src

# Frontend-only analysis
/qa-permission-role src/pages/Dashboard.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): trace through backend API guards. `shallow`: frontend-only analysis |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection                │
│  - Validate file path and extension                          │
│  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)│
│  - Detect auth library (NextAuth/CASL/Firebase/Clerk/etc.)   │
│  - Detect backend framework (if --depth full)                │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Route Protection Analysis                           │
│  - Detect route-level auth guards and role guards            │
│  - Verify redirect behavior (replace, target page)           │
│  - Check loading state during permission check               │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: UI Element Visibility Analysis                      │
│  - Find all role/permission conditional rendering            │
│  - Build visibility matrix: element × role                   │
│  - Detect hardcoded strings and negative patterns            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Permission Check Utility Analysis                   │
│  - Detect hooks, HOCs, directives, context patterns          │
│  - Trace permission data source (JWT, API, context)          │
│  - Check consistency and staleness                           │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: API Authorization Matching (Frontend ↔ Backend)     │
│  - For each hidden action, find the corresponding API guard  │
│  - Build frontend-backend comparison matrix                  │
│  - Detect permission leaks (frontend-only protection)        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Data-Level & Action-Level Permission Analysis       │
│  - Check list API filtering by user scope/ownership          │
│  - Map CRUD actions to required roles                        │
│  - Verify permission definition sync across stack            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Generate QA Report                                  │
│  - Route protection, visibility matrix, auth comparison      │
│  - Permission check audit, data-level filtering              │
│  - Issues, test cases, recommendations                       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 8: Save & Present Results                              │
│  - Save report to .claude-project/qa/                        │
│  - Display summary with actionable items                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Execution Rule: Self-Verification & Auto-Retry

**Every Step MUST follow this rule:**

1. After completing a Step, output the Step's checklist
2. If any item is unchecked (`[ ]`), automatically re-execute ONLY the missing items
3. After re-execution, output the checklist again
4. Repeat until all items are `[x]` (maximum 3 retries)
5. If still incomplete after 3 retries, mark as `[SKIP]` with reason and proceed

**Output format for each Step:**
```
━━━ Step N Complete ━━━
[x] Item 1
[x] Item 2
[ ] Item 3 ← Missing detected, retrying...

━━━ Step N Retry 1/3 ━━━
[x] Item 3 ← Completed

✅ Step N: ALL PASSED (N/N) → Proceeding to Step N+1
```

---

## Execution Steps

### Step 1: Input Validation & Project Detection

Read the file path from `$ARGUMENTS`. Parse optional flags (`--backend`, `--depth`).

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-permission-role <page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow]
```

**1.2 Framework Detection:**

Read the target file and project's `package.json`:

| Signal | Framework |
|--------|-----------|
| `import ... from 'react'` or JSX/TSX + `react` in package.json | React |
| `<template>` + `<script>` + `.vue` extension | Vue |
| `@Component` decorator + `@angular/core` in package.json | Angular |
| `.svelte` extension | Svelte |
| `<form>` + no framework imports + `.html` extension | Plain HTML |

**1.3 Auth/Permission Library Detection:**

| Signal | Library |
|--------|---------|
| `next-auth` / `@auth/core` in package.json | NextAuth / Auth.js |
| `@casl/ability` / `@casl/react` / `@casl/vue` | CASL |
| `firebase/auth` in package.json | Firebase Auth |
| `@clerk/nextjs` / `@clerk/clerk-react` | Clerk |
| `@supabase/supabase-js` + `.auth` usage | Supabase Auth |
| `@auth0/auth0-react` / `@auth0/nextjs-auth0` | Auth0 |
| Custom: `useAuth()`, `usePermission()`, `AuthContext` | Custom auth hook/context |
| `@supabase/ssr` + RLS policies in SQL | Supabase with Row-Level Security |
| `next-auth@5` or `@auth/nextjs` with `auth()` helper | NextAuth v5 / Auth.js v5 |
| `@clerk/nextjs` + `currentUser()` in Server Components | Clerk (Server Component auth) |

**1.4 Backend Detection** (if `--depth full`):

If `--backend` provided, use that path. Otherwise, look for sibling `backend/` directory.

| Signal | Framework |
|--------|-----------|
| `@nestjs/common` in package.json | NestJS |
| `express` in package.json (no nest) | Express |
| `manage.py` in root | Django |
| `pom.xml` or `build.gradle` with spring-boot | Spring Boot |
| `composer.json` with laravel | Laravel |

**Step 1 Checklist:**
```
[ ] File path validated and file exists
[ ] File extension is supported
[ ] Frontend framework detected
[ ] Auth/permission library detected
[ ] Backend framework detected (or --depth shallow acknowledged)
```

---

### Step 2: Route Protection Analysis

Detect how the page itself is protected from unauthorized access.

**2.1 Route Guard Detection:**

**React Router:**
- Wrapper component: `<ProtectedRoute>`, `<RequireAuth>`, `<RoleGuard>`
- Inline check: `if (!user) return <Navigate to="/login" replace />`
- Loader: `loader` function that throws redirect on auth failure
- HOC: `withAuth(Component)`, `withRole('admin')(Component)`

**Next.js:**
- Middleware: `middleware.ts` with `NextResponse.redirect()` on auth check
- Layout guard: `app/admin/layout.tsx` with session check
- `getServerSideProps` / `getServerSession` with redirect
- Client-side: `useSession()` + redirect in `useEffect`

**Next.js (App Router with Server Component auth):**
- `auth()` from NextAuth v5 in `layout.tsx` or `page.tsx` — Server-side session check
- `currentUser()` from Clerk in Server Components — direct user object access
- `<SignedIn>` / `<SignedOut>` components from Clerk — conditional rendering in RSC
- `middleware.ts` with `clerkMiddleware()` or `withAuth()` from NextAuth v5

**Vue Router:**
- `router.beforeEach((to, from, next) => { ... })` global guard
- `meta: { requiresAuth: true, roles: ['admin'] }` in route definition
- Per-route guard: `beforeEnter: (to, from) => { ... }`
- Component guard: `onBeforeRouteLeave`, `onBeforeRouteUpdate`

**Angular:**
- `CanActivate` / `canActivate` functional guard
- `CanActivateChild` for child routes
- `canMatch` for lazy-loaded routes
- `resolve` for data pre-loading with auth check

**Nuxt:**
- `defineNuxtRouteMiddleware` with auth check
- `routeRules` in `nuxt.config.ts` with middleware

**2.2 Guard Behavior Verification:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Guard exists | Page has route-level protection | Present / Missing |
| Redirect target | Where unauthorized users go | `/login` / `/403` / `/unauthorized` / other |
| Redirect method | `replace` (correct) vs `push` (back button returns) | Replace / Push (WARNING) |
| Role check | Specific roles allowed | Roles listed / Auth-only (no role check) |
| Loading state | What shows while permission is being checked | Loading / Flash of content / Nothing |

**2.3 Flash of Unauthorized Content (FOUC):**

Check if the page content is visible briefly before the auth redirect:
- `useEffect` redirect: component renders first, THEN redirects (FOUC risk)
- Middleware redirect: server-side, no FOUC
- Loader redirect: no component render until loader resolves
- Suspense + auth check: shows fallback until auth resolves

| Pattern | FOUC Risk | Severity |
|---------|-----------|----------|
| `useEffect` redirect | High | WARNING |
| `if (!user) return <Navigate>` (render-time check) | Low | OK |
| Middleware/loader | None | OK |
| Layout-level auth wrapper | None | OK |

**Step 2 Checklist:**
```
[ ] Route guard detected for the page (or absence flagged)
[ ] Guard type identified (middleware, layout, HOC, inline, useEffect)
[ ] Redirect target and method verified (replace vs push)
[ ] Role/permission check specificity verified
[ ] Flash of unauthorized content risk assessed
[ ] Loading state during auth check verified
```

---

### Step 3: UI Element Visibility Analysis

Find all elements conditionally rendered based on user role or permission.

**3.1 Conditional Rendering Pattern Detection:**

**React:**
- `{user.role === 'admin' && <DeleteButton />}` — direct role check
- `{hasPermission('users:delete') && <DeleteButton />}` — permission check
- `{can('delete', 'User') && <DeleteButton />}` — CASL ability check
- `{session?.user?.role === 'admin' && ...}` — NextAuth session check
- `<Can I="delete" a="User"><DeleteButton /></Can>` — CASL Can component

**Vue:**
- `v-if="user.role === 'admin'"` — direct check in template
- `v-if="can('delete', 'User')"` — CASL ability check
- `v-permission="'users:delete'"` — custom directive
- `v-role="'admin'"` — custom role directive

**Angular:**
- `*ngIf="authService.hasRole('admin')"` — service method check
- `*ngIf="authService.hasPermission('users:delete')"` — permission check
- `*appHasRole="'admin'"` — custom structural directive
- `*appCan="'delete:user'"` — custom permission directive

**3.2 Build Visibility Matrix:**

For each conditionally rendered element:

| Property | Description |
|----------|-------------|
| `element` | Component/button/link being conditionally rendered |
| `action` | What the element does (delete, edit, create, view, export) |
| `condition` | The role/permission check expression |
| `visibleTo` | Roles that can see it |
| `hiddenFrom` | Roles that cannot see it |
| `apiEndpoint` | The API call this element triggers (if traceable) |
| `sourceLocation` | `file:line` reference |

**3.3 Anti-Pattern Detection:**

| Pattern | Example | Severity | Why Bad |
|---------|---------|----------|---------|
| Hardcoded role string | `role === 'admin'` | WARNING | Typo-prone, not refactor-safe |
| Negative role check | `role !== 'viewer'` | WARNING | Breaks when new roles are added |
| Multiple inconsistent checks | `role === 'admin'` in one place, `hasRole('ADMIN')` in another | WARNING | Case sensitivity, naming inconsistency |
| No centralized permission util | Ad-hoc `user.role` checks scattered | INFO | Hard to maintain and audit |
| String literal permissions | `'users:delete'` not from constants | INFO | Typo-prone |

**Step 3 Checklist:**
```
[ ] All conditionally rendered elements found
[ ] Visibility matrix built (element, condition, visibleTo, hiddenFrom)
[ ] API endpoint traced for each action element (if possible)
[ ] Hardcoded role strings flagged
[ ] Negative role check patterns flagged
[ ] Inconsistent permission check patterns flagged
```

---

### Step 4: Permission Check Utility Analysis

**4.1 Permission Check Pattern Detection:**

| Pattern Type | Examples | Assessment |
|-------------|---------|------------|
| Custom hook | `useAuth()`, `usePermission()`, `useRole()`, `useAbility()` | Good — centralized |
| Context provider | `AuthContext`, `PermissionContext`, `AbilityContext` | Good — shared state |
| HOC | `withAuth(Component)`, `withRole('admin')(Component)` | Good — reusable |
| Vue directive | `v-permission`, `v-role`, `v-can` | Good — declarative |
| Angular directive | `*appHasRole`, `*appCan` | Good — declarative |
| Inline check | `user.role === 'admin'` directly in JSX | Poor — not reusable |
| CASL ability | `useAbility()` + `can('action', 'subject')` | Good — fine-grained |

**4.2 Permission Data Source Tracing:**

Trace where the user's roles/permissions come from:

| Source | Pattern | Freshness Risk |
|--------|---------|----------------|
| JWT decode | `jwtDecode(token).role` | Stale until token refreshed |
| API call | `GET /me` → user object with roles | Fresh on each call |
| Session | `useSession()` (NextAuth), `getServerSession()` | Fresh per session |
| Context | `AuthContext.user.role` | Fresh if context updates on auth change |
| Store | `useAuthStore().user.role` | Fresh if store syncs with auth state |
| Cookie | `document.cookie` parse | Stale, security risk if not httpOnly |

**4.3 Staleness Check:**

| Check | Description | Severity |
|-------|-------------|----------|
| Token refresh on role change | If admin changes a user's role, does the user's token update? | CRITICAL if no refresh |
| Session revalidation | How often is the session/token revalidated? | WARNING if never |
| Permission cache duration | Are permissions cached? For how long? | INFO |
| Real-time permission update | WebSocket or polling for permission changes? | INFO (nice-to-have) |

**4.4 Consistency Check:**

Is the same permission check pattern used across the entire page?

| Check | Description | Severity |
|-------|-------------|----------|
| Mixed patterns | Some use hook, others use inline check | WARNING |
| Mixed casing | `'admin'` vs `'Admin'` vs `'ADMIN'` | WARNING |
| Enum vs string | Some use `Role.ADMIN`, others use `'admin'` | WARNING |
| Different hooks | Some use `useAuth()`, others use `usePermission()` | INFO |

**Step 4 Checklist:**
```
[ ] Permission check pattern type identified (hook/HOC/directive/inline)
[ ] Permission data source traced (JWT/API/session/context/store)
[ ] Staleness risk assessed (token refresh, session revalidation)
[ ] Consistency checked (same pattern used throughout page)
[ ] Mixed casing or naming inconsistencies flagged
```

---

### Step 5: API Authorization Matching (Frontend ↔ Backend)

This step runs if `--depth full`. For `--depth shallow`, this step is skipped.

**5.1 Frontend Action → API Endpoint Mapping:**

For each UI element hidden by role (from Step 3), trace to its API call:

| Frontend Element | Action | API Call | Backend Guard |
|-----------------|--------|----------|---------------|
| Delete button (admin-only) | Delete user | `DELETE /api/users/:id` | `@Roles('admin')` or none? |
| Edit link (editor+) | Edit item | `PATCH /api/items/:id` | `@Roles('admin','editor')` or none? |
| Export button (manager+) | Export CSV | `GET /api/users/export` | `@Roles('manager','admin')` or none? |

**5.2 Backend Guard Detection:**

**NestJS:**
- `@Roles('admin')` decorator on controller method
- `@UseGuards(RolesGuard)` applied to controller or method
- `@SetMetadata('roles', ['admin'])` + custom guard
- `@Public()` decorator for public endpoints

**Express:**
- `router.delete('/users/:id', requireRole('admin'), controller.delete)` — middleware
- `router.use('/admin', authMiddleware, roleMiddleware('admin'))` — route-level
- `req.user.role` check inside handler

**Django:**
- `@permission_required('app.delete_user')` decorator
- `IsAdminUser` permission class
- `DjangoObjectPermissions` for object-level
- `has_perm()` check in view

**Spring Boot:**
- `@PreAuthorize("hasRole('ADMIN')")` on controller method
- `@Secured("ROLE_ADMIN")` annotation
- `@RolesAllowed("ADMIN")` annotation
- `SecurityFilterChain` URL-pattern rules

**Laravel:**
- `Route::middleware('role:admin')` in routes
- `$this->authorize('delete', $user)` in controller
- `Gate::allows('delete-user')` check
- Policy class: `UserPolicy::delete()`

**5.3 Build Authorization Comparison Matrix:**

| Frontend Element | Hidden From | API Endpoint | Backend Guard | Match? |
|-----------------|-------------|-------------|---------------|--------|
| Delete button | non-admin | DELETE /users/:id | `@Roles('admin')` | MATCH |
| Edit button | viewer | PATCH /items/:id | No guard | **MISMATCH — CRITICAL** |
| Export button | viewer, editor | GET /users/export | `@Roles('admin')` | PARTIAL (missing manager) |

**5.4 Permission Leak Detection:**

| Leak Type | Description | Severity |
|-----------|-------------|----------|
| Frontend-only protection | UI hides button, but API has no guard | CRITICAL |
| Inconsistent role set | Frontend checks `admin`, backend checks `admin,manager` | WARNING |
| Missing endpoint guard | New API endpoint without any auth guard | CRITICAL |
| Role name mismatch | Frontend: `'Admin'`, Backend: `'admin'` or `'ROLE_ADMIN'` | CRITICAL |

**5.5 ABAC & Row-Level Security:**

Beyond role-based checks, detect attribute-based access control patterns:

| Pattern | Detection | Assessment |
|---------|-----------|------------|
| Owner-only actions | `if (item.userId === currentUser.id)` on frontend + backend | OK — attribute-based check |
| Org-scoped access | `where: { orgId: currentUser.orgId }` in queries | OK — tenant isolation |
| Supabase RLS policies | `CREATE POLICY` statements in SQL migrations with `auth.uid()` | OK — database-level enforcement |
| Frontend-only owner check | `{item.userId === user.id && <EditButton />}` but no backend ownership check | CRITICAL — bypassed via direct API |
| Missing RLS on table | Supabase table without RLS enabled (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY`) | CRITICAL — all rows accessible to any authenticated user |

**Supabase RLS Detection (if Supabase project):**
- Check `supabase/migrations/` directory for `CREATE POLICY` statements
- Verify RLS is enabled: `ALTER TABLE tablename ENABLE ROW LEVEL SECURITY;`
- Check policy conditions use `auth.uid()` or `auth.jwt()` for user filtering
- Assessment: CRITICAL if table has no RLS but stores user-specific data

**5.6 Middleware Chain Order Analysis:**

Check that auth middleware runs BEFORE route handlers:

| Framework | Check | Assessment |
|-----------|-------|------------|
| Express | `app.use(authMiddleware)` appears BEFORE route definitions | OK |
| Express | Route defined BEFORE `app.use(authMiddleware)` | CRITICAL — route unprotected |
| NestJS | `@UseGuards(AuthGuard)` at controller level or `APP_GUARD` in module | OK |
| NestJS | Guard on specific method but not controller | WARNING — some methods may lack guard |
| Next.js | `middleware.ts` matcher covers the route | OK |
| Next.js | Route not matched by `middleware.ts` config.matcher | WARNING — middleware skipped |

**Step 5 Checklist:**
```
[ ] Frontend actions mapped to API endpoints
[ ] Backend guards detected for each API endpoint
[ ] Authorization comparison matrix built
[ ] Permission leaks detected (frontend-only protection)
[ ] Role name consistency verified across frontend and backend
[ ] Inconsistent role sets flagged
```

---

### Step 6: Data-Level & Action-Level Permission Analysis

**6.1 Data-Level Filtering:**

Check if list/query APIs filter results by the user's scope:

| Check | Description | Assessment |
|-------|-------------|------------|
| Owner-only data | User sees only their own records | `WHERE userId = :currentUser` / Missing |
| Organization-scoped | User sees only their org's data | `WHERE orgId = :currentOrg` / Missing |
| Role-based scope | Admin sees all, user sees own | Conditional query / Missing |
| Sensitive field masking | SSN, password shown only to admin | Masked / Fully visible |

Detection patterns:
- Backend service: `this.repository.find({ where: { userId: currentUser.id } })`
- GraphQL: `@auth(rules: [{ allow: owner }])` directive
- Prisma: `.findMany({ where: { userId } })`

**6.2 Action-Level Permission Map:**

Build a CRUD permission matrix for the page's main entity:

| Action | Required Role | Frontend Check | Backend Guard | Status |
|--------|--------------|----------------|---------------|--------|
| Create | editor, admin | `can('create', 'User')` | `@Roles('editor','admin')` | MATCH |
| Read | all roles | No check (page accessible) | No guard (public) | OK |
| Update | editor, admin | `role !== 'viewer'` | `@Roles('editor','admin')` | MATCH (but frontend uses negative check) |
| Delete | admin only | `role === 'admin'` | `@Roles('admin')` | MATCH |

**6.3 Permission Definition Sync:**

Compare role/permission names and structures:

| Layer | Role Format | Example |
|-------|------------|---------|
| Frontend | String literal | `'admin'`, `'editor'`, `'viewer'` |
| Frontend (typed) | Enum/constant | `Role.ADMIN`, `ROLES.EDITOR` |
| JWT payload | String or array | `{ role: 'admin' }` or `{ roles: ['admin'] }` |
| Backend | Decorator value | `@Roles('admin')`, `Role.ADMIN` |
| Database | Column value | `role VARCHAR`, `role ENUM('admin','editor','viewer')` |

Verify:
- Same role names across all layers (case-sensitive!)
- Same permission granularity (role-based vs permission-based)
- Single role vs multi-role model consistent
- Inherited permissions handled (admin inherits editor permissions?)

**Step 6 Checklist:**
```
[ ] Data-level filtering checked (owner-only, org-scoped, role-based)
[ ] CRUD permission matrix built (action × role × frontend × backend)
[ ] Sensitive field masking verified
[ ] Permission definition sync checked across layers
[ ] Role inheritance model verified
[ ] Permission naming consistency verified (casing, format)
```

---

### Step 7: Generate QA Report

Compile all analysis results into a structured report.

**Report Sections:**

1. **Header** — Component name, file path, framework, auth library, date, overall score, status
2. **Route Protection** — Guard type, redirect target/method, FOUC risk, loading state
3. **UI Element Visibility Matrix** — Element × condition × visibleTo × hiddenFrom × API endpoint
4. **Permission Check Utility Audit** — Pattern type, data source, staleness, consistency
5. **Frontend-Backend Authorization Matrix** — Frontend check × API endpoint × backend guard × match status
6. **Data-Level Filtering** — Owner-only, org-scoped, sensitive field masking
7. **CRUD Permission Matrix** — Action × role × frontend × backend × status
8. **Permission Definition Sync** — Role names across layers, casing, format
9. **Issues Found** — All issues with severity (CRITICAL / WARNING / INFO)
10. **Test Cases** — Complete test case table
11. **Recommendations** — Prioritized action items

**Step 7 Checklist:**
```
[ ] Route Protection section complete
[ ] UI Element Visibility Matrix complete
[ ] Permission Check Utility Audit complete
[ ] Frontend-Backend Authorization Matrix complete (if --depth full)
[ ] Data-Level Filtering assessment complete (if --depth full)
[ ] CRUD Permission Matrix complete
[ ] Permission Definition Sync check complete
[ ] Issues listed with severity levels
[ ] Test cases listed with expected results
[ ] Recommendations prioritized
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_PermissionRole_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_PermissionRole_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Permission/Role Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_PermissionRole_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_PermissionRole_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React + NextAuth / Vue + CASL / etc.]
- Route protection: [Guard type] → [redirect target]
- Role-gated UI elements: [N] elements across [N] roles
- Frontend-Backend match: [N]/[N] actions have matching backend guards
- Permission leaks: [N] Critical, [N] Warning
- Data-level filtering: [Present/Missing]
- Issues found: [N] Critical, [N] Warning, [N] Info
- Test cases generated: [N]

### Action Required
1. [CRITICAL] [description]
2. [WARNING] [description]
...
```

**Step 8 Checklist:**
```
[ ] Report file saved to .claude-project/qa/
[ ] Test cases file saved (if separate)
[ ] Summary displayed with issue counts and action items
```

---

## Output Format

```
╔════════════════════════════════════════════════════════════════════╗
║                QA Permission / Role Report                         ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                    ║
║  File:          [file-path]                                        ║
║  Framework:     [React + NextAuth]                                 ║
║  Backend:       [NestJS + @Roles guard]                            ║
║  Report Date:   [YYYY-MM-DD HH:MM]                                ║
║  Depth:         [full / shallow]                                   ║
║  Overall Score: [N]/100                                            ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ROUTE PROTECTION                                                  ║
║                                                                    ║
║  Guard Type:    Layout-level auth wrapper                          ║
║  Check:         session?.user exists                               ║
║  Role Check:    role === 'admin' || role === 'manager'             ║
║  Redirect:      /login (replace) ✅                                ║
║  FOUC Risk:     None (layout blocks render) ✅                     ║
║  Loading State: Skeleton while checking ✅                         ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  UI ELEMENT VISIBILITY MATRIX                                      ║
║                                                                    ║
║  | Element       | Action  | Visible To      | Hidden From |      ║
║  |---------------|---------|-----------------|-------------|      ║
║  | Create Button | Create  | admin, manager  | viewer      |      ║
║  | Edit Link     | Update  | admin, manager  | viewer      |      ║
║  | Delete Button | Delete  | admin           | manager,    |      ║
║  |               |         |                 | viewer      |      ║
║  | Export Button | Export  | all roles       | —           |      ║
║  | Settings Tab  | Config  | admin           | all others  |      ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  FRONTEND-BACKEND AUTHORIZATION MATRIX                             ║
║                                                                    ║
║  | Element      | API Endpoint      | Backend Guard    | Match?  |║
║  |--------------|-------------------|------------------|---------|║
║  | Create Btn   | POST /users       | @Roles(admin,mgr)| ✅      |║
║  | Edit Link    | PATCH /users/:id  | @Roles(admin,mgr)| ✅      |║
║  | Delete Btn   | DELETE /users/:id | No guard         | ❌ LEAK |║
║  | Export Btn   | GET /users/export | No guard         | ⚠ INFO  |║
║  | Settings Tab | GET /settings     | @Roles(admin)    | ✅      |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  CRUD PERMISSION MATRIX                                            ║
║                                                                    ║
║  | Action | admin | manager | viewer | Backend Guard    |         ║
║  |--------|-------|---------|--------|------------------|         ║
║  | Create | ✅    | ✅      | ❌     | @Roles(admin,mgr)|         ║
║  | Read   | ✅    | ✅      | ✅     | Auth only        |         ║
║  | Update | ✅    | ✅      | ❌     | @Roles(admin,mgr)|         ║
║  | Delete | ✅    | ❌      | ❌     | ❌ NO GUARD      |         ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  PERMISSION CHECK AUDIT                                            ║
║                                                                    ║
║  Pattern: useAuth() hook → AuthContext → JWT decode                ║
║  Consistency: ✅ All checks use useAuth().hasRole()                ║
║  Staleness: ⚠ JWT not refreshed on role change                    ║
║  Casing: ✅ All lowercase role names                               ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ISSUES FOUND                                                      ║
║                                                                    ║
║  [CRITICAL] PERM-01: DELETE /users/:id has no backend guard.      ║
║    Frontend hides delete button for non-admin, but API accepts    ║
║    DELETE from any authenticated user.                             ║
║    Risk: Any user can delete users via direct API call.           ║
║    Fix: Add @Roles('admin') + @UseGuards(RolesGuard) to          ║
║    UsersController.remove() method.                                ║
║                                                                    ║
║  [WARNING] PERM-02: JWT not refreshed on role change.             ║
║    If admin changes a user's role, the user retains old           ║
║    permissions until token expires (24h).                          ║
║    Fix: Reduce token TTL or implement token blacklist.            ║
║                                                                    ║
║  [WARNING] PERM-03: Export endpoint has no role guard.            ║
║    Frontend shows export to all roles, backend has no guard.      ║
║    Review: Is export intentionally public for all auth users?     ║
║                                                                    ║
║  [INFO] PERM-04: Settings tab check uses string literal           ║
║    `role === 'admin'` instead of constant/enum.                   ║
║    Suggestion: Use Role.ADMIN for type safety.                    ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  TEST CASES ([N] total)                                            ║
║    Route: [N] | UI Visibility: [N] | API Auth: [N] | Data: [N]   ║
║    [See full list in test cases file]                              ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  RECOMMENDATIONS                                                   ║
║  1. [Critical] Add @Roles('admin') to DELETE /users/:id           ║
║  2. [Warning] Implement token refresh on role change               ║
║  3. [Warning] Review export endpoint — add guard if needed        ║
║  4. [Info] Replace hardcoded role strings with Role enum           ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Test Case Categories

### Category 1: Route Protection Tests (ROUTE-PERM-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| ROUTE-PERM-01 | Unauthenticated access | 1. Clear session 2. Navigate to page URL | Redirected to /login; page content not visible |
| ROUTE-PERM-02 | Unauthorized role access | 1. Login as viewer 2. Navigate to admin page | Redirected to /403 or /unauthorized |
| ROUTE-PERM-03 | Authorized access | 1. Login as admin 2. Navigate to page | Page renders correctly with all admin elements |
| ROUTE-PERM-04 | Direct URL (deep link) | 1. Paste full page URL while logged out | Redirected to login; after login, returns to original URL |
| ROUTE-PERM-05 | Back after redirect | 1. Access protected page → redirected 2. Press back | Does NOT return to protected page |
| ROUTE-PERM-06 | Token expiry mid-session | 1. On page 2. Token expires 3. Click action | Redirected to login; no error screen |

### Category 2: UI Visibility Tests (UI-PERM-xx)

| Test ID | Scenario | Role | Expected |
|---------|----------|------|----------|
| UI-PERM-01 | Admin sees all actions | admin | Create, Edit, Delete, Export, Settings all visible |
| UI-PERM-02 | Manager sees limited actions | manager | Create, Edit, Export visible; Delete, Settings hidden |
| UI-PERM-03 | Viewer sees read-only | viewer | Only read/export visible; no create/edit/delete |
| UI-PERM-04 | No flash of hidden elements | viewer | Hidden elements never briefly appear during load |
| UI-PERM-05 | Role change reflects in UI | any | After admin changes user's role, UI updates on next load |

### Category 3: API Authorization Tests (API-PERM-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| API-PERM-01 | API call matching visible button | 1. Login as admin 2. Click delete | 200 OK — action succeeds |
| API-PERM-02 | Direct API call (bypass UI) | 1. Login as viewer 2. Call DELETE /users/1 via curl | 403 Forbidden |
| API-PERM-03 | Direct API without auth | 1. Call DELETE /users/1 without token | 401 Unauthorized |
| API-PERM-04 | Expired token API call | 1. Use expired token 2. Call API | 401 Unauthorized, not 500 |
| API-PERM-05 | Role escalation attempt | 1. Login as viewer 2. Modify JWT role claim 3. Call API | 403 — token signature invalid |

### Category 4: Data-Level Tests (DATA-PERM-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| DATA-PERM-01 | User sees own data only | 1. Login as regular user 2. GET /items | Only own items returned |
| DATA-PERM-02 | Admin sees all data | 1. Login as admin 2. GET /items | All items from all users returned |
| DATA-PERM-03 | Cross-user data access | 1. Login as user A 2. GET /items/:idOfUserB | 403 or 404 — cannot access other user's data |
| DATA-PERM-04 | Sensitive field masking | 1. Login as regular user 2. GET /users | SSN/password fields masked or absent |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Route Protection Coverage | 20% | Guard exists, correct redirect, no FOUC |
| UI Element Visibility | 15% | Role-based conditionals identified; no hardcoded strings |
| Permission Check Consistency | 15% | Single pattern used (hook/HOC/directive); no ad-hoc checks |
| Frontend-Backend Auth Match | 25% | Every frontend-hidden action also has backend guard |
| Data-Level Filtering | 10% | List APIs filter by user's scope/role |
| Permission Definition Sync | 10% | Same role/permission names across stack |
| Redirect Security | 5% | Uses replace, redirects to correct page, no open redirect |

**Scoring rules** (see `qa-shared-reference.md` for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Each action with matching frontend+backend guard: +2
  - 100% frontend-backend match: +4
Final Score = min(100, Score + Bonus)
```

**Status thresholds:**
- **PASS**: >= 80 points
- **NEEDS ATTENTION**: 60-79 points
- **FAIL**: < 60 points

---

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| File not found | Invalid path | Check file path and try again |
| Unsupported file type | Wrong extension | Supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js` |
| No permission checks found | Public page with no role logic | Warning shown; page may be intentionally public |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| Auth library not detected | Custom auth implementation | Analyzes generic `useAuth`/`user.role` patterns |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| Guard in separate file | Route guard defined in router config, not component | Follows import; notes as "externally defined guard" |
| CASL abilities defined dynamically | `ability.update(rules)` at runtime | Notes as "dynamic permissions — manual review needed" |
| Multiple auth strategies | OAuth + JWT + API key in same app | Analyzes each strategy separately |

---

## Edge Cases Handled

- **Super-admin vs regular admin** — Partial permissions within "admin" role; notes permission granularity
- **Multi-tenant** — User can access only their organization's data; checks for `orgId` filtering
- **Permission changes mid-session** — Admin changes user's role while user is logged in; checks for token refresh
- **Token expiry during form fill** — Submit fails due to expired token; checks for graceful handling
- **Frontend uses enum, backend uses string** — `Role.ADMIN` vs `'admin'`; checks serialization
- **Negative permission patterns** — `role !== 'viewer'` breaks when new roles are added; flagged
- **Inherited permissions** — Admin inherits all editor permissions; checks if backend implements inheritance
- **Feature flags vs permissions** — Feature disabled for all vs no permission for some; different UX expected
- **OAuth scopes vs app roles** — Different permission layers; notes which layer controls what
- **Public routes with optional auth** — Show "Edit" only if logged in as owner; partial auth pattern
- **Server Component auth (Next.js)** — `getServerSession()` in RSC; no client-side check visible
- **Middleware vs component auth** — Auth checked in middleware but also re-checked in component; redundant but safe
- **Role stored in different places** — JWT payload, session, database; which is source of truth?
- **Soft-delete and permission** — Can deleted user's token still work? Token blacklist check
- **ABAC (Attribute-Based Access Control)** — Owner-only actions checked at frontend and backend; attribute-based conditions beyond role checks
- **Supabase Row-Level Security (RLS)** — PostgreSQL RLS policies using `auth.uid()`; detected in migration files
- **Missing RLS on Supabase tables** — Tables without `ENABLE ROW LEVEL SECURITY` flagged as CRITICAL
- **Middleware chain ordering** — Auth middleware placed after route handler creates security gap; ordering verified
- **NextAuth v5 `auth()` helper** — Server Component auth via `auth()` function in RSC; no client-side check visible
- **Clerk `currentUser()` in RSC** — Server Component direct user access; `<SignedIn>`/`<SignedOut>` for conditional rendering
- **Impersonation** — Admin "login as user" functionality; checks if impersonation token has reduced permissions
- **Audit logging** — Whether permission-sensitive actions (delete, role change, export) are logged; flagged as INFO if missing

---

## Related Commands

- `/qa-input-fields` — QA input fields (form validation per role)
- `/qa-back-navigation` — QA back navigation (auth redirect uses replace)
- `/qa-loading-error-empty` — QA loading/error/empty states (403 error state)
- `/qa-modal-drawer` — QA modals (create/edit modals may be role-gated)
- `/qa-table-list` — QA tables (row actions may be role-gated, data filtered by role)
- `/review-command` — Validate this command's structure and quality

---

## Tips

1. **Backend guards are non-negotiable** — Frontend hiding is UX; backend guards are security. Every hidden action MUST have a backend guard
2. **Test with curl, not just the UI** — The browser hides buttons, but `curl` bypasses all frontend logic; always test API directly
3. **Negative role checks are fragile** — `role !== 'viewer'` breaks silently when you add a `'guest'` role; use positive inclusion
4. **Check token refresh on role change** — The most dangerous gap: admin demotes a user, but the user keeps admin access for hours
5. **Use `--depth full` for security audits** — Frontend-only analysis misses the most critical issues (backend guard gaps)
6. **Combine with `/qa-back-navigation`** — Auth redirects should use `replace` to prevent back-button returning to protected pages
7. **Check FOUC on SSR pages** — Server-side redirect (middleware) has no FOUC; client-side useEffect redirect shows content briefly
8. **Re-run after adding new actions** — Every new button, link, or API endpoint needs both frontend and backend permission checks
