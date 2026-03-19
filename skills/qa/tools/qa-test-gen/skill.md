---
name: qa-test-gen
description: Generate Playwright E2E tests for input field validation across all forms
user-invocable: true
argument-hint: "[module]"
---

# QA Test Gen - Playwright E2E Input Validation Test Generator

## Purpose

Generate Playwright E2E tests that dynamically test every input field by submitting invalid values and verifying proper rejection. Reuses inventory approach (Entity -> DTO -> Zod -> Form UI scanning) to build a test matrix per field, then outputs complete, runnable Playwright spec files.

## Usage

```
/qa-test-gen                    # Generate tests for all forms
/qa-test-gen products           # Generate tests for a specific module only
```

## Execution Flow

```
Stack Detection → Inventory → Cross-Layer Mapping → Test Matrix → Test File Generation
```

---

## Stack Detection

Detect project stack by scanning project files. Adapt scanning patterns accordingly.

**Prerequisites:** Read `qa-shared/reference.md` for the complete framework detection tables, ORM patterns, validation layer patterns, and form library patterns.

### Detection Logic

1. **Read manifest files** — `package.json`, `requirements.txt`, `go.mod`, `pom.xml`, `composer.json`, `build.gradle`
2. **Detect backend framework** using the Backend Framework Detection table in `qa-shared/reference.md`:
   - NestJS, Express, Fastify, Django, Spring Boot, Laravel, FastAPI, etc.
3. **Detect frontend framework** using the Frontend Framework Detection table in `qa-shared/reference.md`:
   - React, Vue, Angular, Svelte, SvelteKit, Nuxt, Next.js, etc.
4. **Detect ORM/data layer** using the ORM / Data Layer Patterns table in `qa-shared/reference.md`:
   - TypeORM, Prisma, Sequelize, Drizzle, Mongoose, Django ORM, SQLAlchemy, Eloquent, etc.
5. **Detect validation layer** using the DTO / Validation Layer Patterns table in `qa-shared/reference.md`:
   - class-validator, Zod, Yup, Joi, Valibot, Django Serializers, Laravel Requests, Spring Validation, etc.
6. **Detect form library** using the Form Library Patterns table in `qa-shared/reference.md`:
   - React Hook Form, Formik, VeeValidate, FormKit, Angular Reactive/Template Forms, etc.
7. **Check for test runner:**
   - Playwright: `@playwright/test` in any `package.json`
   - Cypress: `cypress` in any `package.json`
   - If neither found, note that a test runner needs to be installed (default to Playwright)

### Stack-Specific Scan Patterns

Use the detected framework to select scan patterns:

| Backend + ORM | Entity/Model Scan Pattern | DTO/Validation Scan Pattern |
|--------------|--------------------------|---------------------------|
| NestJS + TypeORM | `{backend}/**/entities/*.entity.ts`, `**/*.entity.ts` | `{backend}/**/dtos/*.dto.ts`, `**/*.dto.ts` |
| NestJS + Prisma | `prisma/schema.prisma` | `{backend}/**/dtos/*.dto.ts` |
| Express + Sequelize | `**/models/*.{js,ts}` | `**/validators/*.{js,ts}` |
| Express + Mongoose | `**/*.schema.ts`, `**/*.model.ts` | `**/validators/*.{js,ts}` |
| Django | `**/models.py` | `**/serializers.py`, `**/forms.py` |
| FastAPI + SQLAlchemy | `**/models.py`, `**/models/*.py` | `**/schemas.py`, `**/schemas/*.py` (Pydantic) |
| Spring Boot + JPA | `**/entity/*.java`, `**/model/*.java` | `**/dto/*.java`, `**/request/*.java` |
| Laravel + Eloquent | `app/Models/*.php` | `app/Http/Requests/*.php` |
| Generic fallback | Grep for table/schema definitions | Grep for validation patterns |

