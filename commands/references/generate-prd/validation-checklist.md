# QA Validation Rules (Machine Verification)

> All rules use **only** counting/existence checks — no subjective judgment.
> "Needs more depth", "could be better" — these judgments are PROHIBITED.

---

## Rule 1: Route Count Match

### Verification Method
1. Count user app routes (N) + admin routes (M) from `screen-inventory.md`
2. Count routes defined in PRD Section 3
3. Count admin pages defined in PRD Section 4
4. Verify `screen-inventory routes ≤ PRD routes`

### PASS Condition
- ALL routes from `screen-inventory.md` exist in PRD
- PRD having additional routes is allowed (agent-enriched)

### FAIL Condition
- Any route from `screen-inventory.md` missing from PRD

### Evidence Format
```
User app: screen-inventory {N} → PRD {M} (missing: [route1, route2])
Admin: screen-inventory {N} → PRD {M} (missing: [route1])
```

---

## Rule 2: Mandatory 8 Items

### Verification Method
Check each route in Section 3 for presence of these 8 item sections:

| # | Item | Keyword Search |
|---|------|----------------|
| 1 | Component Detail | "Component", "UI element", state definitions |
| 2 | Input Validation Rules | "Validation", "Field", "Type", "Required" table |
| 3 | State Screens | "Loading", "Empty", "Error" state definitions |
| 4 | Interactions | "Interaction", "Click", "Hover", "Drag" |
| 5 | Navigation | "Navigation", "Back", reachable screens |
| 6 | Data Display | "Data", "Sort", "Pagination" |
| 7 | Error Handling | "Error", "Network", "Server", "Permission" |
| 8 | Edge Cases | "Edge", "Zero", "Maximum", "Concurrent", "Offline" |

### PASS Condition
- ALL routes in Section 3 have all 8 items present
- "Input Validation: N/A" for inputless routes counts as PASS

### FAIL Condition
- Any route missing any item

### Evidence Format
```
Total {N} routes verified
FAIL: [route] — missing: [item1, item2]
FAIL: [route] — missing: [item3]
```

---

## Rule 3: Admin 1:1 Mapping

### Verification Method
1. Identify **CRUD-capable entities** from Section 3 (Create/Update/Delete operations by users)
2. Verify each entity has a corresponding admin page in Section 4

### PASS Condition
- All CRUD entities from Section 3 have corresponding admin pages in Section 4

### FAIL Condition
- CRUD entity exists without admin page

### Evidence Format
```
CRUD entities identified: {N}
Mapped: {M}
Missing: [entity] — Section 3 location: [route], no admin page in Section 4
```

---

## Rule 4: Admin Standard Features

### Verification Method
Check each admin page in Section 4 for required items from `admin-standards.md`:

**List Page**: Search, Filters, Column Sorting, Checkbox Selection, Bulk Actions, Pagination
**Table UI**: Loading State, Empty State, Error State, Search No Results, Action Column
**Detail/Edit**: Detail Drawer/Modal, Edit Form, Delete Confirmation
**Data Export**: CSV/Excel Download, Date Range Selection
**Common UI/UX**: Toast Notifications, Breadcrumb, Create Modal/Drawer

### PASS Condition
- All admin pages include required standard features (explicitly or via top-level declaration)
- "Admin Dashboard Standard Features Applied" declaration counts as PASS

### FAIL Condition
- Admin page missing standard features with no top-level declaration

### Evidence Format
```
{N} admin pages verified
FAIL: [page] — missing: [feature1, feature2]
```

---

## Rule 5: Terminology Cross-Reference

### Verification Method
1. Extract all terms from Section 1 Terminology tables
2. Check each term's usage in Section 3 + Section 4 body text
3. Find domain terms in Section 3 + Section 4 not defined in Section 1

### PASS Condition
- All glossary terms used in body text at least once
- All domain terms in body text defined in glossary

### FAIL Condition
- **Glossary-only term**: defined but never used in body (unnecessary)
- **Body-only term**: domain term used but not in glossary (missing definition)

### Evidence Format
```
Glossary terms: {N}
Unused in body: [term1, term2]
Body-only terms: [term3, term4]
```

### Note
- Generic IT terms (API, DB, URL, etc.) are excluded
- Only domain-specific terms are checked

---

## Rule 6: TBD Zero

### Verification Method
Search entire PRD for patterns:
- `TBD`
- `undecided`
- `to be determined`
- `to be decided`
- `confirmation needed` (exclude Additional Questions section)
- `[?]`

### PASS Condition
- Zero matches in PRD body
- `[💡 Recommended]` items are NOT TBD (PASS)
- `[📌 Adopted]` items are NOT TBD (PASS)
- Matches inside Additional Questions section are excluded

### FAIL Condition
- Any pattern found (1+ matches)

### Evidence Format
```
TBD pattern search: {N} matches
Locations:
- [section/route]: "[found text]"
```

---

## Rule 7: Route Coverage

### Verification Method
1. Extract all routes from Section 3 Page Map tables
2. Extract all routes from Section 3 Feature List by Route
3. Bidirectional comparison

### PASS Condition
- All Page Map routes exist in Feature List
- All Feature List routes exist in Page Map

