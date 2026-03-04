---
description: Analyze frontend performance patterns and Core Web Vitals readiness — bundle impact, rendering efficiency, image optimization, code splitting, layout stability (CLS), and resource loading
argument-hint: "<page-or-component-file-path> [--backend <backend-src-path>] [--depth full|shallow] [--budget <json-path>]"
---

# QA Frontend Performance / Core Web Vitals

Analyze frontend performance patterns in a page or component, detect bundle bloat, rendering inefficiencies, image optimization gaps, code splitting opportunities, layout stability (CLS) risks, and resource loading issues, then generate a comprehensive QA report with actionable test cases.

---

## Purpose

This command helps you:
1. **Analyze bundle impact** — Detect heavy dependencies, barrel imports, tree-shaking failures
2. **Detect rendering inefficiencies** — Unnecessary re-renders, missing memoization, expensive computations in render
3. **Audit image optimization** — Unoptimized formats, missing dimensions (CLS), lazy loading, responsive images
4. **Check code splitting** — Route-level splitting, dynamic imports, component lazy loading
5. **Analyze data fetching** — Waterfalls, missing prefetch, cache strategies, over-fetching
6. **Verify layout stability** — CLS sources, dynamic content injection, font loading strategy
7. **Detect resource loading issues** — Render-blocking scripts, unoptimized fonts, third-party script impact
8. **Generate test cases** — Produce actionable performance test checklists

**Important:** This command is **read-only**. It analyzes source code and generates reports — it does NOT modify any files.

---

## Prerequisites

- A frontend page or component file (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)
- (Optional) Backend source path for full-stack data fetching analysis
- (Optional) Performance budget JSON file for threshold comparison
- (Optional) Project `CLAUDE.md` with project-specific conventions

---

## Usage

```bash
/qa-performance <page-or-component-file-path>
```

**Examples:**
```bash
# React page with Next.js
/qa-performance src/app/dashboard/page.tsx

# Vue page with Nuxt
/qa-performance pages/products/index.vue

# Full-stack analysis with backend path
/qa-performance src/pages/UserList.tsx --backend backend/src --depth full

# With performance budget comparison
/qa-performance src/app/page.tsx --budget perf-budget.json

# Shallow analysis — only the given file, no import tracing
/qa-performance src/components/Dashboard.tsx --depth shallow
```

**Arguments:**
| Argument | Required | Description |
|----------|----------|-------------|
| `<file-path>` | Yes | Path to the frontend page/component file |
| `--backend <path>` | No | Path to the backend source directory for data fetching analysis |
| `--depth full\|shallow` | No | `full` (default): trace imports and child components. `shallow`: analyze only the given file |
| `--budget <json-path>` | No | Path to performance budget JSON file for threshold comparison |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Input Validation & Project Detection                │
│  - Validate file path and extension                          │
│  - Detect frontend framework (React/Vue/Angular/Svelte/HTML)│
│  - Detect bundler, performance libraries, optimization tools │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Bundle & Dependency Analysis                        │
│  - Heavy dependency detection (lodash, moment, barrel)       │
│  - Tree-shaking and sideEffects verification                 │
│  - Duplicate dependency detection                            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Rendering Efficiency Analysis                       │
│  - Missing memoization (React.memo, useMemo, useCallback)    │
│  - Inline object/array creation in JSX props                 │
│  - Context provider optimization, cascading re-renders       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Image & Media Optimization                          │
│  - Missing width/height (CLS), lazy loading, srcset          │
│  - LCP image preload, framework image components             │
│  - Video/GIF optimization, SVG bundle impact                 │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Code Splitting & Lazy Loading                       │
│  - Route-level splitting (React.lazy, dynamic import)        │
│  - Component-level splitting (heavy libs, modals, charts)    │
│  - Prefetching strategy (link prefetch, data prefetch)       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Layout Stability (CLS) Analysis                     │
│  - Font loading strategy (FOIT/FOUT, font-display, preload)  │
│  - Dynamic content injection above fold                      │
│  - CSS loading and @import chains                            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Resource Loading & Third-Party Analysis             │
│  - Render-blocking scripts/CSS, critical CSS extraction      │
│  - Third-party script impact, preconnect/prefetch hints      │
│  - Caching strategy, service worker registration             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 8: Output Generation                                   │
│  - Generate report with standard header                      │
│  - Save report and test cases to .claude-project/qa/         │
│  - Display summary with actionable items                     │
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

Read the file path from `$ARGUMENTS`. Parse optional flags (`--backend`, `--depth`, `--budget`).

