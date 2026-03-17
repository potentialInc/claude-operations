## Cross-Review Feedback (스코프: 권한 완전성)

> 리뷰어: Section 7 Permission Matrix 작성자
> 검토 대상: Section 3 (User Application) × Section 4 (Admin Dashboard) → Section 7 (Permission Matrix)
> 검토 기준: Section 3/4에 명시된 수정/삭제 기능이 Section 7 Permission Matrix에 해당 Action × Role 행으로 존재하는지

---

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 누락 행 | Section 3 › `/mypage/inquiries` vs Section 7 › Inquiry | Section 3에서 고객이 문의 내역 페이지(`/mypage/inquiries`)에서 문의 수정·삭제 액션을 UI 상에 제공하는 것으로 보이나, Section 7 Inquiry 테이블에 Customer의 `Update` 및 `Delete` 행은 존재한다. 그러나 Section 3의 `/mypage/inquiries` 페이지 스펙에는 수정/삭제 인터랙션이 명시되어 있지 않고 조회 전용(`조회 전용`)으로 기술되어 있다. Section 7에는 허용(`✅own`)이지만 Section 3 UI에는 해당 버튼/인터랙션이 누락되어 있어 권한 정의와 UI 스펙 간 불일치 발생. | Section 3 `/mypage/inquiries` 페이지 스펙에 답변대기 상태 문의에 대한 수정/삭제 버튼 인터랙션을 추가하거나, Section 7 Inquiry `Update` Customer 권한을 재검토하여 실제 노출 여부를 명확히 할 것. |
| 2 | 누락 행 | Section 4 › `/admin/orders` vs Section 7 › Order | Section 4 주문 관리 페이지 Actions 컬럼에 "운송장 번호 입력 및 수정" 기능이 명시되어 있다. 이는 Order 리소스의 부분 Update에 해당한다. Section 7 Order 테이블의 `Update (상태변경)` 행은 Admin만 허용하고 있으나, "운송장 번호 수정"이라는 별도 Action이 Section 7에 독립 행으로 존재하지 않는다. 운송장 번호 수정이 일반 상태변경과 다른 접근 조건(예: 배송 상태에서만 가능)을 가질 수 있으므로 명시 필요. | Section 7 Order 테이블에 `Update (운송장번호 수정)` 행을 Admin: ✅로 추가하거나, 기존 `Update (상태변경)` 행 설명에 운송장 번호 수정 포함 여부를 명시할 것. |
| 3 | 누락 행 | Section 3 › `/partner/reservations/:id` vs Section 7 › Reservation | Section 3 파트너 예약 상세 페이지에 "예약 거절(Reject)" 버튼이 명시되어 있다 (예약확인대기 상태일 때만). Section 7 Reservation 테이블에는 `Cancel` 행(Partner: ✅own)이 있으나 "Reject(거절)"는 별도 Action으로 존재하지 않는다. 거절은 취소와 다른 상태 전환(예약확인대기 → 거절)으로, Cancel(취소, 예약확정 이후)과 구분되는 Action이다. | Section 7 Reservation 테이블에 `Reject (예약거절)` 행을 추가하고 Partner: ✅own, Admin: ✅로 설정할 것. |
| 4 | 누락 행 | Section 4 › `/admin/users` vs Section 7 › User | Section 4 회원 관리 페이지 Bulk Action에 "일괄 활성화 / 일괄 정지 / 일괄 삭제"가 명시되어 있다. Section 7 User 테이블에 `Update (status 변경)`과 `Delete (타인)` 행은 Admin: ✅로 존재한다. 그러나 "일괄(Bulk)" 작업의 권한이 개별 작업과 동일한지 명시되어 있지 않다. Bulk 작업은 개별과 동일 권한을 가지는 것이 일반적이나, 별도 표기 또는 주석이 없어 구현 시 혼선 가능성이 있다. | Section 7 User 테이블 또는 7.5 API-Level Enforcement 규칙에 Bulk 작업(일괄 상태변경, 일괄 삭제)은 개별 작업과 동일한 역할 검증을 적용함을 주석으로 명시할 것. |
| 5 | 누락 행 | Section 4 › `/admin/shops` vs Section 7 › Shop | Section 4 장착점 관리 페이지에서 Admin이 "지도 위치 수정 (카카오맵에서 핀 이동)"을 수행할 수 있다고 명시되어 있다. Section 7 Shop 테이블의 `Update (own shop info)` 행은 Partner: ✅own, Admin: ✅이다. 그러나 "지도 위치(위도/경도) 수정"은 일반 매장 정보 수정(영업시간, 전화번호 등)과 별개의 민감 데이터 변경으로, Partner가 Section 3 `/partner/shop`에서 주소 수정을 할 수 있는지 여부와의 정합성이 불명확하다. Section 3 파트너 매장 정보 관리 페이지에는 주소 입력 필드가 있으나 위도/경도 핀 이동은 명시되어 있지 않다. | Section 7 Shop 테이블에 `Update (위치/좌표 수정)` 행을 별도로 추가하거나, `Update (own shop info)` 행의 주석에 위도/경도 수정은 Admin 전용임을 명시할 것. |
| 6 | 누락 행 | Section 4 › `/admin/reviews` vs Section 7 › Review | Section 4 리뷰 관리 페이지 Actions 컬럼에 "블라인드" 버튼이 명시되어 있다. Section 7 Review 테이블에 `Blind/Unblind` 행이 Admin: ✅로 존재한다. 그러나 Section 4 Bulk Action에는 "일괄 블라인드 / 일괄 노출 해제"가 있다. Section 7에는 개별 `Blind/Unblind`만 있고 Bulk 블라인드에 대한 명시가 없다. (항목 4와 동일한 패턴이나 리뷰 도메인에서도 동일하게 발생.) | Section 7 Review 테이블 또는 7.5 주석에 Bulk Blind/Unblind는 Admin 권한임을 명시할 것. |
| 7 | 빈 셀 의심 | Section 7 › Cart | Section 7 Cart 테이블에 `Update` 행이 없다. Section 3 `/cart` 페이지에서 Customer가 장바구니 상품의 수량을 변경하는 기능이 명시되어 있다 (수량 조절 +/- 버튼). 수량 변경은 CartItem의 Update이므로 Section 7 CartItem 테이블의 `Update (수량변경)` 행(Customer: ✅own)으로 처리됨이 적절하나, Cart 테이블 자체에 Update 행이 없어 Cart 레벨의 수정(예: 선택된 장바구니 항목 일괄 처리)이 명시되지 않은 상태다. Cart와 CartItem의 Update 책임 경계가 불명확하다. | Section 7 Cart 테이블에 Cart 레벨 Update가 없음을 의도적으로 설계한 것인지 주석으로 명시하거나, 필요한 경우 `Update` 행을 추가할 것. 현재 CartItem `Update (수량변경)` 행이 이를 대체하는 것으로 보임을 주석으로 확인할 것. |
| 8 | 누락 행 | Section 3 › `/mypage` vs Section 7 › User | Section 3 마이페이지에서 Customer가 "비밀번호 변경" 기능을 수행한다고 명시되어 있다 (현재 비밀번호 + 새 비밀번호 + 확인 폼). Section 7 User 테이블에 `Reset password (본인)` 행이 Customer: ✅own으로 존재한다. 그러나 Section 3의 마이페이지 비밀번호 변경은 현재 비밀번호를 알고 있는 상태에서의 "변경"이고, Section 7의 `Reset password`는 임시 비밀번호 발송(forgot-password) 맥락으로 혼용될 수 있다. 두 케이스가 동일한 Action으로 묶여 있어 의미가 모호하다. | Section 7 User 테이블의 `Reset password (본인)` 행에 "비밀번호 직접 변경(마이페이지) 및 재설정 링크 요청 포함"임을 주석으로 명시하거나, `Update password (본인)` 와 `Reset password (이메일 재설정)` 로 분리할 것. |

---

### 요약

| 유형 | 건수 |
|------|------|
| 누락 행 (Action이 Section 7에 없음) | 5건 (#2, #3, #5, #6, #8) |
| 불일치 (Section 3 UI와 Section 7 권한 충돌) | 1건 (#1) |
| Bulk 작업 권한 미명시 | 2건 (#4, #6) |
| 빈 셀 / 경계 불명확 | 1건 (#7) |

> **우선순위 높음:** #3 (Reservation Reject — Partner 권한 누락), #5 (Shop 위치 수정 권한 경계), #1 (Inquiry 수정/삭제 UI-권한 불일치)
