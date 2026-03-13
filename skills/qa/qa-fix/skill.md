---
name: qa-fix
description: Run QA diagnosis, auto-fix issues, and verify fixes across all 15 QA skills (inputs, crud, buttons, auth, api-sync, db-integrity, layout, list, screen, test-gen, a11y, back-nav, modal, performance, states)
user-invocable: true
argument-hint: "[module] --check <qa-types> [--lang ko]"
---

# QA Fix - Diagnose, Fix, and Verify

## Purpose

Unified workflow that runs QA diagnosis, automatically fixes discovered issues, and re-runs the same QA checks to verify all fixes are applied correctly. Replaces the manual cycle of `/qa-* → manual fix → /qa-*`.

## Usage

```
/qa-fix exercises --check crud,inputs     # Run crud + inputs QA on exercises, fix, verify
/qa-fix exercises --check crud            # Single QA type
/qa-fix --check inputs,buttons            # All modules, multiple QA types
/qa-fix exercises                         # All QA types on exercises module
/qa-fix                                   # All QA types on all modules (full audit)
/qa-fix --lang ko                         # Full audit with Korean output
/qa-fix exercises --check crud --lang ko  # Korean output for specific checks
```

### Supported QA Types

| Type | Skill | Description |
|------|-------|-------------|
| `inputs` | qa-inputs | Entity → DTO → Zod → Form UI consistency |
| `crud` | qa-crud | CRUD endpoint completeness, error handling, frontend coverage |
| `buttons` | qa-buttons | Button handlers, routes, loading states |
| `auth` | qa-auth | Role guards, route protection, token handling |
| `api-sync` | qa-api-sync | Frontend API calls vs backend endpoints |
| `db-integrity` | qa-db-integrity | Entity vs migrations, FK relations, indexes |
| `layout` | qa-layout | Page layout consistency — title placement, panels, borders, spacing |
| `list` | qa-list | List/table rendering, sorting, pagination, empty states |
| `screen` | qa-screen | Screen-level completeness — missing pages, broken routes, dead links |
| `test-gen` | qa-test-gen | Test coverage gaps — generate missing unit/integration tests |
| `a11y` | qa-a11y | Accessibility — aria labels, keyboard nav, contrast, semantic HTML |
| `back-nav` | qa-back-nav | Back/navigation behavior — history stack, redirects, breadcrumbs |
| `modal` | qa-modal | Modal behavior — scroll lock, focus trap, backdrop close, ESC handling |
| `performance` | qa-performance | Performance — lazy loading, bundle size, unnecessary re-renders |
| `states` | qa-states | UI states — loading, empty, error, skeleton placeholders |

## Execution Algorithm

### Phase 1: DIAGNOSE

Run each specified QA skill and collect all issues.

**1a. Parse arguments:**
- Extract `[module]` — optional module name (e.g., `exercises`, `users`)
- Extract `--check` — comma-separated QA types. If omitted, run ALL types.
- Extract `--lang` — output language code (e.g., `ko`, `ja`, `zh`). If omitted, default to `en`.
- Build the QA run list:
  ```
  QA_TYPES = parsed --check types OR [inputs, crud, buttons, auth, api-sync, db-integrity, layout, list, screen, test-gen, a11y, back-nav, modal, performance, states]
  MODULE = parsed module OR "all"
  LANG = parsed --lang OR "en"
  ```
- **If LANG ≠ "en"**: ALL report output (Phase 1/2/3 reports, issue descriptions, status labels, confirmation prompts, and final summary) MUST be written in the specified language. Table headers, severity labels, and markdown structure remain as-is for readability — only the descriptive text (issue descriptions, change descriptions, reasons, confirmations) is translated.

**1b. Run each QA skill sequentially:**

For each QA type in the run list, execute the corresponding skill's algorithm (from its skill.md). Collect ALL findings into a unified issue list.

