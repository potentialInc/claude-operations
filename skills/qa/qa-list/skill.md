---
name: qa-list
description: Audit all list/table pages for UX consistency - sorting, search, pagination, filters, loading states, empty states, full-stack tracing, data freshness, inline edit, and export
user-invocable: true
argument-hint: "[page-path or module] [--backend <path>] [--depth full|shallow]"
---

# QA List - List/Table UX Consistency Auditor

## Purpose

Verify every list/table page follows consistent UX patterns: default sorting (newest first), working search, sortable columns, resizable columns, proper loading/empty states, filter functionality, URL state synchronization, full-stack API tracing, data freshness strategies, inline edit handling, and export functionality.

## Usage

```
/qa-list                           # Full audit of all list pages
/qa-list src/pages/UserList.tsx    # Single page audit
/qa-list orders --backend backend/src  # With backend tracing
/qa-list products --depth shallow  # Frontend-only analysis
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `[page-path or module]` | No | Path to page/component or module name. If omitted, audits all list pages |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): trace through API and backend. `shallow`: frontend-only analysis |

## 22 Checks

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
| 11 | Missing result count | **Medium** | No "showing X of Y" count display — user can't tell how many items are filtered vs total |
| 12 | Multi-line toolbar | **Medium** | Search, tabs, and filters on separate lines — wastes vertical space, prefer single-line toolbar |
| 13 | Unfillable table column | **High** | Table displays a data field that cannot be entered in any create/edit form — column is always empty |
| 14 | List-detail status mismatch | **High** | Status/badge in list row uses different logic (14a) or different component/styling (14b) than the detail page |
| 15 | Filter component type mismatch | **High** | Status/category filter uses Select dropdown instead of inline button group — inconsistent with other list pages |
| 16 | Filter state not in URL | **High** | Filter/sub-tab state stored in local state only — lost on browser back navigation, should use URL searchParams |
| 17 | Missing full-stack tracing | **High** | Frontend pagination/sort/filter params don't match backend query API |
| 18 | URL state not synced | **Medium** | Pagination/sort/filter state lost on page refresh (not in URL params) |
| 19 | Missing data freshness strategy | **Medium** | No refetch/revalidation after mutations, stale data shown |
| 20 | Inline edit issues | **Medium** | Inline edit mode lacks save/cancel, dirty check, or optimistic update |
| 21 | Missing export functionality | **Low** | List page with >50 items has no CSV/Excel export option |
| 22 | Expanded row index vs entity field | **High** | Expandable/grouped sub-table uses `index + 1` for row numbering instead of the entity's actual field (e.g., `order`, `sequence`) — numbers don't match real data |

## Execution Algorithm

You MUST follow these steps in order. Do NOT skip steps. Do NOT modify any files. This is a READ-ONLY diagnostic.

**Prerequisites:** Read `qa-shared/reference.md` for framework detection tables, pattern mappings, and fallback strategies.

### Step 0: Context Gathering (MANDATORY)

**0a. Read project structure:**
```
Read: CLAUDE.md (project root)
Read: package.json (project root and any workspace packages)
Glob: **/requirements/**/*.md
```

Extract:
- Which frontend apps exist (auto-detect by scanning for `package.json` with framework dependencies)
- Which pages display list/table data
- Known list patterns and hooks used

**0b. Detect frontend framework:**

Use the **Frontend Framework Detection** table from `qa-shared/reference.md`. Detect React, Vue, Angular, Svelte, SvelteKit, or Plain HTML. Record the detected framework — all subsequent steps adapt patterns accordingly.

**0c. Detect table library:**

| Signal | Library |
|--------|---------|
| `@tanstack/react-table` or `@tanstack/vue-table` in package.json | TanStack Table |
| `antd` + `<Table>` usage | Ant Design Table |
| `@mui/x-data-grid` | MUI DataGrid |
| `ag-grid-react` / `ag-grid-vue` / `ag-grid-angular` | AG Grid |
| `@shadcn/ui` + DataTable pattern | shadcn DataTable |
| `element-plus` + `<el-table>` | Element Plus Table |
| `vuetify` + `<v-data-table>` | Vuetify DataTable |
| `@angular/material` + `mat-table` | Angular Material Table |
| `primereact` / `primevue` / `primeng` + DataTable | PrimeNG/PrimeReact/PrimeVue |
| No library — raw `<table>` + `.map()` | Custom/Native Table |

**0d. Detect backend framework** (if `--depth full` or default):

If `--backend` provided, use that path. Otherwise, look for sibling `backend/` or `server/` directory.

| Signal | Framework |
|--------|-----------|
| `@nestjs/common` in package.json | NestJS |
| `express` in package.json (no nest) | Express |
| `fastify` in package.json | Fastify |
| `manage.py` in root | Django |
| `pom.xml` or `build.gradle` with spring-boot | Spring Boot |
| `composer.json` with laravel | Laravel |

**0e. Detect ORM/query builder** (if backend detected):

| Signal | ORM |
|--------|-----|
| `typeorm` in package.json | TypeORM |
| `@prisma/client` in package.json | Prisma |
| `sequelize` in package.json | Sequelize |
| `drizzle-orm` in package.json | Drizzle |
| `django.db.models` imports | Django ORM |
| `spring-boot-starter-data-jpa` | JPA/Hibernate |
| `illuminate/database` (Laravel) | Eloquent |

**0f. Identify the sorting hook/pattern:**

Framework-specific detection:
- **React (custom):** `Glob: **/hooks/use-table-sort*` or `**/hooks/use-sort*`
- **React (TanStack):** `getSortedRowModel()` or `manualSorting` in table config
- **React (Ant Design):** `sorter` property in column definitions
- **React (MUI DataGrid):** `sortModel` / `onSortModelChange` props
- **Vue (Element Plus):** `@sort-change` event on `<el-table>`
- **Vue (Vuetify):** `v-model:sort-by` on `<v-data-table>`
- **Angular (Material):** `matSort` directive + `(matSortChange)` event

Read the hook/pattern to understand:
- Function signature (what params it accepts)
- Whether it supports a default column parameter
- Whether it supports a default direction parameter
- What it returns (sortedData, sortState, handleSort, etc.)

**0g. Identify the column resize pattern:**

Framework-specific detection:
- **Custom hook:** `Glob: **/hooks/use-column-resize*`
- **TanStack Table:** `enableResizing` in column config
- **AG Grid:** Built-in column resize (enabled by default)
- **MUI DataGrid:** `resizable` column property
- **Ant Design:** `<Resizable>` wrapper on column headers

Read the pattern to understand its usage.

---

### Step 1: List Page Inventory

Scan all frontend apps for pages/components that render lists or tables.

**Search for table/list patterns (framework-aware):**

**React:**
```
Grep: "useReactTable|useTableSort|<Table|<DataGrid|<AgGridReact|DataTable" in frontend apps
Grep: "SortableHeader|sortable|getSortedRowModel" in frontend apps
Grep: "\.map\(.*=>" in pages/ directories (potential list renders)
```

**Vue:**
```
Grep: "<el-table|<v-data-table|<DataTable" in frontend apps
Grep: "sortable|@sort-change|v-model:sort-by" in frontend apps
```

**Angular:**
```
Grep: "mat-table|matSort|matColumnDef" in frontend apps
Grep: "DataSource|MatTableDataSource" in frontend apps
```

**Page classification criteria — include ONLY pages whose PRIMARY purpose is displaying a data list/table:**
- Include: Pages with table components, sort hooks, sortable headers, or data grid as their main content
- Include: Reusable table components (e.g., user-table, product-table)
- Exclude: Detail pages that happen to have a small inline table
- Exclude: Multi-panel layout pages — audit the individual panel components instead
- Exclude: Non-tabular list views (e.g., card grids, timeline views, chat message lists)
- Include with note: Pages using custom sort/group logic instead of standard sort hooks

Also search for list pages that may NOT use standard sort patterns:
```
Grep: "\.map\(.*=>" combined with data-fetching hooks (useQuery, useSWR, useFetch) in pages/
```
These pages may use custom sorting or no sorting at all — flag them for review.

For each match, record:
- **File path**
- **Page/component name**
- **Data source** (query key, API endpoint)
- **Hooks/patterns used** (sort hook, table library features, custom sort)
- **Page type**: management (users, orders) | library (products, categories) | history (logs, records)
- **Page purpose**: creation (has create/add button) | reference (lookup/selection only) | mixed

Build a master list:
```
List Pages Found:
  1. pages/users.tsx        — TanStack Table, sort ✅, resize ✅, type: management
  2. pages/products.tsx     — Ant Design Table, sort ✅, resize ❌, type: library
  3. pages/orders.tsx       — Custom table, sort ❌, resize ❌, type: management
  ...
