# DesignPin — Product Requirements Document
**Version:** 1.0
**Date:** 2026-03-12
**Status:** Draft

---

## 0. Project Overview

### Product

| Field | Value |
|-------|-------|
| Name | DesignPin `[💡 권장적용]` |
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
                └── Version (N개)
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
| Screen | 프로젝트 내 개별 디자인 페이지/화면. 여러 버전(Version)을 가질 수 있다. |
| Version | 하나의 Screen에 업로드된 스크린샷 이미지 단위. 버전 히스토리 및 비교의 기준이 된다. |
| Pin | Version 이미지 위에 찍힌 좌표(x, y) 기반 마커. 하나의 Pin에 Comment 스레드가 연결된다. |
| Comment | Pin에 연결된 텍스트 피드백. 상태(open / in_progress / resolved)를 가지며, 파일 첨부 가능. |
| Reply | Comment 하위의 답글. 스레드 형태로 코멘트에 종속된다. |
| Feedback Viewer | 좌측 이미지(핀 표시) + 우측 코멘트 패널로 구성된 핵심 리뷰 화면. 핀↔코멘트 상호 하이라이트 제공. |
| Version Comparison | 두 Version을 overlay(투명도 겹치기) 또는 side-by-side(좌우 배치)로 비교하는 기능. |
| Invitation | 이메일로 조직 또는 프로젝트에 사용자를 초대하는 프로세스. 수락 시 계정 생성 및 멤버십 활성화. |
| Membership | 사용자가 특정 조직 또는 프로젝트에 속하는 관계 레코드. 역할(Role)을 포함한다. |
| Audit Log | 주요 액션(생성/수정/삭제/상태변경)의 수행자, 대상, 일시를 기록하는 이력 데이터. |
| Soft Delete | 레코드를 DB에서 실제 삭제하지 않고 deleted_at 타임스탬프를 기록하여 논리 삭제 처리하는 방식. |
| Multitenancy | 하나의 플랫폼 인스턴스에서 여러 조직의 데이터를 완전히 격리하여 제공하는 SaaS 아키텍처. |

### User Roles

| Role | Description |
|------|-------------|
| SuperAdmin | 플랫폼 전체 운영자. 모든 조직 데이터에 접근하고 조직 상태를 제어한다. 별도 내부 계정으로 관리. `[💡 권장적용: /superadmin 경로 분리 + 별도 인증 미들웨어 적용]` |
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
| `organization_status` | `active`, `suspended`, `deleted` | 조직 상태. SuperAdmin이 제어. `suspended`는 해당 조직 전체 로그인 차단. |
| `notification_type` | `comment_created`, `comment_replied`, `comment_status_changed`, `invitation_received`, `version_uploaded` | 인앱/이메일 알림 트리거 유형. |

### Technical Terms

| Term | Definition |
|------|------------|
| WebSocket | 서버-클라이언트 간 양방향 실시간 통신 프로토콜. 핀/코멘트 실시간 업데이트에 사용. |
| RBAC | Role-Based Access Control. 사용자 역할에 따라 API 및 UI 접근을 제어하는 권한 모델. |
| Soft Delete | `deleted_at` 컬럼에 타임스탬프를 기록하여 데이터를 논리 삭제. 목록 조회 시 `WHERE deleted_at IS NULL` 필터 적용. |
| Presigned URL | Cloud Storage(S3/GCS)에서 제한 시간 내 파일을 업로드/다운로드할 수 있는 서명된 임시 URL. |
| JWT | JSON Web Token. 인증 후 발급되는 액세스 토큰. 만료 시 Refresh Token으로 갱신. |
| Refresh Token | 액세스 토큰 만료 시 재발급에 사용하는 장기 유효 토큰. HttpOnly 쿠키로 저장 권장. `[💡 권장적용]` |
| Multitenancy Row-level Isolation | 모든 DB 쿼리에 `organization_id` 필터를 필수 적용하여 조직 간 데이터 혼용을 방지하는 패턴. |
| Overlay Comparison | 두 Version 이미지를 동일 캔버스에 겹쳐 투명도로 차이를 시각화하는 비교 모드. |
| Side-by-side Comparison | 두 Version 이미지를 좌우로 배치하여 동기화된 스크롤로 비교하는 모드. |
| Debounce | 검색 입력 등 연속 이벤트에서 마지막 이벤트 발생 후 일정 시간(300ms) 대기 후 실행하는 최적화 기법. |

---

## 2. System Modules

### Module 1 — Authentication

