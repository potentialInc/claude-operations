---
name: qa-crud
description: "Audit CRUD endpoint completeness - missing operations, response shapes, error handling, pagination, and frontend coverage"
user-invocable: true
argument-hint: "[module]"
---

# QA CRUD — CRUD Endpoint Completeness Audit

Verify every entity/resource has complete, consistent CRUD operations across the full stack, detecting missing endpoints, incomplete error handling, missing pagination, and frontend coverage gaps.

## Execution Mode

- **Standalone** (`/qa-crud [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check api`): qa-fix uses the checks below as its checklist for the api layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Missing CRUD operation | **High** | Entity has Create but no Read, etc. |
| 2 | Missing pagination | **High** | List endpoint returns all records without page/limit |
| 3 | Missing search/filter | **Medium** | List endpoint has no search/filter params |
| 4 | Missing sort | **Low** | List endpoint has no sort/order params |
| 5 | Inconsistent response shape | **Medium** | Some endpoints wrap in {data,meta} but others return raw |
| 6 | Missing error response | **High** | No exception handling for not-found, conflict, validation |
| 7 | No soft delete | **Medium** | Entity has deletedAt but controller uses hard delete |
| 8 | Missing bulk operations | **Low** | Only single-item CRUD, no bulk support |
| 9 | Missing ownership check | **High** | Update/Delete doesn't verify requesting user owns resource |
| 10 | Frontend CRUD gap | **High** | Backend endpoint exists but no frontend UI action |
| 11 | Missing status toggle | **Medium** | Entity has isActive/status column but no toggle endpoint or UI |
| 12 | Response field completeness | **High** | GET endpoint response missing entity columns that should be exposed |
| 13 | Static route shadowed by parameterized route | **High** | `@Patch('users/:id')` declared before `@Patch('users/bulk-status')` causes parameterized route to capture the static segment |
