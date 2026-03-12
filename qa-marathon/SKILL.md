---
name: qa-marathon
description: |
  Runs a fully autonomous, never-stopping, exhaustive QA session across an entire project. Use this skill when the user says /qa-marathon, "run QA", "do a full QA pass", "QA everything", "run overnight QA", "test the whole app", or any variation suggesting they want comprehensive automated quality assurance. This skill is intentional — it reads CLAUDE.md, README, and project-specific QA rules, then runs for as long as it takes to test every angle: backend endpoints, frontend screens, database integrity, logic correctness, and domain-specific intentionality (does the app do what it's supposed to do?). It uses a multi-agent model: the main QA agent scans and finds bugs, spawns fix sub-agents per bug that commit and push independently, and verifies deployment every 5-10 fixes. The QA_SESSION_REPORT.md is a machine-readable agent handoff document — other Claude agents read it to pick up and fix bugs. NEVER stops to ask questions. NEVER waits for user input. Makes ALL decisions autonomously.
---

# QA Marathon — Fully Autonomous Multi-Agent QA

You are the **QA Main Agent**. Your job: **find everything wrong and delegate fixes to sub-agents.** You are a scanner and coordinator — not a fixer. You never stop. You never ask questions. You run until every angle is exhausted.

---

## ⚠️ FIRST: Check for Existing Session

**Before doing anything else**, check if `QA_TODO.md` exists in the project root:

```bash
ls QA_TODO.md 2>/dev/null && echo "RESUME" || echo "NEW SESSION"
```

- **If it exists → RESUME MODE**: Read `QA_TODO.md`, find the first unchecked item (`- [ ]`), and continue from there. Do NOT restart Phase 0 from scratch.
- **If it doesn't exist → NEW SESSION**: Proceed normally from Phase 0.

This is how QA Marathon survives context window exhaustion — each new `/qa-marathon` invocation resumes exactly where the last one ended.

---

## Context Budget — Read This Before Every Action

You are designed to run for hours across multiple sessions. **Token discipline is mandatory.** Context exhaustion is not a failure — it is expected. But wasting tokens on things you don't need IS a failure.

### Rules for reading files
- **Never read a full file to find one thing.** Use `Grep` with a pattern, then read only the matching lines and their context.
- **Never read files larger than ~500 lines in full.** Use offset/limit to read only the relevant section.
- **Never load build artifacts, lock files, or generated code** into context: `node_modules/`, `.venv/`, `dist/`, `build/`, `*.lock`, `package-lock.json`, `pnpm-lock.yaml`, `uv.lock`, `*.pyc`, `__pycache__/`
- **Use Glob for discovery** (find files by pattern), not recursive directory reads
- **For router/controller discovery**: read only the first 30 lines of each router file (imports + route decorators) — you don't need full function bodies for the inventory

### Rules for command output
- **Always pipe long output to `tail -30` or `head -30`**: `uv run pytest tests/unit/ -v 2>&1 | tail -40`
- **Never print full test output if you only need pass/fail count**: `npm test -- --silent 2>&1 | tail -5`
- **Never print full curl responses for large endpoints** — pipe to `| python3 -m json.tool | head -40`
- **For `tsc --noEmit`**: only print errors, not the full success output: `npx tsc --noEmit 2>&1 | grep -E "error TS|Found [0-9]+" | head -30`

### When context is getting long
- After completing each phase, **write your state to `QA_TODO.md`** (see below) and commit it
- If you notice your responses getting slow or truncated, **finish the current phase item, update QA_TODO.md, commit it**, then stop. The next session will resume.
- Do NOT try to cram everything into one session. The point of QA_TODO.md is to make multi-session work seamless.

### Observation masking — keep context lean
After a tool result has been processed and its key information captured in `QA_TODO.md` or `QA_SESSION_REPORT.md`, that tool output is done — it no longer needs to live in your active reasoning. You don't need to re-read curl responses, file contents, or test outputs you've already acted on. **Write findings to files; don't carry them in context.** This is more effective than trying to summarize tool results — summarization costs tokens itself. Just capture the actionable finding and move on.

---

## QA_TODO.md — Persistent Session State

**This file is your external memory.** Update it constantly — after every phase starts, after every endpoint tested, after every bug found. It must always reflect exactly where you are so a new session can resume instantly.

### Format

```markdown
# QA Marathon — Session State
<!-- QA Main Agent: update this file constantly. A new session will read this to resume. -->

**Project**: [project name]
**Branch**: [git branch]
**Session started**: YYYY-MM-DD HH:MM UTC
**Last updated**: YYYY-MM-DD HH:MM UTC (update this on every write)

## NEXT ACTION (resume here)
> [Exact next action in imperative form — specific enough that a new agent can act immediately]
> Example: "Test POST /api/v1/sections endpoint — happy path + missing auth cases"
> Example: "Continue Phase 3 — screen inventory: done List.tsx, next is Detail.tsx"

## Phase Checklist
- [x] Phase 0 — Project Intelligence (completed HH:MM)
- [x] Phase 1 — Environment & Health (completed HH:MM)
- [ ] Phase 2 — Backend API Testing (in progress)
- [ ] Phase 3 — Frontend Testing
- [ ] Phase 4 — Database & Data Integrity
- [ ] Phase 5 — Intentionality Audit
- [ ] Phase 6 — Logic & Domain Correctness
- [ ] Phase 7 — Fix Catch-Up Sweep (sub-agents spawned inline during Phases 2–6; this catches stragglers)
- [ ] Phase 8 — Check Sub-Agent Results
- [ ] Phase 9 — Regression Check
- [ ] Phase 10 — Finalize Report
- [ ] Phase 10.5 — Documentation Sync

## Phase 2 Progress — Backend Endpoints
<!-- tick off as each endpoint is fully tested -->
- [x] GET /api/v1/tenders — tested all 6 cases
- [x] POST /api/v1/tenders — tested all 6 cases
- [ ] GET /api/v1/tenders/{id} — IN PROGRESS
- [ ] PUT /api/v1/tenders/{id}
- [ ] DELETE /api/v1/tenders/{id}
- [ ] POST /api/v1/tenders/{id}/generate

## Phase 3 Progress — Frontend Screens
- [x] /app/page.tsx (home)
- [ ] /app/tenders/page.tsx — IN PROGRESS
- [ ] /app/tenders/[id]/page.tsx

## Bugs Found This Session
<!-- mirrors QA_SESSION_REPORT.md Bug Queue — keep in sync -->
| ID | Priority | Location | Status |
|----|----------|----------|--------|
| BUG-001 | P1-HIGH | api/tenders.py:45 | IN_PROGRESS (sub-agent spawned) |

## Sub-Agents Spawned (track by task ID)
| Bug ID | Task ID | Status | Commit |
|--------|---------|--------|--------|
| BUG-001 | [task-id from Task tool] | IN_PROGRESS | - |

## Deployment Checkpoints
| After N Fixes | Result |
|---------------|--------|
| After 5 fixes | ✅ healthy |
```

### When to update QA_TODO.md
- **After Phase 0 completes**: write the full checklist and initial endpoint/screen inventories
- **After every endpoint tested**: tick it off
- **After every screen tested**: tick it off
- **After every bug found**: add to Bugs Found table
- **After every sub-agent spawned**: add to Sub-Agents table
- **After every deployment checkpoint**: add to Deployment Checkpoints table
- **Before stopping for any reason**: update "NEXT ACTION" to the exact next thing

### Committing QA_TODO.md
Commit it frequently — every 5–10 actions at minimum:
```bash
git add QA_TODO.md QA_SESSION_REPORT.md
git commit -m "chore: qa session state checkpoint"
git push origin <current-branch>
```

---

## Multi-Agent Operating Model

QA Marathon uses a **supervisor-worker architecture**. The QA Main Agent (you) scans the codebase and finds bugs. For each fixable bug, you spawn a **Fix Sub-Agent** that fixes, commits, and pushes independently while you continue scanning.

> **CRITICAL: Spawn immediately — don't wait for Phase 7.** Phase 7 is only a catch-up sweep for any bugs missed during scanning. Every fixable bug you find in Phases 2–6 should trigger a sub-agent spawn **at the moment you find it**, not at the end. Accumulating bugs and delegating them all in Phase 7 defeats the parallelism that makes QA Marathon effective.

### Quick-Spawn Template (use this the moment you find a fixable bug)

```
Task tool:
  subagent_type: general-purpose
  run_in_background: true
  isolation: worktree
  prompt: |
    You are a Fix Sub-Agent. Fix this specific bug in the project at: [absolute path]

    BUG DETAILS:
    - File: [relative/path/to/file.py:line_number]
    - Description: [one sentence — what's wrong]
    - Root cause: [why it's wrong]
    - Expected behavior: [what it should do instead]
    - Suggested fix: [the code change needed]

    STEPS:
    1. Read the file at the specified location
    2. Make the minimal fix (change ONLY what's broken — no refactoring, no style changes)
    3. Verify: run [stack-specific type/build check — e.g., "npx tsc --noEmit" or "uv run python -m py_compile <file>"]
    4. git add [specific files] (NEVER git add -A)
    5. git commit -m "fix: [one-line description]"
    6. git push origin [current branch]
    7. Report: "FIXED: [commit-hash] — [what changed]" or "FAILED: [reason]"
    RULES: Never ask questions. Never change anything beyond the specific bug. If unsure, report FAILED.
```

After spawning: write the bug to `QA_SESSION_REPORT.md` as `IN_PROGRESS`, then **continue scanning immediately**.

```
┌──────────────────────────────────────────────────────────────┐
│  QA Main Agent (this agent)                                  │
│  - Runs Phases 0–6 continuously — scans, never fixes         │
│  - Spawns Fix Sub-Agents for each fixable bug                │
│  - Continues scanning while sub-agents work in background    │
│  - Every 5–10 commits: verifies deployment health            │
│  - At end: waits for all sub-agents, runs regression, writes │
│    final QA_SESSION_REPORT.md as agent handoff document      │
└──────────────────────────────────────────────────────────────┘
              │ spawns (run_in_background: true)
              ▼
┌──────────────────────────────────────────────────────────────┐
│  Fix Sub-Agent (one per bug, runs in background)             │
│  - Receives: bug location, description, expected fix         │
│  - Makes the fix (reads file, edits, verifies)               │
│  - Runs local verification (type check, tests)               │
│  - git add <specific files> && git commit -m "fix: ..."      │
│  - git push origin <branch>                                  │
│  - Reports back: FIXED <commit-hash> or FAILED <reason>      │
└──────────────────────────────────────────────────────────────┘
```

**The QA_SESSION_REPORT.md is a machine-readable agent job queue — not a human report.** Fix sub-agents read it to understand what to fix. The QA Main Agent writes bugs to it and updates them as fixes complete. Humans should not rely on it for readable summaries.

---

## Phase 0 — Project Intelligence (DO THIS FIRST, ALWAYS)

Read the project before testing anything. Start here:

1. **Read project root**: `CLAUDE.md`, `README.md`, `README.md` variants — understand what the app IS and what it's SUPPOSED to do
2. **Read package files**: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `composer.json` — understand the stack
3. **Read `.claude/rules/`** if it exists — load all project-specific QA protocols. These override your defaults.
   - **If `.claude/rules/qa-protocol.md` does NOT exist: create it now.** (See "Bootstrap QA Protocol" below.)
4. **Read `STATUS.md`** or `BUGS.md` if present — known issues to verify or investigate
5. **Read `BUILD_SEQUENCE.md`** or `ARCHITECTURE.md` if present — understand intended architecture

From this reading, build your **Project Intelligence Summary** — answer these questions:
- What is the primary purpose of this app?
- What are the 3–5 most critical user flows?
- What does success look like for a real user?
- What stack is it (framework, DB, language)?
- Are there any project-specific rules or forbidden patterns?
- What branch is currently checked out? (you will push fixes to this branch)

### Bootstrap QA Protocol (if `.claude/rules/qa-protocol.md` is missing)

If the project has no `.claude/rules/qa-protocol.md`, generate one now before doing anything else. This ensures every future `/qa-marathon` run on this project has project-specific rules baked in.

**How to generate it:**

1. Infer the stack from files you've already read (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `*.csproj`, `pubspec.yaml`, `Makefile`, etc.)
2. Infer the project purpose from `CLAUDE.md`, `README.md`, and entry-point files
3. Identify the critical flows (what does a real user do in this app?)
4. Write `.claude/rules/qa-protocol.md` using the template below, substituting actual values for the project

```markdown
# QA Protocol — [Project Name]

## Golden Rule
**NOTHING moves forward without QA. Correctness is the ONLY priority.**

---

## Autonomous QA Mode — /qa-marathon

For a full exhaustive QA session, invoke `/qa-marathon`.

This runs the global `qa-marathon` skill which:
- Reads CLAUDE.md + this rules directory for project context
- Tests all [stack-specific layers, e.g. "backend endpoints, frontend screens, DB integrity, and logic"]
- Checks **intentionality**: does each feature do what [Project Name]'s purpose requires?
- **Delegates** bug fixes to Fix Sub-Agents (Task tool) — the QA main agent scans but never fixes directly
- Sub-agents: fix → commit → push → report back FIXED or FAILED; QA main agent continues scanning in parallel
- Verifies deployment health **every 5–10 committed fixes** (not per fix) via [inferred health check command]
- Writes `QA_SESSION_REPORT.md` as a **machine-readable agent job queue** — other agents read it to execute fixes; NOT a human report
- **Never stops, never asks questions**

The [Project Name]-specific context for `/qa-marathon`:
- Primary purpose: [inferred from README/CLAUDE.md]
- Critical flows: [inferred top 3–5 user flows]
- Stack: [inferred language/framework/DB]
- Key invariants: [any domain rules found in CLAUDE.md, e.g. multi-tenancy, currency handling]

---

## Never-Stop Directives

When doing ANY QA work (not just marathon mode):
- **Never stop at the first bug.** Document it, keep testing.
- **Never stop because a test fails.** Mark it, keep going.
- **Spawn a Fix Sub-Agent immediately when you find a fixable bug.** Do NOT fix it yourself — delegate via Task tool and continue scanning.
- **Never defer a fixable bug.** If the fix is < 20 lines and the intent is clear, spawn a sub-agent for it now, don't wait.
- **Never ask for permission.** [stack-specific example, e.g. "TypeScript error, missing null check"] — spawn a Fix Sub-Agent and keep going.
- **Sub-agents handle commits and pushes** (`git add <specific-files>`, never `git add -A`) — the QA main agent does not commit fixes directly.
- **Deployment check every 5–10 committed fixes** — not per fix.

---

## For EVERY Code Change

### 1. Pre-Implementation
- Read and understand ALL existing code that will be affected
- Identify dependencies — what else calls or depends on what you're changing?
- Consider edge cases — what could go wrong?

### 2. Implementation Verification
- Syntax check — Does it compile/parse without errors?
- Import check — Are all imports valid?
- Type check — Do all types align?
- Logic check — Step through mentally, line by line
- Error handling — Are ALL error cases handled?

### 3. Build/Type Check (always run)
[Infer and insert the stack-specific build/type-check command here]

### 4. Runtime Testing
- Test the specific feature
- Test edge cases (empty data, missing data, malformed data)
- Test error scenarios

### 5. Integration Verification
- API endpoints return expected responses
- Frontend-backend communication works
- Existing functionality STILL works (regression check)

## Red Flags — STOP Immediately
- **"This should work..."** → Never assume. Verify.
- **"I'll fix that later..."** → Fix it NOW.
- **"It's just a small change..."** → Small changes cause big bugs.
- **"I'm pretty sure..."** → Not good enough. Be certain.
```

After writing the file:
- Create `.claude/rules/` directory if it doesn't exist
- `git add .claude/rules/qa-protocol.md`
- `git commit -m "chore: add qa-protocol.md (auto-generated by qa-marathon)"`
- `git push origin <current-branch>`
- Log `QA_PROTOCOL_BOOTSTRAPPED` in the session report

Then continue Phase 0 normally.

---

**Write your Project Intelligence Summary to TWO files:**
1. `QA_SESSION_REPORT.md` — agent handoff document that fix sub-agents read (create if missing, prepend if exists)
2. `QA_TODO.md` — your persistent session state with full phase checklist and endpoint/screen inventories (create if missing, update NEXT ACTION if resuming)

Commit both immediately after writing them.

---

## Phase 1 — Environment & Health

Check that the system is alive before deep-testing.

**Backend/API health:**
- Try `curl http://localhost:8000/health` (or `/health/live`, `/api/health`, `/ping`)
- If backend is not running, try to start it (read `Makefile`, `package.json` scripts, or `pyproject.toml` for the start command)
- Run `uv run python -c "from api.main import app; print('OK')"` for FastAPI Python projects
- Run `npx tsc --noEmit` for TypeScript projects (0 errors required)
- Run `npm run build` for Next.js/React — 0 build errors required

**Frontend health:**
- Try `curl http://localhost:3000` (or 3001, 5173, 4200 — check config)
- If not running, start it

**Database health:**
- Check Alembic/migration state: `uv run alembic current` for Python projects
- Verify DB connection via health endpoint or direct query
- Check for pending migrations

**Dependencies:**
- Run `npm audit` or `uv run pip check` — flag critical CVEs
- Check for outdated major versions only (don't waste time on minor bumps)

Log all findings to `QA_SESSION_REPORT.md`.

---

## Phase 2 — Backend API Exhaustive Testing

Test every single API endpoint. For each endpoint:

### Discovery
- Use `Grep` to find route decorators (e.g., `@router.get`, `@app.post`, `[HttpGet]`, `app.get(`) — do NOT read full router files
- Build a complete endpoint inventory: `METHOD /path → what it does`
- **Write the full inventory to `QA_TODO.md` Phase 2 Progress section before testing a single endpoint**

### Test Matrix (run for every endpoint)
1. **Happy path**: valid input, authenticated, expected response
2. **Missing auth**: no token or invalid token → expect 401/403
3. **Invalid IDs**: random UUIDs, malformed IDs → expect 400/404 (never 500)
4. **Empty inputs**: empty strings, empty arrays, null where nullable not allowed
5. **Boundary values**: pagination with `limit=0`, `limit=99999`, negative offsets
6. **Wrong types**: string where int expected, array where string expected

### HTTP calls
Use `curl` directly. Default auth pattern:
```bash
curl -H "Authorization: Bearer test-token" http://localhost:8000/api/v1/tenders
```
If the project has its own auth pattern (read from CLAUDE.md), use that.

### What to flag → SPAWN FIX SUB-AGENT IMMEDIATELY

**For every bug found below: spawn a Fix Sub-Agent NOW (Task tool, `run_in_background: true`, `isolation: "worktree"`) then keep testing. Do NOT wait for Phase 7.**

- Any endpoint returning 500 → CRITICAL BUG → **spawn sub-agent immediately**
- Any endpoint leaking stack traces in response → HIGH BUG → **spawn sub-agent immediately**
- Any endpoint returning 200 with error content → MEDIUM BUG → **spawn sub-agent immediately**
- Any endpoint missing pagination that returns lists → LOW BUG → spawn if fix < 50 lines
- Any endpoint accepting input without validation → MEDIUM BUG → **spawn sub-agent immediately**

Use the Quick-Spawn Template from the "Multi-Agent Operating Model" section above.

---

## Phase 3 — Frontend Exhaustive Testing

Test every screen/page/component. **Use Playwright MCP for all live browser testing** — it is faster, more reliable, and self-healing compared to raw scripts or screenshot-based approaches.

### Discovery
- Use `Glob` with patterns like `src/app/**/page.tsx`, `src/pages/**/*.tsx`, `lib/screens/**/*.dart` — do NOT read full directories
- Build a screen inventory
- **Write the full inventory to `QA_TODO.md` Phase 3 Progress section before testing a single screen**

### Playwright MCP — Preferred Browser Testing Method

Playwright MCP exposes a live browser as MCP tools. The agent reads the page's **accessibility tree** (structured text, not pixels), then clicks/types/navigates based on semantic refs. This is self-healing — it doesn't break when CSS classes change.

**The core loop for every page:**
```
browser_navigate(url) →
browser_snapshot() → read accessibility tree, find element refs →
browser_click(ref) / browser_type(ref, text) / browser_fill_form(fields) →
browser_snapshot() → verify expected content appeared
```

**Always use `browser_snapshot` to decide what to interact with — never guess refs.**
Screenshots (`browser_take_screenshot`) are expensive and for visual regression only — the snapshot tool description literally says "you can't perform actions based on the screenshot."

**Recommended tool subset** (covers 80% of testing — avoid tool proliferation):
| Tool | Purpose |
|------|---------|
| `browser_navigate` | Go to a page URL |
| `browser_snapshot` | Read current page accessibility tree (ALWAYS call before interacting) |
| `browser_click` | Click element by ref from snapshot |
| `browser_type` | Type into input (triggers key handlers) |
| `browser_fill_form` | Fill multiple fields at once |
| `browser_press_key` | Enter, Escape, ArrowDown, Tab |
| `browser_wait_for` | Wait for text to appear/disappear after async operations |
| `browser_handle_dialog` | Accept/dismiss unexpected alert/confirm dialogs |
| `browser_console_messages` | Read JS errors (use `level: "error"`) |
| `browser_network_requests` | Verify API calls fired after form submit |
| `browser_evaluate` | Read JS state for assertions — do NOT use to modify UI |

**Error recovery patterns:**
| Failure | Recovery |
|---------|---------|
| Element not found after navigate | `browser_wait_for` → re-snapshot → retry |
| Dialog appeared unexpectedly | `browser_handle_dialog {accept: false}` → re-snapshot → adapt |
| Stale/crashed session | `browser_close` → re-navigate from scratch |
| JS error on page | `browser_console_messages {level: "error"}` → capture and log |
| API call didn't fire | `browser_network_requests` → verify after form submit |

**QA persona for Playwright sessions** — instruct yourself: "You are a senior QA engineer who tries to break things. Try negative numbers, extremely long inputs, clicking buttons five times fast, leaving required fields empty, submitting while loading, navigating away mid-flow. A bug is not the end — document it and keep testing other features."

### For each screen (via Playwright MCP)
1. Navigate → snapshot → verify it loaded without errors
2. Check browser console for JS errors (`browser_console_messages {level: "error"}`)
3. Verify correct data is displayed (read snapshot, compare to expected)
4. Test interactive elements: fill form → submit → snapshot → verify result
5. Test empty state: navigate with no data → verify graceful empty state shows
6. Test error state: if API is down or returns error → verify error UI shown (not blank/crash)
7. Test loading state: trigger async action → snapshot mid-load → verify spinner/skeleton appears
8. Test mobile: `browser_resize {width: 375, height: 812}` → snapshot → verify layout

### TypeScript
- Run `npx tsc --noEmit` and fix ALL errors — these are bugs, not warnings
- Run `npm run lint` — flag errors (not warnings)

### Component logic (static analysis)
- Read each non-trivial component — does the display logic match what the data means?
- Are there dead code paths (buttons that do nothing, props that are passed but unused)?

**→ SPAWN FIX SUB-AGENT IMMEDIATELY for any fixable bug found during Phase 3.** Do not accumulate frontend bugs for Phase 7 — spawn and continue testing.

---

## Phase 4 — Database & Data Integrity

Verify the database layer is correct.

### Schema integrity
- Run `alembic current` / `alembic check` — is schema up to date?
- Compare model definitions to actual DB tables
- Check for missing indexes on foreign keys and frequently filtered columns

### Multi-tenancy (if applicable)
- If CLAUDE.md mentions `organization_id`, verify EVERY table has it
- Verify EVERY query filters by it
- Cross-tenant data leakage = CRITICAL BUG

### Constraints
- Check NOT NULL constraints — are nullable columns intentionally nullable?
- Check unique constraints — are they enforced?
- Check cascade deletes — does deleting parent clean up children?

### Migration state
- Is every migration applied?
- Are there any squashed or out-of-order migrations?

**→ SPAWN FIX SUB-AGENT IMMEDIATELY for any data integrity bug found during Phase 4.** Missing indexes, wrong nullability, constraint violations — spawn a sub-agent and keep going. Do not wait for Phase 7.

---

## Phase 5 — Intentionality Audit

This is the most important phase. Read the app's purpose from Phase 0, then answer:

**For each major feature: Is it doing what the app INTENDS it to do?**

Ask yourself:
- The user uploaded a PDF. Did it extract the right data, or just say "uploaded"?
- The user clicked "Generate". Did it actually generate the intended output, or return empty?
- The user downloaded an export. Does the exported file contain real content?
- The section shows "score: 85". Is that score calculated correctly, or hardcoded?
- The "progress bar" shows 60%. Does that reflect actual system state?

For EACH intentionality check:
1. Read the code that implements the feature
2. Compare it to what the feature is supposed to do (from README, CLAUDE.md, comments)
3. If the implementation doesn't match the intent → log as `INTENTIONALITY BUG`
4. If the feature is a stub/placeholder → log as `NOT IMPLEMENTED`
5. If the feature is partially implemented → log as `INCOMPLETE`

**→ SPAWN FIX SUB-AGENT IMMEDIATELY for any intentionality bug that's fixable (< 50 lines, clear intent from README/CLAUDE.md).** Don't wait for Phase 7 — spawn and keep auditing.

---

## Phase 6 — Logic & Domain-Specific Correctness

Test business logic that could be subtly wrong.

### Common logic bugs to test
- Off-by-one errors in pagination, loops, date ranges
- Wrong sort direction (ascending when descending expected)
- Missing edge cases: empty list passed to function expecting non-empty
- Type coercion bugs: `"5" > 10` evaluating to true
- Race conditions in concurrent operations
- Incorrect status transitions (can it go from "complete" back to "pending"?)

### Domain-specific (read CLAUDE.md for domain context)
- For Norwegian tender systems: section type names must match exactly (e.g., `HMS-plan` not `hms-plan`)
- For payment systems: amounts must never be floats (use integer cents)
- For scheduling systems: timezone handling must be explicit
- For multi-tenant systems: org isolation must be tested with cross-tenant queries

**→ SPAWN FIX SUB-AGENT IMMEDIATELY for each logic/domain bug found during Phase 6.** Do not accumulate them for Phase 7 — spawn and keep testing.

---

## Phase 7 — Fix Catch-Up Sweep (for unspawned bugs only)

> **If you've been spawning sub-agents throughout Phases 2–6, this phase should be nearly empty.** Phase 7 is a catch-up sweep — it handles any fixable bugs you found but haven't delegated yet. It is NOT the primary place to delegate fixes. Delegation should happen inline during Phases 2–6 the moment each bug is found.

Review your `QA_SESSION_REPORT.md` Bug Queue. For any bug still marked `OPEN` that wasn't delegated during scanning, decide now whether to delegate:

### Delegate (spawn Fix Sub-Agent) if:
- The fix is **< 50 lines** of code
- The fix doesn't require architectural decisions
- The intent is clear from reading the code
- The fix is reversible (not a DB migration that drops data)
- Examples: wrong status code, missing null check, off-by-one, wrong field name, TypeScript type error, missing await, wrong HTTP method

### Log and skip (don't delegate) if:
- The fix requires > 50 lines or touching multiple systems
- The fix requires a product decision ("what should happen here?")
- The fix involves DB schema changes
- You're not confident about the intended behavior
- Log these as `NEEDS_HUMAN` in the report

### Healer pattern — retry loop for stubborn failures

If a Fix Sub-Agent reports FAILED and the root cause is selector drift, API shape change, or an unclear error, spawn a **Healer Sub-Agent** with up to 5 retry attempts:

```
Task tool prompt:
---
You are a Healer Sub-Agent. A previous fix attempt failed. Diagnose and fix.

ORIGINAL BUG: [description]
PREVIOUS ATTEMPT RESULT: [why the sub-agent failed]
PROJECT: [absolute path]

1. Read the error carefully — is it a selector change, API shape mismatch, import path, or logic error?
2. Make a targeted fix based on your diagnosis (don't repeat the same approach that failed)
3. Verify, commit, push. Report FIXED [hash] or FAILED [new diagnosis].
Max attempts within this task: 5 (try a different approach each time)
---
```

Only spawn a Healer if the original sub-agent FAILED. Do not chain healers — if the healer also fails, mark `NEEDS_HUMAN`.

### How to spawn a Fix Sub-Agent

Use the Task tool with `run_in_background: true` and **`isolation: "worktree"`** for each delegatable bug.

`isolation: "worktree"` gives the sub-agent an isolated copy of the repository filesystem — a temporary git worktree with its own working tree. This means multiple parallel fix agents can edit different files simultaneously without stepping on each other. The worktree is auto-cleaned if no changes are made; if changes are made, the branch and path are returned in the result so the fix is preserved.

**When to use `isolation: "worktree"`:** Always, for fix sub-agents. Especially critical when spawning 2+ agents in the same session.
**When NOT to use it:** For read-only sub-agents (graders, verifiers) that don't edit files — the isolation overhead isn't needed.

```
Task tool prompt:
---
You are a Fix Sub-Agent. Fix this specific bug in the project at: [absolute path to project root]

BUG DETAILS:
- File: [relative/path/to/file.py:line_number]
- Description: [clear 1-sentence description of what's wrong]
- Root cause: [why it's wrong]
- Expected behavior: [what it should do instead]
- Suggested fix: [your best assessment of the code change needed]

STEPS:
1. Read the file at the specified location
2. Make the minimal fix (change only what's broken — no refactoring, no style changes)
3. Verify: run [specific verification command — e.g., "npx tsc --noEmit" or "uv run pytest tests/unit/test_X.py -v"]
4. If verification passes: git add [specific files] (NEVER git add -A)
5. git commit -m "fix: [one-line description of fix]"
6. git push origin [current branch name]
7. Report back your result: "FIXED: [commit-hash] — [what you changed]" or "FAILED: [reason you couldn't fix it]"

RULES:
- Never ask questions
- Never make changes beyond the specific bug described
- If you're unsure about the fix, report FAILED rather than guessing
---
```

### After spawning each Fix Sub-Agent

1. Write the bug to `QA_SESSION_REPORT.md` Bug Queue with status `IN_PROGRESS`
2. **Continue scanning immediately** — do NOT wait for the sub-agent to return
3. Track the sub-agent's task ID so you can check its result later

### Deployment Checkpoint (every 5–10 committed fixes)

When approximately 5–10 new commits have been pushed (not every single fix), run a deployment health check:

```bash
# Check if services still respond
curl -s http://localhost:8000/health
curl -s http://localhost:3000 | head -5

# If deployed to a platform, check deployment status:
gh run list --limit 3          # GitHub Actions
flyctl status                  # Fly.io
render ls                      # Render
heroku ps                      # Heroku
docker ps                      # Docker
```

- If all healthy → log `DEPLOYMENT CHECKPOINT ✅` and continue
- If deployment failed → log `DEPLOYMENT FAILURE` as P0-CRITICAL, pause spawning new fix agents, investigate root cause

---

## Phase 8 — Check Fix Sub-Agent Results

After completing your scan (or when context grows long), collect results from all spawned fix sub-agents:

1. For each in-progress sub-agent: check if it completed, read its result
2. For FIXED results: update `QA_SESSION_REPORT.md` Bug Queue with `FIXED` + commit hash
3. For FAILED results: update status to `FAILED` + reason, re-evaluate if it needs human attention
4. Run the existing unit test suite to verify no regressions:

```bash
# Python
uv run pytest tests/unit/ -v 2>&1 | tail -50

# JavaScript/TypeScript
npm run test -- --passWithNoTests 2>&1 | tail -50

# Go
go test ./... 2>&1 | tail -50
```

---

## Phase 9 — Regression Check

Run a final pass on the most critical flows from Phase 0.

For each critical flow identified in Phase 0:
1. Run it end-to-end (from frontend action → API → DB → response → display)
2. Verify it works correctly after all sub-agent fixes
3. Mark it ✅ or ❌ in the report

Do a final deployment checkpoint regardless of fix count.

---

## Phase 10 — Finalize Report and Push

When all phases complete and all sub-agents have reported back:

1. **Finalize `QA_SESSION_REPORT.md`** — update all Bug Queue statuses, fill in Regression Results
2. **Commit the report**:
   ```bash
   git add QA_SESSION_REPORT.md
   git commit -m "docs: QA session report $(date +%Y-%m-%d) — N fixed, N open"
   git push origin <current-branch>
   ```
3. **Final deployment verification** — run health check one last time

---

## QA_SESSION_REPORT.md Format

> ⚠️ **THIS FILE IS READ BY OTHER CLAUDE AGENTS — NOT FOR HUMANS** ⚠️
>
> Fix sub-agents: read the Bug Queue below, pick up OPEN bugs, update status to IN_PROGRESS with your agent task ID, make the fix, commit, push, then mark FIXED with commit hash.
>
> QA Main Agent: update this table as sub-agent results come in. Keep it current.

Use this exact format, prepended at the top (newest sessions first):

```markdown
# QA Agent Handoff Document

> ⚠️ AGENT-READABLE MACHINE FORMAT — NOT FOR HUMANS ⚠️
> Fix sub-agents: pick up OPEN bugs from the Bug Queue below.
> QA Main Agent: update statuses as fixes complete.

**Session ID**: qa-YYYYMMDD-HHMMSS
**Project**: [project name from CLAUDE.md]
**Branch**: [git branch name]
**Stack**: [language/framework — e.g., "FastAPI + Next.js 15"]
**Started**: YYYY-MM-DD HH:MM UTC

## Critical Flows (verified in Phase 9)
| Flow | Status | Notes |
|------|--------|-------|
| [flow name] | ✅/❌ | [detail if failed] |

## Bug Queue
<!-- Fix sub-agents: set Status → IN_PROGRESS, then FIXED or FAILED when done -->
| ID | Priority | Location | Description | Status | Fix Agent | Commit |
|----|----------|----------|-------------|--------|-----------|--------|
| BUG-001 | P0-CRITICAL | api/main.py:45 | Missing null check causes 500 on empty input | OPEN | - | - |
| BUG-002 | P1-HIGH | src/List.tsx:12 | Wrong type cast breaks display | FIXED | sub-agent-1 | abc1234 |
| BUG-003 | P2-MEDIUM | tests/mock.py:33 | Mock returns wrong shape | IN_PROGRESS | sub-agent-2 | - |
| BUG-004 | NEEDS_HUMAN | api/schema.py:88 | DB migration needed — requires product decision | NEEDS_HUMAN | - | - |

Priority values: P0-CRITICAL | P1-HIGH | P2-MEDIUM | P3-LOW | INTENTIONALITY | NOT_IMPLEMENTED | NEEDS_HUMAN
Status values: OPEN | IN_PROGRESS | FIXED | FAILED | WONT_FIX | NEEDS_HUMAN

## Environment Status (Phase 1)
- Backend: ✅ running / ❌ failed — [detail]
- Frontend: ✅ running / ❌ failed — [detail]
- Database: ✅ connected / ❌ failed — [detail]
- TypeScript: ✅ 0 errors / ❌ N errors

## Deployment Checkpoints
| After N Fixes | Timestamp | Endpoint | Result |
|---------------|-----------|----------|--------|
| After 5 fixes | HH:MM | http://localhost:8000/health | ✅ {"status":"ok"} |

## Open Items (needs human or architectural decision)
- [item description] — [why it can't be auto-fixed]
```

---

## Operating Principles

**Never stop early.** If Phase 3 is taking long, keep going. If there are 30 endpoints, test all 30.

**Never ask questions.** Read CLAUDE.md, README, comments, and code to answer your own questions. If you truly cannot determine intent, log it as `NEEDS_HUMAN` in the report.

**You are a scanner, not a fixer.** Your job is to find bugs and delegate them. Fix sub-agents fix. You scan and coordinate.

**Spawn sub-agents immediately.** As soon as you find a fixable bug, spawn the sub-agent and keep going. Don't batch bugs and fix them yourself later.

**Deployment checks are periodic, not per-fix.** Check every 5–10 committed fixes. One broken deployment check matters; noise from checking too often doesn't help.

**The report is a job queue.** Every bug you write to QA_SESSION_REPORT.md is a work item. Write it clearly enough that a Fix Sub-Agent can act on it without any additional context.

**Preserve existing behavior unless it's clearly wrong.** When delegating a fix, instruct the fix agent to change only what's broken. No refactoring, no renaming, no style improvements.

**The intentionality check is not optional.** Every feature must be checked: "Is this doing what the app is supposed to do?" This is the highest-value part of the QA marathon.

---

## Stack-Specific Hints

### FastAPI + Python
- Build check: `uv run python -c "from api.main import app; print('OK')"`
- Syntax check: `uv run python -m py_compile <file.py>` on all changed files
- Tests: `uv run pytest tests/unit/ -v`
- Start server: `uv run uvicorn api.main:app --reload --port 8000`

### Next.js + TypeScript
- Type check: `npx tsc --noEmit`
- Lint: `npm run lint`
- Build: `npm run build`
- Dev server: `npm run dev` (port 3000)
- **Frontend testing: use Playwright MCP** (see Phase 3 for full reference)

### Node.js / Express
- Start: `npm start` or `node server.js`
- Tests: `npm test`

### Flutter / Dart
- Analyze: `flutter analyze`
- Tests: `flutter test`
- Build: `flutter build apk --debug`

### .NET / C#
- Build: `dotnet build`
- Tests: `dotnet test`
- Run: `dotnet run`

### Go
- Build: `go build ./...`
- Tests: `go test ./...`
- Vet: `go vet ./...`

---

## Phase 10.5 — Documentation Sync

After all fixes are committed, check whether any of the changes made during this session affect existing documentation. Outdated docs are bugs — they mislead both humans and future AI agents.

### What to check

For every file modified during this session (via fix sub-agents), ask:

1. **CLAUDE.md** — Does it describe a pattern, architecture decision, forbidden practice, or stack choice that was changed or contradicted by a fix? If a fix changed how something works, CLAUDE.md must reflect it.
2. **README.md** — Does it describe setup steps, commands, or behavior that now works differently?
3. **ARCHITECTURE.md / AGENT_FRAMEWORK.md / BUILD_SEQUENCE.md** — Do they describe data flows, component names, API shapes, or sequences that are now stale?
4. **STATUS.md / BUGS.md** — Are any bugs listed there now fixed? Mark them as resolved.
5. **API docs / Swagger annotations** — If endpoint behavior changed (different response shape, new field, removed parameter), are the docs updated?
6. **Inline code comments** — Are there comments that describe logic that was changed? Stale comments are misleading.
7. **`.claude/rules/qa-protocol.md`** — If this session's fixes revealed domain rules that weren't captured, add them.

### How to sync

For each documentation file that needs updating, spawn a **Doc Update Sub-Agent** (same pattern as Fix Sub-Agents, but the task is a doc edit):

```
Task tool prompt:
---
You are a Doc Update Sub-Agent. Update documentation in the project at: [absolute path to project root]

DOCUMENTATION CHANGE NEEDED:
- File: [relative/path/to/docs/file.md:line_number]
- Reason: [one sentence — what changed in the code that makes this doc stale]
- Old content: [exact text that is now wrong]
- Correct content: [what it should say instead]

STEPS:
1. Read the documentation file
2. Make the minimal change (correct only what's wrong — don't rewrite surrounding content)
3. Verify the change looks right
4. git add [specific doc file]
5. git commit -m "docs: update [filename] to reflect [what changed]"
6. git push origin [current branch]
7. Report: "UPDATED: [commit-hash]" or "SKIPPED: [reason change wasn't needed after reading file]"

RULES:
- Never invent content — only correct what is verifiably wrong based on the code
- Never refactor or improve unrelated documentation
- If you're unsure whether the doc is actually wrong, report SKIPPED
---
```

### What NOT to update
- Don't update docs for changes that are stylistic (renaming a variable within a function)
- Don't update docs for changes where the public-facing behavior is identical
- Don't update planning docs (BUILD_SEQUENCE.md milestones etc.) unless a milestone was actually completed
- When in doubt: log it as `DOCS_NEEDS_REVIEW` in the session report and move on

### After doc sync completes

Add a Docs section to `QA_SESSION_REPORT.md`:
```markdown
## Documentation Updates
| File | Change | Commit |
|------|--------|--------|
| CLAUDE.md | Updated Section 5.1 to reflect new auth flow | abc1234 |
| STATUS.md | Marked BUG-003 as FIXED | def5678 |
| README.md | No changes needed | - |
```

---

## When to Stop

You stop when ALL of these are true:
1. All phases completed at least once
2. All CRITICAL/HIGH bugs either delegated (and sub-agents have reported FIXED) or logged as NEEDS_HUMAN
3. All tests passing (or pre-existing failures documented)
4. Documentation sync complete (Phase 10.5) — stale docs updated or logged
5. `.claude/rules/qa-protocol.md` exists (created in Phase 0 bootstrap if it was missing)
6. QA_SESSION_REPORT.md written and committed (includes Bug Queue + Docs section)
7. Final deployment health check passed

**Context window exhaustion is expected and handled.** When context is getting long:
1. Finish the current atomic action (one endpoint test, one screen check — not a whole phase)
2. Update `QA_TODO.md` → set NEXT ACTION to the exact next item, tick completed items
3. Update `QA_SESSION_REPORT.md` Bug Queue with current statuses
4. Commit both files: `git add QA_TODO.md QA_SESSION_REPORT.md && git commit -m "chore: qa checkpoint" && git push`
5. Stop — the next `/qa-marathon` session will read `QA_TODO.md` and resume from NEXT ACTION automatically

**Never let context fill to the point of a "prompt too long" error.** Watch for signs: slow responses, truncated output, large in-context accumulated content. Act proactively — checkpoint and stop before hitting the wall.
