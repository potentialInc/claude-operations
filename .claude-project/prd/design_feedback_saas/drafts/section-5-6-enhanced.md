# Section 5 — Tech Stack & Architecture

## 5.1 Architecture Overview

멀티테넌트 SaaS 구조로, Next.js 기반 SSR/CSR 혼합 프론트엔드와 NestJS REST API 백엔드를 분리 배포하며, PostgreSQL을 단일 데이터베이스로 사용하고 Socket.IO를 통한 실시간 채널을 별도 레이어로 운영한다. 모든 데이터는 `organization_id` 기반으로 테넌트 격리되며, 파일 저장은 AWS S3(또는 호환 스토리지)에 위임한다.

### Directory Structure

```
root/
├── apps/
│   ├── web/                        # Next.js 프론트엔드
│   │   ├── app/                    # App Router (Next.js 14+)
│   │   │   ├── (auth)/             # 로그인/회원가입
│   │   │   ├── (dashboard)/        # 인증된 사용자 레이아웃
│   │   │   │   ├── org/[orgSlug]/
│   │   │   │   │   ├── projects/
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   └── [projectId]/
│   │   │   │   │   │       ├── screens/
│   │   │   │   │   │       │   └── [screenId]/
│   │   │   │   │   │       │       └── feedback/    # 핵심 피드백 뷰어
│   │   │   │   │   │       └── members/
│   │   │   │   │   └── settings/
│   │   │   │   │       ├── organization/
│   │   │   │   │       ├── team/
│   │   │   │   │       └── billing/
│   │   │   └── superadmin/         # 슈퍼어드민 전용 레이아웃
│   │   ├── components/
│   │   │   ├── feedback/
│   │   │   │   ├── PinOverlay.tsx  # 이미지 위 핀 렌더링 (Canvas/절대좌표)
│   │   │   │   ├── CommentPanel.tsx
│   │   │   │   └── VersionComparator.tsx
│   │   │   ├── ui/                 # shadcn/ui 공통 컴포넌트
│   │   │   └── layout/
│   │   ├── hooks/
│   │   │   └── useRealtimeFeedback.ts   # Socket.IO 클라이언트 훅
│   │   ├── lib/
│   │   │   ├── api.ts              # API 클라이언트 (axios/fetch)
│   │   │   └── socket.ts           # Socket.IO 클라이언트 초기화
│   │   └── i18n/
│   │       ├── en.json
│   │       └── ko.json
│   └── api/                        # NestJS 백엔드
│       ├── src/
│       │   ├── auth/
│       │   ├── organizations/
│       │   ├── projects/
│       │   ├── screens/
│       │   ├── versions/
│       │   ├── pins/
│       │   ├── comments/
│       │   ├── notifications/
│       │   ├── realtime/           # Socket.IO Gateway
│       │   ├── billing/            # Stripe 모듈 (추후)
│       │   ├── audit/
│       │   ├── storage/            # S3 연동 추상화
│       │   ├── superadmin/
│       │   └── common/
│       │       ├── guards/
│       │       ├── decorators/
│       │       └── interceptors/
│       └── prisma/
│           └── schema.prisma
├── packages/
│   ├── types/                      # 공유 TypeScript 타입 (monorepo)
│   └── config/                     # 공유 eslint/tsconfig
└── infra/                          # IaC (Terraform 또는 Docker Compose)
```

---

