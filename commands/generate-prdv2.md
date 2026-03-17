---
description: "Generate a dense, comprehensive PRD using multi-agent tiki-taka workflow"
argument-hint: "Path to client answer file (e.g., /path/to/client-answers.md)"
allowed-tools: Agent, Read, Write, Edit, Glob, Grep, AskUserQuestion, Bash(mkdir *)
---

# Generate PRD v2 — 티키타카 멀티에이전트 오케스트레이션

클라이언트 답변 파일로부터 밀도 높은 개발 PRD를 생성하는 8-Phase 워크플로우.

## 핵심 원칙

1. **밀도 > 길이**: "많이 쓰라"가 아니라 "필수 항목이 빠지면 FAIL"
2. **스코프 한정 리뷰**: 각 에이전트는 자기 전문 도메인만 리뷰
3. **TBD 제로**: 불명확 항목은 best practice 권장안으로 채우고 `[💡 권장적용]` 마킹
4. **기계 검증**: QA는 주관적 판단 없이 카운팅/존재여부 기반 규칙만 사용
5. **정규화된 인터페이스**: 에이전트 간 데이터는 구조화된 중간 산출물로 전달
6. **출처 추적**: 권장적용 항목은 승인 후에도 `[📌 권장적용됨]` 마커로 출처 유지

## 에이전트 구성

| Agent | Model | 역할 |
|-------|-------|------|
| overview-writer | sonnet | Section 0+1+2 작성 (프로젝트정보, 용어, 모듈플로우 + API힌트) |
| user-app-writer | opus | Section 3 작성 (라우트 기반 화면/컴포넌트 상세) |
| admin-writer | sonnet | Section 4 작성 (Admin 관리페이지 + 표준기능) |
| tech-writer | sonnet | Section 5+6 작성 (Tech Stack + Data Model) |
| rbac-writer | sonnet | Section 7 작성 (Permission Matrix) |
| prd-synthesizer | opus | Section 0-8 통합, 교차참조, 일관성 감사 |
| question-generator | sonnet | 빠진/불명확 항목 질문 생성 |
| ux-analyzer | sonnet | UX 플로우 이슈 발견 및 개선 제안 |
| qa | sonnet | 12개 규칙 기반 기계 검증 |
| support | sonnet | QA FAIL 항목 수정 (최대 3라운드) |

## Reference 파일 경로

모든 reference 파일은 `commands/references/generate-prd/` 하위에 위치:
- `prd-template.md` — Section 0-8 아웃풋 구조
- `prompt-templates.md` — 에이전트 명세 + 작성/리뷰/보강 3모드 프롬프트
- `depth-guide.md` — 필수 밀도 기준
- `admin-standards.md` — Admin 표준기능 매트릭스
- `validation-checklist.md` — 기계 검증 가능한 12개 QA 규칙

---

## Phase 1: Intake & Validation

`$ARGUMENTS`에서 클라이언트 답변 파일 경로를 읽는다.

### 검증 체크리스트

1. 파일 경로가 제공되었는가?
2. 파일이 존재하고 읽을 수 있는가?
3. 파일이 비어있지 않은가?

**검증 실패 시:**
```
Error: [구체적 에러 메시지]
Usage: /generate-prdv2 /path/to/client-answers.md
```

### 기존 PRD 확인

1. `.claude-project/prd/` 하위 디렉토리에서 앱 이름 매칭 PRD 검색
2. 발견 시 AskUserQuestion으로 확인:
   ```
   기존 PRD 발견: [filename]
   1. 기존 PRD를 참고하여 새로 생성 (기존 PRD를 컨텍스트로 활용)
   2. 완전히 새로 생성
   ```
3. **업데이트 모드**: 기존 PRD 파일 경로를 Phase 3 에이전트에 추가 컨텍스트로 전달

---

## Phase 2: Client Answer Analysis → 정규화된 중간 산출물 생성

클라이언트 입력을 파싱하여 **구조화된 중간 산출물**을 생성한다. 에이전트에는 원본이 아닌 정규화된 데이터를 전달한다.

