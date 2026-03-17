# DesignPin — Product Requirements Document
**Version:** 1.0
**Date:** 2026-03-12
**Status:** Draft

---

## 0. Project Overview

### Product

| Field | Value |
|-------|-------|
| Name | DesignPin `[📌 권장적용됨]` |
| Type | SaaS Web Platform |
| Deadline | 미지정 `[💡 권장적용: 클라이언트 확인 필요]` |
| Status | Draft |

### Description

DesignPin은 디자인 시안 이미지 위에 핀(좌표)을 찍고 코멘트를 남기는 협업 피드백 SaaS 플랫폼입니다. 조직(Organization) 기반 멀티테넌시 구조로 Owner, Admin, Member, Client 4개의 역할이 프로젝트별로 협업하며, 스크린샷 버전 히스토리와 버전 비교 기능을 통해 디자인 변경 이력을 관리합니다. 핀-코멘트 시스템은 WebSocket 기반 실시간 업데이트를 지원하고, 추후 Figma 등 디자인 도구 리뷰 기능으로 확장 가능한 구조를 목표로 합니다.

### Goals

**Primary Goal:**
디자인 시안 리뷰 과정에서 발생하는 피드백을 이미지 위 핀 기반으로 구조화하여, 팀과 클라이언트 간 소통 효율을 극대화한다.

**Secondary Goals:**
1. 조직별 완전한 데이터 격리(멀티테넌시)로 SaaS 보안성 확보
2. 스크린샷 버전 히스토리 및 버전 비교(overlay / side-by-side)로 디자인 변경 이력 추적
3. 코멘트 상태 관리(open → in_progress → resolved)로 피드백 처리 현황 가시화
4. 이메일 초대 및 실시간 알림으로 협업 참여 장벽 최소화
5. Stripe 구독결제 연동을 통한 SaaS 수익화 기반 마련 (추후)

### Target Audience

| Segment | Description | Scale |
|---------|-------------|-------|
| 디자인 에이전시 | 클라이언트 다수를 대상으로 디자인 시안을 반복 검토하는 팀 | 팀 5~50명 |
| 인하우스 디자인팀 | 내부 PM/개발자와 디자인 리뷰를 진행하는 조직 내 팀 | 팀 3~20명 |
| 프리랜서 디자이너 | 개인 작업물을 클라이언트에게 공유하고 피드백을 수집하는 개인 | 1~3명 |

### User Types

| Type | DB Value | Description | Key Actions |
|------|----------|-------------|-------------|
| SuperAdmin | 0 | 플랫폼 전체를 관리하는 내부 운영자. 모든 조직에 접근 가능 | 전체 조직 조회, 조직 상태 변경(활성/비활성/삭제) |
| Owner | 1 | 조직을 생성하고 소유한 최상위 권한 사용자 | 조직 생성/수정, 빌링 관리, 멤버 초대/역할 변경/제거 |
| Admin | 2 | 조직 내 프로젝트를 생성·운영하는 관리자 | 프로젝트 CRUD, 팀원/클라이언트 초대, 피드백·화면 관리 |
| Member | 3 | 조직 내 실무 작업자 (디자이너, 개발자 등) | 화면 등록/관리, 버전 업로드, 핀 코멘트 답변, 상태 변경 |
| Client | 4 | 배정된 프로젝트만 열람 가능한 외부 검토자 | 이미지 위 핀 찍기, 코멘트 작성, 코멘트 답글 |

### User Status

| Status | DB Value | Behavior |
|--------|----------|----------|
| active | 1 | 정상 로그인 및 모든 권한 행사 가능 |
| invited | 2 | 초대 이메일 발송 완료, 수락 대기 중. 로그인 불가 |
| suspended | 3 | 관리자에 의해 비활성화. 로그인 시 접근 거부 메시지 표시 |
| deleted | 0 | Soft delete 상태. 데이터 보존, 로그인 불가, 목록 노출 안 됨 |

### User Relationships

```
Platform
└── Organization (조직, 멀티테넌시 단위)
    ├── Owner (1명, 조직 소유)
    ├── Admin (N명, 조직 멤버)
    ├── Member (N명, 조직 멤버)
    └── Project (N개)
        ├── Admin (배정된 Admin)
        ├── Member (배정된 Member)
        └── Client (배정된 Client, 프로젝트 단위 접근 제한)
            └── Screen (N개)
                └── ScreenVersion (N개)
                    └── Pin (N개)
                        └── Comment (N개)
                            └── Reply (N개)
```

- Owner는 조직 내 모든 프로젝트에 자동 접근
- Admin은 배정 프로젝트 전체를 관리
- Member는 배정 프로젝트의 화면·피드백 작업 수행
- Client는 명시적으로 배정된 프로젝트의 읽기+핀/코멘트 작성만 허용

### Design Reference

| Field | Value |
|-------|-------|
| 참고 앱 | 없음 `[💡 권장적용: Figma Comment, Marvel App 참고 권장]` |
| 색상/로고 | 미지정 `[💡 권장적용: 브랜드 가이드라인 수립 후 반영]` |
| 폰트 | 미지정 `[💡 권장적용: Inter (EN) / Pretendard (KO) 조합 권장]` |
| 반응형 | 반응형 웹 (데스크탑 우선, 태블릿/모바일 지원) |
| i18n | EN / KO |

### MVP Scope

**Included (MVP 포함):**
- 이메일/비밀번호 기반 회원가입 및 로그인 `[💡 권장적용: 소셜 로그인은 Phase 2]`
- 조직 생성 및 멀티테넌시 데이터 격리
- 프로젝트 CRUD 및 멤버 배정
- 화면(Screen) 등록/삭제 및 스크린샷 버전 업로드
- 버전 히스토리 조회 및 버전 비교 (overlay / side-by-side)
- 이미지 위 핀(좌표) 생성 및 코멘트 작성/답글
- 코멘트 상태 관리 (open / in_progress / resolved)
- 이메일 초대 및 역할 관리 (Owner / Admin / Member / Client)
- WebSocket 기반 실시간 핀·코멘트 업데이트
- 인앱 알림 및 이메일 알림 (코멘트, 상태 변경, 초대)
- 코멘트 파일 첨부
- Soft delete 및 Audit log
- i18n (EN / KO)
- Super Admin 패널 (조직 목록/상태 관리)

**Excluded — Deferred (MVP 이후):**
- Stripe 구독결제 연동 (Phase 2)
- 소셜 로그인 (Google, GitHub 등) (Phase 2)
- Figma 등 디자인 도구 직접 리뷰 연동 (Phase 3)
- 데이터 내보내기 (CSV / PDF) (Phase 2)
- 조직/플랜별 프로젝트·멤버 수 제한 정책 (Phase 2, Billing 연동 후)

---

## 1. Terminology

### Core Concepts

| Term | Definition |
|------|------------|
| Organization | 플랫폼의 최상위 멀티테넌시 단위. 하나의 조직은 독립된 데이터 격리 영역을 가지며, Owner가 소유한다. |
| Project | 조직 내에서 관리되는 디자인 작업 단위. 여러 Screen을 포함하며, 멤버별 접근 권한이 배정된다. |
| Screen | 프로젝트 내 개별 디자인 페이지/화면. 여러 버전(ScreenVersion)을 가질 수 있다. |
| ScreenVersion | 하나의 Screen에 업로드된 스크린샷 이미지 단위. 버전 히스토리 및 비교의 기준이 된다. |
| Pin | ScreenVersion 이미지 위에 찍힌 좌표(x_pct, y_pct) 기반 마커. 하나의 Pin에 Comment 스레드가 연결된다. |
| Comment | Pin에 연결된 텍스트 피드백. 상태(open / in_progress / resolved)를 가지며, 파일 첨부 가능. |
| Reply | Comment 하위의 답글. 스레드 형태로 코멘트에 종속된다. |
| Feedback Viewer | 좌측 이미지(핀 표시) + 우측 코멘트 패널로 구성된 핵심 리뷰 화면. 핀↔코멘트 상호 하이라이트 제공. |
| Version Comparison | 두 ScreenVersion을 overlay(투명도 겹치기) 또는 side-by-side(좌우 배치)로 비교하는 기능. |
| Invitation | 이메일로 조직 또는 프로젝트에 사용자를 초대하는 프로세스. 수락 시 계정 생성 및 멤버십 활성화. |
| Membership | 사용자가 특정 조직 또는 프로젝트에 속하는 관계 레코드. 역할(Role)을 포함한다. |
| Audit Log | 주요 액션(생성/수정/삭제/상태변경)의 수행자, 대상, 일시를 기록하는 이력 데이터. |
| Soft Delete | 레코드를 DB에서 실제 삭제하지 않고 deleted_at 타임스탬프를 기록하여 논리 삭제 처리하는 방식. |
| Multitenancy | 하나의 플랫폼 인스턴스에서 여러 조직의 데이터를 완전히 격리하여 제공하는 SaaS 아키텍처. |
| Slug | URL에 사용되는 사람이 읽기 쉬운 고유 식별자. 영문 소문자, 숫자, 하이픈으로 구성되며 조직·프로젝트의 URL 경로에 활용된다. (예: `my-design-team`) |

### User Roles

| Role | Description |
|------|-------------|
| SuperAdmin | 플랫폼 전체 운영자. 모든 조직 데이터에 접근하고 조직 상태를 제어한다. 별도 내부 계정으로 관리. `[💡 권장적용: /admin 경로 분리 + 별도 인증 미들웨어 적용]` |
| Owner | 조직 소유자. 빌링, 멤버 전체 관리, 조직 설정 권한을 가진다. 조직당 1명. |
| Admin | 조직 내 프로젝트 관리자. 프로젝트 생성·삭제, 팀원/클라이언트 초대, 피드백 전체 관리 권한. |
| Member | 실무 작업자. 화면 등록/버전 업로드, 코멘트 답변 및 상태 변경 권한. 프로젝트 삭제 불가. |
| Client | 외부 검토자. 배정된 프로젝트의 화면 열람 및 핀/코멘트 작성만 가능. 설정 접근 불가. |

### Status Values

| Enum | Values | Description |
|------|--------|-------------|
| `user_status` | `active(1)`, `invited(2)`, `suspended(3)`, `deleted(0)` | 사용자 계정 상태. `invited`는 초대 수락 전 임시 상태. `suspended`는 관리자 비활성화. `deleted`는 soft delete. |
| `user_role` | `superadmin(0)`, `owner(1)`, `admin(2)`, `member(3)`, `client(4)` | 조직/프로젝트 내 권한 레벨. 숫자가 낮을수록 상위 권한. |
| `comment_status` | `open`, `in_progress`, `resolved` | 코멘트(피드백) 처리 상태. `open`=신규, `in_progress`=처리 중, `resolved`=완료. |
| `invitation_status` | `pending`, `accepted`, `expired`, `cancelled` | 초대 링크 상태. `pending`=발송 후 대기, `accepted`=수락 완료, `expired`=만료(TTL 초과), `cancelled`=취소됨. |
| `organization_status` | `active`, `suspended`, `deleted` | 조직 상태. SuperAdmin이 제어. `suspended`는 해당 조직 전체 사용자 로그인 차단. |
| `project_status` | `active`, `archived`, `deleted` | 프로젝트 상태. `active`=정상 운영 중, `archived`=보관됨(읽기 전용), `deleted`=soft delete 처리. |
| `plan_tier` | `free`, `pro`, `enterprise` | 조직의 구독 요금제 등급. `free`=무료 플랜(기본), `pro`=유료 월정액, `enterprise`=대용량·전용 지원 플랜. Billing 모듈(Phase 2)에서 관리. |
| `notification_type` | `comment_created`, `comment_replied`, `comment_status_changed`, `invitation_received`, `version_uploaded`, `mention` | 인앱/이메일 알림 트리거 유형. `mention`=코멘트/답글에서 @멘션으로 특정 사용자를 지정했을 때 발생. |

### Technical Terms

| Term | Definition |
|------|------------|
| WebSocket | 서버-클라이언트 간 양방향 실시간 통신 프로토콜. 핀/코멘트 실시간 업데이트에 사용. |
| RBAC | Role-Based Access Control. 사용자 역할에 따라 API 및 UI 접근을 제어하는 권한 모델. |
| Soft Delete | `deleted_at` 컬럼에 타임스탬프를 기록하여 데이터를 논리 삭제. 목록 조회 시 `WHERE deleted_at IS NULL` 필터 적용. |
| Presigned URL | Cloud Storage(S3/GCS)에서 제한 시간 내 파일을 업로드/다운로드할 수 있는 서명된 임시 URL. |
| JWT | JSON Web Token. 인증 후 발급되는 액세스 토큰. 만료 시 Refresh Token으로 갱신. |
| Refresh Token | 액세스 토큰 만료 시 재발급에 사용하는 장기 유효 토큰. HttpOnly 쿠키로 저장 권장. `[📌 권장적용됨]` |
| Multitenancy Row-level Isolation | 모든 DB 쿼리에 `organization_id` 필터를 필수 적용하여 조직 간 데이터 혼용을 방지하는 패턴. |
| Overlay Comparison | 두 ScreenVersion 이미지를 동일 캔버스에 겹쳐 투명도로 차이를 시각화하는 비교 모드. |
| Side-by-side Comparison | 두 ScreenVersion 이미지를 좌우로 배치하여 동기화된 스크롤로 비교하는 모드. |
| Debounce | 검색 입력 등 연속 이벤트에서 마지막 이벤트 발생 후 일정 시간(300ms) 대기 후 실행하는 최적화 기법. |

---

## 2. System Modules

### Module 1 — Authentication

#### Main Features
- 이메일/비밀번호 회원가입 (초대 수락 경로 포함)
- 로그인 / 로그아웃
- JWT 액세스 토큰 + Refresh Token 기반 인증 `[📌 권장적용됨]`
- 비밀번호 재설정 (이메일 링크)
- 인증 가드: 미인증 → 로그인 리다이렉트, 인증됨 → 로그인 페이지 접근 시 대시보드로
- 세션 만료(401) 시 자동 토큰 갱신, 갱신 실패 시 로그인 리다이렉트

#### Technical Flow

**[회원가입 — 초대 수락 경로]**
1. 클라이언트가 초대 이메일의 링크 클릭 → `GET /api/invitations/{token}/verify` 로 토큰 유효성 확인
   - 실패(만료/무효): 토큰 만료 안내 페이지 표시, 재초대 요청 유도
2. 유효한 경우 회원가입 폼 표시 (이메일 필드 자동 채워짐, 수정 불가)
3. 사용자가 이름·비밀번호 입력 → 클라이언트 실시간 유효성 검사 `[버그예방: 폼 유효성 검사 미동작]`
   - 비밀번호 규칙: 최소 8자, 대소문자+숫자+특수문자 포함 `[📌 권장적용됨]`
4. Submit → `POST /api/auth/register` (body: name, password, invitation_token)
   - 성공: JWT 액세스 토큰 + Refresh Token 발급, 대시보드로 리다이렉트 (네비게이션 스택 초기화) `[버그예방: 인증 후 잘못된 화면으로 리다이렉트]`
   - 실패(중복 이메일/만료): 구체적 오류 메시지 표시, 버튼 재활성화

**[로그인]**
1. 이메일/비밀번호 입력 → 실시간 유효성 검사
2. 로그인 버튼 클릭 → `POST /api/auth/login` `[버그예방: 로그인/회원가입 동작 불가]`
   - 성공: 액세스 토큰 메모리 저장, Refresh Token HttpOnly 쿠키 저장, 대시보드로 이동
   - 실패: 에러 메시지 1회 표시 `[버그예방: 세션 만료/자격증명 오류 반복 노출]`
3. 보호된 라우트 접근 시 인증 상태 복원 완료 후 라우트 가드 평가 (초기 로딩 중 스피너) `[버그예방: 페이지 리로드 시 첫 페이지로 리다이렉트]`

**[토큰 갱신]**
1. API 요청 응답 401 수신 → 인터셉터가 `POST /api/auth/refresh` 자동 호출
   - 성공: 새 액세스 토큰으로 원래 요청 재시도
   - 실패: 로그인 페이지로 리다이렉트, 에러 1회 표시 `[버그예방: 세션 만료 반복 노출]`

**[로그아웃]**
1. 로그아웃 버튼 클릭 → `POST /api/auth/logout` (토큰 만료 여부 무관하게 성공 반환) `[버그예방: 로그아웃 API 동작 불가]`
2. 로컬 토큰 초기화, 로그인 페이지로 이동

---

### Module 2 — Organization Management

#### Main Features
- 조직 생성 (Owner 계정 생성과 동시 처리)
- 조직 정보 수정 (이름, 로고)
- 조직별 데이터 완전 격리 (모든 쿼리에 `organization_id` 필터 강제)
- 조직 상태 관리 (SuperAdmin: active / suspended / deleted)

#### Technical Flow

**[조직 생성]**
1. 신규 사용자가 조직 생성 폼 작성 (조직명 필수) → 실시간 유효성 검사 `[버그예방: 폼 유효성 검사 미동작]`
2. Submit → `POST /api/organizations`
   - 성공: Organization 레코드 생성, 요청자에게 `role=owner` Membership 생성, 대시보드로 이동
   - 실패(중복명 등): 구체적 오류 표시
3. 이후 모든 API 요청에 `X-Organization-Id` 헤더 또는 JWT claim의 `org_id` 포함 `[📌 권장적용됨]`

**[조직 정보 수정]**
1. Settings > Organization 화면에서 정보 수정 → `PATCH /api/organizations/{org_id}`
   - PATCH는 partial update 지원, null 필드 무시 `[버그예방: API 500 에러]`
   - 성공: 전역 조직 상태 업데이트
   - 실패: 에러 토스트 표시

**[SuperAdmin 조직 상태 변경]**
1. SuperAdmin이 `GET /api/superadmin/organizations` 목록 조회 (페이지네이션 지원) `[버그예방: 페이지네이션 미구현]`
2. 조직 선택 → `PATCH /api/superadmin/organizations/{org_id}/status` (body: status)
   - `suspended` 처리 시: 해당 조직 전체 사용자 로그인 시 403 반환
   - `deleted` 처리 시: soft delete (`deleted_at` 기록)
   - 모든 상태 변경은 Audit Log에 기록 → `POST /api/audit-logs` (내부 호출)

---

### Module 3 — Project Management

#### Main Features
- 프로젝트 CRUD (조직 내 격리)
- 프로젝트별 멤버(Admin/Member/Client) 배정 및 제거
- 프로젝트 목록 페이지네이션 및 검색
- Client는 배정된 프로젝트만 조회 가능

#### Technical Flow

**[프로젝트 생성]**
1. Admin/Owner가 프로젝트 생성 폼 작성 (이름, 설명) → `POST /api/projects`
   - 성공: Project 레코드 생성(organization_id 자동 설정), 프로젝트 상세로 이동
   - 실패: 오류 메시지 표시
2. Audit Log 기록

**[프로젝트 목록 조회]**
1. `GET /api/projects?page=1&page_size=20&q={검색어}` 호출
   - `organization_id` 필터 서버에서 강제 적용 (멀티테넌시 보안)
   - Client 역할: 자신이 배정된 프로젝트만 반환
   - 검색어: ILIKE 부분 일치 `[버그예방: 검색 기능 미동작]`
   - 기본 정렬: `created_at DESC` `[버그예방: 페이지네이션 미구현 및 정렬 오류]`
   - 0건 시 empty state 표시

**[멤버 배정]**
1. Admin이 프로젝트 상세 > Members 탭에서 초대 또는 기존 멤버 배정
2. 기존 조직 멤버 배정: `POST /api/projects/{project_id}/members` (body: user_id, role)
   - RBAC 검증: Admin 이상만 호출 가능 `[버그예방: 관리자 벌크/단일 액션 오류]`
   - 성공: 멤버십 레코드 생성
3. 신규 클라이언트 초대: Team & Invitation 모듈 호출

**[프로젝트 삭제]**
1. Owner/Admin이 `DELETE /api/projects/{project_id}` 호출
   - Soft delete 처리 (`deleted_at` 기록)
   - 연관된 Screen, ScreenVersion, Pin, Comment 캐스케이드 soft delete `[📌 권장적용됨]`
   - Audit Log 기록

---

### Module 4 — Screen Management

#### Main Features
- 화면(Screen) 등록/삭제 (프로젝트 내)
- 스크린샷 버전(ScreenVersion) 업로드 (Cloud Storage 연동)
- 버전 목록 조회 및 활성 버전 설정
- 파일 형식/크기 검증 (클라이언트 + 서버 양측)

#### Technical Flow

**[화면(Screen) 등록]**
1. Admin/Member가 프로젝트 > Screens 에서 화면 추가 버튼 클릭
2. 화면 이름 입력 → `POST /api/projects/{project_id}/screens`
   - 성공: Screen 레코드 생성, 목록 갱신
   - 실패: 오류 표시

**[스크린샷 버전 업로드]**
1. Admin/Member가 Screen Detail > Version Upload에서 파일 선택
2. 클라이언트 사전 검증: 파일 형식(PNG/JPG/WebP/GIF) `[📌 권장적용됨]`, 최대 크기(20MB) `[📌 권장적용됨]` `[버그예방: 파일 업로드 실패 및 요구사항 오류]`
   - 미충족 시 구체적 에러 메시지 표시, 업로드 진행하지 않음
3. Presigned URL 요청 → `POST /api/screens/{screen_id}/versions/upload-url` (body: filename, content_type)
   - 서버: MIME 타입 화이트리스트 + 최대 크기 검증 후 Presigned URL 반환
4. 클라이언트가 Presigned URL로 직접 S3/GCS 업로드 (PUT 요청)
   - 성공: 업로드 완료 확인 → `POST /api/screens/{screen_id}/versions` (body: storage_key, filename)
   - 실패: 에러 토스트 + 재시도 버튼 표시
5. ScreenVersion 레코드 생성 완료, 버전 목록에 추가, Audit Log 기록
6. 실시간 이벤트 emit: `version:uploaded` (Realtime 모듈 → WebSocket)
7. 알림 발송: `notification_type=version_uploaded` (Notification 모듈)

---

### Module 5 — Feedback System

#### Main Features
- 이미지 위 핀(좌표) 생성
- 핀에 코멘트 작성 및 파일 첨부
- 코멘트 답글(Reply) 작성
- 코멘트 상태 변경 (open → in_progress → resolved)
- Admin: 코멘트 삭제
- 핀↔코멘트 상호 하이라이트 (Feedback Viewer UI)

#### Technical Flow

**[핀 생성 + 코멘트 작성]**
1. 사용자가 Feedback Viewer에서 이미지 클릭 → 좌표(x_pct%, y_pct%) 계산 (이미지 너비/높이 대비 비율로 저장) `[📌 권장적용됨]`
2. 핀 생성 → `POST /api/versions/{version_id}/pins` (body: x_pct, y_pct)
   - 성공: Pin 레코드 생성, 임시 핀 마커 이미지에 표시
3. 코멘트 입력 폼 표시 (파일 첨부 가능)
4. 코멘트 Submit → `POST /api/pins/{pin_id}/comments` (body: content, attachments[])
   - 첨부파일 있을 시 File Attachment 모듈 호출 (Presigned URL 업로드)
   - 성공: Comment 레코드 생성 (initial status: `open`), 코멘트 패널에 추가
   - 실패: 에러 토스트 + 핀 임시 마커 제거
5. WebSocket 이벤트 emit: `pin:created`, `comment:created`
6. 알림 발송: `notification_type=comment_created` (프로젝트 멤버 대상)

**[코멘트 상태 변경]**
1. Member/Admin이 코멘트 상태 버튼 클릭 → `PATCH /api/comments/{comment_id}/status` (body: status)
   - RBAC 검증: Client는 상태 변경 불가
   - 허용 전환: `open → in_progress`, `in_progress → resolved`, `resolved → open` `[📌 권장적용됨]`
   - 성공: 코멘트 상태 업데이트, 코멘트 패널 갱신
   - 실패: 에러 토스트
2. WebSocket 이벤트 emit: `comment:status_changed`
3. 알림 발송: `notification_type=comment_status_changed`
4. Audit Log 기록

**[코멘트 삭제 — Admin]**
1. Admin이 코멘트 삭제 → `DELETE /api/comments/{comment_id}`
   - RBAC 검증: Admin 이상만 가능 `[버그예방: 관리자 벌크/단일 액션 오류]`
   - Soft delete 처리
   - 연관 Reply, 첨부파일 참조 캐스케이드 soft delete `[📌 권장적용됨]`
2. WebSocket 이벤트 emit: `comment:deleted`
3. Audit Log 기록

---

### Module 6 — Version Control

#### Main Features
- 버전 히스토리 목록 조회
- 활성 버전 변경
- 버전 비교: overlay 모드 (투명도 조절)
- 버전 비교: side-by-side 모드 (동기화 스크롤)

#### Technical Flow

**[버전 히스토리 조회]**
1. Feedback Viewer 진입 시 → `GET /api/screens/{screen_id}/versions?page=1&page_size=50`
   - 기본 정렬: `created_at DESC`
   - 응답: ScreenVersion 목록 (id, filename, created_at, uploaded_by)

**[버전 비교]**
1. 사용자가 버전 선택기에서 2개 버전 선택 (Base / Compare)
2. 비교 모드 선택 (overlay / side-by-side) → UI 전환
3. 이미지 URL 요청 → `GET /api/versions/{version_id}/image-url` (Presigned URL 반환)
   - 두 버전 이미지 URL 병렬 요청
4. Overlay 모드: 두 이미지를 동일 캔버스에 렌더링, 투명도 슬라이더 제공 `[💡 권장적용: 기본 50% 불투명도]`
5. Side-by-side 모드: 좌우 패널에 각 이미지 렌더링, 스크롤/줌 동기화 `[📌 권장적용됨]`
6. 비교 뷰에서 핀은 비활성화 (읽기 전용) `[📌 권장적용됨]`
   - 실패(이미지 로드 오류): 오류 메시지 + 재시도 버튼

