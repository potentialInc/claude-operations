---
name: qa-security
description: "Audit backend security vulnerabilities - SQL injection, cryptographic randomness, file upload safety, sensitive logging, production exposure, XSS, infrastructure config"
user-invocable: true
argument-hint: "[module]"
---

# QA Security - Backend Security Deep Auditor

## Purpose
Detect security vulnerabilities that go beyond authentication/authorization (covered by qa-auth). This skill audits code-level security: injection attacks, cryptographic weaknesses, unsafe file handling, sensitive data leakage, production configuration gaps, and client-side security.

## Usage
```
/qa-security                    # Full audit
/qa-security auth               # Single module only
```

## 12 Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | SQL/NoSQL injection | **Critical** | Raw user input interpolated into queries without parameterization |
| 2 | Insecure randomness | **High** | `Math.random()`, `random.random()`, or similar non-cryptographic RNG used for security-sensitive values (OTP, tokens, secrets) |
| 3 | Unsanitized file upload | **Medium** | User-controlled filename used in storage path without sanitization (path traversal risk) |
| 4 | Sensitive data in logs | **High** | `console.log`, `logger.debug`, or equivalent outputs JWT tokens, passwords, OTP codes, cookies, or API keys |
| 5 | Debug/test routes in production | **Medium** | Static file serving, debug endpoints, or test fixtures accessible without environment guard |
| 6 | XSS via unsafe URL handling | **High** | `window.open()`, `href`, `src`, or `location` set from user-controlled data without scheme validation (`javascript:` protocol injection) |
| 7 | Infrastructure secrets | **Medium** | Hardcoded secrets, missing auth on Redis/DB, default credentials, `process.env` used at module evaluation time instead of ConfigService |
| 8 | Missing input sanitization at boundaries | **Medium** | User input from request body/params/query not validated before use in business logic (beyond DTO validation — raw SQL, file ops, external API calls) |
| 9 | Unsafe deserialization | **High** | `JSON.parse()` on untrusted input without try-catch, `eval()`, `new Function()`, or `child_process.exec` with user input |
| 10 | Side-effect logic bugs in auth flows | **Medium** | Password change/reset setting unintended flags (e.g., `isVerified: false`), token refresh not invalidating old token, session not cleared on password change |
| 11 | IME/i18n input compatibility | **Medium** | `onKeyPress` used instead of `onKeyDown` with `isComposing` check (breaks CJK input), or Enter key handling without IME composition guard |
| 12 | Missing request size limits | **Low** | No body size limit on file upload or JSON endpoints, no array length limit on batch endpoints, enabling resource exhaustion |

## Execution Algorithm

You MUST follow these steps in order. Do NOT skip steps. Do NOT modify any files. This is a READ-ONLY diagnostic.