### 2.1 프로젝트 폴더명 결정

앱 이름에서 프로젝트 폴더명을 생성한다:
- 공백 → 언더스코어
- 특수문자 제거
- 예: "타이어뱅크 자사몰" → `타이어뱅크_자사몰`

이하 `{ProjectName}`은 이 폴더명을 의미한다.

### 2.2 출력 디렉토리 생성

```bash
mkdir -p .claude-project/prd/{ProjectName}/intermediate
mkdir -p .claude-project/prd/{ProjectName}/drafts
mkdir -p .claude-project/prd/{ProjectName}/reviews
```

### 2.3 입력 파싱 및 중간 산출물 생성

클라이언트 답변 파일을 읽고 다음 8개 파일을 `.claude-project/prd/{ProjectName}/intermediate/`에 생성:

#### parsed-input.md
원본 입력을 섹션별로 정리:
- Basic Info (앱 이름, 타입, 마감일, 유저타입, 관계)
- Design Reference (참고 앱, 차별점, 색상, 폰트)
- Features (유저타입별 기능, 모듈 플로우, 인증, 커뮤니케이션)
- Data (수집 데이터, 내보내기 데이터)
- Technical (3rd Party, 도메인 용어)
- Other (추가 정보)

#### feature-map.md
유저타입별 기능→화면 매핑 테이블:
```markdown
## [User Type 1]
| 기능 | 화면 | 비고 |
|------|------|------|
| 피드 보기 | Home > Feed List | 메인 기능 |
| 프로필 편집 | My Page > Edit Profile | |
```

#### screen-inventory.md
전체 화면 목록 (QA 검증 기준선):
```markdown
## 유저앱 라우트 (총 N개)
| # | 라우트 | 페이지명 | 유저타입 | 접근 그룹 | 비고 |
|---|--------|---------|---------|----------|------|

## 어드민 라우트 (총 M개)
| # | 라우트 | 페이지명 | 관리 대상 | 비고 |
|---|--------|---------|----------|------|
```

#### module-list.md
시스템 모듈 및 3rd Party 목록:
```markdown
## 시스템 모듈
| 모듈명 | 설명 | 관련 유저타입 |
|--------|------|-------------|

## 3rd Party API
| 서비스 | 목적 | 연동 포인트 |
|--------|------|-----------|
```

#### tbd-items.md
불명확 항목 목록 (에이전트가 `[💡 권장적용]`으로 채울 대상):
```markdown
| # | 항목 | 출처 | 불명확 사유 |
|---|------|------|-----------|
| 1 | 로그인 방식 | 입력파일에 언급 없음 | 소셜 로그인 종류 미지정 |
```

#### tech-hints.md
클라이언트 입력에서 기술 관련 언급 추출:
```markdown
## 기술 선호
| 항목 | 클라이언트 언급 | 출처 |
|------|--------------|------|
| Backend Framework | NestJS 선호 | 입력파일 Section X |
| Database | PostgreSQL | 입력파일 Section Y |

## 3rd Party 서비스
| 서비스 | 용도 | 클라이언트 언급 |
|--------|------|--------------|

## 인프라
| 항목 | 클라이언트 언급 |
|------|--------------|
```

#### permission-hints.md
클라이언트 입력에서 권한 관련 언급 추출:
```markdown
## 역할 목록
| 역할 | 설명 | 출처 |
|------|------|------|

## 명시적 권한 규칙
| 역할 | 가능 | 불가능 | 출처 |
|------|------|--------|------|

## 소유권 규칙
| 규칙 | 출처 |
|------|------|
```

#### bug-patterns-filtered.md
`.claude-project/knowledge/bug-patterns.md` (프로젝트별) + `commands/references/generate-prd/bug-patterns-global.md` (글로벌) 에서 관련 카테고리만 필터링. 파일이 없으면 생성하지 않음.

### 2.4 컨텍스트 분배 전략