#### Main Features
- 이메일/비밀번호 회원가입 (초대 수락 경로 포함)
- 로그인 / 로그아웃
- JWT 액세스 토큰 + Refresh Token 기반 인증 `[💡 권장적용]`
- 비밀번호 재설정 (이메일 링크)
- 인증 가드: 미인증 → 로그인 리다이렉트, 인증됨 → 로그인 페이지 접근 시 대시보드로
- 세션 만료(401) 시 자동 토큰 갱신, 갱신 실패 시 로그인 리다이렉트

#### Technical Flow

**[회원가입 — 초대 수락 경로]**
1. 클라이언트가 초대 이메일의 링크 클릭 → `GET /api/invitations/{token}/verify` 로 토큰 유효성 확인
   - 실패(만료/무효): 토큰 만료 안내 페이지 표시, 재초대 요청 유도
2. 유효한 경우 회원가입 폼 표시 (이메일 필드 자동 채워짐, 수정 불가)
3. 사용자가 이름·비밀번호 입력 → 클라이언트 실시간 유효성 검사 `[버그예방: 폼 유효성 검사 미동작]`
   - 비밀번호 규칙: 최소 8자, 대소문자+숫자+특수문자 포함 `[💡 권장적용]`
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
3. 이후 모든 API 요청에 `X-Organization-Id` 헤더 또는 JWT claim의 `org_id` 포함 `[💡 권장적용]`

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
   - 연관된 Screen, Version, Pin, Comment 캐스케이드 soft delete `[💡 권장적용]`
   - Audit Log 기록

---

### Module 4 — Screen Management

#### Main Features
- 화면(Screen) 등록/삭제 (프로젝트 내)
- 스크린샷 버전(Version) 업로드 (Cloud Storage 연동)
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
2. 클라이언트 사전 검증: 파일 형식(PNG/JPG/WebP/GIF) `[💡 권장적용]`, 최대 크기(20MB) `[💡 권장적용]` `[버그예방: 파일 업로드 실패 및 요구사항 오류]`
   - 미충족 시 구체적 에러 메시지 표시, 업로드 진행하지 않음
3. Presigned URL 요청 → `POST /api/screens/{screen_id}/versions/upload-url` (body: filename, content_type)
   - 서버: MIME 타입 화이트리스트 + 최대 크기 검증 후 Presigned URL 반환
4. 클라이언트가 Presigned URL로 직접 S3/GCS 업로드 (PUT 요청)
   - 성공: 업로드 완료 확인 → `POST /api/screens/{screen_id}/versions` (body: storage_key, filename)
   - 실패: 에러 토스트 + 재시도 버튼 표시
5. Version 레코드 생성 완료, 버전 목록에 추가, Audit Log 기록
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
1. 사용자가 Feedback Viewer에서 이미지 클릭 → 좌표(x%, y%) 계산 (이미지 너비/높이 대비 비율로 저장) `[💡 권장적용]`
2. 핀 생성 → `POST /api/versions/{version_id}/pins` (body: x, y)
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
   - 허용 전환: `open → in_progress`, `in_progress → resolved`, `resolved → open` `[💡 권장적용]`
   - 성공: 코멘트 상태 업데이트, 코멘트 패널 갱신
   - 실패: 에러 토스트
2. WebSocket 이벤트 emit: `comment:status_changed`
3. 알림 발송: `notification_type=comment_status_changed`
4. Audit Log 기록

**[코멘트 삭제 — Admin]**
1. Admin이 코멘트 삭제 → `DELETE /api/comments/{comment_id}`
   - RBAC 검증: Admin 이상만 가능 `[버그예방: 관리자 벌크/단일 액션 오류]`
   - Soft delete 처리
   - 연관 Reply, 첨부파일 참조 캐스케이드 soft delete `[💡 권장적용]`
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
   - 응답: version 목록 (id, filename, created_at, uploaded_by)

**[버전 비교]**
1. 사용자가 버전 선택기에서 2개 버전 선택 (Base / Compare)
2. 비교 모드 선택 (overlay / side-by-side) → UI 전환
3. 이미지 URL 요청 → `GET /api/versions/{version_id}/image-url` (Presigned URL 반환)
   - 두 버전 이미지 URL 병렬 요청
4. Overlay 모드: 두 이미지를 동일 캔버스에 렌더링, 투명도 슬라이더 제공 `[💡 권장적용: 기본 50% 불투명도]`
5. Side-by-side 모드: 좌우 패널에 각 이미지 렌더링, 스크롤/줌 동기화 `[💡 권장적용]`
6. 비교 뷰에서 핀은 비활성화 (읽기 전용) `[💡 권장적용]`
   - 실패(이미지 로드 오류): 오류 메시지 + 재시도 버튼

