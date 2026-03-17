# 타이어뱅크 자사몰 — Product Requirements Document

**Version:** 1.0
**Date:** 2026-03-13
**Status:** Draft

---

## 0. Project Overview

### Product

**Name:** 타이어뱅크 자사몰
**Type:** Web App (반응형 웹)
**Deadline:** 미정
**Status:** Draft

### Description

타이어뱅크 자사몰은 차량번호 기반 타이어 사이즈 자동 매칭과 온라인 구매·장착 예약을 원스톱으로 제공하는 반응형 웹 커머스 플랫폼이다. 고객은 차량번호만 입력하면 적합한 타이어를 추천받고, 자택 배송 또는 제휴 장착점 배송을 선택해 구매·장착까지 완결할 수 있다. 장착점 파트너는 예약 현황 확인과 정산 관리를 전용 화면에서 처리하며, 관리자는 전체 회원·상품·주문·정산·콘텐츠를 통합 관리한다.

### Goals

1. **[Primary]** 차량번호 기반 타이어 추천 → 구매 → 장착 예약까지 단일 플랫폼에서 완결되는 온라인 타이어 판매 채널 구축
2. 제휴 장착점 네트워크와의 연동으로 온·오프라인 구매 경험 통합
3. 관리자·파트너 전용 운영 도구 제공을 통한 효율적 플랫폼 운영 기반 확보
4. 반응형 웹 기반으로 모바일/PC 전 디바이스 접근성 보장

### Target Audience

| Audience | Description |
|----------|-------------|
| **Primary** | 타이어 교체가 필요한 일반 차량 소유 고객 — 차량번호로 맞는 타이어를 쉽게 찾고 장착까지 예약하고 싶은 사용자 |
| **Secondary** | 제휴 장착점 파트너 — 온라인 주문 연계 장착 예약을 관리하고 정산을 확인하려는 정비소 운영자 |

### User Types

| Type | DB Value | Description | Key Actions |
|------|----------|-------------|-------------|
| **Customer (일반 고객)** | `0` | 타이어 구매를 원하는 일반 차량 소유자. 비회원 조회 가능, 구매·예약은 로그인 필요 | 차량번호 조회, 타이어 검색·구매, 장착 예약, 리뷰 작성, 마이페이지 관리 |
| **Partner (장착점 파트너)** | `1` | 타이어뱅크와 제휴된 장착점 운영자. 전용 파트너 대시보드 접근 | 장착 예약 확인·완료 처리, 정산 내역 조회, 매장 정보 관리 |
| **Admin (관리자)** | `99` | 플랫폼 전체를 관리하는 내부 운영자 | 회원·장착점·상품·주문·정산·콘텐츠 전체 관리, CSV/Excel 다운로드 |

### User Status

| Status | DB Value | Behavior |
|--------|----------|----------|
| **Active** | `0` | 정상 이용 가능. 모든 기능 접근 허용 |
| **Suspended** | `1` | 로그인 불가 — 로그인 시도 시 "계정이 정지되었습니다. 고객센터에 문의해 주세요." 메시지 표시. 데이터 보존 유지 |
| **Withdrawn** | `2` | 탈퇴 처리 완료. 개인정보는 탈퇴 후 30일간 보존 후 자동 삭제. 로그인 시 "탈퇴한 계정입니다." 메시지 표시 |

### User Relationships

- **Customer ↔ Partner**: 고객이 장착점 배송을 선택하면 해당 장착점(Partner)과 장착 예약 관계 생성 (1 고객 : N 예약, 1 장착점 : N 예약)
- **Customer → Admin**: 관리자는 모든 고객 계정을 조회·관리할 수 있으며, 문의 답변을 통해 간접 소통
- **Partner → Admin**: 관리자가 장착점을 등록·승인하며 정산을 처리. 장착점은 관리자의 승인 후 활성화
- **계층 구조**: Guest(비로그인) < Customer < Partner < Admin

### Design Reference

| 항목 | 내용 |
|------|------|
| **참고 사이트** | https://abctire.co.kr/ |
| **차별점** | 클라이언트 미지정 — 레퍼런스와 유사한 구조로 구현 |
| **색상/로고** | 레퍼런스 기준: 화이트 배경, 다크그레이 텍스트, 파란색 강조색. 브랜드 색상·로고 미정 `[💡 권장적용]` 클라이언트 브랜드 가이드 확정 후 적용 |
| **레이아웃 참고** | GNB 구조, 메인 히어로 슬라이더+검색바, BEST 상품 섹션, 고객후기 섹션, 푸터 구조를 레퍼런스 기준으로 구현 |
| **특히 중요 화면** | 메인(Home), 상품목록(타이어 목록), 상품상세(타이어 상세), 주문결제, 무료장착점 찾기 — 레퍼런스 시안과 최대한 일치 |

### MVP Scope

