## 5. Tech Stack & Architecture

### Architecture

React(반응형 웹) + NestJS REST API + PostgreSQL 구조의 모노레포 기반 3-tier 웹 애플리케이션으로, 고객·파트너·관리자 세 역할을 단일 백엔드가 처리하며 외부 서비스(차량조회 API, 카카오맵, PG, SMS/알림톡)는 NestJS 서비스 레이어에서 추상화하여 연동한다.

```
tirebank/
├── backend/          ← NestJS REST API (src/modules/*)
│   ├── src/
│   │   ├── modules/
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   ├── vehicles/
│   │   │   ├── products/
│   │   │   ├── cart/
│   │   │   ├── orders/
│   │   │   ├── payments/
│   │   │   ├── shops/
│   │   │   ├── reservations/
│   │   │   ├── settlements/
│   │   │   ├── reviews/
│   │   │   ├── contents/       ← event, notice, faq, banner
│   │   │   ├── inquiries/
│   │   │   └── notifications/
│   │   ├── common/             ← guards, interceptors, decorators, filters
│   │   ├── config/
│   │   └── main.ts
│   └── test/
├── frontend/         ← React 고객 웹앱
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   ├── store/              ← Zustand global state
│   │   ├── hooks/
│   │   ├── api/                ← Axios + React Query
│   │   └── styles/
├── admin/            ← React 관리자 대시보드 (별도 빌드)
│   └── src/
│       ├── pages/
│       ├── components/
│       └── store/
└── shared/           ← 공통 타입, DTO, 상수
```

### Technologies

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Backend | NestJS | 10.x | REST API 서버, 모듈 기반 아키텍처 |
| Language | TypeScript | 5.x | 정적 타입 (Backend + Frontend 공통) |
| ORM | TypeORM | 0.3.x | PostgreSQL 엔티티 매핑, 마이그레이션 관리 |
| Database | PostgreSQL | 16.x | 주 데이터 스토어 (관계형, JSONB 활용) |
| Frontend | React | 18.x | 고객·파트너 반응형 웹 UI |
| Routing | React Router | 6.x | 클라이언트 사이드 라우팅 |
| State | Zustand | 4.x | 전역 상태 관리 (장바구니, 사용자 세션) |
| Server State | React Query (TanStack Query) | 5.x | API 캐싱, 서버 상태 동기화 |
| CSS | Tailwind CSS | 3.x | 유틸리티 퍼스트 스타일링, 반응형 레이아웃 |
| Build | Vite | 5.x | 프론트엔드 번들러 |
| Auth | JWT (passport-jwt) | — | Access Token + Refresh Token 인증 |
| File Storage | AWS S3 (또는 호환 스토리지) `[💡 권장적용]` | — | 상품 이미지, 리뷰 이미지 업로드 |
| Cache | Redis `[💡 권장적용]` | 7.x | 세션 토큰 블랙리스트, 타이어 검색 결과 캐싱 |
| API Validation | class-validator / class-transformer | — | NestJS DTO 입력 검증 |
| Process Manager | PM2 `[💡 권장적용]` | — | 운영 환경 프로세스 관리 |

### Third-Party Integrations

| Service | Purpose |
|---------|---------|
| 카카오맵 API | 장착점 위치 지도 표시, 주소 검색, 반경 기반 장착점 탐색 |
| 차량조회 API (외부) | 차량번호 입력 → 차량 정보(모델명, 연식, 타이어 사이즈) 자동 조회 |
| PG (결제 게이트웨이) | 온라인 결제 처리 — 카드/계좌이체/간편결제 `[💡 권장적용: 토스페이먼츠]` |
| SMS / 알림톡 API | 주문 확인, 배송 상태, 장착 예약 알림 발송 `[💡 권장적용: 카카오 비즈메시지 + NHN Cloud SMS]` |
| AWS S3 (또는 호환) `[💡 권장적용]` | 상품·리뷰 이미지 스토리지 |

### Key Decisions

