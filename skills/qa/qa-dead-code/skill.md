---
name: qa-dead-code
description: Detect dead Entity columns - defined in ORM but never written via DTOs or service logic
user-invocable: true
argument-hint: "[module]"
---

# QA Dead Code - Dead Column Detector

## Purpose

Detect Entity/Model columns that exist in the database schema but are never populated
through any application code path. These "dead columns" occupy storage, appear in API
responses as null/undefined, confuse developers, and add maintenance burden. Auto-detects
ORM (TypeORM, Prisma, Sequelize, Django ORM, SQLAlchemy) and adapts scanning patterns
accordingly. This is a READ-ONLY static analysis tool that does NOT modify any files.

## Usage

```
/qa-dead-code                 # Full audit across all modules
/qa-dead-code users           # Audit a single module
```

When a module argument is provided, scope entity scanning to that module's directory
under the auto-detected backend modules directory. When no argument is provided, scan
all entity files project-wide.

## Future Versions

- **v2**: Unused API endpoints (controller routes with no frontend caller)
- **v3**: Unused service methods / unused imports

## Overview of Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | No Create DTO path | **High** | Entity column not in any Create DTO for that entity |
| 2 | No Update DTO path | **Medium** | Entity column not in any Update DTO for that entity |
| 3 | No programmatic write | **High** | Entity column never assigned in any service/repository method |
| 4 | Response DTO ghost | **Medium** | Entity column in Response DTO but never populated (always null) |
| 5 | Read-only column | **Info** | Entity column only used in WHERE/ORDER BY/SELECT but never written |

**Composite severity escalation:**
- Column fails Check 1 + Check 2 + Check 3 = **Critical** (true dead column - no write path exists)
- Column fails Check 1 + Check 2 but passes Check 3 = **Low** (system-managed column - verify intentional)
- Column fails only Check 1 or only Check 2 = report at individual check severity

---

## Execution Algorithm

Follow these steps in order. Complete each step fully before proceeding to the next.
This skill is READ-ONLY - do NOT modify any files.

---

### Step 0: Context & Requirements Gathering (MANDATORY)

Before auditing, you MUST understand which entities/modules are actively used.

**0a. Read project requirements:**
```
Read: CLAUDE.md (project root)
Glob: .claude-project/requirements/**/*.md
Glob: .claude-project/docs/PROJECT_KNOWLEDGE.md
```

Extract:
- Which entities/modules are in production vs demo/boilerplate
- Expected column usage patterns (e.g., "firstName/lastName are legacy, fullName replaced them")
- Which columns are intentionally system-managed (set by triggers, defaults, or external integrations)

**0b. Ask clarifying questions BEFORE proceeding:**

If you find entities for features not mentioned in requirements, ask:

> "I found [entity X] but it doesn't appear in the project requirements. Should I include it in the dead column audit or skip it as demo/unused code?"

Wait for the user's response. Do NOT flag intentionally unused columns as dead code.

**0c. Build scope list:**
```
IN SCOPE: [entity1, entity2, ...]
OUT OF SCOPE: [entity3 (demo module), entity4 (deprecated), ...]
SPECIAL RULES: [entity5.column (system-managed, skip), ...]
```

---

### Step 1: ORM & Framework Detection

Determine the ORM and backend framework in use.

**1a. Detect ORM:**

```
Glob: **/package.json (read dependencies)
Glob: **/requirements.txt, **/pyproject.toml, **/Gemfile
```

| Signal | ORM |
|--------|-----|
| `typeorm` in package.json dependencies | TypeORM |
| `prisma` in package.json + `prisma/schema.prisma` exists | Prisma |
| `sequelize` in package.json | Sequelize |
| `django` in requirements.txt | Django ORM |
| `sqlalchemy` in requirements.txt | SQLAlchemy |

Record the detected ORM. This determines decorator/annotation patterns for Steps 2-4.

