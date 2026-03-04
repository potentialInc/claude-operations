---
description: Analyze table/list data display components and generate a comprehensive QA report covering pagination, sorting, filtering, empty states, row actions, and data freshness
argument-hint: "<page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow]"
---

# QA Table / List

Analyze table, data grid, and list components in a frontend page, trace pagination/sorting/filtering logic across the full stack, and generate a comprehensive QA report with test cases.

---

## Purpose

This command helps you:
1. **Discover table/list components** — Find every data table, data grid, and list in a page
2. **Analyze pagination** — Offset vs cursor vs infinite scroll, URL sync, boundary cases
3. **Analyze sorting** — Single/multi column, sort direction, server-side vs client-side, default sort
4. **Analyze filtering** — Filter types, debounce, URL param sync, filter reset, combined state
5. **Verify empty states** — No data, no search results, loading, error states for tables
6. **Audit row actions** — Row click, selection, bulk actions, inline edit, delete confirmation
7. **Trace data freshness** — Refetch strategies, optimistic updates, stale data handling
8. **Generate test cases** — Produce actionable QA test checklists

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack pagination/sorting tracing
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```bash
/qa-table-list <page-or-component-file-path>
```

**Examples:**
```bash
# React page with TanStack Table
/qa-table-list src/pages/UserList.tsx

# With backend tracing
/qa-table-list frontend/app/pages/orders/index.tsx --backend backend/src

# Frontend-only analysis
/qa-table-list src/components/ProductTable.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): trace through API and backend. `shallow`: frontend-only analysis |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection                │
│  - Validate file path and extension                          │
│  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)│
│  - Detect table library (TanStack/Ant/MUI/AG Grid/etc.)     │
│  - Detect backend framework (if --depth full)                │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Table/List Component Discovery                      │
│  - Find all table, data grid, list components                │
│  - Extract column definitions, data source                   │
│  - Detect list patterns (.map, virtual scroll, infinite)     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Pagination Analysis                                 │
│  - Detect pagination type (offset/cursor/infinite scroll)    │
│  - Check URL sync, boundary cases, page size selector        │
│  - Trace to backend (if --depth full)                        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Sorting & Filtering Analysis                        │
│  - Detect sort state, single/multi column, direction         │
│  - Detect filter types, debounce, URL sync, reset            │
│  - Check page reset on sort/filter change                    │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Empty State, Loading & Row Action Analysis          │
│  - Verify 4 states: no-data, no-results, loading, error     │
│  - Audit row click, selection, bulk actions, delete confirm  │
│  - Check column rendering: truncation, responsive, custom    │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Data Freshness & API Tracing                        │
│  - Trace data fetch to API endpoint                          │
│  - Check refetch strategies, optimistic updates              │
│  - Backend: verify pagination params pass to DB query        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Generate QA Report                                  │
│  - Table inventory, pagination/sorting/filtering matrices    │
│  - Empty state coverage, row action audit                    │
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
Usage: /qa-table-list <page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow]
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

**1.3 Table Library Detection:**

| Signal | Library |
|--------|---------|
| `@tanstack/react-table` in package.json | TanStack Table (React) |
| `@tanstack/vue-table` in package.json | TanStack Table (Vue) |
| `antd` + `<Table>` usage | Ant Design Table |
| `@mui/x-data-grid` | MUI DataGrid |
| `ag-grid-react` / `ag-grid-vue` / `ag-grid-angular` | AG Grid |
| `@shadcn/ui` + DataTable pattern | shadcn DataTable |
| `element-plus` + `<el-table>` | Element Plus Table |
| `vuetify` + `<v-data-table>` | Vuetify DataTable |
| `@angular/material` + `mat-table` | Angular Material Table |
| `primereact` / `primevue` / `primeng` + DataTable | PrimeNG/PrimeReact/PrimeVue |
| No library — raw `<table>` + `.map()` | Custom/Native Table |

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
[ ] Table library detected (or native table/list pattern)
[ ] Backend framework detected (or --depth shallow acknowledged)
```

---

### Step 2: Table/List Component Discovery

Parse the component/page source and discover ALL table and list components.

**2.1 Framework-Specific Detection Patterns:**

**React + TanStack Table:**
- `useReactTable({ data, columns, getCoreRowModel, ... })` — main table hook
- Column definitions: `columnHelper.accessor('field', { header, cell, enableSorting, enableColumnFilter })`
- `flexRender(header.column.columnDef.header, header.getContext())` — render pattern

**React + Ant Design Table:**
- `<Table dataSource={data} columns={columns} pagination={...} />` — declarative
- Column definitions: `{ title, dataIndex, key, sorter, filters, render }`