### Step 0: Context & Requirements Gathering (MANDATORY)

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
```

Extract:
- Tech stack (ORM, framework, language)
- Database type (SQL vs NoSQL)
- File storage provider (S3, local, GCS)
- Authentication method and token storage
- Deployment model (Docker, serverless, traditional)

**0b. Build In-Scope Module List:**

Same as other QA skills — only audit active, in-scope modules.

### Step 1: Project & Framework Detection

Auto-detect backend framework, ORM, and infrastructure:

| Component | Detection Pattern |
|-----------|------------------|
| ORM | TypeORM, Prisma, Sequelize, Django ORM, SQLAlchemy |
| File storage | S3, multer, formidable, busboy |
| Cache | Redis, Memcached, in-memory |
| Logging | winston, pino, console, bunyan |
| Auth | JWT, session, OAuth |

Record detected components for all subsequent checks.

### Step 2: SQL/NoSQL Injection Audit (Check 1)

**2a. Find all raw query usage:**
```
Grep: "createQueryBuilder|\.query\(|\.execute\(|raw\(|rawQuery|sequelize\.query|cursor\.execute|executeRaw|queryRaw" in backend source files
Grep: "\$\{.*\}.*(?:WHERE|SET|INSERT|UPDATE|DELETE|ORDER BY|GROUP BY|HAVING|LIKE|IN \()" in backend source files (template literals in SQL)
Grep: "' \+ |\" \+ |\.concat\(" combined with SQL keywords in backend source files
```

**2b. For each raw query found, trace the data source:**

1. Identify the variable(s) interpolated into the query
2. Trace back to the method parameter
3. Check if the parameter comes from user input (request body, params, query)
4. Check if the parameter is validated/sanitized before interpolation

**2c. Classification:**

| Pattern | Assessment |
|---------|------------|
| Parameterized query (`$1`, `:param`, `?`) | SAFE |
| ORM method (`.find()`, `.where()`, `.update()`) | SAFE |
| Template literal with user input in SQL string | **CRITICAL** |
| String concatenation with user input in SQL | **CRITICAL** |
| Template literal with validated/whitelisted input | SAFE (with note) |
| `IN (${ids.join(',')})` with unvalidated IDs | **CRITICAL** |
| `ORDER BY ${column}` with whitelisted column names | SAFE |
| `ORDER BY ${column}` with user-provided column name | **HIGH** (SQL injection via ORDER BY) |

**2d. UUID/ID format validation:**

If IDs from user input are used in raw queries, check for format validation:
```
Grep: "uuidRegex|isUUID|uuid\.validate|IsUUID" near the raw query usage
```
If user-provided IDs are interpolated without UUID format validation, flag as CRITICAL even if not directly in a WHERE clause.

### Step 3: Cryptographic Randomness Audit (Check 2)

**3a. Find security-sensitive random value generation:**
```
Grep: "Math\.random\(\)|Math\.floor.*Math\.random|random\.random\(\)|rand\(\)|mt_rand\(" in backend source files
```

**3b. Determine if usage is security-sensitive:**

| Usage Context | Security-Sensitive? |
|---------------|---------------------|
| OTP code generation | YES — must use crypto RNG |
| Password reset token | YES — must use crypto RNG |
| Session ID / token | YES — must use crypto RNG |
| API key generation | YES — must use crypto RNG |
| File name randomization | NO — Math.random() is acceptable |
| UI element key/ID | NO — Math.random() is acceptable |
| Test data generation | NO — Math.random() is acceptable |

**3c. Check for proper alternatives:**

| Language | Secure Alternative |
|----------|-------------------|
| Node.js | `crypto.randomInt()`, `crypto.randomBytes()`, `crypto.randomUUID()` |
| Python | `secrets.token_hex()`, `secrets.randbelow()`, `os.urandom()` |
| Java | `SecureRandom` |
| PHP | `random_int()`, `random_bytes()` |

**3d. OTP keyspace analysis:**

If OTP generation is found, calculate the effective keyspace:
- 4-digit numeric: 10,000 combinations (weak)
- 4-digit no repeats, no leading zero: only 4,536 (very weak)
- 6-digit numeric: 1,000,000 combinations (acceptable)
- 6-digit alphanumeric: 2,176,782,336 (strong)

Flag if effective keyspace is below 10,000.

### Step 4: File Upload Safety Audit (Check 3)

**4a. Find file upload handling:**
```
Grep: "originalname|filename|file\.name|upload|multer|formidable|busboy" in backend source files
Grep: "S3|putObject|PutObjectCommand|Upload|uploadFile" in backend source files
```

**4b. For each file upload handler, check:**

| Check | Pattern | Assessment |
|-------|---------|------------|
| Filename sanitization | `replace(/[^a-zA-Z0-9._-]/g, '_')` or similar | SAFE |
| UUID prefix | `${uuid}-${filename}` | Partially safe (UUID prevents collision, but unsanitized name may contain `../`) |
| Path traversal prevention | Check for `..`, `/`, `\` in filename | Must be present |
| File type validation | MIME type or extension whitelist | Should be present |
| File size limit | `limits: { fileSize: ... }` in multer config | Should be present |
| Content-Type validation | Server-side MIME check, not just extension | Ideal |

### Step 5: Sensitive Data in Logs Audit (Check 4)

**5a. Find logging statements:**
```
Grep: "console\.(log|debug|info|warn|error)|logger\.(log|debug|info|warn|error|verbose)|this\.logger\.(log|debug|warn|error)" in backend source files
```

**5b. For each logging statement, check for sensitive data patterns:**
```
Grep: "console\.log.*token|console\.log.*jwt|console\.log.*cookie|console\.log.*password|console\.log.*otp|console\.log.*secret|console\.log.*apiKey|console\.log.*payload" in backend source files
Grep: "console\.log.*req\.headers|console\.log.*authorization|console\.log.*Bearer" in backend source files
```

**5c. Classification:**

| Log Content | Assessment |
|-------------|------------|
| Full request headers logged | **HIGH** — includes Authorization header |
| JWT token or payload logged | **HIGH** — token can be replayed |
| Cookie values logged | **HIGH** — session hijacking risk |
| Password (even hashed) logged | **HIGH** — aids offline attacks |
| OTP code logged | **MEDIUM** — useful for debugging but risky in production |
| User ID or email logged | **INFO** — generally acceptable for audit trails |
| Error stack trace logged | **INFO** — acceptable if not exposed to client |

**5d. Environment-based logging check:**

Verify if debug/verbose logging is disabled in production:
```
Grep: "NODE_ENV.*production.*log|LOG_LEVEL|logLevel" in config files
```

### Step 6: Debug/Test Routes in Production (Check 5)

**6a. Find static file serving:**
```
Grep: "ServeStaticModule|express\.static|app\.use.*static|staticfiles" in backend source files
```

**6b. For each static route, check for environment guard:**
```
Grep: "NODE_ENV|ENVIRONMENT|isDevelopment|isProduction" near the static file configuration
```

**6c. Check for debug/test endpoints:**
```
Grep: "@Get.*test|@Get.*debug|@Get.*seed|@Post.*seed|/api/test|/api/debug" in controller files
Grep: "swagger|api-docs|openapi" in main entry file — check if conditionally disabled in production
```

**6d. Classification:**

| Route | Environment Guard | Assessment |
|-------|-------------------|------------|
| `/test` serving `src/test/` directory | No guard | **MEDIUM** — test files exposed |
| `/api-docs` (Swagger) | No guard | **MEDIUM** — API documentation exposed |
| `/debug` endpoint | No guard | **HIGH** — debug functionality exposed |
| `/seed` endpoint | No guard | **CRITICAL** — data manipulation exposed |
| Static assets (icons, images) | No guard needed | OK |

### Step 7: XSS via Unsafe URL Handling (Check 6)

**7a. Find URL usage in frontend:**
```
Grep: "window\.open\(|window\.location|location\.href|\.href\s*=|\.src\s*=" in frontend source files
Grep: "dangerouslySetInnerHTML|v-html|innerHTML|__html" in frontend source files
```

**7b. For each URL usage, trace the data source:**

Check if the URL value comes from:
- User input (form field, query param)
- API response (could contain user-generated content)
- Database record (imageUrl, profileUrl, etc.)

**7c. Check for scheme validation:**
```
Grep: "startsWith.*http|protocol.*http|isValidUrl|URL.*try|new URL" near the URL usage
```

**7d. Classification:**

| Pattern | Assessment |
|---------|------------|
| `window.open(userUrl)` without scheme check | **HIGH** — `javascript:alert(1)` injection |
| `<a href={userUrl}>` without scheme check | **HIGH** — same risk |
| `<img src={apiUrl}>` without scheme check | **MEDIUM** — limited XSS vector |
| URL validated with `new URL()` + scheme check | SAFE |
| URL from trusted source (hardcoded, env config) | SAFE |

### Step 8: Infrastructure Secrets Audit (Check 7)

**8a. Find hardcoded secrets:**
```
Grep: "secret.*=.*['\"]|password.*=.*['\"]|apiKey.*=.*['\"]" in backend source files (excluding .env files and test files)
```
Exclude: `.env`, `.env.example`, test files, migration files

**8b. Check secret loading patterns:**

| Pattern | Assessment |
|---------|------------|
| `process.env.SECRET` at module evaluation time | **WARNING** — may be `undefined` if dotenv hasn't loaded |
| `configService.get('SECRET')` in injectable service | SAFE — loads from validated config |
| `JwtModule.register({ secret: process.env.X })` | **WARNING** — evaluated at import time |
| `JwtModule.registerAsync({ useFactory: (config) => ... })` | SAFE — deferred loading |
| Hardcoded string `"my-secret-key"` | **MEDIUM** — should use environment variable |

**8c. Check infrastructure service authentication:**

```
Grep: "redis-server|redis.*command|requirepass" in Docker/config files
Grep: "REDIS_PASSWORD|redisPassword" in config files — check if default is empty
Grep: "POSTGRES_PASSWORD|DB_PASSWORD" in Docker/config files
```

Flag if:
- Redis started without `--requirepass`
- Database connection has no password or uses a weak default
- Docker Compose exposes service ports to host network without auth

### Step 9: Input Sanitization at Boundaries (Check 8)

**9a. Find boundary points where user input enters the system:**

Beyond DTO validation, check where user input is used in:
- Raw SQL queries (covered in Check 1)
- File system operations (`fs.readFile`, `fs.writeFile`, `path.join` with user input)
- External API calls (`axios.get(userUrl)`, `fetch(userInput)`)
- Shell commands (`exec`, `spawn`, `child_process`)
- Email/SMS content (`sendEmail({ body: userContent })`)
- Regex construction (`new RegExp(userInput)`)

**9b. For each boundary, verify sanitization:**

| Boundary | Required Sanitization |
|----------|-----------------------|
| File path | Path traversal prevention, whitelist allowed directories |
| External URL | Scheme validation (http/https only), SSRF prevention |
| Shell command | Never use user input in shell commands; if unavoidable, use parameterized execution |
| Email body | HTML escaping if HTML email |
| Regex | Escape special regex characters (`escapeRegExp`) |

### Step 10: Unsafe Deserialization (Check 9)

```
Grep: "JSON\.parse\(|eval\(|new Function\(|child_process|exec\(|execSync\(|spawn\(" in backend source files
```

For each `JSON.parse()` call, check:
- Is it wrapped in try-catch?
- Does the input come from an untrusted source?

For `eval()`, `new Function()`:
- Flag as **CRITICAL** regardless of context
- These should never appear in production backend code

For `child_process.exec()`:
- Check if user input is passed to the command string
- `exec(userInput)` is **CRITICAL**
- `execFile(binary, [userParam])` is safer but still needs validation

### Step 11: Auth Flow Side-Effect Bugs (Check 10)

**11a. Examine password change/reset flows:**
```
Grep: "changePassword|resetPassword|updatePassword|setPassword" in auth service files
```

For each flow, check for unintended side effects:
- Setting `isVerified: false` after password change (locks out future changes)
- Not invalidating existing sessions/tokens after password change
- Not deleting used OTP/reset tokens after successful use
- Refresh token not rotated after password change
- Remember-me tokens not cleared

**11b. Check token lifecycle:**
```
Grep: "refreshToken|access_token|logout|signOut" in auth service files
```

Verify:
- Old refresh token is invalidated when new one is issued
- Logout actually deletes/invalidates the token (not just client-side)
- Password change invalidates all existing sessions

### Step 12: IME/i18n Input Compatibility (Check 11)

**12a. Find keyboard event handlers in frontend (framework-adaptive):**

| Framework | Deprecated Pattern (Grep) | Safe Pattern (Grep) |
|-----------|--------------------------|---------------------|
| React | `onKeyPress` | `onKeyDown` + `nativeEvent.isComposing` or `e.isComposing` |
| Vue | `@keypress` or `v-on:keypress` | `@keydown` + `e.isComposing` check |
| Angular | `(keypress)` | `(keydown)` + `event.isComposing` check |
| Svelte | `on:keypress` | `on:keydown` + `event.isComposing` check |
| Plain JS | `addEventListener('keypress')` | `addEventListener('keydown')` + `e.isComposing` check |

```
Grep: "onKeyPress|onkeypress|@keypress|v-on:keypress|\(keypress\)|on:keypress|addEventListener.*keypress" in frontend source files
```

`keypress` is deprecated across all frameworks and breaks CJK (Chinese, Japanese, Korean) input because it fires during IME composition.

**12b. Check for proper IME-aware handling:**
```
Grep: "isComposing|compositionend|compositionstart" in frontend source files
```

**12c. Classification:**

| Pattern | Framework | Assessment |
|---------|-----------|------------|
| `onKeyPress` / `@keypress` / `(keypress)` / `on:keypress` | All | **MEDIUM** — breaks CJK input |
| `onKeyDown` / `@keydown` / `(keydown)` without `isComposing` check | All | **MEDIUM** — Enter fires during IME composition |
| Key handler + `isComposing` guard (`e.isComposing` or `e.nativeEvent.isComposing`) | All | SAFE |
| `compositionend` + `keydown` combination | All | SAFE |

### Step 13: Request Size Limits (Check 12)

```
Grep: "bodyParser|json\(\{|urlencoded\(\{|limits|maxFileSize|maxFiles|MAX_FILE_SIZE" in backend config/main files
Grep: "\.length.*>.*100|\.length.*>.*1000|\.slice\(0" in controller/service files (array length limits)
```

Check:
- JSON body size limit configured (default is often unlimited or very large)
- File upload size limit configured
- Array inputs in DTOs have `@ArrayMaxSize()` or manual length validation
- Batch/bulk endpoints limit the number of items per request

### Step 14: Compile Report

Compile all findings into output format. Group by severity. Include file paths and line numbers.

## Output Format

```markdown
# QA Security Audit Report

