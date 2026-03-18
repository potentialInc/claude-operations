# PRD Output Template (v3)

This file defines the exact structure for `/generate-prd` output. Agents must strictly follow this template when generating PRDs.

> **Compatibility**: Sections 0-5 and 8 correspond to the claude-fullstack pipeline.
> Section 6 (Data Model — Full Schema) and Section 7 (Permission Matrix) provide development execution detail.

> **Conditional Sections**: Section 5 contains conditional subsections (Auth Flow, File Pipeline, Real-time Architecture, Billing Flow, Multi-tenancy). Include each subsection ONLY when the corresponding feature is detected in the project. Omit subsections that do not apply.

> **Edge Cases & Real Scenarios**: v3 adds mandatory Real Scenario blocks (Section 2) and Edge Case blocks (Sections 3 and 4). These are NOT optional — every module, route, and admin page must include them.

---

## Section 0: Project Overview

### Product

**[Product Name]** — [One-line product description]

### Description

[2-4 sentence product description covering what the product does, who it serves, and its core value proposition.]

### Goals

| # | Goal | Success Metric |
|---|------|---------------|
| 1 | [Primary goal] | [Measurable metric] |
| 2 | [Secondary goal] | [Measurable metric] |
| 3 | [Tertiary goal] | [Measurable metric] |

### Target Audience

[1-2 sentences describing the primary target audience, demographics, and context of use.]

### User Types

| Type | DB Value | Description | Key Actions |
|------|----------|-------------|-------------|
| [User Type 1] | `[db_enum_or_string]` | [Who this user is] | [Comma-separated list of primary actions] |
| [User Type 2] | `[db_enum_or_string]` | [Who this user is] | [Comma-separated list of primary actions] |
| [Admin] | `[db_enum_or_string]` | [Platform administrator] | [Comma-separated list of admin actions] |

### User Status

| Status | DB Value (int) | Description | Allowed Actions |
|--------|---------------|-------------|-----------------|
| [Active] | `1` | [Normal operational state] | [All standard actions] |
| [Suspended] | `2` | [Temporarily restricted] | [Read-only, contact support] |
| [Banned] | `3` | [Permanently restricted] | [None] |
| [Pending] | `0` | [Awaiting verification] | [Complete profile, verify email] |

### User Relationships

[Describe how user types relate to each other. Example: "A Mentor can have many Students. A Student belongs to exactly one Organization. An Admin can manage all users."]

### Design Reference

| Screen / Flow | Reference | Notes |
|--------------|-----------|-------|
| [Login page] | [Figma link / screenshot path] | [Any design notes] |
| [Dashboard] | [Figma link / screenshot path] | [Any design notes] |
| [Key feature page] | [Figma link / screenshot path] | [Any design notes] |

### MVP Scope

**In Scope:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**Out of Scope (post-MVP):**
- [Deferred feature 1]
- [Deferred feature 2]

---

## Section 1: Terminology

### Core Concepts

| Term | Definition |
|------|-----------|
| [Term 1] | [Clear definition as used in this product] |
| [Term 2] | [Clear definition as used in this product] |

### User Roles

| Role | Internal Name | Description |
|------|--------------|-------------|
| [Role 1] | `[role_db_value]` | [What this role can do] |
| [Role 2] | `[role_db_value]` | [What this role can do] |

### Status Values

| Status Name | DB Integer Value | Context | Description |
|-------------|-----------------|---------|-------------|
| [Active] | `1` | [User / Order / etc.] | [What this status means] |
| [Pending] | `0` | [User / Order / etc.] | [What this status means] |
| [Completed] | `2` | [Order / Task / etc.] | [What this status means] |

### Technical Terms

| Term | Definition |
|------|-----------|
| [Technical term 1] | [Definition relevant to this project] |
| [Technical term 2] | [Definition relevant to this project] |

---

## Section 2: System Modules