**React + MUI DataGrid:**
- `<DataGrid rows={data} columns={columns} />` — declarative
- Column definitions: `{ field, headerName, width, sortable, filterable, renderCell }`

**React + AG Grid:**
- `<AgGridReact rowData={data} columnDefs={columnDefs} />` — declarative
- Column definitions: `{ field, headerName, sortable, filter, cellRenderer }`

**Vue + Element Plus:**
- `<el-table :data="tableData">` + `<el-table-column prop="field" label="Label" sortable />`
- Events: `@sort-change`, `@filter-change`, `@selection-change`

**Vue + Vuetify:**
- `<v-data-table :items="data" :headers="headers" />` — declarative
- Props: `:items-per-page`, `v-model:page`, `v-model:sort-by`

**Angular + Material:**
- `<table mat-table [dataSource]="dataSource">` + `<ng-container matColumnDef="field">`
- `<mat-paginator>`, `matSort`, `(matSortChange)="onSort($event)"`

**Native/Custom:**
- `data.map((item) => <tr>...</tr>)` — manual table rendering
- `<ul>` + `data.map((item) => <li>...)` — list rendering

**2.2 Build Table Registry:**

For each discovered table/list, record:

| Property | Description |
|----------|-------------|
| `tableId` | Auto-generated ID (TABLE-01, TABLE-02, ...) |
| `type` | `dataTable` / `dataGrid` / `list` / `cardGrid` / `virtualList` |
| `library` | TanStack / Ant Design / MUI / AG Grid / Element Plus / Custom |
| `dataSource` | Variable or expression providing data (`data`, `users`, `items`) |
| `columns` | Number of columns and their field names |
| `hasPagination` | Boolean |
| `hasSorting` | Boolean |
| `hasFiltering` | Boolean |
| `hasRowSelection` | Boolean |
| `sourceLocation` | `file:line` reference |

**2.3 Column Definition Extraction:**

For each column, extract:

| Property | Description |
|----------|-------------|
| `field` | Data field name (e.g., `username`, `email`) |
| `header` | Display header text |
| `sortable` | Whether column supports sorting |
| `filterable` | Whether column supports filtering |
| `customRenderer` | Whether a custom cell renderer is used |
| `truncation` | Whether text truncation is applied |
| `responsive` | Whether column hides on small screens |

**2.4 Virtualization Detection:**

Detect virtual scroll/list libraries for large datasets:

| Library | Detection Pattern |
|---------|-------------------|
| `@tanstack/react-virtual` | `useVirtualizer()` hook |
| `react-window` | `<FixedSizeList>`, `<VariableSizeList>` components |
| `react-virtuoso` | `<Virtuoso>`, `<TableVirtualoso>`, `<GroupedVirtualoso>` |
| `vue-virtual-scroller` | `<RecycleScroller>`, `<DynamicScroller>` |
| AG Grid built-in | `rowModelType="infinite"` or `rowModelType="serverSide"` |

Assessment: If table has 100+ rows without virtualization, flag as INFO. If 1000+ rows without virtualization, flag as WARNING for performance.

**Step 2 Checklist:**
```
[ ] All table/dataGrid components discovered
[ ] All list/cardGrid rendering patterns discovered
[ ] Column definitions extracted with field, header, sortable, filterable
[ ] Table registry built with type, library, data source
[ ] Custom renderers and truncation patterns noted
```

---

### Step 3: Pagination Analysis

For each table/list, analyze pagination implementation.

**3.1 Pagination Type Detection:**

| Type | Detection Pattern | Framework Examples |
|------|-------------------|-------------------|
| Offset-based | `page` + `pageSize` (or `limit`/`offset`) state | Ant `pagination={{ current, pageSize, total }}`, MUI `paginationModel` |
| Cursor-based | `cursor` + `nextCursor` / `hasMore` | TanStack Query `useInfiniteQuery` with cursor param |
| Infinite scroll | `IntersectionObserver`, `useInView`, `onEndReached` | `useInfiniteQuery` + `fetchNextPage`, `react-infinite-scroll-component` |
| Load more button | Button triggers next page fetch | `<Button onClick={fetchNextPage}>Load More</Button>` |
| No pagination | All data loaded at once | No page/limit params in API call |

**3.2 URL Synchronization Check:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Page in URL | `?page=2` or search param state | Synced / Not synced |
| Page size in URL | `?pageSize=20` | Synced / Not synced |
| Refresh preserves page | Refreshing browser keeps current page | Yes / No |
| Back preserves page | Browser back returns to previous page number | Yes / No |

