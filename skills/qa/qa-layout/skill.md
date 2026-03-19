---
name: qa-layout
description: Audit page layout consistency - title placement, panel structure, spacing, borders, rounded corners, sidebar alignment, toolbar gaps, search widths, count display placement, and typography hierarchy
user-invocable: true
argument-hint: "[app-path or page-name]"
---

# QA Layout - Page Layout Consistency Auditor

## Purpose

Verify all pages within an app follow consistent layout patterns: title placement, panel structure, padding/margin, border styles, rounded corners, sidebar alignment, and container styling. Catches visual inconsistencies like the title being inside a sub-panel on one page but spanning full width on another.

## Usage

```
/qa-layout                           # Full audit of all apps
/qa-layout admin                     # Only admin dashboard pages
/qa-layout orders                    # Single page audit
```

## 22 Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Title placement inconsistency | **High** | Page title (`<h1>`) is inside a sub-panel instead of spanning full width like sibling pages |
| 2 | Layout padding mismatch | **High** | Same-level pages use different padding negation patterns (`-m-6` vs none) |
| 3 | Border style inconsistency | **Medium** | Panel separators differ across pages (`border-t` vs `border` vs `border rounded-*`) |
| 4 | Rounded corner inconsistency | **Medium** | Same-role containers have `rounded-*` on some pages but not others |
| 5 | Sidebar gap inconsistency | **High** | Shared sidebar component (e.g., SharedSidebar) has different spacing/gap from the app sidebar across pages |
| 6 | Shared component wrapper mismatch | **Medium** | Same component used across pages but parent wrapper classes differ significantly |
| 7 | Content area structure mismatch | **Medium** | Pages with same layout type (sidebar+content) use different flex/grid structures |
| 8 | Header action alignment inconsistency | **Low** | Title-row action buttons (export, refresh) positioned differently across pages |
| 9 | Toolbar gap inconsistency | **High** | Same-type pages use different gap values on the filter/search toolbar (e.g., `gap-2` vs `gap-4`) |
| 10 | Count display placement inconsistency | **Medium** | Result count text (`Showing X of Y`) positioned differently or uses inconsistent margins (e.g., `-mt-3` on some pages) |
| 11 | Search input width inconsistency | **Low** | Search inputs use different widths across same-type pages (e.g., `w-64` vs `w-80`) |
| 12 | Label-value typography hierarchy | **Medium** | Detail/form pages have insufficient visual distinction between labels and values (font-size, font-weight, color) |
| 13 | Filter component type inconsistency | **High** | Same-type pages use different filter UI patterns — Select dropdown on one page vs inline button group on another |
| 14 | Scroll container missing | **High** | Content area inside overflow-hidden layout lacks `overflow-y-auto`, making long content unscrollable |
| 15 | Cross-role page structure divergence | **High** | Same-function pages across roles (e.g., signup/coach vs signup/patient, coach/profile vs patient/profile) use different layout patterns (fixed footer vs scroll-inline CTA, different padding, different scroll strategy) |
| 16 | Minimum font size violation | **High** | Text elements use font sizes below 12px (e.g., `text-[8px]`, `text-[9px]`, `text-[10px]`, `text-[11px]`, or CSS `font-size` < 12px). 12px is the industry-standard minimum for readability across all project types |
| 17 | Arbitrary font size values | **Medium** | Custom pixel-based font sizes (e.g., `text-[13px]`, `text-[14px]`, `text-[15px]`) used instead of the framework's standard scale (e.g., Tailwind `text-xs`/`text-sm`/`text-base`/`text-lg`). Arbitrary values fragment typography and prevent global updates |
| 18 | Page heading size inconsistency | **High** | Same-level page headings use different font sizes across the app (e.g., some pages use `text-xl` for page title while others use `text-2xl` for the same hierarchy level) |
| 19 | Font size variety overload | **Medium** | App uses more than 6 unique font size values. A well-structured typography system should use 4-6 sizes maximum (e.g., `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`). More indicates lack of design system |
| 20 | Form label font size inconsistency | **Medium** | Form labels across different pages use different font sizes (e.g., `text-[11px]` on auth pages but `text-sm` on settings pages). All form labels within an app should use a single consistent size |
| 21 | Arbitrary color values bypassing design tokens | **High** | Hardcoded hex/rgb color values (e.g., `text-[#154D71]`, `bg-[#6B7280]`) used instead of design tokens (e.g., `text-primary-600`, `bg-neutral-500`) when the project defines a color palette. Prevents theme updates and fragments the design system |
| 22 | Z-index hierarchy inconsistency | **High** | Z-index values across the app do not follow a consistent hierarchy (e.g., dropdown at `z-50` while modal at `z-40`). Expected layering: base(0) → content(10) → sticky(20) → dropdown(30) → modal(40) → toast(50). Same-purpose elements should use the same z-index |

## Execution Algorithm

You MUST follow these steps in order. Do NOT skip steps. Do NOT modify any files. This is a READ-ONLY diagnostic.

**Prerequisites:** Read `qa-shared/reference.md` for framework detection tables and pattern mappings.

### Step 0: Context Gathering (MANDATORY)

**0a. Detect frontend framework and CSS framework:**

Use the detection tables from `qa-shared/reference.md`:
1. Read `package.json` (or `requirements.txt`, `composer.json`, etc.) to identify frontend framework
2. Detect CSS framework by checking for Tailwind config, CSS Modules, Styled Components, SCSS, etc.
3. Detect UI component library (shadcn, MUI, Ant Design, Element Plus, Vuetify, etc.)

Record detected stack — all subsequent analysis adapts based on detection results.

**0b. Read project structure:**
```
Read: CLAUDE.md (project root)
```

Extract:
- Which frontend apps exist (auto-detect from project structure)
- Layout component locations
- Known shared components (sidebars, headers)

**0c. Identify the layout wrapper:**

Use page file patterns from `qa-shared/reference.md` to locate layout files:

| Framework | Layout File Patterns |
|-----------|---------------------|
| React Router | `**/layout.tsx`, `**/Layout.tsx`, `**/layouts/*.tsx` |
| Next.js (App) | `app/**/layout.tsx` |
| Next.js (Pages) | `pages/_app.tsx` |
| Vue Router | `**/layouts/*.vue`, `**/App.vue` |
| Nuxt | `layouts/**/*.vue` |
| Angular | `**/*-layout.component.ts`, `**/*-layout.component.html` |
| SvelteKit | `src/routes/**/+layout.svelte` |
| Generic | Grep for components wrapping `children`/`<slot>`/`<Outlet>`/`<router-view>` |

Read each layout file and record:
- Root wrapper element styling (classes, inline styles, or styled-component definitions)
- Main content area styling (padding, margins)
- These define the "default spacing" that pages inherit

**0d. Identify shared layout components:**
```
Grep: "Sidebar|SidePanel|FilterPanel|NavigationMenu|AppBar|NavBar" in components/ directories
```

Read each to record their root element styling (width, borders, background).

---

### Step 1: Page Inventory

Scan all page files in each frontend app using the detected framework's page file patterns (from `qa-shared/reference.md` → Page File Patterns table).

For each page, read the rendered output and extract:

| Field | What to Record |
|-------|---------------|
| **Root wrapper** | The outermost container element and its styling (classes, styles, or styled-component) |
| **Spacing override** | Does it override/negate the layout's default spacing? How? |
| **Title element** | `<h1>` or equivalent heading — what parent container is it in? Full-width or inside a sub-panel? |
| **Title row styling** | Styling on the title's direct parent (layout direction, alignment, spacing) |
| **Panel structure** | How is the page split? (single column / sidebar+content / multi-panel) |
| **Panel separator** | What separates panels? (borders, gaps, dividers, nothing) |
| **Container decoration** | Are there rounded corners, shadows, or borders on major containers? |
| **Shared components** | Does it use shared sidebar, navigation, or layout components? |
| **Header actions** | Buttons/controls in the title row — position and alignment |

