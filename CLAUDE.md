# claude-operations Rules

## All Skills Must Be Written in English

Every skill file (`.md`) in this repo **must be written entirely in English** — including descriptions, check names, instructions, examples, and comments. No other languages allowed.

---

## Framework-Agnostic Requirement

All skills must work with **any framework/language combination**. Verify the following when creating or modifying a skill.

### Prohibited

- Hardcoding a specific framework (e.g., NestJS-only, TypeORM-only, React-only logic)
- Assuming a specific file structure (e.g., fixed paths like `src/modules/`, `src/entities/`)
- Searching only for framework-specific decorators/annotations (e.g., only `@Controller`, `@Entity`)

### Required

- **Framework auto-detection table**: Read `package.json`, `requirements.txt`, `go.mod`, `pom.xml`, `composer.json`, etc. to identify the framework
- **Framework-specific pattern mapping**: Branch file patterns, decorators, and structure based on detected framework
- **Unknown framework fallback**: When detection fails, fall back to generic patterns (filenames, export patterns, etc.)

### Verification Checklist

Before merging any skill PR or modification, **run `/qa-skill-review <skill-name>`** to auto-validate. The manual checklist below is for reference:

- [ ] Has framework detection logic?
- [ ] Works beyond NestJS — Express, FastAPI, Spring Boot, Laravel, Django, etc.?
- [ ] Works beyond React — Vue, Angular, Svelte, etc.?
- [ ] No hardcoded paths (`src/modules/`, `src/entities/`)?
- [ ] Has fallback logic when detection fails?
- [ ] Written entirely in English?

---

## Auto-Sync to Global Skills

After creating or modifying any skill under `skills/qa/`, **always sync to global**:

```bash
rm -rf ~/.claude/skills/qa-*
cp -r skills/qa/qa-* ~/.claude/skills/
```

This ensures `~/.claude/skills/` (used by all projects) stays in sync with this repo.

Do NOT skip this step — the global directory is not a git repo and has no other way to receive updates.

### Reverse Sync: Global → Operations

If QA skills were modified in `~/.claude/skills/` (e.g., while working in a different project), sync back to this repo:

```bash
rm -rf skills/qa/qa-*
cp -r ~/.claude/skills/qa-* skills/qa/
```

**Direction does not matter — both locations must always match.** After any QA skill edit, sync whichever side was NOT edited.

---

### Framework Detection Example

```markdown
| Framework | Detection Signal | File Patterns |
|-----------|-----------------|---------------|
| NestJS | `@nestjs/core` in package.json | `*.controller.ts`, `*.service.ts`, `*.module.ts` |
| Express | `express` in package.json | `routes/*.js`, `*.router.js` |
| FastAPI | `fastapi` in requirements.txt | `routers/*.py`, `main.py` |
| Spring Boot | `spring-boot` in pom.xml/build.gradle | `*Controller.java`, `*Service.java` |
| Django | `django` in requirements.txt | `views.py`, `urls.py`, `models.py` |
| Laravel | `laravel/framework` in composer.json | `*Controller.php`, `routes/*.php` |
```
