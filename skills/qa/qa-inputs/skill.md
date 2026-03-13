---
name: qa-inputs
description: Audit all input fields across the full stack (Entity -> DTO -> Zod -> Form UI) for consistency issues
user-invocable: true
argument-hint: "[module] [--check N] [--group name]"
---

# QA Inputs - Input Field Consistency Auditor

## Purpose

Audit all input fields across the full stack (Entity -> DTO -> Zod Schema -> Form UI) for consistency issues. Detects missing asterisks, validation gaps, field name mismatches, length constraint mismatches, enum inconsistencies, i18n omissions, orphaned DB columns, input type mismatches, and security concerns.

## Supported Stacks

This skill auto-detects the project stack and adapts scanning patterns accordingly:

| Layer | Supported Frameworks |
|-------|---------------------|
| Backend ORM | TypeORM, Prisma, Sequelize, Django ORM, SQLAlchemy |
| Backend Validation | class-validator (NestJS), Joi, express-validator, Django serializers, Pydantic |
| Frontend Schema | Zod, Yup, Joi (browser), Valibot |
| Frontend UI | React (JSX/TSX), Vue (SFC), Svelte, Angular templates |

**Auto-detection:** During Stack Detection, scan for `package.json`, `requirements.txt`, `pyproject.toml`, etc. to detect the stack. Adjust grep patterns per framework.

## Usage

```
/qa-inputs                    # Full audit (all modules)
/qa-inputs products           # Single module only
/qa-inputs --check 1          # Run only Check 1
/qa-inputs --check 1,6,10     # Run specific checks
/qa-inputs --group type       # Run only type-related checks (10-12)
/qa-inputs --group security   # Run only security checks (19-20)
/qa-inputs --group semantic   # Run only semantic checks (21-26)
/qa-inputs --group ux         # Run only UX quality checks (27-31)
/qa-inputs --group form       # Run only form behavior checks (32-37)
```

### Group Mapping

| Group | Checks | Description |
|-------|--------|-------------|
| required | 1, 3, 5 | Required/Optional mismatches |
| type | 10, 11, 12 | Type mismatches |
| constraint | 6, 7, 8, 16, 17 | Constraint mismatches |
| integrity | 4, 13, 14, 15 | Cross-layer integrity |
| quality | 2, 9, 18 | Code quality |
| security | 19, 20 | Security concerns |
| semantic | 21-26, 38-44 | Semantic validation |
| ux | 27-31 | UX quality |
| form | 32-37, 45-48 | Form behavior |

### Quick Start

```
> /qa-inputs products

# QA Inputs Audit Report
**Date**: 2026-03-10  |  **Scope**: products  |  **Stack**: auto-detected

## Summary
| # | Check | Issues |
| 1 | Required without asterisk | 2 |
| 6 | maxLength mismatch | 1 |
| 10 | Numeric type mismatch | 3 |
...
| | **Total** | **11** |
```

### Severity Priority

Findings are ordered by severity. The priority order is:

**CRITICAL > HIGH > MEDIUM > LOW > INFO**

## 48 Checks

> **Note on check numbering:** Checks are numbered by category rather than sequentially (e.g., Required checks are 1/3/5, Type checks are 10/11/12). This is intentional so that `--check N` references remain stable as new checks are added within categories. The summary table lists all 48 checks in order.

### Required / Optional (Checks 1, 3, 5)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Required without asterisk | **High** | DTO `@IsNotEmpty()` but no `*` in frontend |
| 3 | Optional with asterisk | **Medium** | DTO `@IsOptional()` but frontend shows `*` |
| 5 | DB column no input route | **High** | Non-nullable entity column missing from Create DTO |

### Type Matching (Checks 10, 11, 12)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 10 | Numeric type mismatch | **High** | Numeric DB column / DTO `@IsNumber()` but frontend allows text input |
| 11 | Date type mismatch | **High** | Date/timestamp DB column but frontend uses text input instead of date picker |
| 12 | Boolean type mismatch | **High** | Boolean DB column but frontend uses text input instead of checkbox/toggle |

### Constraint Matching (Checks 6, 7, 8, 16, 17)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 6 | maxLength mismatch | **High** | DB/DTO has max length but frontend doesn't enforce |
| 7 | minLength mismatch | **Medium** | DTO `@MinLength(N)` vs Zod `.min(N)` inconsistent |
| 8 | Enum value mismatch | **Medium** | DTO `@IsEnum()` values != frontend dropdown options |
| 16 | Decimal precision mismatch | **Medium** | DB `decimal(10,2)` but frontend allows more decimal places |
| 17 | Numeric range mismatch | **Medium** | DTO `@Min(0)`/`@IsPositive()` but frontend input has no `min` attribute |

### Cross-Layer Integrity (Checks 4, 13, 14, 15)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 4 | Field name mismatch | **Info** | Zod field != DTO field != Entity property |
| 13 | Nullable ↔ Required conflict | **High** | Entity `nullable: false` but DTO `@IsOptional()`, or the reverse |
| 14 | Orphan frontend field | **Medium** | Zod schema has field not in any DTO → silently ignored by server |
| 15 | Orphan DTO field | **Medium** | Create DTO has field not sent by any frontend form → always undefined |

### Quality (Checks 2, 9, 18)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 2 | Regex without error message | **Low** | `@Matches()` or `.regex()` has no user-friendly error message |
| 9 | i18n hardcoded | **Low** | Labels/errors/placeholders without `t()` wrapper |
| 18 | Unique constraint without hint | **Low** | Entity `@Unique()` but no frontend duplicate check → server 500 |

### Security (Checks 19, 20)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 19 | XSS-prone input | **Critical** | User text rendered via `dangerouslySetInnerHTML` / `v-html` without sanitization |
| 20 | File upload without constraint | **Medium** | File input without `accept` type filter or size limit |

### Semantic Validation (Checks 21-26)

Infer expected validation rules from field name patterns. Flag when common-sense rules are missing.

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 21 | Name field too short | **Medium** | `*name*` field allows minLength < 2 (1-char name is unrealistic) |
| 22 | Email field without validation | **High** | `*email*` field missing `@IsEmail()` / `z.string().email()` / `type="email"` |
| 23 | Password field too weak | **High** | `*password*` field has minLength < 6 or missing `type="password"` |
| 24 | URL field without validation | **Medium** | `*url*`/`*Url*`/`*link*` field missing `@IsUrl()` / `.url()` validation |
| 25 | Count/quantity allows negative | **Medium** | `*count*`/`*quantity*`/`*sets*`/`*reps*` field allows negative or decimal |
| 26 | Price/amount allows negative | **Medium** | `*price*`/`*amount*`/`*cost*` field allows negative values |
| 38 | Numeric-string without format constraint | **High** | `varchar` field named `*number*`/`*code*`/`*no*`/`*num*`/`*id*` (identifier) stored as string but has no `pattern`/`@Matches` to restrict input format |
| 39 | Date field missing temporal constraint | **Medium** | `*birthday*`/`*dob*`/`*startDate*` missing past-only or future-only validation |
| 40 | Phone field without format validation | **High** | `*phone*`/`*tel*`/`*mobile*` missing phone number format pattern or `@IsPhoneNumber()` |
| 41 | Zip/postal code without format | **Medium** | `*zip*`/`*postal*` missing format pattern for postal code validation |
| 42 | Percentage/rate out of range | **Medium** | `*rate*`/`*percentage*`/`*ratio*`/`*percent*` missing 0-100 range constraint |
| 43 | Duration/time unrealistic values | **Medium** | `*minutes*`/`*hours*`/`*duration*`/`*seconds*` missing positive constraint or reasonable max limit |
| 44 | Score/rating without range | **High** | `*score*`/`*rating*`/`*intensity*`/`*level*`/`*grade*` missing defined min/max range |

### UX Quality (Checks 27-31)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 27 | Long text in single-line input | **Medium** | `*description*`/`*content*`/`*bio*`/`*memo*`/`*note*` uses `<input>` instead of `<textarea>` |
| 28 | Whitespace-only accepted | **Medium** | String field missing `.trim()` in schema → `"   "` passes validation |
| 29 | Empty string vs null | **Low** | Optional string field sends `""` but DB expects `null` → empty strings saved instead of null |
| 30 | Missing autocomplete attribute | **Low** | `email`/`password`/`name`/`tel` field without HTML `autocomplete` attribute |
| 31 | Format hint missing | **Low** | Field requiring specific format (phone, zip, date) has no `placeholder` example |

### Form Behavior (Checks 32-37)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 32 | Error message not displayed | **High** | Field has schema/DTO validation but no error display component (`<FormMessage>`, `{errors.field}`) in UI |
| 33 | Select default value missing | **High** | Required Select/Dropdown has no default value and placeholder is submittable → meaningless value sent |
| 34 | Double submission possible | **Medium** | Submit button has no loading/disabled state → duplicate data on double-click |
| 35 | Conditional required mismatch | **Medium** | DTO `@ValidateIf()` condition not mirrored in frontend (always required or always optional) |
| 36 | Array field min/max missing | **Medium** | DTO `@IsArray()` + `@ArrayMinSize(1)` but frontend allows empty array submission |
| 37 | Default value not shown | **Low** | Entity column has `default` value but frontend form doesn't pre-fill or show it |
| 45 | Validation mode onSubmit only | **High** | `useForm` uses default `mode: 'onSubmit'` — errors only appear after submit, no real-time feedback |
| 46 | String field without maxLength | **High** | String field has no `@MaxLength` in DTO AND no `.max()` in Zod AND no `maxLength` on HTML input — unbounded input |
| 47 | Schema maxLength without HTML maxLength | **Medium** | Schema has `.max(N)` or DTO has `@MaxLength(N)` but HTML `<input>` missing `maxLength` attribute — user can type beyond limit |
| 48 | Restricted-format field without real-time input filtering | **High** | Field has `pattern`/`inputMode`/`.regex()` constraint but no `onChange`/`onInput`/`onKeyDown` handler to block invalid characters in real-time — user can type disallowed chars |

---

## Execution Algorithm

### Execution Flow

```
Stack Detection → Inventory → Mapping → Checks 1-48 → Report
```

### Step 0: Context & Requirements Gathering (MANDATORY)