각 에이전트 프롬프트에 포함할 중간 산출물:

```
모든 에이전트 공통:     parsed-input.md + feature-map.md
overview-writer:      + module-list.md + tbd-items.md
user-app-writer:      + screen-inventory.md + tbd-items.md
admin-writer:         + screen-inventory.md + tbd-items.md
tech-writer:          + module-list.md + tech-hints.md + tbd-items.md
rbac-writer:          + permission-hints.md + section-3-draft + section-4-draft
qa:                   + screen-inventory.md + parsed-input.md
```

추가로 각 에이전트에 해당 reference 파일 내용을 발췌하여 프롬프트에 인라인으로 포함:
- overview-writer: `depth-guide.md` Section 0-2 + `prd-template.md` Section 0-2
- user-app-writer: `depth-guide.md` Section 3 + `prd-template.md` Section 3
- admin-writer: `depth-guide.md` Section 4 + `prd-template.md` Section 4 + `admin-standards.md` 전문
- tech-writer: `depth-guide.md` Section 5-6 + `prd-template.md` Section 5-6
- rbac-writer: `depth-guide.md` Section 7 + `prd-template.md` Section 7

`bug-patterns-filtered.md`가 존재하면 overview-writer, user-app-writer, admin-writer 모두에게 추가 전달.

---

## Phase 3: Round 1 — 초안 작성 (병렬)

4개 에이전트를 **동시에** 실행한다. (rbac-writer는 Phase 3.5에서 별도 실행)

```
┌──────────────────────────────────────────────┐
│  동시 실행 (Agent tool 병렬)                    │
│                                              │
│  overview-writer   → Section 0+1+2 초안       │
│  user-app-writer   → Section 3 초안           │
│  admin-writer      → Section 4 초안           │
│  tech-writer       → Section 5+6 초안         │
│                                              │
└──────────────────────────────────────────────┘
```

### 에이전트 프롬프트 구조

> 상세 프롬프트: `commands/references/generate-prd/prompt-templates.md` 참조

각 에이전트 프롬프트에 포함:
1. **Context**: 프로젝트 배경 + 중간 산출물 내용 (인라인)
2. **Goal**: 해당 Section 작성 (템플릿 형식 준수)
3. **Reference**: depth-guide의 해당 섹션 (인라인)
4. **Constraints**:
   - TBD 금지: 불명확 항목은 best practice 기반 권장안 작성, `[💡 권장적용]` 마킹
   - 입력파일에 없는 내용 추측 금지
   - 출력 형식은 `commands/references/generate-prd/prd-template.md` 엄격 준수
5. **Output Format**: 해당 Section의 마크다운 전문

### TBD 처리 규칙

- 입력파일에 명시되지 않은 항목 → best practice 기반 권장안 작성
- 마킹: `[💡 권장적용]` 을 권장안 옆에 표시
- 예시: `로그인 방식: Email + Password, 소셜 로그인 (Google, Apple) [💡 권장적용]`

### 버그 패턴 적용 규칙

`bug-patterns-filtered.md`가 존재하면:
- **user-app-writer**: 라우트별로 관련 버그 패턴의 예방 스펙을 `⚠️ 알려진 위험` 테이블로 삽입
- **admin-writer**: 관리 페이지에 관련 위험 패턴 삽입
- **overview-writer**: 시스템 모듈 플로우에 실패 분기 보강

### 결과 저장

각 에이전트 결과를 임시 파일로 저장:
- `.claude-project/prd/{ProjectName}/drafts/section-0-2.md`
- `.claude-project/prd/{ProjectName}/drafts/section-3.md`
- `.claude-project/prd/{ProjectName}/drafts/section-4.md`
- `.claude-project/prd/{ProjectName}/drafts/section-5-6.md`

---

## Phase 3.5: rbac-writer — Section 7 초안 (순차)

Section 3, 4 초안이 완성된 후, rbac-writer를 실행하여 Permission Matrix를 작성한다.

