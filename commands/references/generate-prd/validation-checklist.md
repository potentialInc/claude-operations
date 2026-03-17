# QA 검증 규칙 (기계 검증)

> 모든 규칙은 **주관적 판단 없이** 카운팅/존재여부로만 판정한다.
> "깊이가 부족하다", "더 상세하면 좋겠다" 같은 판단은 금지.

---

## 규칙 1: 라우트 수 일치

### 검증 방법
1. `screen-inventory.md`에서 유저앱 라우트 수(N) + 어드민 라우트 수(M) 카운팅
2. PRD Section 3에서 정의된 라우트 수 카운팅
3. PRD Section 4에서 정의된 관리페이지 수 카운팅
4. `screen-inventory.md 라우트 수 ≤ PRD 라우트 수` 확인

### PASS 조건
- `screen-inventory.md`의 **모든** 라우트가 PRD에 존재
- PRD에 추가 라우트가 있는 것은 허용 (에이전트가 보강한 것)

### FAIL 조건
- `screen-inventory.md`에 있는 라우트가 PRD에 **하나라도** 없음

### Evidence 형식
```
유저앱: screen-inventory {N}개 → PRD {M}개 (누락: [라우트1, 라우트2])
어드민: screen-inventory {N}개 → PRD {M}개 (누락: [라우트1])
```

---

## 규칙 2: 필수 8항목 존재

### 검증 방법
Section 3의 각 라우트(Route/Page)에서 다음 8개 항목 **섹션의 존재 여부** 확인:

| # | 항목 | 키워드 탐색 |
|---|------|-----------|
| 1 | 컴포넌트 상세 | "컴포넌트", "Component", UI 요소 목록 |
| 2 | 입력필드 검증규칙 | "검증", "Validation", "Field", "Type", "Required" 테이블 |
| 3 | 상태 화면 | "Loading", "Empty", "Error" 상태 정의 |
| 4 | 인터랙션 | "인터랙션", "Interaction", "클릭", "호버" |
| 5 | 네비게이션 | "네비게이션", "Navigation", "이동", "뒤로가기" |
| 6 | 데이터 표시 | "데이터", "Data", "정렬", "페이지네이션" |
| 7 | 에러 핸들링 | "에러", "Error", "네트워크", "서버" |
| 8 | 엣지 케이스 | "엣지", "Edge", "0건", "최대치" |

### PASS 조건
- Section 3의 **모든** 라우트에 8개 항목 섹션이 존재
- 입력이 없는 라우트에서 "입력필드 검증규칙: 해당 없음"도 PASS

### FAIL 조건
- 하나라도 빠진 라우트가 있음

### Evidence 형식
```
총 {N}개 라우트 검증
FAIL: [라우트] — 빠진 항목: [항목1, 항목2]
FAIL: [라우트] — 빠진 항목: [항목3]
```

---

## 규칙 3: Admin 1:1 매핑

### 검증 방법
1. Section 3에서 **CRUD 가능한 데이터 엔티티** 식별
   - 생성(Create): 유저가 새로 만들 수 있는 것 (게시물, 프로필, 주문 등)
   - 수정(Update): 유저가 변경할 수 있는 것
   - 삭제(Delete): 유저가 제거할 수 있는 것
2. 각 엔티티에 대응하는 관리페이지가 Section 4에 있는지 확인

### PASS 조건
- Section 3의 모든 CRUD 엔티티에 대응하는 관리페이지가 Section 4에 존재

### FAIL 조건
- 관리페이지가 없는 CRUD 엔티티 존재

### Evidence 형식
```
CRUD 엔티티 {N}개 식별
매핑 완료: {M}개
누락: [엔티티명] — Section 3 위치: [라우트], Section 4에 관리페이지 없음
```

---

## 규칙 4: Admin 표준기능

### 검증 방법
Section 4의 각 관리페이지에서 `admin-standards.md`의 **필수(✅) 항목** 존재 확인:

**체크 항목 (List Page):**
- Search, Filters, Column Sorting, Checkbox Selection, Bulk Actions, Pagination

