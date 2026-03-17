## 7. Permission Matrix

---

### 7.1 Role Definitions

| Role | DB Value | Description |
|------|----------|-------------|
| **Guest** | — | 비로그인 사용자. Public 라우트 접근만 가능 |
| **Customer** | `0` | 일반 고객. 타이어 검색·주문·리뷰·문의·마이페이지 이용 |
| **Partner** | `1` | 장착점 파트너. 본인 매장의 예약·정산·매장정보 관리 |
| **Admin** | `99` | 플랫폼 관리자. 전체 리소스에 대한 완전한 접근 권한 |

---

### 7.2 Role Hierarchy

```
Guest < Customer < Admin
Guest < Partner  < Admin
```

- Customer와 Partner는 별도 도메인 역할이며 상호 권한을 상속하지 않는다.
- Admin은 모든 역할의 권한을 포함하며, 소유권 제약 없이 전체 리소스에 접근한다.
- Partner는 Customer 권한을 상속하지 않는다. 주문·장바구니 등 고객 기능은 Partner 역할에서 제공되지 않는다.

---

### 7.3 Action × Role Matrix

> **Legend**
> - ✅ = 허용 (모든 리소스)
> - ✅own = 허용 (소유 리소스만)
> - ❌ = 거부
> - [💡 권장적용] = 클라이언트 미명시, best practice 기반 권장안 적용

#### Vehicle (차량) — Customer의 마이페이지 차량관리

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Vehicle | Create | ❌ | ✅ | ❌ | ✅ |
| Vehicle | Read (own) | ❌ | ✅own | ❌ | ✅ |
| Vehicle | Read (all) | ❌ | ❌ | ❌ | ✅ |
| Vehicle | Update | ❌ | ✅own | ❌ | ✅ |
| Vehicle | Delete | ❌ | ✅own | ❌ | ✅ |

> - 소유권: `vehicle.user_id === currentUser.id`
> - Guest는 차량번호 기반 타이어 검색(외부 API 조회)은 가능하나, 차량 저장·관리는 불가.
> - Partner는 고객 차량을 직접 관리하지 않는다. [💡 권장적용]

---

#### Product (상품 — 타이어)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Product | Create | ❌ | ❌ | ❌ | ✅ |
| Product | Read (list) | ✅ | ✅ | ✅ | ✅ |
| Product | Read (detail) | ✅ | ✅ | ✅ | ✅ |
| Product | Update | ❌ | ❌ | ❌ | ✅ |
| Product | Delete | ❌ | ❌ | ❌ | ✅ |
| Product | Toggle visibility | ❌ | ❌ | ❌ | ✅ |
| Product | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 상품 조회는 공개 리소스. Guest·Customer·Partner 모두 상품 목록·상세 열람 가능.
> - 재고 수량 및 원가 필드는 Admin만 열람 가능. [💡 권장적용]

---

#### Cart (장바구니)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Cart | Create | ❌ | ✅ | ❌ | ✅ |
| Cart | Read (own) | ❌ | ✅own | ❌ | ✅ |
| Cart | Read (all) | ❌ | ❌ | ❌ | ✅ |
| Cart | Delete | ❌ | ✅own | ❌ | ✅ |

> - 소유권: `cart.user_id === currentUser.id`
> - Guest 비로그인 장바구니는 MVP 범위 외 (로그인 후 이용). [💡 권장적용]
> - Partner는 장바구니·주문 플로우 비접근.
> - Cart 레벨 Update 행은 의도적으로 제외. 수량 변경은 CartItem `Update (수량변경)` 행(Customer: ✅own)으로 처리한다. [💡 권장적용]

---

#### CartItem (장바구니 항목)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| CartItem | Create | ❌ | ✅own | ❌ | ✅ |
| CartItem | Read (own) | ❌ | ✅own | ❌ | ✅ |
| CartItem | Read (all) | ❌ | ❌ | ❌ | ✅ |
| CartItem | Update (수량변경) | ❌ | ✅own | ❌ | ✅ |
| CartItem | Delete | ❌ | ✅own | ❌ | ✅ |

> - 소유권: `cartItem.cart.user_id === currentUser.id`

---