---

### Module 7 — Team & Invitation

#### Main Features
- 이메일 초대 발송 (조직 멤버 초대 / 프로젝트 클라이언트 초대)
- 초대 링크 유효성 관리 (TTL: 7일) `[📌 권장적용됨]`
- 역할 변경 (Owner가 Admin/Member 역할 조정)
- 멤버 제거

#### Technical Flow

**[이메일 초대 발송]**
1. Owner/Admin이 Settings > Team 또는 Project > Members에서 이메일 입력
2. Submit → `POST /api/organizations/{org_id}/invitations` 또는 `POST /api/projects/{project_id}/invitations`
   - body: email, role
   - 서버: 기존 멤버 중복 확인, invitation_token 생성 (UUID), `invitation_status=pending` 저장
   - 실패(이미 멤버): "이미 등록된 이메일입니다" 에러 반환
3. Email Service(SendGrid/SES)로 초대 이메일 발송 (링크: `{base_url}/invitations/{token}`)
4. 성공: 초대 목록에 `pending` 상태로 표시

**[초대 수락]**
1. 수신자가 이메일 링크 클릭 → `GET /api/invitations/{token}/verify`
   - 만료(7일 초과): `invitation_status=expired` 업데이트, 만료 안내 페이지
   - 취소됨: 취소 안내 페이지
2. 신규 사용자: Authentication 모듈 회원가입 플로우로 이동
3. 기존 사용자(로그인 상태): `POST /api/invitations/{token}/accept` → Membership 생성
4. `invitation_status=accepted` 업데이트
5. 알림 발송: 초대자에게 수락 완료 알림

**[멤버 역할 변경 / 제거]**
1. Owner가 Settings > Team에서 역할 변경 → `PATCH /api/organizations/{org_id}/members/{user_id}` (body: role)
   - RBAC: Owner만 가능
   - 자기 자신 역할 변경 불가 `[📌 권장적용됨]`
2. 멤버 제거 → `DELETE /api/organizations/{org_id}/members/{user_id}`
   - Membership soft delete, 해당 사용자의 진행 중 세션은 다음 토큰 갱신 시 403 처리 `[📌 권장적용됨]`
3. Audit Log 기록

---

### Module 8 — Notification

#### Main Features
- 인앱 알림 (코멘트 생성, 답글, 상태 변경, 초대 수신, 버전 업로드)
- 이메일 알림 (동일 트리거, 사용자 설정으로 on/off 가능) `[📌 권장적용됨]`
- 알림 읽음 처리 (단건 / 전체)

#### Technical Flow

**[알림 생성 및 발송]**
1. 트리거 이벤트 발생 (코멘트 생성, 상태 변경 등) → 해당 모듈 내부에서 Notification 서비스 호출
2. 알림 수신 대상자 결정 (프로젝트 멤버 전원 또는 핀 작성자 등) `[💡 권장적용: 자기 자신 제외]`
3. Notification 레코드 생성 → `POST /api/notifications` (내부 호출, body: user_id, type, payload)
4. 이메일 발송: Email Service(SendGrid/SES) API 호출
   - 발송 실패 시: 재시도 큐 등록 (최대 3회) `[📌 권장적용됨]`
5. WebSocket 이벤트 emit: `notification:new` → 해당 사용자 채널 `[버그예방: 실시간 메시지/알림 미수신]`

**[알림 목록 조회]**
1. `GET /api/notifications?page=1&page_size=20&unread_only=false`
   - 기본 정렬: `created_at DESC`
   - 페이지네이션 지원 `[버그예방: 페이지네이션 미구현]`

**[알림 읽음 처리]**
1. 단건: `PATCH /api/notifications/{notification_id}/read`
2. 전체 읽음: `PATCH /api/notifications/read-all`
3. 성공: 인앱 알림 뱃지 카운트 업데이트

---

### Module 9 — Realtime

#### Main Features
- WebSocket 기반 실시간 핀/코멘트/상태 업데이트
- 실시간 알림 push
- 앱 레벨 전역 WebSocket 연결 관리 (포그라운드 복귀 시 자동 재연결)
- 소켓 연결 해제 구간의 놓친 이벤트 재조회

#### Technical Flow

**[WebSocket 연결 초기화]**
1. 사용자 로그인 성공 후 앱 레벨에서 WebSocket 연결 수립 → `WS /ws?token={access_token}`
   - 인증 실패(토큰 만료): 연결 거부, 토큰 갱신 후 재시도
2. 연결 성공: 사용자 개인 채널(`user:{user_id}`) + 현재 진입 프로젝트 채널(`project:{project_id}`) 구독

**[포그라운드 복귀 / 재연결]**
1. 앱 포그라운드 복귀 감지 (visibilitychange 이벤트) → 소켓 연결 상태 확인 `[버그예방: 실시간 메시지/알림 미수신]`
2. 연결 끊김 감지 시: 자동 재연결 (Exponential Backoff) `[📌 권장적용됨]`
3. 재연결 성공 시: 연결 해제 시점 이후 놓친 이벤트 재조회 → `GET /api/events/missed?since={timestamp}`

##### WebSocket Events

| Channel | Event | Payload | Direction |
|---------|-------|---------|-----------|
| `project:{project_id}` | `pin:created` | `{ pin_id, version_id, x_pct, y_pct, created_by }` | Server → Client |
| `project:{project_id}` | `pin:deleted` | `{ pin_id }` | Server → Client |
| `project:{project_id}` | `comment:created` | `{ comment_id, pin_id, content, created_by, status }` | Server → Client |
| `project:{project_id}` | `comment:updated` | `{ comment_id, content, updated_by }` | Server → Client |
| `project:{project_id}` | `comment:deleted` | `{ comment_id, pin_id }` | Server → Client |
| `project:{project_id}` | `comment:status_changed` | `{ comment_id, old_status, new_status, changed_by }` | Server → Client |
| `project:{project_id}` | `reply:created` | `{ reply_id, comment_id, content, created_by }` | Server → Client |
| `project:{project_id}` | `version:uploaded` | `{ screen_id, version_id, filename, uploaded_by }` | Server → Client |
| `user:{user_id}` | `notification:new` | `{ notification_id, type, payload, created_at }` | Server → Client |

---

### Module 10 — File Attachment

#### Main Features
- 코멘트 파일 첨부 (이미지, 문서 등)
- 파일 형식 및 크기 검증 (클라이언트 + 서버)
- Cloud Storage(S3/GCS)에 저장, Presigned URL로 다운로드

#### Technical Flow

**[파일 첨부 업로드]**
1. 코멘트 작성 폼에서 파일 선택
2. 클라이언트 사전 검증: 허용 형식(PNG/JPG/PDF/ZIP/...)`[📌 권장적용됨]`, 최대 10MB `[📌 권장적용됨]` `[버그예방: 파일 업로드 실패 및 요구사항 오류]`
   - 미충족 시 구체적 에러 표시, 업로드 진행하지 않음
3. Presigned URL 요청 → `POST /api/attachments/upload-url` (body: filename, content_type, size)
   - 서버: MIME 타입 화이트리스트 + 최대 크기 검증
4. Presigned URL로 S3/GCS 직접 업로드
   - 성공: `POST /api/attachments` (body: storage_key, filename, size, comment_id)
   - 실패: 에러 토스트 + 재시도 버튼

**[파일 다운로드]**
1. 첨부파일 클릭 → `GET /api/attachments/{attachment_id}/download-url`
   - 서버: 요청자의 프로젝트 접근 권한 검증 후 Presigned URL 반환 (유효시간: 15분) `[📌 권장적용됨]`
2. 클라이언트: 반환된 URL로 직접 다운로드

---

### Module 11 — Audit Log

#### Main Features
- 주요 액션 기록 (생성/수정/삭제/상태변경)
- 행위자(user_id), 대상(resource_type, resource_id), 액션(action), 일시(timestamp) 저장
- 보존 기간: 1년 `[📌 권장적용됨]`
- Owner/Admin/SuperAdmin 조회 가능

#### Technical Flow

**[Audit Log 기록]**
1. 주요 API 처리 완료 후 비동기로 Audit Log 서비스 호출 `[💡 권장적용: 동기 처리 시 응답 지연 방지]`
2. `POST /api/audit-logs` (내부 호출, body: user_id, action, resource_type, resource_id, metadata)
   - `action` 예시: `project.created`, `comment.status_changed`, `member.removed`
   - `metadata`: 변경 전/후 값(before/after) JSON 포함 `[📌 권장적용됨]`
   - 실패 시: 주요 API 응답에 영향 없이 에러 로깅만 처리

**[Audit Log 조회]**
1. `GET /api/audit-logs?resource_type=project&resource_id={id}&page=1&page_size=20`
   - RBAC: Owner/Admin/SuperAdmin만 호출 가능
   - 기본 정렬: `created_at DESC`
   - 페이지네이션 지원 `[버그예방: 페이지네이션 미구현]`

---

### Module 12 — Billing (추후)

#### Main Features
- Stripe 구독결제 연동 (Phase 2)
- 요금제(Plan) 선택 및 변경
- 결제 수단 등록/변경
- 청구서 조회

#### Technical Flow

> **이 모듈은 MVP 범위 외(Phase 2). 플로우는 Stripe 연동 시점에 상세 작성 예정.** `[💡 권장적용: Webhook 기반 이벤트 처리, checkout.session.completed / invoice.payment_failed 등 핵심 이벤트 사전 정의]`

**[구독 생성 — 개요]**
1. Owner가 Settings > Billing에서 요금제 선택 → `POST /api/billing/checkout` → Stripe Checkout Session 생성
2. Stripe Checkout 완료 → `POST /api/webhooks/stripe` (Stripe Webhook)
   - `checkout.session.completed`: 구독 활성화, Organization 플랜 업데이트
   - `invoice.payment_failed`: Owner에게 결제 실패 이메일 알림
3. 구독 취소 → `POST /api/billing/cancel` → Stripe 구독 취소, 기간 만료 후 무료 플랜으로 다운그레이드

---

### Module 13 — Super Admin

#### Main Features
- 전체 조직 목록 조회 및 검색
- 조직 상태 관리 (active / suspended / deleted)
- 별도 경로(`/admin`) 및 추가 인증 미들웨어 `[📌 권장적용됨]`

#### Technical Flow

**[전체 조직 조회]**
1. SuperAdmin이 `/admin/organizations` 접근 → RBAC 미들웨어: `role=superadmin` 검증
2. `GET /api/superadmin/organizations?page=1&page_size=20&q={검색어}&status={filter}`
   - 검색: 조직명 ILIKE 부분 일치 `[버그예방: 검색 기능 미동작]`
   - 필터: 상태별 (`active/suspended/deleted/all`) `[버그예방: 어드민 필터/통계 오류]`
   - '전체' 필터 시 전체 데이터 반환, 페이지네이션 지원 `[버그예방: 페이지네이션 미구현]`
3. 응답: 조직 목록 (id, name, status, member_count, created_at, owner_email)

**[조직 상태 변경]**
1. SuperAdmin이 조직 선택 → 상태 변경 → `PATCH /api/superadmin/organizations/{org_id}/status`
   - body: `{ status: "suspended" | "active" | "deleted" }`
   - `deleted` 처리: soft delete
2. Audit Log 기록 (SuperAdmin 행위자로)

---

### 3rd Party API List

| Service | Purpose | Integration Point | Alternative |
|---------|---------|-------------------|-------------|
| Stripe | 구독결제 처리 (Phase 2). Checkout Session, Webhook 수신, 구독 관리 | Billing 모듈 (`POST /api/billing/checkout`, `POST /api/webhooks/stripe`) | Paddle, LemonSqueezy |
| SendGrid | 이메일 초대 및 알림 발송 (트랜잭셔널 이메일) | Team & Invitation 모듈, Notification 모듈 | AWS SES, Mailgun, Resend |
| AWS S3 / GCS | 스크린샷 이미지 및 코멘트 첨부파일 저장. Presigned URL 기반 직접 업로드/다운로드 | Screen Management 모듈 (버전 업로드), File Attachment 모듈 | Cloudflare R2, Supabase Storage |
| Redis | 세션/토큰 캐싱, Refresh Token 저장소, 실시간 이벤트 Pub/Sub 브로커, 알림 재시도 큐 관리 | Authentication 모듈 (Refresh Token), Realtime 모듈 (Pub/Sub), Notification 모듈 (재시도 큐) | Upstash Redis, Valkey |

---

## 3. User Application

### 3.1 Page Architecture

**Stack:** Next.js 14 (App Router), React 18, Zustand (전역 상태), TanStack Query (서버 상태), Tailwind CSS + shadcn/ui [📌 권장적용됨]

#### Route Groups
| Group | Access |
|-------|--------|
| Auth | 미인증 사용자만 접근. 인증된 사용자 접근 시 `/dashboard`로 리다이렉트 |
| Protected | 인증된 사용자만 접근. 미인증 시 `/auth/login`으로 리다이렉트 (원래 URL 보존) |
| Public | 없음 (모든 페이지가 Auth 또는 Protected) |

#### Page Map

**Auth**
| Route | Page | Access |
|-------|------|--------|
| `/auth/login` | 로그인 | All (미인증) |
| `/auth/register` | 회원가입 | All (미인증) |
| `/auth/forgot-password` | 비밀번호 찾기 | All (미인증) |
| `/auth/reset-password` | 비밀번호 재설정 | All (미인증) |
| `/auth/accept-invite` | 초대 수락 | All (미인증) |

**Protected**
| Route | Page | Access |
|-------|------|--------|
| `/dashboard` | 대시보드 | Owner, Admin, Member |
| `/projects` | 프로젝트 목록 | Owner, Admin, Member |
| `/projects/:id` | 프로젝트 상세 | Owner, Admin, Member |
| `/projects/:id/screens` | 화면 목록 | Owner, Admin, Member |
| `/projects/:id/screens/:screenId` | 피드백 뷰어 | Owner, Admin, Member |
| `/projects/:id/members` | 프로젝트 멤버 관리 | Owner, Admin |
| `/settings` | 설정 메인 | Owner, Admin, Member |
| `/settings/organization` | 조직 설정 | Owner |
| `/settings/team` | 팀 관리 | Owner, Admin |
| `/settings/billing` | 빌링/결제 | Owner |
| `/settings/profile` | 프로필 설정 | All authenticated |
| `/client/projects` | 클라이언트 프로젝트 목록 | Client |
| `/client/projects/:id/screens` | 클라이언트 화면 목록 | Client |
| `/client/projects/:id/screens/:screenId` | 클라이언트 피드백 뷰어 | Client |
| `/notifications` | 알림 목록 | All authenticated |

---

### 3.2 Feature List by Route

---

#### `/auth/login` — 로그인
**Access:** All (미인증)

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 로그인 동작 불가 (5회) | 로그인 버튼은 유효성 검사와 무관하게 클릭 가능, 검사 실패 시 구체적 오류 표시 |
> | 인증 후 잘못된 리다이렉트 (4회) | 인증 가드 적용. 인증 완료 → 대시보드(또는 returnUrl)로 이동. 이미 인증된 사용자 → `/dashboard` 리다이렉트 |
> | 세션 만료 반복 노출 (5회) | 세션 만료(401) 시 자동 토큰 갱신, 갱신 실패 시 로그인 리다이렉트, 에러 1회만 표시 |

1) **LoginForm**
   - **컴포넌트 상세**: 이메일 입력필드, 비밀번호 입력필드 (토글 표시), "로그인" 버튼, "비밀번호 찾기" 링크, "회원가입" 링크, 소셜 로그인 버튼 (Google) [📌 권장적용됨]
     - Default: 입력필드 비어있음, 버튼 활성
     - Active: 입력필드 포커스 시 border 색상 변경
     - Disabled: API 호출 중 버튼에 spinner 표시, 중복 제출 방지
     - Error: 필드 하단 빨간 텍스트, 필드 border 빨간색
   - **데이터 표시**: 없음
   - **인터랙션**: Enter 키로 폼 제출, Tab으로 필드 이동, 비밀번호 표시/숨김 토글 클릭
   - **네비게이션**: "회원가입" → `/auth/register`, "비밀번호 찾기" → `/auth/forgot-password`, 로그인 성공 → `/dashboard` (또는 returnUrl)

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| email | email | Y | -/254 | 이메일 형식 | 올바른 이메일 주소를 입력해 주세요 |
| password | password | Y | 8/128 | - | 비밀번호를 입력해 주세요 |

**상태 화면**
- Loading: 로그인 버튼 내 spinner, 입력필드 disabled
- Empty: 해당 없음
- Error: "이메일 또는 비밀번호가 올바르지 않습니다" 인라인 에러 배너

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" 토스트
- 권한 에러: 해당 없음 (미인증 페이지)

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 로그인 5회 연속 실패 시 30초 쿨다운 안내 [📌 권장적용됨]
- 동시접근: 다른 탭에서 이미 로그인된 경우 `/dashboard`로 리다이렉트
- 오프라인: 오프라인 배너 표시, 로그인 버튼 비활성

---

#### `/auth/register` — 회원가입
**Access:** All (미인증)

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 폼 유효성 검사 미동작 (7회) | 모든 입력 필드 실시간 유효성 검사. Submit 버튼은 모든 필수 필드 유효 시에만 활성화 |
> | 비밀번호 유효성 검사 미흡 (3회) | 비밀번호 실시간 유효성 검사, 규칙 미충족 시 구체적 안내 |
> | 인증 후 잘못된 리다이렉트 (4회) | 가입 완료 후 네비게이션 스택 초기화, `/dashboard`로 이동 |

1) **RegisterForm**
   - **컴포넌트 상세**: 이름 입력필드, 이메일 입력필드, 비밀번호 입력필드 (강도 인디케이터), 비밀번호 확인 입력필드, 조직명 입력필드, "가입하기" 버튼, "로그인" 링크, 이용약관 동의 체크박스
     - Default: 입력필드 비어있음, 가입 버튼 비활성
     - Active: 포커스 필드 하이라이트, 비밀번호 강도 실시간 표시
     - Disabled: API 호출 중 버튼 spinner, 중복 제출 방지
     - Error: 각 필드 하단 개별 에러 메시지
   - **데이터 표시**: 비밀번호 강도 인디케이터 (약함/보통/강함)
   - **인터랙션**: Enter 키로 폼 제출, Tab 필드 이동, 비밀번호 표시/숨김 토글
   - **네비게이션**: "로그인" → `/auth/login`, 가입 성공 → `/dashboard`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| name | text | Y | 2/50 | 한글/영문/숫자 | 이름은 2~50자로 입력해 주세요 |
| email | email | Y | -/254 | 이메일 형식 | 올바른 이메일 주소를 입력해 주세요 |
| password | password | Y | 8/128 | 영문+숫자+특수문자 조합 [📌 권장적용됨] | 비밀번호는 8자 이상, 영문/숫자/특수문자를 포함해야 합니다 |
| passwordConfirm | password | Y | 8/128 | password와 일치 | 비밀번호가 일치하지 않습니다 |
| organizationName | text | Y | 2/100 | - | 조직명은 2~100자로 입력해 주세요 |
| termsAgreed | checkbox | Y | - | true | 이용약관에 동의해 주세요 |

**상태 화면**
- Loading: 가입 버튼 내 spinner, 전체 폼 disabled
- Empty: 해당 없음
- Error: 이메일 중복 시 "이미 사용 중인 이메일입니다" 인라인 에러

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" 토스트
- 권한 에러: 해당 없음

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 다른 탭에서 이미 로그인된 경우 `/dashboard`로 리다이렉트
- 오프라인: 오프라인 배너 표시, 가입 버튼 비활성

---

#### `/auth/forgot-password` — 비밀번호 찾기
**Access:** All (미인증)

1) **ForgotPasswordForm**
   - **컴포넌트 상세**: 이메일 입력필드, "비밀번호 재설정 링크 전송" 버튼, "로그인으로 돌아가기" 링크, 전송 완료 메시지 영역
     - Default: 이메일 필드 비어있음, 버튼 활성
     - Active: 포커스 하이라이트
     - Disabled: API 호출 중 버튼 spinner
     - Error: 이메일 형식 오류 시 하단 에러 메시지
   - **데이터 표시**: 전송 성공 시 "비밀번호 재설정 링크가 이메일로 전송되었습니다" 메시지
   - **인터랙션**: Enter 키로 폼 제출
   - **네비게이션**: "로그인으로 돌아가기" → `/auth/login`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| email | email | Y | -/254 | 이메일 형식 | 올바른 이메일 주소를 입력해 주세요 |

**상태 화면**
- Loading: 버튼 내 spinner
- Empty: 해당 없음
- Error: "등록되지 않은 이메일입니다" 인라인 에러 (보안상 동일 메시지로 통일 권장 [📌 권장적용됨]: "입력하신 이메일로 재설정 링크를 전송했습니다")

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" 토스트
- 권한 에러: 해당 없음

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 이메일 전송 1분 쿨다운 (재전송 방지) [📌 권장적용됨]
- 동시접근: 해당 없음
- 오프라인: 오프라인 배너 표시, 전송 버튼 비활성

---

#### `/auth/reset-password` — 비밀번호 재설정
**Access:** All (미인증, 유효 토큰 필요)

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 비밀번호 유효성 검사 미흡 (3회) | 비밀번호 실시간 유효성 검사, 규칙 미충족 시 구체적 안내 |

1) **ResetPasswordForm**
   - **컴포넌트 상세**: 새 비밀번호 입력필드 (강도 인디케이터), 비밀번호 확인 입력필드, "비밀번호 변경" 버튼
     - Default: 입력필드 비어있음, 버튼 비활성
     - Active: 포커스 하이라이트, 비밀번호 강도 실시간 표시
     - Disabled: API 호출 중 버튼 spinner
     - Error: 필드 하단 개별 에러 메시지
   - **데이터 표시**: 비밀번호 강도 인디케이터
   - **인터랙션**: Enter 키로 폼 제출, 비밀번호 표시/숨김 토글
   - **네비게이션**: 변경 성공 → `/auth/login` (성공 메시지 포함)

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| newPassword | password | Y | 8/128 | 영문+숫자+특수문자 조합 [📌 권장적용됨] | 비밀번호는 8자 이상, 영문/숫자/특수문자를 포함해야 합니다 |
| confirmPassword | password | Y | 8/128 | newPassword와 일치 | 비밀번호가 일치하지 않습니다 |

**상태 화면**
- Loading: 버튼 내 spinner
- Empty: 해당 없음
- Error: 토큰 만료/무효 시 "링크가 만료되었습니다. 비밀번호 찾기를 다시 진행해 주세요" + `/auth/forgot-password` 링크

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" 토스트
- 권한 에러: 토큰 만료/무효 → 에러 페이지 + 비밀번호 찾기 링크

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 같은 토큰으로 두 탭에서 동시 변경 시 먼저 성공한 쪽만 유효, 나머지는 토큰 무효 에러
- 오프라인: 오프라인 배너 표시, 변경 버튼 비활성

---

#### `/auth/accept-invite` — 초대 수락
**Access:** All (미인증, 유효 초대 토큰 필요)

1) **AcceptInviteForm**
   - **컴포넌트 상세**: 초대 정보 표시 (조직명, 초대자명, 역할), 기존 회원: "수락" 버튼, 신규 회원: 이름 + 비밀번호 설정 폼 + "가입 및 수락" 버튼
     - Default: 초대 정보 로딩 후 표시
     - Active: 폼 입력 시 필드 하이라이트
     - Disabled: API 호출 중 버튼 spinner
     - Error: 토큰 무효/만료 시 에러 화면
   - **데이터 표시**: 초대 조직명, 초대자명, 배정 역할
   - **인터랙션**: 수락 버튼 클릭, Enter 키로 폼 제출
   - **네비게이션**: 수락 성공(기존 회원) → `/dashboard`, 가입+수락 성공(신규) → `/dashboard`, 토큰 만료 → 에러 화면 (관리자에게 재초대 요청 안내)

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| name | text | Y (신규만) | 2/50 | 한글/영문/숫자 | 이름은 2~50자로 입력해 주세요 |
| password | password | Y (신규만) | 8/128 | 영문+숫자+특수문자 조합 [📌 권장적용됨] | 비밀번호는 8자 이상, 영문/숫자/특수문자를 포함해야 합니다 |
| passwordConfirm | password | Y (신규만) | 8/128 | password와 일치 | 비밀번호가 일치하지 않습니다 |

**상태 화면**
- Loading: 초대 정보 로딩 시 skeleton (조직명, 역할 영역)
- Empty: 해당 없음
- Error: "초대 링크가 만료되었거나 유효하지 않습니다. 관리자에게 재초대를 요청해 주세요"

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" 토스트
- 권한 에러: 토큰 만료/무효 → 에러 화면

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 이미 수락된 초대 링크 재접근 시 "이미 수락된 초대입니다" 메시지 + 로그인 유도
- 오프라인: 오프라인 배너 표시, 수락 버튼 비활성

---

#### `/dashboard` — 대시보드
**Access:** Owner, Admin, Member

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 인증 후 잘못된 리다이렉트 (4회) | 모든 라우트에 인증 가드 적용. 미인증 → 로그인. 인증됨+Auth페이지 접근 → 대시보드 |
> | 페이지 리로드 시 첫 페이지로 리다이렉트 (1회) | 보호된 라우트 평가는 인증 상태 복원 완료 후 실행. 초기 로딩 중 스피너 표시 |
> | UI 컴포넌트 불일치 (8회) | 공통 UI는 디자인 시스템 컴포넌트 재사용. placeholder 필수 |

