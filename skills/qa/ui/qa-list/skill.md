---
name: qa-list
description: Audit all list/table pages for UX consistency - sorting, search, pagination, filters, loading states, empty states, full-stack tracing, data freshness, inline edit, and export
user-invocable: true
argument-hint: "[page-path or module] [--backend <path>] [--depth full|shallow]"
---

# QA List — List/Table UX Consistency Auditor

Verify every list/table page follows consistent UX patterns: default sorting, working search, sortable columns, proper loading/empty states, filter functionality, URL state synchronization, full-stack API tracing, and data freshness.

## Execution Mode

- **Standalone** (`/qa-list [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Missing default sort | **High** | List renders in arbitrary order — no default sort column specified |
| 2 | Sort UI disconnected | **High** | Sortable header exists but missing sort direction/onClick props |
| 3 | Missing search | **Medium** | List page has no search input for filtering |
| 4 | Missing search debounce | **High** | Search input has no debounce — breaks Korean/CJK IME composition |
| 5 | Missing column resize | **Medium** | Table has no column resize support — columns are fixed width |
| 6 | Missing loading state | **High** | No loading indicator while data is being fetched |
| 7 | Missing empty state | **Medium** | No message shown when list has zero results |
| 8 | Missing filter reset | **Medium** | Filters exist but no way to reset/clear them |
| 9 | Single-select filter only | **Low** | Filter that should support multi-select only allows single selection |
| 10 | Sorted data not used | **High** | Sort hook called but template still maps over unsorted array |
| 11 | Missing result count | **Medium** | No "showing X of Y" count display |
| 12 | Multi-line toolbar | **Medium** | Search, tabs, and filters on separate lines — wastes vertical space |
| 13 | Unfillable table column | **High** | Table displays a data field that cannot be entered in any create/edit form — column is always empty |
| 14 | List-detail status mismatch | **High** | Status/badge in list row uses different logic or styling than detail page |
| 15 | Filter component type mismatch | **High** | Status filter uses Select dropdown instead of inline button group — inconsistent with other list pages |
| 16 | Filter state not in URL | **High** | Filter/sub-tab state stored in local state only — lost on browser back navigation |
| 17 | Missing full-stack tracing | **High** | Frontend pagination/sort/filter params don't match backend query API |
| 18 | URL state not synced | **Medium** | Pagination/sort/filter state lost on page refresh (not in URL params) |
| 19 | Missing data freshness strategy | **Medium** | No refetch/revalidation after mutations, stale data shown |
| 20 | Inline edit issues | **Medium** | Inline edit mode lacks save/cancel, dirty check, or optimistic update |
| 21 | Missing export functionality | **Low** | List page with >50 items has no CSV/Excel export option |
| 22 | Expanded row index vs entity field | **High** | Expandable sub-table uses `index + 1` for row numbering instead of entity's actual field |
| 23 | Hardcoded fake list items | **High** | List contains items fabricated in frontend code — fake entries mixed with real data |
