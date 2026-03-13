---
name: qa-modal
description: "Audit modal, drawer, dialog, and bottom sheet lifecycle - focus management, scroll lock, keyboard handling, form safety, accessibility"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Modal / Drawer / Dialog

Analyze all modal, drawer, dialog, and bottom sheet components in a frontend page or component, verify focus management, scroll lock, keyboard handling, form-in-modal safety, and accessibility compliance, and generate a comprehensive QA report with test cases.

---

## Purpose

This skill helps you:
1. **Discover all overlay components** — Find every modal, drawer, dialog, alert dialog, bottom sheet, and popover in a page
2. **Verify open/close completeness** — Ensure every open trigger has a matching close path
3. **Audit focus management** — Check focus trap inside modals and focus return on close
4. **Verify scroll lock** — Ensure body scroll is disabled when overlays are open
5. **Check keyboard handling** — ESC key closes modal, Tab stays trapped, Enter submits forms
6. **Audit form-in-modal safety** — Dirty check on close, submit+close coordination, state reset
7. **Verify accessibility** — `role="dialog"`, `aria-modal`, `aria-labelledby`, `aria-describedby`
8. **Generate test cases** — Produce actionable QA test checklists

**Important:** This skill is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```bash
/qa-modal <page-or-component-file-path>
```

**Examples:**
```bash
# React page with shadcn Dialog
/qa-modal src/pages/UserManagement.tsx

# Vue page with Element Plus dialogs
/qa-modal pages/orders/index.vue

# Shallow analysis — only the given file
/qa-modal src/components/Settings.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--depth full\|shallow` | No | `full` (default): trace imported modal components. `shallow`: analyze only the given file |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection                │
│  - Validate file path and extension                          │
│  - Auto-detect frontend/backend directories                  │
│  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)│
│  - Detect UI component library (Radix/MUI/Ant/Headless/etc.)│
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Modal/Drawer Component Discovery                    │
│  - Find all Dialog, Modal, Drawer, Sheet, BottomSheet        │
│  - Extract open state, trigger, purpose                      │
│  - Detect custom overlay patterns                            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Open/Close Trigger Analysis                         │
│  - Map every open trigger to close paths                     │
│  - Verify onClose/onOpenChange handlers                      │
│  - Detect auto-open patterns (URL-driven, condition-based)   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Focus Trap & Scroll Lock Analysis                   │
│  - Verify focus stays inside modal when open                 │
│  - Check focus returns to trigger on close                   │
│  - Verify body scroll lock and iOS scroll fix                │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Keyboard & Overlay Behavior                         │
│  - ESC key handling (close, nested modal order)              │
│  - Overlay/backdrop click behavior                           │
│  - Nested modal z-index stacking                             │
│  - Animation/transition presence                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Form-in-Modal & State Reset & Accessibility         │
│  - Dirty check on close, submit+close coordination           │
│  - State reset on close (form values, error states)          │
│  - ARIA attributes and semantic role verification            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Generate QA Report                                  │
│  - Modal inventory, focus/scroll/keyboard matrices           │
│  - Form-in-modal audit, accessibility audit                  │
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

