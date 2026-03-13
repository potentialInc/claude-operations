---
name: qa-buttons
description: Audit all buttons, links, and interactive elements for proper handlers, routes, loading states, and accessibility
user-invocable: true
argument-hint: "[module]"
---

# QA Buttons - Interactive Element Auditor

## Purpose

Scan all frontend components for buttons, links, and interactive elements. Verify they have proper click handlers, valid route targets, loading/disabled states during async operations, and accessibility attributes. Detect dead buttons, broken navigation links, missing confirmation dialogs on destructive actions, and semantic HTML violations that break keyboard and screen reader access.

## Usage

```
/qa-buttons                   # Full audit (all frontend apps)
/qa-buttons products          # Single module only
/qa-buttons users             # Audit users module
```

When a module argument is provided, restrict scanning to paths matching that module name (e.g., `**/products/**/*.tsx`). When no argument is given, scan all `.tsx` files across all auto-detected frontend app directories.

## 10 Checks

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

## Execution Algorithm

You MUST follow these steps in order. Do NOT skip steps. Do NOT modify any files. This is a READ-ONLY diagnostic.

### Step 0: Context & Requirements Gathering (MANDATORY)

Before auditing, you MUST understand which pages/features are actively used.

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
```

Extract:
- Which frontend apps exist and their purpose
- Which pages/features are in production vs planned/deprecated
- Route structure and navigation expectations

**0b. Ask clarifying questions if needed:**

If you find pages/components that appear orphaned or not in routes, ask:

> "I found [component X] that isn't referenced in routes. Should I audit it or skip it as unused?"

Wait for the user's response. Do NOT flag orphaned components as bugs without confirmation.

**0c. Build scope list:**
```
IN SCOPE: [app1/pages, app2/pages, ...]
OUT OF SCOPE: [orphaned pages (reason), ...]
```

---

### Step 1: Component Inventory

Determine the scope of the audit and build the complete list of component files to scan.

If a module argument was provided:
- Set `SCOPE_LABEL` to the module name (e.g., "products")
- Use these glob patterns to find matching files:
  ```
  For each auto-detected frontend directory:
  Glob: {frontend-dir}/**/*{module}*/**/*.tsx
  Glob: {frontend-dir}/**/pages/*{module}*/*.tsx
  Glob: {frontend-dir}/**/components/*{module}*/**/*.tsx
  Glob: {frontend-dir}/**/*{module}*.tsx
  ```

If no argument was provided:
- Set `SCOPE_LABEL` to "All apps"
- Use these glob patterns:
  ```
  For each auto-detected frontend directory:
  Glob: {frontend-dir}/**/*.tsx
  ```

Exclude test files, storybook files, and type definition files:
- Skip files matching `*.test.tsx`, `*.spec.tsx`, `*.stories.tsx`, `*.d.ts`

For each file found, read it and extract:
- **Component name**: from the function/const declaration (e.g., `export function ProductList` or `export const ProductList`)
- **File path**: absolute path for reporting
- **Export type**: default export or named export
- **Whether it is a page component**: lives in a `pages/` directory

Record the total count as `COMPONENTS_SCANNED`. Build the master file list for all subsequent steps.

### Step 2: Extract Interactive Elements

Scan all scoped files for three categories of interactive elements: buttons, links, and clickable non-button elements.

**A) Buttons — Search patterns:**
```
Grep: <button|<Button|<IconButton|<LoadingButton|<SubmitButton
In: all scoped files (output_mode: content, -B 2, -A 5)
```

For each button found, extract and record:
- **Component name**: derived from the file
- **File path and line number**
- **Text content / children**: the visible label
- **onClick handler**: presence and what function it references
- **type attribute**: submit, button, or reset (default is "submit" inside forms, "button" outside)
- **disabled condition**: any `disabled={...}` prop
- **Loading state**: isPending, isLoading, isSubmitting, loading prop, or Spinner/Loader in children
- **className / variant**: check for `variant="destructive"`, `"danger"`, `"delete"` patterns
- **aria-label**: presence of `aria-label` or `aria-labelledby` attribute
- **Parent form context**: is it inside a `<form>` element? (check by reading surrounding lines)
- **Icon-only**: does the button contain only an icon component with no visible text?
- **Trigger wrapper**: is the button inside a `<DialogTrigger>`, `<PopoverTrigger>`, `<DropdownMenuTrigger>`, `<TooltipTrigger>`, or `<AlertDialogTrigger>`?

**B) Links — Search patterns:**
```
Grep: <a |<a>|<Link |<Link>|<NavLink |<NavLink>
In: all scoped files (output_mode: content, -B 1, -A 3)
```
```
Grep: navigate\(|router\.push\(|router\.replace\(|useNavigate|history\.push\(
In: all scoped files (output_mode: content, -B 2, -A 2)
```

For each link found, extract:
- **href or to attribute**: the target path (resolve static strings, note template literals)
- **Target**: `_blank`, `_self`, or none
- **Text content**: visible label
- **onClick handler**: if any additional handler exists
- **File path and line number**
- **Whether it is external**: starts with `http://`, `https://`, `mailto:`, `tel:`

**C) Clickable Non-Button Elements — Search patterns:**
```
Grep: <div[^>]*onClick|<span[^>]*onClick|<tr[^>]*onClick|<td[^>]*onClick|<li[^>]*onClick|<Card[^>]*onClick|<p[^>]*onClick
In: all scoped files (output_mode: content, -B 1, -A 3)
```

For each found, extract:
- **Element type**: div, span, tr, td, li, Card, etc.
- **onClick handler**: what function it calls
- **role attribute**: presence of `role="button"` or `role="link"`
- **tabIndex**: presence of `tabIndex={0}` or `tabIndex="0"`
- **onKeyDown / onKeyPress handler**: keyboard activation support
- **cursor style**: `cursor-pointer` in className
- **File path and line number**
- **Text content or purpose**

Build a structured inventory table per component with counts:
```
Component Inventory:
  ProductList.tsx   — Buttons: 3, Links: 2, Clickables: 1
  UserTable.tsx     — Buttons: 5, Links: 0, Clickables: 2
  ...
```

Record the total as `INTERACTIVE_ELEMENTS_FOUND`.

### Step 3: Route Inventory

Build the complete list of valid routes in all frontend applications.

**3a.** Find router configuration files:
```
For each auto-detected frontend directory:
Glob: {frontend-dir}/**/router*.tsx, {frontend-dir}/**/routes*.tsx, {frontend-dir}/**/App.tsx
```

**3b.** Within those files, extract all route definitions:
```
Grep: path:|path=|<Route
In: router configuration files (output_mode: content, -A 3)
```

Read each router file fully to understand nested route structures.

**3c.** Build the route list accounting for:
- **Nested routes**: parent path + child path concatenation (e.g., `/dashboard` + `/users` = `/dashboard/users`)
- **Dynamic segments**: `:id`, `:userId`, `:productId` etc. — these match any value
- **Lazy loaded routes**: routes using `React.lazy()` or dynamic `import()` — these are valid routes even though the component loads on demand
- **Index routes**: routes with `index` prop (match parent path exactly)
- **Catch-all / 404 routes**: `path="*"` routes — these match everything and should NOT be used to validate links
- **Optional segments**: routes with optional parameters

Record this as `ROUTE_INVENTORY` — the complete list of valid route patterns across all apps. Format:
```
ROUTE_INVENTORY:
  {app-name-1}:
    /                         (index)
    /login                    (public)
    /items                    (list)
    /items/:id                (detail, dynamic)
    ...
  {app-name-2}:
    /                         (index)
    /users                    (list)
    /users/:id                (detail, dynamic)
    ...
```

### Step 4: Mutation/Async Action Mapping

Find all async operations that buttons might trigger and map them to loading state variables.

**4a.** Search for React Query mutations:
```
Grep: useMutation
In: all scoped files (output_mode: content, -B 2, -A 8)
```
Extract the mutation variable name, the mutation function, and the API endpoint it calls.

**4b.** Search for async handlers:
```
Grep: async.*=>\s*\{|\.then\(|await\s+fetch|await\s+api\.|await\s+axios|await\s+http
In: all scoped files (output_mode: content, -B 2, -A 5)
```

**4c.** Map loading state variables to their mutations:
```
Grep: isPending|isLoading|isSubmitting|isFetching|\.status\s*===\s*['"]loading
In: all scoped files (output_mode: content, -B 1, -A 1)
```

Build `ASYNC_ACTION_MAP` — maps component to its async actions and their loading states:
```
ASYNC_ACTION_MAP:
  ProductForm.tsx:
    - mutation: useCreateItem → POST /items
      loadingVar: isPending ✅
    - mutation: useUpdateItem → PATCH /items/:id
      loadingVar: isPending ✅
  UserTable.tsx:
    - mutation: useDeleteUser → DELETE /users/:id
      loadingVar: none ❌
```

### Check 1: Dead Button (CRITICAL)

**Definition**: A button is "dead" if it has no mechanism to perform any action when clicked. The user clicks it and nothing happens.

**Detection logic — a button is dead if ALL of the following are true:**
1. No `onClick` handler prop
2. No `type="submit"` attribute (not a form submit button)
3. Not wrapped in a `<form>` element that would make it an implicit submit
4. No `form` attribute pointing to an external form by ID
5. Not rendered as a Link (e.g., `<Button as={Link}>` or `<Button asChild><Link>`)
6. Not wrapped in a Radix/shadcn trigger component

**Grep patterns to confirm:**
```
Grep: onClick=
Look in the button's props on the same line and adjacent lines.

Grep: type="submit"|type='submit'
Check for form submission role.

Grep: <form
Check surrounding context (read 20-30 lines above the button) for form wrapper.
```

**FALSE POSITIVES to exclude — do NOT flag these as dead:**
- Buttons inside `<DialogTrigger>`, `<PopoverTrigger>`, `<DropdownMenuTrigger>`, `<TooltipTrigger>`, `<AlertDialogTrigger>`, `<CollapsibleTrigger>`, `<AccordionTrigger>` — these are controlled by Radix UI and receive click handlers via the `asChild` pattern
- Buttons with `type="submit"` inside a form — they trigger form submission
- Buttons rendered by `<AlertDialogAction>` or `<AlertDialogCancel>` — built-in handlers
- Buttons with `asChild` prop that wrap a `<Link>` — they navigate
- Buttons with `form="formId"` attribute pointing to an external form
- Buttons marked as placeholder with TODO/coming soon comments

**Record per finding:**
- Component name
- Button text/content (or "icon-only" if no text)
- File path and line number
- Why it is considered dead (no onClick, no form context, no trigger wrapper)

### Check 2: Broken Route Link (HIGH)

**Definition**: A `<Link to="...">` or `navigate("...")` call that targets a route not defined in the router configuration.

**Detection steps:**
1. For each internal link found in Step 2 (links section), extract the target path string.
2. If the path is a template literal like `` `/users/${id}` ``, normalize to pattern `/users/:param` for matching.
3. If the path is a variable reference (not a string literal), record as "dynamic — cannot verify" and skip.
4. If the path starts with `http://`, `https://`, `mailto:`, `tel:`, or `#`, skip (external or anchor link).
5. If the path is a relative path (no leading `/`), resolve it against the component's page context if possible.
6. Match the extracted/normalized path against `ROUTE_INVENTORY` from Step 3.
7. Account for dynamic segments: `/users/123` matches route pattern `/users/:id`.
8. If no matching route is found, record as a broken link.

**Grep patterns:**
```
Grep: to="|to='|to={|navigate\("|navigate\('|navigate\(`
In: all scoped files (output_mode: content, -A 1)
```

**FALSE POSITIVES to exclude:**
- Lazy-loaded routes that may not appear in the static router config but are valid
- Catch-all routes (`path="*"`) that would match any path
- Hash links (`#section`) that reference in-page anchors
- External URLs starting with `http://` or `https://`
- Paths stored in environment variables or constants that resolve at runtime

**Record per finding:**
- Component name
- Link text content
- Target path (as written in code)
- Resolved pattern (after template literal normalization)
- Whether a matching route exists (yes/no/dynamic)
- File path and line number

### Check 3: Missing Loading State (HIGH)

**Definition**: A button that triggers an async operation (API call, mutation) but provides no visual feedback to the user during the operation. Users may double-click, think the app is frozen, or navigate away.

**Detection steps:**
1. For each button in the inventory, identify its onClick handler function name.
2. Trace the handler: does it call a mutation's `mutate()` or `mutateAsync()`? Does it call an async function? Does it dispatch a Redux thunk?
3. Cross-reference with `ASYNC_ACTION_MAP` from Step 4.
4. If the button triggers an async operation, check for loading indicators on the button:
   - `disabled={isPending}` or `disabled={isLoading}` or `disabled={mutation.isPending}`
   - Conditional children: `{isPending ? <Spinner /> : 'Save'}`
   - Loading prop: `loading={isPending}`
   - `<LoadingButton>` component (has built-in loading)
   - The button is inside a form with a global `isSubmitting` state applied

**Grep patterns:**
```
Grep: disabled=\{.*isPending|disabled=\{.*isLoading|disabled=\{.*isSubmitting
In: component file (output_mode: content)

Grep: loading=\{|<Spinner|<Loader|<LoadingSpinner
In: component file near the button (output_mode: content)
```

**Good patterns (do NOT flag):**
```tsx
<Button disabled={isPending} onClick={handleSubmit}>
  {isPending ? <Spinner /> : 'Save'}
</Button>

<Button loading={isLoading} onClick={save}>Save</Button>

<LoadingButton loading={isPending} onClick={submit}>Submit</LoadingButton>
```

**Bad patterns (DO flag):**
```tsx
<Button onClick={handleSubmit}>Save</Button>  // triggers async but no loading state

<Button onClick={() => mutation.mutate(data)}>Delete</Button>  // no disabled/loading
```

**Record per finding:**
- Component name
- Button text
- The async action it triggers (mutation name or function name)
- Whether loading state exists (yes/no)
- File path and line number

### Check 4: Missing Confirmation (MEDIUM)

**Definition**: A button that performs a destructive or irreversible action without asking the user to confirm first. Accidental clicks cause data loss.

**Detect destructive buttons by:**
- Button text contains (case-insensitive): delete, remove, cancel, revoke, reject, block, disable, reset, clear, destroy, drop, purge, terminate, archive, disconnect
- `variant="destructive"` or className contains `danger`, `destructive`, `delete`, `red`, `btn-danger`
- onClick handler calls a function named like `handleDelete`, `removeUser`, `destroyRecord`, etc.
- Handler calls an API DELETE method

**Grep patterns:**
```
Grep: variant="destructive"|variant='destructive'
In: all scoped files (output_mode: content, -B 1, -A 3)

Grep: handleDelete|handleRemove|onDelete|onRemove|\.delete\(|\.remove\(
In: all scoped files (output_mode: content, -B 2, -A 2)
```

**Check if handler includes confirmation:**
```
Grep: window\.confirm|confirm\(|<AlertDialog|<ConfirmDialog|useConfirm|<Dialog.*confirm|openConfirm
In: component file (output_mode: content)
```

Valid confirmation mechanisms:
- `window.confirm()` or `confirm()` call before the action
- Opens a `<Dialog>`, `<Modal>`, `<AlertDialog>`, or `<ConfirmDialog>` component
- Uses a confirmation hook like `useConfirm()` or `useAlert()`
- The button itself is inside a confirmation dialog (it IS the "Yes, delete" button)
- Uses `<AlertDialogTrigger>` pattern where clicking opens confirmation first

**FALSE POSITIVES — do NOT flag:**
- Buttons that ARE the confirmation action (inside `<AlertDialogAction>`, inside a confirm modal)
- Cancel buttons that just close a modal or reset a form (not destructive to data)
- Buttons where "remove" means removing from a UI list without an API call (e.g., removing a filter tag)
- Clear/reset buttons for search inputs or form filters
- Archive/disable actions that are easily reversible

**Record per finding:**
- Component name
- Button text
- The destructive action (handler name or API endpoint)
- Has confirmation? (yes/no)
- File path and line number

### Check 5: Missing Disabled State (MEDIUM)

**Definition**: A form submit button that remains enabled when the form data is invalid, allowing users to submit invalid data and rely entirely on server-side validation.

**Detection steps:**
1. Find all `<form>` elements or components using `useForm()` / React Hook Form.
2. Find the submit button (`type="submit"` or the button that calls `handleSubmit`).
3. Check if the submit button is disabled when appropriate.

**Grep patterns:**
```
Grep: useForm\(|useFormContext\(|<form
In: all scoped files (output_mode: content, -A 5)

Grep: formState|isValid|isDirty|isSubmitting
In: component files with forms (output_mode: content, -B 1, -A 1)

Grep: disabled=\{
In: component files near submit buttons (output_mode: content, -A 1)
```

**Valid disabled patterns (do NOT flag):**
- `disabled={!isValid}` or `disabled={!formState.isValid}`
- `disabled={!isDirty}` or `disabled={!formState.isDirty}`
- `disabled={isSubmitting}` or `disabled={formState.isSubmitting}`
- `disabled={isPending}` (from a mutation's loading state)
- Any custom validation check in the `disabled` prop

**FALSE POSITIVES — do NOT flag:**
- Forms where validation is done entirely on submit (strategic choice)
- Search forms or filter forms (submitting empty is valid)
- Login forms (typically submit is always enabled; validation is server-side)
- Forms with only optional fields
- Forms that use HTML5 native validation (`required` attribute) without React Hook Form

**Record per finding:**
- Component name
- Form purpose (based on context: login, registration, edit profile, create product, etc.)
- Submit button disabled condition (or "none")
- File path and line number

### Check 6: Empty Href (HIGH)

**Definition**: Anchor tags with non-functional `href` values that should be `<button>` elements instead, since they perform actions rather than navigation.

**Grep patterns:**
```
Grep: href="#"|href='#'|href=""|href=''|href="javascript:void\(0\)"|href="javascript:;"
In: all scoped files (output_mode: content, -B 1, -A 3)
```

**Why these are problematic:**
- `href="#"` scrolls to page top unexpectedly and adds `#` to URL history
- `href=""` reloads the current page
- `href="javascript:void(0)"` is a legacy anti-pattern that hurts CSP compliance
- All of these signal that the element should be a `<button>` with an `onClick`

**FALSE POSITIVES — do NOT flag:**
- `href="#section-id"` where it is an actual in-page anchor link targeting a section with that ID — verify the ID exists in the same file or a known layout
- Anchor tags from third-party component libraries' internal rendering
- Temporary placeholder links with a TODO comment indicating future implementation
- Skip links for accessibility (`href="#main-content"`)

**Record per finding:**
- Component name
- Current href value
- Element text content
- Suggested fix (change to `<button>` with onClick, or provide a real route)
- File path and line number

### Check 7: Missing Aria-Label (MEDIUM)

**Definition**: Icon-only buttons with no accessible name. Screen readers announce these as "button" with no context, making the interface unusable for visually impaired users.

**Detection strategy:**
1. Find buttons whose children are ONLY icon components (no text nodes).
2. Icon components typically match: `*Icon`, `Icon*`, `<svg`, `<img`, `<Svg*`, Lucide icons (e.g., `<Trash2 />`, `<Plus />`, `<Edit />`), HeroIcons.
3. Check for shadcn patterns: `<Button size="icon">` or `<Button variant="ghost" size="sm">` with only an icon child.
4. Also check `<IconButton>` components.

**Grep patterns:**
```
Grep: size="icon"|size='icon'
In: all scoped files (output_mode: content, -B 1, -A 3)

Grep: <Button[^>]*>\s*<[A-Z][a-zA-Z]*Icon|<Button[^>]*>\s*<(Trash|Edit|Plus|Minus|X|Check|Search|Settings|Menu|ChevronLeft|ChevronRight|ArrowLeft|ArrowRight|Eye|EyeOff|Copy|Download|Upload|Refresh|Filter|MoreHorizontal|MoreVertical)
In: all scoped files (output_mode: content, -B 1, -A 2)
```

**A button has an accessible name if ANY of these exist:**
- `aria-label="..."` attribute on the button
- `aria-labelledby="..."` attribute referencing a visible label
- `<span className="sr-only">` child inside the button (screen-reader-only text)
- `title="..."` attribute on the button
- Visible text content alongside the icon

**FALSE POSITIVES — do NOT flag:**
- Buttons with visible text alongside an icon
- Buttons where the icon component itself carries `aria-label`
- `<DropdownMenuTrigger>`, `<PopoverTrigger>`, `<TooltipTrigger>` and similar Radix primitives that handle accessibility labels internally through their content/label props
- Buttons in a labelled toolbar where `aria-label` is on the toolbar container

**Record per finding:**
- Component name
- Icon component used (e.g., `<Trash2 />`, `<Plus />`)
- Has aria-label? (yes/no)
- Has sr-only text? (yes/no)
- Has title? (yes/no)
- File path and line number

### Check 8: Non-Button Clickable (MEDIUM)

**Definition**: Elements that have `onClick` handlers but are not semantic button or anchor elements. These are inaccessible to keyboard users and screen readers unless proper ARIA attributes and keyboard handlers are added.

**Detection** — use the clickable non-button elements found in Step 2.

**A non-button clickable is a problem if it lacks proper accessibility. All three are needed:**
1. `role="button"` (or `role="link"` if it navigates)
2. `tabIndex={0}` (allows keyboard focus)
3. `onKeyDown` or `onKeyPress` handler (allows keyboard activation with Enter/Space)

**Flag if ANY of the three is missing.**

**Grep patterns for verification:**
```
Grep: role="button"|role='button'|role="link"|role='link'
In: same line or adjacent lines of the clickable element

Grep: tabIndex=\{0\}|tabIndex="0"|tabIndex='0'
In: same element

Grep: onKeyDown=|onKeyPress=
In: same element
```

**FALSE POSITIVES — do NOT flag:**
- `<tr onClick>` in a data table where the row click is a convenience shortcut AND individual action buttons exist in the row columns. Still note it but mark as "info" severity.
- `<Card onClick>` from shadcn/ui that may handle accessibility internally
- Elements that render a nested `<button>` or `<a>` inside (the onClick just expands the hit target)
- `<label>` with onClick associated with a form input
- List items `<li>` inside a `<Listbox>`, `<Select>`, or `<Combobox>` component (handled by the library)
- Elements with `role="tab"`, `role="menuitem"`, `role="option"` — these are part of composite ARIA widgets

**Record per finding:**
- Component name
- Element type (div, span, tr, td, li, Card, etc.)
- Purpose (what the click does)
- Has role? (yes/no, and what role)
- Has tabIndex? (yes/no)
- Has keyboard handler? (yes/no)
- File path and line number

### Check 9: Duplicate Action (LOW)

**Definition**: The same API endpoint or mutation is triggered by two or more buttons within the same component. This may indicate copy-paste errors, redundant UI elements, or incomplete refactoring.

**Detection steps:**
1. For each component, list all buttons and their onClick handlers.
2. Trace each handler to its API call (mutation endpoint or fetch URL).
3. If two or more buttons in the same component call the same endpoint with the same HTTP method, flag it.

**Grep patterns:**
```
Grep: mutate\(|mutateAsync\(
In: component files (output_mode: content, -B 5, -A 2)
```

Cross-reference the mutation variable with its declaration to determine the API endpoint.

**Legitimate cases (flag as INFO, not warning):**
- Save button in form + Ctrl+S keyboard shortcut (same action, different trigger)
- Multiple entry points to same action (e.g., inline edit + modal edit)
- Header action button + context menu action on same resource
- Toolbar button duplicated in dropdown menu for responsive design

**Record per finding:**
- Component name
- API endpoint that is duplicated
- Button texts involved
- Whether it appears intentional (based on patterns above)
- File path and line numbers

### Check 10: Missing Tooltip (LOW)

**Definition**: Icon-only buttons without a hover explanation for sighted users. While `aria-label` helps screen readers, sighted users also need to understand what an icon button does, especially for non-obvious icons.

**Detection** — overlaps with Check 7 data but focuses on sighted user experience.

**Check if the icon-only button has:**
```
Grep: <Tooltip|<TooltipProvider|<TooltipTrigger
In: surrounding context of icon-only buttons (check 5-10 lines above)

Grep: title=
In: button element props
```

Valid tooltip mechanisms:
- A `<Tooltip>` or `<TooltipTrigger>` wrapper component from shadcn/Radix
- A `title="..."` attribute (native browser tooltip)
- A custom tooltip implementation that renders on hover

**FALSE POSITIVES — do NOT flag:**
- Buttons that already have visible text labels (tooltip unnecessary)
- Buttons with universally recognized icons: close (X), menu (hamburger), search (magnifying glass) — though tooltips are still best practice
- Mobile-only components where hover tooltips don't apply
- Buttons already wrapped in `<TooltipTrigger>` (tooltip exists)

**Record per finding:**
- Component name
- Icon used
- Has Tooltip wrapper? (yes/no)
- Has title attribute? (yes/no)
- File path and line number

### Step 5: Compile Report

After all checks are complete, compile the final report using the Output Format below.

**Counting:**
- `COMPONENTS_SCANNED`: number of unique .tsx files scanned
- `INTERACTIVE_ELEMENTS_FOUND`: total buttons + links + clickable non-buttons
- Per-check issue counts
- Total issues across all checks

**Sorting:**
- Within each check section, sort findings by file path then by line number
- In the Element Overview table, sort by issue count descending
- Summary table preserves the check number order

## Output Format

Present the report as a single markdown document:

```markdown
# QA Buttons Audit Report

**Date**: {current date}
**Scope**: {SCOPE_LABEL}
**Components Scanned**: {COMPONENTS_SCANNED}
**Interactive Elements Found**: {INTERACTIVE_ELEMENTS_FOUND}

## Summary

| # | Check | Severity | Issues |
|---|-------|----------|--------|
| 1 | Dead button | Critical | {count} |
| 2 | Broken route link | High | {count} |
| 3 | Missing loading state | High | {count} |
| 4 | Missing confirmation | Medium | {count} |
| 5 | Missing disabled state | Medium | {count} |
| 6 | Empty href | High | {count} |
| 7 | Missing aria-label | Medium | {count} |
| 8 | Non-button clickable | Medium | {count} |
| 9 | Duplicate action | Low | {count} |
| 10 | Missing tooltip | Low | {count} |
| | **Total** | | **{total}** |

## Element Overview

| Component | File | Buttons | Links | Clickables | Issues |
|-----------|------|---------|-------|------------|--------|
| {name} | {relative path} | {n} | {n} | {n} | {n} |
| ... | ... | ... | ... | ... | ... |
| **Totals** | | **{n}** | **{n}** | **{n}** | **{n}** |

---

## CRITICAL

### Check 1: Dead Buttons

{If no issues: "No dead buttons found."}

| Component | Element | Text/Content | File:Line | Issue |
|-----------|---------|-------------|-----------|-------|
| {name} | `<Button>` | "Click me" | path:42 | No onClick, not in form, no trigger wrapper |

**Risk**: Dead buttons erode user trust. Users click and nothing happens, leading to frustration and repeated clicks. In forms, a dead button may prevent submission entirely.
**Fix**: Add an `onClick` handler, set `type="submit"` if inside a form, or wrap in a trigger component (`<DialogTrigger>`, etc.) if intended to open a dialog.

---

## HIGH

### Check 2: Broken Route Links

{If no issues: "All route links resolve to valid routes."}

| Component | Link Text | Target Path | Matched Route | File:Line |
|-----------|-----------|-------------|---------------|-----------|
| {name} | "Users" | /admin/users | No match | path:55 |

**Risk**: Broken links navigate to a blank page or 404. Users lose context and may lose unsaved work. In SPAs, broken routes often show a white screen with no error.
**Fix**: Verify the target path exists in the router configuration. If the route was renamed or moved, update the link to match. If the page does not exist yet, remove the link or mark it as "coming soon" with a disabled state.

### Check 3: Missing Loading State

{If no issues: "All async action buttons have loading states."}

| Component | Button Text | Async Action | Has Loading? | File:Line |
|-----------|-------------|-------------|--------------|-----------|
| {name} | "Save" | updateUser mutation | No | path:78 |

**Risk**: Without loading feedback, users double-click submit buttons, causing duplicate API calls (duplicate records, double payments, duplicate messages). Users may also think the app is frozen and navigate away mid-operation, causing data corruption.
**Fix**: Destructure `isPending` from the mutation hook and apply: `<Button disabled={isPending} onClick={handleSubmit}>{isPending ? 'Saving...' : 'Save'}</Button>`.

### Check 6: Empty Hrefs

{If no issues: "No empty or placeholder hrefs found."}

| Component | Element | Current Href | Text | Suggested Fix | File:Line |
|-----------|---------|-------------|------|---------------|-----------|
| {name} | `<a>` | `#` | "Click here" | Use `<button>` with onClick | path:33 |

**Risk**: `href="#"` scrolls to page top and pollutes browser history. `href=""` reloads the page. `javascript:void(0)` violates Content Security Policy. All patterns confuse screen readers by announcing a link that goes nowhere.
**Fix**: If the element performs an action, replace with `<button onClick={handler}>`. If it navigates, provide a real route via `<Link to="/path">`.

---

## MEDIUM

### Check 4: Missing Confirmation

{If no issues: "All destructive actions have confirmation dialogs."}

| Component | Button Text | Action | Has Confirm? | File:Line |
|-----------|-------------|--------|--------------|-----------|
| {name} | "Delete" | DELETE /api/users/:id | No | path:92 |

**Risk**: Accidental clicks permanently delete data. In applications where data integrity is critical, accidental deletion can disrupt workflows.
**Fix**: Wrap the button in a confirmation dialog component. Example: "Are you sure you want to delete this item? This action cannot be undone."

### Check 5: Missing Disabled State

{If no issues: "All form submit buttons have appropriate disabled states."}

| Component | Form Purpose | Disabled Condition | File:Line |
|-----------|-------------|-------------------|-----------|
| {name} | User edit form | None | path:45 |

**Risk**: Users can submit invalid data, leading to validation errors returned from the server. This creates a poor user experience with delayed error feedback.
**Fix**: Destructure `formState` from `useForm()` and apply: `<Button type="submit" disabled={!formState.isValid || formState.isSubmitting}>`.

### Check 7: Missing Aria-Label

{If no issues: "All icon-only buttons have accessible names."}

| Component | Icon | Has aria-label | Has sr-only | Has title | File:Line |
|-----------|------|---------------|-------------|----------|-----------|
| {name} | `<Trash2 />` | No | No | No | path:67 |

**Risk**: Screen readers announce "button" with no label. Visually impaired users cannot determine the button's purpose. This is a WCAG 2.1 Level A violation (Success Criterion 4.1.2).
**Fix**: Add `aria-label="Delete item"` to the button, or add a `<span className="sr-only">Delete item</span>` inside the button.

### Check 8: Non-Button Clickable

{If no issues: "All clickable elements use semantic HTML."}

| Component | Element | Purpose | Has role | Has tabIndex | Has keyboard | File:Line |
|-----------|---------|---------|---------|-------------|-------------|-----------|
| {name} | `<div>` | Navigate to detail | No | No | No | path:89 |

**Risk**: Keyboard users cannot focus or activate these elements. Screen readers do not announce them as interactive. This is a WCAG 2.1 Level A violation (Success Criterion 2.1.1).
**Fix**: Replace `<div onClick={...}>` with `<button onClick={...}>` for actions, or `<Link to="...">` for navigation. If a `<div>` must be used, add `role="button"`, `tabIndex={0}`, and `onKeyDown={(e) => e.key === 'Enter' && handler()}`.

---

## LOW

### Check 9: Duplicate Actions

{If no issues: "No duplicate actions detected."}

| Component | API Endpoint | Button Texts | Intentional? | File:Lines |
|-----------|-------------|-------------|--------------|------------|
| {name} | DELETE /api/users/:id | "Delete", "Remove" | No | path:45, path:78 |

**Risk**: Redundant buttons confuse users about which to click. May indicate incomplete refactoring where an old button was not removed when a new one was added.
**Fix**: Remove the duplicate button or consolidate into a single action trigger. If both are intentional (e.g., toolbar + context menu), document the rationale.

### Check 10: Missing Tooltips

{If no issues: "All icon-only buttons have tooltips."}

| Component | Icon | Has Tooltip | Has Title | File:Line |
|-----------|------|------------|----------|-----------|
| {name} | `<Edit />` | No | No | path:34 |

**Risk**: Sighted users must guess icon meanings. Non-standard icons (e.g., a gear for settings vs. a gear for processing) are ambiguous without hover text.
**Fix**: Wrap the button in a tooltip component. Example: `<Tooltip><TooltipTrigger asChild><Button>...</Button></TooltipTrigger><TooltipContent>Edit item</TooltipContent></Tooltip>`.

---

## Recommendations

{Provide 3-5 prioritized recommendations based on the findings, such as:}
1. **Fix {N} critical dead buttons** — These buttons do nothing when clicked, confusing users and blocking workflows.
2. **Add loading states to {N} async buttons** — Users may double-click or think the app is frozen, causing duplicate operations.
3. **Add confirmation to {N} destructive actions** — Prevents accidental data loss in applications where data integrity is critical.
4. **Add aria-labels to {N} icon buttons** — Required for WCAG 2.1 Level A accessibility compliance.
5. **Replace {N} empty hrefs with semantic buttons** — Improves accessibility, keyboard navigation, and CSP compliance.
```

## Scanning Patterns Reference

| What | Grep Pattern | File Glob |
|------|-------------|-----------|
| JSX Buttons | `<button\|<Button\|<IconButton\|<LoadingButton` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Links | `<Link\|<NavLink\|<a ` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Programmatic navigate | `navigate\(\|useNavigate` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| onClick handlers | `onClick=` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Route definitions | `<Route\|path:\|path=` | Router/route config files |
| React Query mutations | `useMutation` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Mutation triggers | `mutate\(\|mutateAsync\(` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Loading states | `isPending\|isLoading\|isSubmitting` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Confirm dialogs | `confirm\(\|window\.confirm\|<AlertDialog\|<ConfirmDialog` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Aria attributes | `aria-label\|aria-labelledby\|sr-only` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Role attributes | `role="button"\|role="link"` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Disabled props | `disabled=\|isDisabled` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Destructive markers | `variant="destructive"\|btn-danger\|handleDelete\|onDelete` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Form elements | `<form\|useForm\|handleSubmit` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Dialog triggers | `DialogTrigger\|PopoverTrigger\|DropdownMenuTrigger\|AlertDialogTrigger\|CollapsibleTrigger` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Tooltip wrappers | `<Tooltip\|<TooltipTrigger\|<TooltipProvider\|title=` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Icon-only buttons | `size="icon"\|<IconButton` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Empty hrefs | `href="#"\|href=""\|href="javascript:` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Keyboard handlers | `onKeyDown=\|onKeyPress=` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| React Hook Form | `useForm\(\|useFormContext\(\|formState` | `{frontend-dir}/**/*.tsx` (all frontend apps) |
| Async handlers | `async.*=>\|await\s+fetch\|await\s+api\.\|await\s+http` | `{frontend-dir}/**/*.tsx` (all frontend apps) |

## Edge Cases to Watch For

1. **shadcn/ui compound components**: `<DropdownMenuTrigger>`, `<AlertDialogTrigger>`, `<PopoverTrigger>`, `<CollapsibleTrigger>`, and `<AccordionTrigger>` pass implicit click handlers to their children via the Radix `asChild` pattern. A `<Button>` inside these wrappers is NOT dead even without an explicit `onClick`.

2. **React.forwardRef buttons**: Buttons wrapped in `forwardRef` receive `onClick` from their parent component at runtime. Check the parent's usage of the component to verify the handler is passed.

3. **Buttons inside map() loops**: A single button definition may render multiple instances. Issues apply to ALL rendered instances. Check the loop's data source to understand the scale.

4. **Conditional rendering**: `{condition && <Button>}` or `{condition ? <ButtonA> : <ButtonB>}` — both branches must be checked. A button may only appear under certain conditions (e.g., role check, feature flag).

5. **Portal-rendered modals**: Buttons inside `<Dialog>`, `<Sheet>`, `<Popover>`, and similar portal components may be defined in a different part of the component tree than where they render in the DOM. Trace the component hierarchy, not the DOM.

6. **Dynamic onClick from props**: `<Button onClick={props.onAction}>` — the handler comes from the parent. You must trace to the parent component to verify it provides a valid handler.

7. **Spread props pattern**: `<Button {...props}>` or `<Button {...rest}>` — onClick may be in the spread object. Check the component's prop type or the calling component.

8. **Button components with custom names**: The project may have wrapper components like `<SaveButton>`, `<CancelButton>`, `<ActionButton>` that render a `<Button>` internally. Search for these custom button components and trace their internals.

9. **Disabled buttons that are intentionally dead**: Buttons with `disabled={true}` or `disabled` (always disabled) may intentionally have no handler. These are UI placeholders or coming-soon features. Do not flag as dead.

10. **Form submission via Enter key**: Forms can be submitted by pressing Enter in an input field, not just by clicking the submit button. The submit button may exist only for visual affordance and may not need its own `onClick`.

## READ-ONLY Diagnostic

This skill is a READ-ONLY diagnostic. It does NOT modify any files. It scans, analyzes, and reports findings. All recommended fixes are presented as suggestions in the report for the developer to implement.
