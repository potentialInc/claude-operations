---
name: qa-buttons
description: Audit all buttons, links, and interactive elements for proper handlers, routes, loading states, and accessibility
user-invocable: true
argument-hint: "[module]"
---

# QA Buttons — Interactive Element Auditor

Scan all frontend components for buttons, links, and interactive elements. Verify proper click handlers, valid route targets, loading/disabled states, confirmation dialogs, and accessibility attributes.

## Execution Mode

- **Standalone** (`/qa-buttons [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Dead button | **Critical** | Button with no onClick handler and no form submission role |
| 2 | Broken route link | **High** | Link to route not defined in router config |
| 3 | Missing loading state | **High** | Async action button with no loading/disabled state during operation |
| 4 | Missing confirmation | **Medium** | Delete/destructive button with no confirmation dialog |
| 5 | Missing disabled state | **Medium** | Submit button not disabled when form is invalid |
| 6 | Empty href | **High** | Anchor with href="#" or empty/javascript href |
| 7 | Missing aria-label | **Medium** | Icon-only button with no accessible name |
| 8 | Non-button clickable | **Medium** | div/span with onClick but no role="button" or keyboard support |
| 9 | Duplicate action | **Low** | Same API call triggered from multiple buttons on same page |
| 10 | Missing tooltip | **Low** | Icon-only button with no title or tooltip for sighted users |
