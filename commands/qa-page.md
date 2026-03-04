---
description: Run all relevant QA commands on a page and generate a unified report with cross-command analysis
argument-hint: "<page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow] [--focus <cmd1,cmd2,...>] [--skip <cmd1,cmd2,...>]"
---

# QA Page (Integrated)

Automatically detect which QA analyses apply to a frontend page, run them in sequence, and generate a unified report with cross-command insights, de-duplicated test cases, and a weighted overall score.

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Purpose

This command helps you:
1. **Auto-detect applicable analyses** — Scan the page to determine which `/qa-*` commands are relevant
2. **Run analyses in optimal order** — Execute applicable commands respecting dependencies
3. **Cross-reference findings** — Identify issues that span multiple QA domains
4. **De-duplicate test cases** — Merge overlapping test cases from different commands
5. **Produce a unified score** — Weighted average across all executed commands
6. **Generate a single report** — One comprehensive report instead of 8 separate files
7. **Track coverage** — Show which QA domains were analyzed and which were skipped (and why)
8. **Suggest follow-ups** — Recommend additional analyses based on findings

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack tracing
- (Optional) `.qa-config.json` at project root (see `qa-shared-reference.md`)
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```bash
/qa-page <page-or-component-file-path>
```

**Examples:**
```bash
# Full analysis of a page (auto-detects all relevant commands)
/qa-page src/pages/UserManagement.tsx

# With backend tracing
/qa-page frontend/app/pages/orders/index.tsx --backend backend/src

# Focus on specific commands only
/qa-page src/pages/Dashboard.tsx --focus table-list,loading-error-empty

# Skip specific commands
/qa-page src/pages/Settings.tsx --skip performance,accessibility

# Frontend-only shallow analysis
/qa-page src/components/ProductCatalog.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): full-stack tracing. `shallow`: frontend-only analysis |
| `--focus <cmds>` | No | Comma-separated list of command suffixes to run exclusively (e.g., `table-list,input-fields`) |
| `--skip <cmds>` | No | Comma-separated list of command suffixes to skip (e.g., `performance,accessibility`) |

**Valid command suffixes for `--focus` and `--skip`:**
`back-navigation`, `input-fields`, `loading-error-empty`, `modal-drawer`, `permission-role`, `table-list`, `accessibility`, `performance`

**Note:** `--focus` and `--skip` are mutually exclusive. If both are provided, `--focus` takes precedence.

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection                │
│  - Validate file path and extension                          │
│  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)│
│  - Detect backend framework (if --depth full)                │
│  - Load .qa-config.json if present                           │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Page Content Scan & Command Selection               │
│  - Scan page for patterns matching each QA domain            │
│  - Build list of applicable commands with confidence levels  │
│  - Apply --focus / --skip filters                            │
│  - Determine execution order based on dependencies           │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Sequential Command Execution                        │
│  - Execute each applicable command in dependency order       │
│  - Collect issues, test cases, and scores from each          │
│  - Track execution time per command                          │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Cross-Command Analysis                              │
│  - Identify issues spanning multiple QA domains              │
│  - Detect patterns only visible when multiple analyses run   │
│  - Flag contradictions between command findings              │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Test Case De-duplication & Merging                  │
│  - Merge overlapping test cases from different commands      │
│  - Assign unique IDs to merged test cases                    │
│  - Group by user workflow instead of QA domain               │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Unified Score Calculation                           │
│  - Calculate weighted average across all executed commands   │
│  - Apply cross-command bonus/penalty                         │
│  - Determine overall page quality status                     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Generate Unified Report                             │
│  - Combine all findings into single report                   │
│  - Include executive summary, detail sections, test plan     │
│  - Add coverage map and follow-up recommendations            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 8: Save & Present Results                              │
│  - Save unified report to output directory                   │
│  - Display summary to user                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Execution Rule: Self-Verification & Auto-Retry

> See `qa-shared-reference.md` for the full self-verification and auto-retry protocol.
> Every Step in this command follows the same checklist-based retry mechanism (max 3 retries).

---

## Execution Steps

### Step 1: Input Validation & Project Detection

1. **Validate file path** — Ensure file exists and has supported extension (see `qa-shared-reference.md`)
2. **Detect frontend framework** — Use common framework detection from `qa-shared-reference.md`
3. **Detect backend framework** — If `--depth full`, use common backend detection from `qa-shared-reference.md`
4. **Load configuration** — Read `.qa-config.json` if present at project root
5. **Read CLAUDE.md** — If present, extract project-specific conventions and known patterns

**Checklist:**
- [ ] File exists and extension is supported
- [ ] Frontend framework identified
- [ ] Backend framework identified (if `--depth full`)
- [ ] Configuration loaded (or defaults applied)
- [ ] CLAUDE.md read (if present)

---

### Step 2: Page Content Scan & Command Selection

Scan the page file and its imports to detect which QA domains are relevant.

#### Detection Matrix

| QA Command | Detection Signals | Confidence |
|------------|-------------------|------------|
| **back-navigation** | Router imports (`useNavigate`, `useRouter`, `$router`, `goto`), `<Link>`, `<a href>`, `history.pushState`, `window.location`, breadcrumbs | HIGH if router detected; MEDIUM if `<Link>` or `<a>` only |
| **input-fields** | `<input>`, `<textarea>`, `<select>`, `<form>`, form library imports (React Hook Form, Formik, Conform, VeeValidate), `useForm`, `useFormState`, Server Actions | HIGH if `<form>` or form library; MEDIUM if individual inputs only |
| **loading-error-empty** | `useQuery`, `useSWR`, `fetch()`, `axios`, `<Suspense>`, `loading.tsx`, `error.tsx`, `async` server component, `use()` hook, `useTransition` | HIGH if data fetching detected; always run as BASELINE |
| **modal-drawer** | Dialog/modal imports (Dialog, Modal, Drawer, Sheet, AlertDialog), `<dialog>`, `showModal()`, `cmdk`, portal rendering | HIGH if modal/dialog component found |
| **permission-role** | Auth imports (`useAuth`, `useSession`, `auth()`, `currentUser()`), role checks, permission guards, `<Can>`, `<ProtectedRoute>`, RLS policies | HIGH if auth detected; MEDIUM if conditional rendering based on user |
| **table-list** | Table imports (DataTable, Table, DataGrid), `<table>`, `.map()` rendering lists, column definitions, pagination, TanStack Table, AG Grid, virtualization | HIGH if table library; MEDIUM if `.map()` with list rendering |
| **accessibility** | Always applicable — every page needs accessibility | BASELINE (always run) |
| **performance** | Always applicable — every page needs performance check | BASELINE (always run) |

#### Confidence Levels

| Level | Meaning | Behavior |
|-------|---------|----------|
| **BASELINE** | Always relevant regardless of page content | Always included unless `--skip` |
| **HIGH** | Strong signals detected in page or direct imports | Always included |
| **MEDIUM** | Indirect signals (child components may contain patterns) | Included in `--depth full`, skipped in `--depth shallow` |
| **LOW** | Weak signals, possibly false positive | Skipped; noted in coverage report |
| **NONE** | No signals detected | Skipped; noted in coverage report |

#### Dependency Order

Commands are executed in this order to allow later commands to leverage earlier findings:

```
1. loading-error-empty  (foundational — most pages need this)
2. input-fields          (forms are common; findings feed into accessibility)
3. table-list            (data display; findings feed into loading + permission)
4. modal-drawer          (overlays; findings feed into back-navigation)
5. permission-role       (cross-cutting; needs knowledge of forms/tables/modals)
6. back-navigation       (navigation; needs knowledge of modals pushing history)
7. accessibility         (cross-cutting; leverages all prior findings)
8. performance           (cross-cutting; leverages all prior findings)
```

#### Apply Filters

- If `--focus` provided: Only run specified commands (plus BASELINE if not explicitly excluded)
- If `--skip` provided: Remove specified commands from the list
- Log which commands were selected and why

**Checklist:**
- [ ] Page file and imports scanned for all 8 QA domain signals
- [ ] Confidence level assigned to each command
- [ ] `--focus` / `--skip` filters applied
- [ ] Execution order determined
- [ ] Command selection logged with reasons

---

### Step 3: Sequential Command Execution

Execute each selected command against the page file. For each command:

1. **Announce start** — Display which command is running and why it was selected
2. **Execute the full command workflow** — Run all 8 internal steps of the QA command
3. **Collect results** — Extract from each command:
   - List of issues with severity and description
   - Test cases with IDs
   - Score breakdown by category
   - Overall score and status
4. **Announce completion** — Display command score and issue count

**Progress display format:**
```
━━━ QA Page Analysis: [ComponentName] ━━━

