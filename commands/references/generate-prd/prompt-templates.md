# PRD Generation v3 — Agent Prompt Templates

This document defines the 6 agents for the v3 PRD generation command. Unlike v2 (10 agents, 3 modes: write/review/enhance), v3 simplifies to 6 agents with write-only mode — no cross-review rounds.

---

## Model Tiers

| Tier | Model | Usage |
|------|-------|-------|
| Strategy | opus | Judgment/synthesis roles (prd-writer, tech-writer) |
| Execution | sonnet | Execution/validation roles (parser, qa, support) |

---

## 1. parser (sonnet)

**Role:** Parse client input into 3 structured intermediate artifacts

**Tools:** Read, Write, Glob, Grep

**Input:** Client answer file (any text format)

**Output:**
- `parsed-input.md`
- `screen-inventory.md`
- `tbd-items.md`

### Write-Mode Rules

1. Extract ALL information from client input, organize by category.
2. Infer screens from features — each feature implies one or more screens.
3. For each mentioned feature: map to route + page name + user type + access group.
4. Detect adaptive complexity flags:
   - SaaS
   - File upload
   - Real-time
   - Billing
   - Chat
5. List all unclear/missing items in `tbd-items.md` with source and reason.
6. Do NOT guess or fill — only extract what is explicitly or strongly implied in input.
7. `parsed-input.md` header must include a "Detected Features" checklist.

### Output Format — parsed-input.md

```markdown
# Parsed Input

## Detected Features
- [ ] Feature A
- [ ] Feature B
- ...

## Adaptive Complexity Flags
| Flag | Detected | Source |
|------|----------|--------|

## Categorized Information
### [Category]
- ...
```

### Output Format — screen-inventory.md

```markdown
# Screen Inventory

| # | Feature | Route | Page Name | User Type | Access Group |
|---|---------|-------|-----------|-----------|--------------|
```

### Output Format — tbd-items.md

```markdown
# TBD Items — {ProjectName}

Items that are unclear, missing, or require client decision before full PRD can be written.
After PRD generation, each item is marked with its resolution status.

**Status Legend:**
- ✅ PRD — Recommendation applied in PRD
- ⏳ Deferred — Postponed to future version
- ❌ Not in PRD — Client decision required

---

## {Category Name}

- **{ID}. {Item name}**
  - Options: {possible choices / context}
  - Status: _(empty — filled in Phase 3)_
```

Phase 1 (parser) leaves Status empty. Phase 3 (integrate) back-fills with:
- `✅ PRD: {chosen value}` — recommendation applied in PRD
- `⏳ Deferred: {version}` — postponed to future version
- `❌ Not in PRD` — not addressed, needs client decision

Document ends with auto-generated summary section (include Options so clients can decide immediately):
```markdown
## ⚠️ Unresolved Items — Client Decision Required

| # | Item | Options | Category |
|---|------|---------|----------|

**Total: {N} items — ✅ {n} resolved / ⏳ {n} deferred / ❌ {n} unresolved**
```

---

## 2. prd-writer (opus)

**Role:** Write Section 0-4 (Overview, Terminology, System Modules, User App, Admin)

**Tools:** Read, Glob, Grep

**Input:**
- `parsed-input.md`
- `screen-inventory.md`
- `tbd-items.md`
- `bug-patterns-filtered.md` (if exists)

**Reference:**
- `depth-guide.md` (Section 0-4)
- `prd-template.md` (Section 0-4)
- `admin-standards.md`

### Write-Mode Rules

#### Section 0 — Overview

- User Types table with 4 columns: Type / Description / Permissions Summary / Status
- User Status table with 3 columns: Status / Description / Allowed Actions
- MVP Scope definition
- Design Reference

#### Section 1 — Terminology

- All domain-specific terms (not generic IT terms)
- Terms MUST match the exact terms used in Section 3 and Section 4
- Format: Term / Definition / Used In (section references)

#### Section 2 — System Modules

Per module:

1. **Features** — bullet list of capabilities
2. **Technical Flow (API Hints)** — 5-10 steps, success + failure branches, API endpoint hints
3. **Real Scenario** — success case + failure case
   - Format: Named user with role -> specific actions -> server processing -> UI result
4. **WebSocket Events table** — for real-time modules only
   - Columns: Event / Direction / Payload / Trigger
5. **3rd Party API List** — at the end of Section 2
   - Columns: Service / Purpose / Endpoint Pattern / Auth Method

#### Section 3 — User App

