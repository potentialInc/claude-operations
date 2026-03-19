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
cp -r skills/qa/_qa-fix ~/.claude/skills/qa-fix
cp -r skills/qa/_qa-shared ~/.claude/skills/qa-shared
cp -r skills/qa/*/qa-* ~/.claude/skills/
```

This ensures `~/.claude/skills/` (used by all projects) stays in sync with this repo.

Do NOT skip this step — the global directory is not a git repo and has no other way to receive updates.

### Reverse Sync: Global → Operations

If QA skills were modified in `~/.claude/skills/` (e.g., while working in a different project), sync back to this repo. Note: the operations repo uses a layered directory structure:

```
skills/qa/
  _qa-fix/         ← orchestrator (top-level, _ prefix for sort order)
  _qa-shared/      ← shared patterns (top-level, _ prefix for sort order)
  data/            ← Layer 1: qa-db-integrity, qa-dead-code
  api/             ← Layer 2: qa-api-sync, qa-crud
  auth/            ← Layer 3: qa-auth, qa-security
  inputs/          ← Layer 4: qa-inputs
  ui/              ← Layer 5: qa-a11y, qa-back-nav, qa-buttons, qa-layout, qa-list, qa-modal, qa-performance, qa-states
  tools/           ← Standalone: qa-page, qa-screen, qa-test-gen, qa-skill-review
```

When syncing back, place each skill in its correct layer directory.

**Direction does not matter — both locations must always match.** After any QA skill edit, sync whichever side was NOT edited.

### Store Skills Sync

Operations uses short folder names (`store/prep/`), but global uses prefixed names (`~/.claude/skills/store-prep/`).

After creating or modifying any skill under `skills/store/`, sync to global:

```bash
rm -rf ~/.claude/skills/store-* ~/.claude/skills/_store-shared
cp -r skills/store/_store-shared ~/.claude/skills/_store-shared
for d in skills/store/*/; do
  name=$(basename "$d")
  [[ "$name" == "_store-shared" ]] && continue
  name="${name#_}"
  cp -r "$d" ~/.claude/skills/store-$name
done
```

Reverse sync (global → operations):

```bash
# Shared reference
rm -rf skills/store/_store-shared
cp -r ~/.claude/skills/_store-shared skills/store/_store-shared

# Skills
for d in ~/.claude/skills/store-*/; do
  name=$(basename "$d")
  short="${name#store-}"
  rm -rf "skills/store/$short"
  cp -r "$d" "skills/store/$short"
done
```

```
skills/store/
  _ship/           ← pipeline orchestrator
  _store-shared/   ← shared rules (gitignore pre-flight, etc.)
  assets/    build/
  deploy/    native/    prep/
  review/    submit/
```

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
