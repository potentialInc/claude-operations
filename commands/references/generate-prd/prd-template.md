# PRD 아웃풋 템플릿

이 파일은 `/generate-prdv2`가 생성하는 PRD의 정확한 구조를 정의한다. 에이전트는 이 템플릿을 엄격히 준수해야 한다.

> **claude-fullstack 호환**: Section 0-5, 8은 claude-fullstack pipeline과 1:1 대응.
> Section 6 (Data Model), Section 7 (Permission Matrix)은 개발 실행력 보강을 위해 추가.

---

## Section 0: Project Overview

```markdown
# [Product Name] — Product Requirements Document

**Version:** 1.0
**Date:** [YYYY-MM-DD]
**Status:** Draft

---

## 0. Project Overview

### Product

**Name:** [Product Name]
**Type:** [Mobile App / Web App / SaaS Platform]
**Deadline:** [Date or milestone]
**Status:** Draft

### Description

[2-4 sentences. What does this product do, who is it for, what makes it distinctive?]

### Goals

1. [Primary goal — main problem this product solves]
2. [Secondary goal]
3. [Secondary goal]

### Target Audience

| Audience | Description |
|----------|-------------|
| **Primary** | [Main user group — who they are, what they want] |
| **Secondary** | [Second group, if applicable] |

### User Types

| Type | DB Value | Description | Key Actions |
|------|----------|-------------|-------------|
| **[General User]** | `0` | [Who they are] | [What they mainly do] |
| **[Power User / Artist / etc.]** | `1` | [Who they are] | [What they mainly do] |
| **Admin** | `99` | Platform administrator | Manage users, content, settings |

### User Status

| Status | DB Value | Behavior |
|--------|----------|----------|
| **Active** | `0` | Full access |
| **Suspended** | `1` | Cannot log in — show: "[suspension message]" |
| **Withdrawn** | `2` | Data retained [X] days then deleted |

### User Relationships

[Describe relationships between user types — 1:1, 1:N, hierarchy, etc.]

### Design Reference

| 항목 | 내용 |
|------|------|
| **참고 사이트** | [URL or "없음"] |
| **차별점** | [참고 사이트 대비 차별화 포인트] |
| **색상/로고** | [브랜드 컬러, 로고 기준] |
| **레이아웃 참고** | [참고 사이트의 어떤 화면/흐름을 참고하는지] |
| **특히 중요 화면** | [레퍼런스와 최대한 맞춰야 할 화면 목록] |

> 참고 사이트가 없으면 이 테이블을 "없음"으로 채운다.

### MVP Scope

**Included:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**Excluded (deferred):**
- [Feature deferred to later phase]
- [Feature deferred to later phase]
```

---

## Section 1: Terminology

```markdown
---

## 1. Terminology

### Core Concepts

| Term | Definition |
|------|------------|
| **[Product Name]** | [One-sentence description] |
| **[Core Entity]** | [What it represents in the system] |
| **[Key Feature]** | [What it does] |

### User Roles

| Role | Description |
|------|-------------|
| **Guest** | [What a guest can access] |
| **User** | [What an authenticated user can do] |
| **Admin** | [What an admin manages] |

### Status Values

| Enum | Values | Description |
|------|--------|-------------|
| **[StatusEnum]** | `PENDING`, `ACTIVE`, `COMPLETED` | [When each applies] |
| **[AnotherEnum]** | `DRAFT`, `PUBLISHED`, `ARCHIVED` | [When each applies] |

### Technical Terms

| Term | Definition |
|------|------------|
| **[Term]** | [Plain-language explanation] |
```

---

## Section 2: System Modules

```markdown
---

## 2. System Modules

### Module 1 — [Module Name]

[1-2 sentence description of what this module does.]

#### Main Features

1. [Feature] — [brief description]
2. [Feature] — [brief description]
3. [Feature] — [brief description]

#### Technical Flow

##### [Flow Name]

1. User [triggers action] → `POST /api/[resource]`
2. App [client-side behavior]
3. [Backend/Service] receives [what] and [does what] → `GET /api/[resource]/:id`
4. On success:
   - [Result A]
   - [Result B]
5. On failure:
   - [Error case] → [what user sees / fallback]

##### WebSocket Events (실시간 기능이 있는 모듈만)

| Channel | Event | Payload | Direction |
|---------|-------|---------|-----------|
| `[channel:id]` | `[event_name]` | `{ field1, field2 }` | Server → Client |

---

### Module 2 — [Module Name]

[Description]

#### Main Features

1. [Feature]
2. [Feature]

#### Technical Flow

1. [Step] → `[HTTP_METHOD] /api/[endpoint]`
2. [Step]
3. On success: [result]
4. On failure: [fallback]

---

### Module 3 — [Module Name]

[Description]

#### Main Features

1. [Feature]
2. [Feature]

#### Technical Flow

1. [Step]
2. [Step]

---

### 3rd Party API List

| Service | Purpose | Integration Point | Alternative |
|---------|---------|-------------------|-------------|
| [Name] | [Purpose] | [Where used] | [Fallback option] |
```

