## 7. Permission Matrix

---

### 7.1 Role Hierarchy

```
Guest < Client < Member < Admin < Owner < SuperAdmin
```

| 역할 | 범위 | 설명 |
|------|------|------|
| Guest | 플랫폼 | 미인증 사용자. 공개 페이지(로그인·회원가입·비밀번호 찾기)만 접근 가능 |
| Client | 프로젝트 | 자신에게 배정된 특정 프로젝트에만 접근. 피드백 작성 목적 외부 협력자 |
| Member | 조직 | 조직 소속 내부 구성원. 화면·버전 관리 및 코멘트 상태 처리 담당 |
| Admin | 조직 | 프로젝트·팀원·피드백 전반 관리. 조직/빌링 설정은 불가 |
| Owner | 조직 | 조직 최고 관리자. 빌링·조직 설정·멤버 역할 전체 관리 |
| SuperAdmin | 플랫폼 | 플랫폼 운영자. 모든 조직 데이터 조회·상태 관리 가능. 일반 조직 라우트 미접근 |

**상속 규칙:**
- 상위 역할은 하위 역할의 권한을 모두 포함합니다 (단, SuperAdmin 제외 — 조직 경계를 넘어 동작하므로 별도 계층으로 취급).
- Client는 Member보다 낮은 계층이나, **프로젝트 내 피드백 작성 권한은 Client에게만 부여**되는 Client 전용 권한이 존재합니다.
- SuperAdmin은 일반 조직 라우트(`/dashboard`, `/projects`, `/settings`)에 접근하지 않고 `/admin/*` 라우트만 사용합니다.
- 모든 역할은 **동일 조직(Organization) 내 데이터에만** 접근할 수 있으며, 다른 조직의 리소스는 SuperAdmin 경로를 통해서만 접근됩니다 (멀티테넌시 경계).

---

### 7.2 Ownership Rules

**Own Resource 정의:**
```
resource.created_by === currentUser.id
  또는
resource.user_id === currentUser.id
```

- **Comment / Reply**: `comment.created_by === currentUser.id` 인 경우에만 수정·삭제 가능 (✅own).
- **Pin**: `pin.created_by === currentUser.id` 인 경우에만 수정·삭제 가능 (✅own).
- **Version (Screenshot)**: `version.uploaded_by === currentUser.id` 또는 Admin 이상인 경우 삭제 가능.
- **Screen**: Member 이상이 생성하며, 생성자(✅own) 또는 Admin 이상이 수정·삭제 가능.
- **Profile (User)**: `user.id === currentUser.id` 인 경우에만 수정 가능.
- **Attachment**: 첨부파일은 업로드(Create)와 삭제(Delete)만 가능하며, 수정(Update)은 지원하지 않습니다. 업로드된 첨부파일의 내용을 변경하려면 삭제 후 재업로드해야 합니다.

**Admin Override 규칙:**
- Admin은 자신이 생성하지 않은 Comment·Pin·Reply도 **삭제**할 수 있습니다 (프로젝트 내 피드백 관리 권한).
- Admin은 자신이 생성하지 않은 Screen·Version도 수정·삭제할 수 있습니다.
- Owner는 Admin의 모든 override 권한을 포함하며, 추가로 조직·멤버·빌링을 관리합니다.
- SuperAdmin은 플랫폼 차원에서 Organization·User·Project의 상태(active/inactive/suspended)를 변경·삭제할 수 있습니다. 단, 개별 Comment·Pin 등 콘텐츠 레벨 데이터는 직접 수정하지 않습니다.

**프로젝트 기반 접근 규칙 (Client):**
- Client는 `project_member` 테이블에 자신의 `user_id`가 포함된 프로젝트에만 접근합니다.
- Client에게 배정되지 않은 프로젝트의 화면·코멘트·핀은 API·UI 모두 차단합니다.
- Client는 자신이 배정된 프로젝트 내에서도 **다른 Client의 코멘트를 수정·삭제할 수 없습니다**.

**조직 기반 격리 규칙:**
- Member·Admin·Owner는 자신이 소속된 Organization의 리소스에만 접근합니다.
- `organization_id` 가 다른 리소스에 대한 API 요청은 403으로 거부합니다.

---

### 7.3 Action × Role Matrix

> 표기 범례
> - ✅ = 무조건 허용 (역할 충족 시)
> - ✅own = 본인 리소스(owned resource)에 한해 허용
> - ✅assigned = 배정된 프로젝트에 한해 허용
> - ❌ = 금지

#### 7.3.1 Authentication (인증)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Session | Login | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Session | Logout | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Session | Register (self) | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Session | Password Reset (self) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Session | Password Reset (other) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Session | Accept Invite | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |

#### 7.3.2 Organization (조직)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Organization | Create | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Organization | Read (own org) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Organization | Read (all orgs) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Organization | Update (name/description/logo) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Organization | Update (status: active/inactive/suspended) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Organization | Delete (soft) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Organization | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

> Owner는 본인이 소유한 조직에 한해 소프트 삭제(Delete soft)가 가능합니다.

#### 7.3.3 Billing (빌링/결제)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Billing | Read | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Billing | Update (plan 변경) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Billing | Read (all orgs billing) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

#### 7.3.4 Membership / Team (조직 멤버십)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Membership | Invite (send email invite) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Membership | Read (list members) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Membership | Update (change role) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Membership | Remove (kick member) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Membership | Leave (self) | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ |

> Owner는 조직의 유일한 Owner인 경우 탈퇴 불가 (최소 1명의 Owner 유지 필수) [💡 권장적용]

#### 7.3.5 User / Profile (사용자 계정)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| User | Read (self profile) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| User | Read (all users list) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| User | Read (org members detail) | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| User | Update (self profile) | ❌ | ✅own | ✅own | ✅own | ✅own | ✅ |
| User | Update (password, self) | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| User | Update (account status) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| User | Delete (soft, self) | ❌ | ✅own | ✅own | ✅own | ✅own | ✅ |
| User | Delete (soft, any) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| User | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

> `/settings/profile`에서 인증된 모든 사용자(Client·Member·Admin·Owner)는 본인 비밀번호를 변경할 수 있습니다.

#### 7.3.6 Project (프로젝트)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Project | Create | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Project | Read (list, own org) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Project | Read (assigned projects only) | ❌ | ✅assigned | ❌ | ❌ | ❌ | ❌ |
| Project | Read (all orgs, platform-wide) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Project | Update (name/settings) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Project | Archive | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Project | Delete (soft) | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Project | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

#### 7.3.7 Project Membership (프로젝트 멤버 배정)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| ProjectMember | Add (invite to project) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| ProjectMember | Read (project member list) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ✅ |
| ProjectMember | Update (role in project) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| ProjectMember | Remove (from project) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |

#### 7.3.8 Screen (화면)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Screen | Create | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Screen | Read (list) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Screen | Read (detail / feedback viewer) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Screen | Update (name/settings) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |
| Screen | Delete (soft) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |

#### 7.3.9 Version / Screenshot (스크린샷 버전)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Version | Upload (create) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Version | Read (list versions) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Version | Read (version detail / compare) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Version | Update (metadata) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |
| Version | Delete (soft) | ❌ | ❌ | ✅own | ✅ | ✅ | ❌ |

#### 7.3.10 Pin (핀)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Pin | Create (place pin on image) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Pin | Read | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Pin | Update (reposition) | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |
| Pin | Delete | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

#### 7.3.11 Comment (코멘트)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Comment | Create | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Comment | Read | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Comment | Update (content) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Comment | Update (status: open→in_progress→resolved) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Comment | Delete (soft) | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

> Client는 코멘트 내용은 본인 것만 수정 가능하며, 코멘트 상태(open/in_progress/resolved) 변경은 불가합니다.
> Admin은 타인의 코멘트도 삭제 가능합니다 (Admin override).

#### 7.3.12 Reply (답글)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Reply | Create | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Reply | Read | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Reply | Update (content) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Reply | Delete (soft) | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

#### 7.3.13 Comment Attachment (파일 첨부)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Attachment | Upload (on comment) | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Attachment | Read / Download | ❌ | ✅assigned | ✅ | ✅ | ✅ | ❌ |
| Attachment | Delete | ❌ | ✅own | ✅own | ✅ | ✅ | ❌ |

> Attachment Update(수정) 행은 의도적으로 제공하지 않습니다. 첨부파일은 업로드(Create)와 삭제(Delete)만 가능하며, 내용 변경이 필요한 경우 삭제 후 재업로드해야 합니다. (참고: 7.2 Ownership Rules)

#### 7.3.14 Invitation (초대)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Invitation | Send (org-level) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Invitation | Send (project-level) | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Invitation | Read (pending list) | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Invitation | Revoke | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Invitation | Accept (via email link) | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |

#### 7.3.15 Notification (알림)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Notification | Read (own) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Notification | Mark as read (own) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |
| Notification | Delete (own) | ❌ | ✅own | ✅own | ✅own | ✅own | ❌ |

#### 7.3.16 Audit Log (감사 로그)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| AuditLog | Read (own org logs) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| AuditLog | Read (all platform logs) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| AuditLog | Export (CSV/Excel) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| AuditLog | Delete | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

> 감사 로그는 어떤 역할도 삭제할 수 없습니다 (불변 기록). [💡 권장적용]

#### 7.3.17 Platform Settings (플랫폼 설정)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| PlatformSettings | Read | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Update (general/security/email) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Update (file upload settings) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Update (billing/plan settings) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Toggle Maintenance Mode | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| PlatformSettings | Run Data Purge | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

#### 7.3.18 Statistics / Dashboard Data (통계)

| Resource | Action | Guest | Client | Member | Admin | Owner | SuperAdmin |
|----------|--------|-------|--------|--------|-------|-------|------------|
| Stats | Read (own org overview) | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |
| Stats | Read (platform-wide statistics) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

### 7.4 멀티테넌시 격리 요약

| 규칙 | 적용 대상 |
|------|-----------|
| 모든 API 요청에 `organization_id` 스코프 적용 | Member·Admin·Owner |
| 다른 org 리소스 접근 시 403 반환 | 전 역할 (SuperAdmin 제외) |
| Client는 `project_member` 테이블 배정 여부로 추가 필터링 | Client |
| SuperAdmin은 org 경계를 넘어 접근 가능하나, `/admin/*` 라우트에서만 동작 | SuperAdmin |
| Soft delete된 리소스는 해당 org 내에서도 일반 조회 API에서 제외 (필터로만 복원 조회 가능) | 전 역할 |