1) **DashboardSummary**
   - **컴포넌트 상세**: 요약 카드 (활성 프로젝트 수, 미해결 피드백 수, 오늘 새 코멘트 수, 멤버 수), 최근 활동 리스트
     - Default: 요약 카드 숫자 표시
     - Active: 카드 호버 시 그림자 효과
     - Disabled: 해당 없음
     - Error: 카드 영역에 "데이터를 불러올 수 없습니다" + 재시도 버튼
   - **데이터 표시**: 활성 프로젝트 수, 미해결 피드백 수(open + in_progress), 오늘 새 코멘트 수, 멤버 수
   - **인터랙션**: 요약 카드 클릭 → 해당 목록 페이지 이동, 최근 활동 항목 클릭 → 해당 피드백 뷰어
   - **네비게이션**: 프로젝트 카드 → `/projects`, 피드백 카드 → 관련 피드백 뷰어

2) **RecentActivityFeed**
   - **컴포넌트 상세**: 활동 목록 (타입 아이콘, 사용자 아바타, 활동 설명, 시간), 최대 20건 표시
     - Default: 시간순 내림차순 목록
     - Active: 항목 호버 시 배경색 변경
     - Disabled: 해당 없음
     - Error: "최근 활동을 불러올 수 없습니다" + 재시도 버튼
   - **데이터 표시**: 활동 타입 (코멘트, 상태변경, 멤버추가 등), 사용자명, 대상 프로젝트/화면명, 상대 시간
   - **인터랙션**: 항목 클릭 → 해당 리소스 페이지 이동
   - **네비게이션**: 각 활동 항목 → 해당 프로젝트/화면/피드백 뷰어

**입력필드 검증규칙**
해당 없음

**상태 화면**
- Loading: 요약 카드 skeleton (4개), 활동 피드 skeleton (5줄)
- Empty: "아직 프로젝트가 없습니다. 첫 번째 프로젝트를 만들어 보세요!" + "프로젝트 만들기" CTA 버튼
- Error: 각 영역 개별 에러 표시 + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트 + 마지막 캐시 데이터 표시
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트 + 재시도 버튼
- 권한 에러: 조직 미소속 → "조직에 소속되어 있지 않습니다" 안내 + 초대 요청 유도

**엣지 케이스**
- 0건: 프로젝트 0건 시 온보딩 CTA 표시
- 최대치: 활동 피드 최대 20건, "더보기"로 페이지네이션
- 동시접근: 실시간 WebSocket으로 활동 피드 자동 갱신
- 오프라인: 오프라인 배너, 마지막 캐시 데이터 표시 (있는 경우)

---

#### `/projects` — 프로젝트 목록
**Access:** Owner, Admin, Member

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 페이지네이션 미구현 및 정렬 오류 (6회) | 목록 API는 page/page_size 파라미터 지원, 기본 정렬 created_at DESC. 10개+ 시 페이지네이션 |
> | 검색 기능 미동작 (6회) | 검색 입력에 300ms 디바운스, 0건 시 empty state. 서버 ILIKE 부분 일치 |
> | 필터 적용 후 자동 초기화 (5회) | 필터 조건은 전역 상태/URL 쿼리 파라미터 보관, 초기화 버튼 별도 제공 |
> | 뒤로가기 시 상태 초기화 (2회) | 페이지 번호/검색어/필터를 URL query parameter에 저장 |

1) **ProjectListHeader**
   - **컴포넌트 상세**: 페이지 타이틀 "프로젝트", 검색 입력필드, 정렬 드롭다운 (최신순/이름순/수정일순), "새 프로젝트" 버튼 (Owner, Admin만)
     - Default: 검색필드 비어있음, 정렬 "최신순" 선택
     - Active: 검색필드 포커스 시 하이라이트
     - Disabled: "새 프로젝트" 버튼은 Member에게 표시되지 않음
     - Error: 해당 없음
   - **데이터 표시**: 총 프로젝트 수
   - **인터랙션**: 검색어 입력 (300ms 디바운스), 정렬 변경, "새 프로젝트" 클릭 → CreateProjectModal 열기
   - **네비게이션**: 해당 없음

2) **ProjectCard (목록)**
   - **컴포넌트 상세**: 프로젝트 썸네일 (첫 화면 스크린샷 또는 placeholder), 프로젝트명, 화면 수, 미해결 피드백 수, 마지막 수정일, 멤버 아바타 (최대 5명 + "+N")
     - Default: 카드 형태 그리드 배열
     - Active: 호버 시 그림자 + 살짝 올라가는 효과
     - Disabled: 해당 없음
     - Error: 썸네일 로드 실패 시 placeholder 이미지
   - **데이터 표시**: 프로젝트명, 화면 수, 미해결 피드백 수, 마지막 수정일 (상대 시간), 멤버 아바타
   - **인터랙션**: 카드 클릭 → 프로젝트 상세, 카드 우클릭 또는 더보기(...) → 컨텍스트 메뉴 (수정/삭제 — Owner, Admin만)
   - **네비게이션**: 카드 클릭 → `/projects/:id`

3) **CreateProjectModal**
   - **컴포넌트 상세**: 모달 (프로젝트명 입력, 설명 입력(선택), "생성" 버튼, "취소" 버튼)
     - Default: 빈 폼
     - Active: 입력 필드 포커스
     - Disabled: API 호출 중 버튼 spinner
     - Error: 필드 하단 에러 메시지
   - **데이터 표시**: 해당 없음
   - **인터랙션**: ESC 키로 모달 닫기, Enter로 생성, 배경 클릭으로 모달 닫기
   - **네비게이션**: 생성 성공 → `/projects/:newId`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| search | text | N | -/100 | - | - |
| projectName (모달) | text | Y | 2/100 | - | 프로젝트명은 2~100자로 입력해 주세요 |
| projectDescription (모달) | textarea | N | -/500 | - | 설명은 500자 이내로 입력해 주세요 |

**상태 화면**
- Loading: 프로젝트 카드 skeleton (6개 그리드)
- Empty: "아직 프로젝트가 없습니다" 일러스트 + "새 프로젝트 만들기" CTA (Owner, Admin), Member는 "관리자에게 프로젝트 생성을 요청해 주세요"
- Error: "프로젝트 목록을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트 + 재시도 버튼
- 권한 에러: 조직 미소속 → `/dashboard`로 리다이렉트

**엣지 케이스**
- 0건: empty state + CTA
- 최대치: 프로젝트 수 제한 없음 (페이지당 12개, 페이지네이션) [📌 권장적용됨]
- 동시접근: 다른 사용자가 프로젝트 삭제 시 목록 자동 갱신 (WebSocket)
- 오프라인: 오프라인 배너, 마지막 캐시 데이터 표시

---

#### `/projects/:id` — 프로젝트 상세
**Access:** Owner, Admin, Member

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 뒤로가기 버튼 잘못된 리다이렉션 (3회) | 뒤로가기 → 명시적으로 `/projects` |
> | UI 컴포넌트 불일치 (8회) | 공통 컴포넌트 재사용, placeholder 필수 |

1) **ProjectHeader**
   - **컴포넌트 상세**: 프로젝트명, 설명, 생성일, 수정일, "수정" 버튼 (Owner, Admin만), "삭제" 버튼 (Owner, Admin), 탭 네비게이션 (화면 목록 / 멤버)
     - Default: 프로젝트 정보 표시
     - Active: 탭 선택 시 underline 표시
     - Disabled: Member에게 수정/삭제 버튼 미표시
     - Error: 해당 없음
   - **데이터 표시**: 프로젝트명, 설명, 생성일, 수정일, 화면 수, 멤버 수
   - **인터랙션**: 탭 클릭 (화면 목록/멤버), "수정" 클릭 → EditProjectModal, "삭제" 클릭 → 확인 다이얼로그
   - **네비게이션**: "화면 목록" 탭 → `/projects/:id/screens`, "멤버" 탭 → `/projects/:id/members`, 뒤로가기 → `/projects`

   **소유권 규칙**: 프로젝트 수정은 Owner, Admin만 가능. 삭제는 Owner, Admin만 가능 (soft delete).

2) **EditProjectModal**
   - **컴포넌트 상세**: 프로젝트명 입력 (기존 값 채워짐), 설명 입력 (기존 값), "저장" 버튼, "취소" 버튼
     - Default: 기존 값 채워진 상태
     - Active: 필드 포커스
     - Disabled: API 호출 중 버튼 spinner
     - Error: 필드 하단 에러 메시지
   - **데이터 표시**: 기존 프로젝트 정보
   - **인터랙션**: ESC로 닫기, Enter로 저장
   - **네비게이션**: 저장 성공 → 모달 닫기, 페이지 새로고침

3) **DeleteConfirmDialog**
   - **컴포넌트 상세**: "정말 삭제하시겠습니까?" 메시지, 프로젝트명 재입력 확인, "삭제" 버튼(빨간색), "취소" 버튼
     - Default: 프로젝트명 확인 입력 비어있음, 삭제 버튼 비활성
     - Active: 프로젝트명 일치 시 삭제 버튼 활성
     - Disabled: API 호출 중
     - Error: 해당 없음
   - **데이터 표시**: 삭제 대상 프로젝트명
   - **인터랙션**: 프로젝트명 입력하여 확인, ESC로 닫기
   - **네비게이션**: 삭제 성공 → `/projects`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| projectName (수정) | text | Y | 2/100 | - | 프로젝트명은 2~100자로 입력해 주세요 |
| projectDescription (수정) | textarea | N | -/500 | - | 설명은 500자 이내로 입력해 주세요 |
| confirmProjectName (삭제) | text | Y | - | 프로젝트명 정확 일치 | 프로젝트명이 일치하지 않습니다 |

**상태 화면**
- Loading: 프로젝트 헤더 skeleton, 탭 콘텐츠 skeleton
- Empty: 해당 없음 (존재하는 프로젝트 접근)
- Error: 프로젝트 미존재 → "프로젝트를 찾을 수 없습니다" + `/projects` 링크

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: 접근 권한 없음 → "이 프로젝트에 접근 권한이 없습니다" + `/projects`로 리다이렉트

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 다른 사용자가 프로젝트 삭제 시 "이 프로젝트가 삭제되었습니다" 토스트 + `/projects`로 이동
- 오프라인: 오프라인 배너, 수정/삭제 비활성

---

#### `/projects/:id/screens` — 화면 목록
**Access:** Owner, Admin, Member

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 파일 업로드 실패 (3회) | 업로드 전 허용 파일 형식/최대 크기 안내, 미충족 시 구체적 에러 |
> | 페이지네이션 미구현 및 정렬 오류 (6회) | 기본 정렬 created_at DESC, 페이지네이션 적용 |
> | 검색 기능 미동작 (6회) | 검색 입력에 300ms 디바운스 |

1) **ScreenListHeader**
   - **컴포넌트 상세**: 프로젝트명 (브레드크럼), "화면 목록" 타이틀, 검색 입력필드, "화면 추가" 버튼 (Owner, Admin, Member)
     - Default: 검색필드 비어있음
     - Active: 검색필드 포커스
     - Disabled: 해당 없음
     - Error: 해당 없음
   - **데이터 표시**: 총 화면 수
   - **인터랙션**: 검색어 입력 (300ms 디바운스), "화면 추가" 클릭 → AddScreenModal
   - **네비게이션**: 브레드크럼 프로젝트명 → `/projects/:id`

2) **ScreenCard (목록)**
   - **컴포넌트 상세**: 화면 썸네일 (최신 버전 스크린샷), 화면명, 버전 수, 미해결 피드백 수, 마지막 수정일, 더보기(...) 메뉴
     - Default: 카드 그리드 배열
     - Active: 호버 시 그림자 효과
     - Disabled: 해당 없음
     - Error: 썸네일 로드 실패 시 placeholder
   - **데이터 표시**: 화면명, 최신 버전 번호, 미해결 피드백 수, 수정일
   - **인터랙션**: 카드 클릭 → 피드백 뷰어, 더보기 → 수정(EditScreenModal 열기)/삭제 (Owner, Admin은 모든 화면 수정·삭제 가능, Member는 본인 등록 화면만 수정·삭제 가능)
   - **네비게이션**: 카드 클릭 → `/projects/:id/screens/:screenId`

3) **AddScreenModal**
   - **컴포넌트 상세**: 화면명 입력, 스크린샷 파일 업로드 영역 (드래그앤드롭 지원), 파일 미리보기, "추가" 버튼, "취소" 버튼
     - Default: 빈 폼, 드래그앤드롭 영역 점선 테두리
     - Active: 파일 드래그 시 영역 하이라이트
     - Disabled: 업로드 중 프로그레스 바
     - Error: 파일 형식/크기 오류 시 에러 메시지
   - **데이터 표시**: 파일명, 파일 크기, 미리보기 이미지
   - **인터랙션**: 파일 드래그앤드롭 또는 클릭하여 파일 선택, ESC로 닫기
   - **네비게이션**: 추가 성공 → 목록 새로고침

   **소유권 규칙**: 화면 삭제는 Owner, Admin은 모든 화면, Member는 본인이 등록한 화면만 가능. 화면 수정은 Owner, Admin은 모든 화면, Member는 본인이 등록한 화면만 가능.

4) **EditScreenModal**
   - **컴포넌트 상세**: 모달 (화면명 입력 — 기존 값 채워짐, 설명 입력(선택) — 기존 값 채워짐, "저장" 버튼, "취소" 버튼)
     - Default: 기존 화면 정보 채워진 상태
     - Active: 입력 필드 포커스
     - Disabled: API 호출 중 버튼 spinner
     - Error: 필드 하단 에러 메시지
   - **데이터 표시**: 기존 화면명, 설명
   - **인터랙션**: ESC로 모달 닫기, Enter로 저장, 배경 클릭으로 모달 닫기
   - **네비게이션**: 저장 성공 → 모달 닫기, 목록 새로고침

   **소유권 규칙**: Owner, Admin은 모든 화면 수정 가능. Member는 본인이 등록한 화면만 수정 가능.

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| search | text | N | -/100 | - | - |
| screenName (추가 모달) | text | Y | 2/100 | - | 화면명은 2~100자로 입력해 주세요 |
| screenshot (추가 모달) | file | Y | -/20MB | PNG, JPG, WebP [📌 권장적용됨] | 20MB 이하의 PNG, JPG, WebP 파일만 업로드 가능합니다 |
| screenName (수정 모달) | text | Y | 2/100 | - | 화면명은 2~100자로 입력해 주세요 |
| screenDescription (수정 모달) | textarea | N | -/500 | - | 설명은 500자 이내로 입력해 주세요 |

**상태 화면**
- Loading: 화면 카드 skeleton (6개 그리드)
- Empty: "등록된 화면이 없습니다" 일러스트 + "화면 추가" CTA
- Error: "화면 목록을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: 프로젝트 접근 권한 없음 → `/projects`로 리다이렉트

**엣지 케이스**
- 0건: empty state + CTA
- 최대치: 프로젝트당 화면 수 제한 없음 (페이지당 12개, 페이지네이션) [📌 권장적용됨]
- 동시접근: 다른 사용자가 화면 삭제 시 목록 자동 갱신 (WebSocket)
- 오프라인: 오프라인 배너, 추가 버튼 비활성, 캐시된 목록 표시

---

#### `/projects/:id/screens/:screenId` — 피드백 뷰어
**Access:** Owner, Admin, Member

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 실시간 메시지/알림 미수신 (5회) | WebSocket 전역 관리. 포그라운드 복귀 시 재연결. 소켓 해제 시 놓친 이벤트 재조회 |
> | 파일 업로드 실패 (3회) | 업로드 전 허용 형식/크기 안내, 서버 MIME 화이트리스트 |
> | UI 컴포넌트 불일치 (8회) | 공통 컴포넌트 재사용, 이미지 없음 시 placeholder |
> | 뒤로가기 잘못된 리다이렉션 (3회) | 뒤로가기 → 명시적으로 `/projects/:id/screens` |

1) **ImageViewer (좌측)**
   - **컴포넌트 상세**: 스크린샷 이미지, 핀 마커 (번호 표시), 버전 선택 드롭다운, 버전 비교 토글 (overlay / side-by-side), 줌 컨트롤 (+/-/fit), 이미지 팬 (드래그)
     - Default: 최신 버전 이미지 표시, 핀 마커 표시
     - Active: 핀 마커 클릭 시 하이라이트(확대+색상 변경), 이미지 클릭 시 새 핀 생성 모드
     - Disabled: Client 역할 외 핀 생성 모드 동일 (전 역할 핀 생성 가능)
     - Error: 이미지 로드 실패 시 "이미지를 불러올 수 없습니다" + 재시도 버튼
   - **데이터 표시**: 스크린샷 이미지, 핀 위치(좌표), 핀 번호, 핀 상태 색상 (open=빨강, in_progress=노랑, resolved=초록), 현재 버전 번호
   - **인터랙션**: 이미지 위 클릭 → 핀 생성 위치 지정 + 코멘트 입력창 열기, 핀 클릭 → 해당 코멘트 하이라이트 (우측 패널 스크롤), 마우스 휠 → 줌, 드래그 → 팬, 버전 드롭다운 변경, 비교 모드 토글
   - **네비게이션**: 뒤로가기 → `/projects/:id/screens`

   **버전 비교 기능**:
   - Overlay 모드: 두 버전을 겹쳐 표시, 슬라이더로 투명도 조절 (0~100%) [📌 권장적용됨]
   - Side-by-side 모드: 좌우 분할 표시, 동기화 스크롤/줌

2) **VersionUploader**
   - **컴포넌트 상세**: "새 버전 업로드" 버튼, 드래그앤드롭 영역, 업로드 프로그레스 바
     - Default: 업로드 버튼 표시
     - Active: 파일 드래그 시 영역 하이라이트
     - Disabled: 업로드 중
     - Error: 파일 형식/크기 오류 메시지
   - **데이터 표시**: 현재 버전 수, 업로드 진행률
   - **인터랙션**: 파일 드래그앤드롭, 클릭하여 파일 선택
   - **네비게이션**: 해당 없음

3) **CommentPanel (우측)**
   - **컴포넌트 상세**: 코멘트 목록, 각 코멘트: 사용자 아바타, 이름, 시간, 내용, 상태 뱃지 (open/in_progress/resolved), 답글 목록, 답글 입력, 첨부파일 표시
     - Default: 시간순 정렬 코멘트 목록
     - Active: 선택된 코멘트 배경색 변경 (핀 클릭 연동)
     - Disabled: 해당 없음
     - Error: 코멘트 로드 실패 시 에러 메시지
   - **데이터 표시**: 코멘트 작성자, 작성 시간 (상대 시간), 코멘트 내용, 상태 뱃지, 답글 수, 첨부파일
   - **인터랙션**: 코멘트 클릭 → 해당 핀 하이라이트 (이미지 뷰어), 상태 드롭다운 변경 (Owner, Admin, Member), 답글 작성, 삭제 (본인 코멘트 또는 Admin+)
   - **네비게이션**: 해당 없음 (패널 내 동작)

   **소유권 규칙**: 코멘트 삭제는 본인 작성분 또는 Admin/Owner만 가능. 상태 변경은 Owner, Admin, Member만 가능. Client는 상태 변경 불가.

4) **NewCommentInput**
   - **컴포넌트 상세**: 텍스트 입력 영역, 파일 첨부 버튼, "게시" 버튼
     - Default: 핀 생성 후 입력창 자동 포커스
     - Active: 텍스트 입력 중
     - Disabled: API 호출 중 spinner
     - Error: 빈 코멘트 제출 시 에러
   - **데이터 표시**: 해당 없음
   - **인터랙션**: 텍스트 입력, 파일 첨부 (드래그 또는 클릭), Cmd/Ctrl+Enter로 제출 [📌 권장적용됨]
   - **네비게이션**: 해당 없음

5) **CommentFilterBar**
   - **컴포넌트 상세**: 상태 필터 (전체/open/in_progress/resolved), 정렬 (최신순/오래된순)
     - Default: "전체" 선택, 최신순
     - Active: 선택된 필터 하이라이트
     - Disabled: 해당 없음
     - Error: 해당 없음
   - **데이터 표시**: 각 상태별 코멘트 수 뱃지
   - **인터랙션**: 필터 클릭, 정렬 변경
   - **네비게이션**: 해당 없음

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| commentText | textarea | Y | 1/2000 | - | 코멘트 내용을 입력해 주세요 |
| replyText | textarea | Y | 1/2000 | - | 답글 내용을 입력해 주세요 |
| attachment | file | N | -/10MB | PNG, JPG, WebP, PDF [📌 권장적용됨] | 10MB 이하의 파일만 첨부 가능합니다 |
| versionFile | file | Y | -/20MB | PNG, JPG, WebP [📌 권장적용됨] | 20MB 이하의 PNG, JPG, WebP 파일만 업로드 가능합니다 |

**상태 화면**
- Loading: 이미지 영역 skeleton + 코멘트 패널 skeleton (5줄)
- Empty: 코멘트 0건 → "아직 피드백이 없습니다. 이미지를 클릭하여 첫 번째 피드백을 남겨보세요!" + 핀 찍기 안내 애니메이션
- Error: "피드백 정보를 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트, 코멘트 작성 실패 시 입력 내용 보존
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트 + 재시도
- 권한 에러: 화면 접근 권한 없음 → `/projects/:id/screens`로 리다이렉트

**엣지 케이스**
- 0건: 코멘트 없음 시 안내 메시지 + CTA
- 최대치: 코멘트 무한스크롤 (한 번에 20개씩 로드) [📌 권장적용됨], 핀 개수 제한 없음
- 동시접근: WebSocket으로 실시간 핀/코멘트 동기화. 다른 사용자 핀 생성 시 즉시 표시. 동시 상태 변경 시 마지막 쓰기 우선 (last-write-wins)
- 오프라인: 오프라인 배너, 코멘트 작성 비활성, 기존 핀/코멘트 읽기 전용

---

#### `/projects/:id/members` — 프로젝트 멤버 관리
**Access:** Owner, Admin

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 관리자 벌크/단일 액션 오류 (3회) | RBAC 적용, 모든 상태 전환 API 사전 구현/테스트 |

1) **MemberListHeader**
   - **컴포넌트 상세**: "프로젝트 멤버" 타이틀, 멤버 수 뱃지, "멤버 초대" 버튼, 검색 입력필드
     - Default: 검색필드 비어있음
     - Active: 검색필드 포커스
     - Disabled: 해당 없음
     - Error: 해당 없음
   - **데이터 표시**: 총 멤버 수
   - **인터랙션**: "멤버 초대" 클릭 → InviteMemberModal, 검색어 입력 (300ms 디바운스)
   - **네비게이션**: 뒤로가기 → `/projects/:id`

2) **MemberTable**
   - **컴포넌트 상세**: 테이블 (아바타, 이름, 이메일, 역할, 참여일, 액션 버튼), 역할 드롭다운 (Admin, Member, Client), "제거" 버튼
     - Default: 역할별 정렬
     - Active: 행 호버 시 배경색 변경
     - Disabled: 자기 자신은 역할 변경/제거 불가
     - Error: 해당 없음
   - **데이터 표시**: 아바타, 이름, 이메일, 역할, 참여일
   - **인터랙션**: 역할 드롭다운 변경, "제거" 클릭 → 확인 다이얼로그
   - **네비게이션**: 해당 없음

   **소유권 규칙**: Owner는 모든 멤버의 역할 변경/제거 가능. Admin은 Member, Client의 역할 변경/제거 가능. 자기 자신은 변경/제거 불가. Owner 역할은 양도 불가 (조직 설정에서만).

3) **InviteMemberModal**
   - **컴포넌트 상세**: 이메일 입력필드 (복수 입력 가능, 태그 형태), 역할 선택 드롭다운, "초대" 버튼, "취소" 버튼
     - Default: 빈 이메일 입력, 역할 기본값 "Member"
     - Active: 이메일 입력 중
     - Disabled: API 호출 중 spinner
     - Error: 중복/무효 이메일 에러
   - **데이터 표시**: 입력된 이메일 태그 목록
   - **인터랙션**: 이메일 입력 후 Enter/쉼표로 태그 추가, 태그 X 클릭 삭제, ESC로 닫기
   - **네비게이션**: 초대 성공 → 모달 닫기 + 멤버 목록 새로고침

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| search | text | N | -/100 | - | - |
| inviteEmail | email (tags) | Y | 1개/20개 [📌 권장적용됨] | 이메일 형식 | 올바른 이메일 주소를 입력해 주세요 |
| inviteRole | select | Y | - | Admin/Member/Client | 역할을 선택해 주세요 |

**상태 화면**
- Loading: 테이블 skeleton (5행)
- Empty: "프로젝트에 멤버가 없습니다" + "멤버 초대" CTA
- Error: "멤버 목록을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: Member/Client → "멤버 관리 권한이 없습니다" + `/projects/:id`로 리다이렉트

**엣지 케이스**
- 0건: Owner 본인만 있는 상태 → 초대 CTA
- 최대치: 한 번에 최대 20명 초대 [📌 권장적용됨], 이미 초대된 이메일 중복 안내
- 동시접근: 동시 역할 변경 시 마지막 쓰기 우선, 실시간 목록 갱신
- 오프라인: 오프라인 배너, 초대/역할변경/제거 비활성

---

#### `/settings` — 설정 메인
**Access:** Owner, Admin, Member

1) **SettingsNavigation**
   - **컴포넌트 상세**: 좌측 사이드 네비게이션, 메뉴 항목: 프로필, 조직 설정 (Owner만), 팀 관리 (Owner, Admin만), 빌링 (Owner만)
     - Default: 현재 선택 메뉴 하이라이트
     - Active: 호버 시 배경색 변경
     - Disabled: 역할에 따라 미표시 (hidden)
     - Error: 해당 없음
   - **데이터 표시**: 메뉴 라벨, 현재 선택 상태
   - **인터랙션**: 메뉴 클릭 → 해당 설정 페이지 이동
   - **네비게이션**: "프로필" → `/settings/profile`, "조직 설정" → `/settings/organization`, "팀 관리" → `/settings/team`, "빌링" → `/settings/billing`, 뒤로가기 → `/dashboard`

**입력필드 검증규칙**
해당 없음

**상태 화면**
- Loading: 사이드 네비게이션 skeleton
- Empty: 해당 없음
- Error: 해당 없음

