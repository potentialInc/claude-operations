# QA Shared Reference

Common rules, scoring, and conventions shared by all QA skills.

This document defines shared rules, detection patterns, scoring mechanisms, and output conventions used by all `qa-*` skills. Individual QA skills reference this document to avoid duplication and ensure consistency.

---

## Execution Rule: Self-Verification & Auto-Retry

**Every Step in every QA skill MUST follow this rule:**

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

Step N: ALL PASSED (N/N) → Proceeding to Step N+1
```

---

## File Validation (Common)

All QA skills validate the input file identically:

1. File path is provided
2. File exists and is readable
3. File extension is supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-<skill> <page-or-component-file-path> [options]
```

---

## Frontend Framework Detection (Common)

All QA skills use the same framework detection logic. Read the target file and project's `package.json`:

| Signal | Framework |
|--------|-----------|
| `import ... from 'react'` or JSX/TSX + `react` in package.json | React |
| `<template>` + `<script>` + `.vue` extension | Vue |
| `@Component` decorator + `@angular/core` in package.json | Angular |
| `.svelte` extension | Svelte |
| `<form>` + no framework imports + `.html` extension | Plain HTML |

---

## Backend Framework Detection (Common)

Skills with `--backend` or `--depth full` support use this detection:

| Signal | Framework |
|--------|-----------|
| `@nestjs/common` in package.json | NestJS |
| `express` in package.json (no nest) | Express |
| `manage.py` in root | Django |
| `pom.xml` or `build.gradle` with spring-boot | Spring Boot |
| `composer.json` with laravel | Laravel |

If `--backend` path provided, use that path. Otherwise, look for sibling `backend/` directory.

---

## Configuration File Support

All QA skills check for an optional `.qa-config.json` file at the project root:

```json
{
  "outputDir": ".claude-project/qa",
  "defaultDepth": "full",
  "backendPath": "backend/src",
  "scoring": {
    "passThreshold": 80,
    "needsAttentionThreshold": 60,
    "criticalPenalty": -15,
    "warningPenalty": -5,
    "maxBonus": 10
  },
  "ignore": {
    "files": ["**/test/**", "**/__mocks__/**"],
    "rules": []
  },
  "framework": {
    "frontend": "react",
    "backend": "nestjs",
    "uiLibrary": "shadcn"
  }
}
```

All fields are optional. Missing fields use defaults. If no config file exists, all defaults apply.

---

## Score Calculation (Standardized)

All QA skills use this scoring system:

### Base Rules

```
Base Score: 100
Deductions:
  - CRITICAL issue: -15 points each
  - WARNING issue: -5 points each
  - INFO issue: 0 points (no deduction)

Score = max(0, 100 - sum(deductions))

Bonus:
  - Bonuses are capped at +10 total (score cannot exceed 100)
  - Bonus items are skill-specific (see individual skills)

Final Score = min(100, Score + Bonus)
```

### Severity Definitions

| Severity | Description | Deduction |
|----------|-------------|-----------|
| **CRITICAL** | Security risk, data loss, UX-breaking bug, accessibility barrier | -15 points |
| **WARNING** | Potential bug, missing feature, degraded UX, inconsistency | -5 points |
| **INFO** | Nice-to-have improvement, documentation note, minor polish | 0 points |

### Status Thresholds

| Status | Score Range | Meaning |
|--------|------------|---------|
| **PASS** | >= 80 | Meets quality standards; minor improvements optional |
| **NEEDS ATTENTION** | 60-79 | Usable but has notable gaps; should address before release |
| **FAIL** | < 60 | Significant issues; must fix before release |

### Coverage Multiplier (Optional)

When analysis is incomplete (e.g., only 2 of 5 tables analyzed because imports unresolvable):

```
Adjusted Category Score = Raw Category Score × (analyzed_items / total_items)
```

This prevents inflated scores from partial analysis.

---

## Common Error Handling

All QA skills handle these common errors:

| Error | Cause | Solution |
|-------|-------|----------|
| File not found | Invalid path provided | Check file path and try again |
| Unsupported file type | Wrong extension | Supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js` |
| Framework not detected | Unusual import structure or no package.json | Falls back to generic pattern scanning |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| Circular imports | Complex import chain | Breaks after 10 levels; notes as "unresolvable" |
| package.json not found | No package manager | Falls back to import-based detection only |

Skill-specific errors are listed in each skill's Error Handling section.

---

## Output Directory & File Naming

### Report Directory

Default: `.claude-project/qa/`
Override: `outputDir` in `.qa-config.json`

### File Naming Convention

```
[ComponentName]_[Feature]_QA_Report_[YYMMDD].md    — Full report
[ComponentName]_[Feature]_TestCases_[YYMMDD].md     — Test cases (if large)
```

Feature suffixes by skill:
| Skill | Feature Suffix |
|-------|---------------|
| `qa-back-nav` | `BackNav` |
| `qa-inputs` | `InputFields` |
| `qa-states` | `LoadingErrorEmpty` |
| `qa-modal` | `ModalDrawer` |
| `qa-auth` | `PermissionRole` |
| `qa-list` | `TableList` |
| `qa-a11y` | `Accessibility` |
| `qa-performance` | `Performance` |
| `qa-screen` | `FullPage` |
| `qa-buttons` | `Buttons` |
| `qa-api-sync` | `ApiSync` |
| `qa-crud` | `CRUD` |
| `qa-db-integrity` | `DbIntegrity` |
| `qa-layout` | `Layout` |
| `qa-test-gen` | `TestGen` |
| `qa-security` | `Security` |

### Report Header Template

All reports use this header format:

```
+====================================================================+
|                   QA [Feature] Report                              |
+====================================================================+
|  Component:     [ComponentName]                                    |
|  File:          [file-path]                                        |
|  Framework:     [React + library / Vue + library / etc.]           |
|  Backend:       [NestJS / Express / N/A]  (if applicable)          |
|  Report Date:   [YYYY-MM-DD HH:MM]                                |
|  Depth:         [full / shallow]                                   |
|  Overall Score: [N]/100                                            |
|  Status:        [PASS | NEEDS ATTENTION | FAIL]                    |
+====================================================================+
```

---

## Cross-Skill Dependency Matrix

When analyzing a page, these skills are commonly run together:

| If you find... | Also run... | Reason |
|---------------|-------------|--------|
| Forms inside modals | `qa-inputs` on modal component | Form validation coverage |
| Modals that push history | `qa-back-nav` | Back button may close modal or navigate |
| Tables with row actions | `qa-auth` | Row actions may be role-gated |
| Tables with data fetching | `qa-states` | Tables need loading/error/empty states |
| Permission-gated delete | `qa-modal` | Delete confirmation modal quality |
| Navigation after form submit | `qa-back-nav` | Post-submit redirect correctness |
| Button-heavy interfaces | `qa-buttons` | Button state and interaction coverage |
| API-driven pages | `qa-api-sync` | API synchronization and caching |
| CRUD operations | `qa-crud` | Create/Read/Update/Delete flow coverage |
| Database operations | `qa-db-integrity` | Data integrity and constraint validation |
| Complex layouts | `qa-layout` | Responsive and layout consistency |

Use `qa-screen` to automatically run all relevant skills for a page.

---

## Supported Extensions

All QA skills support: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`

---

## Read-Only Guarantee

Individual QA skills (qa-auth, qa-crud, etc.) are **read-only** — they analyze and report only.

**Exception:** `qa-fix` is the only QA skill that modifies files (Phase 2: Fix). It follows Safety Rules defined in its own skill.md.

---

## Framework-Specific Pattern Mappings

All QA skills MUST use the detected framework to select the correct scanning patterns from the tables below. Never hardcode patterns for a single framework — always branch based on detection results. When detection fails, use the **Generic Fallback** row.

### Router / Navigation Patterns

