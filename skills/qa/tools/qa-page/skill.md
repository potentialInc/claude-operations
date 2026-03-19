---
name: qa-page
description: Run all relevant QA skills on a single page and generate a unified report with cross-skill analysis
user-invocable: true
argument-hint: "<page-or-component-file-path> [--backend <path>] [--depth full|shallow] [--focus <skill1,skill2,...>] [--skip <skill1,skill2,...>]"
---

# QA Page (Integrated)

Automatically detect which QA analyses apply to a frontend page, run them in sequence, and generate a unified report with cross-skill insights, de-duplicated test cases, and a weighted overall score.

**Important:** This skill is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Purpose

This skill helps you:
1. **Auto-detect applicable analyses** — Scan the page to determine which QA skills are relevant
2. **Run analyses in optimal order** — Execute applicable skills respecting dependencies
3. **Cross-reference findings** — Identify issues that span multiple QA domains
4. **De-duplicate test cases** — Merge overlapping test cases from different skills
5. **Produce a unified score** — Weighted average across all executed skills
6. **Generate a single report** — One comprehensive report instead of 15 separate files
7. **Track coverage** — Show which QA domains were analyzed and which were skipped (and why)
8. **Suggest follow-ups** — Recommend additional analyses based on findings

**Important:** This skill is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack tracing
- (Optional) `.qa-config.json` at project root
- (Optional) Project `CLAUDE.md` with project-specific conventions

See `qa-shared/reference.md` for shared conventions, supported extensions, and configuration options.

---

## Usage

```bash
/qa-page <page-or-component-file-path>
```