Build a master inventory table.

**CSS framework-specific extraction:**
- **Utility-class frameworks** (Tailwind, UnoCSS): Read `className` or `class` attribute directly
- **CSS Modules**: Resolve `styles.xxx` → read the `.module.css/.scss` file for actual CSS properties
- **Styled Components / Emotion**: Read the template literal CSS properties
- **SCSS / Plain CSS**: Resolve class names → read the stylesheet for actual CSS properties
- **Inline styles**: Read `style` attribute or style object directly
- **Vue scoped styles**: Read `<style scoped>` section in `.vue` files

### Classification

Group pages by **layout type**:

| Layout Type | Description | Expected Pattern |
|-------------|-------------|-----------------|
| **Full-width** | Single content area, no sidebar | Title at top, content below, uses layout padding directly |
| **Sidebar+Content** | Left sidebar panel + right main content | `-m-6` to negate layout padding, title above split, `border-t` separator |
| **Multi-panel** | 3+ panels (e.g., complex builder/editor) | `-m-6`, title above panels, panels separated by borders |
| **Detail** | Single entity detail view | May or may not negate padding — but should be consistent with other detail pages |

Pages of the SAME layout type MUST use the SAME structural pattern.

---

### Step 2: Check 1 — Title Placement Inconsistency (HIGH)

For each group of same-type pages:

**Pass condition:**
- `<h1>` title is in the SAME structural position across all pages of the same layout type
- For sidebar+content pages: title is in a full-width row ABOVE the sidebar/content split
- For full-width pages: title is in the main content area

**Fail condition:**
- Page A has `<h1>` spanning full width above the panel split
- Page B has `<h1>` inside the right content panel (after the sidebar)
- Mixed placement within the same layout type group

**How to detect:**
```
For sidebar+content pages, check if <h1> is:
  ✅ BEFORE the flex container that holds sidebar + content
  ❌ INSIDE the flex-1 content area (sibling of sidebar, not above it)
```

Look at the DOM nesting:
```
✅ Correct:
<div root>
  <div title-row><h1>Title</h1></div>     ← full width
  <div flex>
    <Sidebar />
    <div content>...</div>
  </div>
</div>

❌ Wrong:
<div root flex>
  <Sidebar />
  <div content>
    <h1>Title</h1>                         ← inside sub-panel
    ...
  </div>
</div>
```

---

### Step 3: Check 2 — Layout Padding Mismatch (HIGH)

Compare the root wrapper of all pages within the same layout type.

**Pass condition:**
- All sidebar+content pages use `-m-6` (or the same value matching layout.tsx padding)
- All full-width pages either ALL use default padding or ALL negate it

**Fail condition:**
- Page A uses `<div className="flex h-full -m-6">` (negates padding)
- Page B uses `<div className="flex h-[calc(100vh-96px)]">` (manual height calc, no negation)
- Page C uses `<div className="space-y-6">` (default padding, no negation)
- Inconsistent approaches within same layout type

**How to detect:**
```
1. Grep root wrapper for: -m-6|-m-4|-mx-6|-my-4|calc\(100vh
2. Check if the negation value matches layout.tsx padding (px-6 py-4 → -m-6, or -mx-6 -my-4)
3. CRITICAL: Compare the FULL className of root wrappers across same-type pages character by character.
   Pages of the same layout type MUST have identical root wrapper classes.
   Example mismatch:
     Page A: "flex flex-col h-[calc(100vh-96px)] gap-4 pb-4"
     Page B: "flex flex-col h-[calc(100vh-96px)] gap-4"       ← missing pb-4
   This causes visible spacing differences (one page has bottom breathing room, the other doesn't).
4. When the root wrapper uses a fixed height like h-[calc(100vh-Npx)], check if it accounts for
   the layout's bottom padding (pb-8 in layout.tsx). If the calc doesn't subtract bottom padding,
   the page content may overflow beyond the visible area, hiding the layout's pb-* spacing.
```

---

### Step 4: Check 3 — Border Style Inconsistency (MEDIUM)

For pages with the same layout structure, compare how panels are separated.

**Pass condition:**
- All sidebar+content pages use the same border pattern between title and panels (e.g., all use `border-t`)
- Sidebar component consistently uses `border-r` across all pages

**Fail condition:**
- Page A: `<div className="flex flex-1 min-h-0 border-t">`
- Page B: `<div className="flex flex-1 min-h-0">` (no border separator)
- Page C: `<div className="flex flex-1 min-h-0 border rounded-xl">` (full border + rounded)

**How to detect:**
```
Grep: border-t|border-r|border-b|border-l|border\s|border rounded in the panel split container
Compare across all pages of same layout type
```

---

### Step 5: Check 4 — Rounded Corner Inconsistency (MEDIUM)

Scan for `rounded-*` classes on major layout containers (NOT on small UI elements like avatars, badges, buttons).

**Pass condition:**
- All major containers (panel wrappers, content areas, sidebar containers) either ALL have rounding or NONE have rounding
- Small UI elements (avatars, badges, pills, buttons) are excluded from this check

**Fail condition:**
- Content panel wrapper on page A: `border rounded-xl overflow-hidden`
- Content panel wrapper on page B: `border overflow-hidden` (no rounding)
- Same shared component (e.g., CalendarContainer) has `rounded-lg` on its root but the page wrapper expects sharp edges

**Exclude from check:**
- `rounded-full` on avatars, badges, dots, pills, spinners
- `rounded-md` on individual list items, buttons, inputs
- `rounded-lg` on modals, dialogs, popovers, tooltips

**Include in check:**
- `rounded-*` on: page root wrappers, panel containers, card wrappers around main content, shared layout component roots

**How to detect:**
```
For each page, grep for rounded-* on elements that are:
  - Direct children of the page root
  - Panel split containers
  - Shared component root elements
Ignore: small UI elements (check parent context)
```

---

### Step 6: Check 5 — Sidebar Gap Inconsistency (HIGH)

When a shared sidebar component (e.g., SharedSidebar) is used across multiple pages:

**Pass condition:**
- The sidebar is at the same DOM depth relative to the page root across all pages
- The parent flex container has the same gap/spacing across all pages
- No extra padding/margin creates visual gap between app sidebar and page sidebar

**Fail condition:**
- Page A: sidebar is inside a `-m-6` negated container → flush with app sidebar
- Page B: sidebar sits inside default padded area → has a gap from app sidebar
- Page C: sidebar parent has `gap-6` → extra space between sidebar and content

**How to detect:**
```
For each page using a shared sidebar:
  1. Trace the sidebar's parent chain to the page root
  2. Check if -m-* negation is applied to bring it edge-to-edge
  3. Compare the gap/padding values on the flex parent across pages
```

---

### Step 7: Check 6 — Shared Component Wrapper Mismatch (MEDIUM)

When the same component is used across multiple pages, compare the parent wrapper.

**Pass condition:**
- `<SharedSidebar />` has the same parent flex structure on all pages that use it
- The flex parent classes are identical (or semantically equivalent)

**Fail condition:**
- Page A wraps sidebar in: `<div className="flex flex-1 min-h-0 border-t">`
- Page B wraps sidebar in: `<div className="flex h-[calc(100vh-96px)] gap-0">`
- Page C wraps sidebar in: `<div className="flex h-full">`

**How to detect:**
```
For each shared component, find all usage sites
Extract the immediate parent element's className
Compare across all sites — flag significant differences
```

