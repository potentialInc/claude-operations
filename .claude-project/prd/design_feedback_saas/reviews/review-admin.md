## Cross-Review Feedback (스코프: 관리페이지 1:1 매핑)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 누락 | Section 3: `/projects/:id/screens` (AddScreenModal, 수정, 삭제) | Screen 엔티티는 생성/수정/삭제가 가능하나 Section 4에 `/admin/screens` 관리페이지가 없음 | `/admin/screens` 목록 페이지 추가 (조직·프로젝트 필터, 상태 관리, 삭제 포함) |
| 2 | 누락 | Section 3: `/projects/:id/screens/:screenId` (CommentPanel, 상태변경, 삭제) | Comment 엔티티는 생성/상태변경/삭제가 가능하나 Section 4에 `/admin/comments` 관리페이지가 없음 | `/admin/comments` 목록 페이지 추가 (조직·프로젝트·화면 필터, 상태 변경, 삭제 포함) |
| 3 | 누락 | Section 4: `/admin/projects` (Page Map) | Project 엔티티는 `/admin/projects` 목록만 있고 `/admin/projects/:id` 상세 페이지가 없음. Organization·User는 각각 상세 페이지가 있어 일관성 불일치 | `/admin/projects/:id` 상세 페이지 추가 (소속 화면 목록, 멤버 목록, 상태 변경, 삭제 포함) |
| 4 | 누락 | Section 3: `/projects/:id/members`, `/settings/team` (InviteMemberModal, InviteTeamMemberModal) | Invitation(초대) 엔티티는 생성·재전송·취소가 가능하나 Section 4에 초대 현황 관리페이지가 없음 | `/admin/invitations` 또는 조직 상세(`/admin/organizations/:id`) 내 초대 탭 추가 |