> Each module MUST include Real Scenario blocks (Success + Failure). These provide concrete, testable examples that clarify expected behavior.

### Module: [Module Name]

**Description:** [What this module handles and why it exists]

**Main Features:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**Technical Flow:**

```
[Step-by-step flow description]

1. User triggers [action]
2. Client sends request → POST /api/[resource]
3. Server validates [what]
4. Server processes [what]
5. Server responds with [what]
6. Client updates [what]
```

- **Success path:** [User action] -> [API call] -> [Server processing] -> [Response 200] -> [UI update]
- **Failure path:** [User action] -> [API call] -> [Validation fails / Server error] -> [Error response 4xx/5xx] -> [UI error state]

**Real Scenario — Success:**

> **Actor:** [Specific user name, e.g., "Sarah (Mentor)"]
> **Context:** [Specific situation, e.g., "Sarah has 3 active students and wants to schedule a new session"]
> **Action:** [Exact steps, e.g., "Sarah clicks 'New Session', selects student 'John Park', picks March 20 at 2:00 PM, adds topic 'React Hooks Review'"]
> **Server Response:** `201 Created` — returns `{ sessionId: "sess_abc123", status: "scheduled", notificationSent: true }`
> **UI Reaction:** [What the user sees, e.g., "Calendar updates with new session block. Toast notification: 'Session scheduled. John has been notified.' Session appears in upcoming list."]

**Real Scenario — Failure:**

> **Actor:** [Specific user name, e.g., "Sarah (Mentor)"]
> **Context:** [Specific failure situation, e.g., "Sarah tries to schedule a session at a time slot already booked by another mentor"]
> **Action:** [Exact steps, e.g., "Sarah clicks 'New Session', selects student 'John Park', picks March 20 at 2:00 PM (already booked)"]
> **Server Response:** `409 Conflict` — returns `{ error: "TIME_SLOT_CONFLICT", conflictWith: "sess_xyz789", message: "This time slot is already booked" }`
> **UI Reaction:** [What the user sees, e.g., "Time slot highlights in red. Error banner: 'This time is unavailable. Please choose another slot.' Available alternatives shown below."]

**WebSocket Events** *(include only for real-time modules)*

| Channel | Event | Payload | Direction |
|---------|-------|---------|-----------|
| `[channel_name]` | `[event_name]` | `{ [field]: [type] }` | Server -> Client / Client -> Server |

**3rd Party API List**

| Service | Purpose | Integration Point | Alternative |
|---------|---------|-------------------|-------------|
| [Service name] | [Why it is used] | [Which module/flow uses it] | [Fallback service or "None"] |

---

*(Repeat the full module block for each system module)*

---

## Section 3: User Application

### 3.1 Page Architecture

**Stack:**

| Layer | Technology | Notes |
|-------|-----------|-------|
| Framework | [Next.js / Nuxt / SvelteKit / etc.] | [Version, rendering strategy] |
| State Management | [Zustand / Pinia / Redux / etc.] | [Why this choice] |
| Styling | [Tailwind / CSS Modules / etc.] | [Design system reference] |
| HTTP Client | [Axios / fetch / tRPC / etc.] | [Interceptor strategy] |

**Route Groups:**

| Group | Prefix | Auth Required | Layout |
|-------|--------|--------------|--------|
| [Public] | `/` | No | `PublicLayout` |
| [Auth] | `/auth` | No | `AuthLayout` |
| [Dashboard] | `/dashboard` | Yes | `DashboardLayout` |
| [Admin] | `/admin` | Yes (Admin) | `AdminLayout` |

**Page Map:**

| Route | Page Component | Data Source | Guard |
|-------|---------------|-------------|-------|
| `/` | `HomePage` | [Static / API] | None |
| `/login` | `LoginPage` | [None] | Guest only |
| `/dashboard` | `DashboardPage` | `GET /api/[resource]` | Auth |
| `/dashboard/[id]` | `DetailPage` | `GET /api/[resource]/:id` | Auth + Owner |

