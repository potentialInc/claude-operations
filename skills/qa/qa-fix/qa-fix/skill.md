---
name: qa-fix
description: "Universal QA: diagnose → fix → verify. Auto-detects stack. Self-contained checklist with scan procedures covering data, API, auth, inputs, UI, i18n."
user-invocable: true
argument-hint: "[module] [--check data|api|auth|inputs|ui|i18n] [--diff [ref]] [--check-only] [--lang ko]"
---

# QA Fix — Diagnose, Fix, Verify

Universal, self-contained QA workflow. Auto-detects project stack. No external skill dependencies.

For **runtime/browser-based testing** (z-index overlap, actual click failures, JS errors), use `/qa-screen` separately — this skill is static analysis only.

## Usage

```
/qa-fix exercises              # Full QA on module → fix → verify
/qa-fix                        # Full QA on all modules
/qa-fix --diff                 # QA only files changed since last commit
/qa-fix --diff HEAD~3          # QA only files changed in last 3 commits
/qa-fix --diff main            # QA only files changed vs main branch
/qa-fix exercises --check api  # Specific layer on specific module
/qa-fix --check data,auth      # Multiple layers, all modules
/qa-fix --check-only           # Diagnose only (no fixes)
/qa-fix --lang ko              # Output in Korean
```

### Check Layers

| Layer | What it covers |
|-------|---------------|
| `data` | Schema integrity, migration drift, dead columns, FK/indexes |
| `api` | CRUD completeness, frontend↔backend sync, response shapes |
| `auth` | Guards, role checks, token handling, security vulnerabilities |
| `inputs` | Entity/Model → DTO/Serializer → Schema → Form field consistency |
| `ui` | States, buttons, modals, lists, navigation, layout, a11y, performance |
| `i18n` | Hardcoded strings, missing translation keys, locale-aware formatting |

Omit `--check` to run ALL layers.

---

## Step 0: Project Detection (ALWAYS runs first)

### 0a. Stack auto-detection

Read `package.json`, `requirements.txt`, `pyproject.toml`, `composer.json`, `pom.xml`, `build.gradle`, `go.mod` — whichever exist. Determine:

| Detect | Signals |
|--------|---------|
| **Backend** | NestJS (`@nestjs/core`), Express, Django (`django` in requirements), Spring Boot, Laravel, Go |
| **ORM** | TypeORM (`typeorm`), Prisma (`@prisma/client`), Sequelize, Django ORM, SQLAlchemy, Drizzle, Eloquent |
| **Frontend** | React (`react`), Vue (`vue`), Angular (`@angular/core`), Svelte, Next.js, Nuxt |
| **Validation** | class-validator, Zod (`zod`), Yup, Joi, Pydantic, Django serializers |
| **Form lib** | React Hook Form, Formik, VeeValidate, Angular Reactive Forms |
| **CSS** | Tailwind (`tailwindcss`), CSS Modules, Styled Components, SCSS |
| **Data fetching** | TanStack Query, SWR, RTK Query, Apollo, Axios+manual |

Record detected stack — all scan procedures adapt patterns accordingly.

**For detailed detection signals, file patterns, and framework-specific mappings, use the tables in `qa-shared/reference.md`.**

### 0b. Multi-app detection

Scan for multiple frontend apps (e.g., `frontend/`, `frontend-dashboard/`, `admin/`, `web/`, `mobile/`). List each app directory. All frontend scans run across ALL detected apps, and cross-app consistency is checked.

### 0c. --diff mode scoping

If `--diff` flag:
```bash
git diff --name-only {ref} -- .   # default ref = HEAD
```
Filter the resulting file list to determine which layers/modules are affected. Only scan those.

### 0d. Project context

Read project-root docs to enable autonomous decisions. Search in order — read each that exists:

```
1. CLAUDE.md                              → project conventions, roles, patterns, known rules

2. PRD / User Stories (search all common locations):
   .claude-project/prd/**/*.md
   .claude-project/requirements/**/*.md
   docs/prd/**/*.md
   docs/requirements/**/*.md
   PRD.md, USER_STORIES.md, REQUIREMENTS.md (project root)
   → Extract: intended features, acceptance criteria, field specs, user flows

3. Project documentation:
   .claude-project/docs/*                 → API docs, DB schema, architecture
   docs/*                                 → any project docs

4. Implementation status (if exists):
   .claude-project/status/**/*.md         → what's done vs pending
```

