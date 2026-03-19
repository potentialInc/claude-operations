---
name: qa-layout
description: "Audit page layout consistency - title placement, panel structure, spacing, borders, rounded corners, sidebar alignment, toolbar gaps, search widths, count display placement, and typography hierarchy"
user-invocable: true
argument-hint: "[app-path or page-name]"
---

# QA Layout — Page Layout Consistency Auditor

Verify all pages within an app follow consistent layout patterns: title placement, panel structure, padding/margin, border styles, rounded corners, sidebar alignment, typography, color tokens, and z-index hierarchy.

## Execution Mode

- **Standalone** (`/qa-layout [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Title placement inconsistency | **High** | Page title (`<h1>`) is inside a sub-panel instead of spanning full width like sibling pages |
| 2 | Layout padding mismatch | **High** | Same-level pages use different padding negation patterns (`-m-6` vs none) |
| 3 | Border style inconsistency | **Medium** | Panel separators differ across pages (`border-t` vs `border` vs `border rounded-*`) |
| 4 | Rounded corner inconsistency | **Medium** | Same-role containers have `rounded-*` on some pages but not others |
| 5 | Sidebar gap inconsistency | **High** | Shared sidebar component has different spacing/gap from the app sidebar across pages |
| 6 | Shared component wrapper mismatch | **Medium** | Same component used across pages but parent wrapper classes differ significantly |
| 7 | Content area structure mismatch | **Medium** | Pages with same layout type (sidebar+content) use different flex/grid structures |
| 8 | Header action alignment inconsistency | **Low** | Title-row action buttons (export, refresh) positioned differently across pages |
| 9 | Toolbar gap inconsistency | **High** | Same-type pages use different gap values on the filter/search toolbar (e.g., `gap-2` vs `gap-4`) |
| 10 | Count display placement inconsistency | **Medium** | Result count text (`Showing X of Y`) positioned differently or uses inconsistent margins |
| 11 | Search input width inconsistency | **Low** | Search inputs use different widths across same-type pages (e.g., `w-64` vs `w-80`) |
| 12 | Label-value typography hierarchy | **Medium** | Detail/form pages have insufficient visual distinction between labels and values (font-size, font-weight, color) |
| 13 | Filter component type inconsistency | **High** | Same-type pages use different filter UI patterns — Select dropdown on one page vs inline button group on another |
| 14 | Scroll container missing | **High** | Content area inside overflow-hidden layout lacks `overflow-y-auto`, making long content unscrollable |
| 15 | Cross-role page structure divergence | **High** | Same-function pages across roles use different layout patterns (fixed footer vs scroll-inline CTA, different padding) |
| 16 | Minimum font size violation | **High** | Text elements use font sizes below 12px. 12px is the industry-standard minimum for readability |
| 17 | Arbitrary font size values | **Medium** | Custom pixel-based font sizes (e.g., `text-[13px]`) used instead of the framework's standard scale (`text-xs`/`text-sm`/`text-base`) |
| 18 | Page heading size inconsistency | **High** | Same-level page headings use different font sizes across the app |
| 19 | Font size variety overload | **Medium** | App uses more than 6 unique font size values. A well-structured typography system should use 4-6 sizes maximum |
| 20 | Form label font size inconsistency | **Medium** | Form labels across different pages use different font sizes |
| 21 | Arbitrary color values bypassing design tokens | **High** | Hardcoded hex/rgb color values used instead of design tokens when the project defines a color palette |
| 22 | Z-index hierarchy inconsistency | **High** | Z-index values do not follow a consistent hierarchy. Expected: base(0) → content(10) → sticky(20) → dropdown(30) → modal(40) → toast(50) |
| 23 | Fixed spacing on fullscreen mobile pages | **High** | Fullscreen pages use fixed px spacing instead of viewport-relative units (`vh`). On small screens content overflows; on large screens excessive empty space |
| 24 | Overflow text not truncated | **Low** | Long text in table cells, card titles, or list items overflows its container instead of being truncated with ellipsis |
