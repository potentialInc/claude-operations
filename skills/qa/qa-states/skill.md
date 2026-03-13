---
name: qa-states
description: "Audit data fetching states - loading indicators, error handling, empty states, state transitions, optimistic updates, and race conditions"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Loading / Error / Empty States

Analyze all data fetching calls in a frontend page or component, verify that loading indicators, error handling, and empty states are properly implemented, and generate a comprehensive QA report with test cases.

---

## Purpose

This skill helps you:
1. **Discover all data fetching calls** -- Find every `useQuery`, `useSWR`, `useEffect+fetch`, `useFetch`, `HttpClient` call in a page
2. **Verify loading state coverage** -- Check that every fetch has a visible loading indicator (skeleton, spinner, or Suspense fallback)
3. **Verify error state coverage** -- Check for Error Boundaries, per-fetch error UI, retry buttons, and network error handling
4. **Verify empty state coverage** -- Distinguish no-data (initial) vs no-search-results (filtered) vs loading vs error
5. **Detect state transition bugs** -- Find stuck-in-loading loops, missing dependency arrays, and stale state patterns
6. **Audit optimistic updates** -- Verify rollback on failure for mutations
7. **Generate test cases** -- Produce actionable QA test checklists for each state

**Important:** This skill is **read-only**. It analyzes source code and generates reports -- it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- Auto-detect frontend/backend directories from the project structure
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```
qa-states <page-or-component-file-path>
```

**Examples:**
```bash
# React page with TanStack Query
qa-states src/pages/UserList.tsx

# Vue page with Nuxt useFetch
qa-states pages/products/index.vue

# Shallow analysis -- only the given file, no import tracing
qa-states src/components/Dashboard.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--depth full\|shallow` | No | `full` (default): trace imports and child components. `shallow`: analyze only the given file |

---

## Workflow Overview

```
+-----------------------------------------------------------------+
|  Step 1: Input Validation & Project Detection                   |
|  - Validate file path and extension                             |
|  - Auto-detect frontend/backend directories                     |
|  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)   |
|  - Detect data fetching library (TanStack/SWR/Apollo/etc.)     |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 2: Data Fetching Pattern Discovery                        |
|  - Find all useQuery/useSWR/useFetch/HttpClient calls           |
|  - Extract loading, error, data variables per fetch             |
|  - Identify mutations (useMutation, form submits)               |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 3: Loading State Analysis                                 |
|  - Check skeleton/spinner per fetch                             |
|  - Verify initial load vs refetch indicators                    |
|  - Check Suspense boundaries and fallback quality               |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 4: Error State Analysis                                   |
|  - Detect Error Boundary components                             |
|  - Check per-fetch error UI and messages                        |
|  - Verify retry mechanisms and network error handling           |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 5: Empty State Analysis                                   |
|  - Distinguish no-data vs no-search-results                     |
|  - Check CTA buttons, illustrations, messages                   |
|  - Verify data === undefined vs data === [] handling            |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 6: State Transition & Optimistic Update Analysis          |
|  - Verify loading->success and loading->error transitions       |
|  - Detect infinite loading loops (missing deps, no timeout)     |
|  - Check optimistic update rollback on mutation failure         |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 7: Generate QA Report                                     |
|  - Data fetch inventory, loading/error/empty matrices           |
|  - State transition verification                                |
|  - Issues, test cases, recommendations                          |
+-----------------------------------------------------------------+
                          |
                          v
+-----------------------------------------------------------------+
|  Step 8: Save & Present Results                                 |
|  - Save report to .claude-project/qa/                           |
|  - Display summary with actionable items                        |
+-----------------------------------------------------------------+
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
--- Step N Complete ---
[x] Item 1
[x] Item 2
[ ] Item 3 <- Missing detected, retrying...

--- Step N Retry 1/3 ---
[x] Item 3 <- Completed

Step N: ALL PASSED (N/N) -> Proceeding to Step N+1
```

---

## Execution Steps

### Step 1: Input Validation & Project Detection

Read the file path from `$ARGUMENTS`. Parse optional flags (`--depth`).