**Included:**
- 차량번호 기반 타이어 사이즈 조회 및 상품 추천
- 타이어 목록 검색·필터·정렬
- 타이어 상세페이지 (생산년도/주차, 성능등급, 리뷰)
- 장바구니 및 주문·결제 (PG 연동)
- 배송 방식 선택 (자택 배송 / 장착점 배송)
- 무료 장착점 찾기 (지도 기반, 카카오맵 API)
- 장착 예약 생성 및 관리
- 리뷰 작성·조회
- 이벤트·공지사항·FAQ·1:1 문의
- 마이페이지 (주문내역, 차량관리, 리뷰, 문의내역, 회원정보)
- 장착점 파트너 포털 (예약 확인·완료, 정산 조회, 매장 정보 관리)
- 관리자 대시보드 (회원·장착점·상품·주문·정산·콘텐츠·배너·문의 관리, CSV/Excel 내보내기)
- SMS/알림톡 연동 (주문·배송·장착 알림)
- 반응형 웹 (모바일/PC)
- 감사 로그

**Excluded (deferred):**
- 쿠폰/포인트 시스템 — deferred to Phase 2
- 타이어 비교 기능 상세 — deferred to Phase 2
- 앱(iOS/Android) 전환 — deferred to Phase 3
- 소셜 로그인 (OAuth) — deferred to Phase 2 `[💡 권장적용]` 초기에는 이메일 기반 자체 인증 구현
- 상품 데이터 일괄 CSV 업로드 — deferred to Phase 2 (MVP는 관리자 수동 입력)
- 디자인 QA / 구현본 리뷰 프로세스 — deferred to Phase 2

---

## 1. Terminology

### Core Concepts

| Term | Definition |
|------|------------|
| **타이어뱅크 자사몰** | 타이어뱅크가 운영하는 자체 온라인 타이어 판매 플랫폼. 차량번호 기반 추천부터 구매, 장착 예약까지 원스톱 제공 |
| **장착점** | 타이어뱅크와 제휴된 타이어 장착 서비스를 제공하는 정비소. 고객이 주문한 타이어를 장착해 주는 오프라인 파트너 |
| **장착점 배송** | 고객이 구매한 타이어를 장착점 주소로 직배송하는 배송 방식. 배송 완료 후 고객이 장착점 방문하여 장착 |
| **자택 배송** | 고객이 지정한 자택(또는 원하는 주소)으로 타이어를 배송하는 방식. 장착은 별도로 진행 |
| **장착 예약** | 고객이 특정 장착점에 방문 일시를 사전 예약하는 행위. 장착점 배송 선택 시 자동 연결되거나 별도 생성 가능 |
| **차량번호 조회** | 외부 차량조회 API를 통해 차량번호로 차종·배기량·연식 등 차량 정보를 조회하고, 해당 차량에 맞는 타이어 사이즈를 자동 매칭하는 기능 |
| **타이어 사이즈** | 타이어 규격을 나타내는 표기 (예: 205/55R16). 폭/편평비/림 지름으로 구성 |
| **생산년도/주차** | 타이어 측면에 표기된 제조일 정보. 4자리 숫자로 표기 (예: 3224 = 2024년 32주차) |
| **성능등급** | 타이어의 회전저항, 젖은노면 제동력, 소음 수준을 나타내는 EU 표준 등급 |
| **정산** | 장착점 파트너가 제공한 장착 서비스에 대해 타이어뱅크가 지급하는 서비스 수수료 정산 |
| **파트너 포털** | 장착점 파트너가 예약·정산·매장 정보를 관리하는 전용 웹 인터페이스 |

### User Roles

| Role | Description |
|------|-------------|
| **Guest (비로그인)** | 로그인 없이 접근 가능한 사용자. 상품 목록·상세 조회, 장착점 검색, 이벤트·공지 조회 가능. 장바구니·구매·예약·리뷰 작성 불가 |
| **Customer (일반 고객)** | 회원 가입 후 로그인한 일반 사용자. 타이어 구매, 장착 예약, 리뷰 작성, 마이페이지 관리 가능 |
| **Partner (장착점 파트너)** | 관리자 승인을 받은 제휴 장착점 운영자. 파트너 포털에서 장착 예약·정산·매장 정보 관리 |
| **Admin (관리자)** | 플랫폼 전체 운영 권한을 보유한 내부 관리자. 모든 엔티티에 대한 CRUD 및 정산·승인 권한 |

### Status Values

