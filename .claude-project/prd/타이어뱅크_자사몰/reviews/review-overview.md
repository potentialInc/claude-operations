## Cross-Review Feedback (스코프: 용어 일관성)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 불일치 | Section 4 `/admin/settlements` — 정산 상태 필터 및 테이블 badge | 용어집 `SettlementStatus`는 `PENDING / PROCESSED / COMPLETED` 3단계인데, Section 4에서는 "정산대기 / 정산완료 / **정산취소**"로 표기하여 `정산취소` 상태가 불일치. | 용어집에 `CANCELLED` 또는 `REJECTED` 상태를 추가하거나, Section 4 필터 옵션을 용어집 Enum과 일치하도록 수정. |
| 2 | 누락 | Section 4 `/admin/settlements` — Detail Drawer 가능한 액션 및 Delete Flow | "정산 취소 (사유 입력 필수)" 액션과 "취소는 상태 변경(정산취소)으로 처리" 설명이 있으나, 용어집 `SettlementStatus`에 취소 상태가 정의되지 않음. | 용어집 `SettlementStatus`에 `CANCELLED` 상태와 정의 추가 필요. |
| 3 | 불일치 | Section 4 `/admin/shops` — 승인 상태 필터 및 테이블 badge | 용어집 `PartnerStatus`는 `PENDING / ACTIVE / SUSPENDED / WITHDRAWN` 4가지인데, Section 4에서는 "승인대기(Pending) / 승인(Active) / **거절(Rejected)** / 정지(Suspended)"로 `Rejected` 상태가 추가됨. | 용어집 `PartnerStatus`에 `REJECTED` 상태를 추가하거나, Section 4 필터에서 해당 값을 제거하고 `WITHDRAWN` 처리로 대체. |
| 4 | 누락 | Section 4 `/admin/shops` — Creation Modal 승인 상태 선택 필드 | 생성 모달에서 "승인 상태: Pending / Active / Rejected" 선택 옵션으로 `Rejected`를 사용하나 용어집 `PartnerStatus`에 없음. | 항목 3과 동일하게 용어집에 `REJECTED` 추가 또는 해당 선택지 제거. |
| 5 | 불일치 | Section 3 `/partner/reservations` 및 `/partner/reservations/:id` — 예약 상태 badge | 용어집 `ReservationStatus`는 `PENDING / CONFIRMED / COMPLETED / CANCELLED` 4가지인데, Section 3 파트너 화면에서 "예약확인대기 / 예약확정 / **장착중** / 장착완료 / 취소" 5단계를 사용하여 `장착중` 상태가 누락됨. | 용어집 `ReservationStatus`에 `IN_PROGRESS` 또는 이에 상응하는 "장착중" 상태를 추가하거나, Section 3에서 해당 상태를 제거하고 `CONFIRMED` → `COMPLETED` 직전환으로 수정. |
| 6 | 불일치 | Section 3 `/partner/settlements` — 정산 내역 테이블 상태 badge | 용어집 `SettlementStatus`의 한국어 대응이 `PENDING=정산대기`, `PROCESSED=정산처리 중`, `COMPLETED=정산완료`인데, Section 3에서는 "정산대기 / 정산완료 / **지급완료**" 3단계를 사용. `지급완료`는 용어집에 없는 표현. | 용어집 `SettlementStatus`의 `COMPLETED` 한국어 표현을 "지급완료"로 통일하거나, `지급완료`를 별도 상태로 추가 정의. |
| 7 | 누락 | Section 4 `/admin/inquiries` — Detail Drawer 가능한 액션 및 Delete Flow | "슈퍼어드민(superadmin)"이라는 역할이 언급되나 ("슈퍼어드민은 삭제 가능"), 용어집 User Roles에 해당 역할이 정의되지 않음. | 용어집 User Roles에 `SuperAdmin` 역할을 추가하거나, Section 4에서 해당 표현을 "관리자(Admin)" 권한 수준 차이로 재정의. |
| 8 | 불일치 | Section 3 `/partner` — 파트너 라우트 그룹 명칭 및 페이지 제목 | 용어집에 파트너 전용 인터페이스는 "파트너 포털"로 정의됐는데, Section 3 Page Map에서 해당 구역을 "Protected (Partner)"로 표기하고 대표 페이지를 "파트너 대시보드"라고 명명. 동일 Section 2 (Module 8, 9, 10)에서 "파트너 포털"을 사용하는 것과 혼재. | Section 3에서 파트너 전용 구역의 명칭을 "파트너 포털"로 통일하거나, 용어집에 "파트너 포털"과 "파트너 대시보드"의 관계를 명시. |
| 9 | 불일치 | Section 4 `/admin/products` — 테이블 컬럼명 및 Creation Modal 필드명 | 용어집에서 타이어 규격 표기는 "타이어 사이즈"(예: 205/55R16)로 정의됐는데, Section 4 상품 관리에서는 동일 개념을 "규격"으로 표기 (테이블 컬럼 "규격", 검색 플레이스홀더 "타이어 규격"). | "규격" 표현을 "타이어 사이즈"로 통일하거나, 용어집에 "규격"을 "타이어 사이즈"의 동의어로 명시. |