**에러 핸들링**
- 네트워크 에러: 해당 없음 (정적 네비게이션)
- 서버 에러: 해당 없음
- 권한 에러: 해당 없음 (메뉴 자체가 역할별 필터링)

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 해당 없음
- 오프라인: 네비게이션은 동작, 각 하위 페이지에서 오프라인 처리

---

#### `/settings/organization` — 조직 설정
**Access:** Owner

1) **OrganizationForm**
   - **컴포넌트 상세**: 조직명 입력필드, 조직 로고 업로드, "저장" 버튼, "조직 삭제" 섹션 (위험 영역)
     - Default: 기존 조직 정보 채워진 상태
     - Active: 필드 포커스
     - Disabled: API 호출 중 spinner
     - Error: 필드 하단 에러 메시지
   - **데이터 표시**: 조직명, 로고, 생성일, 멤버 수
   - **인터랙션**: 필드 수정, 로고 업로드 (드래그앤드롭 또는 클릭), "저장" 클릭, "조직 삭제" 클릭 → 확인 다이얼로그
   - **네비게이션**: 뒤로가기 → `/settings`

   **소유권 규칙**: Owner만 조직 정보 수정/삭제 가능.

2) **DeleteOrganizationDialog**
   - **컴포넌트 상세**: 경고 메시지 ("모든 프로젝트와 데이터가 삭제됩니다"), 조직명 재입력 확인, "삭제" 버튼(빨간색), "취소" 버튼
     - Default: 입력 비어있음, 삭제 버튼 비활성
     - Active: 조직명 일치 시 삭제 버튼 활성
     - Disabled: API 호출 중
     - Error: 해당 없음
   - **데이터 표시**: 삭제 대상 조직명, 영향 범위 (프로젝트 수, 멤버 수)
   - **인터랙션**: 조직명 입력, ESC로 닫기
   - **네비게이션**: 삭제 성공 → 로그아웃 + `/auth/login`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| organizationName | text | Y | 2/100 | - | 조직명은 2~100자로 입력해 주세요 |
| organizationLogo | file | N | -/2MB [📌 권장적용됨] | PNG, JPG, WebP | 2MB 이하의 이미지 파일만 업로드 가능합니다 |
| confirmOrgName (삭제) | text | Y | - | 조직명 정확 일치 | 조직명이 일치하지 않습니다 |

**상태 화면**
- Loading: 폼 skeleton
- Empty: 해당 없음
- Error: "조직 정보를 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: Owner 외 접근 → "/settings"로 리다이렉트

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 해당 없음 (Owner 단독 관리)
- 오프라인: 오프라인 배너, 저장/삭제 비활성

---

#### `/settings/team` — 팀 관리
**Access:** Owner, Admin

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 관리자 벌크/단일 액션 오류 (3회) | RBAC 적용, 모든 상태 전환 API 사전 구현/테스트 |
> | 페이지네이션 미구현 및 정렬 오류 (6회) | 멤버 10명+ 시 페이지네이션 |
> | 검색 기능 미동작 (6회) | 검색 300ms 디바운스, ILIKE 부분 일치 |

1) **TeamListHeader**
   - **컴포넌트 상세**: "팀 관리" 타이틀, 멤버 수 뱃지, "멤버 초대" 버튼, 검색 입력필드, 역할 필터 드롭다운 (전체/Owner/Admin/Member/Client)
     - Default: "전체" 필터, 검색 비어있음
     - Active: 검색 포커스, 필터 변경
     - Disabled: 해당 없음
     - Error: 해당 없음
   - **데이터 표시**: 총 멤버 수, 역할별 멤버 수
   - **인터랙션**: 검색 (300ms 디바운스), 필터 변경, "초대" 클릭 → InviteTeamMemberModal
   - **네비게이션**: 뒤로가기 → `/settings`

2) **TeamMemberTable**
   - **컴포넌트 상세**: 테이블 (아바타, 이름, 이메일, 역할, 가입일, 상태, 액션), 역할 드롭다운, "제거" 버튼, 초대 대기 중 표시
     - Default: 이름순 정렬
     - Active: 행 호버
     - Disabled: 자기 자신 변경 불가, Admin은 Owner 변경 불가
     - Error: 해당 없음
   - **데이터 표시**: 아바타, 이름, 이메일, 역할, 가입일, 상태 (활성/초대대기)
   - **인터랙션**: 역할 변경, 제거, 초대 재전송
   - **네비게이션**: 해당 없음

   **소유권 규칙**: Owner는 모든 멤버 관리. Admin은 Member/Client만 관리. 자기 자신 역할 변경/제거 불가. Owner 역할 양도 불가.

3) **InviteTeamMemberModal**
   - **컴포넌트 상세**: 이메일 입력 (태그형 복수 입력), 역할 선택, "초대" 버튼, "취소" 버튼
     - Default: 빈 폼, 역할 기본값 "Member"
     - Active: 이메일 입력 중
     - Disabled: API 호출 중
     - Error: 중복/무효 이메일 에러
   - **데이터 표시**: 입력된 이메일 태그
   - **인터랙션**: Enter/쉼표로 태그 추가, X 클릭 삭제, ESC 닫기
   - **네비게이션**: 초대 성공 → 모달 닫기 + 목록 새로고침

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| search | text | N | -/100 | - | - |
| inviteEmail | email (tags) | Y | 1개/20개 [📌 권장적용됨] | 이메일 형식 | 올바른 이메일 주소를 입력해 주세요 |
| inviteRole | select | Y | - | Admin/Member/Client | 역할을 선택해 주세요 |

**상태 화면**
- Loading: 테이블 skeleton (5행)
- Empty: "조직에 멤버가 없습니다" (Owner 본인만) + "멤버 초대" CTA
- Error: "팀 목록을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: Member/Client → "/settings"로 리다이렉트

**엣지 케이스**
- 0건: Owner 본인만 표시 + 초대 CTA
- 최대치: 조직당 멤버 수 제한 없음 (페이지당 20명, 페이지네이션) [📌 권장적용됨]
- 동시접근: 동시 역할 변경 시 마지막 쓰기 우선, 실시간 목록 갱신
- 오프라인: 오프라인 배너, 초대/변경/제거 비활성

---

#### `/settings/billing` — 빌링/결제
**Access:** Owner

1) **BillingOverview**
   - **컴포넌트 상세**: 현재 플랜 표시 카드, 사용량 표시 (프로젝트 수, 멤버 수, 저장 용량), "플랜 변경" 버튼, 다음 결제일 표시
     - Default: 현재 플랜 정보 표시. Stripe 연동 전까지 "무료 플랜" 표시 [📌 권장적용됨]
     - Active: "플랜 변경" 호버 시 하이라이트
     - Disabled: Stripe 미연동 시 결제 관련 버튼 "준비 중" 표시
     - Error: 결제 정보 로드 실패 시 에러 메시지
   - **데이터 표시**: 플랜명, 가격, 사용량 (프로젝트/멤버/저장소), 다음 결제일
   - **인터랙션**: "플랜 변경" 클릭 → Stripe 포탈 (추후) [📌 권장적용됨]
   - **네비게이션**: 뒤로가기 → `/settings`

2) **PaymentHistory**
   - **컴포넌트 상세**: 결제 내역 테이블 (날짜, 금액, 상태, 영수증 링크), 페이지네이션
     - Default: 최신순 정렬
     - Active: 행 호버
     - Disabled: Stripe 미연동 시 빈 상태
     - Error: 내역 로드 실패 시 에러 메시지
   - **데이터 표시**: 결제일, 금액, 결제 상태 (성공/실패/환불), 영수증 링크
   - **인터랙션**: 영수증 링크 클릭 → 새 탭에서 영수증 PDF
   - **네비게이션**: 해당 없음

**입력필드 검증규칙**
해당 없음

**상태 화면**
- Loading: 플랜 카드 skeleton, 결제 내역 테이블 skeleton
- Empty: "결제 내역이 없습니다" (Stripe 미연동 시 "결제 기능이 준비 중입니다" [📌 권장적용됨])
- Error: "결제 정보를 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: Owner 외 접근 → "/settings"로 리다이렉트

**엣지 케이스**
- 0건: 결제 내역 없음 안내
- 최대치: 결제 내역 페이지네이션 (페이지당 10건)
- 동시접근: 해당 없음 (Owner 단독)
- 오프라인: 오프라인 배너, 결제 관련 액션 비활성

---

#### `/settings/profile` — 프로필 설정
**Access:** All authenticated

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 프로필 이미지 변경 실패 (3회) | 업로드 전 최대 2MB 압축, 업로드 성공 후 캐시 무효화. 서버 응답에 새 이미지 URL 포함 |
> | 폼 유효성 검사 미동작 (7회) | 모든 입력 필드 실시간 유효성 검사 |
> | 비밀번호 유효성 검사 미흡 (3회) | 비밀번호 실시간 유효성 검사, 규칙 미충족 시 구체적 안내 |

1) **ProfileForm**
   - **컴포넌트 상세**: 프로필 이미지 업로드 (아바타 클릭하여 변경), 이름 입력필드, 이메일 표시 (읽기 전용), 언어 선택 드롭다운 (EN/KO), "저장" 버튼
     - Default: 기존 프로필 정보 채워진 상태
     - Active: 필드 포커스, 아바타 호버 시 "변경" 오버레이
     - Disabled: API 호출 중 spinner
     - Error: 필드 하단 에러 메시지
   - **데이터 표시**: 아바타, 이름, 이메일 (읽기 전용), 선택된 언어
   - **인터랙션**: 아바타 클릭 → 파일 선택, 이름 수정, 언어 변경, "저장" 클릭
   - **네비게이션**: 뒤로가기 → `/settings`

2) **ChangePasswordSection**
   - **컴포넌트 상세**: 현재 비밀번호 입력, 새 비밀번호 입력 (강도 인디케이터), 비밀번호 확인 입력, "비밀번호 변경" 버튼
     - Default: 입력필드 비어있음
     - Active: 필드 포커스, 강도 인디케이터 실시간 반영
     - Disabled: API 호출 중
     - Error: 필드별 에러 메시지
   - **데이터 표시**: 비밀번호 강도 인디케이터
   - **인터랙션**: 비밀번호 표시/숨김 토글, Enter로 제출
   - **네비게이션**: 해당 없음

3) **LeaveOrganizationSection**
   - **컴포넌트 상세**: "조직 탈퇴" 섹션 (위험 영역, 빨간색 테두리), 현재 소속 조직명 표시, "조직 탈퇴" 버튼 (Client, Member, Admin 가능, Owner는 비활성 — Owner는 조직 삭제만 가능)
     - Default: 조직명 및 탈퇴 안내 텍스트 표시
     - Active: "조직 탈퇴" 버튼 호버 시 빨간색 하이라이트
     - Disabled: Owner인 경우 "조직의 소유자는 탈퇴할 수 없습니다. 조직 설정에서 소유권을 이전하거나 조직을 삭제해 주세요" 안내 표시, 버튼 비활성
     - Error: 해당 없음
   - **데이터 표시**: 소속 조직명, 탈퇴 시 영향 안내 ("탈퇴 시 해당 조직의 모든 프로젝트와 데이터에 접근할 수 없게 됩니다")
   - **인터랙션**: "조직 탈퇴" 클릭 → LeaveOrganizationConfirmDialog 열기
   - **네비게이션**: 탈퇴 성공 → 로그아웃 + `/auth/login`

4) **LeaveOrganizationConfirmDialog**
   - **컴포넌트 상세**: 경고 메시지 ("정말 조직을 탈퇴하시겠습니까?"), 조직명 표시, "탈퇴" 버튼(빨간색), "취소" 버튼
     - Default: "탈퇴" 버튼 활성
     - Active: 해당 없음
     - Disabled: API 호출 중 spinner
     - Error: 해당 없음
   - **데이터 표시**: 탈퇴 대상 조직명
   - **인터랙션**: "탈퇴" 클릭, ESC로 닫기
   - **네비게이션**: 탈퇴 성공 → 로그아웃 + `/auth/login`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| name | text | Y | 2/50 | 한글/영문/숫자 | 이름은 2~50자로 입력해 주세요 |
| profileImage | file | N | -/2MB | PNG, JPG, WebP | 2MB 이하의 이미지 파일만 업로드 가능합니다 |
| language | select | Y | - | en/ko | - |
| currentPassword | password | Y | 8/128 | - | 현재 비밀번호를 입력해 주세요 |
| newPassword | password | Y | 8/128 | 영문+숫자+특수문자 조합 [📌 권장적용됨] | 비밀번호는 8자 이상, 영문/숫자/특수문자를 포함해야 합니다 |
| confirmNewPassword | password | Y | 8/128 | newPassword와 일치 | 비밀번호가 일치하지 않습니다 |

**상태 화면**
- Loading: 프로필 폼 skeleton
- Empty: 해당 없음
- Error: "프로필 정보를 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: 해당 없음 (본인 프로필)

**엣지 케이스**
- 0건: 해당 없음
- 최대치: 해당 없음
- 동시접근: 다른 탭에서 프로필 변경 시 새로고침하면 최신 값 반영
- 오프라인: 오프라인 배너, 저장 비활성

---

#### `/client/projects` — 클라이언트 프로젝트 목록
**Access:** Client

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 페이지네이션 미구현 및 정렬 오류 (6회) | 기본 정렬 created_at DESC, 페이지네이션 적용 |
> | 검색 기능 미동작 (6회) | 검색 300ms 디바운스 |

1) **ClientProjectListHeader**
   - **컴포넌트 상세**: "내 프로젝트" 타이틀, 검색 입력필드, 정렬 드롭다운 (최신순/이름순)
     - Default: 검색 비어있음, 최신순
     - Active: 검색 포커스
     - Disabled: 해당 없음
     - Error: 해당 없음
   - **데이터 표시**: 배정된 프로젝트 수
   - **인터랙션**: 검색 (300ms 디바운스), 정렬 변경
   - **네비게이션**: 해당 없음

2) **ClientProjectCard**
   - **컴포넌트 상세**: 프로젝트 썸네일, 프로젝트명, 화면 수, 미해결 피드백 수, 마지막 수정일
     - Default: 카드 그리드
     - Active: 호버 시 그림자
     - Disabled: 해당 없음
     - Error: 썸네일 로드 실패 시 placeholder
   - **데이터 표시**: 프로젝트명, 화면 수, 미해결 피드백 수, 수정일
   - **인터랙션**: 카드 클릭 → 클라이언트 화면 목록
   - **네비게이션**: 카드 클릭 → `/client/projects/:id/screens`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| search | text | N | -/100 | - | - |

**상태 화면**
- Loading: 카드 skeleton (6개)
- Empty: "배정된 프로젝트가 없습니다. 관리자가 프로젝트에 초대하면 여기에 표시됩니다."
- Error: "프로젝트 목록을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: Client 외 접근 → `/dashboard`로 리다이렉트

**엣지 케이스**
- 0건: empty state (배정 대기 안내)
- 최대치: 페이지당 12개, 페이지네이션 [📌 권장적용됨]
- 동시접근: 프로젝트 배정/해제 시 실시간 반영
- 오프라인: 오프라인 배너, 캐시된 목록 표시

---

#### `/client/projects/:id/screens` — 클라이언트 화면 목록
**Access:** Client

1) **ClientScreenListHeader**
   - **컴포넌트 상세**: 프로젝트명 (브레드크럼), "화면 목록" 타이틀, 검색 입력필드
     - Default: 검색 비어있음
     - Active: 검색 포커스
     - Disabled: 해당 없음 (읽기 전용 — 화면 추가 버튼 없음)
     - Error: 해당 없음
   - **데이터 표시**: 총 화면 수
   - **인터랙션**: 검색 (300ms 디바운스)
   - **네비게이션**: 브레드크럼 → `/client/projects`, 뒤로가기 → `/client/projects`

2) **ClientScreenCard**
   - **컴포넌트 상세**: 화면 썸네일, 화면명, 버전 수, 미해결 피드백 수
     - Default: 카드 그리드
     - Active: 호버 시 그림자
     - Disabled: 해당 없음
     - Error: 썸네일 로드 실패 시 placeholder
   - **데이터 표시**: 화면명, 버전 수, 미해결 피드백 수
   - **인터랙션**: 카드 클릭 → 클라이언트 피드백 뷰어
   - **네비게이션**: 카드 클릭 → `/client/projects/:id/screens/:screenId`

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| search | text | N | -/100 | - | - |

**상태 화면**
- Loading: 카드 skeleton (6개)
- Empty: "이 프로젝트에 등록된 화면이 없습니다."
- Error: "화면 목록을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: 미배정 프로젝트 접근 → `/client/projects`로 리다이렉트

**엣지 케이스**
- 0건: empty state
- 최대치: 페이지당 12개, 페이지네이션 [📌 권장적용됨]
- 동시접근: 화면 추가/삭제 시 실시간 반영
- 오프라인: 오프라인 배너, 캐시된 목록 표시

---

#### `/client/projects/:id/screens/:screenId` — 클라이언트 피드백 뷰어
**Access:** Client

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 실시간 메시지/알림 미수신 (5회) | WebSocket 전역 관리. 포그라운드 복귀 시 재연결 |
> | UI 컴포넌트 불일치 (8회) | 공통 컴포넌트 재사용, placeholder 필수 |
> | 뒤로가기 잘못된 리다이렉션 (3회) | 뒤로가기 → 명시적으로 `/client/projects/:id/screens` |

1) **ClientImageViewer (좌측)**
   - **컴포넌트 상세**: 스크린샷 이미지, 핀 마커 (번호 표시), 버전 선택 드롭다운, 버전 비교 토글 (overlay / side-by-side), 줌 컨트롤 (+/-/fit), 이미지 팬 (드래그)
     - Default: 최신 버전 이미지 + 핀 표시
     - Active: 핀 클릭 시 하이라이트, 이미지 클릭 시 새 핀 생성
     - Disabled: 버전 업로드 불가 (Client는 업로드 권한 없음)
     - Error: 이미지 로드 실패 시 에러 메시지 + 재시도
   - **데이터 표시**: 스크린샷 이미지, 핀 위치/번호/상태색상, 현재 버전
   - **인터랙션**: 이미지 클릭 → 핀 생성 + 코멘트 입력, 핀 클릭 → 코멘트 하이라이트, 마우스 휠 줌, 드래그 팬, 버전 변경, 비교 모드 토글
   - **네비게이션**: 뒤로가기 → `/client/projects/:id/screens`

   **버전 비교 기능**:
   - Overlay 모드: 두 버전을 겹쳐 표시, 슬라이더로 투명도 조절 (0~100%) [📌 권장적용됨]
   - Side-by-side 모드: 좌우 분할 표시, 동기화 스크롤/줌

2) **ClientCommentPanel (우측)**
   - **컴포넌트 상세**: 코멘트 목록, 각 코멘트: 사용자 아바타/이름/시간/내용/상태뱃지, 답글, 코멘트 필터
     - Default: 시간순 코멘트 목록
     - Active: 선택된 코멘트 하이라이트
     - Disabled: 상태 변경 불가 (Client 권한 제한)
     - Error: 로드 실패 시 에러 메시지
   - **데이터 표시**: 코멘트 작성자, 시간, 내용, 상태, 답글
   - **인터랙션**: 코멘트 클릭 → 핀 하이라이트, 답글 작성, 새 코멘트 작성
   - **네비게이션**: 해당 없음

   **소유권 규칙**: Client는 본인 작성 코멘트만 삭제 가능. 상태 변경 불가. 핀 생성 및 코멘트/답글 작성 가능.

3) **ClientNewCommentInput**
   - **컴포넌트 상세**: 텍스트 입력, 파일 첨부 버튼, "게시" 버튼
     - Default: 핀 생성 후 자동 포커스
     - Active: 텍스트 입력 중
     - Disabled: API 호출 중
     - Error: 빈 코멘트 에러
   - **데이터 표시**: 해당 없음
   - **인터랙션**: 텍스트 입력, 파일 첨부, Cmd/Ctrl+Enter 제출 [📌 권장적용됨]
   - **네비게이션**: 해당 없음

**입력필드 검증규칙**
| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|
| commentText | textarea | Y | 1/2000 | - | 코멘트 내용을 입력해 주세요 |
| replyText | textarea | Y | 1/2000 | - | 답글 내용을 입력해 주세요 |
| attachment | file | N | -/10MB | PNG, JPG, WebP, PDF [📌 권장적용됨] | 10MB 이하의 파일만 첨부 가능합니다 |

**상태 화면**
- Loading: 이미지 skeleton + 코멘트 패널 skeleton
- Empty: "아직 피드백이 없습니다. 이미지를 클릭하여 피드백을 남겨보세요!"
- Error: "피드백 정보를 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트, 코멘트 입력 보존
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: 미배정 프로젝트 → `/client/projects`로 리다이렉트

**엣지 케이스**
- 0건: 코멘트 없음 안내 + 핀 찍기 유도
- 최대치: 코멘트 무한스크롤 (20개씩) [📌 권장적용됨]
- 동시접근: WebSocket 실시간 동기화
- 오프라인: 오프라인 배너, 코멘트 작성 비활성, 읽기 전용

---

#### `/notifications` — 알림 목록
**Access:** All authenticated

> **알려진 위험**
>
> | 버그 패턴 | 예방 스펙 |
> |----------|----------|
> | 실시간 메시지/알림 미수신 (5회) | WebSocket 전역 관리. 포그라운드 복귀 시 연결 확인/재연결 |
> | 페이지네이션 미구현 및 정렬 오류 (6회) | 기본 정렬 created_at DESC, 무한스크롤 |

1) **NotificationListHeader**
   - **컴포넌트 상세**: "알림" 타이틀, 읽지 않은 알림 수 뱃지, "모두 읽음 처리" 버튼, 필터 (전체/읽지않음)
     - Default: "전체" 필터
     - Active: 필터 선택 하이라이트
     - Disabled: 읽지 않은 알림 0건 시 "모두 읽음 처리" 비활성
     - Error: 해당 없음
   - **데이터 표시**: 읽지 않은 알림 수
   - **인터랙션**: 필터 변경, "모두 읽음 처리" 클릭
   - **네비게이션**: 뒤로가기 → `/dashboard`

2) **NotificationItem**
   - **컴포넌트 상세**: 알림 아이콘 (타입별), 알림 내용 텍스트, 상대 시간, 읽음/안읽음 상태 (안읽음: 배경색 구분)
     - Default: 안읽음은 배경색 하이라이트
     - Active: 호버 시 배경색 변경
     - Disabled: 해당 없음
     - Error: 해당 없음
   - **데이터 표시**: 알림 타입 아이콘, 내용 (예: "김철수님이 Project A의 화면에 코멘트를 남겼습니다"), 시간
   - **인터랙션**: 클릭 → 읽음 처리 + 해당 리소스로 이동
   - **네비게이션**: 알림 클릭 → 해당 프로젝트/화면/피드백 뷰어

   **알림 유형** [📌 권장적용됨]:
   - 새 코멘트: 내 화면에 코멘트가 달렸을 때
   - 답글: 내 코멘트에 답글이 달렸을 때
   - 상태 변경: 내 코멘트 상태가 변경되었을 때
   - 초대: 프로젝트/조직에 초대되었을 때
   - 멘션: 코멘트에서 멘션되었을 때

**입력필드 검증규칙**
해당 없음

**상태 화면**
- Loading: 알림 항목 skeleton (5줄)
- Empty: "새로운 알림이 없습니다" 일러스트
- Error: "알림을 불러올 수 없습니다" + 재시도 버튼

**에러 핸들링**
- 네트워크 에러: "네트워크 연결을 확인해 주세요" 토스트
- 서버 에러: "일시적인 오류가 발생했습니다" 토스트
- 권한 에러: 해당 없음 (본인 알림만 표시)

**엣지 케이스**
- 0건: empty state 일러스트
- 최대치: 무한스크롤 (20개씩 로드) [📌 권장적용됨], 30일 이상 지난 알림 자동 정리 [📌 권장적용됨]
- 동시접근: WebSocket으로 새 알림 실시간 수신, 헤더 알림 뱃지 실시간 업데이트
- 오프라인: 오프라인 배너, 캐시된 알림 표시, "모두 읽음" 비활성

---

## 4. Admin Dashboard (슈퍼어드민 전용)

> 이 섹션의 모든 라우트는 슈퍼어드민(플랫폼 운영자) 전용이며, 일반 조직 사용자(Owner/Admin/Member/Client)는 접근 불가입니다. 모든 관리 페이지에는 아래 Admin Standards가 자동 적용됩니다.

---

## 4.1 Page Architecture

### Page Map

| Route | Page | 관리 대상 |
|-------|------|-----------|
| `/admin` | 슈퍼어드민 대시보드 | 전체 플랫폼 통계 |
| `/admin/organizations` | 조직 관리 목록 | Organization |
| `/admin/organizations/:id` | 조직 상세 | Organization |
| `/admin/users` | 사용자 관리 목록 | User |
| `/admin/users/:id` | 사용자 상세 | User |
| `/admin/projects` | 프로젝트 관리 목록 | Project |
| `/admin/projects/:id` | 프로젝트 상세 | Project |
| `/admin/screens` | 화면 관리 목록 | Screen |
| `/admin/comments` | 코멘트 관리 목록 | Comment |
| `/admin/invitations` | 초대 관리 목록 | Invitation |
| `/admin/audit-logs` | 감사 로그 | AuditLog |
| `/admin/settings` | 플랫폼 설정 | Platform |

### 공통 레이아웃

- 좌측 사이드바: 어드민 전용 네비게이션 (대시보드 / 조직 / 사용자 / 프로젝트 / 화면 / 코멘트 / 초대 / 감사 로그 / 설정)
- 상단 헤더: 슈퍼어드민 계정 표시, 로그아웃
- 브레드크럼(Breadcrumb): 모든 어드민 페이지 상단에 표시 (예: Admin > Organizations > Acme Corp)
- 전체 배경색은 일반 사용자 앱과 구분되는 어드민 테마 적용 권장