Extract:
- **Intended behavior**: what each feature SHOULD do (from PRD/user stories)
- **Roles & permissions**: which roles, who can do what
- **In-scope features**: active vs deprecated/demo
- **Acceptance criteria**: specific conditions each feature must meet
- **Protected decisions**: intentional choices not to "fix"
- **Project-specific rules**: any rules in CLAUDE.md that map to checklist items — treat violations as CRITICAL
- **Service profile**: infer the service's nature from ALL available signals (target market, language, domain, scale, user base). This profile governs issue classification across ALL layers.

### 0e. Service profile inference

Before scanning any layer, build a **service profile** by reading code signals holistically:

| Signal | Where to look | Inference examples |
|--------|--------------|-------------------|
| Language/locale | i18n config, translation files, UI strings, DB seed data | Single-language vs multi-language |
| Domain | Entity names, business logic, PRD | Healthcare, e-commerce, internal tool, etc. |
| Scale | User model, data volume patterns, pagination presence | Small team tool vs high-traffic public service |
| Target market | Currency, phone format, address fields, timezone config | Domestic vs global |
| Data sensitivity | Auth patterns, encryption, compliance mentions | Standard vs regulated (HIPAA, GDPR, etc.) |
| Maturity | Git history, TODO density, test coverage | Early prototype vs production |

**This profile is NOT informational — it MUST actively gate every issue classification.** For each potential issue, ask: "Given this service profile, is this actually a problem?" Examples:

- Single-language service → i18n multi-locale issues are SKIP, not MANUAL
- Small admin-only lookup table → missing pagination is LOW, not MEDIUM
- Security tokens (OTP) → hard-delete is correct behavior, not a violation
- Migration adds then later removes a column → check final cumulative state, not intermediate steps
- Domestic-only service → global timezone/currency handling is SKIP
- Ghost column follows standard extensibility pattern (OAuth, social login, payment) → SKIP, not FIX (see D2 guide)
- Soft-delete filter missing on user-facing query (chat, feed) → CRITICAL, not MEDIUM (see D8 clarification)

**PRD enables intent-based QA**: without it, QA can only check code consistency (does the code match itself?). With PRD, QA can also check completeness (does the code match what was supposed to be built?). Service profile enables **context-based QA**: filtering out issues that are technically flaggable but irrelevant to the project's actual needs.

No project docs? → infer service profile from code signals alone. Only use MANUAL when the service profile genuinely cannot resolve the ambiguity.

### 0f. Compile check (MANDATORY)

Run the project's type-check/compile command **before any layer analysis**. This catches type-level errors that static file reading cannot detect (type mismatches, import errors, generic inference failures, etc.).

**Procedure:**

1. **Detect compile commands** from `package.json` scripts or project conventions:
   - TypeScript (Node): `npx tsc --noEmit`
   - Python (mypy): `mypy .`
   - Go: `go build ./...`
   - Java/Kotlin: `./gradlew compileJava` or `mvn compile`
   - Rust: `cargo check`
2. **Run for EACH compilable directory** (backend, frontend, frontend-dashboard, etc.)
3. **Collect all errors**, separating source code errors from test file errors
4. **Classify each error**:
   - Source code errors → **CRITICAL** — add to Phase 1 report as `C1` issues, fix in Phase 2 before layer-specific fixes
   - Test file errors → **HIGH** — add to Phase 1 report as `C2` issues, fix in Phase 2 after source fixes
5. **Record baseline** — if errors exist before QA changes, note them so Phase 3 can distinguish pre-existing vs regression

**Why this step exists:** Layer-specific analysis reads files and pattern-matches, but cannot detect:
- Generic type inference failures (e.g., `zodResolver` output type vs form data type)
- Import mode errors (`verbatimModuleSyntax` requiring `import type`)
- Cross-file type mismatches that only the compiler resolves
- Entity type changes not propagated to test fixtures (CLAUDE.md rule 12)

These errors are invisible to static reading but immediately caught by the compiler.

| # | Check | Severity |
|---|-------|----------|
| C1 | Source code compilation error | CRITICAL |
| C2 | Test file compilation error | HIGH |

---

## Layer 1: Data

### Scan procedure