| Enum | Values | Description |
|------|--------|-------------|
| **UserStatus** | `ACTIVE(0)`, `SUSPENDED(1)`, `WITHDRAWN(2)` | 회원 계정 상태. ACTIVE: 정상 이용, SUSPENDED: 관리자 정지(로그인 차단), WITHDRAWN: 탈퇴 완료(30일 후 데이터 삭제) |
| **OrderStatus** | `PENDING_PAYMENT(0)`, `PAID(1)`, `PREPARING(2)`, `SHIPPING(3)`, `DELIVERED(4)`, `CANCELLED(5)`, `REFUNDED(6)` | 주문 상태. PENDING_PAYMENT: 결제 대기, PAID: 결제 완료, PREPARING: 상품 준비 중, SHIPPING: 배송 중, DELIVERED: 배송 완료, CANCELLED: 주문 취소, REFUNDED: 환불 완료 |
| **DeliveryType** | `HOME(0)`, `SHOP(1)` | 배송 방식. HOME: 자택 배송, SHOP: 장착점 배송 |
| **ReservationStatus** | `PENDING(0)`, `CONFIRMED(1)`, `COMPLETED(2)`, `CANCELLED(3)` | 장착 예약 상태. PENDING: 예약 접수 대기, CONFIRMED: 장착점 확인 완료, COMPLETED: 장착 완료 처리, CANCELLED: 예약 취소 |
| **PartnerStatus** | `PENDING(0)`, `ACTIVE(1)`, `SUSPENDED(2)`, `WITHDRAWN(3)` | 장착점 파트너 상태. PENDING: 승인 대기, ACTIVE: 정상 운영, SUSPENDED: 운영 정지, WITHDRAWN: 탈퇴 |
| **SettlementStatus** | `PENDING(0)`, `PROCESSED(1)`, `COMPLETED(2)` | 정산 상태. PENDING: 정산 대기, PROCESSED: 정산 처리 중, COMPLETED: 정산 완료 |
| **InquiryStatus** | `OPEN(0)`, `ANSWERED(1)`, `CLOSED(2)` | 1:1 문의 상태. OPEN: 답변 대기, ANSWERED: 답변 완료, CLOSED: 문의 종료 |
| **ProductStatus** | `ACTIVE(0)`, `INACTIVE(1)`, `SOLDOUT(2)` | 상품 상태. ACTIVE: 판매 중, INACTIVE: 판매 중지, SOLDOUT: 품절 |

### Technical Terms

| Term | Definition |
|------|------------|
| **PG (Payment Gateway)** | 온라인 결제를 중개하는 결제 게이트웨이. 카드사·은행과 쇼핑몰 사이의 결제 처리를 담당. `[💡 권장적용]` 토스페이먼츠 우선 적용 권장 (국내 점유율, 개발 친화적 API) |
| **웹훅 (Webhook)** | PG사가 결제 완료·실패 등 이벤트 발생 시 서버 엔드포인트로 자동 전송하는 HTTP 콜백 |
| **차량조회 API** | 차량번호를 입력받아 차종·연식·배기량·타이어 권장 사이즈 등 차량 정보를 반환하는 외부 API. `[💡 권장적용]` 자동차민원 대국민포털(공공데이터) 또는 카 히스토리 API 활용 |
| **알림톡** | 카카오 비즈메시지를 통해 카카오톡으로 발송되는 기업 알림 메시지 (주문확인, 배송 안내 등) |
| **소프트 딜리트 (Soft Delete)** | 데이터를 물리적으로 삭제하지 않고 deleted_at 타임스탬프를 기록해 논리적으로 삭제 처리하는 방식 |
| **감사 로그 (Audit Log)** | 관리자 액션(생성·수정·삭제·상태 변경)의 수행자·시간·변경 내용을 기록하는 로그 |
| **JWT (JSON Web Token)** | 로그인 인증 후 발급되는 토큰. Access Token(단기)과 Refresh Token(장기)으로 구성 |

---

## 2. System Modules

### Module 1 — 인증 모듈

회원가입, 로그인, 비밀번호 찾기, JWT 토큰 관리를 담당하는 모듈. 고객·파트너·관리자 모두 이 모듈을 통해 인증한다.

#### Main Features

1. 회원가입 — 이메일·비밀번호 기반 자체 회원가입 `[💡 권장적용]`
2. 로그인 — 이메일/비밀번호 인증, JWT Access/Refresh Token 발급
3. 로그아웃 — Refresh Token 무효화
4. 비밀번호 찾기/재설정 — 이메일 인증 링크 발송
5. 토큰 갱신 — Refresh Token으로 Access Token 재발급
6. 인증 가드 — 라우트별 역할 기반 접근 제어

#### Technical Flow

##### 회원가입 Flow

