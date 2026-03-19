---
name: qa-db-integrity
description: Audit database schema consistency - Entity vs migrations, FK relations, index coverage, and migration gaps
user-invocable: true
argument-hint: "[module]"
---

# QA DB Integrity - Database Schema Consistency Auditor

## Purpose

Verify that ORM entities/models, migrations, and database schema expectations are in sync.
Detect missing migrations, orphaned columns, FK relationship issues, missing indexes,
and schema drift. Auto-detects ORM (TypeORM, Prisma, Sequelize, Django ORM, SQLAlchemy)
and adapts scanning patterns accordingly. This is a READ-ONLY static analysis tool that
does NOT connect to the actual database or modify any files.

## Usage

```
/qa-db-integrity              # Full audit across all modules
/qa-db-integrity products     # Audit a single module
/qa-db-integrity users        # Audit users module only
```

When a module argument is provided, scope all scanning to that module's directory
under the auto-detected backend modules directory for that module. When no argument is provided, scan all
entity and migration files project-wide.

## Overview of Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | Entity↔Migration drift | **Critical** | Entity column defined but not created in any migration |
| 2 | Orphan migration column | **High** | Migration creates column that no entity maps to |
| 3 | FK without index | **High** | Foreign key column has no corresponding index |
| 4 | Missing cascade config | **Medium** | Parent is deletable but child has no onDelete option |
| 5 | Nullable inconsistency | **High** | Entity nullable setting != migration nullable setting |
| 6 | Type mismatch | **High** | Entity column type != migration column type |
| 7 | Missing unique constraint | **Medium** | Semantically unique fields (username/email/slug) lack unique constraint |
| 8 | Soft delete gaps | **Medium** | Entity has deletedAt but some queries don't filter soft-deleted rows |

---

## Execution Algorithm

Follow these steps in order. Complete each step fully before proceeding to the next.

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
- Expected database patterns (e.g., "OTP records are ephemeral" → hard delete is acceptable)
- Which tables are expected to exist

**0b. Ask clarifying questions BEFORE proceeding:**

If you find entities or migrations for features not mentioned in requirements, ask:

> "I found [entity/table X] but it doesn't appear in the project requirements. Should I include it in the DB audit or skip it as demo/unused code?"

Wait for the user's response. Do NOT flag demo module inconsistencies as production bugs.

**0c. Build scope list:**
```
IN SCOPE: [entity1, entity2, ...]
OUT OF SCOPE: [entity3 (demo module), entity4 (deprecated), ...]
SPECIAL RULES: [entity5 (ephemeral, hard delete OK), ...]
```

---

### Step 1: Locate and Read Configuration

Before scanning entities and migrations, determine project configuration that affects
how schema synchronization works.

**1a. Find the data source configuration:**

```
Glob: **/data-source*.ts, **/ormconfig*.ts, **/typeorm*.ts, **/*.module.ts, prisma/schema.prisma
```

Read these files and extract:
- `synchronize` setting (true/false) — if `synchronize: true`, TypeORM auto-syncs
  schema in dev, so entity-migration drift is expected. Report as WARNING not CRITICAL.
- `entities` glob/array — confirms which entity files are loaded.
- `migrations` glob/array — confirms which migration files are applied.
- Database type (e.g., `postgres`, `mysql`, `sqlite`).

Record whether synchronize is enabled. This affects severity classification in Check 1.

**1b. Determine the module scope:**

If a module argument was provided:
- Validate that the module directory exists in the auto-detected backend path
- Scope entity scanning to that module directory
- Still scan ALL migrations (migrations may reference any table)

If no module argument:
- Scan all entity files project-wide
- Scan all migration files project-wide

---

### Step 2: Entity Inventory

Scan all entity files to build a structured inventory of the database schema as
defined by TypeORM decorators.

**2a. Find entity files:**

```
Glob: backend/**/entities/*.entity.ts
Glob: backend/**/*.entity.ts
```

Deduplicate results. If module-scoped, filter to only entities under that module path.

**2b. Read each entity file and extract the following:**

For EACH entity file, read the full file content and parse these elements:

**Table identity:**
- Class name (e.g., `Exercise`)
- Table name from `@Entity('table_name')` decorator. If no explicit name, derive by
  converting class name to snake_case (e.g., `OrderItem` -> `order_item`)