### 3.2 Feature List by Route

#### Route: `[/route-path]`

**Access:** [Role(s) allowed] | **Ownership:** [Owner can edit/delete own items only / Admin can edit all / etc.]

**1. Component Detail**

| Component | Default State | Active State | Disabled State | Error State |
|-----------|--------------|-------------|----------------|-------------|
| [Component name] | [Default appearance] | [Active/focused appearance] | [When/why disabled, appearance] | [Error appearance] |

**2. Input Validation Rules**

| Field | Type | Required | Min | Max | Pattern | Error Message |
|-------|------|----------|-----|-----|---------|---------------|
| [field_name] | `string` | Yes | 2 | 100 | `[a-zA-Z ]+` | "[Specific error message shown to user]" |
| [field_name] | `email` | Yes | - | 255 | RFC 5322 | "Please enter a valid email address" |
| [field_name] | `number` | No | 0 | 99999 | `^\d+$` | "Must be a positive number" |

**3. State Screens**

- **Loading:** [Skeleton / Spinner / Shimmer — describe what the user sees]
- **Empty:** [Empty state illustration + message + CTA button, e.g., "No sessions yet. Schedule your first session."]
- **Error:** [Error message + retry button, e.g., "Failed to load data. [Retry]"]

**4. Interactions**

| Trigger | Action | Feedback |
|---------|--------|----------|
| Click [element] | [What happens] | [Visual feedback — toast, animation, redirect] |
| Hover [element] | [What happens] | [Tooltip / highlight / cursor change] |
| Drag [element] | [What happens] | [Drag ghost / drop zone highlight] |
| Keyboard [key] | [What happens] | [Focus ring / shortcut action] |
| Pull-to-refresh | [What happens] | [Spinner / data reload animation] |

**5. Navigation**

- **Reachable screens:** [List of pages this route links to]
- **Back behavior:** [Browser back / custom back / breadcrumb]
- **Deep link behavior:** [What happens when user opens URL directly — auth redirect? data load?]

**6. Data Display**

| Field | Source | Format | Fallback |
|-------|--------|--------|----------|
| [Display field] | `response.[field]` | [Date format / currency / truncation] | [Default value if null] |

- **Sort criteria:** [Default sort field and direction, available sort options]
- **Pagination method:** [Infinite scroll / page numbers / load more button — page size]

**7. Error Handling**

| Error Type | Status Code | User Message | Recovery Action |
|------------|-------------|-------------|-----------------|
| Network error | — | "Connection lost. Check your internet." | Auto-retry after 3s, then manual retry |
| Server error | 500 | "Something went wrong. Please try again." | Manual retry button |
| Not found | 404 | "This [resource] no longer exists." | Redirect to list page |
| Permission denied | 403 | "You don't have access to this [resource]." | Redirect to dashboard |
| Validation error | 422 | [Field-specific messages from server] | Highlight invalid fields |

**8. Edge Cases**

> Minimum 4 edge cases per route. Cover the following categories at minimum:

| # | Category | Scenario | Expected Behavior |
|---|----------|----------|-------------------|
| 1 | Zero data | User has no [resources] yet | Show empty state with CTA to create first [resource] |
| 2 | Max capacity | User has [max limit] [resources] | Show limit reached message, disable create button |
| 3 | Concurrent access | Two users edit the same [resource] simultaneously | Last-write-wins with conflict notification / Optimistic lock with retry prompt |
| 4 | Offline | User loses connection while on this page | Show offline banner, queue actions for retry, disable submit buttons |
| 5 | [Additional] | [Project-specific edge case] | [Expected behavior] |

**Known Risk** *(include only if bug-patterns exist for this route)*

