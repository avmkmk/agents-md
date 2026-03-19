---
name: create-component
description: >-
  Step-by-step workflow for creating a new React component on AgentCanvas.
  Covers TypeScript type, Zod validator, Zustand store action, custom hook,
  and component in the correct order. Followed by the frontend agent.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Create Component

## When to use
Follow this skill whenever creating any new React component — including nodes, panels,
modals, or page-level views. Invoked by the project-manager plan: `[skill: create-component]`.

---

## Workflow

### Step 1 — Read context (do not skip)
- Read `docs/architecture/ARCHITECTURE.md` — component map and state management rules
- Glob `frontend/src/types/*.ts` — match existing type style
- Glob `frontend/src/components/` — match existing component structure and naming
- Glob `frontend/src/store/*.ts` — match existing store patterns

### Step 2 — Define the TypeScript type
File: `frontend/src/types/<domain>.ts`

- [ ] Interface name matches the domain object (e.g., `Flow`, `Execution`, `HITLReview`)
- [ ] All fields explicitly typed — no `any`
- [ ] Field names in `camelCase` matching the backend `snake_case` → `camelCase` convention
- [ ] Optional fields use `field?: Type` not `field: Type | undefined`
- [ ] UUID fields typed as `string` (JSON serialises UUIDs as strings)
- [ ] Datetime fields typed as `string` (ISO 8601 from API) — parse to `Date` in component if needed
- [ ] Confirm every field matches the corresponding backend Pydantic response schema

### Step 3 — Add Zod validator
File: `frontend/src/services/<domain>Service.ts` or `apiClient.ts`

- [ ] Zod schema mirrors the TypeScript type exactly
- [ ] Use `.parse()` on every API response before writing to the store
- [ ] Never write raw API data to the store without validation

### Step 4 — Add Zustand store action
File: `frontend/src/store/<domain>Store.ts`

- [ ] State slice: holds the domain objects (`flows: Flow[]`, `selectedFlow: Flow | null`, etc.)
- [ ] Action: typed correctly — `addFlow: (flow: Flow) => void`
- [ ] No direct `fetch` or `axios` calls inside the store — call the service layer
- [ ] Store is the single source of truth for this domain — no duplicate `useState`

### Step 5 — Create custom hook
File: `frontend/src/hooks/use<Domain>.ts`

- [ ] Hook name: `use<Domain>` (e.g., `useFlow`, `useExecution`)
- [ ] Exposes: state slice + actions (not the raw store)
- [ ] Calls service functions for data fetching — not store directly
- [ ] Handles loading state and error state
- [ ] Component never accesses the store directly — only through this hook

### Step 6 — Create the component
File: `frontend/src/components/<category>/<ComponentName>.tsx`

- [ ] Props interface defined inline or in `types/<domain>.ts`
- [ ] Component calls the hook — never the store, service, or API directly
- [ ] Guard clauses at the top for loading/error/empty states
- [ ] No inline styles — use CSS modules or Tailwind classes
- [ ] Target < 100 lines; split into sub-components if larger
- [ ] No prop drilling beyond 2 levels — use Zustand for deeper sharing

### Step 7 — Lint and format
```bash
docker-compose exec frontend npm run lint
docker-compose exec frontend npm run format
```
Fix all errors. Do not report done until both pass cleanly.

---

## Output

State for each completed component:
```
Type:       frontend/src/types/<domain>.ts — <InterfaceName>
Validator:  frontend/src/services/<domain>Service.ts — Zod schema
Store:      frontend/src/store/<domain>Store.ts — action: <actionName>
Hook:       frontend/src/hooks/use<Domain>.ts
Component:  frontend/src/components/<category>/<Name>.tsx — ~N lines
Lint:       eslint ✓  format ✓
Type match: confirmed against backend <ResponseSchema>
```
