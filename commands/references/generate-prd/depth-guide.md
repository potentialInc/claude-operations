# 필수 밀도 기준 (Depth Guide)

> 목표: "길게 쓰라"가 아니라 "이 항목들이 빠지면 QA FAIL".
> 모든 라우트/페이지에 아래 항목이 **존재**해야 한다.

---

## Section 0 — Project Overview 밀도 기준

### User Types 테이블
- Type, DB Value, Description, Key Actions 4개 컬럼 필수
- DB Value는 정수(0, 1, 2, ..., 99) — ORM enum 매핑용
- 모든 유저타입 포함 (Admin 포함)

### User Status 테이블
- Status, DB Value, Behavior 3개 컬럼 필수
- Suspended/Withdrawn 시 구체적 동작 명시 (표시 메시지, 데이터 보존 기간)

### MVP Scope
- Included / Excluded 모두 명시
- Excluded 항목에 "deferred to [phase/version]" 명시

---

## Section 1 — Terminology 밀도 기준

### Core Concepts
- 프로젝트에서 사용하는 **모든** 도메인 용어 포함
- 일반적 IT 용어는 제외 (API, DB 등)
- Section 3/4 본문에서 사용되는 용어와 1:1 대응

### Status Values
- DB에 저장되는 모든 enum 정의
- 각 enum의 값 목록 + 언제 각 값이 적용되는지 설명
- Section 6 (Data Model)의 Status Enum과 일치해야 함

---

## Section 2 — System Modules 밀도 기준

### Technical Flow
- 모듈당 **5-10 스텝** (최소 5스텝)
- 각 스텝: `[Actor] does [Action]` 형식
- **성공 분기** + **실패 분기** 모두 포함
- 실패 시 사용자에게 보이는 결과까지 명시
- **API Hint 필수**: 각 스텝에 예상 엔드포인트 명시 (`→ POST /api/...`)

### WebSocket Events
- 실시간 기능 모듈에 필수
- Channel, Event, Payload, Direction 4개 컬럼
- Payload는 주요 필드 나열 (`{ field1, field2 }`)

### 3rd Party API List
각 API에 대해 4개 필드 필수:
| 필드 | 설명 |
|------|------|
| Service | 서비스명 |
| Purpose | 사용 목적 |
| Integration Point | 어디서 사용되는지 |
| Alternative | 대안 서비스 (없으면 "N/A") |

---

## Section 3 — 라우트별 필수 8항목

모든 라우트(Route/Page)에 아래 8개 항목이 **반드시** 존재해야 한다. 하나라도 빠지면 QA FAIL.

### 1. 컴포넌트 상세
- 화면의 모든 UI 요소 나열
- 각 컴포넌트의 상태별 정의: **default / active / disabled / error**
- 예: 버튼의 default 상태, 눌렸을 때(active), 비활성화(disabled), 에러 시(error)

### 2. 입력필드 검증규칙
해당 라우트에 입력이 있을 경우 테이블로 정의:

| Field | Type | Required | Min/Max | Pattern | Error Message |
|-------|------|----------|---------|---------|---------------|

입력이 없는 라우트는 "해당 없음" 명시.

### 3. 상태 화면
3가지 상태 모두 정의:
- **Loading**: skeleton UI / spinner / 로딩 텍스트
- **Empty**: 데이터 0건일 때 메시지 + CTA (Call to Action)
- **Error**: 에러 발생 시 메시지 + 재시도 방법

### 4. 인터랙션
사용자가 할 수 있는 모든 인터랙션:
- 클릭 (click)
- 호버 (hover)
- 드래그 (drag & drop)
- 키보드 단축키
- 풀투리프레시 (pull to refresh) — 모바일
- 해당 없으면 "클릭 인터랙션만" 명시

### 5. 네비게이션
- 이 라우트에서 이동 가능한 모든 화면 목록
- 뒤로가기 동작 (이전 화면 / 탭 홈 / 앱 홈)
- 딥링크 진입 시 뒤로가기 동작

### 6. 데이터 표시
- 표시 항목 (어떤 데이터가 보이는지)
- 정렬 기준 (기본 정렬, 변경 가능 여부)
- 페이지네이션 방식 (무한 스크롤 / 페이지 번호 / Load More)

### 7. 에러 핸들링
3가지 에러 유형별 처리:
- **네트워크 에러**: 오프라인 / 타임아웃
- **서버 에러**: 500 / 503
- **권한 에러**: 로그인 필요 / 접근 불가