---

### Module 7 — Team & Invitation

#### Main Features
- 이메일 초대 발송 (조직 멤버 초대 / 프로젝트 클라이언트 초대)
- 초대 링크 유효성 관리 (TTL: 7일) `[💡 권장적용]`
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
   - 자기 자신 역할 변경 불가 `[💡 권장적용]`
2. 멤버 제거 → `DELETE /api/organizations/{org_id}/members/{user_id}`
   - Membership soft delete, 해당 사용자의 진행 중 세션은 다음 토큰 갱신 시 403 처리 `[💡 권장적용]`
3. Audit Log 기록

---

### Module 8 — Notification

#### Main Features
- 인앱 알림 (코멘트 생성, 답글, 상태 변경, 초대 수신, 버전 업로드)
- 이메일 알림 (동일 트리거, 사용자 설정으로 on/off 가능) `[💡 권장적용]`
- 알림 읽음 처리 (단건 / 전체)

#### Technical Flow

**[알림 생성 및 발송]**
1. 트리거 이벤트 발생 (코멘트 생성, 상태 변경 등) → 해당 모듈 내부에서 Notification 서비스 호출
2. 알림 수신 대상자 결정 (프로젝트 멤버 전원 또는 핀 작성자 등) `[💡 권장적용: 자기 자신 제외]`
3. Notification 레코드 생성 → `POST /api/notifications` (내부 호출, body: user_id, type, payload)
4. 이메일 발송: Email Service(SendGrid/SES) API 호출
   - 발송 실패 시: 재시도 큐 등록 (최대 3회) `[💡 권장적용]`
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
2. 연결 끊김 감지 시: 자동 재연결 (Exponential Backoff) `[💡 권장적용]`
3. 재연결 성공 시: 연결 해제 시점 이후 놓친 이벤트 재조회 → `GET /api/events/missed?since={timestamp}`

##### WebSocket Events

| Channel | Event | Payload | Direction |
|---------|-------|---------|-----------|
| `project:{project_id}` | `pin:created` | `{ pin_id, version_id, x, y, created_by }` | Server → Client |
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
2. 클라이언트 사전 검증: 허용 형식(PNG/JPG/PDF/ZIP/...)`[💡 권장적용]`, 최대 10MB `[💡 권장적용]` `[버그예방: 파일 업로드 실패 및 요구사항 오류]`
   - 미충족 시 구체적 에러 표시, 업로드 진행하지 않음
3. Presigned URL 요청 → `POST /api/attachments/upload-url` (body: filename, content_type, size)
   - 서버: MIME 타입 화이트리스트 + 최대 크기 검증
4. Presigned URL로 S3/GCS 직접 업로드
   - 성공: `POST /api/attachments` (body: storage_key, filename, size, comment_id)
   - 실패: 에러 토스트 + 재시도 버튼

**[파일 다운로드]**
1. 첨부파일 클릭 → `GET /api/attachments/{attachment_id}/download-url`
   - 서버: 요청자의 프로젝트 접근 권한 검증 후 Presigned URL 반환 (유효시간: 15분) `[💡 권장적용]`
2. 클라이언트: 반환된 URL로 직접 다운로드

---

### Module 11 — Audit Log

#### Main Features
- 주요 액션 기록 (생성/수정/삭제/상태변경)
- 행위자(user_id), 대상(resource_type, resource_id), 액션(action), 일시(timestamp) 저장
- 보존 기간: 1년 `[💡 권장적용]`
- Owner/Admin/SuperAdmin 조회 가능

#### Technical Flow

**[Audit Log 기록]**
1. 주요 API 처리 완료 후 비동기로 Audit Log 서비스 호출 `[💡 권장적용: 동기 처리 시 응답 지연 방지]`
2. `POST /api/audit-logs` (내부 호출, body: user_id, action, resource_type, resource_id, metadata)
   - `action` 예시: `project.created`, `comment.status_changed`, `member.removed`
   - `metadata`: 변경 전/후 값(before/after) JSON 포함 `[💡 권장적용]`
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
- 별도 경로(`/superadmin`) 및 추가 인증 미들웨어 `[💡 권장적용]`

#### Technical Flow

**[전체 조직 조회]**
1. SuperAdmin이 `/superadmin/organizations` 접근 → RBAC 미들웨어: `role=superadmin` 검증
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
