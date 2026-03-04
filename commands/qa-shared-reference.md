---
description: Shared reference for all QA commands — common rules, framework detection, scoring, and conventions
---

# QA Shared Reference

This document defines shared rules, detection patterns, scoring mechanisms, and output conventions used by all `/qa-*` commands. Individual QA commands reference this document to avoid duplication and ensure consistency.

---

## Execution Rule: Self-Verification & Auto-Retry

**Every Step in every QA command MUST follow this rule:**

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

## File Validation (Common)

All QA commands validate the input file identically:

1. File path is provided
2. File exists and is readable
3. File extension is supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-<command> <page-or-component-file-path> [options]
```

---

## Frontend Framework Detection (Common)

All QA commands use the same framework detection logic. Read the target file and project's `package.json`:

| Signal | Framework |
|--------|-----------|
| `import ... from 'react'` or JSX/TSX + `react` in package.json | React |
| `<template>` + `<script>` + `.vue` extension | Vue |
| `@Component` decorator + `@angular/core` in package.json | Angular |
| `.svelte` extension | Svelte |
| `<form>` + no framework imports + `.html` extension | Plain HTML |

---

## Backend Framework Detection (Common)

Commands with `--backend` or `--depth full` support use this detection:

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

All QA commands check for an optional `.qa-config.json` file at the project root:

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
    "rules": ["MOD-05"]
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

All QA commands use this scoring system:

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
  - Bonus items are command-specific (see individual commands)

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

All QA commands handle these common errors:

| Error | Cause | Solution |
|-------|-------|----------|
| File not found | Invalid path provided | Check file path and try again |
| Unsupported file type | Wrong extension | Supported: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js` |
| Framework not detected | Unusual import structure or no package.json | Falls back to generic pattern scanning |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| Circular imports | Complex import chain | Breaks after 10 levels; notes as "unresolvable" |
| package.json not found | No package manager | Falls back to import-based detection only |

Command-specific errors are listed in each command's Error Handling section.

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

Feature suffixes by command:
| Command | Feature Suffix |
|---------|---------------|
| `/qa-back-navigation` | `BackNav` |
| `/qa-input-fields` | `InputFields` |
| `/qa-loading-error-empty` | `LoadingErrorEmpty` |
| `/qa-modal-drawer` | `ModalDrawer` |
| `/qa-permission-role` | `PermissionRole` |
| `/qa-table-list` | `TableList` |
| `/qa-accessibility` | `Accessibility` |
| `/qa-performance` | `Performance` |
| `/qa-page` | `FullPage` |

### Report Header Template

All reports use this header format:

```
╔════════════════════════════════════════════════════════════════════╗
║                   QA [Feature] Report                              ║
╠════════════════════════════════════════════════════════════════════╣
║  Component:     [ComponentName]                                    ║
║  File:          [file-path]                                        ║
║  Framework:     [React + library / Vue + library / etc.]           ║
║  Backend:       [NestJS / Express / N/A]  (if applicable)          ║
║  Report Date:   [YYYY-MM-DD HH:MM]                                ║
║  Depth:         [full / shallow]                                   ║
║  Overall Score: [N]/100                                            ║
║  Status:        [PASS | NEEDS ATTENTION | FAIL]                    ║
╠════════════════════════════════════════════════════════════════════╣
```

---

## Cross-Command Dependency Matrix

When analyzing a page, these commands are commonly run together:

| If you find... | Also run... | Reason |
|---------------|-------------|--------|
| Forms inside modals | `/qa-input-fields` on modal component | Form validation coverage |
| Modals that push history | `/qa-back-navigation` | Back button may close modal or navigate |
| Tables with row actions | `/qa-permission-role` | Row actions may be role-gated |
| Tables with data fetching | `/qa-loading-error-empty` | Tables need loading/error/empty states |
| Permission-gated delete | `/qa-modal-drawer` | Delete confirmation modal quality |
| Navigation after form submit | `/qa-back-navigation` | Post-submit redirect correctness |

Use `/qa-page` to automatically run all relevant commands for a page.

---

## Supported Extensions

All QA commands support: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`

---

## Read-Only Guarantee

All QA commands are **read-only**. They analyze source code and generate reports — they do NOT modify any files.
