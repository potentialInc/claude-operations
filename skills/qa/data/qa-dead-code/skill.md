---
name: qa-dead-code
description: "Detect dead Entity columns - defined in ORM but never written via DTOs or service logic"
user-invocable: true
argument-hint: "[module]"
---

# QA Dead Code — Dead Column Detector

Detect Entity/Model columns that exist in the database schema but are never populated through any application code path, identifying storage waste, null API fields, and maintenance burden.

## Execution Mode

- **Standalone** (`/qa-dead-code [module]`): Diagnose-only. Scans the codebase, applies checks below, outputs a report. Does NOT modify files.
- **Via qa-fix** (`/qa-fix --check data`): qa-fix uses the checks below as its checklist for the data layer, then applies fixes.

Shared conventions (scoring, framework detection, output format): see `qa-shared/reference.md`.

## Checks

| # | Check | Severity | Description |
|---|-------|----------|-------------|
| 1 | No Create DTO path | **High** | Entity column not in any Create DTO for that entity |
| 2 | No Update DTO path | **Medium** | Entity column not in any Update DTO for that entity |
| 3 | No programmatic write | **High** | Entity column never assigned in any service/repository method |
| 4 | Response DTO ghost | **Medium** | Entity column in Response DTO but never populated (always null) |
| 5 | Read-only column | **Info** | Entity column only used in WHERE/ORDER BY/SELECT but never written |

**Composite severity escalation:**
- Column fails Check 1 + Check 2 + Check 3 = **Critical** (true dead column — no write path exists)
- Column fails Check 1 + Check 2 but passes Check 3 = **Low** (system-managed column — verify intentional)
- Column fails only Check 1 or only Check 2 = report at individual check severity

## Skill-Specific Patterns

### Exclusion Rules

These column types are always excluded from dead column analysis:

| Column Type | Detection Pattern |
|-------------|-------------------|
| Primary key | `@PrimaryGeneratedColumn` |
| createdAt / updatedAt / deletedAt | `@CreateDateColumn`, `@UpdateDateColumn`, `@DeleteDateColumn` |
| Relation properties | `@ManyToOne`, `@OneToMany`, `@OneToOne`, `@ManyToMany` |
| Join columns | `@JoinColumn` + adjacent relation decorator |
| Generated columns | `@Generated` |
| Virtual columns | `@VirtualColumn` |
| BaseEntity inherited columns | Detected via `extends BaseEntity` |

### DTO Inheritance Resolution

| Pattern | Resolution |
|---------|-----------|
| `PartialType(CreateDto)` | All fields from CreateDto, all optional |
| `OmitType(CreateDto, ['field1', 'field2'])` | All fields from CreateDto except listed |
| `PickType(CreateDto, ['field1', 'field2'])` | Only listed fields from CreateDto |
| `IntersectionType(DtoA, DtoB)` | Union of fields from both DTOs |
| `extends PartialType(OmitType(Create, [...]))` | Chain: first Omit, then make all optional |
