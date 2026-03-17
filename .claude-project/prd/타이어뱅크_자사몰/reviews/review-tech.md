## Cross-Review Feedback (스코프: 엔티티 커버리지)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 엔티티 누락 | Section 3 Module 14 → Section 6 Entity List | 알림 모듈(Module 14)에서 "발송 결과 로그 기록"을 명시하고 있으나, Section 6 Entity List에 `NotificationLog` 또는 이에 상응하는 엔티티가 없음. AuditLog는 관리자 행위 감사 목적으로 별도 존재하며, 발송 결과 추적 엔티티와 역할이 다름. | Section 6 Entity List에 `NotificationLog(id, user_id(FK→User), type(ENUM), channel(SMS/KAKAO), message, status, sent_at, created_at)` 엔티티 추가를 검토할 것. 단, MVP에서 로그를 파일/AuditLog로 처리하기로 결정했다면 Section 3 Module 14 설명에서 "DB 엔티티 없음, 파일 로깅 처리" 명시 필요. |
| 2 | 엔티티 미관리 (Admin UI 누락) | Section 3 Module 3 + Section 4 Admin Page Map | Section 3 Module 3(상품 관리 모듈)에서 Brand·Category·TireSize를 별도 관리 가능한 엔티티로 정의하고(상품 등록 폼에서 "등록된 브랜드 중 선택", "등록된 카테고리 중 선택" 명시), Section 6 Entity List에도 `Brand`, `Category`, `TireSize` 엔티티가 존재함. 그러나 Section 4 Admin Dashboard의 Page Map(`/admin/brands`, `/admin/categories`, `/admin/tire-sizes` 등)에 해당 관리 라우트가 없어 관리자가 이 마스터 데이터를 CRUD할 수 있는 화면이 정의되지 않음. | Section 4에 Brand·Category·TireSize 마스터 관리 라우트(`/admin/brands`, `/admin/categories`, `/admin/tire-sizes`)를 추가하거나, 해당 관리를 상품 관리 모달 내 인라인 생성으로 처리하는 방식을 명시할 것. |
| 3 | FK 검증 이상 없음 | Section 6 Entity Relationships | 모든 FK(`user_id→User`, `product_id→Product`, `order_id→Order`, `shop_id→Shop`, `reservation_id→Reservation`, `settlement_id→Settlement`, `order_item_id→OrderItem`, `inquiry_id→Inquiry`, `admin_id→User`, `actor_id→User`, `brand_id→Brand`, `category_id→Category`, `tire_size_id→TireSize`, `cart_id→Cart`, `parent_id→Category self-ref` 등)가 Section 6 Entity List에 실제 존재하는 엔티티를 정확히 참조하고 있음. | 이상 없음. |
| 4 | 3rd Party 매핑 이상 없음 | Section 2 3rd Party API List → Section 5 Third-Party Integrations | Section 2의 5개 서비스(차량조회 API, 카카오맵 API, PG 결제 게이트웨이, SMS API, 카카오 비즈메시지)가 Section 5 Third-Party Integrations 테이블에 모두 커버됨. SMS API와 카카오 비즈메시지는 Section 5에서 "SMS / 알림톡 API" 단일 행으로 통합 표기되어 있으나, Section 2에서도 두 서비스가 알림 모듈 단일 통합점으로 기술되어 있으므로 불일치 없음. | 이상 없음. |

### 요약

- **차단(Blocker):** 없음
- **수정 필요(Required Fix):** #1 (알림 발송 로그 엔티티 정책 미결), #2 (Brand·Category·TireSize Admin UI 정의 누락)
- **확인 완료(OK):** #3 FK 일관성, #4 3rd Party 커버리지