| Frontend | Form Scan Pattern |
|----------|------------------|
| React (Hook Form + Zod) | Grep for `useForm\|zodResolver\|<form` in `**/*.tsx` |
| React (Formik + Yup) | Grep for `useFormik\|<Formik\|yup.object` in `**/*.tsx` |
| Vue (VeeValidate) | Grep for `useForm\|useField\|<Field` in `**/*.vue` |
| Vue (FormKit) | Grep for `<FormKit` in `**/*.vue` |
| Angular (Reactive) | Grep for `FormBuilder\|FormGroup\|formControlName` in `**/*.ts` |
| Svelte | Grep for `bind:value\|on:submit` in `**/*.svelte` |
| Generic fallback | Grep for `<form\|<input\|name=` in all frontend files |

---

## Inventory

Build Entity, DTO, and Frontend Form inventories by scanning the codebase.

### Entity/Model Inventory

Scan entity/model files to extract:
- Column names and database types
- Nullable vs required columns
- Length constraints, precision, scale
- Unique constraints
- Default values
- Enum column values
- Relation definitions

**ORM-specific patterns (use the detected ORM):**

Use the Column/Field Definition column from the **ORM / Data Layer Patterns** table in `qa-shared/reference.md` to identify how columns are defined for the detected ORM. Extract type, nullable, length, and constraint information from each column definition.

### DTO Inventory

Scan DTO/validation files to extract field names, types, and validation rules. Use the detected validation layer:

| Validation Layer | Key Patterns to Extract |
|-----------------|------------------------|
| class-validator (NestJS) | `@IsString()`, `@MinLength(n)`, `@MaxLength(n)`, `@IsOptional()`, `@IsNotEmpty()`, `@IsEmail()`, `@IsEnum()`, `@Matches(regex)`, `@ValidateNested()` |
| Zod | `.string()`, `.min(n)`, `.max(n)`, `.optional()`, `.email()`, `.url()`, `.regex()`, `.array()`, `.enum()` |
| Yup | `.string()`, `.min(n)`, `.max(n)`, `.notRequired()`, `.email()`, `.url()`, `.matches()`, `.array()` |
| Joi | `Joi.string()`, `.min(n)`, `.max(n)`, `.optional()`, `.email()`, `.uri()`, `.pattern()`, `.array()` |
| Valibot | `v.string()`, `v.minLength(n)`, `v.maxLength(n)`, `v.optional()`, `v.email()`, `v.url()` |
| Django Serializers | `CharField(max_length=N, required=True)`, `EmailField()`, `IntegerField(min_value=N)` |
| Laravel Requests | `'field' => 'required\|string\|max:N\|email'` in `rules()` method |
| Spring Validation | `@NotBlank`, `@Size(min=N, max=N)`, `@Email`, `@Min(N)`, `@Max(N)`, `@Pattern(regexp)` |
| Pydantic (FastAPI) | `Field(min_length=N, max_length=N)`, `EmailStr`, `constr(regex)` |

Also extract:
- Nested DTOs and array fields
- Partial/pick/omit types (e.g., `PartialType()`, `.partial()`, `.pick()`)
- API metadata (description, example, required, enum)

### Frontend Form Inventory

Scan frontend form files to extract (adapt to detected form library from `qa-shared/reference.md`):

| Form Library | Schema/Validation | Form Setup | Field Registration |
|-------------|-------------------|------------|-------------------|
| React Hook Form + Zod | `z.object({...})` schema | `useForm({ resolver: zodResolver(schema) })` | `register('field')` or `<Controller>` |
| React Hook Form + Yup | `yup.object({...})` schema | `useForm({ resolver: yupResolver(schema) })` | `register('field')` or `<Controller>` |
| Formik + Yup | `yup.object({...})` schema | `useFormik({ validationSchema })` or `<Formik>` | `<Field name="field">` |
| VeeValidate (Vue) + Zod | `toTypedSchema(z.object({...}))` | `useForm({ validationSchema })` | `useField('field')` or `<Field>` |
| FormKit (Vue) | Built-in validation rules | `<FormKit type="form">` | `<FormKit type="text" name="field">` |
| Angular Reactive Forms | `Validators.required`, `Validators.minLength(n)` | `FormBuilder.group({...})` | `formControlName="field"` |
| Plain HTML | `required`, `minlength`, `maxlength`, `pattern` attributes | `<form>` element | `<input name="field">` |

