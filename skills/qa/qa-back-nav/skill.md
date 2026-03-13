---
name: qa-back-nav
description: "Audit browser back navigation, history stack, mobile back key, redirect chains, and route guard behavior"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Back Navigation

Analyze navigation and back-button behavior in a frontend page or component, detect navigation anti-patterns that break back navigation, and generate a comprehensive QA report with test cases for browser back button and mobile system back key behavior.

---

## Purpose

This skill helps you:
1. **Detect navigation framework** — Identify React Router, Next.js, Vue Router, Angular Router, or Nuxt patterns
2. **Audit replace vs push usage** — Flag `router.replace()` calls that break back navigation
3. **Trace back button components** — Find back/return buttons and verify they use `router.back()` not hardcoded paths
4. **Analyze mobile back key handling** — Detect popstate, BackHandler, and Capacitor/Cordova listeners
5. **Detect redirect chains** — Identify sequential redirects that corrupt history stack
6. **Inspect auth guard redirects** — Verify navigation guards use replace (not push) appropriately
7. **Check after-action navigation** — Verify post-submit/delete redirects preserve correct history
8. **Generate test cases** — Produce actionable QA test checklists for manual and automated testing

**Important:** This skill is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.ts`, `.js`)
- (Optional) Router config file path for route tree analysis
- (Optional) Project `CLAUDE.md` with project-specific navigation conventions
- **Reference:** `qa-shared/reference.md` for framework detection tables and router pattern mappings

---

## Usage

```bash
/qa-back-nav <page-or-component-file-path>
```

**Examples:**
```bash
# Analyze a React page component
/qa-back-nav src/pages/UserDetail.tsx

# Include router config for full route tree analysis
/qa-back-nav src/pages/OrderConfirm.tsx --router src/router/index.ts

# Explicitly include mobile back key analysis
/qa-back-nav src/screens/HomeScreen.tsx --mobile

# Shallow analysis — only analyze the given file, no route tree traversal
/qa-back-nav pages/checkout.vue --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--router <path>` | No | Path to router config file. Enables route tree analysis and deep link inspection |
| `--mobile` | No | Explicitly include mobile back key analysis. Auto-detected for React Native and Capacitor projects |
| `--depth full\|shallow` | No | `full` (default): trace navigation across entire route tree. `shallow`: analyze only the given file |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Framework Detection              │
│  - Validate file path and extension                          │
│  - Auto-detect frontend/backend directories                  │
│  - Detect navigation framework and apply corresponding       │
│    patterns (React Router/Next/Vue/Angular/Nuxt/etc.)        │
│  - Detect project type (SPA / React Native / PWA / Hybrid)  │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Navigation Call Inventory                           │
│  - Extract all router.push / router.replace / navigate()    │
│  - Extract all Link components and their props               │
│  - Extract direct history / window.history calls             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Back Button Component Analysis                      │
│  - Find back/return buttons by text, icon, aria-label        │
│  - Trace onClick handlers — router.back() vs hardcoded path  │
│  - Detect conditional back behavior                          │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Redirect & Guard Analysis                           │
│  - Detect useEffect/mounted redirects (auto-redirect)        │
│  - Detect auth/role guards and their redirect method         │
│  - Detect after-action redirects (post-submit/delete)        │
│  - Detect redirect chains (sequential redirects)             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Mobile Back Key Analysis                            │
│  - Detect popstate event listeners                           │
│  - Detect React Native BackHandler usage                     │
│  - Detect Capacitor/Cordova back button listeners            │
│  - Detect modal/drawer back press handling                   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Route Tree & Deep Link Analysis (--depth full)      │
│  - Read router config for full route definitions             │
│  - Check history mode vs hash mode                           │
│  - Trace deep link entry points and history stack impact     │
│  - Identify orphan routes with no back path                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Generate QA Report                                  │
│  - Navigation inventory, anti-pattern matrix                 │
│  - Mobile back key coverage, route tree analysis             │
│  - Issues with severity, test cases, recommendations         │
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

### Step 1: Input Validation & Framework Detection

Read the file path from `$ARGUMENTS`. Parse optional flags (`--router`, `--mobile`, `--depth`).

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-back-nav <page-or-component-file-path> [--router <router-config-path>] [--mobile] [--depth full|shallow]
```

**1.2 Project Structure Detection:**

Auto-detect frontend and backend directories by scanning for `package.json` files, framework config files (`next.config.*`, `nuxt.config.*`, `vite.config.*`, `angular.json`, etc.), and common directory conventions (`src/`, `app/`, `pages/`, `components/`). Do not assume any hardcoded directory paths.

**1.3 Navigation Framework Detection:**

Read the target file and project's `package.json`. Detect framework and apply corresponding patterns:

| Signal | Framework |
|--------|-----------|
| `import { useNavigate } from 'react-router-dom'` or `useHistory` | React Router v5/v6 |
| `import { useRouter } from 'next/router'` | Next.js Pages Router |
| `import { useRouter } from 'next/navigation'` | Next.js App Router |
| `import { useRouter } from 'vue-router'` | Vue Router |
| `import { Router, ActivatedRoute } from '@angular/router'` | Angular Router |
| `navigateTo()` or `import { useRouter } from '#app'` | Nuxt 3 |
| `import { useRouter } from '@tanstack/react-router'` | TanStack Router |
| `import { goto } from '$app/navigation'` | SvelteKit |
| `import { useRouter } from 'next/navigation'` + `app/` dir with `@parallel` or `(.)intercepting` | Next.js App Router (parallel/intercepting routes) |

**1.4 Project Type Detection:**

| Signal | Project Type |
|--------|-------------|
| `react-native` in package.json | React Native (mobile) |
| `@capacitor/core` in package.json | Capacitor (Hybrid PWA) |
| `cordova` in package.json | Cordova (Hybrid) |
| Next.js `app/` directory present | Next.js App Router SPA |
| Next.js `pages/` directory present | Next.js Pages Router SPA |
| `createWebHashHistory()` or `HashRouter` in router config | SPA (hash mode — FLAG) |
| Other SPA frameworks | SPA (history mode) |

**1.5 Auto-detect `--mobile`:**

If `react-native` or `@capacitor/core` or `cordova` is detected in package.json, automatically enable mobile back key analysis (equivalent to `--mobile` flag being set).

**Step 1 Checklist:**
```
[ ] File path validated and file exists
[ ] File extension is supported
[ ] Frontend/backend directories auto-detected
[ ] Navigation framework detected
[ ] Project type detected (SPA / React Native / PWA / Hybrid)
[ ] --mobile auto-detected from package.json (if applicable)
```

---

### Step 2: Navigation Call Inventory

Parse the component/page source and extract every navigation call site.

**2.1 Framework-Specific Navigation Call Patterns:**

Detect framework and apply corresponding patterns:

**React Router v6:**
- `navigate('/path')` — push (normal)
- `navigate('/path', { replace: true })` — replace (FLAG)
- `navigate(-1)` — back (OK)
- `<Link to="/path">` — push (normal)
- `<Link to="/path" replace>` — replace (FLAG)
- `<Navigate to="/path" replace />` — replace redirect in JSX (FLAG)

**React Router v5:**
- `history.push('/path')` — push (normal)
- `history.replace('/path')` — replace (FLAG)
- `history.goBack()` — back (OK)
- `<Redirect to="/path" />` — replace by default (FLAG if unexpected)

**Next.js (Pages Router):**
- `router.push('/path')` — push (normal)
- `router.replace('/path')` — replace (FLAG)
- `router.back()` — back (OK)

**Next.js (App Router):**
- `router.push('/path')` — push (normal)
- `router.replace('/path')` — replace (FLAG)
- `router.back()` — back (OK)
- `redirect('/path')` in server component — replace, no history entry (FLAG)
- `<Link href="/path">` — push (normal)
- `<Link href="/path" replace>` — replace (FLAG)

**Vue Router:**
- `router.push('/path')` — push (normal)
- `router.push({ path, replace: true })` — replace (FLAG)
- `router.replace('/path')` — replace (FLAG)
- `router.back()` / `router.go(-1)` — back (OK)
- `<router-link :to="path">` — push (normal)
- `<router-link :to="path" replace>` — replace (FLAG)

**Angular Router:**
- `this.router.navigate(['/path'])` — push (normal)
- `this.router.navigate(['/path'], { replaceUrl: true })` — replace (FLAG)
- `this.router.navigateByUrl('/path', { replaceUrl: true })` — replace (FLAG)
- `this.location.back()` — back (OK)

**Nuxt 3:**
- `navigateTo('/path')` — push (normal)
- `navigateTo('/path', { replace: true })` — replace (FLAG)
- `useRouter().back()` — back (OK)

**TanStack Router:**
- `navigate({ to: '/path' })` — push (normal)
- `navigate({ to: '/path', replace: true })` — replace (FLAG)
- `router.history.back()` — back (OK)
- `<Link to="/path">` — push (normal)
- `<Link to="/path" replace>` — replace (FLAG)

**SvelteKit:**
- `goto('/path')` — push (normal)
- `goto('/path', { replaceState: true })` — replace (FLAG)
- `history.back()` — back (OK)
- `<a href="/path">` — push (normal, enhanced by SvelteKit)
- `redirect(303, '/path')` in `+page.server.ts` load/actions — server redirect (FLAG)

**React Router v7 / Remix:**
- `throw redirect('/path')` in loader/action — replace (no history entry)
- `useNavigate()` — same as React Router v6 patterns
- `Form` + `action` — form submissions may trigger redirect in action response

**Direct DOM / History API:**
- `window.history.pushState(...)` — push (review intent)
- `window.history.replaceState(...)` — replace (FLAG)
- `window.location.href = '...'` — full page reload, breaks SPA history (FLAG)
- `window.location.replace('...')` — replace (FLAG)

