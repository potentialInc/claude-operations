# claude-operations

Claude Code operations skills - PRD generation, SOP creation, presentations, deployment, and workflow automation.

## Overview

This repo contains operations-focused commands and skills for Claude Code. Add it as a submodule to projects that need document generation, deployment workflows, or project management capabilities.

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

### QA & Testing

| Command | Description | Usage |
|---------|-------------|-------|
| `qa-input-fields` | Analyze input fields and generate QA test report | `/qa-input-fields <file-path>` |
| `qa-back-navigation` | Analyze back navigation behavior and generate QA report | `/qa-back-navigation <file-path>` |
| `qa-loading-error-empty` | Analyze loading/error/empty states and generate QA report | `/qa-loading-error-empty <file-path>` |
| `qa-modal-drawer` | Analyze modal/drawer/dialog behavior and generate QA report | `/qa-modal-drawer <file-path>` |
| `qa-table-list` | Analyze table/list data display and generate QA report | `/qa-table-list <file-path>` |
| `qa-permission-role` | Analyze permission/role-based access and generate QA report | `/qa-permission-role <file-path>` |

### Project Management

| Command | Description | Usage |
|---------|-------------|-------|
| `generate-random-project` | Generate random project specs for training | `/generate-random-project` |
| `review-command` | Review command file compatibility | `/review-command <command-file>` |

## Available Skills

### Workflow Skills

| Skill | Description | Triggers |
|-------|-------------|----------|
| `fullstack` | Project iteration and pipeline management | fullstack, pipeline, iteration manager |
| `git-workflow` | PR creation and dev branch workflow | create pr, push to dev |
| `deployment` | Production/staging deployment patterns | deploy, release, production |

## Usage Examples

```bash
# Generate a PRD from client requirements
/generate-prd client-requirements.pdf

# Create a presentation
/generate-ppt "Q1 Product Roadmap"

# Create an SOP and add to Notion
/generate-sop "New Employee Onboarding"

# Deploy to staging
"Deploy the latest changes to staging"
→ Suggests: deployment skill
```

## Structure

```
claude-operations/
├── README.md
├── commands/
│   ├── generate-prd.md
│   ├── generate-korean-prd.md
│   ├── generate-ppt.md
│   ├── generate-sop.md
│   ├── generate-random-project.md
│   ├── pdf-to-prd.md
│   ├── qa-back-navigation.md
│   ├── qa-input-fields.md
│   ├── qa-loading-error-empty.md
│   ├── qa-modal-drawer.md
│   ├── qa-permission-role.md
│   ├── qa-table-list.md
│   └── review-command.md
└── skills/
    ├── skill-rules.json
    ├── fullstack/
    │   ├── deployment.md
    │   └── iteration-manager.md
    └── git-workflow/
        └── create-dev-pr.md
```

## When to Use

Add this submodule when your project involves:
- Document generation (PRD, SOP, presentations)
- Project initialization and setup workflows
- Deployment pipelines
- Git workflow automation
- Team training with generated projects

## Related Repos

| Repo | Purpose |
|------|---------|
| [claude-base](https://github.com/potentialInc/claude-base) | Core shared configuration |
| [claude-marketing](https://github.com/potentialInc/claude-marketing) | Marketing and growth skills |
| [claude-react](https://github.com/potentialInc/claude-react) | React frontend skills |
| [claude-nestjs](https://github.com/potentialInc/claude-nestjs) | NestJS backend skills |
