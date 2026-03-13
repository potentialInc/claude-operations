---
name: qa-screen
description: Run Playwright-based screen-level QA across all pages - actionability, console errors, modals, a11y, CLS, user flows, form smoke
user-invocable: true
argument-hint: "[--frontend|--dashboard] [--layer=actionability|console|modal|accessibility|performance|flows|forms] [--critical]"
---

# QA Screen - Browser-Based Screen-Level QA

## Purpose

Unlike code-level QA skills that analyze source statically, this skill **runs Playwright in a real browser** to catch visual/interaction bugs:
- Buttons covered by overlapping elements (z-index, overflow)
- Forms that can't be submitted (submit button off-screen)
- Modals that won't close (ESC broken, no close button)
- JS runtime errors during page navigation
- Layout shift (CLS) causing click-miss issues
- Keyboard navigation gaps (unreachable elements)
- Form validation gaps (empty submit accepted)

## Usage

```
/qa-screen                              # Full 7-layer audit, both apps
/qa-screen --frontend                   # Primary frontend app only
/qa-screen --dashboard                  # Admin dashboard only
/qa-screen --layer=actionability        # Specific layer only
/qa-screen --layer=flows                # User flow scenarios only
/qa-screen --critical                   # Only pages tagged 'critical'
/qa-screen --viewport=mobile            # Mobile viewport only
```

## Prerequisites

1. Backend must be running (auto-detect port from project config)
2. Frontend dev servers must be running (auto-detect ports from project config)
3. Test users must exist in database (check CLAUDE.md or .env.test for credentials)
4. Optional: Install axe-core for Layer 4 (use detected package manager: `npm install -D @axe-core/playwright` / `yarn add -D` / `pnpm add -D` / `bun add -D`)

## 7 Layers

| # | Layer | What It Catches | Severity |
|---|-------|----------------|----------|
| L1 | **Actionability** | Covered/hidden/disabled/zero-size interactive elements, dead clicks, truncated text | Critical |
| L2 | **Console Errors** | JS errors, unhandled rejections, failed/slow network requests | Critical |
| L3 | **Modal Lifecycle** | Modals that won't open/close, broken ESC, focus trap failures | High |
| L4 | **Accessibility** | WCAG 2.0 AA violations (axe-core), keyboard navigation gaps | Medium |
| L5 | **CLS & Performance** | Layout shifts > 0.25, LCP > 4000ms | Medium |
| L6 | **User Flows** | Broken user journeys (login→navigate→interact→verify) | High |
| L7 | **Form Smoke** | Form detection, smart-fill JS errors, validation gaps | High |

## Execution Steps

### Step 1: Check server status
```bash
# Verify ports are listening (auto-detect ports from project config / package.json scripts)
# Example:
curl -s -o /dev/null -w "%{http_code}" http://localhost:{backend-port}/api || echo "Backend not running"
curl -s -o /dev/null -w "%{http_code}" http://localhost:{frontend-port} || echo "Frontend not running"
# Check all frontend apps as detected from project structure
```

### Step 2: Parse arguments and determine scope

| Argument | Effect |
|----------|--------|
| `--frontend` | Only run in the primary frontend app |
| `--dashboard` | Only run in the admin/dashboard app |
| `--layer=X` | Only run `X.spec.ts` |
| `--critical` | Pass `--grep "critical"` to Playwright |
| `--viewport=mobile` | Pass `--project=mobile-chrome` |
| No args | Run all layers in both apps |

### Step 3: Run Playwright tests

For each target app (frontend and/or dashboard):

```bash
cd <app-dir>

# Build the test command based on arguments
npx playwright test test/tests/screen-qa/ \
  --reporter=list \
  --reporter=./test/utils/screen-qa/reporter.ts \
  [layer filter] [grep filter] [project filter]
```

Layer filter examples:
- `--layer=actionability` → `test/tests/screen-qa/actionability.spec.ts`
- `--layer=console` → `test/tests/screen-qa/console-errors.spec.ts`
- `--layer=modal` → `test/tests/screen-qa/modal-lifecycle.spec.ts`
- `--layer=accessibility` → `test/tests/screen-qa/accessibility.spec.ts`
- `--layer=performance` → `test/tests/screen-qa/performance.spec.ts`
- `--layer=flows` → `test/tests/screen-qa/user-flows.spec.ts`
- `--layer=forms` → `test/tests/screen-qa/form-smoke.spec.ts`

### Step 4: Read and summarize results

After tests complete:
1. Read the HTML report at `test/reports/screen-qa/index.html`
2. Read JSON results at `test/reports/screen-qa/results.json`
3. Present a summary table to the user:
   - Layer-by-layer pass/fail counts
   - List of all failed tests with error messages
   - Suggest fixes for common issues

### Step 5: Output format

```
## Screen QA Results

### Summary
| Layer | Pass | Fail | Skip |
|-------|------|------|------|
| Actionability | 20/22 | 2 | 0 |
| Console Errors | 22/22 | 0 | 0 |
| ...

### Critical Issues (fix first)
1. **[Actionability] Item Detail Page** — "Submit" button covered by toast container
2. **[CLS] Admin Dashboard** — CLS 0.32 exceeds 0.25 threshold

### Warnings (review)
- ...

### Reports
- HTML: `{app-dir}/test/reports/screen-qa/index.html`
- JSON: `{app-dir}/test/reports/screen-qa/results.json`
```

## File Locations

### Frontend App
```
{frontend-dir}/test/
├── utils/screen-qa/
│   ├── config.ts                  # Allowlists, thresholds
│   ├── route-registry.ts          # All routes + metadata
│   ├── flow-registry.ts           # User flow scenarios
│   ├── actionability-checker.ts   # DOM audit engine
│   ├── console-monitor.ts         # Console/network monitor
│   ├── cls-measurer.ts            # CLS + LCP measurement
│   ├── form-detector.ts           # Form auto-detection
│   ├── keyboard-auditor.ts        # Tab order + focus check
│   └── reporter.ts                # Custom HTML reporter
└── tests/screen-qa/
    ├── actionability.spec.ts      # L1
    ├── console-errors.spec.ts     # L2
    ├── modal-lifecycle.spec.ts    # L3
    ├── accessibility.spec.ts      # L4
    ├── performance.spec.ts        # L5
    ├── user-flows.spec.ts         # L6
    └── form-smoke.spec.ts         # L7
```

### Additional Frontend Apps — same structure in each app's `test/` directory

## Maintenance

### Adding new routes
Edit `route-registry.ts` in the relevant app. Add:
- `path`, `name`, `auth`
- `navigationSteps` if dynamic route
- `hasModal` if page has modals
- `hasForms` if page has forms
- `tags` for filtering

### Adding new user flows
Edit `flow-registry.ts`. Follow existing step patterns.

### Adjusting thresholds
Edit `config.ts`:
- `consoleAllowlist` — patterns to ignore in console monitoring
- `performanceThresholds` — CLS/LCP limits
- `actionabilityConfig` — touch target size, selectors

### Skipping individual elements
Add `data-screen-qa-ignore` attribute to any element that should be excluded from actionability audit.
