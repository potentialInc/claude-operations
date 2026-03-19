---
name: qa-db-integrity
description: "Audit database schema consistency - Entity vs migrations, FK relations, index coverage, and migration gaps"
user-invocable: true
argument-hint: "[module]"
---

# QA DB Integrity — Database Schema Consistency Audit

Verify that ORM entities/models, migrations, and database schema expectations are in sync, detecting missing migrations, orphaned columns, FK relationship issues, missing indexes, and schema drift.

## Execution Mode

- **Standalone** (`/qa-db-integrity [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check data`): qa-fix uses the checks below as its checklist for the data layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Entity-Migration drift | **Critical** | Entity column defined but not created in any migration |
| 2 | Orphan migration column | **High*** | Migration creates column that no entity maps to (see classification guide below) |
| 3 | FK without index | **High** | Foreign key column has no corresponding index |
| 4 | Missing cascade config | **Medium** | Parent is deletable but child has no onDelete option |
| 5 | Nullable inconsistency | **High** | Entity nullable setting != migration nullable setting |
| 6 | Type mismatch | **High** | Entity column type != migration column type |
| 7 | Missing unique constraint | **Medium** | Semantically unique fields (username/email/slug) lack unique constraint |
| 8 | Soft delete gaps | **Medium*** | Entity has deletedAt but some queries don't filter soft-deleted rows (escalate to **Critical** if query serves user-facing data — see note below) |
| 9 | Decimal without precision/scale | **Medium** | Decimal/float/numeric column missing explicit precision or scale |
| 10 | Default value divergence | **High** | Entity default vs migration DEFAULT mismatch or one side missing |
| 11 | Raw table name mismatch | **Critical** | QueryBuilder `leftJoin`/`innerJoin`/`from` uses hardcoded table name string that doesn't match the `@Entity('table_name')` decorator. TypeScript won't catch this — only fails at runtime. Grep for string-based joins and verify each table name against entity definitions. |

### Check 2 — Orphan Migration Column Classification Guide

Not all ghost columns (migration column with no entity mapping) are defects. Before flagging, determine the column's category:

| Category | Examples | Decision | Rationale |
|----------|----------|----------|-----------|
| **Standard extensibility pattern** | `social_login_type`, `oauth_provider`, `stripe_customer_id`, `two_factor_secret` | **Skip** | Common SaaS/auth features. Column is harmless (nullable, unused by ORM), and removing it destroys future extensibility. Removing + re-adding costs 2 migrations for no runtime benefit. |
| **Orphaned from deleted feature** | Column added for a feature that was explicitly removed (feature flag dropped, module deleted, PR reverted) | **High** — generate drop migration | Dead weight with no future intent. Check git history for deliberate removal signals. |
| **Scaffold/demo leftover** | Columns from boilerplate generators, tutorial code, or demo modules | **Manual** | Ask whether the demo module itself should be removed. If module stays, columns stay. |
| **External system dependency** | Columns written by triggers, ETL pipelines, reporting tools, or other services outside the ORM | **Skip** | ORM doesn't manage these; absence of entity mapping is expected. |
| **Truly unknown** | Cannot determine origin or intent from code, git history, or project docs | **Manual** | Escalate for human review rather than auto-deleting. |

**Key principle:** A ghost column that is nullable, has no index cost, and follows a recognizable pattern is NOT a defect — it is latent infrastructure. The cost of keeping it (a few bytes per row) is far lower than the cost of removing and re-adding it across environments.

### Check 8 — Soft Delete Gap Severity Escalation

Default severity is **Medium**, but escalate to **Critical** when:
- The query result is **rendered in user-facing UI** (chat messages, room lists, activity feeds, notifications)
- The query result is **returned in API responses** consumed by end users

Keep as **Medium** when:
- The query is used **internally only** (analytics, background jobs, admin-only reports, seeder scripts)