```
┌──────────────────────────────────────────────┐
│  순차 실행 (Section 3/4 초안 참조)               │
│                                              │
│  rbac-writer   → Section 7 초안               │
│  입력: parsed-input.md + feature-map.md       │
│        + permission-hints.md                 │
│        + section-3-draft + section-4-draft    │
│                                              │
└──────────────────────────────────────────────┘
```

### 결과 저장

- `.claude-project/prd/{ProjectName}/drafts/section-7.md`

---

## Phase 4: Round 2 — 스코프 한정 크로스 리뷰 (병렬)

각 에이전트는 **자기 전문 도메인의 관점에서만** 다른 Section을 리뷰한다. 범용 피드백 금지.

```
┌──────────────────────────────────────────────────────────┐
│  동시 실행 — 5-way 스코프 한정 리뷰                          │
│                                                          │
│  overview-writer (스코프: 용어 일관성만):                   │
│    읽기: section-3-draft + section-4-draft                │
│    체크: Section 1 용어집 ↔ Section 3/4 불일치             │
│                                                          │
│  user-app-writer (스코프: 기능 커버리지 + 권한 일치):        │
│    읽기: section-0-2-draft (모듈) + section-4-draft (관리)  │
│           + section-7-draft (권한)                        │
│    체크: feature-map 대비 누락 + 권한 매트릭스 일치          │
│                                                          │
│  admin-writer (스코프: 관리페이지 1:1 매핑만):              │
│    읽기: section-3-draft (라우트 목록)                     │
│    체크: CRUD 엔티티 대응 관리페이지 존재 여부               │
│                                                          │
│  tech-writer (스코프: 엔티티 커버리지):                    │
│    읽기: section-3-draft + section-4-draft                │
│           + section-0-2-draft (3rd Party)                 │
│    체크: CRUD 엔티티 ↔ Entity List + 3rd Party 매핑       │
│                                                          │
│  rbac-writer (스코프: 권한 완전성):                        │
│    읽기: section-3-draft + section-4-draft                │
│    체크: 수정/삭제 기능에 대응 Permission Matrix 항목 존재   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 피드백 출력 형식

```markdown
## Cross-Review Feedback (스코프: {리뷰 스코프})

### 발견 항목
| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
```

- **유형**: `누락` | `불일치` 만 허용 (주관적 "깊이 부족" 제거)
- **피드백 0건**: 빈 테이블 반환 → Phase 5에서 해당 에이전트 보강 스킵

### 결과 저장

- `.claude-project/prd/{ProjectName}/reviews/review-overview.md`
- `.claude-project/prd/{ProjectName}/reviews/review-user-app.md`
- `.claude-project/prd/{ProjectName}/reviews/review-admin.md`
- `.claude-project/prd/{ProjectName}/reviews/review-tech.md`
- `.claude-project/prd/{ProjectName}/reviews/review-rbac.md`

---

## Phase 5: Round 3 — 보강 (병렬)

피드백이 있는 에이전트만 실행. 피드백 0건인 에이전트는 스킵.

```
┌──────────────────────────────────────────────────────────┐
│  동시 실행 — 받은 피드백을 반영하여 자기 Section 보강          │
│                                                          │
│  overview-writer:                                        │
│    입력: section-0-2-draft + review-user-app              │
│           + review-admin 중 용어 관련 피드백               │
│    작업: 용어 추가, 모듈 플로우 보강, 누락 3rd Party 추가     │
│                                                          │
│  user-app-writer:                                        │
│    입력: section-3-draft + review-overview                │
│           + review-admin 중 기능 커버리지 피드백            │
│    작업: 빠진 라우트 추가, 컴포넌트 상세화, 엣지케이스 보강    │
│                                                          │
│  admin-writer:                                           │
│    입력: section-4-draft + review-overview                │
│           + review-user-app 중 매핑 피드백                │
│    작업: 빠진 관리페이지 추가, 테이블 컬럼 구체화             │
│                                                          │
│  tech-writer:                                            │
│    입력: section-5-6-draft + review-overview              │
│           + review-user-app 중 엔티티 피드백               │
│    작업: 누락 엔티티 추가, FK 관계 보강, 3rd Party 동기화    │
│                                                          │
│  rbac-writer:                                            │
│    입력: section-7-draft + review-user-app 중 권한 피드백  │
│    작업: 누락 Action×Role 추가, 소유권 규칙 보강             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 에이전트 프롬프트 구조