**Examples:**
```bash
# Full analysis of a page (auto-detects all relevant skills)
/qa-page src/pages/UserManagement.tsx

# With backend tracing
/qa-page src/pages/orders/index.tsx --backend server/src

# Focus on specific skills only
/qa-page src/pages/Dashboard.tsx --focus qa-list,qa-states

# Skip specific skills
/qa-page src/pages/Settings.tsx --skip qa-performance,qa-a11y

# Frontend-only shallow analysis
/qa-page src/components/ProductCatalog.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): full-stack tracing. `shallow`: frontend-only analysis |
| `--focus <skills>` | No | Comma-separated list of skill names to run exclusively (e.g., `qa-list,qa-inputs`) |
| `--skip <skills>` | No | Comma-separated list of skill names to skip (e.g., `qa-performance,qa-a11y`) |

**Valid skill names for `--focus` and `--skip`:**
`qa-states`, `qa-inputs`, `qa-list`, `qa-modal`, `qa-auth`, `qa-back-nav`, `qa-a11y`, `qa-performance`, `qa-buttons`, `qa-api-sync`, `qa-crud`, `qa-db-integrity`, `qa-layout`, `qa-screen`, `qa-test-gen`, `qa-security`

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
│  Step 2: Page Content Scan & Skill Selection                 │
│  - Scan page for patterns matching each QA domain            │
│  - Build list of applicable skills with confidence levels    │
│  - Apply --focus / --skip filters                            │
│  - Determine execution order based on dependencies           │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Sequential Skill Execution                          │
│  - Execute each applicable skill in dependency order         │
│  - Collect issues, test cases, and scores from each          │
│  - Track execution time per skill                            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Cross-Skill Analysis                                │
│  - Identify issues spanning multiple QA domains              │
│  - Detect patterns only visible when multiple analyses run   │
│  - Flag contradictions between skill findings                │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Test Case De-duplication & Merging                  │
│  - Merge overlapping test cases from different skills        │
│  - Assign unique IDs to merged test cases                    │
│  - Group by user workflow instead of QA domain               │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Unified Score Calculation                           │
│  - Calculate weighted average across all executed skills     │
│  - Apply cross-skill bonus/penalty                           │
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

> See `qa-shared/reference.md` for the full self-verification and auto-retry protocol.
> Every Step in this skill follows the same checklist-based retry mechanism (max 3 retries).

---

## Execution Steps

### Step 1: Input Validation & Project Detection

1. **Validate file path** — Ensure file exists and has supported extension (see `qa-shared/reference.md`)
2. **Detect frontend framework** — Use common framework detection from `qa-shared/reference.md`
3. **Detect backend framework** — If `--depth full`, use common backend detection from `qa-shared/reference.md`
4. **Load configuration** — Read `.qa-config.json` if present at project root
5. **Read CLAUDE.md** — If present, extract project-specific conventions and known patterns

**Checklist:**
- [ ] File exists and extension is supported
- [ ] Frontend framework identified
- [ ] Backend framework identified (if `--depth full`)
- [ ] Configuration loaded (or defaults applied)
- [ ] CLAUDE.md read (if present)

---

### Step 2: Page Content Scan & Skill Selection

Scan the page file and its imports to detect which QA domains are relevant.

#### Detection Matrix

| QA Skill | Detection Signals | Confidence |
|----------|-------------------|------------|
| **qa-states** | `useQuery`, `useSWR`, `fetch()`, `axios`, `<Suspense>`, `loading.tsx`, `error.tsx`, `async` server component, `use()` hook, `useTransition` | HIGH if data fetching detected; always run as BASELINE |
| **qa-inputs** | `<input>`, `<form>`, form library imports (React Hook Form, Formik, Conform, VeeValidate), `useForm`, `<textarea>`, `<select>`, `useFormState`, Server Actions | HIGH if `<form>` or form library; MEDIUM if individual inputs only |
| **qa-list** | Table imports (DataTable, Table, DataGrid), `<table>`, `.map()` rendering lists, column definitions, pagination, TanStack Table, AG Grid, virtualization | HIGH if table library; MEDIUM if `.map()` with list rendering |
| **qa-modal** | Dialog/modal imports (Dialog, Modal, Drawer, Sheet, AlertDialog), `<dialog>`, `showModal()`, `cmdk`, portal rendering | HIGH if modal/dialog component found |
| **qa-buttons** | `<button>`, `<Button>`, onClick handlers, form submit buttons, icon buttons, floating action buttons, toggle buttons, button groups | HIGH if interactive buttons detected |
| **qa-auth** | Auth imports (`useAuth`, `useSession`, `auth()`, `currentUser()`), role checks, permission guards, `<Can>`, `<ProtectedRoute>`, RLS policies | HIGH if auth detected; MEDIUM if conditional rendering based on user |
| **qa-back-nav** | Router imports (`useNavigate`, `useRouter`, `$router`, `goto`), `<Link>`, `<a href>`, `history.pushState`, `window.location`, breadcrumbs | HIGH if router detected; MEDIUM if `<Link>` or `<a>` only |
| **qa-a11y** | Always applicable — every page needs accessibility | BASELINE (always run) |
| **qa-performance** | Always applicable — every page needs performance check | BASELINE (always run) |
| **qa-crud** | API service imports, `useMutation`, CRUD operations, form submissions, REST endpoints | HIGH if CRUD patterns detected |
| **qa-api-sync** | API calls, service imports, endpoint references, fetch/axios calls, tRPC routers, GraphQL queries | HIGH if API endpoints detected |
| **qa-db-integrity** | Only for `--depth full` with backend; entity/migration files, ORM usage (Prisma, Drizzle, TypeORM), schema references | HIGH if DB/ORM detected; skipped without backend |
| **qa-security** | Only for `--depth full` with backend; raw SQL queries, `Math.random()` in security context, file uploads, `console.log` with sensitive data, `eval`/`exec`, `window.open` with user URLs | HIGH if backend detected; MEDIUM if frontend-only (XSS/IME checks) |
| **qa-layout** | Always applicable — every page has layout (grid/flex containers, responsive breakpoints, CSS modules) | BASELINE (always run) |
| **qa-screen** | Always applicable — every page needs screen-level testing | BASELINE (always run) |
| **qa-test-gen** | Always applicable — generates tests from all prior findings | BASELINE (always run last) |

#### Confidence Levels

| Level | Meaning | Behavior |
|-------|---------|----------|
| **BASELINE** | Always relevant regardless of page content | Always included unless `--skip` |
| **HIGH** | Strong signals detected in page or direct imports | Always included |
| **MEDIUM** | Indirect signals (child components may contain patterns) | Included in `--depth full`, skipped in `--depth shallow` |
| **LOW** | Weak signals, possibly false positive | Skipped; noted in coverage report |
| **NONE** | No signals detected | Skipped; noted in coverage report |

#### Dependency Order

Skills are executed in this order to allow later skills to leverage earlier findings:

```
 1. qa-states        (foundational — data fetching states)
 2. qa-inputs        (forms — feeds into a11y and auth)
 3. qa-list          (data display — feeds into states + auth)
 4. qa-modal         (overlays — feeds into back-nav)
 5. qa-buttons       (interactive elements — feeds into a11y)
 6. qa-crud          (endpoint completeness)
 7. qa-api-sync      (frontend↔backend sync)
 8. qa-db-integrity  (database layer — only if --depth full)
 9. qa-security      (code-level security — only if --depth full)
