---
name: db-migration
description: >-
  Step-by-step workflow for writing a PostgreSQL schema migration and updating
  the matching SQLAlchemy ORM model on AgentCanvas. Followed by the backend agent.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# DB Migration

## When to use
Follow this skill whenever adding or modifying database tables, columns, indexes, or
constraints. Invoked by the project-manager plan: `[skill: db-migration]`.

---

## Workflow

### Step 1 — Read current schema state
- Read `docs/architecture/DATABASE.md` — full schema, existing tables, column definitions
- Glob `backend/app/db/migrations/` — find the highest existing migration number
- Read the latest migration file to understand current schema state

### Step 2 — Determine migration number and name
- Next migration = highest existing number + 1 (zero-padded to 3 digits)
- Naming: `NNN_<short_description>.sql` (e.g., `002_add_flow_tags.sql`)
- File location: `backend/app/db/migrations/NNN_<name>.sql`

### Step 3 — Write the migration file
File: `backend/app/db/migrations/NNN_<name>.sql`

Structure every migration file with both UP and DOWN:
```sql
-- Migration: NNN_<name>
-- Description: [what this migration does]
-- UP
[SQL to apply the change]

-- DOWN
[SQL to reverse the change — always provide this]
```

Checklist for the UP section:
- [ ] Use `IF NOT EXISTS` for CREATE TABLE, CREATE INDEX
- [ ] Use `IF EXISTS` for DROP TABLE, DROP COLUMN
- [ ] All new string columns have `NOT NULL` or a default value — no silent NULLs
- [ ] All new enum-constrained columns use `CHECK` constraints matching canonical values:
  - `agents.role`: `CHECK (role IN ('researcher','analyst','writer','critic','planner','custom'))`
  - `flow_executions.status`: `CHECK (status IN ('pending','running','paused_hitl','completed','failed','cancelled'))`
  - `hitl_reviews.gate_type`: `CHECK (gate_type IN ('before','after','on_demand'))`
  - `hitl_reviews.status`: `CHECK (status IN ('pending','approved','rejected'))`
- [ ] UUID primary keys use `DEFAULT uuid_generate_v4()`
- [ ] Timestamp columns use `DEFAULT NOW()`
- [ ] Foreign keys have `ON DELETE CASCADE` or `ON DELETE SET NULL` — never silent orphans
- [ ] Indexes added for every foreign key column and every column used in WHERE filters

### Step 4 — Update the SQLAlchemy ORM model
File: `backend/app/models/<domain>.py`

- [ ] New column added with explicit type (`String(255)`, `Text`, `UUID`, `DateTime`, etc.)
- [ ] Enum columns use `SQLAlchemyEnum` with the canonical string values
- [ ] Foreign key columns have `ForeignKey("table.id", ondelete="CASCADE")`
- [ ] No business logic in the model class — column definitions only
- [ ] `__tablename__` matches the SQL table name exactly

### Step 5 — Verify the migration
```bash
# Check SQL syntax by connecting to a running postgres container
docker-compose exec postgres psql -U admin -d orchestrator -f /path/to/migration.sql
```
If the container isn't running, review manually for:
- [ ] No typos in column names
- [ ] Enum values exactly match the canonical list
- [ ] DOWN section reverses the UP section completely

### Step 6 — Update DATABASE.md
- Add/update the table entry in `docs/architecture/DATABASE.md`
- Record new columns, indexes, constraints

---

## Output

```
Migration:  backend/app/db/migrations/NNN_<name>.sql
Model:      backend/app/models/<domain>.py — updated
Changes:    [list: tables/columns added, indexes created]
Enum check: [confirm all CHECK constraints match canonical values]
Down:       [confirm rollback SQL is present]
Docs:       DATABASE.md updated
```