| Framework | Router Import | Programmatic Navigate | Replace (no history) | Go Back | Route Guard / Auth Redirect |
|-----------|--------------|----------------------|---------------------|---------|----------------------------|
| React Router | `import { useNavigate } from 'react-router-dom'` | `navigate('/path')` | `navigate('/path', { replace: true })` | `navigate(-1)` | `<Route>` wrapper or loader redirect |
| Next.js (App) | `import { useRouter } from 'next/navigation'` | `router.push('/path')` | `router.replace('/path')` | `router.back()` | Middleware `middleware.ts` or layout redirect |
| Next.js (Pages) | `import { useRouter } from 'next/router'` | `router.push('/path')` | `router.replace('/path')` | `router.back()` | `getServerSideProps` redirect |
| Vue Router | `import { useRouter } from 'vue-router'` | `router.push('/path')` | `router.replace('/path')` | `router.back()` or `router.go(-1)` | `beforeEach` navigation guard |
| Nuxt | Auto-imported `useRouter()`, `navigateTo()` | `navigateTo('/path')` | `navigateTo('/path', { replace: true })` | `router.back()` | `defineNuxtRouteMiddleware` |
| Angular | `import { Router } from '@angular/router'` | `this.router.navigate(['/path'])` | `this.router.navigate(['/path'], { replaceUrl: true })` | `this.location.back()` | `CanActivate` guard |
| SvelteKit | `import { goto } from '$app/navigation'` | `goto('/path')` | `goto('/path', { replaceState: true })` | `history.back()` | `+page.server.ts` load redirect |
| Generic Fallback | `window.location` or `history` API | `window.location.href = '/path'` | `window.location.replace('/path')` | `history.back()` | Check for redirect patterns in route handlers |

**Grep patterns for detecting navigation calls:**
```
React Router:    navigate\(|useNavigate|<Navigate |<Link
Next.js:         router\.push|router\.replace|router\.back|useRouter|<Link
Vue Router:      router\.push|router\.replace|router\.back|router\.go|useRouter|<router-link|<RouterLink
Nuxt:            navigateTo|useRouter|<NuxtLink
Angular:         this\.router\.navigate|this\.location\.back|routerLink|\[routerLink\]
SvelteKit:       goto\(|<a href
Generic:         window\.location|history\.pushState|history\.replaceState|history\.back
```

### CSS / Styling Framework Patterns

| Framework | Detection Signal | Class Syntax | Layout Patterns | Spacing Patterns |
|-----------|-----------------|-------------|-----------------|-----------------|
| Tailwind CSS | `tailwindcss` in devDependencies or `tailwind.config` file exists | Utility classes: `flex`, `p-4`, `gap-2` | `flex`, `grid`, `flex-col`, `items-center` | `p-{n}`, `m-{n}`, `gap-{n}`, `space-y-{n}` |
| CSS Modules | `*.module.css` or `*.module.scss` files exist | `styles.className` or `styles['class-name']` | Read `.module.css` for layout properties | Read `.module.css` for spacing values |
| Styled Components | `styled-components` in dependencies | `` styled.div`...` `` or `css` prop | Read template literal for `display`, `flex` | Read template literal for `padding`, `margin`, `gap` |
| Emotion | `@emotion/styled` or `@emotion/react` in dependencies | `` styled.div`...` `` or `css` prop | Same as Styled Components | Same as Styled Components |
| SCSS/SASS | `*.scss` or `*.sass` files with class usage | BEM-style: `.block__element--modifier` | Read `.scss` files for layout properties | Read `.scss` files for spacing values |
| Plain CSS | `*.css` files without module suffix | Standard class names | Read `.css` files for layout properties | Read `.css` files for spacing values |
| UnoCSS | `unocss` in devDependencies | Utility classes similar to Tailwind | Same as Tailwind | Same as Tailwind |
| Generic Fallback | None of the above detected | Inspect `className` or `class` attributes | Grep for `display:\s*flex|display:\s*grid` in stylesheets | Grep for `padding|margin|gap` in stylesheets |

**Layout consistency audit strategy by CSS framework:**
- **Utility-class frameworks** (Tailwind, UnoCSS): Compare `className` strings directly across same-type pages
- **CSS-in-JS** (Styled Components, Emotion): Read the styled definitions and compare CSS properties
- **External stylesheets** (CSS Modules, SCSS, Plain CSS): Resolve class → stylesheet → compare computed CSS properties
- **Generic fallback**: Grep for inline `style` attributes and compare values

### ORM / Data Layer Patterns