Significant differences include: different height strategies (`h-full` vs `calc` vs `min-h-0`), different gap values, different border patterns.

Minor differences to IGNORE: additional content-specific classes that don't affect layout.

---

### Step 8: Check 7 — Content Area Structure Mismatch (MEDIUM)

For pages of the same layout type, compare the main content area (the flex-1 panel next to the sidebar).

**Pass condition:**
- All use similar flex direction and spacing (e.g., all use `flex-1 p-4 space-y-4 overflow-auto`)
- Padding values are consistent

**Fail condition:**
- Page A content: `flex-1 p-4 space-y-4 overflow-auto`
- Page B content: `flex-1 flex flex-col gap-6 pl-6 min-w-0` (different padding, gap instead of space-y)
- Page C content: `flex-1 flex flex-col gap-4 p-4 min-w-0`

**How to detect:**
```
Extract className of the main content div (sibling of sidebar) from each page
Compare padding (p-4 vs pl-6 vs p-6), spacing (space-y-4 vs gap-4 vs gap-6), and flex direction
```

---

### Step 9: Check 8 — Header Action Alignment Inconsistency (LOW)

For pages that have action buttons in the title row:

**Pass condition:**
- All pages with header actions use `flex items-center justify-between` on the title row
- Action buttons are consistently right-aligned

**Fail condition:**
- Page A: title row has `flex items-center justify-between` with export button on right
- Page B: title row has no flex, action buttons are below the title
- Page C: title row has action buttons left-aligned next to title

**How to detect:**
```
For each page with header actions (buttons in the title-level container):
  Extract the title row className
  Check for justify-between or ml-auto patterns
  Compare positioning across pages
```

---

### Step 10: Check 9 — Toolbar Gap Inconsistency (HIGH)

For full-width pages that have a search/filter toolbar below the title:

**Pass condition:**
- All same-type pages use the same gap value on the toolbar flex container
- Example standard: `flex items-center gap-2 flex-wrap`

**Fail condition:**
- Page A toolbar: `flex items-center gap-2 flex-wrap`
- Page B toolbar: `flex items-center gap-4 flex-wrap`
- Page C toolbar: `flex items-center gap-3 flex-wrap`

**How to detect:**
```
Grep: flex.*items-center.*gap-\d.*flex-wrap in same-type pages
Extract the gap-N value from each toolbar container
Flag any mismatches within the same layout type group
```

---

### Step 11: Check 10 — Count Display Placement Inconsistency (MEDIUM)

For pages that show a result count (e.g., "Showing X of Y", "Showing X of Y"):

**Pass condition:**
- Count text is in the same position relative to the toolbar across all same-type pages
- Uses consistent styling: `text-sm text-muted-foreground` with same margin/spacing

**Fail condition:**
- Page A: count below toolbar with `-mt-3` (negative margin to reduce gap)
- Page B: count below toolbar with no margin adjustment
- Page C: count inside a stats row with `flex items-center gap-6`

**How to detect:**
```
Grep: text-sm.*text-muted-foreground near count/total/showing text
Check parent container margins: -mt-3, -mt-2, mt-0, etc.
Compare placement relative to toolbar across same-type pages
```

---

### Step 12: Check 11 — Search Input Width Inconsistency (LOW)

For pages with search inputs in the toolbar:

**Pass condition:**
- All same-type pages use the same search input width (e.g., all `w-64`)

**Fail condition:**
- Page A search: `w-64` (256px)
- Page B search: `w-80` (320px)
- Page C search: `w-72` (288px)

**How to detect:**
```
Grep: relative w-\d+ (search input container pattern)
Extract width class from each page's search input
Flag mismatches within the same layout type group
```

---

### Step 13: Check 12 — Label-Value Typography Hierarchy (MEDIUM)

For detail pages and form pages that display label-value pairs (e.g., user-detail, item-detail):

**Pass condition:**
- Labels and values have at least 2 levels of visual distinction (font-size AND font-weight AND/OR color)
- Standard pattern: Label = `text-xs font-medium text-muted-foreground`, Value = `text-base font-medium text-foreground`
- The size gap between label and value is at least one step (e.g., `text-xs` vs `text-base`, not `text-xs` vs `text-sm`)

**Fail condition:**
- Label uses `text-xs text-muted-foreground` and value uses `text-sm text-foreground` — only 1 step size difference, no weight difference, hard to distinguish
- Label and value use the same font-weight (both `font-medium` or both `font-normal`) with only a subtle color change
- Read-only values use `text-muted-foreground` making them look like labels instead of data

**How to detect:**
```
For each detail/form page:
  1. Find all <Label> or label-like elements — extract className (text-*, font-*, text-muted-*)
  2. Find sibling <p> or display elements — extract className
  3. Compare the typography tuple: (font-size, font-weight, color)
  4. Minimum required gap:
     - Font-size: at least 2 steps apart (text-xs→text-base, not text-xs→text-sm)
     - Font-weight: value should be font-medium or font-semibold if label is font-medium
     - Color: value should be text-foreground (not text-muted-foreground) for editable fields
  5. Flag pages where label and value are visually too similar
```

**Reference pattern for detail pages:**
```tsx
{/* Label */}
<Label className="text-xs font-medium text-muted-foreground uppercase tracking-wide">
  {t('field.label')}
</Label>
{/* Value (read mode) */}
<p className="text-base font-medium text-foreground py-2">
  {value || '-'}
</p>
```

---

### Step 14: Check 13 — Filter Component Type Inconsistency (HIGH)

For full-width list pages that have status/category filter controls in the toolbar:

**Pass condition:**
- All same-type pages use the SAME filter UI component pattern
- Standard pattern: **Inline button group** — `<div className="flex items-center gap-1 rounded-lg border p-1">` containing `<Button variant="default"/"ghost" size="sm" className="relative h-7 px-3 text-xs">` for each option
- Active option uses `variant="default"`, inactive uses `variant="ghost"`

**Fail condition:**
- Page A uses `<Select>` dropdown (requires click to open, hides options)
- Page B uses inline button group (all options visible at once, one-click toggle)
- Mixed component types for the same kind of filter across same-type pages

**How to detect:**
```
For each full-width list page with status/category filters:
  Search for: <Select|SelectTrigger|SelectContent|SelectItem → uses dropdown
  Search for: rounded-lg border p-1|variant.*default.*ghost|gap-1.*border.*p-1 → uses inline button group
  Compare filter component type across all same-type pages
```

**Why inline button group is preferred over Select dropdown:**
1. **All options visible** — user can see and compare all filter states at once without clicking
2. **One-click operation** — no open → scroll → select → close cycle
3. **Consistent with tab patterns** — visually similar to TabsList, maintaining design language
4. **Better for small option sets** (2–5 options) — Select is better for 6+ options

**Reference implementation** (canonical filter pattern):
```tsx
<div className="flex items-center gap-1 rounded-lg border p-1">
  {([
    { value: "all", label: t("common.all") },
    { value: "active", label: t("status.active") },
    { value: "inactive", label: t("status.inactive") },
  ]).map(({ value, label }) => (
    <Button
      key={value}
      variant={currentFilter === value ? "default" : "ghost"}
      size="sm"
      className="relative h-7 px-3 text-xs"
      onClick={() => setFilter(value)}
    >
      {label}
    </Button>
  ))}
</div>
```

**When Select dropdown IS acceptable:**
- Filter has 6+ options (too many for inline buttons)
- Filter options are dynamic/user-generated (unpredictable count)
- Filter needs search/typeahead within options

---
### Step 15: Check 14 — Scroll Container Missing on Content Pages (HIGH)

For mobile/SPA pages where the parent layout uses `overflow-hidden` (e.g., `h-screen overflow-hidden flex flex-col`), each page's content area must explicitly enable scrolling.