1. [Customer] 회원가입 폼 제출 (이메일, 비밀번호, 이름, 전화번호) → `POST /api/auth/register`
2. [Server] 이메일 중복 확인, 비밀번호 복잡도 검증 (8자 이상, 영문+숫자+특수문자 `[💡 권장적용]`)
3. [Server] 비밀번호 bcrypt 해싱, User 레코드 생성 (status: ACTIVE)
4. On success: 이메일 인증 메일 발송 → `POST /api/auth/send-verification` → 클라이언트에 가입 완료 응답, 로그인 페이지로 이동
5. On failure (이메일 중복): HTTP 409 반환 → 필드별 오류 메시지 "이미 사용 중인 이메일입니다." 표시
6. On failure (유효성 오류): HTTP 400 반환 → 각 필드 하단에 구체적 오류 메시지 표시 (**버그 예방**: 버튼은 항상 클릭 가능, 오류는 필드별 인라인 표시)

##### 로그인 Flow

1. [Customer/Partner/Admin] 이메일·비밀번호 입력 후 로그인 버튼 클릭 → `POST /api/auth/login`
2. [Server] 이메일로 User 조회, bcrypt 비밀번호 비교
3. [Server] UserStatus 확인: SUSPENDED → HTTP 403 + "계정이 정지되었습니다." 반환, WITHDRAWN → HTTP 403 + "탈퇴한 계정입니다." 반환
4. [Server] Access Token (15분 `[💡 권장적용]`) + Refresh Token (7일 `[💡 권장적용]`) 발급
5. On success: 토큰 저장 (Access Token: Memory, Refresh Token: httpOnly Cookie `[💡 권장적용]`) → 역할별 홈 화면으로 리다이렉트 (Customer → `/`, Partner → `/partner/dashboard`, Admin → `/admin/dashboard`)
6. On failure (자격증명 오류): HTTP 401 → "이메일 또는 비밀번호를 확인해 주세요." 표시 (**버그 예방**: 에러 메시지 1회만 표시)

##### 토큰 갱신 Flow

1. [Client] API 요청 시 Access Token 만료(HTTP 401) 감지
2. [Client] Refresh Token으로 갱신 요청 → `POST /api/auth/refresh`
3. On success: 새 Access Token 발급, 원래 요청 재시도
4. On failure (Refresh Token 만료): 로그인 페이지로 리다이렉트, 에러 메시지 1회 표시 (**버그 예방**: 세션 만료 오류 반복 노출 방지)

---

### Module 2 — 차량 조회 모듈

차량번호를 입력받아 외부 차량조회 API를 통해 차량 정보를 조회하고, 해당 차량에 적합한 타이어 사이즈를 자동으로 매칭한다.

#### Main Features

1. 차량번호 기반 차량 정보 조회 — 외부 API 연동
2. 타이어 권장 사이즈 자동 매칭
3. 조회된 차량 정보 마이페이지 저장/관리
4. 저장된 차량으로 빠른 검색

#### Technical Flow

##### 차량번호 조회 Flow

1. [Customer] 메인 검색바 또는 마이페이지에서 차량번호 입력 후 검색 클릭 → `POST /api/vehicles/lookup`
2. [Server] 입력값 유효성 검사 (차량번호 형식: 숫자+한글+숫자 패턴 `[💡 권장적용]`)
3. [Server] 외부 차량조회 API에 차량번호 전달하여 차종·연식·배기량·타이어 사이즈 조회
4. [Server] 조회 결과를 내부 타이어 사이즈 매핑 테이블과 매칭, 권장 사이즈 반환
5. On success: 차량 정보 + 권장 타이어 사이즈 클라이언트에 반환 → 타이어 목록 페이지로 이동 (사이즈 필터 자동 적용) → `GET /api/products?tire_size={size}`
6. [Customer] (선택) 조회한 차량을 내 차량으로 저장 → `POST /api/my/vehicles`
7. On failure (차량 미조회): HTTP 404 → "차량 정보를 찾을 수 없습니다. 차량번호를 확인해 주세요." 표시
8. On failure (외부 API 오류): HTTP 503 → "일시적인 오류가 발생했습니다. 직접 타이어 사이즈를 입력하여 검색해 주세요." + 수동 사이즈 입력 안내

---

### Module 3 — 상품 관리 모듈

타이어 상품의 CRUD, 재고 관리, 카테고리/브랜드 관리를 담당하는 관리자 전용 모듈.

#### Main Features

1. 타이어 상품 등록·수정·삭제 (관리자)
2. 재고 수량 관리
3. 브랜드·카테고리 관리
4. 상품 상태 관리 (판매중 / 판매중지 / 품절)
5. 상품 이미지 업로드

#### Technical Flow

##### 상품 등록 Flow

1. [Admin] 상품 관리 페이지에서 상품 등록 버튼 클릭 → 상품 등록 모달/폼 오픈
2. [Admin] 상품 정보 입력 (브랜드, 모델명, 타이어 사이즈, 가격, 재고, 생산년도/주차, 성능등급, 이미지) → `POST /api/admin/products`
3. [Server] 입력값 유효성 검사, 이미지 업로드 처리
4. [Server] Product 레코드 생성 (status: ACTIVE)
5. On success: 상품 목록 갱신, 성공 토스트 표시 "상품이 등록되었습니다."
6. On failure (필수 필드 누락): HTTP 400 → 필드별 오류 메시지 표시