| ORM | Detection Signal | Entity/Model Files | Column/Field Definition | Relation Definition |
|-----|-----------------|-------------------|------------------------|-------------------|
| TypeORM | `typeorm` in dependencies | `**/*.entity.ts` | `@Column({ type, nullable, length })` | `@ManyToOne`, `@OneToMany`, `@JoinColumn` |
| Prisma | `@prisma/client` in dependencies | `prisma/schema.prisma` | `fieldName Type @db.VarChar(100)` | `relation` fields with `@relation` |
| Sequelize | `sequelize` in dependencies | `**/models/*.{js,ts}` | `DataTypes.STRING(100)`, `allowNull` | `belongsTo`, `hasMany`, `belongsToMany` |
| Mongoose | `mongoose` in dependencies | `**/*.schema.ts` or `**/*.model.ts` | `{ type: String, required: true, maxlength: 100 }` | `{ type: Schema.Types.ObjectId, ref: 'Model' }` |
| Django ORM | `django` in requirements.txt | `**/models.py` | `models.CharField(max_length=100, null=False)` | `models.ForeignKey`, `models.ManyToManyField` |
| SQLAlchemy | `sqlalchemy` in requirements.txt | `**/models.py` or `**/models/*.py` | `Column(String(100), nullable=False)` | `relationship()`, `ForeignKey` |
| Drizzle | `drizzle-orm` in dependencies | `**/schema.ts` or `**/schema/*.ts` | `varchar('name', { length: 100 })` | `relations()` |
| Eloquent (Laravel) | `laravel/framework` in composer.json | `app/Models/*.php` | `$fillable`, `$casts`, migration `$table->string('name', 100)` | `belongsTo()`, `hasMany()`, `belongsToMany()` |
| Generic Fallback | No ORM detected | Grep for SQL files, raw queries | Look for CREATE TABLE statements | Look for FOREIGN KEY constraints |

### DTO / Validation Layer Patterns

| Framework | Detection Signal | DTO/Schema Files | Validation Decorators/Methods | Partial/Pick Types |
|-----------|-----------------|-----------------|------------------------------|-------------------|
| class-validator (NestJS) | `class-validator` in dependencies | `**/*.dto.ts` | `@IsString()`, `@MinLength(n)`, `@IsOptional()` | `PartialType()`, `PickType()`, `OmitType()` |
| Zod | `zod` in dependencies | Grep for `z.object` in `**/*.{ts,tsx}` | `.string()`, `.min(n)`, `.max(n)`, `.optional()` | `.partial()`, `.pick()`, `.omit()` |
| Yup | `yup` in dependencies | Grep for `yup.object` in `**/*.{ts,tsx}` | `.string()`, `.min(n)`, `.max(n)`, `.notRequired()` | N/A (manual) |
| Joi | `joi` in dependencies | Grep for `Joi.object` in `**/*.{ts,js}` | `Joi.string()`, `.min(n)`, `.max(n)`, `.optional()` | N/A (manual) |
| Django Serializers | `rest_framework` in requirements.txt | `**/serializers.py` | `CharField(max_length=100, required=True)` | `fields = '__all__'` or explicit list |
| Laravel Requests | `laravel/framework` in composer.json | `app/Http/Requests/*.php` | `'name' => 'required\|string\|max:100'` | Rule arrays in `rules()` method |
| Spring Validation | `spring-boot-starter-validation` in pom.xml | `**/dto/*.java` or `**/request/*.java` | `@NotBlank`, `@Size(min, max)`, `@Valid` | N/A (separate classes) |
| Valibot | `valibot` in dependencies | Grep for `v.object` in `**/*.{ts,tsx}` | `v.string()`, `v.minLength(n)`, `v.optional()` | `v.partial()`, `v.pick()`, `v.omit()` |
| Generic Fallback | No validation library detected | Grep for manual if/throw validation | Look for manual length/type checks | N/A |

### Form Library Patterns