Detection patterns:
- `useSearchParams()` / `searchParams.get('page')` — React Router / Next.js
- `useRoute().query.page` — Vue Router / Nuxt
- `ActivatedRoute.queryParams` — Angular
- Manual: `window.location.search` / `URLSearchParams`

**3.3 Boundary Case Analysis:**

| Check | Description | Severity |
|-------|-------------|----------|
| Page 0 handling | Does the system handle `page=0`? (Off-by-one) | WARNING |
| Page exceeds total | What happens when `page > totalPages`? | WARNING |
| Negative page | `page=-1` in URL | WARNING |
| Max page size guard | Does backend accept arbitrary `pageSize=999999`? | CRITICAL |
| Empty last page | Last page with 0 items (total exactly divisible by pageSize) | INFO |

**3.4 Page Size Selector:**

| Check | Description |
|-------|-------------|
| Selector exists | Dropdown allowing user to change page size |
| Options | What page sizes are offered (10, 20, 50, 100)? |
| Backend alignment | Does backend accept all offered page sizes? |
| Default size | Is the default page size reasonable for the data type? |

**3.5 Backend Pagination Tracing** (if `--depth full`):

Trace the API call to the backend:
- Frontend: `GET /api/users?page=2&pageSize=20` → Backend controller
- Controller: extract `page`, `pageSize` from query params
- Service/Repository: translate to `skip`/`take` or `offset`/`limit`
- SQL: verify `LIMIT` and `OFFSET` in query
- Check: is `total` count returned? Is it a separate count query?

**Step 3 Checklist:**
```
[ ] Pagination type detected for each table (offset/cursor/infinite/none)
[ ] URL synchronization checked (page, pageSize in URL params)
[ ] Boundary cases analyzed (page=0, exceeds total, negative, max size)
[ ] Page size selector checked (exists, options, backend alignment)
[ ] Backend pagination traced (if --depth full)
[ ] Total count mechanism identified
```

---

### Step 4: Sorting & Filtering Analysis

**4.1 Sorting Analysis:**

For each table, check:

| Check | Description | Assessment |
|-------|-------------|------------|
| Sort state | How sort column and direction are managed | State var / URL param / Library internal |
| Single vs multi-column | Can user sort by multiple columns? | Single / Multi |
| Default sort | Is there a default sort column on page load? | Present / Missing |
| Sort direction toggle | Click cycles: unsorted → asc → desc (→ unsorted?) | Correct / Missing direction indicator |
| Server-side vs client-side | Does sorting trigger API call or sort locally? | Server / Client |
| Sort + page reset | Does changing sort reset page to 1? | Yes / No |
| Sort indicator UI | Column header shows sort direction (arrow/icon) | Present / Missing |

Detection patterns:
- TanStack: `getSortedRowModel()` (client), `manualSorting` (server), `onSortingChange`
- Ant Design: `sorter: true` or `sorter: (a, b) => ...` in column def, `onChange` 3rd param
- MUI DataGrid: `sortModel`, `onSortModelChange`, `sortingMode="server"`
- Element Plus: `sortable="custom"`, `@sort-change` event
- URL: `?sortBy=name&sortOrder=asc`

**4.2 Filtering Analysis:**

For each table, check:

| Check | Description | Assessment |
|-------|-------------|------------|
| Filter types | Text search, dropdown select, date range, multi-select | Identified per column |
| Debounce on text | Text input triggers API after delay, not per keystroke | Present (ms) / Missing |
| Filter URL sync | Filter values persisted in URL params | Synced / Not synced |
| Filter reset | "Clear All Filters" button exists | Present / Missing |
| Filter + page reset | Does applying a filter reset page to 1? | Yes / No |
| Server-side vs client-side | Does filtering trigger API call or filter locally? | Server / Client |
| Combined filter state | Multiple active filters work correctly together | Yes / Issues |

Detection patterns:
- Search input: `<Input>` + `onChange` → `setSearch(value)` → API call (check debounce)
- Debounce: `useDebouncedValue`, `useDebounce`, `lodash.debounce`, `setTimeout` pattern
- Dropdown filter: `<Select>` + `onChange` → `setFilter(value)` → API call
- Date range: `<DateRangePicker>` → start/end params in API call
- URL sync: filter values in `useSearchParams` or router query

**4.3 Combined State Interaction:**

| Interaction | Expected Behavior | Common Bug |
|-------------|-------------------|------------|
| Sort change → page | Page resets to 1 | Stays on current page (shows wrong data subset) |
| Filter change → page | Page resets to 1 | Stays on page 3 but only 2 pages of results |
| Filter change → sort | Sort preserved (or reset to default) | Sort lost, data appears unordered |
| Clear filters → page/sort | Page resets, sort optionally preserved | Page stays, stale URL params |

