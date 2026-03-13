---
name: qa-api-sync
description: Audit frontend API calls against backend endpoints for parameter, route, and response mismatches
user-invocable: true
argument-hint: "[module]"
---

# QA API Sync - Frontend↔Backend API Integration Auditor

## Purpose

Detect mismatches between frontend API service calls and backend controller endpoints. Finds missing parameters, wrong HTTP methods, mismatched routes, unused endpoints, and response type inconsistencies.

This skill performs a comprehensive, read-only audit that compares every frontend API call against the corresponding backend controller endpoint. It identifies integration bugs before they reach production by systematically checking routes, methods, parameters, response shapes, error handling, authentication, and content types.

This is a **READ-ONLY** diagnostic. It does NOT modify any files. It does NOT create any files.

## Usage

```
/qa-api-sync                  # Full audit (all modules)
/qa-api-sync users            # Single module only
```

When a module argument is provided, only scan the specified module's backend controller(s) and the corresponding frontend service file(s). When no argument is provided, scan all modules.

## 11 Checks

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

---

## Execution Algorithm

You MUST follow every step below in order. Do not skip steps. Do not modify any files. This is a READ-ONLY diagnostic.

---

### Step 0: Context & Requirements Gathering (MANDATORY)

Before auditing, you MUST understand what features are actually in scope for this project. Auditing code that exists but is intentionally unused/deprecated leads to false positives.

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
Glob: .claude-project/docs/PROJECT_KNOWLEDGE.md
```

Extract:
- Which features are actively used vs planned/deprecated
- Which modules are in production
- Supported authentication methods (e.g., "email login only" → social login endpoints are out of scope)
- Which frontend apps exist and their purpose

**0b. Read sprint/feature requirements if available:**
```
Glob: .claude-project/requirements/features/*.md
```

**0c. Ask clarifying questions BEFORE proceeding:**

If you find backend endpoints for features not mentioned in requirements (e.g., OTP, social login, FCM), you MUST ask the user:

> "I found endpoints for [feature X] but it's not mentioned in the project requirements. Should I include it in the audit or skip it as unused/planned code?"

Wait for the user's response before proceeding. Do NOT assume unused code is a bug.

**0d. Build the In-Scope Feature List:**

Create a clear list:
```
IN SCOPE: [feature1, feature2, ...]
OUT OF SCOPE: [feature3 (reason), feature4 (reason), ...]
```

Only audit in-scope features. Out-of-scope endpoints should be listed in Check 7 (Unused) as "Intentionally out of scope" rather than flagged as issues.

---

### Step 1: Determine Scope

If a module argument was provided:
- Set `TARGET_MODULE` to the argument value (e.g., "users", "auth", "products", "orders")
- Only scan controllers and services related to that module

If no argument was provided:
- Set `TARGET_MODULE` to "all"
- Scan every module

---

### Step 2: Check for Global API Prefix

Read the backend entry point to determine if a global prefix is configured:

```
Auto-detect backend entry point:
Glob: **/main.ts, **/index.ts, **/app.ts, **/server.ts, **/manage.py, **/urls.py
```

### Backend Framework Detection

| Framework | Detection Signal | Entry Point Pattern | Prefix Config |
|-----------|-----------------|--------------------|----|
| NestJS | `@nestjs/core` in package.json | `main.ts` | `app.setGlobalPrefix()` |
| Express | `express` in package.json | `index.ts`, `app.ts`, `server.ts` | `app.use('/api', router)` |
| Spring Boot | `spring-boot` in build.gradle/pom.xml | `Application.java` | `server.servlet.context-path` |
| Django | `django` in requirements.txt | `urls.py` | `urlpatterns` prefix |
| Laravel | `laravel/framework` in composer.json | `routes/api.php` | `RouteServiceProvider` prefix |

Look for:
- Global API prefix configuration (per framework)
- API versioning configuration
- Any global middleware that modifies routes
- Validation configuration (e.g., NestJS `whitelist: true` in ValidationPipe — needed for Check 5)

Store the global prefix (e.g., "/api", "/api/v1", or empty string) for path normalization in later steps.

Also check the frontend for how the base URL is configured:

```
Grep: baseURL|BASE_URL|API_URL|VITE_API_URL
Files: auto-detected frontend directories — **/*.ts, **/*.tsx, **/.env*
```

Determine the effective base URL the frontend uses when making API calls. This is critical for path matching.

---

### Step 3: Backend Route Inventory

Scan all controller files to build a complete route inventory.

**3a. Find all controller/route files:**

Auto-detect based on backend framework:

| Framework | Glob Patterns |
|-----------|--------------|
| NestJS | `**/*.controller.ts` |
| Express | `**/routes/*.ts`, `**/router/*.ts`, `**/controllers/*.ts` |
| Spring Boot | `**/*Controller.java`, `**/*Resource.java` |
| Django | `**/views.py`, `**/viewsets.py`, `**/urls.py` |
| Laravel | `**/Controllers/*.php`, `routes/api.php` |

If `TARGET_MODULE` is not "all", filter to only controllers in the target module directory.

**3b. For each controller file, extract:**

Read each controller file completely. Then extract the following information based on the detected framework:

1. **Class-level route prefix:**

   | Framework | Pattern |
   |-----------|---------|
   | NestJS | `@Controller('prefix')` |
   | Express | `router = Router()` + `app.use('/prefix', router)` |
   | Spring Boot | `@RequestMapping("/prefix")` |
   | Django | `urlpatterns` path prefix |
   | Laravel | `Route::prefix('prefix')` |

2. **Whether the controller extends a base/generic controller**: If so, it may automatically inherit standard CRUD endpoints. Read the base controller source to confirm the exact methods and decorators it provides.

3. **Each method's HTTP decorator and route path:**

   | Framework | GET | POST | PUT | PATCH | DELETE |
   |-----------|-----|------|-----|-------|--------|
   | NestJS | `@Get()` | `@Post()` | `@Put()` | `@Patch()` | `@Delete()` |
   | Express | `router.get()` | `router.post()` | `router.put()` | `router.patch()` | `router.delete()` |
   | Spring Boot | `@GetMapping` | `@PostMapping` | `@PutMapping` | `@PatchMapping` | `@DeleteMapping` |
   | Django | `@api_view(['GET'])` | `@api_view(['POST'])` | viewset actions | viewset actions | viewset actions |
   | Laravel | `Route::get()` | `Route::post()` | `Route::put()` | `Route::patch()` | `Route::delete()` |

4. **Parameter extraction** (adapt per framework):
   - Body/request parameters and their DTO/schema types
   - Query parameters
   - Path parameters

5. **Security/auth decorators** (adapt per framework):

   | Framework | Public | Role-based | Guard/Middleware |
   |-----------|--------|-----------|-----------------|
   | NestJS | `@Public()` | `@Roles(...)` | `@UseGuards(...)` |
   | Express | no auth middleware | role middleware | `authenticate` middleware |
   | Spring Boot | `permitAll()` | `@PreAuthorize` | `@Secured` |
   | Django | `AllowAny` | `@permission_classes(...)` | `IsAuthenticated` |
   | Laravel | no `auth` middleware | `middleware('role:...')` | `middleware('auth')` |

6. **File upload handling** (adapt per framework)

7. **Return type**: Check the method signature and any explicit return type annotation. Also check `@ApiResponse` or `@ApiOkResponse` decorators for documented return types.

**3c. Build the route inventory as a structured list:**

For each endpoint, record:
```
{
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE',
  controllerPrefix: string,
  methodPath: string,
  fullPath: string,          // globalPrefix + '/' + controllerPrefix + '/' + methodPath
  bodyDto: string | null,    // DTO type name if @Body() is used
  queryParams: string[],     // list of query parameter names
  pathParams: string[],      // list of path parameter names
  guards: string[],          // guard class names
  roles: string[],           // role values
  isPublic: boolean,         // whether @Public() is present
  expectsFileUpload: boolean,// whether FileInterceptor is used
  returnType: string | null, // return type if detectable
  file: string,              // source file path
  line: number,              // approximate line number
  methodName: string,        // method name in the controller class
  isInherited: boolean       // whether from BaseController
}
```

**3d. Read DTO files for body parameters:**

For each unique `bodyDto` type found, locate and read the DTO file:

```
Grep: class {DtoTypeName}
Files: auto-detected backend directories — **/*.dto.ts, **/dto/*.ts, **/dtos/*.ts
```

From each DTO, extract:
- All property names
- Which properties have `@IsNotEmpty()` or `@IsDefined()` (required fields)
- Which properties have `@IsOptional()` (optional fields)
- Property types (string, number, boolean, enum, nested objects)
- Whether the DTO uses `@Allow()` or whitelist configuration
- Any validation constraints (`@MinLength`, `@MaxLength`, `@IsEnum`, etc.)

Also check if the DTO extends another DTO (NestJS/TypeORM pattern):
- `PartialType(CreateDto)` → all fields become optional
- `PickType(CreateDto, ['field1', 'field2'])` → only listed fields
- `OmitType(CreateDto, ['id'])` → all fields except listed ones
- `IntersectionType(DtoA, DtoB)` → merge fields from both

Resolve the full field list after inheritance.

Store the DTO field inventory:
```
{
  dtoName: 'CreateItemDto',
  requiredFields: ['name', 'description'],
  optionalFields: ['category', 'tags'],
  allFields: ['name', 'description', 'category', 'tags'],
  fieldTypes: { name: 'string', category: 'string', ... },
  file: string,
  extendsFrom: string | null
}
```

---

### Step 4: Frontend Service Inventory

Scan frontend API service files to build a complete call inventory.

**4a. Find all frontend service files:**

```
Auto-detect frontend service directories:
Glob: {frontend-dir}/**/services/**/*.ts
Glob: {frontend-dir}/**/services/**/*.tsx
(scan all detected frontend app directories)
```

If `TARGET_MODULE` is not "all", filter to service files that match the target module name (e.g., `userService.ts`, `productService.ts`). Be flexible with naming conventions: the service file might be named differently than the module.

**4b. Also scan for direct API calls in components and hooks:**

```
Grep: axios\.(get|post|put|patch|delete)|fetch\(|httpClient\.|api\.(get|post|put|patch|delete)|\.get\(|\.post\(|\.put\(|\.patch\(|\.delete\(
Files: auto-detected frontend directories — **/*.ts, **/*.tsx
```

This catches API calls made outside of service files (directly in components, hooks, or utils).

**4c. For each API call found, extract:**

Read each service file completely. Then extract:

1. **HTTP method**: From the axios/fetch method name
   ```
   api.get(...)    → GET
   api.post(...)   → POST
   api.put(...)    → PUT
   api.patch(...)  → PATCH
   api.delete(...) → DELETE
   fetch(url, { method: 'POST' }) → POST
   ```

2. **URL/path**: Resolve the full path
   ```
   api.get('/exercises')                    → '/exercises'
   api.get(`/exercises/${id}`)              → '/exercises/:id'
   api.get(`/exercises/${id}/logs`)         → '/exercises/:id/logs'
   fetch(`${API_URL}/exercises`)            → '/exercises'
   ```

   Handle template literals by converting `${variableName}` to `:paramName` for matching.

3. **Query parameters**: From params object
   ```
   api.get('/exercises', { params: { page, limit, search } })
   → queryParams: ['page', 'limit', 'search']
   ```

   Also check for URL query string construction:
   ```
   api.get(`/exercises?page=${page}&limit=${limit}`)
   → queryParams: ['page', 'limit']
   ```

4. **Request body fields**: From the data/body argument
   ```
   api.post('/exercises', { title, description, videoUrl, sets, reps })
   → bodyFields: ['title', 'description', 'videoUrl', 'sets', 'reps']

   api.post('/exercises', data)  // where data is typed as CreateExerciseDto
   → need to check the type definition to resolve fields
   ```

   If the body is passed as a variable, trace back to its type definition to determine fields.

5. **Response type**: From TypeScript generic or variable type
   ```
   api.get<Exercise[]>('/exercises')  → responseType: 'Exercise[]'
   const response: AxiosResponse<Exercise> = await api.get(...)  → responseType: 'Exercise'
   ```

6. **Error handling**: Check if the call is wrapped in error handling
   ```
   try { ... api.get(...) ... } catch  → hasErrorHandling: true
   api.get(...).catch(...)              → hasErrorHandling: true
   // In React Query with onError      → hasErrorHandling: true
   ```

7. **FormData usage**: Check if request uses FormData
   ```
   const formData = new FormData();
   api.post('/upload', formData)  → usesFormData: true
   ```

8. **Auth headers**: Check if auth is included
   ```
   // Check axios interceptors for automatic auth header injection
   // Check for manual Authorization header
   headers: { Authorization: `Bearer ${token}` }
   // Check if using httpOnly cookies (automatic, no header needed)
   withCredentials: true
   ```

**4d. Build the call inventory:**

For each API call, record:
```
{
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE',
  path: string,              // normalized path
  resolvedPath: string,      // path with base URL resolved
  bodyFields: string[],      // fields sent in body
  queryParams: string[],     // query parameter names
  responseType: string | null,
  hasErrorHandling: boolean,
  hasAuth: boolean,          // whether auth header/cookie is included
  usesFormData: boolean,
  file: string,              // source file path
  line: number,              // line number
  functionName: string,      // enclosing function/method name
  app: string                // auto-detected frontend app name
}
```

**4e. Check frontend TypeScript interfaces/types:**

Locate the frontend type definitions that correspond to backend entities:

```
Glob: {frontend-dir}/**/types/**/*.ts
Glob: {frontend-dir}/**/interfaces/**/*.ts
Glob: {frontend-dir}/**/models/**/*.ts
Grep: (interface|type)\s+\w+
Files: auto-detected frontend directories — **/*.ts
```

Read these type files and extract field names and types for comparison against backend entity/DTO shapes.

---

### Step 5: Path Normalization & Matching

Match each frontend API call to a backend endpoint.

**5a. Normalize paths:**

For both frontend calls and backend endpoints:
1. Remove trailing slashes: `/exercises/` → `/exercises`
2. Remove the global prefix if the frontend base URL already includes it
3. Lowercase the path for comparison (routes are case-insensitive in most frameworks)
4. Convert path parameter syntax:
   - Backend: `:id`, `:exerciseId`, `:userId`
   - Frontend: `${id}`, `${exerciseId}`, `${userId}`
   - Normalized: `:param` (any path segment that is a parameter becomes a generic placeholder)

**5b. Match algorithm:**

For each frontend call:
1. Find all backend endpoints with the same HTTP method
2. Compare normalized paths segment by segment
3. Path parameters match any value (`:id` matches `${exerciseId}`)
4. Score matches: exact path segment match > parameter match
5. If exactly one match found → paired
6. If multiple matches found → pick best match, flag ambiguity
7. If no match found → route mismatch (Check 1)

For each backend endpoint:
1. If no frontend call matches → unused endpoint (Check 7)

---

### Step 6: Execute Check 1 — Route Mismatches (CRITICAL)

For each frontend API call that has NO matching backend endpoint:

Report:
- Frontend file path and line number
- Function/method name
- HTTP method and URL path
- Closest backend endpoint (if any similar path exists, suggest it as a possible match)

Common causes to note:
- Typo in route path
- Missing controller endpoint (endpoint was deleted or never created)
- Wrong module path (e.g., `/exercise` vs `/exercises`, singular vs plural)
- API prefix mismatch (e.g., frontend includes `/api` but backend global prefix is different)
- Version prefix mismatch

---

### Step 7: Execute Check 2 — HTTP Method Mismatches (CRITICAL)

For each frontend API call where the path matches a backend endpoint but the HTTP method differs:

Report:
- Frontend: method + path + file:line
- Backend: expected method + controller file:line
- Common mistake patterns:
  - PUT vs PATCH (update operations — PATCH for partial, PUT for full replacement)
  - POST vs PUT (create vs replace)
  - POST vs PATCH (create vs partial update)
  - GET vs POST (for complex queries/search with body)

---

### Step 8: Execute Check 3 — Missing Query Params (HIGH)

For each matched endpoint pair where the backend accepts query parameters:

Compare backend `@Query()` parameters against frontend query params.

Focus on these commonly missed parameter categories:
- **Pagination**: page, limit, skip, take, offset, per_page, pageSize
- **Search**: search, q, keyword, query
- **Filtering**: status, type, role, category, dateFrom, dateTo, startDate, endDate
- **Sorting**: sort, sortBy, order, orderBy, sortOrder, sortDirection

Report only when backend declares query params that frontend never sends.
Do NOT report optional query params as issues if the frontend intentionally omits them — but DO flag when pagination params are available but unused (likely a bug that causes unbounded queries).

---

### Step 9: Execute Check 4 — Missing Body Params (HIGH)

For each matched POST/PUT/PATCH endpoint pair where the backend has a body DTO:

Compare DTO required fields against frontend body fields.

1. Read the DTO file to get required fields (those with `@IsNotEmpty()`, `@IsDefined()`, or not marked `@IsOptional()`)
2. For update DTOs based on `PartialType()`, all fields are optional — note this and adjust accordingly
3. Check if the frontend sends all required fields

**4a. Validation Constraint Comparison (sub-check):**

Beyond field presence, compare validation constraints between backend class-validator decorators and frontend Zod schemas (or inline validation). For each matched DTO field, check:

| Backend Decorator | Frontend Zod Equivalent | What to Compare |
|-------------------|------------------------|-----------------|
| `@MinLength(n)` | `z.string().min(n)` | min value must match |
| `@MaxLength(n)` | `z.string().max(n)` | max value must match |
| `@Min(n)` / `@Max(n)` | `z.number().min(n).max(n)` | range must match |
| `@Matches(regex)` | `z.string().regex(regex)` | pattern must match |
| `@IsEnum(Enum)` | `z.enum([...])` | allowed values must match |
| `@IsUUID()` | `z.string().uuid()` | format must match |
| `@IsDateString()` | `z.string()` + HTML `type="date"` | format coverage |

Find the corresponding Zod schema by searching for the form component that calls the service method:
```
Grep: useForm|zodResolver|z.object
Files: auto-detected frontend directories — **/*.tsx
```

Report mismatches where frontend validation is **less strict** than backend (allows values that backend will reject with 400). Severity: **Medium** (server catches it, but poor UX).

Do NOT flag cases where frontend is stricter (that's fine — it prevents unnecessary API calls).

Report:
- Endpoint path and method
- DTO name and file path
- Missing required field names
- Frontend file:line
- Validation constraint mismatches (if any)

Do NOT flag optional fields that frontend doesn't send — that is expected behavior.

---

### Step 10: Execute Check 5 — Extra Body Params (MEDIUM)

For each matched POST/PUT/PATCH endpoint pair:

Check if the frontend sends fields that are NOT in the backend DTO.

1. Get all DTO fields (required + optional)
2. Compare against frontend body fields
3. If frontend sends fields not in DTO → extra params

**5a. Detect `Partial<Entity>` anti-pattern:**

Search for frontend service methods that use `Partial<EntityType>` as the body parameter type instead of a dedicated request DTO. This pattern is dangerous because:
- Entity types include read-only fields (`id`, `createdAt`, `updatedAt`, `deletedAt`)
- Entity types may include fields explicitly omitted from the backend DTO (e.g., `password` omitted from UpdateUserDto)
- With `forbidNonWhitelisted: true`, sending these fields causes 400 errors

```
Grep: Partial<\w+>\s*\)
Files: auto-detected frontend directories — **/services/**/*.ts
```

Flag each `Partial<Entity>` usage with severity **High** and recommend replacing with a dedicated request interface.

**5b. Cross-app consistency check:**

When one frontend app has been fixed to use proper DTO types, check if other frontend apps have the same issue for the same endpoint. Report both together.

Report:
- Extra field names
- Whether backend uses class-validator whitelist mode (check for `whitelist: true` in ValidationPipe configuration in main.ts)
- If whitelist is enabled: fields are silently stripped (data loss — lower severity but still noteworthy)
- If whitelist is NOT enabled: potential mass assignment vulnerability (higher severity)
- `Partial<Entity>` usages (if any) with recommendation to use dedicated DTO types

---

### Step 11: Execute Check 6 — Response Type Mismatches (MEDIUM)

For each matched endpoint pair where both sides have type information:

Compare backend return type/shape against frontend TypeScript interface.

Check for:
1. **Missing fields**: Backend returns fields that frontend type doesn't declare
2. **Extra fields**: Frontend type declares fields that backend doesn't return
3. **Type differences**: Backend returns number but frontend expects string, Date vs string, etc.
4. **Array vs object**: Backend returns array but frontend expects single object, or vice versa
5. **Pagination wrapper**: Backend wraps response in `{ data: [], meta: { total, page, limit } }` but frontend doesn't account for wrapper, or frontend expects wrapper but backend returns raw array
6. **Nested object differences**: Backend nests related entities (e.g., `user: { id, name }`) but frontend has flat structure (e.g., `userId: string`)
7. **Enum mismatches**: Backend uses string enum but frontend uses numeric or different values
8. **Type precision**: Backend returns a specific enum type (e.g., `ExerciseUnitEnum` with values `'REPS' | 'SECONDS'`) but frontend uses a loose type (e.g., `string | null`). Compare across all frontend apps — one app may have the correct union type while another uses `string`. Flag loose types as **Low** severity when the frontend only reads the value (no writes), **Medium** when it's used in comparisons or conditionals

This check requires reading both:
- Backend entity/response DTO files
- Frontend TypeScript interface/type files

Report differences with field-by-field comparison where possible.

---

### Step 12: Execute Check 7 — Unused Backend Endpoints (INFO)

For each backend endpoint that has NO matching frontend call:

Classify the endpoint:
- **Intentionally backend-only**: health checks, webhooks, cron endpoints, internal APIs, metrics, OAuth callbacks → INFO, note but do not flag as errors
- **Likely should have frontend**: CRUD operations, user-facing features with no UI → MEDIUM, report

Heuristics for "intentionally backend-only":
- Path contains: health, webhook, cron, internal, metrics, callback, status
- Endpoint is in auth module and handles OAuth callbacks or token refresh
- Endpoint is a WebSocket gateway (not REST)
- Endpoint is an admin/debug endpoint

Report:
- Endpoint method + path
- Controller file:line
- Classification (intentional vs likely missing frontend)

---

### Step 13: Execute Check 8 — Missing Error Handling (MEDIUM)

For each frontend API call:

Check if the call has proper error handling:

1. **try/catch block**: The API call is inside a try block with a corresponding catch
2. **.catch() method**: The promise has a .catch() handler
3. **React Query onError**: If using useQuery/useMutation, check for onError callback or error handling in the calling component
4. **Error boundary**: Component wrapping (harder to detect, note as caveat in report)
5. **Global interceptor**: Check if axios has a response error interceptor (if so, all calls have basic error handling)

**First**, check for global axios error interceptor:
```
Grep: axios\.interceptors\.response\.use|interceptors\.response
Files: auto-detected frontend directories — **/*.ts
```

If a global interceptor exists that handles errors, note this in the report and reduce severity of individual missing handlers to INFO. Individual try/catch may still be needed for specific error recovery logic, but baseline handling is covered.

Report calls without any error handling mechanism.

---

### Step 14: Execute Check 9 — Auth Header Missing (HIGH)

For each matched endpoint pair where the backend endpoint is NOT marked as public (e.g., NestJS `@Public()`, Express no auth middleware, Django `AllowAny`, Spring `permitAll()`):

Verify the frontend includes authentication:

1. **Axios interceptor**: Check for global request interceptor that adds Authorization header
   ```
   Grep: interceptors\.request\.use|Authorization|Bearer
   Files: auto-detected frontend directories — **/*.ts
   ```

2. **HTTP-only cookies**: Check for `withCredentials: true` in axios config or axios.create()
   ```
   Grep: withCredentials
   Files: auto-detected frontend directories — **/*.ts
   ```

3. **Manual header**: Check for Authorization header in individual requests

If the frontend has a global auth mechanism (interceptor or withCredentials), all non-public endpoints are covered → report as "globally handled" and skip individual checks.

If NO global auth mechanism exists, flag each call to a protected endpoint that doesn't include auth — this would mean the request will be rejected with 401.

---

### Step 15: Execute Check 10 — Content-Type Mismatch (MEDIUM)

For each matched endpoint pair:

Check for content type mismatches:

1. **Backend expects file upload** (has FileInterceptor/FilesInterceptor) but **frontend sends JSON** (no FormData)
2. **Backend expects JSON** (standard @Body()) but **frontend sends FormData**
3. **Frontend sets wrong Content-Type header** explicitly (e.g., `'application/json'` when it should be `'multipart/form-data'`)

Check for multipart/form-data vs application/json mismatches.

Report:
- Endpoint path
- Backend expectation (JSON vs multipart)
- Frontend actual (JSON vs multipart)
- File:line for both sides

---

### Step 16: Execute Check 11 — Route Parameter Shadowing (CRITICAL)

Within each backend controller file, check if a **static route** is declared **after** a **parameterized route** that shares the same HTTP method and prefix. When this happens, the framework matches the parameterized route first at runtime, and the static route is never reached.

**Algorithm:**

1. For each controller file, collect all route handler methods in **declaration order** (top to bottom). For each, record:
   - HTTP method (`@Get`, `@Post`, `@Patch`, `@Put`, `@Delete`)
   - Route path (e.g., `'users/:id'`, `'users/bulk-status'`)
   - Line number
   - Method name

2. Group routes by HTTP method (GET, POST, PATCH, etc.).

3. Within each HTTP method group, for each pair of routes (A declared before B):
   - If route A has a **parameter segment** (`:id`, `:userId`, etc.) at the same position where route B has a **static segment**, AND both share the same prefix up to that point → **B is shadowed by A**.
   - Example: A = `users/:id` (line 50), B = `users/bulk-status` (line 80) → `bulk-status` will be captured as `:id` at runtime.

4. Also check for routes inherited from a base controller. If the controller extends a base class that provides `@Patch(':id')` or similar, any static route like `@Patch('bulk-status')` in the child controller may be shadowed depending on the framework's route resolution order. Flag these for manual review.

**Detection patterns (adapt per framework):**

| Framework | Route shadowing behavior |
|-----------|------------------------|
| NestJS | Routes matched in **declaration order** within a controller. Static routes MUST be declared before parameterized routes. |
| Express | Routes matched in **registration order** via `router.get()`, `router.post()`, etc. Same rule applies. |
| Spring Boot | Uses **most-specific-match** — static routes win over parameterized. No shadowing issue (but still worth noting if ordering is confusing). |
| Django | URL patterns matched in **declaration order** in `urlpatterns`. Static patterns must come before parameterized. |

**Report each finding with:**
- Controller file and line numbers of both routes
- HTTP method
- Shadowing route (parameterized, declared first)
- Shadowed route (static, declared after — unreachable)
- Impact: which frontend API call will fail

---

### Step 17: Generate Audit Report

After completing all 11 checks, compile the full report in the format specified in the Output Format section below.

---

## Output Format

Present the report in this exact structure:

```markdown
# QA API Sync Audit Report

**Date**: {current date}
**Scope**: {All modules | module name}
**Backend controllers scanned**: {count}
**Frontend service files scanned**: {count}
**Total backend endpoints**: {count}
**Total frontend API calls**: {count}

## Summary

| # | Check | Severity | Issues Found |
|---|-------|----------|-------------|
| 1 | Route mismatch | Critical | {count} |
| 2 | HTTP method mismatch | Critical | {count} |
| 3 | Missing query params | High | {count} |
| 4 | Missing body params | High | {count} |
| 5 | Extra body params | Medium | {count} |
| 6 | Response type mismatch | Medium | {count} |
| 7 | Unused backend endpoint | Info | {count} |
| 8 | Missing error handling | Medium | {count} |
| 9 | Auth header missing | High | {count} |
| 10 | Content-Type mismatch | Medium | {count} |
| 11 | Route parameter shadowing | Critical | {count} |
| | **Total** | | **{total count}** |

## Route Mapping

| Backend Endpoint | Method | Frontend Service | Frontend File | Status |
|-----------------|--------|-----------------|---------------|--------|
| /exercises | GET | exerciseService.getAll | exerciseService.ts:12 | Matched |
| /exercises | POST | exerciseService.create | exerciseService.ts:42 | Matched |
| /exercises/:id | PATCH | exerciseService.update | exerciseService.ts:56 | Matched |
| /exercises/:id | DELETE | exerciseService.delete | exerciseService.ts:68 | Matched |
| /exercises/stats | GET | - | - | No frontend caller |
| - | - | exerciseService.archive | exerciseService.ts:80 | No backend endpoint |

---

## CRITICAL Issues

### Check 1: Route Mismatches ({count})

{If count > 0:}
| # | Frontend Call | Method | Path | Closest Backend Match | File |
|---|-------------|--------|------|-----------------------|------|
| 1 | serviceName.methodName | POST | /exercises/bulk | POST /exercises (similar) | exerciseService.ts:92 |

**Details:**
1. `POST /exercises/bulk` called in `exerciseService.ts:92` — No matching backend endpoint exists. The closest match is `POST /exercises`. Either the frontend route has a typo or the backend endpoint needs to be created.

{If count == 0:}
No route mismatches found.

### Check 2: HTTP Method Mismatches ({count})

{If count > 0:}
| # | Path | Frontend Method | Backend Method | Frontend File | Backend File |
|---|------|----------------|----------------|---------------|-------------|
| 1 | /exercises/:id | PUT | PATCH | exerciseService.ts:56 | exercise.controller.ts:34 |

**Details:**
1. `/exercises/:id` — Frontend uses `PUT` but backend expects `PATCH`. Update the frontend to use `PATCH` for partial updates.

{If count == 0:}
No HTTP method mismatches found.

### Check 11: Route Parameter Shadowing ({count})

{If count > 0:}
| # | Controller | Method | Shadowing Route (declared first) | Shadowed Route (unreachable) | Impact |
|---|-----------|--------|----------------------------------|------------------------------|--------|
| 1 | admin.controller.ts | PATCH | `users/:id` (line 50) | `users/bulk-status` (line 80) | `PATCH /admin/users/bulk-status` captured by `:id` param → ParseUUIDPipe fails with 400 |

**Details:**
1. In `admin.controller.ts`, `@Patch('users/:id')` at line 50 is declared before `@Patch('users/bulk-status')` at line 80. NestJS matches routes in declaration order, so `bulk-status` is captured as the `:id` parameter. The `ParseUUIDPipe` then rejects `"bulk-status"` as an invalid UUID, returning a 400 error. **Fix**: Move `@Patch('users/bulk-status')` above `@Patch('users/:id')` in the controller.

{If count == 0:}
No route parameter shadowing issues found.

---

## HIGH Issues

### Check 3: Missing Query Params ({count})

{If count > 0:}
| # | Endpoint | Backend Params | Frontend Sends | Missing | File |
|---|----------|---------------|----------------|---------|------|
| 1 | GET /exercises | page, limit, search, status | page, limit | search, status | exerciseService.ts:12 |

{If count == 0:}
No missing query parameter issues found.

### Check 4: Missing Body Params ({count})

{If count > 0:}
| # | Endpoint | DTO | Required Fields | Frontend Sends | Missing Fields | File |
|---|----------|-----|----------------|----------------|----------------|------|
| 1 | POST /exercises | CreateExerciseDto | title, description, videoUrl | title, description | videoUrl | exerciseService.ts:42 |

{If count == 0:}
No missing body parameter issues found.

### Check 9: Auth Header Missing ({count})

{If global auth exists:}
Global auth mechanism detected: {describe mechanism — interceptor, withCredentials, cookie-based, etc.}. All protected endpoints are covered.

{If count > 0:}
| # | Endpoint | Auth Mechanism | Issue | File |
|---|----------|---------------|-------|------|
| 1 | GET /exercises | None detected | No Authorization header or cookie auth | exerciseService.ts:12 |

{If count == 0:}
No auth header issues found.

---

## MEDIUM Issues

### Check 5: Extra Body Params ({count})

{If count > 0:}

**Whitelist status**: {whitelist: true | whitelist: false | not configured} (from ValidationPipe in main.ts)

| # | Endpoint | DTO | Extra Fields | Risk | File |
|---|----------|-----|-------------|------|------|
| 1 | POST /exercises | CreateExerciseDto | categoryId, tags | {Silently stripped / Mass assignment risk} | exerciseService.ts:42 |

{If count == 0:}
No extra body parameter issues found.

### Check 6: Response Type Mismatches ({count})

{If count > 0:}
| # | Endpoint | Backend Field | Backend Type | Frontend Field | Frontend Type | Issue |
|---|----------|-------------|-------------|---------------|--------------|-------|
| 1 | GET /exercises | createdAt | Date | createdAt | string | Type mismatch |
| 2 | GET /exercises/:id | coach | User (nested) | coachId | string | Shape mismatch |

{If count == 0:}
No response type mismatches found.

### Check 8: Missing Error Handling ({count})

{If global interceptor exists:}
Note: Global axios error interceptor detected in {file}. Individual calls without try/catch still have baseline error handling. The following calls lack specific error recovery logic:

{If count > 0:}
| # | Frontend Call | Method | Path | File |
|---|-------------|--------|------|------|
| 1 | exerciseService.getStats | GET | /exercises/stats | exerciseService.ts:78 |

{If count == 0:}
No missing error handling issues found.

### Check 10: Content-Type Mismatches ({count})

{If count > 0:}
| # | Endpoint | Backend Expects | Frontend Sends | Frontend File | Backend File |
|---|----------|----------------|----------------|---------------|-------------|
| 1 | POST /exercises/upload | multipart/form-data | application/json | exerciseService.ts:95 | exercise.controller.ts:110 |

{If count == 0:}
No content-type mismatches found.

---

## INFO

### Check 7: Unused Backend Endpoints ({count})

{If count > 0:}
| # | Endpoint | Method | Classification | Controller File |
|---|----------|--------|---------------|----------------|
| 1 | /exercises/stats | GET | Likely needs frontend | exercise.controller.ts:89 |
| 2 | /auth/refresh | POST | Intentional (internal) | auth.controller.ts:45 |

{If count == 0:}
All backend endpoints have corresponding frontend callers.

---

## Recommendations

{Provide 3-5 prioritized recommendations based on findings. Examples:}

1. **Fix critical route mismatches first** — These represent API calls that will fail at runtime with 404 errors.
2. **Align HTTP methods** — Use PATCH for partial updates, PUT for full replacements. Mismatched methods cause 405 errors.
3. **Add missing pagination params** — Backend supports pagination but frontend doesn't use it, leading to unbounded queries that may cause performance issues.
4. **Add error handling to unprotected calls** — Consider adding a global axios error interceptor if most calls lack individual handling.
5. **Update frontend types** — Several response types are out of sync with backend entities, which may cause runtime type errors.
```

If a check has zero issues, still include the section header but write "No issues found." underneath.

---

## Scanning Patterns Reference

Use these patterns when performing the audit steps above.

| What | Grep Pattern | File Glob |
|------|-------------|-----------|
| NestJS controllers | `@Controller\(` | `{backend-dir}/**/*.controller.ts` |
| Route decorators | `@(Get\|Post\|Put\|Patch\|Delete)\(` | `{backend-dir}/**/*.controller.ts` |
| Body DTO | `@Body\(\)\s+\w+:\s+(\w+)` | `{backend-dir}/**/*.controller.ts` |
| Query params (object) | `@Query\(\)` | `{backend-dir}/**/*.controller.ts` |
| Individual query param | `@Query\(['"](\w+)['"]\)` | `{backend-dir}/**/*.controller.ts` |
| Path params | `@Param\(` | `{backend-dir}/**/*.controller.ts` |
| Public endpoint | `@Public\(\)` | `{backend-dir}/**/*.controller.ts` |
| Role restriction | `@Roles\(` | `{backend-dir}/**/*.controller.ts` |
| Guards | `@UseGuards\(` | `{backend-dir}/**/*.controller.ts` |
| File upload interceptor | `FileInterceptor\|FilesInterceptor` | `{backend-dir}/**/*.controller.ts` |
| Uploaded file param | `@UploadedFile\|@UploadedFiles` | `{backend-dir}/**/*.controller.ts` |
| Validation pipe config | `ValidationPipe\|whitelist` | auto-detected backend entry point |
| Base controller usage | `extends BaseController` | `{backend-dir}/**/*.controller.ts` |
| Parameterized routes | `@(Get\|Post\|Put\|Patch\|Delete)\(['"]:` | `{backend-dir}/**/*.controller.ts` |
| Static sub-routes | `@(Get\|Post\|Put\|Patch\|Delete)\(['"][a-z]` | `{backend-dir}/**/*.controller.ts` |
| DTO classes | `class\s+\w+Dto` | `{backend-dir}/**/*.dto.ts` |
| DTO required validators | `@IsNotEmpty\|@IsDefined` | `{backend-dir}/**/*.dto.ts` |
| DTO optional marker | `@IsOptional` | `{backend-dir}/**/*.dto.ts` |
| DTO type validators | `@IsString\|@IsNumber\|@IsBoolean\|@IsEnum\|@IsArray\|@IsDate\|@IsUUID` | `{backend-dir}/**/*.dto.ts` |
| DTO inheritance | `PartialType\|PickType\|OmitType\|IntersectionType` | `{backend-dir}/**/*.dto.ts` |
| Entity classes | `class\s+\w+\s+extends\s+BaseEntity` | `{backend-dir}/**/*.entity.ts` |
| Entity columns | `@Column\(` | `{backend-dir}/**/*.entity.ts` |
| Entity relations | `@ManyToOne\|@OneToMany\|@ManyToMany\|@OneToOne` | `{backend-dir}/**/*.entity.ts` |
| Axios calls | `axios\.(get\|post\|put\|patch\|delete)` | `{frontend-dir}/**/*.ts` |
| Fetch calls | `fetch\(` | `{frontend-dir}/**/*.ts` |
| API instance calls | `\.(get\|post\|put\|patch\|delete)\(` | `{frontend-dir}/**/services/**/*.ts` |
| API base URL | `baseURL\|BASE_URL\|API_URL\|VITE_API_URL` | `{frontend-dir}/**/*.ts`, `.env*` |
| Axios create instance | `axios\.create\(` | `{frontend-dir}/**/*.ts` |
| React Query hooks | `useQuery\|useMutation` | `{frontend-dir}/**/*.tsx` |
| Error handling (try) | `try\s*\{` | `{frontend-dir}/**/*.ts` |
| Error handling (catch) | `\.catch\(` | `{frontend-dir}/**/*.ts` |
| Error handling (RQ) | `onError` | `{frontend-dir}/**/*.tsx` |
| FormData usage | `new FormData\|FormData` | `{frontend-dir}/**/*.ts` |
| Axios request interceptors | `interceptors\.request\.use` | `{frontend-dir}/**/*.ts` |
| Axios response interceptors | `interceptors\.response\.use` | `{frontend-dir}/**/*.ts` |
| Auth headers | `Authorization\|Bearer` | `{frontend-dir}/**/*.ts` |
| Cookie credentials | `withCredentials` | `{frontend-dir}/**/*.ts` |
| Frontend interfaces | `(interface\|type)\s+\w+` | `{frontend-dir}/**/types/**/*.ts`, `{frontend-dir}/**/interfaces/**/*.ts` |
| Global prefix | `setGlobalPrefix\|enableVersioning` | auto-detected backend entry point |
| API response decorators | `@ApiResponse\|@ApiOkResponse\|@ApiCreatedResponse` | `{backend-dir}/**/*.controller.ts` |

---

## Edge Cases & Notes

1. **BaseController inheritance**: When a controller extends BaseController, it automatically provides 5 standard CRUD endpoints (findAll, findOne, create, update, remove). These MUST be included in the route inventory even though they are not explicitly declared in the controller file. Read the BaseController source to confirm the exact endpoints, HTTP methods, and any parameter decorators.

2. **Overridden base methods**: If a controller extends BaseController but also declares a method with the same route as a base method (e.g., custom `findAll` with `@Get()`), the custom method takes precedence. Do not double-count these endpoints. The custom version overrides the inherited one.

3. **Dynamic routes**: Some routes use dynamic segments. Use pattern matching, not exact string comparison:
   - `/exercises/:id` should match `/exercises/${exerciseId}` or `/exercises/${id}`
   - `/users/:userId/exercises` should match `/users/${userId}/exercises`
   - Any segment starting with `:` in backend or wrapped in `${}` in frontend is a path parameter

4. **API prefix resolution**: The frontend may configure its base URL in multiple places:
   - `.env` file: `VITE_API_URL=http://localhost:{port}` or similar
   - Axios create: `baseURL: '/api'`
   - Config file: `API_BASE = import.meta.env.VITE_API_URL`
   - Hardcoded in individual calls

   You must resolve the full path before comparing. The global prefix from `main.ts` and the baseURL from the axios instance must be accounted for together.

5. **Partial DTOs**: Update DTOs often extend `PartialType(CreateDto)`, making all fields optional. Account for this when checking required fields (Check 4). A `PATCH` endpoint with `PartialType` DTO has no required fields.

6. **Multiple frontends**: The project may have multiple frontend apps (auto-detect from project structure). Scan ALL frontend apps when doing a full audit. A backend endpoint might be used by one frontend but not the other — this is expected, not an issue.

7. **Socket.IO endpoints**: WebSocket gateways are NOT REST endpoints. Do not include them in the route inventory. However, note if a frontend service file contains both REST and WebSocket calls for context.

8. **Intentionally unused endpoints**: Some endpoints are for future features, webhooks, or internal use. Classify them as INFO, not issues. Use the heuristics in Check 7 to classify.

9. **Cookie-based auth**: The project may use HTTP-only cookies for auth. If `withCredentials: true` is set globally on the axios instance, all requests automatically include auth cookies. Check 9 should account for this — if global cookie auth is configured, all protected endpoints are covered.

10. **Module-scoped audit**: When auditing a single module, still check the frontend globally for calls to that module's endpoints — the calls might be in unexpected files (e.g., a dashboard component calling exercise endpoints directly).

11. **Nested controllers**: Some modules may have multiple controllers (e.g., `exercise.controller.ts` and `exercise-log.controller.ts`). Make sure to scan ALL controllers in the module directory, not just the primary one.

12. **Route parameter naming**: Backend might use `:id` while frontend uses `${exerciseId}`. These should still match as long as the path structure is the same. The parameter name itself doesn't matter for matching — only the position in the path.

13. **Query string vs params object**: Frontend may pass query parameters either as a params object (`{ params: { page: 1 } }`) or as part of the URL string (`?page=1`). Both should be detected and included in the query params inventory.

14. **Conditional API calls**: Some frontend code may conditionally include parameters (e.g., `params: search ? { search } : {}`). Note these as partially covered — the parameter is sent sometimes but not always.

15. **Re-exported services**: Some frontend service files may re-export functions from other files. Follow the import chain to find the actual API call.

16. **Route parameter shadowing**: In NestJS (and Express, Django), route handlers are matched in declaration order. A parameterized route like `@Patch('users/:id')` declared before a static route like `@Patch('users/bulk-status')` will capture `bulk-status` as the `:id` parameter. This causes runtime failures (e.g., `ParseUUIDPipe` rejects `"bulk-status"`) even though both routes are correctly defined in the code. This bug is invisible to static route-existence checks — the route exists in the controller, but is unreachable at runtime. Always check declaration order when a controller has both parameterized and static routes sharing the same prefix and HTTP method.

---

## IMPORTANT RULES

- This is a **READ-ONLY** diagnostic. Do NOT modify any files.
- Do NOT create any files.
- Do NOT suggest code fixes inline — only report issues in the audit table format.
- Be thorough: scan ALL controller files and ALL service files in scope.
- Be precise: report exact file paths and line numbers.
- Be practical: do not report optional query params as "missing" — only flag when required params or commonly expected params (pagination) are absent.
- Account for inheritance: BaseController endpoints, DTO inheritance (PartialType, OmitType, PickType, IntersectionType).
- When in doubt about whether something is an issue, include it with appropriate severity and let the developer decide.
- Present the full report inline when the audit is complete. Do not write the report to a file unless the user explicitly asks.
- Always include the Recommendations section with actionable next steps based on findings.
