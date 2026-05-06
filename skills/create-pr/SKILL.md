---
name: create-pr
description: Creates a PR following NexaCore company standards — feature branch, Jira ticket reference, 2 required reviewers, squash merge. Use when the user says "create a PR", "open a PR", "ship this", or "make a PR".
---

# Create PR (NexaCore Standards)

Opens a pull request following NexaCore's engineering standards. Assumes code is already committed — if not, run `/create-commit` first.

## NexaCore PR Standards

| Rule | Requirement |
|------|------------|
| Title format | `[JIRA-XXX] type(scope): description` — use `[NO-TICKET]` if no Jira ticket |
| Base branch | `develop` (never `main` or `production`) |
| Merge strategy | Squash merge only |
| Reviewers | Minimum 2 approvals before merge |
| CI | All checks must pass before merge |
| Branch cleanup | Delete branch after merge |
| PR size | < 400 lines changed; if larger, add justification |
| UI changes | Screenshots required in PR body |

---

## Workflow

### 1. Ensure a feature branch

Check current branch: `git branch --show-current`

If on `develop`, `main`, or `production` — create a branch first:

| Prefix | Use case |
|--------|----------|
| `feat/` | New feature |
| `fix/` | Bug fix |
| `refactor/` | Code restructure |
| `test/` | Tests only |
| `chore/` | Maintenance / tooling |

Example: `fix/ENG-4821-null-payment-amount`

### 2. Push

```
git push -u origin <branch>
```

### 3. Build the PR body

Check for `.github/pull_request_template.md` — fill it in if present. Otherwise use:

```markdown
## [JIRA-XXX] Summary
<!-- One-sentence description of what this PR does -->

## Root Cause / Motivation
<!-- Why is this change needed? What was broken or missing? -->

## Changes
<!-- Bullet list of files modified and why -->
-

## Testing
- [ ] Unit tests added/updated
- [ ] Manual testing performed
- [ ] No regressions in existing tests

## Checklist
- [ ] PR title follows `[JIRA-XXX] type(scope): description` format
- [ ] Branch is NOT `develop`, `main`, or `production`
- [ ] No secrets committed
- [ ] PR size < 400 lines (or justification provided)
- [ ] Screenshots attached (if UI change)
```

Fill in every section with real content — no placeholder comments left behind.

### 4. Create the PR

```
gh pr create --base develop --title "[JIRA-XXX] type(scope): description" --body "$(cat <<'EOF'
<filled-in body>
EOF
)"
```

Use `[NO-TICKET]` in the title if no Jira ticket exists.

### 5. Assign and request reviewers

```
gh pr edit <pr-number> --add-assignee @me
gh pr edit <pr-number> --add-reviewer <handle1>,<handle2>
```

If reviewers are unknown, leave a comment:
```
gh pr comment <pr-number> --body "⚠️ Please add 2 reviewers before merging (NexaCore policy: 2 approvals required)."
```

### 6. Return the PR URL

---

## Critical Rules

- **NEVER push directly to `develop`, `main`, or `production`**
- **NEVER stage `.env`, credentials, or API keys**
- **ALWAYS request at least 2 reviewers** (or leave the warning comment)
- **Base branch is `develop`** unless user specifies otherwise
- **PR body must be fully filled in** — no open placeholder comments
- **Remind user to squash merge** — not merge commit or rebase

## Author
NexaCore Engineering Platform Team
