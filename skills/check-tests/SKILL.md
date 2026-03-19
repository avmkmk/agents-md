---
name: check-tests
description: >-
  Assess whether unit or integration tests should be written now or deferred
  to M6 for AgentCanvas. Writes tests immediately when the decision is WRITE NOW.
  Executed by the qa-testing agent.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Check Tests

## When to use
Follow this skill after implementation and code review — before closing a task.
Invoked by the project-manager plan: `[skill: check-tests]`. Also invoked with `/check-tests`.

---

## Workflow

### Step 1 — Read the implementation
- Read every file created or modified this session
- Read `docs/development/TESTING_STRATEGY.md` — test patterns and coverage requirements
- Check `docs/planning/BACKLOG.md` — confirm current milestone

### Step 2 — Apply the WRITE NOW vs DEFER decision

**WRITE NOW — mandatory immediate tests:**

| Condition | Test Type |
|-----------|-----------|
| File is in `core/` (FlowExecutor, AgentRunner, HITLManager) | Unit — mock LLM and DB |
| File is in `memory/memory_service.py` | Unit — mock Redis/Mongo |
| File is in `services/llm_service.py` | Unit — mock Anthropic client |
| File is `utils/prompt_sanitizer.py` or any validator | Unit — test each injection pattern |
| New API endpoint with non-trivial business rules in service | Integration — real test DB |
| HITL gate state machine logic | Unit — test all state transitions |

**DEFER TO M6 — do not write now:**

| Condition | Reason |
|-----------|--------|
| Docker / environment config | Infrastructure, not logic |
| Skeleton files with no implementation | Nothing to test |
| Pydantic schema (no custom validators) | FastAPI validates automatically |
| SQLAlchemy model (column definitions only) | ORM, not business logic |
| React component without logic branches | Visual — deferred to M6 |
| Zustand store (basic set/get actions) | Trivial, no business logic |
| Vite config, tsconfig, ESLint config | Tooling, not application |

**Decision output must be explicit:**
```
Decision: WRITE NOW  — reason: [FlowExecutor contains branching logic critical to product]
Decision: DEFER TO M6 — reason: [file is a Pydantic schema with no custom validators]
```

### Step 3 — If WRITE NOW: identify test locations

```
backend/tests/unit/test_<module>.py        ← for core services
backend/tests/integration/test_<domain>_api.py  ← for API endpoints
frontend/src/__tests__/hooks/use<Domain>.test.ts  ← for hooks
frontend/src/__tests__/components/<Name>.test.tsx  ← for components
```

### Step 4 — Write the tests

**Backend rules:**
- All tests `async` with `@pytest.mark.asyncio`
- Use `mock_llm` fixture — never call real Anthropic API
- Use `db_session` fixture — never use dev database
- Test name pattern: `test_<function>_<scenario>_<expected_outcome>`
- Write: 1 happy path + at least 1 failure path per service method

**Frontend rules:**
- Use `@testing-library/react` — test behaviour, not implementation
- Mock API calls with `vi.fn()` — never call the real backend
- Pattern: render → act → assert

**Always write `conftest.py` first** if it doesn't exist yet.

### Step 5 — Run the tests
```bash
# Backend
docker-compose exec backend pytest tests/unit/test_<module>.py -v
docker-compose exec backend pytest --cov=app --cov-report=term-missing

# Frontend
docker-compose exec frontend npm test -- --reporter=verbose
docker-compose exec frontend npm run test:coverage
```

Fix all failures before reporting done.

---

## Output

```
## QA Assessment — [feature / files assessed]

Decision: WRITE NOW / DEFER TO M6
Reason:   [one sentence]

[If WRITE NOW:]
Tests written:
  - backend/tests/unit/test_<module>.py  — N tests (happy path + N failure paths)
  - [additional test files]

Coverage:
  - <module>: N%  (requirement: N%)
  - Overall: ✅ meets minimum / ❌ below minimum

[If DEFER:]
Deferred items logged → BACKLOG.md [T-xx] M6
```

---

## Coverage Requirements Reference

| Module | Minimum |
|--------|---------|
| `core/flow_executor.py` | 80% |
| `core/hitl_manager.py` | 80% |
| `memory/memory_service.py` | 75% |
| `services/llm_service.py` | 70% |
| `api/*.py` routers | 70% |
| Frontend hooks | 70% |
| Critical components (HITL, Canvas) | 60% |