**Pass condition:**
- All page content areas within the same app use `flex-1 overflow-y-auto min-h-0` (or equivalent scroll-enabling pattern)
- The scrollable area is correctly placed below any sticky/fixed header

**Fail condition:**
- Page A content: `flex-1 overflow-y-auto min-h-0 px-5 pt-4 pb-4` ✅
- Page B content: `px-5 pt-4 pb-4` ❌ (no overflow-y-auto → content is clipped, cannot scroll)
- Page C content: `flex-1 overflow-y-auto no-scrollbar` ✅

**How to detect:**
```
1. Read layout.tsx for each app — check if root uses overflow-hidden
2. CHECK LAYOUT ITSELF FIRST: If layout has overflow-hidden, the Outlet wrapper div MUST have overflow-y-auto
   - e.g., <div className="... overflow-hidden ..."><div className="... flex-1 ... overflow-y-auto"><Outlet /></div></div>
   - If the Outlet wrapper lacks overflow-y-auto, NO child page can scroll regardless of its own classes
3. If layout Outlet wrapper has overflow-y-auto, then child pages don't need their own scroll container
4. If layout Outlet wrapper does NOT have overflow-y-auto, ALL child pages MUST have their own scroll container
5. Grep page content areas for overflow-y-auto or overflow-y-scroll
6. Flag any page that lacks scroll enablement on its main content div
7. Cross-check: pages with long content (forms, lists) are especially at risk
8. FLEX CHAIN CHECK: For every element with `flex-1 min-h-0`, trace upward — every ancestor
   up to the nearest `h-[calc(...)]` or `h-screen` root MUST have `flex flex-col` (or `flex flex-row`
   for horizontal layouts). Common breakpoints:
   - TabsContent, AccordionContent, DialogContent — these are often plain divs, NOT flex containers
   - Conditional wrappers: `{condition && <div>...</div>}` — the wrapping div may lack flex
   - forceMount containers that use `hidden` class toggling
   Quick grep: find pages using Tabs/Accordion inside a flex layout, then check if the
   *Content component's className includes `flex flex-col`. If not → broken chain → FAIL level 4.
```

**Four levels of failure:**
1. **Layout-level (blocks ALL pages):** Layout has `overflow-hidden` but Outlet wrapper div lacks `overflow-y-auto` → every page under this layout is unscrollable
2. **Page-level (blocks one page):** Layout Outlet wrapper delegates scrolling to pages, but a specific page forgot `overflow-y-auto`
3. **Height conflict (clips footer):** Layout uses `min-h-screen` + `overflow-hidden` → container grows beyond viewport but overflow is clipped, cutting off fixed footers. Fix: use `h-screen` (fixed) instead of `min-h-screen` (growing) when `overflow-hidden` is present
4. **Broken flex constraint chain (overflow past parent):** A child has `flex-1 min-h-0` but an intermediate wrapper (e.g., TabsContent, AccordionContent, DialogContent) lacks `flex flex-col`, breaking the constraint propagation. The scroll container renders at natural height and overflows past its grandparent's boundary, hiding bottom padding/spacing. Symptom: `overflow-auto` is present but scrollbar never appears; content bleeds past the visible container edge.

**Common root causes:**
- Layout-level: Developer adds `overflow-hidden` to prevent body scroll bounce but forgets to add `overflow-y-auto` on the inner content div that wraps `<Outlet />`
- Page-level: Developer copies the header from another page but builds the content area from scratch, forgetting to add `flex-1 overflow-y-auto min-h-0`
- Height conflict: Developer uses `min-h-screen` for "at least full height" but combines with `overflow-hidden` — the container grows past viewport, then gets clipped. `min-h-screen` + `overflow-hidden` is always a bug; use `h-screen` instead
- Broken flex chain: Developer nests a flex child (`flex-1 min-h-0`) inside a wrapper component that is NOT a flex container. `flex-1` only works when the parent has `display: flex`. Common offenders: `TabsContent`, `AccordionContent`, conditional wrappers, `forceMount` containers. Fix: add `flex flex-col` to every intermediate container between the flex root and the scroll container

**Reference pattern (mobile patient/coach layout):**
```tsx
{/* Layout root — overflow-hidden to contain the app */}
<div className="max-w-md mx-auto h-screen bg-neutral-50 overflow-hidden flex flex-col pb-28">
  {/* Outlet wrapper — MUST have overflow-y-auto to enable page scrolling */}
  <div className="relative z-10 flex-1 flex flex-col min-h-0 overflow-y-auto">
    <Outlet />
  </div>
  <BottomNav />
</div>
```

**Reference pattern (individual page within layout):**
```tsx
{/* Page root — flex column to fill layout outlet */}
<div className="flex-1 flex flex-col min-h-0">
  {/* Sticky header */}
  <div className="sticky top-0 z-20 bg-white">...</div>

  {/* Scrollable content — MUST have overflow-y-auto */}
  <div className="flex-1 overflow-y-auto min-h-0 px-5 pt-4 pb-4">
    {/* Page content */}
  </div>
</div>
```

---

### Step 16: Check 15 — Cross-Role Page Structure Divergence (HIGH)

For pages that serve the same function across different user roles (e.g., signup/coach vs signup/patient, coach/profile vs patient/profile, coach/notifications vs patient/notifications):

**Pass condition:**
- Both pages use the same layout skeleton: same scroll strategy, same CTA placement, same padding values
- Structural differences are limited to content (labels, icons, role-specific fields), NOT layout patterns

**Fail condition:**
- Page A (patient signup): submit button in **fixed footer** (`shrink-0 p-6 pt-2 border-t`)
- Page B (coach signup): submit button **inside scrollable area** (position varies by screen size)
- Page A (patient layout): Outlet wrapper has `overflow-y-auto`
- Page B (coach layout): Outlet wrapper missing `overflow-y-auto`

**How to detect:**

```
1. Identify cross-role page pairs by matching file names or route patterns:
   - signup/coach.tsx ↔ signup/patient.tsx
   - coach/profile.tsx ↔ patient/profile.tsx
   - coach/notifications.tsx ↔ patient/notifications.tsx
   - coach/chat.tsx ↔ patient/chat.tsx
   - coach/chat-detail.tsx ↔ patient/chat-detail.tsx
   - coach/layout.tsx ↔ patient/layout.tsx

2. For each pair, compare:
   a. Root wrapper className (flex direction, height strategy)
   b. Scroll strategy (overflow-y-auto presence and location)
   c. CTA/submit button placement (fixed footer vs inline)
   d. Content area padding (px-5 vs px-6 vs px-8)
   e. Header component (PageHeader vs custom)

3. Flag pairs where structural layout patterns differ, ignoring:
   - Role-specific labels, icons, text content
   - Role-specific fields (e.g., license number for coach)
   - Color/theme differences
```

**Common divergence causes:**
- One page updated, counterpart forgotten (most common)
- Pages created at different times by different approaches
- One page refactored, counterpart left in legacy state

**Comparison table format in report:**

```markdown
| Aspect | coach/profile | patient/profile | Match |
|--------|--------------|----------------|-------|
| Scroll strategy | flex-1 overflow-y-auto | flex-1 overflow-y-auto | ✅ |
| Content padding | px-5 pt-4 pb-4 | px-5 pt-4 pb-4 | ✅ |
| CTA placement | inline | inline | ✅ |
| Header | PageHeader | PageHeader | ✅ |

| Aspect | signup/coach | signup/patient | Match |
|--------|-------------|---------------|-------|
| Scroll strategy | flex-1 overflow-y-auto | flex-1 overflow-y-auto | ✅ |
| Content padding | px-6 py-4 | px-6 pb-4 | ⚠️ |
| CTA placement | fixed footer | fixed footer | ✅ |
| Header | custom back link | custom back link | ✅ |
```