1. **Detect ORM** from Step 0 → select entity/model file patterns from `qa-shared/reference.md` ORM / Data Layer Patterns table
2. **Read** each model/entity file → extract columns: name, type, nullable, length, unique, default, precision
3. **Read** ALL migrations **in timestamp order** → track cumulative schema changes (ADD COLUMN, DROP COLUMN, ALTER COLUMN) to derive the **final DB schema state**. Never compare entity against a single migration — always against the final cumulative result.
4. **Cross-reference** model columns vs **final migration schema** (bidirectional: entity→migration AND migration→entity). Apply service profile from Step 0e to classify mismatches.
5. **Find** DTO/serializer files → check each column appears in create/update paths
6. **Find** service/repository files → check if column is written programmatically
7. **Grep** raw queries → verify soft-delete filter applied where applicable

### Checks

| # | Check | Severity |
|---|-------|----------|
| D1 | Model/Entity column not in any migration | CRITICAL |
| D2 | Migration column with no model mapping (see D2 classification guide below) | HIGH* |
| D3 | Nullable setting differs between model and migration | HIGH |
| D4 | Column type differs between model and migration | HIGH |
| D5 | FK column has no index | HIGH |
| D6 | Parent deletable but child has no cascade/SET NULL config | MEDIUM |
| D7 | Semantically unique field (email, username, slug) lacks unique constraint | MEDIUM |
| D8 | Soft-delete enabled but some queries don't filter deleted rows | MEDIUM |
| D9 | Column never written by any DTO/serializer or service (dead column) | HIGH |
| D10 | Column in response DTO but never populated (always null) | MEDIUM |
| D11 | Decimal/float column without precision/scale | MEDIUM |
| D12 | Default value in model but not in migration (or vice versa) | MEDIUM |

#### D2 Classification Guide

D2 flags migration columns that have no corresponding entity/model property. **Not all ghost columns are problems.** Before classifying, determine the column's category:

| Category | Examples | Decision | Rationale |
|----------|----------|----------|-----------|
| **Standard extensibility pattern** | `social_login_type`, `oauth_provider`, `stripe_customer_id`, `two_factor_secret` | **SKIP** | Common SaaS/auth features. Column is harmless (nullable, unused by ORM), and removing it destroys future extensibility. Removing + re-adding costs 2 migrations for no runtime benefit. |
| **Orphaned from deleted feature** | Column added for a feature that was explicitly removed (feature flag dropped, module deleted, PR reverted) | **FIX** — generate drop migration | Dead weight with no future intent. Check git history for deliberate removal signals. |
| **Scaffold/demo leftover** | Columns from boilerplate generators, tutorial code, or demo modules (e.g., `features` table from starter kit) | **MANUAL** | Ask whether the demo module itself should be removed. If module stays, columns stay. |
| **External system dependency** | Columns written by triggers, ETL pipelines, reporting tools, or other services outside the ORM | **SKIP** | ORM doesn't manage these; absence of entity mapping is expected. |
| **Truly unknown** | Cannot determine origin or intent from code, git history, or project docs | **MANUAL** | Escalate for human review rather than auto-deleting. |

**Key principle:** A ghost column that is nullable, has no index cost, and follows a recognizable pattern is NOT a defect — it is latent infrastructure. The cost of keeping it (a few bytes per row) is far lower than the cost of removing and re-adding it across environments.

#### D8 Severity Clarification

D8 default severity is MEDIUM, but **escalate to CRITICAL when the query directly serves user-facing data** (e.g., chat messages, room lists, activity feeds). Soft-deleted records appearing in user-visible UI is a runtime bug, not just a code hygiene issue. Classify as:
- **CRITICAL** — query result is rendered in UI or returned in API response to end users
- **MEDIUM** — query is used internally (analytics, background jobs, admin-only reports)

---

## Layer 2: API

### Scan procedure

1. **Detect backend framework** → select controller/route file patterns from `qa-shared/reference.md` Backend Framework Detection table
2. **Read** each endpoint → extract: HTTP method, route path, input DTO/schema, guards/middleware
3. **Detect frontend apps** (Step 0b) → for EACH app:
   - Find API service files → extract: URL, HTTP method, request params, response types
4. **Cross-reference** each frontend call against backend endpoints
5. **Check** list endpoints for pagination, search, sort params
6. **Check** service methods for error handling (not-found, conflict, validation)
7. **Check** route declaration order (static before parameterized)

### Checks

