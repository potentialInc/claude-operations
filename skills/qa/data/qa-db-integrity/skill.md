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
| 2 | Orphan migration column | **High** | Migration creates column that no entity maps to |
| 3 | FK without index | **High** | Foreign key column has no corresponding index |
| 4 | Missing cascade config | **Medium** | Parent is deletable but child has no onDelete option |
| 5 | Nullable inconsistency | **High** | Entity nullable setting != migration nullable setting |
| 6 | Type mismatch | **High** | Entity column type != migration column type |
| 7 | Missing unique constraint | **Medium** | Semantically unique fields (username/email/slug) lack unique constraint |
| 8 | Soft delete gaps | **Medium** | Entity has deletedAt but some queries don't filter soft-deleted rows |
| 9 | Decimal without precision/scale | **Medium** | Decimal/float/numeric column missing explicit precision or scale |
| 10 | Default value divergence | **High** | Entity default vs migration DEFAULT mismatch or one side missing |
| 11 | Raw table name mismatch | **Critical** | QueryBuilder `leftJoin`/`innerJoin`/`from` uses hardcoded table name string that doesn't match the `@Entity('table_name')` decorator. TypeScript won't catch this — only fails at runtime. Grep for string-based joins and verify each table name against entity definitions. |
