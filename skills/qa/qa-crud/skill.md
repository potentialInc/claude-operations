---
name: qa-crud
description: Audit CRUD endpoint completeness - missing operations, response shapes, error handling, pagination, and frontend coverage
user-invocable: true
argument-hint: "[module]"
---

# QA CRUD - CRUD Endpoint Completeness Auditor

## Purpose

Verify every entity/resource has complete, consistent CRUD operations across the full stack. Detect missing endpoints, incomplete error handling, missing pagination, inconsistent response shapes, and frontend coverage gaps.

## Usage

```
/qa-crud                      # Full audit of all modules
/qa-crud products             # Single module audit
```

## 11 Checks

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

## Execution Algorithm

You MUST follow these steps in order. Do NOT skip steps. Do NOT modify any files. This is a READ-ONLY diagnostic.

### Step 0: Context & Requirements Gathering (MANDATORY)

Before auditing, you MUST understand what features are actually in scope for this project. Auditing code for unused/deprecated features leads to false positives.

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
Glob: .claude-project/docs/PROJECT_KNOWLEDGE.md
```

Extract:
- Which modules/entities are actively used in production
- Which modules are demo/boilerplate code (e.g., Feature module)
- Expected CRUD patterns per module (some entities are append-only by design)
- User roles and their CRUD permissions

**0b. Ask clarifying questions BEFORE proceeding:**

If you find entities/modules not mentioned in requirements, ask:

> "I found [entity/module X] but it doesn't appear in the project requirements. Should I include it in the CRUD audit or skip it?"

For entities that appear to be append-only (logs, surveys), ask:

> "Should [entity X] support Update/Delete operations, or is it intentionally append-only?"

Wait for the user's response before proceeding. Do NOT flag intentionally missing CRUD operations as bugs.

**0c. Build the In-Scope Module List:**
```
IN SCOPE: [module1, module2, ...]
OUT OF SCOPE: [module3 (reason), ...]
APPEND-ONLY (by design): [entity1, entity2, ...]
```

---

### Step 1: Resource Inventory

Scan all entities to build a complete resource list.

**Search for entity/model files:**

Auto-detect based on ORM:

| ORM | Glob Patterns |
|-----|--------------|
| TypeORM | `**/*.entity.ts`, `**/entities/*.entity.ts` |
| Prisma | `prisma/schema.prisma` |
| Sequelize | `**/models/*.ts`, `**/models/*.js` |
| Django ORM | `**/models.py` |
| SQLAlchemy | `**/models/*.py`, `**/models.py` |

For each entity/model file found, read the file and record:
- **Entity/model class name** (e.g., `User`, `Product`, `Order`)
- **Table name** from decorator/configuration
- **Module path** (which module directory it lives under)
- **Extends a base entity?** Check for base class inheritance — if yes, note auto-provided fields (e.g., `id`, `createdAt`, `updatedAt`, `deletedAt`)
- **Key relationships** — look for relation decorators/definitions to determine ownership
- **Soft delete support** — check for soft delete column or configuration
- **Unique constraints** — look for unique constraint definitions, as these inform conflict handling requirements

Build a master list like:
```
Resources:
  1. User         — module: users,      extends BaseEntity: yes, owner: self,    softDelete: yes
  2. Product      — module: products,   extends BaseEntity: yes, owner: admin,   softDelete: yes
  3. Order        — module: orders,     extends BaseEntity: yes, owner: user,    softDelete: yes
  4. OrderItem    — module: orders,     extends BaseEntity: yes, owner: order,   softDelete: yes
  5. Category     — module: categories, extends BaseEntity: yes, owner: system,  softDelete: yes
  6. Comment      — module: comments,   extends BaseEntity: yes, owner: user,    softDelete: yes
  ...