For each form, also extract:
- Form field components and their props (type, placeholder, label, name)
- Submit handler function name and API endpoint called
- Form location: inline page, modal dialog, wizard step
- Required vs optional visual indicators (asterisks, labels)

---

## Cross-Layer Mapping

Map Entity columns <-> DTO fields <-> Frontend schema fields <-> Form UI elements.

For each field, build a unified record:
```
{
  fieldName: string,
  entityColumn: { type, nullable, length, constraints },
  dtoField: { type, validators[], optional },
  zodField: { type, validators[], transforms },
  formElement: { inputType, selector, label, placeholder }
}
```

Flag mismatches between layers (e.g., DTO says required but entity says nullable). These mismatches should be noted in the summary report but should not block test generation.

---

## Test Matrix Generation

For each field, generate test cases based on field type. Include ALL these matrices:

### String Fields

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Required field empty | `""` | Error shown |
| Required field whitespace only | `"   "` | Error shown |
| Below minLength | `"a"` (if minLength=2) | Error shown |
| Above maxLength | `"a".repeat(maxLength + 1)` | Error shown or truncated |
| Valid string | `"Valid Input"` | No error |

### Name Fields (*name*)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Single character | `"A"` | Error shown |
| Numbers only | `"12345"` | Error shown |
| Valid name | `"John"` | No error |

### Email Fields (*email*)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| No @ symbol | `"notanemail"` | Error shown |
| No domain | `"user@"` | Error shown |
| Double @ | `"user@@domain.com"` | Error shown |
| Valid email | `"user@example.com"` | No error |

### Password Fields (*password*)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Too short | `"123"` (if minLength=6) | Error shown |
| Empty | `""` | Error shown |
| Valid password | `"SecurePass123!"` | No error |
| Input type is password | - | `type="password"` attribute exists |

### URL Fields (*url*)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| No protocol | `"example.com"` | Error shown |
| Invalid format | `"not a url"` | Error shown |
| Valid URL | `"https://example.com"` | No error |

### Number Fields (numeric type)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Text in number field | Type `"abc"` | Input rejects or error shown |
| Negative when min >= 0 | `"-5"` | Error shown |
| Decimal when integer required | `"3.5"` | Error shown |
| Above max | `"999999"` | Error shown |
| Below min | `"-1"` | Error shown |
| Valid number | `"10"` | No error |

### Count/Quantity Fields (*count*, *sets*, *reps*)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Zero | `"0"` | Error shown (if min=1) |
| Negative | `"-1"` | Error shown |
| Decimal | `"2.5"` | Error shown |
| Valid count | `"5"` | No error |

### Price/Amount Fields (*price*, *amount*)

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Negative | `"-10"` | Error shown |
| Too many decimals | `"10.999"` (if scale=2) | Error shown |
| Valid price | `"19.99"` | No error |

### Date Fields

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Invalid format | `"not-a-date"` | Error shown |
| Future date (if restricted) | `"2099-01-01"` | Error shown |
| Valid date | `"2025-06-15"` | No error |

### Boolean Fields

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Required but unchecked | Leave unchecked | Error shown |
| Checked | Check the box | No error |

### Enum/Select Fields

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Empty selection | Select placeholder | Error shown |
| Invalid value (via JS) | Set value to `"INVALID"` | Backend rejects |
| Valid selection | Select valid option | No error |

### File Upload Fields

| Condition | Test Input | Expected |
|-----------|-----------|----------|
| Wrong file type | Upload `.exe` | Rejected |
| No file when required | Submit without file | Error shown |
| Valid file | Upload valid file | No error |

---

## Selector Strategy

Priority order for Playwright selectors:

1. `data-testid="field-name"` -> `page.getByTestId('field-name')`
2. `name="fieldName"` -> `page.locator('[name="fieldName"]')`
3. `id="fieldName"` -> `page.locator('#fieldName')`
4. Label text -> `page.getByLabel('Label Text')`
5. Placeholder -> `page.getByPlaceholder('placeholder text')`
6. Role -> `page.getByRole('textbox', { name: 'Label' })`

### Error Message Selectors

Try these in order until one is found in the form markup:

1. `data-testid="field-name-error"` -> `page.getByTestId('field-name-error')`
2. Sibling `.text-destructive` element near the field
3. `[role="alert"]` within the form item container
4. `<FormMessage>` Shadcn component (renders as `<p class="text-destructive">`)
5. Toast notification for form-level errors (check `[data-sonner-toaster]` or `.Toastify`)

### Submit Button Selectors

1. `button[type="submit"]`
2. `page.getByRole('button', { name: /submit|save|create|update|register/i })`
3. `data-testid="submit-button"` -> `page.getByTestId('submit-button')`

---

## Test File Generation

### File Naming

Format: `{form-name}.input-validation.spec.ts`

Examples:
- `create-product.input-validation.spec.ts`
- `edit-user.input-validation.spec.ts`
- `create-order.input-validation.spec.ts`
- `login.input-validation.spec.ts`

### File Location Detection

1. Check for `playwright.config.ts` and read its `testDir` setting
2. Check for existing `e2e/`, `tests/e2e/`, `__tests__/e2e/` directories
3. Default to `tests/e2e/input-validation/`

---

## Auth Helpers

Generate `auth.setup.ts` if any form requires authentication. Use test credentials from CLAUDE.md or `.env.test`.

```typescript
// tests/e2e/input-validation/auth.setup.ts
// Credentials should be sourced from CLAUDE.md, .env.test, or seed data
import { test as setup, expect } from '@playwright/test';

// Generate setup blocks for each role defined in the project.
// Example for a typical role-based app:

setup('authenticate as admin', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Username').fill('{admin-username-from-config}');
  await page.getByLabel('Password').fill('{admin-password-from-config}');
  await page.getByRole('button', { name: /login|sign in/i }).click();
  await page.waitForURL('**/dashboard**');
  await page.context().storageState({ path: 'playwright/.auth/admin.json' });
});

setup('authenticate as user', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Username').fill('{user-username-from-config}');
  await page.getByLabel('Password').fill('{user-password-from-config}');
  await page.getByRole('button', { name: /login|sign in/i }).click();
  await page.waitForURL('**/dashboard**');
  await page.context().storageState({ path: 'playwright/.auth/user.json' });
});

// Add additional role setups as needed based on roles defined in CLAUDE.md
```

---

## Test Code Template

Full Playwright test template used when generating each spec file:

```typescript
import { test, expect } from '@playwright/test';

// Use authenticated session for protected forms
test.use({ storageState: 'playwright/.auth/{role}.json' });

test.describe('{FormName} - Input Validation', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('{form-page-url}');
    await page.waitForSelector('form');
  });

  // ========================
  // Per-Field Validation
  // ========================

  // --- {fieldName} ({fieldType}) ---

  test('{fieldName}: should reject empty value', async ({ page }) => {
    const field = page.locator('{selector}');
    await field.clear();
    await page.locator('{submitSelector}').click();
    const error = page.locator('{errorSelector}');
    await expect(error).toBeVisible();
  });

  test('{fieldName}: should reject invalid input ({invalidDescription})', async ({ page }) => {
    const field = page.locator('{selector}');
    await field.fill('{invalidValue}');
    await page.locator('{submitSelector}').click();
    const error = page.locator('{errorSelector}');
    await expect(error).toBeVisible();
  });

  test('{fieldName}: should accept valid input', async ({ page }) => {
    // Fill ALL required fields with valid data to isolate this field
    {fillAllRequiredFields}
    await page.locator('{submitSelector}').click();
    const error = page.locator('{errorSelector}');
    await expect(error).not.toBeVisible();
  });

  // ========================
  // Type-Specific Attribute Tests
  // ========================

  test('{fieldName}: input should have type="{expectedType}"', async ({ page }) => {
    const field = page.locator('{selector}');
    await expect(field).toHaveAttribute('type', '{expectedType}');
  });

  test('{fieldName}: should have maxlength attribute', async ({ page }) => {
    const field = page.locator('{selector}');
    await expect(field).toHaveAttribute('maxlength', '{maxLength}');
  });

  // ========================
  // Form-Level Tests
  // ========================

  test('should not submit with all fields empty', async ({ page }) => {
    // Clear any default values
    const inputs = page.locator('form input:not([type="hidden"])');
    const count = await inputs.count();
    for (let i = 0; i < count; i++) {
      await inputs.nth(i).clear();
    }
    await page.locator('{submitSelector}').click();
    const errors = page.locator('.text-destructive, .text-red-500, [role="alert"]');
    await expect(errors.first()).toBeVisible();
  });

  test('should prevent double submission', async ({ page }) => {
    {fillAllRequiredFields}
    // Intercept API call to add delay
    await page.route('{apiEndpoint}', async route => {
      await new Promise(resolve => setTimeout(resolve, 2000));
      await route.continue();
    });
    await page.locator('{submitSelector}').click();
    // Button should be disabled or show loading state
    await expect(page.locator('{submitSelector}')).toBeDisabled();
  });

  test('happy path: should submit successfully with all valid data', async ({ page }) => {
    {fillAllFields}
    await page.locator('{submitSelector}').click();
    // Wait for success indicator (toast, redirect, or success message)
    const success = page.locator('[data-sonner-toaster] [data-type="success"], .toast-success, [role="status"]');
    await expect(success).toBeVisible({ timeout: 5000 });
  });
});
```

