---
description: "Generate a dense, development-ready PRD using optimized multi-agent workflow"
argument-hint: "Path to client answer file (e.g., /path/to/client-answers.md)"
allowed-tools: Agent, Read, Write, Edit, Glob, Grep, AskUserQuestion, Bash(mkdir *)
---

# Generate PRD — Optimized Multi-Agent Orchestration

Generate a dense, development-ready PRD from client input using a streamlined 4-Phase pipeline with 5 agents.

## Core Principles

1. **Density > Length**: "Items missing = QA FAIL", not "write more"
2. **TBD Zero**: Fill unclear items with best-practice recommendations marked `[💡 Recommended]`
3. **Machine Verification**: QA uses only counting/existence rules — no subjective judgment
4. **Real Scenarios**: Every module includes concrete success/failure scenarios with names, data, and UI reactions
5. **Full Schema**: Database design at column-level detail, not just entity summaries
6. **Adaptive Complexity**: Conditional sections activate only when relevant (SaaS, real-time, file upload, billing)
7. **Source Tracking**: Recommended items retain `[📌 Adopted]` marker after approval for traceability

## Agent Configuration

| Agent | Model | Role |
|-------|-------|------|
| **parser** | sonnet | Parse input → 3 intermediate artifacts |
| **prd-writer** | opus | Write Section 0-4 (Overview + Terminology + Modules + User App + Admin) |
| **tech-writer** | opus | Write Section 5-7 (System Design + Full Schema + Permission Matrix) |
| **qa** | sonnet | Validate against 14 machine-verifiable rules |
| **support** | sonnet | Fix QA FAIL items (max 3 rounds) |

## Reference Files

All references located under `commands/references/generate-prd/`:
- `prd-template.md` — Section 0-8 output structure
- `prompt-templates.md` — Agent specs + write-mode prompts
- `depth-guide.md` — Mandatory density requirements
- `admin-standards.md` — Admin standard feature matrix
- `validation-checklist.md` — 14 machine-verifiable QA rules

---

## Phase 1: Parse

### 1.1 Intake & Validation

Read client answer file path from `$ARGUMENTS`.

**Validation checklist:**
1. File path provided?
2. File exists and readable?
3. File not empty?

**On failure:**
```
Error: [specific error message]
Usage: /generate-prd /path/to/client-answers.md
```

### 1.2 Existing PRD Check

1. Search `.claude-project/prd/` for PRD matching the app name
2. If found, ask user via AskUserQuestion:
   ```
   Existing PRD found: [filename]
   1. Reference existing PRD and generate new (use as context)
   2. Generate completely fresh
   ```
3. If referencing: Pass existing PRD path to Phase 2 agents as additional context

### 1.3 Project Folder Setup

Derive folder name from app name:
- Spaces → underscores, remove special characters
- Example: "TireBank Mall" → `TireBank_Mall`

```bash
mkdir -p .claude-project/prd/{ProjectName}/intermediate
mkdir -p .claude-project/prd/{ProjectName}/drafts
```

### 1.4 Parser Agent → 3 Intermediate Artifacts

Launch **parser** (sonnet) to read the client answer file and produce 3 structured files in `.claude-project/prd/{ProjectName}/intermediate/`:

#### parsed-input.md
Organize raw input by section:
- Basic Info (app name, type, deadline, user types, relationships)
- Design Reference (reference apps, differentiators, colors, fonts)
- Features (per-user-type features, module flows, auth, communication)
- Data (collected data, exportable data)
- Technical (3rd party, domain terms)
- Other (additional info)

#### screen-inventory.md
Complete screen inventory (QA baseline):
```markdown
## User App Routes (Total: N)
| # | Route | Page Name | User Type | Access Group | Notes |
|---|-------|-----------|-----------|-------------|-------|

## Admin Routes (Total: M)
| # | Route | Page Name | Managed Entity | Notes |
|---|-------|-----------|----------------|-------|
```

#### tbd-items.md
Unclear items list (agents will fill with `[💡 Recommended]`):
```markdown
| # | Item | Source | Reason Unclear |
|---|------|--------|----------------|
```

