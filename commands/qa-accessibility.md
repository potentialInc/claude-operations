---
description: Analyze frontend components for WCAG 2.2 accessibility compliance and generate a comprehensive QA report covering semantic structure, ARIA usage, keyboard navigation, form accessibility, visual accessibility, and dynamic content
argument-hint: "<file-path> [--level A|AA|AAA] [--backend <backend-src-path>] [--depth full|shallow]"
---

# QA Accessibility (WCAG 2.2)

Analyze frontend pages and components for WCAG 2.2 accessibility compliance, audit semantic HTML structure, ARIA patterns, keyboard navigation, form accessibility, visual accessibility, and dynamic content handling, and generate a comprehensive QA report with test cases.

---

## Purpose

This command helps you:
1. **Discover semantic HTML structure** — Heading hierarchy, landmarks, proper HTML elements vs div-soup
2. **Analyze ARIA usage** — Roles, states, properties, live regions correctness and misuse detection
3. **Verify keyboard navigation** — Tab order, focus management, keyboard traps, skip links
4. **Audit form accessibility** — Label association, error announcement, required field indication, grouping
5. **Check visual accessibility** — Color contrast indicators, focus indicators, text sizing, motion preferences
6. **Analyze dynamic content** — aria-live regions, route change announcements, toast notifications, loading states
7. **Verify touch accessibility** — Target sizes (24x24 minimum, 44x44 recommended), spacing between targets
8. **Generate test cases** — Produce actionable WCAG 2.2 compliance checklists

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack accessibility tracing (e.g., server-rendered HTML)
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```bash
/qa-accessibility <file-path>
```

**Examples:**
```bash
# React page — default AA level
/qa-accessibility src/pages/Dashboard.tsx

# Enforce AAA compliance level
/qa-accessibility src/pages/Checkout.tsx --level AAA

# With backend tracing for server-rendered markup
/qa-accessibility frontend/app/pages/profile/index.tsx --backend backend/src

# Frontend-only shallow analysis
/qa-accessibility src/components/NavigationMenu.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--level A\|AA\|AAA` | No | WCAG 2.2 conformance level to audit against. Default: `AA` |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): trace through imports and related components. `shallow`: single file analysis |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection                │
│  - Validate file path and extension                          │
│  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)│
│  - Detect accessibility libraries (react-aria, Radix, etc.) │
│  - Detect a11y testing tools (axe-core, pa11y, jest-axe)    │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Semantic Structure Analysis                         │
│  - Heading hierarchy (h1→h2→h3, no skipping)                │
│  - Landmark regions (main, nav, header, footer, aside)      │
│  - Semantic elements vs div-soup detection                   │
│  - Tables, lists, and framework-specific patterns            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: ARIA Analysis                                       │
│  - Detect ARIA misuse (redundant roles on semantic HTML)     │
│  - Interactive widget patterns (tabs, accordion, dialog)     │
│  - Live regions and state attributes                         │
│  - Error description linkage via aria-describedby            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Keyboard Navigation Analysis                        │
│  - Tab order and tabIndex audit                              │
│  - Focus management (modal, route, dynamic content)          │
│  - Keyboard trap detection                                   │
│  - Skip links and roving tabindex patterns                   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Form Accessibility Analysis                         │
│  - Label association (htmlFor, aria-label, aria-labelledby)  │
│  - Error messaging (aria-invalid, aria-describedby)          │
│  - Required fields and fieldset/legend grouping              │
│  - Autocomplete attributes and validation timing             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Visual Accessibility Analysis                       │
│  - Color contrast indicators (inline styles, CSS variables)  │
│  - Focus indicator visibility (outline, box-shadow)          │
│  - Text sizing (rem/em vs px), motion preferences            │
│  - Dark mode contrast, zoom/reflow support                   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Dynamic Content & Route Changes                     │
│  - SPA route change announcements                            │
│  - Toast/notification accessibility                          │
│  - Loading states, infinite scroll, skeleton screens         │
│  - Real-time updates, time limits                            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 8: Save & Present Results                              │
│  - Save report to .claude-project/qa/                        │
│  - Display summary with issue counts and action items        │
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

Read the file path from `$ARGUMENTS`. Parse optional flags (`--level`, `--backend`, `--depth`).

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-accessibility <file-path> [--level A|AA|AAA] [--backend <backend-src-path>] [--depth full|shallow]
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

**1.3 Accessibility Library Detection:**

| Signal | Library |
|--------|---------|
| `@axe-core/react` in package.json | axe-core React integration |
| `react-aria` or `@react-aria/*` in package.json | React Aria (Adobe) |
| `@radix-ui/*` in package.json | Radix UI primitives |
| `@headlessui/react` or `@headlessui/vue` in package.json | Headless UI (Tailwind) |
| `ark-ui` or `@ark-ui/*` in package.json | Ark UI |
| `@reach/*` in package.json | Reach UI |
| `ember-a11y` in package.json | Ember A11y |
| `@angular/cdk/a11y` imports | Angular CDK A11y module |
| `vue-a11y-utils` in package.json | Vue A11y Utils |

**1.4 A11y Testing Tool Detection:**

| Signal | Tool |
|--------|------|
| `axe-core` in package.json or devDependencies | axe-core (runtime scanner) |
| `pa11y` in package.json or devDependencies | Pa11y (CLI/CI scanner) |
| `jest-axe` in package.json or devDependencies | jest-axe (unit test assertions) |
| `cypress-axe` in package.json or devDependencies | cypress-axe (E2E test assertions) |
| `@testing-library` in package.json | Testing Library (role-based queries encourage a11y) |
| `playwright` + `@axe-core/playwright` | Playwright axe integration |
| `eslint-plugin-jsx-a11y` in package.json | ESLint a11y linting |

**1.5 WCAG Level Determination:**

Default: `AA`. Override with `--level` flag.

| Level | Coverage |
|-------|----------|
| `A` | Minimum accessibility — most basic requirements |
| `AA` | Standard compliance — required by most regulations (ADA, EN 301 549) |
| `AAA` | Enhanced accessibility — highest conformance, includes stricter contrast (7:1) and more |

