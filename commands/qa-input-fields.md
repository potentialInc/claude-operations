---
description: Analyze input fields in a page/component and generate a comprehensive QA test report covering validation, CRUD operations, and error messages
argument-hint: "<page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow]"
---

# QA Input Fields

Analyze all input fields in a frontend page/component, trace validation rules across the full stack (Frontend → Backend DTO → Database Entity), and generate a comprehensive QA report with test cases.

---

## Purpose

This command helps you:
1. **Discover all input fields** - Find every form input in a page/component
2. **Extract validation rules** - From frontend schemas, backend DTOs, and DB entities
3. **Trace CRUD operations** - Map data flow from form submit to database
4. **Detect validation gaps** - Find mismatches across the 3-layer stack
5. **Verify input restrictions** - Check type constraints, masks, and placeholder accuracy
6. **Audit error messages** - Verify existence, clarity, and placement of user-facing errors
7. **Generate test cases** - Produce actionable QA test checklists

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack tracing
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```bash
/qa-input-fields <page-or-component-file-path>
```

**Examples:**
```bash
# React component (auto-detect backend)
/qa-input-fields frontend-dashboard/app/components/admin/create-user-modal.tsx

# Explicit backend path
/qa-input-fields frontend/app/pages/auth/signup.tsx --backend backend/src

# Frontend-only analysis (no backend tracing)
/qa-input-fields src/pages/login.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--backend <path>` | No | Path to backend source root. If omitted, auto-detects sibling `backend/` directory |
| `--depth full\|shallow` | No | `full` (default): trace through entire stack. `shallow`: frontend-only analysis |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection               │
│  - Validate file path and extension                         │
│  - Detect frontend framework (React/Vue/Angular/HTML)       │
│  - Detect backend framework (NestJS/Express/Django/etc.)    │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Input Field Discovery                              │
│  - Parse component for all input elements                   │
│  - Extract: name, type, element, label, placeholder         │
│  - Map component library inputs (Shadcn, MUI, Ant, etc.)   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Validation Schema Extraction                       │
│  - Frontend: Zod / Yup / HTML5 attributes                   │
│  - Backend: class-validator / Joi / decorators              │
│  - Database: entity column constraints                       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: CRUD Operation Tracing                             │
│  - Form submit → API service → HTTP endpoint                │
│  - Controller → DTO → Service → Repository → Entity         │
│  - Build full traceability chain                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Validation Gap Analysis                            │
│  5a: Input restriction verification                         │
│  5b: Field-type edge cases & error message verification     │
│  5c: Form behavior verification                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Test Case Generation                               │
│  - Per-field validation tests                               │
│  - Cross-field validation tests                              │
│  - CRUD scenario tests                                       │
│  - Edge case & boundary tests                                │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Generate QA Report                                 │
│  - Field inventory, validation matrix, CRUD trace           │
│  - Input restriction & error message audit                  │
│  - Issues, test cases, recommendations                       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 8: Save & Present Results                             │
│  - Save report to .claude-project/qa/                       │
│  - Display summary with actionable items                    │
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

Read the file path from `$ARGUMENTS`. Parse optional flags (`--backend`, `--depth`).

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-input-fields <page-or-component-file-path> [--backend <path>] [--depth full|shallow]
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

