# Module List

## 시스템 모듈

| 모듈명 | 설명 | 관련 유저타입 |
|--------|------|-------------|
| Authentication | 로그인, 회원가입, 비밀번호 관리, 초대 수락 | All |
| Organization Management | 조직 생성/수정, 멀티테넌시 데이터 격리 | Owner |
| Project Management | 프로젝트 CRUD, 멤버 배정 | Owner, Admin |
| Screen Management | 화면 등록/삭제, 스크린샷 버전 업로드/관리 | Owner, Admin, Member |
| Feedback System | 핀 생성, 코멘트 작성, 답글, 상태 관리 (open/in_progress/resolved) | All |
| Version Control | 스크린샷 버전 히스토리, 버전 비교 (overlay/side-by-side) | All |
| Team & Invitation | 이메일 초대, 역할 관리, 멤버 추가/제거 | Owner, Admin |
| Notification | 이메일 알림, 인앱 알림 (코멘트, 상태 변경, 초대) | All |
| Realtime | WebSocket 기반 실시간 핀/코멘트/상태 업데이트 | All |
| Billing | Stripe 구독결제 (추후) | Owner |
| Audit Log | 주요 액션 기록 (생성/수정/삭제/상태변경) | Owner, Admin, SuperAdmin |
| Super Admin | 전체 조직 조회, 상태 관리 | SuperAdmin |
| File Attachment | 코멘트 파일 첨부 | All |
| i18n | 다국어 지원 (EN/KO) | All |

## 3rd Party API

| 서비스 | 목적 | 연동 포인트 |
|--------|------|-----------|
| Stripe | 구독결제 (추후) | Billing 모듈 |
| Email Service (SendGrid/SES) | 이메일 초대, 알림 발송 | Invitation, Notification 모듈 |
| Cloud Storage (S3/GCS) | 스크린샷, 첨부파일 저장 | Screen Management, File Attachment 모듈 |