10. qa-auth          (cross-cutting — needs knowledge of forms/tables/modals)
11. qa-back-nav      (navigation — needs knowledge of modals)
12. qa-layout        (visual consistency)
13. qa-a11y          (cross-cutting — leverages all prior findings)
14. qa-performance   (cross-cutting — leverages all prior findings)
14. qa-screen        (runtime testing — benefits from all static analysis)
15. qa-test-gen      (test generation — runs last, uses all findings)
```

#### Apply Filters

- If `--focus` provided: Only run specified skills (plus BASELINE if not explicitly excluded)
- If `--skip` provided: Remove specified skills from the list
- Log which skills were selected and why

**Checklist:**
- [ ] Page file and imports scanned for all 15 QA domain signals
- [ ] Confidence level assigned to each skill
- [ ] `--focus` / `--skip` filters applied
- [ ] Execution order determined
- [ ] Skill selection logged with reasons

---

### Step 3: Sequential Skill Execution

Execute each selected skill against the page file. For each skill:

1. **Announce start** — Display which skill is running and why it was selected
2. **Execute the full skill workflow** — Run all internal steps of the QA skill
3. **Collect results** — Extract from each skill:
   - List of issues with severity and description
   - Test cases with IDs
   - Score breakdown by category
   - Overall score and status
4. **Announce completion** — Display skill score and issue count

**Progress display format:**
```
━━━ QA Page Analysis: [ComponentName] ━━━

Skills to run: N of 15
  ✓ qa-states (BASELINE)
  ✓ qa-inputs (HIGH — <form> + React Hook Form detected)
  ✓ qa-list (HIGH — DataTable import detected)
  ✓ qa-modal (HIGH — Dialog import detected)
  ✓ qa-buttons (HIGH — button elements detected)
  ✗ qa-auth (NONE — no auth detected)
  ✗ qa-back-nav (NONE — no router imports)
  ✓ qa-a11y (BASELINE)
  ✓ qa-performance (BASELINE)
  ✓ qa-api-sync (HIGH — API endpoints detected)
  ✗ qa-crud (NONE — no CRUD patterns)
  ✗ qa-db-integrity (NONE — no ORM detected)
  ✓ qa-layout (BASELINE)
  ✓ qa-screen (BASELINE)
  ✓ qa-test-gen (BASELINE)

━━━ Running 1/N: qa-states ━━━
  ... [skill output] ...
  Score: 85/100 (PASS) | Issues: 2 WARNING, 1 INFO

━━━ Running 2/N: qa-inputs ━━━
  ... [skill output] ...
  Score: 72/100 (NEEDS ATTENTION) | Issues: 1 CRITICAL, 1 WARNING

