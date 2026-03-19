---
name: qa-a11y
description: "Audit WCAG 2.2 accessibility - semantic HTML, ARIA, keyboard navigation, form a11y, visual contrast, dynamic content, touch targets"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Accessibility — WCAG 2.2 Compliance Audit

Analyze frontend pages and components for WCAG 2.2 accessibility compliance across semantic structure, ARIA, keyboard, forms, visual, dynamic content, and touch targets.

## Execution Mode

- **Standalone** (`/qa-a11y [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Heading hierarchy | **High** | Heading levels must not skip (h1→h3 without h2). Each page should have exactly one `<h1>` |
| 2 | Landmark regions | **High** | Page must have `<main>`, `<nav>`, `<header>`, `<footer>` landmarks. No content outside landmarks |
| 3 | Semantic elements vs div-soup | **Medium** | Interactive elements must use semantic HTML (`<button>`, `<a>`, `<input>`) not `<div onClick>` or `<span onClick>` |
| 4 | Image alt text | **High** | All `<img>` must have `alt` attribute. Decorative images use `alt=""`. Informative images have descriptive alt text |
| 5 | ARIA role correctness | **Medium** | No redundant roles on semantic HTML (e.g., `<button role="button">`). ARIA roles match widget patterns (tabs, accordion, dialog) |
| 6 | ARIA states and properties | **Medium** | Interactive widgets have required ARIA states (`aria-expanded`, `aria-selected`, `aria-checked`). `aria-describedby`/`aria-labelledby` reference existing IDs |
| 7 | Live regions | **High** | Dynamic status messages use `aria-live="polite"` or `aria-live="assertive"`. Toast notifications are announced to screen readers |
| 8 | Tab order | **High** | No positive `tabIndex` values (> 0). Interactive elements are reachable via Tab. Logical reading order matches visual order |
| 9 | Focus management | **Critical** | Modal open must trap focus. Modal close must return focus to trigger. Route changes must move focus to new content |
| 10 | Keyboard traps | **Critical** | No element traps keyboard focus without an escape mechanism. All interactive elements are keyboard-operable |
| 11 | Skip links | **Medium** | Skip-to-content link present for pages with navigation before main content |
| 12 | Form label association | **High** | Every `<input>`, `<select>`, `<textarea>` has a visible label via `<label htmlFor>`, `aria-label`, or `aria-labelledby` |
| 13 | Form error messaging | **High** | Invalid fields use `aria-invalid="true"` + `aria-describedby` linking to error message element |
| 14 | Required field indication | **Medium** | Required fields indicated visually and via `aria-required="true"` or `required` attribute |
| 15 | Fieldset and legend grouping | **Low** | Related form controls (radio groups, checkbox groups) wrapped in `<fieldset>` with `<legend>` |
| 16 | Color contrast indicators | **High** | Information not conveyed by color alone. Focus indicators visible (`:focus-visible` styles present) |
| 17 | Text sizing | **Medium** | Text resizable to 200% without loss of content. No fixed `px` font sizes that prevent scaling |
| 18 | Motion preferences | **Medium** | Animations respect `prefers-reduced-motion`. CSS includes `@media (prefers-reduced-motion: reduce)` |
| 19 | Route change announcements | **High** | SPA route changes announced to screen readers via `aria-live` region or document title update |
| 20 | Touch target size | **Medium** | Interactive touch targets minimum 24x24px (WCAG 2.2 Level AA). Recommended 44x44px. Adequate spacing between targets |

## Skill-Specific Patterns

### Accessibility Library Detection

| Signal | Library |
|--------|---------|
| `@axe-core/react` in package.json | axe-core React integration |
| `react-aria` or `@react-aria/*` | React Aria (Adobe) |
| `@radix-ui/*` | Radix UI primitives |
| `@headlessui/react` or `@headlessui/vue` | Headless UI (Tailwind) |
| `ark-ui` or `@ark-ui/*` | Ark UI |

### WCAG 2.2 Level Mapping

| Check | Level A | Level AA | Level AAA |
|-------|---------|----------|-----------|
| Heading hierarchy | Required | Required | Required |
| Image alt text | Required | Required | Required |
| Keyboard operability | Required | Required | Required |
| Focus management | Required | Required | Required |
| Color contrast (4.5:1 text) | — | Required | Required |
| Color contrast (7:1 text) | — | — | Required |
| Touch target 24x24 | — | Required | Required |
| Touch target 44x44 | — | — | Required |