Commands to run: 6 of 8
  ✓ loading-error-empty (BASELINE)
  ✓ input-fields (HIGH — <form> + React Hook Form detected)
  ✓ table-list (HIGH — DataTable import detected)
  ✓ modal-drawer (HIGH — Dialog import detected)
  ✓ permission-role (MEDIUM — useSession detected)
  ✗ back-navigation (NONE — no router imports)
  ✓ accessibility (BASELINE)
  ✓ performance (BASELINE)

━━━ Running 1/6: loading-error-empty ━━━
  ... [command output] ...
  Score: 85/100 (PASS) | Issues: 2 WARNING, 1 INFO

━━━ Running 2/6: input-fields ━━━
  ... [command output] ...
  Score: 72/100 (NEEDS ATTENTION) | Issues: 1 CRITICAL, 1 WARNING

[continues for each command]
```

**Checklist:**
- [ ] Each selected command executed completely
- [ ] Issues collected from all commands
- [ ] Test cases collected from all commands
- [ ] Scores collected from all commands
- [ ] Progress displayed throughout execution

---

### Step 4: Cross-Command Analysis

After all individual commands complete, analyze findings across commands for patterns only visible in combination.

#### Cross-Command Issue Patterns

| Pattern | Commands Involved | Severity | Description |
|---------|-------------------|----------|-------------|
| **Unprotected delete flow** | permission-role + modal-drawer | CRITICAL | Delete action lacks both permission check AND confirmation modal |
| **Inaccessible modal form** | modal-drawer + input-fields + accessibility | CRITICAL | Modal with form but missing focus trap, label association, or keyboard close |
| **Table without loading state** | table-list + loading-error-empty | CRITICAL | Data table has no loading skeleton/spinner for initial or page-change loads |
| **Form submit without error handling** | input-fields + loading-error-empty | WARNING | Form submits to API but has no error state for failed submission |
| **Modal breaks back navigation** | modal-drawer + back-navigation | WARNING | Modal pushes history entry but back button doesn't close modal |
| **Permission-gated content without empty state** | permission-role + loading-error-empty | WARNING | Content hidden by permission but no "no access" empty state shown |
| **Table filter without accessibility** | table-list + accessibility | WARNING | Filter inputs missing labels or ARIA attributes |
| **Heavy modal content not code-split** | modal-drawer + performance | WARNING | Modal contains heavy components (editor, chart) loaded eagerly |
| **Form validation without ARIA** | input-fields + accessibility | WARNING | Error messages not linked via aria-describedby |
| **Navigation without focus management** | back-navigation + accessibility | WARNING | Route change doesn't move focus to main content |
| **Large table without virtualization** | table-list + performance | INFO | Table renders 100+ rows without virtualization; may affect performance |
| **Image-heavy page without lazy loading** | performance + loading-error-empty | INFO | Multiple images without loading="lazy" on below-fold content |

#### Contradiction Detection

Flag when commands report conflicting findings:
- `loading-error-empty` says Suspense is used correctly, but `performance` flags unnecessary Suspense boundaries
- `accessibility` requires focus indicators, but `performance` flags CSS animations on focus (usually not a real conflict — note both)
- `input-fields` says validation is client-only, but `permission-role` says server validates — may be intentional layered validation

**Checklist:**
- [ ] All cross-command patterns checked
- [ ] Cross-command issues identified and severities assigned
- [ ] Contradictions flagged (if any)
- [ ] Cross-command issues added to unified issue list

---

### Step 5: Test Case De-duplication & Merging

#### De-duplication Rules

1. **Exact overlap** — Same scenario from two commands → keep one, note both sources
   - Example: "Modal close on Escape" from `modal-drawer` and `accessibility` → single test case
2. **Subset overlap** — One test is a subset of another → keep the more comprehensive one
   - Example: "Form error shown" (input-fields) vs "Form error announced to screen reader" (accessibility) → combine
3. **Complementary** — Tests cover different aspects of same interaction → group together
   - Example: "Delete button visible only to admin" (permission) + "Delete confirmation modal" (modal) → group as "Delete Flow"

#### Workflow-Based Grouping

Re-organize test cases by user workflow instead of QA domain:

| Workflow Group | Source Commands | Example Tests |
|---------------|-----------------|---------------|
| **Data Viewing** | table-list, loading-error-empty, performance | Load table, pagination, sorting, empty state, loading |
| **Data Creation** | input-fields, modal-drawer, accessibility | Open create modal, fill form, validate, submit, error handling |
| **Data Editing** | input-fields, modal-drawer, permission-role | Edit permission, open edit modal, pre-fill form, save |
| **Data Deletion** | permission-role, modal-drawer, table-list | Delete permission, confirmation modal, success/error, table refresh |
| **Navigation** | back-navigation, accessibility, performance | Route transition, back button, scroll restoration, focus management |
| **Page Load** | loading-error-empty, performance, accessibility | Initial load, skeleton, data fetch, error recovery, screen reader |

**Assign unified test IDs:**
```
[WorkflowGroup]-[NN]
e.g., DATA-VIEW-01, DATA-CREATE-03, NAV-02, PAGE-LOAD-05
```

**Checklist:**
- [ ] Duplicate test cases identified and merged
- [ ] Subset test cases combined
- [ ] Complementary test cases grouped
- [ ] All test cases re-organized by workflow
- [ ] Unified test IDs assigned

---

### Step 6: Unified Score Calculation

#### Per-Command Weights

Not all commands are equally important for every page. Weight based on relevance:

| Command | Base Weight | Adjusted When... |
|---------|-------------|-------------------|
| loading-error-empty | 15% | +5% if data-heavy page (tables, API calls) |
| input-fields | 15% | +5% if form-heavy page; -10% if no forms (set to 5%) |
| table-list | 15% | +5% if data-display page; -10% if no tables (set to 5%) |
| modal-drawer | 10% | +5% if modal-heavy page; -5% if no modals (set to 5%) |
| permission-role | 10% | +5% if admin/auth page; -5% if public page (set to 5%) |
| back-navigation | 10% | +5% if multi-step flow; -5% if standalone page (set to 5%) |
| accessibility | 15% | Fixed — always 15% |
| performance | 10% | Fixed — always 10% |

**Note:** Weights are automatically normalized to total 100% based on which commands actually ran.

#### Unified Score Formula

```
Per-Command Score: Each command calculates its own score (0-100) per qa-shared-reference.md