**2.2 Build Navigation Call Registry:**

For each call found, record:

| Property | Description |
|----------|-------------|
| `callType` | `push` / `replace` / `back` / `link` / `redirect` |
| `targetPath` | Destination path or `-1` for back |
| `method` | e.g., `router.replace`, `navigate`, `<Link replace>` |
| `flagged` | `true` if `replace` or full-page reload |
| `context` | Surrounding code context (useEffect, onClick, render, etc.) |
| `sourceLocation` | `file:line` reference |

**Step 2 Checklist:**
```
[ ] All router.push / navigate calls identified
[ ] All router.replace / navigate({replace:true}) calls identified and flagged
[ ] All Link/router-link components found and replace prop checked
[ ] All back() / goBack() calls identified
[ ] Direct window.history and window.location calls found
[ ] Navigation call registry built with callType, path, flag, context, location
```

---

### Step 3: Back Button Component Analysis

**3.1 Back Button Discovery:**

Search for back/return button components using multiple detection strategies:

| Detection Strategy | Pattern Examples |
|-------------------|-----------------|
| Text content | Button text: "Back", "뒤로", "뒤로가기", "이전", "Return", "Go Back", "戻る" |
| Icon component | `<ArrowLeft>`, `<ChevronLeft>`, `<BackIcon>`, `<IoArrowBack>`, `<MdArrowBack>` |
| Icon class | `fa-arrow-left`, `icon-back`, `bi-arrow-left` |
| `aria-label` | `aria-label="back"`, `aria-label="go back"`, `aria-label="return"` |
| Component name | `BackButton`, `ReturnButton`, `GoBackBtn`, `HeaderBack`, `BackNavigation` |
| Router link with back | `<Link to={-1}>`, `<router-link to="..">` |

**3.2 onClick Handler Tracing:**

For each back button found, trace its `onClick` / event handler:

| Handler Type | Example | Assessment |
|-------------|---------|------------|
| CORRECT — router back | `router.back()`, `navigate(-1)`, `history.goBack()`, `location.back()` | OK |
| CORRECT — contextual | `onClose()` prop from parent (modal/drawer close) | OK — context-dependent |
| WARNING — hardcoded path | `router.push('/users')`, `navigate('/dashboard')` | WARNING — breaks when accessed from different entry points |
| WARNING — conditional | `isModal ? onClose() : router.back()` | Review — may be intentional |
| CRITICAL — window.location | `window.location.href = '/home'` | CRITICAL — destroys entire history stack |

**3.3 Conditional Back Behavior:**

Detect back buttons that conditionally behave differently:
- Form dirty check before going back (e.g., "Unsaved changes" confirmation)
- Role-based back destination
- State-dependent navigation

Flag these as requiring manual testing.

**Step 3 Checklist:**
```
[ ] All back/return buttons discovered (text, icon, aria-label, name-based)
[ ] onClick handlers traced for every back button found
[ ] Handler classified: router.back() / hardcoded path / window.location / conditional
[ ] Hardcoded path navigations flagged as WARNING
[ ] window.location usage flagged as CRITICAL
[ ] Conditional back behaviors noted for manual testing
```

---

### Step 4: Redirect & Guard Analysis

**4.1 Lifecycle Hook Redirects (Auto-Redirect on Load):**

Detect navigation calls inside lifecycle hooks. The hook name varies by framework:

| Framework | Lifecycle Hook | Replace Example (OK) | Push Example (WARNING) |
|-----------|---------------|---------------------|----------------------|
| React | `useEffect(() => { ... }, [])` | `navigate('/login', { replace: true })` | `navigate('/login')` |
| Next.js | `useEffect(() => { ... }, [])` | `router.replace('/login')` | `router.push('/login')` |
| Vue | `onMounted(() => { ... })` | `router.replace('/login')` | `router.push('/login')` |
| Nuxt | `onMounted()` or route middleware | `navigateTo('/login', { replace: true })` | `navigateTo('/login')` |
| Angular | `ngOnInit()` | `this.router.navigate(['/login'], { replaceUrl: true })` | `this.router.navigate(['/login'])` |
| SvelteKit | `+page.server.ts` load function | `redirect(303, '/login')` (server-side, always replace) | N/A |
| Generic | Any initialization code | Look for redirect calls in init/mount handlers | Same |

**Key rule:** Auth guards that redirect unauthenticated users SHOULD use `replace` so the login page does not appear in back history. Flag uses of `push` in auth guard redirects.

**4.2 Router Guard / Middleware Analysis:**

If `--router` path is provided or router config is auto-located:

| Guard Type | Pattern | Check |
|-----------|---------|-------|
| React Router loader | `loader` function with `redirect()` | redirect() = replace (OK for auth) |
| Vue Router beforeEach | `router.beforeEach((to, from, next) => ...)` | `next('/path')` = push (WARNING), `next({ path, replace: true })` = OK |
| Angular CanActivate | `canActivate()` returning `router.parseUrl('/login')` | Is `replaceUrl` used? |
| Next.js middleware | `middleware.ts` with `NextResponse.redirect()` | Redirect = server redirect (OK) |
| Nuxt middleware | `defineNuxtRouteMiddleware(...)` | Check for `replace: true` in `navigateTo` |
| TanStack Router beforeLoad | `beforeLoad` function in route definition with `redirect()` | redirect() = replace (OK for auth) |
| SvelteKit load redirect | `redirect(303, '/login')` in `+page.server.ts` or `+layout.server.ts` | Server redirect (OK) |
| React Router v7 loader | `loader` with `throw redirect()` | replace (OK for auth) |

**4.3 After-Action Redirects:**

Find navigation calls in form submit handlers, delete handlers, and async action callbacks:

| Scenario | Expected Pattern | Assessment |
|----------|-----------------|------------|
| After create (POST success) | `router.push('/items')` | OK — should push to new page |
| After edit/update (PATCH success) | `router.push('/items/[id]')` or `router.back()` | OK |
| After delete | `router.replace('/items')` | OK — replace prevents back to deleted resource |
| After delete | `router.push('/items')` | WARNING — back leads to deleted resource (404) |
| After logout | `router.replace('/login')` | Required — must use replace |
| After logout | `router.push('/login')` | CRITICAL — back returns to authenticated state |
| After form cancel | `router.back()` | OK |
| After form cancel | `router.push('/list')` | WARNING — user loses back history |

**4.4 Redirect Chain Detection:**

Look for patterns where one redirect leads to another. Flag if redirect depth > 2 hops:
- Navigation call inside the handler of another navigation event
- Multiple sequential `navigate()` calls in the same useEffect
- Route A redirects to B, which redirects to C

**Step 4 Checklist:**
```
[ ] useEffect/mounted auto-redirects identified
[ ] Auth guard redirects checked: replace used (OK) vs push used (WARNING)
[ ] Router config guards analyzed (if --router provided or auto-located)
[ ] After-action redirects found and categorized (post-submit, post-delete, logout)
[ ] Redirect chains detected (sequential navigations > 2 hops)
[ ] After-delete navigation checked: does back lead to deleted resource?
```

---

### Step 5: Mobile Back Key Analysis

This step runs if `--mobile` is explicitly set, or auto-detected (React Native / Capacitor / Cordova project).

For web-only SPA projects without `--mobile`, only check popstate handling for modals.

**5.1 Browser/Web: popstate Event Listener:**

| Pattern | Assessment |
|---------|-----------|
| `window.addEventListener('popstate', handler)` present | OK — handles browser back |
| `popstate` listener absent in SPA with modals/drawers | WARNING — modal may not close on back |
| Handler calls `history.pushState(null, '', location.href)` on modal open | OK — proper modal back-close pattern |

**5.2 React Native: BackHandler:**

| Pattern | Assessment |
|---------|-----------|
| `BackHandler.addEventListener('hardwareBackPress', handler)` present | OK |
| Handler returns `true` (event consumed) | OK — custom back behavior |
| Handler returns `false` (falls through to default) | OK — uses default behavior |
| No BackHandler in screen with modal/drawer | WARNING — back key closes app instead of modal |
| Cleanup in useEffect return / componentWillUnmount | OK — proper cleanup |
| Missing cleanup (no `removeEventListener` / `subscription.remove()`) | WARNING — memory leak, double-fire risk |

**5.3 Capacitor: App.addListener('backButton'):**

| Pattern | Assessment |
|---------|-----------|
| `App.addListener('backButton', handler)` present | OK |
| Handler calls `App.exitApp()` on root screen | OK — intentional exit |
| Handler calls `router.back()` on non-root screens | OK — navigates back |
| No Capacitor back listener in a Capacitor project | WARNING — Android back may exit app |

**5.4 Cordova / Ionic:**

| Pattern | Assessment |
|---------|-----------|
| `document.addEventListener('backbutton', handler, false)` | OK |
| IonicVue / IonicReact with `<IonBackButton>` component | OK — handled by Ionic framework |
| No back button config in Cordova project | WARNING |

**5.5 Modal and Drawer Back Handling:**

Check if modals, drawers, dialogs, and bottom sheets handle back key correctly:

| Modal Pattern | Back Key Behavior Check |
|--------------|------------------------|
| Modal opens + `history.pushState(null, '', location.href)` + `popstate` listener closes it | OK — back closes modal |
| Modal opens without history entry | WARNING — back navigates to previous route instead of closing |
| React Native Modal with BackHandler returning `true` + closing modal | OK |
| React Native Modal without BackHandler | WARNING — back navigates away |

**5.6 iOS Swipe-Back Gesture:**

For React Native and Capacitor projects:

| Pattern | Assessment |
|---------|-----------|
| `gestureEnabled: false` on a navigation screen | WARNING — user expects swipe-back |
| `swipeBackEnabled: false` in Capacitor config | WARNING — blocks iOS swipe gesture |
| `gestureEnabled: false` on a modal screen | OK — modals typically don't have swipe-back |

**Step 5 Checklist:**
```
[ ] popstate event listener checked for web/SPA (browser back intercept)
[ ] React Native BackHandler analyzed (if React Native project)
[ ] BackHandler cleanup (removeEventListener) verified
[ ] Capacitor App.addListener('backButton') checked (if Capacitor project)
[ ] Cordova backbutton event listener checked (if Cordova project)
[ ] Modal/Drawer back key handling verified (history push pattern)
[ ] iOS gesture navigation (swipeBack) settings checked (if mobile project)
```

---

### Step 6: Route Tree & Deep Link Analysis (`--depth full`)

This step only runs if `--depth full` (the default) AND a `--router` path is provided or router config can be auto-located.

**6.1 Router Config Auto-Location:**

If `--router` is not provided, auto-detect frontend directories and look for:

| Pattern | Location |
|---------|----------|
| React Router | `src/App.tsx` with `<Routes>`, `src/router.tsx`, `src/routes/index.tsx` |
| Vue Router | `src/router/index.ts`, `src/router.ts` |
| Angular | `app-routing.module.ts`, `app.routes.ts` |
| Next.js App Router | `app/` directory structure |
| Next.js Pages Router | `pages/` directory structure |
| Nuxt | `pages/` directory structure, `nuxt.config.ts` |

**6.2 History Mode vs Hash Mode:**

| Finding | Assessment |
|---------|-----------|
| `createWebHistory()` (Vue) | OK |
| `createWebHashHistory()` (Vue) | WARNING — hash URLs break some back patterns |
| `<BrowserRouter>` (React) | OK |
| `<HashRouter>` (React) | WARNING — hash mode |
| Next.js — always history mode | OK |

**6.3 Deep Link Entry Points:**

For each route accessible via direct URL (external link, push notification, bookmark):
- Does back button on this page lead somewhere meaningful?
- If the page has no parent in the history stack (first page load via deep link), does the back button handle this gracefully?
- Does the route have a `redirectTo` that skips the intended history entry?

**6.4 Orphan Route Detection:**

Identify routes that:
- Have no `<Link>` or `<router-link>` pointing to them from any parent view
- Are only reachable via programmatic `router.push` from a single location
- Have no logical "back" destination defined

**Step 6 Checklist:**
```
[ ] Router config file located and read
[ ] Router mode confirmed (history vs hash)
[ ] All routes enumerated from config
[ ] Deep link entry routes identified
[ ] Deep link back behavior assessed (meaningful back destination exists)
[ ] Orphan routes with no back path identified
[ ] Nested route structures analyzed for back behavior
```

---

### Step 7: Generate QA Report

Compile all analysis results into a structured report.

**Report Sections:**

1. **Header** — Component name, file path, framework, project type, date, overall score, status
2. **Navigation Call Inventory** — Table of all navigation calls with type, path, flagged status, context, location
3. **Replace vs Push Analysis** — Matrix showing replace calls with intent assessment (auth guard OK vs. post-submit WARNING)
4. **Back Button Analysis** — Each back button, its handler, classification (OK / WARNING / CRITICAL)
5. **Redirect & Guard Analysis** — Auth guard redirects, after-action redirects, redirect chains
6. **Mobile Back Key Coverage** — Coverage matrix for each platform's back key mechanism
7. **Route Tree Summary** (if `--depth full`) — History mode, orphan routes, deep link assessment
8. **Issues Found** — All issues with severity (CRITICAL / WARNING / INFO) and fix guidance
9. **Test Cases** — Complete test case table
10. **Recommendations** — Prioritized action items