**Date**: YYYY-MM-DD
**Scope**: All modules | {specified module}
**Backend Framework**: {auto-detected}
**ORM**: {auto-detected}
**File Storage**: {auto-detected}
**Infrastructure**: {Redis/DB/Cache status}

## Summary

| Severity | Check | Issues Found |
|----------|-------|-------------|
| CRITICAL | 1. SQL/NoSQL injection | X |
| HIGH | 2. Insecure randomness | X |
| MEDIUM | 3. Unsanitized file upload | X |
| HIGH | 4. Sensitive data in logs | X |
| MEDIUM | 5. Debug/test routes in production | X |
| HIGH | 6. XSS via unsafe URL handling | X |
| MEDIUM | 7. Infrastructure secrets | X |
| MEDIUM | 8. Missing input sanitization | X |
| HIGH | 9. Unsafe deserialization | X |
| MEDIUM | 10. Auth flow side-effect bugs | X |
| MEDIUM | 11. IME/i18n input compatibility | X |
| LOW | 12. Missing request size limits | X |
| | **Total** | **X** |

## CRITICAL Issues

### Check 1: SQL/NoSQL Injection

| # | File | Line | Query Type | User Input | Parameterized | Risk |
|---|------|------|------------|------------|---------------|------|
| 1 | path/to/service.ts | 42 | Raw SQL | orderedIds[] | No — string interpolation | Full SQL injection |