---

## Valid Data Fixtures

Rules for generating realistic test data per field type:

| Field Type | Generation Rule | Example |
|------------|----------------|---------|
| String | Realistic value based on field name context | `"Physical Therapy Session"` |
| Number | Value within min/max constraints | `"10"` |
| Email | `test-{timestamp}@example.com` | `"test-1706000000@example.com"` |
| Name | Common test names | `"Jane Doe"` |
| Date | Within valid range, ISO format | `"2025-06-15"` |
| Enum | First valid enum value from schema | `"ACTIVE"` |
| URL | `https://example.com/test-resource` | `"https://example.com/video.mp4"` |
| Password | Passes all validators | `"TestPass123!"` |
| Phone | Valid format for locale | `"010-1234-5678"` (Korean) |
| File | Minimal valid fixture file | `"test-fixtures/sample.jpg"` |
| Boolean | `true` | checked state |
| Integer | Whole number within range | `"5"` |
| Decimal | Number with appropriate precision | `"19.99"` |
| Text/Description | Multi-sentence realistic content | `"This is a test description for the item."` |

### Example Fixture Object

```typescript
const createProductFixture = {
  title: { selector: '[name="title"]', value: 'Example Product', type: 'text' },
  description: { selector: '[name="description"]', value: 'A detailed product description for testing purposes.', type: 'textarea' },
  imageUrl: { selector: '[name="imageUrl"]', value: 'https://example.com/images/product.jpg', type: 'url' },
  price: { selector: '[name="price"]', value: '29.99', type: 'number' },
  quantity: { selector: '[name="quantity"]', value: '100', type: 'number' },
  weight: { selector: '[name="weight"]', value: '1.5', type: 'number' },
  category: { selector: '[name="category"]', value: 'ELECTRONICS', type: 'select' },
};
```

---

## Helper Utilities

Create shared helpers at `tests/e2e/helpers/form-helpers.ts`:

```typescript
import { Page, Locator, expect } from '@playwright/test';

/**
 * Fill a single form field based on its type.
 * Handles text inputs, selects, checkboxes, file uploads, and textareas.
 */
export async function fillField(
  page: Page,
  selector: string,
  value: string,
  type?: string
): Promise<void> {
  const element = page.locator(selector);

  switch (type) {
    case 'select':
      await element.selectOption(value);
      break;
    case 'checkbox':
      if (value === 'true') await element.check();
      else await element.uncheck();
      break;
    case 'file':
      await element.setInputFiles(value);
      break;
    case 'date':
      await element.fill(value);
      break;
    case 'textarea':
      await element.clear();
      await element.fill(value);
      break;
    default:
      await element.clear();
      await element.fill(value);
      break;
  }
}

/**
 * Fill all required fields in a form using a data map.
 * Each entry maps a field name to its selector, value, and optional type.
 */
export async function fillRequiredFields(
  page: Page,
  formData: Record<string, { selector: string; value: string; type?: string }>
): Promise<void> {
  for (const [_field, { selector, value, type }] of Object.entries(formData)) {
    await fillField(page, selector, value, type);
  }
}

/**
 * Check if any form validation errors are visible on the page.
 * Scans common error selectors used by Shadcn/UI, Tailwind, and standard patterns.
 */
export async function hasFormErrors(page: Page): Promise<boolean> {
  const errorSelectors = [
    '[role="alert"]',
    '.text-destructive',
    '.text-red-500',
    '.error-message',
    '[data-testid*="error"]',
    '.field-error',
    '.invalid-feedback',
  ];
  for (const selector of errorSelectors) {
    const visible = await page.locator(selector).first().isVisible().catch(() => false);
    if (visible) return true;
  }
  return false;
}

/**
 * Get the error message text near a specific field.
 * Tries multiple strategies to locate the error element.
 */
export async function getFieldError(
  page: Page,
  fieldName: string
): Promise<string | null> {
  const strategies = [
    page.locator(`[data-testid="${fieldName}-error"]`),
    page.locator(`[name="${fieldName}"] ~ .text-destructive`),
    page.locator(`[name="${fieldName}"]`).locator('..').locator('.text-destructive'),
    page.locator(`#${fieldName}-error`),
    page.locator(`[aria-describedby="${fieldName}-error"]`).locator('..').locator('[role="alert"]'),
  ];

  for (const errorLocator of strategies) {
    const visible = await errorLocator.isVisible().catch(() => false);
    if (visible) {
      return await errorLocator.textContent();
    }
  }
  return null;
}

/**
 * Wait for form submission result - either success or error response.
 * Returns 'success' or 'error' based on what appears on screen.
 */
export async function waitForSubmissionResult(
  page: Page,
  options?: { timeout?: number; successSelector?: string; errorSelector?: string }
): Promise<'success' | 'error' | 'timeout'> {
  const timeout = options?.timeout ?? 5000;
  const successSel = options?.successSelector ?? '[data-sonner-toaster] [data-type="success"], .toast-success';
  const errorSel = options?.errorSelector ?? '[data-sonner-toaster] [data-type="error"], .toast-error, [role="alert"]';

  try {
    const result = await Promise.race([
      page.locator(successSel).first().waitFor({ state: 'visible', timeout }).then(() => 'success' as const),
      page.locator(errorSel).first().waitFor({ state: 'visible', timeout }).then(() => 'error' as const),
    ]);
    return result;
  } catch {
    return 'timeout';
  }
}

/**
 * Clear all visible form fields to reset state.
 */
export async function clearAllFields(page: Page): Promise<void> {
  const inputs = page.locator('form input:not([type="hidden"]):not([type="checkbox"]):not([type="radio"])');
  const count = await inputs.count();
  for (let i = 0; i < count; i++) {
    await inputs.nth(i).clear();
  }

  const textareas = page.locator('form textarea');
  const textareaCount = await textareas.count();
  for (let i = 0; i < textareaCount; i++) {
    await textareas.nth(i).clear();
  }
}
```

---

## Test Generation Rules

### Selector Strategy Rules

- **Prefer semantic selectors**: `getByRole`, `getByLabel`, `getByTestId`
- **Avoid fragile selectors**: `nth-child`, deep CSS nesting, auto-generated class names (e.g., `css-1a2b3c`)
- **Use `name` attribute as primary fallback**: Most forms bind inputs via `name`
- **Note missing test IDs**: Flag fields that lack `data-testid` in the summary report
- **Shadcn/UI components**: Look for the underlying `<input>` inside wrapper `<div>` elements; the `FormControl` wraps the actual input
- **Radix UI selects**: Use `[role="combobox"]` for Radix/Shadcn select triggers, not `<select>`

### Error Detection Strategy

```typescript
// Try multiple error detection approaches in order of specificity
// 1. Direct test ID for the field's error
const fieldError = page.locator(`[data-testid="${fieldName}-error"]`);

// 2. Shadcn FormMessage pattern: sibling .text-destructive within FormItem
const formMessage = page
  .locator(`[name="${fieldName}"]`)
  .locator('..')
  .locator('.text-destructive');

