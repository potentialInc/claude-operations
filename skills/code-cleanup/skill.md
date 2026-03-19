---
name: code-cleanup
description: Analyze and remove dead code across any project - unused imports, exports, functions, files, npm packages, entity columns, and duplicate code
user-invocable: true
argument-hint: "[--fix] [--category imports|exports|functions|files|packages|columns|duplicates] [module]"
---

# Code Cleanup - Dead Code Detector & Remover

## Purpose

Detect and optionally remove unnecessary code from any project. This is a universal tool
that auto-detects the project's tech stack (language, framework, ORM) and adapts its
scanning patterns accordingly. Unlike QA skills (read-only), this skill can modify files
when run with `--fix`.

## Usage

```
/code-cleanup                           # Scan all categories, report only
/code-cleanup --fix                     # Scan all categories and remove dead code
/code-cleanup --category imports        # Scan only unused imports
/code-cleanup --category packages --fix # Find and remove unused npm packages
/code-cleanup users                     # Scan specific module only
```

### Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `--fix` | off | Actually remove dead code (without this, report only) |
| `--category` | all | Specific category to scan (see categories below) |
| `[module]` | all | Scope to a specific module directory |

### Categories

| ID | Category | Description |
|----|----------|-------------|
| `imports` | Unused Imports | import statements that are never referenced |
| `exports` | Unused Exports | exported symbols that no other file imports |
| `functions` | Unused Functions/Classes | defined but never called/instantiated |
| `files` | Unused Files | files that nothing imports or requires |
| `packages` | Unused Packages | dependencies in package.json/requirements.txt never imported |
| `columns` | Dead Entity Columns | ORM columns with no write path (delegates to qa-dead-code logic) |
| `duplicates` | Duplicate Code | near-identical code blocks across files |

---

## Execution Algorithm

Follow these steps in order. Complete each step fully before proceeding.

---

### Step 0: Project Detection & Configuration

**0a. Detect project language and framework:**

```
Glob: **/package.json, **/requirements.txt, **/pyproject.toml, **/go.mod, **/Cargo.toml, **/pom.xml, **/build.gradle
```

| Signal | Language | Ecosystem |
|--------|----------|-----------|
| `package.json` with `typescript` | TypeScript | Node.js |
| `package.json` without `typescript` | JavaScript | Node.js |
| `requirements.txt` or `pyproject.toml` | Python | pip/poetry |
| `go.mod` | Go | Go modules |
| `Cargo.toml` | Rust | Cargo |
| `pom.xml` or `build.gradle` | Java/Kotlin | Maven/Gradle |
| `composer.json` | PHP | Composer |

**0b. Detect monorepo structure:**

```
Glob: **/package.json (multiple = monorepo or multi-app)
```

If multiple `package.json` files exist (e.g., `backend/package.json`, `frontend/package.json`),
treat each as a separate project root for package analysis.

**0c. Build file inclusion/exclusion lists:**

Always EXCLUDE from scanning:
- `node_modules/`, `vendor/`, `venv/`, `.venv/`, `__pycache__/`
- `dist/`, `build/`, `.next/`, `.nuxt/`, `out/`
- `*.min.js`, `*.min.css`, `*.map`
- `*.test.ts`, `*.spec.ts`, `*.test.tsx`, `*.spec.tsx` (for function/export analysis, NOT for import analysis)
- `*.d.ts` (type declaration files - these are consumed, not authored dead code)
- `.claude*/`, `.git/`

**0d. Parse arguments:**
- If `--fix` is present, set `mode = "fix"`. Otherwise `mode = "scan"`.
- If `--category X` is present, only run that category. Otherwise run all.
- If `[module]` is present, scope all globs to that module path.

**0e. Output scan plan:**