- Whether the class `extends BaseEntity` — if yes, it automatically has these columns:
  - `id` (uuid, primary key)
  - `createdAt` (timestamp)
  - `updatedAt` (timestamp)
  - `deletedAt` (timestamp, nullable) — soft delete column

**Column definitions — parse every `@Column(...)` decorator:**

For each column, extract:
- Property name in TypeScript (e.g., `title`)
- Column name: from `@Column({ name: 'db_column_name' })` or auto snake_case of property name
- Column type: from `@Column({ type: 'varchar' })` or `@Column('varchar')` or inferred from TS type
- nullable: from `@Column({ nullable: true/false })` — default is `false` if not specified
- default: from `@Column({ default: 'value' })` — record the default value
- length: from `@Column({ length: 255 })` — for varchar columns
- unique: from `@Column({ unique: true })` — boolean
- precision/scale: from `@Column({ precision: X, scale: Y })` — for decimal columns
- enum: from `@Column({ type: 'enum', enum: EnumType })` — record enum values if possible
- array: from `@Column({ array: true })` — PostgreSQL array column
- select: from `@Column({ select: false })` — excluded from default queries

**Primary key columns — parse `@PrimaryGeneratedColumn(...)`:**
- Strategy: 'uuid', 'increment', 'rowid'
- Column name (usually 'id')

**Relation definitions — parse each relation decorator:**

`@ManyToOne(() => ParentEntity, parent => parent.children, { options })`:
- Related entity class
- Inverse property name
- Options: cascade, onDelete, eager, nullable, orphanedRowAction

`@OneToMany(() => ChildEntity, child => child.parent, { options })`:
- Related entity class
- Inverse property name
- Options: cascade, eager

`@OneToOne(() => RelatedEntity, { options })`:
- Related entity class
- Options: cascade, onDelete, eager

`@ManyToMany(() => RelatedEntity, { options })`:
- Related entity class
- Options: cascade
- Associated `@JoinTable()` if this is the owning side

**Join columns — parse `@JoinColumn(...)` decorators:**
- `name`: the FK column name in this table (e.g., `'user_id'`)
- `referencedColumnName`: column in the referenced table (default `'id'`)

If `@JoinColumn` is absent on a `@ManyToOne`, TypeORM auto-generates the FK column
name as `{propertyName}Id` in camelCase -> snake_case (e.g., `user` -> `userId` -> `user_id`).

**Index definitions — parse `@Index(...)` decorators:**
- On the class level: `@Index(['column1', 'column2'])` — composite index
- On the class level: `@Index(['column1'], { unique: true })` — unique index
- On property level: `@Index()` — single column index
- Record column names and whether the index is unique

**Unique constraints — parse `@Unique(...)` decorators:**
- `@Unique(['column1', 'column2'])` — composite unique constraint
- Record column names

**2c. Build entity inventory data structure:**

For each entity, record:
```
{
  className: string,
  tableName: string,
  extendsBaseEntity: boolean,
  columns: [
    {
      propertyName: string,
      columnName: string,
      type: string,
      nullable: boolean,
      hasDefault: boolean,
      defaultValue: any,
      length: number | null,
      unique: boolean,
      precision: number | null,
      scale: number | null,
      isEnum: boolean,
      enumValues: string[] | null,
      isArray: boolean,
    }
  ],
  relations: [
    {
      type: 'ManyToOne' | 'OneToMany' | 'OneToOne' | 'ManyToMany',
      relatedEntity: string,
      propertyName: string,
      inverseProperty: string | null,
      cascade: boolean | string[],
      onDelete: string | null,
      eager: boolean,
      nullable: boolean | null,
    }
  ],
  joinColumns: [
    {
      propertyName: string,
      fkColumnName: string,
      referencedColumn: string,
      referencedTable: string,
    }
  ],
  indexes: [
    {
      columns: string[],
      unique: boolean,
    }
  ],
  uniqueConstraints: [
    {
      columns: string[],
    }
  ],
  filePath: string,
}
```

---

### Step 3: Migration Inventory

Scan all migration files to build a cumulative picture of the database schema as
defined by migrations.

**3a. Find migration files:**

