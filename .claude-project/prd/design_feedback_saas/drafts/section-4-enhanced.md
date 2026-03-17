# Section 4. Admin Dashboard (슈퍼어드민 전용)

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
| 필터 | Plan Dropdown | All / Free / Pro / Enterprise (TBD: 요금제 확정 시 업데이트) |
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

> TBD: 감사 로그 보존 기간은 미정입니다. 보존 정책 확정 필요 (참고: `tbd-items.md` #14).

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
- 날짜 범위 선택 필수 (내보내기 시 최대 범위 제한 TBD)
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
| 파일 업로드 설정 | 최대 파일 크기, 허용 파일 형식 (TBD: 확정 값 필요) |
| 이메일 설정 | 발신자 이메일, 이메일 서비스 연동 설정 |
| 빌링/플랜 설정 | 기본 플랜, 플랜별 제한값 (프로젝트/멤버 수), Stripe 연동 키 (TBD) |
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

> TBD: 비밀번호 규칙 구체적 기준은 클라이언트 확인 필요 (`tbd-items.md` #3)

#### 4.2.12.4 파일 업로드 설정 폼

| 필드 | 타입 | 필수 | 검증 규칙 |
|------|------|------|-----------|
| 최대 파일 크기 (MB) | Number Input | ✓ | 1~100 정수 |
| 허용 이미지 포맷 | Checkbox Group | ✓ | PNG / JPG / WebP / GIF (최소 1개 선택) |
| 허용 첨부파일 포맷 | Checkbox Group | - | PDF / DOC / DOCX / XLS / XLSX / ZIP 등 |

> TBD: 파일 크기 제한 및 지원 포맷 기본값은 클라이언트 확인 필요 (`tbd-items.md` #7, #8, #12)

#### 4.2.12.5 유지보수 설정

| 기능 | 설명 |
|------|------|
| 점검 모드 활성화 | Toggle 켜면 일반 사용자 접근 차단, 슈퍼어드민만 접속 가능. 활성화 전 확인 Dialog 필수 |
| 점검 배너 메시지 | Textarea, 점검 모드 ON 시 일반 사용자 로그인 화면에 표시 |
| Soft Delete 데이터 정리 | 보존 기간 초과 soft-deleted 레코드 영구 삭제 실행 버튼. 실행 전 영향 건수 미리보기 + 확인 Dialog (TBD: 보존 기간 `tbd-items.md` #13) |

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
