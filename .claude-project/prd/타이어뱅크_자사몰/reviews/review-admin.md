## Cross-Review Feedback (스코프: 관리페이지 1:1 매핑)

> 리뷰어: Section 4 (Admin Dashboard) 작성자
> 검토 대상: Section 3 (User Application) 라우트 목록
> 검토 기준: Section 3에서 생성/수정/삭제 가능한 데이터 엔티티가 Section 4 관리페이지에 대응하는지 여부

---

### 엔티티-관리페이지 매핑 현황

| 엔티티 | Section 3 출처 | Section 4 관리페이지 | 매핑 상태 |
|--------|---------------|---------------------|----------|
| User | `/auth/register`, `/mypage` (이름/전화/비밀번호 수정, 탈퇴) | `/admin/users`, `/admin/users/:id` | 매핑됨 |
| Vehicle | `/mypage/vehicles` (차량 CRUD) | 없음 — 독립 관리페이지 미존재 | **누락** |
| Product | `/tire/:id` (장바구니/구매 트리거), `/admin/products` 직접 관리 | `/admin/products`, `/admin/products/:id` | 매핑됨 |
| Order | `/order/checkout` (주문 생성), `/mypage/orders/:id` (취소) | `/admin/orders`, `/admin/orders/:id` | 매핑됨 |
| CartItem | `/cart` (담기/수량변경/삭제) | 없음 — 관리페이지 미존재 | **누락** |
| Review | `/mypage/reviews` (리뷰 작성/수정/삭제) | `/admin/reviews` | 매핑됨 |
| Inquiry | `/cs/inquiry` (문의 작성), `/mypage/inquiries` (조회) | `/admin/inquiries` | 매핑됨 |
| Shop | `/partner/shop` (전화번호/주소/소개/영업시간/이미지 수정) | `/admin/shops`, `/admin/shops/:id` | 매핑됨 |
| Reservation | `/partner/reservations` (상태 변경: 확정/시작/완료/거절) | 없음 — 독립 관리페이지 미존재 | **누락** |
| Settlement | `/partner/settlements` (조회 전용) | `/admin/settlements` | 매핑됨 |
| Event | `/event`, `/event/:id` (조회 전용) | `/admin/events` | 매핑됨 |
| Notice | `/cs/notice`, `/cs/notice/:id` (조회 전용) | `/admin/notices` | 매핑됨 |
| FAQ | `/cs/faq` (조회 전용) | `/admin/faqs` | 매핑됨 |
| Banner | `/` 히어로 슬라이더, 중간 프로모션 (조회 전용) | `/admin/banners` | 매핑됨 |

---

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 누락 | Section 3 `/mypage/vehicles` | **Vehicle 엔티티 관리페이지 없음.** 고객은 차량을 등록/수정/삭제할 수 있으나, 관리자가 특정 고객의 차량을 직접 조회·수정·삭제하는 전용 페이지(`/admin/vehicles` 또는 회원 상세 내 차량 탭)가 Section 4에 없음. 현재 `/admin/users/:id` Detail Drawer의 "등록 차량 목록" 은 조회만 가능하고, 수정/삭제 액션이 명시되어 있지 않음 | `/admin/users/:id` Detail Drawer에 차량 수정/삭제 액션을 명시하거나, 별도 `/admin/vehicles` 관리페이지 추가 |
| 2 | 누락 | Section 3 `/partner/reservations`, `/partner/reservations/:id` | **Reservation 엔티티 관리페이지 없음.** 파트너가 예약 상태를 변경(확정/거절/장착시작/장착완료)하며 Reservation 데이터를 직접 변경하지만, 관리자가 전체 예약을 조회·개입·강제 상태변경할 수 있는 `/admin/reservations` 페이지가 Section 4 Page Map에 없음. 주문 상세(`/admin/orders/:id`)에 장착 예약 정보가 표시되긴 하나, 예약 목록 전체 조회 및 상태 강제 변경 기능이 별도로 명세되어 있지 않음 | `/admin/reservations` 관리페이지 추가 또는 `/admin/orders/:id`에 예약 상태 강제변경 액션 범위를 명확히 명세 |
| 3 | 누락 | Section 3 `/cart` | **CartItem 엔티티 관리 범위 미정의.** 장바구니 항목은 고객이 담기/수량변경/삭제를 통해 생성·수정·삭제되는 데이터임. 운영 이슈(예: 특정 상품 품절로 인한 장바구니 일괄 제거, 또는 CS 대응 시 고객 장바구니 확인) 발생 시 관리자가 접근할 수단이 Section 4에 없음 | 운영 필요성이 낮다고 판단되면 스코프 외로 명시 처리; 필요하다면 `/admin/users/:id` Detail Drawer에 해당 고객의 장바구니 조회 탭 추가 |
| 4 | 불일치 | Section 3 `/partner/shop` vs Section 4 `/admin/shops/:id` | **파트너가 수정 가능한 필드와 관리자 수정 가능 필드 범위 불일치.** Section 3 `/partner/shop`에서 파트너는 전화번호, 주소, 소개, 영업시간, 이미지를 수정할 수 있고 "매장명은 읽기 전용 — 관리자만 변경 가능"이라고 명시되어 있음. 반면 Section 4 `/admin/shops/:id` 수정 가능 액션에는 "영업시간, 전화번호, 소개, 주소" 만 나열되어 있고 매장명 수정과 이미지 수정이 포함되지 않음 | `/admin/shops/:id` Detail Drawer의 수정 가능 필드에 매장명 및 매장 이미지를 명시적으로 추가 |
| 5 | 불일치 | Section 3 `/mypage/reviews` vs Section 4 `/admin/reviews` | **리뷰 작성 권한 불일치 가능성.** Section 3에서 고객은 배송완료/장착완료 상태인 주문에 대해서만 리뷰를 작성할 수 있음(주문 연결 구조). 그러나 Section 4 `/admin/reviews` Creation Modal은 "없음 (고객이 작성하는 리뷰만 관리)"으로 명세되어 있어 일치함. 단, 관리자가 특정 리뷰의 연결 주문 정보를 조회하거나 검증할 수 있는 필드가 Detail Drawer에 없음 | `/admin/reviews` Detail Drawer 표시 필드에 "연결 주문번호" 항목 추가 검토 |