---

### Step 17: Check 16 — Minimum Font Size Violation (HIGH)

Scan all page and component files for font sizes below 12px.

**Pass condition:**
- No text element uses a font size below 12px
- Only exception: badge/pill elements where space is extremely constrained (must be flagged as acknowledged exception)

**Fail condition:**
- Any element uses `text-[8px]`, `text-[9px]`, `text-[10px]`, `text-[11px]` (Tailwind)
- Any element uses `font-size: 8px/9px/10px/11px` (CSS)
- Any element uses `fontSize: '0.5rem'` or similar sub-12px value (CSS-in-JS)

**How to detect:**
```
1. Utility-class frameworks (Tailwind/UnoCSS):
   Grep: text-\[\d+px\] in all page/component files
   Filter results where the number is < 12
   Also check: text-\[0\.\d+rem\] where value < 0.75rem (12px)

2. CSS/SCSS:
   Grep: font-size:\s*\d+px in stylesheets
   Filter results where the number is < 12

3. CSS-in-JS (Styled Components/Emotion):
   Grep: fontSize:\s*['"]?\d+px in component files
   Filter results where the number is < 12

4. Count occurrences per file and total
5. Classify each occurrence:
   - VIOLATION: interactive/informational text (labels, timestamps, descriptions)
   - EXCEPTION: purely decorative (badge counters in constrained space)
```

**Why 12px minimum:**
- Chrome/Firefox default minimum font size setting is 10-12px
- Material Design minimum: 12sp (≈12px)
- Apple HIG minimum: 11pt (≈14.67px)
- WCAG 1.4.4 requires 200% zoom readability — sub-12px text becomes unreadable at any zoom level
- This is a universal standard, not project-specific

---

### Step 18: Check 17 — Arbitrary Font Size Values (MEDIUM)

Detect custom pixel-based font sizes that bypass the framework's standard typographic scale.

**Pass condition:**
- All font sizes use the framework's standard scale (e.g., Tailwind: `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`)
- No `text-[Npx]` or `text-[N.Nrem]` arbitrary values

**Fail condition:**
- Elements use `text-[13px]`, `text-[14px]`, `text-[15px]`, `text-[23px]` etc.
- Multiple arbitrary values indicate fragmented typography

**How to detect:**
```
1. Utility-class frameworks:
   Grep: text-\[\d+px\] in all page/component files
   List all unique arbitrary values and their occurrence counts
   Compare against the framework's built-in scale

2. CSS/SCSS:
   Grep: font-size in stylesheets
   List all unique values
   Check if they map to standard scale values or are custom

3. Report format:
   | Arbitrary Value | Standard Equivalent | Count | Files |
   |----------------|-------------------|-------|-------|
   | text-[13px]    | text-sm (14px)    | 18    | 5     |
   | text-[15px]    | text-base (16px)  | 5     | 4     |

4. Recommendation: map each arbitrary value to nearest standard equivalent
```

**Standard Tailwind font size scale reference:**

| Class | Size | Use Case |
|-------|------|----------|
| `text-xs` | 12px / 0.75rem | Captions, badges, timestamps |
| `text-sm` | 14px / 0.875rem | Body text, descriptions, labels |
| `text-base` | 16px / 1rem | Default body, input text |
| `text-lg` | 18px / 1.125rem | Section headings, card titles |
| `text-xl` | 20px / 1.25rem | Page headings (mobile) |
| `text-2xl` | 24px / 1.5rem | Page headings (desktop), hero |
| `text-3xl` | 30px / 1.875rem | Landing page titles |

---

### Step 19: Check 18 — Page Heading Size Inconsistency (HIGH)

Compare `<h1>` font sizes across all pages within the same app and hierarchy level.

**Pass condition:**
- All main page headings (`<h1>`) within the same app use the same font size
- Detail page headings may use a different (larger) size but must be consistent among detail pages
- Auth/landing pages may use a different size but must be consistent among auth pages

**Fail condition:**
- Page A `<h1>`: `text-xl` (20px), Page B `<h1>`: `text-2xl` (24px) — both are list pages
- Coach home: `text-xl`, Coach chat: `text-2xl` — same hierarchy level

**How to detect:**
```
1. Find all <h1> elements in page files
2. Extract font-size class/property from each
3. Group pages by category:
   - List pages (patients, exercises, notifications, chat list)
   - Detail pages (patient-detail, exercise-detail)
   - Auth pages (login, signup, forgot-password)
4. Within each group, all <h1> sizes must match
5. Report mismatches with file:line references

Output table:
| Page Category | Page | h1 Size | Expected | Match |
|--------------|------|---------|----------|-------|
| List | coach/home | text-xl | text-xl | ✅ |
| List | coach/chat | text-2xl | text-xl | ❌ |
| List | patient/exercise | text-2xl | text-xl | ❌ |
| Detail | patient/exercise-detail | text-2xl | text-2xl | ✅ |
| Auth | login | text-3xl | text-3xl | ✅ |
```

---

### Step 20: Check 19 — Font Size Variety Overload (MEDIUM)

Count the total number of unique font size values used across the app.

**Pass condition:**
- App uses 6 or fewer unique font size values
- Recommended scale: 4-6 sizes (e.g., `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`)

**Fail condition:**
- App uses 7+ unique font sizes
- Example: `text-[8px]`, `text-[9px]`, `text-[10px]`, `text-[11px]`, `text-xs`, `text-[13px]`, `text-sm`, `text-[14px]`, `text-[15px]`, `text-base`, `text-lg`, `text-xl`, `text-2xl`, `text-[23px]` = 14 unique sizes

**How to detect:**
```
1. Grep all font-size declarations across all page/component files
2. Normalize to pixel values:
   - Tailwind: text-xs=12px, text-sm=14px, text-base=16px, etc.
   - text-[Npx] = Npx
   - CSS: font-size: Npx = Npx
3. Deduplicate and count unique sizes
4. If count > 6, flag as MEDIUM severity
5. If count > 10, flag as HIGH severity

Report format:
Total unique font sizes: N (threshold: ≤6)

| Size | Pixel Value | Occurrences | Recommended Action |
|------|------------|-------------|-------------------|
| text-[10px] | 10px | 56 | → text-xs (12px) |
| text-[11px] | 11px | 20 | → text-xs (12px) |
| text-xs | 12px | 146 | ✅ Keep |
| text-[13px] | 13px | 18 | → text-sm (14px) |
| text-sm | 14px | 176 | ✅ Keep |
| text-base | 16px | 27 | ✅ Keep |
| text-lg | 18px | 17 | ✅ Keep |
| text-xl | 20px | 23 | ✅ Keep |
| text-2xl | 24px | 11 | ✅ Keep |
```

---

### Step 21: Check 20 — Form Label Font Size Inconsistency (MEDIUM)

Compare font sizes used on form field labels across all pages.

**Pass condition:**
- All `<label>`, `<Label>`, or label-like elements across the app use the same font size
- Size may differ between label categories (field labels vs section labels) but must be consistent within each category

**Fail condition:**
- Auth pages use `text-[11px]` for field labels
- Settings page uses `text-sm` for field labels
- Profile page uses `text-xs` for field labels