| # | Pattern | Frequency | Prevention Spec |
|---|---------|-----------|-----------------|
| 1 | [Known bug pattern, e.g., "Race condition on rapid double-click submit"] | [How often observed] | [Specific prevention: "Disable button on first click, debounce 500ms"] |

---

*(Repeat the full route block for each route in the application)*

---

## Section 4: Admin Dashboard

> Each admin page MUST include Edge Cases and may include Known Risk tables.

### Admin Page: [Resource Management]

**Access:** Admin only | **URL:** `/admin/[resources]`

**1. Table Column Definition**

| Column | Type | Sortable | Description |
|--------|------|----------|-------------|
| [ID] | `string` | Yes | [Unique identifier, truncated display] |
| [Name] | `string` | Yes | [Full name with avatar thumbnail] |
| [Status] | `badge` | Yes | [Color-coded status badge] |
| [Created At] | `datetime` | Yes | [Relative time format, hover for absolute] |
| [Actions] | `actions` | No | [Edit / Disable / Delete dropdown] |

**2. Filter Options**

| Filter | Type | Options | Default |
|--------|------|---------|---------|
| [Status] | Dropdown | [All, Active, Suspended, Banned] | All |
| [Date Range] | Date picker | [Custom range] | Last 30 days |
| [Search] | Text input | [Searches name, email, ID] | Empty |
| [Role] | Dropdown | [All roles] | All |

**3. Detail Drawer/Modal**

**Display Fields:**

| Field | Source | Format |
|-------|--------|--------|
| [Profile image] | `[entity].avatar` | [Image thumbnail 80x80] |
| [Full name] | `[entity].name` | [Plain text] |
| [Email] | `[entity].email` | [Clickable mailto link] |
| [Status] | `[entity].status` | [Color-coded badge] |
| [Created] | `[entity].createdAt` | [Full datetime format] |

**Possible Actions:**
- [Edit] — opens edit modal with pre-filled fields
- [Change Status] — dropdown to change user status with confirmation
- [Delete] — requires typing resource name to confirm
- [View Activity] — opens activity log filtered to this resource

**Related Data:**
- [Related entity list, e.g., "Sessions (paginated, 10 per page)"]
- [Related statistics, e.g., "Total revenue from this user"]

**4. Creation Modal**

| Field | Type | Required | Validation | Error Message |
|-------|------|----------|-----------|---------------|
| [Name] | `text` | Yes | Min 2, Max 100 | "Name must be 2-100 characters" |
| [Email] | `email` | Yes | Valid email format | "Please enter a valid email" |
| [Role] | `select` | Yes | Must be valid role | "Please select a role" |
| [Password] | `password` | Yes | Min 8, 1 uppercase, 1 number | "Password must meet requirements" |

**5. Standard Features Applied**

Reference `admin-standards.md` for these standard features applied to this page:
- Bulk actions (select multiple, bulk status change, bulk delete)
- Export (CSV, Excel)
- Audit log per record
- Pagination (default 20 per page, configurable)

**Edge Cases**

| # | Category | Scenario | Expected Behavior |
|---|----------|----------|-------------------|
| 1 | Zero data | No [resources] exist yet | Show empty table with "No [resources] found" message and create button |
| 2 | Bulk operation | Admin selects all 500+ items and clicks bulk delete | Show confirmation with count, process in background with progress indicator |
| 3 | Concurrent edit | Two admins edit the same record simultaneously | Show "Record was modified by another admin. Refresh to see changes." |
| 4 | Filter combination | Multiple filters applied result in zero records | Show "No results match your filters" with clear-all-filters button |
| 5 | [Additional] | [Project-specific admin edge case] | [Expected behavior] |

**Known Risk** *(include only if bug-patterns exist for this page)*

| # | Pattern | Frequency | Prevention Spec |
|---|---------|-----------|-----------------|
| 1 | [Known bug pattern] | [How often observed] | [Prevention specification] |

---

*(Repeat the full admin page block for each admin management page)*

---