**1b. Detect base entity pattern:**

```
Glob: **/base.entity.ts, **/base-entity.ts, **/abstract-entity.ts
Glob: **/core/**/base*.ts
```

Read the base entity file and extract all inherited column names. These columns are
EXCLUDED from dead column analysis because they are framework-managed:
- Primary key columns (id)
- Timestamp columns (createdAt, updatedAt)
- Soft delete columns (deletedAt)

Record the base entity column list for exclusion in Step 2.

---

### Step 2: Entity Column Inventory

Scan all entity files to build a complete column inventory.

**2a. Find entity files:**

Adapt glob patterns by ORM:

| ORM | Glob Patterns |
|-----|--------------|
| TypeORM | `backend/**/entities/*.entity.ts`, `backend/**/*.entity.ts` |
| Prisma | `prisma/schema.prisma` |
| Sequelize | `**/models/*.ts`, `**/models/*.js` |
| Django ORM | `**/models.py` |
| SQLAlchemy | `**/models/*.py`, `**/models.py` |

If module-scoped, filter to only entities under that module path.

**2b. Read each entity file and extract columns:**

For EACH entity file, read the full content and parse:

**TypeORM pattern:**
```
Grep: @Column\( in entity file -> extract property name, column options
Grep: @PrimaryGeneratedColumn\( -> mark as auto-generated (EXCLUDE)
Grep: @CreateDateColumn\(|@UpdateDateColumn\(|@DeleteDateColumn\( -> mark as framework-managed (EXCLUDE)
Grep: @ManyToOne\(|@OneToMany\(|@OneToOne\(|@ManyToMany\( -> mark as relation (EXCLUDE)
Grep: @JoinColumn\( -> mark as FK relation column (EXCLUDE)
Grep: extends BaseEntity|extends TypeOrmBaseEntity -> inherit base columns (EXCLUDE)
```

**For each column, record:**
```
{
  entityName: string,          // e.g., "User"
  propertyName: string,        // e.g., "firstName"
  columnName: string,          // e.g., "first_name" (DB column name)
  type: string,                // e.g., "varchar"
  nullable: boolean,
  hasDefault: boolean,
  isExcluded: boolean,         // true if BaseEntity/relation/auto-generated
  excludeReason: string|null,  // e.g., "BaseEntity column", "relation"
  filePath: string,
}
```

**2c. Build the auditable column list:**

Filter out all excluded columns. The remaining columns are "auditable" - they should
have at least one write path (DTO or programmatic assignment) to justify their existence.

---

### Step 3: Write Path Analysis - DTOs

For each entity with auditable columns, find all related DTOs and map which columns
have a DTO write path.

**3a. Find DTO files for each entity:**

```
Glob: backend/**/dtos/*{entity-name}*.dto.ts
Glob: backend/**/dto/*{entity-name}*.dto.ts
Glob: backend/**/*{entity-name}*.dto.ts
```

Also search for DTOs that reference the entity name (case-insensitive):
```
Grep: class.*Create.*{EntityName}|class.*{EntityName}.*Create in DTO files
Grep: class.*Update.*{EntityName}|class.*{EntityName}.*Update in DTO files
Grep: class.*Response.*{EntityName}|class.*{EntityName}.*Response in DTO files
```

**3b. Classify each DTO:**

| Pattern | Classification |
|---------|---------------|
| `Create{Entity}Dto`, `{Entity}CreateDto` | **Create DTO** |
| `Update{Entity}Dto`, `{Entity}UpdateDto` | **Update DTO** |
| `{Entity}ResponseDto`, `{Entity}Response` | **Response DTO** |
| `Register*Dto`, `Login*Dto`, `Submit*Dto` | **Specialized Create DTO** (treat as Create) |
| `PartialType(CreateDto)`, `OmitType(CreateDto, [...])` | **Derived DTO** - resolve inheritance |

**3c. Resolve DTO inheritance:**