| Decision | Rationale |
|----------|-----------|
| NestJS 모듈 기반 아키텍처 채택 | 도메인(인증/상품/주문/장착점/정산 등)이 명확히 분리되어 있어 NestJS Module 단위로 격리하면 유지보수성과 테스트 커버리지 확보가 용이하다. 향후 MSA 전환 시 모듈 단위 분리도 가능하다. |
| 단일 백엔드에서 3개 역할(Customer·Partner·Admin) 처리 | MVP 단계에서 운영 복잡도를 낮추기 위해 Guard + Role Decorator로 역할 분기 처리. 트래픽 급증 시 Admin 전용 서버 분리로 확장 가능하다. |
| PostgreSQL + TypeORM 선택 (NoSQL 미사용) | 주문·결제·정산 등 트랜잭션 무결성이 필수인 도메인이 핵심이므로 RDBMS가 적합. 타이어 스펙(JSONB)은 PostgreSQL JSONB 컬럼으로 유연하게 저장. |
| React Query + Zustand 조합 `[💡 권장적용]` | 서버 상태(API 데이터)는 React Query, 클라이언트 전역 상태(장바구니·사용자)는 Zustand로 역할을 분리하여 전역 Redux 복잡도를 줄인다. |
| Soft Delete 전략 전면 적용 | 주문·결제·리뷰 등 감사 추적(Audit) 필요 엔티티는 물리 삭제 대신 deleted_at 타임스탬프로 소프트 딜리트 처리하여 데이터 복구 및 감사 로그 일관성을 보장한다. |
| 알림 발송 결과를 NotificationLog DB 엔티티로 관리 | SMS·알림톡 등 외부 채널 발송 결과(성공/실패/재시도)를 DB에 기록하여 발송 이력 추적, 재발송 처리, 관리자 모니터링을 지원한다. AuditLog는 관리자 행위 감사 전용이므로 역할이 상이하며 별도 엔티티로 분리한다. MVP에서는 단순 INSERT 기반으로 구현하고, 대량 발송 규모 증가 시 별도 큐(Queue) 아키텍처로 전환을 검토한다. |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL 연결 문자열 (host, port, db, user, password 포함) |
| `JWT_ACCESS_SECRET` | Access Token 서명 비밀키 |
| `JWT_REFRESH_SECRET` | Refresh Token 서명 비밀키 |
| `JWT_ACCESS_EXPIRES_IN` | Access Token 만료 시간 (예: `15m`) |
| `JWT_REFRESH_EXPIRES_IN` | Refresh Token 만료 시간 (예: `7d`) |
| `REDIS_URL` | Redis 연결 URL (토큰 블랙리스트·캐싱) |
| `KAKAO_MAP_API_KEY` | 카카오맵 JavaScript API 키 |
| `VEHICLE_API_BASE_URL` | 차량조회 외부 API 베이스 URL |
| `VEHICLE_API_KEY` | 차량조회 외부 API 인증 키 |
| `PG_CLIENT_ID` | PG사 클라이언트 ID (예: 토스페이먼츠 ClientKey) |
| `PG_SECRET_KEY` | PG사 시크릿 키 (서버 사이드 전용) |
| `PG_WEBHOOK_SECRET` | PG 웹훅 서명 검증 시크릿 |
| `SMS_API_KEY` | SMS/알림톡 API 키 |
| `SMS_API_SECRET` | SMS/알림톡 API 시크릿 |
| `SMS_SENDER_NUMBER` | 발신자 전화번호 |
| `KAKAO_BIZTALK_SENDER_KEY` | 카카오 알림톡 발신 프로필 키 |
| `S3_BUCKET_NAME` | 파일 업로드 S3 버킷명 |
| `S3_REGION` | S3 리전 |
| `AWS_ACCESS_KEY_ID` | AWS 액세스 키 |
| `AWS_SECRET_ACCESS_KEY` | AWS 시크릿 키 |
| `FRONTEND_URL` | 프론트엔드 베이스 URL (CORS 허용, 이메일 링크 생성용) |
| `ADMIN_URL` | 관리자 대시보드 베이스 URL |
| `NODE_ENV` | 실행 환경 (`development` / `production`) |
| `PORT` | 백엔드 서버 포트 (기본 `3000`) |

---

## 6. Data Model

### Entity List