**Step 4 Checklist:**
```
[ ] Sort state management identified (state var, URL, library internal)
[ ] Default sort column verified
[ ] Sort direction toggle and UI indicator checked
[ ] Server-side vs client-side sorting determined
[ ] Filter types identified for each filterable column
[ ] Debounce verified on text filter inputs
[ ] Filter URL synchronization checked
[ ] Filter reset (clear all) button checked
[ ] Page reset on sort/filter change verified
[ ] Combined state interactions verified
```

---

### Step 5: Empty State, Loading & Row Action Analysis

**5.1 Table-Specific State Coverage:**

For each table/list, verify these 4 states:

| State | Condition | Expected UI |
|-------|-----------|-------------|
| Loading (initial) | Data not yet fetched | Table skeleton / spinner (not empty table) |
| Loading (page/sort/filter change) | Refetching with new params | Subtle overlay or row skeleton (data may remain visible) |
| No data (empty) | `data.length === 0` with no active filters | "No items yet" + CTA ("Create your first item") |
| No search results | `data.length === 0` with active filters/search | "No results for [query]" + "Clear filters" button |
| Error | Fetch failed | Error message + retry button (not blank table) |

**5.2 Row Action Analysis:**

| Action | Detection | Check |
|--------|-----------|-------|
| Row click (navigate) | `onRow={{ onClick }}` / `<tr onClick>` / `<Link>` wrapping row | Does it navigate to detail page? |
| Row selection (checkbox) | `rowSelection` state, `<Checkbox>` in first column | Selection state persisted across pages? |
| Bulk actions | Toolbar/button bar appears when rows selected | What actions available? (delete, export, etc.) |
| Inline edit | `editable` column, `<Input>` inside cell | Save/cancel mechanism? |
| Delete (single) | Delete button/icon per row | Confirmation dialog before delete? |
| Delete (bulk) | Delete selected button | Confirmation with count? |
| Context menu | Right-click menu on row | Actions listed? |
| Drag & drop reorder | `@dnd-kit/sortable`, `react-beautiful-dnd`, `vue-draggable` | Server sync after reorder? Undo mechanism? |
| Column resize | TanStack `enableResizing`, AG Grid column resize | Persistence to localStorage or user settings? |
| Column reorder | TanStack `enableColumnReordering`, drag column header | Persistence to localStorage or user settings? |
| Copy cell content | `onClick` on cell copies to clipboard | Feedback (toast/tooltip) on copy? |

**5.3 Row Action Safety:**

| Check | Description | Severity |
|-------|-------------|----------|
| Delete without confirmation | Delete triggers immediately, no dialog | CRITICAL |
| Bulk delete without confirmation | Bulk delete with no "Are you sure? N items" | CRITICAL |
| Selection lost on page change | Selecting rows, changing page, returning — selection gone | WARNING |
| Bulk action bar not visible | Rows selected but no visible action bar | WARNING |
| Row click conflicts with checkbox | Clicking row navigates AND toggles checkbox | WARNING |

**5.4 Column Rendering Quality:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Text truncation | Long text truncated with ellipsis + tooltip | Present / Missing |
| Responsive columns | Columns hidden on mobile or table scrolls horizontally | Responsive / Fixed / No handling |
| Custom renderers | Status badges, avatars, dates formatted, links | Appropriate / Raw data shown |
| Column alignment | Numbers right-aligned, text left-aligned | Correct / Incorrect |
| Date formatting | Dates in human-readable format, not raw ISO | Formatted / Raw |

**5.5 Export Analysis:**

If table has an export button, check:

| Check | Description | Assessment |
|-------|-------------|------------|
| Export scope | Current page only vs all pages vs filtered results | Documented? User knows? |
| Server-side export | Large datasets use backend CSV generation | Present / Frontend-only (may crash on large data) |
| Export respects filters | Applied filters reflected in exported data | Yes / No |
| Export respects sort | Applied sort order reflected in exported data | Yes / No |
| Export format | CSV, Excel (.xlsx), PDF | Appropriate for data type? |
| Export loading state | Progress indicator for large exports | Present / Missing |

**5.6 Real-Time Data Analysis:**

If table data updates in real-time (WebSocket, polling):

| Check | Description | Assessment |
|-------|-------------|------------|
| New row insertion | New rows appear while user is viewing table | Animated / Flash / No indication |
| Row update | Existing row data changes while visible | Highlighted / Silent update |
| Row deletion | Row disappears while user is viewing | Animated removal / Sudden disappear |
| Selection state | Selected rows still valid after real-time update removes them | Handled / Selection breaks |
| Scroll position | Scroll position maintained when new data arrives | Maintained / Jumps to top |
| Pagination count | Total count updates when real-time data changes item count | Updated / Stale count |