[continues for each skill]
```

**Checklist:**
- [ ] Each selected skill executed completely
- [ ] Issues collected from all skills
- [ ] Test cases collected from all skills
- [ ] Scores collected from all skills
- [ ] Progress displayed throughout execution

---

### Step 4: Cross-Skill Analysis

After all individual skills complete, analyze findings across skills for patterns only visible in combination.

#### Cross-Skill Issue Patterns

| Pattern | Skills Involved | Severity | Description |
|---------|-----------------|----------|-------------|
| **Unprotected delete flow** | qa-auth + qa-modal | CRITICAL | Delete action lacks both permission check AND confirmation modal |
| **Inaccessible modal form** | qa-modal + qa-inputs + qa-a11y | CRITICAL | Modal with form but missing focus trap, label association, or keyboard close |
| **Table without loading state** | qa-list + qa-states | CRITICAL | Data table has no loading skeleton/spinner for initial or page-change loads |
| **Form submit without error handling** | qa-inputs + qa-states | WARNING | Form submits to API but has no error state for failed submission |
| **Modal breaks back navigation** | qa-modal + qa-back-nav | WARNING | Modal pushes history entry but back button doesn't close modal |
| **Permission-gated content without empty state** | qa-auth + qa-states | WARNING | Content hidden by permission but no "no access" empty state shown |
| **Table filter without accessibility** | qa-list + qa-a11y | WARNING | Filter inputs missing labels or ARIA attributes |
| **Heavy modal content not code-split** | qa-modal + qa-performance | WARNING | Modal contains heavy components (editor, chart) loaded eagerly |
| **Form validation without ARIA** | qa-inputs + qa-a11y | WARNING | Error messages not linked via aria-describedby |
| **Navigation without focus management** | qa-back-nav + qa-a11y | WARNING | Route change doesn't move focus to main content |
| **Large table without virtualization** | qa-list + qa-performance | INFO | Table renders 100+ rows without virtualization; may affect performance |
| **Image-heavy page without lazy loading** | qa-performance + qa-states | INFO | Multiple images without loading="lazy" on below-fold content |
| **API endpoint missing auth middleware** | qa-api-sync + qa-auth | CRITICAL | API route lacks authentication/authorization middleware |
| **CRUD operation without DB constraint** | qa-crud + qa-db-integrity | WARNING | Delete/update operation without foreign key or cascade constraint |
| **Layout shift from async content** | qa-layout + qa-states | WARNING | Layout containers lack fixed dimensions for async-loaded content |
| **Button without accessible label** | qa-buttons + qa-a11y | WARNING | Icon-only button missing aria-label or screen reader text |

#### Contradiction Detection

Flag when skills report conflicting findings:
- `qa-states` says Suspense is used correctly, but `qa-performance` flags unnecessary Suspense boundaries
- `qa-a11y` requires focus indicators, but `qa-performance` flags CSS animations on focus (usually not a real conflict — note both)
- `qa-inputs` says validation is client-only, but `qa-auth` says server validates — may be intentional layered validation

**Checklist:**
- [ ] All cross-skill patterns checked
- [ ] Cross-skill issues identified and severities assigned
- [ ] Contradictions flagged (if any)
- [ ] Cross-skill issues added to unified issue list

---

### Step 5: Test Case De-duplication & Merging

#### De-duplication Rules

1. **Exact overlap** — Same scenario from two skills → keep one, note both sources
   - Example: "Modal close on Escape" from `qa-modal` and `qa-a11y` → single test case
2. **Subset overlap** — One test is a subset of another → keep the more comprehensive one
   - Example: "Form error shown" (qa-inputs) vs "Form error announced to screen reader" (qa-a11y) → combine
3. **Complementary** — Tests cover different aspects of same interaction → group together
   - Example: "Delete button visible only to admin" (qa-auth) + "Delete confirmation modal" (qa-modal) → group as "Delete Flow"

#### Workflow-Based Grouping

Re-organize test cases by user workflow instead of QA domain:

| Workflow Group | Source Skills | Example Tests |
|---------------|---------------|---------------|
| **Data Viewing** | qa-list, qa-states, qa-performance | Load table, pagination, sorting, empty state, loading |
| **Data Creation** | qa-inputs, qa-modal, qa-a11y | Open create modal, fill form, validate, submit, error handling |
| **Data Editing** | qa-inputs, qa-modal, qa-auth | Edit permission, open edit modal, pre-fill form, save |
| **Data Deletion** | qa-auth, qa-modal, qa-list | Delete permission, confirmation modal, success/error, table refresh |
| **Navigation** | qa-back-nav, qa-a11y, qa-performance | Route transition, back button, scroll restoration, focus management |
| **Page Load** | qa-states, qa-performance, qa-a11y, qa-layout | Initial load, skeleton, data fetch, error recovery, screen reader |
| **API & Data** | qa-api-sync, qa-crud, qa-db-integrity | Endpoint coverage, CRUD completeness, schema consistency |
| **Interactive Elements** | qa-buttons, qa-a11y, qa-layout | Button states, click handlers, keyboard access, visual consistency |

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

#### Per-Skill Weights

Not all skills are equally important for every page. Weight based on relevance:

| Skill | Base Weight | Adjusted When... |
|-------|-------------|-------------------|
| qa-states | 10% | +3% if data-heavy page (tables, API calls) |
| qa-inputs | 10% | +3% if form-heavy page; -5% if no forms (set to 5%) |
| qa-list | 8% | +3% if data-display page; -3% if no tables (set to 5%) |
| qa-modal | 5% | +3% if modal-heavy page; -2% if no modals (set to 3%) |
| qa-buttons | 5% | +3% if button-heavy page; -2% if minimal buttons (set to 3%) |
| qa-auth | 7% | +3% if admin/auth page; -2% if public page (set to 5%) |
| qa-back-nav | 5% | +3% if multi-step flow; -2% if standalone page (set to 3%) |
| qa-a11y | 10% | Fixed — always 10% |
| qa-performance | 8% | Fixed — always 8% |
| qa-api-sync | 7% | +3% if API-heavy page; -2% if no API calls (set to 5%) |
| qa-crud | 5% | +3% if CRUD page; -2% if read-only (set to 3%) |
| qa-db-integrity | 5% | +3% if DB-heavy page; -2% if no DB access (set to 3%) |
| qa-layout | 5% | Fixed — always 5% |
| qa-screen | 5% | Fixed — always 5% |
| qa-test-gen | 5% | Fixed — always 5% |

**Note:** Weights are automatically normalized to total 100% based on which skills actually ran.

#### Unified Score Formula

```
Per-Skill Score: Each skill calculates its own score (0-100) per qa-shared/reference.md

