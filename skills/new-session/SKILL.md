---
name: new-session
description: >-
  Start a new AgentCanvas coding session. Creates the session log, reads the
  backlog for context, and triggers the project-manager agent to produce a
  structured task plan for the session.
allowed-tools: Read, Write, Glob
---

# New Session

## When to use
Invoke at the very start of every coding session — before any code is written.
Trigger: user says "new session", "let's start", "begin session", or "continue from last time".

---

## Workflow

### Step 1 — Determine session ID
- Check `docs/sessions/` for existing files matching today's date (`YYYY-MM-DD-*.md`)
- Next number is `001` if none exist today, else increment
- Session ID format: `YYYY-MM-DD-NNN` (e.g., `2026-03-05-002`)

### Step 2 — Create session log
- Read `docs/sessions/SESSION_TEMPLATE.md`
- Create `docs/sessions/YYYY-MM-DD-NNN.md` filling in:
  - Session ID, Date, Start Time (current time IST), Agent Model: `claude-sonnet-4-6`
  - Session Mode: ask user or default to `Mixed`
  - Goals: from the user's stated task, or leave as `[ ]` for PM agent to fill

### Step 3 — Read current project state
- Read `docs/planning/BACKLOG.md` — note all items with status `in-progress` (these carry over)
- Read last entry in `docs/sessions/SESSION_INDEX.md` — understand where last session ended
- Check for any blockers listed in the previous session log

### Step 4 — Establish this session's task
Three cases:
- **User gave a specific task** → pass to project-manager agent to decompose
- **User said "continue"** → pick up all `in-progress` backlog items from the last session
- **No direction given** → suggest the next `open` + `critical` item from the current milestone

### Step 5 — Hand off to project-manager agent
The project-manager agent will:
- Confirm or adjust the task selection
- Produce the full sub-task plan with agent and skill assignments
- Mark selected backlog items as `in-progress`

---

## Output

```
Session started: docs/sessions/YYYY-MM-DD-NNN.md
Task this session: [issue IDs + one-line description]
Handing off to project-manager for task decomposition...
```

---

## Checklist

- [ ] Session log file created at correct path
- [ ] Session ID is sequential (no duplicates for today)
- [ ] In-progress items from last session identified
- [ ] Task for this session confirmed with user
- [ ] project-manager agent invoked to produce task plan