| # | Check | Severity |
|---|-------|----------|
| A1 | Resource missing CRUD operations | HIGH |
| A2 | List endpoint has no pagination | HIGH |
| A3 | List endpoint has no search/filter support | MEDIUM |
| A4 | List endpoint has no sort/order support | LOW |
| A5 | Missing error responses (not-found, conflict, validation) | HIGH |
| A6 | Uses hard delete when model supports soft delete | MEDIUM |
| A7 | Update/Delete doesn't verify resource ownership | HIGH |
| A8 | Backend endpoint with no frontend caller (across ALL apps) | MEDIUM |
| A9 | Frontend API path ≠ backend route | CRITICAL |
| A10 | Frontend HTTP method ≠ backend method | CRITICAL |
| A11 | Backend requires fields frontend doesn't send | HIGH |
| A12 | Frontend sends fields backend doesn't accept | MEDIUM |
| A13 | Frontend response type ≠ actual backend response shape | MEDIUM |
| A14 | Static route shadowed by parameterized route declared before it | CRITICAL |
| A15 | Inconsistent response wrapper format across endpoints | MEDIUM |
| A16 | Resource with list selection but no bulk operations | LOW |
| A17 | Model has status/active field but no toggle endpoint | MEDIUM |
| A18 | Same feature exists in multiple apps but API integration differs | HIGH |

---

## Layer 3: Auth & Security

### Scan procedure

1. **Detect auth pattern** → find guard/middleware/decorator files based on detected backend framework
2. **List** all endpoints → check each has auth protection or explicit public marker
3. **Detect frontend apps** → for EACH app: check route protection (auth guard, redirect)
4. **Grep** for token storage patterns: `localStorage.*token`, `sessionStorage.*token`
5. **Grep** for sensitive data in responses: password, secret, token in DTOs/serializers
6. **Grep** for rate limiting config and verify it's actually active (not just declared)
7. **Grep** for injection vectors: raw SQL with string interpolation, `innerHTML`/`dangerouslySetInnerHTML`/`v-html`
8. **Grep** for sensitive data in logs: `console.log`, `logger.*password`, `print.*token`
9. **Grep** for insecure randomness: `Math.random` in auth/security context
10. **Check** file upload endpoints for type/size validation
11. **Check** CORS configuration

### Checks

| # | Check | Severity |
|---|-------|----------|
| S1 | Endpoint has no auth protection and is not explicitly public | CRITICAL |
| S2 | Endpoint needs role restriction but has none | HIGH |
| S3 | Frontend page accessible without auth redirect | HIGH |
| S4 | Frontend shows action for role backend doesn't allow | HIGH |
| S5 | Auth token stored in localStorage/sessionStorage (not HTTP-only cookie) | CRITICAL |
| S6 | API response includes password hash, tokens, or secrets | MEDIUM |
| S7 | Auth endpoints (login/register) have no rate limiting | MEDIUM |
| S8 | Rate limiter declared but not globally registered (dead code) | CRITICAL |
| S9 | OTP/token verification has no brute-force protection | HIGH |
| S10 | SQL/NoSQL injection vector (unsanitized input in query) | CRITICAL |
| S11 | XSS vector (unsanitized HTML rendering) | HIGH |
| S12 | File upload with no type/size restriction | MEDIUM |
| S13 | Sensitive data in application logs | HIGH |
| S14 | Debug/test routes exposed in production config | MEDIUM |
| S15 | `Math.random()` for security-sensitive values | HIGH |
| S16 | CORS allows all origins or is misconfigured | MEDIUM |
| S17 | Missing request body size limit | LOW |
| S18 | Same role/permission definitions inconsistent across apps | HIGH |

---

## Layer 4: Inputs

### Scan procedure

1. **Detect validation stack** from Step 0 (class-validator, Zod, Yup, Pydantic, etc.)
2. **Read** model/entity → extract each field: name, type, nullable, length, enum, default, precision
3. **Read** create/update DTOs or serializers → extract validated fields and their decorators/rules
4. **Find** frontend validation schemas (grep `z.object`, `yup.object`, `Joi.object`, `v.object`)
5. **Read** each schema → extract: field name, type, validators (min, max, optional, email, regex)
6. **Find** form components (grep `<form`, `useForm`, `useFormContext`, `FormKit`, `<Formik`)
7. **Read** each form → extract: input names, types, required/optional, maxLength, labels, error display
8. **Cross-reference ALL layers** for each field: Model ↔ DTO ↔ Schema ↔ Form
9. **Semantic check**: field named `*email*` → email validation? `*password*` → min length? `*phone*` → format? `*url*` → URL validation? `*score*`/`*rating*` → min/max range?