```
ISSUES = []
for qa_type in QA_TYPES:
    result = run_qa_skill(qa_type, MODULE)
    ISSUES.append({ type: qa_type, findings: result })
```

**1c. Produce Phase 1 Report:**

Display the consolidated diagnosis to the user:

```markdown
# QA Fix - Phase 1: Diagnosis

**Module**: {module or "All"}
**QA Types**: {comma-separated types}
**Total Issues Found**: {count}

| # | QA Type | Severity | Issue | File(s) | Auto-fixable |
|---|---------|----------|-------|---------|--------------|
| 1 | inputs | HIGH | Required field missing validation | create-item.dto.ts, create-item-modal.tsx | Yes |
| 2 | crud | MEDIUM | Missing status toggle UI | items-page.tsx | Yes |
| 3 | auth | HIGH | No role guard on DELETE endpoint | items.controller.ts | Yes |
| ... | ... | ... | ... | ... | ... |
```

**1d. Classify fixability:**

Each issue is classified as:
- **Auto-fixable**: Can be fixed by editing code (DTO decorators, Zod schemas, form labels, guard decorators, etc.)
- **Manual-required**: Needs architectural decision or new feature (e.g., adding a whole new endpoint, migration design)
- **Skip**: Intentional by design (confirmed in Step 0 of the QA skill)

**1e. qa-page deep analysis (optional):**

If specific pages are identified with 3+ issues concentrated on them, offer to run `qa-page` for deep per-page analysis:

> "Page `{page-name}` has {N} issues across multiple QA types. Run `/qa-page {page-name}` for a deep per-page analysis?"

If the user accepts, run qa-page on those pages and merge additional findings into the ISSUES list before proceeding.

**1f. Ask user for confirmation before proceeding:**

> "Found {X} auto-fixable issues and {Y} manual-required issues. Proceed with auto-fixing {X} issues?"

Wait for user confirmation. If denied, output the full report and stop.

---

### Phase 2: FIX

Apply fixes for all auto-fixable issues, grouped by file to minimize edits.

**2a. Group issues by file:**
```
FILE_GROUPS = group_by_file(ISSUES.where(auto_fixable))
# e.g., {
#   "create-item.dto.ts": [issue1, issue2],
#   "create-item-modal.tsx": [issue3, issue4, issue5],
#   ...
# }
```

**2b. Fix order:**

Apply fixes in this dependency order to avoid cascading issues:

1. **Database layer** — db-integrity findings (entity changes, migrations, indexes)
2. **Backend layer** — crud, api-sync, auth findings (DTOs, services, controllers, guards)
3. **Frontend types/validation** — inputs findings (TypeScript types, Zod schemas)
4. **Frontend UI** — buttons, list, modal, states, a11y, layout, back-nav, performance findings (components, pages, routes)

**2c. For each file group, apply fixes:**

Read the file, apply ALL fixes for that file in one pass, then move to the next file.

For each fix applied, log:
```
FIX_LOG.append({
    issue_id: N,
    file: "path/to/file.ts",
    change: "Added required validation to field",
    status: "APPLIED"
})
```

**2d. Run type checking after backend fixes:**
```bash
npx tsc --noEmit --pretty 2>&1
```

If TypeScript errors are found, fix them before proceeding to frontend.

**2e. Run type checking after frontend fixes:**
```bash
npx tsc --noEmit --pretty 2>&1
```

If TypeScript errors are found, fix them before proceeding to verification.

**2f. Run linting:**
```bash
npm run lint 2>&1
```

Fix any lint errors introduced by the fixes.

**2g. Produce Phase 2 Report:**

```markdown
# QA Fix - Phase 2: Fix Applied

**Issues Fixed**: {count}
**Issues Skipped (manual)**: {count}
**TypeScript**: {pass/fail}
**Lint**: {pass/fail}

| # | Issue | File | Change Applied | Status |
|---|-------|------|----------------|--------|
| 1 | Required field missing validation | create-item.dto.ts | Added required decorator | APPLIED |
| 2 | Missing * on required label | create-item-modal.tsx | Added * to FormLabel | APPLIED |
| 3 | No role guard on DELETE | items.controller.ts | Added role guard decorator | APPLIED |
| ... | ... | ... | ... | ... |
```