Weighted Score = Σ (skill_score × normalized_weight) for all executed skills

Cross-Skill Adjustments:
  - Each cross-skill CRITICAL issue: -10 points
  - Each cross-skill WARNING issue: -3 points
  - Cross-skill adjustments capped at -20 total

Unified Score = max(0, min(100, Weighted Score + Cross-Skill Adjustments))
```

#### Unified Status

| Status | Score Range | Meaning |
|--------|------------|---------|
| **PASS** | >= 80 | Page meets quality standards across all QA domains |
| **NEEDS ATTENTION** | 60-79 | Page has notable gaps in one or more QA domains |
| **FAIL** | < 60 | Page has significant quality issues; must address before release |

**Checklist:**
- [ ] Per-skill weights calculated based on page content
- [ ] Weights normalized to 100%
- [ ] Weighted average calculated
- [ ] Cross-skill adjustments applied
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
║  Backend:       [auto-detected framework / N/A]                    ║
║  Report Date:   [YYYY-MM-DD HH:MM]                                ║
║  Depth:         [full / shallow]                                   ║
║  Skills Run:    [N] of 15                                          ║
║  Overall Score: [N]/100                                            ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  Section 1: Executive Summary                                      ║
║  Section 2: Skill Scorecard                                        ║
║  Section 3: Critical Issues (cross-skill first)                    ║
║  Section 4: Warning Issues                                         ║
║  Section 5: Cross-Skill Findings                                   ║
║  Section 6: Test Plan (workflow-grouped)                            ║
║  Section 7: Coverage Map                                           ║
║  Section 8: Recommendations                                        ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

#### Section 1: Executive Summary

A 3-5 sentence summary:
- What the page does (inferred from component name and content)
- How many skills were run and overall result
- Top 1-2 most critical findings
- Overall recommendation (ship as-is, fix criticals then ship, needs major rework)

#### Section 2: Skill Scorecard

| Skill | Score | Status | Weight | Issues (C/W/I) |
|-------|-------|--------|--------|-----------------|
| qa-states | 85/100 | PASS | 12% | 0/2/1 |
| qa-inputs | 72/100 | NEEDS ATTENTION | 13% | 1/1/0 |
| qa-list | 90/100 | PASS | 11% | 0/1/2 |
| qa-modal | 65/100 | NEEDS ATTENTION | 8% | 1/2/0 |
| qa-buttons | 88/100 | PASS | 5% | 0/1/0 |
| qa-auth | — | SKIPPED (no auth) | 0% | — |
| qa-back-nav | — | SKIPPED (no router) | 0% | — |
| qa-a11y | 78/100 | NEEDS ATTENTION | 10% | 0/3/2 |
| qa-performance | 88/100 | PASS | 8% | 0/1/3 |
| qa-api-sync | 82/100 | PASS | 7% | 0/1/1 |
| qa-crud | — | SKIPPED (no CRUD) | 0% | — |
| qa-db-integrity | — | SKIPPED (no ORM) | 0% | — |
| qa-layout | 91/100 | PASS | 5% | 0/0/2 |
| qa-screen | 80/100 | PASS | 5% | 0/2/1 |
| qa-test-gen | 95/100 | PASS | 5% | 0/0/1 |
| **Cross-Skill** | | | | 0/1/0 |
| **OVERALL** | **80/100** | **PASS** | **100%** | **2/15/13** |

#### Section 3: Critical Issues

Listed in priority order:
1. Cross-skill critical issues first (they affect multiple domains)
2. Per-skill critical issues by severity

For each issue:
```
[CRITICAL] [Source Skill(s)] Issue title
  Description: What's wrong
  Location: File path + line reference
  Impact: User impact description
  Fix: Recommended solution
