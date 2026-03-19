---
name: qa-skill-review
description: "Validate QA skill files for framework-agnosticism, English-only content, required sections, and CLAUDE.md compliance"
user-invocable: true
argument-hint: "[skill-path or skill-name]"
---

# QA Skill Review

Validate a QA skill file against the framework-agnostic requirements defined in `CLAUDE.md`. Detects hardcoded frameworks, missing detection tables, non-English content, and structural gaps.

---

## Purpose

Run this skill **before committing any QA skill modification** to catch framework-agnosticism regressions. It enforces the rules from `claude-operations/CLAUDE.md` automatically.

**Important:** This skill is **read-only**. It analyzes skill source files and generates a compliance report -- it does NOT modify any files.

---

## Usage

```
/qa-skill-review qa-states
/qa-skill-review skills/qa/qa-inputs/skill.md
/qa-skill-review --all                          # Review all qa-* skills
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<skill-name or path>` | Yes (unless `--all`) | Skill name (e.g., `qa-states`) or full path to `skill.md` |
| `--all` | No | Review all `skills/qa/qa-*/skill.md` files |
| `--fix` | No | Output specific fix suggestions with line numbers |

---

## 8 Checks

| # | Check | Severity | What it catches |
|---|-------|----------|-----------------|
| 1 | Framework detection table present | **Critical** | Skill has no table mapping signals to frameworks |
| 2 | Multi-framework coverage | **Critical** | Only one framework mentioned (e.g., React-only patterns) |
| 3 | Fallback logic for unknown frameworks | **High** | No "fallback" or "generic" or "unknown" pattern when detection fails |
| 4 | No hardcoded file paths | **High** | Literal paths like `src/modules/`, `src/entities/`, `src/pages/` |
| 5 | English-only content | **Critical** | Non-ASCII text that is not code (Korean, Chinese, Japanese, etc.) |
| 6 | Required structural sections | **Medium** | Missing: Purpose, Usage, Prerequisites, Execution Steps, Output Format, Error Handling |
| 7 | Framework-specific patterns have equivalents | **High** | React pattern listed without Vue/Angular/Svelte equivalent in the same check |
| 8 | No single-framework grep patterns | **Medium** | Grep instruction searches for only one framework's syntax (e.g., only `useQuery` without `useFetch`/`HttpClient`) |

---

**Shared conventions:** See `qa-shared/reference.md` for the shared conventions that all QA skills must follow (scoring, framework detection, self-verification, output formatting). This skill validates compliance with those conventions.

## Execution Steps

### Step 0: Resolve Target Files

If `--all` flag: glob `skills/qa/qa-*/skill.md` to get all skill files.
If skill name given (e.g., `qa-states`): resolve to `skills/qa/qa-states/skill.md`.
If full path given: use directly.

Read each target file fully before running checks.

---

### Step 1: Framework Detection Table (CRITICAL)

**What to look for:**
A markdown table or structured list that maps **detection signals** (package.json entries, file patterns, imports) to **framework names**.

**Pass criteria:**
- At least one table with columns like `Signal | Framework` or `Framework | Detection` or equivalent
- Table covers **3+ distinct frameworks** (not just variations of one, e.g., "React + TanStack" and "React + SWR" count as 1)

**Detection method:**
1. Search for markdown table syntax (`|...|...|`) near keywords: `detect`, `signal`, `framework`, `package.json`, `auto-detect`
2. Count distinct framework names mentioned in detection context

**Fail examples:**
- No detection table at all
- Table only lists React variants (TanStack Query, SWR, RTK Query) without Vue/Angular/Svelte

**Pass examples:**
- Table with React, Vue, Angular, Svelte, Plain HTML rows
- Separate backend table (NestJS, Express, Django, Spring Boot, Laravel)

---

### Step 2: Multi-Framework Coverage (CRITICAL)

**What to look for:**
Throughout the skill's execution steps and check descriptions, verify that framework-specific patterns are not limited to a single framework.

**Detection method:**
1. Count occurrences of framework-specific terms in **instruction/check sections** (not in detection tables):
   - React group: `useQuery`, `useEffect`, `useState`, `useSWR`, `JSX`, `tsx`, `React.lazy`, `Suspense` (React-specific), `ErrorBoundary` (React-specific)
   - Vue group: `useFetch`, `onMounted`, `ref()`, `watch()`, `<template>`, `.vue`, `Nuxt`
   - Angular group: `HttpClient`, `subscribe`, `Observable`, `AsyncPipe`, `@Component`, `ngOnInit`
   - Svelte group: `{#await}`, `onMount`, `.svelte`, `$:`
   - Generic group: `fetch()`, `axios`, `loading`, `error`, `empty state`

