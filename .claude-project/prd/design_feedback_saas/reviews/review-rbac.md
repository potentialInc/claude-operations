## Cross-Review Feedback (스코프: 권한 완전성)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 불일치 | Section 3 `/projects/:id` ProjectHeader 소유권 규칙 vs Section 7 7.3.6 | Section 3은 "삭제는 Owner만 가능 (soft delete)"라고 명시하지만, Section 7 Project Delete(soft) 행에서 Admin=✅, Owner=✅로 Admin도 삭제 가능하도록 정의되어 있음. 두 문서 중 하나가 잘못됨. | 정책 결정 후 통일 필요: Admin이 삭제 가능하다면 Section 3 소유권 규칙 수정; Owner만 가능하다면 Section 7에서 Admin=❌로 수정. |
| 2 | 불일치 | Section 3 `/projects/:id/screens` ScreenCard 소유권 규칙 vs Section 7 7.3.8 | Section 3은 "화면 삭제는 Owner, Admin만 가능"이라고 명시하지만, Section 7 Screen Delete(soft) 행에서 Member=✅own으로 Member도 본인 화면 삭제 가능하도록 정의되어 있음. | 정책 결정 후 통일 필요: Member의 own 화면 삭제를 허용한다면 Section 3 소유권 규칙 수정; 불허한다면 Section 7에서 Member=❌로 수정. |
| 3 | 불일치 | Section 3 `/settings/organization` DeleteOrganizationDialog 소유권 규칙 vs Section 7 7.3.2 | Section 3은 "Owner만 조직 정보 수정/삭제 가능"이라고 명시하고, OrganizationForm에서 Owner가 "조직 삭제" 버튼을 사용할 수 있도록 UI가 설계되어 있음. 그러나 Section 7 Organization Delete(soft) 행에서 Owner=❌, SuperAdmin만=✅로 정의되어 있어 Owner의 조직 삭제 경로가 Permission Matrix에서 완전히 차단됨. | 두 문서 중 하나가 잘못됨. Owner의 자기 조직 삭제 허용 여부를 정책적으로 결정한 후, Section 7의 Organization Delete(soft) Owner 셀을 ✅로 수정하거나 Section 3에서 해당 기능을 제거해야 함. |
| 4 | 누락 | Section 7 7.3.5 User 섹션 | Section 3 `/settings/profile` ChangePasswordSection에서 모든 인증 사용자가 비밀번호 변경(Update) 기능을 사용할 수 있지만, Section 7에는 `User \| Update (password)` 또는 그에 해당하는 Action × Role 행이 없음. 현재 "Update (self profile)" 행으로 묵시적으로 포함될 수 있으나 Password Reset(other)만 별도 행으로 존재하고, 인증된 사용자의 비밀번호 변경(self)은 명시적으로 정의되지 않음. | Section 7 7.3.5에 `User \| Update (password, self)` 행을 추가하고 Client~Owner=✅own, Guest=❌, SuperAdmin=✅로 명시. |
| 5 | 누락 | Section 7 7.3.13 Attachment 섹션 | Section 7에 Attachment Update 행이 없음. Upload/Read/Delete 세 가지 Action만 정의되어 있음. 첨부파일 수정(예: 메타데이터 변경) 기능이 Section 3/4에서 명시적으로 언급되지 않으나, 완전성 차원에서 Update Action의 존재 여부가 Matrix에 명확하지 않음. | Attachment Update가 기능상 불필요하다면 "Update 없음 — 의도된 설계"라는 주석을 Section 7에 추가하여 누락이 아님을 명시. |
| 6 | 누락 | Section 7 7.3.2 Organization 섹션 | Section 3 `/settings/organization` OrganizationForm에서 Owner가 조직 로고(logo) 업로드/변경 기능을 사용할 수 있음. Section 7 Organization Update 행은 "(name/description)"으로만 한정되어 있어 logo 필드 업데이트 권한이 Matrix에 포함되지 않음. | Section 7 Organization Update 행의 괄호를 "(name/description/logo)"로 확장하거나, `Organization \| Update (logo)` 행을 별도로 추가. |