---

## 4.2 Feature List by Route

---

### 4.2.1 `/admin` — 슈퍼어드민 대시보드

#### Overview

플랫폼 전체 현황을 한눈에 파악하는 홈 화면입니다. 주요 통계 카드, 추이 차트, 최근 활동 피드를 제공합니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 어드민 필터/통계 오류 | 6회 | 통계 API는 모든 상태값 집계. '전체' 필터 시 전체 데이터 반환. 화면 포커스 시 재조회 또는 30초 자동 갱신 |

#### 4.2.1.1 Period Filter

- 기간 선택 옵션: 오늘 / 최근 7일 / 최근 30일 / 최근 90일 / 커스텀 날짜 범위 (Date Picker)
- 기간 변경 시 통계 카드 및 차트 즉시 갱신
- 기본값: 최근 30일
- 선택된 기간은 URL query parameter(`?period=30d`)에 유지

#### 4.2.1.2 Statistics Cards

| 카드 | 지표 | 전후 비교 표시 |
|------|------|----------------|
| 전체 조직 수 | 누적 조직 수 | 전 기간 대비 ±N (%) |
| 신규 조직 | 기간 내 신규 생성 | 전 기간 대비 ±N (%) |
| 전체 사용자 수 | 누적 사용자 수 | 전 기간 대비 ±N (%) |
| 신규 사용자 | 기간 내 신규 가입 | 전 기간 대비 ±N (%) |
| 전체 프로젝트 수 | 누적 프로젝트 수 | 전 기간 대비 ±N (%) |
| 활성 조직 수 | status=active 조직 | 전 기간 대비 ±N (%) |
| 총 코멘트 수 | 누적 코멘트 수 | 전 기간 대비 ±N (%) |
| 총 스크린 수 | 누적 Screen 수 | 전 기간 대비 ±N (%) |

- 각 카드에 증감 방향 아이콘(↑ 녹색 / ↓ 빨강) 표시
- 카드 클릭 시 해당 관리 목록 페이지로 이동

#### 4.2.1.3 Charts

| 차트 | 종류 | X축 | Y축 |
|------|------|-----|-----|
| 신규 조직 추이 | Line Chart | 날짜 (일/주) | 신규 조직 수 |
| 신규 사용자 추이 | Line Chart | 날짜 (일/주) | 신규 사용자 수 |
| 조직 상태 분포 | Donut Chart | — | active / suspended |
| 코멘트 상태 분포 | Bar Chart | 날짜 | open / in_progress / resolved 누적 |

- 차트 hover 시 툴팁으로 수치 표시
- 차트 범례(Legend) 클릭으로 데이터 시리즈 on/off 가능

#### 4.2.1.4 Recent Activity Feed

- 최근 30건의 플랫폼 주요 이벤트 표시 (조직 생성, 사용자 가입, 프로젝트 생성 등)
- 각 항목: 이벤트 아이콘 + 설명 + 대상 링크 + 시간 (YYYY-MM-DD HH:mm + 상대시간)
- "모두 보기" 클릭 시 `/admin/audit-logs`로 이동

---

### 4.2.2 `/admin/organizations` — 조직 관리 목록

#### Overview

플랫폼에 등록된 모든 조직을 조회·관리하는 페이지입니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 어드민 필터/통계 오류 | 6회 | '전체' 필터 시 전체 데이터 반환. 통계 API는 모든 상태값 집계 |
| 관리자 벌크/단일 액션 오류 | 3회 | 어드민 API에 RBAC 적용, 모든 상태 전환 API 사전 구현/테스트 |
| 페이지네이션 미구현 및 정렬 오류 | 6회 | 목록 API는 page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건은 전역 상태 보관, 필터 초기화 버튼 별도 제공 |
| 검색 기능 미동작 | 6회 | 검색 입력에 300ms 디바운스, 서버는 ILIKE 부분 일치 지원 |