## Section 5: Tech Stack & System Design

### Technologies

| Category | Technology | Version | Rationale |
|----------|-----------|---------|-----------|
| Frontend Framework | [Next.js / Nuxt / etc.] | [Version] | [Why chosen] |
| Backend Framework | [NestJS / Express / FastAPI / etc.] | [Version] | [Why chosen] |
| Database | [PostgreSQL / MySQL / MongoDB / etc.] | [Version] | [Why chosen] |
| ORM | [Prisma / TypeORM / Sequelize / etc.] | [Version] | [Why chosen] |
| Cache | [Redis / Memcached / None] | [Version] | [Why chosen] |
| Search | [Elasticsearch / Meilisearch / None] | [Version] | [Why chosen] |
| File Storage | [S3 / GCS / Local / None] | - | [Why chosen] |
| Deployment | [Vercel / AWS / Docker / etc.] | - | [Why chosen] |

### Third-Party Integrations

| Service | Purpose | SDK/API | Env Variables Required |
|---------|---------|---------|----------------------|
| [Stripe] | [Payment processing] | [stripe npm package] | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` |
| [SendGrid] | [Transactional email] | [REST API] | `SENDGRID_API_KEY` |
| [AWS S3] | [File storage] | [aws-sdk] | `AWS_ACCESS_KEY`, `AWS_SECRET_KEY`, `S3_BUCKET` |

### Key Architectural Decisions

| # | Decision | Rationale | Trade-offs |
|---|----------|-----------|------------|
| 1 | [Decision, e.g., "Server-side rendering for public pages"] | [Why, e.g., "SEO requirements"] | [Trade-off, e.g., "Higher server load, longer TTFB"] |
| 2 | [Decision] | [Rationale] | [Trade-offs] |

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | — | [Database connection string] |
| `JWT_SECRET` | Yes | — | [Secret for signing JWT tokens] |
| `NEXT_PUBLIC_API_URL` | Yes | `http://localhost:3000/api` | [Public API base URL] |
| `[VARIABLE_NAME]` | [Yes/No] | [Default value or —] | [Description] |

---

### Conditional Section: Auth Flow

> **Include this section ONLY when authentication features are detected in the project.**

**Authentication Method:** [JWT / Session / OAuth2 / etc.]

**Token Strategy:**
- Access token: [Type, expiry, storage location (httpOnly cookie / localStorage)]
- Refresh token: [Type, expiry, rotation policy]
- Token refresh flow: [Automatic silent refresh / redirect to login on expiry]

**Login Flow:**
```
1. User submits credentials → POST /api/auth/login
2. Server validates credentials against [database/provider]
3. Server issues access token + refresh token
4. Client stores tokens in [storage location]
5. Client redirects to [dashboard/home]
```

**Social Login Flow** *(if applicable)*
```
1. User clicks [Provider] login button
2. Redirect to [Provider] OAuth consent screen
3. [Provider] redirects back with authorization code
4. Server exchanges code for tokens → POST [provider_token_url]
5. Server creates/links user account
6. Server issues application tokens
7. Client redirects to [dashboard/home]
```

**Password Reset Flow:**
```
1. User submits email → POST /api/auth/forgot-password
2. Server generates reset token (expires in [duration])
3. Server sends email with reset link
4. User clicks link → GET /reset-password?token=[token]
5. User submits new password → POST /api/auth/reset-password
6. Server validates token, updates password, invalidates all sessions
7. User redirected to login
```

---

### Conditional Section: File Pipeline

> **Include this section ONLY when file upload features are detected in the project.**

**Upload Flow:**
- Max file size: [e.g., 10MB]
- Allowed types: [e.g., image/jpeg, image/png, application/pdf]
- Upload method: [Direct to storage / Server proxy / Presigned URL]
- Progress feedback: [Progress bar / percentage / indeterminate]

**Processing Pipeline:**