NestJS commonly uses mapped types:
- `PartialType(CreateUserDto)` - all fields from CreateUserDto, all optional
- `OmitType(CreateUserDto, ['password'])` - all fields except password
- `PickType(CreateUserDto, ['username', 'email'])` - only specified fields
- `IntersectionType(A, B)` - merge fields from A and B
- `extends PartialType(OmitType(CreateDto, [...]))` - chained

When encountering these patterns:
1. Read the referenced DTO class
2. Apply the transformation (Partial, Omit, Pick, Intersection)
3. Determine the final set of field names in the derived DTO

**3d. Read each DTO and extract field names:**

```
Grep: @IsString|@IsNumber|@IsOptional|@IsNotEmpty|@IsEnum|@IsDateString|@ApiProperty in DTO files
```

For each DTO, list all property names defined in the class body.
For derived DTOs (PartialType/OmitType), compute the effective field list.

**3e. Cross-reference entity columns against DTOs:**

For each auditable entity column:
1. Check if `propertyName` exists in ANY Create DTO for that entity -> record `hasCreateDto: true/false`
2. Check if `propertyName` exists in ANY Update DTO for that entity -> record `hasUpdateDto: true/false`
3. Check if `propertyName` exists in ANY Response DTO for that entity -> record `inResponseDto: true/false`

---

### Step 4: Write Path Analysis - Programmatic Assignment

For each auditable entity column that is NOT in any Create/Update DTO, check if it
is set programmatically in service or repository code.

**4a. Find service and repository files for each entity:**

```
Glob: backend/**/{entity-module}/**/*.service.ts
Glob: backend/**/{entity-module}/**/*.repository.ts
Glob: backend/**/*.service.ts (for cross-module writes)
```

**4b. Search for direct property assignment:**

For each column `propertyName` of entity `EntityName`:

**Pattern A - Direct entity property assignment:**
```typescript
user.firstName = 'John';
entity.columnName = someValue;
```
Grep: `\.{propertyName}\s*=\s*` in service files

**Pattern B - Object literal in repository .create() or .save():**
```typescript
this.repository.create({ firstName: dto.firstName, ... });
this.repository.save({ ...dto, firstName: 'value' });
```
Grep: `{propertyName}\s*:` within 20 lines of `.create(` or `.save(` or `.insert(` or `.update(`

**Pattern C - Spread operator with the column name present in the spread source:**
```typescript
this.repository.save({ ...createDto }); // only counts if createDto HAS the property
```
If the spread source is a DTO already analyzed in Step 3, use the DTO field list.
Otherwise, flag as UNCERTAIN.

**Pattern D - QueryBuilder .set() or .values():**
```typescript
queryBuilder.set({ firstName: 'value' });
queryBuilder.values({ firstName: () => "'John'" });
```
Grep: `\.set\(\s*\{[^}]*{propertyName}` or `\.values\(\s*\{[^}]*{propertyName}`

**Pattern E - Raw SQL INSERT/UPDATE:**
```typescript
queryRunner.query(`UPDATE users SET first_name = $1`, [value]);
```
Grep: `{columnName}` (DB column name) in raw SQL strings within service/repository files

**4c. Search for write paths in OTHER modules' services:**

A column may be set by a different module's service (cross-module write).

```
Grep: \.{propertyName}\s*= in ALL backend/**/*.service.ts files
Grep: {propertyName}\s*: in ALL backend/**/*.service.ts files (within save/create/update context)
```

**4d. Check seeder files (informational only):**

```
Glob: backend/**/seeders/*.ts, backend/**/seeds/*.ts, backend/**/seed*.ts
```

If a column is ONLY set in seeders but never in application code, it is still
effectively dead in production. Note as INFO: "Only set in seeder, not in app code."

**4e. Record results:**

For each auditable column, record `hasProgrammaticWrite: true/false`
and `writeLocations: string[]` (file paths where writes were found).

