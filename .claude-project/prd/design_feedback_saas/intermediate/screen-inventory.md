# Screen Inventory

## 유저앱 라우트 (총 20개)

| # | 라우트 | 페이지명 | 유저타입 | 접근 그룹 | 비고 |
|---|--------|---------|---------|----------|------|
| 1 | `/auth/login` | 로그인 | All | Auth | |
| 2 | `/auth/register` | 회원가입 | All | Auth | |
| 3 | `/auth/forgot-password` | 비밀번호 찾기 | All | Auth | |
| 4 | `/auth/reset-password` | 비밀번호 재설정 | All | Auth | |
| 5 | `/auth/accept-invite` | 초대 수락 | All | Auth | 이메일 링크 |
| 6 | `/dashboard` | 대시보드 | Owner, Admin, Member | Protected | |
| 7 | `/projects` | 프로젝트 목록 | Owner, Admin, Member | Protected | |
| 8 | `/projects/:id` | 프로젝트 상세 | Owner, Admin, Member | Protected | |
| 9 | `/projects/:id/screens` | 화면 목록 | Owner, Admin, Member | Protected | |
| 10 | `/projects/:id/screens/:screenId` | 피드백 뷰어 | Owner, Admin, Member | Protected | 핵심 화면 |
| 11 | `/projects/:id/members` | 프로젝트 멤버 관리 | Owner, Admin | Protected | |
| 12 | `/settings` | 설정 메인 | Owner, Admin, Member | Protected | |
| 13 | `/settings/organization` | 조직 설정 | Owner | Protected | |
| 14 | `/settings/team` | 팀 관리 | Owner, Admin | Protected | |
| 15 | `/settings/billing` | 빌링/결제 | Owner | Protected | |
| 16 | `/settings/profile` | 프로필 설정 | All authenticated | Protected | |
| 17 | `/client/projects` | 클라이언트 프로젝트 목록 | Client | Protected | 배정된 것만 |
| 18 | `/client/projects/:id/screens` | 클라이언트 화면 목록 | Client | Protected | |
| 19 | `/client/projects/:id/screens/:screenId` | 클라이언트 피드백 뷰어 | Client | Protected | 핀+코멘트 작성 |
| 20 | `/notifications` | 알림 목록 | All authenticated | Protected | |

## 어드민 라우트 (총 12개)

| # | 라우트 | 페이지명 | 관리 대상 | 비고 |
|---|--------|---------|----------|------|
| 1 | `/admin` | 슈퍼어드민 대시보드 | 전체 통계 | |
| 2 | `/admin/organizations` | 조직 관리 | Organization | |
| 3 | `/admin/organizations/:id` | 조직 상세 | Organization | 상태 변경 |
| 4 | `/admin/users` | 사용자 관리 | User | 전체 사용자 |
| 5 | `/admin/users/:id` | 사용자 상세 | User | |
| 6 | `/admin/projects` | 프로젝트 관리 | Project | 전체 프로젝트 |
| 7 | `/admin/projects/:id` | 프로젝트 상세 | Project | 소속 화면/코멘트 링크 |
| 8 | `/admin/screens` | 화면 관리 | Screen | 전체 화면 목록 |
| 9 | `/admin/comments` | 코멘트 관리 | Comment | 신고 처리 포함 |
| 10 | `/admin/invitations` | 초대 관리 | Invitation | 초대 만료/취소 |
| 11 | `/admin/audit-logs` | 감사 로그 | AuditLog | |
| 12 | `/admin/settings` | 플랫폼 설정 | Platform | |