| Entity | Key Fields | Description |
|--------|-----------|-------------|
| **User** | id(PK), email(UNIQUE), password_hash, name, phone, role(ENUM), status(ENUM), deleted_at, created_at, updated_at | 플랫폼 사용자 계정 (고객·파트너·관리자 통합) |
| **Vehicle** | id(PK), user_id(FK→User), license_plate, manufacturer, model, year, trim, tire_size_id(FK→TireSize), is_primary, deleted_at, created_at | 고객이 등록한 차량 정보 (차량번호 기반 외부 API 조회 후 저장) |
| **TireSize** | id(PK), width, aspect_ratio, rim_diameter, size_label(예: 225/45R18), created_at | 타이어 규격 마스터 (차량-상품 연결 기준) |
| **Brand** | id(PK), name, logo_url, country, description, sort_order, is_active, created_at | 타이어 제조사 브랜드 (한국타이어, 금호 등) |
| **Category** | id(PK), name, parent_id(FK→Category self-ref), depth, sort_order, is_active, created_at | 상품 카테고리 (승용차용/SUV용 등, 계층 구조) |
| **Product** | id(PK), brand_id(FK→Brand), category_id(FK→Category), tire_size_id(FK→TireSize), name, sku(UNIQUE), price, stock_quantity, production_year, production_week, performance_grade(JSONB), description, thumbnail_url, status(ENUM), deleted_at, created_at, updated_at | 타이어 상품 (생산년도/주차, 성능등급 포함) |
| **ProductImage** | id(PK), product_id(FK→Product), url, alt_text, sort_order, created_at | 상품 이미지 (다중 이미지 지원) |
| **Cart** | id(PK), user_id(FK→User UNIQUE), created_at, updated_at | 장바구니 (사용자당 1개) |
| **CartItem** | id(PK), cart_id(FK→Cart), product_id(FK→Product), quantity, unit_price, created_at, updated_at | 장바구니 상품 항목 |
| **Order** | id(PK), user_id(FK→User), order_number(UNIQUE), total_amount, delivery_type(ENUM), delivery_address(JSONB), shop_id(FK→Shop nullable), status(ENUM), memo, deleted_at, created_at, updated_at | 주문 (자택 배송 또는 장착점 배송 선택) |
| **OrderItem** | id(PK), order_id(FK→Order), product_id(FK→Product), quantity, unit_price, subtotal, created_at | 주문 상품 항목 (주문 시점 가격 스냅샷) |
| **Payment** | id(PK), order_id(FK→Order UNIQUE), pg_transaction_id(UNIQUE), amount, method(ENUM), status(ENUM), paid_at, refunded_at, refund_amount, raw_response(JSONB), created_at, updated_at | PG 결제 정보 (원본 응답 보관) |
| **Shop** | id(PK), partner_id(FK→User), name, business_number, address, latitude, longitude, phone, operating_hours(JSONB), description, thumbnail_url, status(ENUM), approved_at, deleted_at, created_at, updated_at | 장착점(제휴 정비소) 정보 |
| **Reservation** | id(PK), order_id(FK→Order), shop_id(FK→Shop), user_id(FK→User), reserved_date, reserved_time_slot, status(ENUM), completed_at, memo, created_at, updated_at | 장착 예약 (장착점 배송 선택 시 생성) |
| **Settlement** | id(PK), shop_id(FK→Shop), period_start, period_end, total_amount, fee_rate, fee_amount, net_amount, status(ENUM), settled_at, memo, created_at, updated_at | 장착점 정산 내역 (주기별) |
| **SettlementItem** | id(PK), settlement_id(FK→Settlement), reservation_id(FK→Reservation), order_item_id(FK→OrderItem), amount, created_at | 정산 세부 항목 (예약·주문 항목별) |
| **Review** | id(PK), user_id(FK→User), product_id(FK→Product), order_item_id(FK→OrderItem UNIQUE), rating(1-5), content, images(JSONB), is_blinded, blinded_reason, deleted_at, created_at, updated_at | 구매후기 (주문 항목 기반, 1건만 허용) |
| **Event** | id(PK), title, content, thumbnail_url, start_at, end_at, is_active, sort_order, deleted_at, created_at, updated_at | 이벤트 게시글 |
| **Notice** | id(PK), title, content, is_pinned, view_count, deleted_at, created_at, updated_at | 공지사항 |
| **FAQ** | id(PK), category, question, answer, sort_order, is_active, created_at, updated_at | 자주 묻는 질문 |
| **Banner** | id(PK), title, image_url, link_url, position(ENUM), sort_order, start_at, end_at, is_active, created_at, updated_at | 메인/중간 프로모션 배너 |
| **Inquiry** | id(PK), user_id(FK→User), order_id(FK→Order nullable), category, title, content, status(ENUM), answered_at, deleted_at, created_at, updated_at | 1:1 고객 문의 |
| **InquiryAnswer** | id(PK), inquiry_id(FK→Inquiry UNIQUE), admin_id(FK→User), content, created_at, updated_at | 문의 답변 (관리자 작성) |
| **NotificationLog** | id(PK), user_id(FK→User nullable), type(ENUM), channel(ENUM), recipient(전화번호 또는 카카오 ID), message, status(ENUM), sent_at, error_message, created_at | 알림 발송 결과 로그 — SMS·알림톡 외부 채널 발송 이력 추적. AuditLog(관리자 행위 감사 전용)와 역할이 다르므로 별도 엔티티로 분리한다. `[💡 권장적용]` |
| **AuditLog** | id(PK), actor_id(FK→User nullable), actor_role, action, target_entity, target_id, before_data(JSONB), after_data(JSONB), ip_address, user_agent, created_at | 관리자·파트너 행위 감사 로그 (물리 삭제 없음) |