---

## Section 3: User Application

```markdown
---

## 3. User Application

### 3.1 Page Architecture

**Stack:** [Framework, Router, State management, CSS]

#### Route Groups

| Group | Access |
|-------|--------|
| Public | Anyone |
| Auth | Unauthenticated only |
| Protected | Logged-in users |

#### Page Map

**Public**
| Route | Page | Access |
|-------|------|--------|
| `/` | Home | Anyone |
| `/[resource]` | [Resource] List | Anyone |
| `/pricing` | Pricing | Anyone |

**Auth**
| Route | Page | Access |
|-------|------|--------|
| `/auth/login` | Login | Unauthenticated |
| `/auth/register` | Register | Unauthenticated |
| `/auth/forgot-password` | Forgot Password | Unauthenticated |

**Protected**
| Route | Page | Access |
|-------|------|--------|
| `/dashboard` | Dashboard | All authenticated |
| `/[resource]` | My [Resources] | [Role] |
| `/[resource]/:id` | [Resource] Editor | [Role, Owner] |
| `/settings` | Settings | All authenticated |
| `/billing` | Billing | All authenticated |

---

### 3.2 Feature List by Route

#### `/` — Home

1) [Component 1]
   - **컴포넌트 상세**: [UI 요소, 상태별: default/active/disabled/error]
   - **데이터 표시**: [표시 항목, 정렬, 페이지네이션]
   - **인터랙션**: [클릭, 호버, 드래그, 키보드 단축키]
   - **네비게이션**: [이동 가능 화면, 뒤로가기]

2) [Component 2]
   - ...

**입력필드 검증규칙** (해당 라우트에 입력이 있을 경우)
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|

**상태 화면**
- Loading: [skeleton/spinner 상세]
- Empty: [0건일 때 메시지 + CTA]
- Error: [에러 시 메시지 + 재시도]

**에러 핸들링**
- 네트워크 에러: [처리]
- 서버 에러: [처리]
- 권한 에러: [처리]

**엣지 케이스**
- 0건: [처리]
- 최대치: [처리]
- 동시접근: [처리]
- 오프라인: [처리]

⚠️ **알려진 위험 (버그 패턴 기반)** ← bug-patterns 존재 시에만
| # | 패턴 | 빈도 | 예방 스펙 |
|---|------|------|----------|

---

#### `/auth/login` — Login

- **Input**:
  - [Field 1] — [검증규칙: 타입, min/max, 패턴, 필수여부]
  - [Field 2] — [검증규칙]
- **상태 화면**: Loading / Error
- **인터랙션**: [탭, 소셜 로그인 버튼 등]
- **네비게이션**: [이동 가능 화면, 뒤로가기]
- **에러 핸들링**: 네트워크 / 잘못된 자격증명 / 계정 잠금
- **엣지 케이스**: [케이스들]

---

#### `/[resource]` — [Resource] List

- Search by keyword
- Filter by: [filter1], [filter2]
- Sort by: [option1], [option2]
- [Resource] card with: [fields shown]
- Actions: [view, quick-action]

[+ 8항목 밀도 적용]

---

#### `/dashboard` — Dashboard

- Stats: [stat1], [stat2], [stat3]
- Recent [items]
- Quick actions

[+ 8항목 밀도 적용]

---

#### `/[resource]/:id` — [Resource] Editor

- [Resource] preview/viewer
- Edit: [how — inline / form / modal]
- Manage [sub-items] (if applicable)
- Actions: Save, Export, Delete

[+ 8항목 밀도 적용]

---

#### `/settings` — Settings

- **Profile:** name, avatar
- **Security:** change password
- **Account:** change email, delete account

[+ 8항목 밀도 적용]

---

#### `/billing` — Billing

- Current plan and renewal date
- Usage vs. quota
- Payment history
- Manage subscription

[+ 8항목 밀도 적용]
```

---

## Section 4: Admin Dashboard

