---
name: review
description: >-
  Review all changed AgentCanvas code against 10 coding standards, the security
  checklist, and architecture constraints. Produces a structured report with
  severity levels and agent hand-off instructions. Executed by the code-reviewer agent.
allowed-tools: Bash, Glob, Grep, Read
---

# Review

## When to use
Follow this skill after implementation is complete and before marking any backlog item done.
Invoked by the project-manager plan: `[skill: review]`. Also invoked directly with `/review`.

---

## Workflow

### Step 1 — Identify changed files
```bash
git diff --name-only HEAD           # uncommitted changes vs last commit
git diff --name-only HEAD~1 HEAD    # last commit vs one before
git status --short                  # staged + unstaged overview
```
Skip: `*.min.js`, `dist/`, `build/`, `__pycache__/`, `node_modules/`, `*.lock`

If no changed files detected — ask the user to specify a commit range or stage files.

### Step 2 — Read every changed file in full
For each file: read completely before forming any judgement. Do not skim.

### Step 3 — Apply the 10 coding standards
Check every standard against every changed file. Mark: ✅ pass | ❌ fail (cite file:line).

| # | Standard | Flag When You See |
|---|----------|-------------------|
| 1 | Simple Control Flow | Recursion; `reduce` for sequential ops; call depth > 5 |
| 2 | Resource Management | DB/Redis not in `async with`; unclosed file handle |
| 3 | Explicitness | Missing type hints on public functions; `any` in TypeScript |
| 4 | Consistency | Two patterns for same operation; style diverges from adjacent code |
| 5 | Error Handling | `except: pass`; silent swallow; missing error check after I/O |
| 6 | Readability | Python fn > 50 lines; TS component > 100 lines; nesting > 2 levels |
| 7 | Static Analysis | Would fail `ruff`/`mypy` or `eslint`/`tsc` — run to confirm |
| 8 | Third-Party Wrapping | External API called without a service class wrapper |
| 9 | Safe Resources | Missing Pydantic/Zod validation; raw SQL; unsanitized LLM prompt |
| 10 | Reproducibility | `eval()`/`exec()`; non-deterministic logic without explanation |

### Step 4 — Apply security checklist (every backend file)

- [ ] Pydantic schema: `min_length`, `max_length`, enum constraints present
- [ ] `name_no_dangerous_chars` validator on user-facing string fields
- [ ] `verify_api_key` dependency on every protected route
- [ ] `sanitize_prompt()` called before every LLM invocation
- [ ] `@limiter.limit("10/minute")` on every LLM-calling endpoint
- [ ] No `allow_origins=["*"]` in CORS configuration
- [ ] LLM response validated: non-empty content, ≤ 50,000 chars

### Step 5 — Apply architecture checklist

**Backend files:**
- [ ] Router body = validation + one service call + return (nothing else)
- [ ] No DB queries in router files
- [ ] All I/O is `async/await`

**Frontend files:**
- [ ] No direct `fetch`/`axios` in components
- [ ] Server state in Zustand, not local `useState`
- [ ] API responses Zod-validated before entering store
- [ ] No prop drilling beyond 2 levels

### Step 6 — Run lint (confirm findings)
```bash
# Backend changed files
docker-compose exec backend ruff check app/
docker-compose exec backend python -m mypy app/

# Frontend changed files
docker-compose exec frontend npm run lint
```

### Step 7 — Produce the report

```
## Code Review — [date] — [N files reviewed]

### Summary
Files reviewed: N  |  Critical: N  |  Major: N  |  Minor: N
Overall: PASS ✅ / NEEDS FIXES ❌

### CRITICAL Issues  (fix before any commit)
1. [Standard N violated] — `path/to/file.py:line`
   Problem: [what the code does]
   Risk:    [why it is dangerous or incorrect]
   Fix:     [what to change — description only, no replacement code]

### MAJOR Issues  (fix before marking task done)
[same format]

### MINOR Issues  (fix before M6)
- `file:line` — [one-line description]

### Passed Checks
- Standards: [list numbers that passed cleanly]
- Security: [list items that passed]

### Hand-off
- backend agent: issues [N, N, N]
- frontend agent: issues [N, N]
- integration agent: issues [N]
```

---

## Severity Guide

| Severity | Examples | Blocking? |
|----------|---------|-----------|
| CRITICAL | Missing auth, unsanitized LLM prompt, architecture layer violation | Blocks commit |
| MAJOR | Missing type hints, resource leak, silent error | Blocks done |
| MINOR | Naming, missing comment, style drift | Blocks M6 |