| Library | Detection Signal | Form Hook/Setup | Field Registration | Submit Handler |
|---------|-----------------|----------------|-------------------|---------------|
| React Hook Form | `react-hook-form` in dependencies | `useForm()`, `useFormContext()` | `register('field')` or `<Controller>` | `handleSubmit(onSubmit)` |
| Formik | `formik` in dependencies | `useFormik()` or `<Formik>` | `<Field name="field">` | `onSubmit` prop |
| VeeValidate (Vue) | `vee-validate` in dependencies | `useForm()`, `useField()` | `useField('field')` or `<Field>` | `handleSubmit(onSubmit)` |
| FormKit (Vue) | `@formkit/vue` in dependencies | `<FormKit type="form">` | `<FormKit type="text" name="field">` | `@submit` event |
| Angular Reactive Forms | `@angular/forms` in package.json | `FormBuilder`, `FormGroup`, `FormControl` | `formControlName="field"` | `(ngSubmit)="onSubmit()"` |
| Angular Template Forms | `@angular/forms` in package.json | `ngModel` | `[(ngModel)]="field"` | `(ngSubmit)="onSubmit()"` |
| Svelte Forms | Native or `svelte-forms-lib` | `bind:value` or `createForm()` | `bind:value={field}` | `on:submit` |
| Plain HTML | No form library | `<form>` element | `<input name="field">` | `form.addEventListener('submit')` or `onsubmit` |
| Generic Fallback | Unknown library | Grep for `<form` or form submission | Grep for `name=` attributes on inputs | Grep for submit handlers |

### Table / List Library Patterns

| Library | Detection Signal | Table Component | Column Definition | Sorting | Pagination |
|---------|-----------------|----------------|-------------------|---------|-----------|
| TanStack Table (React) | `@tanstack/react-table` in dependencies | `useReactTable()`, `flexRender` | `columnHelper.accessor()` or column defs array | `getSortedRowModel()` | `getPaginationRowModel()` |
| TanStack Table (Vue) | `@tanstack/vue-table` in dependencies | `useVueTable()`, `FlexRender` | `columnHelper.accessor()` or column defs array | `getSortedRowModel()` | `getPaginationRowModel()` |
| AG Grid | `ag-grid-react` or `ag-grid-vue` in dependencies | `<AgGridReact>` or `<ag-grid-vue>` | `columnDefs` prop | `sortable: true` in colDef | `pagination: true` prop |
| Ant Design Table | `antd` in dependencies | `<Table>` | `columns` prop array | `sorter` in column def | `pagination` prop |
| Material UI Table | `@mui/material` in dependencies | `<Table>`, `<DataGrid>` | `<TableCell>` or `columns` for DataGrid | `<TableSortLabel>` or `sortModel` | `<TablePagination>` or `paginationModel` |
| Element Plus Table (Vue) | `element-plus` in dependencies | `<el-table>` | `<el-table-column>` | `sortable` prop | `<el-pagination>` |
| Vuetify Table | `vuetify` in dependencies | `<v-data-table>` | `headers` prop | Built-in with `headers` sort | Built-in pagination |
| PrimeVue Table | `primevue` in dependencies | `<DataTable>` | `<Column>` | `sortable` prop | `<Paginator>` or built-in |
| Angular Material Table | `@angular/material` in package.json | `<mat-table>` | `<ng-container matColumnDef>` | `matSort`, `matSortHeader` | `<mat-paginator>` |
| Plain HTML | No table library | `<table>` element | `<th>`, `<td>` | Manual click handlers | Manual page controls |
| Generic Fallback | Unknown library | Grep for `<table` or list rendering | Grep for column/header definitions | Grep for sort-related props/handlers | Grep for page/offset/limit patterns |

### Data Fetching Patterns

| Library | Detection Signal | Query Hook/Method | Mutation Hook/Method | Cache Invalidation |
|---------|-----------------|------------------|---------------------|--------------------|
| TanStack Query (React) | `@tanstack/react-query` in dependencies | `useQuery()`, `useSuspenseQuery()` | `useMutation()` | `queryClient.invalidateQueries()` |
| TanStack Query (Vue) | `@tanstack/vue-query` in dependencies | `useQuery()` | `useMutation()` | `queryClient.invalidateQueries()` |
| SWR | `swr` in dependencies | `useSWR()` | `useSWRMutation()` | `mutate()` |
| Apollo Client | `@apollo/client` in dependencies | `useQuery()`, `useLazyQuery()` | `useMutation()` | `refetchQueries`, `cache.modify()` |
| RTK Query | `@reduxjs/toolkit` with `createApi` | `useGetXQuery()` (generated) | `useUpdateXMutation()` (generated) | `invalidatesTags` |
| Axios + manual | `axios` in dependencies (no query lib) | `axios.get()` in `useEffect` or lifecycle | `axios.post()`, `axios.put()` | Manual state update or re-fetch |
| Fetch API | No HTTP library | `fetch()` in `useEffect` or lifecycle | `fetch()` with method POST/PUT/DELETE | Manual state update or re-fetch |
| Nuxt useFetch | `nuxt` in dependencies | `useFetch()`, `useAsyncData()` | `$fetch()` | `refreshNuxtData()`, `refresh()` |
| Angular HttpClient | `@angular/common/http` | `this.http.get()` | `this.http.post()` | Manual re-fetch via service |
| Generic Fallback | No pattern matched | Grep for HTTP calls in components | Grep for POST/PUT/DELETE calls | Grep for refetch/reload patterns |