```
━━━ Code Cleanup - Scan Plan ━━━
Project:      {project name from package.json or directory name}
Language:     {detected language}
Framework:    {detected framework}
Mode:         {scan | fix}
Categories:   {all | specific category}
Scope:        {all | module name}
Excluded:     {list of excluded patterns}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 1: Unused Imports (category: `imports`)

Detect import statements where the imported symbol is never used in the file.

**1a. Find all source files:**

```
TypeScript/JS: Glob: **/*.ts, **/*.tsx, **/*.js, **/*.jsx (excluding node_modules, dist, test files)
Python:        Glob: **/*.py (excluding venv, __pycache__)
Go:            Glob: **/*.go
```

**1b. For each file, extract imports:**

| Language | Import Pattern | Extract Symbol |
|----------|---------------|----------------|
| TypeScript/JS | `import { A, B } from 'module'` | A, B |
| TypeScript/JS | `import A from 'module'` | A |
| TypeScript/JS | `import * as A from 'module'` | A |
| TypeScript/JS | `import type { A } from 'module'` | A (type-only) |
| Python | `from module import A, B` | A, B |
| Python | `import module` | module |
| Go | `import "package"` | last segment of package path |

**1c. For each imported symbol, check usage:**

Search the REST of the file (excluding the import line itself) for usage of the symbol:
- Direct reference: `A.method()`, `new A()`, `A as prop`, `<A />` (JSX)
- Type usage: `A[]`, `A | B`, `: A`, `<A>` (generics)
- Re-export: `export { A }` or `export { A as B }`
- Decorator usage: `@A()`, `@A`

**IMPORTANT - False positive prevention:**
- Comments and strings do NOT count as usage
- `import type` symbols used only in type positions ARE valid usage
- Side-effect imports (`import './styles.css'`, `import 'reflect-metadata'`) are NEVER unused
- Barrel file re-exports (`export { A } from './a'`) - the import IS the usage
- NestJS module imports in `@Module({ imports: [...] })` ARE usage
- Decorator imports (`@Injectable`, `@Column`, etc.) ARE usage

**1d. Collect findings:**

```
{
  filePath: string,
  line: number,
  importedSymbol: string,
  importSource: string,
  isTypeOnly: boolean,
  confidence: "HIGH" | "MEDIUM"  // HIGH = definitely unused, MEDIUM = might be used dynamically
}
```

**1e. Fix action (if --fix):**

For each file with unused imports:
- If ALL symbols from an import are unused, remove the entire import line
- If SOME symbols are unused, remove only the unused symbols from the destructured import
- Preserve formatting (semicolons, quotes style) consistent with the file

---

### Step 2: Unused Exports (category: `exports`)

Detect exported symbols that no other file in the project imports.

**2a. Build export inventory:**

For each source file, find all exports:

| Pattern | Type |
|---------|------|
| `export function name()` | named function export |
| `export class Name` | named class export |
| `export const name =` | named const export |
| `export { A, B }` | re-export |
| `export default` | default export |
| `export type Name =` | type export |
| `export interface Name` | interface export |
| `export enum Name` | enum export |

**2b. Build import graph:**

For each export, search ALL other files for an import of that symbol:

```
Grep: import.*{symbolName}.*from.*['"].*{relativePath}['"]
Grep: require\(['"].*{relativePath}['"]\)
```

Also check for dynamic imports:
```
Grep: import\(['"].*{relativePath}['"]\)
```

**2c. Exclusion rules (NOT dead even if no import found):**

- Entry points: `main.ts`, `index.ts`, `app.ts`, `server.ts`, files in `bin/`
- Route handlers / controllers (may be discovered via decorators, not imports)
- Module files: `*.module.ts` (NestJS modules auto-discovered)
- Config files: `*.config.ts`, `*.config.js`
- Type declaration files: `*.d.ts`
- Test files: `*.test.ts`, `*.spec.ts`
- Migration files: files in `migrations/` directory
- Seeder files: files in `seeders/`, `seeds/` directory
- Files with `@Controller`, `@Injectable`, `@Module` decorators (NestJS auto-wiring)
- Django views/models (URL routing is string-based)
- Files explicitly referenced in configuration (e.g., `ormconfig`, `jest.config`)

**2d. Confidence scoring:**

| Scenario | Confidence |
|----------|-----------|
| Named export, zero imports found, not in exclusion list | HIGH |
| Default export, zero imports found | MEDIUM (dynamic import possible) |
| Export from index/barrel file, barrel itself not imported | HIGH |
| Export used only in test files | INFO (test-only dependency) |

**2e. Fix action (if --fix):**

- Remove the `export` keyword (convert to local symbol) for HIGH confidence items
- If the local symbol is ALSO unused (no reference in the same file), remove the entire declaration
- Ask user confirmation for MEDIUM confidence items before removing

---

### Step 3: Unused Functions & Classes (category: `functions`)

Detect functions and classes defined but never called or instantiated.

**3a. Build function/class inventory:**

For each source file, extract:

| Pattern | Type |
|---------|------|
| `function name()` | function declaration |
| `const name = () =>` | arrow function |
| `const name = function()` | function expression |
| `class Name` | class declaration |
| `methodName() {` inside class | class method (skip - method analysis is step 3c) |

**3b. Search for usage across all files:**

For each function/class name:

```
Grep: {name}\( in all source files (function call)
Grep: new {Name}\( in all source files (class instantiation)
Grep: {name} in all source files (reference as value, callback, etc.)
```

**IMPORTANT - Exclude self-reference:**
- The definition line itself does NOT count
- Recursive calls within the function DO count as "used"

**3c. Class method analysis (secondary):**

For each class, check methods that are never called:
- External call: `instance.method()` or `ClassName.staticMethod()`
- Internal call: `this.method()` within the same class
- Inherited/override methods (check `extends` and `implements`) are ALWAYS considered used
- Methods matching interface contracts are ALWAYS considered used
- Lifecycle methods (`constructor`, `ngOnInit`, `componentDidMount`, `onModuleInit`, etc.) are ALWAYS used
- Decorated methods (`@Get()`, `@Post()`, `@EventHandler()`, etc.) are ALWAYS used

**3d. Exclusion rules:**

- Exported functions (handled in Step 2, not this step)
- Functions assigned to module.exports or default export
- Callback functions passed as arguments
- Event handlers assigned via `addEventListener`, `on()`, etc.
- Functions registered in framework configs (routes, middleware, guards)

**3e. Fix action (if --fix):**

- Remove the entire function/class declaration for HIGH confidence items
- If removal creates unused imports, also remove those imports
- Never remove constructors or framework lifecycle methods

---

### Step 4: Unused Files (category: `files`)

Detect files that no other file imports, requires, or references.

**4a. Build file dependency graph:**

For each source file, extract all import/require targets and resolve to absolute paths:

```typescript
import { A } from './utils'          → resolves to ./utils.ts or ./utils/index.ts
import { B } from '@/services/api'   → resolve path alias from tsconfig.json
require('./config')                  → resolves to ./config.js or ./config.json
```

**4b. Find orphan files:**

Files with ZERO incoming edges in the dependency graph (nothing imports them).

**4c. Exclusion rules (files that are NOT dead even with zero imports):**

| File Type | Reason |
|-----------|--------|
| Entry points (`main.ts`, `index.ts` at root, `app.ts`) | Application bootstrap |
| Config files (`*.config.ts`, `*.config.js`, `.env*`) | Loaded by tools, not imported |
| Migration files (`migrations/**`) | Run by ORM CLI |
| Seeder files (`seeders/**`, `seeds/**`) | Run by ORM CLI |
| Test files (`*.test.ts`, `*.spec.ts`) | Run by test runner |
| Type declarations (`*.d.ts`) | Consumed by TypeScript compiler |
| Scripts (`scripts/**`, `bin/**`) | Run directly |
| Static assets (`*.css`, `*.scss`, `*.svg`, `*.png`) unless CSS modules | Loaded by bundler |
| HTML templates (`*.html`) | Loaded by framework |
| Route/page files matching framework convention | Auto-discovered by framework |
| Module files (`*.module.ts` for NestJS) | Auto-discovered |
| `index.ts` barrel files that re-export | May be the public API |
| Entity files (`*.entity.ts`) | Loaded by ORM |
| Guard/filter/pipe/interceptor files | Loaded by framework decorators |

**4d. Confidence scoring:**

| Scenario | Confidence |
|----------|-----------|
| Source file, zero imports, not in exclusion list, no dynamic reference | HIGH |
| Source file, zero imports, but name appears in string literals | LOW (dynamic reference possible) |
| Test helper file only imported by deleted tests | HIGH |

**4e. Fix action (if --fix):**

- Delete the file for HIGH confidence items
- Stage the deletion in git (do NOT commit automatically)
- If deleting a file causes other files to have broken imports, flag them

---

### Step 5: Unused Packages (category: `packages`)

Detect dependencies listed in package.json (or requirements.txt, etc.) that are never
imported in source code.

**5a. Extract declared dependencies:**

| Ecosystem | File | Section |
|-----------|------|---------|
| Node.js | `package.json` | `dependencies`, `devDependencies` |
| Python | `requirements.txt` | all lines |
| Python | `pyproject.toml` | `[project.dependencies]`, `[project.optional-dependencies]` |
| Go | `go.mod` | `require` block |
| PHP | `composer.json` | `require`, `require-dev` |

**5b. For each package, search for usage:**

| Ecosystem | Search Pattern |
|-----------|---------------|
| Node.js | `from '{package}'`, `require('{package}')`, `import '{package}'` |
| Node.js (scoped) | `from '@scope/pkg'`, also check sub-path: `from '@scope/pkg/sub'` |
| Python | `import {package}`, `from {package} import` |
| Go | `import "{module/path}"` |

**5c. Special handling for Node.js packages:**

Packages that are used WITHOUT explicit import (not dead even if no import found):

| Package | How It's Used |
|---------|--------------|
| `typescript` | Used by `tsc` CLI |
| `eslint`, `prettier` | Used by CLI/config |
| `jest`, `vitest`, `mocha` | Test runner CLI |
| `ts-node`, `tsx`, `ts-jest` | TypeScript execution |
| `husky`, `lint-staged` | Git hooks |
| `@types/*` | TypeScript type definitions |
| `tailwindcss`, `postcss`, `autoprefixer` | PostCSS/build pipeline |
| `vite`, `webpack`, `esbuild` | Bundler (CLI) |
| `nodemon` | Dev runner |
| `dotenv` | May be loaded via `-r dotenv/config` |
| `reflect-metadata` | Side-effect import or CLI flag |
| `class-transformer`, `class-validator` | Used via decorators (implicit) |
| Packages in `scripts` section of package.json | CLI usage |
| `@nestjs/cli`, `@angular/cli` | Framework CLI tools |
| `prisma` | CLI tool (`npx prisma`) |
| Babel plugins/presets (`@babel/*`, `babel-*`) | Babel config |
| ESLint plugins (`eslint-plugin-*`, `@typescript-eslint/*`) | ESLint config |
| PostCSS plugins (`postcss-*`) | PostCSS config |
| Vite plugins (`vite-plugin-*`, `@vitejs/*`) | Vite config |

Also check these config files for package references:
```
Glob: .eslintrc*, eslint.config.*, prettier.config.*, .prettierrc*
Glob: tailwind.config.*, postcss.config.*, vite.config.*, webpack.config.*
Glob: jest.config.*, vitest.config.*, tsconfig.json, babel.config.*
Glob: .babelrc, nest-cli.json, angular.json
```

**5d. Confidence scoring:**

| Scenario | Confidence |
|----------|-----------|
| Package not found in any source file or config file | HIGH |
| Package found only in lockfile but not package.json | N/A (not declared) |
| `devDependency` not found in any config or test file | HIGH |
| Package name is substring of another used package | LOW (verify manually) |

**5e. Fix action (if --fix):**

For Node.js:
```bash
npm uninstall {package}          # for dependencies
npm uninstall -D {package}       # for devDependencies
```

For Python:
- Remove line from `requirements.txt`
- Or remove from `pyproject.toml` dependencies

IMPORTANT: After removing packages, run the project's build/compile command to verify
nothing breaks. If build fails, revert and mark as FALSE POSITIVE.

---

### Step 6: Dead Entity Columns (category: `columns`)

This category reuses the analysis logic from `qa-dead-code`. See that skill for the
detailed algorithm. The key difference:

- In `scan` mode: produce the same report as qa-dead-code
- In `fix` mode: for each CRITICAL dead column (no write path at all):
  1. Remove `@Column()` decorated property from entity file
  2. Remove the property from any Response DTO
  3. Generate a migration to DROP the column:
     ```bash
     npm run migration:generate src/database/migrations/RemoveDeadColumn_{ColumnName}
     ```
  4. If the generated migration is wrong, manually create:
     ```typescript
     await queryRunner.query(`ALTER TABLE "{table}" DROP COLUMN "{column}"`);
     ```

IMPORTANT: Only fix CRITICAL severity (columns with no write path AND no read path).
HIGH/MEDIUM columns require human decision.

---

### Step 7: Duplicate Code Detection (category: `duplicates`)

Detect near-identical code blocks across different files.

**7a. Strategy - Token-based similarity:**

For each source file, extract "meaningful" code blocks:
- Functions/methods with 5+ lines
- Conditional blocks (if/switch) with 5+ lines
- Loop bodies with 5+ lines

**7b. Compare blocks across files:**

Two blocks are "duplicates" if:
- They have 80%+ structural similarity (ignoring variable names, string values)
- They are in DIFFERENT files (same-file duplication is usually intentional)
- They are 10+ lines each (ignore small utility patterns)

**7c. Similarity detection approach:**

1. Normalize code: strip comments, normalize whitespace, replace identifiers with placeholders
2. Compare normalized blocks using line-by-line diff
3. Calculate similarity ratio: `matching_lines / max(lines_A, lines_B)`

**7d. Report format for duplicates:**

```
| # | File A | Lines | File B | Lines | Similarity | Suggestion |
|---|--------|-------|--------|-------|-----------|------------|
| 1 | auth.service.ts | 45-62 | user.service.ts | 23-40 | 92% | Extract to shared util |
```

**7e. Fix action (if --fix):**

Duplicates are NOT auto-fixed. They require human decision about:
- Where to place the shared function
- What to name it
- How to parameterize differences

Instead, output specific refactoring suggestions:
```
Suggestion: Extract to `src/shared/utils/{name}.ts`
Proposed signature: function {name}(param1: Type1, param2: Type2): ReturnType
Differences to parameterize:
  - Line 3: File A uses 'users', File B uses 'posts' → parameter: tableName
```

---

### Step 8: Compile Report

After all categories are scanned, compile a unified report.

**8a. Report structure:**

```markdown
# Code Cleanup Report

**Date**: {current date}
**Project**: {project name}
**Mode**: {scan | fix}
**Categories Scanned**: {list}

---

## Summary

| Category | Items Found | Severity | Fixed |
|----------|------------|----------|-------|
| Unused Imports | {n} | - | {n if fix mode} |
| Unused Exports | {n} | {H/M/L} | {n if fix mode} |
| Unused Functions | {n} | {H/M/L} | {n if fix mode} |
| Unused Files | {n} | {H/M/L} | {n if fix mode} |
| Unused Packages | {n} | {H/M/L} | {n if fix mode} |
| Dead Columns | {n} | {C/H/M} | {n if fix mode} |
| Duplicate Code | {n} | - | manual |
| **Total** | **{total}** | | **{fixed}** |

---

## Detailed Findings

### 1. Unused Imports ({n} found)

| # | File | Line | Import | Source | Confidence |
|---|------|------|--------|--------|-----------|
{findings}

### 2. Unused Exports ({n} found)

| # | File | Line | Export | Type | Imported By | Confidence |
|---|------|------|--------|------|-------------|-----------|
{findings}

### 3. Unused Functions/Classes ({n} found)

| # | File | Line | Name | Type | References | Confidence |
|---|------|------|------|------|-----------|-----------|
{findings}

### 4. Unused Files ({n} found)

| # | File | Imported By | Last Modified | Size | Confidence |
|---|------|------------|--------------|------|-----------|
{findings}

### 5. Unused Packages ({n} found)

| # | Package | Version | Type | Referenced In | Confidence |
|---|---------|---------|------|--------------|-----------|
{findings}

### 6. Dead Entity Columns ({n} found)

| # | Entity | Column | Create DTO | Update DTO | Service Write | Severity |
|---|--------|--------|-----------|-----------|--------------|----------|
{findings}

### 7. Duplicate Code ({n} found)

| # | File A | Lines | File B | Lines | Similarity | Suggestion |
|---|--------|-------|--------|-------|-----------|------------|
{findings}

---

## Fix Summary (if --fix mode)

### Successfully Fixed
{list of changes made}

### Skipped (requires human decision)
{list of items not auto-fixed with reason}

### Verification
- [ ] Build passes after changes
- [ ] Tests pass after changes
- [ ] No runtime errors
```

**8b. Save report:**

```
.claude-project/qa/CodeCleanup_Report_{YYMMDD}.md
```

---

### Step 9: Post-Fix Verification (only in --fix mode)

After making changes, verify the project still works:

**9a. Run type checking:**

| Ecosystem | Command |
|-----------|---------|
| TypeScript | `npx tsc --noEmit` or `npm run type-check` |
| Python | `mypy .` (if configured) |
| Go | `go build ./...` |

**9b. Run linting:**

| Ecosystem | Command |
|-----------|---------|
| Node.js | `npm run lint` (if script exists) |
| Python | `ruff check .` or `flake8` |
| Go | `golangci-lint run` |

**9c. If verification fails:**

1. Identify which removal caused the failure
2. Revert that specific change
3. Mark the item as FALSE POSITIVE in the report
4. Re-run verification
5. Repeat until verification passes

**9d. Final output:**

```
━━━ Code Cleanup Complete ━━━
Scanned:  {n} files across {m} categories
Found:    {total} items of dead code
Fixed:    {fixed} items (--fix mode)
Skipped:  {skipped} items (manual review needed)
Reverted: {reverted} items (false positives)
Build:    {PASS | FAIL}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Extensibility

To add new categories in the future:

1. Add category entry to the Categories table in this file
2. Add a new Step (N) following the same pattern:
   - Na: Find relevant files
   - Nb: Extract items
   - Nc: Check for usage
   - Nd: Exclusion rules (false positive prevention)
   - Ne: Fix action
3. Add category to the report template in Step 8
4. Update argument-hint in frontmatter

Planned future categories:
- `css`: Unused CSS classes/selectors
- `env`: Unused environment variables
- `routes`: API endpoints with no frontend caller
- `types`: Unused type/interface definitions
- `hooks`: Custom React hooks defined but never used
- `stores`: Redux/Zustand store slices with no consumer
- `translations`: i18n keys defined but never referenced

---

## Important Caveats

1. **Static analysis only.** Dynamic imports (`import()`, `require(variable)`), reflection,
   and string-based references cannot be fully detected. Items flagged with LOW confidence
   may be dynamically referenced.

2. **Framework magic.** Many frameworks auto-discover files (NestJS modules, Next.js pages,
   Angular components). The exclusion rules cover common patterns but custom conventions
   may cause false positives.

3. **Monorepo cross-references.** In monorepos, one package may import from another via
   package name rather than relative path. The scanner resolves workspace protocol references
   (`workspace:*`) but custom linking may be missed.

4. **CSS/Style analysis is NOT included** in this version. CSS-in-JS (styled-components,
   emotion) and utility classes (Tailwind) make CSS dead code detection a separate problem.

5. **Test files.** Unused test helpers are flagged, but test files themselves are excluded
   from "unused files" because they're discovered by the test runner, not by imports.

6. **Build verification is essential** when using `--fix` mode. Always review changes
   before committing.
