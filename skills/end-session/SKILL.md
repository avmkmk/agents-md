---
name: end-session
description: >-
  Close a coding session on AgentCanvas. Fills in the session log end fields,
  updates backlog item statuses, and appends the summary entry to SESSION_INDEX.
allowed-tools: Read, Write, Edit, Glob
---

# End Session

## When to use
Invoke at the end of every coding session — after all code is written, reviewed, and any
open issues are documented. Trigger: user says "end session", "wrap up", "close session",
"done for today", or "finish session".

---

## Workflow

### Step 1 — Identify the current session log
- Find today's active session log in `docs/sessions/` (highest NNN for today)
- Read the full session log to understand what was accomplished

### Step 2 — Fill in end fields
Update the session log with:
- **End Time**: current time IST (ask user if uncertain)
- **Duration**: calculated from start → end time
- **Tasks Done**: mark completed `[ ]` items as `[x]`
- **Files Changed**: list every file created or modified this session
  - Use `git status` or ask the user for the list
- **Decisions Made**: any architectural or approach decisions taken
- **Analytics**: estimate tokens/cost based on session length and context

### Step 3 — Update backlog statuses
Read `docs/planning/BACKLOG.md` and update:
- Items completed this session → `in-progress` → `done`
- Items reviewed but needing fixes → `in-progress` → `review`
- Items started but not finished → remain `in-progress`
- Never mark `done` if code-reviewer flagged unresolved CRITICAL issues

### Step 4 — Update Next Session Prep
In the session log, fill in:
- What to continue in the next session
- Any unresolved blockers
- Which backlog items to pick up first

### Step 5 — Append to SESSION_INDEX
- Read `docs/sessions/SESSION_INDEX.md`
- Append one-line entry to the Index table:
  `| [YYYY-MM-DD-NNN](./YYYY-MM-DD-NNN.md) | date | duration | mode | goals summary | issues | cost |`
- Update Cumulative Stats: increment Total Sessions, add duration, add cost, add files created

---

## Output

```
Session closed: docs/sessions/YYYY-MM-DD-NNN.md
Duration: X hr Y min
Tasks completed: N  |  Files changed: N
Backlog updated: [list of IDs and their new status]
SESSION_INDEX updated.
Next session: [what to pick up]
```

---

## Checklist

- [ ] Session log end time and duration filled in
- [ ] All completed tasks marked `[x]`
- [ ] Files changed table populated
- [ ] Backlog statuses updated (done / review / in-progress)
- [ ] Next session prep section filled
- [ ] SESSION_INDEX entry appended
- [ ] Cumulative stats updated
