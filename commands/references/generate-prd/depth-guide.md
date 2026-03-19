# Mandatory Density Requirements (Depth Guide)

> Goal: Not "write more" but "if these items are missing, QA FAIL".
> Every route/page must have the items below **present**.

---

## Section 0 — Project Overview Density

### User Types Table
- Type, DB Value, Description, Key Actions — 4 columns required
- DB Value is integer (0, 1, 2, ..., 99) — for ORM enum mapping
- All user types included (including Admin)

### User Status Table
- Status, DB Value, Behavior — 3 columns required
- Suspended/Withdrawn: specify exact behavior (displayed message, data retention period)

### MVP Scope
- Both Included and Excluded must be specified
- Excluded items must state "deferred to [phase/version]"

---

## Section 1 — Terminology Density

### Core Concepts
- Include ALL domain terms used in the project
- Exclude generic IT terms (API, DB, etc.)
- 1:1 correspondence with terms used in Section 3/4 body text

### Status Values
- All enums stored in DB
- Each enum: list of values + when each value applies
- Must match Section 6 Status Enums

---

## Section 2 — System Modules Density

### Technical Flow
- 5-10 steps per module (minimum 5)
- Each step: `[Actor] does [Action]` format
- Both **success branch** and **failure branch** included
- Failure: specify what user sees
- **API Hint required**: expected endpoint per step (`→ POST /api/...`)

### Real Scenarios (NEW — MANDATORY)
Each module MUST have:
- **Real Scenario — Success**: Concrete scenario with:
  - Named user with role (e.g., "Kim Designer (owner)")
  - Specific action sequence
  - Server-side processing description
  - UI result (toast, redirect, state change)
- **Real Scenario — Failure**: Concrete failure scenario with:
  - Specific failure trigger (e.g., "50MB file upload attempt")
  - Error code and message
  - User-visible result
- Minimum: 1 success + 1 failure per module
- Complex modules (auth, payment): 2+ scenarios each

### WebSocket Events
- Required for real-time feature modules
- Channel, Event, Payload, Direction — 4 columns
- Payload lists key fields (`{ field1, field2 }`)

### 3rd Party API List
Each API requires 4 fields:

| Field | Description |
|-------|-------------|
| Service | Service name |
| Purpose | Usage purpose |
| Integration Point | Where used |
| Alternative | Fallback service ("N/A" if none) |

---

## Section 3 — Route-Level Mandatory 8 Items

Every route/page MUST have all 8 items below. Missing even one = QA FAIL.

### 1. Component Detail
- List all UI elements on the screen
- Define each component's states: **default / active / disabled / error**
- Example: button default state, pressed (active), disabled, error state

### 2. Input Validation Rules
If the route has inputs, define as table:

| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|

Routes without inputs: state "N/A".

### 3. State Screens
Define all 3 states:
- **Loading**: skeleton UI / spinner / loading text
- **Empty**: message when 0 items + CTA (Call to Action)
- **Error**: error message + retry method

### 4. Interactions
All possible user interactions:
- Click
- Hover
- Drag & drop
- Keyboard shortcuts
- Pull to refresh (mobile)
- If none apply: state "Click interactions only"

### 5. Navigation
- All reachable screens from this route
- Back button behavior (previous screen / tab home / app home)
- Deep link entry: back button behavior

### 6. Data Display
- Display fields (which data is shown)
- Sort criteria (default sort, changeable?)
- Pagination method (infinite scroll / page numbers / Load More)

### 7. Error Handling
3 error types:
- **Network error**: offline / timeout
- **Server error**: 500 / 503
- **Permission error**: login required / access denied

### 8. Edge Cases
Minimum 4:
- **Zero items**: behavior when no data
- **Maximum capacity**: behavior with large data (performance, display limits)
- **Concurrent access**: multiple users manipulating same data
- **Offline**: behavior without network

### Additional Density Requirements
- Each route: specify **accessible roles** (must match Page Map Access column)
- Edit/delete features: specify **ownership rules** (own only / all)

---

## Section 4 — Admin Page Density

### Per Admin Page Required Items

1. **Table Column Definition** (as table):

   | Column | Type | Sortable | Description |
   |--------|------|----------|-------------|

2. **Filter Options**:
   - Dropdown value lists
   - Date range picker
   - Search target fields

3. **Detail Drawer/Modal**:
   - Complete display field list
   - Available actions (status change, delete, edit, etc.)
   - Related data display (activity log, related entities)

4. **Creation Modal**:
   - Validation rules per input field (table)
   - Required/optional distinction
   - Error messages

5. **Standard Features Applied**:
   - Confirm all required items from `references/admin-standards.md` are included
   - If any excluded, state reason

6. **Edge Cases** (minimum 3):
   - Bulk operation on 100+ items
   - Concurrent edit by multiple admins
   - Data with foreign key dependencies (delete cascade behavior)

---

## Section 5 — Tech Stack & System Design Density

### Technologies Table
- Layer, Technology, Version, Purpose — 4 columns required
- All layers: Backend, Language, ORM, Database, Frontend, Routing, State, CSS, Build

### Third-Party Integrations
- All services from Section 2's 3rd Party API List must also appear here
- Service, Purpose — minimum 2 columns

### Key Decisions
- Major technology choices with rationale
- Minimum 2

### Environment Variables
- All env vars needed for external service integration
- Variable, Description — 2 columns