- **Route Groups table:** Public / Auth / Protected
- **Page Map tables** per group
  - Columns: Route / Page Name / Component / Access
- Per route, ALL 8 mandatory items:
  1. Component detail
  2. Input validation
  3. State screens (loading, empty, error, success)
  4. Interactions (click, hover, drag, etc.)
  5. Navigation (where it leads, back behavior)
  6. Data display (what data, format, sorting)
  7. Error handling (per-field, toast, modal, retry)
  8. Edge cases (minimum 4: zero state, max limit, concurrent access, offline)
- Component states: default / active / disabled / error
- Input validation table per form:
  - Columns: Field / Type / Required / Min-Max / Pattern / Error Message
- Access role + ownership rules per route
- Known Risk table (if `bug-patterns-filtered.md` exists)

#### Section 4 — Admin

- Per admin page:
  1. **Table Column Definition** — Column / Type / Sortable / Filterable
  2. **Filters** — Field / Type / Options
  3. **Detail Drawer** — fields and layout
  4. **Creation Modal** — fields with validation
  5. **Standard Features** — apply all required items from `admin-standards.md`
  6. **Edge cases** per admin page
  7. **Known Risk table** (if `bug-patterns-filtered.md` exists)

#### General Rules

- TBD handling: fill with best-practice defaults, mark each with `[Recommended]`
- NEVER guess beyond what is in the input file
- Follow `prd-template.md` format strictly

---

## 3. tech-writer (opus)

**Role:** Write Section 5-7 (Tech Stack & System Design, Data Model — Full Schema, Permission Matrix)

**Tools:** Read, Glob, Grep

**Input:**
- `parsed-input.md`
- `screen-inventory.md`
- `tbd-items.md`
- Section 3/4 drafts (when available)

**Reference:**
- `depth-guide.md` (Section 5-7)
- `prd-template.md` (Section 5-7)

### Write-Mode Rules

#### Section 5 — Tech Stack & System Design

- **Technologies table** — Layer / Technology / Version / Purpose
- **Third-Party Integrations** — Service / Purpose / SDK/API
- **Key Decisions** — Decision / Rationale / Alternatives Considered
- **Environment Variables** — Variable / Required / Description / Example

Conditional sections (only include when detected in `parsed-input.md`):

| Flag | Section | Contents |
|------|---------|----------|
| Auth | Auth Flow | Method, token strategy, refresh logic, social login providers |
| File upload | File Pipeline | Size limits, allowed types, processing steps, storage provider |
| Real-time | Real-time Architecture | Transport (WS/SSE), channel structure, reconnection strategy |
| Billing | Billing Flow | Provider, plan types, webhook events, trial/grace period |
| SaaS | Multi-tenancy | Isolation strategy, tenant identification, cross-tenant security |

#### Section 6 — Data Model (Full Schema)

1. **Entity Relationships**
   - All 1:1, 1:N, N:N, self-referencing relationships
   - Include join table names for N:N
   - Format: `EntityA (1) --- (N) EntityB` or `EntityA (N) --- join_table --- (N) EntityB`

2. **Full Schema** — per entity:
   - Column / Type / Constraints table
   - Minimum 5 columns per entity (excluding timestamps)
   - PK with generation strategy (e.g., UUID v4, auto-increment)
   - FK with `(FK -> entity.column)` notation
   - Constraints: NOT NULL, UNIQUE, DEFAULT values
   - `created_at`, `updated_at` timestamps on every entity

3. **Status Enums table** — matching terms from Section 1
   - Columns: Enum Name / Values / Used By (entity)

4. **Index Hints table**
   - Columns: Entity / Column(s) / Type / Reason

5. **Soft Delete table**
   - Columns: Entity / Soft Delete (yes/no) / Retention Period

#### Section 7 — Permission Matrix

1. **Action x Role Matrix**
   - All CRUD resources x all roles
   - No empty cells allowed
   - Notation: `Y` (allowed) / `N` (denied) / `Y(own)` (allowed for owned resources only)

2. **Ownership Rules**
   - Resource / Owner Field / Definition

3. **Role Hierarchy**
   - Role / Inherits From / Additional Permissions

#### General Rules

- TBD handling: fill with best-practice defaults, mark each with `[Recommended]`
- Extract entities from Section 3/4 drafts; fall back to `screen-inventory.md` if drafts are not yet available

---

## 4. qa (sonnet)

**Role:** Validate PRD against 14 machine-verifiable rules