```

If a specific module argument was provided, filter the resource list to only entities within that module directory.

### Step 2: Controller/Route Inventory

For each resource found in Step 1, find its controller.

**Search for controller/route files:**

Auto-detect based on framework (same detection as qa-api-sync):

| Framework | Glob Patterns |
|-----------|--------------|
| NestJS | `**/*.controller.ts` |
| Express | `**/routes/*.ts`, `**/controllers/*.ts` |
| Spring Boot | `**/*Controller.java` |
| Django | `**/views.py`, `**/viewsets.py` |
| Laravel | `**/Controllers/*.php` |

For each controller, determine:

**A) Base route prefix:**
Read the route prefix from the framework-specific decorator/configuration.

**B) Base controller detection:**
Check if the controller extends a generic/base controller class. If it does, standard CRUD endpoints may be automatically provided without explicit method definitions. Read the base controller source to confirm the exact methods.

**C) Explicit route mapping:**
Even if the controller extends a base controller, it may override or add methods. Scan for explicit route definitions (adapt per framework):
- CREATE operation (POST)
- LIST operation (GET collection)
- DETAIL operation (GET single)
- UPDATE operation (PATCH/PUT)
- DELETE operation (DELETE)

Also look for non-standard CRUD routes:
- Bulk create/update/delete
- Dedicated search endpoint
- Restore soft-deleted record

**D) Guard/auth analysis:**
For each route, note (adapt per framework):
- Public/unauthenticated access markers
- Role-based access restrictions
- User context injection (current user parameter)

Build a CRUD matrix entry for each resource:
```
Product:
  CREATE:  POST   /products         — explicit route              ✅
  LIST:    GET    /products         — from base controller        ✅
  DETAIL:  GET    /products/:id     — from base controller        ✅
  UPDATE:  PATCH  /products/:id     — explicit route              ✅
  DELETE:  DELETE /products/:id     — from base controller        ✅
  EXTRA:   GET    /products/search  — explicit route              ✅
```

### Step 3: Service Layer Audit

For each controller method identified in Step 2, verify the corresponding service method exists and handles errors properly.

**Search for service files:**

Auto-detect based on framework:

| Framework | Glob Patterns |
|-----------|--------------|
| NestJS | `**/*.service.ts` |
| Express | `**/services/*.ts`, `**/services/*.js` |
| Spring Boot | `**/*Service.java`, `**/*ServiceImpl.java` |
| Django | `**/services.py`, `**/views.py` (business logic) |
| Laravel | `**/Services/*.php` |

**A) Base service detection:**
Check if service extends a base service class. If yes, standard methods may be inherited:
- `findAll(options?)` — list with pagination support
- `findOne(id)` — find by ID (should throw NotFoundException if not found)
- `create(dto)` — create new record
- `update(id, dto)` — update existing record (should throw NotFoundException if not found)
- `remove(id)` — soft delete (should throw NotFoundException if not found)
- `count(options?)` — count records

**B) Error handling verification:**
For each service method (whether inherited or explicit), check for:

| Method | Required Exception | Trigger |
|--------|--------------------|---------|
| `findOne` | `NotFoundException` | Record with given ID not found |
| `update` | `NotFoundException` | Record with given ID not found |
| `remove` | `NotFoundException` | Record with given ID not found |
| `create` | `ConflictException` | Unique constraint violation |
| Any | `BadRequestException` | Invalid input data |
| Any | `ForbiddenException` | User lacks permission |

Search patterns:
```
Grep: "NotFoundException" in each service file
Grep: "ConflictException" in each service file
Grep: "throw new" in each service file
```

**C) Transaction handling:**
For operations that modify multiple tables (e.g., creating a user and assigning roles), check for:
```
Grep: "transaction|@Transaction|queryRunner|manager.transaction" in service files
```

### Step 4: Repository Layer Audit

Check repository methods for completeness and correctness.

**Search for repository/data access files:**

Auto-detect based on ORM:

| ORM | Glob Patterns |
|-----|--------------|
| TypeORM | `**/*.repository.ts` |
| Prisma | service files using `prisma.model.*` |
| Sequelize | `**/models/*.ts` (includes query methods) |
| Django ORM | `**/managers.py`, model managers |
| SQLAlchemy | `**/repositories/*.py` |

**A) Base repository detection:**
Check if repository extends a base repository class. If yes, standard query methods are inherited.

**B) Pagination support:**
For any findAll/list method, check:
```
Grep: "skip|take|limit|offset|page" in repository files
```
If the repository's list method does NOT include skip/take or page/limit params, flag as missing pagination.

**C) Soft delete compliance:**
For delete operations, verify:
```
Grep: "softRemove|softDelete" in repository files — GOOD (soft delete)
Grep: "\.remove\(|\.delete\(" in repository files — POTENTIAL ISSUE (might be hard delete)
```
Cross-reference with entity: if entity has `deletedAt` column but repository uses `.delete()` or `.remove()` without soft prefix, flag as hard delete issue.

**D) Relation loading:**
Check if repository methods load necessary relations:
```
Grep: "relations:|leftJoinAndSelect|innerJoinAndSelect|loadRelationIds" in repository files
```

### Step 5: Frontend Action Inventory

For each backend resource, find corresponding frontend CRUD implementations.

**Search in frontend directories:**
```
For each auto-detected frontend directory:
Glob: {frontend-dir}/**/*.ts
Glob: {frontend-dir}/**/*.tsx
```

**A) HTTP service layer:**
Look for API call definitions:
```
Grep: "post\(|get\(|patch\(|put\(|delete\(" in {frontend-dir}/**/services/**
Grep: "axios\.|fetch\(|httpClient\." in {frontend-dir}/**/services/**
```

Map each API call to a backend endpoint.

**B) UI component layer:**
For each resource, check for:
- **Create form/modal** — search for form components that POST to the resource
- **List/table view** — search for components that GET the resource list
- **Detail view** — search for components that GET a single resource
- **Edit form/modal** — search for components that PATCH/PUT a resource
- **Delete button/confirmation** — search for components that DELETE a resource

Search patterns:
```
Grep: "create|add|new|save" combined with resource name in frontend components
Grep: "edit|update|modify" combined with resource name in frontend components
Grep: "delete|remove|destroy" combined with resource name in frontend components
Grep: "list|table|grid|index" combined with resource name in frontend components
```

**C) React Query / Redux coverage:**
Check for data fetching hooks:
```
Grep: "useQuery|useMutation|createAsyncThunk|createApi" in frontend files
```

Map to CRUD operations:
- `useQuery` with GET — Read operations
- `useMutation` with POST — Create operation
- `useMutation` with PATCH/PUT — Update operation
- `useMutation` with DELETE — Delete operation

### Check 1: Missing CRUD Operation (HIGH)

For each entity in the resource inventory, compare against the controller CRUD matrix.

**Classification logic:**
- If ALL 5 CRUD operations are missing (no controller at all):
  - Check if entity is a join table or embedded entity — **INFO** (intentional, no direct CRUD needed)
  - Check if entity is only managed through a parent resource — **INFO** (e.g., OrderItem managed via Order)
  - Otherwise — **HIGH** (missing controller)

- If entity has Create but no Read:
  - If entity is an event/log type (name contains Log, Event, Audit, History) — **INFO** (write-only is intentional)
  - Otherwise — **HIGH**

- If entity has Create + List but no Update or Delete:
  - If entity is immutable by design (surveys, logs) — **INFO**
  - Otherwise — **HIGH** (likely incomplete implementation)

- If entity has Create + List + Detail but no Delete:
  - Could be intentional (never delete records) — **MEDIUM**

- If entity is missing only Detail (GET /:id):
  - If list returns full objects — **LOW** (detail might be unnecessary)
  - If list returns summaries — **HIGH** (detail needed for full view)

Record each finding with:
- Resource name
- Operations present
- Operations missing
- Classification reasoning

### Check 2: Missing Pagination (HIGH)

For each list endpoint (GET / or equivalent):

**A) Controller level:**
```
Grep: "@Query.*page|@Query.*limit|@Query.*skip|@Query.*take|@Query.*offset" in the controller
```
Check if the list method accepts pagination parameters.

**B) Service/Repository level:**
If controller delegates to BaseService.findAll or BaseRepository.findAll, check if pagination is built in.
```
Grep: "skip.*take|page.*limit|offset.*limit" in the service/repository findAll method
```

**C) Response shape:**
Check if the list response includes pagination metadata:
```
Grep: "total|count|page|pages|hasNext|hasPrevious|meta" in the service return value
```

**Flag if:**
- List endpoint accepts no pagination params AND
- Service/repository has no skip/take AND
- Response has no pagination metadata
- This means ALL records are returned, which is a scalability risk

**Exception:** If the resource is bounded (e.g., roles, categories with <50 items), pagination may not be needed — **INFO**

### Check 3: Missing Search/Filter (MEDIUM)

For each list endpoint:
```
Grep: "search|keyword|query|filter|where|q=" in controller and service
Grep: "@Query.*search|@Query.*filter|@Query.*status|@Query.*type" in controller
```

Check for:
- Full-text search parameter (search, q, keyword)
- Status filter (status, active, enabled)
- Date range filter (startDate, endDate, from, to)
- Type/category filter (type, category, role)
- Relation filter (userId, createdById, ownerId)

**Flag if:**
- List endpoint has no filtering capability at all
- Resource is likely to have many records and users would need to filter

### Check 4: Missing Sort (LOW)

For each list endpoint:
```
Grep: "sort|order|orderBy|sortBy" in controller and service
Grep: "addOrderBy|orderBy" in repository QueryBuilder
```

Check for:
- Sort parameter in controller query params
- Default sort in repository (e.g., `orderBy('createdAt', 'DESC')`)

**Flag if:**
- No sort parameter available AND no default sort defined
- Lower priority since most lists have a reasonable default sort

### Check 5: Inconsistent Response Shape (MEDIUM)

Compare response patterns across ALL list endpoints.

**A) Collect response shapes:**
For each list endpoint, determine what shape is returned:
- `{ data: [], meta: { total, page, limit } }` — paginated wrapper
- `{ data: [], total: number }` — simple wrapper
- `[]` — raw array
- `{ items: [], count: number }` — alternative wrapper

**B) Compare across resources:**
All list endpoints should use the SAME wrapper pattern. Flag any that differ.

**C) Detail endpoint shapes:**
Similarly, compare GET /:id responses:
- `{ data: entity }` — wrapped
- `entity` — raw
- `{ ...entity, relations: {...} }` — flat with relations

All detail endpoints should be consistent.

**D) Error response shapes:**
Check if error responses follow a consistent pattern:
```
{ statusCode: number, message: string, error: string }
```
NestJS provides this by default, but custom exception filters might change it.

### Check 6: Missing Error Response (HIGH)

For each service method, verify proper exception handling.

**A) findOne / getById:**
```
Grep: "NotFoundException|not found|does not exist" in service findOne method
```
If no NotFoundException is thrown when record is null, flag as HIGH.

**B) create:**
```
Grep: "ConflictException|already exists|duplicate" in service create method
```
If entity has unique constraints but create doesn't handle ConflictException, flag as MEDIUM.

**C) update:**
Check for both NotFoundException (record not found) and ConflictException (unique violation on update).

**D) remove/delete:**
Check for NotFoundException before attempting delete.

**E) General validation:**
Check if DTOs use class-validator decorators:
```
Grep: "@IsString|@IsNumber|@IsEmail|@IsNotEmpty|@IsOptional|@IsEnum|@IsUUID" in DTO files
```
If DTO exists but has no validators, flag as MEDIUM.

### Check 7: No Soft Delete (MEDIUM)

For each entity that supports soft delete (extends BaseEntity or has `@DeleteDateColumn()`):

**A) Check delete implementation:**
```
Grep: "softRemove|softDelete" in service/repository delete method — CORRECT
Grep: "\.remove\(|\.delete\(|DELETE FROM" in service/repository — POTENTIAL HARD DELETE
```

**B) Check for restore capability:**
```
Grep: "restore|recover|undelete" in service and controller
```
If entity supports soft delete but has no restore endpoint, note as INFO (not always needed).

**C) Check query filtering:**
```
Grep: "withDeleted|includeDeleted" in repository
```
If soft-deleted records are never queryable (no withDeleted option), note as INFO.

### Check 8: Missing Bulk Operations (LOW)

For each resource, check for bulk operation support:

```
Grep: "bulk|batch|many|multiple" in controller and service
Grep: "Promise\.all|Promise\.allSettled|insert\(\[|save\(\[" in service
```

**Also check frontend for sequential loop API calls that should be bulk:**
```
Grep: "for.*of.*mutateAsync|for.*of.*await.*update|for.*of.*await.*delete|for.*of.*await.*service" in frontend components
```

Flag if frontend code loops through selected items and calls individual API endpoints one-by-one instead of using a single bulk endpoint. This causes N sequential HTTP requests instead of 1, resulting in poor UX (slow operation, no atomic error handling).

**Flag if:**
- Resource is commonly created/updated in batches (e.g., order items, notifications)
- No bulk endpoint exists
- Frontend uses sequential `for` loop with `await` to call individual CRUD endpoints for batch operations (e.g., bulk activate/deactivate users)
- This is LOW priority — only flag for resources where bulk ops make practical sense

Resources that typically need bulk operations:
- Items assigned to users (assign multiple items at once)
- Notifications (send to multiple users)
- User management (bulk status change, bulk role assignment)

### Check 9: Missing Ownership Check (HIGH)

For user-specific resources, verify that update/delete endpoints check ownership.

**A) Determine resource ownership:**
From Step 1 entity analysis, identify the owner field:
- `userId`, `ownerId`, `createdById`, `authorId` — direct ownership
- Through relation: `item.order.userId` — indirect ownership

**B) Check controller/service for ownership verification:**
```
Grep: "@CurrentUser|currentUser|req\.user" in controller
Grep: "userId.*===|ownerId.*===|\.userId\s*!==|\.createdById\s*!==|user\.id" in service
```

**C) Flag if:**
- Resource has an owner field (userId, ownerId, etc.)
- Update or Delete endpoint does NOT use `@CurrentUser()` to get requesting user
- Service does NOT compare requesting user ID with resource owner ID
- No `@Roles()` guard that restricts to admin-only

**Exception:** Admin endpoints that intentionally allow cross-user access should use role-based guards (e.g., `@Roles(Role.ADMIN)`) — verify the appropriate guard is present.

### Check 10: Frontend CRUD Gap (HIGH)

Match each backend endpoint to a frontend implementation.

**A) Build backend endpoint list:**
From Step 2, list all CRUD endpoints with their HTTP method and path.

**B) Search frontend for matching API calls:**
For each backend endpoint:
```
Grep: "endpoint-path|resource-name" in {frontend-dir}/**/services/**
```

**C) Search for UI components:**
For each CRUD operation:
- **Create**: Look for form components, "Add" or "New" buttons, create modals
- **List**: Look for table components, list views, index pages
- **Detail**: Look for detail/view pages, profile components
- **Update**: Look for edit forms, inline editing, update modals
- **Delete**: Look for delete buttons, confirmation dialogs

```
Grep: resource name in {frontend-dir}/**/pages/**
Grep: resource name in {frontend-dir}/**/components/**
```

**D) Classification:**
- Backend endpoint exists AND frontend action exists — **Covered** (checkmark)
- Backend endpoint exists but NO frontend action — **Gap** (X mark)
- Frontend action exists but calls non-existent backend endpoint — **Orphan** (warning)

### Check 11: Missing Status Toggle (MEDIUM)

For each entity, check if it has a status/active field and whether the full stack supports toggling it.

**A) Detect status columns in entities:**
```
Grep: "isActive|is_active|status|enabled|disabled|inactive" in entity files
Grep: "@Column.*boolean.*default" in entity files
```

Look for columns like:
- `isActive: boolean` (common active/inactive toggle)
- `status: enum` (e.g., ACTIVE, INACTIVE, SUSPENDED)
- `enabled: boolean`

**B) Check backend toggle endpoint:**
For each entity with a status column:
```
Grep: "activate|deactivate|toggle|status|isActive" in controller files
Grep: "@Patch.*activate|@Patch.*status|@Put.*status" in controller files
```

Check for:
- Dedicated toggle endpoint (e.g., `PATCH /items/:id/toggle-active`)
- Status update via general PATCH (included in UpdateDto)
- Bulk status change (e.g., `PATCH /items/bulk/status`)

**C) Check UpdateDto includes status field:**
```
Grep: "isActive|status|enabled" in update DTO files
```
If entity has `isActive` but UpdateDto does NOT include it, the field cannot be changed via API.

**D) Check frontend toggle UI:**
```
Grep: "isActive|active|inactive|toggle|switch|activate|deactivate" in frontend components
Grep: "Switch|Toggle|Checkbox.*active" in frontend components for the resource
```

Look for:
- Toggle switch or checkbox in list/table view
- Status badge that is clickable
- Activate/Deactivate button in detail or edit view
- Confirmation dialog before status change

**E) Classification:**

| Scenario | Severity | Description |
|----------|----------|-------------|
| Entity has isActive, no backend endpoint | **MEDIUM** | Status column exists but cannot be changed |
| Backend endpoint exists, no frontend UI | **MEDIUM** | Can change via API but no user-facing toggle |
| Frontend sends toggle but wrong HTTP method/path | **HIGH** | Toggle button exists but doesn't work |
| Entity has isActive, both backend + frontend work | **OK** | Fully covered |
| Entity has NO status column | **N/A** | Not applicable — skip |

**F) Additional checks:**
- If status column exists, does the List endpoint support filtering by status? (e.g., `?isActive=true`)
- If soft delete AND isActive both exist, clarify the semantic difference (isActive = hidden from users, deletedAt = removed from system)
- Check if toggling status has cascading effects (e.g., deactivating a parent resource should affect child resource visibility)

---

## Output Format

After completing all 10 checks, produce a single consolidated report in this exact format:

```markdown
# QA CRUD Audit Report

**Date**: YYYY-MM-DD
**Scope**: All modules | {specific module name}
**Resources Scanned**: {count}
**Total Issues Found**: {count}

---

## CRUD Matrix

| Resource | Create | List | Detail | Update | Delete | Soft Del | Status Toggle | Pagination | Search | Sort | Frontend |
|----------|--------|------|--------|--------|--------|----------|---------------|------------|--------|------|----------|
| Product | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| OrderItem | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | N/A | ✅ | ❌ | ✅ | Partial |
| User | ✅ | ✅ | ✅ | ✅ | ⚠️ hard | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Comment | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | N/A | ❌ | ❌ | ❌ | ✅ |

Legend: ✅ = Present | ❌ = Missing | ⚠️ = Issue | N/A = Not applicable

---

## Summary

| # | Check | Severity | Issues Found |
|---|-------|----------|--------------|
| 1 | Missing CRUD Operation | HIGH | X |
| 2 | Missing Pagination | HIGH | X |
| 3 | Missing Search/Filter | MEDIUM | X |
| 4 | Missing Sort | LOW | X |
| 5 | Inconsistent Response Shape | MEDIUM | X |
| 6 | Missing Error Response | HIGH | X |
| 7 | No Soft Delete | MEDIUM | X |
| 8 | Missing Bulk Operations | LOW | X |
| 9 | Missing Ownership Check | HIGH | X |
| 10 | Frontend CRUD Gap | HIGH | X |
| 11 | Missing Status Toggle | MEDIUM | X |
| | **Total** | | **X** |

---

## HIGH Severity Issues

### Check 1: Missing CRUD Operations

| Resource | Present | Missing | Notes |
|----------|---------|---------|-------|
| OrderItem | Create, List | Detail, Update, Delete | Items may be intentionally immutable |
| ... | ... | ... | ... |

**Risk**: Incomplete CRUD means users cannot fully manage the resource. Missing Delete may leave orphan data.
**Fix**: Add the missing controller methods (or verify the omission is intentional and document why).

### Check 2: Missing Pagination

| Resource | List Endpoint | Pagination | Risk |
|----------|---------------|------------|------|
| Comment | GET /comments | ❌ None | Returns all records — will degrade with scale |
| ... | ... | ... | ... |

**Risk**: Unbounded list queries become slower as data grows. Can cause OOM or timeout in production.
**Fix**: Add page/limit query params to the controller and skip/take to the repository query.

### Check 6: Missing Error Response

| Resource | Method | Missing Exception | Risk |
|----------|--------|-------------------|------|
| Product | findOne | NotFoundException | Returns null instead of 404 |
| User | create | ConflictException | Duplicate username returns 500 |
| ... | ... | ... | ... |

**Risk**: Missing NotFoundException returns null/200 instead of 404 — confusing for frontend. Missing ConflictException returns raw 500 — bad UX.
**Fix**: Add proper exception throwing in service methods before DB operations.

### Check 9: Missing Ownership Check

| Resource | Endpoint | Owner Field | Has Check | Guard |
|----------|----------|-------------|-----------|-------|
| OrderItem | PATCH /order-items/:id | userId | ❌ | None |
| Comment | DELETE /comments/:id | authorId | ❌ | None |
| ... | ... | ... | ... | ... |

**Risk**: Any authenticated user can modify/delete another user's data by guessing IDs. Privacy and security violation.
**Fix**: Compare `@CurrentUser().id` with `resource.userId` in service and throw `ForbiddenException` on mismatch.

### Check 10: Frontend CRUD Gaps

| Resource | Backend Endpoint | Frontend Status | Details |
|----------|------------------|-----------------|---------|
| Product | DELETE /products/:id | ❌ Missing | No delete button in UI |
| Order | PATCH /orders/:id | ❌ Missing | No edit form exists |
| ... | ... | ... | ... |

**Risk**: Backend capability exists but users can't access it through the UI. Dead code or incomplete feature.
**Fix**: Add the corresponding UI component (button, form, page) to the frontend app.

---

## MEDIUM Severity Issues

### Check 3: Missing Search/Filter

| Resource | List Endpoint | Has Search | Has Filters | Suggested Filters |
|----------|---------------|------------|-------------|-------------------|
| OrderItem | GET /order-items | ❌ | ❌ | date range, user, order |
| ... | ... | ... | ... | ... |

**Risk**: Users must scroll through all records to find what they need. Poor UX on large datasets.
**Fix**: Add @Query() search/filter params and implement WHERE clauses in the repository.

### Check 5: Inconsistent Response Shape

| Resource | Endpoint | Current Shape | Expected Shape | Issue |
|----------|----------|---------------|----------------|-------|
| Product | GET /products | { data: [], meta } | — | Baseline |
| Comment | GET /comments | [] (raw array) | { data: [], meta } | Not wrapped |
| ... | ... | ... | ... | ... |

**Risk**: Frontend must handle different response shapes per endpoint. Increases bug likelihood and maintenance cost.
**Fix**: Standardize all list endpoints to use `{ data: [], meta: { total, page, limit } }` wrapper.

### Check 11: Missing Status Toggle

| Resource | Status Column | Backend Toggle | UpdateDto | Frontend UI | Issue |
|----------|---------------|----------------|-----------|-------------|-------|
| Product | isActive (boolean) | PATCH /products/:id | ✅ included | ❌ No toggle | Frontend missing |
| User | isActive (boolean) | ❌ No endpoint | ❌ Not in DTO | ❌ No toggle | Full stack missing |
| ... | ... | ... | ... | ... | ... |

**Risk**: Status column exists in entity but users cannot change it. Records are either always active or require direct DB manipulation.
**Fix**: Add status field to UpdateDto, ensure PATCH endpoint accepts it, and add toggle UI (switch/checkbox) in the frontend list or detail view.

### Check 7: Soft Delete Issues

| Resource | Has deletedAt | Delete Method | Issue |
|----------|---------------|---------------|-------|
| User | ✅ | .remove() | Uses hard delete despite soft delete support |
| ... | ... | ... | ... |

**Risk**: Hard-deleted records are permanently lost. Cannot audit or restore accidentally deleted data.
**Fix**: Replace `.remove()` / `.delete()` with `.softRemove()` / `.softDelete()` in the service/repository.

---

## LOW Severity Issues

### Check 4: Missing Sort

| Resource | List Endpoint | Default Sort | Custom Sort Param |
|----------|---------------|--------------|-------------------|
| Notification | GET /notifications | ❌ None | ❌ |
| ... | ... | ... | ... |

**Risk**: List order may be unpredictable. Users expect newest-first or alphabetical by default.
**Fix**: Add `.orderBy('createdAt', 'DESC')` as default sort in repository.

### Check 8: Missing Bulk Operations

| Resource | Likely Needs Bulk | Current Support | Suggestion |
|----------|-------------------|-----------------|------------|
| Product | Yes (batch import) | ❌ | POST /products/bulk for batch creation |
| Notification | Yes (broadcast) | ❌ | POST /notifications/bulk for mass notify |
| ... | ... | ... | ... |

**Risk**: Users must create items one-by-one. Tedious for batch operations.
**Fix**: Add bulk endpoint that accepts array of items. Use `Promise.allSettled()` for partial success handling.

---

## Recommendations

1. **[Priority 1]** Fix all HIGH severity issues first — missing error handling and ownership checks are security risks.
2. **[Priority 2]** Add pagination to all list endpoints returning unbounded results.
3. **[Priority 3]** Standardize response shapes across all endpoints.
4. **[Priority 4]** Fill frontend CRUD gaps for complete user workflows.
5. **[Priority 5]** Add search/filter to high-volume resources.
```

## Scanning Patterns Reference

| What | Regex/Glob Pattern (adapt per framework) | Where to Search |
|------|--------------------|-----------------|
| Entity/model classes | `@Entity\(` (TypeORM), `model` (Prisma), `class.*Model` (Django) | auto-detected backend entity files |
| Controllers/routes | `@Controller\(` (NestJS), `router\.\|Router()` (Express), `@RestController` (Spring) | auto-detected controller files |
| Base class usage | `extends Base` | controller/service/repository files |
| HTTP method decorators | `@Get\|@Post\|@Put\|@Patch\|@Delete` (NestJS), `router\.get\|router\.post` (Express) | controller files |
| Route params | `@Param\(` (NestJS), `req\.params` (Express), `@PathVariable` (Spring) | controller files |
| Query params | `@Query\(` (NestJS), `req\.query` (Express), `@RequestParam` (Spring) | controller files |
| Pagination params | `page\|limit\|skip\|take\|offset` | controller/service/repository files |
| Search params | `search\|keyword\|query\|filter\|q=` | controller files |
| Sort params | `sort\|order\|orderBy\|sortBy` | controller/service files |
| NotFoundException | `NotFoundException\|throw new Not` | service files |
| ConflictException | `ConflictException\|already exists\|duplicate` | service files |
| Soft delete methods | `softRemove\|softDelete` | service/repository files |
| Hard delete methods | `\.remove\(\|\.delete\(\|DELETE FROM` | service/repository files |
| CurrentUser decorator | `@CurrentUser\(\)` | controller files |
| Roles guard | `@Roles\(` | controller files |
| Class validators | `@IsString\|@IsNumber\|@IsNotEmpty\|@IsOptional\|@IsEnum\|@IsUUID` | DTO files |
| Frontend API calls | `\.get\(\|\.post\(\|\.patch\(\|\.put\(\|\.delete\(` | frontend services |
| React Query hooks | `useQuery\|useMutation` | frontend hooks/components |
| Redux async thunks | `createAsyncThunk\|createApi` | frontend redux |
| Frontend forms | `onSubmit\|handleSubmit\|useForm` | frontend components |
| Delete confirmations | `confirm\|dialog\|modal.*delete` | frontend components |

## Edge Cases to Watch For

1. **Base controller inherited methods**: When a controller extends a base controller, standard CRUD methods may be inherited. Class-level decorators/middleware on the child controller may or may not apply to inherited methods depending on the framework. Verify for non-standard base class setups.

2. **Service method overrides**: A service that extends `BaseService` may override `findAll()` to add custom filtering. The overridden method may lose built-in pagination support — check the override implementation.

3. **Nested resources**: Some resources are accessed via nested routes (e.g., `/users/:userId/orders`). These are valid CRUD but won't match the standard pattern. Map them manually.

4. **Multi-entity operations**: Creating a parent record may create child records in one transaction. The "Create" operation for the parent covers the children — don't flag children as missing Create.

5. **Soft delete with cascades**: When a parent entity is soft-deleted, child entities may need cascade soft-delete too. Check if deleting a parent also soft-deletes its child records.

6. **Dual frontend coverage**: Some resources exist in multiple frontend apps (different user roles see resources differently). Check all auto-detected frontend apps for coverage.

7. **Paginated vs. unpaginated by design**: Chat messages may intentionally load all messages in a room (with cursor/infinite scroll instead of page/limit). Don't flag cursor-based pagination as missing.

## Important Notes

- **BaseController/BaseService inheritance**: When a controller extends BaseController or a service extends BaseService, standard CRUD methods are automatically provided. Do NOT flag these as missing — only flag if the base class itself lacks the method.
- **Intentionally partial CRUD**: Some resources are designed with limited CRUD:
  - Audit logs / event logs — CREATE + LIST only (immutable records)
  - System config — READ + UPDATE only (no create/delete)
  - Join tables — managed through parent resources, no direct CRUD
- **Ownership vs. role-based access**: Some endpoints use role-based guards instead of ownership checks. This is valid — admins should be able to manage all resources. Flag only when NEITHER ownership check NOR role guard is present.
- **Pagination threshold**: Resources expected to have fewer than ~50 records (roles, categories, system settings) may not need pagination. Use judgment.
- **When auditing a single module**, still check frontend coverage for that module's resources across all frontend directories.
- **Check all auto-detected frontend directories** for full coverage across apps.

## READ-ONLY Diagnostic

This skill is a READ-ONLY diagnostic. It does NOT modify any files. It scans, analyzes, and reports findings. All recommended fixes are presented as suggestions in the report for the developer to implement.