## 5.2 Technologies

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Frontend Framework | Next.js (React) | 14.x (App Router) | SSR/CSR 혼합 렌더링, 라우팅, API Routes |
| UI Component | shadcn/ui + Tailwind CSS | shadcn latest / Tailwind 3.x | 공통 UI 컴포넌트 라이브러리, 반응형 스타일링 |
| State Management | Zustand | 4.x | 클라이언트 글로벌 상태 (핀 선택, 활성 버전 등) |
| Backend Framework | NestJS (Node.js) | 10.x | REST API 서버, 모듈 기반 구조, Guard/Interceptor |
| Language | TypeScript | 5.x | 프론트엔드/백엔드 공통, 타입 안전성 |
| ORM | Prisma | 5.x | PostgreSQL 스키마 관리, 타입세이프 쿼리, 마이그레이션 |
| Database | PostgreSQL | 16.x | 메인 관계형 DB, 멀티테넌시 데이터 저장 |
| Realtime | Socket.IO | 4.x | WebSocket 기반 실시간 핀/코멘트/상태 브로드캐스트 |
| File Storage | AWS S3 (또는 S3-compatible) | AWS SDK v3 | 스크린샷 이미지, 첨부파일 저장 |
| Authentication | JWT + Refresh Token (jose) | jose 5.x | Stateless 인증, 토큰 갱신 전략 |
| Email | SendGrid (또는 AWS SES) | SendGrid v3 API | 초대 이메일, 알림 이메일 발송 |
| Payment | Stripe | stripe-node 14.x | 구독결제 (추후 연동) |
| Image Comparison | react-compare-slider | 3.x | 버전 side-by-side / overlay 비교 |
| Pin Overlay | CSS 절대좌표 오버레이 (자체 구현) | — | 이미지 위 핀 배치 (x_pct/y_pct 기반 상대좌표) |
| i18n | next-intl | 3.x | EN/KO 다국어 지원, App Router 통합 |
| Validation | class-validator + class-transformer | 0.14.x | NestJS DTO 입력 검증 |
| Caching | Redis (ioredis) | 7.x / ioredis 5.x | 세션, 실시간 방(room) 메타, Rate Limit |
| Process Manager | PM2 (개발) / Docker (운영) | PM2 5.x / Docker 24.x | 서버 프로세스 관리, 컨테이너화 |
| Monorepo | Turborepo | 2.x | apps/packages 빌드 오케스트레이션 |

---

## 5.3 Third-Party Integrations

| Service | Purpose |
|---------|---------|
| Stripe | 조직 구독결제 (Free/Pro/Enterprise 요금제), 웹훅 수신으로 구독 상태 동기화 (추후 연동) |
| SendGrid (또는 AWS SES) | 이메일 초대 발송 (팀원/클라이언트), 알림 이메일 (코멘트, 상태 변경), 비밀번호 재설정 |
| AWS S3 (또는 GCS / S3-compatible 스토리지) | 스크린샷 버전 이미지 저장, 코멘트 첨부파일 저장, 조직 로고 이미지 저장, Presigned URL로 직접 업로드/다운로드. **대안**: Google Cloud Storage(GCS) — GCS XML API의 S3 호환 인터페이스 또는 GCS 네이티브 SDK 사용 가능 |
| Redis Cloud (또는 AWS ElastiCache) | Socket.IO 어댑터 (다중 서버 인스턴스 간 이벤트 브로드캐스트), 세션 캐시, Rate Limit 저장소 |

---

## 5.4 Key Decisions

| Decision | Rationale |
|----------|-----------|
| Next.js App Router + NestJS 분리 배포 | 프론트엔드/백엔드 독립 배포 및 확장성 확보. 향후 모바일 앱 추가 시 동일 API 재사용 가능. BFF(Backend for Frontend) 패턴이 필요해질 경우 Next.js API Routes로 래핑 가능 |
| Prisma ORM + PostgreSQL | 타입세이프 쿼리와 마이그레이션 자동화로 개발 속도 향상. PostgreSQL의 행 수준 보안(Row Level Security) 옵션을 통해 멀티테넌시 격리 강화 가능. Prisma의 스키마 기반 마이그레이션으로 팀 협업 시 DB 변경 이력 관리 |
| Socket.IO (WebSocket) 기반 실시간 | 핀/코멘트 실시간 반영 요구사항 충족. SSE 대비 양방향 통신 지원으로 미래 기능(공동 커서, 라이브 리뷰) 확장에 유리. Redis Adapter로 수평 확장(horizontal scaling) 지원 |
| 핀 좌표를 이미지 크기 대비 백분율(x_pct, y_pct)로 저장 | 절대 픽셀값 저장 시 이미지 렌더링 크기 변화에 따라 핀 위치가 어긋나는 문제 방지. 반응형 뷰어에서 화면 크기와 무관하게 정확한 위치 재현 가능 [💡 권장적용] |
| Soft Delete + Audit Log 분리 | 데이터 복구 가능성 확보 및 규정 준수(GDPR 대응 기반 마련). Audit Log는 별도 테이블로 분리하여 주 테이블 성능에 영향 없이 이력 추적 |
| Monorepo (Turborepo) | 프론트엔드/백엔드 간 TypeScript 타입 공유로 API 계약 불일치 방지. 단일 저장소에서 일관된 린팅/빌드 파이프라인 적용 [💡 권장적용] |