### Checks

| # | Check | Severity |
|---|-------|----------|
| I1 | Backend "required" but model allows null (nullable conflict) | HIGH |
| I2 | Required field has no visual indicator (`*`) in form | HIGH |
| I3 | Optional field incorrectly shows required indicator | MEDIUM |
| I4 | Field name mismatch across model/DTO/schema/form | MEDIUM |
| I5 | maxLength differs across layers | HIGH |
| I6 | minLength differs across layers | MEDIUM |
| I7 | Enum/choice values differ between backend and frontend | MEDIUM |
| I8 | Numeric model field but text input in form | HIGH |
| I9 | Date model field but plain text input in form | HIGH |
| I10 | Boolean model field but no toggle/checkbox in form | HIGH |
| I11 | Model field with no form input (DB orphan) | HIGH |
| I12 | Form field with no backend field (frontend orphan) | MEDIUM |
| I13 | Email field without email validation in backend AND frontend | HIGH |
| I14 | Password field allows < 8 characters | HIGH |
| I15 | Phone field without format validation | HIGH |
| I16 | URL field without URL validation | MEDIUM |
| I17 | String field has no maxLength in any layer | HIGH |
| I18 | Rich text / HTML input without sanitization (XSS) | CRITICAL |
| I19 | No submit-button disable during request (double submit) | MEDIUM |
| I20 | Validation only on submit (not real-time onChange/onBlur) | HIGH |
| I21 | Date pair (start/end) missing cross-field validation | HIGH |
| I22 | Validation error not displayed adjacent to field | HIGH |
| I23 | Select/dropdown with no default or placeholder | MEDIUM |
| I24 | Score/rating/percentage field missing min/max range | HIGH |
| I25 | Decimal field without precision in validation | MEDIUM |
| I26 | Required field accepts whitespace-only value | MEDIUM |
| I27 | Array/multi-select field missing min/max items | MEDIUM |
| I28 | Regex validation without user-visible error message | LOW |
| I29 | Formatted field (phone, date) without input hint/placeholder | LOW |
| I30 | Formatted field without real-time input mask/filter | MEDIUM |

---

## Layer 5: UI/UX

### Scan procedure

1. **Detect frontend framework** and locate ALL app directories (Step 0b)
2. For EACH frontend app, **glob** all page/component files (`.tsx`, `.jsx`, `.vue`, `.svelte`)
3. **Component inventory**: glob shared/common component directories (`components/ui/`, `components/shared/`, `components/common/`, `ui/`) → list all reusable components and their roles (Loading, Empty, Modal, Button, Input, Header, Card, etc.)
4. For data-fetching components (detect via: `useQuery`, `useSWR`, `useFetch`, `fetch(`, `axios.`, `$http`, `HttpClient`):
   - Check loading state, error state, empty state, retry mechanism
5. For buttons/interactive elements (`<button>`, `<Button>`, `<a>`, `<Link>`, `<router-link>`):
   - Check handlers, route validity, loading/disabled states, confirmation dialogs, aria-labels
6. For modals/dialogs (detect via: `Dialog`, `Modal`, `Drawer`, `Sheet`, `<dialog>`, `showModal`):
   - Check ESC, backdrop, portal, scroll lock, focus trap, dirty form warning
7. For list/table pages (detect via: `Table`, `DataTable`, `DataGrid`, `<table>`, `.map()` rendering):
   - Check pagination, search, sort, empty state, URL state sync, bulk actions
8. For navigation: trace form-submit redirects, back-button behavior, unsaved-form warnings
9. **Design token consistency**: collect page header patterns across all pages — extract font size, font weight, padding, margin, color. Compare for uniformity. Also check section headers, card headers, modal headers.
10. **Component reuse audit**: for each shared component found in step 3, grep all page files for hardcoded alternatives that should use the shared component instead
11. For a11y: semantic HTML, keyboard nav, aria attributes, labels
12. For performance: code splitting, memoization, image optimization

### Checks — States & Data