**Step 7 Checklist:**
```
[ ] Navigation Call Inventory table complete
[ ] Replace vs Push Analysis matrix complete with intent assessment
[ ] Back Button Analysis complete for all discovered buttons
[ ] Redirect & Guard Analysis complete
[ ] Mobile Back Key Coverage complete (for applicable platforms)
[ ] Route Tree Summary complete (if --depth full)
[ ] Issues listed with severity levels
[ ] Test cases listed with expected results
[ ] Recommendations prioritized
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_BackNav_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_BackNav_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Back Navigation Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_BackNav_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_BackNav_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React Router v6 / Next.js App Router / Vue Router / etc.]
- Navigation calls found: [N] total ([N] push, [N] replace (flagged), [N] back)
- Back buttons found: [N] ([N] correct, [N] hardcoded path, [N] window.location)
- Mobile back key: [Covered / Not Covered / Not Applicable]
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
║                   QA Back Navigation Report                       ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                   ║
║  File:          [file-path]                                       ║
║  Framework:     [React Router v6 / Next.js App / Vue Router / …]  ║
║  Project Type:  [SPA / React Native / Capacitor PWA / Hybrid]     ║
║  Report Date:   [YYYY-MM-DD HH:MM]                               ║
║  Depth:         [full / shallow]                                  ║
║  Mobile:        [Yes (auto-detected) / Yes (--mobile) / No]      ║
║  Overall Score: [N]/100                                           ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                   ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  NAVIGATION CALL INVENTORY ([N] total)                            ║
║                                                                    ║
║  | # | Method            | Path          | Type    | Flagged |    ║
║  |---|-------------------|---------------|---------|---------|    ║
║  | 1 | navigate()        | /dashboard    | push    | No      |    ║
║  | 2 | router.replace()  | /login        | replace | Yes     |    ║
║  | 3 | navigate(-1)      | (back)        | back    | No      |    ║
║  | 4 | <Link>            | /settings     | push    | No      |    ║
║  | 5 | window.location   | /home         | direct  | Yes     |    ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  REPLACE vs PUSH ANALYSIS                                         ║
║                                                                    ║
║  | Location          | Call                    | Intent   | OK?  | ║
║  |-------------------|-------------------------|----------|------|  ║
║  | useEffect:45      | navigate('/login',repl.)| auth     | OK   | ║
║  | handleSubmit:102  | router.replace('/home') | post-    | WARN | ║
║  |                   |                         | submit   |      | ║
║  | handleDelete:150  | router.replace('/list') | post-    | OK   | ║
║  |                   |                         | delete   |      | ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  BACK BUTTON ANALYSIS ([N] buttons)                               ║
║                                                                    ║
║  | # | Element          | Handler          | Type       | Status |║
║  |---|------------------|------------------|------------|--------|║
║  | 1 | <Button>Back</>  | navigate(-1)     | router.back| OK     |║
║  | 2 | <ArrowLeft />    | router.push('/') | hardcoded  | WARN   |║
║  | 3 | <BackBtn />      | window.location  | direct     | CRIT   |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  REDIRECT & GUARD ANALYSIS                                        ║
║                                                                    ║
║  [AUTH GUARD] useEffect:45                                        ║
║    Condition: !isAuthenticated                                     ║
║    Action: navigate('/login', { replace: true })                  ║
║    Status: OK — uses replace, login won't appear in back history  ║
║                                                                    ║
║  [POST-DELETE] handleDelete:150                                   ║
║    Action: router.replace('/items')                               ║
║    Status: OK — prevents back to deleted resource                 ║
║                                                                    ║
║  [POST-LOGOUT] handleLogout:88                                    ║
║    Action: router.push('/login')                                  ║
║    Status: CRITICAL — must use replace, back returns to auth state║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  MOBILE BACK KEY COVERAGE                                         ║
║                                                                    ║
║  | Mechanism              | Present | Cleanup | Status |          ║
║  |------------------------|---------|---------|--------|          ║
║  | popstate listener      | Yes     | N/A     | OK     |          ║
║  | BackHandler (RN)       | No      | N/A     | N/A    |          ║
║  | Capacitor backButton   | No      | N/A     | WARN   |          ║
║  | Modal back intercept   | Yes     | Yes     | OK     |          ║
║  | iOS swipe gesture      | Enabled | N/A     | OK     |          ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ISSUES FOUND                                                     ║
║                                                                    ║
║  [CRITICAL] BACK-01: Back button on OrderDetail uses              ║
║    router.push('/orders') instead of router.back().               ║
║    Risk: If user navigated from Notification, pressing back       ║
║    returns to /orders, not Notification page.                     ║
║    Fix: Replace with navigate(-1) or router.back().               ║
║                                                                    ║
║  [CRITICAL] BACK-02: handleLogout() uses router.push('/login').   ║
║    Risk: Back button returns to authenticated state after logout. ║
║    Fix: Use router.replace('/login').                             ║
║                                                                    ║
║  [WARNING] BACK-03: handleSubmit() uses router.replace('/home')   ║
║    after form submission.                                         ║
║    Risk: User cannot go back to form page from /home.             ║
║    Fix: Use router.push('/home') if back-to-form is desirable.   ║
║                                                                    ║
║  [WARNING] BACK-04: No Capacitor back button listener detected.   ║
║    Project uses Capacitor but App.addListener('backButton')       ║
║    not found. Android hardware back key may close the app.        ║
║    Fix: Add Capacitor back button handler in app root.            ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  TEST CASES ([N] total)                                           ║
║    Back button: [N] | Mobile back: [N] | Replace/Push: [N]       ║
║    Auth guard: [N]                                                ║
║    [See full list in test cases file]                             ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  RECOMMENDATIONS                                                  ║
║  1. [Critical] Replace router.push('/orders') with navigate(-1)  ║
║  2. [Critical] Change logout to router.replace('/login')          ║
║  3. [Warning] Review post-submit replace — switch to push if     ║
║     back-to-form is desired                                       ║
║  4. [Warning] Add Capacitor back button listener in app root      ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Test Case Categories

### Category 1: Back Button Tests (BACK-BTN-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| BACK-BTN-01 | Navigate to page from list, click back button | 1. Go to /items 2. Click item 3. Click back | Returns to /items with same scroll position |
| BACK-BTN-02 | Navigate to page from search, click back button | 1. Search → click result 2. Click back | Returns to search results, not /items |
| BACK-BTN-03 | Navigate to page from notification, click back | 1. Click notification → land on detail 2. Click back | Returns to notifications, not /items |
| BACK-BTN-04 | Direct URL access (no history), click back | 1. Open /items/123 directly 2. Click back | Navigates to logical parent (/items) or shows fallback |
| BACK-BTN-05 | Browser back after clicking in-page back button | 1. Click back button 2. Press browser back | Normal browser back behavior |

### Category 2: Replace/Push Tests (NAV-REPLACE-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| NAV-REPLACE-01 | Auth redirect uses replace | 1. Navigate to protected route while logged out | Redirected to /login; browser back does NOT return to protected route |
| NAV-REPLACE-02 | After form submit uses push | 1. Fill form 2. Submit | Navigated to success page; browser back returns to form |
| NAV-REPLACE-03 | After delete uses replace | 1. Delete an item | Navigated to list; browser back does NOT return to deleted item |
| NAV-REPLACE-04 | Logout uses replace | 1. Click logout | Redirected to /login; browser back does NOT return to authenticated page |

### Category 3: Mobile Back Key Tests (MOB-BACK-xx)

| Test ID | Scenario | Platform | Expected |
|---------|----------|----------|----------|
| MOB-BACK-01 | System back on main screen | Android | Exits app or shows exit confirmation |
| MOB-BACK-02 | System back on detail screen | Android | Returns to list screen |
| MOB-BACK-03 | System back when modal is open | Android | Closes modal, does not navigate |
| MOB-BACK-04 | Swipe-back gesture on detail screen | iOS | Returns to list with animation |
| MOB-BACK-05 | System back when drawer is open | Android | Closes drawer, does not navigate |
| MOB-BACK-06 | System back when bottom sheet is open | Android | Closes bottom sheet, does not navigate |

### Category 4: Auth Guard Tests (AUTH-GUARD-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| AUTH-GUARD-01 | Access protected → login → back | 1. Navigate to /dashboard 2. Redirected to /login 3. Login 4. Press back | Does NOT return to /login; stays on /dashboard |
| AUTH-GUARD-02 | Direct URL after logout → back | 1. Logout 2. Paste /dashboard URL 3. Redirected to /login 4. Press back | Does NOT show /dashboard |
| AUTH-GUARD-03 | Token expired mid-session → back | 1. On /dashboard 2. Token expires 3. Redirected to /login 4. Re-login 5. Press back | Returns to /dashboard, not /login |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Navigation Call Classification | 15% | All push/replace/back calls found and correctly classified |
| Replace Usage Correctness | 20% | Replace used only in appropriate contexts (auth guards, post-delete, logout) |
| Back Button Handler Quality | 20% | Back buttons use router.back() / navigate(-1), not hardcoded paths |
| Redirect & Guard Correctness | 15% | Auth guards use replace; after-action redirects are appropriate |
| Mobile Back Key Coverage | 15% | popstate / BackHandler / Capacitor listener present and correct |
| Route Tree & Deep Link Safety | 10% | History mode correct, no orphan routes, deep links have meaningful back |
| Test Coverage Completeness | 5% | Test cases cover all detected scenarios |

**Scoring rules** (see `qa-shared-reference` skill for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Each back button using router.back() correctly: +2
  - Mobile back key fully covered: +5
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
| File not found | Invalid path provided | Check file path and try again |
| Unsupported file type | Wrong extension | Supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.ts`, `.js` |
| Framework not detected | Unusual import structure or no package.json | Falls back to generic pattern scanning |
| No navigation calls found | Static/display-only component | Warning shown; component may not perform navigation |
| `--router` path not found | Invalid router config path | Skips Step 6; runs without route tree analysis |
| Router config unparseable | Unusual config format | Notes as "manual review needed"; skips route tree steps |
| Back button handler unresolvable | Handler is a prop from parent | Flags as "externally controlled — trace to parent component" |
| Circular redirect detected | Route A → B → A | Flags as CRITICAL with full trace chain |
| package.json not found | No package manager | Falls back to import-based framework detection only |