**1.6 Backend Detection** (if `--depth full`):

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
[ ] Accessibility libraries detected (or none found)
[ ] A11y testing tools detected (or none found)
[ ] WCAG level determined (A / AA / AAA)
[ ] Backend framework detected (or --depth shallow acknowledged)
```

---

### Step 2: Semantic Structure Analysis

Parse the component/page source and analyze all semantic HTML patterns.

**2.1 Heading Hierarchy:**

Check heading levels for logical order without skipping:

| Check | Description | Severity |
|-------|-------------|----------|
| Single `<h1>` per page | Page has exactly one `<h1>` (or zero in components — page-level only) | WARNING |
| No skipped levels | `<h1>` → `<h2>` → `<h3>` without jumping from `<h1>` to `<h3>` | WARNING |
| Heading inside landmark | All headings should be inside a landmark region | INFO |
| No empty headings | `<h2></h2>` or heading with only whitespace | CRITICAL |
| No headings used for styling | `<h3>` used for font size only, not semantic structure | WARNING |

Detection patterns:
- JSX/TSX: `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>` elements
- Vue: `<h1>` through `<h6>` in `<template>` block
- Dynamic headings: `<Heading level={n}>` or `:is="'h' + level"` — note as dynamic, verify range

**2.2 Landmark Regions:**

| Landmark | HTML Element | ARIA Role | Required? |
|----------|-------------|-----------|-----------|
| Banner | `<header>` (page-level) | `role="banner"` | Required for pages |
| Navigation | `<nav>` | `role="navigation"` | Required if navigation present |
| Main | `<main>` | `role="main"` | Required for pages |
| Footer | `<footer>` (page-level) | `role="contentinfo"` | Required for pages |
| Complementary | `<aside>` | `role="complementary"` | If sidebar present |
| Search | `<search>` or `<form role="search">` | `role="search"` | If search present |
| Form | `<form>` with `aria-label` or `aria-labelledby` | `role="form"` | Named forms only |

Check:
- Multiple same-type landmarks must be differentiated with `aria-label` (e.g., two `<nav>` elements)
- No duplicate landmarks without labels: two `<nav>` without `aria-label` is a WARNING

**2.3 Semantic Elements vs Div-Soup:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| `<div onClick={...}>` without `role` and `tabIndex` | Non-interactive element used as button | CRITICAL |
| `<div>` wrapping list items instead of `<ul>/<li>` | Styled div list instead of semantic list | WARNING |
| `<span>` used as heading for styling | Semantic heading missing | WARNING |
| `<div>` used for table layout | Use `<table>` or `role="table"` | WARNING |
| `<a>` without `href` used as button | Use `<button>` instead or add `role="button"` | WARNING |
| `<img>` without `alt` attribute | Missing alternative text | CRITICAL |
| `<img alt="">` on informative images | Decorative alt on meaningful image | WARNING |
| `<img>` decorative without `alt=""` or `role="presentation"` | Decorative image announced by screen reader | INFO |

**2.4 Table Semantics:**

| Check | Description | Severity |
|-------|-------------|----------|
| `<th>` with `scope` | Column headers have `scope="col"`, row headers have `scope="row"` | WARNING |
| `<caption>` present | Table has a visible or `sr-only` caption describing its purpose | WARNING |
| `<thead>` / `<tbody>` | Table structure uses `<thead>` and `<tbody>` sections | INFO |
| Layout table | Table used for layout should have `role="presentation"` | WARNING |

**2.5 List Semantics:**

| Check | Description | Severity |
|-------|-------------|----------|
| `<ul>` / `<ol>` for lists | Lists use semantic list elements, not styled `<div>` groups | WARNING |
| `<dl>` for key-value pairs | Definition lists use `<dl>` / `<dt>` / `<dd>` | INFO |
| `.map()` renders `<li>` inside `<ul>` | React/Vue list rendering uses semantic wrapper | WARNING |

**2.6 Framework-Specific Semantic Checks:**

| Framework | Check | Description |
|-----------|-------|-------------|
| React | Fragment overuse | `<>...</>` hiding semantic wrappers; `<Fragment>` where `<section>` is appropriate |
| Next.js | `layout.tsx` landmarks | Root layout has `<html>`, `<body>`, `<main>` landmark structure |
| Next.js | Page metadata | `metadata` export or `<Head>` with `<title>` for page titles |
| Vue | Dynamic components | `<component :is="...">` rendering — verify semantic output |
| Svelte | `{#each}` list wrapper | Ensure `{#each}` block is inside `<ul>` or `<ol>` |
| Angular | `ng-container` | Verify `<ng-container>` does not strip necessary semantic elements |

**Step 2 Checklist:**
```
[ ] Heading hierarchy analyzed (levels, order, gaps)
[ ] Landmark regions identified and validated
[ ] Semantic elements vs div-soup patterns detected
[ ] Tables checked for <th>, scope, caption
[ ] Lists checked for proper <ul>/<ol>/<dl> usage
[ ] Framework-specific semantic patterns checked
[ ] Images checked for alt text
```

---

### Step 3: ARIA Analysis

Audit all ARIA attribute usage for correctness and completeness.

**3.1 ARIA Misuse Detection:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| `role="button"` on `<button>` | Redundant role on semantic element | INFO |
| `aria-label` on non-interactive `<div>` | aria-label has no effect on non-interactive elements | WARNING |
| `role="link"` on `<a href>` | Redundant role on semantic element | INFO |
| `aria-hidden="true"` on focusable element | Screen reader skips it but keyboard still focuses | CRITICAL |
| `role="presentation"` on element with focusable children | Children lose semantic meaning | CRITICAL |
| `aria-label` on `<div>` without a role | Label not exposed without an interactive role | WARNING |

**3.2 Interactive Widget Patterns:**

For each custom interactive widget, verify complete ARIA pattern:

**Tabs:**
| Required | Attribute/Role | On Element |
|----------|---------------|------------|
| Yes | `role="tablist"` | Tab container |
| Yes | `role="tab"` | Each tab button |
| Yes | `role="tabpanel"` | Each tab content panel |
| Yes | `aria-selected="true/false"` | Each tab |
| Yes | `aria-controls="panel-id"` | Each tab → links to panel |
| Yes | `aria-labelledby="tab-id"` | Each panel → links to tab |

**Accordion:**
| Required | Attribute/Role | On Element |
|----------|---------------|------------|
| Yes | `<button>` or `role="button"` | Accordion trigger |
| Yes | `aria-expanded="true/false"` | Trigger button |
| Yes | `aria-controls="content-id"` | Trigger → links to content |
| Recommended | `role="region"` | Content panel (when expanded) |
| Recommended | `aria-labelledby="trigger-id"` | Content panel |

**Combobox (Autocomplete):**
| Required | Attribute/Role | On Element |
|----------|---------------|------------|
| Yes | `role="combobox"` | Input element |
| Yes | `role="listbox"` | Dropdown options container |
| Yes | `role="option"` | Each dropdown option |
| Yes | `aria-expanded="true/false"` | Combobox input |
| Yes | `aria-activedescendant="option-id"` | Combobox → currently focused option |
| Yes | `aria-autocomplete="list/both/none"` | Combobox input |

**Dialog / Modal:**
| Required | Attribute/Role | On Element |
|----------|---------------|------------|
| Yes | `role="dialog"` or `role="alertdialog"` | Dialog container |
| Yes | `aria-modal="true"` | Dialog container |
| Yes | `aria-labelledby="title-id"` | Dialog → links to title |
| Recommended | `aria-describedby="desc-id"` | Dialog → links to description |

**Menu:**
| Required | Attribute/Role | On Element |
|----------|---------------|------------|
| Yes | `role="menu"` or `role="menubar"` | Menu container |
| Yes | `role="menuitem"` | Each menu item |
| Recommended | `role="menuitemcheckbox"` | Checkable menu items |
| Recommended | `role="menuitemradio"` | Radio menu items |

**Tree View:**
| Required | Attribute/Role | On Element |
|----------|---------------|------------|
| Yes | `role="tree"` | Tree container |
| Yes | `role="treeitem"` | Each tree node |
| Yes | `aria-expanded="true/false"` | Expandable tree items |
| Recommended | `role="group"` | Nested tree item container |

**3.3 Live Regions:**

| Check | Description | Severity |
|-------|-------------|----------|
| `aria-live="polite"` | Used for non-urgent updates (search results count, status) | — |
| `aria-live="assertive"` | Used for urgent updates (errors, alerts) | — |
| `aria-live` on wrong element | Live region must exist in DOM before content changes | WARNING |
| Missing `aria-atomic` | If entire region should be announced, add `aria-atomic="true"` | INFO |
| `aria-relevant` | Specify `additions`, `removals`, `text`, `all` as needed | INFO |
| Too many live regions | Multiple `aria-live="assertive"` elements overwhelm screen reader | WARNING |

**3.4 State Attributes:**

| Attribute | Usage | Check |
|-----------|-------|-------|
| `aria-expanded` | Expandable sections, dropdowns, accordion | Toggles between `true` and `false` |
| `aria-selected` | Tabs, listbox options, tree items | Only one selected in single-select contexts |
| `aria-checked` | Custom checkboxes, toggle switches | Reflects actual checked state |
| `aria-disabled` | Disabled interactive elements | Prefer HTML `disabled` on form controls |
| `aria-pressed` | Toggle buttons | Toggles between `true` and `false` |
| `aria-current` | Current page in nav, current step in wizard | Correct value: `page`, `step`, `location`, `date`, `true` |

**3.5 Error Description Linkage:**

| Check | Description | Severity |
|-------|-------------|----------|
| `aria-describedby` on fields | Links to error message element by ID | WARNING |
| `aria-errormessage` on fields | WCAG 2.2 preferred for error messages | INFO |
| Error element has `role="alert"` or `aria-live` | Error announcement to screen reader | WARNING |
| `aria-invalid="true"` set on invalid field | Screen reader announces invalid state | WARNING |

**Step 3 Checklist:**
```
[ ] ARIA misuse patterns detected (redundant, incorrect, conflicting)
[ ] All interactive widgets audited (tabs, accordion, combobox, dialog, menu, tree)
[ ] Live regions checked (polite vs assertive, atomic, relevant)
[ ] State attributes verified (expanded, selected, checked, disabled, pressed)
[ ] Error description linkage checked (aria-describedby, aria-invalid)
```

---

### Step 4: Keyboard Navigation Analysis

Verify all interactive elements are keyboard accessible with logical tab order.

**4.1 Tab Order Audit:**

| Check | Description | Severity |
|-------|-------------|----------|
| No positive `tabIndex` (> 0) | `tabIndex="1"`, `tabIndex="5"` disrupt natural order | CRITICAL |
| `tabIndex="0"` on custom interactive elements | Custom buttons/links must be in tab order | CRITICAL |
| `tabIndex="-1"` used correctly | Programmatically focusable but not in tab order | — |
| Logical tab flow | Tab order follows visual layout (left-to-right, top-to-bottom) | WARNING |
| Hidden content not focusable | Off-screen or hidden elements (`display:none`, `visibility:hidden`) not in tab order | WARNING |

**4.2 Focus Management:**

| Scenario | Expected Behavior | Severity if Missing |
|----------|-------------------|---------------------|
| Modal open | Focus moves to first focusable element or modal title | CRITICAL |
| Modal close | Focus returns to the trigger element that opened the modal | CRITICAL |
| Dynamic content added | Focus moves to new content or aria-live announces it | WARNING |
| Item deleted from list | Focus moves to logical next element (next item or parent) | WARNING |
| Route change (SPA) | Focus moves to `<main>` content or aria-live announces page | WARNING |
| Dropdown open | Focus moves to first option or stays on trigger with `aria-activedescendant` | WARNING |
| Error on form submit | Focus moves to first invalid field or error summary | WARNING |

Detection patterns:
- React: `useRef()` + `.focus()`, `autoFocus` prop, `FocusTrap` component
- Vue: `this.$refs.element.focus()`, `v-focus` directive
- Angular: `ViewChild` + `nativeElement.focus()`, `cdkTrapFocus`

**4.3 Keyboard Trap Detection:**

| Check | Description | Severity |
|-------|-------------|----------|
| Can Tab out of every component | No component traps Tab key indefinitely | CRITICAL |
| Modal has intentional focus trap | Focus should be trapped inside open modals | Expected behavior |
| Focus trap releases on close | When modal closes, focus trap is removed | CRITICAL |
| Escape key closes overlays | ESC key closes all modals, dropdowns, popovers | WARNING |

**4.4 Custom Keyboard Patterns:**

| Widget | Expected Keys | Check |
|--------|--------------|-------|
| Menu / Menubar | Arrow keys navigate items, Enter/Space activates, Escape closes | All keys handled |
| Tabs | Arrow keys switch tabs, Tab moves to panel, Home/End to first/last | All keys handled |
| Tree view | Arrow Up/Down navigates, Arrow Right expands, Arrow Left collapses | All keys handled |
| Combobox | Arrow Down opens list, Arrow keys navigate, Enter selects, Escape closes | All keys handled |
| Listbox | Arrow keys navigate, Space selects, Home/End to first/last | All keys handled |
| Slider | Arrow Left/Right adjusts value, Home/End to min/max | All keys handled |
| Toolbar | Arrow keys between items, Tab to move out of toolbar | Roving tabindex used |

**4.5 Skip Links:**

| Check | Description | Severity |
|-------|-------------|----------|
| Skip link present | First focusable element is "Skip to main content" link | WARNING |
| Skip link targets `<main>` | Link target is `#main-content` or `<main id="...">` | WARNING |
| Skip link visible on focus | Link is `sr-only` but becomes visible on keyboard focus | INFO |
| Multiple skip links | "Skip to navigation", "Skip to search" for complex layouts | INFO |

**4.6 Roving Tabindex:**

Composite widgets should use roving tabindex pattern:

| Widget | Pattern | Check |
|--------|---------|-------|
| Toolbar | Single Tab stop; Arrow keys move between tools | Only one `tabIndex="0"`, rest `-1` |
| Listbox | Single Tab stop; Arrow keys move between options | Only active option `tabIndex="0"` |
| Radio group | Single Tab stop; Arrow keys change selection | Only checked radio `tabIndex="0"` |
| Tab list | Single Tab stop; Arrow keys switch tabs | Only active tab `tabIndex="0"` |

**4.7 Framework-Specific Keyboard Checks:**

| Framework/Library | Check | Description |
|-------------------|-------|-------------|
| React | `FocusTrap` / `FocusLock` | Focus trap component wrapping modals |
| React | `useRef` + `.focus()` | Programmatic focus management |
| React | `autoFocus` prop | Acceptable on search pages; warning on other pages |
| @headlessui | Built-in keyboard handling | Verify not overridden or broken by customization |
| @radix-ui | Built-in keyboard handling | Verify not overridden or broken by customization |
| shadcn/ui | Radix primitives intact | Verify Radix keyboard handling not stripped during customization |
| Vue | `v-focus` directive | Custom focus directives working correctly |
| Angular | `cdkTrapFocus` | Angular CDK focus trap applied to modals |

**Step 4 Checklist:**
```
[ ] Tab order audited (no positive tabIndex, logical flow)
[ ] Focus management verified (modal, route change, delete, dynamic content)
[ ] Keyboard traps detected and assessed
[ ] Custom keyboard patterns verified (menus, tabs, trees, comboboxes)
[ ] Skip links checked (present, targets main, visible on focus)
[ ] Roving tabindex patterns verified for composite widgets
[ ] Framework-specific keyboard handling checked
```

---

### Step 5: Form Accessibility Analysis

Audit all form controls for proper labeling, error handling, and grouping.

**5.1 Label Association:**

| Check | Description | Severity |
|-------|-------------|----------|
| Every `<input>` has a label | `<label htmlFor="id">`, `aria-label`, or `aria-labelledby` | CRITICAL |
| Every `<select>` has a label | Same methods as input | CRITICAL |
| Every `<textarea>` has a label | Same methods as input | CRITICAL |
| Placeholder is NOT the label | `placeholder` alone does not count as accessible label | CRITICAL |
| Visible label present | `aria-label` without visible text is a lesser alternative | INFO |
| Label clicks focus the input | `<label htmlFor>` correctly targets the input `id` | WARNING |

Detection patterns:
- React: `<label htmlFor="email">` + `<input id="email">`, `aria-label="Email"`, `<Label>` components from UI libraries
- Vue: `<label :for="inputId">`, `v-bind:aria-label`
- Angular: `<label [for]="inputId">`, `[attr.aria-label]`

**5.2 Error Messages:**

| Check | Description | Severity |
|-------|-------------|----------|
| `aria-invalid="true"` on invalid fields | Screen reader announces field as invalid | WARNING |
| `aria-describedby` links to error | Error message element ID matches `aria-describedby` value | WARNING |
| `aria-errormessage` (WCAG 2.2) | Preferred method linking field to error message | INFO |
| Error has `role="alert"` | Error text announced immediately to screen reader | WARNING |
| Visual error indicator beyond color | Error uses icon + text, not just red border | WARNING |

**5.3 Required Fields:**

| Check | Description | Severity |
|-------|-------------|----------|
| `aria-required="true"` or `required` attribute | Screen reader announces field as required | WARNING |
| Visible required indicator | Asterisk (*) or "Required" text visible | WARNING |
| Required indicator explained | Form legend or note explains what * means | INFO |

**5.4 Field Grouping:**

| Check | Description | Severity |
|-------|-------------|----------|
| Radio groups in `<fieldset>` | `<fieldset>` + `<legend>` wrapping radio group | WARNING |
| Checkbox groups in `<fieldset>` | `<fieldset>` + `<legend>` wrapping related checkboxes | WARNING |
| Address fields in `<fieldset>` | Street, city, zip grouped with "Address" legend | INFO |
| `role="group"` alternative | Custom grouping with `role="group"` + `aria-labelledby` | Acceptable |

**5.5 Autocomplete Attributes:**

| Field Type | Expected `autocomplete` Value |
|-----------|-------------------------------|
| Full name | `name` |
| Email | `email` |
| Phone | `tel` |
| Street address | `street-address` |
| City | `address-level2` |
| Zip/Postal code | `postal-code` |
| Country | `country-name` |
| Credit card | `cc-number`, `cc-exp`, `cc-csc` |
| Password | `current-password` or `new-password` |
| Username | `username` |

Check: User data fields should have appropriate `autocomplete` attribute for autofill support.

**5.6 Validation Timing & Announcement:**

| Check | Description | Severity |
|-------|-------------|----------|
| Error on submit | Errors shown after form submission attempt | Standard behavior |
| Error on blur | Errors shown when leaving a field | Better UX if properly announced |
| Error summary at top | After submit, focus moves to error summary listing all errors | WARNING if missing on complex forms |
| Individual field errors | Each field shows its own error message inline | WARNING |
| Error count in summary | "3 errors found" announced to screen reader | INFO |

**5.7 Framework-Specific Form Checks:**

| Library | Check | Description |
|---------|-------|-------------|
| React Hook Form | `Controller` / `register` | Verify `aria-invalid`, `aria-describedby` passed to rendered input |
| Formik | `ErrorMessage` component | Check `ErrorMessage` renders accessible element linked to field |
| Conform | Built-in a11y | Verify Conform's accessibility integration is not overridden |
| Zod / Yup | Validation messages | Ensure validation error messages are surfaced to DOM accessibly |
| VeeValidate (Vue) | `ErrorMessage` | Check component renders accessible error linked to field |

**Step 5 Checklist:**
```
[ ] Every input/select/textarea has an accessible label
[ ] Placeholder-only labels flagged
[ ] Error messages linked via aria-describedby or aria-errormessage
[ ] aria-invalid set on invalid fields
[ ] Required fields indicated (aria-required + visible indicator)
[ ] Related fields grouped with fieldset/legend
[ ] Autocomplete attributes checked on user data fields
[ ] Validation timing and error announcement verified
[ ] Framework-specific form library patterns checked
```

---

### Step 6: Visual Accessibility Analysis

Analyze styles for contrast, focus visibility, text sizing, and motion preferences.

**Note:** Static analysis can only detect inline styles, CSS-in-JS, Tailwind classes, and CSS variable usage. Recommend runtime tools (axe DevTools, Lighthouse) for full color contrast audit.

**6.1 Color Contrast Indicators:**

| Level | Text Type | Minimum Ratio |
|-------|-----------|---------------|
| AA | Normal text (<18pt / <14pt bold) | 4.5:1 |
| AA | Large text (>=18pt or >=14pt bold) | 3:1 |
| AA | UI components and graphical objects | 3:1 |
| AAA | Normal text | 7:1 |
| AAA | Large text | 4.5:1 |

Detection patterns:
- Inline styles: `style={{ color: '#999', backgroundColor: '#fff' }}` — can estimate contrast
- Tailwind: `text-gray-400 bg-white` — map to approximate contrast values
- CSS variables: `var(--text-muted)` — note for manual review
- Hardcoded light gray text: `color: #ccc`, `text-gray-300` — likely fails contrast

| Check | Description | Severity |
|-------|-------------|----------|
| Light gray text on white | `color: #999` or lighter on white/light background | WARNING |
| Disabled state contrast | Disabled elements exempt from contrast but should be distinguishable | INFO |
| Link color vs surrounding text | Links must be distinguishable (not color alone — underline or 3:1 contrast) | WARNING |
| Placeholder text contrast | `placeholder` text often fails contrast requirements | WARNING |

**6.2 Focus Indicators:**

| Check | Description | Severity |
|-------|-------------|----------|
| `outline: none` without replacement | Interactive element with outline removed | CRITICAL |
| `outline: 0` without replacement | Same as above — focus ring removed | CRITICAL |
| `:focus-visible` styles present | Modern focus style targeting keyboard focus | Recommended |
| Custom focus ring with sufficient contrast | `box-shadow`, `outline` with >= 3:1 against adjacent colors | WARNING if too subtle |
| All interactive elements have visible focus | Buttons, links, inputs, selects all show focus ring | CRITICAL |
| Focus ring visible in both light and dark themes | If dark mode supported, focus ring visible in both | WARNING |

Detection patterns:
- CSS: `*:focus { outline: none }` — global focus removal, CRITICAL
- Tailwind: `focus:outline-none` without `focus:ring-*` — CRITICAL
- Tailwind: `focus-visible:ring-2 focus-visible:ring-blue-500` — correct pattern
- CSS-in-JS: `'&:focus': { outline: 'none' }` — CRITICAL

**6.3 Text Sizing:**

| Check | Description | Severity |
|-------|-------------|----------|
| `font-size` in `px` | Fixed pixel sizes prevent user text scaling | WARNING |
| `font-size` in `rem` or `em` | Relative units allow user scaling | Correct |
| Root `font-size` not fixed to `px` | `html { font-size: 16px }` prevents scaling — use `100%` or `62.5%` | WARNING |
| `line-height` without unit | Unitless `line-height: 1.5` is preferred | INFO |
| `max-width` in `ch` or `rem` | Content width scales with text size | INFO |

**6.4 Motion Preferences:**

| Check | Description | Severity |
|-------|-------------|----------|
| `prefers-reduced-motion` media query | CSS transitions/animations have reduced-motion alternative | WARNING |
| Framer Motion `reducedMotion` | `<MotionConfig reducedMotion="user">` or `useReducedMotion()` | WARNING if animations present but no reduced-motion handling |
| React Spring `skipAnimation` | `useReducedMotion()` from react-spring | WARNING |
| CSS `animation` without reduced-motion | `@keyframes` used without `@media (prefers-reduced-motion: reduce)` check | WARNING |
| Auto-playing video/carousel | Must have pause button; stop on reduced-motion preference | CRITICAL |
| Parallax scrolling | Must be disabled on reduced-motion preference | WARNING |

**6.5 Dark Mode:**

If dark mode is supported:

| Check | Description | Severity |
|-------|-------------|----------|
| Contrast in dark theme | Text contrast meets WCAG requirements in dark mode | WARNING |
| Focus ring in dark theme | Focus indicator visible against dark background | WARNING |
| Images in dark mode | Logos/icons have dark mode variants or sufficient contrast | INFO |
| CSS `prefers-color-scheme` | Dark mode respects system preference | INFO |

**6.6 Zoom and Reflow:**

| Check | Description | Severity |
|-------|-------------|----------|
| Content usable at 200% zoom | No overlapping text, no truncated content | WARNING |
| Content reflows at 400% zoom | No horizontal scroll required | WARNING |
| Content usable at 320px width | Responsive design works at minimum viewport | WARNING |
| No `user-scalable=no` in viewport | `<meta name="viewport" content="user-scalable=no">` prevents pinch zoom | CRITICAL |
| No `maximum-scale=1` | Limits zoom capability | WARNING |

**Step 6 Checklist:**
```
[ ] Color contrast indicators analyzed (inline styles, Tailwind, CSS variables)
[ ] Focus indicators verified on all interactive elements
[ ] outline:none without replacement flagged
[ ] Text sizing checked (rem/em vs px)
[ ] Motion preferences checked (prefers-reduced-motion, animation libraries)
[ ] Dark mode contrast verified (if applicable)
[ ] Zoom/reflow support checked (viewport meta, responsive design)
```

---

### Step 7: Dynamic Content & Route Changes

Analyze how dynamic content changes are communicated to assistive technology.

**7.1 Route Change Announcements:**

| Framework | Expected Pattern | Check |
|-----------|-----------------|-------|
| Next.js (App Router) | Built-in route announcer — verify presence and working | Check if custom override breaks announcement |
| Next.js (Pages Router) | Route change should announce new page title | Look for `Router.events` + aria-live or focus management |
| React Router | `NavigationAnnouncer` or custom aria-live on route change | Verify presence and correct page title |
| Vue Router | `router.afterEach()` hook with aria-live update or focus management | Verify route-level announcement |
| SvelteKit | Built-in announcer — verify not disabled | Check for route change handling |
| Angular Router | `NavigationEnd` event + title update + focus management | Verify router event subscription |

| Check | Description | Severity |
|-------|-------------|----------|
| Route change announced | Screen reader hears new page title on SPA navigation | WARNING |
| Focus moved on route | Focus moves to `<main>` or `<h1>` of new page | WARNING |
| Page title updated | `document.title` reflects current page | WARNING |

**7.2 Toast / Notification Accessibility:**

| Type | Expected ARIA | Role |
|------|--------------|------|
| Success message | `aria-live="polite"` | `role="status"` |
| Error message | `aria-live="assertive"` | `role="alert"` |
| Warning message | `aria-live="polite"` | `role="status"` |
| Info message | `aria-live="polite"` | `role="status"` |

| Check | Description | Severity |
|-------|-------------|----------|
| Toast container has `aria-live` | Toast mount point has live region configured | WARNING |
| Toast is keyboard dismissible | Escape key or close button to dismiss | WARNING |
| Toast pause on hover/focus | Auto-dismiss timer pauses when user hovers or focuses | INFO |
| Toast action focusable | If toast has an action button, it is keyboard accessible | WARNING |

Detection patterns:
- `react-hot-toast`, `react-toastify`, `sonner`, `@radix-ui/toast` — check library configuration
- Custom toast: look for `aria-live`, `role="alert"`, or `role="status"` on toast container

**7.3 Loading States:**

| Check | Description | Severity |
|-------|-------------|----------|
| `aria-busy="true"` on loading container | Indicates content is being updated | WARNING |
| `aria-live` region for "Loading..." | Screen reader hears loading state | WARNING |
| Skeleton screens | Hidden from screen readers with `aria-hidden="true"` or `aria-label="Loading"` | WARNING |
| Spinner without text | `<Spinner>` alone needs `aria-label="Loading"` or associated text | WARNING |
| Loading complete announcement | "Content loaded" or focus to new content when loading ends | INFO |

**7.4 Infinite Scroll:**

| Check | Description | Severity |
|-------|-------------|----------|
| New content announcement | Screen reader notified of new items loaded | WARNING |
| Focus management | Focus does not jump to top when new items load | WARNING |
| End of content | "No more items" announcement when all loaded | INFO |
| Alternative pagination | Non-infinite option for screen readers (load more button) | INFO |

**7.5 Skeleton Screens:**

| Check | Description | Severity |
|-------|-------------|----------|
| `aria-hidden="true"` on skeleton | Skeleton shapes not announced individually | WARNING |
| Container `aria-label="Loading"` | Loading state communicated to screen reader | WARNING |
| Replacement announced | When content replaces skeleton, screen reader is notified | INFO |

**7.6 Data Updates:**

| Check | Description | Severity |
|-------|-------------|----------|
| Real-time updates in `aria-live` | WebSocket data changes announced appropriately | WARNING |
| Update frequency | Too-frequent updates overwhelm screen reader; batch or throttle | WARNING |
| Visual indicator of update | Highlight or animation on changed data (with reduced-motion check) | INFO |

**7.7 Time Limits:**

| Check | Description | Severity |
|-------|-------------|----------|
| Session timeout warning | User warned before session expires | CRITICAL |
| Timeout extension | User can extend session timeout | WARNING |
| Timeout accessible | Warning dialog is accessible (focus, keyboard, announced) | WARNING |
| Auto-refresh with warning | Auto-refresh pages warn user before refresh | WARNING |

**Step 7 Checklist:**
```
[ ] Route change announcements verified for SPA navigation
[ ] Toast/notification accessibility checked (aria-live, role, keyboard dismiss)
[ ] Loading states have aria-busy or aria-live announcements
[ ] Infinite scroll has content loaded announcements
[ ] Skeleton screens hidden from or labeled for screen readers
[ ] Real-time data updates appropriately announced
[ ] Time limits have warning and extension mechanisms
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_Accessibility_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_Accessibility_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Accessibility Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_Accessibility_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_Accessibility_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React + Radix UI / Vue + Headless UI / etc.]
- WCAG Level: [A / AA / AAA]
- A11y libraries: [react-aria, @radix-ui / none detected]
- A11y testing tools: [axe-core, jest-axe / none detected]
- Semantic structure: [N] issues ([N] headings, [N] landmarks, [N] div-soup)
- ARIA correctness: [N] issues ([N] misuse, [N] missing patterns)
- Keyboard navigation: [N] issues ([N] traps, [N] focus management, [N] tab order)
- Form accessibility: [N] issues ([N] labels, [N] errors, [N] grouping)
- Visual accessibility: [N] issues ([N] focus visible, [N] contrast, [N] motion)
- Dynamic content: [N] issues ([N] route, [N] toast, [N] loading)
- Touch targets: [N] issues
- Issues found: [N] Critical, [N] Warning, [N] Info
- Test cases generated: [N]

### Action Required
1. [CRITICAL] [description]
2. [CRITICAL] [description]
3. [WARNING] [description]
...
```

**Step 8 Checklist:**
```
[ ] Report file saved to .claude-project/qa/
[ ] Test cases file saved (if separate)
[ ] Summary displayed with issue counts and action items
```

---

## Test Case Categories

### Category 1: Semantic Structure Tests (SEM-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| SEM-01 | Heading hierarchy | 1. Open screen reader heading list (NVDA: H key; VO: Rotor) | Headings appear in logical order h1→h2→h3 without skips |
| SEM-02 | Landmark navigation | 1. Navigate by landmarks (NVDA: D key; VO: Rotor) | All major sections reachable: banner, nav, main, footer |
| SEM-03 | Multiple nav landmarks | 1. Check two `<nav>` elements | Each has unique `aria-label` (e.g., "Main navigation", "Footer navigation") |
| SEM-04 | Image alt text | 1. Navigate to images with screen reader | Informative images have descriptive alt; decorative have `alt=""` |
| SEM-05 | List semantics | 1. Navigate to list content with screen reader | Screen reader announces "list, N items" for semantic lists |
| SEM-06 | Table semantics | 1. Navigate data table with screen reader | Column and row headers announced when moving between cells |

### Category 2: ARIA Pattern Tests (ARIA-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| ARIA-01 | Tab panel ARIA | 1. Navigate to tabs with screen reader 2. Use arrow keys | "Tab 1, selected, tab, 1 of 3" announced; panel content changes |
| ARIA-02 | Accordion ARIA | 1. Open screen reader 2. Navigate to accordion trigger | "Section title, collapsed, button" → after click "expanded" |
| ARIA-03 | Dialog ARIA | 1. Open modal 2. Check screen reader announcement | "Dialog, [title]" announced; content inside described |
| ARIA-04 | Combobox ARIA | 1. Focus autocomplete input 2. Type 3. Navigate options | "Combobox, expanded" → options announced with "option N of M" |
| ARIA-05 | Live region update | 1. Trigger status update (e.g., save) | Screen reader announces "Saved successfully" without focus change |
| ARIA-06 | Error announcement | 1. Submit form with invalid field | "Error: [message]" announced; field announced as "invalid" |

### Category 3: Keyboard Navigation Tests (KBD-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| KBD-01 | Tab through page | 1. Press Tab from top of page to bottom | All interactive elements reachable in logical order |
| KBD-02 | Skip link | 1. Press Tab once on page load | "Skip to main content" link appears; pressing Enter skips nav |
| KBD-03 | Modal focus trap | 1. Open modal 2. Press Tab repeatedly | Focus stays within modal; does not escape to background |
| KBD-04 | Modal focus return | 1. Open modal 2. Close with Escape | Focus returns to the button that opened the modal |
| KBD-05 | Menu keyboard | 1. Open dropdown menu 2. Press arrow keys | Arrow Down/Up moves between items; Enter activates; Escape closes |
| KBD-06 | No keyboard trap | 1. Tab through entire page | No component traps focus (except intentional modal trap) |
| KBD-07 | Custom component keyboard | 1. Tab to custom slider/tabs/tree 2. Use arrow keys | Widget-appropriate keyboard interaction works |
| KBD-08 | Button activation | 1. Tab to buttons 2. Press Space or Enter | Both Space and Enter activate the button |

### Category 4: Form Accessibility Tests (FORM-A11Y-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| FORM-A11Y-01 | Input labels | 1. Navigate to form fields with screen reader | Every field has an announced label |
| FORM-A11Y-02 | Required fields | 1. Navigate to required fields | Screen reader announces "required" for each required field |
| FORM-A11Y-03 | Error on submit | 1. Submit form with empty required fields | Errors announced; focus moves to first error or error summary |
| FORM-A11Y-04 | Error linked to field | 1. Focus on invalid field | Screen reader announces field name + error message |
| FORM-A11Y-05 | Radio group | 1. Navigate to radio group with screen reader | "Group label, radio button, option 1, 1 of 3" announced |
| FORM-A11Y-06 | Autocomplete | 1. Focus on name/email/phone fields | Browser autofill works correctly via `autocomplete` attribute |

### Category 5: Visual Accessibility Tests (VIS-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| VIS-01 | Focus visible | 1. Tab through all interactive elements | Every element shows visible focus ring |
| VIS-02 | Color not sole indicator | 1. View error states, success states, links | Color + text + icon used (not color alone) |
| VIS-03 | Text zoom 200% | 1. Set browser zoom to 200% 2. Navigate page | No overlapping text; all content readable; no truncation |
| VIS-04 | Reduced motion | 1. Enable prefers-reduced-motion 2. Trigger animations | Animations reduced or removed; essential motion preserved |
| VIS-05 | Viewport meta | 1. Check `<meta name="viewport">` | No `user-scalable=no` or `maximum-scale=1` |
| VIS-06 | Dark mode contrast | 1. Switch to dark mode 2. Check text and UI contrast | Contrast meets WCAG requirements in dark theme |

### Category 6: Dynamic Content Tests (DYN-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| DYN-01 | Route change | 1. Click internal link (SPA navigation) | New page title announced; focus managed appropriately |
| DYN-02 | Toast notification | 1. Trigger success/error action | Toast message announced by screen reader |
| DYN-03 | Loading state | 1. Trigger data loading | "Loading" announced; "Content loaded" when done |
| DYN-04 | Infinite scroll | 1. Scroll to load more content | New items loaded; screen reader notified |
| DYN-05 | Session timeout | 1. Let session approach timeout | Warning dialog appears with option to extend |

### Category 7: Screen Reader Tests (SR-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| SR-01 | Full page read | 1. Start screen reader continuous reading | All content read in logical order; nothing skipped or repeated |
| SR-02 | Icon-only buttons | 1. Navigate to icon-only buttons | Screen reader announces button purpose (aria-label or sr-only text) |
| SR-03 | Truncated text | 1. Navigate to truncated content | Full text accessible (tooltip keyboard accessible or expandable) |
| SR-04 | Multi-language content | 1. Navigate to content in different language | `lang` attribute present; screen reader switches pronunciation |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Semantic Structure | 15% | Heading hierarchy, landmarks, semantic elements, image alt text |
| ARIA Correctness | 15% | Proper roles, states, properties, live regions, widget patterns |
| Keyboard Navigation | 20% | Tab order, focus management, keyboard traps, skip links, roving tabindex |
| Form Accessibility | 15% | Labels, errors, required indicators, grouping, autocomplete |
| Visual Accessibility | 15% | Color contrast indicators, focus visible, text sizing, motion preferences |
| Dynamic Content | 10% | Route announcements, toasts, loading states, live updates |
| Touch Targets | 10% | Minimum sizes (24x24), recommended sizes (44x44), spacing |

**Scoring rules** (see `qa-shared-reference.md` for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Screen reader testing evidence in codebase (test files using getByRole, etc.): +3
  - axe-core or similar integrated in test suite (jest-axe, cypress-axe): +3
  - prefers-reduced-motion implemented across all animations: +2
  - Skip link present and functional: +2
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
| No interactive elements found | Static content page with no forms or widgets | Report on semantic structure only; note limited scope |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| A11y library not detected | No known accessibility library in dependencies | Analyzes raw HTML/ARIA patterns; flags as higher risk |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| CSS not resolvable | CSS modules, styled-components, or external stylesheets | Notes as "styles unresolvable — manual contrast audit needed" |
| Dynamic ARIA values | ARIA attributes set by runtime JS logic | Notes as "dynamic ARIA — manual verification needed" |
| Component library abstraction | ARIA handled inside library component (e.g., Radix Dialog) | Assumes library provides correct ARIA; verifies usage props |

---

## Edge Cases Handled

- **Shadow DOM** — ARIA attributes may not cross shadow boundary; custom elements need internal ARIA or `ElementInternals`
- **Canvas / WebGL** — Completely inaccessible without fallback content; flag as CRITICAL if no `<canvas>` fallback text or `aria-label`
- **PDF viewers** — Embedded PDFs (`<iframe>`, `<embed>`, `<object>`) need text alternatives or accessible PDF source
- **Third-party widgets** — reCAPTCHA, social embeds, payment forms (Stripe Elements) may lack accessibility; flag for manual review
- **Virtualized lists** — Off-screen items must not be focusable; check that `tabIndex` is managed dynamically (set to `-1` when virtualized out of view)
- **Drag and drop** — Alternative keyboard method required (move up/down buttons, keyboard shortcuts); flag CRITICAL if drag-only
- **Data visualization (charts)** — Screen reader alternatives required: data table, `aria-label` description, or long description link
- **Autofocus on page load** — May disorient screen reader users; acceptable for search-focused pages, WARNING elsewhere
- **Portaled content** — React portals, Vue `<Teleport>`, Angular CDK overlay may break tab order or landmark containment
- **Scroll-linked animations** — Must respect `prefers-reduced-motion`; parallax and scroll-triggered animations checked
- **Custom scrollbars** — May hide content from keyboard-only users if scroll container is not keyboard focusable
- **Multi-language content** — `lang` attribute on elements with different language from page `<html lang>`; affects screen reader pronunciation
- **Icon-only buttons** — Must have `aria-label` or visually hidden text; SVG icons need `aria-hidden="true"` when label is external
- **Truncated text** — Full text must be accessible: tooltip must be keyboard accessible (not hover-only), or expandable text pattern
- **Data tables vs layout tables** — Layout tables must have `role="presentation"` to prevent screen reader table navigation
- **Error summary pattern** — After form submission with errors, focus should move to error summary at top listing all field errors
- **Nested interactive elements** — `<button>` inside `<a>` or `<a>` inside `<button>` is invalid HTML; screen readers behave unpredictably
- **CSS-only hover menus** — Hover-based navigation inaccessible to keyboard users; must have keyboard-triggered alternative
- **Emoji in content** — Screen readers announce emoji names; excessive emoji creates verbose reading experience

---

## Related Commands

- `/qa-input-fields` — Form validation (accessibility of form interactions and error handling)
- `/qa-modal-drawer` — Modal focus trap, escape handling, aria-modal, aria-labelledby
- `/qa-loading-error-empty` — Loading/error states need aria-live announcements and aria-busy
- `/qa-table-list` — Table semantics, column headers, scope attributes, row actions
- `/qa-back-navigation` — Route change screen reader announcements, focus management on navigation
- `/review-command` — Validate this command's structure and quality

---

## Tips

1. **Run axe-core first** — Automated tools catch approximately 30% of accessibility issues; this command supplements with code pattern analysis for the remaining 70% that requires manual review
2. **Test with keyboard only** — Tab through the entire page without using a mouse; any interaction that requires a mouse is a CRITICAL issue for keyboard-only users
3. **Use a screen reader for verification** — VoiceOver (Mac), NVDA (Windows, free), JAWS (Windows), TalkBack (Android) for real-world testing beyond code analysis
4. **Don't rely on color alone** — Errors need text + icon, not just red color; links need underline or 3:1 contrast ratio against surrounding text
5. **Check every custom component** — Standard HTML elements (`<button>`, `<input>`, `<select>`) are accessible by default; custom widgets need manual ARIA, keyboard handling, and focus management
6. **Focus visible is non-negotiable** — If a user cannot see where keyboard focus is, keyboard navigation is completely broken; this is always CRITICAL
7. **Live regions are tricky** — Too many `aria-live` announcements overwhelm screen reader users; too few leave them uninformed; test with a real screen reader to find the balance
8. **Touch targets affect everyone** — WCAG 2.2 requires 24x24px minimum targets (Level AA); 44x44px is recommended; larger targets reduce errors for all users, not just those with motor impairments