```

#### Section 4: Warning Issues

Same format as critical issues, but for warnings.

#### Section 5: Cross-Skill Findings

Dedicated section for issues found through cross-skill analysis (from Step 4). This is the unique value of `/qa-page` over running skills individually.

#### Section 6: Test Plan (Workflow-Grouped)

Test cases organized by workflow (from Step 5):

```
### Data Viewing Workflow
| Test ID | Scenario | Steps | Expected | Source |
|---------|----------|-------|----------|--------|
| DATA-VIEW-01 | ... | ... | ... | qa-list, qa-states |
| DATA-VIEW-02 | ... | ... | ... | qa-list |

### Data Creation Workflow
| Test ID | Scenario | Steps | Expected | Source |
|---------|----------|-------|----------|--------|
| DATA-CREATE-01 | ... | ... | ... | qa-inputs, qa-modal |
```

#### Section 7: Coverage Map

Visual overview of what was analyzed:

```
Page Coverage Map:
  ✅ Data fetching & states      (qa-states — 85/100)
  ✅ Form validation             (qa-inputs — 72/100)
  ✅ Table/list display           (qa-list — 90/100)
  ✅ Modals & drawers             (qa-modal — 65/100)
  ✅ Buttons & interactions       (qa-buttons — 88/100)
  ⬜ Permission & roles           (qa-auth — SKIPPED: no auth detected)
  ⬜ Back navigation              (qa-back-nav — SKIPPED: no router imports)
  ✅ Accessibility (WCAG 2.2)     (qa-a11y — 78/100)
  ✅ Performance (Core Web Vitals) (qa-performance — 88/100)
  ✅ API synchronization          (qa-api-sync — 82/100)
  ⬜ CRUD completeness            (qa-crud — SKIPPED: no CRUD patterns)
  ⬜ DB integrity                 (qa-db-integrity — SKIPPED: no ORM detected)
  ✅ Layout consistency           (qa-layout — 91/100)
  ✅ Screen runtime testing       (qa-screen — 80/100)
  ✅ Test generation              (qa-test-gen — 95/100)

  Analyzed: N/15 skills | Coverage: N%
```

#### Section 8: Recommendations

1. **Immediate fixes** — Critical issues that must be fixed
2. **Before release** — Warning issues to address
3. **Follow-up analyses** — Suggested additional QA runs
   - "Run `qa-auth` if authentication is added to this page"
   - "Run `qa-back-nav` when this page is part of a multi-step flow"
4. **Improvement opportunities** — INFO items worth considering

**Checklist:**
- [ ] Executive summary written
- [ ] Skill scorecard compiled
- [ ] Critical issues listed with fix recommendations
- [ ] Warning issues listed
- [ ] Cross-skill findings documented
- [ ] Test plan organized by workflow
- [ ] Coverage map generated
- [ ] Recommendations provided

---

### Step 8: Save & Present Results

1. **Determine output path:**
   - Default: `.qa-reports/[ComponentName]_FullPage_QA_Report_[YYMMDD].md`
   - Override: `outputDir` from `.qa-config.json`

2. **Save the unified report**

3. **Display summary:**

```
━━━ QA Page Analysis Complete ━━━

Component: [ComponentName]
File:      [file-path]
Skills:    [N]/15 executed

╭──────────────────────────────────╮
│  Overall Score: [N]/100 — [STATUS] │
╰──────────────────────────────────╯

Skill Scorecard:
  qa-states          85  ████████░░  PASS
  qa-inputs          72  ███████░░░  NEEDS ATTENTION
  qa-list            90  █████████░  PASS
  qa-modal           65  ██████░░░░  NEEDS ATTENTION
  qa-buttons         88  ████████░░  PASS
  qa-a11y            78  ███████░░░  NEEDS ATTENTION
  qa-performance     88  ████████░░  PASS
  qa-api-sync        82  ████████░░  PASS
  qa-layout          91  █████████░  PASS
  qa-screen          80  ████████░░  PASS
  qa-test-gen        95  █████████░  PASS

Issues: [N] CRITICAL | [N] WARNING | [N] INFO
Cross-Skill Issues: [N]

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

See individual skill score calculations and `qa-shared/reference.md` for base scoring rules.

