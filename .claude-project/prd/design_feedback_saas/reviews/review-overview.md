## Cross-Review Feedback (스코프: 용어 일관성)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 불일치 | Section 4 — 4.2.2.1 Top Area, 4.2.2.2 Table Component, 4.2.3.2 Detail Section, 4.2.4.1 Top Area, 4.2.4.2 Table Component, 4.2.5.2 Detail Section, 4.2.1.3 Charts (Donut Chart) | `Inactive` 상태값이 반복 사용됨. Section 1의 `organization_status`는 `active`, `suspended`, `deleted`만 정의하며, `user_status`도 `active`, `invited`, `suspended`, `deleted`만 정의함. `inactive`는 정의된 Enum 값이 아님. | Section 1에 `inactive` 값을 추가하거나, Section 4에서 `inactive`를 정의된 값(`suspended` 또는 `deleted`)으로 대체. |
| 2 | 누락 | Section 4 — 4.2.6.1 Top Area, 4.2.6.2 Table Component, 4.2.6.4 Bulk Action | `Archived` 및 `아카이브` 상태가 프로젝트 상태값으로 사용됨. Section 1에 `project_status` Enum이 정의되어 있지 않으며, Module 3에도 `archived` 상태가 없고 Soft Delete만 언급됨. | Section 1에 `project_status` Enum(`active`, `archived`, `deleted`)을 추가하거나, Section 4에서 `archived`를 제거하고 soft delete로 통일. |
| 3 | 누락 | Section 4 — 4.2.2.1 Top Area, 4.2.2.2 Table Component, 4.2.2.4 Creation Modal, 4.2.3.2 Detail Section | `슬러그(slug)`가 Organization의 속성으로 반복 사용됨. Section 1 용어집 및 Section 2 Module 2에 `slug` 또는 `슬러그`가 정의되어 있지 않음. | Section 1 Core Concepts 또는 Technical Terms에 `Slug` 정의 추가 (예: "조직을 URL에서 식별하는 고유 문자열 식별자"). |
| 4 | 누락 | Section 4 — 4.2.2.1 Top Area, 4.2.2.2 Table Component, 4.2.2.4 Creation Modal, 4.2.3.2 Detail Section, 4.2.3.4 탭 구성 (빌링 탭), 4.2.8.1 탭 구성 (빌링/플랜 설정) | `Plan` (Free / Pro / Enterprise)이 Organization의 속성으로 반복 사용됨. Section 1에 `organization_plan` 또는 `plan` Enum이 정의되어 있지 않음. Section 1 Module 12 (Billing)에서도 Plan 언급이 없음. | Section 1 Status Values에 `organization_plan` Enum(`free`, `pro`, `enterprise`)을 추가하거나, 해당 값이 TBD임을 명시. |
| 5 | 누락 | Section 3 — `/notifications` 알림 유형 목록 (line: "멘션: 코멘트에서 멘션되었을 때") | `멘션(mention)` 알림 유형이 Section 3에서 언급됨. Section 1의 `notification_type` Enum에는 `comment_created`, `comment_replied`, `comment_status_changed`, `invitation_received`, `version_uploaded`만 정의되어 있으며 `mention`(또는 `comment_mentioned`)이 없음. | Section 1의 `notification_type` Enum에 `comment_mentioned` 항목을 추가하거나, Section 3에서 해당 알림 유형을 제거. |