Weighted Score = Σ (command_score × normalized_weight) for all executed commands

Cross-Command Adjustments:
  - Each cross-command CRITICAL issue: -10 points
  - Each cross-command WARNING issue: -3 points
  - Cross-command adjustments capped at -20 total

Unified Score = max(0, min(100, Weighted Score + Cross-Command Adjustments))
```

#### Unified Status

| Status | Score Range | Meaning |
|--------|------------|---------|
| **PASS** | >= 80 | Page meets quality standards across all QA domains |
| **NEEDS ATTENTION** | 60-79 | Page has notable gaps in one or more QA domains |
| **FAIL** | < 60 | Page has significant quality issues; must address before release |

**Checklist:**
- [ ] Per-command weights calculated based on page content
- [ ] Weights normalized to 100%
- [ ] Weighted average calculated
- [ ] Cross-command adjustments applied
- [ ] Overall status determined

---

### Step 7: Generate Unified Report

Generate the unified report with the following sections:

```
╔════════════════════════════════════════════════════════════════════╗
║                   QA Full Page Report                              ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                    ║
║  File:          [file-path]                                        ║
║  Framework:     [React + library / Vue + library / etc.]           ║
║  Backend:       [NestJS / Express / N/A]                           ║
║  Report Date:   [YYYY-MM-DD HH:MM]                                ║
║  Depth:         [full / shallow]                                   ║
║  Commands Run:  [N] of 8                                           ║
║  Overall Score: [N]/100                                            ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  Section 1: Executive Summary                                      ║
║  Section 2: Command Scorecard                                      ║
║  Section 3: Critical Issues (cross-command first)                  ║
║  Section 4: Warning Issues                                         ║
║  Section 5: Cross-Command Findings                                 ║
║  Section 6: Test Plan (workflow-grouped)                            ║
║  Section 7: Coverage Map                                           ║
║  Section 8: Recommendations                                        ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

