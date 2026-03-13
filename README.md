# claude-operations

Claude Code operations skills - PRD generation, SOP creation, presentations, deployment, workflow automation, and full-stack QA.

## Overview

This repo contains operations-focused commands and skills for Claude Code. Add it as a submodule to projects that need document generation, deployment workflows, project management, or automated QA capabilities.

## Installation

```bash
cd your-project/.claude
git submodule add https://github.com/potentialInc/claude-operations.git operations
```

## Available Commands

### Document Generation

| Command | Description | Usage |
|---------|-------------|-------|
| `generate-prd` | Generate comprehensive PRD from client input | `/generate-prd <input-file>` |
| `generate-korean-prd` | Generate Korean PRD with company watermark | `/generate-korean-prd <input-file>` |
| `pdf-to-prd` | Convert PRD PDF to structured markdown | `/pdf-to-prd <pdf-file>` |
| `generate-ppt` | Generate HTML presentations with branding | `/generate-ppt <topic>` |
| `generate-sop` | Generate Standard Operating Procedure | `/generate-sop <process>` |
| `generate-invoice` | Generate invoice document | `/generate-invoice` |
| `update-prd` | Update PRD with client feedback | `/update-prd <feedback-file>` |

### Project Management

| Command | Description | Usage |
|---------|-------------|-------|
| `generate-random-project` | Generate random project specs for training | `/generate-random-project` |
| `review-command` | Review command file compatibility | `/review-command <command-file>` |

## Available Agents

| Agent | Description | Invocation |
|-------|-------------|------------|
| `prd-manager` | PRD lifecycle dashboard, question tracking, Safety Gate guardian, change history audit | "PRD status" or "Review these changes" |

## Available Skills

### PRD Skills

| Skill | Description | Enforcement |
|-------|-------------|-------------|
| `input-classifier` | Auto-classifies input as Q&A, change request, or mixed before PRD modification | `block` (auto-runs on PRD edit) |

### Workflow Skills

| Skill | Description | Triggers |
|-------|-------------|----------|
| `fullstack` | Project iteration and pipeline management | fullstack, pipeline, iteration manager |
| `git-workflow` | PR creation and dev branch workflow | create pr, push to dev |
| `deployment` | Production/staging deployment patterns | deploy, release, production |

### QA Skills (15 individual + 2 orchestrators)

Framework-agnostic full-stack QA auditing. Works with any frontend (React, Vue, Angular, Svelte) and backend (NestJS, Express, Spring Boot, Django, Laravel).

#### Individual QA Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| `qa-inputs` | Entity → DTO → Zod → Form UI consistency | `/qa-inputs [module]` |
| `qa-crud` | CRUD endpoint completeness, error handling, frontend coverage | `/qa-crud [module]` |
| `qa-buttons` | Button handlers, routes, loading states, accessibility | `/qa-buttons [module]` |
| `qa-auth` | Role guards, route protection, permission gaps, token handling | `/qa-auth [module]` |
| `qa-api-sync` | Frontend API calls vs backend endpoints sync | `/qa-api-sync [module]` |
| `qa-db-integrity` | Entity vs migrations, FK relations, index coverage | `/qa-db-integrity [module]` |
| `qa-layout` | Page layout consistency — titles, panels, spacing, borders | `/qa-layout [module]` |
| `qa-list` | List/table pages — sorting, pagination, filters, empty states | `/qa-list [module]` |
| `qa-screen` | Playwright screen-level QA — console errors, a11y, CLS | `/qa-screen [module]` |
| `qa-test-gen` | Generate Playwright E2E tests for form validation | `/qa-test-gen [module]` |
| `qa-a11y` | WCAG 2.2 accessibility — semantic HTML, ARIA, keyboard nav | `/qa-a11y [page-path]` |
| `qa-back-nav` | Back navigation — history stack, redirects, scroll restore | `/qa-back-nav [page-path]` |
| `qa-modal` | Modal/drawer — focus trap, scroll lock, ESC, form handling | `/qa-modal [page-path]` |
| `qa-performance` | Performance — bundle size, lazy loading, Core Web Vitals | `/qa-performance [page-path]` |
| `qa-states` | Loading/error/empty states, optimistic updates, race conditions | `/qa-states [page-path]` |

#### QA Orchestrators

| Skill | Description | Usage |
|-------|-------------|-------|
| `qa-fix` | Project-wide: diagnose → auto-fix → verify across all QA skills | `/qa-fix [module] --check <types>` |
| `qa-page` | Page-level: deep analysis with cross-skill insights on a single page | `/qa-page <page-path>` |

#### QA Shared Reference

| File | Description |
|------|-------------|
| `qa-shared/reference.md` | Common scoring, framework detection, output conventions |

## Usage Examples

```bash
# Generate a PRD from client requirements
/generate-prd client-requirements.pdf

# Create a presentation
/generate-ppt "Q1 Product Roadmap"

# Update PRD with client answers or scope changes (auto-detects)
/update-prd client-feedback.md

# Create an SOP and add to Notion
/generate-sop "New Employee Onboarding"

# Deploy to staging
"Deploy the latest changes to staging"
→ Suggests: deployment skill

# Run full QA audit on a module
/qa-fix users --check crud,inputs,buttons

# Deep QA analysis on a single page
/qa-page src/pages/UserManagement.tsx

# Run specific QA check
/qa-crud products
/qa-a11y src/pages/Dashboard.tsx
```

## Structure

```
claude-operations/
├── README.md
├── agents/
│   └── prd-manager.md
├── commands/
│   ├── generate-prd.md
│   ├── generate-korean-prd.md
│   ├── generate-ppt.md
│   ├── generate-sop.md
│   ├── generate-invoice.md
│   ├── generate-random-project.md
│   ├── pdf-to-prd.md
│   ├── review-command.md
│   └── update-prd.md
└── skills/
    ├── skill-rules.json
    ├── prd/
    │   └── input-classifier.md
    ├── fullstack/
    │   ├── deployment.md
    │   └── iteration-manager.md
    ├── git-workflow/
    │   └── create-dev-pr.md
    └── qa/
        ├── qa-shared/reference.md        # shared scoring & conventions
        ├── qa-fix/skill.md               # orchestrator: project-wide
        ├── qa-page/skill.md              # orchestrator: page-level
        ├── qa-a11y/skill.md
        ├── qa-api-sync/skill.md
        ├── qa-auth/skill.md
        ├── qa-back-nav/skill.md
        ├── qa-buttons/skill.md
        ├── qa-crud/skill.md
        ├── qa-db-integrity/skill.md
        ├── qa-inputs/skill.md
        ├── qa-layout/skill.md
        ├── qa-list/skill.md
        ├── qa-modal/skill.md
        ├── qa-performance/skill.md
        ├── qa-screen/skill.md
        ├── qa-states/skill.md
        └── qa-test-gen/skill.md
```

## When to Use

Add this submodule when your project involves:
- Document generation (PRD, SOP, presentations)
- Project initialization and setup workflows
- Deployment pipelines
- Git workflow automation
- Team training with generated projects
- **Automated QA auditing** (full-stack, framework-agnostic)

## Related Repos

| Repo | Purpose |
|------|---------|
| [claude-base](https://github.com/potentialInc/claude-base) | Core shared configuration |
| [claude-marketing](https://github.com/potentialInc/claude-marketing) | Marketing and growth skills |
| [claude-react](https://github.com/potentialInc/claude-react) | React frontend skills |
| [claude-nestjs](https://github.com/potentialInc/claude-nestjs) | NestJS backend skills |