### FAIL Condition
- Page Map route not in Feature List
- Feature List route not in Page Map

### Evidence Format
```
Page Map routes: {N}
Feature List routes: {M}
Page Map only: [route1, route2]
Feature List only: [route3]
```

---

## Rule 8: Numeric Consistency

### Verification Method
1. Extract all `number+unit` patterns from PRD (e.g., 10MB, 30 days, 5 min, 100 items)
2. Check if same concept has different values

### PASS Condition
- All numeric values for same concept are consistent throughout document

### FAIL Condition
- Same concept with different values (e.g., "file size limit 10MB" vs "max 20MB")

### Evidence Format
```
Numeric patterns extracted: {N}
Inconsistency found:
- [concept]: Section [X] "[value1]" vs Section [Y] "[value2]"
```

---

## Rule 9: Tech Stack Completeness

### Verification Method
1. Section 5 Technologies table is not empty
2. All services from Section 2's 3rd Party API List appear in Section 5 Third-Party Integrations
3. Key Decisions has at least 1 entry
4. Environment Variables has at least 1 entry

### PASS Condition
- Technologies table has minimum 5 layers
- All 3rd party services present in both sections
- Key Decisions ≥ 1
- Environment Variables ≥ 1

### FAIL Condition
- Technologies empty or < 5 layers
- 3rd party service missing from Section 5
- Key Decisions or Environment Variables empty

### Evidence Format
```
Technologies: {N} layers
3rd Party match: Section 2 has {N}, Section 5 has {M}
Missing: [service]
Key Decisions: {N}
Environment Variables: {N}
```

---

## Rule 10: Entity Coverage

### Verification Method
1. Identify CRUD-capable entities from Section 3/4 (create/update/delete actions)
2. Verify each appears in Section 6 Entity List/Full Schema

### PASS Condition
- All CRUD entities from Section 3/4 exist in Section 6

### FAIL Condition
- CRUD entity in Section 3/4 but missing from Section 6

### Evidence Format
```
CRUD entities: {N} (Section 3: {A}, Section 4: {B})
Section 6 entities: {M}
Missing: [entity] — Section [3/4] [route] has [create/update/delete] but not in Section 6
```

---

## Rule 11: Permission Completeness

### Verification Method
1. Identify edit/delete features in Section 3/4
2. Verify corresponding Resource × Action exists in Section 7 Permission Matrix
3. Check for empty cells (`?` or blank)

### PASS Condition
- All edit/delete features have corresponding Permission Matrix entries
- No empty cells in matrix

### FAIL Condition
- Feature exists without Permission Matrix entry
- Matrix contains `?` or empty cells

### Evidence Format
```
Edit/delete features: {N}
Permission Matrix mapped: {M}
Missing: [Resource] × [Action] — Section [3/4] [route] exists but not in Section 7
Empty cells: [Resource] × [Role] — value not specified
```

---

## Rule 12: Client Requirement Tracking

### Verification Method
1. Extract explicit requirements from `parsed-input.md`
   - Functional requirements (specific features, workflows)
   - Non-functional requirements (responsive, i18n, security, performance)
2. Verify each requirement has corresponding spec in PRD

### PASS Condition
- All explicit requirements exist as concrete specs in PRD
- OR registered as unresolved items in Section 8 (Open Questions)

### FAIL Condition
- Explicit requirement missing from both PRD and Open Questions (silently dropped)

### Evidence Format
```
Client requirements: {N}
In PRD: {M}
In Open Questions: {K}
Missing: [requirement] — stated in parsed-input but absent from PRD and Open Questions
```

---

## Rule 13: Real Scenario Existence (NEW)

### Verification Method
1. Identify all modules in Section 2
2. Check each module has at least 1 Real Scenario — Success block
3. Check each module has at least 1 Real Scenario — Failure block

### PASS Condition
- Every module has both success and failure Real Scenarios
- Scenarios include: named user, specific action, result

### FAIL Condition
- Module without Real Scenario — Success
- Module without Real Scenario — Failure
- Scenario too vague (no specific user, no concrete data)

### Evidence Format
```
Modules: {N}
With both scenarios: {M}
Missing: [module] — missing [Success/Failure] Real Scenario
```

---

## Rule 14: Full Schema Completeness (NEW)

### Verification Method
1. Extract all entities from Section 6 Entity Relationships / Entity List
2. Check each entity has a Full Schema table (Column/Type/Constraints)
3. Verify minimum 5 columns per entity (excluding timestamps)
4. Verify PK and FK columns are present

### PASS Condition
- Every entity in Entity List has a corresponding Full Schema table
- Each table has ≥ 5 non-timestamp columns
- PK column exists with generation strategy
- FK columns use `(FK → entity.column)` notation

### FAIL Condition
- Entity in list but no Full Schema table
- Schema table with < 5 non-timestamp columns
- Missing PK or FK notation

### Evidence Format
```
Entities in list: {N}
Full Schema tables: {M}
Missing schema: [entity] — in Entity List but no Full Schema table
Insufficient columns: [entity] — {K} columns (minimum 5 required)
Missing PK/FK: [entity] — [PK/FK] not properly defined
```