### Entity Relationships

```
User 1:N Vehicle                    ← 한 고객은 여러 차량을 등록할 수 있다
User 1:1 Cart                       ← 고객 1명당 장바구니 1개
Cart 1:N CartItem                   ← 장바구니는 여러 상품 항목을 포함
CartItem N:1 Product                ← 각 장바구니 항목은 하나의 상품을 참조

User 1:N Order                      ← 고객은 여러 주문을 생성할 수 있다
Order 1:N OrderItem                 ← 주문은 여러 상품 항목을 포함
OrderItem N:1 Product               ← 각 주문 항목은 하나의 상품을 참조
Order 1:1 Payment                   ← 주문에는 결제 1건
Order N:1 Shop (nullable)           ← 장착점 배송 주문은 하나의 장착점에 연결
Order 1:1 Reservation (nullable)    ← 장착점 배송 주문은 예약 1건 생성

User 1:N Shop (partner_id)          ← 파트너 사용자는 장착점을 1개 이상 관리 [💡 권장적용: MVP에서는 파트너 1인 1점 제한]
Shop 1:N Reservation                ← 장착점은 여러 예약을 받는다
Shop 1:N Settlement                 ← 장착점은 여러 정산 주기 내역을 가진다
Settlement 1:N SettlementItem       ← 정산 1건은 여러 예약·주문 항목을 포함
SettlementItem N:1 Reservation      ← 정산 항목은 특정 예약을 참조
SettlementItem N:1 OrderItem        ← 정산 항목은 특정 주문 상품을 참조

User 1:N Review                     ← 고객은 여러 상품에 리뷰를 작성할 수 있다
Review N:1 Product                  ← 리뷰는 하나의 상품에 속한다
Review 1:1 OrderItem                ← 리뷰는 특정 주문 항목과 1:1 (중복 작성 방지)

Product N:1 Brand                   ← 상품은 하나의 브랜드에 속한다
Product N:1 Category                ← 상품은 하나의 카테고리에 속한다
Product N:1 TireSize                ← 상품은 하나의 타이어 규격에 속한다
Vehicle N:1 TireSize                ← 차량은 권장 타이어 규격을 가진다
Product 1:N ProductImage            ← 상품은 여러 이미지를 가진다

Category self-ref (parent_id)       ← 카테고리는 부모 카테고리를 가질 수 있다 (최대 2depth [💡 권장적용])

User 1:N Inquiry                    ← 고객은 여러 문의를 작성할 수 있다
Inquiry N:1 Order (nullable)        ← 문의는 특정 주문과 연결될 수 있다
Inquiry 1:1 InquiryAnswer           ← 문의에는 답변 1건

User 1:N NotificationLog (nullable) ← 사용자에게 발송된 알림 로그 (비회원 발송 시 user_id NULL 허용)

Product N:N TireSize via ProductTireSize (join table) [💡 권장적용: 호환 사이즈 복수 지원 시]
  ← 현재 Product.tire_size_id 1:1이 기본, 호환 사이즈 확장 필요 시 중간 테이블 ProductTireSize(product_id, tire_size_id) 추가
```