| # | Check | Severity |
|---|-------|----------|
| U1 | Data fetch with no loading indicator | HIGH |
| U2 | Data fetch with no error state | HIGH |
| U3 | Error state with no retry mechanism | MEDIUM |
| U4 | List/table with no empty state | HIGH |
| U5 | Skeleton layout doesn't match actual content | MEDIUM |
| U6 | Optimistic update with no rollback on failure | HIGH |
| U7 | Race condition: navigation causes stale data | MEDIUM |
| U8 | Loading state stuck (no timeout/error fallback) | HIGH |
| U9 | State not cleaned up on unmount (memory leak) | MEDIUM |
| U10 | No feedback after mutation (missing toast/notification) | MEDIUM |

### Checks — Buttons & Links

| # | Check | Severity |
|---|-------|----------|
| U11 | Button with no handler and no form role | CRITICAL |
| U12 | Async button with no loading/disabled state | HIGH |
| U13 | Link to undefined route | HIGH |
| U14 | Anchor with `href="#"` or `javascript:void` | HIGH |
| U15 | Destructive action with no confirmation | MEDIUM |
| U16 | Submit button not disabled when form is invalid | MEDIUM |
| U17 | Same API call triggered from multiple buttons on one page | LOW |

### Checks — Modals & Dialogs

| # | Check | Severity |
|---|-------|----------|
| U18 | Modal doesn't close on ESC | HIGH |
| U19 | Modal doesn't close on backdrop click | MEDIUM |
| U20 | Modal not rendered via portal (z-index/stacking issue) | HIGH |
| U21 | Scroll not locked when modal open | MEDIUM |
| U22 | Focus not trapped inside modal | HIGH |
| U23 | Focus not returned to trigger element on close | MEDIUM |
| U24 | Form in modal: close without save doesn't warn | MEDIUM |
| U25 | Submit + close race condition | MEDIUM |
| U26 | Multiple modals conflict (z-index, scroll, ESC) | MEDIUM |

### Checks — Lists & Tables

| # | Check | Severity |
|---|-------|----------|
| U27 | List page missing pagination | HIGH |
| U28 | List page missing search | MEDIUM |
| U29 | List page missing sort | MEDIUM |
| U30 | No default sort order | MEDIUM |
| U31 | Filter/pagination state not synced to URL | HIGH |
| U32 | Pagination resets on filter change | MEDIUM |
| U33 | Bulk actions enabled when no rows selected | MEDIUM |
| U34 | Overflow text not truncated | LOW |
| U35 | Stale list data after mutation elsewhere | MEDIUM |

### Checks — Navigation

| # | Check | Severity |
|---|-------|----------|
| U36 | Back button navigates incorrectly | HIGH |
| U37 | Unsaved form changes not warned on navigate away | MEDIUM |
| U38 | Post-submit redirects to wrong page | MEDIUM |
| U39 | Push used where replace needed (redirect after action) | MEDIUM |
| U40 | Auth redirect corrupts browser history stack | MEDIUM |

### Checks — Accessibility

| # | Check | Severity |
|---|-------|----------|
| U41 | Icon-only button missing aria-label | MEDIUM |
| U42 | Clickable div/span without role="button" + keyboard support | MEDIUM |
| U43 | Form input not associated with label | MEDIUM |
| U44 | Interactive element unreachable by keyboard | HIGH |
| U45 | Non-semantic HTML (div instead of nav, main, section, button) | MEDIUM |
| U46 | Image missing alt text | MEDIUM |
| U47 | Dynamic content not announced (missing aria-live) | LOW |

### Checks — Component Reuse

| # | Check | Severity |
|---|-------|----------|
| U48 | Shared loading component exists but page uses hardcoded loading UI | HIGH |
| U49 | Shared empty state component exists but page uses inline empty message | HIGH |
| U50 | Shared modal/dialog component exists but page builds its own | HIGH |
| U51 | Shared form input component exists but page uses raw HTML input | MEDIUM |
| U52 | Shared button component exists but page uses raw `<button>` with custom styling | MEDIUM |
| U53 | Shared card/panel component exists but page hardcodes wrapper with same styles | MEDIUM |
| U54 | Shared confirmation dialog exists but page uses `window.confirm()` | HIGH |
| U55 | Utility function exists (date format, API call wrapper) but logic is duplicated inline | MEDIUM |

### Checks — Design Token & Visual Consistency