**Tools:** Read, Glob, Grep

**Input:**
- Integrated PRD draft
- `screen-inventory.md`
- `parsed-input.md`

### Validation Rules

| # | Rule | What It Checks |
|---|------|----------------|
| 1 | Route Count Match | Number of routes in Page Map = number of routes detailed below |
| 2 | Mandatory 8 Items | Every route has all 8 mandatory items (component detail, input validation, state screens, interactions, navigation, data display, error handling, edge cases) |
| 3 | Admin 1:1 Mapping | Every user-facing entity has a corresponding admin page |
| 4 | Admin Standard Features | Each admin page includes all required standard features from admin-standards.md |
| 5 | Terminology Cross-Reference | Every term in Section 1 is used in Section 3 or 4; no undefined terms appear in Section 3/4 |
| 6 | TBD Zero | No unresolved TBD markers remain (all must be filled with `[Recommended]` or resolved) |
| 7 | Route Coverage | Page Map routes match Feature List routes — no orphans in either direction |
| 8 | Numeric Consistency | Numbers cited in overview match actual counts in detail sections |
| 9 | Tech Stack Completeness | All technologies referenced in the PRD appear in the Tech Stack table |
| 10 | Entity Coverage | All entities referenced in Section 3/4 have a full schema in Section 6 |
| 11 | Permission Completeness | All CRUD resources x all roles have entries in the Permission Matrix — no empty cells |
| 12 | Client Requirement Tracking | Every item in parsed-input.md is addressed somewhere in the PRD |
| 13 | Real Scenario Existence | Every module in Section 2 has at least one success + one failure real scenario |
| 14 | Full Schema Completeness | Every entity has minimum 5 columns (excl. timestamps), PK, and created_at/updated_at |

### Output Format

```markdown
## QA Validation Results

| # | Rule | Verdict | Evidence |
|---|------|---------|----------|
| 1 | Route Count Match | PASS/FAIL | [specific count or discrepancy] |
| 2 | Mandatory 8 Items | PASS/FAIL | [missing items and which routes] |
| 3 | Admin 1:1 Mapping | PASS/FAIL | [missing admin pages] |
| 4 | Admin Standard Features | PASS/FAIL | [missing features and which pages] |
| 5 | Terminology Cross-Reference | PASS/FAIL | [orphan or undefined terms] |
| 6 | TBD Zero | PASS/FAIL | [remaining TBD locations] |
| 7 | Route Coverage | PASS/FAIL | [orphan routes] |
| 8 | Numeric Consistency | PASS/FAIL | [mismatched numbers] |
| 9 | Tech Stack Completeness | PASS/FAIL | [missing technologies] |
| 10 | Entity Coverage | PASS/FAIL | [entities without schema] |
| 11 | Permission Completeness | PASS/FAIL | [empty cells in matrix] |
| 12 | Client Requirement Tracking | PASS/FAIL | [unaddressed requirements] |
| 13 | Real Scenario Existence | PASS/FAIL | [modules missing scenarios] |
| 14 | Full Schema Completeness | PASS/FAIL | [entities missing columns/PK/timestamps] |

**Overall: PASS / FAIL**
```

### Rules

- NO subjective judgment ("needs more detail", "could be better")
- ONLY counting, existence, and matching checks
- Each FAIL must include specific evidence: which item, where, what is missing
- PASS verdict requires ALL 14 rules to pass

---

## 5. support (sonnet)

**Role:** Fix QA FAIL items

**Tools:** Read, Write, Edit, Glob, Grep

**Input:**
- Integrated PRD draft
- QA validation results

### Fix Rules

1. ONLY fix items marked FAIL in QA results.
2. Do NOT modify PASS sections.
3. Do NOT add new content beyond what is needed to fix FAILs.
4. Maintain existing format and structure.
5. After fixing: output the complete fixed PRD.
6. Track what was fixed in a summary.

### Output Format

```markdown
## Support Fix Summary

| # | Rule | Fix Applied |
|---|------|-------------|
| [n] | [Rule Name] | [Description of what was fixed] |

---

[Complete fixed PRD follows]
```

---

## 6. checklist (sonnet)

**Role:** Generate client preparation checklist as a standalone deliverable

**Tools:** Read, Write, Glob, Grep, Bash

**Input:**
- `parsed-input.md` (App Type + Adaptive Complexity Flags + 3rd party integrations)

**Reference:**
- `prd-template.md` (Section 9 — Client Checklist Template)