| Step | Process | Input | Output | Async |
|------|---------|-------|--------|-------|
| 1 | [Virus scan] | [Raw upload] | [Clean file] | [Yes/No] |
| 2 | [Image resize] | [Original image] | [Multiple sizes: thumb 150x150, medium 600x600, large 1200x1200] | [Yes/No] |
| 3 | [Metadata extraction] | [File] | [EXIF data, dimensions, duration] | [Yes/No] |

**Storage Strategy:**
- Provider: [S3 / GCS / local filesystem / CDN]
- Path pattern: `[bucket]/[entity-type]/[entity-id]/[timestamp]-[hash].[ext]`
- Access control: [Public / Signed URLs with expiry / Private]
- CDN: [CloudFront / Cloudflare / None] — cache TTL [duration]
- Cleanup policy: [Orphan files deleted after X days / manual cleanup]

---

### Conditional Section: Real-time Architecture

> **Include this section ONLY when WebSocket or real-time features are detected in the project.**

**Connection Strategy:**
- Protocol: [WebSocket / SSE / Socket.IO / Pusher]
- Connection URL: `[wss://api.example.com/ws]`
- Authentication: [Token in query param / Cookie / First message auth]
- Heartbeat interval: [30s]

**Channel Structure:**

| Channel Pattern | Purpose | Subscribers | Publishers |
|----------------|---------|-------------|-----------|
| `user:{userId}` | [Personal notifications] | [The user] | [Server only] |
| `room:{roomId}` | [Chat room messages] | [Room members] | [Room members] |
| `admin:dashboard` | [Live admin stats] | [Admins] | [Server only] |

**Reconnection Policy:**
- Strategy: [Exponential backoff]
- Initial delay: [1s]
- Max delay: [30s]
- Max attempts: [10, then show manual reconnect button]
- State recovery: [Fetch missed events since last received event ID / Full state refresh]

**Event Catalog:**

| Event | Direction | Payload Schema | Description |
|-------|-----------|---------------|-------------|
| `[event:name]` | [S->C / C->S / Bidirectional] | `{ [field]: [type] }` | [What triggers this event and what it means] |

---

### Conditional Section: Billing Flow

> **Include this section ONLY when billing or subscription features are detected in the project.**

**Payment Provider:** [Stripe / Paddle / LemonSqueezy / etc.]

**Subscription Plans:**

| Plan | Price | Billing Cycle | Features | Limits |
|------|-------|--------------|----------|--------|
| [Free] | $0 | — | [Feature list] | [Usage limits] |
| [Pro] | $[X]/mo | Monthly/Annual | [Feature list] | [Usage limits] |
| [Enterprise] | Custom | Annual | [Feature list] | [Usage limits] |

**Payment Flow:**
```
1. User selects plan → Client creates checkout session
2. Redirect to [Provider] checkout page
3. User completes payment
4. [Provider] sends webhook → POST /api/webhooks/[provider]
5. Server updates subscription status
6. Server provisions plan features/limits
7. User redirected to success page
```

**Webhook Events Handled:**

| Event | Action | Retry Policy |
|-------|--------|-------------|
| `checkout.session.completed` | [Activate subscription, provision features] | [3 retries, exponential backoff] |
| `invoice.payment_failed` | [Downgrade plan, notify user] | [3 retries] |
| `customer.subscription.deleted` | [Revoke plan features, move to free tier] | [3 retries] |

**Refund Policy:**
- [Automatic / Manual review]
- [Prorated / Full refund / No refund]
- [Refund window: X days from purchase]

---

### Conditional Section: Multi-tenancy

> **Include this section ONLY when SaaS or multi-tenant features are detected in the project.**

**Tenant Isolation Strategy:** [Row-level security / Schema per tenant / Database per tenant]

**Data Partitioning:**
- Tenant identifier: [Column `tenantId` on all tenant-scoped tables / Schema name / Database name]
- Query enforcement: [Middleware / ORM global filter / RLS policy]
- Shared data: [List tables/resources that are NOT tenant-scoped, e.g., "plans", "system_config"]