### 1.5 Adaptive Complexity Detection

Parser also detects and flags in `parsed-input.md` header:
```markdown
## Detected Features
- [ ] SaaS / Multi-tenancy
- [ ] File Upload / Image Processing
- [ ] Real-time / WebSocket
- [ ] Billing / Subscription
- [ ] Chat / Messaging
```

Only checked features trigger conditional sections in Phase 2.

---

## Phase 2: Write (Parallel)

Launch **prd-writer** and **tech-writer** simultaneously.

```
┌──────────────────────────────────────────────┐
│  Parallel Execution                          │
│                                              │
│  prd-writer (opus)  → Section 0-4            │
│  tech-writer (opus) → Section 5-7            │
│                                              │
└──────────────────────────────────────────────┘
```

### Agent Input Distribution

```
Both agents receive:     parsed-input.md + screen-inventory.md + tbd-items.md

prd-writer also gets:    depth-guide.md (Section 0-4)
                         prd-template.md (Section 0-4)
                         admin-standards.md
                         bug-patterns-filtered.md (if exists)

tech-writer also gets:   depth-guide.md (Section 5-7)
                         prd-template.md (Section 5-7)
                         Section 3/4 drafts (WAIT for prd-writer to finish,
                         OR read screen-inventory.md for entity extraction)
```

**Critical dependency**: tech-writer needs Section 3/4 entity information for Schema and Permission Matrix. Two approaches:
- **Option A (Preferred)**: Run prd-writer first, then tech-writer with Section 3/4 output
- **Option B (Faster)**: Run in parallel — tech-writer uses screen-inventory.md + parsed-input.md to infer entities, then QA catches mismatches

Choose Option A for accuracy, Option B when speed is critical.

### Agent Prompt Structure

> Detailed prompts: see `commands/references/generate-prd/prompt-templates.md`

Each agent prompt includes:
1. **Context**: Project background + intermediate artifact content (inline)
2. **Goal**: Write assigned Sections following template format strictly
3. **Reference**: Relevant depth-guide sections (inline)
4. **Constraints**:
   - No TBD: Fill unclear items with best-practice recommendations, mark `[💡 Recommended]`
   - No guessing beyond input file
   - Strictly follow `prd-template.md` output format
   - Include Real Scenarios for every module (prd-writer)
   - Include Full Schema with column-level detail (tech-writer)
   - Include Edge Cases for every route/admin page
5. **Output Format**: Complete Section markdown

### TBD Handling Rules

- Items not specified in input → write best-practice recommendation
- Mark: `[💡 Recommended]` next to the recommendation
- Example: `Login method: Email + Password, Social Login (Google, Apple) [💡 Recommended]`
- Phase 3.1 will back-fill `tbd-items.md` with PRD selection status and generate unresolved items summary

### Bug Pattern Integration

If `.claude-project/knowledge/bug-patterns.md` or `commands/references/generate-prd/bug-patterns-global.md` exists:
- **prd-writer**: Insert `⚠️ Known Risk` tables per route with relevant bug patterns
- Filter by project category relevance

### Output Files

- `.claude-project/prd/{ProjectName}/drafts/section-0-4.md`
- `.claude-project/prd/{ProjectName}/drafts/section-5-7.md`

---

## Phase 3: QA + Fix (Max 3 Rounds)

### 3.1 Integrate

Combine Section 0-4 and Section 5-7 into a single PRD draft:
1. Assemble Section 0-8 in order
2. Cross-reference terminology (Section 1 terms ↔ body text)
3. Generate `[💡 Recommended]` summary table
4. Unify terminology (same concept, different wording → standardize)
5. Add Additional Questions section (Section 8)
6. Add Feature Change Log
7. **Consistency audit**:
   - Numeric consistency: extract number+unit patterns → flag conflicts
   - Entity relationship verification: Section 6 FKs reference existing entities
   - Client requirement tracking: all explicit requirements from parsed-input.md present in PRD
   - Filler detection: move recommendation summaries to appendix, not body