---

## 5.5 Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL 연결 문자열 (Prisma용, e.g. `postgresql://user:pass@host:5432/dbname`) |
| `REDIS_URL` | Redis 연결 문자열 (Socket.IO Adapter, 세션 캐시용) |
| `JWT_SECRET` | Access Token 서명 키 (최소 32자 랜덤 문자열) |
| `JWT_REFRESH_SECRET` | Refresh Token 서명 키 (Access Token과 별도 키 사용 필수) |
| `JWT_ACCESS_EXPIRES_IN` | Access Token 만료 시간 (권장: `15m`) |
| `JWT_REFRESH_EXPIRES_IN` | Refresh Token 만료 시간 (권장: `7d`) |
| `AWS_ACCESS_KEY_ID` | AWS S3 접근 키 ID |
| `AWS_SECRET_ACCESS_KEY` | AWS S3 비밀 접근 키 |
| `AWS_S3_BUCKET_NAME` | 스크린샷/첨부파일 저장 S3 버킷명 |
| `AWS_S3_REGION` | S3 버킷 리전 (e.g. `ap-northeast-2`) |
| `AWS_S3_PRESIGNED_URL_EXPIRES` | Presigned URL 만료 초 (권장: `3600`) |
| `SENDGRID_API_KEY` | SendGrid API 키 (초대/알림 이메일 발송) |
| `EMAIL_FROM_ADDRESS` | 발신자 이메일 주소 (e.g. `no-reply@yourapp.com`) |
| `STRIPE_SECRET_KEY` | Stripe 비밀 키 (구독결제 처리, 추후 연동) |
| `STRIPE_WEBHOOK_SECRET` | Stripe 웹훅 서명 검증 시크릿 (추후 연동) |
| `NEXT_PUBLIC_API_BASE_URL` | 프론트엔드에서 사용하는 백엔드 API 베이스 URL |
| `NEXT_PUBLIC_SOCKET_URL` | 프론트엔드 Socket.IO 연결 URL |
| `SUPERADMIN_EMAIL` | 슈퍼어드민 초기 계정 이메일 (시드 스크립트용) [💡 권장적용] |
| `APP_BASE_URL` | 서비스 베이스 URL (초대 링크 생성 시 사용) |
| `NODE_ENV` | 실행 환경 (`development` / `production` / `test`) |

---

# Section 6 — Data Model

## 6.1 Entity List

