# Section 3 Cross-Review

**작성자:** Section 3 (User Application) 작성자
**리뷰 기준:** feature-map.md, Section 4 (Admin Dashboard), Section 7 (Permission Matrix)
**리뷰 스코프:** 기능 커버리지 + 권한 일치

---

## Cross-Review Feedback (스코프: 기능 커버리지 + 권한 일치)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 불일치 | Section 3 `/projects/:id` ProjectHeader 소유권 규칙 vs Section 7 §7.3.6 | Section 3은 프로젝트 삭제를 "Owner만 가능"으로 명시하나, Section 7 Permission Matrix에서 Project Delete는 Admin ✅, Owner ✅ 모두 허용. | Section 3의 DeleteConfirmDialog 접근 제어를 "Owner, Admin" 으로 수정하고, ProjectHeader의 "삭제" 버튼 표시 조건도 동일하게 변경 |
| 2 | 불일치 | Section 3 `/projects/:id/screens` ScreenCard 더보기 메뉴 소유권 규칙 vs Section 7 §7.3.8 | Section 3은 화면 삭제를 "Owner, Admin만" 가능으로 명시하나, Section 7에서 Screen Delete는 Member ✅own (본인 등록 화면)도 허용. | ScreenCard 더보기 메뉴에서 Member가 본인 등록 화면에 한해 삭제 버튼 표시 및 동작하도록 소유권 조건 추가 |
| 3 | 누락 | Section 3 전체 라우트 (Protected 구간) vs Section 7 §7.3.4 Membership Leave (self) | Section 7은 Client, Member, Admin이 조직을 본인 탈퇴(Leave self)할 수 있도록 허용하나, Section 3의 어떤 라우트(Settings, Profile 등)에도 "조직 탈퇴" UI 및 흐름이 존재하지 않음. | `/settings/profile` 또는 `/settings/team` 내에 "조직 탈퇴" 섹션(위험 영역)을 추가. Owner는 유일한 Owner인 경우 탈퇴 불가 조건(Section 7 §7.3.4 주석) 포함 |
| 4 | 불일치 | Section 3 `/projects/:id/screens` ScreenCard 더보기 메뉴 vs Section 7 §7.3.8 Screen Update | Section 7은 Screen Update를 Member ✅own, Admin ✅, Owner ✅로 허용하나, Section 3에는 화면 수정(이름 변경 등) UI가 명시되어 있지 않음. AddScreenModal은 생성 전용으로 기술됨. | ScreenCard 더보기 메뉴에 "화면 수정" 액션 및 EditScreenModal을 추가. Member는 본인 등록 화면에 한해 수정 가능하도록 소유권 조건 적용 |
| 5 | 누락 | Section 3 `/client/projects/:id/screens/:screenId` ClientImageViewer vs feature-map.md Client 행 | feature-map.md의 Client 기능에 "피드백 뷰어 - 핀 찍기 + 코멘트 작성"은 포함되어 있으나, Section 7 §7.3.9 Version Read (version detail / compare) = Client ✅assigned 임에도 Client 피드백 뷰어(`ClientImageViewer`)에 버전 비교(overlay / side-by-side) 기능이 없음. | Client 피드백 뷰어에도 버전 선택 드롭다운은 이미 존재하나 버전 비교 토글이 없음. Section 7 권한(Version Read compare = ✅assigned)에 맞춰 버전 비교 UI(overlay / side-by-side 토글)를 ClientImageViewer에 추가하거나, Client의 버전 비교 허용 여부를 Section 7에서 재정의 필요 |