8. **TBD Items back-fill + summary**:
   - Scan integrated PRD for all `[💡 Recommended]` markers
   - Match each to `tbd-items.md` entries, fill status:
     - `✅ PRD: {value}` — recommendation applied
     - `⏳ Deferred: {version}` — postponed to future version
     - `❌ Not in PRD` — not addressed, needs client decision
   - Append `## ⚠️ Unresolved Items — Client Decision Required` summary table listing only ❌ items with ID, Item name, Options, and Category
   - Append total counts: `✅ resolved / ⏳ deferred / ❌ unresolved`
   - Save updated `intermediate/tbd-items.md`

Save: `.claude-project/prd/{ProjectName}/drafts/prd-integrated.md`

### 3.2 QA Validation

Launch **qa** (sonnet) to validate against 14 rules.

> Detailed rules: see `commands/references/generate-prd/validation-checklist.md`

| # | Rule | FAIL Condition |
|---|------|---------------|
| 1 | Route Count Match | screen-inventory route missing from PRD |
| 2 | Mandatory 8 Items | Section 3 route missing any of 8 required items |
| 3 | Admin 1:1 Mapping | CRUD entity without corresponding admin page |
| 4 | Admin Standard Features | Admin page missing required standard features |
| 5 | Terminology Cross-Ref | Term only in glossary or only in body |
| 6 | TBD Zero | "TBD", "undecided", "to be determined" found |
| 7 | Route Coverage | Page Map ↔ Feature List mismatch |
| 8 | Numeric Consistency | Same concept, different values |
| 9 | Tech Stack Completeness | Section 5 empty or 3rd party missing |
| 10 | Entity Coverage | CRUD entity missing from Section 6 |
| 11 | Permission Completeness | Edit/delete action without Permission Matrix entry |
| 12 | Client Requirement Tracking | Explicit requirement missing from PRD and Open Questions |
| 13 | Real Scenario Existence | Module without success+failure Real Scenario |
| 14 | Full Schema Completeness | Entity in Entity List but missing from Full Schema |

### QA Output Format

```markdown
## QA Validation Results

| # | Rule | Verdict | Evidence |
|---|------|---------|----------|
| 1 | Route Count Match | PASS/FAIL | {evidence} |
...

Overall: PASS / FAIL
```

### 3.3 FAIL Handling

1. QA FAIL → **support** (sonnet) fixes only FAIL items
2. After fix → **qa** re-validates
3. **Max 3 rounds**
4. After 3 rounds still FAIL:
   ```
   ## QA Validation Failed — Manual Intervention Required

   ### Repeatedly Failing Items
   - Rule {N}: {name} — {reason}

   ### Attempted Fixes
   1. {attempt 1}
   2. {attempt 2}
   3. {attempt 3}

   ### Recommended Action
   {what needs manual fixing}
   ```

---

## Phase 4: Deliver

### 4.1 Recommended Items Bulk Confirmation

After QA PASS, present all `[💡 Recommended]` items to user:

```markdown
## Recommended Items Confirmation

The following items were not specified in the input file. Best-practice recommendations have been applied.
Approved items will be converted to `[📌 Adopted]` and included as formal specs.
Rejected items will move to Additional Questions.

| # | Item | Recommendation | Rationale | Approve? |
|---|------|---------------|-----------|----------|
| 1 | Login method | Email+PW + Social (Google, Apple) | B2C app standard | ✅/❌ |
| 2 | Password rules | 8+ chars, alphanumeric + special | OWASP guidelines | ✅/❌ |
```

Use AskUserQuestion for confirmation:
- **Approve** → `[💡 Recommended]` → `[📌 Adopted]`
- **Reject** → Move to Additional Questions
- **Modify** → Apply user's modification + `[📌 Adopted]`

### 4.2 UX Analysis (Optional)

Ask user via AskUserQuestion:
```
PRD generation complete. Include UX flow analysis?

Analysis covers:
- Navigation flow issues
- User journey optimization
- Information architecture improvements
- Common UX problem patterns

[Yes] / [No]
```

- Yes → Append UX analysis to PRD end
- No → Skip

### 4.3 Save Final File