#### Section 1: Executive Summary

A 3-5 sentence summary:
- What the page does (inferred from component name and content)
- How many commands were run and overall result
- Top 1-2 most critical findings
- Overall recommendation (ship as-is, fix criticals then ship, needs major rework)

#### Section 2: Command Scorecard

| Command | Score | Status | Weight | Issues (C/W/I) |
|---------|-------|--------|--------|-----------------|
| loading-error-empty | 85/100 | PASS | 20% | 0/2/1 |
| input-fields | 72/100 | NEEDS ATTENTION | 20% | 1/1/0 |
| table-list | 90/100 | PASS | 20% | 0/1/2 |
| modal-drawer | 65/100 | NEEDS ATTENTION | 10% | 1/2/0 |
| permission-role | — | SKIPPED (no auth) | 0% | — |
| back-navigation | — | SKIPPED (no router) | 0% | — |
| accessibility | 78/100 | NEEDS ATTENTION | 15% | 0/3/2 |
| performance | 88/100 | PASS | 15% | 0/1/3 |
| **Cross-Command** | | | | 0/1/0 |
| **OVERALL** | **80/100** | **PASS** | **100%** | **2/11/8** |

#### Section 3: Critical Issues

Listed in priority order:
1. Cross-command critical issues first (they affect multiple domains)
2. Per-command critical issues by severity