2. Calculate framework distribution:
   - If one group has >70% of all framework-specific mentions AND other groups have <5% each → **FAIL**
   - If at least 2 non-React groups have meaningful mentions → **PASS**

**Note:** A skill focused on a specific concern (e.g., accessibility) may naturally have fewer framework-specific patterns. In that case, verify that the patterns it DOES have are not single-framework.

---

### Step 3: Fallback Logic (HIGH)

**What to look for:**
Explicit handling for when framework detection fails or returns unknown.

**Detection method:**
Search for any of these patterns:
- "fallback" / "fall back" / "generic" / "unknown" / "not detected" / "default"
- "if detection fails" / "if no framework" / "otherwise"
- A catch-all row in detection table (e.g., `Other | generic patterns`)

**Pass criteria:**
- At least ONE explicit mention of fallback/generic behavior in execution steps
- OR a default/catch-all case in pattern tables

---

### Step 4: No Hardcoded Paths (HIGH)

**What to look for:**
Literal directory paths that assume a specific project structure.

**Detection method:**
Search for these patterns (case-insensitive) OUTSIDE of code examples and detection tables:

**Hardcoded path patterns (FAIL if found in instructions):**
- `src/modules/`
- `src/entities/`
- `src/pages/`
- `src/components/`
- `src/services/`
- `app/api/`
- `backend/src/`

**Acceptable patterns (NOT a violation):**
- Inside detection table as one of multiple options: `{backend-dir}/**/entities/*.entity.ts`
- Template variables: `{backend-dir}`, `{frontend-dir}`
- Glob patterns with framework branching: "If NestJS: `**/*.controller.ts`, If Express: `routes/*.js`"
- In example output blocks (clearly labeled as examples)

---

### Step 5: English-Only Content (CRITICAL)

**What to look for:**
Non-English text in the skill file. Code identifiers and framework names are exempt.