**Step 5 Checklist:**
```
[ ] Loading state verified for initial load and page/sort/filter changes
[ ] No-data empty state verified (with CTA)
[ ] No-search-results empty state verified (with clear filters)
[ ] Error state verified (with retry button)
[ ] Row actions identified (click, select, bulk, delete, inline edit)
[ ] Delete actions have confirmation dialogs
[ ] Selection persistence across pages checked
[ ] Column rendering quality assessed (truncation, responsive, formatting)
```

---

### Step 6: Data Freshness & API Tracing

**6.1 Data Fetching Pattern:**

Trace how table data is fetched:

| Pattern | Detection |
|---------|-----------|
| TanStack Query | `useQuery({ queryKey: ['users', page, sort, filters], queryFn })` |
| SWR | `useSWR(\`/users?page=${page}\`, fetcher)` |
| RTK Query | `useGetUsersQuery({ page, sort })` |
| Nuxt | `useFetch('/api/users', { query: { page, sort } })` |
| Manual | `useEffect(() => { fetch(url) }, [page, sort, filters])` |

**6.2 Refetch Strategy Check:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Window focus refetch | `refetchOnWindowFocus` enabled | Yes / No |
| Polling interval | `refetchInterval` set for real-time data | Set (ms) / None |
| Manual refetch | Refresh button available to user | Present / Missing |
| After mutation | Data refetched after create/update/delete | `invalidateQueries` / `refresh` / Missing |
| Stale time | `staleTime` configured to prevent unnecessary refetches | Configured / Default (0) |

**6.3 Optimistic Update Analysis:**

For mutations that affect table data (create, update, delete):

| Check | Description | Assessment |
|-------|-------------|------------|
| Optimistic update | UI updates before API responds | Used / Not used |
| Rollback on error | UI reverts if API fails | Present / Missing |
| Cache invalidation | Table data refreshed after successful mutation | Present / Missing |

**6.4 Backend API Tracing** (if `--depth full`):

Trace the full data flow:

```
Frontend: useQuery(['users', { page: 2, sort: 'name', filter: 'active' }])
  → GET /api/users?page=2&pageSize=20&sortBy=name&sortOrder=asc&status=active
    → Controller: @Get() findAll(@Query() query: ListUsersDto)
      → Service: this.userRepository.findAndCount({ skip, take, order, where })
        → SQL: SELECT * FROM users WHERE status='active' ORDER BY name ASC LIMIT 20 OFFSET 20
```

Check:
- Are all frontend params passed to backend? (page, sort, filter)
- Are params sanitized? (SQL injection risk for raw sort column name)
- Is `total` count accurate? (separate COUNT query or `findAndCount`)
- Does backend support all sort columns the frontend offers?
- Does backend validate sort direction (only `ASC`/`DESC`)?

**Step 6 Checklist:**
```
[ ] Data fetching pattern identified (library, query key, endpoint)
[ ] Refetch strategy checked (window focus, polling, manual, after mutation)
[ ] Optimistic updates verified (if applicable)
[ ] Cache invalidation after mutation verified
[ ] Backend API traced (if --depth full)
[ ] Sort/filter params sanitized on backend (if --depth full)
[ ] Total count mechanism verified (if --depth full)
```

---

### Step 7: Generate QA Report

Compile all analysis results into a structured report.

**Report Sections:**

1. **Header** — Component name, file path, framework, table library, date, overall score, status
2. **Table/List Inventory** — Table of all tables/lists with type, library, data source, columns
3. **Pagination Analysis** — Type, URL sync, boundary cases, page size selector, backend trace
4. **Sorting Analysis** — Sort state, single/multi, default, direction, server/client, UI indicator
5. **Filtering Analysis** — Filter types, debounce, URL sync, reset, page reset
6. **Empty State Coverage Matrix** — No-data, no-results, loading, error per table
7. **Row Action Audit** — Click, selection, bulk, delete confirmation, inline edit
8. **Data Freshness Assessment** — Refetch strategy, optimistic updates, cache invalidation
9. **Issues Found** — All issues with severity (CRITICAL / WARNING / INFO)
10. **Test Cases** — Complete test case table
11. **Recommendations** — Prioritized action items

