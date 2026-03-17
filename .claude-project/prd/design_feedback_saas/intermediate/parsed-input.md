# Parsed Input

## Basic Info
- **앱 이름**: (미지정 — 디자인 피드백 SaaS)
- **타입**: SaaS Web Platform
- **마감일**: 미지정
- **유저타입**: Owner, Admin, Member, Client
- **관계**: 조직(Organization) 기반 멀티테넌시

## Design Reference
- **참고 앱**: 없음
- **차별점**: 없음
- **색상/로고**: 미지정
- **폰트**: 미지정

## Features

### Owner
- 조직 생성/관리
- 빌링/결제 관리
- 멤버 관리 (초대, 역할 변경, 제거)

### Admin
- 프로젝트 생성/관리
- 팀원/클라이언트 초대
- 피드백 관리 (상태 변경, 삭제)
- 화면(Screen) 관리

### Member
- 화면(Screen) 등록/관리
- 스크린샷 버전 업로드
- 핀 코멘트 답변
- 코멘트 상태 변경 (open → in_progress → resolved)

### Client
- 자기 배정 프로젝트만 열람
- 이미지 위 핀 찍기 + 코멘트 작성
- 코멘트 답글

### 공통 기능
- 이메일 초대, 알림
- 실시간 반영 (실시간 코멘트/핀 업데이트)
- 추후 Stripe 구독결제 연동 예정

### 핵심 기능 플로우
1. 조직 생성 → 프로젝트 생성 → 화면(Screen) 등록
2. 화면마다 스크린샷 버전 업로드
3. 이미지 위에 핀(좌표) 찍고 코멘트 작성
4. 코멘트에 답글 가능
5. 상태 관리: open / in_progress / resolved

### 피드백 뷰어 (핵심 화면)
- 좌측: 이미지 (핀 표시)
- 우측: 코멘트 패널
- 핀 클릭 → 코멘트 하이라이트
- 코멘트 클릭 → 해당 핀 하이라이트
- 버전 선택 가능
- 버전 비교 (overlay / side-by-side)

## Data
- **수집 데이터**: 조직, 프로젝트, 화면, 스크린샷 버전, 핀, 코멘트, 답글, 사용자, 초대, 멤버십
- **내보내기 데이터**: 미지정

## Technical
- **3rd Party**: Stripe (추후), 이메일 서비스 (초대/알림)
- **도메인 용어**: Organization, Project, Screen, Version, Pin, Comment, Reply

## Non-functional Requirements
- Soft delete
- Audit log
- 파일 첨부 (코멘트에)
- i18n (EN/KO)
- 반응형 웹
- 조직별 데이터 완전 분리 (멀티테넌시)

## Future Considerations
- 사진 뿐만 아니라 디자인 작업(Figma 등) 리뷰 가능하도록 확장 가능 구조