### Conditional Sections Density (NEW)
Only include when detected by parser. When included, minimum requirements:

**Auth Flow** (when auth features present):
- Authentication method (email/PW, OAuth providers)
- Token strategy (JWT access + refresh, session, etc.)
- Token storage location (httpOnly cookie, localStorage, etc.)
- Password requirements
- Social login callback flow

**File Pipeline** (when file upload present):
- Max file size
- Allowed MIME types
- Processing steps (resize dimensions, thumbnail generation)
- Storage destination (S3 bucket, CDN URL pattern)
- Cleanup policy for orphaned files

**Real-time Architecture** (when WebSocket/real-time present):
- Transport (WebSocket, SSE, polling fallback)
- Channel naming convention
- Authentication for WS connections
- Reconnection strategy (backoff, max retries)
- Key events summary table

**Billing Flow** (when billing/subscription present):
- Payment provider (Stripe, PG, etc.)
- Plan structure (free/pro/enterprise)
- Webhook events to handle
- Refund/cancellation policy
- Grace period for failed payments

**Multi-tenancy** (when SaaS detected):
- Isolation level (row-level, schema-level, DB-level)
- Tenant identification (subdomain, header, path)
- Cross-tenant data access rules
- Tenant-specific customization scope

---

## Section 6 — Data Model Density (ENHANCED)

### Entity Relationships
- All relationship types: 1:1, 1:N, N:N, self-reference
- N:N relationships: **join table name required**
- No ambiguous relationships — specific cardinality required

### Full Schema (NEW — MANDATORY)
Every entity MUST have a column-level table:

| Column | Type | Constraints |
|--------|------|-------------|

Requirements per entity:
- **Minimum 5 columns** (excluding timestamps)
- PK column with generation strategy (cuid, uuid, autoincrement)
- All FK columns with `(FK → entity.column)` notation
- NOT NULL / UNIQUE / DEFAULT constraints specified
- Enum columns reference Status Enum name
- created_at, updated_at timestamps included

### Status Enums
- Must match Section 1 Status Values
- Include DB stored value (integer)
- Specify which entity uses each enum

### Index Hints
- Index hints for search/filter target columns
- Entity, Column(s), Type, Reason — 4 columns
- Distinguish: UNIQUE, COMPOSITE, BTREE

### Soft Delete
- List of entities with soft delete
- Data retention period per entity

---

## Section 7 — Permission Matrix Density

### Action × Role Matrix
- **All CRUD resources** × **All roles** — no empty cells
- Edit/delete: specify ownership condition: `✅own` = own resources only
- No missing cells: `?` not allowed — must be `✅` / `❌` / `✅own`

### Ownership Rules
- "Own resource" definition: `resource.user_id === currentUser.id`
- Admin override rules
- Project-specific ownership (team-based, org-based, etc.)

### Role Hierarchy
- Lower role → higher role order
- Inheritance rules specified

---

## QA Usage

QA agent validates against this guide:
- Section 2 modules: missing Real Scenario → FAIL (rule 13)
- Section 3 routes: missing any of 8 items → FAIL (rule 2)
- Section 4 admin pages: missing required items → FAIL (rule 4)
- Section 6 entities: CRUD entity from Section 3/4 missing → FAIL (rule 10)
- Section 6 schema: entity in Entity List but no Full Schema table → FAIL (rule 14)
- Section 7 permissions: action from Section 3/4 without Permission Matrix entry → FAIL (rule 11)
- Section 9 checklist: required subsection missing for detected App Type/features → FAIL (rule 15)

---

## Section 9 — Client Preparation Checklist Density (Separate File)

> Section 9 is generated as `client-checklist.md` by the checklist agent, NOT included in the PRD.

### Structure
- **Single flat list** — NO phase/stage separation (1단계/2단계/3단계 구분 없음)
- Grouped by type: 계정, 브랜드 소재, 스토어 제출물, 법률 문서, 콘텐츠
- All items collected upfront — everything the client needs to prepare from the start

### Always Required Sections
- 계정: 클라우드(AWS), 도메인 + 플랫폼별 개발자 계정
- 브랜드 소재: 로고, 색상, 폰트
- 법률 문서: 개인정보처리방침, 이용약관
- 콘텐츠: 담당자

### Conditional Section Rules
- Only include items matching detected App Type and features
- Web-only: MUST NOT include iOS/Android 스토어 제출물
- App-only: MUST NOT include 웹 소재 (파비콘, OG 이미지)
- Each conditional item: remove providers not applicable to this project

### Account Items — Dev Team Handles Setup
- For accounts (AWS, Apple, Google, social login providers, PG, etc.): client only needs to **create the account and share access**
- Do NOT include setup instructions, configuration steps, or technical details
- Dev team handles all setup after receiving account access
- Only include **decision points** the client must make (e.g., individual vs organization account)

### Key Platform Rules
- Google Play 개인 계정: 공개 출시 전 **20명 이상의 테스터와 최소 14일 이상** 비공개 테스트를 반드시 거쳐야 함
- Apple 조직 계정: D-U-N-S 번호 필요
- iOS 소셜 로그인: 다른 소셜 로그인이 있으면 Apple 로그인 **의무**

### SOP Link Attachment
- Query Notion SOP database at runtime
- Attach matching SOP link to each item: `📋 [SOP: title](url)`