For each issue:
```
[CRITICAL] [Source Command(s)] Issue title
  Description: What's wrong
  Location: File path + line reference
  Impact: User impact description
  Fix: Recommended solution
```

#### Section 4: Warning Issues

Same format as critical issues, but for warnings.

#### Section 5: Cross-Command Findings

Dedicated section for issues found through cross-command analysis (from Step 4). This is the unique value of `/qa-page` over running commands individually.

#### Section 6: Test Plan (Workflow-Grouped)

Test cases organized by workflow (from Step 5):

```
### Data Viewing Workflow
| Test ID | Scenario | Steps | Expected | Source |
|---------|----------|-------|----------|--------|
| DATA-VIEW-01 | ... | ... | ... | table-list, loading |
| DATA-VIEW-02 | ... | ... | ... | table-list |

### Data Creation Workflow
| Test ID | Scenario | Steps | Expected | Source |
|---------|----------|-------|----------|--------|
| DATA-CREATE-01 | ... | ... | ... | input-fields, modal |
```

#### Section 7: Coverage Map

Visual overview of what was analyzed:

```
Page Coverage Map:
  ✅ Data fetching & states    (loading-error-empty — 85/100)
  ✅ Form validation           (input-fields — 72/100)
  ✅ Table/list display         (table-list — 90/100)
  ✅ Modals & drawers           (modal-drawer — 65/100)
  ⬜ Permission & roles         (permission-role — SKIPPED: no auth detected)
  ⬜ Back navigation            (back-navigation — SKIPPED: no router imports)
  ✅ Accessibility (WCAG 2.2)   (accessibility — 78/100)
  ✅ Performance (Core Web Vitals) (performance — 88/100)

  Analyzed: 6/8 commands | Coverage: 75%
```

#### Section 8: Recommendations

1. **Immediate fixes** — Critical issues that must be fixed
2. **Before release** — Warning issues to address
3. **Follow-up analyses** — Suggested additional QA runs
   - "Run `/qa-permission-role` if auth is added to this page"
   - "Run `/qa-back-navigation` when this page is part of a multi-step flow"
4. **Improvement opportunities** — INFO items worth considering

**Checklist:**
- [ ] Executive summary written
- [ ] Command scorecard compiled
- [ ] Critical issues listed with fix recommendations
- [ ] Warning issues listed
- [ ] Cross-command findings documented
- [ ] Test plan organized by workflow
- [ ] Coverage map generated
- [ ] Recommendations provided

---

### Step 8: Save & Present Results

1. **Determine output path:**
   - Default: `.claude-project/qa/[ComponentName]_FullPage_QA_Report_[YYMMDD].md`
   - Override: `outputDir` from `.qa-config.json`

2. **Save the unified report**

3. **Display summary:**

```
━━━ QA Page Analysis Complete ━━━

Component: [ComponentName]
File:      [file-path]
Commands:  [N]/8 executed

╭──────────────────────────────────╮
│  Overall Score: [N]/100 — [STATUS] │
╰──────────────────────────────────╯

Command Scorecard:
  loading-error-empty    85  ████████░░  PASS
  input-fields           72  ███████░░░  NEEDS ATTENTION
  table-list             90  █████████░  PASS
  modal-drawer           65  ██████░░░░  NEEDS ATTENTION
  accessibility          78  ███████░░░  NEEDS ATTENTION
  performance            88  ████████░░  PASS

Issues: [N] CRITICAL | [N] WARNING | [N] INFO
Cross-Command Issues: [N]

Top Issues:
  1. [CRITICAL] [source] Issue description
  2. [WARNING] [source] Issue description
  3. [WARNING] [source] Issue description

Report saved to: [output-path]
```

**Checklist:**
- [ ] Report saved to output directory
- [ ] Summary displayed with scorecard
- [ ] Top issues highlighted
- [ ] Report path shown

---

## Score Calculation

