---
name: create-endpoint
description: >-
  Step-by-step workflow for creating a new FastAPI endpoint on AgentCanvas.
  Covers Pydantic schemas, router, service method, security checklist, and
  linting in the correct order. Followed by the backend agent.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Create Endpoint

## When to use
Follow this skill whenever creating any new API endpoint — from scratch or adding to an
existing router. Invoked by the project-manager plan: `[skill: create-endpoint]`.

---

## Workflow

### Step 1 — Read context (do not skip)
- Read `docs/architecture/ARCHITECTURE.md` — layer rules and existing endpoint map
- Read `docs/security/SECURITY.md` — mandatory security patterns
- Glob `backend/app/schemas/*.py` — match existing schema style exactly
- Glob `backend/app/api/*.py` — match existing router structure

### Step 2 — Create Pydantic request schema
File: `backend/app/schemas/<domain>.py`

- [ ] `min_length` and `max_length` on every string field
- [ ] Enum constraints where field values are fixed
- [ ] `@field_validator` with `name_no_dangerous_chars` on any user-facing string field
  ```python
  @field_validator("name")
  @classmethod
  def name_no_dangerous_chars(cls, v: str) -> str:
      if any(tag in v.lower() for tag in ["<script", "javascript:", "on="]):
          raise ValueError("Invalid characters in name")
      return v.strip()
  ```
- [ ] All fields explicitly typed — no bare `dict` or `Any`

### Step 3 — Create Pydantic response schema
File: `backend/app/schemas/<domain>.py`

- [ ] Every field typed with explicit Python type
- [ ] UUID fields typed as `UUID`, not `str`
- [ ] Datetime fields typed as `datetime`
- [ ] `model_config = ConfigDict(from_attributes=True)` if mapping from SQLAlchemy model

### Step 4 — Create or update the router
File: `backend/app/api/<domain>.py`

- [ ] `@router.METHOD("/path", response_model=ResponseSchema, status_code=NNN)`
- [ ] `dependencies=[Depends(verify_api_key)]` on every protected route
- [ ] `@limiter.limit("10/minute")` if this endpoint calls LLM
- [ ] Router body is exactly: validate input → call ONE service method → return response
- [ ] Zero business logic in the router — no queries, conditionals on data, calculations

### Step 5 — Create or update the service method
File: `backend/app/core/<service>.py` or `backend/app/services/<service>.py`

- [ ] Method signature: `async def method_name(self, param: Type, ...) -> ReturnType:`
- [ ] All DB access uses `async with self.db.begin() as session:` pattern
- [ ] All Redis access uses `async with self.redis.client() as conn:`
- [ ] Enum values validated against canonical list (researcher|analyst|writer|critic|planner|custom)
- [ ] Raise specific exceptions — never bare `Exception` or `raise` without message
- [ ] If calling LLM: `clean = sanitize_prompt(user_content)` before the call
- [ ] If calling LLM: validate response (non-empty content, ≤ 50,000 chars)

### Step 6 — Full security checklist
Before moving on, verify every item:

- [ ] Pydantic schema has length limits and enum constraints
- [ ] `name_no_dangerous_chars` validator present on user-facing string fields
- [ ] `verify_api_key` dependency on route
- [ ] `@limiter.limit("10/minute")` on LLM-calling endpoint
- [ ] `sanitize_prompt()` called before every LLM call
- [ ] LLM response validated (non-empty, ≤ 50,000 chars) if LLM is used
- [ ] No `allow_origins=["*"]` introduced or left in CORS config

### Step 7 — Lint and type-check
```bash
docker-compose exec backend ruff check app/
docker-compose exec backend python -m mypy app/
```
Fix all errors. Do not report done until both pass cleanly.

---

## Output

State for each completed endpoint:
```
Endpoint:  METHOD /api/<path>
Request:   <RequestSchema> — fields: [list]
Response:  <ResponseSchema> — fields: [list]
Service:   <ServiceClass>.<method_name>()
Migration: [filename if schema changed, else "none"]
Lint:      ruff ✓  mypy ✓
```