Read the file path from `$ARGUMENTS`. Parse optional flags (`--depth`).

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-modal <page-or-component-file-path> [--depth full|shallow]
```

**1.2 Project Structure Detection:**

Auto-detect frontend and backend directories by scanning the project root for common patterns:
- Look for `package.json`, `tsconfig.json`, `vite.config.*`, `next.config.*`, `nuxt.config.*`, `angular.json`, etc.
- Identify the frontend source directory (e.g., `src/`, `app/`, `pages/`, `components/`)
- Detect framework and apply corresponding patterns

**1.3 Framework Detection:**

Read the target file and project's `package.json`:

| Signal | Framework |
|--------|-----------|
| `import ... from 'react'` or JSX/TSX + `react` in package.json | React |
| `<template>` + `<script>` + `.vue` extension | Vue |
| `@Component` decorator + `@angular/core` in package.json | Angular |
| `.svelte` extension | Svelte |
| `<form>` + no framework imports + `.html` extension | Plain HTML |

**1.4 UI Component Library Detection:**

| Signal | Library |
|--------|---------|
| `@radix-ui/react-dialog` or `~/components/ui/dialog` | Radix / shadcn Dialog |
| `@radix-ui/react-alert-dialog` or `~/components/ui/alert-dialog` | Radix / shadcn AlertDialog |
| `~/components/ui/sheet` or `@radix-ui/react-dialog` with Sheet | shadcn Sheet (Drawer) |
| `@headlessui/react` + `Dialog` | Headless UI Dialog |
| `@mui/material` + `Dialog` / `Drawer` / `Modal` | MUI |
| `antd` + `Modal` / `Drawer` | Ant Design |
| `element-plus` + `ElDialog` / `ElDrawer` | Element Plus |
| `vuetify` + `v-dialog` / `v-bottom-sheet` | Vuetify |
| `@angular/cdk/dialog` or `@angular/material/dialog` | Angular CDK / Material |
| `bootstrap` + `Modal` | Bootstrap |
| `react-modal` | React Modal |
| `vaul` | Vaul (Drawer) |
| Native `<dialog>` element with `.showModal()` / `.show()` | HTML Native Dialog |
| `cmdk` or `@radix-ui/react-command` + dialog pattern | Command Palette (cmdk) |

**Step 1 Checklist:**
```
[ ] File path validated and file exists
[ ] File extension is supported
[ ] Frontend framework detected
[ ] UI component library detected (or custom modal pattern)
```

---

### Step 2: Modal/Drawer Component Discovery

Parse the component/page source and discover ALL overlay components.

**2.1 Framework-Specific Detection Patterns:**

**React (Radix / shadcn):**
- `<Dialog>` / `<DialogTrigger>` / `<DialogContent>` — modal dialog
- `<AlertDialog>` / `<AlertDialogTrigger>` / `<AlertDialogContent>` — confirmation dialog
- `<Sheet>` / `<SheetTrigger>` / `<SheetContent>` — side drawer
- `<Popover>` / `<PopoverTrigger>` / `<PopoverContent>` — popover (not full modal)
- State: `open` prop + `onOpenChange` callback, or uncontrolled via `<DialogTrigger>`

**React (Headless UI):**
- `<Dialog>` + `<Dialog.Panel>` — modal
- `<Transition>` wrapping `<Dialog>` — animated modal
- State: `open` prop + `onClose` callback

**React (MUI):**
- `<Dialog>` / `<Drawer>` / `<Modal>` — core components
- `<Backdrop>` — overlay
- State: `open` prop + `onClose` callback with `reason` parameter

**React (Ant Design):**
- `<Modal>` — modal with `open` / `onCancel` / `onOk`
- `<Drawer>` — drawer with `open` / `onClose`
- `Modal.confirm()` / `Modal.warning()` — imperative modals

**React (Custom):**
- `useState(false)` + conditional render: `{isOpen && <div className="overlay">...</div>}`
- `createPortal(modal, document.body)` — portal-based modal

**Vue (Element Plus):**
- `<el-dialog>` — `v-model` / `:visible` + `@close`
- `<el-drawer>` — `v-model` / `:visible` + `@close`

**Vue (Vuetify):**
- `<v-dialog>` — `v-model` + `persistent` prop
- `<v-bottom-sheet>` — `v-model`

**Angular (Material / CDK):**
- `MatDialog.open(Component, config)` — imperative
- `MatBottomSheet.open(Component, config)` — imperative
- `<ng-template>` with `cdkConnectedOverlay` — CDK overlay

**Svelte:**
- Custom modal with `{#if isOpen}` + overlay div
- `svelte-simple-modal` or `svelte-accessible-dialog`

**HTML Native `<dialog>` Element:**
- `<dialog>` element with `.showModal()` method — modal with built-in backdrop, ESC handling, focus trap
- `<dialog>` element with `.show()` method — non-modal dialog (no backdrop, no focus trap)
- State: `dialog.open` property, `close` event, `cancel` event
- Built-in features: ESC closes modal, `::backdrop` pseudo-element for overlay, focus trap automatic
- Note: `.showModal()` provides focus trap and scroll lock; `.show()` does NOT

**Command Palette (cmdk):**
- `<Command.Dialog>` from `cmdk` — command palette rendered as a dialog
- `<CommandDialog>` from shadcn/ui `~/components/ui/command` — shadcn wrapper around cmdk
- Keyboard: Ctrl+K / Cmd+K to open; ESC or click outside to close; arrow keys to navigate items
- State: `open` prop + `onOpenChange` callback (same as Radix Dialog)
- Note: keyboard interaction is list-based (up/down), not standard Tab-based focus trap

**2.2 Build Modal Registry:**

For each discovered overlay, record:

| Property | Description |
|----------|-------------|
| `modalId` | Auto-generated ID (MODAL-01, MODAL-02, ...) |
| `type` | `dialog` / `alertDialog` / `drawer` / `sheet` / `bottomSheet` / `popover` / `commandPalette` |
| `library` | Radix/shadcn / MUI / Ant Design / Headless UI / Element Plus / Custom |
| `purpose` | `form` / `confirmation` / `info` / `selection` / `navigation` |
| `openStateVar` | Variable controlling open state (e.g., `isOpen`, `showModal`) |
| `triggerElement` | Button/link that opens it |
| `hasForm` | Whether a form exists inside the modal |
| `sourceLocation` | `file:line` reference |

**2.3 Custom Modal Detection:**

For custom (non-library) modals, detect the pattern:
- `useState(false)` or `ref(false)` for open state
- Conditional render: `{isOpen && ...}` or `v-if="isOpen"`
- Overlay div: `position: fixed`, `inset: 0`, `bg-black/50` or `backdrop`
- Portal: `createPortal(...)`, `<Teleport to="body">`

Flag custom modals as needing manual verification for focus trap, scroll lock, and accessibility.

**Step 2 Checklist:**
```
[ ] All Dialog/Modal components discovered
[ ] All Drawer/Sheet/BottomSheet components discovered
[ ] All AlertDialog/confirmation dialogs discovered
[ ] Custom overlay patterns detected
[ ] Modal registry built with type, library, purpose, state variable
[ ] Forms inside modals identified
```

---

### Step 3: Open/Close Trigger Analysis

**3.1 Open Trigger Detection:**

For each modal, identify how it opens:

| Trigger Type | Pattern | Example |
|-------------|---------|---------|
| Button click | `onClick={() => setOpen(true)}` | `<Button onClick={() => setOpen(true)}>Edit</Button>` |
| Library trigger | `<DialogTrigger>` / `<SheetTrigger>` | `<DialogTrigger asChild><Button>Edit</Button></DialogTrigger>` |
| Programmatic | `setOpen(true)` called from handler | `onRowClick → setEditModal(true)` |
| URL-driven | `searchParams.get('modal')` | `?modal=edit&id=5` → opens edit modal |
| Auto-open | `useEffect` or `onMounted` condition | `useEffect(() => { if (showWelcome) setOpen(true) }, [])` |
| Imperative API | `Modal.confirm()` / `dialog.open()` | `Modal.confirm({ title: 'Delete?', onOk: ... })` |

**3.2 Close Trigger Detection:**

For each modal, identify ALL close paths:

| Close Path | Pattern | Required? |
|-----------|---------|-----------|
| Close button (X) | `<DialogClose>` / `<button onClick={onClose}>` | Yes |
| Overlay click | `onOverlayClick` / backdrop click | Configurable |
| ESC key | `onEscapeKeyDown` / library default | Yes |
| Cancel button | `<Button onClick={onClose}>Cancel</Button>` | For forms: Yes |
| Form submit success | `onSubmit → API success → setOpen(false)` | For forms: Yes |
| Programmatic | `setOpen(false)` from parent/external | Optional |

**3.3 Open/Close Completeness Check:**

For each modal:
- Does every open path have at least one close path?
- Is `onOpenChange` / `onClose` properly wired?
- For URL-driven modals: does closing the modal remove the URL param?
- For auto-open modals: is there a way to dismiss permanently (e.g., `localStorage` flag)?

**Step 3 Checklist:**
```
[ ] Open triggers identified for every modal
[ ] All close paths identified (X button, overlay, ESC, cancel, submit)
[ ] Open/Close completeness verified (every open has ≥1 close)
[ ] onOpenChange/onClose handler properly wired
[ ] URL-driven modals clean up URL params on close
[ ] Auto-open modals have dismiss mechanism
```

---

### Step 4: Focus Trap & Scroll Lock Analysis

**4.1 Focus Trap Verification:**

| Library | Built-in Focus Trap? | Manual Check Needed? |
|---------|---------------------|---------------------|
| Radix / shadcn Dialog | Yes (automatic) | No |
| Headless UI Dialog | Yes (automatic) | No |
| MUI Dialog/Modal | Yes (`disableEnforceFocus` to disable) | Check if disabled |
| Ant Design Modal | Yes (automatic) | No |
| Angular CDK Dialog | Yes (via `FocusTrapFactory`) | No |
| Element Plus Dialog | Yes (`trap-focus` prop, default true) | Check if disabled |
| Vuetify Dialog | Partial (focus on open, no strict trap) | May need check |
| Bootstrap Modal | Yes (automatic) | No |
| Custom modal | **No** | **MUST verify** |
| HTML Native `<dialog>` (.showModal()) | Yes (browser built-in) | No |
| HTML Native `<dialog>` (.show()) | **No** | **MUST verify if used as modal** |
| cmdk Command Dialog | Yes (uses Radix Dialog internally) | No |

For custom modals, check for:
- `<FocusTrap>` wrapper component (react-focus-trap, focus-trap-react)
- `inert` attribute on background content
- Manual `tabIndex` management
- `document.addEventListener('keydown', trapTab)` pattern

**4.2 Focus Return Verification:**

On modal close, does focus return to the element that opened it?

| Library | Auto-returns focus? | Manual Check |
|---------|--------------------|--------------|
| Radix / shadcn | Yes | No |
| Headless UI | Yes | No |
| MUI | Yes (to previously focused element) | No |
| Ant Design | Yes | No |
| Angular CDK | Yes (via `FocusTrap`) | No |
| Custom | **No** | Check for `triggerRef.current.focus()` in onClose |

**4.3 Scroll Lock Verification:**

| Library | Built-in Scroll Lock? | iOS Fix? |
|---------|----------------------|----------|
| Radix / shadcn | Yes (via `@radix-ui/react-remove-scroll`) | Yes |
| Headless UI | Yes (via `<Dialog>` component) | Partial |
| MUI | Yes (`disableScrollLock` to disable) | Check |
| Ant Design | Yes (automatic) | Partial |
| Angular CDK | Yes (`BlockScrollStrategy`) | Check |
| Element Plus | Yes (`lock-scroll` prop, default true) | Partial |
| Custom | **No** | **MUST verify** |
| HTML Native `<dialog>` (.showModal()) | Yes (browser built-in) | Yes (browser handles) |
| HTML Native `<dialog>` (.show()) | **No** | N/A |
| cmdk Command Dialog | Yes (uses Radix internals) | Yes |

For custom modals, check for:
- `document.body.style.overflow = 'hidden'` on open
- `document.body.style.overflow = ''` on close (restored)
- iOS fix: `position: fixed` + `top: -${scrollY}px` pattern on body
- Cleanup: is scroll restored if component unmounts while modal is open?

**Step 4 Checklist:**
```
[ ] Focus trap verified for each modal (built-in or manual)
[ ] Custom modals flagged if no focus trap detected
[ ] Focus return verified (returns to trigger element on close)
[ ] Scroll lock verified for each modal (body overflow hidden)
[ ] iOS scroll lock fix checked (if applicable)
[ ] Scroll position restored on close
```

---

### Step 5: Keyboard & Overlay Behavior

**5.1 ESC Key Handling:**

| Check | Assessment |
|-------|------------|
| ESC closes the modal | Library default or manual `onKeyDown` handler |
| ESC in nested modals closes ONLY the topmost | Check z-index stacking and event propagation |
| ESC does not trigger page-level actions | `event.stopPropagation()` in modal |
| ESC disabled for critical actions | `<AlertDialog>` typically blocks ESC; confirmation modals may intentionally disable |

Detection patterns:
- Radix/shadcn: `onEscapeKeyDown` prop (can prevent default)
- MUI: `onClose` with `reason === 'escapeKeyDown'`
- Ant Design: `keyboard` prop (default true)
- Element Plus: `close-on-press-escape` prop (default true)
- Custom: `addEventListener('keydown', e => { if (e.key === 'Escape') close() })`

**5.2 Overlay/Backdrop Click:**

| Check | Assessment |
|-------|------------|
| Overlay click closes info/selection modals | OK — expected behavior |
| Overlay click closes form modals without dirty check | WARNING — potential data loss |
| Overlay click disabled for confirmation dialogs | OK — prevents accidental dismiss |
| `<AlertDialog>` prevents overlay close by default | Verify library behavior |

Detection patterns:
- Radix: `onInteractOutside` / `onPointerDownOutside`
- MUI: `onClose` with `reason === 'backdropClick'`
- Ant Design: `maskClosable` prop
- Element Plus: `close-on-click-modal` prop
- Vuetify: `persistent` prop

**5.3 Nested Modal/Drawer Detection:**

Check for modals opened from within other modals:
- A button inside Modal A opens Modal B
- Both modals visible simultaneously (stacking)

Verify:
- Z-index ordering: Modal B above Modal A above overlay
- ESC closes Modal B first, then Modal A
- Closing Modal B does not close Modal A
- Each overlay layer has proper backdrop

**5.4 Animation/Transition Detection:**

| Library | Animation Pattern |
|---------|-------------------|
| Radix / shadcn | CSS `data-[state=open/closed]` animations |
| Headless UI | `<Transition>` / `<TransitionChild>` |
| MUI | `TransitionComponent` prop (Fade, Slide, Grow) |
| Ant Design | Built-in animation (configurable via `motion`) |
| Framer Motion | `<AnimatePresence>` + `<motion.div>` |
| Vue | `<Transition>` / `<TransitionGroup>` |
| Angular | `@angular/animations` |
| Custom | CSS `transition`, `@keyframes`, or no animation |
| HTML Native `<dialog>` | CSS `[open]` attribute selector + `::backdrop` transitions; `@starting-style` for entry animation (modern browsers) |

Check: is animation present for both open AND close? Does interaction wait for animation to complete?

**5.5 Responsive Modal Behavior:**

Check if modals adapt to viewport size:

| Pattern | Detection | Assessment |
|---------|-----------|------------|
| Desktop Dialog → Mobile Bottom Sheet | `vaul` Drawer on mobile, Dialog on desktop (via media query or responsive prop) | OK — adaptive UX |
| Sheet side variants | `<SheetContent side="left\|right\|top\|bottom">` — different animation direction | INFO — note side variant for testers |
| Full-screen on mobile | `DialogContent` with responsive `max-w-full h-full` or `fullScreen` prop on mobile | OK if intentional |
| Modal with stepper/wizard | Multi-step modal where "back" inside modal goes to previous step | NOTE — internal "back" is not route navigation |
| No mobile adaptation | Desktop-sized dialog on mobile viewport | WARNING — may be cut off or hard to dismiss |

**Step 5 Checklist:**
```
[ ] ESC key handling verified for each modal
[ ] Nested modal ESC order verified (topmost closes first)
[ ] Overlay/backdrop click behavior checked per modal purpose
[ ] Form modals with overlay close checked for dirty-check protection
[ ] Nested modal z-index stacking verified
[ ] Open/close animation presence detected
```

---

### Step 6: Form-in-Modal & State Reset & Accessibility

**6.1 Form-in-Modal Analysis:**

For each modal that contains a form (`hasForm: true`):

| Check | Description | Assessment |
|-------|-------------|------------|
| Dirty check on close | Closing modal with modified form shows "Unsaved changes?" | Present / Missing |
| Submit + close coordination | Successful submit → close modal (not before API responds) | Correct / Premature close |
| Error display | Validation errors shown inside modal (not behind it) | Present / Missing |
| Double-submit prevention | Submit button disabled during API call / loading indicator | Present / Missing |
| Loading state | Form shows loading during submission | Present / Missing |

Detection patterns for dirty check:
- `form.formState.isDirty` (react-hook-form) checked in `onClose`
- `form.isFieldsTouched()` (Ant Design Form) checked in `onCancel`
- Custom: `hasChanges` state compared before closing
- `window.confirm('Unsaved changes...')` or `<AlertDialog>` on close attempt

**6.2 State Reset on Close:**

When modal closes and reopens, is state clean?

| Check | Description | Assessment |
|-------|-------------|------------|
| Form values reset | `form.reset()` called on close, or `key` prop forces remount | Present / Missing |
| Error states cleared | Validation errors removed when modal closes | Present / Missing |
| Loading states cleared | `isSubmitting` reset to false | Present / Missing |
| Selected items cleared | Multi-select, checkbox states reset | Present / Missing / N/A |

Detection patterns:
- `useEffect(() => { if (!open) form.reset() }, [open])` — reset on close
- `key={modalKey}` prop that changes on open — force remount
- `form.reset(defaultValues)` in `onOpenChange` callback
- `onAfterClose` / `afterClose` callback (Ant Design) with reset logic

**6.3 Accessibility Verification:**

| Attribute | Required | Detection |
|-----------|----------|-----------|
| `role="dialog"` | Yes | On modal container (most libraries add automatically) |
| `aria-modal="true"` | Yes | On modal container |
| `aria-labelledby` | Yes | Points to modal title element's `id` |
| `aria-describedby` | Recommended | Points to modal description element's `id` |
| `aria-label` | Alternative | If no visible title, use `aria-label` |
| Confirmation button focus | Recommended | `AlertDialog` should auto-focus the confirm/cancel button |

For each modal, check:
- Library components: usually handle ARIA automatically — verify not overridden
- Custom modals: **must manually verify** all ARIA attributes

**Step 6 Checklist:**
```
[ ] Form-in-modal dirty check verified (confirmation on unsaved close)
[ ] Submit+close coordination verified (waits for API response)
[ ] Double-submit prevention verified (disabled button during submit)
[ ] State reset on close verified (form values, errors, loading cleared)
[ ] role="dialog" present on each modal
[ ] aria-modal="true" present on each modal
[ ] aria-labelledby pointing to modal title
[ ] aria-describedby present (if description exists)
```

---

### Step 7: Generate QA Report

Compile all analysis results into a structured report.

**Report Sections:**

1. **Header** — Component name, file path, framework, UI library, date, overall score, status
2. **Modal/Drawer Inventory** — Table of all overlays with type, library, purpose, open state
3. **Open/Close Completeness Matrix** — Per modal: open triggers, close paths, completeness
4. **Focus Trap & Scroll Lock Matrix** — Per modal: focus trap, focus return, scroll lock, iOS fix
5. **Keyboard Handling Matrix** — Per modal: ESC, Tab trap, Enter submit
6. **Overlay Click Behavior Matrix** — Per modal: overlay close enabled, dirty-check protection
7. **Form-in-Modal Audit** — Per form modal: dirty check, submit+close, double-submit, state reset
8. **Accessibility Audit** — Per modal: role, aria-modal, aria-labelledby, aria-describedby
9. **Issues Found** — All issues with severity (CRITICAL / WARNING / INFO)
10. **Test Cases** — Complete test case table
11. **Recommendations** — Prioritized action items

**Step 7 Checklist:**
```
[ ] Modal/Drawer Inventory table complete
[ ] Open/Close Completeness Matrix complete
[ ] Focus Trap & Scroll Lock Matrix complete
[ ] Keyboard Handling Matrix complete
[ ] Overlay Click Behavior Matrix complete
[ ] Form-in-Modal Audit complete
[ ] Accessibility Audit complete
[ ] Issues listed with severity levels
[ ] Test cases listed with expected results
[ ] Recommendations prioritized
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_ModalDrawer_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_ModalDrawer_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Modal/Drawer Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_ModalDrawer_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_ModalDrawer_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React + Radix/shadcn / Vue + Element Plus / etc.]
- Modals/Drawers found: [N] total ([N] dialog, [N] drawer, [N] alertDialog, [N] custom)
- Focus trap: [N]/[N] modals have focus trap
- Scroll lock: [N]/[N] modals have scroll lock
- ESC handling: [N]/[N] modals handle ESC key
- Forms in modals: [N] (dirty check: [N]/[N], state reset: [N]/[N])
- Accessibility: [N]/[N] modals have full ARIA compliance
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
║               QA Modal / Drawer / Dialog Report                    ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                    ║
║  File:          [file-path]                                        ║
║  Framework:     [React + Radix/shadcn UI]                          ║
║  Report Date:   [YYYY-MM-DD HH:MM]                                ║
║  Depth:         [full / shallow]                                   ║
║  Overall Score: [N]/100                                            ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  MODAL/DRAWER INVENTORY ([N] total)                                ║
║                                                                    ║
║  | # | Type        | Library | Purpose      | Has Form | State  | ║
║  |---|-------------|---------|--------------|----------|--------|  ║
║  | 1 | Dialog      | shadcn  | Edit User    | Yes      | open   |  ║
║  | 2 | AlertDialog | shadcn  | Delete Confirm| No      | open   |  ║
║  | 3 | Sheet       | shadcn  | Filters      | No       | open   |  ║
║  | 4 | Custom      | —       | Image Preview| No       | isOpen |  ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  FOCUS TRAP & SCROLL LOCK MATRIX                                   ║
║                                                                    ║
║  | Modal        | Focus Trap | Focus Return | Scroll Lock | iOS  |║
║  |--------------|------------|--------------|-------------|------|║
║  | Edit User    | ✅ built-in| ✅ built-in  | ✅ built-in | ✅   |║
║  | Delete Conf  | ✅ built-in| ✅ built-in  | ✅ built-in | ✅   |║
║  | Filters      | ✅ built-in| ✅ built-in  | ✅ built-in | ✅   |║
║  | Image Preview| ❌ MISSING | ❌ MISSING   | ❌ MISSING  | ❌   |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  KEYBOARD HANDLING MATRIX                                          ║
║                                                                    ║
║  | Modal        | ESC Close | Tab Trap | Enter Submit |           ║
║  |--------------|-----------|----------|--------------|           ║
║  | Edit User    | ✅        | ✅       | ✅           |           ║
║  | Delete Conf  | ❌ blocked| ✅       | ✅           |           ║
║  | Filters      | ✅        | ✅       | N/A          |           ║
║  | Image Preview| ❌        | ❌       | N/A          |           ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  FORM-IN-MODAL AUDIT                                               ║
║                                                                    ║
║  | Modal     | Dirty Check | Submit+Close | 2x Submit | Reset  | ║
║  |-----------|-------------|--------------|-----------|--------|  ║
║  | Edit User | ❌ MISSING  | ✅ waits API | ✅ disabled| ✅     |  ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ACCESSIBILITY AUDIT                                               ║
║                                                                    ║
║  | Modal        | role     | aria-modal | labelledby | describedby|║
║  |--------------|----------|------------|------------|------------|║
║  | Edit User    | ✅       | ✅         | ✅         | ❌         |║
║  | Delete Conf  | ✅       | ✅         | ✅         | ✅         |║
║  | Image Preview| ❌       | ❌         | ❌         | ❌         |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ISSUES FOUND                                                      ║
║                                                                    ║
║  [CRITICAL] MOD-01: Custom Image Preview modal has no focus trap. ║
║    Tab key escapes to background content while modal is open.     ║
║    Fix: Wrap with <FocusTrap> or use a library Dialog component.  ║
║                                                                    ║
║  [CRITICAL] MOD-02: Custom Image Preview has no role="dialog".    ║
║    Screen readers cannot identify this as a modal dialog.         ║
║    Fix: Add role="dialog" and aria-modal="true".                  ║
║                                                                    ║
║  [WARNING] MOD-03: Edit User form modal has no dirty check.       ║
║    Clicking overlay or ESC discards form changes without warning. ║
║    Fix: Check form.formState.isDirty before closing.              ║
║                                                                    ║
║  [WARNING] MOD-04: Image Preview has no scroll lock.              ║
║    Background page scrolls while preview overlay is open.         ║
║    Fix: Add body overflow:hidden when modal is open.              ║
║                                                                    ║
║  [INFO] MOD-05: Edit User Dialog missing aria-describedby.        ║
║    Fix: Add id to description text and reference via aria.        ║
║                                                                    ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  TEST CASES ([N] total)                                            ║
║    Focus/Scroll: [N] | Keyboard: [N] | Form: [N] | A11y: [N]    ║
║    [See full list in test cases file]                              ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  RECOMMENDATIONS                                                   ║
║  1. [Critical] Add focus trap to Image Preview custom modal        ║
║  2. [Critical] Add role="dialog" and ARIA attrs to Image Preview  ║
║  3. [Warning] Add dirty check to Edit User form modal              ║
║  4. [Warning] Add scroll lock to Image Preview modal               ║
║  5. [Info] Add aria-describedby to Edit User dialog                ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Test Case Categories

### Category 1: Focus & Scroll Tests (FOCUS-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| FOCUS-01 | Tab key stays in modal | 1. Open modal 2. Press Tab repeatedly | Focus cycles through modal elements only, never escapes to background |
| FOCUS-02 | Shift+Tab stays in modal | 1. Open modal 2. Focus first element 3. Shift+Tab | Focus wraps to last focusable element in modal |
| FOCUS-03 | Focus returns on close | 1. Click button to open modal 2. Close modal | Focus returns to the button that opened it |
| FOCUS-04 | Body scroll locked | 1. Open modal on a long page | Background page cannot scroll while modal is open |
| FOCUS-05 | Scroll restored on close | 1. Scroll down 2. Open modal 3. Close modal | Page is at same scroll position as before opening |
| FOCUS-06 | iOS scroll lock | 1. Open modal on iOS Safari | No background scroll; no position jump |

### Category 2: Keyboard Tests (KEY-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| KEY-01 | ESC closes modal | 1. Open modal 2. Press ESC | Modal closes |
| KEY-02 | ESC closes topmost nested modal | 1. Open Modal A 2. Open Modal B from A 3. Press ESC | Modal B closes; Modal A remains open |
| KEY-03 | ESC blocked on AlertDialog | 1. Open confirmation dialog 2. Press ESC | Dialog stays open (if configured to block ESC) |
| KEY-04 | Enter submits form in modal | 1. Open form modal 2. Fill form 3. Press Enter | Form submits (if submit button is focused or form handles Enter) |
| KEY-05 | Overlay click closes info modal | 1. Open info modal 2. Click backdrop | Modal closes |
| KEY-06 | Overlay click on form modal | 1. Open form modal with changes 2. Click backdrop | Unsaved changes warning shown (or overlay close disabled) |

### Category 3: Form-in-Modal Tests (FORM-MOD-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| FORM-MOD-01 | Close with unsaved changes | 1. Open form modal 2. Modify a field 3. Click X/overlay/ESC | "Unsaved changes" confirmation appears |
| FORM-MOD-02 | Submit and close | 1. Fill form 2. Click submit | Loading state → API success → modal closes → data refreshed |
| FORM-MOD-03 | Submit with server error | 1. Fill form 2. Submit (API returns error) | Error shown inside modal; modal stays open; can retry |
| FORM-MOD-04 | Double-click submit | 1. Fill form 2. Rapidly click submit twice | Only one API call; button disabled after first click |
| FORM-MOD-05 | Reopen after close | 1. Open modal, enter data 2. Close 3. Reopen | Form is clean (no stale data from previous session) |
| FORM-MOD-06 | Validation errors on reopen | 1. Open modal 2. Trigger validation errors 3. Close 4. Reopen | No validation errors visible on reopen |

### Category 4: Accessibility Tests (A11Y-MOD-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| A11Y-MOD-01 | Screen reader announces modal | 1. Open modal with screen reader | Announces dialog role and title |
| A11Y-MOD-02 | Screen reader reads description | 1. Open modal with description | aria-describedby content is announced |
| A11Y-MOD-03 | Background inert | 1. Open modal | Background content not reachable by screen reader |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Open/Close Completeness | 15% | Every open has close; triggers properly wired |
| Focus Trap Correctness | 15% | Focus stays in modal; returns on close |
| Scroll Lock | 10% | Body scroll disabled; restored on close |
| ESC Key Handling | 10% | ESC closes modal; correct in nested scenarios |
| Overlay Click Behavior | 10% | Configurable; appropriate for modal purpose |
| Form-in-Modal Safety | 15% | Dirty check, submit+close, double-submit prevention |
| State Reset on Close | 10% | Form values, error states cleared on reopen |
| Accessibility | 15% | role, aria-modal, aria-labelledby, aria-describedby |

**Scoring rules** (see `qa-shared-reference` skill for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Each modal with full accessibility: +2
  - Each form-in-modal with dirty check: +2
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
| No modals found | No overlay components in file | Warning shown; component may not use modals |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| UI library not detected | Custom implementation or uncommon library | Analyzes as custom modal; flags for manual review |
| Modal component imported but not rendered | Conditional or dynamic import | Notes as "imported but not rendered in this file" |
| Imperative modal API | `Modal.confirm()` pattern | Analyzes config object; limited focus/scroll analysis |

---

## Edge Cases Handled

- **Modal opened via URL params** — `?modal=edit&id=5`; checks URL cleanup on close and refresh behavior
- **Nested modals** — Modal A opens Modal B; z-index, ESC order, overlay stacking verified
- **Drawer with long content** — Scroll inside drawer vs page scroll; internal scroll should work while body is locked
- **Bottom sheet on desktop** — Notes if bottom sheet renders differently or converts to dialog on desktop
- **Modal with async content** — Data loading inside modal; checks for skeleton/spinner inside modal
- **Imperative modals** — `Modal.confirm()`, `dialog.open()` patterns; limited but analyzed
- **Portal rendering** — `createPortal`, `<Teleport>` verified for correct mount point
- **Modal on mobile viewport** — Keyboard opening changes viewport height; modal should not be pushed off-screen
- **iOS body scroll position jump** — `position: fixed` + `top` pattern detected
- **Chat/message list in modal** — Modal containing a scrollable message list must scroll to the latest (bottom) message on open. `scrollIntoView({ behavior: 'smooth' })` fails with many messages (animation stops midway). Use `container.scrollTop = container.scrollHeight` with `setTimeout` (150ms+) to wait for Dialog open animation to finish. Never use smooth scroll for initial load.
- **Animation interruption** — Clicking close during open animation; interaction during transition
- **Multiple modals of same type** — Same component rendered with different data via key prop; state isolation verified
- **Server component modals** — Framework-specific patterns (e.g., Next.js App Router: modal trigger in server component, modal in client component)
- **HTML Native `<dialog>` element** — `.showModal()` provides built-in focus trap, scroll lock, ESC handling; `.show()` does NOT
- **Command Palette (cmdk)** — `<Command.Dialog>` uses arrow keys for navigation instead of Tab; keyboard interaction is list-based
- **Responsive modal behavior** — Dialog on desktop converting to Bottom Sheet on mobile; detected via media query or responsive props
- **Sheet side variants** — `<SheetContent side="left|right|top|bottom">` affects animation direction and mobile usability
- **Modal with stepper/wizard** — Multi-step modal where internal "back" navigates to previous step, not route back
- **`@starting-style` CSS** — Modern CSS entry animation for native `<dialog>` elements; browser support varies

---

## Related Skills

- `qa-input-fields` — QA all input fields (useful for form-in-modal validation)
- `qa-back-navigation` — QA back navigation (modals may affect back button behavior)
- `qa-loading-error-empty` — QA loading/error/empty states (modals may load data)
- `qa-table-list` — QA table/list components
- `qa-permission-role` — QA permission/role access (modals may be role-gated)
- `review-command` — Validate this skill's structure and quality

---

## Tips

1. **Custom modals are the biggest risk** — Library modals handle focus trap, scroll lock, and accessibility automatically; custom `div` overlays do not
2. **Always check form-in-modal dirty check** — The #1 user complaint is losing form data by accidentally clicking the overlay or pressing ESC
3. **State reset is easily forgotten** — Opening a modal, entering data, closing, and reopening should show a clean form — not stale data
4. **Nested modals need manual testing** — Automated analysis can detect them, but ESC order and z-index stacking need visual verification
5. **AlertDialog should NOT close on ESC/overlay** — Confirmation dialogs are intentionally hard to dismiss; verify this is the case
6. **Combine with `qa-input-fields` skill** — Forms inside modals should also be analyzed for input validation coverage
7. **Check scroll lock on iOS specifically** — The most common bug: body still scrolls on iOS Safari even when `overflow: hidden` is set
8. **Re-run after adding new modals** — Every new modal/drawer should be analyzed for the full checklist