**1.3 Backend Detection** (if `--depth full`):

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
[ ] Backend framework detected (or --depth shallow acknowledged)
```

---

### Step 2: Input Field Discovery

Parse the component/page source code and discover ALL input fields.

**2.1 Framework-Specific Detection Patterns:**

**React:**
- `useForm()` + `register('fieldName')` or `<Controller name="fieldName">` (react-hook-form)
- `useState` + `onChange` handler (controlled components)
- `<Formik>` or `useFormik()` + `<Field name="...">` (Formik)
- Native `<input>`, `<select>`, `<textarea>` elements

**Vue:**
- `v-model="fieldName"` directives in `<template>`
- `useField('fieldName')` / `useForm()` (VeeValidate)
- `useVuelidate()` rules object

**Angular:**
- `FormGroup` / `FormControl` / `FormBuilder` in component `.ts`
- `formControlName="fieldName"` in template `.html`
- `Validators.required`, `Validators.minLength(n)`, etc.

**Plain HTML:**
- All `<input>`, `<select>`, `<textarea>` elements
- `name` attribute, HTML5 validation attributes

**2.2 Component Library Input Mapping:**

| Library | Import Pattern | Input Components |
|---------|---------------|-----------------|
| Shadcn/UI | `~/components/ui/...` | `<Input>`, `<Select>`, `<Textarea>`, `<Checkbox>`, `<RadioGroup>`, `<DatePicker>`, `<Combobox>` |
| Ant Design | `antd` | `<Input>`, `<Select>`, `<DatePicker>`, `<InputNumber>`, `<Upload>`, `<Checkbox>` |
| MUI | `@mui/material` | `<TextField>`, `<Select>`, `<Checkbox>`, `<RadioGroup>`, `<DatePicker>` |
| Vuetify | `vuetify` | `<v-text-field>`, `<v-select>`, `<v-checkbox>`, `<v-date-picker>` |
| Element Plus | `element-plus` | `<el-input>`, `<el-select>`, `<el-checkbox>`, `<el-date-picker>`, `<el-upload>` |

**2.3 Build Field Registry:**

For each discovered field, extract:

| Property | Description |
|----------|-------------|
| `name` | Field/register name (e.g., `username`) |
| `htmlType` | Input type attribute (`text`, `email`, `number`, `tel`, `date`, `file`, etc.) |
| `elementType` | Component type (`Input`, `Select`, `Textarea`, `Checkbox`, `RadioGroup`, `DatePicker`, `FileUpload`, `SplitInput`) |
| `required` | From schema or HTML `required` attribute |
| `label` | Extracted from `<Label>` or `<label>` nearby |
| `placeholder` | Placeholder attribute value |
| `defaultValue` | From `defaultValues` or `value` prop |
| `sourceLocation` | `file:line` reference |

**2.4 Special Pattern Detection:**

**Split Input (OTP/PIN/Verification Code):**
> Detection condition: 3+ consecutive same-type inputs with `maxLength={1}` each AND focus-shift logic (onKeyUp/onChange + ref) present.
> Only applies when ALL conditions are met. Does NOT apply to regular input fields.

**Step 2 Checklist:**
```
[ ] All input elements discovered from component source
[ ] Component library inputs mapped to standard types
[ ] Field registry built with name, type, element, label, placeholder
[ ] Special patterns detected (Split Input, dynamic fields, conditional fields)
```

---

### Step 3: Validation Schema Extraction

For each discovered field, extract validation rules from up to 3 layers.

**3.1 Frontend Validation Layer:**

| Source | Detection Pattern | Extracted Rule |
|--------|-------------------|----------------|
| Zod `.min(N)` | `z.string().min(N, 'msg')` | `minLength: N` |
| Zod `.max(N)` | `z.string().max(N, 'msg')` | `maxLength: N` |
| Zod `.regex()` | `z.string().regex(/pattern/)` | `pattern: /regex/` |
| Zod `.email()` | `z.string().email()` | `format: email` |
| Zod `.optional()` | `z.string().optional()` | `required: false` |
| Zod `.refine()` | `.refine(fn, {path})` | `crossField: true` |
| Yup `.required()` | `yup.string().required('msg')` | `required: true` |
| HTML5 `required` | `<input required>` | `required: true` |
| HTML5 `pattern` | `<input pattern="...">` | `pattern: ...` |
| HTML5 `maxLength` | `<input maxLength={N}>` | `maxLength: N` |

Follow imports to locate schema files (e.g., `import { loginSchema } from '~/validations/auth'`).

**3.2 Backend Validation Layer** (NestJS class-validator example):

| Decorator | Rule |
|-----------|------|
| `@IsNotEmpty()` | required |
| `@IsString()` | type: string |
| `@MinLength(N)` | minLength: N |
| `@MaxLength(N)` | maxLength: N |
| `@IsEmail()` | format: email |
| `@IsOptional()` | required: false |
| `@IsEnum(EnumType)` | enum values |
| `@Matches(/pattern/)` | regex pattern |
| `@IsDateString()` | format: ISO date |
| `@IsNumber()` | type: number |
| `@Min(N)` / `@Max(N)` | value range |

Trace the DTO from the controller's `@Body()` parameter. Follow `extends` chains for inherited DTOs.

**3.3 Database/Entity Layer:**

| Constraint | Source |
|-----------|--------|
| `{ nullable: true/false }` | NULL allowed |
| `{ unique: true }` | Unique constraint |
| `{ length: N }` | Max column length |
| `{ type: 'enum', enum: X }` | Enum constraint |
| `{ type: 'integer' }` | Integer type |
| `{ type: 'date' }` | Date type |

**Step 3 Checklist:**
```
[ ] Frontend validation rules extracted for all fields
[ ] Validation schema file located and parsed (Zod/Yup/HTML5)
[ ] Backend DTO decorators extracted (if --depth full)
[ ] Database entity constraints extracted (if --depth full)
[ ] Cross-field validations identified (refine, custom validators)
```

---

### Step 4: CRUD Operation Tracing

Trace data flow from frontend form submission through the entire stack.

**4.1 Frontend Tracing:**

1. Find the form's `onSubmit` / `handleSubmit` handler
2. Trace the handler to the API call:
   - React Query: `useMutation({ mutationFn: ... })` → follow mutationFn
   - Redux: `dispatch(asyncThunk(...))` → follow thunk → API call
   - Direct: `axios.post(...)` / `fetch(...)`
3. Follow the import to the service file
4. Extract HTTP method and endpoint path:
   - `httpService.post('/users', data)` → `POST /users`
   - `httpService.patch('/users/${id}', data)` → `PATCH /users/:id`

**4.2 Backend Tracing** (if `--depth full`):

1. Match `POST /users` → find `@Controller('users')` + `@Post()` decorator
2. Extract DTO: `@Body() dto: CreateUserDto`
3. Follow service call: `this.usersService.create(dto)`
4. Follow to repository/entity: `this.repository.save(entity)`
5. Identify the entity class and database table

**4.3 Build Traceability Chain:**

```
Component [file:line]
  → onSubmit() handler
    → service.create(data) [serviceFile:line]
      → POST /api/users
        → Controller.create() [controllerFile:line]
          → @Body() CreateUserDto [dtoFile]
            → Service.create() [serviceFile:line]
              → Entity → 'users' table