| # | Check | Severity |
|---|-------|----------|
| U56 | Page header font size inconsistent across pages (e.g., some `text-2xl`, some `text-xl`) | HIGH |
| U57 | Page header font weight inconsistent (e.g., some `font-bold`, some `font-semibold`) | MEDIUM |
| U58 | Page header padding/margin inconsistent across pages | MEDIUM |
| U59 | Section header style inconsistent within same app | MEDIUM |
| U60 | Card/panel header style inconsistent (font, padding, background, border) | MEDIUM |
| U61 | Modal header style inconsistent across modals | MEDIUM |
| U62 | Primary action button style inconsistent (size, color, radius) | MEDIUM |
| U63 | Spacing between sections inconsistent (e.g., some `gap-4`, some `gap-6`, some `mb-8`) | MEDIUM |
| U64 | Page container max-width or padding inconsistent | MEDIUM |
| U65 | Same feature in multiple apps has inconsistent layout/styling | HIGH |
| U66 | Responsive layout breaks at standard breakpoints | HIGH |
| U67 | Color values hardcoded instead of using theme/design tokens | MEDIUM |
| U68 | Border/corner radius style inconsistent across same-type elements | LOW |

### Checks — Performance

| # | Check | Severity |
|---|-------|----------|
| U69 | Route-level code splitting missing | MEDIUM |
| U70 | Large library imported whole (not tree-shaken) | MEDIUM |
| U71 | Unnecessary re-renders (missing memoization) | MEDIUM |
| U72 | Image missing dimensions (CLS) | MEDIUM |
| U73 | Below-fold images not lazy loaded | LOW |

---

## Layer 6: i18n

### Scan procedure

1. **Detect i18n** → grep for: `i18next`, `react-intl`, `vue-i18n`, `$t(`, `t(`, `gettext`, `_(`
2. If i18n library found:
   - **Glob** translation files (`.json`, `.po`, `.yaml` in `locales/`, `i18n/`, `translations/`)
   - **List** all translation keys defined
   - **Grep** all components for translation function calls → list all keys used
   - Cross-reference: defined keys vs used keys
3. **Grep** all components for hardcoded user-facing strings:
   - Text content in JSX/HTML that is NOT inside a translation function
   - String props like `placeholder=`, `title=`, `label=`, `aria-label=` with literal text
   - Exclude: CSS class names, IDs, event names, technical strings
4. **Check** date/number formatting for locale-awareness

### Checks

| # | Check | Severity |
|---|-------|----------|
| N1 | User-facing string hardcoded (not using translation function) | MEDIUM |
| N2 | Translation key used but not defined in any locale file | HIGH |
| N3 | Translation key defined but never used (dead key) | LOW |
| N4 | Translation exists in one locale but missing in another | HIGH |
| N5 | Date displayed without locale-aware formatting | MEDIUM |
| N6 | Number/currency displayed without locale-aware formatting | MEDIUM |
| N7 | Error message hardcoded (not translatable) | MEDIUM |
| N8 | Plural forms not handled (hardcoded "item" vs "items") | LOW |

---

## Execution

### Phase 1: DIAGNOSE

**1a.** Run Step 0 (detection + context + compile check). Report detected stack and any compile errors found in Step 0f.

**1b.** Determine scope:
- `[module]` → that module only. Omit → all modules.
- `--check` → those layers only. Omit → all layers.
- `--diff` → only files from `git diff`. Auto-select affected layers.
- `--lang` → output language. Default: `en`.

**1c.** For each layer in scope: follow scan procedure, read files, cross-reference, evaluate every check. **Before classifying any issue, apply the service profile from Step 0e.** Classify each issue:

- **FIX** — clear from code AND service profile how to fix
- **SKIP** — intentional (documented in CLAUDE.md, project docs, OR inferable from service profile). If the service profile makes the issue irrelevant, SKIP it — do not escalate to MANUAL.
- **MANUAL** — genuinely ambiguous even after applying service profile

**Service profile gates ALL classifications.** A check that is technically flaggable but irrelevant to the service's nature is SKIP, not MANUAL. Examples:
- Data layer: migrations must be evaluated as cumulative final state, not per-file. A column added then dropped = no issue.
- API layer: pagination severity depends on expected data volume (admin lookup table with 10 rows ≠ user-facing list with 10K rows).
- Auth layer: security token hard-delete is correct behavior, not a soft-delete violation.
- i18n layer: single-language service → all multi-locale checks are SKIP.
- UI layer: feature complexity determines what's "missing" vs "unnecessary".