See individual command score calculations and `qa-shared-reference.md` for base scoring rules.

The unified score is a weighted average of per-command scores, adjusted by cross-command findings:

```
Unified Score = max(0, min(100,
  Σ(command_score × normalized_weight) + cross_command_adjustment
))

Cross-Command Adjustments (capped at -20 total):
  - Cross-command CRITICAL: -10 each
  - Cross-command WARNING: -3 each

Status:
  PASS:            >= 80
  NEEDS ATTENTION: 60-79
  FAIL:            < 60
```

---

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| File not found | Invalid path | Check file path and try again |
| Unsupported file type | Wrong extension | Supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js` |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| Backend path not found | `--backend` invalid | Runs frontend-only analysis (`--depth shallow`) |
| No commands applicable | Only BASELINE commands detected | Runs accessibility + performance only |
| Command execution failed | Individual command error | Logs error, skips command, notes in coverage map |
| Invalid `--focus` value | Unknown command suffix | Lists valid suffixes and exits |
| Invalid `--skip` value | Unknown command suffix | Lists valid suffixes and exits |
| All commands skipped | `--skip` excludes everything | Error: "At least one command must be executable" |

---

## Edge Cases Handled

- **Single-component page** — Page exports only one small component; may only trigger BASELINE commands. Report notes limited scope.
- **Layout/wrapper component** — Page is a layout (`Layout.tsx`, `_app.tsx`); children not directly analyzable. Report suggests running on child pages instead.
- **Re-export file** — Page file re-exports from another file (`export { default } from './UserManagementPage'`). Follows re-export to actual component.
- **Very large page** — Page with 1000+ lines and many sub-components. Each command analyzes relevant portions; report may be long.
- **Micro-frontend host** — Page loads remote modules; remote content not analyzable. Noted in coverage map.
- **Server Component page** — Next.js App Router `page.tsx` may be async server component. Commands adapt detection accordingly.
- **Shared component analyzed standalone** — A shared `Button.tsx` analyzed in isolation; most commands will be BASELINE only. Suggests analyzing in context of a page instead.
- **Page with dynamic imports** — Lazy-loaded sections only partially analyzable without runtime. Commands note unresolvable imports.
- **Test file provided** — User provides `*.test.tsx` or `*.spec.tsx`; warn that this is a test file and suggest the source file instead.
- **Storybook story provided** — User provides `*.stories.tsx`; warn and suggest the component file instead.
- **Multiple tables/forms/modals on one page** — Each table/form/modal analyzed separately within its command; unified report aggregates all.
- **Config file conflicts** — `.qa-config.json` `ignore.rules` may suppress findings from some commands but not cross-command checks.
- **Circular component references** — Component A imports Component B which imports Component A. Break at 10 levels; note as unresolvable.
- **CSS Modules / Tailwind** — Styling approach affects accessibility (contrast) and performance (CSS size) analysis; detected and adapted.

---

## Related Commands

- `/qa-back-navigation` — Standalone back navigation analysis
- `/qa-input-fields` — Standalone form and input field analysis
- `/qa-loading-error-empty` — Standalone loading/error/empty state analysis
- `/qa-modal-drawer` — Standalone modal and drawer analysis
- `/qa-permission-role` — Standalone permission and role analysis
- `/qa-table-list` — Standalone table and list analysis
- `/qa-accessibility` — Standalone WCAG 2.2 accessibility analysis
- `/qa-performance` — Standalone frontend performance analysis
- `/review-command` — Validate this command's structure and quality

---

## Tips

1. **Start with `/qa-page` for new pages** — Run this first to get a comprehensive overview, then dive into specific commands for deeper analysis
2. **Use `--focus` for targeted re-checks** — After fixing issues, re-run only the affected commands: `--focus input-fields,accessibility`
3. **Cross-command findings are the key value** — Issues found across command boundaries are the ones most often missed in manual QA
4. **BASELINE commands always run** — Accessibility and performance are universal concerns; don't skip them
5. **Workflow-grouped test cases save time** — Test by user workflow (create, edit, delete) rather than by QA domain for more efficient manual testing
6. **Compare scores over time** — Re-run `/qa-page` after fixes to track improvement; the coverage map shows at-a-glance progress
7. **Don't ignore SKIPPED commands** — If a command is skipped because no patterns were detected, verify this is correct — the feature may be in a child component
8. **Shallow mode is fast but incomplete** — Use `--depth shallow` for quick checks during development; always run `--depth full` before release