---

### Phase 3: VERIFY

Re-run the exact same QA checks from Phase 1 to confirm all issues are resolved.

**3a. Re-run each QA skill:**

For each QA type that was run in Phase 1, re-run the same checks with the same module scope. This includes ALL skill types that were originally checked (inputs, crud, buttons, auth, api-sync, db-integrity, layout, list, screen, test-gen, a11y, back-nav, modal, performance, states — whichever were in the original run list).

```
VERIFY_ISSUES = []
for qa_type in QA_TYPES:
    result = run_qa_skill(qa_type, MODULE)
    VERIFY_ISSUES.append({ type: qa_type, findings: result })
```

**3b. Compare Phase 1 vs Phase 3 results:**

For each issue found in Phase 1:
- If NOT found in Phase 3 → **RESOLVED**
- If still found in Phase 3 → **UNRESOLVED** (fix failed or was skipped)

For each issue found in Phase 3 but NOT in Phase 1:
- **REGRESSION** — the fix introduced a new issue

**3c. Produce Phase 3 Report (Final):**

```markdown
# QA Fix - Phase 3: Verification

**Original Issues**: {Phase 1 count}
**Resolved**: {count}
**Unresolved**: {count}
**Regressions**: {count}

## Resolution Summary

| # | QA Type | Issue | Phase 1 | Phase 3 | Status |
|---|---------|-------|---------|---------|--------|
| 1 | inputs | Required field missing validation | FOUND | CLEAR | RESOLVED |
| 2 | crud | Missing status toggle UI | FOUND | CLEAR | RESOLVED |
| 3 | auth | No role guard on DELETE | FOUND | FOUND | UNRESOLVED |
| 4 | a11y | Missing aria-label on icon button | — | FOUND | REGRESSION |

## Unresolved Issues (require manual fix)

| # | QA Type | Issue | File(s) | Reason |
|---|---------|-------|---------|--------|
| 3 | auth | No role guard on DELETE | items.controller.ts | Needs architectural decision on role |

## Regressions (introduced by fixes)

| # | QA Type | Issue | File(s) | Likely Cause |
|---|---------|-------|---------|--------------|
| 4 | a11y | Missing aria-label on icon button | edit-item-modal.tsx | New button added without aria-label |
```

**3d. If regressions found:**

Automatically attempt to fix regressions (go back to Phase 2 for regression issues only). Maximum 1 retry to avoid infinite loops.

**3e. Final summary:**

```markdown
# QA Fix - Complete

**Result**: {ALL CLEAR | X issues remaining}
**Files Modified**: {count}
**Checks Passed**: {list of QA types that are now clean}
**Checks With Remaining Issues**: {list with counts}
```

---

## Fix Patterns by QA Type

### inputs fixes
| Issue | Fix Pattern |
|-------|-------------|
| Required field missing `*` in label | Add ` *` to `<FormLabel>` or `<Label>` text |
| DTO optional but entity NOT NULL | Remove optional decorator, add required decorator |
| Zod `.optional()` but should be required | Remove `.optional()`, add `.min(1, "Field is required")` |
| maxLength mismatch | Align `.max(N)` in Zod and `maxLength={N}` in HTML to match entity constraint |
| Missing maxLength on input | Add `maxLength={N}` attribute to input element |

### crud fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing soft delete | Replace hard delete with soft delete method |
| Missing NotFoundException | Add entity existence check before return |
| Missing ConflictException | Add try/catch around save with conflict error on unique violation |
| Missing status toggle UI | Add Switch/Toggle component in list table for status column |
| Missing status in UpdateDto | Add optional boolean status field to UpdateDto |