##### 재고 관리 Flow

1. [Server] 주문 결제 완료(PAID) 시 해당 상품 재고 차감 → `PATCH /api/admin/products/:id/stock`
2. [Server] 재고 0이 되면 status 자동으로 SOLDOUT 전환
3. [Admin] 수동 재고 수정 가능 → `PATCH /api/admin/products/:id/stock`
4. On success: 재고 수량 즉시 반영
5. On failure (재고 부족 상태에서 주문): HTTP 409 → "재고가 부족합니다." 표시, 주문 불가 처리

---

### Module 4 — 상품 검색 모듈

고객이 타이어를 검색·필터링·정렬하고, 차량번호 기반 자동 추천 결과를 탐색하는 모듈.

#### Main Features

1. 키워드 검색 (브랜드명, 모델명)
2. 필터링 — 브랜드, 타이어 사이즈, 가격 범위, 성능등급
3. 정렬 — 인기순, 낮은 가격순, 높은 가격순, 최신순
4. 차량번호 기반 사이즈 자동 필터 적용
5. 페이지네이션

#### Technical Flow

##### 상품 목록 조회 Flow

1. [Customer] 타이어 목록 페이지 진입 또는 검색/필터 조건 변경 → `GET /api/products?keyword={}&brand={}&tire_size={}&price_min={}&price_max={}&sort={}&page={}&page_size=20`
2. [Server] 쿼리 파라미터 기반 필터링·정렬·페이지네이션 적용하여 상품 목록 반환
3. [Client] 필터 조건을 URL query parameter에 저장 (뒤로가기 시 상태 복원 **버그 예방**)
4. On success: 상품 카드 목록 렌더링 (상품명, 브랜드, 사이즈, 가격, 별점, 리뷰 수)
5. On failure (0건): "검색 결과가 없습니다. 다른 조건으로 검색해 보세요." + 필터 초기화 버튼 표시
6. On failure (서버 오류): HTTP 500 → 에러 메시지 표시 + 재시도 버튼

---

### Module 5 — 장바구니 모듈

고객이 구매할 타이어 상품을 장바구니에 담고 수량을 조정하는 모듈.

#### Main Features

1. 장바구니 상품 추가
2. 수량 변경
3. 상품 삭제
4. 장바구니 목록 조회 (가격 합계 계산)

#### Technical Flow

##### 장바구니 추가/관리 Flow

1. [Customer] 상품 상세 페이지에서 "장바구니 담기" 클릭 → `POST /api/cart/items`
2. [Server] 로그인 여부 확인 — 비로그인 시 로그인 페이지로 리다이렉트
3. [Server] 이미 담긴 상품이면 수량 합산, 신규이면 장바구니 아이템 생성
4. On success: 장바구니 아이콘 뱃지 수량 업데이트, 토스트 "장바구니에 담겼습니다."
5. [Customer] 장바구니 페이지에서 수량 변경 → `PATCH /api/cart/items/:id`
6. [Customer] 아이템 삭제 → `DELETE /api/cart/items/:id`
7. On failure (품절 상품): HTTP 409 → "현재 품절된 상품입니다." 표시
8. [Customer] 주문하기 버튼 클릭 → 주문/결제 모듈로 이동

---

### Module 6 — 주문/결제 모듈

주문 생성, PG 결제 처리, 배송 방식 선택, 주문 상태 관리를 담당하는 핵심 모듈.

#### Main Features

1. 주문 정보 확인 및 주문 생성
2. 배송 방식 선택 (자택 배송 / 장착점 배송)
3. PG 결제 처리 (카드, 계좌이체, 간편결제 `[💡 권장적용]`)
4. 결제 웹훅 처리 및 주문 상태 업데이트
5. 주문 취소 및 환불 처리
6. 주문 상태 Enum 기반 상태 머신 관리

#### Technical Flow

##### 주문·결제 Flow

1. [Customer] 장바구니에서 주문하기 클릭 → 주문 정보 입력 페이지
2. [Customer] 배송지 주소 입력, 배송 방식 선택 (HOME / SHOP), 장착점 배송 선택 시 장착점 선택 및 예약 일시 입력
3. [Customer] 최종 가격 확인 (상품금액, 배송비, 총 결제금액 전체 명시 **버그 예방**) 후 결제하기 클릭
4. [Client] 주문 임시 생성 요청 → `POST /api/orders` (status: PENDING_PAYMENT)
5. [Client] PG SDK 호출하여 결제창 오픈 → PG사 결제 진행
6. [Server] PG사 웹훅 수신 → `POST /api/payments/webhook`
7. [Server] 웹훅 서명 검증, 결제 금액 일치 확인, Order status → PAID 업데이트, 재고 차감 (**버그 예방**: 웹훅 수신 즉시 DB 반영)
8. On success: 주문 완료 페이지로 이동 → `GET /api/orders/:id` → 고객에게 주문확인 SMS/알림톡 발송
9. On failure (결제 실패): PG사 실패 응답 → "결제에 실패했습니다. 다시 시도해 주세요." 표시, Order status → CANCELLED
10. On failure (웹훅 검증 실패): HTTP 400 → 주문 상태 변경 없음, 결제 불일치 로그 기록