---

### Step 5: Read Path Analysis (for Check 4 and Check 5)

For columns that have NO write path (from Steps 3-4), check if they are read anywhere.

**5a. Check Response DTOs (Check 4 - Response DTO Ghost):**

For columns with `inResponseDto: true` but `hasCreateDto: false` AND `hasUpdateDto: false`
AND `hasProgrammaticWrite: false`:

This column appears in API responses but will ALWAYS be null/undefined because nothing
ever sets its value. This is a "Response DTO ghost" - it wastes bandwidth and confuses
API consumers.

**5b. Check read-only usage (Check 5 - Read-Only Column):**

For columns with no write path, check if they are read in queries:

```
Grep: WHERE.*{propertyName}|where.*{propertyName} in repository/service files
Grep: ORDER.*BY.*{columnName}|orderBy.*{propertyName} in repository/service files
Grep: SELECT.*{columnName}|select.*{propertyName} in repository/service files
Grep: \.{propertyName} in service files (property access for business logic)
```

If a column has read usage but no write path, classify as **Info** - it might be
populated by a database trigger, migration, or external system not visible to static
analysis. Flag it for human review.

---

### Step 6: Execute All 5 Checks

Run each check using data collected in Steps 2-5 and collect findings.

---

#### Check 1: No Create DTO Path (HIGH)

For each auditable entity column:
- If `hasCreateDto: false` -> finding
- SKIP if `hasProgrammaticWrite: true` (column is system-managed, not user-created)

**Output format:**
```
| Entity | Column | Column Type | In Create DTO? | Programmatic Write? | Severity |
```

---

#### Check 2: No Update DTO Path (MEDIUM)

For each auditable entity column:
- If `hasUpdateDto: false` -> finding
- SKIP if column is immutable by design (e.g., `role` only changeable by admin)

**Output format:**
```
| Entity | Column | Column Type | In Update DTO? | Programmatic Write? | Severity |
```

---

#### Check 3: No Programmatic Write (HIGH)

For each auditable entity column:
- If `hasProgrammaticWrite: false` AND `hasCreateDto: false` AND `hasUpdateDto: false`:
  -> **CRITICAL** - true dead column, no write path exists
- If `hasProgrammaticWrite: false` AND (`hasCreateDto: true` OR `hasUpdateDto: true`):
  -> Not a finding for this check (DTO provides the write path)

**Output format:**
```
| Entity | Column | Any DTO? | Any Service Write? | Seeder Only? | Severity |
```

---

#### Check 4: Response DTO Ghost (MEDIUM)

For each auditable entity column where:
- `inResponseDto: true`
- `hasCreateDto: false`
- `hasUpdateDto: false`
- `hasProgrammaticWrite: false`

This column is exposed in API responses but always null.

**Output format:**
```
| Entity | Column | Response DTO | Always Null? | API Impact |
```

---

#### Check 5: Read-Only Column (INFO)

For each auditable entity column where:
- No write path exists (all false)
- BUT column is read in WHERE, ORDER BY, SELECT, or business logic

Flag for human review - may be populated by external system.

**Output format:**
```
| Entity | Column | Read Locations | Possible External Source | Action |
```

---

### Step 7: Composite Severity Assessment

After individual checks, compute composite severity for each column:

| Write Paths | Composite Severity | Label |
|-------------|-------------------|-------|
| No Create DTO + No Update DTO + No Service Write | **Critical** | True Dead Column |
| No Create DTO + No Update DTO + Has Service Write | **Low** | System-Managed (verify intentional) |
| No Create DTO + Has Update DTO + No Service Write | **High** | Created empty, updatable |
| Has Create DTO + No Update DTO + No Service Write | **Medium** | Write-once, immutable |
| In Response DTO + True Dead Column | **Critical** | Ghost in API responses |

---

### Step 8: Compile Report

After all checks are complete, compile the final audit report.

