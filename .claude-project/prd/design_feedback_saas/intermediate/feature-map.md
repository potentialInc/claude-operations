# Feature-Screen Mapping

## Owner
| 기능 | 화면 | 비고 |
|------|------|------|
| 조직 관리 | Settings > Organization | 조직 정보 수정 |
| 빌링 관리 | Settings > Billing | Stripe 결제 연동 (추후) |
| 멤버 관리 | Settings > Team | 초대, 역할 변경, 제거 |

## Admin
| 기능 | 화면 | 비고 |
|------|------|------|
| 프로젝트 생성/관리 | Projects > List / Detail | CRUD |
| 팀원/클라이언트 초대 | Settings > Team / Project Detail > Members | 이메일 초대 |
| 피드백 관리 | Project > Screen > Feedback Viewer | 상태 변경, 삭제 |
| 화면 관리 | Project > Screens | 화면 추가/삭제, 버전 업로드 |

## Member
| 기능 | 화면 | 비고 |
|------|------|------|
| 화면 등록/관리 | Project > Screens | 화면 추가 |
| 스크린샷 버전 업로드 | Screen Detail > Version Upload | 버전 히스토리 |
| 핀 코멘트 답변 | Feedback Viewer | 답글 작성 |
| 코멘트 상태 변경 | Feedback Viewer | open/in_progress/resolved |

## Client
| 기능 | 화면 | 비고 |
|------|------|------|
| 내 프로젝트 목록 | Client > Projects | 배정된 것만 |
| 화면 목록 | Client > Project > Screens | 읽기 전용 목록 |
| 피드백 뷰어 | Client > Feedback Viewer | 핀 찍기 + 코멘트 작성 |

## Super Admin
| 기능 | 화면 | 비고 |
|------|------|------|
| 전체 조직 조회 | SuperAdmin > Organizations | 조직 목록/상세 |
| 조직 상태 관리 | SuperAdmin > Organization Detail | 활성/비활성/삭제 |
| 전체 사용자 관리 | SuperAdmin > Users | 사용자 목록/상세, 계정 정지/삭제 |
| 전체 프로젝트 관리 | SuperAdmin > Projects / Project Detail | 프로젝트 목록/상세, 강제 삭제 |
| 전체 화면 관리 | SuperAdmin > Screens | 화면 목록, 필터, 강제 삭제 |
| 코멘트 관리 | SuperAdmin > Comments | 신고 처리, 강제 삭제 |
| 초대 관리 | SuperAdmin > Invitations | 초대 목록, 취소/만료 처리 |
| 감사 로그 조회 | SuperAdmin > Audit Logs | 읽기 전용, CSV/Excel 내보내기 |
| 플랫폼 설정 | SuperAdmin > Settings | 보안/파일/빌링/유지보수 설정 |