**Cross-tenant Security Rules:**
- API layer: [Every authenticated request scoped to tenant from JWT claim]
- Database layer: [RLS policies / WHERE clause injection via ORM middleware]
- File storage: [Tenant-prefixed paths: `/{tenantId}/uploads/...`]
- Cache: [Tenant-prefixed keys: `tenant:{id}:cache:{key}`]
- Admin access: [Super-admin can switch tenant context / read-only cross-tenant view]

---

## Section 6: Data Model — Full Schema

### Entity Relationships

> High-level overview of all entity relationships in the system.

| Entity A | Relationship | Entity B | Description |
|----------|-------------|----------|-------------|
| [User] | 1:N | [Post] | [A user can create many posts] |
| [Post] | N:N | [Tag] | [A post can have many tags, a tag can be on many posts (via PostTag join)] |
| [User] | 1:1 | [Profile] | [Each user has exactly one profile] |
| [Comment] | Self-ref | [Comment] | [A comment can be a reply to another comment (parentId)] |
| [Post] | N:N | [Tag] | [Join table: PostTag] |

**Join Tables:**

| Join Table | Left Entity | Right Entity | Extra Columns |
|-----------|-------------|-------------|---------------|
| [PostTag] | [Post] | [Tag] | [None / createdAt] |
| [UserRole] | [User] | [Role] | [assignedAt, assignedBy] |

---

### Full Schema

> Column-level schema for every entity. Include ALL columns — no omissions.

#### [users]

| Column | Type | Constraints |
|--------|------|------------|
| id | String | PK, cuid() |
| email | String | UNIQUE, NOT NULL |
| passwordHash | String | NOT NULL |
| name | String | NOT NULL |
| role | Int | NOT NULL, DEFAULT 0 |
| status | Int | NOT NULL, DEFAULT 0 |
| avatarUrl | String | NULLABLE |
| lastLoginAt | DateTime | NULLABLE |
| createdAt | DateTime | NOT NULL, DEFAULT now() |
| updatedAt | DateTime | NOT NULL, auto-update |
| deletedAt | DateTime | NULLABLE (soft delete) |

#### [posts]

| Column | Type | Constraints |
|--------|------|------------|
| id | String | PK, cuid() |
| title | String | NOT NULL, MAX 200 |
| content | Text | NOT NULL |
| status | Int | NOT NULL, DEFAULT 0 |
| authorId | String | FK -> users.id, NOT NULL |
| publishedAt | DateTime | NULLABLE |
| createdAt | DateTime | NOT NULL, DEFAULT now() |
| updatedAt | DateTime | NOT NULL, auto-update |
| deletedAt | DateTime | NULLABLE (soft delete) |

#### [Entity Name]

| Column | Type | Constraints |
|--------|------|------------|
| [column] | [Type] | [Constraints] |

*(Repeat for every entity in the system)*

---

### Status Enums

> Map all integer status values to their meaning and the entities that use them.

| Enum Name | Value | Label | Used By |
|-----------|-------|-------|---------|
| UserStatus.PENDING | 0 | Pending | users |
| UserStatus.ACTIVE | 1 | Active | users |
| UserStatus.SUSPENDED | 2 | Suspended | users |
| UserStatus.BANNED | 3 | Banned | users |
| PostStatus.DRAFT | 0 | Draft | posts |
| PostStatus.PUBLISHED | 1 | Published | posts |
| PostStatus.ARCHIVED | 2 | Archived | posts |
| [EnumName.VALUE] | [int] | [Label] | [entity] |

### Index Hints

> Recommended database indexes for query performance.