---

## Modern Pattern Detection

The following modern patterns are detected in addition to the standard patterns above:

### View Transitions API
- `document.startViewTransition()` — Detects usage that affects visual navigation transitions
- `viewTransition` prop on `<Link>` (React Router v7) or `transition:` option in `router.push` (experimental)
- Assessment: INFO — note for testers that visual transitions may affect perceived navigation timing

### Next.js App Router Parallel & Intercepting Routes
- `@folder` conventions in `app/` directory — parallel routes that render simultaneously
- `(.)folder`, `(..)folder`, `(...)folder` — intercepting routes that show modal overlays on navigation
- Back behavior: intercepting routes create shadow history entries that affect back button
- Assessment: WARNING if intercepting routes used without clear back navigation testing

### Scroll Restoration
- React Router: `<ScrollRestoration>` component presence
- Vue Router: `scrollBehavior` option in router config
- Next.js: `scroll` prop on `<Link>` or `router.push`
- Manual: `sessionStorage` scroll position save/restore pattern
- Assessment: WARNING if no scroll restoration mechanism detected for pages with scrollable content

### Back/Forward Cache (bfcache)
- `pageshow` event listener with `event.persisted` check — handles bfcache restoration
- `pagehide` / `beforeunload` listeners that may prevent bfcache
- Assessment: INFO — note that bfcache may serve stale data on back navigation