```
Glob: backend/**/migrations/*.ts
Glob: **/database/migrations/*.ts
```

Deduplicate results. Always scan ALL migrations regardless of module scope,
because migrations may create tables for any module.

**3b. Sort migrations by timestamp:**

TypeORM migration filenames follow the pattern: `{timestamp}-{MigrationName}.ts`
(e.g., `1700000000000-CreateExerciseTable.ts`). Sort by the numeric timestamp prefix
to determine application order.

**3c. Read each migration file and parse the `up()` method:**

Migrations can use two styles. Detect and parse BOTH:

**Style 1: Raw SQL via queryRunner.query()**

```typescript
await queryRunner.query(`CREATE TABLE "exercise" (
  "id" uuid NOT NULL DEFAULT uuid_generate_v4(),
  "title" varchar(255) NOT NULL,
  "description" text,
  "sets" integer NOT NULL DEFAULT 3,
  "created_at" TIMESTAMP NOT NULL DEFAULT now(),
  CONSTRAINT "PK_exercise" PRIMARY KEY ("id")
)`);

await queryRunner.query(`ALTER TABLE "exercise" ADD "video_url" varchar(500)`);
await queryRunner.query(`ALTER TABLE "exercise" DROP COLUMN "old_field"`);
await queryRunner.query(`ALTER TABLE "exercise" ALTER COLUMN "title" SET NOT NULL`);
await queryRunner.query(`CREATE INDEX "IDX_exercise_title" ON "exercise" ("title")`);
await queryRunner.query(`CREATE UNIQUE INDEX "IDX_exercise_slug" ON "exercise" ("slug")`);
await queryRunner.query(`ALTER TABLE "exercise_log" ADD CONSTRAINT "FK_exercise_log_exercise"
  FOREIGN KEY ("exercise_id") REFERENCES "exercise"("id") ON DELETE CASCADE`);
```

Extract from CREATE TABLE statements:
- Table name
- Each column: name, type, NOT NULL / nullable, DEFAULT value
- PRIMARY KEY constraint
- UNIQUE constraints

Extract from ALTER TABLE statements:
- ADD column: column name, type, nullable, default
- DROP COLUMN: column name (mark as removed)
- ALTER COLUMN: changes to existing column properties

Extract from CREATE INDEX:
- Index name, table, column(s), whether UNIQUE

Extract from ADD CONSTRAINT ... FOREIGN KEY:
- FK column, referenced table, referenced column, ON DELETE action

**Style 2: TypeORM Table Builder API**

```typescript
await queryRunner.createTable(new Table({
  name: 'exercise',
  columns: [
    { name: 'id', type: 'uuid', isPrimary: true, generationStrategy: 'uuid', default: 'uuid_generate_v4()' },
    { name: 'title', type: 'varchar', length: '255', isNullable: false },
    { name: 'description', type: 'text', isNullable: true },
  ],
}));

await queryRunner.addColumn('exercise', new TableColumn({
  name: 'video_url', type: 'varchar', length: '500', isNullable: true,
}));

await queryRunner.dropColumn('exercise', 'old_field');

await queryRunner.createIndex('exercise', new TableIndex({
  name: 'IDX_exercise_title', columnNames: ['title'],
}));

await queryRunner.createForeignKey('exercise_log', new TableForeignKey({
  columnNames: ['exercise_id'],
  referencedTableName: 'exercise',
  referencedColumnNames: ['id'],
  onDelete: 'CASCADE',
}));
```

Extract equivalent data from these builder calls.

**3d. Build cumulative migration schema:**

Apply migrations in timestamp order to build the final expected schema state:
- Start with empty schema
- For each migration (in order):
  - CREATE TABLE -> add table with all columns
  - ADD column -> add column to existing table
  - DROP COLUMN -> remove column from table
  - ALTER COLUMN -> modify column properties
  - CREATE INDEX -> record index
  - DROP INDEX -> remove index
  - ADD FOREIGN KEY -> record FK constraint
  - DROP FOREIGN KEY -> remove FK constraint
  - DROP TABLE -> remove entire table

The result is the "migration-expected schema" — what the database should look like
after all migrations have been applied.

