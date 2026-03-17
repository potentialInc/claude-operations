# Bug Patterns - Part 3

분석 기준일: 2026-03-12
원본 파일: bugs-part3.md
총 버그 수: ~100건

---

## 카테고리별 패턴 목록

### 1. 푸시 알림

---

### Pattern: 알림 미수신 및 백그라운드 알림 불가
- **빈도**: 5회 (높음)
- **증상**: 앱이 꺼져 있거나 백그라운드 상태일 때 푸시 알림이 도달하지 않음. 알림을 아예 받지 못하는 경우도 발생.
- **근본 원인**: 푸시 알림 전송 로직이 앱 포그라운드 상태에만 의존하거나, FCM/APNs 토큰 갱신 및 백그라운드 처리 설정 누락
- **영향 화면**: 알림 수신 전반, 푸시 알림 설정
- **예방 스펙**: "[클라이언트] FCM/APNs 백그라운드 모드(background fetch, remote notifications)를 반드시 활성화하고, 앱 종료/백그라운드 상태에서도 silent push 및 data payload 방식으로 알림이 도달하도록 구현한다. [서버] 알림 전송 시 priority=high 설정, TTL(time-to-live) 지정 필수. 알림 전송 실패 시 재시도 로직 포함."
- **사례**: [Roadmap #Notification issues], [Roadmap #Push notifications should be delivered regardless of phone state], [K Talk #Notifications are not sent], [Near Work #Notification titles are not providing accurate information], [K Talk #notification]

---

### Pattern: 알림 뱃지/카운트 오류
- **빈도**: 6회 (높음)
- **증상**: 읽지 않은 알림이 없는데도 빨간 점(red dot)이 표시됨. 알림 숫자가 잘려 보이거나 정적인 값으로 고정됨. 채팅 알림 카운트가 읽음 처리 후에도 초기화되지 않음.
- **근본 원인**: 알림 카운트 업데이트 API 호출 누락 또는 로컬 상태와 서버 상태 간 동기화 미흡. 뱃지 숫자 UI 컴포넌트의 최대 표시 길이 미처리.
- **영향 화면**: 알림 탭 뱃지, 채팅 알림 카운트, 홈 화면 아이콘 뱃지
- **예방 스펙**: "[클라이언트] 알림 목록 진입 시 서버에서 unread count를 재조회하여 로컬 상태를 동기화한다. 뱃지 숫자는 99+처럼 오버플로우 처리 필수. [서버] '모두 읽음' 처리 API는 실제 DB 업데이트 후 count=0 응답을 보장해야 하며, 클라이언트는 응답 수신 후 즉시 뱃지를 갱신한다."
- **사례**: [Ivory #No new notification but showing red dot], [Ivory #There is no notification but still showing red dot], [Coaching Record #Notification number cut and static], [Coaching Record #Notification number cut and static (2)], [Coaching Record #notification number need the actual numbers], [Coaching Record #Chat notification count number not clearing]

---

### Pattern: 알림 중복 표시 및 페이지 오동작
- **빈도**: 4회 (중간)
- **증상**: 같은 알림이 두 번 표시됨. 알림 클릭 시 반응 없음. 알림 페이지 자체가 동작하지 않음. 회원가입 후 이전 알림 데이터가 노출됨.
- **근본 원인**: 알림 리스너 중복 등록, 딥링크 핸들러 미연결, 신규 가입 사용자에 대한 알림 데이터 초기화 누락
- **영향 화면**: 알림 목록 페이지, 알림 클릭 딥링크, 회원가입 후 알림 초기 상태
- **예방 스펙**: "[클라이언트] 알림 리스너는 컴포넌트 마운트 시 1회만 등록하고 언마운트 시 반드시 해제한다. 알림 클릭 시 딥링크 또는 해당 화면으로 이동하는 핸들러를 필수 구현한다. [서버] 신규 회원 생성 시 알림 데이터를 빈 상태로 초기화한다."
- **사례**: [Ivory #Notification shows two times], [K Talk #Notification is not clickable], [Ivory #Notification Page Not working], [Roadmap #Notification issues(old data)]

---

### Pattern: 알림 아이콘 위치 및 메시지 정확도 오류
- **빈도**: 3회 (중간)
- **증상**: 알림 아이콘 위치가 잘못됨. 알림 제목/메시지가 부정확하거나 잘못된 정보를 포함. 알림 메시지가 올바르게 표시되지 않음.
- **근본 원인**: 알림 템플릿 변수 바인딩 오류, 알림 아이콘 레이아웃 미정의, 알림 발송 시 컨텍스트 데이터 오삽입
- **영향 화면**: 알림 아이콘 UI, 알림 메시지 본문
- **예방 스펙**: "[서버] 알림 메시지 템플릿에 삽입되는 변수(사용자명, 콘텐츠명 등)는 발송 전 null/empty 여부를 검증한다. [클라이언트] 알림 아이콘은 디자인 시스템의 표준 아이콘을 사용하고, 위치는 Figma 스펙 기준 픽셀 단위로 고정한다."
- **사례**: [K Talk #Notification icon placement], [Near Work #Notification titles are not providing accurate information], [Coaching Record #Notification message is not Shown properly]

---

### 2. 이미지/파일 업로드

---

### Pattern: 파일 업로드 실패 및 요구사항 오류
- **빈도**: 3회 (중간)
- **증상**: 파일 업로드가 실패함. 업로드 파일 요구사항(크기, 포맷)이 명확하지 않거나 잘못 표시됨. STL 파일 업로드가 작동하지 않음.
- **근본 원인**: 파일 타입/크기 유효성 검사 로직 누락 또는 서버 측 파일 처리 오류, 에러 메시지 미표시
- **영향 화면**: 파일 업로드 화면, STL 파일 업로드, 주문 첨부 파일
- **예방 스펙**: "[클라이언트] 업로드 전 허용 파일 형식과 최대 크기를 명확히 안내하고, 조건 미충족 시 구체적인 에러 메시지를 표시한다. [서버] 파일 업로드 API는 허용 MIME 타입 화이트리스트와 최대 파일 크기를 서버 측에서도 검증하고, 실패 시 명확한 오류 코드를 반환한다."
- **사례**: [Elite4Print #Upload New File Issues], [Elite4Print #Upload File Requirement Issue], [Ivory #Upload approved designed STL not working]

---

### Pattern: 프로필 이미지 변경 실패 및 앱 크래시
- **빈도**: 3회 (중간)
- **증상**: 프로필 이미지 변경 후 반영되지 않음. 카메라로 프로필 사진 변경 시 앱이 크래시됨(iOS/Android 모두 발생).
- **근본 원인**: 카메라 권한 처리 미흡, 이미지 압축/리사이징 처리 중 메모리 오류, 업로드 후 UI 캐시 갱신 누락
- **영향 화면**: 프로필 편집 화면, 이미지 선택기
- **예방 스펙**: "[클라이언트] 카메라/갤러리 접근 전 권한 요청 플로우를 반드시 구현하고, 권한 거부 시 설정 안내 팝업을 표시한다. 이미지 업로드 전 최대 2MB로 압축하고, 업로드 성공 후 즉시 로컬 캐시를 무효화하여 새 이미지를 표시한다. [서버] 이미지 업로드 완료 응답에 새 이미지 URL을 포함하여 반환한다."
- **사례**: [K Talk #Profile image didn't change], [Coaching Record #Changing profile picture on iphone/Android by camera crashes app], [Coaching Record #Changing profile picture on iphone/Android by camera crashes app (2)]

---

### Pattern: 갤러리 접근 권한 미요청
- **빈도**: 1회 (낮음)
- **증상**: 앱이 갤러리 접근 권한을 요청하지 않고 사진이 업로드됨
- **근본 원인**: 권한 요청 로직 누락 (OS 권한 API 미호출)
- **영향 화면**: 사진 업로드 화면
- **예방 스펙**: "[클라이언트] 갤러리/카메라 접근 시 반드시 OS 권한 API를 통해 사용자 동의를 구한다. iOS는 NSPhotoLibraryUsageDescription, Android는 READ_EXTERNAL_STORAGE/READ_MEDIA_IMAGES 권한을 명세하고 런타임에 요청한다."
- **사례**: [Entermong #180 uploading photos]

---

### 3. 리스트/페이지네이션

---

### Pattern: 페이지네이션 미구현 및 정렬 오류
- **빈도**: 5회 (높음)
- **증상**: 목록 페이지에 페이지네이션이 없어 모든 데이터가 한 번에 표시됨. 최신 항목이 목록 상단에 표시되지 않음. 페이지 번호가 제대로 동작하지 않음.
- **근본 원인**: 페이지네이션 API 파라미터 미적용, 기본 정렬 조건 미설정(created_at DESC 누락), UI 컴포넌트 미구현
- **영향 화면**: 사용자 목록, 블로그 목록, 어드민 패널 목록, 주문 목록
- **예방 스펙**: "[서버] 목록 조회 API는 반드시 page, page_size 파라미터를 지원하고 기본 정렬은 created_at DESC로 설정한다. [클라이언트] 10개 이상의 데이터가 예상되는 모든 목록 화면에 페이지네이션 또는 무한 스크롤을 구현하며, 빈 상태(empty state) UI도 함께 제공한다."
- **사례**: [Near Work #Need a pagination system], [Thrll #website blog page don't have pagination], [Onui CRM #need Sorting by created at], [Onui CRM #New user should be display top of the list], [Onui CRM #328 parent management main list sorting needs to be descending order]

---

### Pattern: 뒤로가기 시 페이지/검색 상태 초기화
- **빈도**: 2회 (낮음)
- **증상**: 목록에서 상세 페이지 이동 후 뒤로가기 시 페이지 번호와 검색 결과가 초기화됨. 필터 조건도 함께 리셋됨.
- **근본 원인**: 목록 페이지 상태가 컴포넌트 로컬 state로만 관리되어 언마운트 시 소멸. 전역 상태 또는 URL query param 활용 미흡.
- **영향 화면**: 주문 목록, 검색 결과 목록
- **예방 스펙**: "[클라이언트] 목록 페이지의 현재 페이지 번호, 검색어, 필터 조건은 URL query parameter 또는 전역 상태 관리(Redux, Zustand 등)에 저장하여 뒤로가기 후 복원되도록 구현한다."
- **사례**: [Ivory #Pagination and search state reset after navigating back], [Ivory #1160 Issue on order details after data is revised with back buttons]

---

### Pattern: 필터 적용 후 자동 초기화
- **빈도**: 5회 (높음)
- **증상**: 필터 모달을 닫고 다시 열면 설정한 필터(진단, 날짜, 통화 결과)가 초기화됨. 필터 적용 자체가 동작하지 않음.
- **근본 원인**: 필터 상태가 모달 컴포넌트 내부 로컬 state로만 관리되어 닫힘 시 소멸. 필터 파라미터가 API 요청에 포함되지 않음.
- **영향 화면**: 콜로그 관리 필터, 리드 목록 필터, 투데이 어사인먼트 필터, 플레이북 필터
- **예방 스펙**: "[클라이언트] 필터 조건은 상위 컴포넌트 또는 전역 상태에 보관하여 모달 재오픈 시 이전 선택값이 유지되도록 한다. 필터 적용 버튼 클릭 시 해당 조건을 API 요청 파라미터에 포함하고, 필터 초기화 버튼을 별도 제공한다."
- **사례**: [Onui CRM #1478 Need to check Calllog management filter], [Onui CRM #1489 call log management page], [Onui CRM #1490 lead management page], [Playbook #1486 Filter issue], [Playbook #1485 filter issue]

---

### 4. 결제/인앱구매

---

### Pattern: 결제 완료 후 UI 미반영 및 히스토리 오류
- **빈도**: 7회 (높음)
- **증상**: 결제 완료 후 성공 메시지 없이 회색 화면 표시. 결제 내역 페이지가 업데이트되지 않음. 결제하지 않았는데 히스토리에 데이터 표시. 대시보드 결제 금액과 결제 내역 불일치.
- **근본 원인**: 결제 콜백/웹훅 처리 후 UI 상태 갱신 로직 누락. 카드 등록 이벤트와 실제 결제 완료 이벤트 혼동. 대시보드 집계 쿼리 오류.
- **영향 화면**: 결제 완료 화면, 결제 내역, 대시보드 결제 금액
- **예방 스펙**: "[서버] 결제 완료 웹훅 수신 후 즉시 결제 상태를 DB에 반영하고, 클라이언트 폴링 또는 소켓 이벤트로 결과를 전달한다. 카드 등록과 결제 완료는 별도 이벤트로 분리 처리한다. [클라이언트] 결제 완료 시 성공/실패 메시지를 명확히 표시하고 결제 내역 페이지로 자동 이동한다."
- **사례**: [Roadmap #dashboard payment amount different], [Roadmap #I didn't payment complete but history has data], [Roadmap #payment history still not showing], [Coaching Record #1674 Payment success is showing raw message], [Bright Pixel #673 Not getting success message after payment], [APP #1414 payment issue], [Roadmap #App payment information table issue]

---

### Pattern: 결제 취소 및 환불 처리 오류
- **빈도**: 3회 (중간)
- **증상**: 결제가 취소되었는데 취소 결제와 현재 결제가 동시에 표시됨. 환불 요청 후 후속 알림/액션 없음. 구독 취소가 작동하지 않음.
- **근본 원인**: 결제 취소 상태 반영 로직 미구현. 환불 요청 후 상태 업데이트 및 알림 발송 플로우 누락.
- **영향 화면**: 결제 내역, 환불 요청 화면, 구독 관리 화면
- **예방 스펙**: "[서버] 결제 취소/환불 완료 시 해당 결제 레코드 상태를 즉시 업데이트하고, 취소된 결제는 목록에서 '취소됨' 상태로 표시한다. 환불 처리 완료 시 사용자에게 알림을 발송한다. [클라이언트] 취소된 결제는 별도 취소 이력 탭 또는 상태 배지로 구분 표시한다."
- **사례**: [Ivory #Even the payment is cancelled it is showing cancel payment and current payment], [Ivory #No follow-up action/notification with Refund request], [Ivory #Cancel subscription is not working]

---

### Pattern: 쿠폰/프로모션 적용 오류 및 가격 계산 오류
- **빈도**: 3회 (중간)
- **증상**: 쿠폰 코드 적용 시 체크아웃 불가. 장바구니 금액이 소수점 3자리까지 표시됨. 배송비/추가 금액이 인보이스에 미반영. 특정 치료 항목이 가격에 미산정.
- **근본 원인**: 쿠폰 적용 후 최종 금액 계산 로직 오류. 부동소수점 반올림 처리 미흡. 추가 요금 항목이 인보이스 생성 로직에 포함되지 않음.
- **영향 화면**: 장바구니, 체크아웃, 인보이스, 가격 견적
- **예방 스펙**: "[서버] 모든 금액 계산은 정수(최소 단위: 원/센트) 기반으로 처리하여 부동소수점 오류를 방지한다. 쿠폰 적용 후 최종 금액을 서버에서 재계산하여 반환한다. 인보이스 생성 시 배송비, 추가 요금 등 모든 비용 항목을 포함하는지 체크리스트로 검증한다."
- **사례**: [Elite4Print #2094 Coupon Code/Promotion], [Elite4Print #2286 Local Delivery Fees], [Ivory #pontic crown not counted in the price]

---

### Pattern: 결제 후 페이지 리다이렉트 오류
- **빈도**: 2회 (낮음)
- **증상**: 결제 완료 후 다시 결제 페이지로 리다이렉트됨. 다른 이메일로 가입해도 이전 세션의 페이지로 이동됨.
- **근본 원인**: 결제 완료 후 세션/상태 클리어 미흡. 라우팅 로직에서 이전 세션 데이터 잔존.
- **영향 화면**: 결제 완료 후 리다이렉트, 회원가입 후 리다이렉트
- **예방 스펙**: "[클라이언트] 결제 완료 또는 로그아웃 시 결제 관련 세션 데이터와 로컬 스토리지를 완전히 초기화한다. 리다이렉트 URL은 서버에서 검증된 값만 사용한다."
- **사례**: [Thrll #Some payment redirecting to payment page again from stripe], [Thrll #987 Redirecting Issue]

---

### 5. 인증/로그인

---

### Pattern: 로그인/회원가입 동작 불가
- **빈도**: 5회 (높음)
- **증상**: 로그인 버튼이 반응하지 않음. 회원가입이 작동하지 않음. 가입 중 멈춤. 로그아웃 시 401 에러 발생.
- **근본 원인**: 폼 유효성 검사 로직 오류로 인한 버튼 비활성화 유지. API 엔드포인트 오류 또는 토큰 처리 실패. 로그아웃 API 호출 시 만료된 토큰 전송.
- **영향 화면**: 로그인 화면, 회원가입 화면, 로그아웃 처리
- **예방 스펙**: "[클라이언트] 로그인/회원가입 버튼은 폼 유효성 검사 통과 여부와 무관하게 클릭 가능하도록 하고, 검사 실패 시 각 필드에 구체적인 오류 메시지를 표시한다. [서버] 로그아웃 API는 토큰 만료 여부와 관계없이 성공 응답을 반환하고 서버 측 토큰을 무효화한다."
- **사례**: [Coaching Record #Login button not responding], [K Talk #517 Sign Up is not working], [K Talk #stuck at signup], [K Talk #Login issue in client Server-site], [#1064 User log out error]

---

### Pattern: 소셜 로그인 화면 누락 및 기존 사용자 재가입 허용
- **빈도**: 3회 (중간)
- **증상**: 소셜 회원가입 시 중간 화면이 누락됨. 이미 가입된 이메일로 재가입이 가능하고 이름/전화번호 입력 후 홈으로 이동됨.
- **근본 원인**: 소셜 로그인 플로우 일부 화면 미구현. 회원가입 시 이메일 중복 검사 로직 누락.
- **영향 화면**: 소셜 회원가입 플로우, 이메일 중복 확인
- **예방 스펙**: "[서버] 회원가입 API는 이메일/소셜 계정 중복 여부를 먼저 확인하고, 기존 사용자에게는 '이미 가입된 계정입니다' 메시지와 함께 로그인 화면으로 안내한다. [클라이언트] 소셜 로그인 플로우의 모든 단계를 Figma 스펙 기준으로 구현하고 QA 체크리스트에 포함한다."
- **사례**: [Coaching Record #Social signup one screen missing], [Coaching Record #Social signup one screen missing (2)], [Ivory #If existed user tries to signup]

---

### 6. 관리자 대시보드

---

### Pattern: 어드민 필터/통계 오류
- **빈도**: 4회 (중간)
- **증상**: 주문 목록의 'All Filter'가 대기 상태만 표시. 어드민 통계 지표가 일부 상태만 보임(MQL). 콜로그 편집 권한 설정이 적용되지 않음.
- **근본 원인**: 필터 API 파라미터 기본값 오설정. 통계 쿼리에서 전체 상태 값 미포함. 권한 체크 로직 미구현.
- **영향 화면**: 주문 목록 필터, MQL 통계, 권한 관리, 콜로그 관리
- **예방 스펙**: "[서버] '전체' 필터 선택 시 status 파라미터를 생략하거나 all 값으로 전체 데이터를 반환한다. 통계 API는 모든 상태값을 집계에 포함하고, 값이 0인 상태도 응답에 포함한다. [클라이언트] 권한 설정 저장 후 해당 권한을 즉시 적용하고 재로그인 없이 반영되는지 검증한다."
- **사례**: [Ivory #All Filter in Order List Displays Only Pending Status], [Onui CRM #MQL - status], [Onui CRM #Permission management: Call log edit is not working], [Onui CRM #799 My info permission]

---

### Pattern: 관리자 벌크/단일 액션 오류
- **빈도**: 3회 (중간)
- **증상**: 어드민에서 사용자 삭제 불가. 어드민이 어드민을 생성할 수 없음. 주문 보류(Hold) 처리 시 에러 발생.
- **근본 원인**: 삭제/생성 API 권한 설정 오류. Hold 상태 처리 로직 미완성.
- **영향 화면**: 사용자 관리, 어드민 관리, 주문 관리
- **예방 스펙**: "[서버] 어드민 전용 API 엔드포인트는 role 기반 접근 제어(RBAC)를 적용하고, superadmin이 admin을 생성/삭제할 수 있는 권한 체계를 명확히 정의한다. 모든 상태 전환(Hold 포함)에 대한 API를 사전 구현하고 테스트한다."
- **사례**: [#789 Unable to delete any user from the user table], [#484 admin can't create the admin], [#1212 Hold order error]

---

### 7. 검색/필터

---

### Pattern: 검색 기능 미동작
- **빈도**: 3회 (중간)
- **증상**: 교사 검색 필터가 작동하지 않음. 주문 목록 검색바 오동작. 투데이 어사인먼트 필터에서 리드가 표시되지 않음.
- **근본 원인**: 검색 API 파라미터 연결 누락. 검색어 디바운스 처리 미흡으로 불필요한 API 호출 또는 결과 미반영.
- **영향 화면**: 교사 검색, 주문 목록 검색, 리드 필터 검색
- **예방 스펙**: "[클라이언트] 검색 입력 필드는 300ms 디바운스를 적용하고, 입력값을 API 요청 파라미터로 연결한다. 검색 결과가 없을 경우 '검색 결과 없음' 빈 상태 UI를 표시한다. [서버] 검색 API는 대소문자 구분 없는 부분 일치(ILIKE)를 지원한다."
- **사례**: [K Talk #Search with teacher is not working], [Ivory #Searce bar is not working properly for Order List Page], [#548 today assignment filter issue]

---

### 8. 실시간 데이터/채팅

---

### Pattern: 채팅 UI 오류 및 메시지 오동작
- **빈도**: 4회 (중간)
- **증상**: 채팅 텍스트가 UI 영역을 벗어나 overflow됨. 채팅 상대방 이름이 잘못 표시됨. 메시지 내용이 잘못됨. 채팅 페이지 전반적 오류.
- **근본 원인**: 채팅 말풍선 최대 너비 CSS 미설정. 채팅 참여자 데이터 바인딩 오류. 메시지 발신자/수신자 구분 로직 오류.
- **영향 화면**: 채팅 화면, 메시지 목록
- **예방 스펙**: "[클라이언트] 채팅 메시지 말풍선은 최대 너비(max-width: 70-80%)를 설정하고 긴 텍스트는 word-break: break-word를 적용한다. 채팅 참여자 정보는 채팅방 입장 시 서버에서 재조회하여 최신 데이터를 표시한다."
- **사례**: [K Talk #Chat Page], [Ivory #chat text running over], [Coaching Record #Chat restricted. Chat person name incorrect.], [Ivory #Message is wrong]

---

### Pattern: 실시간 데이터 업데이트 미반영
- **빈도**: 2회 (낮음)
- **증상**: 주문 상태 변경이 즉시 반영되지 않음. 주차장 상세 페이지 실시간 섹션이 동작하지 않음.
- **근본 원인**: 실시간 업데이트 폴링 또는 웹소켓 구독 미구현.
- **영향 화면**: 주문 상태, 주차장 실시간 현황
- **예방 스펙**: "[클라이언트] 실시간 업데이트가 필요한 화면은 웹소켓 구독 또는 최소 30초 간격 폴링을 구현한다. [서버] 상태 변경 이벤트 발생 시 구독 중인 클라이언트에 즉시 푸시한다."
- **사례**: [Ivory #Order Status Update Not Reflected Instantly], [Roadmap #Parking lot detail page, realtime section]

---

### 9. 폼/입력

---

### Pattern: 폼 유효성 검사 미동작
- **빈도**: 4회 (중간)
- **증상**: 비밀번호 확인 유효성 검사가 작동하지 않음. 전화번호 입력 형식 안내/자동 변환 없음. 입력 필드 자체가 동작하지 않음. 오류 메시지가 올바르지 않음.
- **근본 원인**: 유효성 검사 규칙 미정의 또는 연결 누락. 입력 포맷 마스킹 미적용. 에러 메시지 하드코딩 오류.
- **영향 화면**: 비밀번호 변경, 전화번호 입력, 각종 폼 필드
- **예방 스펙**: "[클라이언트] 모든 입력 필드는 실시간 유효성 검사를 적용하고, 각 오류 유형에 맞는 구체적인 안내 메시지를 표시한다. 전화번호 필드는 허용 자릿수와 포맷(예: 010-XXXX-XXXX)을 자동 마스킹 또는 입력 안내 placeholder로 제공한다."
- **사례**: [Onui CRM #Myprofile edit password: Validation is not working], [Thrll #1094 Office Phone number Error issue], [Trustix #Input fields are not working], [#900 Validation error message needs minor modification]

---

### Pattern: 다단계 폼 데이터 유지 오류
- **빈도**: 2회 (낮음)
- **증상**: 뒤로가기 버튼 사용 시 이전에 입력한 데이터가 초기화되거나 이전 데이터로 덮어써짐. 주문 수정 시 변경사항이 반영되지 않음.
- **근본 원인**: 다단계 폼 상태가 각 단계 컴포넌트 로컬 state로만 관리되어 뒤로가기 시 소멸.
- **영향 화면**: 주문 생성/수정 다단계 폼
- **예방 스펙**: "[클라이언트] 다단계 폼의 입력 데이터는 전역 상태 또는 폼 컨텍스트에 저장하여 단계 간 이동 시 데이터가 유지되도록 한다. '뒤로가기' 버튼은 이전 단계 데이터를 복원하고, 최종 제출 전까지 데이터를 보존한다."
- **사례**: [Ivory #1160 Issue on order details after data is revised with back buttons], [Ivory #Edit order selection/summary discrepancy]

---

### 10. 네비게이션/라우팅

---

### Pattern: 딥링크 및 라우팅 오류
- **빈도**: 2회 (낮음)
- **증상**: 특정 섹션에서 계속 버튼 클릭 시 이전 섹션으로 루프백됨. 로그인 후 이전 세션의 페이지로 잘못 리다이렉트됨.
- **근본 원인**: 네비게이션 스택 관리 오류. 세션 초기화 없이 이전 라우트 히스토리가 유지됨.
- **영향 화면**: 주문 플로우 내 네비게이션, 회원가입 후 리다이렉트
- **예방 스펙**: "[클라이언트] 다단계 플로우(주문, 온보딩 등)의 각 단계는 고유한 라우트를 가지며, 완료 후 히스토리 스택을 초기화(replace 방식)하여 뒤로가기로 완료 단계에 재진입하지 못하도록 한다."
- **사례**: [Ivory #1120 Unknown looping issue], [Thrll #987 Redirecting Issue]

---

### 11. UI/UX 일관성

---

### Pattern: UI 컴포넌트 불일치 및 레이아웃 오류
- **빈도**: 8회 (높음)
- **증상**: 패딩/마진 오류. 아이콘이 앱 전반의 디자인 시스템과 다름. 네비게이션 바가 페이지마다 다르게 표시됨. 프로필 이미지 미설정 시 UI 깨짐. 반응형 디자인 오류.
- **근본 원인**: 디자인 시스템(공통 컴포넌트) 미사용. Figma 스펙 기준 구현 미흡. 엣지 케이스(이미지 없음, 빈 상태) 처리 누락.
- **영향 화면**: 전체 앱 UI, 네비게이션 바, 프로필, 아이콘
- **예방 스펙**: "[클라이언트] 공통 UI 요소(아이콘, 버튼, 입력 필드, 네비게이션 바)는 디자인 시스템 컴포넌트를 재사용하고, 신규 화면 구현 시 Figma 스펙 1:1 매칭을 QA 기준으로 삼는다. 이미지/데이터 없음 상태의 placeholder UI를 모든 화면에 반드시 구현한다."
- **사례**: [Bright Pixel #533 Responsive Design], [Roadmap #1402 Parking lot UI issue], [K Talk #441 UI Issue in Class Detail Page], [Design Flow #2944 Inconsistent Navbar], [Thrll #1130 UI Issue], [Thrll #1195 UI Issue], [Roadmap #1465 parking lot detail page design], [K Talk #325 Need to change text format]

---

### Pattern: 텍스트/카피라이팅 오류
- **빈도**: 6회 (중간)
- **증상**: 버튼 텍스트가 잘못됨(예: 'Contact Us' → 'Enroll Now'). 비밀번호 확인 필드 레이블 오류. 클래스 일정 제목 형식 변경 필요. 텍스트 대소문자 오류. 에러 메시지 문구 부정확.
- **근본 원인**: 기획 변경 사항이 개발에 미반영. 텍스트 하드코딩 및 중앙화 미흡.
- **영향 화면**: 버튼, 레이블, 에러 메시지, 제목 전반
- **예방 스펙**: "[클라이언트] 사용자에게 표시되는 모든 텍스트는 i18n 또는 중앙화된 상수 파일로 관리하여 일괄 수정이 가능하도록 한다. QA 단계에서 Figma 스펙과 텍스트 일치 여부를 체크리스트로 검증한다."
- **사례**: [K Talk #579 All of contact us button], [Playbook #208 Confirm Password], [K Talk #341 Change Title of Class], [#966 text capitalization], [#1309 Error message is wrong], [#1310 error message is not correct]

---

### 12. 데이터 동기화

---

### Pattern: 데이터 생성 후 목록 미반영
- **빈도**: 3회 (중간)
- **증상**: 공지 생성 성공 메시지 후 목록에 미표시. 리드 생성 API가 성공 반환하지만 실제 생성 안 됨. 클래스 포럼 생성 후 미표시.
- **근본 원인**: 생성 API 성공 응답과 실제 DB 저장 불일치. 생성 후 목록 재조회 로직 누락.
- **영향 화면**: 공지 목록, 리드 목록, 클래스 포럼 목록
- **예방 스펙**: "[서버] 생성 API는 DB 저장 완료 후에만 성공 응답을 반환하고, 트랜잭션으로 원자성을 보장한다. [클라이언트] 생성 성공 응답 수신 후 즉시 해당 목록을 재조회하거나 낙관적 업데이트로 신규 항목을 추가한다."
- **사례**: [Entermong #186 notice creation], [Onui CRM #1424 lead create api has some problem], [K Talk #Class forum is not working]

---

### Pattern: 잘못된 데이터 표시 (Stale/Incorrect Data)
- **빈도**: 3회 (중간)
- **증상**: 태스크 클릭 시 이전 태스크의 잘못된 정보가 순간 표시됨. 리드 자녀 학년이 고객 정보 변경 시 자동으로 따라 변경됨. 진단 제출 중복 처리됨.
- **근본 원인**: 이전 데이터 캐시 미초기화. 서로 다른 엔티티 간 의도치 않은 참조 관계. 중복 제출 방지 로직 누락.
- **영향 화면**: 태스크 상세, 리드 상세, 진단 제출
- **예방 스펙**: "[클라이언트] 상세 페이지 진입 시 이전 데이터를 초기화(null/loading 상태)한 후 API를 호출하여 최신 데이터를 표시한다. [서버] 관계가 없는 엔티티 간 데이터 자동 동기화는 명시적 연결을 통해서만 허용한다. 중복 제출 방지를 위해 제출 버튼은 요청 중 비활성화하고 서버 측 idempotency key를 사용한다."
- **사례**: [Onui CRM #Clicking task briefly shows incorrect info], [Playbook #624 Lead child grade follows customer child grade], [#856 Handle Duplicate Learning Diagnosis Submission]

---

### 13. 성능

---

### Pattern: 프리뷰/미리보기 미동작
- **빈도**: 2회 (낮음)
- **증상**: 디자인 전체 프리뷰 기능이 작동하지 않음. 일반 프리뷰 기능이 작동하지 않음.
- **근본 원인**: 프리뷰 서비스에서 관계 데이터 미로드 시 폴백 처리 누락. 모듈 의존성 등록 누락.
- **영향 화면**: 디자인 프리뷰, 콘텐츠 프리뷰
- **예방 스펙**: "[서버] 프리뷰 생성 서비스는 관계 데이터 로드 실패 시 직접 조회(fallback query)를 수행하도록 구현한다. 관련 Repository는 모듈에 반드시 등록한다. [클라이언트] 프리뷰 로딩 실패 시 명확한 오류 메시지를 표시한다."
- **사례**: [Roadmap #preview is not working], [Design Flow #2966 Full Preview Option Not Working]

---

## 카테고리별 요약

| 카테고리 | 패턴 수 | 총 버그 수 | 최고 빈도 패턴 |
|---|---|---|---|
| 푸시 알림 | 4 | 18건 | 알림 뱃지/카운트 오류 (6회) |
| 결제/인앱구매 | 4 | 15건 | 결제 완료 후 UI 미반영 (7회) |
| UI/UX 일관성 | 2 | 14건 | UI 컴포넌트 불일치 (8회) |
| 리스트/페이지네이션 | 3 | 12건 | 필터 자동 초기화 (5회) |
| 인증/로그인 | 2 | 8건 | 로그인/회원가입 동작 불가 (5회) |
| 이미지/파일 업로드 | 3 | 7건 | 프로필 이미지 변경 실패 (3회) |
| 관리자 대시보드 | 2 | 7건 | 어드민 필터/통계 오류 (4회) |
| 실시간 데이터/채팅 | 2 | 6건 | 채팅 UI 오류 (4회) |
| 검색/필터 | 1 | 3건 | 검색 기능 미동작 (3회) |
| 데이터 동기화 | 2 | 6건 | 데이터 생성 후 목록 미반영 (3회) |
| 폼/입력 | 2 | 6건 | 폼 유효성 검사 미동작 (4회) |
| 네비게이션/라우팅 | 1 | 2건 | 딥링크/라우팅 오류 (2회) |
| 성능 | 1 | 2건 | 프리뷰 미동작 (2회) |