**1.0 Project Structure Detection:**
- Auto-detect frontend directory (scan for `package.json` with frontend framework dependencies)
- Auto-detect backend directory (scan for server-side framework indicators)
- Detect framework and apply corresponding patterns

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: qa-states <page-or-component-file-path> [--depth full|shallow]
```

**1.2 Framework Detection:**

Read the target file and project's `package.json`:

| Signal | Framework |
|--------|-----------|
| `import ... from 'react'` or JSX/TSX + `react` in package.json | React |
| `react-native` in package.json + `import { View, Text } from 'react-native'` | React Native |
| `<template>` + `<script>` + `.vue` extension | Vue |
| `@Component` decorator + `@angular/core` in package.json | Angular |
| `.svelte` extension | Svelte |
| `<form>` + no framework imports + `.html` extension | Plain HTML |

**1.3 Data Fetching Library Detection:**

| Signal | Library |
|--------|---------|
| `@tanstack/react-query` or `react-query` in package.json | TanStack Query (React) |
| `@tanstack/vue-query` in package.json | TanStack Query (Vue) |
| `swr` in package.json | SWR |
| `@apollo/client` in package.json | Apollo Client |
| `@reduxjs/toolkit` + `createApi` usage | RTK Query |
| Nuxt project + `useFetch` / `useAsyncData` | Nuxt built-in |
| `axios` or `fetch` in `useEffect` / `onMounted` | Manual fetching |
| `@angular/common/http` | Angular HttpClient |
| `async` function in Server Component (no `'use client'`) | React Server Component (RSC) |
| `use(promise)` hook in client component | React 19 `use()` hook |
| `app/` directory + `loading.tsx` file convention | Next.js App Router Streaming |
| `@supabase/supabase-js` in package.json | Supabase Client |
| `supabase.from(...)` or `supabase.rpc(...)` usage | Supabase Client (direct usage) |

**Step 1 Checklist:**
```
[ ] File path validated and file exists
[ ] File extension is supported
[ ] Frontend/backend directories auto-detected
[ ] Frontend framework detected
[ ] Data fetching library detected
```

---

### Step 2: Data Fetching Pattern Discovery

Parse the component/page source code and discover ALL data fetching calls.

**2.1 Framework-Specific Detection Patterns:**

Detect framework and apply corresponding patterns:

**React (TanStack Query):**
- `useQuery({ queryKey, queryFn })` -> extract `isLoading`, `isPending`, `isFetching`, `isError`, `error`, `data`
- `useSuspenseQuery({ queryKey, queryFn })` -> no `isLoading` (handled by Suspense), extract `data`, `error`
- `useInfiniteQuery({ queryKey, queryFn })` -> extract `isLoading`, `isFetchingNextPage`, `hasNextPage`, `data`
- `useMutation({ mutationFn })` -> extract `isPending`, `isError`, `error`, `mutate`/`mutateAsync`

**React (SWR):**
- `useSWR(key, fetcher)` -> extract `isLoading`, `isValidating`, `error`, `data`
- `useSWRInfinite(getKey, fetcher)` -> extract `isLoading`, `isValidating`, `size`, `setSize`
- `useSWRMutation(key, fetcher)` -> extract `trigger`, `isMutating`, `error`

**React (Manual):**
- `useEffect(() => { fetch(...).then(setData) }, [deps])` -> look for manual `setLoading(true/false)`, `setError(err)`, `setData(res)`
- `useEffect(() => { axios.get(...) }, [deps])` -> same pattern

**React (RTK Query):**
- `useGetXQuery(arg)` -> extract `isLoading`, `isFetching`, `isError`, `error`, `data`
- `useXMutation()` -> extract `isLoading`, `isError`

**Vue (Nuxt):**
- `useFetch(url)` -> extract `pending`, `error`, `data`, `refresh`
- `useAsyncData(key, fn)` -> extract `pending`, `error`, `data`, `refresh`
- `useLazyFetch(url)` -> extract `pending`, `error`, `data`

**Vue (vue-query):**
- `useQuery({ queryKey, queryFn })` -> extract `isLoading`, `isFetching`, `isError`, `error`, `data`

**Vue (Manual):**
- `onMounted(async () => { ... fetch ... })` -> look for `loading` ref, `error` ref, `data` ref

**Angular:**
- `this.http.get<T>(url).subscribe({ next, error })` -> look for `loading` flag in component
- `this.http.get<T>(url).pipe(catchError(...))` -> check for error handling in pipe
- Resolver pattern: `resolve()` in route config -> data pre-loaded before component renders
- `AsyncPipe`: `observable$ | async` in template -> automatic subscription management

**Svelte:**
- `{#await promise}...{:then data}...{:catch error}...{/await}` -> built-in loading/error/success
- `onMount(async () => { ... })` -> manual loading/error state

**React Server Components (RSC) / Next.js App Router:**
- `async function Page()` -- Server Component with direct data fetching (no hooks needed)
- `await fetch(url)` or `await db.query(...)` in Server Component body -- data fetching at render time
- No `isLoading` variable -- loading state handled by `loading.tsx` file or `<Suspense>` boundary
- Error handling via `error.tsx` file convention (automatic Error Boundary per route segment)
- `notFound()` function call -- triggers `not-found.tsx` for 404 empty state

**React 19 `use()` hook:**
- `const data = use(promise)` -- reads a promise during render; suspends until resolved
- Requires `<Suspense>` boundary above in the tree
- No manual `isLoading` state -- Suspense fallback handles loading

**Supabase Client:**
- `supabase.from('table').select('*')` -> returns `{ data, error }` -- check for manual loading/error state handling
- `supabase.from('table').insert/update/upsert/delete(...)` -> mutation with `{ data, error }` response
- `supabase.rpc('function_name', params)` -> RPC call with `{ data, error }` response
- `supabase.auth.signIn/signUp/signOut(...)` -> auth mutations with `{ data, error }` response
- `supabase.channel('name').on('postgres_changes', ...).subscribe()` -> real-time subscription (see Step 6.4)
- When wrapped in TanStack Query: `useQuery({ queryFn: () => supabase.from(...).select() })` -> standard TanStack patterns apply
- When used directly in `useEffect` / `onMounted`: treat as manual fetch pattern with `{ data, error }` destructuring

**React `useTransition`:**
- `const [isPending, startTransition] = useTransition()` -- non-urgent state transitions
- `startTransition(() => { setFilters(newFilters) })` -- keeps previous UI visible while new data loads
- `isPending` can show subtle loading indicator without hiding current content

**2.2 Build Fetch Registry:**

For each discovered fetch, record:

| Property | Description |
|----------|-------------|
| `fetchId` | Auto-generated ID (FETCH-01, FETCH-02, ...) |
| `type` | `query` / `mutation` / `infinite` / `manual` |
| `library` | TanStack Query / SWR / RTK Query / Manual / Nuxt / Angular HttpClient |
| `endpoint` | API URL or query key |
| `loadingVar` | Variable name for loading state (`isLoading`, `pending`, etc.) |
| `errorVar` | Variable name for error state (`isError`, `error`, etc.) |
| `dataVar` | Variable name for data (`data`, `items`, etc.) |
| `sourceLocation` | `file:line` reference |

**2.3 Mutation Discovery:**

For form submits, delete actions, and update operations:
- React: `useMutation()`, `mutate()`, `mutateAsync()`
- Vue: `useMutation()`, manual `axios.post/patch/delete` in methods
- Angular: `this.http.post/patch/delete().subscribe()`

Record: mutation type (create/update/delete), loading state, error handling, success callback.

**2.4 Backend API Response Structure Awareness:**

When the backend is detected (Django REST Framework, Express, NestJS, etc.), understand the response shape to verify frontend error handling matches:

| Backend | Success Response | Error Response | Pagination |
|---------|-----------------|----------------|------------|
| Django REST Framework | `{ results: [...], count, next, previous }` (paginated) or direct data | `{ detail: "..." }` or `{ field: ["error"] }` (400) | `?page=N` or `?offset=N&limit=N` |
| Supabase | `{ data: [...], error: null }` | `{ data: null, error: { message, code } }` | `.range(from, to)` |
| NestJS | Direct data or custom DTO | `{ statusCode, message, error }` | Custom |
| Express | Varies by convention | Varies by convention | Varies |

Check: Does the frontend error handling match the actual backend error response shape? (e.g., accessing `error.detail` for Django vs `error.message` for Supabase)

**Step 2 Checklist:**
```
[ ] All useQuery/useSWR/useFetch calls discovered
[ ] All manual fetch patterns (useEffect+fetch, onMounted+axios) discovered
[ ] All mutations (useMutation, form submit, delete handler) discovered
[ ] Fetch registry built with type, library, endpoint, loading/error/data vars
[ ] Source locations recorded for each fetch
```

---

### Step 3: Loading State Analysis

For each discovered fetch, verify loading state implementation.

**3.1 Loading Indicator Type Detection:**

| Indicator Type | Detection Pattern |
|----------------|-------------------|
| Skeleton | `<Skeleton>`, `<Skeleton.Input>`, `<Skeleton.Avatar>`, `loading.io`, CSS `shimmer`, `animate-pulse` class |
| Spinner | `<Spinner>`, `<Spin>`, `<CircularProgress>`, `<Loading>`, CSS `spin` animation, `loader` class |
| Progress bar | `<Progress>`, `<LinearProgress>`, `<NProgress>`, `nprogress` |
| Placeholder text | `"Loading..."` string literal in JSX/template |
| Full page overlay | `position: fixed` + `z-index` + spinner pattern |
| React Native ActivityIndicator | `<ActivityIndicator>`, `<ActivityIndicator size="large">` |
| React Native RefreshControl | `<FlatList refreshControl={<RefreshControl refreshing={...} />}>` -- pull-to-refresh |
| React Native skeleton libs | `react-native-skeleton-placeholder`, `react-native-shimmer-placeholder` |

**3.2 Loading State Coverage Check:**

For each fetch in the registry:

| Check | Description | Assessment |
|-------|-------------|------------|
| Initial load indicator | `isLoading` / `isPending` -> shows skeleton/spinner | Present / Missing |
| Refetch indicator | `isFetching` (while data exists) -> subtle indicator | Present / Missing / N/A |
| Mutation loading | `isPending` on mutation -> button disabled/spinner | Present / Missing |
| Pagination loading | Loading during page change -> table skeleton or overlay | Present / Missing / N/A |
| Loading timeout fallback | Loading exceeds threshold (e.g. 10s) -> timeout message with retry/guidance | Present / Missing |

**3.3 Suspense Boundary Check:**

If `useSuspenseQuery` or `<Suspense>` is used:
- Is `<Suspense>` wrapping the correct boundary? (too high = everything suspends; too low = waterfall)
- Is the `fallback` meaningful? (skeleton matching actual layout vs generic spinner)
- Are multiple fetches wrapped in the same `<Suspense>`? (parallel) or nested `<Suspense>`? (waterfall -- FLAG)
- Vue: `<Suspense>` with `#default` and `#fallback` slots

**3.4 Loading Indicator Quality:**

| Quality Check | Good | Bad |
|---------------|------|-----|
| Layout shift | Skeleton matches content dimensions | Spinner causes content jump on load |
| Partial loading | Independent sections load independently | One slow fetch blocks entire page |
| Refetch experience | Stale content shown + subtle refresh indicator | Content disappears during refetch |
| Loading timeout UX | After threshold (e.g. 10s), shows timeout message with role-appropriate guidance (admin: "Check server status", user: "Please try again later") | Spinner spins forever with no feedback |

**3.5 Server Component / Streaming Loading Patterns:**

| Pattern | Detection | Assessment |
|---------|-----------|------------|
| `loading.tsx` file in same route segment | Automatic `<Suspense>` boundary by framework | OK -- streaming loading state |
| `<Suspense fallback={<Skeleton />}>` wrapping async component | Manual Suspense boundary | OK -- explicit control |
| `error.tsx` file in same route segment | Automatic Error Boundary by framework | OK -- streaming error state |
| `not-found.tsx` file + `notFound()` call | 404 empty state via framework convention | OK -- explicit not-found handling |
| No `loading.tsx` and no `<Suspense>` for async Server Component | Full page blocks until data loads | WARNING -- no streaming, poor UX |
| `useTransition` + `isPending` for non-urgent updates | Subtle loading while keeping previous UI | OK -- good UX pattern |

**Step 3 Checklist:**
```
[ ] Loading indicator type identified for each fetch
[ ] Initial load coverage verified (every fetch has a loading state)
[ ] Refetch indicator checked (isFetching vs isLoading distinction)
[ ] Mutation loading states checked (button disabled/spinner)
[ ] Suspense boundaries verified (if applicable)
[ ] Loading indicator quality assessed (skeleton vs spinner, layout shift)
[ ] Loading timeout fallback verified (10s+ shows timeout message with role-appropriate guidance)
```

---

### Step 4: Error State Analysis

For each discovered fetch, verify error handling.

**4.1 Error Boundary Detection:**

| Framework | Error Boundary Pattern |
|-----------|----------------------|
| React | `<ErrorBoundary>` from `react-error-boundary`, custom class with `componentDidCatch` / `getDerivedStateFromError` |
| React | `errorElement` in React Router route definition |
| Vue | `onErrorCaptured()` lifecycle hook in parent component |
| Vue | `<Suspense>` with `@error` event or `#fallback` on error |
| Angular | Custom `ErrorHandler` class, HTTP interceptor with error handling |
| Svelte | `{:catch error}` block in `{#await}` |
| React (App Router) | `error.tsx` file convention -- automatic Error Boundary per route segment. `not-found.tsx` for 404 states. `global-error.tsx` for root layout errors |

Check: is there at least ONE error boundary wrapping the component? How high in the tree is it?

**4.2 Per-Fetch Error Handling:**

For each fetch:

| Check | Description | Assessment |
|-------|-------------|------------|
| Error UI present | `if (isError)` -> renders error message | Present / Missing |
| Error message quality | Specific API message vs generic "Something went wrong" | Specific / Generic |
| Retry button | Button that calls `refetch()` / `refresh()` / re-triggers fetch | Present / Missing |
| Auto-retry config | `retry: 3` (TanStack), `shouldRetryOnError` (SWR) | Configured / Default / None |

**4.3 Network Error Handling:**

| Check | Detection Pattern |
|-------|-------------------|
| Offline detection | `navigator.onLine`, `online`/`offline` event listeners, `useOnlineStatus()` hook |
| Timeout handling | `AbortSignal.timeout(ms)`, `axios.defaults.timeout`, `signal` in fetch options |
| Network error UI | Specific message for network vs server errors (e.g., "Check your connection" vs "Server error") |
| Retry on reconnect | `refetchOnReconnect` (TanStack), manual `online` event -> refetch |

**4.4 Mutation Error Handling:**

For each mutation:

| Check | Description | Assessment |
|-------|-------------|------------|
| Error feedback | Toast/alert/inline message on mutation failure | Present / Missing |
| Form field errors | Server validation errors mapped to specific fields | Present / Missing / N/A |
| Rollback on failure | Optimistic update reverted | Present / Missing / N/A |
| User can retry | Submit button re-enabled after error | Present / Missing |

**Step 4 Checklist:**
```
[ ] Error Boundary detected (or absence flagged as CRITICAL)
[ ] Per-fetch error UI verified for each query
[ ] Error message quality assessed (specific vs generic)
[ ] Retry mechanism checked (button and/or auto-retry config)
[ ] Network error handling verified (offline, timeout)
[ ] Mutation error feedback verified (toast/inline/field-level)
```

---

### Step 5: Empty State Analysis

For each data display area (table, list, card grid, detail view), verify empty state handling.

**5.1 Empty State Type Detection:**

| State | Condition | Expected UI |
|-------|-----------|-------------|
| No data (initial) | `data !== undefined && data.length === 0` (no filters active) | "No items yet" + CTA button (e.g., "Create your first item") |
| No search results | `data.length === 0` && filters/search active | "No results found for [query]" + "Clear filters" button |
| Loading | `data === undefined && isLoading === true` | Skeleton or spinner (NOT empty state) |
| Error | `isError === true` | Error message + retry (NOT empty state) |

**5.2 Common Anti-Patterns:**

| Anti-Pattern | Detection | Severity |
|--------------|-----------|----------|
| No empty state at all | `data.map(...)` with no empty check before | WARNING |
| `data?.length` without empty UI | Conditional render prevents crash but shows nothing | WARNING |
| Same message for no-data and no-results | Single "No data" for both cases | INFO |
| Empty state shown during loading | `data.length === 0` checked before `isLoading` | WARNING |
| CTA button missing in no-data state | Empty state has text but no action | INFO |

**5.3 Empty State Quality Check:**

| Quality Check | Good | Bad |
|---------------|------|-----|
| Visual design | Illustration/icon + descriptive text + CTA | Plain text "No data" |
| Context awareness | Different message for no-data vs filtered-empty | Same generic message |
| Actionable | "Create your first project" button | "Nothing here" with no guidance |
| State ordering | Check `isLoading` -> `isError` -> `data.length === 0` -> render data | Check `data.length` before loading |

**Step 5 Checklist:**
```
[ ] Empty state check present for each data display area
[ ] No-data vs no-search-results distinguished
[ ] Empty state not shown during loading (correct ordering)
[ ] CTA button present in no-data empty state
[ ] Empty state quality assessed (visual, context, actionable)
```

---

### Step 6: State Transition & Optimistic Update Analysis

**6.1 State Transition Verification:**

Verify correct state flow for each fetch:

| Transition | Expected | Anti-Pattern |
|------------|----------|--------------|
| Initial -> Loading -> Success | `isLoading=true` -> `isLoading=false, data=...` | Never leaves loading (infinite loop) |
| Initial -> Loading -> Error | `isLoading=true` -> `isError=true, error=...` | Error caught but loading persists |
| Success -> Refetching -> Success | `isFetching=true` (data still visible) -> new data | Data disappears during refetch |
| Error -> Retry -> Loading -> Success | User clicks retry -> loading -> data | Retry does not trigger refetch |

**6.2 Infinite Loading Detection:**

| Pattern | Framework | Detection | Severity |
|---------|-----------|-----------|----------|
| Effect with missing dependency | React | `useEffect(() => { fetch... setData... }, [])` where `setData` triggers re-render that re-runs effect | CRITICAL |
| Fetch in render body | React | `fetch(url).then(...)` outside `useEffect` | CRITICAL |
| Missing `enabled` condition | React (TanStack) | `useQuery` runs before required params are available | WARNING |
| Watcher infinite loop | Vue | `watch(() => state, () => { fetch... state = newVal })` where watcher modifies its own dependency | CRITICAL |
| Fetch in `setup()` without lifecycle | Vue | Bare `fetch()` in `<script setup>` outside `onMounted` (runs on every re-render in some setups) | WARNING |
| Subscribe without unsubscribe | Angular | `this.http.get().subscribe()` in `ngOnInit` without `takeUntil` / `unsubscribe` in `ngOnDestroy` | CRITICAL |
| Async pipe with side effects | Angular | `observable$` triggers fetch on each template re-evaluation | WARNING |
| Reactive statement loop | Svelte | `$: data = await fetch(url)` where `data` assignment triggers re-run of same reactive block | CRITICAL |
| No timeout on manual fetch | All | `fetch(url)` without `AbortController` timeout or library-level timeout | WARNING |
| Supabase unhandled error | All | `supabase.from(...).select()` without checking `{ error }` in response | WARNING |

**6.3 Optimistic Update Analysis:**

For each mutation, check:

| Check | Description | Assessment |
|-------|-------------|------------|
| Optimistic update present | `onMutate` updates cache before API responds | Present / Not used |
| Rollback on error | `onError` reverts cache to previous value | Present / Missing |
| Cache invalidation on success | `onSuccess` -> `queryClient.invalidateQueries()` or `refresh()` | Present / Missing |
| Visual flash | Optimistic update -> error -> rollback visible to user | Needs manual test |

**6.4 Real-time/Socket Event Cache Consistency:**

When the app uses WebSocket or real-time events (Socket.IO, Pusher, SSE), verify that ALL related caches are invalidated when state changes:

| Check | Description | Severity |
|-------|-------------|----------|
| List cache after item update | Socket event updates individual item cache but does NOT invalidate parent list cache (e.g., `message:read` updates message `isRead` but rooms list `unreadCount` stays stale) | CRITICAL |
| Aggregation staleness | Derived/aggregated values (counts, badges, summaries) not refreshed after detail-level socket events | WARNING |
| Cross-page cache sync | Socket event received on detail page but list page cache not invalidated — navigating back shows stale data | CRITICAL |
| Event handler parity | Compare which socket events invalidate list cache vs which don't — all state-changing events should invalidate | WARNING |

**Detection pattern:** For each socket event handler, check if it calls `queryClient.invalidateQueries()` for the parent list query key. Compare with other event handlers (e.g., `message:new` invalidates rooms but `message:read` doesn't → bug).

**Supabase Real-time specific:**
- `supabase.channel('name').on('postgres_changes', { event, schema, table }, callback).subscribe()` -> verify callback invalidates related query caches
- Check `event` filter: `INSERT`, `UPDATE`, `DELETE`, `*` -> each should trigger appropriate cache invalidation
- Verify `supabase.removeChannel(channel)` or `channel.unsubscribe()` on component unmount to prevent memory leaks

**6.5 Race Condition Detection:**

| Pattern | Detection | Severity |
|---------|-----------|----------|
| Stale closure | `useEffect` + `setState` without cleanup / abort | WARNING |
| No abort on unmount | Missing `return () => controller.abort()` in useEffect | WARNING |
| Rapid re-fetch | No `keepPreviousData` / `placeholderData` causing flash | INFO |
| No deduplication | Same endpoint called multiple times simultaneously | WARNING |

**Step 6 Checklist:**
```
[ ] State transitions verified for each fetch (loading->success, loading->error)
[ ] Infinite loading patterns checked (missing deps, fetch in render)
[ ] Optimistic updates verified (rollback on error, cache invalidation)
[ ] Real-time/socket event cache consistency verified (all related list caches invalidated)
[ ] Race condition patterns checked (stale closure, missing abort)
[ ] Refetch behavior verified (data preserved during refetch)
```

---

### Step 7: Generate QA Report

Compile all analysis results into a structured report.

**Report Sections:**

1. **Header** -- Component name, file path, framework, data fetching library, date, overall score, status
2. **Data Fetch Inventory** -- Table of all fetches with type, library, endpoint, state variables
3. **Loading State Coverage Matrix** -- Per fetch: initial load, refetch, mutation, indicator type
4. **Error State Coverage Matrix** -- Per fetch: error UI, message quality, retry, error boundary
5. **Empty State Coverage Matrix** -- Per data display: no-data, no-results, CTA, illustration
6. **Network Error Handling** -- Offline detection, timeout, retry on reconnect
7. **State Transition Verification** -- Per fetch: correct transitions, infinite loading check
8. **Optimistic Update Assessment** -- Per mutation: optimistic update, rollback, cache invalidation
9. **Issues Found** -- All issues with severity (CRITICAL / WARNING / INFO)
10. **Test Cases** -- Complete test case table
11. **Recommendations** -- Prioritized action items

**Step 7 Checklist:**
```
[ ] Data Fetch Inventory table complete
[ ] Loading State Coverage Matrix complete
[ ] Error State Coverage Matrix complete
[ ] Empty State Coverage Matrix complete
[ ] Network Error Handling assessment complete
[ ] State Transition Verification complete
[ ] Optimistic Update Assessment complete
[ ] Issues listed with severity levels
[ ] Test cases listed with expected results
[ ] Recommendations prioritized
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_LoadingErrorEmpty_QA_Report_[YYMMDD].md` -- Full report
- `[ComponentName]_LoadingErrorEmpty_TestCases_[YYMMDD].md` -- Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Loading/Error/Empty Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_LoadingErrorEmpty_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_LoadingErrorEmpty_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React + TanStack Query / Vue + Nuxt / etc.]
- Data fetches found: [N] queries, [N] mutations
- Loading state coverage: [N]/[N] fetches have loading indicators
- Error state coverage: [N]/[N] fetches have error handling
- Empty state coverage: [N]/[N] data displays have empty states
- Error Boundary: [Present / Missing]
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
+====================================================================+
|              QA Loading/Error/Empty States Report                   |
+====================================================================+
|  Component:     [ComponentName]                                    |
|  File:          [file-path]                                        |
|  Framework:     [React + TanStack Query v5]                        |
|  Report Date:   [YYYY-MM-DD HH:MM]                                |
|  Depth:         [full / shallow]                                   |
|  Overall Score: [N]/100                                            |
|  Status:        [PASS | NEEDS ATTENTION | FAIL]                    |
+====================================================================+
|                                                                    |
|  DATA FETCH INVENTORY ([N] fetches)                                |
|                                                                    |
|  | # | Type     | Library  | Endpoint      | Loading | Error |    |
|  |---|----------|----------|---------------|---------|-------|    |
|  | 1 | query    | TanStack | GET /users    | OK      | OK    |    |
|  | 2 | query    | TanStack | GET /roles    | MISS    | MISS  |    |
|  | 3 | mutation | TanStack | POST /users   | OK      | OK    |    |
|  | 4 | manual   | axios    | GET /stats    | OK      | MISS  |    |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  LOADING STATE COVERAGE MATRIX                                     |
|                                                                    |
|  | Fetch    | Initial | Refetch | Type     | Quality   |          |
|  |----------|---------|---------|----------|-----------|          |
|  | /users   | OK      | OK      | Skeleton | Good      |          |
|  | /roles   | MISS    | MISS    | None     | MISSING   |          |
|  | /stats   | OK      | MISS    | Spinner  | OK        |          |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  ERROR STATE COVERAGE MATRIX                                       |
|                                                                    |
|  | Fetch    | Error UI | Message  | Retry  | Boundary |           |
|  |----------|----------|----------|--------|----------|           |
|  | /users   | OK       | Specific | OK     | OK       |           |
|  | /roles   | MISS     | None     | MISS   | OK       |           |
|  | /stats   | MISS     | None     | MISS   | MISS     |           |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  EMPTY STATE COVERAGE MATRIX                                       |
|                                                                    |
|  | Display   | No-Data | No-Results | CTA  | Quality  |          |
|  |-----------|---------|------------|------|----------|          |
|  | User List | OK      | OK         | OK   | Good     |          |
|  | Stats     | MISS    | N/A        | MISS | MISSING  |          |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  STATE TRANSITION VERIFICATION                                     |
|                                                                    |
|  | Fetch    | Load>OK | Load>Err | Refetch | Infinite? |         |
|  |----------|---------|----------|---------|-----------|         |
|  | /users   | OK      | OK       | OK      | No        |         |
|  | /roles   | OK      | MISS     | N/A     | No        |         |
|  | /stats   | OK      | MISS     | MISS    | WARNING   |         |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  ISSUES FOUND                                                      |
|                                                                    |
|  [CRITICAL] LEE-01: No error boundary wrapping the Stats           |
|    section. Unhandled fetch error will crash the entire page.      |
|    Fix: Wrap <StatsSection> with <ErrorBoundary>.                  |
|                                                                    |
|  [CRITICAL] LEE-02: GET /roles has no loading indicator.           |
|    Users see empty dropdown while roles are loading.               |
|    Fix: Add skeleton or spinner while isLoading is true.           |
|                                                                    |
|  [WARNING] LEE-03: GET /stats error is silently ignored.           |
|    No error UI or retry button when stats fetch fails.             |
|    Fix: Add error state check and retry button.                    |
|                                                                    |
|  [WARNING] LEE-04: Stats section may have infinite loading.        |
|    useEffect dependency array may trigger re-fetch loop.           |
|    Fix: Review dependency array and add abort controller.          |
|                                                                    |
|  [INFO] LEE-05: Empty state for user list uses same message       |
|    for no-data and no-search-results.                              |
|    Suggestion: Show different messages for better UX.              |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  TEST CASES ([N] total)                                            |
|    Loading: [N] | Error: [N] | Empty: [N] | Transition: [N]      |
|    [See full list in test cases file]                              |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  RECOMMENDATIONS                                                   |
|  1. [Critical] Add ErrorBoundary wrapping Stats section            |
|  2. [Critical] Add loading indicator for /roles fetch              |
|  3. [Warning] Add error UI and retry for /stats fetch              |
|  4. [Warning] Review useEffect deps in Stats to prevent loop      |
|  5. [Info] Distinguish no-data vs no-results empty states          |
|                                                                    |
+====================================================================+
```

---

## Test Case Categories

### Category 1: Loading State Tests (LOAD-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| LOAD-01 | Initial page load | 1. Navigate to page (clear cache) | Skeleton/spinner shown for all data sections |
| LOAD-02 | Slow network (3G) | 1. Throttle network 2. Navigate to page | Loading indicator visible for extended time, no broken layout |
| LOAD-03 | Refetch after mutation | 1. Create/update item 2. Observe list | Subtle refetch indicator, data remains visible |
| LOAD-04 | Pagination loading | 1. Click next page | Loading indicator for table, not full page |
| LOAD-05 | Mutation loading | 1. Click submit/delete button | Button shows loading state, disabled during request |
| LOAD-06 | Loading timeout (10s+) | 1. Throttle network or mock slow API 2. Wait 10+ seconds | Spinner replaced by timeout message with role-appropriate guidance (admin: "Check server status", user: "Please try again later") and optional retry button |

### Category 2: Error State Tests (ERR-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| ERR-01 | API server error (500) | 1. Mock 500 response | Error message displayed with retry button |
| ERR-02 | Network offline | 1. Disconnect network 2. Trigger fetch | Offline message displayed, retry on reconnect |
| ERR-03 | Request timeout | 1. Mock very slow response | Timeout message after threshold, retry available |
| ERR-04 | Partial failure | 1. Mock one fetch fails, others succeed | Failed section shows error, other sections render normally |
| ERR-05 | Retry after error | 1. Trigger error 2. Click retry button | Refetch triggered, loading shown, data loaded on success |
| ERR-06 | Mutation failure | 1. Submit form with server error | Error toast/message shown, form remains editable |

### Category 3: Empty State Tests (EMPTY-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| EMPTY-01 | No data (fresh account) | 1. Load page with zero items | Empty state with illustration + CTA ("Create your first...") |
| EMPTY-02 | No search results | 1. Enter search term with no matches | "No results" message + "Clear search" button |
| EMPTY-03 | No filter results | 1. Apply filters that match nothing | "No items match your filters" + "Clear filters" button |
| EMPTY-04 | Data deleted to zero | 1. Delete last item in list | Transitions to no-data empty state |

### Category 4: State Transition Tests (TRANS-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| TRANS-01 | Loading to success | 1. Load page | Loading -> data displayed, no flash |
| TRANS-02 | Loading to error | 1. Mock API error 2. Load page | Loading -> error state, not stuck in loading |
| TRANS-03 | Error to retry to success | 1. Mock error 2. Fix mock 3. Click retry | Error -> loading -> success |
| TRANS-04 | Optimistic update rollback | 1. Optimistically update 2. API fails | Updated UI -> reverts to previous state |
| TRANS-05 | Rapid navigation | 1. Navigate to page 2. Immediately navigate away 3. Return | No stale data, no memory leak warnings |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Loading State Coverage | 20% | Every fetch has a loading indicator; appropriate type used |
| Error State Coverage | 20% | Every fetch has error handling; Error Boundary exists; retry available |
| Empty State Coverage | 15% | No-data vs no-results distinguished; CTA present |
| Network Error Handling | 10% | Offline detection, timeout, retry on network failure |
| State Transition Correctness | 15% | No stuck-in-loading, no error-then-loading-forever |
| Optimistic Update Safety | 10% | Rollback on failure, cache invalidation on success |
| Suspense/Boundary Design | 10% | Correct boundary placement, no waterfall, meaningful fallback |

**Scoring rules** (see `qa-shared-reference` skill for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Each fetch with full loading+error+empty coverage: +2
  - Error Boundary wrapping the page: +4
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
| No data fetching found | Static/display-only component | Warning shown; component may not fetch data |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| Data fetching library not detected | Custom wrapper or uncommon library | Falls back to manual fetch pattern detection |
| Import chain unresolvable | Complex re-exports or circular imports | Breaks after 10 levels; notes as "unresolvable" |
| Suspense boundary not in analyzed file | Suspense defined in parent/layout | Notes as "externally managed -- check parent component" |

---

## Edge Cases Handled

- **Multiple independent fetches** -- One errors while others succeed; partial rendering analysis
- **Suspense waterfall** -- Nested `<Suspense>` causing sequential loading; detected and flagged
- **Skeleton mismatch** -- Skeleton layout does not match actual content; flagged as INFO
- **Loading state during mutation** -- Delete button should show loading, not whole page
- **Error Boundary vs fetch error** -- Render errors caught by boundary vs data errors caught by query; different handling
- **Infinite scroll error** -- Error at bottom of list, not replacing all content
- **SSR/SSG pre-rendered pages** -- Skip loading state analysis for statically rendered pages
- **Race condition on unmount** -- State update on unmounted component; check for abort controller / cleanup
- **`keepPreviousData` / `placeholderData`** -- Stale data shown during refetch; noted as intentional
- **Conditional fetching** -- `enabled: false` until params ready; verify loading state handles disabled state
- **Parallel vs sequential fetches** -- Multiple fetches that should load in parallel but are serial due to Suspense
- **i18n error/empty messages** -- `t('key')` patterns; note actual text in locale files
- **React Server Components** -- `async` Server Components fetch data at render time; loading state via `loading.tsx` or `<Suspense>`, not `isLoading` variable
- **Streaming SSR** -- HTML sent in chunks via Suspense boundaries; each chunk can independently show loading/error states
- **React 19 `use()` hook** -- Reads promises in render; requires `<Suspense>` above in tree; no manual loading state
- **`useTransition` for non-urgent updates** -- `isPending` flag for subtle loading indicator while keeping previous UI visible
- **Framework-specific error files** -- Route-segment-level Error Boundary files (e.g., `error.tsx`); different from `<ErrorBoundary>` component
- **Framework-specific not-found handling** -- Dedicated 404 empty state triggered by framework conventions
- **React Native FlatList empty** -- `ListEmptyComponent` prop for empty state rendering; `ListFooterComponent` for pagination loading
- **Framework-specific loading files** -- Automatic Suspense boundary per route segment for streaming
- **Partial data + partial error** -- GraphQL partial errors where some fields succeed and others fail in single response
- **Progressive loading** -- Dashboard pages loading critical data first, secondary data later via nested Suspense
- **Socket event cache staleness** -- Real-time event (e.g., `message:read`) updates detail cache but not list cache; navigating back shows stale aggregated values (e.g., unread badge count). All socket event handlers must invalidate related list queries

---

## Related Skills

- `qa-input-fields` -- QA all input fields in the same page/component
- `qa-back-navigation` -- QA back navigation behavior
- `qa-table-list` -- QA table/list data display (tables need loading/error/empty states)
- `qa-modal-drawer` -- QA modal/drawer behavior (modals may load data)
- `qa-permission-role` -- QA permission/role access (403 error states)
- `review-command` -- Validate this skill's structure and quality

---

## Tips

1. **Every fetch needs all three states** -- Loading, error, and empty. If any is missing, it's at minimum a WARNING
2. **Error Boundaries are non-negotiable** -- Without one, a single fetch error crashes the entire page (white screen)
3. **Distinguish no-data from no-results** -- Users need different guidance: "Create your first item" vs "Try different filters"
4. **Check `isLoading` before `data.length`** -- The most common anti-pattern is showing empty state while data is still loading
5. **Mutations need error feedback too** -- A failed delete with no error message leaves users confused
6. **Suspense waterfalls kill performance** -- If you see nested `<Suspense>`, check if the inner fetches depend on outer data
7. **Re-run after adding new fetches** -- Every new `useQuery` or API call should be analyzed for state coverage
8. **Combine with `qa-table-list`** -- Tables are the #1 component needing loading/error/empty states