**How to detect:**
```
1. Find all label elements:
   - HTML: <label>, <Label> (shadcn/ui)
   - Vue: <el-form-item label>, <a-form-item label>
   - Angular: <mat-label>, <mat-form-field>
   - Generic: elements with "label" in className or role

2. Extract font-size class/property from each
3. Group by form type:
   - Auth forms (login, signup, forgot-password)
   - Settings/profile forms
   - Data entry forms (create/edit modals)
4. Compare within and across groups
5. Flag inconsistencies

Output table:
| Form Type | Page | Label Size | Expected | Match |
|-----------|------|-----------|----------|-------|
| Auth | login | text-[11px] | text-xs | ❌ |
| Auth | signup/patient | text-[11px] | text-xs | ❌ |
| Settings | patient/profile | text-sm | text-xs | ❌ |
| Data entry | create-user modal | text-sm | text-xs | ❌ |
```

---

### Step 22: Check 21 — Arbitrary Color Values Bypassing Design Tokens (HIGH)

Detect hardcoded color values that should use design tokens from the project's theme.

**Pass condition:**
- All color values reference design tokens (e.g., `text-primary-600`, `bg-neutral-500`, `border-success-200`)
- Only exception: third-party brand colors (social login buttons), inline SVG fills, or gradient stops with no token equivalent

**Fail condition:**
- Elements use hardcoded hex values like `text-[#154D71]` when `text-primary-600` exists in the theme
- Same hex value appears in multiple files instead of using a shared token
- Color values that are exact or near-exact matches to existing theme colors

**How to detect:**
```
1. Read the project's color theme definition:
   - Tailwind CSS 4: @theme inline { --color-* } in app.css or global CSS
   - Tailwind CSS 3: theme.extend.colors in tailwind.config.js/ts
   - CSS variables: :root { --color-* } in global CSS
   - CSS-in-JS: theme object in ThemeProvider
   Build a lookup table: { hex_value → token_name }

2. Find all arbitrary color values in components:
   - Tailwind: Grep for text-\[#|bg-\[#|border-\[#|from-\[#|to-\[#|via-\[#|ring-\[#|fill-\[#|stroke-\[#
   - CSS: Grep for color:\s*#|background(-color)?:\s*#|border(-color)?:\s*#
   - CSS-in-JS: Grep for color:\s*['"]#|backgroundColor:\s*['"]#

3. Cross-reference each arbitrary color against the theme lookup table:
   - EXACT MATCH → flag as HIGH (should definitely use token)
   - NEAR MATCH (within ~10% hue/lightness) → flag as MEDIUM (consider using closest token)
   - NO MATCH → classify as:
     - Brand color (social login, partner logos) → EXCEPTION (document but don't flag)
     - Semantic color (chart/event indicators, rating colors) → SUGGESTION (add to theme)
     - One-off color → WARNING (justify or add to theme)

4. Report format:
   | Arbitrary Color | Context | Matching Token | Action |
   |----------------|---------|---------------|--------|
   | bg-[#154D71] | calendar selected date | bg-primary-600 | REPLACE |
   | text-[#154D71] | calendar text | text-primary-600 | REPLACE |
   | bg-[#FEE500] | Kakao login button | N/A (brand) | EXCEPTION |
   | bg-[#2DC071] | survey event dot | N/A (semantic) | ADD TO THEME |
```

**Common sources of arbitrary colors:**
- AI design tools (AURA, v0, Figma export) generating hex values instead of tokens
- Copy-pasting from browser inspector
- Prototyping without design system awareness
- Social login/brand integration (legitimate exceptions)

---

### Step 23: Check 22 — Z-index Hierarchy Inconsistency (HIGH)

Audit all z-index values across the app and verify they follow a consistent layering hierarchy.

**Pass condition:**
- All z-index values follow a defined hierarchy with clear separation between layers
- Same-purpose elements (e.g., all modals) use the same z-index value
- No z-index value conflicts between layers (e.g., dropdown overlapping modal)

**Fail condition:**
- Different z-index values for same-purpose elements (e.g., modal A uses `z-40`, modal B uses `z-50`)
- Inverted hierarchy (dropdown `z-50` > modal `z-40`)
- Excessive z-index values (`z-[9999]`, `z-[999]`) indicating z-index wars
- Missing z-index on elements that need stacking (sticky headers, floating buttons)

**Recommended hierarchy:**

| Layer | Z-index | Purpose | Examples |
|-------|---------|---------|----------|
| Base | 0 | Default content flow | Page content, cards |
| Elevated | 10 | Slightly raised content | Floating action buttons, cards with shadow |
| Sticky | 20 | Sticky/fixed navigation | Sticky headers, bottom navigation |
| Dropdown | 30 | Dropdown menus, popovers | Select menus, date pickers, tooltips |
| Modal backdrop | 40 | Modal overlay background | Dark overlay behind modals |
| Modal | 50 | Modal dialogs | Confirmation dialogs, full-screen modals |
| Toast/Notification | 60 | Toast messages | Success/error toasts, snackbars |

**How to detect:**
```
1. Collect all z-index values:
   - Tailwind: Grep for z-\d+ and z-\[.+\] in all page/component files
   - CSS: Grep for z-index:\s*\d+ in stylesheets
   - CSS-in-JS: Grep for zIndex:\s*\d+ in component files

2. Build an inventory table:
   | File | Element | Z-index | Purpose | Layer |
   |------|---------|---------|---------|-------|
   | layout.tsx | sticky header | z-20 | navigation | Sticky |
   | bottom-nav.tsx | bottom nav | z-30 | navigation | Sticky |
   | modal.tsx | dialog | z-50 | modal | Modal |

3. Validate hierarchy:
   a. Group by purpose → check all same-purpose elements have same z-index
   b. Check ordering → lower layers must have lower z-index values
   c. Check for z-index wars → values > 100 are almost always wrong
   d. Check stacking context → elements inside position:relative parents
      create new stacking contexts; z-50 inside a z-10 parent is
      effectively lower than z-20 at the root level

4. Report conflicts:
   | Conflict | Element A | Element B | Issue |
   |----------|----------|----------|-------|
   | Same layer, different z | header (z-20) | bottom-nav (z-30) | Both are navigation but different z-index |
   | Inverted | dropdown (z-50) | modal (z-40) | Dropdown would overlay modal |
   | Z-index war | tooltip (z-[9999]) | - | Excessive value, indicates workaround |
```

**Stacking context awareness:**
Elements with `position: relative/absolute/fixed` + `z-index` create new stacking contexts. A `z-50` element inside a `z-10` parent will never overlay a `z-20` element at the root level. The audit must trace parent stacking contexts to detect false positives.

---

## Output Format