### Navigation Blocking (Form Protection)
- React Router v6+: `useBlocker()` / `unstable_useBlocker()` hook
- React Router v5: `<Prompt>` component
- Vue Router: `onBeforeRouteLeave()` with `next(false)` to block
- `beforeunload` event listener for full page reloads
- Assessment: Check if forms with unsaved data use navigation blocking

---

## Edge Cases Handled

- **Back button as prop** — When `onBack` / `onClose` is a prop from parent, flags as "externally controlled — trace to parent component"
- **Conditional rendering of back button** — Back button shown only in certain states; notes the condition
- **Dynamic routes** — `/items/[id]` patterns; verifies `router.back()` is used rather than hardcoded `/items`
- **Multi-page wizard with custom back** — Step wizard managing its own step index; treats internal step-back as distinct from route-back
- **Modal stack on mobile** — Multiple nested modals where each needs to close on back; checks for proper history stack management
- **Nested navigators (React Navigation)** — Stack navigators inside tab navigators; notes navigation scope
- **Next.js server-side redirect()** — In server components and Route Handlers; flags as replace-equivalent (no history entry)
- **Hash router detection** — Flags hash-mode routers as WARNING for compatibility with mobile back buttons
- **iframe navigation** — Navigation inside iframe noted as out-of-scope
- **Soft navigation (Next.js App Router)** — Distinguishes `router.push` (soft, keeps scroll) from full page navigation
- **React Navigation nested stacks** — Detects `navigation.goBack()` vs `navigation.navigate()` within nested stack navigators
- **Query parameter navigation** — `router.push({ query: ... })` without path change; notes this does not affect back behavior
- **Browser forward button** — Notes that `router.replace()` also affects forward button behavior
- **View Transitions API** — `document.startViewTransition()` affects perceived navigation timing; noted for testers
- **TanStack Router patterns** — `navigate()` and `router.history` patterns detected with same push/replace/back classification
- **SvelteKit navigation** — `goto()`, `redirect()` in load functions, and enhanced `<a>` links detected
- **React Router v7/Remix convergence** — `throw redirect()` in loaders/actions treated as replace (no history entry)
- **Next.js parallel routes** — `@folder` routes render simultaneously and affect history stack
- **Next.js intercepting routes** — `(.)folder` routes create shadow history entries visible on back press
- **bfcache restoration** — Browser may restore page from Back/Forward Cache with stale data; `pageshow` check detection
- **useBlocker / navigation blocking** — Forms with unsaved data blocking back navigation via `useBlocker` or `<Prompt>`
- **Scroll restoration** — `<ScrollRestoration>` component or `scrollBehavior` config preserving scroll on back navigation

---

## Related Skills

- `qa-input-fields` — QA all input fields in the same page/component
- `qa-loading-error-empty` — QA loading/error/empty states
- `qa-modal-drawer` — QA modal/drawer behavior (modals affect back navigation)
- `qa-table-list` — QA table/list data display (back preserves table state)
- `qa-permission-role` — QA permission/role access (auth redirect uses replace)
- `review-command` — Validate this skill's structure and quality

---

## Tips

1. **Start with pages that have back buttons** — Detail pages, modals, and form pages are the highest-priority targets
2. **Always check after-delete navigation** — The most common back navigation bug is pressing back after deletion and landing on a 404/blank page
3. **Use `--router` for full analysis** — Route tree analysis catches orphan routes and deep link issues invisible from a single file
4. **Mobile projects: always use `--mobile`** — Even web-only projects deployed as Capacitor/PWA apps need hardware back button handling
5. **Auth guard redirects should always use replace** — `router.replace('/login')` not `router.push('/login')` in guards — this is the #1 rule
6. **Combine with `qa-input-fields`** — After-submit navigation should be reviewed alongside input validation for complete UX coverage
7. **Re-run after adding new screens** — Every new page with a back button or auth guard should be analyzed
8. **Test deep links manually** — Open the page URL directly in a new tab and verify back button behavior with no history stack
