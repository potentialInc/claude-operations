---
name: qa-performance
description: "Audit frontend performance and Core Web Vitals - bundle impact, rendering efficiency, image optimization, code splitting, CLS, resource loading"
user-invocable: true
argument-hint: "[module or page-path]"
---

# QA Performance — Frontend Performance & Core Web Vitals Audit

Detect bundle bloat, rendering inefficiencies, image optimization gaps, code splitting opportunities, CLS risks, and resource loading issues via static analysis.

## Execution Mode

- **Standalone** (`/qa-performance [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check ui`): qa-fix uses the checks below as its checklist for the ui layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Heavy dependency detection | **High** | Full imports of heavy libraries (`lodash`, `moment`, `date-fns`) instead of tree-shakeable imports (e.g., `import get from 'lodash/get'`) |
| 2 | Barrel import bloat | **High** | Importing from barrel `index.ts` files that re-export entire modules, defeating tree-shaking |
| 3 | Duplicate dependencies | **Medium** | Multiple versions of the same library in the bundle (e.g., two versions of `lodash`) |
| 4 | Missing memoization | **Medium** | Components re-rendering on every parent render without `React.memo`, `useMemo`, or `useCallback` where beneficial |
| 5 | Inline object/array in JSX props | **Medium** | Object literals or array literals created inline in JSX props (e.g., `style={{ color: 'red' }}`) causing unnecessary re-renders |
| 6 | Expensive computation in render | **High** | Heavy computations (sort, filter, map over large arrays) inside render without `useMemo` |
| 7 | Image missing dimensions | **High** | `<img>` without explicit `width`/`height` attributes causes CLS (Cumulative Layout Shift) |
| 8 | Image lazy loading | **Medium** | Below-the-fold images without `loading="lazy"`. LCP image should NOT be lazy-loaded |
| 9 | Unoptimized image format | **Medium** | Using PNG/JPG when WebP/AVIF would be smaller. Not using framework image components (`next/image`, `nuxt-img`) |
| 10 | Missing route-level code splitting | **High** | Route components imported synchronously instead of using `React.lazy()`, `defineAsyncComponent()`, or dynamic `import()` |
| 11 | Heavy component not lazy-loaded | **Medium** | Large components (modals, charts, rich editors) imported statically instead of dynamically |
| 12 | Font loading CLS | **Medium** | Web fonts loaded without `font-display: swap` or `font-display: optional`, causing invisible text or layout shift |
| 13 | Dynamic content without reserved space | **High** | Content injected dynamically (ads, embeds, async data) without placeholder/skeleton causing CLS |
| 14 | Render-blocking resources | **High** | CSS/JS in `<head>` without `async`/`defer` blocking first paint |
| 15 | Third-party script impact | **Medium** | Third-party scripts (analytics, ads, chat widgets) loaded synchronously in the critical path |
| 16 | Data fetching waterfall | **High** | Sequential API calls that could be parallelized (`await a; await b` instead of `Promise.all`) |
| 17 | Missing data prefetch/cache | **Medium** | Pages that always fetch on mount without cache strategy (SWR, React Query, Apollo cache) |
| 18 | Over-fetching | **Medium** | API responses returning significantly more data than the component needs, without field selection |