#### Order (주문)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Order | Create | ❌ | ✅ | ❌ | ✅ |
| Order | Read (own) | ❌ | ✅own | ❌ | ✅ |
| Order | Read (by shop) | ❌ | ❌ | ✅own | ✅ |
| Order | Read (all) | ❌ | ❌ | ❌ | ✅ |
| Order | Update (상태변경) | ❌ | ❌ | ❌ | ✅ |
| Order | Update (운송장번호 수정, UpdateTracking) | ❌ | ❌ | ❌ | ✅ |
| Order | Cancel | ❌ | ✅own | ❌ | ✅ |
| Order | Refund 처리 | ❌ | ❌ | ❌ | ✅ |
| Order | Delete | ❌ | ❌ | ❌ | ❌ |
| Order | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 소유권(Customer): `order.user_id === currentUser.id`
> - 소유권(Partner): `order.shop_id === currentPartner.shop_id` — 장착점 배송 주문만 열람 가능
> - 주문 삭제는 Admin도 불가 (데이터 보존 필수, 취소·환불 상태 변경으로 처리)
> - 고객의 주문 취소는 허용 상태(결제완료, 배송준비)에서만 가능. [💡 권장적용]
> - `Update (운송장번호 수정, UpdateTracking)`: Admin 전용. 자택배송 주문에 한하며, 배송 상태(SHIPPING) 진입 시점에 수행. 운송장 번호는 고객 주문 상세(`/mypage/orders/:id`)에 노출된다.

---

#### Review (리뷰)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Review | Create | ❌ | ✅ | ❌ | ❌ |
| Review | Read (list/public) | ✅ | ✅ | ✅ | ✅ |
| Review | Read (detail) | ✅ | ✅ | ✅ | ✅ |
| Review | Read (all, including blinded) | ❌ | ❌ | ❌ | ✅ |
| Review | Update | ❌ | ✅own | ❌ | ❌ |
| Review | Delete (own) | ❌ | ✅own | ❌ | ✅ |
| Review | Blind/Unblind | ❌ | ❌ | ❌ | ✅ |
| Review | Bulk Blind/Unblind | ❌ | ❌ | ❌ | ✅ |
| Review | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 소유권: `review.user_id === currentUser.id`
> - 리뷰 작성 조건: 해당 상품 주문 완료 고객만 작성 가능 (구매 인증). [💡 권장적용]
> - Admin은 리뷰 직접 생성 불가, 블라인드 처리·삭제만 가능.
> - 블라인드 처리된 리뷰는 Guest·Customer·Partner에게 노출되지 않음.
> - `Bulk Blind/Unblind`: Admin 전용. 개별 `Blind/Unblind`와 동일한 역할 검증을 일괄 적용한다. [💡 권장적용]

---

#### Inquiry (1:1 문의)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Inquiry | Create | ❌ | ✅ | ❌ | ❌ |
| Inquiry | Read (own) | ❌ | ✅own | ❌ | ✅ |
| Inquiry | Read (all) | ❌ | ❌ | ❌ | ✅ |
| Inquiry | Update (내용수정) | ❌ | ✅own | ❌ | ❌ |
| Inquiry | Delete | ❌ | ✅own | ❌ | ✅ |
| Inquiry | Answer (답변작성) | ❌ | ❌ | ❌ | ✅ |
| Inquiry | Answer Update | ❌ | ❌ | ❌ | ✅ |
| Inquiry | Status Change | ❌ | ❌ | ❌ | ✅ |
| Inquiry | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 소유권: `inquiry.user_id === currentUser.id`
> - Customer는 답변대기 상태의 문의만 수정·삭제 가능. [💡 권장적용]
> - Admin은 답변완료 상태 문의 삭제 불가 (슈퍼어드민 제외). [💡 권장적용]
> - Guest는 문의 작성 불가 — 로그인 유도.
> - Customer `Update (내용수정)` 및 `Delete` ✅own 권한은 유지. Section 3 `/mypage/inquiries`의 수정/삭제 버튼 인터랙션은 Section 3에서 UI 추가 예정 (답변대기 상태 문의에 한정). [💡 권장적용]

---

