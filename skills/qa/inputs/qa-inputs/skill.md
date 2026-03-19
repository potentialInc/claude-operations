---
name: qa-inputs
description: Audit all input fields across the full stack (Entity -> DTO -> Zod -> Form UI) for consistency issues
user-invocable: true
argument-hint: "[module] [--check N] [--group name]"
---

# QA Inputs — Full-Stack Input Field Consistency

Audit all input fields across the full stack (Entity -> DTO -> Zod Schema -> Form UI) for consistency issues including missing asterisks, validation gaps, field name mismatches, type mismatches, and security concerns.

## Execution Mode

- **Standalone** (`/qa-inputs [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check inputs`): qa-fix uses the checks below as its checklist for the inputs layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Check Filtering

```
/qa-inputs --check 1          # Run only Check 1
/qa-inputs --check 1,6,10     # Run specific checks
/qa-inputs --group type        # Run only type-related checks (10-12)
/qa-inputs --group security    # Run only security checks (19-20)
```

| Group | Checks | Description |
|-------|--------|-------------|
| required | 1, 3, 5 | Required/Optional mismatches |
| type | 10, 11, 12 | Type mismatches |
| constraint | 6, 7, 8, 16, 17 | Constraint mismatches |
| integrity | 4, 13, 14, 15 | Cross-layer integrity |
| quality | 2, 9, 18 | Code quality |
| security | 19, 20 | Security concerns |
| semantic | 21-26, 38-44, 49 | Semantic validation |
| ux | 27-31 | UX quality |
| form | 32-37, 45-49 | Form behavior |

## Checks

> Check numbering is by category (not sequential) so `--check N` references remain stable as new checks are added.

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
| 13 | Nullable / Required conflict | **High** | Entity `nullable: false` but DTO `@IsOptional()`, or the reverse |
| 14 | Orphan frontend field | **Medium** | Zod schema has field not in any DTO — silently ignored by server |
| 15 | Orphan DTO field | **Medium** | Create DTO has field not sent by any frontend form — always undefined |

### Quality (Checks 2, 9, 18)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 2 | Regex without error message | **Low** | `@Matches()` or `.regex()` has no user-friendly error message |
| 9 | i18n hardcoded | **Low** | Labels/errors/placeholders without `t()` wrapper |
| 18 | Unique constraint without hint | **Low** | Entity `@Unique()` but no frontend duplicate check — server 500 |

### Security (Checks 19, 20)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 19 | XSS-prone input | **Critical** | User text rendered via `dangerouslySetInnerHTML` / `v-html` without sanitization |
| 20 | File upload without constraint | **Medium** | File input without `accept` type filter or size limit |

### Semantic Validation (Checks 21-26, 38-44, 49)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 21 | Name field too short | **Medium** | `*name*` field allows minLength < 2 (1-char name is unrealistic) |
| 22 | Email field without validation | **High** | `*email*` field missing `@IsEmail()` / `z.string().email()` / `type="email"` |
| 23 | Password field too weak | **High** | `*password*` field has minLength < 6 or missing `type="password"` |
| 24 | URL field without validation | **Medium** | `*url*`/`*link*` field missing `@IsUrl()` / `.url()` validation |
| 25 | Count/quantity allows negative | **Medium** | `*count*`/`*quantity*`/`*sets*`/`*reps*` field allows negative or decimal |
| 26 | Price/amount allows negative | **Medium** | `*price*`/`*amount*`/`*cost*` field allows negative values |
| 38 | Numeric-string without format constraint | **High** | `varchar` field named `*number*`/`*code*` stored as string but has no pattern to restrict format |
| 39 | Date field missing temporal constraint | **Medium** | `*birthday*`/`*startDate*` missing past-only or future-only validation |
| 40 | Phone field without format validation | **High** | `*phone*`/`*tel*`/`*mobile*` missing phone number format pattern |
| 41 | Zip/postal code without format | **Medium** | `*zip*`/`*postal*` missing format pattern for postal code validation |
| 42 | Percentage/rate out of range | **Medium** | `*rate*`/`*percentage*`/`*percent*` missing 0-100 range constraint |
| 43 | Duration/time unrealistic values | **Medium** | `*minutes*`/`*hours*`/`*duration*` missing positive constraint or reasonable max |
| 44 | Score/rating without range | **High** | `*score*`/`*rating*`/`*level*` missing defined min/max range |
| 49 | Date pair missing cross-field constraint | **High** | Paired date fields (`startDate`/`endDate`) where end date can precede start date — check UI layer AND schema layer |

### UX Quality (Checks 27-31)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 27 | Long text in single-line input | **Medium** | `*description*`/`*content*`/`*bio*`/`*memo*` uses `<input>` instead of `<textarea>` |
| 28 | Whitespace-only accepted | **Medium** | String field missing `.trim()` in schema — `"   "` passes validation |
| 29 | Empty string vs null | **Low** | Optional string field sends `""` but DB expects `null` |
| 30 | Missing autocomplete attribute | **Low** | `email`/`password`/`name`/`tel` field without HTML `autocomplete` attribute |
| 31 | Format hint missing | **Low** | Field requiring specific format (phone, zip, date) has no `placeholder` example |

### Form Behavior (Checks 32-37, 45-49)

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 32 | Error message not displayed | **High** | Field has validation but no error display component in UI |
| 33 | Select default value missing | **High** | Required Select has no default value and placeholder is submittable |
| 34 | Double submission possible | **Medium** | Submit button has no loading/disabled state — duplicate data on double-click |
| 35 | Conditional required mismatch | **Medium** | DTO `@ValidateIf()` condition not mirrored in frontend |
| 36 | Array field min/max missing | **Medium** | DTO `@IsArray()` + `@ArrayMinSize(1)` but frontend allows empty array |
| 37 | Default value not shown | **Low** | Entity column has `default` value but frontend form doesn't pre-fill it |
| 45 | Validation mode onSubmit only | **High** | `useForm` uses default `mode: 'onSubmit'` — no real-time feedback |
| 46 | String field without maxLength | **High** | No `@MaxLength` in DTO AND no `.max()` in Zod AND no `maxLength` on HTML input — unbounded |
| 47 | Schema maxLength without HTML maxLength | **Medium** | Schema has `.max(N)` but HTML `<input>` missing `maxLength` attribute |
| 48 | Restricted-format field without real-time filtering | **High** | Field has pattern constraint but no onChange handler to block invalid characters |
| 49 | Default value bypasses required input | **High** | Numeric field has default value (0) that passes validation without user interaction |
