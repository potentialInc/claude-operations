---
name: qa-api-sync
description: "Audit frontend API calls against backend endpoints for parameter, route, and response mismatches"
user-invocable: true
argument-hint: "[module]"
---

# QA API Sync — Frontend-Backend API Integration Audit

Detect mismatches between frontend API service calls and backend controller endpoints, including missing parameters, wrong HTTP methods, mismatched routes, unused endpoints, and response type inconsistencies.

## Execution Mode

- **Standalone** (`/qa-api-sync [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check api`): qa-fix uses the checks below as its checklist for the api layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Route mismatch | **Critical** | Frontend calls URL that doesn't exist in backend |
| 2 | HTTP method mismatch | **Critical** | Frontend uses POST but backend expects PATCH |
| 3 | Missing query params | **High** | Backend accepts search/filter/pagination but frontend doesn't send |
| 4 | Missing body params | **High** | DTO requires fields frontend doesn't include |
| 5 | Extra body params | **Medium** | Frontend sends fields not in DTO |
| 6 | Response type mismatch | **Medium** | Frontend TypeScript interface doesn't match backend response |
| 7 | Unused backend endpoint | **Info** | Controller endpoint with no frontend caller |
| 8 | Missing error handling | **Medium** | Frontend service call with no try/catch |
| 9 | Auth header missing | **High** | Protected endpoint but frontend doesn't include auth |
| 10 | Content-Type mismatch | **Medium** | Endpoint expects multipart but frontend sends JSON |
| 11 | Route parameter shadowing | **Critical** | Static route declared after parameterized route with same prefix — never reachable at runtime |
| 12 | Incomplete submit payload | **High** | Frontend form collects a value but the submit handler doesn't include it in the API call payload |
| 13 | Service layer field drop | **High** | Service method receives a DTO field but doesn't pass it through to the repository/database layer |
| 14 | Response construction field omission | **High** | Service method hand-builds a response object that omits entity fields the frontend expects |
