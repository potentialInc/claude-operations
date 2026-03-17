# Admin Dashboard 표준기능 매트릭스

> 이 파일의 모든 필수(✅) 항목은 **모든 관리페이지에 자동 적용**된다.
> 클라이언트가 언급하지 않아도 포함하며, `[💡 권장적용]` 마킹 없이 디폴트로 적용한다.
> 클라이언트가 명시적으로 제외한 경우에만 빼기.

---

## 기존 표준기능 (generate-prd.md에서 계승)

### List Page Standard Features

| Feature | Description | Required |
|---------|-------------|:--------:|
| **Search** | Keyword search field (name, ID, email, etc.) | ✅ |
| **Filters** | Status / Date / Category dropdown filters | ✅ |
| **Column Sorting** | Click table header to sort ASC/DESC | ✅ |
| **Checkbox Selection** | Row checkboxes + Select All checkbox | ✅ |
| **Bulk Actions** | Bulk delete / Status change / Export for selected items | ✅ |
| **Pagination** | Page navigation + Items per page selector (10/25/50/100) | ✅ |

### Table UI Standard Features

| Feature | Description | Required |
|---------|-------------|:--------:|
| **Loading State** | Skeleton or spinner while data loads | ✅ |
| **Empty State** | Message displayed when no data exists | ✅ |
| **Error State** | API 에러 시 "데이터를 불러올 수 없습니다" + Retry 버튼 | ✅ |
| **Search No Results** | 검색 결과 0건 시 "검색 결과 없음" + 필터 초기화 링크 | ✅ |
| **Action Column** | Edit / Delete / View Detail buttons per row | ✅ |

### Detail/Edit Standard Features

| Feature | Description | Required |
|---------|-------------|:--------:|
| **Detail Drawer/Modal** | Click row to open detail panel | ✅ |
| **Edit Form** | Switch to edit mode within detail view | ✅ |
| **Delete Confirmation** | Confirmation dialog before deletion | ✅ |
| **Audit Log** | Track who/when/what was modified | Optional |

### Data Export Standard Features

| Feature | Description | Required |
|---------|-------------|:--------:|
| **CSV/Excel Download** | Export current filtered/searched results | ✅ |
| **Date Range Selection** | Period filter for export | ✅ |

### Common UI/UX Standard Features

| Feature | Description | Required |
|---------|-------------|:--------:|
| **Toast Notifications** | Success/Error feedback messages | ✅ |
| **Breadcrumb** | Current location navigation | ✅ |
| **Create Modal/Drawer** | Form for adding new items | ✅ |

### Dashboard Home Standard Features

| Feature | Description | Required |
|---------|-------------|:--------:|
| **Statistics Cards** | Key metrics with 전후비교% | ✅ |
| **Period Filter** | Today / Last 7 days / Last 30 days / Custom date range | ✅ |
| **Charts** | Trend visualization (line/bar charts) | ✅ |
| **Recent Activity** | Recently created/modified items list | ✅ |

---

## v2 추가 표준

| # | Feature | Description | Required |
|---|---------|-------------|:--------:|
| 1 | **Bulk Action 확인** | 벌크 삭제/변경 시 "N건을 삭제하시겠습니까?" 확인 다이얼로그 | ✅ |
| 2 | **Soft Delete 기본** | 삭제 시 soft delete (is_deleted 플래그), 복구 가능 | ✅ |
| 3 | **Date/Time 표준** | 목록: YYYY-MM-DD HH:mm, 상대시간 병기 (3시간 전), timezone: 서버 기준 | ✅ |
| 4 | **RBAC** | 어드민 권한 레벨별 메뉴/액션 노출 차이 | 권장 |
| 5 | **Responsive** | 1024px 이하: 테이블 가로스크롤, 768px 이하: 카드뷰 전환 | 권장 |
| 6 | **Concurrent Edit** | 동시 수정 시 optimistic lock + "다른 사용자가 수정함" 알림 | 권장 |
| 7 | **Column Customization** | 사용자별 보이는 컬럼 선택/순서 변경 | 선택 |
| 8 | **Keyboard Shortcuts** | Esc: 모달 닫기, Enter: 확인, 화살표: 테이블 네비게이션 | 선택 |

---

## admin-writer 에이전트 적용 규칙

1. 위 표준기능 중 **✅ Required** 항목은 모든 관리페이지에 자동 포함
2. **권장** 항목도 디폴트 포함하되, 프로젝트 특성에 따라 제외 가능
3. **선택** 항목은 클라이언트 요청 시에만 포함
4. `[💡 권장적용]` 마킹은 사용하지 않음 (표준이므로)
5. 각 관리페이지에 적용된 표준기능을 명시적으로 표시할 필요 없음 (모든 페이지에 동일하게 적용)
6. 예외적으로 제외된 항목이 있으면 사유를 명시

## 관리페이지 작성 시 필수 포함 항목

각 관리페이지마다 다음을 반드시 정의:

1. **Top Area**: Search, Filters, Create button, Bulk Action dropdown
2. **Table Component**: Checkbox column + Data columns (with type, sortable) + Action column
3. **Creation Modal**: 입력필드별 검증규칙 테이블
4. **Detail Drawer/Modal**: 표시 필드 전체 + 가능한 액션
5. **Delete Flow**: Confirmation dialog + Soft delete
6. **Export**: CSV/Excel 다운로드 옵션