// 3. aria-describedby pattern (accessible forms)
const ariaError = page.locator(`#${fieldName}-form-item-message`);

// 4. Generic fallback: any visible error on page
const anyError = page
  .locator('.text-destructive, .text-red-500, [role="alert"]')
  .first();
```

### Form Navigation

Different form types require different navigation strategies:

- **Direct URL**: React Router routes like `/products/create`, `/users/edit/123`
- **Modal**: Click trigger button, then `await page.waitForSelector('[role="dialog"]')`
- **Tab/Accordion**: Click tab trigger, then wait for panel to be visible
- **Drawer**: Click trigger, `await page.waitForSelector('[role="dialog"]')` (Shadcn uses dialog role for drawers too)

---

## Handling Special Cases

### Modal Forms

```typescript
test.describe('Create Product (Modal) - Input Validation', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/products');
    // Open the create modal
    await page.getByRole('button', { name: /create|add|new/i }).click();
    await page.waitForSelector('[role="dialog"]');
    await page.waitForSelector('[role="dialog"] form');
  });

  test('title: should reject empty value', async ({ page }) => {
    const dialog = page.locator('[role="dialog"]');
    await dialog.locator('[name="title"]').clear();
    await dialog.getByRole('button', { name: /save|create|submit/i }).click();
    await expect(dialog.locator('.text-destructive').first()).toBeVisible();
  });
});
```

### Multi-Step Forms (Wizard)

```typescript
test.describe('User Onboarding (Multi-Step) - Input Validation', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/onboarding');
    await page.waitForSelector('form');
  });

  test.describe('Step 1: Personal Information', () => {
    test('name: should reject empty value', async ({ page }) => {
      await page.locator('[name="name"]').clear();
      await page.getByRole('button', { name: /next|continue/i }).click();
      await expect(page.locator('.text-destructive').first()).toBeVisible();
    });
  });

  test.describe('Step 2: Medical History', () => {
    test.beforeEach(async ({ page }) => {
      // Fill step 1 with valid data first
      await page.locator('[name="name"]').fill('Jane Doe');
      await page.locator('[name="email"]').fill('jane@example.com');
      await page.getByRole('button', { name: /next|continue/i }).click();
      await page.waitForSelector('[data-step="2"]');
    });

    test('condition: should reject empty selection', async ({ page }) => {
      await page.getByRole('button', { name: /next|continue/i }).click();
      await expect(page.locator('.text-destructive').first()).toBeVisible();
    });
  });
});
```

### Edit Forms (Pre-filled)

```typescript
test.describe('Edit User - Input Validation', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to edit page for a known test record
    await page.goto('/users/test-user-id/edit');
    await page.waitForSelector('form');
    // Wait for form to be pre-filled with existing data
    const nameField = page.locator('[name="name"]');
    await expect(nameField).not.toHaveValue('');
  });

  test('name: should reject clearing to empty', async ({ page }) => {
    await page.locator('[name="name"]').clear();
    await page.getByRole('button', { name: /save|update/i }).click();
    await expect(page.locator('.text-destructive').first()).toBeVisible();
  });

  test('email: should reject invalid modification', async ({ page }) => {
    await page.locator('[name="email"]').clear();
    await page.locator('[name="email"]').fill('not-an-email');
    await page.getByRole('button', { name: /save|update/i }).click();
    await expect(page.locator('.text-destructive').first()).toBeVisible();
  });
});
```

### i18n Forms (Korean/English)

```typescript
test.describe('Feedback Form (i18n) - Input Validation', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/feedback');
    await page.waitForSelector('form');
  });

  // Use name attribute selectors instead of label text
  // to avoid language-dependent selectors (Korean vs English)
  test('rating: should reject empty value', async ({ page }) => {
    // DO: use name attribute
    await page.locator('[name="rating"]').clear();
    // DON'T: use label text that may be in a non-English language
    await page.locator('button[type="submit"]').click();
    await expect(page.locator('.text-destructive').first()).toBeVisible();
  });
});
```

### Dynamic/Conditional Forms

```typescript
test.describe('Product Form (Conditional Fields)', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/products/create');
    await page.waitForSelector('form');
  });

  test('weight: should appear when product type is "physical"', async ({ page }) => {
    // Select product type that triggers weight field
    await page.locator('[name="productType"]').selectOption('PHYSICAL');
    // Wait for conditional field to appear
    const weightField = page.locator('[name="weight"]');
    await expect(weightField).toBeVisible();
  });

  test('weight: should reject empty when visible', async ({ page }) => {
    await page.locator('[name="productType"]').selectOption('PHYSICAL');
    await page.waitForSelector('[name="weight"]');
    await page.locator('[name="weight"]').clear();
    await page.locator('button[type="submit"]').click();
    await expect(page.locator('.text-destructive').first()).toBeVisible();
  });
});
```

---

## Output

### Generated Files Structure

```
tests/e2e/input-validation/
├── helpers/
│   └── form-helpers.ts
├── auth.setup.ts
├── create-product.input-validation.spec.ts
├── edit-product.input-validation.spec.ts
├── create-user.input-validation.spec.ts
├── edit-user.input-validation.spec.ts
├── create-order.input-validation.spec.ts
├── login.input-validation.spec.ts
└── ...
```

### E2E Summary Report Format

Print this summary after generating all files:

```markdown
## E2E Test Generation Summary