**Risk**: Attacker can execute arbitrary SQL commands, extract data, modify data, or drop tables.
**Fix**: Use parameterized queries or ORM methods. Validate input format (UUID regex for IDs).

---

## HIGH Issues
...

## MEDIUM Issues
...

## LOW Issues
...

## Recommendations

1. **[Priority 1]** Fix all SQL injection vulnerabilities immediately — these allow full database compromise.
2. **[Priority 2]** Replace Math.random() with crypto.randomInt() for all security-sensitive values.
3. **[Priority 3]** Remove sensitive data from log statements or gate behind environment check.
4. **[Priority 4]** Add scheme validation to all user-controlled URLs before use in window.open() or href.
5. **[Priority 5]** Guard debug/test routes behind NODE_ENV check.
6. **[Priority 6]** Add authentication to Redis and verify infrastructure secrets.
7. **[Priority 7]** Fix auth flow side-effects (isVerified, token invalidation).
8. **[Priority 8]** Replace onKeyPress with onKeyDown + isComposing for CJK compatibility.
```

## Scanning Patterns Reference

| What to Find | Regex Pattern | Where to Search |
|------|---------|-------|
| Raw SQL queries | `createQueryBuilder\|\.query\(\|\.execute\(\|rawQuery\|executeRaw` | Backend service/repository files |
| Template literal SQL | `\$\{.*\}.*WHERE\|SET\|INSERT\|ORDER BY` | Backend service/repository files |
| Math.random() | `Math\.random\(\)\|Math\.floor.*Math\.random` | Backend source files |
| crypto.randomInt | `crypto\.randomInt\|crypto\.randomBytes\|randomUUID` | Backend source files |
| File upload handling | `originalname\|multer\|formidable\|busboy` | Backend source files |
| S3 operations | `S3Client\|PutObjectCommand\|Upload\|uploadFile` | Infrastructure files |
| Logging with secrets | `console\.log.*token\|console\.log.*jwt\|console\.log.*cookie\|console\.log.*password` | Backend source files |
| Static file serving | `ServeStaticModule\|express\.static\|staticfiles` | Backend config files |
| Debug endpoints | `@Get.*test\|@Get.*debug\|@Post.*seed` | Controller files |
| window.open XSS | `window\.open\(\|location\.href\|\.href\s*=` | Frontend source files |
| dangerouslySetInnerHTML | `dangerouslySetInnerHTML\|v-html\|innerHTML` | Frontend source files |
| Hardcoded secrets | `secret.*=.*['\"]\|password.*=.*['\"]` | Backend source (excluding .env) |
| Redis no auth | `redis-server(?!.*requirepass)` | Docker/config files |
| eval/exec | `eval\(\|new Function\(\|child_process\|exec\(` | Backend source files |
| JSON.parse without try | `JSON\.parse\(` | Backend source files |
| onKeyPress (deprecated) | `onKeyPress\|onkeypress` | Frontend source files |
| IME composition guard | `isComposing\|compositionend\|compositionstart` | Frontend source files |
| Body size limits | `bodyParser\|json\(\{\|limits\|maxFileSize` | Backend config files |
| Array size limits | `@ArrayMaxSize\|@ArrayMinSize\|\.length.*>` | DTO and service files |
| Auth side effects | `isVerified.*false\|isActive.*false` in password/auth methods | Auth service files |
| Token invalidation | `refreshToken.*null\|delete.*token\|invalidate` | Auth service files |

## Edge Cases to Watch For

1. **ORM query builder is NOT always safe**: `createQueryBuilder` with user input in `.orderBy()`, `.where()` template strings, or `.addSelect()` can still allow injection. Only parameterized values (`:param` syntax) are safe.

2. **UUID format as injection prevention**: Validating that an ID matches UUID format (`/^[0-9a-f]{8}-...$/i`) effectively prevents SQL injection for that parameter, even in raw queries. But this is defense-in-depth — parameterized queries are still preferred.

3. **Math.random() in tests is fine**: Only flag security-sensitive production code. Test data generation, UI element keys, and non-security randomization are acceptable uses.

4. **console.log in development vs production**: If the project has a logger that strips debug logs in production, `console.log` with sensitive data is lower severity. Check for environment-based log level configuration.

5. **S3 keys are flat**: S3 does not have true directories, so `../` in a key name does not traverse directories in the traditional sense. However, it can cause unexpected key paths and confuse URL parsers downstream, especially if the bucket serves content via a reverse proxy.

6. **Swagger/OpenAPI in production**: Some teams intentionally expose API docs in production (internal tools). Ask before flagging as an issue.

7. **CJK input (Korean, Japanese, Chinese) and IME**: `onKeyPress` fires during IME composition, causing premature Enter key handling (e.g., submitting a chat message while typing Korean). The fix is `onKeyDown` + `!e.nativeEvent.isComposing` check.

8. **Multiple verification paths for same OTP**: The same OTP code may be checkable via a dedicated verify endpoint AND inline in a reset-password flow. If only one path has brute-force protection, the other is exploitable.

9. **process.env at module evaluation time**: In NestJS, `JwtModule.register({ secret: process.env.X })` evaluates at module import time, before `ConfigModule` loads `.env`. If `.env` is not pre-loaded via `dotenv.config()` in `main.ts`, the secret will be `undefined`. Use `JwtModule.registerAsync()` with `ConfigService` injection instead.

10. **Auth flow side-effects are subtle**: Setting `isVerified: false` after `changePassword` seems like a security measure (force re-verification), but if there's no re-verification flow accessible to the user, it permanently locks them out of future password changes.

## Related Skills

- `qa-auth` — Authentication guards, role checks, token storage (complementary — qa-auth checks access control, qa-security checks code-level vulnerabilities)
- `qa-inputs` — Input field validation (DTO/Zod/form level — qa-security checks deeper, beyond-DTO validation)
- `qa-api-sync` — API parameter mismatches (qa-security checks what happens with those parameters in business logic)
- `qa-db-integrity` — Database schema consistency (qa-security checks query safety against that schema)
- `qa-modal` — Modal rendering (portal rendering for security-relevant stacking context issues)
- `qa-performance` — Bundle and rendering (overlaps with request size limits)

## Score Calculation

| Category | Weight | Description |
|----------|--------|-------------|
| Injection Prevention | 25% | SQL/NoSQL injection, unsafe deserialization, command injection |
| Cryptographic Safety | 15% | Secure RNG for tokens/OTP, proper keyspace |
| Data Leakage Prevention | 15% | Sensitive logs, debug routes, response data |
| File Upload Safety | 10% | Filename sanitization, type validation, size limits |
| XSS Prevention | 10% | URL scheme validation, innerHTML safety |
| Infrastructure Security | 10% | Redis auth, secret management, env config |
| Auth Flow Integrity | 10% | Side-effect bugs, token lifecycle |
| Input Compatibility | 5% | IME handling, i18n safety |

**Scoring rules** (see `qa-shared-reference` for full scoring system):
```
Base Score: 100
Deductions: CRITICAL = -15, HIGH = -5, MEDIUM = -3, LOW = 0
Score = max(0, 100 - sum(deductions))
```

**Status thresholds:**
- **PASS**: >= 80 points
- **NEEDS ATTENTION**: 60-79 points
- **FAIL**: < 60 points

## READ-ONLY Diagnostic

This skill is a READ-ONLY diagnostic. It does NOT modify any files. It scans, analyzes, and reports findings. All recommended fixes are presented as suggestions in the report for the developer to implement.