```

If a specific page/module argument was provided, filter to only matching pages.

---

### Step 2: Per-Page Deep Analysis

For EACH page found in Step 1, read the full file and analyze the following:

**A) Sort Implementation:**

Framework-specific analysis:

| Framework | What to check |
|-----------|--------------|
| React (custom hook) | Is sort hook called? defaultColumn param? defaultDirection? Does template iterate sorted array? |
| React (TanStack) | `initialSorting` in table options? `getSortedRowModel()` present? `manualSorting` for server-side? |
| React (Ant Design) | `defaultSortOrder` on column? `sorter` function defined? `onChange` handler processes sort? |
| React (MUI DataGrid) | `initialState.sorting` defined? `sortModel` state managed? `sortingMode="server"` for server-side? |
| Vue (Element Plus) | `default-sort` prop on `<el-table>`? `sortable` on columns? `@sort-change` handler? |
| Vue (Vuetify) | `v-model:sort-by` bound? Default sort array populated? |
| Angular (Material) | `matSortActive` + `matSortDirection` set? `(matSortChange)` handler defined? |

Record:
- Is sorting configured? (yes/no)
- What is the default sort column? (value or undefined)
- What is the default sort direction? (value or omitted)
- Does the template iterate over sorted data or the original unsorted array?

**B) Sort Header/Column Props:**

For each sortable column header:
- Does it have a sort direction indicator?
- Does it have a click handler for sorting?
- Does it have explicit `sortable={false}` or equivalent? (intentionally non-sortable)

Record columns that appear sortable but are MISSING direction indicator or click handler.

**C) Search Implementation:**
```
Search for: <Input.*search|<Search|searchTerm|searchQuery|inputValue
Search for: .filter(.*search|.includes(.*search|toLowerCase.*includes
```
Record:
- Search input exists? (yes/no)
- Search is connected to filter logic? (yes/no)
- Debounced? (yes/no) — detect by checking:
  - Debounced: `setTimeout` + `clearTimeout` pattern, `useDebouncedValue`, `lodash.debounce`, `useDebounce`
  - Debounced: Local `inputValue` state that syncs to filter state via `useEffect` with `setTimeout`
  - Not debounced: `onChange={(e) => setSearchTerm(e.target.value)}` directly updating filter state

**D) Column Resize:**

Framework-specific detection:
- **Custom hook:** `useColumnResize` called? containerRef attached? resizable props on headers?
- **TanStack Table:** `enableResizing: true` in column config?
- **AG Grid:** Column resize enabled by default; check `suppressResize` for disabled columns
- **MUI DataGrid:** `resizable` property in column definitions?

Record:
- Resize support present? (yes/no)
- Partial: some columns resizable, others not? (inconsistency)

**E) Loading State:**
```
Search for: isLoading|isFetching|isPending|loading
Search for: skeleton|Skeleton|animate-pulse|spinner|Spinner|LoadingOverlay
```
Record:
- Data fetching provides loading state? (yes/no)
- Loading UI rendered when loading=true? (yes/no)
- Type of loading UI (skeleton, spinner, overlay, text)

**F) Empty State:**
```
Search for: .length === 0|noData|emptyState|no results|no records
```
Record:
- Empty state checked? (yes/no)
- Empty state UI rendered? (yes/no)
- Message is user-friendly and translated? (yes/no)

**G) Result Count Display:**
```
Search for: showingCount|totalCount|of.*total|건.*중|명.*중
Search for: filteredData.length|sortedData.length combined with data.length|total
```
Record:
- Count display exists? (yes/no)
- Shows filtered count vs total? (e.g., "Showing 5 of 20")
- Translated via i18n? (yes/no)
- Positioned above or below table? (location)

**H) Table-to-Form Field Mapping:**
For each table column, record the entity field it displays:
```
Search for: entity field references inside .map() row render or column cell renderers
```
Then find the create/edit forms for the same module:
```
Glob: **/create-*-modal*, **/edit-*-modal*, **/create-*-form*, **/edit-*-form*
Glob: **/*-create*, **/*-edit*, **/*-form*
```
Extract form schema fields and build a mapping:
- Table field → In create form? → In edit form?
Record any fields that appear in the table but NOT in any form.

**I) Filter Analysis:**
```
Search for: filter|Filter|filterState|activeFilter|selectedFilter
Search for: reset|clear|clearFilter
```
Record:
- Filter UI exists? (yes/no)
- Filter types (dropdown, checkbox group, toggle buttons, etc.)
- Multi-select supported? (yes/no — check if state is array/Set vs single value)
- Reset/clear button exists? (yes/no)
- Filter state persisted? (URL params, localStorage, etc.)

---

### Step 3: Check 1 — Missing Default Sort (HIGH)

For each page from Step 2:

**Pass condition:**
- Sort is configured with a non-undefined default column parameter
- OR the API endpoint itself guarantees ORDER BY (verify in backend)
- OR the page explicitly states "no sorting needed" (rare — e.g., real-time dashboards)

**Fail condition:**
- Sort configured without a default column
- OR sort not used at all, and data rendered in API response order
- OR default column is set but default direction is not 'desc' for time-based columns

**Classification by page purpose:**

A) **Creation pages** (have create/add/new button, or new items are generated by user actions):
   - Expected default: `createdAt DESC` or `updatedAt DESC` (newest first)
   - Missing default sort → **HIGH** (clear bug — users expect newest first)

B) **Reference/lookup pages** (list existing data for selection, no create button):
   - Expected default: must be defined in PRD/DB schema (e.g., number ASC, name ASC)
   - If PRD specifies order → verify implementation matches PRD. Mismatch → **HIGH**
   - If PRD does NOT specify order → **generate a question** in the "Questions for Clarification" section
   - Missing default sort → **MEDIUM** (PRD gap, not necessarily a code bug)

C) **Mixed pages** (can both create and browse/select existing data):
   - Default to creation page rules (`createdAt DESC`)
   - But **generate a question** if the reference use case has different sort expectations

---

### Step 4: Check 2 — Sort UI Disconnected (HIGH)

For each sortable column header found:

**Pass condition:**
- Has sort direction prop connected to sort state
- Has click handler connected to sort function
- OR has explicit `sortable={false}` or equivalent

**Fail condition:**
- Sortable header component but NO sort direction prop → column header looks sortable but does nothing
- Has click handler but NO sort direction → clicking sorts but no visual indicator
- Has sort direction but NO click handler → shows indicator but clicking does nothing

---

### Step 5: Check 3 — Missing Search (MEDIUM)

For each page:

**Pass condition:**
- Search input exists and is wired to filter logic
- OR the list is small enough that search is unnecessary (<20 fixed items)

**Fail condition:**
- No search input on a page that lists user-generated or growing data
- Search input exists but is not connected to any filtering (decorative only)

---

### Step 5.1: Check 4 — Missing Search Debounce (HIGH)

For each page that HAS a search input:

**Pass condition:**
- Search input uses debounce (setTimeout, useDebouncedValue, lodash.debounce) with 300ms+ delay
- OR search is triggered on explicit submit (button click / Enter key), not on every keystroke

**Fail condition:**
- Search input directly updates filter state on every `onChange` event without debounce
- This causes Korean/Chinese/Japanese IME composition to break — characters are submitted mid-composition

**How to detect:**
```
Search for: setTimeout.*search|debounce.*search|useDebouncedValue|useDebounce
Search for: onChange.*setSearchTerm|onChange.*setInputValue (direct state update = no debounce)
```

If the page uses a local input state that syncs to filter/URL via setTimeout, that IS debounced.
If the page directly sets the filter state in onChange without any delay, that is NOT debounced.

---

### Step 6: Check 5 — Missing Column Resize (MEDIUM)

For each table/grid page:

**Pass condition:**
- Column resize support is enabled (custom hook, library feature, or built-in)
- Resize handles visible on column headers

**Fail condition:**
- No column resize support → columns are fixed width
- Partial: some columns are resizable but others are not (inconsistency)

---

### Step 7: Check 6 — Missing Loading State (HIGH)

**Pass condition:**
- Loading state from data-fetching hook is checked
- A visible loading indicator is rendered (skeleton, spinner, overlay)

**Fail condition:**
- No loading check → content area is blank or shows "no data" flash before data arrives
- Loading variable exists but is not used in rendering

---

### Step 8: Check 7 — Missing Empty State (MEDIUM)

**Pass condition:**
- After loading completes, checks if data array is empty
- Renders a user-friendly empty state message (translated if i18n project)

**Fail condition:**
- No empty state check → renders an empty table with just headers
- Empty state shows untranslated text in an i18n project

---

### Step 9: Check 8 — Missing Filter Reset (MEDIUM)

For pages with filters:

**Pass condition:**
- Reset/clear button exists
- Clicking it restores all filters to default state
- URL params are also cleared if filters use URL state

**Fail condition:**
- Filters exist but no way to reset them except page reload
- Reset button exists but only clears some filters (e.g., clears text but not dropdown)

---

### Step 10: Check 9 — Single-Select Filter Only (LOW)

For library/catalog pages where multi-select makes sense:

**Pass condition:**
- Filter state uses array or Set, allowing multiple selections
- UI reflects multi-select (checkboxes, multi-select dropdown)

**Fail condition:**
- Filter that logically should support multi-select (e.g., category, type, tag) only allows one value
- State is a single value instead of array

---

### Step 11: Check 10 — Sorted Data Not Used (HIGH)

For each page using a sort hook or sort configuration:

**Pass condition:**
- The template/render iterates over the sorted data output

**Fail condition:**
- Sort is configured but template still maps over original unsorted data
- This means sorting appears to work (column headers toggle) but list order never changes

---

### Step 12: Check 12 — Multi-line Toolbar (MEDIUM)

For each list page:

**Pass condition:**
- Search, tabs/navigation, and filters are all on a SINGLE horizontal line
- Uses flexbox layout so items flow in one row
- Graceful wrapping ONLY on narrow viewports

**Fail condition:**
- Search bar on one line, tabs on another line, filters on yet another line
- Each section stacked vertically, wasting 2-3 rows of vertical space

**How to detect:**
```
Search for: Separate container blocks wrapping search bar and filter controls
Check: Are search, tabs, and filter controls in the SAME parent flex container?
```

**Why this matters:**
- Data-heavy dashboards need every pixel of vertical space for table rows
- Single-line toolbar reduces toolbar from 3 rows (~120px) to 1 row (~40px) = ~80px more table content visible

---

### Step 13: Check 11 — Missing Result Count (MEDIUM)

For each list page:

**Pass condition:**
- A text element displays both the filtered/visible count AND total count (e.g., "Showing 5 of 20")
- Count updates reactively when filters/search change

**Fail condition:**
- No count display at all — user has no idea how many items exist or how many are filtered out
- Only shows total count without filtered count

**Why this matters:**
- Users need to know if their search/filter is narrowing results or showing everything
- Helps users discover that filters are active when they see "3 of 50" instead of just a short list

---

### Step 14: Check 13 — Unfillable Table Column (HIGH)

For each table/list page that has a corresponding create/edit form:

**A) Extract table-rendered fields:**
Inside the row render, extract all entity field references:
- Direct: `item.notes`, `user.phone`
- Nested: `item.type?.name` (relation fields — skip these, they're set via FK select)
- Conditional: `item.notes || "-"`, `item.notes ?? "N/A"`

Record the list of fields rendered in each table column.

**B) Find create/edit forms for the same module:**
```
Glob: **/create-*-modal*, **/edit-*-modal*, **/create-*-form*, **/edit-*-form*
Glob: **/*-create*, **/*-edit*, **/*-form*
```
Extract form field names from:
- Zod/Yup/Joi schema (`.object({ fieldName: ... })`)
- Form hook fields (`useForm`, `register('fieldName')`)
- Input name attributes (`<input name="...">`)

Build a set of all fillable fields across ALL create AND edit forms for this module.

**C) Cross-reference:**
For each field rendered in the table:
1. Check if it exists in ANY create or edit form for the same module
2. If field is in NEITHER create NOR edit form → **FAIL**

**Exclude from check (auto-generated/system fields):**
- `id`, `uuid`
- `createdAt`, `updatedAt`, `deletedAt`
- `createdBy`, `updatedBy`
- Relation display fields (e.g., `type.name`) — these are set via FK select
- Computed/derived fields (e.g., `fullName` from `firstName` + `lastName`)
- Status fields changed via dedicated actions (not form fields)

**Fail sub-categories:**
- **13a - Never fillable**: Field exists but no form includes it → column is always empty → **HIGH**
- **13b - Only on create**: Field in create form but not edit form → cannot be corrected after creation → **INFO**
- **13c - Only on edit**: Field in edit form but not create form → empty until first edit → **INFO**

---

### Step 15: Check 14 — List-Detail Status Mismatch (HIGH)

For each list page that has a corresponding detail page:

**A) Identify list-detail pairs:**
```
Search for: navigate(.*detail|navigate(.*/${|useNavigate|<Link.*to=|router.push
```
Find which list pages link to detail pages. Build pairs.

**B) Extract status display logic from BOTH pages:**

For the **list page**, find the status column render logic inside the row render.
For the **detail page**, find the status display logic.

**C) Compare the two logic chains (14a — Logic Mismatch):**

**Pass condition:**
- Both pages use the SAME conditional hierarchy to determine status display
- Both check the same fields in the same priority order

**Fail condition:**
- Logic divergence: list checks field A first, detail checks field B first
- One page has more status states than the other

**D) Compare the rendering component/styling (14b — Component/Style Mismatch):**

**Pass condition:**
- Both pages use the SAME component with the SAME variant for each status value

**Fail condition:**
- List uses shared Badge component but detail uses inline styled span for the same status
- Same status value rendered with visually different colors/styles across pages

---

### Step 16: Check 15 — Filter Component Type Mismatch (HIGH)

For each list page with status/category filter controls:

**A) Identify filter UI component type** across all list pages.

**B) Compare across all same-type pages:**
Build a comparison table of which component type each page uses.

**Pass condition:**
- All same-type list pages use the SAME filter UI component
- Standard pattern: Inline button group for filters with ≤5 options

**Fail condition:**
- Page A uses dropdown for status filter, Page B uses inline button group for the same kind of filter
- Mixed component types within the same page type group

**When dropdown IS acceptable:**
- 6+ filter options (too many buttons would overflow)
- Dynamic/user-generated option lists (unpredictable count)
- Filter needs search/typeahead within options

---

### Step 17: Check 16 — Filter State Not in URL (HIGH)

For each list page with filter/sub-tab controls:

**A) Identify filter state variables (framework-aware):**

| Framework | Local State Pattern | URL State Pattern |
|-----------|-------------------|-------------------|
| React | `useState(filter)`, `useReducer` | `useSearchParams()`, `new URLSearchParams` |
| Next.js | `useState(filter)` | `useSearchParams()`, `router.query` |
| Vue | `ref(filter)`, `reactive({ filter })` | `useRoute().query`, `useRouter().push({ query })` |
| Nuxt | `ref(filter)`, `useState()` | `useRoute().query`, `navigateTo({ query })` |
| Angular | Component property, `BehaviorSubject` | `ActivatedRoute.queryParams`, `Router.navigate({ queryParams })` |
| Svelte | `let filter = ...`, `writable()` | `$page.url.searchParams`, `goto('?filter=x')` |

Record how each filter state is stored:
- Local component state only → **volatile** — lost on navigation
- URL params (searchParams, router query) → **persistent** — survives back/forward navigation
- localStorage / sessionStorage → **persistent** but not shareable via URL

**Pass condition:**
- All user-facing filter/sub-tab states are stored in URL searchParams
- Browser back button restores the exact filter state
- Default values don't clutter the URL

**Fail condition:**
- Filter state uses local state only — resets to default on back navigation
- User loses their filter context after viewing a detail page and returning

---

### Step 18: Check 17 — Missing Full-Stack Tracing (HIGH)

For each list page with server-side pagination, sorting, or filtering (skip if `--depth shallow`):

**A) Trace the frontend API call:**
Identify the data-fetching call and extract all query parameters sent to the backend:
- Pagination: `page`, `pageSize`, `limit`, `offset`, `cursor`
- Sorting: `sortBy`, `sortOrder`, `orderBy`, `order`
- Filtering: `search`, `status`, `category`, custom filter params

**B) Trace the backend handler:**

Follow the API endpoint to the controller/handler:

| Backend | Where to look |
|---------|--------------|
| NestJS | `@Get()` decorator + `@Query()` params in controller |
| Express | `router.get()` + `req.query` in handler |
| Fastify | Route handler + `request.query` |
| Django | `ViewSet` + `queryset` filter/order |
| Spring Boot | `@GetMapping` + `@RequestParam` in controller |
| Laravel | Controller method + `$request->query()` |

**C) Trace to the database query:**

| ORM | What to check |
|-----|--------------|
| TypeORM | `findAndCount({ skip, take, order, where })` — are all params passed? |
| Prisma | `findMany({ skip, take, orderBy, where })` — are all params passed? |
| Sequelize | `findAndCountAll({ offset, limit, order, where })` — are all params passed? |
| Drizzle | `select().from().where().orderBy().limit().offset()` chain |
| Django ORM | `queryset.filter().order_by()[offset:limit]` |
| JPA/Hibernate | `Pageable` + `Sort` + `Specification` |
| Eloquent | `->where()->orderBy()->paginate()` |
| Supabase | `supabase.from('table').select('*', { count: 'exact' }).order().range(from, to).ilike()` |

**D) GraphQL / Supabase API patterns:**

If the project uses GraphQL or Supabase instead of REST:

| Pattern | What to check |
|---------|--------------|
| GraphQL (Apollo/urql) | `useQuery(GET_ITEMS, { variables: { page, sort, filter } })` — verify variables match schema arguments |
| GraphQL pagination | `first/after` (cursor) or `limit/offset` in query variables — verify backend resolver applies them |
| Supabase client | `supabase.from('table').select('*', { count: 'exact' })` — verify `.order()`, `.range()`, `.ilike()`/`.eq()` are applied |
| Supabase RPC | `supabase.rpc('function_name', { p_page, p_sort })` — verify function accepts and uses pagination params |

**Pass condition:**
- All frontend params (page, sort column, sort direction, filter values) are received by backend
- Backend translates them correctly to database query
- Backend returns total count for pagination
- Backend validates/sanitizes sort column names (prevents SQL injection via raw column name)
- Backend enforces max page size (prevents `pageSize=999999` DoS)

**Fail condition:**
- Frontend sends `sortBy=name` but backend ignores it — always returns default order
- Frontend sends filter params but backend query has no WHERE clause for them
- Backend accepts arbitrary sort column names without validation
- No max page size guard on backend

---

### Step 19: Check 18 — URL State Not Synced (MEDIUM)

For each list page with pagination, sorting, or filtering:

**A) Check which states are in URL:**

| State | URL param example | Detection |
|-------|-------------------|-----------|
| Current page | `?page=2` | `searchParams.get('page')` or router query |
| Page size | `?pageSize=20` | `searchParams.get('pageSize')` or router query |
| Sort column | `?sortBy=name` | `searchParams.get('sortBy')` or router query |
| Sort direction | `?sortOrder=asc` | `searchParams.get('sortOrder')` or router query |
| Search text | `?search=john` | `searchParams.get('search')` or router query |
| Filter values | `?status=active` | `searchParams.get('status')` or router query |

**Pass condition:**
- All interactive state (page, sort, filter, search) reflected in URL params
- Refreshing the browser preserves the exact view state
- Sharing the URL shows the same filtered/sorted/paged view

**Fail condition:**
- Pagination state in local state only — refresh goes back to page 1
- Sort state in local state only — refresh loses sort order
- Any interactive state not persisted in URL

**Combined state interactions to verify:**

| Interaction | Expected Behavior | Common Bug |
|-------------|-------------------|------------|
| Sort change → page | Page resets to 1 | Stays on current page (shows wrong data subset) |
| Filter change → page | Page resets to 1 | Stays on page 3 but only 2 pages of results |
| Filter change → sort | Sort preserved (or reset to default) | Sort lost, data appears unordered |
| Clear filters → page/sort | Page resets, sort optionally preserved | Page stays, stale URL params |

---

### Step 20: Check 19 — Missing Data Freshness Strategy (MEDIUM)

For each list page:

**A) Identify the data fetching pattern:**

| Pattern | Detection |
|---------|-----------|
| TanStack Query | `useQuery({ queryKey, queryFn })` |
| SWR | `useSWR(key, fetcher)` |
| RTK Query | `useGetXxxQuery()` |
| Nuxt | `useFetch()` or `useAsyncData()` |
| Angular | `HttpClient.get()` in service |
| Manual | `useEffect(() => { fetch(url) }, [deps])` |

**B) Check refetch strategies:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Window focus refetch | Data refreshes when user returns to tab | Enabled / Disabled |
| Polling interval | Periodic refetch for frequently-changing data | Set (ms) / None |
| Manual refetch | Refresh button available to user | Present / Missing |
| After mutation refetch | Data refetched after create/update/delete | Cache invalidation / manual refetch / Missing |
| Stale time | Configured to prevent unnecessary refetches | Configured / Default |

**C) Check optimistic update handling:**

For mutations that affect table data (create, update, delete):

| Check | Description | Assessment |
|-------|-------------|------------|
| Optimistic update | UI updates before API responds | Used / Not used |
| Rollback on error | UI reverts if API fails | Present / Missing |
| Cache invalidation | Table data refreshed after successful mutation | Present / Missing |

**Pass condition:**
- After a create/update/delete mutation, the list data is refetched or optimistically updated
- User sees current data, not stale data from before the mutation
- Cache invalidation strategy is consistent across the page

**Fail condition:**
- User creates a new item but the list doesn't show it until manual page refresh
- User deletes an item but it still appears in the list
- No cache invalidation or refetch after mutations

---

### Step 21: Check 20 — Inline Edit Issues (MEDIUM)

For each table with inline editing capability:

**A) Detect inline edit patterns:**

| Library | Detection |
|---------|-----------|
| TanStack Table | `meta.updateData` or custom `editableCell` renderer |
| AG Grid | `editable: true` in column def, `onCellValueChanged` |
| Ant Design | `<EditableCell>` wrapper, `editable` column property |
| MUI DataGrid | `editable: true` in column, `processRowUpdate` |
| Element Plus | `<el-input>` inside `<el-table-column>` template |
| Custom | `<input>` or `contentEditable` inside cell renderer |

**B) Check inline edit behavior:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Edit trigger | How editing starts (click, double-click, button) | Documented / Unclear |
| Save mechanism | How changes are saved (blur, Enter key, save button) | Present / Missing |
| Cancel mechanism | How to discard changes (Escape key, cancel button) | Present / Missing |
| Dirty check | Visual indicator that cell has unsaved changes | Present / Missing |
| Validation | Input validation before save (required, format, range) | Present / Missing |
| Optimistic update | UI updates immediately, reverts on API error | Used / Not used |
| Error handling | What happens when save API fails | Shown / Silent fail |
| Multi-cell edit | Can user edit multiple cells before saving? | Supported / Single only |

**Pass condition:**
- Clear save and cancel mechanism exists
- User can identify unsaved changes visually
- API errors are communicated to the user
- Escape key cancels the edit

**Fail condition:**
- No cancel mechanism — user must save or refresh the page
- No dirty indicator — user can't tell if changes are pending
- Save fails silently — user thinks data was saved but it wasn't
- No validation — invalid data accepted and sent to API

---

### Step 22: Check 21 — Missing Export Functionality (LOW)

For each list page:

**A) Estimate data volume:**
- Check if the list can grow beyond 50 items (user-generated or growing data)
- Static/small lists (<50 items) → export not needed → **SKIP**

**B) Check for export functionality:**
```
Search for: export|Export|download|Download|CSV|csv|Excel|xlsx|toCSV|toExcel
```

**C) If export exists, verify quality:**

| Check | Description | Assessment |
|-------|-------------|------------|
| Export scope | Current page only vs all pages vs filtered results | Documented? User knows? |
| Server-side export | Large datasets use backend CSV generation | Present / Frontend-only (may crash on large data) |
| Export respects filters | Applied filters reflected in exported data | Yes / No |
| Export respects sort | Applied sort order reflected in exported data | Yes / No |
| Export format | CSV, Excel (.xlsx), PDF | Appropriate for data type? |
| Export loading state | Progress indicator for large exports | Present / Missing |

**Pass condition:**
- List page with >50 potential items has export functionality
- Export respects current filters and sort order
- Large datasets use server-side export generation

**Fail condition:**
- Data-heavy list page with no export option — users resort to manual copy-paste or screenshots
- Export exists but only exports current page (user expects all filtered data)
- Frontend-only export on a list with 10,000+ potential rows (browser may crash)

---

## Output Format

```markdown
# QA List Audit Report

**Date**: YYYY-MM-DD
**Scope**: All apps | {specific app/page}
**Framework**: {detected frontend framework + table library}
**Backend**: {detected backend framework + ORM} (or "Frontend-only analysis")
**Pages Scanned**: {count}
**Total Issues Found**: {count}

---

## List Pages Matrix

| Page | Purpose | Default Sort | Sort UI | Search | Debounce | Col Resize | Loading | Empty | Filters | Filter Reset | Count | Toolbar | Fillable | List-Detail | Filter UI | Filter URL | Full-Stack | URL Sync | Freshness | Inline Edit | Export |
|------|---------|-------------|---------|--------|----------|------------|---------|-------|---------|-------------|-------|---------|----------|-------------|-----------|------------|------------|----------|-----------|-------------|--------|
| pages/users | creation | ✅ createdAt DESC | ✅ | ✅ | ✅ 300ms | ✅ | ✅ | ✅ | ✅ status | ✅ | ✅ X/Y | ✅ 1-line | ✅ | ⚠️ status | ✅ btn | ✅ | ✅ | ✅ | ✅ | N/A | ✅ CSV |
| pages/products | library | ✅ createdAt DESC | ⚠️ 3/8 | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ cat | ❌ | ✅ X/Y | ❌ 3-line | ❌ notes | N/A | ❌ Select | ❌ useState | ⚠️ | ❌ | ❌ | N/A | ❌ |

Legend: ✅ = Pass | ❌ = Fail | ⚠️ = Partial | N/A = Not applicable

---

## Summary

| # | Check | Severity | Issues |
|---|-------|----------|--------|
| 1 | Missing Default Sort | HIGH | X |
| 2 | Sort UI Disconnected | HIGH | X |
| 3 | Missing Search | MEDIUM | X |
| 4 | Missing Search Debounce | HIGH | X |
| 5 | Missing Column Resize | MEDIUM | X |
| 6 | Missing Loading State | HIGH | X |
| 7 | Missing Empty State | MEDIUM | X |
| 8 | Missing Filter Reset | MEDIUM | X |
| 9 | Single-Select Filter | LOW | X |
| 10 | Sorted Data Not Used | HIGH | X |
| 11 | Missing Result Count | MEDIUM | X |
| 12 | Multi-line Toolbar | MEDIUM | X |
| 13 | Unfillable Table Column | HIGH | X |
| 14a | List-Detail Status Logic Mismatch | HIGH | X |
| 14b | List-Detail Status Style Mismatch | HIGH | X |
| 15 | Filter Component Type Mismatch | HIGH | X |
| 16 | Filter State Not in URL | HIGH | X |
| 17 | Missing Full-Stack Tracing | HIGH | X |
| 18 | URL State Not Synced | MEDIUM | X |
| 19 | Missing Data Freshness Strategy | MEDIUM | X |
| 20 | Inline Edit Issues | MEDIUM | X |
| 21 | Missing Export Functionality | LOW | X |
| | **Total** | | **X** |

---

## HIGH Severity Issues

### Check 1: Missing Default Sort

| Page | Current Default | Expected Default | Fix |
|------|----------------|------------------|-----|
| {page} | None | createdAt DESC | Add default sort to table/hook configuration |

**Risk**: Users see data in arbitrary/insertion order. Most recent items buried at bottom.

### Check 2: Sort UI Disconnected

| Page | Column | Has Direction | Has Click | Issue |
|------|--------|--------------|-----------|-------|
| {page} | {col} | ❌ | ❌ | Header looks sortable but does nothing |

**Risk**: Misleading UI — user clicks column header expecting sort, nothing happens.

### Check 10: Sorted Data Not Used

| Page | Sort Returns | Template Uses | Issue |
|------|------------|---------------|-------|
| {page} | sortedData | rawData | Sorting logic runs but result ignored |

**Risk**: Sort appears to work (header arrows toggle) but list never reorders. Silent bug.

### Check 13: Unfillable Table Column

| Page | Column | Field | In Create | In Edit | Sub-type | Issue |
|------|--------|-------|-----------|---------|----------|-------|
| {page} | {label} | {field} | ❌ | ❌ | 13a | Column always shows empty |

**Risk**: Table column permanently shows empty/default value. Wastes space and confuses users.

### Check 17: Missing Full-Stack Tracing

| Page | Frontend Param | Backend Receives | Backend Queries | Issue |
|------|---------------|-----------------|----------------|-------|
| {page} | sortBy=name | ❌ ignored | ORDER BY id | Frontend sort param not applied in backend query |
| {page} | N/A | N/A | pageSize uncapped | No max page size guard — DoS risk |

**Risk**: Frontend and backend disagree on data ordering/filtering. Users see wrong results.

---

## MEDIUM Severity Issues

### Check 18: URL State Not Synced

| Page | State | In URL | Issue |
|------|-------|--------|-------|
| {page} | page number | ❌ useState | Refresh goes back to page 1 |
| {page} | sort order | ❌ useState | Refresh loses sort |

**Risk**: Users lose their place after refresh or back navigation.

### Check 19: Missing Data Freshness Strategy

| Page | After Create | After Update | After Delete | Issue |
|------|-------------|-------------|-------------|-------|
| {page} | ❌ no refetch | ❌ no refetch | ❌ no refetch | List shows stale data after mutations |

**Risk**: User performs CRUD action but list doesn't reflect changes. Confusing.
**Fix**: Add cache invalidation (e.g., `invalidateQueries`, `mutate()`, `refresh()`) after mutations.

### Check 20: Inline Edit Issues

| Page | Edit Trigger | Save | Cancel | Dirty Check | Issue |
|------|-------------|------|--------|-------------|-------|
| {page} | double-click | ❌ blur only | ❌ none | ❌ | No cancel mechanism, no dirty indicator |

**Risk**: User accidentally edits a cell and can't undo. Or doesn't realize changes are unsaved.

---

## LOW Severity Issues

### Check 21: Missing Export Functionality

| Page | Est. Items | Export Available | Issue |
|------|-----------|----------------|-------|
| {page} | 500+ | ❌ | No way to export data — users resort to screenshots |

**Risk**: Admin users need to share/analyze list data outside the app.
**Fix**: Add CSV/Excel export. For large datasets (>1000 rows), use server-side generation.

---

### Check 22: Expanded Row Index vs Entity Field

Scan for expandable/grouped tables (accordion rows, collapsible sub-tables). When a sub-table renders a `#` or numbering column, verify it uses the entity's actual field (`order`, `sequence`, `sortOrder`, `position`) — not `index + 1` from `.map()`.

**Detection pattern:**
1. Find expandable row patterns: `isExpanded`, `expandedGroups`, `expandedRows`, `Accordion`, `Collapsible`
2. Inside expanded content, find nested `.map((item, index)` or `.map((item, idx)`
3. Check if `index + 1` or `idx + 1` is rendered in a `<td>` — this is the bug
4. Cross-reference with the entity type: does it have an `order`, `sequence`, or `position` field?

| Page | Sub-table | Numbering Used | Entity Field | Match? | Issue |
|------|-----------|---------------|-------------|--------|-------|
| {page} | {group}.prescriptions | `index + 1` | `order` | ❌ | Shows 1,2,3 instead of actual order values |

**Risk**: Users see sequential 1,2,3 numbering that doesn't match the actual record order/sequence stored in the database. When items are reordered, deleted, or have non-sequential ordering, the displayed number is misleading.
**Fix**: Replace `index + 1` with `item.order` (or the appropriate entity field). If sequential display numbering is truly intended (not a DB field), rename the column header from `#` to something like "순서" to avoid confusion.

---

## Questions for Clarification

Issues found during audit that require project-specific decisions.
These cannot be resolved by code inspection alone — they need PRD/business input.

| # | Page | Question | Context | Suggested Default |
|---|------|----------|---------|-------------------|
| 1 | {page} | What should be the default sort? | Reference list with no creation | name ASC |

**When to generate questions:**
- Reference/lookup page has no default sort AND no PRD specification
- A page's sort order is ambiguous
- Page mixes creation + reference patterns
- Sort column exists but direction is debatable
- Filter/sort behavior differs from similar pages (inconsistency detected)

---

## Recommendations

1. **[Priority 1]** Fix all Check 10 issues first — sorted data not used means sorting is completely broken.
2. **[Priority 2]** Fix Check 17 — full-stack param mismatches mean backend ignores frontend user's sort/filter intent.
3. **[Priority 3]** Add default sort (createdAt/updatedAt DESC) to all time-based lists.
4. **[Priority 4]** Add debounce (300ms) to all search inputs for Korean/CJK IME support.
5. **[Priority 5]** Fix Check 19 — add cache invalidation after mutations so lists show current data.
6. **[Priority 6]** Connect all sortable column headers to sort handlers.
7. **[Priority 7]** Sync all interactive state to URL params (Check 18) for refresh/back-nav resilience.
8. **[Priority 8]** Add filter reset buttons to all pages with filters.
9. **[Priority 9]** Add "showing X of Y" result count to all list pages.
10. **[Priority 10]** Merge multi-line toolbars into single-line flex layout.
11. **[Priority 11]** Fix unfillable table columns — add missing form fields or remove dead columns.
12. **[Priority 12]** Fix list-detail status mismatches.
13. **[Priority 13]** Standardize filter component types across same-type pages.
14. **[Priority 14]** Fix inline edit UX issues (save/cancel/dirty check).
15. **[Priority 15]** Add export functionality to data-heavy list pages (>50 items).
16. **[Priority 16]** Fix expanded sub-table numbering — use entity field instead of array index.
```

## Scanning Patterns Reference

| What | Pattern | Where to Search |
|------|---------|-----------------|
| Sort hook/config | `useTableSort\(\|getSortedRowModel\|manualSorting\|sorter:\|matSort\|sortable` | frontend pages/components |
| Default sort column | Default column in sort hook/config (varies by library) | frontend pages/components |
| Sortable header | `SortableHeader\|sortable\|enableSorting\|matSortActive` | frontend pages/components |
| Sort direction prop | Sort direction binding on column headers | frontend pages/components |
| Column resize | `useColumnResize\|enableResizing\|resizable\|suppressResize` | frontend pages/components |
| Search input | `type="search"\|placeholder=.*search\|Search.*className` | frontend pages/components |
| Search filter logic | `\.filter\(.*search\|\.includes\(.*search\|toLowerCase` | frontend pages/components |
| Search debounce | `setTimeout.*search\|debounce.*search\|useDebouncedValue\|useDebounce` | frontend pages/components |
| No debounce (direct) | `onChange.*setSearchTerm\|onChange.*setQuery` (no setTimeout wrapper) | frontend pages/components |
| Loading state | `isLoading\|isFetching\|isPending\|loading` | frontend pages/components |
| Loading UI | `animate-pulse\|Spinner\|LoadingOverlay\|Skeleton\|skeleton` | frontend pages/components |
| Empty state | `\.length === 0\|noData\|emptyState` | frontend pages/components |
| Filter state | `filter\|Filter\|useState.*filter` | frontend pages/components |
| Filter reset | `reset\|clear\|clearFilter\|clearAll` | frontend pages/components |
| Multi-select filter | `Set<\|new Set\|useState<.*\[\]>` for filter state | frontend pages/components |
| Data iteration | `\.map\(` after sort hook | frontend pages/components |
| Result count display | `showingCount\|totalCount\|of.*total` | frontend pages/components |
| Filtered vs total | `filteredData\.length.*data\.length\|\.length.*\.length` | frontend pages/components |
| Multi-line toolbar | Stacked vertical containers wrapping search+filter sections | frontend pages/components |
| Single-line toolbar | Flex row container (utility: `flex.*items-center.*gap`, CSS: `display: flex`) containing search+tabs+filters | frontend pages/components |
| Table row field refs | `\.\w+\s*\|\|\s*["\-]` (e.g., `item.notes \|\| "-"`) | frontend pages/components |
| Create/edit forms | `create-.*-modal\|edit-.*-modal\|create-.*-form\|edit-.*-form` | frontend components |
| Form schema fields | `z\.object\|yup\.object\|Joi\.object` then field names | frontend components |
| Form register fields | `register\(['"](\w+)['"]\)\|name=['"](\w+)['"]` | frontend components |
| List-detail pairs | `navigate\(.*detail\|router\.push\|<Link.*to=` | frontend pages |
| Expandable rows | `isExpanded\|expandedGroups\|expandedRows\|Accordion\|Collapsible` | frontend pages/components |
| Sub-table index numbering | `index \+ 1\|idx \+ 1` inside expanded/nested table `<td>` | frontend pages/components |
| Entity order field | `\.order\|\.sequence\|\.position\|\.sortOrder` | frontend types, backend entities |
| Status display logic | `Badge\|status\|isActive\|active\|inactive` | frontend pages/components |
| Status conditional chain | `isActive\|status ===\|status !==` | frontend pages/components |
| Status component type | `<Badge.*variant\|<span.*className.*rounded.*font-medium` | frontend pages/components |
| Filter component type | `<Select\|SelectTrigger\|<el-select\|mat-select` in toolbar | frontend pages/components |
| Inline button group | `rounded-lg border p-1\|gap-1.*border.*p-1\|btn-group` | frontend pages/components |
| URL state sync | `useSearchParams\|searchParams\.get\|useRoute.*query\|ActivatedRoute.*queryParams` | frontend pages/components |
| Data fetching | `useQuery\|useSWR\|useFetch\|useAsyncData\|HttpClient\.get` | frontend pages/components |
| Cache invalidation | `invalidateQueries\|mutate\(\|refetch\|refresh\|revalidate` | frontend pages/components |
| Optimistic update | `onMutate.*setQueryData\|optimisticUpdate\|previousData` | frontend pages/components |
| Inline edit | `editable.*true\|editableCell\|contentEditable\|processRowUpdate\|onCellValueChanged` | frontend pages/components |
| Export functionality | `export\|Export\|download\|Download\|CSV\|csv\|Excel\|xlsx\|toCSV\|toExcel` | frontend pages/components |
| Backend controller | `@Get\|@Controller\|router\.get\|app\.get\|@GetMapping\|Route::get` | backend controllers |
| Backend query params | `@Query\|req\.query\|request\.query\|@RequestParam\|\$request->query` | backend controllers |
| ORM pagination | `findAndCount\|findMany.*skip.*take\|paginate\|LIMIT.*OFFSET\|Pageable` | backend services/repositories |
| ORM ordering | `order:\|orderBy:\|order_by\|ORDER BY\|Sort\.by` | backend services/repositories |

## Important Notes

- **Framework auto-detection**: This skill auto-detects frontend framework, table library, backend framework, and ORM. It adapts its checks to the detected stack. Refer to `qa-shared/reference.md` for the complete detection tables and fallback strategies.
- **Korean IME support**: Search inputs MUST use debounce (300ms+) for Korean/Chinese/Japanese input. Without debounce, characters are submitted mid-composition.
- **URL state**: Filters, sort, pagination, and search should all persist in URL params for back-nav and sharing.
- **Creation vs. reference pages**: Creation pages (have add/create button) default to `createdAt DESC`. Reference/lookup pages (no create, just browse/select) must have sort order defined in PRD.
- **Full-stack tracing**: When `--depth full` (default), the skill traces frontend params through backend controller to database query to verify consistency.
- **Data freshness**: After any mutation (create/update/delete), the list should refetch or optimistically update. Stale data is a common and frustrating bug.
- **Inline edit safety**: Inline editing must have clear save/cancel mechanics and error handling. Silent save failures are dangerous.
- **Export for data-heavy lists**: Lists with >50 items should offer CSV/Excel export. Lists with >1000 items should use server-side export generation.

## Related Skills

- `qa-inputs` — Input fields (filter inputs on table pages)
- `qa-back-nav` — Back navigation (back from detail preserves table state)
- `qa-states` — Loading/error/empty states (tables need all three)
- `qa-modal` — Modals (create/edit modals triggered from table actions)
- `qa-auth` — Permissions (table row actions may be role-gated)
- `qa-a11y` — Accessibility (table keyboard navigation, screen reader support)
- `qa-api-sync` — API sync (full-stack param tracing)

## READ-ONLY Diagnostic

This skill is a READ-ONLY diagnostic. It does NOT modify any files. It scans, analyzes, and reports findings. All recommended fixes are presented as suggestions in the report for the developer to implement.