Filename: `[AppName]_PRD_[YYMMDD].md`
- Sanitize: spaces → underscores, remove special characters
- Location: `.claude-project/prd/{ProjectName}/`

### 4.4 Cleanup

Intermediate artifacts preserved by default (for debugging/audit).
If user requests cleanup:
```bash
rm -rf .claude-project/prd/{ProjectName}/intermediate/
rm -rf .claude-project/prd/{ProjectName}/drafts/
```

### 4.5 Result Report

```markdown
## PRD Generation Complete

### Output
- File: `.claude-project/prd/{ProjectName}/[filename].md`
- Mode: [New / Reference Update]

### Summary
- Section 0: Project Overview ✅
- Section 1: Terminology ✅
- Section 2: System Modules ({N} modules, {N} Real Scenarios) ✅
- Section 3: User Application ({N} routes, 8-item density) ✅
- Section 4: Admin Dashboard ({M} pages) ✅
- Section 5: Tech Stack & System Design ✅
- Section 6: Data Model — Full Schema ({K} entities, {K} tables) ✅
- Section 7: Permission Matrix ({R} roles × {A} resources) ✅
- Section 8: Open Questions ({Q} items) ✅
- QA Validation: PASS ({N} rounds, 14 rules)
- Recommended items: {N} approved [📌 Adopted], {M} moved to questions
- Additional Questions: {N} items
- UX Suggestions: {N} items (or "Skipped")
- Bug prevention: {N} routes (or "No bug-patterns found")
- Conditional sections: {list of activated conditional sections}

### Output Structure
.claude-project/prd/{ProjectName}/
├── intermediate/              ← Intermediate artifacts
├── drafts/                    ← Agent drafts
└── {AppName}_PRD_{YYMMDD}.md  ← Final PRD

### Next Steps
1. Review the Additional Questions section
2. Send questions to client for clarification
3. Update PRD with client responses using `/generate-prd [answer-file]`
```

---

## Error Handling

| Situation | Response |
|-----------|----------|
| Agent timeout/failure | Retry once. On second failure, mark "⚠️ Generation failed", continue with rest |
| QA 3-round FAIL | Present FAIL items + manual intervention request to user |
| Input file unparseable | Error message + supported format guide, then stop |
| Bug patterns not found | Normal — generate without bug prevention sections (print warning) |
| Existing PRD name mismatch | Confirm via AskUserQuestion before proceeding |
| Token limit exceeded | Assemble sections sequentially, run consistency audit separately |

---

## Client Question Template

PMs can use this template when gathering requirements:

```
=== Basic Info ===
1. App Name:
2. App Type (Web / App / Dashboard):
3. Deadline:
4. All User Types:
5. User Roles & Permissions (what can each user do?):
6. User Relationships (1:1, 1:N, independent, etc.):

=== Design Reference ===
7. Reference App + Points to reference:
8. What makes your app unique (vs existing services):
9. Preferred Main Color:
10. Preferred Font:

=== Features ===
11. Main Features (per user type):
12. Main Feature Module Flow (Actor → Action → Result) for MVP:
13. Authentication Method:
    - Login: (ID/PW, Social login, Phone verification, etc.)
    - Signup required fields:
    - Signup optional fields:
14. Communication Features:
    - Chat: (Yes/No, type)
    - Video call: (Yes/No, provider)
    - Push notifications: (Yes/No)

=== Data ===
15. Data to Collect (forms, surveys, logs, etc.):
16. Data to Export (CSV downloads — what data?):

=== Technical ===
17. 3rd Party Integrations (SMS, payment, maps, analytics, etc.):
18. Domain Terminology (special terms needing definition):

=== Other ===
19. Additional important information:
```

---

## Reference File List

| File | Purpose |
|------|---------|
| `commands/references/generate-prd/prd-template.md` | Section 0-8 output structure |
| `commands/references/generate-prd/prompt-templates.md` | Agent specs + write-mode prompts |
| `commands/references/generate-prd/depth-guide.md` | Mandatory density requirements |
| `commands/references/generate-prd/admin-standards.md` | Admin standard feature matrix |
| `commands/references/generate-prd/validation-checklist.md` | 14 machine-verifiable QA rules |
