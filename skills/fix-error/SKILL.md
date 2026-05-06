---
name: fix-error
description: Command: "fix this", "fix this error". This skill reads the Jira ticket, analyzes AWS logs, clones the affected service repo, plans a fix using TDD, implements it, and opens a PR. Use when a production error needs diagnosing and fixing from log errors.
---

# Fix Error

Flow: Jira ticket → logs → root cause → TDD → fix → PR.

---

## Workflow

### Step 1 — Read the Jira ticket

Ask the user for the Jira ticket ID if not already provided. Invoke the `/jira-ticket` skill in **read** mode to fetch the ticket details. Extract and retain:

- `JIRA_TICKET_ID` (e.g. `ENG-4821`) — used in branch name and PR title
- Ticket summary and description — used to understand the reported problem
- Acceptance criteria — used to guide the fix plan and test strategy

If no Jira ticket exists yet, ask the user whether to create one first (via `/jira-ticket` create) or proceed without one (branch and PR will use `[NO-TICKET]`).

---

### Step 2 — Analyze AWS logs

Launch the `aws-log-analyzer` agent. Extract and retain:

- `SERVICE_NAME` — service producing the error
- `ERROR_TYPE` — exception class or error code
- `ERROR_MESSAGE` — full error message
- `STACK_TRACE` — first 10 lines of the stack trace
- `TIMESTAMP` — when the error occurred
- `SEVERITY` — ERROR / CRITICAL / WARN

---

### Step 3 — Map service to repo and clone

1. Look up `SERVICE_NAME` in [Service Mapping](SERVICE_MAPPING.md) to find the corresponding repo URL.
   - If not found: **halt and ask the user for the repo URL**.
2. Clone the repo:
   ```
   git clone <repo-url> /tmp/fix-error/<SERVICE_NAME>
   ```
3. Work inside `/tmp/fix-error/<SERVICE_NAME>` for all remaining steps.

---

### Step 4 — Explore the codebase

Read in priority order:

1. `CLAUDE.md` (may be empty — note and continue)
2. `docs/` recursively (may be empty — note and continue)
3. Fallback: `README.md`, `package.json` / `pyproject.toml` / `pom.xml`, entry-point source files, files referenced in `STACK_TRACE`

Goal: understand the service's purpose, tech stack, entry points, and modules involved in the error.

---

### Step 5 — Check the database (Optional)

Use the `/postgresql` skill (read-only) to investigate the database state relevant to the error:

- Inspect the schema of tables referenced in the error or stack trace
- Query for data anomalies that may have triggered the bug (e.g. unexpected nulls, constraint violations, missing rows)
- Check for recent schema changes if the error suggests a structural mismatch

Record any suspicious findings in `FIX_PLAN.md` under a "Database Observations" section. This step is informational only — **do not modify any data**.

---

### Step 6 — Create a fix plan

Write `FIX_PLAN.md` at `/tmp/fix-error/<SERVICE_NAME>/FIX_PLAN.md`:

```markdown
# Fix Plan: <SERVICE_NAME> (<JIRA_TICKET_ID>)

## Error Summary
- Service: <SERVICE_NAME>
- Jira: <JIRA_TICKET_ID>
- Error: <ERROR_TYPE>: <ERROR_MESSAGE>
- Timestamp: <TIMESTAMP>

## Root Cause Hypothesis
<!-- Why this error is likely occurring based on the code -->

## Files to Modify
<!-- Exact file paths and the changes needed in each -->

## Test Strategy
<!-- Behaviors to assert to prove the fix works -->
```

---

### Step 7 — Write failing tests (TDD)

Launch the `unittest-writer` agent with:

- Contents of `FIX_PLAN.md`
- Source files listed under "Files to Modify"
- Instruction: **"Write tests asserting correct post-fix behavior. Tests must fail against the current unfixed code. Do not modify source files."**

After tests are written:
1. Run the test suite — new tests must **fail**
2. Existing tests must still pass

If new tests pass before the fix, they are wrong — ask the agent to revise.

---

### Step 8 — Implement the fix

Follow `FIX_PLAN.md` exactly:
- Modify only the listed files
- Minimal change that resolves the root cause
- No refactoring, cleanup, or unrelated edits

---

### Step 9 — Evaluate with tests

Run the full test suite:
- All Step 6 tests must now **pass**
- No pre-existing tests may regress

If tests fail: diagnose → adjust fix → re-run. **Maximum 3 iterations.** If still failing after 3 attempts, halt and report state to the user.

---

### Step 10 — Open a PR

Invoke `/create-pr` with:

- **Branch:** `fix/<JIRA_TICKET_ID>-<short-description>` (e.g. `fix/ENG-4821-null-payment-amount`)
- **PR title:** `[<JIRA_TICKET_ID>] fix(<SERVICE_NAME>): <one-line description>`
- **PR body context:** error block from Step 2, root cause from `FIX_PLAN.md`, files changed, tests added

Return the PR URL to the user.

---

## Critical Rules

- **NEVER skip Step 7** — TDD requires tests written and confirmed failing before the fix
- **NEVER modify files outside `/tmp/fix-error/<SERVICE_NAME>`**
- **NEVER commit directly to `main` or `develop`** — always use a `fix/` branch
- **If service not in SERVICE_MAPPING.md, halt and ask** — never guess the repo URL
- **If tests still fail after 3 iterations, halt** — do not open a PR with failing tests
- **Never hardcode secrets** — ask the user if credentials are needed