**External:**
- Notion SOP Database (queried at runtime via API)

**Output:**
- `.claude-project/prd/{ProjectName}/intermediate/client-checklist.md`

### Write-Mode Rules

1. Read `parsed-input.md` to determine:
   - App Type (Web / iOS / Android / Web+App)
   - Detected features (social login, payment, SMS, email, map, video, AI, chat)
   - Specific 3rd party services mentioned
   - Target market (Korea, global, EU — affects provider choices)

2. **Account items — dev team handles setup**:
   - For accounts (AWS, Apple, Google, social login providers, PG, etc.): only tell client to **create account and share access**
   - Do NOT include setup instructions, configuration steps, API key generation, or technical details
   - Only include **decision points** the client must make (e.g., individual vs organization account — with comparison table)
   - Replace all `[서비스명]` placeholders with actual recommended services
   - Mark recommendations with `💡`

3. **Single flat list** — NO phase/stage separation:
   - All items collected upfront — group by type (계정, 브랜드 소재, 스토어 제출물, 법률 문서, 콘텐츠)
   - Do NOT split into 1단계/2단계/3단계

4. **Conditional inclusion by App Type**:
   - Web → 파비콘, OG 이미지
   - iOS → Apple 개발자 계정, iOS 앱스토어 제출물
   - Android → Google Play 개발자 계정 (⚠️ 개인 계정은 공개 출시 전 20명 이상의 테스터와 최소 14일 이상 비공개 테스트 필수 — 반드시 명시), Google Play 스토어 제출물

5. **Conditional inclusion by detected features**:
   - Social login → 해당 제공자 계정만 (카카오, 네이버, Google, Apple, LINE, Facebook 등)
   - Billing/payment → PG사 또는 Stripe 계정
   - SMS/OTP → SMS 발신번호 등록 (한국: KISA)
   - Map → 지도 서비스 계정
   - Video/chat → 영상/채팅 서비스 계정
   - AI → AI 서비스 계정

6. **Conditional inclusion by content needs**:
   - Pre-existing content → 초기 데이터, 관리자 계정
   - Multi-language → 번역 파일

7. **SOP link attachment** (Notion API lookup):
   - After generating all checklist items, query the Notion SOP database to find matching guides
   - **Database ID**: `15ab6d88d2cf8042a9effff908507e5f`
   - **API call**: Use Bash to run:
     ```bash
     curl -s -X POST 'https://api.notion.com/v1/databases/15ab6d88d2cf8042a9effff908507e5f/query' \
       -H "Authorization: Bearer $NOTION_API_KEY" \
       -H 'Notion-Version: 2022-06-28' \
       -H 'Content-Type: application/json' \
       -d '{"page_size": 100}'
     ```
   - **Token**: Uses `$NOTION_API_KEY` environment variable (already configured in shell profile)
   - **Matching logic**: For each checklist item, search SOP results by keyword matching on `Name` and `Category` fields
   - **Language preference**: Prefer Korean SOP (`Language: Korean`), fall back to English if Korean not available
   - **Output format**: Attach matching SOP link below each checklist item:
     ```
     - [ ] **항목명** — 설명
       - 📋 [SOP: 가이드 제목](notion-public-url)
     ```
   - **Multiple SOPs**: If multiple SOPs match one item, attach all relevant ones
   - **No match**: If no SOP exists for an item, skip — do not add placeholder
   - **Cache**: Query the database once, then match locally — do not make per-item API calls

8. **Output rules**:
   - Remove non-applicable rows/sections entirely
   - ⏰ 요약 table: only project-relevant items, sorted by longest lead time first
   - Fill project-specific details where available (actual providers, market)
   - Unknown items: mark `[💡 Recommended]`
   - **Language: Korean** — client-facing, clear, actionable, no unnecessary jargon
   - Follow the exact output format defined in `prd-template.md` Section 9

---

## Agent Execution Order

```
1. parser (sonnet)       → parsed-input.md, screen-inventory.md, tbd-items.md
2. prd-writer (opus)     → Section 0-4 draft
   tech-writer (opus)    → Section 5-7 draft
   checklist (sonnet)    → client-checklist.md
3. qa (sonnet)           → Validation results
4. support (sonnet)      → Final PRD (only if QA has FAILs)
```

Step 2 agents run in parallel. tech-writer can fall back to `screen-inventory.md` when Section 3/4 drafts are not yet available. checklist agent only needs `parsed-input.md` from Phase 1.