```markdown
---

## 4. Admin Dashboard

> Skip this section if there is no separate admin interface.

### 4.1 Page Architecture

**Access:** Admin role only

#### Page Map

| Route | Page |
|-------|------|
| `/` | Dashboard Overview |
| `/users` | User Management |
| `/users/:id` | User Detail |
| `/[entity]` | [Entity] Management |
| `/[entity]/:id` | [Entity] Detail |
| `/[operations]` | Operations Monitor |
| `/metadata` | Metadata Management |
| `/settings` | Platform Settings |

---

### 4.2 Feature List by Route

#### `/` — Dashboard Overview

**Statistics Cards**
- [Key metrics — total users, today's signups, active sessions, etc.]
- Show increase/decrease percentage compared to previous period

**Period Filter**
- Today / Last 7 days / Last 30 days / Custom date range

**Charts**
- [Trend visualization — line/bar charts based on app metrics]

**Recent Activity**
- [Recently created/modified items list]

---

#### `/users` — User Management

**Main Page**
1. Top Area:
   - Search: [Keyword search — name, email, ID]
   - Filters: [Status (Active/Inactive), Date range, Role]
   - Create button → Creation Modal
   - Bulk Action dropdown: [Delete / Activate / Deactivate / Export]

2. Table Component:
   | Column | Type | Sortable | Description |
   |--------|------|----------|-------------|
   | Checkbox | checkbox | ❌ | Row selection + Select All |
   | [Column 1] | text | ✅ | [Description] |
   | [Column 2] | date | ✅ | [Description] |
   | Actions | button | ❌ | View / Edit / Delete |

3. Standard Features Applied:
   → `references/admin-standards.md` 전체 적용

**Creation Modal**
| Field | Type | Required | Validation | Error Message |
|-------|------|----------|------------|---------------|
| [Field 1] | text | ✅ | [Rule] | [Message] |

**Detail Drawer**
- Header Info: [User profile information]
- Account Actions: Activate / Deactivate / Reset password / Change role
- Activity Log: [Recent user activities]
- Timestamps: Created at / Last login / Last modified

⚠️ **알려진 위험 (버그 패턴 기반)** ← bug-patterns 존재 시에만
| # | 패턴 | 빈도 | 예방 스펙 |
|---|------|------|----------|

---

#### `/[entity]` — [Entity] Management

[Same structure as User Management with entity-specific columns]

---

#### `/[operations]` — Operations Monitor

- All [jobs/sessions] with status filter
- Status summary cards
- Per-item actions: view details, cancel, retry

---

#### `/metadata` — Metadata Management

- CRUD for dynamic lookup values
- Organized by type (categories, tags, etc.)
- Usage count per item
- Reorder capability

---

#### `/settings` — Platform Settings

- General config (name, logo, limits)
- API key management (masked view, update)
- Email / SMTP configuration
- Storage configuration

---

### Export / Data Download

**Data Download**
- [Downloadable data list]
- Format: CSV / Excel
- Filter Options: All time / Custom date range / Current filtered results
```

---

## Section 5: Tech Stack & Architecture

```markdown
---

## 5. Tech Stack & Architecture

### Architecture

[1-2 sentence overview]

```
[project]/
├── [backend]/    ← [Backend framework] API
├── [frontend]/   ← [Frontend framework] user app
└── [admin]/      ← Admin dashboard (if separate)
```

### Technologies

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| Backend | [Framework] | [x.x] | API server |
| Language | [Language] | [x.x] | — |
| ORM | [ORM] | [x.x] | Database access |
| Database | [DB] | [x.x] | Primary data store |
| Frontend | [Framework] | [x.x] | UI |
| Routing | [Router] | [x.x] | Client routing |
| State | [State mgmt] | — | Global state |
| CSS | [CSS lib] | [x.x] | Styling |
| Build | [Build tool] | — | Bundler |

### Third-Party Integrations

| Service | Purpose |
|---------|---------|
| [Auth provider] | [Authentication] |
| [AI service] | [AI features] |
| [Payment] | [Subscriptions/payments] |
| [Storage] | [File storage] |
| [Email] | [Transactional email] |

### Key Decisions

| Decision | Rationale |
|----------|-----------|
| [Choice] | [Why] |
| [Choice] | [Why] |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `[DB_URL]` | Database connection |
| `[API_KEY]` | External service key |
| `[FRONTEND_URL]` | Frontend base URL |
```

---

## Section 6: Data Model