**체크 항목 (Table UI):**
- Loading State, Empty State, Error State, Search No Results, Action Column

**체크 항목 (Detail/Edit):**
- Detail Drawer/Modal, Edit Form, Delete Confirmation

**체크 항목 (Data Export):**
- CSV/Excel Download, Date Range Selection

**체크 항목 (Common UI/UX):**
- Toast Notifications, Breadcrumb, Create Modal/Drawer

**체크 항목 (v2 추가):**
- Bulk Action 확인 dialog, Soft Delete, Date/Time 표준

### PASS 조건
- 모든 관리페이지에 위 필수 항목이 명시적 또는 암시적으로 포함
- "Admin Dashboard Standard Features 적용" 같은 상위 선언도 PASS

### FAIL 조건
- 표준기능이 빠진 관리페이지 존재 (상위 선언도 없는 경우)

### Evidence 형식
```
{N}개 관리페이지 검증
FAIL: [페이지명] — 누락: [기능1, 기능2]
```

---

## 규칙 5: 용어 교차참조

### 검증 방법
1. Section 1 Terminology 테이블에서 모든 용어 추출
2. Section 3 + Section 4 본문에서 각 용어 사용 여부 확인
3. Section 3 + Section 4 본문에서 Terminology에 없는 도메인 용어 발견

### PASS 조건
- 용어집의 모든 용어가 본문에서 1회 이상 사용됨
- 본문의 모든 도메인 용어가 용어집에 정의됨

### FAIL 조건
- **용어집에만 있는 용어**: 정의했지만 본문에서 안 쓰임 (불필요 용어)
- **본문에만 있는 용어**: 도메인 용어인데 용어집에 없음 (정의 누락)

### Evidence 형식
```
용어집 {N}개 용어
본문에서 미사용: [용어1, 용어2]
본문에만 있는 용어: [용어3, 용어4]
```

### 참고
- 일반 IT 용어 (API, DB, URL 등)는 제외
- 도메인 특화 용어만 대상

---

## 규칙 6: TBD 제로

### 검증 방법
전체 PRD에서 다음 패턴 검색:
- `TBD`
- `미정`
- `추후 결정`
- `확인 필요` (Additional Questions 섹션 내 제외)
- `To be determined`
- `To be decided`
- `[?]`

### PASS 조건
- 위 패턴이 PRD 본문에 0건
- `[💡 권장적용]` 마킹이 있는 항목은 TBD가 아님 (PASS)
- `[📌 권장적용됨]` 마킹이 있는 항목은 TBD가 아님 (PASS)
- Additional Questions 섹션 내 `확인 필요`는 제외

### FAIL 조건
- 위 패턴이 1건이라도 발견됨

### Evidence 형식
```
TBD 패턴 검색 결과: {N}건
발견 위치:
- [섹션/라우트명]: "[발견된 텍스트]"
```

---

## 규칙 7: 라우트 커버리지

### 검증 방법
1. Section 3의 Page Map 테이블에서 모든 라우트 추출
2. Section 3의 Feature List by Route에서 정의된 라우트 추출
3. 양방향 비교

### PASS 조건
- Page Map의 모든 라우트가 Feature List에 존재
- Feature List의 모든 라우트가 Page Map에 존재

### FAIL 조건
- Page Map에 있는데 Feature List에 없는 라우트 존재
- Feature List에 있는데 Page Map에 없는 라우트 존재

### Evidence 형식
```
Page Map 라우트: {N}개
Feature List 라우트: {M}개
Page Map에만 있는 라우트: [라우트1, 라우트2]
Feature List에만 있는 라우트: [라우트3]
```

---

## 규칙 8: 수치 일관성

### 검증 방법
1. 전체 PRD에서 `숫자+단위` 패턴 추출 (예: 10MB, 30일, 5분, 100개)
2. 같은 개념을 지칭하는 값이 서로 다른지 확인

### PASS 조건
- 동일 개념에 대한 수치가 문서 전체에서 일관됨

### FAIL 조건
- 같은 개념이 서로 다른 값으로 기술됨 (예: "파일 크기 제한 10MB" vs "최대 20MB")