> 상세 프롬프트: `commands/references/generate-prd/prompt-templates.md` § 보강 모드 참조

1. **Context**: 자기 초안 전문 (인라인)
2. **Goal**: 피드백 항목을 반영하여 보강된 Section 전문 출력
3. **Feedback**: 받은 피드백 (인라인)
4. **Constraints**: 피드백에 없는 부분은 수정하지 않을 것
5. **Output Format**: 보강된 Section 전문 (기존 구조 유지)

### 결과 저장

- `.claude-project/prd/{ProjectName}/drafts/section-0-2-enhanced.md`
- `.claude-project/prd/{ProjectName}/drafts/section-3-enhanced.md`
- `.claude-project/prd/{ProjectName}/drafts/section-4-enhanced.md`
- `.claude-project/prd/{ProjectName}/drafts/section-5-6-enhanced.md`
- `.claude-project/prd/{ProjectName}/drafts/section-7-enhanced.md`

(피드백 0건으로 스킵된 Section은 draft를 그대로 enhanced로 복사)

---

## Phase 6: Synthesis + 검증 (순차 → 병렬)

### 6.1 prd-synthesizer (opus) — 순차

보강된 Section 0-2 + 3 + 4 + 5-6 + 7을 통합:

1. Section 0-8 조립
2. 교차참조 최종 검증 (용어집 ↔ 본문)
3. `[💡 권장적용]` 항목 요약표 생성
4. 용어 통일 (동일 개념에 다른 표현 사용 시 통일)
5. Additional Questions 섹션 생성
6. Feature Change Log 섹션 생성
7. **일관성 감사 (4개 하위 작업)**:
   - **수치 일관성**: 숫자+단위 패턴 추출 → 같은 개념 다른 값 발견 시 통일
   - **엔티티 관계 검증**: Section 6의 FK가 실제 존재하는 엔티티 참조하는지
   - **클라이언트 요구사항 추적**: parsed-input.md의 명시적 요구사항이 PRD에 존재하는지. 누락 시 보완 또는 Open Questions로 이동
   - **필러 감지**: 권장적용 요약표, UX 제안은 본문이 아닌 별첨(Appendix)으로 분리

결과: `.claude-project/prd/{ProjectName}/drafts/prd-integrated.md`

### 6.2 병렬 검증

통합 PRD 완성 후 3개 에이전트를 **동시에** 실행:

```
┌─────────────────────────────────────┐
│  동시 실행                           │
│  question-generator → 질문 목록 생성  │
│  ux-analyzer → UX 개선 제안 생성      │
│  qa → 12개 규칙 검증                 │
└─────────────────────────────────────┘
```

### 6.3 QA 검증 규칙 (기계 검증, 주관적 판단 없음)

> 상세 규칙: `commands/references/generate-prd/validation-checklist.md` 참조

| # | 규칙 | FAIL 조건 |
|---|------|----------|
| 1 | 라우트 수 일치 | `screen-inventory.md`에 있는 라우트가 PRD에 없음 |
| 2 | 필수 8항목 존재 | Section 3 라우트에 8항목 중 빠진 것 있음 |
| 3 | Admin 1:1 매핑 | Section 3의 CRUD 엔티티에 대응하는 관리페이지 없음 |
| 4 | Admin 표준기능 | 관리페이지에 표준기능 빠짐 |
| 5 | 용어 교차참조 | 용어집에만 있거나 본문에만 있는 용어 존재 |
| 6 | TBD 제로 | "TBD", "미정", "추후 결정" 등 빈 항목 존재 |
| 7 | 라우트 커버리지 | Page Map ↔ Feature List 불일치 |
| 8 | 수치 일관성 | 같은 개념이 서로 다른 값 |
| 9 | Tech Stack 완전성 | Section 5 비어있거나 3rd Party 누락 |
| 10 | 엔티티 커버리지 | CRUD 엔티티가 Section 6에 없음 |
| 11 | 권한 완전성 | 수정/삭제 기능에 대응 Permission Matrix 항목 없음 |
| 12 | 클라이언트 요구사항 추적 | 명시적 요구사항이 PRD/Open Questions에 없음 |