**1d.** Output Phase 1 report:

```markdown
# QA Fix — Phase 1: Diagnosis

Stack: {backend} + {frontend} + {ORM} | Apps: {list}
Module: {name or "All"} | Layers: {list} | Mode: {full / diff ref}
Issues: {total} ({fix} fix, {skip} skip, {manual} manual)

| # | ID | Issue | File | Decision |
|---|----|-------|------|----------|
```

If `--check-only` → stop. Otherwise → Phase 2.

---

### Phase 2: FIX

**2a.** Group FIX issues by file. Apply in dependency order:

1. **Database** (D-checks): Models/entities, migrations
2. **Backend** (A/S-checks): Controllers/views, services, DTOs/serializers, guards/middleware
3. **Input chain** (I-checks): Validation schemas, TypeScript/Python types
4. **Frontend** (U/N-checks): Components, pages, routes, translations

**2b.** For each file: read → apply ALL fixes in one pass → next file.

**2c.** After backend fixes: run project's type-check/compile command. Fix errors.

**2d.** After frontend fixes: run project's build/type-check command. Fix errors.

**2e.** Run linter (detect from package.json scripts or project config). Fix errors.

**2f.** Output Phase 2 report.

---

### Phase 3: VERIFY

**3a.** Re-read ONLY modified files. Re-check ONLY their flagged items.

**3b.** Classify: RESOLVED / UNRESOLVED / REGRESSION.

**3c.** Regressions → fix once (max 1 retry).

**3d.** Output final report:

```markdown
# QA Fix — Complete

Files modified: {N} | Resolved: {N} | Unresolved: {N} | Regressions: {N}

| # | ID | Issue | Status |
|---|----|-------|--------|
```

---

## Fix Patterns

### Data
| Issue | Fix |
|-------|-----|
| Column not in migration | Generate migration with the column |
| FK without index | Add index to FK column |
| Missing cascade | Add cascade/SET NULL based on relationship semantics |

### API
| Issue | Fix |
|-------|-----|
| Missing soft delete | Replace hard delete with soft delete method |
| Missing error response | Add existence/validation checks |
| Route mismatch | Align frontend path to backend route |
| Missing pagination | Add page/limit params to list endpoint |

### Auth & Security
| Issue | Fix |
|-------|-----|
| No auth protection | Add auth guard/middleware or mark as public |
| No role restriction | Add role check based on project role definitions |
| Sensitive in response | Exclude sensitive fields from response |
| Injection vector | Use parameterized queries / sanitize input |

### Inputs
| Issue | Fix |
|-------|-----|
| Constraint mismatch | Align constraints across all layers (model → DTO → schema → HTML) |
| Missing required indicator | Add `*` or required component to label |
| Missing validation | Add validator in both backend and frontend |

### UI/UX
| Issue | Fix |
|-------|-----|
| Missing loading/error/empty state | Add appropriate state handling |
| Modal issues | Add portal, ESC handler, focus trap, scroll lock |
| Missing URL sync | Sync filter/page state with URL query params |
| Missing a11y | Add aria-label, role, keyboard handlers, semantic HTML |
| Hardcoded UI instead of shared component | Replace with shared component import |
| Inconsistent header/spacing/style | Align to the most common pattern across pages, or to design tokens if defined |
| Hardcoded color values | Replace with theme variable/design token |

### i18n
| Issue | Fix |
|-------|-----|
| Hardcoded string | Wrap with translation function, add key to locale files |
| Missing translation | Add missing key to all locale files |
| Dead translation key | Remove from locale files |

---

## Safety Rules

1. **Read before edit** — Never modify without reading first
2. **One file, one pass** — All fixes for a file in single edit
3. **Type check between layers** — Run project's compile/check command after each layer
4. **No destructive changes** — Never delete endpoints, models, columns
5. **Migration safety** — Generate but do NOT auto-run
6. **Preserve behavior** — Don't break working functionality
7. **Intent-gated additions** — New endpoints/components only if referenced in project docs, else MANUAL
8. **Max 1 regression retry** — Fix regressions once, then report
9. **Multi-app consistency** — When fixing a feature, check and fix across ALL detected frontend apps
10. **Respect project rules** — CLAUDE.md rules from Step 0d override default severity. If project says "always X", treat violation as CRITICAL regardless of default.
