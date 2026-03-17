---
description: "Learn bug patterns from reports and accumulate into knowledge base"
argument-hint: "Path to bug reports (file, folder) or external source (notion://db-id)"
allowed-tools: Agent, Read, Write, Edit, Glob, Grep, Bash(mkdir *), WebFetch, mcp__claude_ai_Notion__*
---

# Learn Bugs — 누적 버그 학습 시스템

버그리포트를 분석하여 패턴화된 지식으로 변환하고, 반복 실행 시 누적 업데이트한다.

## 핵심 원칙

1. **누적 학습**: 매 실행마다 기존 지식에 추가 (덮어쓰기 아님)
2. **중복 방지**: 동일 패턴은 빈도수 증가 + 새 사례만 추가
3. **구조화**: `commands/references/generate-prd/bug-pattern-schema.md` 형식 엄격 준수
4. **즉시 활용**: `/generate-prdv2` 실행 시 자동 참조

---

## Workflow

```
Phase 1: 입력 소스 파싱
       ↓
Phase 2: bug-analyzer 에이전트 실행
       ↓
Phase 3: knowledge 파일 업데이트
       ↓
Phase 4: 학습 결과 리포트 출력
```

---

## Phase 1: 입력 소스 파싱

`$ARGUMENTS`에서 입력 소스를 파싱한다.

### 입력 유형 판별

| 패턴 | 유형 | 처리 |
|------|------|------|
| `/path/to/file.md` | 단일 파일 | 파일 직접 읽기 |
| `/path/to/folder/` | 폴더 | Glob으로 `**/*.{md,csv,json,txt}` 파일 수집 |
| `notion://database-id` | Notion DB | MCP `notion-query-database-view` 호출 |
| `linear://project-slug` | Linear | Linear MCP 호출 |

### --global 플래그

- `--global` 포함 시: 글로벌 패턴 파일에 저장
  - 위치: `commands/references/generate-prd/bug-patterns-global.md`
- 미포함 시 (기본): 프로젝트별 패턴 파일에 저장
  - 위치: `.claude-project/knowledge/bug-patterns.md`

### 유효성 검증

1. 파일/폴더 존재 확인
2. 지원 포맷 확인
3. MCP 소스의 경우 MCP 서버 연결 확인

**검증 실패 시:**
```
Error: [구체적 에러 메시지]
Usage: /learn-bugs /path/to/bug-reports/
Supported formats: .md, .csv, .json, .txt
External sources: notion://database-id, linear://project-slug
```

**MCP 미설정 시:**
```
Error: Notion MCP server is not configured.
To use Notion as a bug report source, add to .mcp.json:

{
  "mcpServers": {
    "notion": {
      "type": "stdio",
      "command": "npx",
      "args": ["@anthropic/claude-ai-notion"]
    }
  }
}

Alternative: Use file/folder input instead.
/learn-bugs /path/to/bug-reports/
```

---

## Phase 2: bug-analyzer 에이전트 실행

수집된 버그리포트를 bug-analyzer 에이전트에게 전달한다.

### 에이전트 설정

| 항목 | 값 |
|------|-----|
| **Model** | sonnet |
| **Tools** | Read, Write, Glob, Grep, WebFetch |

### 에이전트 프롬프트

```
## Context
당신은 버그리포트를 분석하여 패턴화된 지식으로 변환하는 에이전트입니다.

### 버그리포트 내용
{수집된 버그리포트 전문}

### 기존 패턴 지식 (있을 경우)
{기존 bug-patterns.md 내용 또는 "없음 (첫 학습)"}

### 패턴 분류 스키마
{bug-pattern-schema.md 내용 — commands/references/generate-prd/bug-pattern-schema.md 에서 Read}

## Goal
1. 각 버그리포트를 분류하고 패턴화하세요
2. 기존 패턴과 중복 여부를 확인하세요
3. 중복 패턴: 빈도수 증가 + 새 사례 추가
4. 새 패턴: 신규 항목 추가
5. 업데이트된 전체 bug-patterns.md를 출력하세요

## Constraints
- bug-pattern-schema.md 형식 엄격 준수
- 기존 패턴의 내용을 삭제하지 않음 (누적만)
- 각 패턴에 반드시 예방 스펙 포함
- Last updated, Total patterns, Sources 메타데이터 업데이트

## Output Format
1. 업데이트된 bug-patterns.md 전문
2. 학습 결과 요약 (아래 형식)

### 학습 결과
- 분석한 버그리포트: {N}건
- 새로운 패턴 추가: {N}건
- 기존 패턴 빈도 업데이트: {N}건
- 무시 (중복/부적합): {N}건
- 새로 추가된 패턴 목록: [{카테고리: 패턴명}]
```

---

## Phase 3: knowledge 파일 업데이트

### 저장 위치 결정

```
--global 플래그 여부에 따라:

글로벌:
  → commands/references/generate-prd/bug-patterns-global.md

프로젝트별:
  mkdir -p .claude-project/knowledge/
  → .claude-project/knowledge/bug-patterns.md
```

### 파일 저장

bug-analyzer의 출력을 해당 위치에 Write로 저장한다.

---

## Phase 4: 학습 결과 리포트 출력

```markdown
## Bug Learning Complete

### 이번 학습
- 입력 소스: {소스 경로/URL}
- 분석한 버그리포트: {N}건
- 새로운 패턴 추가: {N}건
- 기존 패턴 빈도 업데이트: {N}건
- 무시 (중복/부적합): {N}건

### 누적 현황
- 전체 패턴: {N}개 (카테고리 {M}개)
- 고빈도 패턴 (5회+): {N}개
- 소스: {N} 프로젝트, {M} 버그리포트

### 새로 추가된 패턴
| # | 카테고리 | 패턴 | 예방 스펙 요약 |
|---|---------|------|-------------|

### 활용 안내
이 패턴들은 `/generate-prdv2` 실행 시 자동으로 참조됩니다.
각 화면에 관련 패턴의 예방 스펙이 `⚠️ 알려진 위험` 섹션으로 삽입됩니다.
```

---

## 에러 핸들링

| 상황 | 대응 |
|------|------|
| 파일 없음 | 에러 메시지 + 사용법 출력 |
| 빈 파일 | 스킵, 경고 메시지 |
| MCP 미설정 | 설정 가이드 출력 |
| MCP 연결 실패 | 에러 메시지 + 파일 입력 대안 안내 |
| 파싱 불가 파일 | 스킵, 경고 메시지 (나머지 파일 계속 처리) |
| 기존 bug-patterns.md 손상 | 백업 후 새로 생성 |

---

## /generate-prdv2에서의 활용

PRD 생성 시 bug-patterns는 다음과 같이 활용된다:

1. **Phase 2** (중간 산출물 생성): `bug-patterns-filtered.md` 생성
   - 글로벌 + 프로젝트별 패턴 병합
   - 프로젝트별 패턴이 글로벌과 충돌 시 프로젝트별 우선
   - 입력파일의 화면 유형에 관련된 카테고리만 필터링

2. **Phase 3** (초안 작성): 각 에이전트가 `bug-patterns-filtered.md` 참조
   - user-app-writer: 화면별 `⚠️ 알려진 위험` 테이블 삽입
   - admin-writer: 관리 페이지에 관련 위험 패턴 삽입
   - basic-info-writer: 모듈 플로우에 실패 분기 보강