### QA 출력 형식

```markdown
## QA Validation Results

| # | 규칙 | Verdict | Evidence |
|---|------|---------|----------|
| 1 | 라우트 수 일치 | PASS/FAIL | {근거} |
...

Overall: PASS / FAIL
```

### FAIL 처리

1. qa FAIL → **support 에이전트**(sonnet)가 FAIL 항목만 수정
2. support 수정 후 → qa 재검증
3. **최대 3라운드** 반복
4. 3라운드 후에도 FAIL:
   ```
   ## QA 검증 실패 — 수동 개입 필요

   ### 반복 실패 항목
   - 규칙 {N}: {규칙명} — {실패 사유}

   ### 시도한 수정
   1. {시도 1}
   2. {시도 2}
   3. {시도 3}

   ### 권장 조치
   {수동으로 해야 할 것}
   ```

---

## Phase 6.5: 업데이트 모드 (v1 단순화)

- Phase 1에서 기존 PRD를 발견하고 사용자가 "참고하여 새로 생성"을 선택한 경우에만 해당
- 기존 PRD 파일을 Phase 3 에이전트에 추가 컨텍스트로 전달
- 에이전트가 기존 PRD를 참고하여 신규 입력 기반으로 **처음부터 새로 작성**
- Change Log에 "이전 버전 참고하여 재생성" 기록
- Change Log 자동 생성, diff 비교 등 고급 기능은 **v2에서 구현**

---

## Phase 7: 권장적용 일괄 확인

QA PASS 후, `[💡 권장적용]` 항목을 사용자에게 일괄 제시:

```markdown
## 권장적용 항목 확인

다음 항목들은 입력파일에 명시되지 않아 best practice 기반으로 권장안을 적용했습니다.
승인하시면 `[📌 권장적용됨]` 마커로 변환되어 정식 스펙에 포함됩니다.
거부하시면 Additional Questions로 이동합니다.

| # | 항목 | 권장안 | 근거 | 승인? |
|---|------|--------|------|-------|
| 1 | 로그인 방식 | Email+PW + 소셜(Google, Apple) | B2C 앱 표준 | ✅/❌ |
| 2 | 비밀번호 규칙 | 8자+, 영문+숫자+특수문자 | OWASP 가이드라인 | ✅/❌ |
| ... | | | | |
```

AskUserQuestion으로 확인 후:
- **승인** → `[💡 권장적용]` → `[📌 권장적용됨]` 마커 변환 (출처 유지)
- **거부** → Additional Questions 섹션으로 이동
- **수정** → 사용자 수정 내용 반영 + `[📌 권장적용됨]` 마커 유지

> **마커 변환 규칙**: 승인된 항목의 `[💡 권장적용]`을 `[📌 권장적용됨]`으로 **일괄 치환**.
> 이는 최종 PRD에서 "이 스펙이 클라이언트 직접 요구가 아닌 AI 권장에서 비롯됨"을 표시한다.
> 개발자/이해관계자가 추후 스펙 변경 시 우선순위 판단 근거로 활용 가능.

---

## Phase 8: Delivery

### 8.1 UX 분석 포함 여부

AskUserQuestion으로 확인:
```
PRD 생성이 완료되었습니다. UX 플로우 분석을 포함하시겠습니까?

분석 내용:
- 네비게이션 플로우 이슈
- 유저 여정 최적화
- 정보 구조 개선
- 일반적 UX 문제 패턴

[Yes] / [No]
```