The unified score is a weighted average of per-skill scores, adjusted by cross-skill findings:

```
Unified Score = max(0, min(100,
  Σ(skill_score × normalized_weight) + cross_skill_adjustment
))

Cross-Skill Adjustments (capped at -20 total):
  - Cross-skill CRITICAL: -10 each
  - Cross-skill WARNING: -3 each

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
| No skills applicable | Only BASELINE skills detected | Runs baseline skills only |
| Skill execution failed | Individual skill error | Logs error, skips skill, notes in coverage map |
| Invalid `--focus` value | Unknown skill name | Lists valid skill names and exits |
| Invalid `--skip` value | Unknown skill name | Lists valid skill names and exits |
| All skills skipped | `--skip` excludes everything | Error: "At least one skill must be executable" |

---

## Edge Cases Handled

- **Single-component page** — Page exports only one small component; may only trigger BASELINE skills. Report notes limited scope.
- **Layout/wrapper component** — Page is a layout (`Layout.tsx`, `_app.tsx`); children not directly analyzable. Report suggests running on child pages instead.
- **Re-export file** — Page file re-exports from another file (`export { default } from './UserManagementPage'`). Follows re-export to actual component.
- **Very large page** — Page with 1000+ lines and many sub-components. Each skill analyzes relevant portions; report may be long.
- **Micro-frontend host** — Page loads remote modules; remote content not analyzable. Noted in coverage map.
- **Server Component page** — Next.js App Router `page.tsx` may be async server component. Skills adapt detection accordingly.
- **Shared component analyzed standalone** — A shared `Button.tsx` analyzed in isolation; most skills will be BASELINE only. Suggests analyzing in context of a page instead.
- **Page with dynamic imports** — Lazy-loaded sections only partially analyzable without runtime. Skills note unresolvable imports.
- **Test file provided** — User provides `*.test.tsx` or `*.spec.tsx`; warn that this is a test file and suggest the source file instead.
- **Storybook story provided** — User provides `*.stories.tsx`; warn and suggest the component file instead.
- **Multiple tables/forms/modals on one page** — Each table/form/modal analyzed separately within its skill; unified report aggregates all.
- **Config file conflicts** — `.qa-config.json` `ignore.rules` may suppress findings from some skills but not cross-skill checks.
- **Circular component references** — Component A imports Component B which imports Component A. Break at 10 levels; note as unresolvable.
- **CSS Modules / Tailwind** — Styling approach affects accessibility (contrast) and performance (CSS size) analysis; detected and adapted.

---

## Related Skills

- `/qa-states` — Loading, error, and empty state analysis
- `/qa-inputs` — Form and input field analysis
- `/qa-list` — Table and list page analysis
- `/qa-buttons` — Button and interactive element analysis
- `/qa-crud` — CRUD endpoint completeness
- `/qa-api-sync` — Frontend↔backend API sync
- `/qa-db-integrity` — Database schema consistency
- `/qa-modal` — Modal and drawer analysis
- `/qa-auth` — Authentication and authorization
- `/qa-back-nav` — Back navigation analysis
- `/qa-layout` — Layout consistency
- `/qa-a11y` — WCAG 2.2 accessibility
- `/qa-performance` — Frontend performance
- `/qa-screen` — Playwright screen-level QA
- `/qa-test-gen` — E2E test generation
- `/qa-fix` — Auto-fix orchestrator

---

## Tips

1. **Start with `/qa-page` for new pages** — Run this first to get a comprehensive overview, then dive into specific skills for deeper analysis
2. **Use `--focus` for targeted re-checks** — After fixing issues, re-run only the affected skills: `--focus qa-inputs,qa-a11y`
3. **Cross-skill findings are the key value** — Issues found across skill boundaries are the ones most often missed in manual QA
4. **BASELINE skills always run** — Accessibility, performance, layout, screen, and test-gen are universal concerns; don't skip them
5. **Workflow-grouped test cases save time** — Test by user workflow (create, edit, delete) rather than by QA domain for more efficient manual testing
6. **Compare scores over time** — Re-run `/qa-page` after fixes to track improvement; the coverage map shows at-a-glance progress
7. **Don't ignore SKIPPED skills** — If a skill is skipped because no patterns were detected, verify this is correct — the feature may be in a child component
8. **Shallow mode is fast but incomplete** — Use `--depth shallow` for quick checks during development; always run `--depth full` before release