Before auditing, you MUST understand which features/modules are in scope.

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
Glob: .claude-project/docs/PROJECT_KNOWLEDGE.md
```

Extract:
- Which modules/forms are actively used in production
- Expected input fields per feature
- Which frontend apps exist and their target users

**0b. Ask clarifying questions BEFORE proceeding:**

If you find form fields or types that don't match any requirement, ask:

> "I found [field/type X] that doesn't match the project requirements. Is this actively used or legacy code?"

Wait for the user's response. Do NOT flag stale types for unused features as production bugs.

**0c. Build scope list:**
```
IN SCOPE: [module1 forms, module2 forms, ...]
OUT OF SCOPE: [module3 (unused), ...]
```

---

### Stack Detection

Detect the project's tech stack before scanning:

1. **Backend framework:** Check for `@nestjs/core` in package.json (NestJS), `django` in requirements.txt (Django), `express` (Express), `fastify` (Fastify)
2. **ORM:** Check for `typeorm` (TypeORM), `@prisma/client` (Prisma), `sequelize` (Sequelize), `django.db` (Django ORM), `@supabase/supabase-js` (Supabase)
3. **Frontend framework:** Check for `react` (React), `vue` (Vue), `svelte` (Svelte), `@angular/core` (Angular)
4. **Validation library:** Check for `class-validator` (NestJS), `joi`, `zod`, `yup`, `valibot`
5. **Frontend validation:** Check for `zod`, `yup`, `joi`, `valibot` in frontend package.json

Adapt all grep patterns in subsequent steps based on detected stack.

**Path detection:**
- Scan for backend directory: `backend/`, `server/`, `api/`, `src/` (if no separate dir)
- Scan for frontend directories: `frontend/`, `client/`, `web/`, `app/`, `admin/`, `dashboard/` and any other directories with frontend framework markers
- Scan for entity/model files based on ORM
- Scan for DTO/serializer/schema files based on framework

### Inventory

Build three inventories by scanning the codebase.

#### A. Entity Inventory

Scan all entity/model files based on detected ORM:

**TypeORM:**
```
Glob: {backend-dir}/**/entities/*.entity.ts, {backend-dir}/**/*.entity.ts
```

**Prisma:**
```
Read: prisma/schema.prisma → extract models
```

**Django:**
```
Glob: **/models.py, **/models/*.py
```

**Supabase:**
```
Glob: supabase/migrations/*.sql (extract CREATE TABLE / ALTER TABLE for column definitions)
Also: **/database.types.ts, **/supabase.types.ts (generated via `supabase gen types typescript`)
```
When using generated types, parse the `Database['public']['Tables']` interface to extract table names, column types, and nullable status. When SQL migrations exist, parse `CREATE TABLE` statements for constraints (NOT NULL, DEFAULT, CHECK, VARCHAR(N)).

For each entity/model, extract:
- **Entity class name** and module path
- **All column/field definitions**: field name, type, nullable status, default value, length constraint
- **All relation fields**: foreign key column names
- **Auto-generated fields to SKIP**: primary keys, auto timestamps
- **Base class inherited fields to SKIP**: `id`, `createdAt`, `updatedAt`, `deletedAt` (or framework equivalents)

Record as structured data:
```
EntityName: {
  module: "products",
  file: "product.entity.ts",
  columns: [
    { name: "title", type: "string", nullable: false, length: 255, hasDefault: false, isNumeric: false, isDate: false, isBoolean: false, isUnique: false, decimalPrecision: null },
    { name: "reps", type: "number", nullable: false, length: null, hasDefault: false, isNumeric: true, isDate: false, isBoolean: false, isUnique: false, decimalPrecision: null },
    ...
  ]
}
```

#### B. DTO Inventory

Scan all DTO/serializer/schema files based on detected framework:

**NestJS (class-validator):**
```
Glob: {backend-dir}/**/dtos/*.dto.ts, {backend-dir}/**/dto/*.dto.ts
```

**Django REST:**
```
Glob: **/serializers.py, **/serializers/*.py
```

**Express (Joi/Yup):**
```
Glob: **/validators/*.ts, **/schemas/*.ts
```

**Supabase (no server-side DTO):**
Supabase projects typically have NO dedicated DTO layer -- validation is handled entirely on the frontend (Zod/Yup) + database constraints (NOT NULL, CHECK, FK). In this case:
- Skip DTO inventory -- use Entity (database schema) as the source of truth for required/type/length
- Cross-layer mapping becomes: DB column ↔ Frontend schema field (2-layer instead of 3-layer)
- RLS policies may restrict which fields are writable -- check `supabase/migrations/*.sql` for `USING` / `WITH CHECK` clauses

For each DTO, extract:
- **DTO class name** (distinguish Create vs Update vs other)
- **Each field** with its decorators/validators:
  - Required: `@IsNotEmpty()` / `.required()` / `required=True`
  - Optional: `@IsOptional()` / `.optional()` / `required=False`
  - Regex: `@Matches(regex, { message })` / `.regex()`
  - Max length: `@MaxLength(N)` / `.max(N)` / `max_length=N`
  - Min length: `@MinLength(N)` / `.min(N)` / `min_length=N`
  - Enum: `@IsEnum(EnumType)` / `.oneOf()` / `choices=`
  - Numeric: `@IsNumber()` / `@IsInt()` / `@IsPositive()` / `@Min()` / `@Max()`
  - Numeric coercion: `@Type(() => Number)`
  - Date: `@IsDate()` / `@IsDateString()` / `@IsISO8601()`
  - Boolean: `@IsBoolean()`
  - Conditional: `@ValidateIf(condition)`
- **PartialType / OmitType usage** in Update DTOs

Record as structured data:
```
CreateProductDto: {
  module: "products",
  file: "create-product.dto.ts",
  type: "create",
  fields: [
    { name: "title", required: true, maxLength: 255, minLength: 1, regex: null, enum: null, isNumeric: false, isDate: false, isBoolean: false, numericMin: null, numericMax: null, decimalPrecision: null },
    ...
  ]
}
```

#### C. Frontend Form Inventory

Find all forms in frontend apps:

**React (Zod):**
```
Grep for: useForm|zodResolver
In: auto-detected frontend directories — **/*.tsx
```

**React (Yup):**
```
Grep for: useForm|yupResolver
```

**Vue:**
```
Grep for: useForm|useField|vee-validate
In: **/*.vue
```

For each form file, extract:

**Schema layer (Zod/Yup/Valibot):**
- Field names and types (`z.string()`, `z.number()`, `z.boolean()`, `z.date()`, `z.coerce.number()`)
- Required/optional status
- Regex, max, min constraints
- Enum values
- Transform/coerce chains

**UI Elements:**
- Labels with asterisk detection
- Input attributes: `type`, `maxLength`, `min`, `max`, `step`, `accept`, `inputMode`, `pattern`
- Select/dropdown option values
- Checkbox/toggle for booleans
- Date picker components
- File input elements
- i18n usage

**API mapping:**
- Form submission handler → API endpoint → maps to DTO

Record as structured data:
```
FormFile: {
  app: "frontend-dashboard",
  file: "edit-product-modal.tsx",
  apiEndpoint: "POST /products",
  fields: [
    { name: "title", zodType: "string", zodRequired: true, hasAsterisk: true, maxLength: null, minLength: null, hasRegex: false, regexHasMsg: null, enumValues: null, inputType: "text", inputMin: null, inputMax: null, inputStep: null, inputAccept: null, labelI18n: true, errorI18n: true, placeholderI18n: true },
    ...
  ]
}
```

### Cross-Layer Mapping

For each module, build a mapping table connecting:
- Entity columns ↔ Create DTO fields (by matching field names)
- Create DTO fields ↔ Frontend schema fields (by matching field names + API endpoint)
- Update DTO fields (usually PartialType of Create, so inherits mapping)

**Important rules:**
- Match by language property name (camelCase), NOT by SQL column name (snake_case)
- ORM column name options are the DB column name, the property name is what matters
- Foreign key fields like `userId` may not have a form input (set programmatically) - note these

Output a mapping table per module (internal, not in final report):
```
Module: products
| Entity Column | Create DTO Field | Schema Field | Form File | Notes |
```

### Check 1 - Required Field Without Asterisk (*)

**Logic:**
For each mapped field where:
1. DTO field is required (`@IsNotEmpty()` / no `@IsOptional()` / no `@ValidateIf()`)
2. AND corresponding frontend form exists
3. AND the form label does NOT have `*` AND does NOT use `required` prop

**Exclude:**
- Fields without a frontend form (backend-only fields set programmatically)
- Fields with conditional validation
- Hidden fields (`type="hidden"`)

**Report as:** `CHECK-1: Required field without asterisk`

### Check 2 - Regex Without Error Message

**Logic - Backend:**
Find any regex validation without a user-friendly error message.
- NestJS: `@Matches(/.../)` without `{ message: '...' }`
- Joi: `.pattern(/.../)` without `.messages({})`

**Logic - Frontend:**
Find any `.regex(/.../)` in schema without a second string argument.

**Report as:** `CHECK-2: Regex validation without error message`

### Check 3 - Optional Field With Asterisk

**Logic:**
For each mapped field where:
1. DTO field is optional (`@IsOptional()`)
2. AND corresponding frontend form label HAS `*` or uses `required` prop

**Report as:** `CHECK-3: Optional field incorrectly marked as required`

### Check 4 - Field Name Mismatch

**Logic:**
1. For each frontend schema field name, check if there is a DTO field with the SAME name (case-sensitive)
2. For each DTO field name, check if there is an Entity property with the SAME name (case-sensitive)

**Important:** Compare language property names, NOT SQL column names.

**Exclude:**
- Frontend-only fields (e.g., `confirmPassword`, `rememberMe`)
- DTO fields that have no entity column (computed/virtual fields)

**Report as:** `CHECK-4: Field name mismatch across layers`

### Check 5 - DB Column With No Input Route

**Logic:**
For each entity column that is:
1. NOT nullable
2. NOT auto-generated (primary key, auto timestamps)
3. NOT inherited from base class
4. Has NO default value
5. NOT a relation object (skip relation properties, keep FK columns)

**Step A — Missing from DTO entirely:**
Check if ANY Create DTO includes a field mapping to this column.
If not → the API will fail when this field is not provided.

**Step B — Present in DTO but Optional (soft missing):**
If the field EXISTS in the Create DTO but is marked `@IsOptional()` (or has no `@IsNotEmpty()`):
- The API can accept requests WITHOUT this field
- The DB will reject the insert (NOT NULL violation) UNLESS the service layer sets it programmatically or the frontend always sends a non-null value (e.g., empty string `""`)
- Report as a **separate sub-finding** with severity HIGH if no fallback exists, or MEDIUM if the frontend sends a fallback value (like `""`)

**Checking for fallback values:**
- If the frontend form sends a default/fallback (e.g., `data.videoUrl || ""` or `defaultValues: { field: "" }`), note it as "mitigated by frontend fallback" but still flag as a design inconsistency
- If no fallback exists → HIGH severity (API will 500 on missing field)

**Exclude from Step B:**
- Fields typically set programmatically: `*Id` (userId, createdById), `*By` (createdBy), `status`, `role`
- Fields set by interceptors/guards/service layer logic

**Report as:** `CHECK-5: Non-nullable DB column with no input route`
- Step A findings: `CHECK-5A: Non-nullable column completely missing from Create DTO`
- Step B findings: `CHECK-5B: Non-nullable column is @IsOptional in Create DTO (soft missing)`

### Check 6 - maxLength Mismatch

**Logic:**
For each mapped field, compare max length constraints across layers:
1. Entity: column length constraint
2. DTO: max length validator
3. Frontend schema: `.max(N)`
4. Frontend HTML: `maxLength={N}` on input

Report if values differ or are missing in any layer.

**Report as:** `CHECK-6: maxLength constraint mismatch`

### Check 7 - minLength Mismatch

**Logic:**
Compare min length constraints between DTO and frontend schema.
Only flag when N > 1 for strings (`.min(1)` is typically a "required" check, not a length check).

**Report as:** `CHECK-7: minLength constraint mismatch`

### Check 8 - Enum Value Mismatch

**Logic:**
1. Find all enums used in DTOs
2. Locate the enum definition and extract all values
3. Find the corresponding frontend form field (select/dropdown)
4. Extract all option values from the frontend

Report if backend enum has values not in frontend options, or vice versa.

**Report as:** `CHECK-8: Enum value mismatch between backend and frontend`

### Check 9 - i18n Hardcoded Strings

**Logic:**
Scan frontend form files for hardcoded strings in labels, error messages, and placeholders that should use i18n wrappers (`t()`, `$t()`, `i18n.t()`, etc.).

**Exceptions to IGNORE:** `*`, pure numbers, single characters, variable references, files that don't use i18n at all.

**Report as:** `CHECK-9: Hardcoded string without i18n wrapper`

### Check 10 - Numeric Type Mismatch

**Logic:**
For each mapped field, detect mismatches between numeric backend type and frontend input.

**Step 1 - Identify numeric fields from backend:**
A field is "numeric" if ANY of these are true:
1. Entity: column type is `int`, `integer`, `float`, `decimal`, `numeric`, `smallint`, `bigint`, `double precision`, `real`
2. Entity: TypeScript/Python type annotation is `number` / `int` / `float`
3. DTO: has numeric validators (`@IsNumber()`, `@IsInt()`, `@IsPositive()`, `@Min(N)`, `@Max(N)`)
4. DTO: has numeric coercion (`@Type(() => Number)`)

**Step 2 - Check frontend for numeric enforcement:**

Frontend schema should use:
- `z.number()` / `z.coerce.number()` / `yup.number()` → OK
- `z.string()` with `.transform(Number)` or `.pipe(z.number())` → OK
- `z.string()` without any numeric transform → MISMATCH

HTML input should use:
- `type="number"` → OK
- `inputMode="numeric"` or `inputMode="decimal"` → OK
- `pattern="[0-9]*"` or similar numeric-only pattern → OK
- No type attribute or `type="text"` → MISMATCH

**Exclude:** fields not rendered in frontend, select/dropdown fields, phone numbers, intentionally alphanumeric IDs.

**Report as:** `CHECK-10: Numeric type mismatch (numeric field allows text input)`

### Check 11 - Date Type Mismatch

**Logic:**

**Step 1 - Identify date fields from backend:**
A field is a "date" if ANY of these are true:
1. Entity: column type is `timestamp`, `timestamptz`, `date`, `datetime`, `time`
2. Entity: TypeScript type is `Date`
3. DTO: has `@IsDate()`, `@IsDateString()`, `@IsISO8601()`
4. Prisma: field type is `DateTime`

**Step 2 - Check frontend for date input:**

Frontend schema should use:
- `z.date()` / `z.coerce.date()` / `yup.date()` → OK
- `z.string()` with date format validation (`.regex()` for date pattern) → OK
- `z.string()` without date handling → MISMATCH

HTML input should use:
- `type="date"`, `type="datetime-local"`, `type="time"` → OK
- Date picker component (`DatePicker`, `Calendar`, `react-datepicker`, etc.) → OK
- `type="text"` without date picker → MISMATCH

**Exclude:** fields not rendered in frontend, fields set programmatically.

**Report as:** `CHECK-11: Date type mismatch (date field uses text input)`

### Check 12 - Boolean Type Mismatch

**Logic:**

**Step 1 - Identify boolean fields from backend:**
A field is "boolean" if ANY of these are true:
1. Entity: column type is `boolean`, `bool`
2. Entity: TypeScript/Python type is `boolean` / `bool`
3. DTO: has `@IsBoolean()`
4. Prisma: field type is `Boolean`

**Step 2 - Check frontend for boolean input:**

Frontend should use:
- `z.boolean()` / `yup.boolean()` → OK
- Checkbox (`type="checkbox"`), Switch, Toggle component → OK
- `z.string()` or `type="text"` → MISMATCH
- Select with only true/false options → OK (acceptable pattern)

**Exclude:** fields not rendered in frontend, fields set programmatically.

**Report as:** `CHECK-12: Boolean type mismatch (boolean field uses text input)`

### Check 13 - Nullable ↔ Required Conflict

**Logic:**
For each mapped field, compare entity nullable status with DTO required status:

**Conflict A - Strict (entity NOT NULL but DTO optional):**
1. Entity column `nullable: false` (or absent, which defaults to NOT NULL)
2. DTO field has `@IsOptional()`
3. No default value on entity column
→ This means API can accept `undefined`, but DB will reject the insert.

**Conflict B - Data Integrity Risk (entity NULL but DTO required):**
1. Entity column `nullable: true`
2. DTO field has `@IsNotEmpty()` without `@IsOptional()`
→ **Report as HIGH.** DTO validation only guards the API layer. Data inserted via migrations, seeders, or direct DB access can have NULL values, breaking frontend display and business logic. If the field is truly required, the Entity column should also be NOT NULL to enforce at DB level.

**Exclude:**
- Fields with default values
- Fields set programmatically (e.g., `createdBy`, `userId`)
- Update DTOs (PartialType makes everything optional)

**Report as:** `CHECK-13: Nullable ↔ Required conflict`

### Check 14 - Orphan Frontend Field

**Logic:**
For each frontend schema field, check if a matching field exists in the target DTO (the DTO used by the API endpoint the form submits to).

If the frontend schema has a field that does NOT exist in the DTO:
- The server will silently ignore it (class-validator `whitelist: true`)
- OR it will pass through if whitelist is off (potential security issue)

**Exclude:**
- Known UI-only fields: `confirmPassword`, `rememberMe`, `agree`, `terms`
- Fields that are transformed before submission (e.g., combining `firstName` + `lastName` → `fullName`)

**Report as:** `CHECK-14: Orphan frontend field (not in DTO, silently ignored)`

### Check 15 - Orphan DTO Field

**Logic:**
For each Create DTO field, check if ANY frontend form that submits to the corresponding endpoint includes this field.

If the DTO has a field that is NOT in any frontend form:
- It will always be `undefined` unless set programmatically
- If required (`@IsNotEmpty()`), the API will always reject → dead endpoint

**Exclude:**
- Fields typically set programmatically: `*Id` (userId, createdById), `*By` (createdBy), `status`, `role`
- Fields set by interceptors/guards
- Update DTOs (partial, so missing fields are OK)

**Report as:** `CHECK-15: Orphan DTO field (no frontend sends this field)`

### Check 16 - Decimal Precision Mismatch

**Logic:**
For each mapped numeric field:
1. Entity: check for `decimal(P,S)`, `numeric(P,S)`, `@Column({ type: 'decimal', precision: P, scale: S })`
2. Frontend: check for `step` attribute on `<input type="number">`

Report if:
- Entity has `scale: 2` (2 decimal places) but frontend input has no `step` or `step` allows more decimals
- Recommended: `step="0.01"` for `scale: 2`, `step="0.001"` for `scale: 3`, etc.
- Frontend schema has no `.multipleOf()` or equivalent precision check

**Report as:** `CHECK-16: Decimal precision mismatch`

### Check 17 - Numeric Range Mismatch

**Logic:**
For each mapped numeric field, compare range constraints:

**Backend ranges:**
- DTO: `@Min(N)`, `@Max(N)`, `@IsPositive()` (implies min > 0)
- Entity: check constraints if any

**Frontend ranges:**
- Schema: `.min(N)`, `.max(N)`, `.positive()`, `.nonnegative()`
- HTML input: `min={N}`, `max={N}` attributes

Report if:
- DTO has `@Min(0)` or `@IsPositive()` but frontend input has no `min` attribute → user can type negative numbers
- DTO has `@Max(N)` but frontend has no `max` attribute
- Values differ between DTO and frontend

**Report as:** `CHECK-17: Numeric range mismatch`

### Check 18 - Unique Constraint Without Hint

**Logic:**
1. Find entity fields with unique constraints: `@Column({ unique: true })`, `@Unique()`, `@Index({ unique: true })`, Prisma `@unique`
2. Check if the corresponding frontend form provides any feedback before submission:
   - Async validation / debounced uniqueness check on blur → OK
   - No pre-submit check → the user sees a raw 500/409 error

**Report as:** `CHECK-18: Unique constraint without frontend duplicate check`

### Check 19 - XSS-Prone Input

**Logic:**
Trace user-submitted text fields from form → API → DB → rendering.

**Step 1 - Identify user text fields:**
All string fields from Create DTOs that come from frontend forms (not system-set).

**Step 2 - Check rendering:**
Search for dangerous rendering patterns:
- React: `dangerouslySetInnerHTML` with user data
- Vue: `v-html` with user data
- Angular: `[innerHTML]` with user data
- Raw string interpolation in HTML templates

**Step 3 - Check sanitization:**
If dangerous rendering is found, check for sanitization:
- `DOMPurify.sanitize()` → OK
- `sanitize-html` → OK
- Backend sanitization in DTO pipe/interceptor → OK
- No sanitization → CRITICAL

**Exclude:**
- Admin-only views (lower risk, but still note)
- Fields rendered only as plain text (React JSX `{variable}` is safe by default)

**Report as:** `CHECK-19: XSS-prone input (user text rendered as HTML without sanitization)`

### Check 20 - File Upload Without Constraint

**Logic:**
Find all file upload inputs in frontend forms:
- `<input type="file">`
- File upload components (Dropzone, Upload, FileInput, etc.)

Check for constraints:
1. **Accept filter**: `accept="image/*"`, `accept=".pdf,.doc"`, etc. → OK / Missing
2. **Size limit**: `maxSize`, file size validation in handler, or backend `@MaxFileSize()` → OK / Missing
3. **Backend validation**: Multer limits, file type validation pipe → OK / Missing

Report if:
- No `accept` attribute AND no backend MIME type validation → user can upload any file type
- No size limit on frontend AND no backend size limit → potential DoS

**Report as:** `CHECK-20: File upload without type/size constraint`

### Check 21 - Name Field Too Short

**Logic:**
For each field whose name matches `*name*`, `*Name*` (e.g., `name`, `firstName`, `lastName`, `displayName`, `username`):

**Step 1 - Check backend:**
- DTO: does it have `@MinLength(N)` where N >= 2? If N < 2 or missing → flag
- Entity: does it have `length` constraint? If length is unreasonably small (< 2) → flag

**Step 2 - Check frontend:**
- Schema: does it have `.min(N)` where N >= 2? If `.min(1)` only → flag (1-char name)
- Note: `.min(1, "required")` is a required check, NOT a length check. Still flag if no separate `.min(2+)` exists

**Semantic rules by field pattern:**
| Pattern | Expected minLength | Expected maxLength |
|---------|-------------------|-------------------|
| `firstName`, `lastName`, `name` | >= 2 | <= 50-100 |
| `username` | >= 3 | <= 30-50 |
| `displayName`, `nickname` | >= 2 | <= 50 |

**Report as:** `CHECK-21: Name field allows unrealistic length`

### Check 22 - Email Field Without Validation

**Logic:**
For each field whose name matches `*email*`, `*Email*`:

**Check all layers:**
1. DTO: must have `@IsEmail()` → missing = flag
2. Frontend schema: must have `.email()` → missing = flag
3. Frontend input: should have `type="email"` → missing = flag (browser-level validation + mobile keyboard)

If ALL THREE are missing → HIGH severity (no email validation at all)
If only input type missing → MEDIUM (validated but bad UX)

**Report as:** `CHECK-22: Email field without proper validation`

### Check 23 - Password Field Too Weak

**Logic:**
For each field whose name matches `*password*`, `*pwd*`:

**Check backend:**
- DTO: `@MinLength(N)` where N >= 6 (ideally >= 8) → N < 6 or missing = flag

**Check frontend:**
- Schema: `.min(N)` where N >= 6 → N < 6 or missing = flag
- Input: must have `type="password"` → `type="text"` = HIGH (password visible)

**Additional checks:**
- Registration/signup forms: should have `confirmPassword` field → missing = flag as INFO
- No `autocomplete="new-password"` on registration → flag as LOW
- No `autocomplete="current-password"` on login → flag as LOW

**Report as:** `CHECK-23: Password field with weak constraints`

### Check 24 - URL Field Without Validation

**Logic:**
For each field whose name matches `*url*`, `*Url*`, `*URL*`, `*link*`, `*Link*`, `*website*`:

**Check all layers:**
1. DTO: should have `@IsUrl()` or `@Matches()` with URL regex → missing = flag
2. Frontend schema: should have `.url()` or `.regex()` with URL pattern → missing = flag
3. Frontend input: should have `type="url"` → missing = LOW (nice-to-have)

**Exclude:**
- Fields that accept relative paths (e.g., `imagePath`, `filePath`)
- Fields with enum values that happen to contain "url" in name

**Report as:** `CHECK-24: URL field without URL validation`

### Check 25 - Count/Quantity Allows Negative or Decimal

**Logic:**
For each field whose name matches `*count*`, `*Count*`, `*quantity*`, `*qty*`, `*sets*`, `*reps*`, `*repetition*`, `*round*`:

**Expected constraints:**
- Must be integer (not float/decimal)
- Must be non-negative (>= 0) or positive (>= 1)

**Check backend:**
- DTO: should have `@IsInt()` AND (`@Min(0)` or `@IsPositive()`) → missing = flag
- Entity: column type should be `int`/`integer` (not `float`/`decimal`)

**Check frontend:**
- Schema: should have `z.number().int()` or `z.coerce.number().int()` → missing `.int()` = flag
- Schema: should have `.min(0)` or `.positive()` → missing = flag
- Input: should have `type="number"` with `step="1"` and `min="0"` → missing = flag

**Report as:** `CHECK-25: Count/quantity field allows negative or decimal values`

### Check 26 - Price/Amount Allows Negative

**Logic:**
For each field whose name matches `*price*`, `*Price*`, `*amount*`, `*Amount*`, `*cost*`, `*Cost*`, `*fee*`, `*Fee*`, `*charge*`:

**Expected constraints:**
- Must be numeric
- Must be non-negative (>= 0)
- Should have decimal precision (typically 2 decimal places)

**Check backend:**
- DTO: should have `@IsNumber()` AND `@Min(0)` → missing = flag
- Entity: column type should be `decimal` with precision defined

**Check frontend:**
- Schema: should have `z.number().min(0)` or `.nonnegative()` → missing = flag
- Input: should have `type="number"` with `min="0"` and `step="0.01"` → missing = flag

**Report as:** `CHECK-26: Price/amount field allows negative values`

### Check 38 - Numeric-String Without Format Constraint

**Logic:**
Detect `varchar`/`string` fields that semantically represent codes, identifiers, or numbers but have no format restriction — allowing arbitrary text where only digits (or a specific pattern) are expected.

**Step 1 - Identify candidate fields:**
A field is a "numeric-string identifier" if ALL of these are true:
1. Entity: column type is `varchar`/`text`/`string` (NOT `int`/`number`)
2. Field name matches ANY of these patterns:
   - `*number*`, `*Number*`, `*num*`, `*Num*` (e.g., `number`, `orderNumber`, `phoneNum`)
   - `*code*`, `*Code*` (e.g., `productCode`, `zipCode`, `verificationCode`)
   - `*no*` as suffix (e.g., `orderNo`, `invoiceNo`) — but NOT `*note*`, `*notification*`, `*node*`
   - `*serial*`, `*Serial*` (e.g., `serialNumber`)
   - `*sku*`, `*SKU*`
   - `*barcode*`, `*Barcode*`

**Step 2 - Check if format is constrained:**

**Backend (DTO):**
- Has `@Matches(/pattern/)` or `@IsNumberString()` → OK (format restricted)
- Has ONLY `@IsString()` + `@MaxLength()` → MISMATCH (accepts any string)

**Frontend (Zod/Yup):**
- Has `.regex(/pattern/)` → OK
- Has `.refine()` with numeric check → OK
- Has ONLY `.string().min().max()` → MISMATCH (accepts any string)

**Frontend (HTML input):**
- Has `pattern="[0-9]*"` or `inputMode="numeric"` → OK
- Has `type="number"` → OK (but field is string — may need `type="text"` + `inputMode="numeric"` instead)
- Has `type="text"` with no `pattern` or `inputMode` → MISMATCH

**Step 3 - Determine expected format:**

If the field has no existing constraint, suggest one based on context:
- If field is `number` on entity and existing data is all digits → suggest `@Matches(/^\d+$/)`
- If field is `code` → suggest `@Matches(/^[A-Z0-9-]+$/i)` (alphanumeric + dash)
- If field is `zipCode` → suggest `@Matches(/^\d{5}(-\d{4})?$/)` for US or appropriate locale pattern
- If field is `phoneNum` → suggest `@Matches(/^\+?[\d-]+$/)` or `@IsPhoneNumber()`
- If uncertain, flag for user decision: "Field `{name}` is varchar but name implies numeric/code — does it need a format pattern?"

**Step 4 - Flag mismatches:**

| Layer | Has Constraint | Issue |
|-------|---------------|-------|
| DTO only has `@IsString` | No format | **Flag**: DTO accepts any string for identifier field |
| Zod only has `z.string()` | No format | **Flag**: Frontend accepts any string for identifier field |
| HTML input `type="text"` | No pattern/inputMode | **Flag**: User can type non-numeric chars in identifier field |
| All layers lack constraint | No format anywhere | **Flag**: No format validation across entire stack |

**Exclude:**
- Fields that are intentionally free-text despite name (e.g., `description`, `content`)
- Fields with `unique: true` that serve as human-readable slugs
- Foreign key reference fields (ending in `Id` / `_id`)
- Fields that already have `@Matches()` or `.regex()` at any layer

**Report as:** `CHECK-38: Numeric-string field without format constraint (identifier field allows arbitrary text)`

**Example finding:**
```
CHECK-38: Product.skuNumber (varchar(50)) — DTO has @IsString only, Zod has z.string() only,
HTML input type="text" with no pattern. Field name implies numeric identifier but accepts any text.
Suggestion: Add @Matches(/^\d+$/) to DTO, .regex(/^\d+$/) to Zod, inputMode="numeric" to HTML.
```

### Check 39 - Date Field Missing Temporal Constraint

**Logic:**
For each field whose name matches `*birthday*`, `*Birthday*`, `*dob*`, `*Dob*`, `*DOB*`, `*birthDate*`, `*dateOfBirth*`, `*startDate*`, `*endDate*`, `*expiryDate*`, `*expireDate*`, `*dueDate*`, `*scheduledDate*`, `*appointmentDate*`, `*meetingDate*`:

**Step 1 - Classify temporal direction:**

| Pattern | Expected Direction | Reasoning |
|---------|-------------------|-----------|
| `birthday`, `dob`, `dateOfBirth`, `birthDate` | **Past only** | Birth dates cannot be in the future |
| `startDate`, `scheduledDate`, `appointmentDate`, `meetingDate`, `dueDate` | **Future only** (or today) | Scheduling dates are typically future |
| `endDate` | **After startDate** | End must be >= start |
| `expiryDate`, `expireDate` | **Future only** | Expiration is always future |

**Step 2 - Check backend:**
- DTO: should have `@MaxDate()` for past-only fields or `@MinDate()` for future-only fields, or custom `@ValidateIf()` / `@Validate()` with temporal logic → missing = flag
- Alternative: service layer validation that checks date direction → OK

**Step 3 - Check frontend:**
- Schema: should have `.refine(date => date < new Date())` for past-only, or `.refine(date => date >= new Date())` for future-only → missing = flag
- Input: `max` attribute for past-only date pickers (e.g., `max={today}`), `min` attribute for future-only → missing = flag
- Date picker component: `maxDate` / `minDate` prop → missing = flag

**Step 4 - Additional checks:**
- Birthday/DOB: should also have reasonable age limit (e.g., year > 1900) → missing = LOW
- endDate with startDate: should validate endDate >= startDate → missing cross-field validation = flag

**Exclude:**
- Generic `*date*` fields without clear temporal semantics (e.g., `createdDate`, `updatedDate` — auto-managed)
- Fields that serve as filters/search criteria (date range for querying, not data entry)

**Report as:** `CHECK-39: Date field missing temporal direction constraint`

**Example finding:**
```
CHECK-39: User.birthday (timestamp) — DTO has @IsDateString() only, no @MaxDate().
Frontend date picker has no max attribute. User can select future birth date.
Suggestion: Add @MaxDate(() => new Date()) to DTO, max={today} to date picker.
```

### Check 40 - Phone Field Without Format Validation

**Logic:**
For each field whose name matches `*phone*`, `*Phone*`, `*tel*`, `*Tel*`, `*mobile*`, `*Mobile*`, `*cellphone*`, `*contact*Number*`:

**Check all layers:**

**Backend (DTO):**
1. Has `@IsPhoneNumber()` (from class-validator) → OK
2. Has `@Matches()` with phone regex (e.g., `/^\+?[\d\s-()]+$/`) → OK
3. Has ONLY `@IsString()` → MISMATCH (accepts any string for phone number)

**Frontend (Zod/Yup):**
1. Has `.regex()` with phone pattern → OK
2. Has `.refine()` with phone validation logic → OK
3. Has ONLY `z.string()` / `.min()` / `.max()` → MISMATCH

**Frontend (HTML input):**
1. Has `type="tel"` → OK (enables phone keyboard on mobile)
2. Has `inputMode="tel"` → OK
3. Has `pattern` attribute with digit pattern → OK
4. Has `type="text"` with no `inputMode` or `pattern` → MISMATCH

**Semantic rules by field pattern:**
| Pattern | Expected Format | Expected Length |
|---------|----------------|----------------|
| `phone`, `tel`, `mobile` | Digits, +, -, spaces, parens | 7-15 chars |
| `contactNumber` | Same as above | 7-15 chars |

**Exclude:**
- Fields clearly used as identifiers, not actual phone numbers (e.g., `phoneVerified`, `phoneType`)
- Fields with enum/select type (phone type selector)

**Report as:** `CHECK-40: Phone field without format validation`

**Example finding:**
```
CHECK-40: User.phone (varchar(20)) — DTO has @IsString() + @MaxLength(20) only.
No @Matches or @IsPhoneNumber. Frontend uses type="text" with no inputMode.
Suggestion: Add @Matches(/^\+?[\d\s\-()]+$/) to DTO, type="tel" to input,
.regex(/^\+?[\d\s\-()]+$/) to Zod.
```

### Check 41 - Zip/Postal Code Without Format Validation

**Logic:**
For each field whose name matches `*zip*`, `*Zip*`, `*postal*`, `*Postal*`, `*zipCode*`, `*postalCode*`:

**Check all layers:**

**Backend (DTO):**
1. Has `@Matches()` with postal code regex → OK
2. Has `@IsPostalCode()` (class-validator) → OK
3. Has ONLY `@IsString()` → MISMATCH

**Frontend (Zod/Yup):**
1. Has `.regex()` with postal code pattern → OK
2. Has ONLY `z.string()` → MISMATCH

**Frontend (HTML input):**
1. Has `inputMode="numeric"` → OK
2. Has `pattern` attribute → OK
3. Has `maxLength` matching expected postal code length → OK

**Locale-aware suggestions:**
| Locale | Pattern | Length |
|--------|---------|--------|
| US | `/^\d{5}(-\d{4})?$/` | 5 or 10 |
| KR | `/^\d{5}$/` | 5 |
| UK | `/^[A-Z]{1,2}\d[A-Z\d]?\s?\d[A-Z]{2}$/i` | 5-8 |
| Generic | `/^[\dA-Z\s-]{3,10}$/i` | 3-10 |

**Note:** If the project has a known locale (from i18n config, CLAUDE.md, or `.env`), suggest the locale-specific pattern. Otherwise suggest the generic pattern and note for user decision.

**Report as:** `CHECK-41: Zip/postal code field without format validation`

### Check 42 - Percentage/Rate Out of Range

**Logic:**
For each field whose name matches `*rate*`, `*Rate*`, `*percentage*`, `*Percentage*`, `*percent*`, `*Percent*`, `*ratio*`, `*Ratio*`, `*progress*`, `*Progress*` (when numeric):

**Step 1 - Confirm the field is a percentage/rate (not a name collision):**
- Entity column type is numeric (`int`, `float`, `decimal`, `number`) → proceed
- Entity column type is `varchar`/`string` → skip (likely a label like `interestRateType`)
- If uncertain, check if field has numeric validators in DTO

**Step 2 - Check expected range:**

| Pattern | Expected Range | Reasoning |
|---------|---------------|-----------|
| `percentage`, `percent` | 0-100 | Standard percentage |
| `rate` (generic) | 0-100 or 0-1 | Depends on convention — flag if no range exists |
| `ratio` | 0-1 or 0-100 | Depends on convention |
| `progress` | 0-100 | Progress percentage |
| `discountRate`, `taxRate` | 0-100 | Business percentages |
| `completionRate`, `successRate` | 0-100 | Status percentages |

**Check backend:**
- DTO: should have `@Min(0)` AND `@Max(100)` (or `@Max(1)` for ratio) → missing either = flag
- Entity: column type should be numeric with appropriate precision

**Check frontend:**
- Schema: should have `.min(0).max(100)` or `.min(0).max(1)` → missing = flag
- Input: should have `min="0"` and `max="100"` (or `max="1"`) attributes → missing = flag
- Input: should have `step` attribute (e.g., `step="0.1"` or `step="1"`) → missing = LOW

**Report as:** `CHECK-42: Percentage/rate field missing range constraint`

**Example finding:**
```
CHECK-42: Discount.discountRate (decimal) — DTO has @IsNumber() only, no @Min/@Max.
Frontend schema has z.number() only, no .min()/.max(). Input has no min/max attributes.
User can enter 999% or -50%. Suggestion: Add @Min(0) @Max(100) to DTO, .min(0).max(100) to Zod.
```

### Check 43 - Duration/Time Allows Unrealistic Values

**Logic:**
For each field whose name matches `*minutes*`, `*Minutes*`, `*hours*`, `*Hours*`, `*duration*`, `*Duration*`, `*seconds*`, `*Seconds*`, `*time*` (when numeric, exclude `*timestamp*`, `*dateTime*`):

**Step 1 - Confirm the field represents a time duration (not a timestamp):**
- Entity column type is numeric (`int`, `float`, `number`) → proceed
- Entity column type is `timestamp`/`datetime` → skip (this is a point in time, not duration)
- Field name contains `timestamp`, `dateTime`, `At` suffix (e.g., `createdAt`) → skip

**Step 2 - Determine expected range by unit:**

| Pattern | Unit | Expected Min | Expected Max | Reasoning |
|---------|------|-------------|-------------|-----------|
| `*minutes*`, `*Minutes` | minutes | 0 | 1440 (24h) | Minutes in a day |
| `*hours*`, `*Hours` | hours | 0 | 24 (or 168 for weekly) | Hours in a day/week |
| `*seconds*`, `*Seconds` | seconds | 0 | 86400 (24h) | Seconds in a day |
| `*duration*`, `*Duration` | context-dependent | 0 | context-dependent | Check unit from docs/context |
| Domain-specific (e.g., `*walkingMinutes*`) | context-dependent | 0 | context-dependent | Infer from field name |

**Check backend:**
- DTO: should have `@Min(0)` (duration cannot be negative) → missing = flag as HIGH
- DTO: should have `@Max(N)` with reasonable upper bound → missing = flag as MEDIUM
- DTO: should have `@IsInt()` for whole-unit fields (minutes, hours) → missing `.int()` = LOW

**Check frontend:**
- Schema: should have `.min(0)` → missing = flag
- Schema: should have `.max(N)` with reasonable bound → missing = flag
- Input: should have `type="number"` with `min="0"` → missing = flag
- Input: ideally `step="1"` for whole-unit fields

**Exclude:**
- Timer/stopwatch fields that auto-increment (not user input)
- Timestamp/datetime fields (point in time, not duration)

**Report as:** `CHECK-43: Duration/time field allows unrealistic values`

**Example finding:**
```
CHECK-43: Task.estimatedMinutes (int) — DTO has @IsNumber() only, no @Min(0) or @Max().
User can enter -30 minutes or 99999. Suggestion: Add @Min(0) @Max(1440) to DTO,
.min(0).max(1440) to Zod, min="0" max="1440" to input.
```

### Check 44 - Score/Rating Without Defined Range

**Logic:**
For each field whose name matches `*score*`, `*Score*`, `*rating*`, `*Rating*`, `*intensity*`, `*Intensity*`, `*level*`, `*Level*`, `*grade*`, `*Grade*`, `*scale*`, `*Scale*`, `*satisfaction*`:

**Step 1 - Confirm the field is a numeric score (not a string label):**
- Entity column type is numeric (`int`, `float`, `decimal`) → proceed
- Entity column type is `varchar`/`string` or `enum` → skip (likely a category label like `gradeLevel: 'A' | 'B'`)

**Step 2 - Determine expected range by context:**

| Pattern | Expected Range | Reasoning |
|---------|---------------|-----------|
| `vasScore`, `painScore`, `painLevel` | 0-10 | VAS (Visual Analog Scale) standard |
| `intensity` | 1-10 or 0-10 | Exercise/pain intensity |
| `rating` | 1-5 or 0-5 | Common star rating |
| `satisfaction` | 1-5 or 1-10 | Survey satisfaction |
| `score` (generic) | context-dependent | Check requirements/docs |
| `level` | context-dependent | Could be 1-5, 1-10, etc. |
| `grade` | context-dependent | Numeric grade scale |
| `scale` | context-dependent | Check requirements |

**Step 3 - Check if range is defined anywhere:**

**Backend (DTO):**
- Has BOTH `@Min(N)` AND `@Max(N)` → OK (range defined)
- Has only `@Min(N)` or only `@Max(N)` → partial — flag as MEDIUM
- Has neither → flag as HIGH (unlimited range for a bounded concept)
- Has `@IsEnum()` with numeric enum → OK (enum restricts range)

**Frontend (Zod/Yup):**
- Has `.min(N).max(N)` → OK
- Has `.refine()` with range check → OK
- Has no range constraint → flag

**Frontend (HTML input):**
- Has `min` and `max` attributes → OK
- Renders as range slider or star-rating component → OK (inherently bounded)
- Has `type="number"` with no `min`/`max` → flag

**Step 4 - Cross-reference with project requirements:**
- If PRD or requirements docs define a specific scale (e.g., "VAS 0-10"), verify all layers match
- Mismatch between documented scale and implemented range → flag as HIGH

**Exclude:**
- String/enum fields that use levels as labels (e.g., `skillLevel: 'beginner' | 'intermediate'`)
- Computed scores that are calculated, not user-entered
- Fields ending in `Id` (e.g., `scorecardId`)

**Report as:** `CHECK-44: Score/rating field without defined min/max range`

**Example finding:**
```
CHECK-44: Review.rating (int) — DTO has @IsNumber() + @Min(1) but no @Max().
Frontend schema has z.number().min(1) but no .max(). Should be 1-5 or 1-10.
Suggestion: Add @Max(5) to DTO, .max(5) to Zod, max="5" to input.
```

### Check 27 - Long Text in Single-Line Input

**Logic:**
For each field whose name matches `*description*`, `*Description*`, `*content*`, `*Content*`, `*bio*`, `*Bio*`, `*memo*`, `*Memo*`, `*note*`, `*Note*`, `*summary*`, `*Summary*`, `*comment*`, `*Comment*`, `*message*`, `*Message*`, `*feedback*`, `*reason*`:

**Check frontend:**
- If the field renders as `<input>` or `<Input>` (single-line) → flag
- Should render as `<textarea>`, `<Textarea>`, or rich text editor component

**Additional evidence:**
- Entity/DTO maxLength > 255 → strong signal it should be textarea
- Entity column type is `text` (not `varchar`) → strong signal

**Exclude:**
- Fields already using textarea or rich text
- Fields with maxLength <= 100 (short enough for single line)
- `title` fields (typically single-line despite being descriptive)

**Report as:** `CHECK-27: Long text field uses single-line input instead of textarea`

### Check 28 - Whitespace-Only Accepted

**Logic:**
For each required string field in frontend forms:

**Check frontend schema:**
- Does the field chain include `.trim()` before `.min(1)`?
- Without `.trim()`, a value of `"   "` (spaces only) passes `.min(1)` → user thinks they entered data but it's just whitespace

**Patterns to check:**
- `z.string().min(1)` without preceding `.trim()` → flag
- `z.string().trim().min(1)` → OK
- `yup.string().required()` without `.trim()` → flag (Yup doesn't auto-trim)

**Check backend DTO:**
- Does DTO use `@Transform(({ value }) => value?.trim())` or similar? → OK
- No trim transform → flag

**Scope:** Only check required string fields (optional fields accepting whitespace is less critical).

**Report as:** `CHECK-28: Required string field accepts whitespace-only input`

### Check 29 - Empty String vs Null

**Logic:**
For optional string fields that map to nullable DB columns:

**The problem:**
- Frontend form sends `""` (empty string) for unfilled optional fields
- Backend saves `""` to DB instead of `null`
- This causes inconsistent data: some rows have `null`, others have `""` for the same semantic meaning (no value)

**Check:**
1. Entity column is `nullable: true`
2. DTO field is `@IsOptional()`
3. Frontend schema uses `.optional()` but does NOT transform empty string to null/undefined

**Fix patterns to look for (if present → OK):**
- Schema: `.transform(val => val === "" ? undefined : val)` or `.transform(val => val || undefined)`
- DTO: `@Transform(({ value }) => value || null)` or `@Transform(({ value }) => value === '' ? null : value)`
- Global validation pipe with `transform: true` and empty string handling

**Report as:** `CHECK-29: Optional string field may save empty string instead of null`

### Check 30 - Missing Autocomplete Attribute

**Logic:**
For form inputs matching known autocomplete-eligible field patterns:

| Field Pattern | Expected `autocomplete` |
|--------------|------------------------|
| `email` | `email` |
| `password` (login) | `current-password` |
| `password` (register) | `new-password` |
| `firstName` / `name` | `given-name` / `name` |
| `lastName` | `family-name` |
| `phone` / `tel` | `tel` |
| `address` | `street-address` |
| `city` | `address-level2` |
| `zip` / `postalCode` | `postal-code` |
| `country` | `country-name` |

**Check:**
- Does the `<input>` or `<Input>` have an `autoComplete` prop (React) or `autocomplete` attribute?
- Missing → flag as LOW (affects UX, browser autofill, password managers)

**Exclude:**
- Search inputs
- Custom filter inputs
- Fields inside admin/internal tools

**Report as:** `CHECK-30: Input missing autocomplete attribute for browser autofill`

### Check 31 - Format Hint Missing

**Logic:**
For fields that require a specific format but have no placeholder or helper text to guide the user:

| Field Pattern | Expected Format Hint |
|--------------|---------------------|
| `phone` / `tel` | `placeholder="010-1234-5678"` or format mask |
| `zip` / `postalCode` | `placeholder="12345"` |
| `*date*` (if text input) | `placeholder="YYYY-MM-DD"` |
| `*time*` (if text input) | `placeholder="HH:MM"` |
| `*url*` / `*Url*` | `placeholder="https://..."` |
| `*color*` (if text) | `placeholder="#FF0000"` |

**Check:**
1. Field requires specific format (has `@Matches()` regex or `.regex()` in schema)
2. Frontend input does NOT have `placeholder` attribute OR helper text nearby

**Also check for fields with regex but no format clue:**
- Backend has `@Matches(/pattern/)` → frontend should show expected format somewhere

**Report as:** `CHECK-31: Field requires specific format but has no placeholder hint`

### Check 32 - Error Message Not Displayed

**Logic:**
For each field that has validation rules (either in schema or DTO), check if the frontend form renders an error display component for that field.

**Step 1 - Identify validated fields:**
Any field in the frontend schema with validation chains (`.min()`, `.max()`, `.email()`, `.regex()`, `.url()`, etc.) or required fields.

**Step 2 - Check for error display:**
Look for error rendering patterns near each validated field:

**React Hook Form + Shadcn:**
- `<FormMessage />` inside `<FormItem>` for the field → OK
- `{errors.fieldName && ...}` → OK
- `formState.errors.fieldName` referenced → OK

**React Hook Form (manual):**
- `{errors.fieldName?.message}` → OK
- `<p>{errors.fieldName?.message}</p>` → OK

**Vee-validate (Vue):**
- `<ErrorMessage name="fieldName" />` → OK
- `errors.fieldName` in template → OK

**Missing = flag:**
- Field has validation but NO error display component found within the same form
- The error message exists in schema but will never be shown to the user

**Report as:** `CHECK-32: Validated field has no error message display`

### Check 33 - Select Default Value Missing

**Logic:**
For each required Select/Dropdown field:

**Step 1 - Identify required selects:**
- Schema field is required (no `.optional()`) AND
- Frontend renders `<Select>`, `<select>`, or custom dropdown component

**Step 2 - Check for proper default handling:**

**Problem patterns:**
- `<option value="">선택하세요</option>` as first option + no `defaultValue` → user can submit the empty placeholder
- `<SelectItem value="">` as placeholder → empty string submits
- No `defaultValue` prop AND no first option pre-selected → first option auto-selected (may be unintended)
- Schema allows empty string (`z.string().min(1)` catches it, but `z.string()` alone doesn't)

**OK patterns:**
- `<option value="" disabled>선택하세요</option>` → disabled placeholder, can't submit
- `defaultValue={someValue}` with a valid default → OK
- Schema has `.min(1)` or `.refine(val => val !== "")` → catches empty selection
- `placeholder` prop on Select component (Shadcn) + schema validation → OK if schema rejects empty

**Report as:** `CHECK-33: Required select has submittable empty placeholder`

### Check 34 - Double Submission Possible

**Logic:**
For each form that submits data (POST/PUT/PATCH), check if the submit button prevents double-click:

**Step 1 - Find form submission handlers:**
- `onSubmit`, `handleSubmit`, `mutation.mutate`, `mutateAsync`

**Step 2 - Check for submission guards:**

**OK patterns (any one is sufficient):**
- `disabled={isSubmitting}` or `disabled={isPending}` or `disabled={isLoading}` on submit button
- `loading` prop on button component
- `mutation.isPending` used to disable button
- `formState.isSubmitting` from react-hook-form used
- Custom `isLoading` state that disables button during API call
- `e.preventDefault()` with debounce/throttle on handler

**MISMATCH patterns:**
- Submit button has no `disabled` prop tied to loading state
- `useMutation` used but `isPending` never referenced for button state
- Form has `onSubmit` but button stays enabled throughout

**Severity context:**
- Forms that create data (POST) → MEDIUM (duplicate records)
- Forms that trigger payments/transactions → HIGH (mention in report)
- Forms that update data (PUT/PATCH) → LOW (idempotent, less risk)

**Report as:** `CHECK-34: Form submit button not disabled during submission`

### Check 35 - Conditional Required Mismatch

**Logic:**
For fields with conditional validation in the DTO (`@ValidateIf()`):

**Step 1 - Find conditional fields:**
- DTO: `@ValidateIf(o => o.someField === 'value')` → field is required only when `someField === 'value'`
- Extract the condition and the triggering field

**Step 2 - Check frontend mirrors the condition:**

**Check schema:**
- Does the Zod schema use `.refine()` or `.superRefine()` to implement the same conditional logic?
- Does it use a discriminated union (`z.discriminatedUnion()`) or `.and()` for conditional fields?
- Or does it use a simple `.optional()` (always optional, ignoring condition)?

**Check UI:**
- When the condition is NOT met, is the field hidden or disabled?
- When the condition IS met, does the field show `*` (required indicator)?
- Does the field stay visible and required regardless of condition? → mismatch

**Common patterns:**
- DTO: `@ValidateIf(o => o.type === 'ONLINE')` on `meetingUrl`
  - Frontend should: show `meetingUrl` only when type select = 'ONLINE', and mark as required
  - Mismatch: `meetingUrl` always shown and always optional, OR always shown and always required

**Report as:** `CHECK-35: Conditional required field not synced with frontend`

### Check 36 - Array Field Min/Max Missing

**Logic:**
For fields validated as arrays in the DTO:

**Step 1 - Identify array fields from backend:**
- DTO: `@IsArray()`, `@ArrayMinSize(N)`, `@ArrayMaxSize(N)`, `@ArrayNotEmpty()`
- Entity: column type `simple-array`, `jsonb`, or relation with `@OneToMany()`

**Step 2 - Check frontend enforcement:**

**Schema:**
- `z.array(...)` with `.min(N)` → OK
- `z.array(...)` without `.min()` when DTO has `@ArrayMinSize(N)` → MISMATCH
- `z.array(...)` with `.max(M)` matching `@ArrayMaxSize(M)` → OK

**UI:**
- If `@ArrayMinSize(1)` or `@ArrayNotEmpty()`: does the UI prevent submitting with 0 items?
- Does the UI show "at least N items required" message?
- Add/remove item buttons: does removing the last item get blocked when min = 1?

**Report as:** `CHECK-36: Array field min/max size not enforced in frontend`

### Check 37 - Default Value Not Shown

**Logic:**
For entity columns with default values:

**Step 1 - Find columns with defaults:**
- Entity: `@Column({ default: 'VALUE' })`, `@Column({ default: 0 })`, `@Column({ default: true })`
- Prisma: `@default(value)`

**Step 2 - Check frontend shows the default:**

**Check form initialization:**
- `useForm({ defaultValues: { field: 'VALUE' } })` → OK
- `defaultValue="VALUE"` on input → OK
- `placeholder="VALUE"` showing the default → Acceptable
- No default shown, field appears empty → flag

**Why this matters:**
- User sees an empty field, doesn't know there's a default
- User might enter a value unnecessarily, or worry that leaving it blank will cause an error
- If the field is optional + has DB default: user should know "if you leave this blank, it defaults to X"

**Exclude:**
- Auto-increment / UUID defaults (system-generated, not user-relevant)
- `createdAt`/`updatedAt` timestamp defaults
- Boolean fields where the default is obvious from the UI (checkbox unchecked = false)
- Fields not shown in the form (backend-only defaults)

**Report as:** `CHECK-37: Entity has default value but form doesn't show it`

### Check 45 - Validation Mode onSubmit Only

**Logic:**
For each form file that uses `useForm` (React Hook Form), `useForm` (Vee-validate), or similar:

**Step 1 - Find useForm calls:**
- React Hook Form: `useForm<...>({ ... })`
- Vee-validate: `useForm({ ... })`

**Step 2 - Check for validation mode:**

**OK patterns (any one is sufficient):**
- `mode: 'onBlur'` → validates when field loses focus → OK
- `mode: 'onChange'` → validates on every keystroke → OK
- `mode: 'onTouched'` → validates after first interaction → OK
- `mode: 'all'` → validates on both change and blur → OK

**MISMATCH patterns:**
- `mode: 'onSubmit'` → explicit onSubmit mode → errors only show after clicking submit button
- No `mode` property at all → defaults to `'onSubmit'` in React Hook Form → SAME PROBLEM
- The user types invalid input (e.g., Korean in a username field, 3-char password) and gets NO feedback until they submit

**Why this matters:**
- Users type invalid data and see no indication of error
- Help text says "only letters and numbers" but Korean is freely accepted with no error
- Password hints say "minimum 8 characters" but no error appears as they type
- The validation rules exist (Zod schema has `.regex()`, `.min(8)`) but the UX doesn't show errors until submit
- This is a common cause of "validation exists but doesn't work" bug reports

**Exclude:**
- Simple forms with only 1-2 fields (login form) where onSubmit is acceptable
- Forms that use custom onChange handlers with manual `trigger()` calls (already has real-time validation)

**Report as:** `CHECK-45: Form uses onSubmit-only validation mode (no real-time feedback)`

### Check 46 - String Field Without maxLength

**Logic:**
For each mapped string field that is user-input (not system-set):

**Step 1 - Identify string fields:**
- Entity: column type is `varchar`, `text`, `string` (or TypeScript `string`)
- DTO: `@IsString()` without `@MaxLength()` — no upper bound
- Frontend: `z.string()` without `.max()` — no upper bound

**Step 2 - Check if ANY layer constrains the length:**
A field is "unbounded" if ALL of the following are true:
1. Entity column has no explicit `length` (e.g., `@Column({ length: 255 })` is bounded)
2. DTO has no `@MaxLength(N)` decorator
3. Frontend schema has no `.max(N)` constraint
4. HTML input has no `maxLength` attribute

**Step 3 - Evaluate risk by field type:**
- `*name*`, `*title*` fields: should typically be 50-255 chars max
- `*description*`, `*content*`, `*memo*` fields: should typically be 500-5000 chars max
- `*username*` fields: should typically be 3-50 chars max
- Any other string field accepting user input: should have SOME upper bound

**Exclude:**
- Text/longtext columns that are intentionally unbounded (e.g., rich text editors, markdown content)
- Fields not exposed to user input (system-managed strings)
- Password fields (hashed, length doesn't matter for storage)

**Report as:** `CHECK-46: String field has no maxLength constraint at any layer (unbounded input)`

### Check 47 - Schema maxLength Without HTML maxLength

**Logic:**
For each mapped string field where the schema or DTO has a max length constraint:

**Step 1 - Find fields with schema/DTO maxLength:**
- DTO: `@MaxLength(N)` — extract N
- Frontend schema: `.max(N)` on string field — extract N

**Step 2 - Check if HTML input has matching maxLength:**
- `maxLength={N}` on `<Input>` or `<input>` → OK (prevents typing beyond limit)
- `maxLength` attribute present with same or smaller value → OK
- No `maxLength` attribute → MISMATCH

**Why this matters:**
- Without HTML `maxLength`, the user can type unlimited characters
- The error only appears after blur/submit (depending on mode) — poor UX
- HTML `maxLength` prevents input at the browser level — instant feedback, no error needed
- The schema `.max(N)` catches it on validation, but by then the user has already typed a long string

**Exclude:**
- Textarea fields (long text is expected, maxLength would be UX-hostile)
- Fields where the max is very large (> 500) — HTML maxLength less useful
- Select/dropdown fields (no typing involved)

**Report as:** `CHECK-47: Schema has maxLength but HTML input missing maxLength attribute`

### Check 48 - Restricted-Format Field Without Real-Time Input Filtering

**Logic:**
Detect fields that have format constraints (pattern, regex, inputMode) but lack real-time input filtering — allowing users to type disallowed characters that are only caught on submit/blur.

**Why this matters:**
- HTML `pattern` attribute does NOT prevent typing invalid characters — it only prevents form submission
- `inputMode="numeric"` only suggests a keyboard on mobile — on desktop, any character can still be typed
- Zod `.regex()` only validates on submit/blur — user can type freely until then
- `type="number"` blocks letters on most browsers, but `type="text"` with `pattern="[0-9]*"` does NOT

**Step 1 - Identify restricted-format fields:**
A field needs real-time filtering if ANY of these are true:
1. Has `pattern` attribute (e.g., `pattern="[0-9]*"`)
2. Has `inputMode="numeric"` or `inputMode="tel"` with `type="text"`
3. Has Zod `.regex()` with a character-class restriction (e.g., `/^\d+$/`, `/^[A-Z0-9]+$/`)
4. Is identified as numeric-string by Check 38 (field name implies numeric but stored as string)
5. Has `type="number"` with `onKeyDown` that blocks `-`, `e`, `+` but no `onChange` to strip pasted content

**Step 2 - Check for real-time filtering handler:**

A field has real-time filtering if ANY of these exist:
- `onChange` handler with `.replace(/[^allowed]/g, "")` or similar regex stripping → OK
- `onInput` handler that filters characters → OK
- `onKeyDown` handler that calls `e.preventDefault()` for disallowed keys → PARTIAL (blocks keyboard but not paste)
- `onPaste` handler that filters pasted content → OK (for paste specifically)
- Custom input mask component (e.g., `react-input-mask`, `IMaskInput`) → OK

**Completeness check:**
- `onKeyDown` alone is INSUFFICIENT — user can paste invalid content via Ctrl+V or right-click paste
- `onChange` with `.replace()` is BEST — catches both typing and pasting
- `onInput` with filtering is also acceptable

**Step 3 - Flag mismatches:**

| Has Constraint | Has onChange/onInput Filter | Issue |
|---------------|---------------------------|-------|
| `pattern="[0-9]*"` | No handler | **Flag**: pattern only validates on submit, user can type letters |
| `inputMode="numeric"` | No handler | **Flag**: inputMode is keyboard hint only, no enforcement on desktop |
| Zod `.regex(/^\d+$/)` | No handler | **Flag**: regex only validates on blur/submit |
| `type="number"` + `onKeyDown` | No onChange | **Partial**: blocks keyboard but not paste of invalid content |
| `pattern` + `onChange` with `.replace()` | Yes | OK — real-time filtering in place |

**Exclude:**
- Fields with `type="number"` (browser natively blocks non-numeric keyboard input for most cases)
- Fields where the format restriction is complex and real-time filtering would be impractical (e.g., email format)
- Select/dropdown fields (no free typing)
- Fields with input mask libraries applied

**Report as:** `CHECK-48: Restricted-format field without real-time input filtering (user can type disallowed characters)`

**Example finding:**
```
CHECK-48: Exercise.number — Has pattern="[0-9]*" + inputMode="numeric" + Zod .regex(/^\d+$/),
but no onChange/onInput handler to strip non-digit characters in real-time.
User can type letters/special chars on desktop; error only appears on submit.
Fix: Add onChange handler with e.target.value.replace(/[^0-9]/g, "") to filter in real-time.
```

**Fix pattern (React Hook Form):**
```tsx
{...register("fieldName", {
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => {
    e.target.value = e.target.value.replace(/[^0-9]/g, "");
    setValue("fieldName", e.target.value, { shouldValidate: true });
  },
})}
```

---

## Output Format

Print the report directly in the conversation as markdown:

```markdown
# QA Inputs Audit Report

**Date**: YYYY-MM-DD
**Scope**: All modules | {module name}
**Checks Run**: All | {check numbers}
**Stack Detected**: {backend framework} + {ORM} + {frontend framework} + {validation library}

## Summary

| # | Check | Issues |
|---|-------|--------|
| 1 | Required without asterisk | X |
| 2 | Regex without error message | X |
| 3 | Optional with asterisk | X |
| 4 | Field name mismatch | X |
| 5 | DB column no input route | X |
| 6 | maxLength mismatch | X |
| 7 | minLength mismatch | X |
| 8 | Enum value mismatch | X |
| 9 | i18n hardcoded | X |
| 10 | Numeric type mismatch | X |
| 11 | Date type mismatch | X |
| 12 | Boolean type mismatch | X |
| 13 | Nullable ↔ Required conflict | X |
| 14 | Orphan frontend field | X |
| 15 | Orphan DTO field | X |
| 16 | Decimal precision mismatch | X |
| 17 | Numeric range mismatch | X |
| 18 | Unique constraint without hint | X |
| 19 | XSS-prone input | X |
| 20 | File upload without constraint | X |
| 21 | Name field too short | X |
| 22 | Email without validation | X |
| 23 | Password too weak | X |
| 24 | URL without validation | X |
| 25 | Count allows negative/decimal | X |
| 26 | Price allows negative | X |
| 27 | Long text in single-line | X |
| 28 | Whitespace-only accepted | X |
| 29 | Empty string vs null | X |
| 30 | Missing autocomplete | X |
| 31 | Format hint missing | X |
| 32 | Error message not displayed | X |
| 33 | Select default value missing | X |
| 34 | Double submission possible | X |
| 35 | Conditional required mismatch | X |
| 36 | Array field min/max missing | X |
| 37 | Default value not shown | X |
| 38 | Numeric-string no format | X |
| 45 | Validation mode onSubmit only | X |
| 46 | String field without maxLength | X |
| 47 | Schema maxLength no HTML maxLength | X |
| | **Total** | **X** |

## CRITICAL Severity

### Check 19: XSS-Prone Input

| Module | Field | DTO File | Rendered In | Sanitized | Risk |
|--------|-------|----------|-------------|-----------|------|
| chat | content | create-message.dto.ts | chat-bubble.tsx:42 | No | User HTML rendered unsanitized |

## HIGH Severity

### Check 1: Required Fields Without Asterisk (*)

| Module | DTO Field | DTO File | Frontend File | Label Text |
|--------|-----------|----------|---------------|------------|

### Check 5A: DB Columns Completely Missing From Create DTO

| Entity | Column | Type | Nullable | Has Default |
|--------|--------|------|----------|-------------|

### Check 5B: Non-nullable Columns With @IsOptional in Create DTO (Soft Missing)

| Entity | Column | Type | DTO Field | DTO Decorator | Frontend Fallback | Risk |
|--------|--------|------|-----------|---------------|-------------------|------|

### Check 6: maxLength Mismatch

| Module | Field | Entity | DTO | Schema | Frontend | Gap |
|--------|-------|--------|-----|--------|----------|-----|

### Check 10: Numeric Type Mismatch

| Module | Field | Entity Type | DTO Decorator | Schema Type | Input Type | Gap |
|--------|-------|-------------|---------------|-------------|------------|-----|

### Check 11: Date Type Mismatch

| Module | Field | Entity Type | DTO Decorator | Schema Type | Input Type | Gap |
|--------|-------|-------------|---------------|-------------|------------|-----|

### Check 12: Boolean Type Mismatch

| Module | Field | Entity Type | DTO Decorator | Schema Type | Input Type | Gap |
|--------|-------|-------------|---------------|-------------|------------|-----|

### Check 13: Nullable ↔ Required Conflict

| Module | Field | Entity Nullable | DTO Required | Has Default | Risk |
|--------|-------|-----------------|-------------|-------------|------|

### Check 22: Email Field Without Validation

| Module | Field | DTO Has @IsEmail | Schema Has .email() | Input type | Gap |
|--------|-------|-----------------|---------------------|------------|-----|

### Check 23: Password Field Too Weak

| Module | Field | DTO MinLength | Schema MinLength | Input Type | Gap |
|--------|-------|--------------|-----------------|------------|-----|

### Check 45: Validation Mode onSubmit Only

| App | Form File | useForm Mode | Fields With Validation | Impact |
|-----|-----------|-------------|----------------------|--------|
| frontend-dashboard | create-user-modal.tsx | default (onSubmit) | username (.regex), password (.min(8)) | Errors only visible after submit — no real-time feedback |

### Check 46: String Field Without maxLength (Unbounded Input)

| Module | Field | DTO @MaxLength | Schema .max() | HTML maxLength | Suggested Max |
|--------|-------|---------------|---------------|----------------|---------------|
| users | fullName | missing | missing | missing | 50 |

### Check 32: Error Message Not Displayed

| App | Form File | Field | Has Validation | Has Error Display | Gap |
|-----|-----------|-------|---------------|-------------------|-----|

### Check 33: Select Default Value Missing

| App | Form File | Field | Required | Has Disabled Placeholder | Schema Rejects Empty | Gap |
|-----|-----------|-------|----------|--------------------------|---------------------|-----|

## MEDIUM Severity

### Check 3: Optional Fields With Asterisk

| Module | DTO Field | DTO File | Frontend File |
|--------|-----------|----------|---------------|

### Check 7: minLength Mismatch

| Module | Field | DTO Value | Schema Value | Gap |
|--------|-------|-----------|--------------|-----|

### Check 8: Enum Value Mismatch

| Module | Enum | Backend Values | Frontend Values | Missing In |
|--------|------|----------------|-----------------|------------|

### Check 14: Orphan Frontend Field

| App | Form File | Field | Target DTO | Status |
|-----|-----------|-------|------------|--------|

### Check 15: Orphan DTO Field

| Module | DTO | Field | Required | Any Frontend Sends? |
|--------|-----|-------|----------|---------------------|

### Check 16: Decimal Precision Mismatch

| Module | Field | DB Precision | Frontend Step | Gap |
|--------|-------|-------------|---------------|-----|

### Check 17: Numeric Range Mismatch

| Module | Field | DTO Min | DTO Max | Input Min | Input Max | Gap |
|--------|-------|---------|---------|-----------|-----------|-----|

### Check 20: File Upload Without Constraint

| App | Form File | Field | Has Accept | Has Size Limit | Backend Validates |
|-----|-----------|-------|------------|----------------|-------------------|

### Check 21: Name Field Too Short

| Module | Field | DTO MinLength | Schema MinLength | Expected Min | Gap |
|--------|-------|--------------|-----------------|-------------|-----|

### Check 24: URL Field Without Validation

| Module | Field | DTO Has @IsUrl | Schema Has .url() | Input Type | Gap |
|--------|-------|---------------|-------------------|------------|-----|

### Check 25: Count/Quantity Allows Negative or Decimal

| Module | Field | Has @IsInt | Has @Min(0) | Schema .int() | Input step/min | Gap |
|--------|-------|-----------|-------------|---------------|----------------|-----|

### Check 26: Price/Amount Allows Negative

| Module | Field | Has @Min(0) | Schema .min(0) | Input min | Gap |
|--------|-------|------------|---------------|-----------|-----|

### Check 38: Numeric-String Without Format Constraint

| Module | Field | Entity Type | DTO Constraint | Zod Constraint | HTML Input | Gap |
|--------|-------|-------------|----------------|----------------|------------|-----|
| products | skuCode | varchar(50) | @IsString only | z.string() only | type="text", no pattern | No format validation — accepts arbitrary text |

### Check 39: Date Field Missing Temporal Constraint

| Module | Field | Direction | DTO Has MaxDate/MinDate | Schema Has Refine | Input Has max/min | Gap |
|--------|-------|-----------|------------------------|-------------------|-------------------|-----|

### Check 40: Phone Field Without Format Validation

| Module | Field | DTO Has @IsPhoneNumber/@Matches | Schema Has .regex() | Input type/inputMode | Gap |
|--------|-------|---------------------------------|---------------------|----------------------|-----|

### Check 42: Percentage/Rate Out of Range

| Module | Field | DTO Min | DTO Max | Schema .min() | Schema .max() | Input min/max | Gap |
|--------|-------|---------|---------|---------------|---------------|---------------|-----|

### Check 43: Duration/Time Allows Unrealistic Values

| Module | Field | Unit | DTO @Min | DTO @Max | Schema Constraints | Input min/max | Gap |
|--------|-------|------|---------|---------|-------------------|---------------|-----|

### Check 44: Score/Rating Without Defined Range

| Module | Field | Expected Range | DTO @Min | DTO @Max | Schema Constraints | Input min/max | Gap |
|--------|-------|---------------|---------|---------|-------------------|---------------|-----|

### Check 27: Long Text in Single-Line Input

| App | Form File | Field | Input Element | MaxLength | Suggested |
|-----|-----------|-------|---------------|-----------|-----------|

### Check 28: Whitespace-Only Accepted

| App | Form File | Field | Schema Has .trim() | DTO Has Trim Transform | Gap |
|-----|-----------|-------|--------------------|----------------------|-----|

### Check 47: Schema maxLength Without HTML maxLength

| App | Form File | Field | Schema/DTO Max | HTML maxLength | Gap |
|-----|-----------|-------|---------------|----------------|-----|
| frontend-dashboard | create-user-modal.tsx | username | .max(20) | missing | User can type > 20 chars, error only on blur/submit |

### Check 48: Restricted-Format Field Without Real-Time Input Filtering

| App | Form File | Field | Format Constraint | Has onChange/onInput Filter | Gap |
|-----|-----------|-------|-------------------|---------------------------|-----|
| {app-name} | create-item-modal.tsx | quantity | pattern="[0-9]*" + inputMode="numeric" + .regex(/^\d+$/) | No handler | User can type letters/special chars on desktop |

### Check 34: Double Submission Possible

| App | Form File | Method | Submit Button Has Disabled/Loading | Gap |
|-----|-----------|--------|-----------------------------------|-----|

### Check 35: Conditional Required Mismatch

| Module | Field | DTO Condition | Frontend Mirrors Condition | Gap |
|--------|-------|--------------|---------------------------|-----|

### Check 36: Array Field Min/Max Missing

| Module | Field | DTO ArrayMinSize | DTO ArrayMaxSize | Schema .min() | Schema .max() | Gap |
|--------|-------|-----------------|-----------------|---------------|---------------|-----|

## LOW Severity

### Check 2: Regex Without Error Message

| Layer | Field | File | Pattern |
|-------|-------|------|---------|

### Check 9: i18n Hardcoded Strings

| App | File | Line | Type | Hardcoded String |
|-----|------|------|------|------------------|

### Check 18: Unique Constraint Without Hint

| Entity | Field | Constraint | Frontend Check | Risk |
|--------|-------|------------|----------------|------|

### Check 29: Empty String vs Null

| Module | Field | Entity Nullable | DTO Optional | Schema Transforms Empty | Gap |
|--------|-------|-----------------|-------------|------------------------|-----|

### Check 30: Missing Autocomplete Attribute

| App | Form File | Field | Expected Autocomplete | Has Attribute |
|-----|-----------|-------|-----------------------|---------------|

### Check 41: Zip/Postal Code Without Format Validation

| Module | Field | DTO Has @Matches/@IsPostalCode | Schema Has .regex() | Input pattern/inputMode | Gap |
|--------|-------|---------------------------------|---------------------|------------------------|-----|

### Check 31: Format Hint Missing

| App | Form File | Field | Has Regex/Format | Has Placeholder | Gap |
|-----|-----------|-------|------------------|-----------------|-----|

### Check 37: Default Value Not Shown

| Module | Field | Entity Default | Form Shows Default | Gap |
|--------|-------|---------------|-------------------|-----|

## INFO

### Check 4: Field Name Mismatches

| Layer Pair | Field A | Field B | File A | File B |
|------------|---------|---------|--------|--------|

---

## Clarifying Questions

> The following items need your input before fixes can be applied.
> Straightforward fixes (e.g., password min mismatch, missing .trim(), showPassword default) will proceed without asking.

**Q1. [Topic]**
- Finding: [description]
- Options:
  - A) [option]
  - B) [option]
- Default recommendation: [recommendation]
```

## Scanning Patterns Reference

### Backend Patterns (NestJS / class-validator)

| What | Grep Pattern | Extract |
|------|-------------|---------|
| Entity files | `Glob: **/entities/*.entity.ts` | Class name, columns |
| DTO files | `Glob: **/dtos/*.dto.ts` or `**/dto/*.dto.ts` | Class name, fields |
| Required field | `@IsNotEmpty()` | Field below decorator |
| Optional field | `@IsOptional()` | Field below decorator |
| Regex validation | `@Matches(` | Pattern + message option |
| Max length | `@MaxLength(` | Number in parentheses |
| Min length | `@MinLength(` | Number in parentheses |
| Enum validation | `@IsEnum(` | Enum type name |
| Conditional | `@ValidateIf(` | Condition |
| Column def | `@Column(` | nullable, length, default, type, precision, scale |
| Enum definition | `export enum` | Values |
| Numeric type (DTO) | `@IsNumber(` or `@IsInt(` | Field below decorator |
| Numeric constraint | `@IsPositive(` or `@Min(` or `@Max(` | Numeric constraints |
| Numeric coercion | `@Type(() => Number)` | Field below decorator |
| Numeric column | `type: 'int'` or `'integer'` or `'float'` or `'decimal'` | Column type |
| Date type (DTO) | `@IsDate(` or `@IsDateString(` or `@IsISO8601(` | Field below decorator |
| Date column | `type: 'timestamp'` or `'date'` or `'datetime'` | Column type |
| Boolean type (DTO) | `@IsBoolean(` | Field below decorator |
| Boolean column | `type: 'boolean'` or `'bool'` | Column type |
| Unique constraint | `unique: true` or `@Unique(` or `@Index.*unique` | Column/entity |
| File upload | `@UseInterceptors(FileInterceptor` or `@UploadedFile` | Controller |
| Phone validation | `@IsPhoneNumber(` or `@Matches(.*phone` | DTO decorator |
| Postal code validation | `@IsPostalCode(` | DTO decorator |
| Max date | `@MaxDate(` | DTO decorator |
| Min date | `@MinDate(` | DTO decorator |

### Frontend Patterns (React + Zod)

| What | Grep Pattern | Extract |
|------|-------------|---------|
| Form files | `useForm\|zodResolver` | File paths |
| Zod schema | `z.object({` | Field definitions |
| Required (zod) | `.min(1,` without `.optional()` | Field name |
| Optional (zod) | `.optional()` or `.nullable()` | Field name |
| Regex (zod) | `.regex(` | Pattern + message |
| Max (zod) | `.max(` | Number |
| Min (zod) | `.min(` | Number |
| Enum (zod) | `z.nativeEnum(` or `z.enum(` | Values |
| Asterisk (label) | `} *</Label>` or `" *</Label>` | Label text |
| Asterisk (prop) | `required` prop | Component |
| maxLength (input) | `maxLength={` or `maxLength=` | Number |
| Select options | `<SelectItem value=` or `<option value=` | Values |
| i18n wrapper | `{t(` or `t("` | Key |
| Placeholder | `placeholder="` | Text |
| Numeric zod type | `z.number(` or `z.coerce.number(` | Field name |
| Date zod type | `z.date(` or `z.coerce.date(` | Field name |
| Boolean zod type | `z.boolean(` | Field name |
| Input type attr | `type="number"` or `type="text"` or `type="date"` | Input element |
| Input mode attr | `inputMode="numeric"` or `inputMode="decimal"` | Input element |
| Input range | `min={` or `max={` or `step={` | Input attributes |
| Date picker | `DatePicker` or `Calendar` or `type="date"` | Component |
| File input | `type="file"` or `Dropzone` or `FileInput` or `Upload` | Component |
| File accept | `accept=` | Allowed types |
| XSS danger | `dangerouslySetInnerHTML` | Component + data source |
| Textarea | `<textarea` or `<Textarea` | Component |
| Autocomplete | `autoComplete=` or `autocomplete=` | Attribute value |
| Trim (zod) | `.trim()` | Chain position |
| Transform empty | `.transform(` near empty string check | Transform logic |
| Error display | `<FormMessage` or `errors\.` or `formState.errors` | Error component |
| Select component | `<Select` or `<select` or `<Dropdown` | Component |
| Disabled placeholder | `<option.*disabled` or `disabled` on placeholder | Option element |
| Submit button | `type="submit"` or `onSubmit` | Button element |
| Button disabled | `disabled={` on submit button | Loading state |
| isPending/isLoading | `isPending\|isLoading\|isSubmitting` | State variable |
| Array schema | `z.array(` | Array field |
| Array validators | `@IsArray\|@ArrayMinSize\|@ArrayMaxSize\|@ArrayNotEmpty` | DTO decorator |
| Default value (form) | `defaultValues` or `defaultValue=` | Form initialization |
| Default value (entity) | `default:` in `@Column(` | Entity column |
| useForm mode | `mode:` inside `useForm({` | Validation trigger mode |
| useForm mode value | `'onBlur'\|'onChange'\|'onTouched'\|'all'\|'onSubmit'` | Mode setting |
| ValidateIf | `@ValidateIf(` | Condition expression |
| Date picker max/min | `maxDate\|minDate\|max=\|min=` on date components | Date constraint |
| Phone input type | `type="tel"\|inputMode="tel"` | Input element |
| Slider/range | `<Slider\|type="range"\|<RangeInput` | Rating/score component |
| Star rating | `<Rating\|<StarRating\|<Stars` | Rating component |

### Semantic Field Name Patterns

These patterns drive Checks 21-26. Match field names case-insensitively:

| Pattern | Matches | Check |
|---------|---------|-------|
| `*name*`, `*Name` | name, firstName, lastName, displayName, username | 21 |
| `*email*`, `*Email` | email, userEmail, contactEmail | 22 |
| `*password*`, `*pwd*` | password, newPassword, confirmPassword, pwd | 23 |
| `*url*`, `*Url*`, `*link*`, `*website*` | url, videoUrl, imageUrl, websiteLink | 24 |
| `*count*`, `*quantity*`, `*qty*`, `*sets*`, `*reps*` | count, itemCount, quantity, sets, reps | 25 |
| `*price*`, `*amount*`, `*cost*`, `*fee*`, `*charge*` | price, totalAmount, shippingCost, fee | 26 |
| `*description*`, `*content*`, `*bio*`, `*memo*`, `*note*`, `*summary*`, `*comment*`, `*message*`, `*feedback*`, `*reason*` | description, content, bio, notes | 27 |
| `*birthday*`, `*dob*`, `*birthDate*`, `*dateOfBirth*`, `*startDate*`, `*endDate*`, `*expiryDate*`, `*dueDate*`, `*scheduledDate*`, `*appointmentDate*`, `*meetingDate*` | birthday, dob, startDate, endDate, dueDate | 39 |
| `*phone*`, `*tel*`, `*mobile*`, `*cellphone*`, `*contact*Number*` | phone, tel, mobile, contactNumber | 40 |
| `*zip*`, `*postal*`, `*zipCode*`, `*postalCode*` | zipCode, postalCode | 41 |
| `*rate*`, `*percentage*`, `*percent*`, `*ratio*`, `*progress*` (numeric) | discountRate, taxRate, progress, completionRate | 42 |
| `*minutes*`, `*hours*`, `*duration*`, `*seconds*` (numeric, exclude `*timestamp*`, `*dateTime*`) | totalMinutes, sessionDuration, breakHours, duration | 43 |
| `*score*`, `*rating*`, `*intensity*`, `*level*`, `*grade*`, `*scale*`, `*satisfaction*` (numeric) | score, rating, intensity, satisfaction, priorityLevel | 44 |

---

## Skip Patterns (False-Positive Avoidance)

These field patterns commonly trigger false positives and should be skipped or deprioritized during auditing:

- **Fields ending in `Id`** that are set programmatically (e.g., `userId`, `createdById`, `organizationId`) -- these are not user inputs
- **Fields set by decorators/interceptors** (e.g., `createdBy`, `updatedBy`, `createdAt`, `updatedAt`) -- injected server-side
- **Computed/virtual fields** that exist in the entity but are derived from other data, not directly stored
- **File path fields** that look like URLs but are not user-entered URLs (e.g., `imagePath`, `avatarPath`, `filePath`) -- exclude from Check 24 (URL validation)
- **Internal status fields** (e.g., `status`, `isActive`, `isVerified`) that are managed by business logic, not user forms
- **Auto-managed timestamp fields** (e.g., `createdAt`, `updatedAt`, `deletedAt`, `lastLoginAt`) -- exclude from Check 39 (date temporal constraint)
- **Timer/stopwatch fields** that auto-increment and are not user-entered -- exclude from Check 43 (duration constraint)
- **Computed score fields** that are calculated server-side, not entered by users -- exclude from Check 44 (score range)
- **String fields named `*level*`/`*grade*`** that use enum/string labels (e.g., `skillLevel: 'beginner'`) -- exclude from Check 44 (only check numeric fields)
- **Search filter date fields** used for querying ranges, not data entry -- exclude from Check 39

---

## Notes

### General

- For module-scoped audits, only files within the module's backend directory and frontend files that import from that module's services are scanned.
- ORM column name options (e.g., TypeORM `name:`) are the DB column name. Always compare language property names for Check 4.
- For multi-app projects, audit each frontend app separately and note which app each issue belongs to.

### Audit-Specific

- **Audit mode** (default): produces a READ-ONLY diagnostic report. Does NOT modify any files.
- Use `/fix-gaps` or manual fixes to resolve found issues.
- Conditional validation fields (`@ValidateIf()`) are excluded from Checks 1/3.
- Update DTOs using `PartialType(CreateDto)` inherit all fields as optional -- skip them for Checks 1, 13, 15.
- Check 19 (XSS) traces data flow, which requires reading both form submission and rendering code. If data flow is unclear, flag as "needs manual review" rather than false-positive.
- Check 45 (validation mode): React Hook Form defaults to `mode: 'onSubmit'` when no `mode` is specified. This is the most common cause of "validation rules exist but user sees no error feedback" issues. Recommended fix: `mode: 'onTouched'` (validates on first blur, then real-time on every change — gives debounce-like UX without custom code). Exclude simple 2-field login forms from this check.
- Check 46 (unbounded string): Every user-input string field should have a maxLength at minimum in the DTO layer. Even if the DB column is `text` (unlimited), the API should bound it to prevent abuse. When flagging, suggest a default maxLength from the table below.
- Check 46 default maxLength recommendations:

| Field Pattern | Suggested maxLength | Reasoning |
|--------------|-------------------|-----------|
| `*name*`, `*Name`, `fullName` | 50 | Personal names |
| `*username*` | 20-30 | Login identifiers |
| `*title*` | 255 | Short descriptive text |
| `*description*`, `*content*`, `*bio*` | 2000 | Long descriptive text |
| `*notes*`, `*memo*`, `*comment*`, `*feedback*`, `*reason*` | 500 | Medium free-text |
| `*summary*` | 500 | Medium summary text |
| `*message*` | 5000 | Chat/notification messages |
| `*phone*`, `*tel*`, `*mobile*` | 20 | Phone numbers |
| `*url*`, `*Url*`, `*link*` | 500 | URLs |
| `*address*` | 200 | Street addresses |
| `*email*` | 254 | RFC 5321 max email length |
| `*password*` | 128 | Pre-hash password (no need for more) |
| `*code*`, `*sku*` | 100 | Identifier codes |
| Other string fields | 255 | Safe varchar default |
- Check 47 (HTML maxLength): Adding `maxLength` to HTML inputs is a UX best practice for short string fields (name, username, title) — it prevents the user from typing beyond the limit entirely, rather than showing an error after the fact.
- Check 48 (real-time filtering): HTML `pattern` and `inputMode` attributes do NOT prevent typing invalid characters — they only hint or validate on submit. For fields that must only contain specific characters (digits-only, alphanumeric-only), an `onChange` handler with `.replace()` is required to strip disallowed characters in real-time. This is the most common cause of "pattern exists but user can still type letters" issues.

### Clarifying Questions (MANDATORY)

**After printing the audit report, you MUST append a "Clarifying Questions" section at the end.**

This section asks the user about ambiguous findings that have multiple valid fix strategies. Do NOT silently assume defaults — ask first, then fix.

**When to generate questions:**

Analyze each finding and generate a question when ANY of these conditions are true:

1. **Format/pattern choice**: A field needs validation but the correct format depends on business rules (e.g., phone number format: E.164 vs local, postal code locale)
2. **Constraint value choice**: A field needs a max/min but the correct value is not obvious from context (e.g., maxLength for `description` — 500? 1000? 2000?)
3. **Cross-layer inconsistency with two valid fixes**: The fix could go either direction (e.g., entity says nullable but DTO says required — should we change entity or DTO?)
4. **Validation mode preference**: Multiple valid options exist (`onBlur` vs `onChange` vs `onTouched`)
5. **Scope decision**: Whether to include related work (e.g., "Should we also i18n-wrap these hardcoded messages, or handle separately?")
6. **Migration risk**: A fix requires a database migration that could affect existing data

**Question format:**

```markdown
---

## Clarifying Questions

> The following items need your input before fixes can be applied.
> Straightforward fixes (e.g., password min mismatch, missing .trim(), showPassword default) will proceed without asking.

**Q1. [Short topic]**
- Finding: [Brief description of the issue]
- Options:
  - A) [Option A description]
  - B) [Option B description]
- Default recommendation: [A or B with brief reasoning]

**Q2. [Short topic]**
...
```

**Question generation rules:**
- Number questions sequentially (Q1, Q2, ...)
- Keep each question concise — max 3-4 lines
- Always provide concrete options (A/B or A/B/C), not open-ended questions
- Always state a default recommendation so user can just say "go with defaults"
- Group related questions (e.g., all maxLength values in one question with a table)
- Do NOT ask about findings where the fix is obvious and unambiguous:
  - Password min(6) vs min(8) → obviously fix to 8, no need to ask
  - Missing `.trim()` → obviously add it
  - `showPassword: true` → obviously should be `false`
  - Missing error display → obviously add it
- DO ask about findings where business context matters:
  - Phone format (E.164 vs local vs any)
  - maxLength values for unbounded text fields
  - Whether to create a migration for entity column changes
  - Validation mode preference (onBlur vs onChange)
  - Whether to tackle i18n in this fix or defer

**After the user answers**, proceed with fixes incorporating their decisions. If the user says "go with defaults" or "기본값으로", use the recommended defaults for all questions.