**Report structure:**

```markdown
# QA Dead Code (Dead Columns) Audit Report

**Date**: {current date}
**Scope**: {All modules | module name}
**Entities Scanned**: {count}
**Columns Analyzed**: {count} (excluding {excluded count} BaseEntity/relation/auto-gen columns)
**Dead Columns Found**: {count}

---

## Dead Column Summary

| Entity | Column | Type | Create DTO | Update DTO | Service Write | Response DTO | Verdict |
|--------|--------|------|------------|------------|---------------|-------------|---------|

Legend: YES = present | - = absent | ghost = in Response DTO but always null

---

## Summary

| # | Check | Severity | Issues Found |
|---|-------|----------|-------------|
| 1 | No Create DTO path | High | {count} |
| 2 | No Update DTO path | Medium | {count} |
| 3 | No programmatic write | High | {count} |
| 4 | Response DTO ghost | Medium | {count} |
| 5 | Read-only column | Info | {count} |
| | **Total** | | **{total}** |

### Composite Results

| Severity | Count | Columns |
|----------|-------|---------|
| Critical (true dead) | {n} | {list} |
| High | {n} | {list} |
| Medium | {n} | {list} |
| Low | {n} | {list} |
| Info | {n} | {list} |

---

## CRITICAL Issues

### True Dead Columns (No Write Path)

| # | Entity | Column | DB Column | Type | In Response DTO? | Impact |
|---|--------|--------|-----------|------|-----------------|--------|
{findings}

**Recommendation per finding:**
- **Option A (Remove):** Delete column from Entity, create migration to DROP COLUMN, remove from Response DTO
- **Option B (Populate):** Add to Create/Update DTO if the column should be user-settable
- **Option C (System-Manage):** Set programmatically in service if the column should be auto-computed

---

## HIGH Issues
{findings or "No issues found."}

---

## MEDIUM Issues
{findings or "No issues found."}

---

## INFO
{findings or "No issues found."}

---

## Recommendations

### Critical (fix immediately)
- {list}

### High (fix before next release)
- {list}

### Medium (fix when convenient)
- {list}
```

---

### Step 9: Append to TECH_DEBT.md

After generating the report, append a summary to the project's `TECH_DEBT.md` file.

**9a. Check if TECH_DEBT.md exists:**
```
Glob: TECH_DEBT.md (project root)
```

**9b. If file exists, APPEND (do not overwrite). If not, create with header.**

**Format to append:**
```markdown

---

## Dead Columns - {date}

> Auto-generated by `/qa-dead-code`. Re-run to update.

| Entity | Column | Severity | Recommendation |
|--------|--------|----------|----------------|

**Total: {n} dead columns across {m} entities**
```

---

## Scanning Patterns Reference

Quick reference for grep/glob patterns used throughout the audit:

| What to Find | Regex Pattern | File Scope |
|-------------|--------------|-----------|
| Entity files (TypeORM) | `@Entity\(` | backend/**/*.entity.ts |
| Entity files (Prisma) | `^model\s+\w+\s*\{` | prisma/schema.prisma |
| Column definitions (TypeORM) | `@Column\(` | entity files |
| Auto-generated PK | `@PrimaryGeneratedColumn\(` | entity files |
| Timestamp columns | `@CreateDateColumn\|@UpdateDateColumn\|@DeleteDateColumn` | entity files |
| Relation columns | `@ManyToOne\|@OneToMany\|@OneToOne\|@ManyToMany` | entity files |
| Join columns | `@JoinColumn\(` | entity files |
| Base entity inheritance | `extends BaseEntity` | entity files |
| Create DTOs | `class\s+Create.*Dto\|class\s+.*CreateDto` | DTO files |
| Update DTOs | `class\s+Update.*Dto\|class\s+.*UpdateDto` | DTO files |
| Response DTOs | `class\s+.*Response.*Dto\|class\s+.*ResponseDto` | DTO files |
| PartialType usage | `PartialType\(` | DTO files |
| OmitType usage | `OmitType\(` | DTO files |
| PickType usage | `PickType\(` | DTO files |
| Direct assignment | `\.{propertyName}\s*=` | service files |
| Object literal write | `{propertyName}\s*:` near `.save\(\|.create\(\|.update\(` | service files |
| QueryBuilder set | `\.set\(\s*\{[^}]*{propertyName}` | service/repository files |
| Raw SQL write | `INSERT.*{columnName}\|UPDATE.*SET.*{columnName}` | service/repository files |
| Seeder writes | `{propertyName}` | seeder files |
| Read usage (WHERE) | `where.*{propertyName}\|WHERE.*{columnName}` | repository/service files |
| Read usage (ORDER) | `orderBy.*{propertyName}\|ORDER BY.*{columnName}` | repository/service files |
| DTO validators | `@IsString\|@IsNumber\|@IsOptional\|@IsNotEmpty\|@IsEnum` | DTO files |