##### 주문 취소/환불 Flow

1. [Customer] 마이페이지 주문내역에서 취소 요청 → `POST /api/orders/:id/cancel`
2. [Server] 취소 가능 상태 확인 (PAID, PREPARING 단계만 가능 `[💡 권장적용]`)
3. [Server] PG사 환불 API 호출, Order status → CANCELLED → REFUNDED
4. On success: 취소 완료 알림 SMS/알림톡 발송
5. On failure (취소 불가 상태): HTTP 409 → "배송 시작 이후에는 취소가 불가합니다." 표시

---

### Module 7 — 배송 관리 모듈

자택 배송과 장착점 배송의 배송 상태를 추적·관리하는 모듈.

#### Main Features

1. 배송 상태 추적 (PREPARING → SHIPPING → DELIVERED)
2. 자택 배송 / 장착점 배송 분기 처리
3. 관리자 배송 상태 수동 업데이트
4. 배송 상태 변경 시 고객 알림 발송

#### Technical Flow

##### 배송 상태 업데이트 Flow

1. [Admin] 주문 관리 페이지에서 배송 상태 변경 → `PATCH /api/admin/orders/:id/delivery-status`
2. [Server] 허용된 상태 전환만 처리 (PAID → PREPARING → SHIPPING → DELIVERED, 상태 머신 **버그 예방**)
3. [Server] 배송 상태 업데이트, 변경 이력 기록
4. On success: 고객에게 배송 상태 변경 SMS/알림톡 발송 → `POST /api/notifications/send`
5. [Customer] 마이페이지 주문내역에서 실시간 배송 상태 확인 → `GET /api/my/orders/:id`
6. On failure (잘못된 상태 전환): HTTP 422 → "허용되지 않는 상태 변경입니다." 표시

---

### Module 8 — 장착점 관리 모듈

장착점 등록·승인, 매장 정보 관리, 지도 기반 조회를 담당하는 모듈.

#### Main Features

1. 장착점 등록 및 관리자 승인
2. 매장 정보 관리 (주소, 영업시간, 연락처)
3. 카카오맵 기반 지도 표시 및 지역 검색
4. 장착점 목록 조회 (지역별 필터)

#### Technical Flow

##### 장착점 찾기 Flow

1. [Customer] 무료장착점 찾기 페이지 진입 → `GET /api/shops?region={}&keyword={}`
2. [Client] 카카오맵 SDK 초기화, 장착점 마커 렌더링
3. [Customer] 지역 검색 또는 지도 영역 이동으로 주변 장착점 조회
4. [Customer] 장착점 마커 클릭 → 상세 정보 팝업 (상호명, 주소, 전화번호, 영업시간) → `GET /api/shops/:id`
5. On success: 장착점 목록 및 지도 마커 표시
6. On failure (위치 정보 없음): 전체 장착점 목록 기본 표시

##### 장착점 등록/승인 Flow

1. [Admin] 관리자 장착점 관리 페이지에서 장착점 등록 → `POST /api/admin/shops`
2. [Server] Shop 레코드 생성 (status: PENDING)
3. [Admin] 서류 확인 후 승인 처리 → `PATCH /api/admin/shops/:id/status` (status: ACTIVE)
4. On success: 파트너에게 승인 완료 SMS/알림톡 발송
5. [Partner] 파트너 포털에서 매장 정보 수정 → `PATCH /api/partner/shop`
6. On failure (필수 정보 누락): HTTP 400 → 필드별 오류 메시지 표시

---

### Module 9 — 장착 예약 모듈

고객의 장착 예약 생성, 파트너의 예약 확인 및 완료 처리를 담당하는 모듈.

#### Main Features

1. 장착 예약 생성 (주문 완료 시 연동 또는 별도 생성)
2. 예약 가능 일시 슬롯 조회
3. 파트너의 예약 확인 및 완료 처리
4. 예약 취소

#### Technical Flow

##### 장착 예약 생성 Flow