```

For each CRUD operation:

| Operation | Frontend Check | Backend Check |
|-----------|---------------|---------------|
| **Create** | Submit handler calls POST | Controller has `@Post()` with DTO |
| **Read** | Component loads via GET (useQuery/useEffect) | Controller has `@Get()` |
| **Update** | Edit handler calls PATCH/PUT | Controller has `@Patch()`/`@Put()` |
| **Delete** | Delete handler calls DELETE | Controller has `@Delete()`, check soft vs hard delete |

**Step 4 Checklist:**
```
[ ] Form submit handler identified
[ ] API service call traced with HTTP method + endpoint
[ ] Backend controller matched (if --depth full)
[ ] DTO parameter identified (if --depth full)
[ ] Service → Repository → Entity chain traced (if --depth full)
[ ] CRUD operations mapped (Create/Read/Update/Delete)
```

---

### Step 5: Validation Gap Analysis

Compare validation rules across all layers and verify input behavior.

**5.0 Three-Layer Comparison:**

For each field, compare Frontend ↔ Backend DTO ↔ DB Entity:

| Gap Type | Severity | Description |
|----------|----------|-------------|
| Backend allows what frontend blocks | CRITICAL | Security risk — bypassing frontend is trivial |
| Frontend allows what backend blocks | WARNING | Bad UX — user gets server error |
| DTO/Entity mismatch | WARNING | Data integrity risk |
| Frontend-only field (e.g., confirmPassword) | INFO | By design |
| Backend field not in frontend form | INFO | May be auto-populated |

---

#### 5a: Input Restriction Verification

For each field, verify:

1. **Type constraint matching**: HTML `type="number"` ↔ DTO `@IsNumber()` ↔ DB `integer` column — do they align?
2. **Input mask presence**: For number-only fields, is there `inputMode="numeric"`, `pattern="[0-9]*"`, or `onKeyPress` filtering?
3. **Placeholder accuracy**: Does the placeholder text reflect actual validation rules?
   - Example: placeholder says "Enter 10-digit number" but `maxLength` is 15 → MISMATCH
4. **Select/Enum synchronization**: Frontend option list vs `@IsEnum()` vs DB enum values — are they identical?
5. **Min/Max synchronization**: HTML `min/max` attributes vs `@Min/@Max` decorators vs DB constraints — do they match?

**Step 5a Checklist:**
```
[ ] Type constraints matched across layers for all fields
[ ] Input masks verified for restricted-type fields
[ ] Placeholder text accuracy verified against validation rules
[ ] Select/Enum options synchronized across layers
[ ] Min/Max values synchronized across layers
```

---

#### 5b: Field-Type Edge Cases & Error Message Verification

For each discovered field, apply edge case tests based on its type:

| Field Type | Edge Cases to Test | Error Message Check |
|------------|-------------------|---------------------|
| **User ID / Username** | Spaces, special chars (`@#$%`), non-ASCII, leading/trailing spaces, consecutive spaces | Per-case error message exists? Clear enough? |
| **Email** | Missing `@`, missing domain, spaces, duplicates | Format error vs duplicate error distinguished? |
| **Password** | Below min length, missing uppercase/number/special char requirements | Specific message per unmet condition? |
| **Phone** | Letters input, hyphen handling, country code | Format guide message? |
| **Number** | Negative, zero, decimals, letters, extreme values | Range guide message? |
| **Date** | Future/past restrictions, invalid format, empty | Valid range message? |
| **File Upload** | Extension restriction, file size, empty file | Allowed conditions message? |
| **Split Input (OTP/PIN/Code)** | Auto-advance on digit entry, backspace returns to previous cell + deletes, paste fills all cells, non-digit input blocked | Expiry/resend message? |

