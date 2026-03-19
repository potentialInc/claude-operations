---
name: qa-modal
description: "Audit modal, drawer, dialog, and bottom sheet lifecycle - focus management, scroll lock, keyboard handling, form safety, accessibility"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Modal — Modal / Drawer / Dialog Lifecycle Auditor

Analyze all modal, drawer, dialog, and bottom sheet components for focus management, scroll lock, keyboard handling, form-in-modal safety, and accessibility compliance.

## Execution Mode

- **Standalone** (`/qa-modal [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Missing close path | **Critical** | Modal open trigger has no matching close path (X button, overlay, ESC, cancel) |
| 2 | Missing focus trap | **High** | Custom modal with no focus trap — Tab key moves focus outside modal |
| 3 | Missing focus return | **Medium** | Focus does not return to trigger element when modal closes |
| 4 | Missing scroll lock | **High** | Body scroll not disabled when modal is open — background scrolls behind overlay |
| 5 | Missing iOS scroll fix | **Medium** | No `position: fixed` + scroll offset pattern for iOS body scroll lock |
| 6 | ESC key not handled | **High** | Custom modal has no ESC key close handler |
| 7 | Nested ESC order wrong | **Medium** | ESC closes parent modal instead of topmost nested modal |
| 8 | Overlay close without dirty check | **High** | Form modal closes on overlay/backdrop click without "Unsaved changes?" confirmation |
| 9 | Missing dirty check on close | **High** | Closing modal with modified form shows no unsaved changes confirmation |
| 10 | Premature close on submit | **High** | Modal closes before API response — error display lost |
| 11 | Missing double-submit prevention | **Medium** | Submit button not disabled during API call in modal form |
| 12 | State not reset on close | **High** | Form values, errors, or loading states not cleared when modal closes and reopens |
| 13 | Missing role="dialog" | **Medium** | Custom modal container missing `role="dialog"` attribute |
| 14 | Missing aria-modal | **Medium** | Modal missing `aria-modal="true"` attribute |
| 15 | Missing aria-labelledby | **Medium** | Modal not linked to its title via `aria-labelledby` |
| 16 | Missing open/close animation | **Low** | Modal appears/disappears instantly with no transition |
| 17 | No mobile adaptation | **Medium** | Desktop-sized dialog on mobile viewport — may be cut off or hard to dismiss |
| 18 | Nested modal z-index issue | **Medium** | Nested modals not stacking correctly — child modal behind parent overlay |