| Entity | Key Fields | Description |
|--------|-----------|-------------|
| **Organization** | `id` (PK, UUID), `name` (VARCHAR 100, NOT NULL), `slug` (VARCHAR 50, UNIQUE, NOT NULL), `logo_url` (VARCHAR 500, NULL), `logo_key` (VARCHAR 500, NULL), `status` (SMALLINT, NOT NULL, default 1), `plan_id` (FK → Plan), `owner_id` (FK → User), `created_at`, `updated_at`, `deleted_at` | 멀티테넌시의 최상위 단위. `slug`는 URL 식별자. `status`는 OrganizationStatus enum. `logo_url`/`logo_key`는 조직 로고 이미지의 공개 URL 및 스토리지 객체 키 |
| **User** | `id` (PK, UUID), `email` (VARCHAR 255, UNIQUE, NOT NULL), `password_hash` (VARCHAR 255), `name` (VARCHAR 100, NOT NULL), `avatar_url` (VARCHAR 500), `is_super_admin` (BOOLEAN, default false), `locale` (VARCHAR 5, default `en`), `last_login_at`, `created_at`, `updated_at`, `deleted_at` | 플랫폼 전체 사용자. 소셜 로그인 추가 시 `password_hash` NULL 허용 [💡 권장적용] |
| **OrganizationMember** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `user_id` (FK → User, NOT NULL), `role` (SMALLINT, NOT NULL), `invited_by` (FK → User), `joined_at`, `created_at`, `updated_at`, `deleted_at` | Organization ↔ User N:N 조인 테이블. `role`은 OrgRole enum (Owner/Admin/Member) |
| **Project** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `name` (VARCHAR 200, NOT NULL), `description` (TEXT), `status` (SMALLINT, NOT NULL, default 1), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 조직 내 단위 프로젝트. `organization_id` 기반 테넌트 격리 |
| **ProjectMember** | `id` (PK, UUID), `project_id` (FK → Project, NOT NULL), `user_id` (FK → User, NOT NULL), `role` (SMALLINT, NOT NULL), `invited_by` (FK → User), `created_at`, `updated_at`, `deleted_at` | Project ↔ User N:N 조인 테이블. Client 역할 포함. `role`은 ProjectRole enum |
| **Screen** | `id` (PK, UUID), `project_id` (FK → Project, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `name` (VARCHAR 200, NOT NULL), `description` (TEXT), `order` (INTEGER, default 0), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 프로젝트 내 개별 화면. `organization_id` 비정규화로 테넌트 격리 쿼리 최적화 |
| **ScreenVersion** | `id` (PK, UUID), `screen_id` (FK → Screen, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `version_number` (INTEGER, NOT NULL), `label` (VARCHAR 100), `image_url` (VARCHAR 500, NOT NULL), `image_key` (VARCHAR 500, NOT NULL), `width` (INTEGER), `height` (INTEGER), `file_size` (BIGINT), `mime_type` (VARCHAR 50), `uploaded_by` (FK → User, NOT NULL), `created_at`, `deleted_at` | 화면의 스크린샷 버전. `version_number`는 screen 내 단조 증가. `image_key`는 S3 객체 키 |
| **Pin** | `id` (PK, UUID), `screen_version_id` (FK → ScreenVersion, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `x_pct` (DECIMAL(5,2), NOT NULL), `y_pct` (DECIMAL(5,2), NOT NULL), `index` (INTEGER, NOT NULL), `status` (SMALLINT, NOT NULL, default 1), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 이미지 위 핀 좌표. `x_pct`/`y_pct`는 이미지 크기 대비 백분율(0.00~100.00)로 저장하여 반응형 뷰어에서 위치 불변 보장 [💡 권장적용]. `index`는 핀 번호(표시용) |
| **Comment** | `id` (PK, UUID), `pin_id` (FK → Pin, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `content` (TEXT, NOT NULL), `status` (SMALLINT, NOT NULL, default 1), `created_by` (FK → User, NOT NULL), `resolved_by` (FK → User, NULL), `resolved_at`, `created_at`, `updated_at`, `deleted_at` | 핀에 연결된 최상위 코멘트. `status`는 CommentStatus enum (open/in_progress/resolved) |
| **Reply** | `id` (PK, UUID), `comment_id` (FK → Comment, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `content` (TEXT, NOT NULL), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 코멘트 하위 답글. 단일 depth (Reply의 Reply 불허) |
| **FileAttachment** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `comment_id` (FK → Comment, NULL), `reply_id` (FK → Reply, NULL), `uploader_id` (FK → User, NOT NULL), `file_name` (VARCHAR 255, NOT NULL), `file_key` (VARCHAR 500, NOT NULL), `file_url` (VARCHAR 500, NOT NULL), `file_size` (BIGINT, NOT NULL), `mime_type` (VARCHAR 100, NOT NULL), `created_at`, `deleted_at` | 코멘트/답글에 첨부된 파일. `comment_id` 또는 `reply_id` 중 하나만 NOT NULL [💡 권장적용] |
| **Invitation** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `project_id` (FK → Project, NULL), `email` (VARCHAR 255, NOT NULL), `role` (SMALLINT, NOT NULL), `token` (VARCHAR 255, UNIQUE, NOT NULL), `status` (SMALLINT, NOT NULL, default 1), `invited_by` (FK → User, NOT NULL), `expires_at` (TIMESTAMPTZ, NOT NULL), `accepted_at`, `created_at`, `updated_at` | 이메일 초대 토큰 관리. `project_id`가 NULL이면 조직 초대, NOT NULL이면 프로젝트 초대 |
| **PasswordResetToken** | `id` (PK, UUID), `user_id` (FK → User, NOT NULL), `token` (VARCHAR 255, UNIQUE, NOT NULL), `expires_at` (TIMESTAMPTZ, NOT NULL), `used_at` (TIMESTAMPTZ, NULL), `created_at` | 비밀번호 재설정 토큰 관리. `token`은 서버에서 생성한 고유 랜덤 문자열(해시 저장 권장). `used_at`이 NULL이 아니면 이미 사용된 토큰으로 간주하여 재사용 불가. 만료(`expires_at`) 및 사용 여부 검증 후 비밀번호 변경 처리 |
| **PlatformSettings** | `id` (PK, UUID), `key` (VARCHAR 100, UNIQUE, NOT NULL), `value` (TEXT, NOT NULL), `description` (VARCHAR 500, NULL), `updated_by` (FK → User, NOT NULL), `created_at`, `updated_at` | 플랫폼 전역 설정 값 저장. 슈퍼어드민 설정 관리 페이지에서 조회/수정. `key`-`value` 구조로 설정 항목 확장에 유연하게 대응 (e.g. `max_orgs_per_user`, `maintenance_mode`, `default_plan_slug` 등) |
| **Notification** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `user_id` (FK → User, NOT NULL), `type` (SMALLINT, NOT NULL), `title` (VARCHAR 255, NOT NULL), `body` (TEXT), `is_read` (BOOLEAN, default false), `ref_entity_type` (VARCHAR 50), `ref_entity_id` (UUID), `created_at`, `read_at` | 인앱 알림. `ref_entity_type`/`ref_entity_id`로 관련 엔티티 참조(다형성 참조) [💡 권장적용] |
| **AuditLog** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `actor_id` (FK → User, NOT NULL), `action` (VARCHAR 100, NOT NULL), `entity_type` (VARCHAR 50, NOT NULL), `entity_id` (UUID, NOT NULL), `old_value` (JSONB), `new_value` (JSONB), `ip_address` (VARCHAR 45), `user_agent` (VARCHAR 500), `created_at` | 주요 액션 이력. `old_value`/`new_value`는 JSONB로 변경 전후 스냅샷 저장. Soft Delete 미적용(불변 이력) |
| **Plan** | `id` (PK, UUID), `name` (VARCHAR 50, NOT NULL), `slug` (VARCHAR 50, UNIQUE, NOT NULL), `price_monthly` (DECIMAL(10,2)), `price_yearly` (DECIMAL(10,2)), `max_projects` (INTEGER), `max_members` (INTEGER), `max_storage_gb` (INTEGER), `stripe_price_id_monthly` (VARCHAR 100), `stripe_price_id_yearly` (VARCHAR 100), `is_active` (BOOLEAN, default true), `created_at`, `updated_at` | 구독 요금제 정의. `max_*` 필드로 플랜별 제한 설정 [💡 권장적용] |
| **Subscription** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL, UNIQUE), `plan_id` (FK → Plan, NOT NULL), `stripe_customer_id` (VARCHAR 100), `stripe_subscription_id` (VARCHAR 100), `status` (SMALLINT, NOT NULL), `billing_cycle` (SMALLINT), `current_period_start`, `current_period_end`, `canceled_at`, `created_at`, `updated_at` | 조직의 현재 구독 상태. `organization_id`에 UNIQUE 제약으로 1:1 보장. Stripe 웹훅으로 상태 동기화 |

---

## 6.2 Entity Relationships

```
Organization (1) ─── (N) OrganizationMember (N) ─── (1) User
  │  [User ↔ Organization N:N 조인 테이블: OrganizationMember]
  │
Organization (1) ─── (N) Project
  │
Organization (1) ─── (1) Subscription
  │
Organization (1) ─── (N) AuditLog
  │
Organization (1) ─── (N) Notification
  │
Project (1) ─── (N) ProjectMember (N) ─── (1) User
  │  [User ↔ Project N:N 조인 테이블: ProjectMember]
  │
Project (1) ─── (N) Screen
  │
Screen (1) ─── (N) ScreenVersion
  │
ScreenVersion (1) ─── (N) Pin
  │
Pin (1) ─── (N) Comment
  │  [Pin당 최초 Comment가 핀의 대표 코멘트]
  │
Comment (1) ─── (N) Reply
  │
Comment (1) ─── (N) FileAttachment  [comment_id NOT NULL, reply_id NULL]
Reply   (1) ─── (N) FileAttachment  [reply_id NOT NULL, comment_id NULL]
  │
Organization (1) ─── (N) Invitation
Project      (1) ─── (N) Invitation  [project_id NULL이면 조직 초대, NOT NULL이면 프로젝트 초대]
  │
User (1) ─── (N) PasswordResetToken  [비밀번호 재설정 토큰. used_at NULL = 미사용]
  │
Plan (1) ─── (N) Organization  [Organization.plan_id FK]
Plan (1) ─── (N) Subscription
```

**카디널리티 상세:**

- `Organization` : `OrganizationMember` = **1:N** (한 조직에 다수 멤버십 레코드)
- `User` : `OrganizationMember` = **1:N** (한 유저가 다수 조직 멤버 가능)
- `Organization` ↔ `User` = **N:N** (중간 테이블: `OrganizationMember`)
- `Organization` : `Project` = **1:N**
- `Project` : `ProjectMember` = **1:N**
- `User` : `ProjectMember` = **1:N**
- `Project` ↔ `User` = **N:N** (중간 테이블: `ProjectMember`)
- `Organization` : `Subscription` = **1:1** (`Subscription.organization_id` UNIQUE)
- `Plan` : `Subscription` = **1:N**
- `Project` : `Screen` = **1:N**
- `Screen` : `ScreenVersion` = **1:N** (버전 히스토리)
- `ScreenVersion` : `Pin` = **1:N**
- `Pin` : `Comment` = **1:N** (핀 하나에 여러 코멘트 스레드 가능)
- `Comment` : `Reply` = **1:N** (단일 depth)
- `Comment` : `FileAttachment` = **1:N** (`comment_id` 기준)
- `Reply` : `FileAttachment` = **1:N** (`reply_id` 기준)
- `Organization` : `Invitation` = **1:N**
- `Project` : `Invitation` = **1:N** (project_id NULL 가능 → 조직 초대)
- `User` : `PasswordResetToken` = **1:N** (한 유저가 여러 토큰 발급 가능, 최신 토큰만 유효 처리)
- `User` : `Notification` = **1:N**
- `Organization` : `AuditLog` = **1:N**
- `User` : `AuditLog` = **1:N** (actor 기준)

**Self-referencing:**
- `User.invited_by` → `User` (self-ref, NULL 허용 — 초대로 가입한 경우 초대자 추적) [💡 권장적용]

---

## 6.3 Status Enums

| Enum Name | Values (DB 저장값 — 정수) | Used By |
|-----------|--------------------------|---------|
| `OrganizationStatus` | `1` = active, `2` = suspended, `3` = deleted | Organization.status |
| `OrgRole` | `1` = owner, `2` = admin, `3` = member | OrganizationMember.role |
| `ProjectRole` | `1` = admin, `2` = member, `3` = client | ProjectMember.role |
| `ProjectStatus` | `1` = active, `2` = archived, `3` = deleted | Project.status |
| `CommentStatus` | `1` = open, `2` = in_progress, `3` = resolved | Comment.status, Pin.status |
| `PinStatus` | `1` = open, `2` = in_progress, `3` = resolved | Pin.status (Pin의 대표 코멘트 상태 반영) |
| `InvitationStatus` | `1` = pending, `2` = accepted, `3` = expired, `4` = canceled | Invitation.status |
| `SubscriptionStatus` | `1` = active, `2` = past_due, `3` = canceled, `4` = trialing, `5` = incomplete | Subscription.status |
| `BillingCycle` | `1` = monthly, `2` = yearly | Subscription.billing_cycle |
| `NotificationType` | `1` = comment_added, `2` = comment_status_changed, `3` = reply_added, `4` = invitation_received, `5` = invitation_accepted, `6` = member_removed, `7` = pin_resolved | Notification.type |

---

## 6.4 Index Hints

| Entity | Column(s) | Type | Reason |
|--------|-----------|------|--------|
| Organization | `slug` | UNIQUE INDEX | 조직 URL 조회 (org slug 기반 라우팅) |
| OrganizationMember | `(organization_id, user_id)` | UNIQUE INDEX | 동일 유저의 중복 멤버십 방지 + 조인 쿼리 최적화 |
| OrganizationMember | `user_id` | INDEX | 특정 유저가 속한 조직 목록 조회 |
| Project | `organization_id` | INDEX | 조직별 프로젝트 목록 (테넌트 격리 쿼리) |
| Project | `(organization_id, status)` | COMPOSITE INDEX | 활성 프로젝트 필터링 |
| ProjectMember | `(project_id, user_id)` | UNIQUE INDEX | 중복 멤버십 방지 + 프로젝트 접근 권한 확인 |
| ProjectMember | `user_id` | INDEX | 유저가 속한 프로젝트 목록 |
| Screen | `(project_id, deleted_at)` | COMPOSITE INDEX | 프로젝트 내 활성 화면 목록 |
| Screen | `organization_id` | INDEX | 테넌트 격리 (organization_id 포함 쿼리) |
| ScreenVersion | `(screen_id, version_number)` | UNIQUE INDEX | 버전 중복 방지 + 버전 순서 조회 |
| ScreenVersion | `screen_id` | INDEX | 화면의 버전 히스토리 목록 |
| Pin | `screen_version_id` | INDEX | 특정 버전의 핀 전체 조회 (피드백 뷰어 렌더링) |
| Pin | `(screen_version_id, deleted_at)` | COMPOSITE INDEX | 활성 핀 필터링 |
| Comment | `pin_id` | INDEX | 핀별 코멘트 목록 |
| Comment | `(pin_id, status)` | COMPOSITE INDEX | 상태별 코멘트 필터링 (open/resolved) |
| Comment | `organization_id` | INDEX | 테넌트 격리 |
| Reply | `comment_id` | INDEX | 코멘트 답글 목록 |
| FileAttachment | `comment_id` | INDEX | 코멘트 첨부파일 조회 |
| FileAttachment | `reply_id` | INDEX | 답글 첨부파일 조회 |
| Invitation | `token` | UNIQUE INDEX | 초대 수락 시 토큰 조회 |
| Invitation | `(organization_id, email, status)` | COMPOSITE INDEX | 중복 초대 방지 + 미처리 초대 조회 |
| PasswordResetToken | `token` | UNIQUE INDEX | 비밀번호 재설정 요청 시 토큰 조회 |
| PasswordResetToken | `(user_id, used_at, expires_at)` | COMPOSITE INDEX | 유저의 유효한 미사용 토큰 조회 |
| PlatformSettings | `key` | UNIQUE INDEX | 설정 키 기반 단건 조회 |
| Notification | `(user_id, is_read, created_at DESC)` | COMPOSITE INDEX | 인앱 알림 목록 (읽지 않은 것 우선) |
| AuditLog | `(organization_id, created_at DESC)` | COMPOSITE INDEX | 조직별 감사 로그 최신순 조회 |
| AuditLog | `(entity_type, entity_id)` | COMPOSITE INDEX | 특정 엔티티의 변경 이력 조회 |
| User | `email` | UNIQUE INDEX | 로그인, 초대 수락 시 이메일 조회 |

---

## 6.5 Soft Delete

| Entity | Soft Delete | Retention |
|--------|-------------|-----------|
| Organization | ✅ (`deleted_at` TIMESTAMPTZ) | 90일 후 하드 삭제 [💡 권장적용] |
| User | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 (GDPR 대응 고려) [💡 권장적용] |
| OrganizationMember | ✅ (`deleted_at` TIMESTAMPTZ) | 즉시 논리 삭제, 30일 후 하드 삭제 |
| Project | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 [💡 권장적용] |
| ProjectMember | ✅ (`deleted_at` TIMESTAMPTZ) | 즉시 논리 삭제, 30일 후 하드 삭제 |
| Screen | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| ScreenVersion | ✅ (`deleted_at` TIMESTAMPTZ) | 90일 후 하드 삭제 (S3 파일은 별도 정책으로 관리) |
| Pin | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| Comment | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| Reply | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| FileAttachment | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 (S3 파일 동시 삭제) |
| Invitation | ❌ (status 컬럼으로 상태 관리) | 만료/취소 후 90일 보관 후 하드 삭제 [💡 권장적용] |
| PasswordResetToken | ❌ (하드 삭제 허용) | 사용 완료(`used_at` NOT NULL) 또는 만료(`expires_at` 경과) 후 배치 정리 |
| PlatformSettings | ❌ (하드 삭제 또는 값 갱신으로 관리) | 영구 보존 (설정 이력은 AuditLog로 추적) |
| Notification | ❌ (하드 삭제 허용) | 읽은 후 90일 보관, 미읽음은 영구 보존 [💡 권장적용] |
| AuditLog | ❌ (불변 이력, 삭제 금지) | 1년 보존 후 Cold Storage(S3 Glacier) 아카이빙 [💡 권장적용] |
| Plan | ❌ (`is_active` 플래그로 비활성화) | 영구 보존 (참조 무결성) |
| Subscription | ❌ (status 컬럼으로 상태 관리) | 영구 보존 (결제 이력 규정 준수) |