- Yes → ux-analyzer 결과를 PRD 끝에 추가
- No → 스킵

### 8.2 최종 파일 저장

파일명: `[AppName]_PRD_[YYMMDD].md`
- 앱 이름 sanitize: 공백 → 언더스코어, 특수문자 제거
- 위치: `.claude-project/prd/{ProjectName}/`

### 8.3 임시 파일 정리

중간 산출물과 초안은 보존한다 (추후 디버깅/감사 추적용).
단, 사용자가 명시적으로 요청하면 정리:

```bash
rm -rf .claude-project/prd/{ProjectName}/intermediate/
rm -rf .claude-project/prd/{ProjectName}/drafts/
rm -rf .claude-project/prd/{ProjectName}/reviews/
```

### 8.4 결과 리포트

```markdown
## PRD Generation Complete

### Output
- File: `.claude-project/prd/{ProjectName}/[filename].md`
- Mode: [New / Update Reference]

### Summary
- Section 0: Project Overview ✅
- Section 1: Terminology ✅
- Section 2: System Modules ({N} modules) ✅
- Section 3: User Application ({N} routes) ✅
- Section 4: Admin Dashboard ({M} pages) ✅
- Section 5: Tech Stack ✅
- Section 6: Data Model ({K} entities) ✅
- Section 7: Permission Matrix ({R} roles × {A} resources) ✅
- Section 8: Open Questions ({Q} items) ✅
- QA Validation: PASS ({N} rounds, 12 rules)
- 권장적용 항목: {N}개 승인 [📌 권장적용됨], {M}개 질문으로 이동
- Additional Questions: {N} items
- UX Suggestions: {N} items (or "Skipped")
- 버그 예방 섹션: {N} routes에 적용 (or "bug-patterns 없음")

### Output Structure
```
.claude-project/prd/{ProjectName}/
├── intermediate/           ← 중간 산출물
├── drafts/                 ← 에이전트 초안/보강본
├── reviews/                ← 크로스 리뷰 피드백
└── {AppName}_PRD_{YYMMDD}.md  ← 최종 PRD
```

### Next Steps
1. Additional Questions 섹션을 검토하세요
2. 클라이언트에게 질문을 전달하세요
3. 답변을 받으면 `/generate-prdv2 [answer-file]`로 업데이트하세요
```

---

## 에러 핸들링

| 상황 | 대응 |
|------|------|
| 에이전트 타임아웃/실패 | 1회 재시도. 재실패 시 "⚠️ 생성 실패" 마킹, 나머지 진행 |
| Round 2 피드백 0건 | 정상 — Round 3에서 해당 에이전트 스킵 |
| QA 3라운드 FAIL | 사용자에게 FAIL 항목 리스트 + 수동 개입 요청 |
| 입력파일 파싱 불가 | 에러 메시지 + 지원 포맷 안내 후 중단 |
| bug-patterns 없음 | 정상 — 버그 예방 섹션 없이 생성 (경고 출력) |
| 기존 PRD 앱 이름 불일치 | AskUserQuestion으로 확인 후 진행 |
| 업데이트 모드에서 기존 PRD 파싱 실패 | 경고 출력 후 신규 모드로 폴백 |
| prd-synthesizer 토큰 초과 | Section별 순차 조립 후 일관성 감사만 별도 실행 |

---

## Reference 파일 목록

| 파일 | 용도 |
|------|------|
| `commands/references/generate-prd/prd-template.md` | Section 0-8 아웃풋 구조 |
| `commands/references/generate-prd/prompt-templates.md` | 에이전트 명세 + 작성/리뷰/보강 프롬프트 |
| `commands/references/generate-prd/depth-guide.md` | 필수 밀도 기준 |
| `commands/references/generate-prd/admin-standards.md` | Admin 표준기능 매트릭스 |
| `commands/references/generate-prd/validation-checklist.md` | 기계 검증 가능한 12개 QA 규칙 |
