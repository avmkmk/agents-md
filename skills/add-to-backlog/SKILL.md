---
name: add-to-backlog
description: >-
  Add a properly formatted feature, bug, or task to the AgentCanvas BACKLOG.md
  in the correct track with the right issue ID, priority, and milestone.
  Followed by the project-manager agent.
allowed-tools: Read, Edit
---

# Add to Backlog

## When to use
Follow this skill when a new feature, bug, or task needs to be tracked — discovered during
development, requested mid-session, or arising from a code review finding.
Invoked by the project-manager plan or user request: `[skill: add-to-backlog]`.

---

## Workflow

### Step 1 — Read the backlog
- Read `docs/planning/BACKLOG.md` in full
- Identify the correct track for this item:

| Track | Use When |
|-------|----------|
| `track:devops` | Docker, CI/CD, environment, deployment |
| `track:database` | Schema, migrations, indexes, ORM models |
| `track:backend-api` | FastAPI routes, Pydantic schemas |
| `track:backend-core` | Core services: FlowExecutor, HITLManager, etc. |
| `track:frontend` | React components, hooks, stores, types |
| `track:testing` | Unit tests, integration tests, test fixtures |
| `track:security` | Auth, input validation, prompt sanitization |

### Step 2 — Determine the next issue ID
- Each track has its own prefix: `D-`, `DB-`, `BA-`, `BC-`, `FE-`, `T-`, `S-`
- Find the highest existing number in the target track section
- Next ID = highest + 1 (e.g., if last is `FE-20`, next is `FE-21`)

### Step 3 — Determine priority and milestone
**Priority:**
- `critical` — blocks other work; product cannot function without it
- `high` — important, needed in current milestone
- `medium` — useful, can be deferred one milestone
- `low` — nice to have, M6 or post-MVP

**Milestone:**
- M1 (foundation), M2 (flow CRUD), M3 (execution), M4 (HITL), M5 (analytics), M6 (tests+CI)
- Match to the milestone where this item logically belongs, not where we currently are

### Step 4 — Add the entry
Add to the correct track section in `BACKLOG.md`:

```markdown
| <ID> | <Title — imperative verb, specific> | <priority> | M<N> | open |
```

Rules for the title:
- Start with an imperative verb: "Add", "Fix", "Create", "Implement", "Update"
- Be specific: "Add pagination to GET /flows" not "Improve flows endpoint"
- Max 60 characters

---

## Output

```
Added: <ID> — "<Title>"
Track: track:<name>
Priority: <priority>  |  Milestone: M<N>
Status: open
```