> Split Input detection: ONLY applies when 3+ consecutive same-type inputs with `maxLength={1}` each AND focus-shift logic exist. Never applies to regular inputs.

**Error Message Verification Items:**
- **Existence**: Does an error message appear when validation fails?
- **Placement**: Below the field? Toast? Modal? Is it appropriate?
- **Clarity**: "Input error" (BAD) vs "Only letters, numbers, and _ are allowed" (GOOD)
- **Timing**: Real-time feedback during input? Or only on form submit?
- **Clearance**: Does the error message disappear when valid input is provided?

**Step 5b Checklist:**
```
[ ] Edge cases identified for all field types present in the form
[ ] Error message existence verified for each validation rule
[ ] Error message clarity assessed (specific vs generic)
[ ] Error message placement identified (inline/toast/modal)
[ ] Error timing verified (real-time vs submit-time)
[ ] Error clearance behavior verified
```

---

#### 5c: Form Behavior Verification

1. **Double-submit prevention**: Is the submit button disabled during submission? Does double-clicking create duplicate records?
2. **Loading state**: Is a loading indicator shown during form submission? Does the user know the action is in progress?
3. **Tab order**: Does pressing Tab move focus through fields in logical order (top→bottom, left→right)?
4. **Unique field duplicate handling**: For fields with DB `unique: true` constraint:
   - Does the backend return 409 Conflict on duplicates?
   - Does the frontend display a clear duplicate error message to the user?
   - Is there a real-time uniqueness check API? (e.g., `/users/check-username`)

**Step 5c Checklist:**
```
[ ] Double-submit prevention mechanism verified (button disabled/loading)
[ ] Loading state indicator verified during form submission
[ ] Tab order verified for logical field navigation
[ ] Unique constraint duplicate handling verified (409 + user message)
```

---

### Step 6: Test Case Generation

Generate specific, actionable test cases for each discovered field.

**6.1 Per-Field Validation Tests:**

For each field, generate tests based on its extracted rules. Example for a `username` field with rules `required, min(3), max(20), regex(/^[a-zA-Z0-9_]+$/)`:

| Test ID | Test Case | Input | Expected |
|---------|-----------|-------|----------|
| USR-V01 | Empty (required) | `""` | Error: required message |
| USR-V02 | Below min (3) | `"ab"` | Error: min length message |
| USR-V03 | At min boundary | `"abc"` | Pass |
| USR-V04 | Above max (20) | `"a" x 21` | Error: max length message |
| USR-V05 | At max boundary | `"a" x 20` | Pass |
| USR-V06 | Invalid chars (space) | `"john doe"` | Error: pattern message |
| USR-V07 | Invalid chars (special) | `"john@doe"` | Error: pattern message |
| USR-V08 | Valid with underscore | `"john_doe"` | Pass |
| USR-V09 | Valid with numbers | `"user123"` | Pass |

**6.2 Cross-Field Validation Tests:**

For fields with `.refine()` or similar cross-field rules:

| Test ID | Test Case | Input | Expected |
|---------|-----------|-------|----------|
| XF-01 | Matching values | `pass=Test1234, confirm=Test1234` | Pass |
| XF-02 | Mismatching values | `pass=Test1234, confirm=Test5678` | Error on confirm field |

**6.3 CRUD Scenario Tests:**