### Status Enums

| Enum Name | Values | Used By |
|-----------|--------|---------|
| `UserRole` | `CUSTOMER(0)`, `PARTNER(1)`, `ADMIN(99)` | User |
| `UserStatus` | `ACTIVE(0)`, `SUSPENDED(1)`, `WITHDRAWN(2)` | User |
| `ProductStatus` | `DRAFT(0)`, `ACTIVE(1)`, `OUT_OF_STOCK(2)`, `DISCONTINUED(3)` | Product |
| `DeliveryType` | `HOME(0)`, `SHOP(1)` | Order |
| `OrderStatus` | `PENDING_PAYMENT(0)`, `PAID(1)`, `PREPARING(2)`, `SHIPPED(3)`, `DELIVERED(4)`, `RESERVATION_PENDING(5)`, `INSTALLATION_COMPLETED(6)`, `CANCELLED(7)`, `REFUNDED(8)` | Order |
| `PaymentMethod` | `CARD(0)`, `BANK_TRANSFER(1)`, `VIRTUAL_ACCOUNT(2)`, `EASY_PAY(3)` | Payment |
| `PaymentStatus` | `PENDING(0)`, `COMPLETED(1)`, `FAILED(2)`, `CANCELLED(3)`, `REFUNDED(4)`, `PARTIAL_REFUNDED(5)` | Payment |
| `ShopStatus` | `PENDING_APPROVAL(0)`, `ACTIVE(1)`, `SUSPENDED(2)`, `CLOSED(3)` | Shop |
| `ReservationStatus` | `PENDING(0)`, `CONFIRMED(1)`, `COMPLETED(2)`, `CANCELLED(3)`, `NO_SHOW(4)` | Reservation |
| `SettlementStatus` | `PENDING(0)`, `PROCESSING(1)`, `COMPLETED(2)`, `DISPUTED(3)` | Settlement |
| `InquiryStatus` | `PENDING(0)`, `ANSWERED(1)`, `CLOSED(2)` | Inquiry |
| `BannerPosition` | `HERO(0)`, `MID_PROMO(1)`, `FOOTER(2)` | Banner |
| `NotificationType` | `ORDER_CONFIRMED(0)`, `ORDER_SHIPPED(1)`, `ORDER_DELIVERED(2)`, `RESERVATION_CONFIRMED(3)`, `RESERVATION_REMINDER(4)`, `INSTALLATION_COMPLETED(5)`, `PAYMENT_REFUNDED(6)` | NotificationLog |
| `NotificationChannel` | `SMS(0)`, `KAKAO_ALIMTALK(1)` | NotificationLog |
| `NotificationStatus` | `PENDING(0)`, `SENT(1)`, `FAILED(2)`, `RETRYING(3)` | NotificationLog |

### Index Hints