#### Shop (장착점)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Shop | Create | ❌ | ❌ | ❌ | ✅ |
| Shop | Read (list/public) | ✅ | ✅ | ✅ | ✅ |
| Shop | Read (detail/public) | ✅ | ✅ | ✅ | ✅ |
| Shop | Read (own shop) | ❌ | ❌ | ✅own | ✅ |
| Shop | Read (all, admin) | ❌ | ❌ | ❌ | ✅ |
| Shop | Update (own shop info) | ❌ | ❌ | ✅own | ✅ |
| Shop | Update (위치/좌표 수정) | ❌ | ❌ | ❌ | ✅ |
| Shop | Update (approval status) | ❌ | ❌ | ❌ | ✅ |
| Shop | Delete | ❌ | ❌ | ❌ | ✅ |
| Shop | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 소유권(Partner): `shop.id === currentPartner.shop_id`
> - Partner는 본인 매장의 영업시간·주소·연락처·소개만 수정 가능 (`Update (own shop info)`). 위도/경도 좌표 직접 수정은 불가.
> - `Update (위치/좌표 수정)`: 위도/경도 좌표(카카오맵 핀 이동) 변경은 Admin 전용. Partner의 `/partner/shop` 화면에는 주소 입력 필드만 제공되며, 좌표 직접 수정 UI는 제공하지 않는다. 파트너는 좌표 오류 시 관리자에게 수정 요청한다. [💡 권장적용]
> - 승인 상태(Pending/Active/Rejected/Suspended) 변경은 Admin 전용.
> - 장착점 목록·상세는 Public 리소스 (지도 기반 검색 포함).

---

#### Reservation (장착 예약)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Reservation | Create | ❌ | ✅ | ❌ | ✅ |
| Reservation | Read (own, customer) | ❌ | ✅own | ❌ | ✅ |
| Reservation | Read (own shop) | ❌ | ❌ | ✅own | ✅ |
| Reservation | Read (all) | ❌ | ❌ | ❌ | ✅ |
| Reservation | Update (완료처리) | ❌ | ❌ | ✅own | ✅ |
| Reservation | Update (상태변경) | ❌ | ❌ | ✅own | ✅ |
| Reservation | Reject (예약거절) | ❌ | ❌ | ✅own | ✅ |
| Reservation | Cancel | ❌ | ✅own | ✅own | ✅ |
| Reservation | Delete | ❌ | ❌ | ❌ | ❌ |

> - 소유권(Customer): `reservation.order.user_id === currentUser.id`
> - 소유권(Partner): `reservation.shop_id === currentPartner.shop_id`
> - 예약 삭제는 불가 — 취소 상태 변경으로 처리. [💡 권장적용]
> - 장착 완료 처리(상태 변경)는 해당 장착점 Partner 또는 Admin만 가능.
> - `Reject (예약거절)`: Cancel(취소)과 별도 Action. 예약확인대기(PENDING) 상태일 때만 수행 가능하며, 상태를 REJECTED로 전환한다. Partner는 본인 매장 예약에 한해 수행 가능. Cancel은 예약 확정(CONFIRMED) 이후에도 적용 가능한 별개의 상태 전환이다. [💡 권장적용]

---

#### Settlement (정산)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Settlement | Create | ❌ | ❌ | ❌ | ✅ |
| Settlement | Read (own shop) | ❌ | ❌ | ✅own | ✅ |
| Settlement | Read (all) | ❌ | ❌ | ❌ | ✅ |
| Settlement | Update (금액수정) | ❌ | ❌ | ❌ | ✅ |
| Settlement | Process (정산완료) | ❌ | ❌ | ❌ | ✅ |
| Settlement | Cancel | ❌ | ❌ | ❌ | ✅ |
| Settlement | Delete | ❌ | ❌ | ❌ | ❌ |
| Settlement | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 소유권(Partner): `settlement.shop_id === currentPartner.shop_id`
> - Partner는 본인 매장 정산 내역 열람만 가능 (조회 전용).
> - 정산 삭제는 Admin도 불가 (재무 기록 보존 필수).
> - 정산 금액 수정은 정산대기 상태일 때만 Admin이 수행 가능.

---

#### Event (이벤트)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Event | Create | ❌ | ❌ | ❌ | ✅ |
| Event | Read (list/public) | ✅ | ✅ | ✅ | ✅ |
| Event | Read (detail) | ✅ | ✅ | ✅ | ✅ |
| Event | Read (hidden) | ❌ | ❌ | ❌ | ✅ |
| Event | Update | ❌ | ❌ | ❌ | ✅ |
| Event | Toggle visibility | ❌ | ❌ | ❌ | ✅ |
| Event | Delete | ❌ | ❌ | ❌ | ✅ |
| Event | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 노출 상태가 "숨김"인 이벤트는 Admin만 열람 가능.
> - 이벤트 목록·상세는 노출 중인 경우 Public 리소스.