### buttons fixes
| Issue | Fix Pattern |
|-------|-------------|
| Button with no onClick handler | Add appropriate handler function |
| Missing loading state | Add `disabled={isLoading}` and loading spinner |
| Missing confirmation dialog | Wrap destructive action in confirm dialog |

### auth fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing role guard | Add appropriate role guard decorator |
| Missing public route marker | Add public route decorator |
| No current user on user-specific route | Add current user parameter extraction |

### api-sync fixes
| Issue | Fix Pattern |
|-------|-------------|
| Frontend calls wrong path | Update API path to match backend route |
| Missing query params | Add missing params to frontend API call |
| Response shape mismatch | Update frontend type to match actual backend response |

### db-integrity fixes
| Issue | Fix Pattern |
|-------|-------------|
| Entity column not in migration | Generate new migration with the missing column |
| Missing FK constraint | Add proper foreign key relation decorator |
| Missing index | Add index decorator to entity column |

### layout fixes
| Issue | Fix Pattern |
|-------|-------------|
| Inconsistent page title placement | Align title to standard layout position |
| Missing panel borders | Add border styling to match design system |
| Spacing inconsistency | Apply standard spacing tokens |

### list fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing default sort | Add defaultSort to table/list config |
| Missing empty state | Add empty state component when list has no items |
| Missing pagination | Add pagination component with page size config |

### a11y fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing aria-label | Add aria-label to icon-only buttons |
| Missing keyboard navigation | Add tabIndex and onKeyDown handlers |
| Missing semantic HTML | Replace generic div with semantic element (nav, main, section) |

### back-nav fixes
| Issue | Fix Pattern |
|-------|-------------|
| router.push instead of replace on redirect | Change to router.replace |
| Missing breadcrumb | Add breadcrumb component with route hierarchy |
| Back button missing on detail page | Add back navigation button |

### modal fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing scroll lock | Add body overflow hidden on open |
| Missing focus trap | Add focus trap logic to modal container |
| No ESC key handler | Add onKeyDown handler for Escape key |
| Missing backdrop close | Add onClick handler on backdrop overlay |

### performance fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing lazy load on route | Add React.lazy/dynamic import for route component |
| Unnecessary re-renders | Wrap component with memo, extract stable callbacks |
| Large bundle import | Replace full library import with specific module import |

### states fixes
| Issue | Fix Pattern |
|-------|-------------|
| Missing loading state | Add loading check with skeleton placeholder |
| Missing error state | Add error boundary or error message display |
| Missing empty state | Add empty state UI when data array is empty |

---

## Safety Rules

1. **Always read before edit** — Never modify a file without reading it first
2. **One file, one pass** — Apply all fixes for a file in a single edit session to avoid conflicts
3. **Type check after each layer** — Run `tsc --noEmit` after backend fixes and after frontend fixes
4. **No destructive changes** — Never delete endpoints, entities, or columns. Only add or modify.
5. **Preserve existing behavior** — Fixes should not change the behavior of working code
6. **Migration safety** — If entity changes require migration, generate it but do NOT auto-run. Report it for manual review.
7. **Max 1 regression retry** — If Phase 3 finds regressions, fix them once. If they persist, report as unresolved.
8. **Ask before architectural changes** — If a fix requires adding a new endpoint, new component, or new migration, ask the user first.

## Important Notes

- This skill combines diagnosis + fix + verification in a single workflow
- Each QA type's diagnosis follows its own skill.md algorithm exactly
- Fixes are applied in dependency order (DB → Backend → Frontend types → Frontend validation → Frontend UI)
- Phase 3 verification re-runs the SAME checks, not a subset — all skill types from Phase 1 are re-verified
- The final report clearly shows what was resolved, what remains, and any regressions
- If `--check` is omitted, ALL 15 QA types are run (full audit mode)
- TypeScript compilation and lint checks are mandatory between Phase 2 and Phase 3
- When pages have 3+ concentrated issues, qa-page integration provides deeper per-page analysis