1. [Customer] 장착점 배송 선택 주문 완료 후 장착 예약 일시 선택 → `GET /api/shops/:id/slots?date={}`
2. [Server] 해당 장착점의 가용 예약 슬롯 반환 (30분 단위 `[💡 권장적용]`)
3. [Customer] 원하는 일시 선택 후 예약 확정 → `POST /api/reservations`
4. [Server] 예약 레코드 생성 (status: PENDING), 해당 슬롯 임시 점유
5. On success: 예약 완료 확인 화면, SMS/알림톡 발송 (고객 + 파트너)
6. [Partner] 파트너 포털에서 예약 확인 → `PATCH /api/partner/reservations/:id/confirm` (status: CONFIRMED)
7. [Partner] 장착 완료 후 처리 → `PATCH /api/partner/reservations/:id/complete` (status: COMPLETED)
8. On success (완료): 정산 내역 자동 생성, 고객에게 장착 완료 알림 발송
9. On failure (슬롯 중복): HTTP 409 → "선택한 시간은 이미 예약되었습니다. 다른 시간을 선택해 주세요."

---

### Module 10 — 정산 모듈

장착점 파트너에 대한 정산 처리 및 내역 관리를 담당하는 모듈.

#### Main Features

1. 장착 완료 건 기반 정산 데이터 자동 생성
2. 정산 내역 조회 (파트너·관리자)
3. 관리자 정산 처리 및 완료 처리
4. 정산 내역 CSV/Excel 내보내기

#### Technical Flow

##### 정산 처리 Flow

1. [Server] 장착 완료(ReservationStatus: COMPLETED) 시 정산 레코드 자동 생성 (status: PENDING) → 내부 이벤트 트리거
2. [Admin] 정산 관리 페이지에서 정산 대상 목록 조회 → `GET /api/admin/settlements?status=PENDING&page={}&page_size=20`
3. [Admin] 정산 처리 버튼 클릭 → `PATCH /api/admin/settlements/:id/status` (status: PROCESSED → COMPLETED)
4. On success: 파트너에게 정산 완료 SMS/알림톡 발송
5. [Partner] 파트너 포털에서 정산 내역 조회 → `GET /api/partner/settlements`
6. [Admin] CSV/Excel 다운로드 → `GET /api/admin/settlements/export?format=csv`
7. On failure (Export 버튼 미동작): Export API 연결 확인 후 활성화, 미연결 시 버튼 비활성화 표시 (**버그 예방**)

---

### Module 11 — 리뷰 모듈

구매 확정 고객의 리뷰 작성·조회와 관리자의 리뷰 관리를 담당하는 모듈.

#### Main Features

1. 구매 완료 상품에 대한 별점·리뷰 작성
2. 상품 상세 페이지 내 리뷰 목록 조회
3. 구매후기 전체 목록 조회 (페이지네이션)
4. 관리자 리뷰 블라인드·삭제

#### Technical Flow

##### 리뷰 작성 Flow

1. [Customer] 마이페이지 주문내역에서 배송 완료 상품의 "리뷰 작성" 버튼 클릭
2. [Customer] 별점(1-5) 선택, 리뷰 텍스트 입력 (최소 10자 `[💡 권장적용]`) → `POST /api/reviews`
3. [Server] 주문 상태 DELIVERED 확인 (구매 인증), 중복 리뷰 방지 확인
4. On success: 리뷰 등록 완료, 상품 평균 별점 업데이트
5. [Customer] 상품 상세 페이지에서 리뷰 목록 조회 → `GET /api/products/:id/reviews?page={}&page_size=10`
6. On failure (미구매 상품 리뷰 시도): HTTP 403 → "구매한 상품에만 리뷰를 작성할 수 있습니다."

---

### Module 12 — 콘텐츠 관리 모듈

이벤트, 공지사항, FAQ, 배너의 CRUD를 담당하는 관리자 전용 모듈.

#### Main Features

1. 이벤트 등록·수정·삭제 및 노출 기간 관리
2. 공지사항 등록·수정·삭제
3. FAQ 등록·수정·삭제·카테고리 관리
4. 메인/서브 배너 등록·수정·삭제·순서 관리

#### Technical Flow

##### 배너/콘텐츠 관리 Flow

1. [Admin] 콘텐츠 관리 페이지에서 아이템 등록/수정/삭제 → `POST /api/admin/banners`, `PATCH /api/admin/banners/:id`, `DELETE /api/admin/banners/:id`
2. [Server] 입력값 유효성 검사, 이미지 업로드 처리
3. [Server] 이벤트/배너의 경우 노출 시작일~종료일 기반 자동 노출/비노출 처리
4. On success: 변경 사항 즉시 반영, 토스트 "저장되었습니다."
5. [Customer] 메인 페이지 배너/이벤트 조회 → `GET /api/banners`, `GET /api/events`
6. On failure (이미지 업로드 실패): HTTP 413 또는 422 → "이미지 업로드에 실패했습니다. 파일 크기와 형식을 확인해 주세요."

---

### Module 13 — 문의 모듈

고객의 1:1 문의 작성·조회와 관리자의 답변·상태 관리를 담당하는 모듈.

#### Main Features

