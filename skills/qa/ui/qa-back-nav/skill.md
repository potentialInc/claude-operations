---
name: qa-back-nav
description: "Audit browser back navigation, history stack, mobile back key, redirect chains, and route guard behavior"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Back Navigation — History & Back Button Audit

Detect navigation anti-patterns that break browser back button, mobile system back key, and history stack integrity.

## Execution Mode

- **Standalone** (`/qa-back-nav [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Replace vs push classification | **High** | All `router.replace()` calls reviewed — replace should only be used for auth guards, post-delete, logout. Flagged if used in normal navigation |
| 2 | Back button handler quality | **Critical** | Back/return buttons must use `router.back()` / `navigate(-1)`, not hardcoded paths. `window.location.href` in back button is Critical |
| 3 | Auth guard redirect method | **High** | Auth guards that redirect unauthenticated users must use `replace` (not `push`) so login page does not appear in back history |
| 4 | After-action redirect correctness | **High** | Post-delete must use `replace` (back to deleted resource = 404). Post-logout must use `replace`. Post-create can use `push` |
| 5 | Redirect chain detection | **Medium** | Sequential redirects > 2 hops corrupt the history stack. Flag redirect-inside-redirect patterns |
| 6 | Mobile back key coverage | **High** | React Native `BackHandler`, Capacitor `App.addListener('backButton')`, Cordova `backbutton` event, or web `popstate` listener present for modals/drawers |
| 7 | Modal back-close pattern | **Medium** | Modals/drawers should push a history entry on open and close on `popstate`, so back key closes modal instead of navigating away |
| 8 | History mode vs hash mode | **Medium** | `createWebHashHistory()` (Vue) or `<HashRouter>` (React) breaks some back patterns. Flag hash mode usage |
| 9 | Deep link back behavior | **Medium** | Pages accessed via deep link (no prior history) must have a meaningful back destination, not an empty history stack |
| 10 | iOS swipe-back gesture | **Low** | `gestureEnabled: false` or `swipeBackEnabled: false` on non-modal screens blocks expected swipe-back behavior |

## Skill-Specific Patterns

### Navigation Framework Detection

| Signal | Framework |
|--------|-----------|
| `import { useNavigate } from 'react-router-dom'` or `useHistory` | React Router v5/v6 |
| `import { useRouter } from 'next/router'` | Next.js Pages Router |
| `import { useRouter } from 'next/navigation'` | Next.js App Router |
| `import { useRouter } from 'vue-router'` | Vue Router |
| `import { Router, ActivatedRoute } from '@angular/router'` | Angular Router |
| `navigateTo()` or `import { useRouter } from '#app'` | Nuxt 3 |
| `import { goto } from '$app/navigation'` | SvelteKit |
| `import { useRouter } from '@tanstack/react-router'` | TanStack Router |

### After-Action Redirect Rules

| Scenario | Expected Method | Wrong Method → Issue |
|----------|----------------|---------------------|
| After create (POST) | `push` | — |
| After delete | `replace` | `push` → back leads to deleted resource (404) |
| After logout | `replace` | `push` → back returns to authenticated state |
| After form cancel | `back()` | `push('/list')` → user loses back history |
