## Cross-Review Feedback (스코프: 엔티티 커버리지)

### 발견 항목

| # | 유형 | 위치 | 문제 | 제안 |
|---|------|------|------|------|
| 1 | 누락 | Section 3 `/auth/forgot-password`, `/auth/reset-password` → Section 6 | 비밀번호 재설정 토큰 플로우가 Section 3에 존재하지만(`GET /api/invitations/{token}/verify` 와 별개로 비밀번호 재설정 전용 토큰 발급·검증), 해당 토큰을 저장할 엔티티가 Section 6 Entity List에 없음. Invitation 엔티티는 멤버 초대 전용이므로 재사용 불가. | `PasswordResetToken` 엔티티 추가 (`id`, `user_id` FK→User, `token` UNIQUE, `expires_at`, `used_at`, `created_at`) 또는 User 엔티티에 `reset_token`, `reset_token_expires_at` 컬럼 추가를 Section 6에 명시 |
| 2 | 누락 | Section 3 `/settings/organization` → Section 6 Organization 엔티티 | 조직 설정 페이지에서 조직 로고 업로드 기능(`organizationLogo` 파일 필드)이 있으나, Section 6 Organization 엔티티의 Key Fields에 `logo_url` 또는 `logo_key` 컬럼이 정의되어 있지 않음. | Section 6 Organization 엔티티에 `logo_url` (VARCHAR 500, NULL 허용)과 `logo_key` (VARCHAR 500, NULL 허용, S3 객체 키) 컬럼 추가 |
| 3 | 누락 | Section 4 `/admin/settings` → Section 6 | 슈퍼어드민 플랫폼 설정 페이지에서 플랫폼 이름, 지원 이메일, 기본 언어, 비밀번호 정책, 세션 만료 시간, 파일 업로드 제한, 점검 모드 등의 플랫폼 전역 설정을 저장·수정하는 기능이 있으나, 이 데이터를 영속화할 `PlatformSettings` 또는 동등한 엔티티가 Section 6 Entity List에 없음. | Section 6에 `PlatformSettings` 엔티티 추가 (`id`, `key` UNIQUE, `value` JSONB 또는 TEXT, `updated_by` FK→User, `updated_at`) 형태의 Key-Value 설정 테이블, 또는 설정 항목별 컬럼을 가진 단일 레코드 테이블로 정의 |
| 4 | 누락 | Section 2 3rd Party API List → Section 5.3 Third-Party Integrations | Section 2의 3rd Party API List에는 Stripe, SendGrid, AWS S3/GCS 3개 서비스만 기재되어 있으나, Section 5.3에는 Redis Cloud(또는 AWS ElastiCache)가 Fourth-Party Integration으로 추가 기재되어 있음. Section 2에서 Redis가 누락된 상태. | Section 2의 3rd Party API List에 Redis Cloud(또는 AWS ElastiCache) 항목 추가 — 용도: Socket.IO 어댑터, 세션 캐시, Rate Limit 저장소 |
| 5 | 불일치 | Section 2 3rd Party API List ↔ Section 5.3 Third-Party Integrations | Section 2에서 파일 스토리지 대안으로 `GCS (Google Cloud Storage)`를 명시하고 있으나, Section 5.3에서는 `AWS S3 (또는 호환 스토리지)`로만 기재되어 GCS가 대안 목록에서 빠져 있음. | Section 5.3의 AWS S3 항목 설명에 `Cloudflare R2, Supabase Storage, GCS` 등 대안을 Section 2와 통일하여 명시, 또는 Section 2를 Section 5.3 기준으로 맞춰 GCS 제거 |