```markdown
# QA Layout Audit Report

**Date**: YYYY-MM-DD
**Scope**: All apps | {specific app/page}
**Pages Scanned**: {count}
**Total Issues Found**: {count}

---

## Layout Wrapper Context

| App | Layout File | Main Padding | Content Area |
|-----|------------|-------------|--------------|
| {app-name} | pages/admin/layout.tsx | px-6 py-4 | flex-1 overflow-auto |
| frontend | pages/layout.tsx | ... | ... |

---

## Page Inventory

| Page | Layout Type | Root Wrapper | Padding Negation | Title Position | Panel Separator | Rounded | Shared Components |
|------|------------|-------------|------------------|---------------|----------------|---------|-------------------|
| admin/home | sidebar+content | flex flex-col h-full -m-6 | ✅ -m-6 | ✅ Full-width above | border-t | ❌ None | SharedSidebar |
| admin/orders | sidebar+content | flex flex-col h-full -m-6 | ✅ -m-6 | ✅ Full-width above | border-t | ❌ None | SharedSidebar |
| admin/messages | sidebar+content | flex flex-col h-full -m-6 | ✅ -m-6 | ✅ Full-width above | border-t | ❌ None | SharedSidebar |
| admin/users | full-width | space-y-6 | ❌ None | ✅ Top | N/A | ❌ None | None |
| admin/products | full-width | space-y-6 | ❌ None | ✅ Top | N/A | ❌ None | None |

---

## Layout Type Groups

### Group: sidebar+content
Pages: home, orders, messages
Expected pattern:
- Root: `flex flex-col h-full -mx-6 -mt-4 -mb-8`
- Title: `px-6 py-4 shrink-0 flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between` (full-width above split)
- Split: `flex flex-1 min-h-0 border-t`
- Sidebar: shared component with `border-r`
- Content: `flex-1 p-4 ... overflow-auto`

### Group: full-width
Pages: users, products, orders, categories
Expected pattern:
- Root: `flex flex-col h-[calc(100vh-96px)] gap-4`
- Title: `flex items-center justify-between` with `text-xl font-bold tracking-tight`
- Toolbar: `flex items-center gap-2 flex-wrap`
- Search: `relative w-64` with icon at `left-3 top-1/2`
- Count: `text-sm text-muted-foreground` (no negative margins)
- Table: `Card className="flex-1 min-h-0 py-0"` with `admin-grid-table`

### Group: detail
Pages: user-detail, product-detail
Expected pattern:
- Root: `space-y-6` (uses layout padding directly)
- Title: `text-xl font-bold tracking-tight` with back button
- Content: `Card` with `CardHeader` + `CardContent`
- Labels: `text-xs font-medium text-muted-foreground uppercase tracking-wide`
- Values: `text-base font-medium text-foreground py-2`
- Label-value gap: `space-y-1` per field, `grid gap-6 md:grid-cols-2` for field grid

---

## Summary

| # | Check | Severity | Issues |
|---|-------|----------|--------|
| 1 | Title Placement | HIGH | X |
| 2 | Layout Padding | HIGH | X |
| 3 | Border Style | MEDIUM | X |
| 4 | Rounded Corners | MEDIUM | X |
| 5 | Sidebar Gap | HIGH | X |
| 6 | Component Wrapper | MEDIUM | X |
| 7 | Content Area Structure | MEDIUM | X |
| 8 | Header Action Alignment | LOW | X |
| 9 | Toolbar Gap | HIGH | X |
| 10 | Count Display Placement | MEDIUM | X |
| 11 | Search Input Width | LOW | X |
| 12 | Label-Value Typography | MEDIUM | X |
| 13 | Filter Component Type | HIGH | X |
| 14 | Scroll Container Missing | HIGH | X |
| 15 | Cross-Role Page Divergence | HIGH | X |
| 16 | Minimum Font Size Violation | HIGH | X |
| 17 | Arbitrary Font Size Values | MEDIUM | X |
| 18 | Page Heading Size Inconsistency | HIGH | X |
| 19 | Font Size Variety Overload | MEDIUM | X |
| 20 | Form Label Font Size Inconsistency | MEDIUM | X |
| 21 | Arbitrary Color Values Bypassing Tokens | HIGH | X |
| 22 | Z-index Hierarchy Inconsistency | HIGH | X |
| | **Total** | | **X** |

---

## HIGH Severity Issues

### Check 1: Title Placement Inconsistency

| Page | Layout Type | Title Position | Expected | Issue |
|------|------------|---------------|----------|-------|
| admin/messages | sidebar+content | Inside content panel | Full-width above split | Title only visible in right panel, not above sidebar |

**Risk**: Visual inconsistency — users see title in different positions when navigating between pages of the same type.
**Fix**: Move `<h1>` into a `px-6 py-4` wrapper ABOVE the flex container that holds sidebar + content.

### Check 2: Layout Padding Mismatch

| Page | Layout Type | Root Pattern | Expected | Issue |
|------|------------|-------------|----------|-------|
| admin/messages | sidebar+content | `h-[calc(100vh-96px)]` | `-m-6` | Uses manual height calc instead of padding negation |
| admin/home | sidebar+content | `flex flex-col h-full gap-6` | `-m-6` | Missing padding negation, sidebar has gap from app sidebar |

**Risk**: Sidebar alignment differs — some pages have gap between app sidebar and page content, others are flush.
**Fix**: Use consistent `-m-6` pattern matching the layout.tsx padding value.

### Check 5: Sidebar Gap Inconsistency

| Page | Sidebar | Gap from App Sidebar | Expected | Issue |
|------|---------|---------------------|----------|-------|
| admin/home | SharedSidebar | 24px (px-6 padding) | 0px (flush) | Layout padding creates visible gap |
| admin/orders | SharedSidebar | 0px | 0px | ✅ Correct |

**Risk**: The shared sidebar appears detached from the app sidebar on some pages but flush on others.

---

## MEDIUM Severity Issues

### Check 3: Border Style Inconsistency
...

### Check 4: Rounded Corner Inconsistency
...

### Check 6: Shared Component Wrapper Mismatch
...

### Check 7: Content Area Structure Mismatch
...

---

## LOW Severity Issues

### Check 8: Header Action Alignment
...

---

## Reference Pattern

> **Note:** The examples below use Tailwind CSS + React syntax for illustration. When auditing other stacks, translate to the equivalent CSS properties. The **structural rules** (title above split, consistent spacing, consistent separators) apply universally regardless of CSS framework or frontend framework.

The **canonical layout pattern** for sidebar+content pages (derived from the most consistent pages):

```tsx
// Page root — negate layout.tsx padding (px-6 pt-4 pb-8)
<div className="flex flex-col h-full -mx-6 -mt-4 -mb-8">

  {/* Title row — full width, restore padding, responsive wrap */}
  <div className="px-6 py-4 shrink-0 flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
    <h1 className="text-xl font-bold tracking-tight">
      {t('page.title')}
    </h1>
    {/* Optional: action buttons right-aligned */}
    <Button variant="outline">Action</Button>
  </div>

  {/* Panel split — edge to edge, border-t separator */}
  <div className="flex flex-1 min-h-0 border-t">

    {/* Left sidebar — shared component */}
    <SharedSidebar ... />

    {/* Main content — consistent padding */}
    <div className="flex-1 p-4 space-y-4 overflow-auto min-w-0">
      {/* Page-specific content */}
    </div>

  </div>
</div>
```

**Key rules:**
1. **`-mx-6 -mt-4 -mb-8`** must match layout.tsx main padding (`px-6 pt-4 pb-8`)
2. **Title** always ABOVE the panel split, never inside a sub-panel
3. **`border-t`** separates title from panels (no `border rounded-*` on the split container)
4. **Sidebar** has `border-r` (provided by the component itself)
5. **Content area** uses `p-4` padding, `overflow-auto`, `min-w-0` for flex truncation
6. **No `rounded-*`** on major panel containers (sharp edges for panel separation)
7. **Shared components** must NOT have `rounded-*` or `border` on their root — the page controls separation

---

## Reference Pattern: Full-Width List Pages

The **canonical layout pattern** for full-width list/table pages (no sidebar, e.g., products, users, orders):

```tsx
// Page root — uses layout.tsx padding directly, consistent height calculation
<div className="flex flex-col h-[calc(100vh-96px)] gap-4">

  {/* Title row — flex with actions right-aligned */}
  <div className="flex items-center justify-between">
    <div>
      <h1 className="text-xl font-bold tracking-tight">
        {t('page.title')}
      </h1>
    </div>
    {/* Optional: action buttons */}
    <div className="flex items-center gap-2">
      <Button variant="outline">Secondary</Button>
      <Button>+ Primary</Button>
    </div>
  </div>

  {/* Toolbar — single-line search + filters + bulk actions */}
  <div className="flex items-center gap-2 flex-wrap">
    {/* Search input — consistent width */}
    <div className="relative w-64">
      <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-muted-foreground" />
      <Input type="search" className="pl-9" ... />
    </div>

    {/* Optional: separator */}
    <div className="h-6 w-px bg-border" />

    {/* Filters — inline button group (NOT Select dropdown) for ≤5 options */}
    <div className="flex items-center gap-1 rounded-lg border p-1">
      <Button variant={filter === 'all' ? 'default' : 'ghost'} size="sm" className="relative h-7 px-3 text-xs" onClick={...}>All</Button>
      <Button variant={filter === 'active' ? 'default' : 'ghost'} size="sm" className="relative h-7 px-3 text-xs" onClick={...}>Active</Button>
      ...
    </div>

    {/* Bulk actions — right-aligned */}
    <div className="ml-auto flex items-center gap-2">
      <BulkAction ... />
    </div>
  </div>

  {/* Count display — consistent styling, no negative margins */}
  <p className="text-sm text-muted-foreground">
    Showing {filtered} of {total}
  </p>

  {/* Table card — fills remaining space */}
  <Card className="flex-1 min-h-0 py-0">
    <div className="flex-1 min-h-0 overflow-auto">
      <table className="admin-grid-table" style={{ tableLayout: 'fixed', width: totalWidth }}>
        <thead className="border-b bg-background sticky top-0 z-20">...</thead>
        <tbody>...</tbody>
      </table>
    </div>
  </Card>