---

#### Notice (공지사항)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Notice | Create | ❌ | ❌ | ❌ | ✅ |
| Notice | Read (list/public) | ✅ | ✅ | ✅ | ✅ |
| Notice | Read (detail) | ✅ | ✅ | ✅ | ✅ |
| Notice | Read (hidden) | ❌ | ❌ | ❌ | ✅ |
| Notice | Update | ❌ | ❌ | ❌ | ✅ |
| Notice | Toggle visibility | ❌ | ❌ | ❌ | ✅ |
| Notice | Toggle 중요공지 | ❌ | ❌ | ❌ | ✅ |
| Notice | Delete | ❌ | ❌ | ❌ | ✅ |
| Notice | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 노출 상태가 "숨김"인 공지는 Admin만 열람 가능.
> - 공지사항 목록·상세는 노출 중인 경우 Public 리소스.

---

#### FAQ

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| FAQ | Create | ❌ | ❌ | ❌ | ✅ |
| FAQ | Read (list/public) | ✅ | ✅ | ✅ | ✅ |
| FAQ | Read (detail) | ✅ | ✅ | ✅ | ✅ |
| FAQ | Read (hidden) | ❌ | ❌ | ❌ | ✅ |
| FAQ | Update | ❌ | ❌ | ❌ | ✅ |
| FAQ | Reorder | ❌ | ❌ | ❌ | ✅ |
| FAQ | Toggle visibility | ❌ | ❌ | ❌ | ✅ |
| FAQ | Delete | ❌ | ❌ | ❌ | ✅ |
| FAQ | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 노출 상태가 "숨김"인 FAQ는 Admin만 열람 가능.
> - FAQ는 카테고리 내 순서 재정렬(드래그앤드롭) 포함.

---

#### Banner (배너)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Banner | Create | ❌ | ❌ | ❌ | ✅ |
| Banner | Read (public/노출중) | ✅ | ✅ | ✅ | ✅ |
| Banner | Read (all, including hidden) | ❌ | ❌ | ❌ | ✅ |
| Banner | Update | ❌ | ❌ | ❌ | ✅ |
| Banner | Reorder | ❌ | ❌ | ❌ | ✅ |
| Banner | Toggle visibility | ❌ | ❌ | ❌ | ✅ |
| Banner | Delete | ❌ | ❌ | ❌ | ✅ |
| Banner | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 노출 중인 배너는 메인화면에서 Public으로 렌더링.
> - 숨김 배너 관리·순서 변경은 Admin 전용.

---

#### User (회원 계정)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| User | Create (회원가입) | ✅ | ❌ | ❌ | ✅ |
| User | Read (own profile) | ❌ | ✅own | ✅own | ✅ |
| User | Read (all) | ❌ | ❌ | ❌ | ✅ |
| User | Update (own profile) | ❌ | ✅own | ✅own | ✅ |
| User | Update (status 변경) | ❌ | ❌ | ❌ | ✅ |
| User | Bulk Update (일괄 활성화/정지) | ❌ | ❌ | ❌ | ✅ |
| User | Reset password (본인) | ❌ | ✅own | ✅own | ✅ |
| User | Reset password (타인) | ❌ | ❌ | ❌ | ✅ |
| User | Delete (own) | ❌ | ✅own | ✅own | ✅ |
| User | Delete (타인) | ❌ | ❌ | ❌ | ✅ |
| User | Bulk Delete (일괄 삭제) | ❌ | ❌ | ❌ | ✅ |
| User | Export (CSV/Excel) | ❌ | ❌ | ❌ | ✅ |

> - 소유권: `user.id === currentUser.id`
> - 회원가입은 Guest(비로그인)만 수행 (로그인 상태에서는 홈 리다이렉트).
> - Customer·Partner 모두 본인 프로필 수정 및 탈퇴(소프트 딜리트) 가능.
> - 회원 상태 변경(활성화/정지)은 Admin 전용.
> - `Bulk Update (일괄 활성화/정지)` 및 `Bulk Delete (일괄 삭제)`: Admin 전용. 개별 `Update (status 변경)` 및 `Delete (타인)` 과 동일한 역할 검증을 일괄 적용한다. [💡 권장적용]
> - `Reset password (본인)`: 마이페이지 비밀번호 직접 변경(현재 비밀번호 확인 방식) 및 이메일 재설정 링크 요청(forgot-password) 모두 포함한다. [💡 권장적용]
> - Partner의 본인 프로필 관리(`Read/Update/Delete own`, `Reset password 본인`)는 `/partner/profile` 또는 `/partner/account` 라우트를 통해 제공한다. Section 3에 해당 라우트 추가 예정. [💡 권장적용]