| Entity | Column(s) | Type | Reason |
|--------|-----------|------|--------|
| users | email | UNIQUE | Login lookup |
| users | status, createdAt | COMPOSITE | Admin user list filter + sort |
| posts | authorId, status | COMPOSITE | Author's post list filtered by status |
| posts | publishedAt | B-TREE | Public feed sorted by date |
| [entity] | [column(s)] | [UNIQUE / COMPOSITE / B-TREE / GIN] | [Why this index is needed] |

### Soft Delete

| Entity | Soft Delete | Retention Period | Hard Delete Policy |
|--------|------------|-----------------|-------------------|
| users | Yes | 90 days | GDPR request or after retention |
| posts | Yes | 30 days | After retention period |
| [entity] | [Yes/No] | [Duration or N/A] | [When permanently deleted] |

---

## Section 7: Permission Matrix

### Action x Role Matrix

> `own` means the user can only perform the action on resources they own.

| Action | [Guest] | [User] | [Moderator] | [Admin] |
|--------|---------|--------|-------------|---------|
| View public [resources] | Yes | Yes | Yes | Yes |
| Create [resource] | No | Yes | Yes | Yes |
| Edit [resource] | No | own | Yes | Yes |
| Delete [resource] | No | own | Yes | Yes |
| View all users | No | No | Yes | Yes |
| Manage users | No | No | No | Yes |
| Access admin dashboard | No | No | No | Yes |
| [Additional action] | [Yes/No/own] | [Yes/No/own] | [Yes/No/own] | [Yes/No/own] |

### Ownership Rules

| Resource | Owner Field | Ownership Determined By |
|----------|-----------|------------------------|
| [Post] | `authorId` | [The user who created the post] |
| [Comment] | `userId` | [The user who wrote the comment] |
| [Profile] | `userId` | [The user the profile belongs to] |
| [Resource] | `[field]` | [How ownership is established] |

### Role Hierarchy

```
[Guest] (no auth)
  └── [User] (authenticated)
       └── [Moderator] (elevated privileges)
            └── [Admin] (full access)
```

**Escalation rules:**
- [Only Admin can promote User to Moderator]
- [Only Admin can promote Moderator to Admin]
- [Role changes require confirmation and are logged in audit trail]

---

## Section 8: Open Questions

| # | Question | Context / Impact | Owner | Status |
|---|----------|-----------------|-------|--------|
| 1 | [Open question about the product] | [Why it matters and what depends on the answer] | [Person/team responsible] | [Open / Resolved / Deferred] |
| 2 | [Open question] | [Context / Impact] | [Owner] | [Status] |

---

## Additional Sections

### Additional Questions

**Required** — Must be answered before development begins:

| # | Question | Blocks |
|---|----------|--------|
| 1 | [Critical question, e.g., "What is the maximum number of concurrent users expected?"] | [What it blocks, e.g., "Infrastructure sizing, WebSocket architecture"] |
| 2 | [Critical question] | [What it blocks] |

**Recommended** — Should be answered but development can start:

| # | Question | Affects |
|---|----------|---------|
| 1 | [Important question, e.g., "Should users be able to export their data?"] | [What it affects, e.g., "User settings page, GDPR compliance"] |
| 2 | [Important question] | [What it affects] |

---

### Feature Change Log

> Track all changes to the PRD since initial version. Every modification must be logged here.

| Version | Change Type | Risk | Before | After | Source |
|---------|------------|------|--------|-------|--------|
| [v1.1] | [Added / Modified / Removed] | [Low / Medium / High] | [Previous state or "N/A" for new] | [New state] | [Stakeholder request / User feedback / Technical constraint] |

---

### UX Flow Improvement Suggestions

> Optional section. Include when the agent identifies potential UX improvements during PRD generation.

| Current Flow | Issue | Suggestion | Priority | Complexity |
|-------------|-------|-----------|----------|------------|
| [Describe current user flow] | [UX problem identified] | [Proposed improvement] | [P0 / P1 / P2 / P3] | [Low / Medium / High] |
