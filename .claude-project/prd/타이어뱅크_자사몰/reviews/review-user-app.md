## Cross-Review Feedback (스코프: 기능 커버리지 + 권한 일치)

> **리뷰어 역할:** Section 3 (User Application) 작성자
> **검토 기준:** feature-map.md 기능 커버리지 + Section 4 Admin 대응 화면 + Section 7 Permission Matrix 권한 일치
> **검토 일자:** 2026-03-13

---

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 기능 커버리지 누락 | feature-map.md → Section 3 | feature-map의 "리뷰 작성" 기능이 `마이페이지 > 리뷰` 화면으로 명시되어 있으나, Section 3의 `/mypage/reviews`는 리뷰 조회/수정/삭제만 라우트로 정의하고, 리뷰 작성은 주문 상세(`/mypage/orders/:id`) 내 모달로만 처리함. 독립적인 "리뷰 작성" 진입점 라우트가 없어 feature-map 기술과 불일치 | `/mypage/reviews/write` 라우트 추가 또는 `/mypage/reviews` 내 리뷰 작성 가능 상품 목록 탭을 명시적으로 Section 3에 정의 |
| 2 | 기능 커버리지 누락 | feature-map.md → Section 3 | feature-map "일반 고객" 항목에 "리뷰 목록 조회 — 구매후기" 화면이 있고 Section 3에 `/reviews` Public 라우트가 존재하지만, 상품 상세(`/tire/:id`) 리뷰 탭 이외에 고객이 본인 외 전체 리뷰를 탐색하는 경로가 Section 3 GNB 메뉴에는 "구매후기"로 연결되어 있음. 그러나 리뷰 상세 페이지(`/reviews/:id`) 라우트가 정의되어 있지 않아 리뷰 카드 클릭 시 목적지가 `/reviews`로만 표기되고 상세 화면이 없음 | `/reviews/:id` 라우트 추가 또는 리뷰 카드 클릭 동작(모달/앵커 방식)을 Section 3에 명시 |
| 3 | Admin 대응 화면 누락 | Section 4 `/admin/orders` → Section 3 | Section 4 주문 상세에서 Admin이 "운송장 번호 입력 및 수정(자택배송)" 액션을 수행할 수 있으며, 이 운송장 번호가 고객 화면에 노출되어야 함. Section 3 `/mypage/orders/:id`의 "배송 정보"에 "배송 추적 링크(자택 배송 시)"만 명시되어 있고, 운송장 번호 표시 및 택배사 조회 연동 화면 명세가 누락됨 | `/mypage/orders/:id` 배송 정보 섹션에 운송장 번호 표시 필드 및 택배사별 배송 조회 링크 생성 규칙을 Section 3에 명시 |
| 4 | Admin 대응 화면 누락 | Section 4 `/admin/shops` → Section 3 | Section 4 장착점 상세에서 "지도 위치 수정(카카오맵에서 핀 이동)" 액션이 있어 관리자가 장착점 위치를 변경할 수 있음. 그러나 파트너의 `/partner/shop`(매장 정보 관리)에는 주소 변경 폼은 있으나 지도 핀 위치를 직접 수정하는 UI가 정의되어 있지 않아, 관리자 변경 결과(좌표 업데이트)가 파트너 화면에 어떻게 반영되는지 대응이 누락됨 | `/partner/shop`에 카카오맵 미니 지도 표시 및 "위치가 맞지 않으면 관리자에게 문의" 안내 추가, 또는 파트너의 좌표 수정 허용 여부를 명시 |
| 5 | 권한 불일치 | Section 7 `Reservation | Cancel` (Partner ✅own) → Section 3 `/partner/reservations/:id` | Section 7에서 Partner는 본인 매장 예약에 대해 Cancel(취소)이 ✅own으로 허용됨. 그러나 Section 3 `/partner/reservations/:id` 상태 변경 버튼에는 "예약 거절(예약확인대기 시만)" 버튼만 존재하고, 이미 확정된 예약에 대한 취소 버튼이 정의되어 있지 않음. 거절(Reject)과 취소(Cancel)는 별개의 동작이므로 권한 명세와 화면이 불일치 | `/partner/reservations/:id`에 예약 취소(Cancel) 버튼을 별도 추가하고, 허용 상태(CONFIRMED 등)를 명시. 또는 Section 7의 Partner Cancel 권한 범위를 "예약확인대기(PENDING) 상태의 거절만 해당"으로 재정의 |
| 6 | 권한 불일치 | Section 7 `User | Read/Update/Delete (own)` (Partner ✅own) → Section 3 Partner 라우트 | Section 7에서 Partner는 본인 프로필 조회(`User | Read own profile` ✅own), 수정(`User | Update own profile` ✅own), 비밀번호 재설정(`User | Reset password 본인` ✅own), 탈퇴(`User | Delete own` ✅own) 권한이 있음. 그러나 Section 3 파트너 라우트(`/partner`, `/partner/reservations`, `/partner/settlements`, `/partner/shop`)에 파트너 자신의 회원정보 조회/수정/비밀번호 변경/탈퇴 라우트가 전혀 정의되어 있지 않음 | `/partner/profile` 또는 `/partner/account` 라우트를 Section 3에 추가하여 파트너의 회원정보 수정, 비밀번호 변경, 계정 탈퇴 기능을 명시 |
| 7 | 기능 커버리지 누락 | feature-map.md "장착점 파트너" → Section 3 | feature-map "장착점 파트너" 항목에 "대시보드 — 예약현황/정산 요약"이 있고, Section 3 `/partner` 대시보드에 이 내용이 포함되어 있음. 그러나 feature-map에는 파트너의 "장착 완료 처리" 기능이 "장착 예약 관리" 화면으로 명시되어 있는데, Section 3의 `/partner/reservations`에서 상태 버튼이 "예약 확정 → 장착 시작 → 장착 완료" 3단계로 정의되어 있으나 Module 9(장착 예약 모듈) Flow에는 "PENDING → CONFIRMED → COMPLETED"만 있어 "장착 시작(중간 단계)" 상태가 Section 2 모듈 및 Section 7 ReservationStatus Enum과 불일치 | Section 3의 "장착 시작" 버튼 및 "장착중" 상태를 Section 7 ReservationStatus Enum(`PENDING/CONFIRMED/COMPLETED/CANCELLED`)에 추가하거나, Section 3에서 "장착 시작" 단계를 제거하고 CONFIRMED → COMPLETED 2단계로 통일 |