Record per table:
```
{
  tableName: string,
  columns: [
    {
      name: string,
      type: string,
      isNullable: boolean,
      defaultValue: string | null,
      length: string | null,
      isPrimary: boolean,
    }
  ],
  indexes: [
    {
      name: string,
      columns: string[],
      isUnique: boolean,
    }
  ],
  foreignKeys: [
    {
      columns: string[],
      referencedTable: string,
      referencedColumns: string[],
      onDelete: string,
    }
  ],
  migrationFile: string,  // last migration that modified this table
}
```

---

### Step 4: Cross-Reference Analysis

Now compare the entity inventory (Step 2) against the migration inventory (Step 3)
to detect inconsistencies.

**4a. Build column mapping:**

For each entity column, determine the expected database column name:
1. If `@Column({ name: 'explicit_name' })` -> use `explicit_name`
2. Otherwise, convert property name to snake_case:
   - `firstName` -> `first_name`
   - `videoUrl` -> `video_url`
   - `userId` -> `user_id`

For BaseEntity columns, map:
- `id` -> `id`
- `createdAt` -> `created_at`
- `updatedAt` -> `updated_at`
- `deletedAt` -> `deleted_at`

**4b. Build FK column mapping:**

For each `@ManyToOne` relation, determine the FK column name:
1. If `@JoinColumn({ name: 'explicit_fk' })` -> use `explicit_fk`
2. Otherwise, TypeORM generates: `{propertyName}Id` -> snake_case
   - Property `user` -> FK column `userId` -> `user_id`
   - Property `exercise` -> FK column `exerciseId` -> `exercise_id`
   - Property `parentCategory` -> FK column `parentCategoryId` -> `parent_category_id`

---

### Step 5: Execute All 8 Checks

Run each check and collect findings. Each finding should include the severity level,
the affected entity/table, the specific column or relation, and a description.

---

#### Check 1: Entity-Migration Drift (CRITICAL)

For each entity in the inventory:
1. Find the corresponding table in the migration schema (by table name)
2. If NO table exists in migrations -> report CRITICAL: entire table missing from migrations
3. For each entity column (SKIP BaseEntity columns: id, createdAt, updatedAt, deletedAt):
   - Look up the mapped column name in the migration table's columns
   - If NOT found -> report: entity defines column but no migration creates it
4. For each FK column (from @ManyToOne relations):
   - Look up the FK column name in the migration table's columns
   - If NOT found -> report: entity defines FK but no migration creates the column

**Severity adjustment:**
- If `synchronize: true` is detected -> report as WARNING instead of CRITICAL
- Add note: "synchronize:true auto-creates columns, but this is unsafe for production"

**Output format per finding:**
```
| Entity | Column | Type | In Migration? | Notes |
|--------|--------|------|---------------|-------|
| Exercise | videoUrl | varchar(500) | NO | Missing migration for this column |
| OrderItem | orderId (FK) | uuid | NO | FK column not in any migration |
```

---

#### Check 2: Orphan Migration Columns (HIGH)

For each table in the migration schema:
1. Find the corresponding entity (by table name)
2. If NO entity exists -> report: migration creates table with no corresponding entity
   - This might be intentional (e.g., join tables for ManyToMany) — note this
3. For each column in the migration table (SKIP BaseEntity columns):
   - Check if any entity column maps to this column name
   - Also check FK columns from relations
   - If NOT found in any entity -> report: orphaned migration column

**Exceptions to skip:**
- BaseEntity columns (id, created_at, updated_at, deleted_at)
- TypeORM auto-generated join table columns (for @ManyToMany)
- Columns in tables that have no corresponding entity (already reported above)

**Output format per finding:**
```
| Migration File | Table | Column | In Entity? | Notes |
|----------------|-------|--------|------------|-------|
| 1700000000000-Init.ts | exercise | old_field | NO | Column exists in DB but not in entity |
```

---

#### Check 3: FK Without Index (HIGH)

For each entity's @ManyToOne relation:
1. Determine the FK column name (from @JoinColumn or auto-generated)
2. Check if an index exists on this column:
   a. Entity-level: @Index() on the property or @Index(['fk_column']) on the class
   b. Migration-level: CREATE INDEX on this column
3. If NO index -> report: FK column without index