**Step 7 Checklist:**
```
[ ] Table/List Inventory complete
[ ] Pagination Analysis complete
[ ] Sorting Analysis complete
[ ] Filtering Analysis complete
[ ] Empty State Coverage Matrix complete
[ ] Row Action Audit complete
[ ] Data Freshness Assessment complete
[ ] Issues listed with severity levels
[ ] Test cases listed with expected results
[ ] Recommendations prioritized
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_TableList_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_TableList_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Table/List Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_TableList_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_TableList_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React + TanStack Table / Vue + Element Plus / etc.]
- Tables/Lists found: [N] total ([N] dataTable, [N] list, [N] cardGrid)
- Pagination: [type] (URL sync: [Yes/No])
- Sorting: [N] sortable columns (server/client)
- Filtering: [N] filter types (debounce: [Yes/No])
- Empty states: [N]/[N] tables have full coverage
- Row actions: [list] (delete confirmation: [Yes/No])
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
║                   QA Table / List Report                           ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                    ║
║  File:          [file-path]                                        ║
║  Framework:     [React + TanStack Table + TanStack Query]          ║
║  Backend:       [NestJS + TypeORM]                                 ║
║  Report Date:   [YYYY-MM-DD HH:MM]                                ║
║  Depth:         [full / shallow]                                   ║
║  Overall Score: [N]/100                                            ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  TABLE/LIST INVENTORY ([N] tables)                                 ║
║                                                                    ║
║  | # | Type      | Library  | Columns | Rows Source    |          ║
║  |---|-----------|----------|---------|----------------|          ║
║  | 1 | DataTable | TanStack | 8       | useQuery /users|          ║
║  | 2 | CardGrid  | Custom   | N/A     | useQuery /stats|          ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  PAGINATION ANALYSIS                                               ║
║                                                                    ║
║  | Table  | Type    | URL Sync | Page Reset | Boundary | Backend |║
║  |--------|---------|----------|------------|----------|---------|║
║  | /users | Offset  | ✅ page  | ✅         | ⚠ no max | ✅      |║
║  |        |         | ✅ size  |            | size     |         |║
║  | /stats | None    | N/A      | N/A        | N/A      | N/A     |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  SORTING ANALYSIS                                                  ║
║                                                                    ║
║  | Column   | Sortable | Default | Server | URL Sync | Indicator |║
║  |----------|----------|---------|--------|----------|-----------|║
║  | name     | ✅       | ✅ ASC  | ✅     | ✅       | ✅        |║
║  | email    | ✅       | —       | ✅     | ✅       | ✅        |║
║  | created  | ✅       | —       | ✅     | ✅       | ✅        |║
║  | status   | ❌       | —       | N/A    | N/A      | N/A       |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  FILTERING ANALYSIS                                                ║
║                                                                    ║
║  | Filter     | Type       | Debounce | URL Sync | Page Reset |  ║
║  |------------|------------|----------|----------|------------|  ║
║  | Search     | Text       | ✅ 300ms | ✅       | ✅         |  ║
║  | Status     | Dropdown   | N/A      | ✅       | ✅         |  ║
║  | Date Range | DatePicker | N/A      | ❌       | ❌ MISSING |  ║
║                                                                    ║
║  Clear Filters Button: ✅                                         ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  EMPTY STATE COVERAGE                                              ║
║                                                                    ║
║  | Table  | No-Data | No-Results | Loading | Error |             ║
║  |--------|---------|------------|---------|-------|             ║
║  | /users | ✅ +CTA | ✅ +Clear  | ✅ Skel | ✅    |             ║
║  | /stats | ❌      | N/A        | ✅ Spin | ❌    |             ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ROW ACTION AUDIT                                                  ║
║                                                                    ║
║  | Action     | Present | Confirmation | Safe? |                  ║
║  |------------|---------|--------------|-------|                  ║
║  | Row click  | ✅ → detail page     | N/A   | ✅    |           ║
║  | Checkbox   | ✅ multi-select      | N/A   | ✅    |           ║
║  | Delete     | ✅ per-row           | ✅    | ✅    |           ║
║  | Bulk delete| ✅                   | ❌    | ⚠     |           ║
║  | Export     | ✅ CSV               | N/A   | ✅    |           ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ISSUES FOUND                                                      ║
║                                                                    ║
║  [CRITICAL] TBL-01: No max page size guard on backend.            ║
║    Frontend offers pageSize=100 but backend accepts any value.    ║
║    Risk: pageSize=999999 fetches entire table, DoS risk.          ║
║    Fix: Add max pageSize validation in DTO (e.g., Max(100)).      ║
║                                                                    ║
║  [CRITICAL] TBL-02: Bulk delete has no confirmation dialog.       ║
║    Selecting 50 rows and clicking delete triggers immediately.    ║
║    Fix: Add AlertDialog showing "Delete 50 items?" confirmation.  ║
║                                                                    ║
║  [WARNING] TBL-03: Date range filter does not reset page to 1.   ║
║    Applying date filter stays on current page, may show empty.    ║
║    Fix: Reset page to 1 in date filter onChange handler.          ║
║                                                                    ║
║  [WARNING] TBL-04: Date range filter not synced to URL.           ║
║    Refreshing browser loses date filter selection.                ║
║    Fix: Serialize date range to URL search params.                ║
║                                                                    ║
║  [WARNING] TBL-05: Stats card grid has no empty state.            ║
║    Shows blank area when no stats data available.                 ║
║    Fix: Add empty state with appropriate message.                 ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  TEST CASES ([N] total)                                            ║
║    Pagination: [N] | Sorting: [N] | Filtering: [N]               ║
║    Empty: [N] | Row Action: [N]                                   ║
║    [See full list in test cases file]                              ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  RECOMMENDATIONS                                                   ║
║  1. [Critical] Add max pageSize validation on backend DTO          ║
║  2. [Critical] Add confirmation dialog for bulk delete             ║
║  3. [Warning] Reset page to 1 on date filter change               ║
║  4. [Warning] Sync date range filter to URL params                 ║
║  5. [Warning] Add empty state to stats card grid                   ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Test Case Categories

### Category 1: Pagination Tests (PAGE-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| PAGE-01 | Navigate to page 2 | 1. Load table 2. Click page 2 | Page 2 data loaded; URL updated to ?page=2 |
| PAGE-02 | Change page size | 1. Select 50 items/page | Table shows 50 rows; page resets to 1 |
| PAGE-03 | Refresh preserves page | 1. Go to page 3 2. Refresh browser | Still on page 3 with same data |
| PAGE-04 | Browser back preserves page | 1. Go to page 3 2. Click row → detail 3. Browser back | Returns to page 3 |
| PAGE-05 | Last page boundary | 1. Navigate to last page | Correct number of rows; no "next" available |
| PAGE-06 | Invalid page in URL | 1. Manually set ?page=9999 | Redirects to last valid page or shows empty with message |
| PAGE-07 | Page size limit | 1. Try ?pageSize=999999 | Backend rejects or caps at max allowed |

### Category 2: Sorting Tests (SORT-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| SORT-01 | Click column header | 1. Click "Name" column | Sorted ascending; arrow indicator shown |
| SORT-02 | Toggle sort direction | 1. Click "Name" again | Sorted descending; arrow reverses |
| SORT-03 | Sort resets page | 1. Go to page 3 2. Click sort | Returns to page 1 with sorted data |
| SORT-04 | Sort persists on refresh | 1. Sort by name 2. Refresh | Same sort applied (if URL synced) |
| SORT-05 | Default sort on load | 1. Load page fresh | Table has default sort applied |

### Category 3: Filtering Tests (FILTER-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| FILTER-01 | Text search with debounce | 1. Type "john" quickly | Only 1 API call after typing stops (debounced) |
| FILTER-02 | Dropdown filter | 1. Select "Active" status | Table filters to active items; page resets to 1 |
| FILTER-03 | Clear all filters | 1. Apply multiple filters 2. Click "Clear" | All filters removed; table shows all data; URL cleaned |
| FILTER-04 | Filter + sort combination | 1. Filter by status 2. Sort by name | Both applied correctly; data is filtered AND sorted |
| FILTER-05 | Filter persists on refresh | 1. Apply filter 2. Refresh | Filter still applied (if URL synced) |
| FILTER-06 | No results for filter | 1. Apply filter that matches nothing | "No results" empty state with "Clear filters" button |

### Category 4: Row Action Tests (ROW-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| ROW-01 | Row click navigation | 1. Click a row | Navigates to detail page |
| ROW-02 | Select single row | 1. Click checkbox | Row highlighted; bulk action bar appears |
| ROW-03 | Select all on page | 1. Click header checkbox | All visible rows selected |
| ROW-04 | Bulk delete | 1. Select 3 rows 2. Click "Delete" | Confirmation: "Delete 3 items?" |
| ROW-05 | Delete single row | 1. Click delete icon on row | Confirmation: "Delete [item name]?" |
| ROW-06 | Selection across pages | 1. Select rows on page 1 2. Go to page 2 3. Return to page 1 | Page 1 selections preserved |

### Category 5: Empty/Loading Tests (TABLE-STATE-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| TABLE-STATE-01 | Initial loading | 1. Load page (clear cache) | Table skeleton or spinner, not empty table |
| TABLE-STATE-02 | No data (empty table) | 1. Load page with 0 items | "No items yet" + CTA button |
| TABLE-STATE-03 | Loading on page change | 1. Click page 2 | Subtle loading (overlay or row skeleton), previous data may stay |
| TABLE-STATE-04 | Error state | 1. Mock API error | Error message with retry button |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Pagination Correctness | 20% | Type detected, boundary cases handled, URL sync, backend alignment |
| Sorting Implementation | 15% | Sort state, default sort, direction toggle, page reset, UI indicator |
| Filtering Implementation | 15% | Filter types, debounce, URL sync, filter reset, page reset on filter |
| Empty State Coverage | 15% | No-data, no-results, loading, error states all present |
| Row Action Safety | 10% | Click handlers, selection, bulk actions, delete confirmation |
| Column Rendering Quality | 10% | Truncation, responsive, custom renderers, alignment, date format |
| Data Freshness | 15% | Refetch strategy, optimistic updates, stale data handling |

**Scoring rules** (see `qa-shared-reference.md` for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Each fully-covered empty state: +2
  - Each correctly URL-synced param (page, sort, filter): +1
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
| No table/list found | No data display component in file | Warning shown; component may not display tabular data |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| Table library not detected | Custom implementation | Analyzes native `<table>` or `.map()` patterns |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| Column definitions unresolvable | Dynamic columns or complex generator | Notes as "dynamic columns — manual review needed" |
| Pagination in separate component | `<Pagination>` imported from child | Follows import to analyze pagination component |

---

## Edge Cases Handled

- **Server-side vs client-side mismatch** — Frontend sorts locally but backend returns paginated subset; flagged as CRITICAL
- **Stale closure in callbacks** — Pagination/sort callbacks capture old state in React closures
- **Race conditions on rapid page change** — Previous page request returns after next page; check `keepPreviousData`
- **Virtual scroll + dynamic row heights** — Virtual list libraries with variable height rows; performance note
- **Table inside tabs** — Tab switch remounts table, losing pagination/sort/filter state
- **Bulk select across pages** — Selection state persistence when changing pages
- **Column order/visibility persistence** — User preferences saved to localStorage
- **Responsive table on mobile** — Horizontal scroll vs card layout vs hidden columns
- **Expandable rows** — Rows with expand/collapse for detail content
- **Nested tables** — Table inside expandable row; separate analysis needed
- **Dynamic columns** — Columns generated from API response or user configuration
- **Export with filters** — Export button should export filtered/sorted data, not all data
- **Empty table vs no table** — Component conditionally renders table only when data exists; empty state shown outside table
- **Infinite scroll with filters** — Changing filter should reset scroll position and accumulated data
- **Virtualized lists** — `@tanstack/react-virtual`, `react-window`, `react-virtuoso` detected; scroll restoration and item measurement checked
- **Drag and drop reordering** — `@dnd-kit`, `react-beautiful-dnd` detected; server sync after reorder and undo mechanism checked
- **Column resize/reorder persistence** — User column preferences saved to localStorage or user settings API
- **Large dataset performance** — Tables with 1000+ rows without virtualization flagged as WARNING
- **CSV/Excel export analysis** — Export scope (current page vs all), filter/sort respect, server-side generation for large data
- **Real-time data updates** — WebSocket-driven row changes; selection state, scroll position, pagination count impact
- **Grouped/nested data** — `groupBy` functionality with expand/collapse headers; different from expandable rows
- **Table state URL sharing** — When user shares URL with page/sort/filter state, recipient sees same view
- **Print-friendly table layout** — `@media print` styles for tables; noted as INFO if missing

---

## Related Commands

- `/qa-input-fields` — QA input fields (filter inputs on table pages)
- `/qa-back-navigation` — QA back navigation (back from detail preserves table state)
- `/qa-loading-error-empty` — QA loading/error/empty states (tables need all three)
- `/qa-modal-drawer` — QA modals (create/edit modals triggered from table actions)
- `/qa-permission-role` — QA permissions (table row actions may be role-gated)
- `/review-command` — Validate this command's structure and quality

---

## Tips

1. **Always check page reset on filter/sort** — The #1 table bug: user is on page 5, applies filter, stays on page 5 but only 2 pages of results exist
2. **URL sync is crucial for tables** — Users share table URLs, bookmark filtered views, and use browser back/forward
3. **Debounce text filters** — Without debounce, every keystroke triggers an API call; 300ms is a good default
4. **Test boundary pages** — Page 0, last page, page exceeding total, negative page in URL
5. **Backend page size guard is security** — Without a max `pageSize`, attackers can fetch your entire database in one request
6. **Empty states differ by context** — "No users yet" (fresh state) vs "No users match your search" (filtered) require different UI
7. **Combine with `/qa-loading-error-empty`** — Tables are the #1 component needing comprehensive state coverage
8. **Re-run after adding columns or filters** — New sortable/filterable columns need URL sync and page reset checks