### Evidence 형식
```
수치 패턴 {N}개 추출
불일치 발견:
- [개념]: Section [X]에서 "[값1]" vs Section [Y]에서 "[값2]"
```

---

## 규칙 9: Tech Stack 완전성

### 검증 방법
1. Section 5 Technologies 테이블이 비어있지 않은지 확인
2. Section 2의 3rd Party API List에 있는 서비스가 Section 5 Third-Party Integrations에도 있는지 확인
3. Section 5의 Key Decisions가 1개 이상 있는지 확인
4. Section 5의 Environment Variables가 1개 이상 있는지 확인

### PASS 조건
- Technologies 테이블에 최소 5개 레이어 존재
- 3rd Party 서비스가 양쪽 섹션에 모두 존재
- Key Decisions 1개 이상
- Environment Variables 1개 이상

### FAIL 조건
- Technologies 비어있거나 5개 미만
- Section 2의 3rd Party가 Section 5에 누락
- Key Decisions 또는 Environment Variables 없음

### Evidence 형식
```
Technologies: {N}개 레이어
3rd Party 일치: Section 2에 {N}개, Section 5에 {M}개
누락: [서비스명]
Key Decisions: {N}개
Environment Variables: {N}개
```

---

## 규칙 10: 엔티티 커버리지

### 검증 방법
1. Section 3/4에서 CRUD 가능한 데이터 엔티티 식별 (생성/수정/삭제 동작이 있는 것)
2. Section 6 Entity List에 해당 엔티티가 있는지 확인

### PASS 조건
- Section 3/4의 모든 CRUD 엔티티가 Section 6 Entity List에 존재

### FAIL 조건
- Section 3/4에 CRUD 동작이 있는 엔티티가 Section 6에 없음

### Evidence 형식
```
CRUD 엔티티 {N}개 식별 (Section 3: {A}개, Section 4: {B}개)
Section 6 Entity List: {M}개
누락: [엔티티명] — Section [3/4] [라우트]에서 [생성/수정/삭제] 있으나 Section 6에 없음
```

---

## 규칙 11: 권한 완전성

### 검증 방법
1. Section 3/4에서 데이터 수정/삭제 기능이 있는 곳 식별
2. 해당 Resource × Action이 Section 7 Permission Matrix에 존재하는지 확인
3. 빈 셀(?) 또는 누락 행이 없는지 확인

### PASS 조건
- Section 3/4의 모든 수정/삭제 기능에 대응하는 Permission Matrix 항목이 존재
- Permission Matrix에 빈 셀 없음

### FAIL 조건
- Section 3/4에 기능(특히 수정/삭제)이 있는데 Section 7에 해당 Action × Role이 없음
- Permission Matrix에 `?` 또는 빈 셀 존재

### Evidence 형식
```
수정/삭제 기능 {N}개 식별
Permission Matrix 매핑 완료: {M}개
누락: [Resource] × [Action] — Section [3/4] [라우트]에 있으나 Section 7에 없음
빈 셀: [Resource] × [Role] — 값 미지정
```

---

## 규칙 12: 클라이언트 요구사항 추적

### 검증 방법
1. `parsed-input.md`에서 클라이언트가 명시적으로 요구한 사항 추출
   - 기능적 요구사항 (특정 기능, 워크플로우)
   - 비기능적 요구사항 (반응형, i18n, 보안, 확장성, 성능 등)
2. PRD에서 각 요구사항에 대응하는 구체적 스펙이 있는지 확인

### PASS 조건
- 모든 명시적 요구사항이 PRD에 구체적 스펙으로 존재
- 또는 Section 8 (Open Questions)에 미해결 항목으로 등록됨

### FAIL 조건
- 명시적 요구사항이 PRD에 없고, Open Questions에도 없음 (조용히 누락)

### Evidence 형식
```
클라이언트 요구사항 {N}개 식별
PRD 반영: {M}개
Open Questions 이동: {K}개
누락: [요구사항] — parsed-input에 명시되었으나 PRD/Open Questions 어디에도 없음
```