**Why this matters:**
PostgreSQL does NOT automatically create indexes on foreign key columns (unlike MySQL
InnoDB). Without an index on the FK column:
- JOIN queries do full table scans on the child table
- CASCADE DELETE operations scan the entire child table
- This becomes a serious performance issue as tables grow

**Note:** TypeORM's `@ManyToOne` does NOT auto-create an index. It only creates the
FK constraint. The developer must explicitly add `@Index()`.

**Output format per finding:**
```
| Entity | FK Column | Referenced Table | Has Index? | Impact |
|--------|-----------|-----------------|------------|--------|
| OrderItem | order_id | order | NO | Full table scan on JOINs |
| Comment | post_id | post | NO | Slow comment queries |
```

---

#### Check 4: Missing Cascade Config (MEDIUM)

For each @ManyToOne relation in every entity:
1. Get the parent entity class name
2. Check if the parent entity has any mechanism for deletion:
   - Extends BaseEntity -> has soft delete capability
   - Has a controller with DELETE endpoint
   - Has a service with `remove()` or `delete()` method
3. If parent is deletable, check the child's @ManyToOne options:
   - `onDelete: 'CASCADE'` -> child auto-deleted (OK)
   - `onDelete: 'SET NULL'` -> FK set to null (OK, verify column is nullable)
   - `onDelete: 'RESTRICT'` -> deletion blocked (OK, intentional protection)
   - `onDelete: 'NO ACTION'` -> same as RESTRICT in PostgreSQL (OK)
   - No `onDelete` specified -> defaults to 'NO ACTION' in PostgreSQL
     - This may cause unexpected FK constraint violation errors
     - Report as potential issue

**Additional check for SET NULL:**
If `onDelete: 'SET NULL'` is used, verify that the FK column is nullable:
- `@ManyToOne(() => Parent, { nullable: true, onDelete: 'SET NULL' })` -> OK
- `@ManyToOne(() => Parent, { nullable: false, onDelete: 'SET NULL' })` -> CONFLICT
  Cannot SET NULL on a NOT NULL column — this will cause a runtime error.

**Output format per finding:**
```
| Child Entity | Relation Property | Parent Entity | onDelete | Risk |
|-------------|-------------------|---------------|----------|------|
| OrderItem | order | Order | (none) | FK violation on parent delete |
| Comment | post | Post | SET NULL | FK column is NOT NULL — will error |
```

---

#### Check 5: Nullable Inconsistency (HIGH)

For each entity column that also exists in the migration schema:
1. Get the entity's nullable setting:
   - `@Column({ nullable: true })` -> nullable
   - `@Column({ nullable: false })` -> not nullable
   - `@Column()` with no nullable -> defaults to NOT nullable (false)
2. Get the migration's nullable setting:
   - `isNullable: true` in Table builder -> nullable
   - `isNullable: false` or absent -> not nullable
   - Raw SQL: presence or absence of `NOT NULL`
3. Compare:
   - Entity says nullable BUT migration says NOT NULL -> MISMATCH
     - Risk: TypeORM may try to insert NULL, causing DB constraint error
   - Entity says NOT NULL BUT migration says nullable -> MISMATCH
     - Risk: TypeORM validation passes but DB allows NULL, data integrity issue

**Output format per finding:**
```
| Entity | Column | Entity Nullable | Migration Nullable | Risk |
|--------|--------|----------------|-------------------|------|
| Exercise | description | true | false | INSERT will fail if NULL sent |
| User | phone | false (default) | true | DB allows NULL despite entity constraint |
```

---

#### Check 6: Type Mismatch (HIGH)

For each entity column that also exists in the migration schema:
1. Get the entity's TypeORM type and normalize it
2. Get the migration's PostgreSQL type and normalize it
3. Compare using this mapping:

```
TypeORM Entity Type    ->  PostgreSQL Migration Type(s)
------------------------------------------------------
'varchar'              ->  varchar, character varying
'text'                 ->  text
'int', 'integer'       ->  integer, int, int4
'bigint'               ->  bigint, int8
'smallint'             ->  smallint, int2
'float', 'double'      ->  double precision, float8
'real'                 ->  real, float4
'decimal', 'numeric'   ->  numeric, decimal
'boolean', 'bool'      ->  boolean, bool
'timestamp'            ->  timestamp, timestamp without time zone
'timestamptz'          ->  timestamptz, timestamp with time zone
'date'                 ->  date
'time'                 ->  time, time without time zone
'timetz'               ->  timetz, time with time zone
'json'                 ->  json
'jsonb'                ->  jsonb
'uuid'                 ->  uuid
'bytea'                ->  bytea
'enum'                 ->  enum (check user-defined type name)
'simple-array'         ->  text (TypeORM stores as comma-separated)
'simple-json'          ->  text (TypeORM stores as JSON string)
'simple-enum'          ->  enum or varchar (depends on DB)
```

**Length comparison:**
- Entity `varchar(255)` vs migration `varchar(500)` -> MISMATCH
- Entity `varchar` (no length) vs migration `varchar(255)` -> POSSIBLE MISMATCH (note it)

**Precision/scale comparison for decimal:**
- Entity `decimal(10,2)` vs migration `numeric(10,4)` -> MISMATCH (scale differs)

**Output format per finding:**
```
| Entity | Column | Entity Type | Migration Type | Risk |
|--------|--------|------------|----------------|------|
| Exercise | duration | float | integer | Decimal values will be truncated |
| User | age | varchar(50) | varchar(255) | Length mismatch, potential truncation |
```

---

#### Check 7: Missing Unique Constraint (MEDIUM)

Scan all entity columns for fields that semantically should be unique:

**High-confidence patterns** (almost always need unique):
- Column name contains `username` (case-insensitive)
- Column name contains `email` (case-insensitive)
- Column name is exactly `slug`

**Medium-confidence patterns** (often need unique):
- Column name contains `code` and is not `zip_code` or `postal_code`
- Column name contains `phone` or `mobile`
- Column name is `ssn`, `national_id`, `license_number`

For each matching column, check if uniqueness is enforced:
1. `@Column({ unique: true })` -> has unique constraint (OK)
2. `@Unique(['column_name'])` on the class -> has unique constraint (OK)
3. `@Index(['column_name'], { unique: true })` -> has unique index (OK)
4. Migration creates unique index on this column -> has unique constraint (OK)
5. None of the above -> MISSING unique constraint

**Additional business-logic check:**
Search service files for uniqueness validation without DB constraint:
```
Grep: findOne.*where.*username|findOne.*where.*email|ConflictException|already exists
```
If the service manually checks for duplicates but the DB has no unique constraint,
there is a race condition risk — two concurrent requests could both pass the check
and insert duplicate values.

**Output format per finding:**
```
| Entity | Column | Semantic Pattern | Has Unique? | Confidence |
|--------|--------|-----------------|------------|------------|
| User | username | *username* | NO | High — almost certainly needs unique |
| User | email | *email* | NO | High — almost certainly needs unique |
| Exercise | slug | slug | NO | High — slugs must be unique |
| Coach | licenseCode | *code* | NO | Medium — may need unique |
```

---

#### Check 8: Soft Delete Gaps (MEDIUM)

For each entity that extends BaseEntity (and therefore has `deletedAt` / soft delete):

**8a. Check repository files:**

```
Glob: backend/**/repositories/*.repository.ts
Glob: backend/**/*.repository.ts
```

Read each repository file and look for queries that might return soft-deleted records:

- `createQueryBuilder()` chains that do NOT include:
  - `.where('entity.deletedAt IS NULL')` or
  - `.andWhere('entity.deletedAt IS NULL')` or
  - Use of TypeORM's built-in soft delete filtering

- Direct `queryRunner.query()` or raw SQL that:
  - SELECTs from a soft-deletable table
  - Does NOT include `WHERE deleted_at IS NULL` or `WHERE "deleted_at" IS NULL`

- `.find()`, `.findOne()`, `.findAndCount()` calls are OK — TypeORM automatically
  filters soft-deleted records for these methods unless `.withDeleted()` is used.

- `.withDeleted()` usage -> explicitly includes soft-deleted records. This is
  intentional and should be noted but NOT flagged as an error.

**8b. Check service files:**

```
Glob: backend/**/services/*.service.ts
Glob: backend/**/*.service.ts
```

Look for the same patterns as in repositories.

**8c. Check for hard delete usage:**