</div>
```

**Key rules for full-width pages:**
1. **Container**: `flex flex-col h-[calc(100vh-96px)] gap-4` — consistent height and spacing
2. **Title row**: `flex items-center justify-between` — even for pages without action buttons (wrap h1 in div for consistency)
3. **Toolbar gap**: Always `gap-2` — NOT gap-3 or gap-4
4. **Search width**: Always `w-64` — NOT w-80 or w-72
5. **Count display**: `text-sm text-muted-foreground` with NO negative margins (`-mt-*`)
6. **Table card**: `Card className="flex-1 min-h-0 py-0"` with `admin-grid-table`
7. **Table header**: `border-b bg-background sticky top-0 z-20` — always include `border-b`
8. **Empty states**: `p-8 text-center text-muted-foreground`

---

## Recommendations

1. **[Priority 1]** Fix title placement — move all titles above the panel split for sidebar+content pages.
2. **[Priority 2]** Standardize padding negation — all sidebar+content pages must use `-m-6`.
3. **[Priority 3]** Fix sidebar gap — ensure shared sidebar is flush with app sidebar (no layout padding gap).
4. **[Priority 4]** Remove `rounded-*` from major panel containers — use `border-t` / `border-r` only.
5. **[Priority 5]** Standardize content area classes across same-type pages.
6. **[Priority 6]** Ensure shared component roots don't add their own `border` or `rounded-*` — let parent pages control.
7. **[Priority 7]** Standardize toolbar gap — all full-width pages use `gap-2` on filter toolbar.
8. **[Priority 8]** Standardize search input width — all full-width pages use `w-64`.
9. **[Priority 9]** Standardize count display — no negative margins, consistent placement below toolbar.
```

## Scanning Patterns Reference

Use the **Page File Patterns** table from `qa-shared/reference.md` to determine the correct file glob for each framework. The patterns below show **what to search for** — adapt the file extension and path patterns to the detected framework.

| What | Utility-Class Pattern (Tailwind/UnoCSS) | CSS-in-JS / Stylesheet Pattern | Where to Search |
|------|----------------------------------------|-------------------------------|-----------------|
| Page root wrapper | First container after `return (` — read `className` | Read styled-component or resolve class → stylesheet | Page files (framework-specific glob) |
| Spacing override | `-m-*`, `-mx-*`, `-my-*`, `-p-*` in className | `margin: -N` or `padding` overrides in CSS | Page files |
| Height calc | `calc\(100vh` in className or style | `height: calc(100vh` in CSS | Page files |
| Page title | `<h1` element | `<h1` element | Page files |
| Title wrapper | Parent container of `<h1>` | Parent container of `<h1>` | Page files |
| Panel split | `flex` + `flex-1` containers with sidebar child | `display: flex` + `flex: 1` in CSS | Page files |
| Border separator | `border-t`, `border-r`, `border-b` in className | `border-top`, `border-right` in CSS | Page files |
| Rounded containers | `rounded-*` in className | `border-radius` in CSS | Page + component files |
| Shared sidebar usage | `<Sidebar`, `<SidePanel`, `<FilterPanel`, `<NavigationMenu` | Same component names | Page files |
| Sidebar root styling | First container in sidebar component | First container in sidebar component | Component files matching `*Sidebar*`, `*SidePanel*` |
| Content area styling | `flex-1` sibling of sidebar | `flex: 1` sibling of sidebar | Page files |
| Header actions | `<Button`, `<button`, `<el-button`, `<v-btn`, `<button mat-button` | Same component names | Page files |
| Layout spacing | `p-*`, `px-*`, `py-*` on `<main>` or content wrapper | `padding` in CSS on main/content wrapper | Layout files (framework-specific glob) |
| Toolbar gap | `gap-*` on filter toolbar flex container | `gap` in CSS on toolbar | Page files |
| Overflow control | `overflow-hidden` on layout root | `overflow: hidden` in CSS | Layout files |
| Scroll container | `overflow-y-auto`, `overflow-y-scroll` | `overflow-y: auto/scroll` in CSS | Page files |
| Cross-role page pairs | Match filenames across role directories | Same approach | Page files |
| CTA placement | Fixed footer (shrink-0 + border) vs inline | `position: fixed/sticky` vs flow | Page files |
| Sub-12px font sizes | `text-[8px]`, `text-[9px]`, `text-[10px]`, `text-[11px]` | `font-size` < 12px in CSS | Page + component files |
| Arbitrary font sizes | `text-[Npx]` where N is any custom value | Custom `font-size` values not in standard scale | Page + component files |
| Page heading sizes | `text-xl`, `text-2xl`, `text-3xl` on `<h1>` elements | Heading `font-size` in CSS | Page files |
| Font size inventory | All `text-*` size classes | All `font-size` declarations | Page + component files |
| Form label sizes | `text-*` on `<label>`, `<Label>` elements | `font-size` on label elements | Page + component files |
| Arbitrary colors | `text-[#...]`, `bg-[#...]`, `border-[#...]` | `color: #...`, `background: #...` in CSS | Page + component files |
| Color theme tokens | `@theme inline { --color-* }` or `theme.extend.colors` | `:root { --color-* }` or theme config | Theme/config files |
| Z-index values | `z-\d+`, `z-[...]` | `z-index: \d+` in CSS | Page + component + layout files |

## Important Notes

- **Layout spacing values vary by app** — always read the layout file first to know the base spacing. Override/negation values must match.
- **Shared components may nest other shared components** — e.g., a wrapper component may contain a shared sidebar. Audit the nesting chain, not just direct usage.
- **Detail pages** (user-detail, product-detail) may legitimately differ from list pages — group them separately.
- **Multi-panel pages** (complex builders/editors) have their own pattern — don't compare with simple sidebar+content pages.
- **Only audit structural decoration** — border-radius on avatars/badges/buttons is fine. Only flag decoration on major layout containers.
- **Check all auto-detected frontend apps** — layout patterns may differ between apps but must be consistent WITHIN each app.
- **CSS framework adaptation** — The examples in this skill use utility-class syntax (e.g., Tailwind) for brevity. When auditing non-utility-class projects, resolve class names to their CSS property equivalents and compare those instead. The **checks are about visual consistency, not specific class names**.

## READ-ONLY Diagnostic

This skill is a READ-ONLY diagnostic. It does NOT modify any files. It scans, analyzes, and reports findings. All recommended fixes are presented as suggestions in the report for the developer to implement.