**Generated**: X test files, Y test cases total

| Form | File | Fields | Test Cases | Auth Required |
|------|------|--------|------------|---------------|
| Create Product | create-product.input-validation.spec.ts | 8 | 24 | Yes (role from CLAUDE.md) |
| Edit User | edit-user.input-validation.spec.ts | 5 | 15 | Yes (role from CLAUDE.md) |
| Create Order | create-order.input-validation.spec.ts | 4 | 12 | Yes (role from CLAUDE.md) |
| Login | login.input-validation.spec.ts | 2 | 6 | No |

### Test Cases Per Field Type

| Field Type | Negative Tests | Positive Tests | Attribute Tests |
|------------|---------------|----------------|-----------------|
| String | 3 | 1 | 1 (maxlength) |
| Email | 3 | 1 | 1 (type=email) |
| Password | 2 | 1 | 1 (type=password) |
| Number | 4 | 1 | 1 (type=number) |
| Select/Enum | 2 | 1 | 0 |
| URL | 2 | 1 | 0 |
| Date | 2 | 1 | 0 |
| Boolean | 1 | 1 | 0 |
| File | 2 | 1 | 0 |

### How to Run

npx playwright test tests/e2e/input-validation/
npx playwright test tests/e2e/input-validation/ --ui
npx playwright test tests/e2e/input-validation/ --headed

### Missing Test IDs

| Form | Field | Current Selector | Suggested TestID |
|------|-------|-----------------|------------------|
| Create Product | title | [name="title"] | data-testid="product-title" |
| Create Product | description | [name="description"] | data-testid="product-description" |
| Edit User | email | [name="email"] | data-testid="user-email" |

### Layer Mismatches Found

| Field | Entity | DTO | Frontend | Issue |
|-------|--------|-----|----------|-------|
| product.title | varchar(100) | @MaxLength(200) | z.string().max(100) | DTO maxLength disagrees with entity |
```

---

## Notes

- Generated tests use `[name="..."]` selectors by default. Add `data-testid` attributes to form fields for more resilient selectors.
- For forms behind auth, ensure test credentials from CLAUDE.md are available and the auth setup runs first.
- Happy-path submission tests may need API mocking or a test database to avoid creating real records.
- For edit forms, a test record ID from seed data is needed. Check the seed files or use a known ID.
- Run with `--headed` first to visually verify selectors work before running in CI.
- This tool GENERATES test files. It does NOT run them.
- If a module argument is provided (e.g., `/qa-test-gen products`), only scan and generate tests for that module's entities, DTOs, and forms.
- When no module argument is given, scan all modules and generate tests for every discovered form.
- For Shadcn/UI forms using `react-hook-form` + `zod`, the Zod schema is the primary source of validation rules. DTO decorators serve as the backend safety net.
- Tests should be deterministic and independent. Each test should not depend on another test's state.
- Use `test.describe.serial()` only for multi-step wizard forms where step order matters.