| Test ID | Operation | Scenario | Expected |
|---------|-----------|----------|----------|
| CRUD-C01 | Create | Submit valid form data | 201 Created, data persisted |
| CRUD-C02 | Create | Submit with duplicate unique field | 409 Conflict with message |
| CRUD-C03 | Create | Submit with invalid data | 400 Bad Request with validation errors |
| CRUD-R01 | Read | Load existing record | All fields populated correctly |
| CRUD-U01 | Update | Modify field and save | 200 OK, changes persisted |
| CRUD-D01 | Delete | Delete existing record | Soft-deleted (if applicable) |

**Step 6 Checklist:**
```
[ ] Validation test cases generated for every field
[ ] Boundary value tests included (min, max, exact boundary)
[ ] Cross-field validation tests generated (if applicable)
[ ] CRUD scenario tests generated for each traced operation
[ ] Edge case tests generated per field type (from Step 5b)
```

---

### Step 7: Generate QA Report

Compile all analysis results into a structured report.

**Report Sections:**

1. **Header** — Component name, file path, framework, date, overall score, status
2. **Field Inventory** — Table of all discovered fields with properties
3. **Validation Coverage Matrix** — 3-layer comparison (Frontend / Backend DTO / DB Entity) per field
4. **Input Restriction Matrix** — Type constraints, masks, placeholder accuracy per field
5. **Error Message Audit** — Per-field error message existence, clarity, placement, timing
6. **CRUD Traceability** — Full trace chain for each CRUD operation
7. **Field-Type Edge Case Checklist** — Per field type from Step 5b
8. **Issues Found** — All issues with severity (CRITICAL / WARNING / INFO)
9. **Test Cases** — Complete test case table
10. **Recommendations** — Prioritized action items

**Step 7 Checklist:**
```
[ ] Field Inventory table complete
[ ] Validation Coverage Matrix complete (3-layer comparison)
[ ] Input Restriction Matrix complete
[ ] Error Message Audit complete
[ ] CRUD Traceability chains complete
[ ] Issues listed with severity levels
[ ] Test cases listed with expected results
[ ] Recommendations prioritized
```

---

### Step 8: Save & Present Results

**8.1 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_QA_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.2 Display Summary:**