---

## DTO Inheritance Resolution

NestJS mapped types require special handling:

| Pattern | Resolution |
|---------|-----------|
| `PartialType(CreateDto)` | All fields from CreateDto, all optional |
| `OmitType(CreateDto, ['field1', 'field2'])` | All fields from CreateDto except listed |
| `PickType(CreateDto, ['field1', 'field2'])` | Only listed fields from CreateDto |
| `IntersectionType(DtoA, DtoB)` | Union of fields from both DTOs |
| `extends PartialType(OmitType(Create, [...]))` | Chain: first Omit, then make all optional |

To resolve:
1. Find the referenced DTO class
2. Read its fields
3. Apply the mapped type transformation
4. The result is the effective field list for comparison

---

## Exclusion Rules

These column types are ALWAYS excluded from dead column analysis:

| Column Type | Reason | Detection Pattern |
|-------------|--------|-------------------|
| Primary key | Auto-generated | `@PrimaryGeneratedColumn` |
| createdAt | Framework-managed | `@CreateDateColumn` |
| updatedAt | Framework-managed | `@UpdateDateColumn` |
| deletedAt | Framework-managed | `@DeleteDateColumn` |
| Relation properties | Not DB columns | `@ManyToOne`, `@OneToMany`, `@OneToOne`, `@ManyToMany` |
| Join columns | FK managed by ORM | `@JoinColumn` + adjacent relation decorator |
| Columns with `@Generated()` | Database-generated | `@Generated` |
| Computed/virtual columns | Not persisted | `@VirtualColumn` (TypeORM 0.3.12+) |

**BaseEntity inherited columns** are also excluded. Read the project's base entity
file (Step 1b) to determine the exact list.

---

## Important Caveats

1. **Static analysis only.** This skill cannot detect writes from database triggers,
   stored procedures, external ETL jobs, or admin tools that bypass the application.
   Columns flagged as dead may be populated by external systems.

2. **Spread operator limitations.** When a service does `this.repo.save({ ...dto })`,
   static analysis can only verify the write if the DTO's fields are known from Step 3.
   Unknown spread sources are flagged as UNCERTAIN.

3. **Cross-module writes.** A column may be written by a service in a different module
   (e.g., admin service updating user fields). Step 4c scans all service files, but
   complex indirect writes (via event handlers, queue consumers) may be missed.

4. **Derived DTOs.** NestJS's `PartialType`, `OmitType`, `PickType` create DTOs at
   runtime. The skill resolves these statically by reading the source DTO class.
   Deeply nested or dynamic derivations may not resolve correctly.

5. **Seeder-only writes.** If a column is only written in database seeders but never
   in application code, it will be flagged. Seeder data is development/testing only
   and does not represent a production write path.

6. **This tool is READ-ONLY.** It does not modify any files. All output is diagnostic
   only. Fixes must be applied manually by the developer.