#### 4.2.2.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 조직명, 슬러그로 검색. 300ms 디바운스 적용. 0건 시 Empty State |
| 필터 | Status Dropdown | All / Active / Suspended |
| 필터 | Plan Dropdown | All / Free / Pro / Enterprise (요금제 변경 시 Open Question #TBD-10 참조) |
| 필터 | 가입일 Date Range | YYYY-MM-DD ~ YYYY-MM-DD |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |
| 정렬 | 기본값 | created_at DESC |
| Create 버튼 | "조직 생성" | 클릭 시 Creation Modal 오픈 |
| Bulk Action | Dropdown | 선택된 행에 일괄 적용 (정지, 삭제) |

#### 4.2.2.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| Checkbox | Boolean | - | 전체 선택/해제 |
| 조직명 | String | ✓ | 상세 페이지 링크 |
| 슬러그 | String | ✓ | 고유 식별자 |
| Plan | Enum | ✓ | Free / Pro / Enterprise |
| 상태 | Enum Badge | ✓ | Active(녹) / Suspended(빨) |
| 멤버 수 | Number | ✓ | 조직 소속 전체 사용자 수 |
| 프로젝트 수 | Number | ✓ | 조직 소속 전체 프로젝트 수 |
| 생성일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 액션 | - | - | 상세보기 / 상태변경 / 삭제 |

> **상태 enum 통일:** 조직 상태는 `active` / `suspended` 두 가지만 사용합니다. 별도의 `inactive` 상태는 존재하지 않으며, Section 1의 enum 정의와 일치합니다. 조직 비활성화가 필요한 경우 `suspended`로 전환합니다.

**Table State 처리:**
- Loading State: 스켈레톤 로딩 (행 단위)
- Empty State: "등록된 조직이 없습니다" 메시지 + 조직 생성 버튼
- Error State: "데이터를 불러오지 못했습니다. 다시 시도해 주세요." + 재시도 버튼
- Search No Results: "검색 결과가 없습니다. 검색어를 변경해 보세요."

#### 4.2.2.3 Pagination

- 기본 페이지 크기: 20건
- 페이지 크기 선택: 20 / 50 / 100
- 현재 페이지 및 총 건수 표시 (예: 1-20 / 총 148건)
- URL query parameter로 페이지 상태 유지 (`?page=2&size=20`)

#### 4.2.2.4 Creation Modal

| 필드 | 타입 | 필수 | 검증 규칙 |
|------|------|------|-----------|
| 조직명 | Text Input | ✓ | 2~100자, 중복 불가, XSS 방지 |
| 슬러그 | Text Input | ✓ | 소문자/숫자/하이픈, 3~50자, 중복 불가, 자동생성(조직명 기반) 후 수동 편집 가능 |
| Plan | Select | ✓ | Free / Pro / Enterprise 중 선택 |
| Owner 이메일 | Email Input | ✓ | 유효한 이메일 형식, 기존 사용자 or 신규 초대 |
| 설명 | Textarea | - | 최대 500자 |

- Submit 버튼: 모든 필수 필드 유효 시에만 활성화
- 슬러그 중복 실시간 검사 (blur 이벤트)
- 성공 시: Toast "조직이 생성되었습니다." + 목록 갱신
- 실패 시: 서버 오류 메시지 필드 단위로 표시

#### 4.2.2.5 Bulk Action

| 액션 | 대상 | 확인 Dialog |
|------|------|------------|
| 정지(Suspend) | 선택된 조직 전체 | "N개 조직을 정지하시겠습니까? 모든 접근이 즉시 차단됩니다." |
| 삭제 | 선택된 조직 전체 | "N개 조직을 삭제하시겠습니까? 이 작업은 되돌릴 수 없습니다. (Soft Delete)" |

- 0건 선택 시 Bulk Action 버튼 비활성화
- 각 Bulk Action 수행 전 확인 Dialog 필수 표시 (취소 / 확인 버튼)

#### 4.2.2.6 Export

- CSV / Excel(.xlsx) 다운로드 버튼 제공 (목록 상단 우측)
- 내보내기 범위: 현재 필터 조건 적용 결과 전체 (페이지 무관)
- Date Range 선택 가능 (기본: 전체 기간)
- 내보내기 컬럼: 조직명, 슬러그, Plan, 상태, 멤버 수, 프로젝트 수, 생성일, 삭제일(soft delete 포함)
- 다운로드 중 Progress Indicator 표시

---

### 4.2.3 `/admin/organizations/:id` — 조직 상세

#### Overview

특정 조직의 상세 정보, 소속 멤버, 소속 프로젝트, 빌링 현황을 조회하고 상태를 관리하는 페이지입니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 관리자 벌크/단일 액션 오류 | 3회 | 모든 상태 전환 API 사전 구현/테스트, RBAC 적용 |
| API 500 에러 | 2회 | UPDATE 엔드포인트는 partial update 지원, null 필드 무시 |

#### 4.2.3.1 Breadcrumb

Admin > Organizations > [조직명]

#### 4.2.3.2 Detail Section — 조직 기본 정보

| 필드 | 타입 | 표시 방식 |
|------|------|-----------|
| 조직명 | String | 인라인 편집 가능 |
| 슬러그 | String | 읽기 전용 (변경 불가) |
| Plan | Enum | Dropdown 편집 가능 |
| 상태 | Enum Badge | Dropdown 편집 가능 (Active / Suspended) |
| 설명 | Text | 인라인 편집 가능 |
| 생성일 | DateTime | YYYY-MM-DD HH:mm + 상대시간, 읽기 전용 |
| 최종 수정일 | DateTime | YYYY-MM-DD HH:mm + 상대시간, 읽기 전용 |
| 삭제일 | DateTime | soft delete 시에만 표시, YYYY-MM-DD HH:mm |

**가능한 액션 (상단 우측 버튼 그룹):**
- 편집 저장: 수정된 필드 저장, 성공 시 Toast "조직 정보가 업데이트되었습니다."
- 상태 변경: Active ↔ Suspended 전환, 확인 Dialog 표시
- 조직 삭제: Soft Delete, 확인 Dialog 후 실행, 삭제 후 목록 페이지로 이동

#### 4.2.3.3 Delete Flow

1. "조직 삭제" 버튼 클릭
2. 확인 Dialog 표시:
   - 제목: "조직 삭제"
   - 본문: "'{조직명}' 조직을 삭제하시겠습니까? 소속 프로젝트, 화면, 코멘트 데이터는 soft delete 처리됩니다."
   - 추가 확인: 조직명 직접 입력 필드 (입력값이 조직명과 일치해야 삭제 버튼 활성화)
   - 버튼: 취소(Secondary) / 삭제(Danger/빨강)
3. Soft Delete 실행: `deleted_at` 타임스탬프 기록, 목록에서 제외 (필터로 복원 조회 가능)
4. 성공 시: Toast "조직이 삭제되었습니다." + `/admin/organizations`로 리다이렉트

#### 4.2.3.4 탭 구성

| 탭 | 내용 |
|----|------|
| 멤버 | 소속 사용자 목록 (이름, 이메일, 역할, 가입일, 상태) |
| 프로젝트 | 소속 프로젝트 목록 (프로젝트명, 상태, 생성일, 스크린 수) |
| 빌링 | Plan 정보, 구독 상태, 결제 이력 (Stripe 연동 시 확장) |
| 감사 로그 | 해당 조직 관련 액션 로그 (최근 50건, `/admin/audit-logs` 링크 제공) |

**멤버 탭:**
- 역할 필터: All / Owner / Admin / Member / Client
- 각 멤버 행의 액션: 역할 변경 / 계정 정지 / 조직에서 제거
- 역할 변경 시 확인 Dialog 표시

**프로젝트 탭:**
- 각 프로젝트 행 클릭 시 `/admin/projects/:id` 상세로 이동

---

### 4.2.4 `/admin/users` — 사용자 관리 목록

#### Overview

플랫폼에 가입된 모든 사용자를 조회·관리하는 페이지입니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 페이지네이션 미구현 및 정렬 오류 | 6회 | 목록 API는 page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건 전역 상태 보관, 필터 초기화 버튼 제공 |
| 검색 기능 미동작 | 6회 | 검색 입력에 300ms 디바운스, ILIKE 부분 일치 지원 |
| 관리자 벌크/단일 액션 오류 | 3회 | 어드민 API에 RBAC 적용, 모든 상태 전환 API 사전 구현 |

#### 4.2.4.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 이름, 이메일로 검색. 300ms 디바운스 |
| 필터 | Status Dropdown | All / Active / Suspended |
| 필터 | Role Dropdown | All / Owner / Admin / Member / Client |
| 필터 | 조직 Dropdown | All + 조직명 목록 (검색 가능 Select) |
| 필터 | 가입일 Date Range | YYYY-MM-DD ~ YYYY-MM-DD |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |
| Bulk Action | Dropdown | 선택된 행에 일괄 적용 (정지, 삭제) |

> 슈퍼어드민 페이지에서는 신규 사용자를 직접 생성하지 않습니다. 사용자는 회원가입 또는 조직 초대를 통해 생성됩니다. (Create 버튼 없음)

> **상태 enum 통일:** 사용자 상태는 `active` / `suspended` 두 가지만 사용합니다. 별도의 `inactive` 상태는 존재하지 않으며, Section 1의 enum 정의와 일치합니다.

#### 4.2.4.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| Checkbox | Boolean | - | 전체 선택/해제 |
| 이름 | String | ✓ | 상세 페이지 링크, 프로필 아바타 포함 |
| 이메일 | String | ✓ | |
| 소속 조직 | String | ✓ | 조직명, 클릭 시 조직 상세로 이동 |
| 역할 | Enum Badge | ✓ | Owner / Admin / Member / Client |
| 상태 | Enum Badge | ✓ | Active(녹) / Suspended(빨) |
| 가입일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 마지막 로그인 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 액션 | - | - | 상세보기 / 상태변경 / 삭제 |

**Table State 처리:**
- Loading State: 스켈레톤 로딩
- Empty State: "등록된 사용자가 없습니다."
- Error State: 재시도 버튼 포함 오류 메시지
- Search No Results: "검색 결과가 없습니다."

#### 4.2.4.3 Pagination

- 기본 페이지 크기: 20건
- 페이지 크기 선택: 20 / 50 / 100
- URL query parameter로 상태 유지

#### 4.2.4.4 Bulk Action

| 액션 | 확인 Dialog |
|------|------------|
| 정지(Suspend) | "N명의 사용자를 정지하시겠습니까? 즉시 로그인이 차단됩니다." |
| 삭제 | "N명의 사용자를 삭제하시겠습니까? (Soft Delete)" |

#### 4.2.4.5 Export

- CSV / Excel 다운로드
- 내보내기 컬럼: 이름, 이메일, 소속 조직, 역할, 상태, 가입일, 마지막 로그인, 삭제일
- Date Range 선택 가능 (가입일 기준)
- 현재 필터 조건 적용 결과 전체 내보내기

---

### 4.2.5 `/admin/users/:id` — 사용자 상세

#### Overview

특정 사용자의 상세 정보, 소속 조직/역할, 활동 내역을 조회하고 계정 상태를 관리합니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 관리자 벌크/단일 액션 오류 | 3회 | 모든 상태 전환 API 사전 구현/테스트 |
| API 500 에러 | 2회 | UPDATE는 partial update 지원, null 필드 무시 |

#### 4.2.5.1 Breadcrumb

Admin > Users > [사용자 이름]

#### 4.2.5.2 Detail Section — 사용자 기본 정보

| 필드 | 타입 | 표시 방식 |
|------|------|-----------|
| 프로필 이미지 | Image | 읽기 전용 |
| 이름 | String | 읽기 전용 |
| 이메일 | String | 읽기 전용 |
| 상태 | Enum Badge | Dropdown 편집 가능 (Active / Suspended) |
| 가입일 | DateTime | YYYY-MM-DD HH:mm, 읽기 전용 |
| 마지막 로그인 | DateTime | YYYY-MM-DD HH:mm + 상대시간, 읽기 전용 |
| 삭제일 | DateTime | soft delete 시에만 표시 |

**가능한 액션:**
- 상태 변경 (Active / Suspended): 확인 Dialog 후 실행
- 계정 삭제 (Soft Delete): 확인 Dialog (이메일 직접 입력 재확인), 삭제 후 목록으로 이동
- 비밀번호 초기화 메일 발송: 확인 Dialog, 성공 시 Toast "비밀번호 재설정 이메일을 발송했습니다."

#### 4.2.5.3 Delete Flow

1. "계정 삭제" 버튼 클릭
2. 확인 Dialog:
   - 제목: "계정 삭제"
   - 본문: "'{이름}({이메일})' 계정을 삭제하시겠습니까? 해당 사용자의 모든 데이터는 soft delete 처리됩니다."
   - 추가 확인 입력: 이메일 직접 입력 (일치 시 삭제 버튼 활성화)
   - 버튼: 취소 / 삭제(Danger)
3. Soft Delete: `deleted_at` 기록, 해당 사용자 세션 즉시 만료
4. 성공 시: Toast "계정이 삭제되었습니다." + `/admin/users` 이동

#### 4.2.5.4 탭 구성

| 탭 | 내용 |
|----|------|
| 소속 조직 | 사용자가 속한 조직 목록 (조직명, 역할, 가입일) |
| 활동 로그 | 해당 사용자 관련 감사 로그 (최근 50건) |
| 프로젝트 | 참여 중인 프로젝트 목록 (프로젝트명, 조직명, 역할) |

---

### 4.2.6 `/admin/projects` — 프로젝트 관리 목록

#### Overview

플랫폼 전체 조직의 모든 프로젝트를 조회·관리하는 페이지입니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 페이지네이션 미구현 및 정렬 오류 | 6회 | page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건 전역 상태 보관 |
| 검색 기능 미동작 | 6회 | 300ms 디바운스, ILIKE 부분 일치 |

#### 4.2.6.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 프로젝트명으로 검색. 300ms 디바운스 |
| 필터 | 조직 Dropdown | All + 조직명 (검색 가능 Select) |
| 필터 | Status Dropdown | All / Active / Archived / Deleted |
| 필터 | 생성일 Date Range | YYYY-MM-DD ~ YYYY-MM-DD |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |
| Bulk Action | Dropdown | 아카이브, 삭제 |

> 슈퍼어드민은 프로젝트를 직접 생성하지 않습니다. (Create 버튼 없음)

#### 4.2.6.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| Checkbox | Boolean | - | |
| 프로젝트명 | String | ✓ | `/admin/projects/:id` 상세 링크 |
| 소속 조직 | String | ✓ | 조직 상세로 이동 링크 |
| 상태 | Enum Badge | ✓ | Active(녹) / Archived(회) / Deleted(빨) |
| 스크린 수 | Number | ✓ | |
| 멤버 수 | Number | ✓ | |
| 코멘트 수 | Number | ✓ | |
| 생성일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 최종 수정일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 액션 | - | - | 상세보기 / 아카이브 / 삭제 |

**Table State 처리:**
- Loading / Empty / Error / No Results 상태 모두 처리

#### 4.2.6.3 Pagination

- 기본 페이지 크기: 20건
- URL query parameter 상태 유지

#### 4.2.6.4 Bulk Action

| 액션 | 확인 Dialog |
|------|------------|
| 아카이브 | "N개 프로젝트를 아카이브하시겠습니까? 조직 내 접근이 제한됩니다." |
| 삭제 | "N개 프로젝트를 삭제하시겠습니까? (Soft Delete)" |

#### 4.2.6.5 Export

- CSV / Excel 다운로드
- 내보내기 컬럼: 프로젝트명, 소속 조직, 상태, 스크린 수, 멤버 수, 코멘트 수, 생성일, 삭제일
- 현재 필터 조건 적용 결과 전체 내보내기

---

### 4.2.7 `/admin/projects/:id` — 프로젝트 상세

#### Overview

특정 프로젝트의 상세 정보, 소속 스크린, 멤버, 코멘트 현황을 조회하고 상태를 관리하는 페이지입니다. 슈퍼어드민은 모든 조직의 프로젝트에 읽기 + 상태 관리 권한을 가집니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 관리자 벌크/단일 액션 오류 | 3회 | 모든 상태 전환 API 사전 구현/테스트, RBAC 적용 |
| API 500 에러 | 2회 | UPDATE 엔드포인트는 partial update 지원, null 필드 무시 |

#### 4.2.7.1 Breadcrumb

Admin > Projects > [프로젝트명]

#### 4.2.7.2 Detail Section — 프로젝트 기본 정보

| 필드 | 타입 | 표시 방식 |
|------|------|-----------|
| 프로젝트명 | String | 읽기 전용 |
| 소속 조직 | String | 조직 상세 링크 (`/admin/organizations/:id`) |
| 상태 | Enum Badge | Dropdown 편집 가능 (Active / Archived) |
| 설명 | Text | 읽기 전용 |
| 스크린 수 | Number | 읽기 전용 |
| 멤버 수 | Number | 읽기 전용 |
| 코멘트 수 | Number | 읽기 전용 |
| 생성일 | DateTime | YYYY-MM-DD HH:mm + 상대시간, 읽기 전용 |
| 최종 수정일 | DateTime | YYYY-MM-DD HH:mm + 상대시간, 읽기 전용 |
| 삭제일 | DateTime | soft delete 시에만 표시, YYYY-MM-DD HH:mm |

**가능한 액션 (상단 우측 버튼 그룹):**
- 상태 변경: Active ↔ Archived 전환, 확인 Dialog 표시
- 프로젝트 삭제: Soft Delete, 확인 Dialog (프로젝트명 직접 입력 재확인) 후 실행, 삭제 후 목록 페이지로 이동

#### 4.2.7.3 Delete Flow

1. "프로젝트 삭제" 버튼 클릭
2. 확인 Dialog:
   - 제목: "프로젝트 삭제"
   - 본문: "'{프로젝트명}' 프로젝트를 삭제하시겠습니까? 소속 화면, 코멘트 데이터는 soft delete 처리됩니다."
   - 추가 확인 입력: 프로젝트명 직접 입력 (일치 시 삭제 버튼 활성화)
   - 버튼: 취소(Secondary) / 삭제(Danger/빨강)
3. Soft Delete: `deleted_at` 타임스탬프 기록, 목록에서 제외
4. 성공 시: Toast "프로젝트가 삭제되었습니다." + `/admin/projects`로 리다이렉트

#### 4.2.7.4 탭 구성

| 탭 | 내용 |
|----|------|
| 스크린 | 소속 화면(Screen) 목록 (화면명, 생성일, 코멘트 수) |
| 멤버 | 프로젝트 참여 멤버 목록 (이름, 이메일, 역할, 가입일) |
| 코멘트 | 프로젝트 내 전체 코멘트 목록 (작성자, 상태, 생성일) |
| 감사 로그 | 해당 프로젝트 관련 액션 로그 (최근 50건, `/admin/audit-logs` 링크 제공) |

**스크린 탭:**
- 각 스크린 행 클릭 시 `/admin/screens`로 이동 (해당 스크린 필터 적용)

**코멘트 탭:**
- 각 코멘트 행 클릭 시 `/admin/comments`로 이동 (해당 코멘트 필터 적용)

---

### 4.2.8 `/admin/screens` — 화면 관리 목록

#### Overview

플랫폼 전체 프로젝트의 모든 화면(Screen)을 조회·관리하는 페이지입니다. Screen은 생성/수정/삭제 가능한 엔티티로, 슈퍼어드민은 읽기 전용 조회 및 강제 삭제 권한을 가집니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 페이지네이션 미구현 및 정렬 오류 | 6회 | page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건 전역 상태 보관 |
| 검색 기능 미동작 | 6회 | 300ms 디바운스, ILIKE 부분 일치 |
| 관리자 벌크/단일 액션 오류 | 3회 | 어드민 API에 RBAC 적용, 강제 삭제 API 사전 구현/테스트 |

#### 4.2.8.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 화면명으로 검색. 300ms 디바운스 |
| 필터 | 조직 Dropdown | All + 조직명 (검색 가능 Select) |
| 필터 | 프로젝트 Dropdown | All + 프로젝트명 (조직 선택 시 해당 조직 프로젝트만 표시) |
| 필터 | 생성일 Date Range | YYYY-MM-DD ~ YYYY-MM-DD |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |
| Bulk Action | Dropdown | 강제 삭제 |

> 슈퍼어드민은 화면을 직접 생성하지 않습니다. (Create 버튼 없음)

#### 4.2.8.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| Checkbox | Boolean | - | 전체 선택/해제 |
| 화면명 | String | ✓ | |
| 소속 프로젝트 | String | ✓ | `/admin/projects/:id` 링크 |
| 소속 조직 | String | ✓ | `/admin/organizations/:id` 링크 |
| 코멘트 수 | Number | ✓ | |
| 생성일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 최종 수정일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 액션 | - | - | 상세 조회 / 강제 삭제 |

> **읽기 전용 조회:** 화면 행 클릭 또는 "상세 조회" 액션 클릭 시 Detail Drawer를 통해 화면 상세 정보(화면명, 소속 프로젝트, 생성자, 생성일, 코멘트 목록)를 조회합니다. 어드민은 화면 내용을 수정하지 않습니다.

**Table State 처리:**
- Loading / Empty / Error / No Results 상태 모두 처리

#### 4.2.8.3 Pagination

- 기본 페이지 크기: 20건
- 페이지 크기 선택: 20 / 50 / 100
- URL query parameter 상태 유지

#### 4.2.8.4 Bulk Action — 강제 삭제

| 액션 | 확인 Dialog |
|------|------------|
| 강제 삭제 | "N개 화면을 삭제하시겠습니까? 소속 코멘트 데이터도 soft delete 처리됩니다. 이 작업은 되돌릴 수 없습니다." |

- 0건 선택 시 Bulk Action 버튼 비활성화
- 확인 Dialog 필수 표시 (취소 / 삭제(Danger) 버튼)
- Soft Delete: `deleted_at` 타임스탬프 기록
- 성공 시: Toast "N개 화면이 삭제되었습니다." + 목록 갱신

#### 4.2.8.5 Detail Drawer

화면 행 클릭 시 우측 Drawer 오픈:

| 필드 | 내용 |
|------|------|
| 화면명 | 읽기 전용 |
| 소속 프로젝트 | 프로젝트명 + `/admin/projects/:id` 링크 |
| 소속 조직 | 조직명 + `/admin/organizations/:id` 링크 |
| 생성자 | 이름 + 이메일 + `/admin/users/:id` 링크 |
| 생성일 | YYYY-MM-DD HH:mm:ss |
| 최종 수정일 | YYYY-MM-DD HH:mm:ss |
| 코멘트 수 | 숫자, `/admin/comments` 링크 (해당 화면 필터 적용) |

#### 4.2.8.6 Export

- CSV / Excel 다운로드
- 내보내기 컬럼: 화면명, 소속 프로젝트, 소속 조직, 코멘트 수, 생성일, 삭제일
- 현재 필터 조건 적용 결과 전체 내보내기

---

### 4.2.9 `/admin/comments` — 코멘트 관리 목록

#### Overview

플랫폼 전체의 모든 코멘트를 조회·관리하는 페이지입니다. 신고된 코멘트 관리, 강제 삭제 등의 운영 기능을 제공합니다. Comment는 생성/상태 변경/삭제 가능한 엔티티입니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 페이지네이션 미구현 및 정렬 오류 | 6회 | page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건 전역 상태 보관 |
| 검색 기능 미동작 | 6회 | 300ms 디바운스, ILIKE 부분 일치 |
| 관리자 벌크/단일 액션 오류 | 3회 | 어드민 API에 RBAC 적용, 강제 삭제 API 사전 구현/테스트 |

#### 4.2.9.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 코멘트 내용, 작성자 이름/이메일로 검색. 300ms 디바운스 |
| 필터 | Status Dropdown | All / Open / In Progress / Resolved |
| 필터 | 신고 여부 Dropdown | All / 신고된 코멘트만 |
| 필터 | 조직 Dropdown | All + 조직명 (검색 가능 Select) |
| 필터 | 프로젝트 Dropdown | All + 프로젝트명 |
| 필터 | 생성일 Date Range | YYYY-MM-DD ~ YYYY-MM-DD |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |
| Bulk Action | Dropdown | 강제 삭제 |

> 슈퍼어드민은 코멘트를 직접 생성하지 않습니다. (Create 버튼 없음)

#### 4.2.9.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| Checkbox | Boolean | - | 전체 선택/해제 |
| 코멘트 내용 | String | - | 최대 100자 미리보기, 클릭 시 Detail Drawer |
| 작성자 | String | ✓ | 이름 + 이메일, `/admin/users/:id` 링크 |
| 소속 화면 | String | ✓ | 화면명, `/admin/screens` 링크 |
| 소속 프로젝트 | String | ✓ | 프로젝트명, `/admin/projects/:id` 링크 |
| 소속 조직 | String | ✓ | 조직명, `/admin/organizations/:id` 링크 |
| 상태 | Enum Badge | ✓ | Open(파) / In Progress(노) / Resolved(녹) |
| 신고 | Badge | - | 신고된 경우 "신고됨" 배지 표시 |
| 생성일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 액션 | - | - | 상세 조회 / 강제 삭제 |

**Table State 처리:**
- Loading / Empty / Error / No Results 상태 모두 처리

#### 4.2.9.3 Pagination

- 기본 페이지 크기: 20건
- 페이지 크기 선택: 20 / 50 / 100
- URL query parameter 상태 유지

#### 4.2.9.4 Bulk Action — 강제 삭제

| 액션 | 확인 Dialog |
|------|------------|
| 강제 삭제 | "N개 코멘트를 삭제하시겠습니까? 이 작업은 되돌릴 수 없습니다. (Soft Delete)" |

- 0건 선택 시 Bulk Action 버튼 비활성화
- 확인 Dialog 필수 표시 (취소 / 삭제(Danger) 버튼)
- Soft Delete: `deleted_at` 타임스탬프 기록
- 성공 시: Toast "N개 코멘트가 삭제되었습니다." + 목록 갱신

#### 4.2.9.5 Detail Drawer

코멘트 행 클릭 시 우측 Drawer 오픈:

| 필드 | 내용 |
|------|------|
| 코멘트 전문 | 전체 내용 표시 (읽기 전용) |
| 작성자 | 이름, 이메일, `/admin/users/:id` 링크 |
| 소속 화면 | 화면명, `/admin/screens` 링크 |
| 소속 프로젝트 | 프로젝트명, `/admin/projects/:id` 링크 |
| 소속 조직 | 조직명, `/admin/organizations/:id` 링크 |
| 상태 | Enum Badge |
| 신고 여부 | 신고된 경우 신고 사유 표시 |
| 생성일 | YYYY-MM-DD HH:mm:ss |
| 최종 수정일 | YYYY-MM-DD HH:mm:ss |

**Drawer 내 액션:**
- 강제 삭제: Soft Delete, 확인 Dialog 후 실행

#### 4.2.9.6 Export

- CSV / Excel 다운로드
- 내보내기 컬럼: 코멘트 내용, 작성자 이름, 작성자 이메일, 소속 화면, 소속 프로젝트, 소속 조직, 상태, 신고 여부, 생성일, 삭제일
- 현재 필터 조건 적용 결과 전체 내보내기

---

### 4.2.10 `/admin/invitations` — 초대 관리 목록

#### Overview

플랫폼 전체의 모든 조직 초대(Invitation)를 조회·관리하는 페이지입니다. Invitation은 생성/재전송/취소 가능한 엔티티로, 슈퍼어드민은 전체 초대 현황을 조회하고 필요 시 취소할 수 있습니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 페이지네이션 미구현 및 정렬 오류 | 6회 | page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건 전역 상태 보관 |
| 검색 기능 미동작 | 6회 | 300ms 디바운스, ILIKE 부분 일치 |
| 관리자 벌크/단일 액션 오류 | 3회 | 어드민 API에 RBAC 적용, 초대 취소 API 사전 구현/테스트 |

#### 4.2.10.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 초대 이메일, 초대한 사용자 이름으로 검색. 300ms 디바운스 |
| 필터 | Status Dropdown | All / Pending / Accepted / Expired / Cancelled |
| 필터 | 조직 Dropdown | All + 조직명 (검색 가능 Select) |
| 필터 | 생성일 Date Range | YYYY-MM-DD ~ YYYY-MM-DD |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |
| Bulk Action | Dropdown | 초대 취소 |

> 슈퍼어드민은 초대를 직접 생성하지 않습니다. 초대는 조직 Owner/Admin이 생성합니다. (Create 버튼 없음)

#### 4.2.10.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| Checkbox | Boolean | - | 전체 선택/해제 |
| 초대 이메일 | String | ✓ | 초대 대상자 이메일 |
| 초대한 사용자 | String | ✓ | 이름 + 이메일, `/admin/users/:id` 링크 |
| 소속 조직 | String | ✓ | 조직명, `/admin/organizations/:id` 링크 |
| 초대 역할 | Enum Badge | ✓ | Admin / Member / Client |
| 상태 | Enum Badge | ✓ | Pending(노) / Accepted(녹) / Expired(회) / Cancelled(빨) |
| 만료일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 생성일 | DateTime | ✓ | YYYY-MM-DD HH:mm + 상대시간 |
| 액션 | - | - | 상세 조회 / 초대 취소 |

**Table State 처리:**
- Loading / Empty / Error / No Results 상태 모두 처리

#### 4.2.10.3 Pagination

- 기본 페이지 크기: 20건
- 페이지 크기 선택: 20 / 50 / 100
- URL query parameter 상태 유지

#### 4.2.10.4 Bulk Action — 초대 취소

| 액션 | 대상 | 확인 Dialog |
|------|------|------------|
| 초대 취소 | 상태가 Pending인 초대만 | "N개 초대를 취소하시겠습니까? 해당 초대 링크는 즉시 무효화됩니다." |

- 0건 선택 시 Bulk Action 버튼 비활성화
- Accepted / Expired 상태의 초대는 취소 불가 (선택 시 경고 메시지 표시)
- 확인 Dialog 필수 표시 (취소 / 확인(Danger) 버튼)
- 성공 시: Toast "N개 초대가 취소되었습니다." + 목록 갱신

#### 4.2.10.5 Detail Drawer

초대 행 클릭 시 우측 Drawer 오픈:

| 필드 | 내용 |
|------|------|
| 초대 이메일 | 읽기 전용 |
| 초대한 사용자 | 이름, 이메일, `/admin/users/:id` 링크 |
| 소속 조직 | 조직명, `/admin/organizations/:id` 링크 |
| 초대 역할 | Enum Badge |
| 상태 | Enum Badge |
| 만료일 | YYYY-MM-DD HH:mm:ss |
| 생성일 | YYYY-MM-DD HH:mm:ss |
| 수락일 | Accepted 상태인 경우 YYYY-MM-DD HH:mm:ss |

**Drawer 내 액션:**
- 초대 취소: Pending 상태인 경우에만 활성화, 확인 Dialog 후 실행

#### 4.2.10.6 Export

- CSV / Excel 다운로드
- 내보내기 컬럼: 초대 이메일, 초대한 사용자, 소속 조직, 초대 역할, 상태, 만료일, 생성일, 수락일
- 현재 필터 조건 적용 결과 전체 내보내기

---

### 4.2.11 `/admin/audit-logs` — 감사 로그

#### Overview

플랫폼 전체에서 발생하는 주요 이벤트(생성, 수정, 삭제, 상태 변경, 로그인 등)의 감사 로그를 조회합니다. 로그는 읽기 전용이며 수정/삭제 불가합니다.

> 감사 로그 보존 기간: 1년 보존 후 Cold Storage 아카이빙 [📌 권장적용됨]

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 페이지네이션 미구현 및 정렬 오류 | 6회 | page/page_size 파라미터 지원, 기본 정렬 created_at DESC |
| 필터 적용 후 자동 초기화 | 5회 | 필터 조건 전역 상태 보관 |
| 검색 기능 미동작 | 6회 | 300ms 디바운스, ILIKE 부분 일치 |

#### 4.2.11.1 Top Area

| 영역 | 구성요소 | 상세 |
|------|---------|------|
| 검색 | 검색창 | 액터 이름, 이메일, 대상 리소스명으로 검색. 300ms 디바운스 |
| 필터 | 이벤트 타입 Dropdown | All / 조직 / 사용자 / 프로젝트 / 화면 / 코멘트 / 인증 / 빌링 |
| 필터 | 액터 Dropdown | All + 사용자 검색 |
| 필터 | 조직 Dropdown | All + 조직명 |
| 필터 | 날짜 범위 Date Range | YYYY-MM-DD ~ YYYY-MM-DD (필수 기간 제한 권장) |
| 필터 초기화 | 버튼 | 모든 필터 초기화 |

> Create / Bulk Action 없음 (읽기 전용)

#### 4.2.11.2 Table Component

| 컬럼명 | 타입 | 정렬 | 비고 |
|--------|------|------|------|
| 발생일시 | DateTime | ✓ | YYYY-MM-DD HH:mm:ss + 상대시간 |
| 이벤트 타입 | Enum Badge | ✓ | CREATE / UPDATE / DELETE / LOGIN / LOGOUT / STATUS_CHANGE 등 |
| 액터 | String | ✓ | 수행자 이름 + 이메일, 사용자 상세 링크 |
| 대상 리소스 | String | ✓ | 리소스 타입 + 이름 (예: Organization: Acme Corp) |
| 소속 조직 | String | ✓ | 조직 상세 링크 |
| IP 주소 | String | - | 로그인/인증 이벤트에서 표시 |
| 상세 | - | - | 클릭 시 Detail Drawer 오픈 |

**Table State 처리:**
- Loading / Empty / Error / No Results 상태 모두 처리

#### 4.2.11.3 Pagination

- 기본 페이지 크기: 50건 (감사 로그 특성상 더 많은 기본 표시)
- 페이지 크기 선택: 50 / 100 / 200
- URL query parameter 상태 유지

#### 4.2.11.4 Detail Drawer

감사 로그 행 클릭 시 우측 Drawer 오픈:

| 필드 | 내용 |
|------|------|
| 이벤트 타입 | 배지 표시 |
| 발생일시 | YYYY-MM-DD HH:mm:ss |
| 액터 | 이름, 이메일, 역할, 사용자 상세 링크 |
| 대상 리소스 | 타입, ID, 이름, 해당 상세 링크 |
| 소속 조직 | 조직명, 조직 상세 링크 |
| IP 주소 | 로그인 이벤트 시 |
| User Agent | 브라우저/OS 정보 |
| 변경 내용 | Before / After JSON diff (수정 이벤트 시) |

#### 4.2.11.5 Export

- CSV / Excel 다운로드
- 날짜 범위 선택 필수 (내보내기 시 최대 범위: 90일 [📌 권장적용됨])
- 내보내기 컬럼: 발생일시, 이벤트 타입, 액터 이름, 액터 이메일, 대상 리소스, 소속 조직, IP 주소, 변경 내용(JSON)

---

### 4.2.12 `/admin/settings` — 플랫폼 설정

#### Overview

플랫폼 전체에 적용되는 운영 설정을 관리합니다. 설정 변경은 즉시 반영되며 감사 로그에 기록됩니다.

#### 알려진 위험

| 위험 | 빈도 | 예방 스펙 |
|------|------|-----------|
| 폼 유효성 검사 미동작 | 7회 | 모든 입력 필드 실시간 유효성 검사, Submit 버튼은 유효 시에만 활성화 |
| API 500 에러 | 2회 | UPDATE는 partial update 지원, null 필드 무시 |

#### 4.2.12.1 탭 구성

| 탭 | 내용 |
|----|------|
| 일반 설정 | 플랫폼 이름, 지원 이메일, 기본 언어(i18n), 타임존 |
| 사용자/보안 설정 | 비밀번호 정책, 세션 만료 시간, 신규 가입 허용 여부 |
| 파일 업로드 설정 | 최대 파일 크기 (기본 10MB), 허용 파일 형식 (PNG / JPG / WebP / GIF) [📌 권장적용됨] |
| 이메일 설정 | 발신자 이메일, 이메일 서비스 연동 설정 |
| 빌링/플랜 설정 | 기본 플랜, 플랜별 제한값 (프로젝트/멤버 수), Stripe 연동 키: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` (Stripe 대시보드에서 발급) [📌 권장적용됨] |
| 유지보수 | 점검 모드 설정, 배너 메시지, 데이터 정리 작업 |

#### 4.2.12.2 일반 설정 폼

| 필드 | 타입 | 필수 | 검증 규칙 |
|------|------|------|-----------|
| 플랫폼 이름 | Text Input | ✓ | 2~100자 |
| 지원 이메일 | Email Input | ✓ | 유효한 이메일 형식 |
| 기본 언어 | Select | ✓ | EN / KO |
| 기본 타임존 | Select | ✓ | IANA 타임존 목록 |

#### 4.2.12.3 사용자/보안 설정 폼

| 필드 | 타입 | 필수 | 검증 규칙 |
|------|------|------|-----------|
| 비밀번호 최소 길이 | Number Input | ✓ | 6~32 정수 |
| 대문자 필수 여부 | Toggle | - | Boolean |
| 숫자 필수 여부 | Toggle | - | Boolean |
| 특수문자 필수 여부 | Toggle | - | Boolean |
| 세션 만료 시간 | Number Input | ✓ | 분 단위 양수 정수 (최소 15) |
| 신규 가입 허용 | Toggle | ✓ | Boolean (비활성화 시 초대 전용) |

> 비밀번호 규칙: 최소 8자, 영문+숫자+특수문자 포함 [📌 권장적용됨]

#### 4.2.12.4 파일 업로드 설정 폼

| 필드 | 타입 | 필수 | 검증 규칙 |
|------|------|------|-----------|
| 최대 파일 크기 (MB) | Number Input | ✓ | 1~100 정수 |
| 허용 이미지 포맷 | Checkbox Group | ✓ | PNG / JPG / WebP / GIF (최소 1개 선택) |
| 허용 첨부파일 포맷 | Checkbox Group | - | PDF / DOC / DOCX / XLS / XLSX / ZIP 등 |

> 파일 업로드 기본값: 최대 10MB, 이미지 포맷 PNG / JPG / WebP / GIF, 첨부파일 포맷 PDF / DOC / DOCX / XLS / XLSX / ZIP [📌 권장적용됨]

#### 4.2.12.5 유지보수 설정

| 기능 | 설명 |
|------|------|
| 점검 모드 활성화 | Toggle 켜면 일반 사용자 접근 차단, 슈퍼어드민만 접속 가능. 활성화 전 확인 Dialog 필수 |
| 점검 배너 메시지 | Textarea, 점검 모드 ON 시 일반 사용자 로그인 화면에 표시 |
| Soft Delete 데이터 정리 | 보존 기간 초과 soft-deleted 레코드 영구 삭제 실행 버튼. 실행 전 영향 건수 미리보기 + 확인 Dialog (보존 기간: 90일 [📌 권장적용됨]) |

#### 4.2.12.6 저장 공통 규칙

- 각 탭의 설정 변경 후 "저장" 버튼 클릭 시 적용 (자동 저장 없음)
- 변경 사항이 있을 경우 탭 이동 시 "저장하지 않은 변경 사항이 있습니다. 계속하시겠습니까?" 경고 Dialog 표시
- 설정 저장 성공 시: Toast "설정이 저장되었습니다."
- 설정 변경은 감사 로그에 자동 기록 (변경 전/후 값 포함)

---

## 4.3 Admin Standards 적용 요약

모든 어드민 페이지에 자동 적용되는 표준 항목입니다.

| 표준 항목 | 적용 범위 |
|-----------|-----------|
| Search (300ms 디바운스) | 목록 페이지 전체 |
| Filters + Filter 초기화 버튼 | 목록 페이지 전체 |
| Column Sorting | 모든 테이블 데이터 컬럼 |
| Checkbox Selection | 모든 목록 테이블 |
| Bulk Actions + 확인 Dialog | 목록 페이지 전체 (읽기 전용 제외) |
| Pagination (URL query 유지) | 목록 페이지 전체 |
| Loading / Empty / Error / No Results State | 모든 테이블 |
| Action Column | 모든 테이블 |
| Detail Drawer/Modal | 상세 조회 가능 모든 페이지 |
| Delete Confirmation Dialog + 재확인 입력 | 모든 삭제 액션 |
| Soft Delete (deleted_at 기록) | 모든 삭제 대상 |
| Toast Notifications | 모든 CUD 성공/실패 |
| Breadcrumb | 모든 어드민 페이지 |
| CSV/Excel Export + Date Range 선택 | 목록 페이지 전체 |
| Statistics Cards (전후 비교%) | 대시보드 홈 |
| Period Filter | 대시보드 홈 |
| Charts | 대시보드 홈 |
| Recent Activity Feed | 대시보드 홈 |
| Date/Time 표시 형식 (YYYY-MM-DD HH:mm + 상대시간) | 모든 DateTime 필드 |
| i18n (EN/KO) | 모든 UI 텍스트 |
| 필터/페이지/검색 URL query parameter 유지 | 목록 페이지 전체 |

---

## 4.4 미결 항목 (TBD) — Admin 관련

| # | 항목 | 영향 페이지 | 비고 |
|---|------|------------|------|
| TBD-14 | 감사 로그 보존 기간 | `/admin/audit-logs`, `/admin/settings` | 보존 정책 확정 필요 |
| TBD-16 | 슈퍼어드민 접근 방식 | 전체 어드민 라우트 | 별도 URL/별도 로그인 여부 미지정 |
| TBD-10 | Stripe 요금제 구조 | `/admin/organizations/:id` (빌링 탭), `/admin/settings` (빌링 설정) | 요금제 확정 후 Plan 필드 업데이트 |
| TBD-11 | 조직당 프로젝트/멤버 제한 | `/admin/settings` (빌링/플랜 설정) | 플랜별 제한값 확정 필요 |
| TBD-7 | 최대 파일 크기 제한 | `/admin/settings` (파일 업로드 설정) | 기본값 확정 필요 |
| TBD-8 | 지원 이미지 포맷 | `/admin/settings` (파일 업로드 설정) | PNG/JPG/WebP 등 확정 필요 |
| TBD-13 | 데이터 보존 정책 | `/admin/settings` (유지보수) | Soft delete 보존 기간 확정 필요 |
| TBD-17 | 코멘트 신고 기능 상세 | `/admin/comments` | 신고 사유 enum, 신고 임계값, 자동 처리 정책 확정 필요 |
| TBD-18 | 초대 만료 기간 | `/admin/invitations` | 초대 링크 유효 기간(기본값) 확정 필요 |

---

## 5. Tech Stack & Architecture

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
| 핀 좌표를 이미지 크기 대비 백분율(x_pct, y_pct)로 저장 | 절대 픽셀값 저장 시 이미지 렌더링 크기 변화에 따라 핀 위치가 어긋나는 문제 방지. 반응형 뷰어에서 화면 크기와 무관하게 정확한 위치 재현 가능 [📌 권장적용됨] |
| Soft Delete + Audit Log 분리 | 데이터 복구 가능성 확보 및 규정 준수(GDPR 대응 기반 마련). Audit Log는 별도 테이블로 분리하여 주 테이블 성능에 영향 없이 이력 추적 |
| Monorepo (Turborepo) | 프론트엔드/백엔드 간 TypeScript 타입 공유로 API 계약 불일치 방지. 단일 저장소에서 일관된 린팅/빌드 파이프라인 적용 [📌 권장적용됨] |

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
| `SUPERADMIN_EMAIL` | 슈퍼어드민 초기 계정 이메일 (시드 스크립트용) [📌 권장적용됨] |
| `APP_BASE_URL` | 서비스 베이스 URL (초대 링크 생성 시 사용) |
| `NODE_ENV` | 실행 환경 (`development` / `production` / `test`) |

---

## 6. Data Model

## 6.1 Entity List

| Entity | Key Fields | Description |
|--------|-----------|-------------|
| **Organization** | `id` (PK, UUID), `name` (VARCHAR 100, NOT NULL), `slug` (VARCHAR 50, UNIQUE, NOT NULL), `logo_url` (VARCHAR 500, NULL), `logo_key` (VARCHAR 500, NULL), `status` (SMALLINT, NOT NULL, default 1), `plan_id` (FK → Plan), `owner_id` (FK → User), `created_at`, `updated_at`, `deleted_at` | 멀티테넌시의 최상위 단위. `slug`는 URL 식별자. `status`는 OrganizationStatus enum. `logo_url`/`logo_key`는 조직 로고 이미지의 공개 URL 및 스토리지 객체 키 |
| **User** | `id` (PK, UUID), `email` (VARCHAR 255, UNIQUE, NOT NULL), `password_hash` (VARCHAR 255), `name` (VARCHAR 100, NOT NULL), `avatar_url` (VARCHAR 500), `is_super_admin` (BOOLEAN, default false), `locale` (VARCHAR 5, default `en`), `last_login_at`, `created_at`, `updated_at`, `deleted_at` | 플랫폼 전체 사용자. 소셜 로그인 추가 시 `password_hash` NULL 허용 [📌 권장적용됨] |
| **OrganizationMember** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `user_id` (FK → User, NOT NULL), `role` (SMALLINT, NOT NULL), `invited_by` (FK → User), `joined_at`, `created_at`, `updated_at`, `deleted_at` | Organization ↔ User N:N 조인 테이블. `role`은 OrgRole enum (Owner/Admin/Member) |
| **Project** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `name` (VARCHAR 200, NOT NULL), `description` (TEXT), `status` (SMALLINT, NOT NULL, default 1), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 조직 내 단위 프로젝트. `organization_id` 기반 테넌트 격리 |
| **ProjectMember** | `id` (PK, UUID), `project_id` (FK → Project, NOT NULL), `user_id` (FK → User, NOT NULL), `role` (SMALLINT, NOT NULL), `invited_by` (FK → User), `created_at`, `updated_at`, `deleted_at` | Project ↔ User N:N 조인 테이블. Client 역할 포함. `role`은 ProjectRole enum |
| **Screen** | `id` (PK, UUID), `project_id` (FK → Project, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `name` (VARCHAR 200, NOT NULL), `description` (TEXT), `order` (INTEGER, default 0), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 프로젝트 내 개별 화면. `organization_id` 비정규화로 테넌트 격리 쿼리 최적화 |
| **ScreenVersion** | `id` (PK, UUID), `screen_id` (FK → Screen, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `version_number` (INTEGER, NOT NULL), `label` (VARCHAR 100), `image_url` (VARCHAR 500, NOT NULL), `image_key` (VARCHAR 500, NOT NULL), `width` (INTEGER), `height` (INTEGER), `file_size` (BIGINT), `mime_type` (VARCHAR 50), `uploaded_by` (FK → User, NOT NULL), `created_at`, `deleted_at` | 화면의 스크린샷 버전. `version_number`는 screen 내 단조 증가. `image_key`는 S3 객체 키 |
| **Pin** | `id` (PK, UUID), `screen_version_id` (FK → ScreenVersion, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `x_pct` (DECIMAL(5,2), NOT NULL), `y_pct` (DECIMAL(5,2), NOT NULL), `index` (INTEGER, NOT NULL), `status` (SMALLINT, NOT NULL, default 1), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 이미지 위 핀 좌표. `x_pct`/`y_pct`는 이미지 크기 대비 백분율(0.00~100.00)로 저장하여 반응형 뷰어에서 위치 불변 보장 [📌 권장적용됨]. `index`는 핀 번호(표시용) |
| **Comment** | `id` (PK, UUID), `pin_id` (FK → Pin, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `content` (TEXT, NOT NULL), `status` (SMALLINT, NOT NULL, default 1), `created_by` (FK → User, NOT NULL), `resolved_by` (FK → User, NULL), `resolved_at`, `created_at`, `updated_at`, `deleted_at` | 핀에 연결된 최상위 코멘트. `status`는 CommentStatus enum (open/in_progress/resolved) |
| **Reply** | `id` (PK, UUID), `comment_id` (FK → Comment, NOT NULL), `organization_id` (FK → Organization, NOT NULL), `content` (TEXT, NOT NULL), `created_by` (FK → User, NOT NULL), `created_at`, `updated_at`, `deleted_at` | 코멘트 하위 답글. 단일 depth (Reply의 Reply 불허) |
| **FileAttachment** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `comment_id` (FK → Comment, NULL), `reply_id` (FK → Reply, NULL), `uploader_id` (FK → User, NOT NULL), `file_name` (VARCHAR 255, NOT NULL), `file_key` (VARCHAR 500, NOT NULL), `file_url` (VARCHAR 500, NOT NULL), `file_size` (BIGINT, NOT NULL), `mime_type` (VARCHAR 100, NOT NULL), `created_at`, `deleted_at` | 코멘트/답글에 첨부된 파일. `comment_id` 또는 `reply_id` 중 하나만 NOT NULL [📌 권장적용됨] |
| **Invitation** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `project_id` (FK → Project, NULL), `email` (VARCHAR 255, NOT NULL), `role` (SMALLINT, NOT NULL), `token` (VARCHAR 255, UNIQUE, NOT NULL), `status` (SMALLINT, NOT NULL, default 1), `invited_by` (FK → User, NOT NULL), `expires_at` (TIMESTAMPTZ, NOT NULL), `accepted_at`, `created_at`, `updated_at` | 이메일 초대 토큰 관리. `project_id`가 NULL이면 조직 초대, NOT NULL이면 프로젝트 초대 |
| **PasswordResetToken** | `id` (PK, UUID), `user_id` (FK → User, NOT NULL), `token` (VARCHAR 255, UNIQUE, NOT NULL), `expires_at` (TIMESTAMPTZ, NOT NULL), `used_at` (TIMESTAMPTZ, NULL), `created_at` | 비밀번호 재설정 토큰 관리. `token`은 서버에서 생성한 고유 랜덤 문자열(해시 저장 권장). `used_at`이 NULL이 아니면 이미 사용된 토큰으로 간주하여 재사용 불가. 만료(`expires_at`) 및 사용 여부 검증 후 비밀번호 변경 처리 |
| **PlatformSettings** | `id` (PK, UUID), `key` (VARCHAR 100, UNIQUE, NOT NULL), `value` (TEXT, NOT NULL), `description` (VARCHAR 500, NULL), `updated_by` (FK → User, NOT NULL), `created_at`, `updated_at` | 플랫폼 전역 설정 값 저장. 슈퍼어드민 설정 관리 페이지에서 조회/수정. `key`-`value` 구조로 설정 항목 확장에 유연하게 대응 (e.g. `max_orgs_per_user`, `maintenance_mode`, `default_plan_slug` 등) |
| **Notification** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `user_id` (FK → User, NOT NULL), `type` (SMALLINT, NOT NULL), `title` (VARCHAR 255, NOT NULL), `body` (TEXT), `is_read` (BOOLEAN, default false), `ref_entity_type` (VARCHAR 50), `ref_entity_id` (UUID), `created_at`, `read_at` | 인앱 알림. `ref_entity_type`/`ref_entity_id`로 관련 엔티티 참조(다형성 참조) [📌 권장적용됨] |
| **AuditLog** | `id` (PK, UUID), `organization_id` (FK → Organization, NOT NULL), `actor_id` (FK → User, NOT NULL), `action` (VARCHAR 100, NOT NULL), `entity_type` (VARCHAR 50, NOT NULL), `entity_id` (UUID, NOT NULL), `old_value` (JSONB), `new_value` (JSONB), `ip_address` (VARCHAR 45), `user_agent` (VARCHAR 500), `created_at` | 주요 액션 이력. `old_value`/`new_value`는 JSONB로 변경 전후 스냅샷 저장. Soft Delete 미적용(불변 이력) |
| **Plan** | `id` (PK, UUID), `name` (VARCHAR 50, NOT NULL), `slug` (VARCHAR 50, UNIQUE, NOT NULL), `price_monthly` (DECIMAL(10,2)), `price_yearly` (DECIMAL(10,2)), `max_projects` (INTEGER), `max_members` (INTEGER), `max_storage_gb` (INTEGER), `stripe_price_id_monthly` (VARCHAR 100), `stripe_price_id_yearly` (VARCHAR 100), `is_active` (BOOLEAN, default true), `created_at`, `updated_at` | 구독 요금제 정의. `max_*` 필드로 플랜별 제한 설정 [📌 권장적용됨] |
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
- `User.invited_by` → `User` (self-ref, NULL 허용 — 초대로 가입한 경우 초대자 추적) [📌 권장적용됨]

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
| `NotificationType` | `1` = comment_created, `2` = comment_status_changed, `3` = comment_replied, `4` = invitation_received, `5` = version_uploaded, `6` = mention | Notification.type |

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
| Organization | ✅ (`deleted_at` TIMESTAMPTZ) | 90일 후 하드 삭제 [📌 권장적용됨] |
| User | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 (GDPR 대응 고려) [📌 권장적용됨] |
| OrganizationMember | ✅ (`deleted_at` TIMESTAMPTZ) | 즉시 논리 삭제, 30일 후 하드 삭제 |
| Project | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 [📌 권장적용됨] |
| ProjectMember | ✅ (`deleted_at` TIMESTAMPTZ) | 즉시 논리 삭제, 30일 후 하드 삭제 |
| Screen | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| ScreenVersion | ✅ (`deleted_at` TIMESTAMPTZ) | 90일 후 하드 삭제 (S3 파일은 별도 정책으로 관리) |
| Pin | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| Comment | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| Reply | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 |
| FileAttachment | ✅ (`deleted_at` TIMESTAMPTZ) | 30일 후 하드 삭제 (S3 파일 동시 삭제) |
| Invitation | ❌ (status 컬럼으로 상태 관리) | 만료/취소 후 90일 보관 후 하드 삭제 [📌 권장적용됨] |
| PasswordResetToken | ❌ (하드 삭제 허용) | 사용 완료(`used_at` NOT NULL) 또는 만료(`expires_at` 경과) 후 배치 정리 |
| PlatformSettings | ❌ (하드 삭제 또는 값 갱신으로 관리) | 영구 보존 (설정 이력은 AuditLog로 추적) |
| Notification | ❌ (하드 삭제 허용) | 읽은 후 90일 보관, 미읽음은 영구 보존 [📌 권장적용됨] |
| AuditLog | ❌ (불변 이력, 삭제 금지) | 1년 보존 후 Cold Storage(S3 Glacier) 아카이빙 [📌 권장적용됨] |
| Plan | ❌ (`is_active` 플래그로 비활성화) | 영구 보존 (참조 무결성) |
| Subscription | ❌ (status 컬럼으로 상태 관리) | 영구 보존 (결제 이력 규정 준수) |

---

## 7. Permission Matrix

---

### 7.1 Role Hierarchy

```
Guest < Client < Member < Admin < Owner < SuperAdmin
```

| 역할 | 범위 | 설명 |
|------|------|------|
| Guest | 플랫폼 | 미인증 사용자. 공개 페이지(로그인·회원가입·비밀번호 찾기)만 접근 가능 |
| Client | 프로젝트 | 자신에게 배정된 특정 프로젝트에만 접근. 피드백 작성 목적 외부 협력자 |
| Member | 조직 | 조직 소속 내부 구성원. 화면·버전 관리 및 코멘트 상태 처리 담당 |
| Admin | 조직 | 프로젝트·팀원·피드백 전반 관리. 조직/빌링 설정은 불가 |
| Owner | 조직 | 조직 최고 관리자. 빌링·조직 설정·멤버 역할 전체 관리 |
| SuperAdmin | 플랫폼 | 플랫폼 운영자. 모든 조직 데이터 조회·상태 관리 가능. 일반 조직 라우트 미접근 |

**상속 규칙:**
- 상위 역할은 하위 역할의 권한을 모두 포함합니다 (단, SuperAdmin 제외 — 조직 경계를 넘어 동작하므로 별도 계층으로 취급).
- Client는 Member보다 낮은 계층이나, **프로젝트 내 피드백 작성 권한은 Client에게만 부여**되는 Client 전용 권한이 존재합니다.
- SuperAdmin은 일반 조직 라우트(`/dashboard`, `/projects`, `/settings`)에 접근하지 않고 `/admin/*` 라우트만 사용합니다.
- 모든 역할은 **동일 조직(Organization) 내 데이터에만** 접근할 수 있으며, 다른 조직의 리소스는 SuperAdmin 경로를 통해서만 접근됩니다 (멀티테넌시 경계).

---

### 7.2 Ownership Rules

**Own Resource 정의:**
```
resource.created_by === currentUser.id
  또는
resource.user_id === currentUser.id
```

- **Comment / Reply**: `comment.created_by === currentUser.id` 인 경우에만 수정·삭제 가능 (✅own).
- **Pin**: `pin.created_by === currentUser.id` 인 경우에만 수정·삭제 가능 (✅own).
- **Version (Screenshot)**: `version.uploaded_by === currentUser.id` 또는 Admin 이상인 경우 삭제 가능.
- **Screen**: Member 이상이 생성하며, 생성자(✅own) 또는 Admin 이상이 수정·삭제 가능.
- **Profile (User)**: `user.id === currentUser.id` 인 경우에만 수정 가능.
- **Attachment**: 첨부파일은 업로드(Create)와 삭제(Delete)만 가능하며, 수정(Update)은 지원하지 않습니다. 업로드된 첨부파일의 내용을 변경하려면 삭제 후 재업로드해야 합니다.

**Admin Override 규칙:**
- Admin은 자신이 생성하지 않은 Comment·Pin·Reply도 **삭제**할 수 있습니다 (프로젝트 내 피드백 관리 권한).
- Admin은 자신이 생성하지 않은 Screen·Version도 수정·삭제할 수 있습니다.
- Owner는 Admin의 모든 override 권한을 포함하며, 추가로 조직·멤버·빌링을 관리합니다.
- SuperAdmin은 플랫폼 차원에서 Organization·User·Project의 상태(active/suspended/deleted)를 변경·삭제할 수 있습니다. 단, 개별 Comment·Pin 등 콘텐츠 레벨 데이터는 직접 수정하지 않습니다.

**프로젝트 기반 접근 규칙 (Client):**
- Client는 `project_member` 테이블에 자신의 `user_id`가 포함된 프로젝트에만 접근합니다.
- Client에게 배정되지 않은 프로젝트의 화면·코멘트·핀은 API·UI 모두 차단합니다.
- Client는 자신이 배정된 프로젝트 내에서도 **다른 Client의 코멘트를 수정·삭제할 수 없습니다**.

**조직 기반 격리 규칙:**
- Member·Admin·Owner는 자신이 소속된 Organization의 리소스에만 접근합니다.
- `organization_id` 가 다른 리소스에 대한 API 요청은 403으로 거부합니다.

---

### 7.3 Action × Role Matrix

> 표기 범례
> - ✅ = 무조건 허용 (역할 충족 시)
> - ✅own = 본인 리소스(owned resource)에 한해 허용
> - ✅assigned = 배정된 프로젝트에 한해 허용
> - ❌ = 금지

#### 7.3.1 Authentication (인증)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Session | Login | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Session | Logout | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Session | Register (self) | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Session | Password Reset (self) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Session | Password Reset (other) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Session | Accept Invite | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |

#### 7.3.2 Organization (조직)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Organization | Create | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Organization | Read (own org) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Organization | Read (all orgs) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Organization | Update (name/description/logo) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Organization | Update (status: active/suspended/deleted) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Organization | Delete (soft) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Organization | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

> Owner는 본인이 소유한 조직에 한해 소프트 삭제(Delete soft)가 가능합니다.

#### 7.3.3 Billing (빌링/결제)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Billing | Read | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Billing | Update (plan 변경) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Billing | Read (all orgs billing) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

#### 7.3.4 Membership / Team (조직 멤버십)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Membership | Invite (send email invite) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Membership | Read (list members) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Membership | Update (change role) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Membership | Remove (kick member) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Membership | Leave (self) | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ |

> Owner는 조직의 유일한 Owner인 경우 탈퇴 불가 (최소 1명의 Owner 유지 필수) [📌 권장적용됨]

#### 7.3.5 User / Profile (사용자 계정)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| User | Read (self profile) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| User | Read (all users list) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| User | Read (org members detail) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| User | Update (self profile) | ❌ | ✅own | ✅own | ✅own | ✅own | ✅ |
| User | Update (password, self) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| User | Update (account status) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| User | Delete (soft, self) | ❌ | ✅own | ✅own | ✅own | ✅own | ✅ |
| User | Delete (soft, any) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| User | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

> `/settings/profile`에서 인증된 모든 사용자(Client·Member·Admin·Owner)는 본인 비밀번호를 변경할 수 있습니다.

#### 7.3.6 Project (프로젝트)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Project | Create | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Project | Read (list, own org) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Project | Read (assigned projects only) | ❌ | ✅assigned | ❌ | ❌ | ❌ | ❌ |
| Project | Read (all orgs, platform-wide) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Project | Update (name/settings) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Project | Archive | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Project | Delete (soft) | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Project | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

#### 7.3.7 Project Membership (프로젝트 멤버 배정)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| ProjectMember | Add (invite to project) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| ProjectMember | Read (project member list) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ✅ |
| ProjectMember | Update (role in project) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| ProjectMember | Remove (from project) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |

#### 7.3.8 Screen (화면)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Screen | Create | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Screen | Read (list) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Screen | Read (detail / feedback viewer) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Screen | Update (name/settings) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |
| Screen | Delete (soft) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |

#### 7.3.9 Version / Screenshot (스크린샷 버전)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Version | Upload (create) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Version | Read (list versions) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Version | Read (version detail / compare) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Version | Update (metadata) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |
| Version | Delete (soft) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |

#### 7.3.10 Pin (핀)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Pin | Create (place pin on image) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Pin | Read | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Pin | Update (reposition) | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |
| Pin | Delete | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

#### 7.3.11 Comment (코멘트)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Comment | Create | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Comment | Read | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Comment | Update (content) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Comment | Update (status: open→in_progress→resolved) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Comment | Delete (soft) | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

> Client는 코멘트 내용은 본인 것만 수정 가능하며, 코멘트 상태(open/in_progress/resolved) 변경은 불가합니다.
> Admin은 타인의 코멘트도 삭제 가능합니다 (Admin override).

#### 7.3.12 Reply (답글)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Reply | Create | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Reply | Read | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Reply | Update (content) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Reply | Delete (soft) | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

#### 7.3.13 Comment Attachment (파일 첨부)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Attachment | Upload (on comment) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Attachment | Read / Download | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Attachment | Delete | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

> Attachment Update(수정) 행은 의도적으로 제공하지 않습니다. 첨부파일은 업로드(Create)와 삭제(Delete)만 가능하며, 내용 변경이 필요한 경우 삭제 후 재업로드해야 합니다. (참고: 7.2 Ownership Rules)

#### 7.3.14 Invitation (초대)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Invitation | Send (org-level) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Invitation | Send (project-level) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Invitation | Read (pending list) | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Invitation | Revoke | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Invitation | Accept (via email link) | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |

#### 7.3.15 Notification (알림)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Notification | Read (own) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Notification | Mark as read (own) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Notification | Delete (own) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |

#### 7.3.16 Audit Log (감사 로그)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| AuditLog | Read (own org logs) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| AuditLog | Read (all platform logs) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| AuditLog | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| AuditLog | Delete | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

> 감사 로그는 어떤 역할도 삭제할 수 없습니다 (불변 기록). [📌 권장적용됨]

#### 7.3.17 Platform Settings (플랫폼 설정)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| PlatformSettings | Read | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Update (general/security/email) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Update (file upload settings) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Update (billing/plan settings) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Toggle Maintenance Mode | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Run Data Purge | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

#### 7.3.18 Statistics / Dashboard Data (통계)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Stats | Read (own org overview) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Stats | Read (platform-wide statistics) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

### 7.4 멀티테넌시 격리 요약

| 규칙 | 적용 대상 |
|------|-----------|
| 모든 API 요청에 `organization_id` 스코프 적용 | Member·Admin·Owner |
| 다른 org 리소스 접근 시 403 반환 | 전 역할 (SuperAdmin 제외) |
| Client는 `project_member` 테이블 배정 여부로 추가 필터링 | Client |
| SuperAdmin은 org 경계를 넘어 접근 가능하나, `/admin/*` 라우트에서만 동작 | SuperAdmin |
| Soft delete된 리소스는 해당 org 내에서도 일반 조회 API에서 제외 (필터로만 복원 조회 가능) | 전 역할 |

---

## 8. Open Questions

| # | Question | Context / Impact | Owner | Status |
|:-:|----------|-----------------|-------|--------|
| OQ-1 | 스크린샷 버전 업로드 최대 파일 크기 확정 | Section 2 Module 4에서 20MB로 권장, Section 3 UI에서는 일부 10MB 언급. 통합 문서에서 20MB로 통일했으나 클라이언트 확인 필요 | Client | Open |
| OQ-2 | 코멘트 첨부파일 최대 크기 확정 | Module 10에서 10MB 권장. 스크린샷(20MB)과 별도 제한인지 확인 필요 | Client | Open |
| OQ-3 | 비밀번호 정책 세부 규칙 확정 | 최소 8자, 대소문자+숫자+특수문자 조합으로 권장 중이나 클라이언트 최종 확인 필요 | Client | Open |
| OQ-4 | 지원 이미지 포맷 확정 | PNG/JPG/WebP/GIF 중 GIF 포함 여부 확인 필요 | Client | Open |
| OQ-5 | Stripe 요금제 구조 (Free/Pro/Enterprise) 세부 제한값 | max_projects, max_members, max_storage_gb 각 플랜별 구체 값 미정 | Client | Open |
| OQ-6 | 조직당 프로젝트/멤버 수 제한 정책 | Phase 2 Billing 연동 후 적용 예정이나, 무료 플랜 기본 제한 유무 확인 필요 | Client | Open |
| OQ-7 | 감사 로그 보존 기간 | 1년 권장 중이나 최종 확인 필요. Cold Storage 아카이빙 정책도 미정 | Client | Open |
| OQ-8 | Soft Delete 데이터 보존 기간 | 엔티티별 30일/90일 권장 중이나 최종 확인 필요 | Client | Open |
| OQ-9 | 슈퍼어드민 접근 방식 | 별도 URL/별도 로그인 vs 동일 앱 내 `/admin` 경로 분리 | Client | Open |
| OQ-10 | 코멘트 신고 기능 상세 | Section 4 어드민 코멘트 관리에 "신고됨" 필터가 있으나, 신고 기능 자체의 MVP 포함 여부 미정 | Client | Open |
| OQ-11 | 초대 링크 유효 기간 기본값 | 7일로 권장 중이나 클라이언트 확인 필요 | Client | Open |
| OQ-12 | 앱 이름 확정 | "DesignPin"은 권장 이름. 클라이언트 최종 확인 필요 | Client | Open |
| OQ-13 | 프로젝트 마감일 | 전체 프로젝트 마감일 미지정 | Client | Open |
| OQ-14 | 데이터 내보내기(CSV/PDF) 범위 | Phase 2로 분류했으나, 어드민 Export 기능은 MVP에 포함. 일반 사용자 Export는 Phase 2 확정 필요 | Client | Open |

---

# Additional Questions (Client Confirmation Required)

## Required Clarifications

| # | Question | Context |
|:-:|----------|---------|
| RQ-1 | 스크린샷 최대 파일 크기를 20MB로 확정해도 되는가? | Section 2에서 20MB, Section 3 일부에서 10MB 언급. 서버 비용/UX 트레이드오프 존재 |
| RQ-2 | 코멘트 첨부파일 최대 크기를 10MB로 확정해도 되는가? | Module 10에서 10MB 권장 |
| RQ-3 | 무료 플랜(Free)에서 프로젝트/멤버/저장소 제한이 있는가? | Billing 모듈 Phase 2이지만, MVP에서도 기본 제한 적용 여부 결정 필요 |
| RQ-4 | 코멘트 신고(Report) 기능은 MVP에 포함되는가? | Section 4 어드민 페이지에 신고 필터가 존재하나, 사용자 앱에서 신고 UI가 정의되지 않음 |
| RQ-5 | SuperAdmin 계정은 별도 시드 스크립트로 생성하는가, 회원가입 경로가 존재하는가? | 보안상 시드 스크립트 권장 |

## Recommended Clarifications

| # | Question | Context |
|:-:|----------|---------|
| RC-1 | 이미지 포맷에 GIF를 포함할 것인가? | 애니메이션 GIF의 경우 핀 좌표 처리 복잡도 증가 |
| RC-2 | 코멘트 멘션(@) 기능의 상세 동작은? | notification_type에 `mention`이 포함되어 있으나 UI 상세가 미정 |
| RC-3 | 브랜드 가이드라인(색상, 로고, 폰트)은 언제 제공되는가? | 디자인 착수 전 확정 권장 |
| RC-4 | 모바일 반응형의 최소 지원 해상도는? | 데스크탑 우선이나 태블릿/모바일 breakpoint 확정 필요 |

---

# Feature Change Log

## Version 1.0 (2026-03-12)

| Change Type | Risk | Before | After | Source |
|-------------|------|--------|-------|--------|
| 용어 통일 | Low | Section별 `Version` / `ScreenVersion` 혼용 | `ScreenVersion` (엔티티명), `버전` (일반 용어)으로 통일 | 일관성 감사 |
| 용어 통일 | Low | `notification_type` enum: Section 1은 `comment_created` / Section 6은 `comment_added` | `comment_created`, `comment_replied`, `comment_status_changed`, `invitation_received`, `version_uploaded`, `mention`으로 통일 | 일관성 감사 |
| 상태 enum 통일 | Low | Section 7에서 Organization 상태를 `active/inactive/suspended`로 기술 | Section 1 정의에 맞춰 `active/suspended/deleted`로 통일 | 일관성 감사 |
| 파일 크기 통일 | Low | 스크린샷 업로드 최대 크기: Section 2(20MB) vs Section 3 일부(10MB) | 20MB로 통일 (Open Question OQ-1로 클라이언트 확인 등록) | 일관성 감사 |
| 첨부파일 크기 통일 | Low | 코멘트 첨부파일 크기: Section 3(5MB) vs Section 2 Module 10(10MB) | 10MB로 통일 (Open Question OQ-2로 클라이언트 확인 등록) | 일관성 감사 |
| SuperAdmin 경로 통일 | Low | Section 0에서 `/superadmin`, Section 4에서 `/admin` 사용 | `/admin`으로 통일 (Section 4의 상세 정의 기준) | 일관성 감사 |
| 핀 좌표 필드명 통일 | Low | Section 2에서 `x, y`, Section 6에서 `x_pct, y_pct` | `x_pct, y_pct` (백분율 좌표)로 통일 | 일관성 감사 |
| WebSocket 핀 이벤트 페이로드 통일 | Low | `pin:created` 페이로드에 `x, y` 사용 | `x_pct, y_pct`로 통일 | 일관성 감사 |

---

# Appendix A: 권장적용 항목 요약

| # | 항목 | 권장안 | 근거 | Section |
|:-:|------|--------|------|---------|
| 1 | 앱 이름 | DesignPin | 제품 정체성 및 SEO | 0 |
| 2 | 프로젝트 마감일 | 클라이언트 확인 필요 | 일정 관리 | 0 |
| 3 | 참고 앱 | Figma Comment, Marvel App 참고 | UX 벤치마크 | 0 |
| 4 | 브랜드 가이드라인 | 브랜드 가이드라인 수립 후 반영 | 디자인 일관성 | 0 |
| 5 | 폰트 | Inter (EN) / Pretendard (KO) 조합 | 가독성, SaaS 표준 | 0 |
| 6 | 소셜 로그인 | Phase 2에서 Google/GitHub 추가 | 가입 전환율 향상 | 0 |
| 7 | SuperAdmin 경로 분리 | /admin 경로 + 별도 인증 미들웨어 | 보안 격리 | 1, 4 |
| 8 | Refresh Token | HttpOnly 쿠키 저장 | XSS 방지 | 1 |
| 9 | JWT 인증 | 액세스 토큰 + Refresh Token 기반 | Stateless 인증 표준 | 2 |
| 10 | 비밀번호 규칙 | 최소 8자, 대소문자+숫자+특수문자 | 보안 강화 | 2, 3 |
| 11 | 조직 ID 전달 | X-Organization-Id 헤더 또는 JWT claim | 멀티테넌시 격리 | 2 |
| 12 | 캐스케이드 soft delete | 프로젝트 삭제 시 연관 Screen/Version/Pin/Comment 함께 soft delete | 데이터 무결성 | 2 |
| 13 | 파일 형식 | PNG/JPG/WebP/GIF (스크린샷), PNG/JPG/PDF/ZIP (첨부) | 업종 표준 커버리지 | 2 |
| 14 | 스크린샷 최대 크기 | 20MB | 고해상도 디자인 파일 지원 | 2 |
| 15 | 첨부파일 최대 크기 | 10MB | 업로드 UX/서버 비용 밸런스 | 2 |
| 16 | 핀 좌표 백분율 저장 | x_pct, y_pct (이미지 크기 대비 %) | 반응형 뷰어 핀 위치 불변 | 2, 5 |
| 17 | 코멘트 상태 전환 | open → in_progress → resolved → open (순환) | 피드백 재오픈 시나리오 지원 | 2 |
| 18 | 코멘트/답글 캐스케이드 삭제 | 코멘트 삭제 시 연관 Reply, 첨부파일 soft delete | 데이터 무결성 | 2 |
| 19 | 초대 TTL | 7일 | 보안 + UX 밸런스 | 2 |
| 20 | 자기 자신 역할 변경 불가 | Owner의 자기 역할 변경 차단 | 운영 안전성 | 2 |
| 21 | 멤버 제거 시 세션 처리 | 다음 토큰 갱신 시 403 | 즉시 접근 차단 | 2 |
| 22 | 이메일 알림 on/off | 사용자 설정으로 제어 | 알림 피로도 감소 | 2 |
| 23 | 알림 자기 자신 제외 | 본인 행위에 대한 알림 미발송 | UX 개선 | 2 |
| 24 | 이메일 재시도 큐 | 최대 3회 재시도 | 발송 실패 복구 | 2 |
| 25 | WebSocket Exponential Backoff | 재연결 시 지수 백오프 | 서버 부하 방지 | 2 |
| 26 | 첨부파일 Presigned URL 유효시간 | 15분 | 보안 + UX 밸런스 | 2 |
| 27 | Audit Log 비동기 처리 | 동기 처리 시 응답 지연 방지 | 성능 최적화 | 2 |
| 28 | Audit Log metadata | 변경 전/후 값 JSON 포함 | 변경 이력 추적 | 2 |
| 29 | Audit Log 보존 | 1년 | 규정 준수 | 2 |
| 30 | Stripe Webhook 사전 정의 | checkout.session.completed, invoice.payment_failed | 결제 연동 준비 | 2 |
| 31 | Overlay 기본 투명도 | 50% | UX 기본값 | 2 |
| 32 | Side-by-side 스크롤/줌 동기화 | 좌우 패널 동기화 | 비교 UX | 2 |
| 33 | 비교 뷰 핀 비활성화 | 읽기 전용 | 비교 모드 집중 | 2 |
| 34 | Stack: TanStack Query + Zustand | 서버/클라이언트 상태 분리 | 상태 관리 모범사례 | 3 |
| 35 | 소셜 로그인 버튼 (Google) | Phase 2 준비용 UI | 전환율 향상 | 3 |
| 36 | 로그인 실패 쿨다운 | 5회 연속 실패 시 30초 대기 | 브루트포스 방지 | 3 |
| 37 | 비밀번호 찾기 보안 메시지 통일 | 등록 여부 무관하게 동일 메시지 | 이메일 열거 공격 방지 | 3 |
| 38 | 이메일 전송 쿨다운 | 1분 재전송 방지 | 스팸 방지 | 3 |
| 39 | 프로젝트 목록 페이지네이션 | 페이지당 12개 | UX 최적화 | 3 |
| 40 | 코멘트 무한스크롤 | 20개씩 로드 | 성능 최적화 | 3 |
| 41 | 코멘트 제출 단축키 | Cmd/Ctrl+Enter | 파워 유저 UX | 3 |
| 42 | 초대 최대 동시 인원 | 한 번에 20명 | 대량 초대 지원 | 3 |
| 43 | 조직 로고 최대 크기 | 2MB | 업로드 최적화 | 3 |
| 44 | Billing: Stripe 미연동 시 | "무료 플랜" / "준비 중" 표시 | MVP 대응 | 3 |
| 45 | 알림 자동 정리 | 30일 이상 지난 알림 자동 정리 | 스토리지 최적화 | 3 |
| 46 | Monorepo (Turborepo) | 프론트/백엔드 타입 공유 | API 계약 일관성 | 5 |
| 47 | 소셜 로그인 시 password_hash NULL 허용 | Phase 2 소셜 로그인 대비 | 확장성 | 6 |
| 48 | FileAttachment 다형성 참조 | comment_id 또는 reply_id 중 하나만 NOT NULL | 데이터 무결성 | 6 |
| 49 | Notification 다형성 참조 | ref_entity_type/ref_entity_id로 관련 엔티티 참조 | 유연한 알림 대상 추적 | 6 |
| 50 | Plan max 필드 | max_projects, max_members, max_storage_gb | 플랜별 제한 설정 | 6 |
| 51 | Organization 90일 soft delete 보존 | 90일 후 하드 삭제 | 복구 가능성 확보 | 6 |
| 52 | User 30일 soft delete 보존 | GDPR 대응 고려 | 규정 준수 | 6 |
| 53 | Invitation 90일 보관 | 만료/취소 후 90일 보관 후 하드 삭제 | 이력 추적 | 6 |
| 54 | Notification 90일 보관 | 읽은 후 90일 보관, 미읽음 영구 보존 | 스토리지 최적화 | 6 |
| 55 | AuditLog Cold Storage | 1년 보존 후 S3 Glacier 아카이빙 | 비용 최적화 | 6 |
| 56 | User.invited_by self-ref | 초대로 가입한 경우 초대자 추적 | 성장 분석 | 6 |
| 57 | Owner 최소 1명 유지 | 유일한 Owner인 경우 탈퇴 불가 | 조직 안전성 | 7 |
| 58 | 감사 로그 삭제 불가 | 불변 기록 원칙 | 규정 준수 | 7 |
| 59 | SUPERADMIN_EMAIL 환경변수 | 시드 스크립트용 초기 계정 | 운영 편의 | 5 |

---

# Appendix B: 일관성 감사 결과

## B.1 수치 일관성

| 항목 | Section 2 값 | Section 3 값 | Section 6 값 | 통일 결과 |
|------|-------------|-------------|-------------|-----------|
| 스크린샷 최대 크기 | 20MB (Module 4) | 10MB (일부 UI 검증규칙) | - | **20MB**로 통일 (OQ-1 등록) |
| 코멘트 첨부파일 최대 크기 | 10MB (Module 10) | 5MB (일부 UI 검증규칙) | - | **10MB**로 통일 (OQ-2 등록) |
| 검색 디바운스 | 300ms | 300ms | - | 300ms (일치) |
| 초대 TTL | 7일 | - | - | 7일 (일치) |
| 비밀번호 최소 길이 | 8자 | 8자 | - | 8자 (일치) |
| 페이지 기본 크기 (목록) | 20건 | 12개 (카드), 20명 (테이블) | - | 카드 UI=12, 테이블 UI=20 (용도별 구분, 일치) |
| Presigned URL 유효시간 | 15분 (첨부), 1시간 (스크린샷) | - | - | 15분/1시간 (용도별 구분, 일치) |

## B.2 엔티티 관계(FK) 검증

| FK | Source Entity | Target Entity | 존재 여부 |
|----|--------------|---------------|-----------|
| Organization.plan_id | Organization | Plan | OK |
| Organization.owner_id | Organization | User | OK |
| OrganizationMember.organization_id | OrganizationMember | Organization | OK |
| OrganizationMember.user_id | OrganizationMember | User | OK |
| Project.organization_id | Project | Organization | OK |
| Project.created_by | Project | User | OK |
| ProjectMember.project_id | ProjectMember | Project | OK |
| ProjectMember.user_id | ProjectMember | User | OK |
| Screen.project_id | Screen | Project | OK |
| Screen.organization_id | Screen | Organization | OK (비정규화) |
| ScreenVersion.screen_id | ScreenVersion | Screen | OK |
| Pin.screen_version_id | Pin | ScreenVersion | OK |
| Comment.pin_id | Comment | Pin | OK |
| Reply.comment_id | Reply | Comment | OK |
| FileAttachment.comment_id | FileAttachment | Comment | OK (NULL 가능) |
| FileAttachment.reply_id | FileAttachment | Reply | OK (NULL 가능) |
| Invitation.organization_id | Invitation | Organization | OK |
| Invitation.project_id | Invitation | Project | OK (NULL 가능) |
| PasswordResetToken.user_id | PasswordResetToken | User | OK |
| Notification.user_id | Notification | User | OK |
| AuditLog.actor_id | AuditLog | User | OK |
| Subscription.organization_id | Subscription | Organization | OK |
| Subscription.plan_id | Subscription | Plan | OK |

모든 FK가 실제 존재하는 엔티티를 참조합니다.

## B.3 클라이언트 요구사항 추적

| parsed-input.md 요구사항 | PRD 반영 위치 | 상태 |
|--------------------------|-------------|------|
| 조직 기반 멀티테넌시 | Section 0 (Goals), Section 1 (Terminology), Section 6 (Data Model) | 반영됨 |
| Owner/Admin/Member/Client 4역할 | Section 0 (User Types), Section 1 (User Roles), Section 7 (Permission Matrix) | 반영됨 |
| 조직 생성/관리 (Owner) | Section 2 Module 2, Section 3 `/settings/organization` | 반영됨 |
| 빌링/결제 관리 (Owner) | Section 2 Module 12, Section 3 `/settings/billing` | 반영됨 (Phase 2) |
| 멤버 관리 (Owner) | Section 2 Module 7, Section 3 `/settings/team` | 반영됨 |
| 프로젝트 CRUD (Admin) | Section 2 Module 3, Section 3 `/projects` | 반영됨 |
| 팀원/클라이언트 초대 (Admin) | Section 2 Module 7, Section 3 `/projects/:id/members` | 반영됨 |
| 피드백 관리 (Admin) | Section 2 Module 5, Section 3 피드백 뷰어 | 반영됨 |
| 화면 등록/관리 (Member) | Section 2 Module 4, Section 3 `/projects/:id/screens` | 반영됨 |
| 스크린샷 버전 업로드 (Member) | Section 2 Module 4, Section 3 VersionUploader | 반영됨 |
| 핀 코멘트 답변 (Member) | Section 2 Module 5, Section 3 CommentPanel | 반영됨 |
| 코멘트 상태 변경 (Member) | Section 2 Module 5, Section 7 (7.3.11) | 반영됨 |
| 자기 배정 프로젝트만 열람 (Client) | Section 2 Module 3, Section 7 (7.3.6), Section 3 `/client/projects` | 반영됨 |
| 이미지 위 핀 찍기 + 코멘트 (Client) | Section 2 Module 5, Section 3 Client피드백뷰어 | 반영됨 |
| 코멘트 답글 (Client) | Section 2 Module 5, Section 7 (7.3.12) | 반영됨 |
| 이메일 초대, 알림 | Section 2 Module 7/8, Section 3 `/notifications` | 반영됨 |
| 실시간 코멘트/핀 업데이트 | Section 2 Module 9, WebSocket Events | 반영됨 |
| Stripe 구독결제 (추후) | Section 2 Module 12, MVP Scope (Excluded) | 반영됨 (Phase 2) |
| 피드백 뷰어 (좌측 이미지 + 우측 코멘트) | Section 1 (Terminology), Section 3 피드백 뷰어 | 반영됨 |
| 핀↔코멘트 상호 하이라이트 | Section 3 피드백 뷰어 ImageViewer + CommentPanel | 반영됨 |
| 버전 비교 (overlay / side-by-side) | Section 2 Module 6, Section 3 버전 비교 기능 | 반영됨 |
| Soft delete | Section 1 (Terminology), Section 6 (6.5 Soft Delete) | 반영됨 |
| Audit log | Section 2 Module 11, Section 6 (AuditLog 엔티티) | 반영됨 |
| 파일 첨부 (코멘트) | Section 2 Module 10, Section 6 (FileAttachment 엔티티) | 반영됨 |
| i18n (EN/KO) | Section 0 (Design Reference), Section 5 (next-intl) | 반영됨 |
| 반응형 웹 | Section 0 (Design Reference) | 반영됨 |
| 조직별 데이터 완전 분리 | Section 1 (Multitenancy), Section 6 (organization_id FK), Section 7 (7.4) | 반영됨 |
| Figma 등 확장 가능 구조 | Section 0 (Description, MVP Excluded - Phase 3) | 반영됨 |

모든 클라이언트 명시적 요구사항이 PRD에 구체적 스펙으로 반영되었습니다.

## B.4 필러 감지

권장적용 요약표 및 UX 제안 항목은 Appendix A/C로 분리 완료. PRD 본문에는 `[📌 권장적용됨]` 태그로 인라인 표시만 유지하며, 의사결정은 Phase 7 클라이언트 확인 단계에서 처리됩니다.

---

# Appendix C. UX 플로우 분석

## C.1 Navigation Flow 분석

### 데드엔드 페이지

| 페이지 | 문제 설명 | 다음 단계 부재 이유 |
|--------|----------|-------------------|
| `/auth/reset-password` | 비밀번호 변경 성공 후 `/auth/login`으로만 이동. 성공 상태에서 "대시보드로 바로 로그인" 옵션 없음 | 성공 상태에서 재로그인 강제, UX 단절 |
| `/auth/accept-invite` (토큰 만료) | 만료 안내 화면에 "관리자에게 재초대 요청" 텍스트만 있고, 연락 수단(이메일/링크)이나 명시적 CTA 없음 | 사용자가 앱 내에서 취할 수 있는 다음 행동 없음 |
| `/settings/billing` | Stripe 미연동(MVP) 상태에서 "준비 중" 표시만 있고, 현재 이용 가능한 대안이나 플랜 비교 정보 없음 | 페이지 진입 후 아무것도 할 수 없는 상태 |
| `/notifications` | 알림 항목 클릭 시 해당 뷰어로 이동하나, 알림 목록으로 돌아오는 "뒤로가기" 명시적 네비게이션 없음 | `/dashboard`로만 뒤로가기 명시, 알림 목록 자체에 복귀 경로 불명확 |
| `/client/projects` (빈 상태) | "배정된 프로젝트가 없습니다. 관리자가 초대하면 여기에 표시됩니다" — 클라이언트가 할 수 있는 다음 행동이 전혀 없음 | Client 역할 특성상 수동 대기만 가능하나, 알림 설정이나 지원 요청 경로 없음 |
| `/admin/screens` 상세 Drawer | Drawer 내에서 코멘트 수 클릭 시 `/admin/comments`로 이동하지만, Drawer 닫기 외 내부 네비게이션 없음 | Drawer 내 행동이 강제 삭제 1가지뿐 |

### 깊이 분석

**엔트리 포인트 기준: 로그인(`/auth/login`)**

| 화면 | 클릭 수 | 경로 | 문제점 / 개선 제안 |
|------|--------|------|-------------------|
| 피드백 뷰어 (일반 사용자) | 4클릭 | 로그인 → 대시보드 → 프로젝트 목록 → 화면 목록 → 피드백 뷰어 | 핵심 기능까지 4단계. 대시보드 최근 활동에서 바로 뷰어 진입 가능하지만, 신규 사용자는 항상 4단계 필요 |
| 클라이언트 피드백 뷰어 | 4클릭 | 로그인 → 클라이언트 프로젝트 목록 → 화면 목록 → 피드백 뷰어 | 클라이언트는 대시보드 없이 바로 프로젝트 목록 진입이나 `/client/projects`가 시작점 — 동일 4단계 |
| 프로젝트 멤버 관리 | 3클릭 | 로그인 → 프로젝트 목록 → 프로젝트 상세 → 멤버 탭 | 적정 수준 |
| 버전 비교 | 5클릭 | 로그인 → 프로젝트 목록 → 화면 목록 → 피드백 뷰어 → 버전 드롭다운 → 비교 모드 토글 | 5단계. 버전 히스토리와 비교 기능이 피드백 뷰어에 내장되어 있어 해당 화면에 이미 들어와야만 접근 가능 |

**3단계 이상 깊이의 개선 제안:**
- 대시보드에 "최근 피드백" 바로가기 카드를 고정 노출하여 피드백 뷰어까지 1~2클릭으로 단축
- 화면 목록 카드에서 "피드백 보기" 바로가기 버튼 추가
- 헤더 글로벌 검색을 통해 프로젝트명/화면명 검색 후 피드백 뷰어 직행 경로 추가

### 잦은 전환 필요

| 워크플로우 | 전환 발생 경로 | 빈도 | 설명 |
|------------|--------------|------|------|
| 팀원 초대 + 프로젝트 배정 | Settings > Team 초대 → Projects > Members 배정 | 매 신규 멤버마다 | 조직 레벨 초대와 프로젝트 레벨 배정이 분리되어 두 번 작업 필요 |
| 버전 업로드 후 피드백 확인 | 피드백 뷰어 내 VersionUploader → 버전 드롭다운 선택 → 코멘트 패널 확인 | 버전 업로드마다 | 업로드 성공 후 자동으로 새 버전이 선택되지 않으면 수동 전환 필요 |
| 코멘트 상태 관리 일괄 처리 | 피드백 뷰어 내 각 코멘트 개별 상태 변경 | 코멘트 수 × 1 | 코멘트 일괄(bulk) 상태 변경 기능 없음 |
| 알림 → 피드백 뷰어 → 알림 복귀 | `/notifications` → 피드백 뷰어 → 브라우저 뒤로가기 | 알림 처리마다 | 알림 목록으로 명시적 복귀 버튼 없이 브라우저 히스토리에 의존 |

---

## C.2 User Journey 분석

### 핵심 플로우 단계 수

| Journey 이름 | 현재 단계 수 | 개선 제안 | 예상 단계 |
|-------------|-----------|----------|----------|
| 신규 Owner 온보딩 (조직 생성 → 첫 피드백) | 8단계 | 온보딩 위저드 도입: 가입 직후 "첫 프로젝트 만들기" 단일 플로우로 통합 | 4~5단계 |
| Client 첫 피드백 작성 | 4단계 | 초대 수락 직후 배정된 화면 목록으로 바로 이동, 빈 상태 안내 강화 | 3단계 |
| Member 버전 업로드 후 코멘트 확인 | 5단계 | 업로드 완료 직후 자동으로 신규 버전 선택 | 3단계 |
| Admin 전체 코멘트 상태 처리 | N×3단계 | 대시보드에 "미해결 피드백 목록" 뷰 추가, 인라인 상태 변경 | 1화면에서 일괄 처리 |
| Owner 팀원 초대 + 프로젝트 배정 | 6단계 | 초대 발송 직후 "이 멤버를 프로젝트에 배정하시겠습니까?" 원스텝 모달 | 3~4단계 |

### Aha Moment 경로

**Owner/Admin**: "이미지 위 핀을 찍자마자 코멘트가 달리는 순간" — 현재 8단계 소요, 온보딩 위저드로 4단계 목표

**Member**: "내가 업로드한 버전에 클라이언트 핀이 실시간으로 찍히는 순간" — 초대 수락 → 프로젝트 진입 → 버전 업로드 → WebSocket 실시간 확인

**Client**: "이미지를 클릭하면 내 의견이 정확한 위치에 핀으로 남겨지는 순간" — 핀 찍기 인터랙션 시각적 가이드(손가락 커서 애니메이션, 클릭 핫스팟 강조) 추가 필요

---

## C.3 Information Architecture

### 기능 그룹핑 분석

**잘 그룹핑된 기능:**

| 그룹 | 위치 | 내용 |
|------|------|------|
| 피드백 뷰어 통합 | `/projects/:id/screens/:screenId` | 이미지 뷰어 + 핀 + 코멘트 + 버전 선택/비교/업로드 — 단일 화면 집중 |
| 설정 영역 통합 | `/settings/*` | 프로필/조직/팀/빌링이 사이드 네비게이션으로 일관 구성 |
| SuperAdmin 전용 분리 | `/admin/*` | 일반 사용자 앱과 완전 분리된 관리자 영역 |

**분산된 기능:**

| 기능 | 위치 A | 위치 B | 문제 |
|------|--------|--------|------|
| 멤버 초대 | `/settings/team` (조직 레벨) | `/projects/:id/members` (프로젝트 레벨) | 두 곳의 초대 UI가 유사하나 API/권한 범위가 다름. 사용자 혼란 가능 |
| 코멘트 상태 필터 | 피드백 뷰어 내 CommentFilterBar | 대시보드 "미해결 피드백 수" 카드(집계값만) | 개별 필터링은 각 피드백 뷰어 진입 필요 |

### 중복/분산 정보

| 정보 항목 | 위치 | 불일치 위험 |
|----------|------|-----------|
| 미해결 피드백 수 | 대시보드 / 프로젝트 카드 / 화면 카드 | 집계 기준 다름, WebSocket 동기화 필요 |
| 멤버 수 | 대시보드 / Settings > Team / Project Members | 조직 전체 vs 프로젝트별 의미 차이에도 동일 레이블 |
| 코멘트 상태 변경 흔적 | 코멘트 패널 뱃지 / Audit Log / Notification | 코멘트 패널에 변경 행위자 미표시 |

---

## C.4 Common UX Problems

### Orphan Pages

| 페이지 | 문제 |
|--------|------|
| `/auth/forgot-password` | 로그인 페이지 "비밀번호 찾기" 링크로만 접근 가능 |
| `/notifications` | 헤더 알림 뱃지 클릭으로만 접근 가능, 사이드바/메인 네비게이션에 명시적 링크 없음 |
| `/admin/settings` | Page Map에 존재하나 기능 명세 없음 |

### Hidden Features

| 기능 | 숨겨진 이유 | 개선 제안 |
|------|-----------|----------|
| 핀 생성 (이미지 클릭) | 인터랙션 안내 UI 힌트 부재 | 첫 진입 시 클릭 가이드 오버레이, 커서 crosshair 변경 |
| 버전 비교 (overlay/side-by-side) | 피드백 뷰어 내부에만 존재 | 화면 목록 카드에 "버전 비교" 퀵 액션 노출 |
| @멘션 기능 | `notification_type=mention` 존재하나 UI 트리거 방법 미명세 | 코멘트 입력창에 "@" 힌트 + 자동완성 드롭다운 명세 |
| Cmd/Ctrl+Enter 단축키 | UI에 미표시 | "게시" 버튼 옆에 `⌘↵` 힌트 표시 |

### Broken Flows

| 플로우 | 문제 | 영향도 |
|--------|------|--------|
| 초대 만료 후 재초대 경로 | 재초대 요청 인앱 경로 없음 | High |
| 비밀번호 재설정 후 재로그인 강제 | 이미 이메일/비밀번호 입력한 흐름에서 다시 로그인 폼 작성 | Medium |
| Member 화면 수정 권한 UI 불일치 | RBAC 서버 처리 시 UI에 권한 없는 메뉴 노출 가능 | High |
| 알림 클릭 → 컨텍스트 손실 | 피드백 뷰어에서 어느 코멘트인지 수동 검색 필요 | Medium |
| 프로젝트 삭제 시 진행 중 세션 | 코멘트 작성 중 데이터 손실 가능 | Medium |

---

## C.5 개선 제안 요약

| # | 카테고리 | 현재 문제 | 개선 제안 | 우선순위 |
|---|---------|----------|----------|---------|
| 1 | User Journey | 신규 Owner 온보딩까지 8단계 | 가입 직후 "첫 프로젝트 설정" 인라인 온보딩 위저드 도입 | High |
| 2 | Broken Flows | 초대 만료 후 재초대 요청 경로 없음 | 만료 안내 화면에 "재초대 요청" 폼 또는 자동 알림 발송 | High |
| 3 | Hidden Features | 핀 생성 인터랙션 시각적 안내 없음 | 첫 진입 시 클릭 가이드 오버레이, 커서 crosshair, 툴팁 상시 표시 | High |
| 4 | Navigation | 알림 → 피드백 뷰어 이동 후 핀/코멘트 하이라이트 미명세 | 딥링크에 `?pin_id=xxx&comment_id=yyy` 쿼리 포함, 자동 포커스 | High |
| 5 | IA | 팀원 초대가 조직/프로젝트 두 곳에 분산 | 초대 후 "프로젝트에도 배정하시겠습니까?" 연속 플로우 추가 | High |
| 6 | Broken Flows | Member 화면 수정/삭제 RBAC UI 미반영 | 더보기 메뉴를 권한 기준 동적 렌더링 | High |
| 7 | User Journey | 코멘트 상태 일괄 변경 기능 없음 | 멀티셀렉트 + 일괄 상태 변경 기능 제공 | High |
| 8 | Navigation | 대시보드 → 피드백 뷰어 4단계 | "최근 피드백 바로가기" 카드, 글로벌 검색 추가 | Medium |
| 9 | Broken Flows | 비밀번호 재설정 후 재로그인 강제 | 재설정 완료 후 자동 로그인 또는 이메일 자동 채우기 | Medium |
| 10 | Hidden Features | 버전 비교 진입점 피드백 뷰어 내부에만 존재 | 화면 목록 카드에 "버전 비교" 퀵 액션 버튼 추가 | Medium |
| 11 | IA | 미해결 피드백 수 집계 기준 불일치 위험 | 집계 범위 레이블 명시, WebSocket 동기화 보장 | Medium |
| 12 | Hidden Features | @멘션 UI 진입점 미명세 | "@" 입력 시 자동완성 드롭다운 명세, 플레이스홀더 힌트 | Medium |
| 13 | Broken Flows | 코멘트 작성 중 화면 삭제 시 입력 손실 | 삭제 전 접속자 경고 알림, 30초 유예 | Medium |
| 14 | Navigation | `/notifications` 복귀 경로 불명확 | "알림 목록으로 돌아가기" 브레드크럼/뒤로가기 버튼 | Medium |
| 15 | Dead-end | `/settings/billing` MVP에서 빈 페이지 | "출시 알림 받기" CTA 또는 플랜 미리보기 | Medium |
| 16 | Hidden Features | Cmd/Ctrl+Enter 단축키 UI 미표시 | "게시" 버튼 옆에 키 힌트 뱃지 | Low |
| 17 | Navigation | 버전 업로드 후 자동 전환 미명세 | 업로드 완료 후 신규 버전 자동 선택 + 토스트 | Low |
| 18 | IA | 역할 선택 드롭다운에 설명 없음 | 각 옵션에 짧은 설명 또는 툴팁 추가 | Low |
| 19 | Broken Flows | Client 상태 변경 버튼 비활성 이유 미설명 | 호버 시 권한 안내 툴팁 표시 | Low |
| 20 | Dead-end | `/admin/settings` 기능 명세 없음 | 기능 범위 정의 또는 MVP에서 라우트 제거 | Low |

---

**분석 요약:** DesignPin의 핵심 UX 강점은 피드백 뷰어의 단일 화면 집중 설계와 실시간 WebSocket 업데이트이다. 주요 개선 필요 영역은 (1) 신규 사용자 온보딩 단계 단축(8→4단계), (2) 알림-피드백 딥링크 연결 완성, (3) 팀원 초대/프로젝트 배정 통합 플로우이며, 이 세 가지가 사용자 이탈에 가장 직접적인 영향을 미친다.