**1.1 File Validation:**
1. File path is provided
2. File exists and is readable
3. File extension is supported (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.ts`, `.js`)

**If validation fails:**
```
Error: [specific error message]
Usage: /qa-performance <page-or-component-file-path> [--backend <path>] [--depth full|shallow] [--budget <json-path>]
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

**1.3 Bundler Detection:**

| Signal | Bundler |
|--------|---------|
| `next.config.js` or `next.config.mjs` or `next.config.ts` | Next.js (Webpack/Turbopack) |
| `vite.config.ts` or `vite.config.js` | Vite |
| `webpack.config.js` or `webpack.config.ts` | Webpack |
| `esbuild` in package.json scripts or `esbuild.config.js` | esbuild |
| `turbo.json` + `turbopack` flag in next dev | Turbopack |
| `rspack.config.js` or `@rspack/core` in package.json | Rspack |
| `rollup.config.js` or `rollup.config.ts` | Rollup |
| `parcel` in package.json or `.parcelrc` | Parcel |

**1.4 Performance Library Detection:**

| Signal | Library |
|--------|---------|
| `web-vitals` in package.json | web-vitals (Real User Metrics) |
| `@next/bundle-analyzer` in package.json | Next.js Bundle Analyzer |
| `webpack-bundle-analyzer` in package.json | Webpack Bundle Analyzer |
| `source-map-explorer` in package.json | Source Map Explorer |
| `@lhci/cli` or `lighthouse-ci` in package.json | Lighthouse CI |

**1.5 Optimization Library Detection:**

| Signal | Library |
|--------|---------|
| `next/image` import | Next.js Image Optimization |
| `@vercel/og` in package.json | Vercel OG Image Generation |
| `sharp` in package.json | Sharp (image processing) |
| `plaiceholder` in package.json | Plaiceholder (blur placeholders) |
| `next/font` import | Next.js Font Optimization |

**1.6 Configuration Reading:**
- Read bundler config if present (`next.config.js`, `vite.config.ts`, `webpack.config.js`)
- Read performance budget if `--budget` provided
- Read `.qa-config.json` if present (see `qa-shared-reference.md`)

**Step 1 Checklist:**
```
[ ] File path validated and file exists
[ ] File extension is supported
[ ] Frontend framework detected
[ ] Bundler detected
[ ] Performance libraries detected
[ ] Optimization libraries detected
[ ] Bundler config read (if present)
[ ] Performance budget loaded (if --budget provided)
```

---

### Step 2: Bundle & Dependency Analysis

Analyze imports and dependencies for bundle size impact.

**2.1 Heavy Dependency Detection:**

Scan all imports in the file and its dependency tree:

| Pattern | Issue | Severity | Recommendation |
|---------|-------|----------|----------------|
| `import _ from 'lodash'` | Full lodash bundle (~70KB gzipped) | WARNING | Use `import debounce from 'lodash/debounce'` |
| `import { debounce } from 'lodash'` | Named import still pulls full bundle without tree-shaking | WARNING | Use `lodash/debounce` or `lodash-es` |
| `import moment from 'moment'` | moment.js (~67KB gzipped, not tree-shakeable) | WARNING | Use `date-fns` or `dayjs` |
| `import { Button } from '@mui/material'` | Barrel import may pull entire MUI | WARNING | Use `@mui/material/Button` or configure `modularizeImports` |
| `import { Table } from 'antd'` | Barrel import pulls entire Ant Design | WARNING | Use `antd/es/table` or configure tree-shaking |
| `import { Icon } from '@iconify/react'` + `import { FaHome } from 'react-icons/fa'` | Multiple icon libraries loaded | INFO | Consolidate to one icon solution |
| `import 'core-js'` or `polyfill.io` script | Large polyfill bundle for modern-only targets | WARNING | Use `browserslist` with targeted polyfills |

**2.2 Barrel Import Analysis:**

Detect barrel re-exports that defeat tree-shaking:

| Pattern | Issue | Check |
|---------|-------|-------|
| `index.ts` with `export * from './ComponentA'` | Barrel re-export | Does bundler support tree-shaking for this? |
| `export { default as X } from './X'` | Named re-export | Less problematic but check `sideEffects` |
| `sideEffects: false` missing in `package.json` | Tree-shaking may be disabled | CRITICAL for library packages |

Check bundler configuration:
- Next.js: `optimizePackageImports` in `next.config.js` (auto-tree-shakes configured packages)
- Next.js: `modularizeImports` for custom barrel import rewriting
- Webpack: `sideEffects` flag in `package.json`
- Vite: Automatic tree-shaking via Rollup

**2.3 Duplicate Dependency Detection:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| Multiple React versions in bundle | Hooks will break; increases bundle | CRITICAL |
| Multiple versions of lodash (lodash + lodash-es) | Duplicate utility code | WARNING |
| Multiple CSS-in-JS runtimes (styled-components + emotion) | Double runtime overhead | WARNING |
| Multiple state management libraries (Redux + Zustand + MobX) | Unnecessary bundle weight | INFO |

**2.4 Dynamic Import Opportunities:**

| Pattern | Issue | Recommendation |
|---------|-------|----------------|
| `import ChartComponent from './Chart'` used inside `{showChart && <ChartComponent />}` | Heavy component conditionally rendered but eagerly loaded | Use `React.lazy(() => import('./Chart'))` |
| `import Editor from '@monaco-editor/react'` at top level | Heavy editor (~200KB) loaded on page entry | Dynamic import when editor is needed |
| `import MapView from './MapView'` rendered in a tab | Below-fold content loaded eagerly | Lazy load tab content |
| Route components imported with static `import` | All routes bundled together | Use dynamic `import()` for route components |

**2.5 Framework-Specific Bundle Checks:**

**Next.js:**
- `output: 'standalone'` in `next.config.js` — optimizes for container deployment
- `optimizePackageImports` — auto-transforms barrel imports
- `modularizeImports` — rewrites specific imports to deep paths
- Server Components — code stays on server, not shipped to client

**Vite:**
- `build.rollupOptions.output.manualChunks` — custom chunk splitting
- `optimizeDeps.include` — pre-bundle dependencies for faster dev
- `build.cssCodeSplit` — CSS per-chunk splitting

**Webpack:**
- `optimization.splitChunks` configuration — vendor chunk strategy
- `optimization.usedExports` — tree-shaking enabled
- `externals` — exclude large deps from bundle (CDN)

**Step 2 Checklist:**
```
[ ] All imports scanned for heavy dependencies
[ ] Barrel import patterns identified
[ ] sideEffects configuration checked in package.json
[ ] Duplicate dependencies detected (if any)
[ ] Dynamic import opportunities identified
[ ] Framework-specific bundle config checked
[ ] Bundle optimization recommendations generated
```

---

### Step 3: Rendering Efficiency Analysis

Analyze component rendering patterns for performance anti-patterns.

**3.1 React-Specific Patterns:**

| Pattern | Issue | Severity | Fix |
|---------|-------|----------|-----|
| List items without `React.memo` | Every parent re-render re-renders all list items | WARNING | Wrap list item component with `React.memo` |
| Missing `useMemo` for expensive computation | `items.filter().sort().map()` runs on every render | WARNING | Wrap with `useMemo(() => ..., [items])` |
| Missing `useCallback` for event handlers | `onClick={() => handleClick(id)}` recreated each render, triggers child re-render | WARNING | Use `useCallback` or extract handler |
| `style={{color: 'red'}}` in loops | Inline object creates new reference every render | WARNING | Extract to constant or `useMemo` |
| `options={[{label: 'A', value: 1}]}` in JSX | Array literal recreated every render | WARNING | Extract to constant or `useMemo` |
| `useEffect` with no dependency array | Effect runs on every render | CRITICAL | Add dependency array `[]` or `[deps]` |
| `useEffect` with missing dependencies | Stale closure, unexpected behavior | WARNING | Add missing deps or use ref |
| Multiple `setState` calls in sequence | Each `setState` may trigger separate re-render (pre-React 18 batching) | INFO | Use single state object or `useReducer` |
| Large context object without splitting | Any context value change re-renders all consumers | WARNING | Split context by update frequency |
| Context provider at root with frequently changing value | Entire subtree re-renders on context change | WARNING | Memoize context value or split providers |

**3.2 Vue-Specific Patterns:**

| Pattern | Issue | Severity | Fix |
|---------|-------|----------|-----|
| Method used instead of `computed` for derived data | Recalculates on every render, not cached | WARNING | Use `computed(() => ...)` |
| `watch` with `deep: true` on large objects | Deep comparison on every change is expensive | WARNING | Watch specific nested paths instead |
| `v-for` without `:key` | Forces full list re-render on any change | CRITICAL | Add unique `:key` binding |
| `v-for` with `index` as `:key` on reorderable lists | Incorrect element reuse on reorder/delete | WARNING | Use unique item ID as key |
| Reactive object unnecessarily large | Entire object tracked reactively | INFO | Use `shallowRef` or `shallowReactive` |

**3.3 Svelte-Specific Patterns:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| Reactive declaration `$:` doing expensive work without guard | Runs on any dependency change | WARNING |
| Large array/object in store without derived | Entire store update triggers all subscribers | INFO |

**3.4 General Patterns:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| Large component tree without lazy boundaries | Entire tree renders before first paint | WARNING |
| Scroll/resize handler without throttle/debounce | Handler fires 60+ times/second | WARNING |
| Layout thrashing: interleaved DOM read/write in loops | Forces synchronous layout recalculation | CRITICAL |
| `document.querySelectorAll` in render/update cycle | DOM queries in hot path | WARNING |

**Step 3 Checklist:**
```
[ ] React memo/useMemo/useCallback patterns checked
[ ] Inline object/array creation in JSX identified
[ ] useEffect dependency arrays validated
[ ] Context provider optimization checked
[ ] Framework-specific rendering patterns analyzed
[ ] General rendering anti-patterns detected
[ ] Rendering optimization recommendations generated
```

---

### Step 4: Image & Media Optimization

Audit all image and media usage for Core Web Vitals impact.

**4.1 Image Issue Detection:**

| Pattern | Issue | CWV Impact | Severity |
|---------|-------|------------|----------|
| `<img>` without `width` and `height` attributes | Causes layout shift when image loads | CLS | WARNING |
| `<img>` without `loading="lazy"` on below-fold images | All images load on page entry | LCP/FCP | WARNING |
| No `srcset` or `<picture>` for responsive images | Desktop-size image served to mobile | LCP | WARNING |
| BMP or uncompressed PNG used for photos | Dramatically larger file size than WebP/AVIF | LCP | WARNING |
| LCP image not preloaded (`<link rel="preload" as="image">`) | LCP image discovered late by browser | LCP | CRITICAL |
| LCP image uses `loading="lazy"` | Contradicts preload; delays LCP | LCP | CRITICAL |
| `<img>` without `alt` attribute | Accessibility and SEO impact | A11y | WARNING |
| Images larger than 200KB without compression | Excessive download size | LCP | WARNING |

**4.2 Framework-Specific Image Checks:**

| Framework | Pattern | Issue | Severity |
|-----------|---------|-------|----------|
| Next.js | Using `<img>` instead of `next/image` | Missing automatic optimization, resizing, lazy loading | WARNING |
| Next.js | `next/image` without `priority` on LCP image | LCP image not preloaded | CRITICAL |
| Next.js | `next/image` with `unoptimized: true` | Bypasses image optimization pipeline | INFO |
| Next.js | `next/image` without `sizes` prop | Cannot generate optimal srcset | WARNING |
| Nuxt | Using `<img>` instead of `<nuxt-img>` | Missing Nuxt image optimization | WARNING |
| Astro | Using `<img>` instead of `<Image />` | Missing Astro image optimization | WARNING |

**4.3 Video & Media Checks:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| `<video autoplay>` without `preload="none"` or `preload="metadata"` | Full video downloads immediately on page load | WARNING |
| Large GIF files (> 500KB) | GIFs are extremely inefficient | WARNING — suggest `<video>` or animated WebP |
| `<video>` without `poster` attribute | Shows blank frame until video loads | INFO |

**4.4 SVG Analysis:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| Large inline SVGs in component (> 5KB each) | Increases JS bundle size | INFO |
| Many inline SVGs (> 10 in one component) | Consider SVG sprite or external file | WARNING |
| SVG without `viewBox` attribute | Cannot scale responsively | INFO |

**Step 4 Checklist:**
```
[ ] All <img> tags analyzed for width/height, lazy loading, srcset
[ ] LCP image identified and preload status checked
[ ] Framework-specific image component usage verified
[ ] Video/media autoplay and preload checked
[ ] SVG usage analyzed (inline vs sprite vs external)
[ ] Image optimization recommendations generated
```

---

### Step 5: Code Splitting & Lazy Loading

Analyze code splitting strategy and lazy loading implementation.

**5.1 Route-Level Splitting:**

| Framework | Expected Pattern | Issue if Missing |
|-----------|-----------------|------------------|
| React | `React.lazy(() => import('./Page'))` + `<Suspense>` | All routes in single bundle |
| Next.js (App Router) | Automatic per-route splitting | N/A — handled by framework |
| Next.js (Pages Router) | `dynamic(() => import('./Component'))` | Components eagerly loaded |
| Vue Router | `() => import('./views/Page.vue')` in route config | All routes in single bundle |
| Nuxt | Automatic per-page splitting | N/A — handled by framework |
| Svelte | `await import('./Page.svelte')` in route | All routes in single bundle |
| Angular | `loadChildren: () => import('./module')` | Eager module loading |

**5.2 Component-Level Splitting:**

| Pattern | Issue | Severity | Recommendation |
|---------|-------|----------|----------------|
| `import RichTextEditor from '@tiptap/react'` at top level | Heavy editor (~200KB) loaded on page entry | WARNING | Dynamic import when editor is needed |
| `import Chart from 'recharts'` or `chart.js` at top level | Charting library loaded before chart is visible | WARNING | Lazy load chart component |
| `import MapView from 'react-map-gl'` at top level | Map library (~300KB) loaded eagerly | WARNING | Lazy load map component |
| Modal/drawer content imported statically | Modal content loaded before modal opens | INFO | Lazy load modal content component |
| Below-fold sections imported at top level | Content not visible on initial load shipped in main bundle | INFO | Lazy load below-fold sections |

**5.3 Library-Level Splitting:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| `import pdf from 'pdfjs-dist'` at top level | PDF library loaded for all users | WARNING |
| Admin panel code bundled with user code | Admin features shipped to non-admin users | WARNING |
| Premium features bundled with free tier | Unnecessary code shipped to free users | INFO |
| `import * as xlsx from 'xlsx'` at top level | Excel library loaded before export action | WARNING |

**5.4 Prefetching Strategy:**

| Pattern | Assessment | Check |
|---------|------------|-------|
| Next.js `<Link>` default prefetch on viewport | Good — prefetches visible links | Ensure not too many links on page (bandwidth) |
| `<link rel="prefetch" href="/next-page.js">` | Manual prefetch for likely next page | Appropriate for known navigation paths |
| TanStack Query `prefetchQuery` on hover | Good — data ready when user clicks | Verify cache time is sufficient |
| SWR `preload(key, fetcher)` | Good — preload data before render | Verify deduplication with actual fetch |
| Prefetching all routes eagerly | Over-prefetching wastes bandwidth on mobile | WARNING — limit to likely navigation paths |

**Step 5 Checklist:**
```
[ ] Route-level code splitting verified
[ ] Heavy component lazy loading checked (editors, charts, maps)
[ ] Library-level splitting opportunities identified
[ ] Modal/drawer content lazy loading checked
[ ] Prefetching strategy assessed (link prefetch, data prefetch)
[ ] Code splitting recommendations generated
```

---

### Step 6: Layout Stability (CLS) Analysis

Analyze patterns that cause Cumulative Layout Shift.

**6.1 Font Loading:**

| Pattern | Issue | CLS Impact | Severity |
|---------|-------|------------|----------|
| No `font-display` declaration | Browser default (block then swap) causes FOIT | High | WARNING |
| `font-display: swap` without size-adjust fallback | Text shifts when web font loads | Medium | INFO |
| `font-display: optional` | No layout shift (font may not show) | None | Good pattern |
| Missing `<link rel="preload" as="font" crossorigin>` for critical fonts | Font discovered late, delays render | Medium | WARNING |
| Next.js: Not using `next/font` | Missing zero-layout-shift font loading | Medium | WARNING |
| Multiple custom fonts loaded | Each font is a separate request | Cumulative | INFO |

**6.2 Dynamic Content Injection:**

| Pattern | Issue | CLS Impact | Severity |
|---------|-------|------------|----------|
| Ad/embed slots without reserved dimensions | Content shifts down when ad loads | High | CRITICAL |
| `<img>` without width/height (also in Step 4) | Layout shifts when image loads | High | WARNING |
| Banner/alert injected above existing content (e.g., toast at top) | Pushes all content down | High | WARNING |
| Skeleton screen dimensions don't match actual content | Shift when real content replaces skeleton | Medium | INFO |
| Cookie consent banner shifts page content | Overlay preferred over inline push | Medium | INFO |
| Lazy-loaded components without min-height placeholder | Content area collapses then expands | Medium | WARNING |

**6.3 CSS-Related CLS Sources:**

| Pattern | Issue | CLS Impact | Severity |
|---------|-------|------------|----------|
| `@import` in CSS files | Cascading requests; delays render | Medium | WARNING |
| CSS loaded asynchronously without critical CSS inline | Flash of unstyled content (FOUC) | High | WARNING |
| Large CSS file blocking render | Delays first paint | High | WARNING |
| CSS-in-JS runtime generating styles after hydration | Style shift after JavaScript executes | Medium | INFO |

**6.4 Aspect Ratio & Reserved Space:**

| Pattern | Assessment |
|---------|------------|
| `aspect-ratio` CSS property on media containers | Good — reserves space before content loads |
| `padding-bottom` percentage trick for aspect ratio | Good — legacy but effective |
| `min-height` on dynamic content containers | Acceptable — prevents collapse |
| No dimension reservation on dynamic content | Bad — causes layout shift |

**Step 6 Checklist:**
```
[ ] Font loading strategy analyzed (font-display, preload, next/font)
[ ] Dynamic content injection patterns checked
[ ] Ad/embed reserved space verified
[ ] Skeleton screen dimension matching checked
[ ] CSS loading and @import chains analyzed
[ ] Aspect ratio / reserved space patterns verified
[ ] CLS mitigation recommendations generated
```

---

### Step 7: Resource Loading & Third-Party Analysis

Analyze resource loading order and third-party script impact.

**7.1 Render-Blocking Resources:**

| Pattern | Issue | FCP Impact | Severity |
|---------|-------|------------|----------|
| `<script src="...">` in `<head>` without `async` or `defer` | Blocks HTML parsing until script loads and executes | High | CRITICAL |
| Large CSS file in `<head>` without critical CSS extraction | Blocks first paint until full CSS loads | High | WARNING |
| Third-party script in `<head>` without `async` | Blocks render for external dependency | High | CRITICAL |
| Multiple synchronous scripts in `<head>` | Cascading blocking | Very High | CRITICAL |

**7.2 Third-Party Script Impact:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| Google Analytics loaded synchronously | Blocks render for analytics | WARNING |
| Chat widget (Intercom, Drift, Zendesk) loaded eagerly | 200-500ms added to initial load | WARNING |
| Social embeds (Twitter, Instagram) loaded on page entry | Heavy iframes loaded eagerly | WARNING |
| A/B testing script (Optimizely, LaunchDarkly) in critical path | Blocks render for experiment assignment | CRITICAL |
| Tag manager loading multiple trackers synchronously | Cascading third-party loads | WARNING |
| `<iframe>` without `loading="lazy"` | Third-party content loaded eagerly | INFO |

**7.3 Preconnect & Resource Hints:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| No `<link rel="preconnect">` for API domain | DNS + TLS handshake delayed until first request | WARNING |
| No `<link rel="preconnect">` for CDN domain | Static asset DNS lookup delayed | WARNING |
| No `<link rel="preconnect">` for font provider (Google Fonts) | Font loading delayed | WARNING |
| No `<link rel="dns-prefetch">` for third-party domains | DNS lookup delayed until script executes | INFO |
| Missing `crossorigin` attribute on font preconnect | Preconnect ineffective for CORS fonts | WARNING |

**7.4 Caching Strategy:**

| Pattern | Issue | Severity |
|---------|-------|----------|
| Static assets without `Cache-Control` headers | Browser re-downloads assets on every visit | WARNING |
| No content hash in static asset filenames | Cache invalidation requires manual clearing | WARNING |
| API responses without cache headers | No client-side HTTP caching | INFO |
| `stale-while-revalidate` not used for cacheable API data | Unnecessary loading on repeat visits | INFO |
| No service worker for offline/cache strategy | No offline capability; no cache-first strategy | INFO |
| Service worker with aggressive caching but no update strategy | Users may see stale content | WARNING |

**7.5 Backend Data Fetching Analysis** (if `--depth full`):

| Pattern | Issue | Severity |
|---------|-------|----------|
| Sequential `await` calls that could be parallel (`await a; await b`) | Request waterfall; doubles loading time | WARNING |
| API response > 100KB without pagination | Large payload slows network transfer | WARNING |
| No data pagination (entire dataset returned) | Excessive data transfer | WARNING |
| Missing `select` or field filtering (over-fetching) | More data than needed transferred | INFO |
| N+1 query pattern (loop of API calls) | Linear scaling of requests | CRITICAL |

**Step 7 Checklist:**
```
[ ] Render-blocking scripts identified (sync scripts in <head>)
[ ] Third-party script loading strategy assessed
[ ] Preconnect/prefetch resource hints checked
[ ] Caching strategy reviewed (Cache-Control, service worker)
[ ] Backend data fetching patterns analyzed (if --depth full)
[ ] Resource loading recommendations generated
```

---

### Step 8: Output Generation

Compile all analysis results into a structured report.

**8.1 Report Sections:**

1. **Header** — Component name, file path, framework, bundler, date, overall score, status
2. **Bundle & Dependency Inventory** — Heavy deps, barrel imports, tree-shaking status
3. **Rendering Efficiency Matrix** — Memoization coverage, re-render risks, context optimization
4. **Image & Media Audit** — Per-image: dimensions, lazy loading, format, LCP status
5. **Code Splitting Assessment** — Route splitting, dynamic imports, prefetch strategy
6. **Layout Stability (CLS) Report** — Font loading, dynamic content, reserved space
7. **Resource Loading Analysis** — Blocking resources, third-party impact, cache strategy
8. **Performance Budget Comparison** — Budget vs actual (if `--budget` provided)
9. **Issues Found** — All issues with severity (CRITICAL / WARNING / INFO)
10. **Test Cases** — Complete test case table
11. **Recommendations** — Prioritized action items

**8.2 Save Report:**

Save to `.claude-project/qa/` directory:
- `[ComponentName]_Performance_QA_Report_[YYMMDD].md` — Full report
- `[ComponentName]_Performance_TestCases_[YYMMDD].md` — Detailed test case table (if large)

**8.3 Display Summary:**

```
## QA Performance Report Generated

### Output
- Report: .claude-project/qa/[ComponentName]_Performance_QA_Report_[YYMMDD].md
- Test Cases: .claude-project/qa/[ComponentName]_Performance_TestCases_[YYMMDD].md

### Summary
- Framework detected: [React + Next.js / Vue + Nuxt / etc.]
- Bundler detected: [Webpack / Vite / Turbopack / etc.]
- Heavy dependencies found: [N] (estimated excess: ~[N]KB gzipped)
- Rendering issues found: [N] (missing memo: [N], inline objects: [N])
- Image issues found: [N] (missing dimensions: [N], no lazy: [N])
- Code splitting gaps: [N] (heavy eager imports: [N])
- CLS risk sources: [N] (fonts: [N], dynamic content: [N])
- Blocking resources: [N] (sync scripts: [N], large CSS: [N])
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
[ ] Performance budget comparison included (if --budget provided)
[ ] Summary displayed with issue counts and action items
```

---

## Test Case Categories

### Category 1: Bundle Size Tests (BUNDLE-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| BUNDLE-01 | Check initial JS bundle size | 1. Build production 2. Measure main chunk size | Main JS bundle < 200KB gzipped |
| BUNDLE-02 | Verify tree-shaking works | 1. Import single function from utility library 2. Build 3. Check bundle | Only imported function included, not full library |
| BUNDLE-03 | Verify code splitting per route | 1. Build production 2. Check chunk files | Each route has separate chunk file |
| BUNDLE-04 | Check for duplicate dependencies | 1. Run `npm ls <library>` 2. Check for multiple versions | Single version of each dependency |
| BUNDLE-05 | Verify vendor chunk separation | 1. Build production 2. Check vendor chunk | Third-party code separated from app code |
| BUNDLE-06 | Check dynamic import chunks | 1. Build production 2. Verify lazy-loaded components have separate chunks | Heavy components in separate chunks |

### Category 2: Rendering Performance Tests (RENDER-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| RENDER-01 | List re-render on parent state change | 1. Open React DevTools Profiler 2. Change parent state unrelated to list | List items do NOT re-render |
| RENDER-02 | Expensive computation in component | 1. Profile component render time 2. Change unrelated state | Memoized computation does NOT re-execute |
| RENDER-03 | Context consumer re-render | 1. Update context value 2. Profile consumers | Only affected consumers re-render |
| RENDER-04 | Scroll handler performance | 1. Scroll page rapidly 2. Monitor frame rate | Maintains 60fps; handler is throttled/debounced |
| RENDER-05 | Large list rendering | 1. Render list with 1000+ items | Uses virtualization or pagination; no jank |
| RENDER-06 | Form input responsiveness | 1. Type rapidly in input field 2. Monitor frame rate | No lag; input stays responsive at 60fps |

### Category 3: Image Optimization Tests (IMG-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| IMG-01 | Image format optimization | 1. Check served image formats via DevTools Network | Modern formats (WebP/AVIF) served to supporting browsers |
| IMG-02 | Image dimension attributes | 1. Audit all `<img>` tags | Every image has explicit width and height or aspect-ratio |
| IMG-03 | Below-fold image lazy loading | 1. Load page 2. Check Network tab for image requests | Only above-fold images loaded initially; others load on scroll |
| IMG-04 | LCP image preload | 1. Check `<head>` for preload link 2. Measure LCP | LCP image preloaded; LCP < 2.5s on 4G |
| IMG-05 | Responsive images | 1. Resize browser 2. Check served image via DevTools | Appropriate size served for viewport width |
| IMG-06 | Image compression | 1. Check image file sizes via Network tab | No single image exceeds 200KB; photos use lossy compression |

### Category 4: Code Splitting Tests (SPLIT-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| SPLIT-01 | Route-level splitting | 1. Navigate to different routes 2. Watch Network tab | New JS chunk loaded for each route |
| SPLIT-02 | Modal content lazy load | 1. Load page 2. Open modal 3. Watch Network tab | Modal content chunk loaded on open, not on page load |
| SPLIT-03 | Heavy library lazy load | 1. Load page with chart/editor 2. Watch Network tab | Chart/editor library loaded when component renders, not on page entry |
| SPLIT-04 | Link prefetch on hover | 1. Hover over navigation link 2. Watch Network tab | Next page chunk prefetched on hover |
| SPLIT-05 | Data prefetch | 1. Hover over link with data prefetch 2. Click link | Data available immediately; no loading spinner |

### Category 5: Layout Stability Tests (CLS-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| CLS-01 | Font loading layout shift | 1. Throttle network 2. Load page 3. Watch for text shift | No visible text shift when web font loads |
| CLS-02 | Image loading layout shift | 1. Throttle network 2. Load page 3. Watch for content shift | No content shift when images load |
| CLS-03 | Dynamic banner injection | 1. Load page 2. Wait for dynamic banner/alert | Banner does not push content down (overlay or reserved space) |
| CLS-04 | Lazy-loaded content shift | 1. Scroll to lazy-loaded section 2. Watch for shift | Placeholder maintains correct height; no shift on load |
| CLS-05 | CLS score measurement | 1. Run Lighthouse or Web Vitals 2. Check CLS score | CLS < 0.1 (Good threshold per Core Web Vitals) |

### Category 6: Resource Loading Tests (RES-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| RES-01 | Render-blocking script check | 1. Audit `<head>` scripts 2. Check async/defer attributes | No synchronous third-party scripts in `<head>` |
| RES-02 | Third-party script impact | 1. Load page with/without third-party scripts 2. Compare FCP | Third-party scripts add < 200ms to FCP |
| RES-03 | Preconnect for known origins | 1. Check `<head>` for preconnect links | Preconnect set for API, CDN, and font origins |
| RES-04 | Cache header verification | 1. Load page 2. Check `Cache-Control` on static assets | Static assets have long cache (1 year); hashed filenames |
| RES-05 | Service worker caching | 1. Load page 2. Go offline 3. Reload | Critical assets served from cache; offline fallback shown |

### Category 7: Data Fetching Performance Tests (FETCH-xx)

| Test ID | Scenario | Steps | Expected |
|---------|----------|-------|----------|
| FETCH-01 | Request waterfall detection | 1. Load page 2. Check Network waterfall | API calls execute in parallel, not sequentially |
| FETCH-02 | API response size | 1. Check Network response sizes | No single API response > 100KB; pagination used for lists |
| FETCH-03 | Cache hit on revisit | 1. Visit page 2. Navigate away 3. Return | Data served from cache; no redundant API call |
| FETCH-04 | Over-fetching check | 1. Compare API response fields with rendered fields | No large unused data fields in API response |
| FETCH-05 | Prefetch on likely navigation | 1. Hover over list item 2. Click to navigate | Detail data prefetched on hover; instant navigation |

---

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Bundle & Dependencies | 20% | Heavy deps, barrel imports, tree-shaking, splitting config |
| Rendering Efficiency | 20% | Re-renders, memoization, context optimization |
| Image & Media | 15% | Optimization, dimensions, lazy loading, LCP preload |
| Code Splitting | 15% | Route splitting, dynamic imports, prefetch |
| Layout Stability (CLS) | 10% | Font loading, reserved space, dynamic injection |
| Resource Loading | 10% | Blocking resources, third-party, caching |
| Data Fetching | 10% | Waterfalls, cache strategy, prefetch |

**Scoring rules** (see `qa-shared-reference.md` for full scoring system):

```
Base Score: 100
Deductions: CRITICAL = -15, WARNING = -5, INFO = 0
Score = max(0, 100 - sum(deductions))
Bonus (capped at +10 total):
  - Performance budget defined and enforced: +3
  - Bundle analyzer configured in CI: +2
  - web-vitals library tracking real user metrics: +3
  - Service worker with caching strategy: +2
Final Score = min(100, Score + Bonus)
```

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
| No imports found | Static HTML or display-only component | Warning shown; limited bundle analysis possible |
| Framework not detected | Unusual structure | Falls back to generic pattern scanning |
| Bundler not detected | No config file or non-standard setup | Falls back to import-based analysis only |
| package.json not found | No package manager | Falls back to import-based detection; bundle analysis limited |
| Performance budget file not found | Invalid `--budget` path | Warning shown; budget comparison skipped |
| Backend path not found | `--backend` invalid or no sibling `backend/` | Runs frontend-only analysis (`--depth shallow`) |
| Import chain unresolvable | Complex re-exports or circular imports | Breaks after 10 levels; notes as "unresolvable" |
| Bundler config syntax error | Invalid JS/TS in config | Notes config as "unreadable — manual review needed" |

---

## Edge Cases Handled

- **Server Components** — RSC payload is not JS bundle; do not flag `async` server component as missing Suspense; server-only code excluded from bundle analysis
- **Streaming SSR** — `loading.tsx` is a code-splitting boundary, not a performance issue; treat as correct Suspense usage
- **Static generation (SSG/ISR)** — Pre-rendered pages have different performance profile than SSR/CSR; skip runtime rendering analysis for static pages
- **Micro-frontends** — Module federation chunks analyzed separately; shared dependencies expected across host and remotes
- **Monorepo shared packages** — Internal packages may be pre-bundled; `workspace:*` dependencies handled differently than npm packages
- **CSS-in-JS runtime cost** — `styled-components` and `emotion` have runtime overhead; flag if zero-runtime alternatives (vanilla-extract, Panda CSS, Tailwind) would reduce bundle
- **Hydration cost** — Large interactive component trees increase TTI; React Server Components reduce hydration payload
- **Edge runtime** — Smaller bundle requirements for Vercel Edge Functions and Cloudflare Workers; stricter size limits apply
- **Web Workers** — Heavy computation offloaded to workers is a positive pattern; do not flag worker imports as missing lazy loading
- **WebAssembly** — WASM modules need different loading and caching strategy; `.wasm` files not analyzed as JS bundles
- **Progressive Web App** — Service worker caching, offline mode, and install prompt are positive patterns; check cache update strategy
- **Prefetch over-eager** — Prefetching too many routes wastes bandwidth on mobile connections; flag if more than 5 routes prefetched simultaneously
- **Third-party script sandbox** — `<iframe>` isolation for third-party widgets is a positive pattern; do not flag as performance issue
- **Font subsetting** — CJK fonts can be several MB; subsetting is critical for performance; flag full CJK font loads
- **Large JSON payloads** — API responses > 100KB without pagination indicate over-fetching; flag as WARNING
- **Client-side rendering fallback** — SSR-disabled pages may be intentional (e.g., dashboard with auth); check for `"use client"` or `ssr: false` config
- **Animation performance** — Only `transform` and `opacity` should be animated for 60fps; `width`, `height`, `top`, `left` trigger layout recalculation
- **Memory leaks** — Event listeners not cleaned up in `useEffect` cleanup, `setInterval` not cleared, WebSocket subscriptions not unsubscribed on unmount

---

## Related Commands

- `/qa-loading-error-empty` — Loading states, Suspense boundaries, error recovery (complements performance analysis)
- `/qa-table-list` — Large data virtualization, pagination vs infinite scroll performance impact
- `/qa-accessibility` — `prefers-reduced-motion`, animation alternatives, reduced data mode
- `/qa-back-navigation` — Route transition performance, scroll restoration, navigation prefetch

---

## Tips

1. **Static analysis has limits** — This command analyzes code patterns; actual performance requires runtime measurement with Lighthouse, WebPageTest, or real user monitoring (RUM)
2. **Don't memoize everything** — `React.memo` has overhead; only memoize components that re-render frequently with the same props; profiling should guide optimization
3. **Images are the lowest-hanging fruit** — Proper image optimization alone can improve LCP by 30-50%; start here for the biggest impact
4. **Code splitting at route level is the minimum** — Every route should be a separate chunk; component-level splitting is a bonus for heavy components
5. **Watch for waterfall requests** — Sequential API calls that could be parallel; look for `await a; await b` patterns that should be `Promise.all([a, b])`
6. **Third-party scripts are silent killers** — Analytics and chat widgets can add 200-500ms to initial load; defer them with `async` or load after first interaction
7. **Bundle size is a proxy metric** — What matters is what ships to the browser on first load (initial JS); lazy-loaded chunks are less urgent
8. **Test on slow devices** — Performance issues often only surface on mid-range Android devices with throttled 3G; Chrome DevTools throttling simulates this