```
## QA Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_QA_TestCases_[YYMMDD].md

### Summary
- Fields discovered: [N]
- Validation rules extracted: [N] (Frontend: [N], Backend: [N], DB: [N])
- CRUD operations traced: [list]
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
║                    QA Input Fields Report                         ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                   ║
║  File:          [file-path]                                       ║
║  Framework:     [React + react-hook-form + zod]                   ║
║  Backend:       [NestJS + class-validator + TypeORM]              ║
║  Report Date:   [YYYY-MM-DD HH:MM]                               ║
║  Depth:         [full / shallow]                                  ║
║  Overall Score: [N]/100                                           ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                   ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  FIELD INVENTORY ([N] fields)                                     ║
║                                                                    ║
║  | # | Field    | Type   | Element  | Required | Label    |       ║
║  |---|----------|--------|----------|----------|----------|       ║
║  | 1 | username | text   | Input    | Yes      | Username |       ║
║  | 2 | password | pass   | Input    | Yes      | Password |       ║
║  | 3 | email    | email  | Input    | No       | Email    |       ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  VALIDATION COVERAGE MATRIX                                       ║
║                                                                    ║
║  | Field    | Frontend    | Backend DTO  | DB Entity  | Status   |║
║  |----------|-------------|--------------|------------|----------|║
║  | username | min(3)      | @MinLen(3)   | nullable   | WARNING  |║
║  |          | max(20)     | @MaxLen(20)  |            |          |║
║  |          | regex       | @Matches     | unique     |          |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  INPUT RESTRICTION MATRIX                                         ║
║                                                                    ║
║  | Field    | HTML Type | DTO Type    | DB Type | Mask | Match? |║
║  |----------|-----------|-------------|---------|------|--------|║
║  | age      | number    | @IsNumber() | integer | Yes  | OK     |║
║  | phone    | tel       | @IsString() | varchar | No   | WARN   |║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ERROR MESSAGE AUDIT                                              ║
║                                                                    ║
║  | Field    | Rule      | Message Exists | Clear? | Timing     | ║
║  |----------|-----------|----------------|--------|------------|║
║  | username | required  | Yes            | Yes    | Real-time  | ║
║  | username | minLength | Yes            | Yes    | Real-time  | ║
║  | phone    | format    | No             | N/A    | N/A        | ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  CRUD TRACEABILITY                                                ║
║                                                                    ║
║  [CREATE] ✅ TRACED                                               ║
║    Form → userService.create() → POST /users                     ║
║    → UsersController.create() → CreateUserDto                    ║
║    → UsersService.create() → User entity (users table)           ║
║                                                                    ║
║  [READ] ✅ TRACED                                                 ║
║    useQuery → userService.getAll() → GET /users                  ║
║    → UsersController.findAll()                                   ║
║                                                                    ║
║  [UPDATE] ❌ NOT FOUND                                            ║
║  [DELETE] ❌ NOT FOUND                                            ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  ISSUES FOUND                                                     ║
║                                                                    ║
║  [CRITICAL] V-GAP-01: username nullable in entity but required   ║
║    in frontend and backend DTO.                                   ║
║    Risk: Migration could set NULL for existing rows.             ║
║    Fix: Add NOT NULL constraint or handle NULL in code.          ║
║                                                                    ║
║  [WARNING] V-GAP-02: phone has no format validation on backend.  ║
║    Frontend validates regex but DTO only uses @IsString().       ║
║    Risk: Malformed phone could bypass via direct API call.       ║
║    Fix: Add @Matches() to DTO.                                   ║
║                                                                    ║
║  [WARNING] FORM-01: No double-submit prevention detected.        ║
║    Submit button is not disabled during API call.                ║
║    Fix: Add isPending/isLoading check to disable button.         ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  TEST CASES ([N] total)                                           ║
║    Validation: [N] | Cross-field: [N] | CRUD: [N] | Edge: [N]   ║
║    [See full list in test cases file]                             ║
║                                                                    ║
╠════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║  RECOMMENDATIONS                                                  ║
║  1. [Critical] Add NOT NULL to username column                   ║
║  2. [Critical] Add @Matches phone regex to backend DTO           ║
║  3. [Warning] Add submit button loading/disabled state           ║
║  4. [Warning] Add date format validation to frontend schema      ║
║                                                                    ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Field Discovery Completeness | 10% | All fields found and typed |
| Frontend Validation Coverage | 15% | Each field has frontend validation |
| Backend Validation Coverage | 15% | Each field has backend validation |
| Frontend-Backend Consistency | 20% | Validations match across layers |
| Input Restriction & Placeholder | 10% | Type constraints, masks, placeholder accuracy |
| Error Message Coverage | 15% | Error messages exist, are clear, properly placed |
| CRUD Traceability | 15% | Operations traced end-to-end |

**Severity scoring:**
- CRITICAL issue: -15 points
- WARNING issue: -5 points
- INFO issue: 0 points
- Each fully-traced CRUD path: +5 points
- Each field with consistent 3-layer validation: +3 points

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
| No input fields found | Display-only component | Warning shown; may be a read-only page |
| Framework not detected | Unusual structure | Falls back to generic HTML detection |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| Validation schema not found | Unresolvable import | Fields listed without validation rules; flagged as WARNING |
| Circular imports | Complex import chain | Breaks after 10 levels; notes as "unresolvable" |

---

## Edge Cases Handled

- **Dynamic form fields** — `.map()` patterns rendering inputs from arrays
- **Conditional fields** — Ternary/conditional rendering; notes which condition controls visibility
- **Multi-step wizard forms** — Follows step state; analyzes each step's fields
- **File upload fields** — Detects `<input type="file">`, `<Upload>`, `FormData` usage
- **Rich text editors** — Detects CKEditor, TinyMCE, Quill imports; flags as "manual validation needed"
- **Multiple forms in one component** — Detects multiple `useForm()` or `<form>` elements; analyzes each
- **Validation schema in separate file** — Follows imports to locate schema definition
- **Inherited/base class DTO** — Follows `extends` chain to collect all validators
- **Custom validators** — Flags as "custom validator — manual review needed"
- **i18n error messages** — Detects `t('key')` patterns; notes actual text is in locale files

---

## Related Commands

- `/review-command` - Validate this command's structure and quality
- (Future) `/qa-api-endpoints` - QA test all API endpoints in a controller
- (Future) `/qa-e2e-flows` - Generate E2E test scenarios for complete user flows

---

## Tips

1. **Start with high-traffic pages** - Login, signup, and user creation forms are the highest-value targets
2. **Use `--depth full`** - Full-stack tracing catches the most critical issues (frontend-backend mismatches)
3. **Check CRITICAL issues first** - These represent security risks where backend allows unvalidated input
4. **Run after schema changes** - Whenever Zod schemas, DTOs, or entities change, re-run this command
5. **Combine with CLAUDE.md** - Add project-specific conventions to CLAUDE.md for better detection accuracy