```markdown
---

## 6. Data Model

### Entity List

| Entity | Key Fields | Description |
|--------|-----------|-------------|
| **User** | id, email, name, role, status, created_at | Platform user account |
| **[Entity]** | id, [field1], [field2], user_id(FK), status | [Description] |
| **[Entity]** | id, [field1], [field2], [entity]_id(FK) | [Description] |

### Entity Relationships

```
User 1:N [Entity]          ← A user owns many [entities]
[Entity] 1:N [SubEntity]   ← Each [entity] has many [sub-entities]
User N:N [Entity] via [JoinTable]  ← Many-to-many with join table
[Entity] self-ref (parent_id)      ← Self-referencing hierarchy
```

### Status Enums

| Enum Name | Values | Used By |
|-----------|--------|---------|
| `UserStatus` | `ACTIVE(0)`, `SUSPENDED(1)`, `WITHDRAWN(2)` | User |
| `[EntityStatus]` | `DRAFT(0)`, `PUBLISHED(1)`, `ARCHIVED(2)` | [Entity] |

### Index Hints

| Entity | Column(s) | Type | Reason |
|--------|-----------|------|--------|
| User | email | UNIQUE | Login lookup |
| [Entity] | user_id, created_at | COMPOSITE | List query by owner |
| [Entity] | status | BTREE | Filter by status |

### Soft Delete

| Entity | Soft Delete | Retention |
|--------|------------|-----------|
| User | ✅ | 30 days |
| [Entity] | ✅ | 90 days |
| [Log Entity] | ❌ | — |
```

---

## Section 7: Permission Matrix

```markdown
---

## 7. Permission Matrix

### Action × Role Matrix

| Resource | Action | Guest | User | [Power User] | Admin |
|----------|--------|-------|------|-------------|-------|
| [Entity] | Create | ❌ | ✅ | ✅ | ✅ |
| [Entity] | Read (own) | ❌ | ✅own | ✅own | ✅ |
| [Entity] | Read (all) | ❌ | ❌ | ❌ | ✅ |
| [Entity] | Update | ❌ | ✅own | ✅own | ✅ |
| [Entity] | Delete | ❌ | ✅own | ✅own | ✅ |
| [Sub-Entity] | Create | ❌ | ✅ | ✅ | ✅ |
| [Sub-Entity] | Read | ❌ | ✅own | ✅own | ✅ |
| [Sub-Entity] | Update | ❌ | ✅own | ✅own | ✅ |
| [Sub-Entity] | Delete | ❌ | ✅own | ✅own | ✅ |
| User Profile | Read | ❌ | ✅own | ✅own | ✅ |
| User Profile | Update | ❌ | ✅own | ✅own | ✅ |
| Admin Dashboard | Access | ❌ | ❌ | ❌ | ✅ |

> **Legend**: ✅ = allowed, ❌ = denied, ✅own = allowed for own resources only

### Ownership Rules

- **Own resource**: User can only CRUD resources where `resource.user_id === currentUser.id`
- **Admin override**: Admin can CRUD all resources regardless of ownership
- [Additional ownership rules specific to the project]

### Role Hierarchy

```
Guest < User < [Power User] < Admin
```

- Higher roles inherit all permissions of lower roles
- [Special rules, e.g., "Power User can do X that regular User cannot"]
```

---

## Section 8: Open Questions

```markdown
---

## 8. Open Questions

| # | Question | Context / Impact | Owner | Status |
|:-:|----------|-----------------|-------|--------|
| 1 | [Question?] | [Why it matters — what gets blocked if unresolved] | [Owner] | ⏳ Open |
| 2 | [Question?] | [Context] | [Owner] | ⏳ Open |
```

---

## Additional Sections

```markdown
---

# Additional Questions (Client Confirmation Required)

## Required Clarifications
| # | Question | Context |
|:-:|:---------|:--------|
| 1 | [Question 1] | [Why this is needed] |

## Recommended Clarifications
| # | Question | Context |
|:-:|:---------|:--------|
| 1 | [Question 1] | [Why this is needed] |

---

# Feature Change Log

## Version 1.0 ([TODAY'S DATE: YYYY-MM-DD])

| Change Type | Risk | Before | After | Source |
|:-----------|:-----|:-------|:------|:-------|
| **Initial PRD** | - | - | PRD generated from client input | [Source document] |

### Change Details
#### Initial Generation
- **Source Document**: [Client input filename]
- **Change Description**: Initial PRD generated from client questionnaire answers
- **권장적용 항목**: [N]개 승인 `[📌 권장적용됨]`, [M]개 질문으로 이동

---

# UX Flow Improvement Suggestions (Optional)

UX flow issues identified during PRD creation:

## Suggestions
| # | Current Flow | Issue | Suggestion | Priority | Complexity |
|:-:|:------------|:------|:-----------|:--------:|:----------:|
| 1 | [Current] | [Issue] | [Suggestion] | High/Med/Low | Simple/Med/Complex |

### Suggestion Details
#### Suggestion 1: [Title]
- **Current**: [Current state]
- **Issue**: [Problem]
- **Suggestion**: [Improvement]
- **Expected Benefit**: [Benefit]
- **Affected Users**: [Which user types]
```