---

#### Admin Dashboard (어드민 대시보드)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Admin Dashboard | Access | ❌ | ❌ | ❌ | ✅ |
| Admin Dashboard | Read (통계/KPI) | ❌ | ❌ | ❌ | ✅ |
| Admin Dashboard | Read (최근활동) | ❌ | ❌ | ❌ | ✅ |

---

#### Partner Dashboard (파트너 대시보드)

| Resource | Action | Guest | Customer | Partner | Admin |
|----------|--------|-------|----------|---------|-------|
| Partner Dashboard | Access | ❌ | ❌ | ✅own | ✅ |
| Partner Dashboard | Read (예약현황/정산요약) | ❌ | ❌ | ✅own | ✅ |

> - Partner는 본인 매장(`currentPartner.shop_id`) 데이터만 열람.
> - Admin은 전체 파트너 대시보드 데이터 열람 가능. [💡 권장적용]

---

### 7.4 Ownership Rules

#### Customer 소유권

```
resource.user_id === currentUser.id
```

Customer는 자신이 생성한 리소스(차량, 주문, 장바구니, 리뷰, 문의)에만 수정·삭제 권한을 가진다.

#### Partner 소유권

```
resource.shop_id === currentPartner.shop_id
```

Partner는 본인이 운영하는 매장(`shop_id`)과 연결된 리소스(예약, 정산, 매장 정보)에만 접근·수정 권한을 가진다. 다른 장착점 데이터에는 일체 접근 불가.

#### Admin Override

Admin은 소유권 조건 없이 모든 리소스에 대해 Create·Read·Update·Delete를 수행할 수 있다. 단, 아래 예외는 Admin도 불가:

| 예외 작업 | 이유 |
|-----------|------|
| Order 삭제 | 주문 데이터 보존 필수 (취소/환불 상태 변경으로 처리) |
| Settlement 삭제 | 재무 기록 보존 필수 |
| Reservation 삭제 | 예약 기록 보존 필수 (취소 상태 변경으로 처리) |
| Review 직접 작성 | 구매 고객만 작성 가능 (플랫폼 신뢰 보호) |
| Inquiry 직접 작성 | 고객만 작성 가능, 관리자는 답변만 가능 |

#### 블라인드·소프트 딜리트 규칙

- 소프트 딜리트(`is_deleted = true`)된 리소스는 Guest·Customer·Partner 조회에서 제외된다.
- Admin은 소프트 딜리트된 리소스를 포함하여 조회할 수 있다. [💡 권장적용]
- 블라인드 처리된 Review는 `status = BLINDED`로 설정되며 Guest·Customer·Partner에게 비노출.

---

### 7.5 API-Level Enforcement Rules

| 규칙 | 설명 |
|------|------|
| 인증 미보유 요청 | Protected 라우트 접근 시 `401 Unauthorized` 반환, 로그인 페이지 리다이렉트 |
| 권한 불일치 요청 | 역할 불일치 시 `403 Forbidden` 반환, 에러 메시지 표시 |
| 소유권 위반 요청 | 타인 리소스 접근 시 `403 Forbidden` 반환 (존재 여부 노출 금지) [💡 권장적용] |
| Partner 타 매장 접근 | `403 Forbidden` 반환, shop_id 기반 서버사이드 필터링 필수 |
| Admin 세션 만료 | `401` 수신 시 토큰 자동 갱신 시도, 실패 시 로그인 리다이렉트 |
| 감사 로그 | Admin의 모든 CUD 작업은 감사 로그(audit log)에 기록 [💡 권장적용] |
| Bulk 작업 권한 | Bulk 작업(일괄 상태변경, 일괄 삭제, 일괄 블라인드 등)은 해당 개별 작업과 동일한 역할 검증을 각 대상 리소스에 적용한다. Bulk 작업 수행 권한은 Admin 전용이며, 서버사이드에서 각 항목별 권한을 개별 검증 후 처리한다. [💡 권장적용] |