Search for `.delete()` or `queryRunner.query('DELETE FROM ...')` on entities that
have soft delete. These bypass soft delete:
- `.delete()` -> hard delete, should use `.softDelete()` or `.remove()`
- `DELETE FROM table` -> hard delete via raw SQL

**Output format per finding:**
```
| Entity | File | Location | Issue | Risk |
|--------|------|----------|-------|------|
| Product | product.repository.ts | line 45 | createQueryBuilder without deletedAt filter | May return deleted records |
| User | user.service.ts | line 102 | .delete() used instead of .softDelete() | Hard deletes user record |
| OrderItem | order-item.repo.ts | line 78 | Raw SQL without deleted_at IS NULL | Returns deleted items |
```

---

### Step 6: Compile Report

After all 8 checks are complete, compile the final audit report.

**Report structure:**

```markdown
# QA DB Integrity Audit Report

**Date**: {current date}
**Scope**: {All modules | module name}
**Entities Scanned**: {count}
**Migrations Scanned**: {count}
**Synchronize Mode**: {true (WARNING) | false}

---

## Summary

| # | Check | Severity | Issues Found |
|---|-------|----------|-------------|
| 1 | Entity-Migration drift | Critical | {count} |
| 2 | Orphan migration columns | High | {count} |
| 3 | FK without index | High | {count} |
| 4 | Missing cascade config | Medium | {count} |
| 5 | Nullable inconsistency | High | {count} |
| 6 | Type mismatch | High | {count} |
| 7 | Missing unique constraint | Medium | {count} |
| 8 | Soft delete gaps | Medium | {count} |
| | **Total** | | **{total}** |

---

## CRITICAL Issues

### Check 1: Entity-Migration Drift

{If synchronize:true, display warning banner}

| Entity | Column | Type | In Migration? | Notes |
|--------|--------|------|---------------|-------|
{findings or "No issues found."}

---

## HIGH Issues

### Check 2: Orphan Migration Columns

| Migration File | Table | Column | In Entity? | Notes |
|----------------|-------|--------|------------|-------|
{findings or "No issues found."}

### Check 3: FK Without Index

| Entity | FK Column | Referenced Table | Has Index? | Impact |
|--------|-----------|-----------------|------------|--------|
{findings or "No issues found."}

### Check 5: Nullable Inconsistency

| Entity | Column | Entity Nullable | Migration Nullable | Risk |
|--------|--------|----------------|-------------------|------|
{findings or "No issues found."}

### Check 6: Type Mismatch

| Entity | Column | Entity Type | Migration Type | Risk |
|--------|--------|------------|----------------|------|
{findings or "No issues found."}

---

## MEDIUM Issues

### Check 4: Missing Cascade Config

| Child Entity | Relation Property | Parent Entity | onDelete | Risk |
|-------------|-------------------|---------------|----------|------|
{findings or "No issues found."}

### Check 7: Missing Unique Constraint

| Entity | Column | Semantic Pattern | Has Unique? | Confidence |
|--------|--------|-----------------|------------|------------|
{findings or "No issues found."}

### Check 8: Soft Delete Gaps

| Entity | File | Location | Issue | Risk |
|--------|------|----------|-------|------|
{findings or "No issues found."}

---

## Recommendations

{Prioritized list of fixes, grouped by severity}

### Critical (fix immediately)
- {list}

### High (fix before next release)
- {list}

### Medium (fix when convenient)
- {list}
```

---

## Scanning Patterns Reference

Quick reference for the Grep patterns used throughout the audit:

| What to Find | Regex Pattern | File Scope |
|-------------|--------------|-----------|
| Entity files | `@Entity\(` | backend/**/*.entity.ts |
| Column definitions | `@Column\(` | entity files |
| Primary keys | `@PrimaryGeneratedColumn\(` | entity files |
| ManyToOne relations | `@ManyToOne\(` | entity files |
| OneToMany relations | `@OneToMany\(` | entity files |
| OneToOne relations | `@OneToOne\(` | entity files |
| ManyToMany relations | `@ManyToMany\(` | entity files |
| Join columns | `@JoinColumn\(` | entity files |
| Join tables | `@JoinTable\(` | entity files |
| Indexes | `@Index\(` | entity files |
| Unique constraints | `@Unique\(` | entity files |
| Unique column option | `unique:\s*true` | entity files |
| Migration classes | `class.*implements MigrationInterface` | migration files |
| Create table (SQL) | `CREATE TABLE` | migration files |
| Create table (builder) | `createTable` | migration files |
| Add column (SQL) | `ALTER TABLE.*ADD` | migration files |
| Add column (builder) | `addColumn` | migration files |
| Drop column (SQL) | `DROP COLUMN` | migration files |
| Drop column (builder) | `dropColumn` | migration files |
| Create index (SQL) | `CREATE\s+(UNIQUE\s+)?INDEX` | migration files |
| Create index (builder) | `createIndex` | migration files |
| Foreign key (SQL) | `FOREIGN KEY` | migration files |
| Foreign key (builder) | `createForeignKey` | migration files |
| Synchronize setting | `synchronize:\s*(true\|false)` | data-source, ormconfig, *.module.ts |
| BaseEntity usage | `extends BaseEntity` | entity files |
| Soft delete column | `deletedAt\|deleted_at` | entity, repository, service files |
| WithDeleted flag | `withDeleted\(\)` | repository, service files |
| Hard delete | `\.delete\(\|DELETE FROM` | repository, service files |
| QueryBuilder | `createQueryBuilder\(` | repository, service files |
| Raw SQL queries | `queryRunner\.query\(\|\.query\(` | repository, service, migration files |
| Uniqueness checks | `findOne.*where.*username\|ConflictException\|already exists` | service files |

---

## TypeORM to PostgreSQL Type Mapping

| TypeORM Entity Type | PostgreSQL Equivalent(s) |
|---------------------|-------------------------|
| `varchar` | `character varying`, `varchar` |
| `text` | `text` |
| `int`, `integer` | `integer`, `int4` |
| `bigint` | `bigint`, `int8` |
| `smallint` | `smallint`, `int2` |
| `float`, `double` | `double precision`, `float8` |
| `real` | `real`, `float4` |
| `decimal`, `numeric` | `numeric`, `decimal` |
| `boolean`, `bool` | `boolean`, `bool` |
| `timestamp` | `timestamp without time zone` |
| `timestamptz` | `timestamp with time zone` |
| `date` | `date` |
| `time` | `time without time zone` |
| `timetz` | `time with time zone` |
| `uuid` | `uuid` |
| `json` | `json` |
| `jsonb` | `jsonb` |
| `bytea` | `bytea` |
| `enum` | `enum` (user-defined type) |
| `simple-array` | `text` (comma-separated) |
| `simple-json` | `text` (JSON string) |
| `simple-enum` | `enum` or `varchar` |

---

## Important Caveats

1. **Static analysis only.** This skill does NOT connect to the live database. It
   compares entity decorators against migration file contents. Actual database state
   may differ if migrations were manually run, skipped, or modified.

2. **synchronize:true.** When TypeORM's synchronize mode is enabled (common in dev),
   the framework auto-creates/modifies tables to match entities. Entity-migration
   drift is expected in this case. However, synchronize should NEVER be true in
   production. The audit flags this as a warning.

3. **BaseEntity columns.** Columns inherited from BaseEntity (id, createdAt,
   updatedAt, deletedAt) are excluded from drift checks because they are defined
   once in the base class and should be handled by an initial migration.

4. **Eager loading.** Relations with `eager: true` are noted for N+1 query awareness
   but are not flagged as errors. They can cause performance issues with large
   datasets but are a design choice, not a schema consistency issue.

5. **ManyToMany join tables.** TypeORM auto-generates join tables for @ManyToMany
   relations with @JoinTable. These tables may not have explicit entity classes and
   should not be flagged as orphaned.

6. **Enum types.** PostgreSQL enums are user-defined types created separately from
   the table. Enum type mismatches may require checking CREATE TYPE statements in
   migrations in addition to column definitions.

7. **Column defaults.** TypeORM and PostgreSQL handle defaults differently. TypeORM
   may set defaults at the application level (before INSERT), while migration defaults
   are set at the database level (DEFAULT clause). Both are valid approaches but
   may cause confusion if they differ.

8. **This tool is READ-ONLY.** It does not modify any files, create migrations,
   or alter the database. All output is diagnostic only. Fixes must be applied
   manually by the developer.