**Detection method:**
1. Scan all text content for non-ASCII characters outside of:
   - Code blocks (``` fenced blocks)
   - Inline code (`backtick` content)
   - URLs and file paths
   - Framework/library names (e.g., `견적서` is NOT a framework name)
2. Flag any Korean (U+AC00-U+D7AF), Chinese (U+4E00-U+9FFF), Japanese (U+3040-U+30FF) characters found in prose

**Exempt:**
- Transliteration examples explicitly about i18n testing (e.g., "CJK input like 한글")

---

### Step 6: Required Structural Sections (MEDIUM)

**What to look for:**
Every QA skill should have these sections (by heading or equivalent):

| Required Section | Detection Pattern |
|-----------------|-------------------|
| Purpose / Description | `## Purpose` or `## Description` or description in frontmatter |
| Usage | `## Usage` with argument table |
| Execution Steps | `## Execution Steps` or `## Steps` or numbered `### Step N:` |
| Output Format | `## Output Format` or `## Report` or example output block |
| Error Handling | `## Error Handling` or `## Error` section with table |
| Checklist per step | `[ ]` checkbox items within or after each step |

**Pass criteria:**
- At least 4 of 6 sections present
- Execution Steps is mandatory (FAIL if missing)

---

### Step 7: Framework-Specific Patterns Have Equivalents (HIGH)

**What to look for:**
When a skill lists a framework-specific pattern in a check/step, it should list equivalents for other major frameworks.

**Detection method:**
For each execution step that contains framework-specific code patterns:
1. Identify the pattern (e.g., `useEffect` cleanup check)
2. Check if the same step also mentions Vue, Angular, or Svelte equivalents
3. A "Framework" column in the table satisfies this (like `| Pattern | Framework | Detection |`)

**Acceptable patterns:**
- Combined table with Framework column covering 3+ frameworks
- Separate sub-sections per framework (e.g., `**React:**`, `**Vue:**`, `**Angular:**`)
- Generic description that applies to all frameworks (e.g., "Check for loading indicator component")

**Fail examples:**
- Step says "Check `useEffect` cleanup" with no mention of `onMounted`/`ngOnDestroy`/`onMount` equivalents
- Grep pattern is `"useQuery|useSWR"` with no `"useFetch|HttpClient|{#await}"` equivalent

---

### Step 8: No Single-Framework Grep Patterns (MEDIUM)

**What to look for:**
Grep/search instructions that only cover one framework's syntax.

**Detection method:**
Find all lines containing `Grep:` or `grep` or `Search:` or `Glob:` instructions.
For each:
1. Extract the search pattern
2. Check if all terms belong to a single framework group
3. If yes AND the step is not explicitly gated by "If [framework] detected" → **FLAG**

**Pass examples:**
- `Grep: "useQuery|useSWR|useFetch|HttpClient"` → covers React + Vue + Angular
- `If React detected: Grep: "useQuery|useSWR"` → gated by detection, OK
- `Grep: "loading|spinner|skeleton"` → generic, OK

**Fail examples:**
- `Grep: "useQuery|useSWR"` (ungated, React-only)
- `Grep: "onKeyPress|onKeyDown"` (ungated, React-only — should include `@keypress|@keydown|(keypress)|(keydown)|on:keypress`)

---

## Output Format

```
+====================================================================+
|              QA Skill Review Report                                 |
+====================================================================+
|  Skill:        [skill-name]                                        |
|  File:         [file-path]                                         |
|  Review Date:  [YYYY-MM-DD HH:MM]                                  |
|  Status:       [PASS | NEEDS FIX | FAIL]                           |
+====================================================================+
|                                                                    |
|  CHECK RESULTS                                                     |
|                                                                    |
|  | # | Check                        | Result | Details           | |
|  |---|------------------------------|--------|-------------------| |
|  | 1 | Framework detection table    | PASS   |                   | |
|  | 2 | Multi-framework coverage     | PASS   |                   | |
|  | 3 | Fallback logic               | PASS   |                   | |
|  | 4 | No hardcoded paths           | PASS   |                   | |
|  | 5 | English-only content         | FAIL   | Line 391: Korean  | |
|  | 6 | Required sections            | PASS   | 6/6               | |
|  | 7 | Pattern equivalents          | WARN   | Step 6.2 React-only | |
|  | 8 | Grep pattern coverage        | PASS   |                   | |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  ISSUES                                                            |
|                                                                    |
|  [CRITICAL] CHECK-5: Non-English content at line 391               |
|    Found: "서버 상태 확인" in loading timeout description           |
|    Fix: Replace with English equivalent                            |
|                                                                    |
|  [HIGH] CHECK-7: Step 6.2 has React-only patterns                  |
|    Found: useEffect, Suspense patterns without Vue/Angular equiv   |
|    Fix: Add Vue watch loop, Angular subscribe leak, Svelte $: loop |
|                                                                    |
+--------------------------------------------------------------------+
|                                                                    |
|  SUMMARY                                                           |
|  Total checks: 8                                                   |
|  Passed: 6  |  Warning: 1  |  Failed: 1                           |
|  Status: NEEDS FIX                                                 |
|                                                                    |
+====================================================================+
```

**When `--all` is used, output a summary table after individual reports:**

```
## All Skills Review Summary

| Skill | Status | Critical | High | Medium | Details |
|-------|--------|----------|------|--------|---------|
| qa-a11y | PASS | 0 | 0 | 0 | |
| qa-states | NEEDS FIX | 1 | 1 | 0 | Check 5, 7 |
| qa-inputs | PASS | 0 | 0 | 0 | |
...
| **Total** | | **1** | **1** | **0** | |
```

---

## Score Calculation

```
Each check has a weight based on severity:
  CRITICAL (checks 1, 2, 5): 20 points each (60 total)
  HIGH (checks 3, 4, 7):     10 points each (30 total)
  MEDIUM (checks 6, 8):       5 points each (10 total)

Score = sum of passed check weights
Max score = 100

Status:
  PASS:      >= 90 (no CRITICAL failures)
  NEEDS FIX: >= 60 OR has any CRITICAL failure
  FAIL:      < 60
```

---

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| File not found | Invalid skill name or path | Check skill name matches `skills/qa/qa-*/skill.md` pattern |
| No frontmatter | Missing `---` YAML block | Add frontmatter with name, description, user-invocable fields |
| Empty file | Skill file has no content | Skill needs to be written first |
| Not a QA skill | File is not under `skills/qa/` | This review is designed for QA skills only |

---

## Tips

1. **Run before every commit** — Framework-agnosticism is easy to break when adding a new check or pattern
2. **Check 7 is the most common failure** — When adding a React pattern, always ask "what's the Vue/Angular/Svelte equivalent?"
3. **Grep patterns are sneaky** — A search for `useQuery` looks correct but misses Vue `useFetch` and Angular `HttpClient`
4. **English-only includes comments** — Even inline notes like `// 여기 수정` count as violations
5. **Use `--fix` for actionable output** — Shows exactly which lines need changes