| Entity | Column(s) | Type | Reason |
|--------|-----------|------|--------|
| User | email | UNIQUE | 로그인 시 이메일 조회 |
| User | phone | UNIQUE | 전화번호 중복 가입 방지 |
| User | role, status | COMPOSITE (BTREE) | 관리자 회원 목록 필터 (role + status 복합) |
| Vehicle | user_id | BTREE | 고객 차량 목록 조회 |
| Vehicle | license_plate | UNIQUE | 차량번호 중복 등록 방지 |
| Product | tire_size_id, status | COMPOSITE (BTREE) | 타이어 사이즈 기반 상품 검색 |
| Product | brand_id, status | COMPOSITE (BTREE) | 브랜드 필터 조회 |
| Product | price | BTREE | 가격 정렬 |
| Product | status, created_at | COMPOSITE (BTREE) | 최신 상품 목록 기본 정렬 |
| CartItem | cart_id | BTREE | 장바구니 항목 조회 |
| Order | user_id, created_at | COMPOSITE (BTREE) | 마이페이지 주문내역 (최신순) |
| Order | status | BTREE | 관리자 주문 상태 필터 |
| Order | order_number | UNIQUE | 주문번호 단건 조회 |
| Payment | order_id | UNIQUE | 주문-결제 1:1 조회 |
| Payment | pg_transaction_id | UNIQUE | PG 웹훅 결제 대사 |
| Shop | partner_id | BTREE | 파트너 자신의 장착점 조회 |
| Shop | latitude, longitude | BTREE | 지도 좌표 기반 반경 검색 (PostGIS 확장 고려) |
| Shop | status | BTREE | 관리자 장착점 상태 필터 |
| Reservation | shop_id, reserved_date | COMPOSITE (BTREE) | 장착점 날짜별 예약 목록 |
| Reservation | user_id | BTREE | 고객 예약 내역 조회 |
| Reservation | order_id | BTREE | 주문 기반 예약 조회 |
| Settlement | shop_id, period_start | COMPOSITE (BTREE) | 장착점 정산 기간별 조회 |
| Review | product_id, created_at | COMPOSITE (BTREE) | 상품 리뷰 목록 (최신순) |
| Review | user_id | BTREE | 마이페이지 리뷰 목록 |
| Review | order_item_id | UNIQUE | 주문 항목당 리뷰 중복 작성 방지 |
| Inquiry | user_id, created_at | COMPOSITE (BTREE) | 마이페이지 문의 내역 |
| Inquiry | status | BTREE | 관리자 미답변 문의 필터 |
| NotificationLog | user_id, created_at | COMPOSITE (BTREE) | 사용자별 발송 이력 조회 |
| NotificationLog | status, created_at | COMPOSITE (BTREE) | 실패 건 재발송 배치 처리 |
| AuditLog | actor_id, created_at | COMPOSITE (BTREE) | 행위자별 감사 로그 조회 |
| AuditLog | target_entity, target_id | COMPOSITE (BTREE) | 특정 엔티티 변경 이력 추적 |

### Soft Delete

| Entity | Soft Delete | Retention |
|--------|------------|-----------|
| User | ✅ (deleted_at) | 탈퇴 후 30일 보존, 이후 개인정보 비식별화 처리 `[💡 권장적용]` |
| Vehicle | ✅ (deleted_at) | 연결 주문 보존 목적, User 삭제 시 함께 soft delete |
| Product | ✅ (deleted_at) | 주문 이력 참조 보존 목적, 90일 보존 후 관리자 판단으로 영구 삭제 |
| Order | ✅ (deleted_at) | 결제·정산 데이터 연동, 5년 보존 (전자상거래법 준수) `[💡 권장적용]` |
| Shop | ✅ (deleted_at) | 예약·정산 이력 보존 목적, 계약 종료 후 1년 보존 `[💡 권장적용]` |
| Review | ✅ (deleted_at) | 삭제 후 30일 보존 (블라인드 처리 후 실제 삭제) |
| Inquiry | ✅ (deleted_at) | 고객 문의 이력 2년 보존 `[💡 권장적용]` |
| Event | ✅ (deleted_at) | 이벤트 종료 후 90일 보존 |
| Notice | ✅ (deleted_at) | 삭제 후 30일 보존 |
| CartItem | ❌ | 물리 삭제 (장바구니는 휘발성 데이터) |
| AuditLog | ❌ | 감사 로그는 물리 삭제 없음, 영구 보존 |
| Payment | ❌ | 결제 원본 데이터는 물리 삭제 없음, 영구 보존 (금융 감사) |
| Settlement | ❌ | 정산 데이터는 물리 삭제 없음, 영구 보존 |
| SettlementItem | ❌ | 정산 세부 항목 영구 보존 |
| OrderItem | ❌ | 주문 항목은 Order soft delete 연동, 별도 삭제 없음 |
| NotificationLog | ❌ | 발송 이력은 물리 삭제 없음, 1년 보존 후 아카이빙 처리 `[💡 권장적용]` |