### 8. 엣지 케이스
최소 4가지:
- **0건**: 데이터 없을 때
- **최대치**: 데이터 대량일 때 (성능, 표시 제한)
- **동시접근**: 여러 사용자가 동시에 같은 데이터 조작
- **오프라인**: 네트워크 연결 없을 때

### 추가 밀도 요구사항
- 각 라우트에 **접근 가능 역할** 명시 (Page Map의 Access 컬럼과 일치)
- 데이터 수정/삭제 기능에 **소유권 규칙** 명시 (본인 것만 / 모든 것)

---

## Section 4 — 관리페이지 밀도 기준

### 각 관리페이지 필수 항목

1. **테이블 컬럼 정의** (테이블로):
   | Column | Type | Sortable | Description |
   |--------|------|----------|-------------|

2. **필터 옵션**:
   - 드롭다운 값 목록
   - 날짜범위 선택기
   - 검색 대상 필드

3. **Detail Drawer/Modal**:
   - 표시 필드 전체 목록
   - 가능한 액션 (상태 변경, 삭제, 수정 등)
   - 연관 데이터 표시 (활동 로그, 관련 엔티티)

4. **Creation Modal**:
   - 입력필드별 검증규칙 (테이블)
   - 필수/선택 구분
   - 에러 메시지

5. **표준기능 적용 확인**:
   - `references/admin-standards.md`의 모든 필수 항목 포함 여부
   - 제외된 항목이 있으면 사유 명시

---

## Section 5 — Tech Stack 밀도 기준

### Technologies 테이블
- Layer, Technology, Version, Purpose 4개 컬럼 필수
- 모든 레이어 포함 (Backend, Language, ORM, Database, Frontend, Routing, State, CSS, Build)

### Third-Party Integrations
- Section 2의 3rd Party API List에 나온 모든 서비스가 여기에도 존재해야 함
- Service, Purpose 최소 2개 컬럼

### Key Decisions
- 주요 기술 선택과 그 근거
- 최소 2개 이상

### Environment Variables
- 모든 외부 서비스 연동에 필요한 환경변수
- Variable, Description 2개 컬럼

---

## Section 6 — Data Model 밀도 기준

### Entity List
- Entity당: PK, 주요 컬럼 **5개 이상**, FK 관계
- FK는 `[entity]_id(FK)` 형식으로 명시
- 모든 CRUD 가능 엔티티 포함

### Entity Relationships
- 모든 관계 유형 명시: 1:1, 1:N, N:N, self-reference
- N:N 관계는 **중간 테이블(join table)** 명시
- 모호한 관계 해결 필수 — "관계 있음" 수준 불허, 구체적 카디널리티 명시

### Status Enums
- Section 1의 Status Values와 일치
- DB 저장 값 (정수) 포함
- 어떤 엔티티에서 사용되는지 명시

### Index Hints
- 검색/필터 대상 컬럼에 인덱스 힌트
- Entity, Column(s), Type, Reason 4개 컬럼
- UNIQUE, COMPOSITE, BTREE 구분

### Soft Delete
- 적용 대상 엔티티 목록
- 데이터 보존 기간

---

## Section 7 — Permission Matrix 밀도 기준

### Action × Role Matrix
- **모든 CRUD 가능 리소스** × **모든 역할** 조합 — 빈 셀 없음
- 수정/삭제에 소유권 조건 명시: `✅own` = 본인 것만
- 누락 셀 금지: `?` 불허, 반드시 `✅` / `❌` / `✅own` 중 하나

### Ownership Rules
- "Own resource" 정의: `resource.user_id === currentUser.id`
- Admin override 규칙
- 프로젝트 특화 소유권 규칙 (팀 기반, 조직 기반 등)

### Role Hierarchy
- 하위 역할 → 상위 역할 순서
- 상속 규칙 명시

---

## QA에서의 활용

qa 에이전트는 이 가이드를 기준으로 검증한다:
- Section 3 각 라우트: 8항목 중 빠진 것 있으면 → FAIL (규칙 2)
- Section 4 각 관리페이지: 필수 항목 빠졌으면 → FAIL (규칙 4)
- Section 6 엔티티: Section 3/4의 CRUD 엔티티 누락 → FAIL (규칙 10)
- Section 7 권한: Section 3/4의 기능에 대응 Action 없으면 → FAIL (규칙 11)
