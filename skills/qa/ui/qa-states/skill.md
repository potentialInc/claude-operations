---
name: qa-states
description: "Audit data fetching states - loading indicators, error handling, empty states, state transitions, optimistic updates, and race conditions"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA States — Loading / Error / Empty State Auditor

Analyze all data fetching calls in a frontend page or component, verify that loading indicators, error handling, and empty states are properly implemented, and detect state transition bugs.

## Execution Mode

- **Standalone** (`/qa-states [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Missing initial load indicator | **High** | Data fetch has no skeleton/spinner shown during `isLoading`/`isPending` |
| 2 | Missing refetch indicator | **Medium** | `isFetching` while data exists shows no subtle refresh indicator |
| 3 | Missing mutation loading | **High** | Mutation `isPending` not disabling/showing spinner on submit button |
| 4 | Missing pagination loading | **Medium** | No loading indicator during page change in paginated list |
| 5 | Missing loading timeout | **Medium** | Spinner spins forever with no timeout message or retry guidance |
| 6 | Layout shift on load | **Medium** | Spinner causes content jump — skeleton not matching content dimensions |
| 7 | Content disappears on refetch | **High** | Data disappears during refetch instead of showing stale content with refresh indicator |
| 8 | Missing Error Boundary | **Critical** | No Error Boundary wrapping the component tree |
| 9 | Missing per-fetch error UI | **High** | `isError` state has no rendered error message |
| 10 | Generic error message | **Medium** | Error shows "Something went wrong" instead of specific API message |
| 11 | Missing retry button | **Medium** | Error state has no button to call `refetch()` / re-trigger fetch |
| 12 | No network error handling | **Medium** | No offline detection, timeout handling, or network-specific error UI |
| 13 | Missing mutation error feedback | **High** | Mutation failure shows no toast/alert/inline error message |
| 14 | No empty state | **High** | `data.map(...)` with no empty check — nothing rendered when list is empty |
| 15 | Same message for no-data and no-results | **Medium** | Single "No data" for both initial empty and filtered-empty states |
| 16 | Empty state shown during loading | **High** | `data.length === 0` checked before `isLoading` — flash of empty state |
| 17 | Missing CTA in empty state | **Low** | Empty state has text but no "Create your first item" action button |
| 18 | Infinite loading loop | **Critical** | Missing dependency array, fetch in render body, or watcher modifying its own dependency |
| 19 | Missing optimistic rollback | **High** | `onMutate` updates cache but `onError` does not revert to previous value |
| 20 | Stale cache after mutation | **High** | No `invalidateQueries()` / `refresh()` after mutation success |
| 21 | Real-time cache inconsistency | **Critical** | Socket event updates item cache but parent list cache not invalidated |
| 22 | Stale closure / race condition | **Medium** | `useEffect` + `setState` without cleanup or AbortController |
| 23 | Missing abort on unmount | **Medium** | No `return () => controller.abort()` in useEffect cleanup |