1. 고객 1:1 문의 작성
2. 문의 내역 조회 (고객·관리자)
3. 관리자 답변 등록
4. 문의 상태 관리 (OPEN → ANSWERED → CLOSED)

#### Technical Flow

##### 문의 작성/답변 Flow

1. [Customer] 고객센터 > 문의하기에서 문의 유형·제목·내용 입력 → `POST /api/inquiries`
2. [Server] Inquiry 레코드 생성 (status: OPEN)
3. [Admin] 관리자 문의 관리 페이지에서 문의 목록 조회 → `GET /api/admin/inquiries?status={}&page={}`
4. [Admin] 문의 상세 확인 후 답변 등록 → `POST /api/admin/inquiries/:id/answer` (status: ANSWERED)
5. On success: 고객에게 답변 완료 SMS/알림톡 발송
6. [Customer] 마이페이지 문의내역에서 답변 확인 → `GET /api/my/inquiries`
7. On failure (필수 필드 누락): HTTP 400 → 필드별 오류 메시지 표시

---

### Module 14 — 알림 모듈

주문·배송·장착 예약 관련 SMS 및 카카오 알림톡 발송을 담당하는 모듈.

#### Main Features

1. 주문 확인 알림 (결제 완료 시)
2. 배송 상태 변경 알림
3. 장착 예약 확인·완료 알림
4. 문의 답변 완료 알림
5. 정산 완료 알림 (파트너)

#### Technical Flow

##### 알림 발송 Flow

1. [Server] 이벤트 트리거 발생 (주문 완료, 배송 상태 변경, 장착 완료 등)
2. [Server] 수신자 연락처 및 알림 유형 확인, 알림 발송 큐에 추가
3. [Server] SMS/알림톡 API 호출 → `POST /api/notifications/send` (내부 서비스 레이어)
4. [3rd Party] SMS 발송사 또는 카카오 비즈메시지 API를 통해 메시지 발송
5. On success: 발송 결과 로그 기록
6. On failure (발송 실패): 재시도 최대 3회 `[💡 권장적용]`, 최종 실패 시 로그 기록. 알림 실패가 주문 프로세스에 영향을 주지 않도록 비동기 처리

---

### Module 15 — 마이페이지 모듈

로그인한 고객이 주문내역, 차량 관리, 리뷰, 문의내역, 회원정보를 관리하는 모듈.

#### Main Features

1. 주문내역 조회 및 배송·장착 상태 추적
2. 내 차량 등록·수정·삭제
3. 작성한 리뷰 조회
4. 1:1 문의내역 조회
5. 회원정보 수정 및 비밀번호 변경
6. 회원 탈퇴

#### Technical Flow

##### 마이페이지 조회 Flow

1. [Customer] 마이페이지 접근 → 인증 가드 확인 (비로그인 시 로그인 페이지로 리다이렉트 **버그 예방**)
2. [Customer] 주문내역 탭 → `GET /api/my/orders?page={}&page_size=10&sort=created_at_DESC`
3. [Server] 페이지네이션 적용하여 고객의 주문 목록 반환 (**버그 예방**: page, page_size 파라미터 지원, 기본 정렬 created_at DESC)
4. [Customer] 주문 상세 클릭 → `GET /api/my/orders/:id` → 배송 상태, 장착 예약 정보 표시
5. On success: 주문 목록 렌더링 (주문번호, 상품명, 금액, 주문 상태, 배송 상태)
6. On failure (0건): "주문 내역이 없습니다." + "타이어 구경하기" CTA 버튼 표시

---

### 3rd Party API List

| Service | Purpose | Integration Point | Alternative |
|---------|---------|-------------------|-------------|
| **차량조회 API** | 차량번호로 차종·연식·타이어 사이즈 조회 | 메인 검색바, 마이페이지 차량관리 | 공공데이터포털 자동차 API `[💡 권장적용]` |
| **카카오맵 API** | 장착점 위치 지도 표시, 지역 기반 검색, 주소 좌표 변환 | 무료장착점 찾기 페이지 | 네이버 지도 API |
| **PG (결제 게이트웨이)** | 신용카드·계좌이체·간편결제 온라인 결제 처리 및 웹훅 수신 | 주문/결제 페이지, 결제 웹훅 엔드포인트 | NHN KCP, 이니시스 `[💡 권장적용]` 토스페이먼츠 우선 권장 |
| **SMS API** | 주문확인·배송상태·장착예약 등 문자 알림 발송 | 알림 모듈 전체 | 네이버 클라우드 SENS `[💡 권장적용]` NHN Cloud SMS 또는 카카오 알림톡으로 대체 가능 |
| **카카오 비즈메시지 (알림톡)** | 카카오톡을 통한 주문·배송·장착 알림 발송 | 알림 모듈 전체 | SMS 대체 발송 (알림톡 실패 시 SMS 폴백 `[💡 권장적용]`) |
