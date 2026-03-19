# QA Skills

Automated quality audit skills — framework-agnostic, stack-independent.

## Orchestrator

**`/qa-fix`** — Universal QA: diagnose, fix, verify. Auto-detects stack and runs relevant checks.

## Skill Layers

```
Layer 1 (Data)    →  Layer 2 (API)     →  Layer 3 (Auth)
Layer 4 (Inputs)  →  Layer 5 (UI)      →  Tools (standalone)
```

### Layer 1: Data

| Skill | Command | Description |
|-------|---------|-------------|
| [qa-db-integrity](data/qa-db-integrity/) | `/qa-db-integrity` | Entity vs migration, FK, index, migration gaps |
| [qa-dead-code](data/qa-dead-code/) | `/qa-dead-code` | Dead Entity columns — defined but never written |

### Layer 2: API

| Skill | Command | Description |
|-------|---------|-------------|
| [qa-api-sync](api/qa-api-sync/) | `/qa-api-sync` | Frontend API calls vs backend endpoints mismatch |
| [qa-crud](api/qa-crud/) | `/qa-crud` | CRUD completeness, response shapes, error handling |

### Layer 3: Auth

| Skill | Command | Description |
|-------|---------|-------------|
| [qa-auth](auth/qa-auth/) | `/qa-auth` | Role guards, route protection, token handling |
| [qa-security](auth/qa-security/) | `/qa-security` | SQL injection, XSS, file upload, crypto, config |

### Layer 4: Inputs

| Skill | Command | Description |
|-------|---------|-------------|
| [qa-inputs](inputs/qa-inputs/) | `/qa-inputs` | Entity → DTO → Zod → Form UI consistency |

### Layer 5: UI

| Skill | Command | Description |
|-------|---------|-------------|
| [qa-a11y](ui/qa-a11y/) | `/qa-a11y` | WCAG 2.2 accessibility audit |
| [qa-back-nav](ui/qa-back-nav/) | `/qa-back-nav` | Browser back, history stack, redirect chains |
| [qa-buttons](ui/qa-buttons/) | `/qa-buttons` | Button/link handlers, routes, loading states |
| [qa-layout](ui/qa-layout/) | `/qa-layout` | Page layout consistency, spacing, typography |
| [qa-list](ui/qa-list/) | `/qa-list` | List/table UX — sort, search, pagination, filters |
| [qa-modal](ui/qa-modal/) | `/qa-modal` | Modal/drawer lifecycle, focus, scroll lock |
| [qa-performance](ui/qa-performance/) | `/qa-performance` | Core Web Vitals, bundle, rendering, CLS |
| [qa-states](ui/qa-states/) | `/qa-states` | Loading, error, empty states, transitions |

### Tools (Standalone)

| Skill | Command | Description |
|-------|---------|-------------|
| [qa-page](tools/qa-page/) | `/qa-page` | Run all relevant QA on a single page |
| [qa-screen](tools/qa-screen/) | `/qa-screen` | Playwright-based screen-level QA |
| [qa-test-gen](tools/qa-test-gen/) | `/qa-test-gen` | Generate Playwright E2E tests for forms |
| [qa-skill-review](tools/qa-skill-review/) | `/qa-skill-review` | Validate QA skill files for compliance |

### Shared

| Skill | Description |
|-------|-------------|
| [qa-shared](qa-shared/) | Shared patterns and utilities used by other QA skills |

## Naming Convention

- **Operations repo**: Layered directories (`qa/data/qa-db-integrity/`, `qa/ui/qa-a11y/`)
- **Global skills** (`~/.claude/skills/`): Flat with prefix (`qa-db-integrity/`, `qa-a11y/`)
- **Slash commands**: Always `/qa-db-integrity`, `/qa-a11y`, etc.

See sync commands in [CLAUDE.md](../../CLAUDE.md#auto-sync-to-global-skills).