### UI Component Library Detection

| Library | Detection Signal | Button | Modal/Dialog | Select/Dropdown | Toast/Notification |
|---------|-----------------|--------|-------------|----------------|--------------------|
| shadcn/ui | `@radix-ui/*` + `components/ui/` dir | `<Button>` | `<Dialog>`, `<AlertDialog>` | `<Select>`, `<Combobox>` | Sonner: `toast()` |
| Radix UI | `@radix-ui/*` in dependencies | `<Button>` | `<Dialog.Root>` | `<Select.Root>` | Custom |
| Material UI (MUI) | `@mui/material` in dependencies | `<Button>` | `<Dialog>`, `<Modal>` | `<Select>`, `<Autocomplete>` | `<Snackbar>`, `<Alert>` |
| Ant Design | `antd` in dependencies | `<Button>` | `<Modal>` | `<Select>` | `message.success()`, `notification.open()` |
| Chakra UI | `@chakra-ui/react` in dependencies | `<Button>` | `<Modal>` | `<Select>` | `toast()` |
| Element Plus | `element-plus` in dependencies | `<el-button>` | `<el-dialog>` | `<el-select>` | `ElMessage()`, `ElNotification()` |
| Vuetify | `vuetify` in dependencies | `<v-btn>` | `<v-dialog>` | `<v-select>` | `<v-snackbar>` |
| PrimeVue | `primevue` in dependencies | `<Button>` | `<Dialog>` | `<Dropdown>`, `<Select>` | `toast.add()` via ToastService |
| Headless UI | `@headlessui/react` or `@headlessui/vue` | N/A (unstyled) | `<Dialog>` | `<Listbox>`, `<Combobox>` | Custom |
| Angular Material | `@angular/material` in package.json | `<button mat-button>` | `MatDialog.open()` | `<mat-select>` | `MatSnackBar.open()` |
| Bootstrap | `bootstrap` or `react-bootstrap` in deps | `<Button>` / `<button class="btn">` | `<Modal>` / `.modal` | `<Form.Select>` / `<select class="form-select">` | Toast component or custom |
| Generic Fallback | No UI lib detected | `<button>` | `[role="dialog"]` or custom | `<select>` | Grep for toast/notification/alert patterns |

### Page File Patterns

| Framework | Page Files | Layout Files | Route Config |
|-----------|-----------|-------------|-------------|
| React Router | `pages/**/*.tsx` or route-defined components | `**/layout.tsx`, `**/Layout.tsx` | `createBrowserRouter()` or `<Route>` JSX |
| Next.js (App) | `app/**/page.tsx` | `app/**/layout.tsx` | File-system routing |
| Next.js (Pages) | `pages/**/*.tsx` | `pages/_app.tsx`, `pages/_document.tsx` | File-system routing |
| Vue Router | `views/**/*.vue` or `pages/**/*.vue` | `layouts/**/*.vue`, `App.vue` | `createRouter()` in `router/index.ts` |
| Nuxt | `pages/**/*.vue` | `layouts/**/*.vue` | File-system routing |
| Angular | `**/*.component.ts` | `**/*-layout.component.ts` | `Routes` array in `*-routing.module.ts` or `app.routes.ts` |
| SvelteKit | `src/routes/**/+page.svelte` | `src/routes/**/+layout.svelte` | File-system routing |
| Generic Fallback | Grep for components rendering `<main>` or full pages | Grep for components wrapping `children`/`<slot>`/`<Outlet>`/`<router-view>` | Grep for route definitions |

**How skills should use this section:**

1. Run framework detection (Frontend + Backend + CSS + ORM as needed)
2. Look up the detected framework in the relevant table above
3. Use the framework-specific patterns for scanning, grepping, and analysis
4. If the framework is not in the table, use the **Generic Fallback** row
5. Report the detected framework in the output header
