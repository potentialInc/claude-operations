# Screen Inventory — 타이어뱅크 자사몰

## 유저앱 라우트 (총 24개)

| # | 라우트 | 페이지명 | 유저타입 | 접근 그룹 | 비고 |
|---|--------|---------|---------|----------|------|
| 1 | `/` | 메인 (Home) | All | Public | 히어로+검색+BEST+후기 |
| 2 | `/tire` | 타이어 목록 | All | Public | 검색/필터/정렬 |
| 3 | `/tire/:id` | 타이어 상세 | All | Public | 상세정보+리뷰+구매 |
| 4 | `/mount` | 무료 장착점 찾기 | All | Public | 지도+지역검색 |
| 5 | `/mount/:id` | 장착점 상세 | All | Public | 매장정보+예약 |
| 6 | `/event` | 이벤트 목록 | All | Public | |
| 7 | `/event/:id` | 이벤트 상세 | All | Public | |
| 8 | `/cs/notice` | 공지사항 | All | Public | |
| 9 | `/cs/notice/:id` | 공지사항 상세 | All | Public | |
| 10 | `/cs/faq` | FAQ | All | Public | |
| 11 | `/cs/inquiry` | 1:1 문의 작성 | Customer | Protected | |
| 12 | `/reviews` | 구매후기 목록 | All | Public | |
| 13 | `/auth/login` | 로그인 | All | Auth | |
| 14 | `/auth/register` | 회원가입 | All | Auth | |
| 15 | `/auth/forgot-password` | 비밀번호 찾기 | All | Auth | |
| 16 | `/cart` | 장바구니 | Customer | Protected | |
| 17 | `/order/checkout` | 주문/결제 | Customer | Protected | 배송방식 선택 |
| 18 | `/order/complete` | 주문 완료 | Customer | Protected | |
| 19 | `/mypage` | 마이페이지 | Customer | Protected | |
| 20 | `/mypage/orders` | 주문내역 | Customer | Protected | 배송/장착 상태 |
| 21 | `/mypage/orders/:id` | 주문 상세 | Customer | Protected | |
| 22 | `/mypage/vehicles` | 차량 관리 | Customer | Protected | |
| 23 | `/mypage/reviews` | 내 리뷰 | Customer | Protected | |
| 24 | `/mypage/inquiries` | 문의내역 | Customer | Protected | |

## 장착점 파트너 라우트 (총 5개)

| # | 라우트 | 페이지명 | 유저타입 | 접근 그룹 | 비고 |
|---|--------|---------|---------|----------|------|
| 1 | `/partner` | 파트너 대시보드 | Partner | Protected | |
| 2 | `/partner/reservations` | 장착 예약 관리 | Partner | Protected | |
| 3 | `/partner/reservations/:id` | 장착 예약 상세 | Partner | Protected | |
| 4 | `/partner/settlements` | 정산 관리 | Partner | Protected | |
| 5 | `/partner/shop` | 매장 정보 관리 | Partner | Protected | |

## 어드민 라우트 (총 14개)

| # | 라우트 | 페이지명 | 관리 대상 | 비고 |
|---|--------|---------|----------|------|
| 1 | `/admin` | 대시보드 | 전체 통계 | |
| 2 | `/admin/users` | 회원 관리 | User | CRUD |
| 3 | `/admin/users/:id` | 회원 상세 | User | |
| 4 | `/admin/shops` | 장착점 관리 | Shop | CRUD + 승인 |
| 5 | `/admin/shops/:id` | 장착점 상세 | Shop | |
| 6 | `/admin/products` | 상품 관리 | Product | CRUD |
| 7 | `/admin/products/:id` | 상품 상세 | Product | |
| 8 | `/admin/orders` | 주문 관리 | Order | 상태변경/취소/환불 |
| 9 | `/admin/orders/:id` | 주문 상세 | Order | |
| 10 | `/admin/settlements` | 정산 관리 | Settlement | |
| 11 | `/admin/events` | 이벤트 관리 | Event | CRUD |
| 12 | `/admin/notices` | 공지사항 관리 | Notice | CRUD |
| 13 | `/admin/faqs` | FAQ 관리 | FAQ | CRUD |
| 14 | `/admin/banners` | 배너 관리 | Banner | CRUD |
| 15 | `/admin/reviews` | 리뷰 관리 | Review | 조회/삭제/블라인드 |
| 16 | `/admin/inquiries` | 문의 관리 | Inquiry | 답변/상태변경 |
