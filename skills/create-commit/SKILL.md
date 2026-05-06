---
name: create-commit
description: Stages and commits changes using NexaCore conventional commit format. Splits into multiple commits for separation of concern. Use when the user says "commit this", "create a commit", or "stage and commit".
---

# Create Commit (NexaCore Standards)

## Workflow

1. **Audit** — run `git status` and `git diff`. Read every changed, added, and untracked file before touching anything.
2. **Classify** — for each file decide: stage it, stage it in a separate commit, or leave it out. Do not assume a file belongs just because it changed.
3. **Stage by name** — `git add src/foo.ts`. Never `git add .`, `-A`, or `-u`.
4. **Verify** — run `git diff --staged` and read every line. Unstage anything unexpected with `git restore --staged <file>`.
5. **Commit** — one concern per commit. If changes span multiple concerns, make multiple commits.

## Never stage

- Secrets — `.env`, `.env.*`, API keys, private keys
- Tmp / generated — `*.tmp`, `*.log`, `dist/`, `build/`, `*.map`
- Debug leftovers — `console.log`, commented-out investigation code
- IDE artifacts — `.vscode/`, `.idea/`, `*.swp`
- Lock file changes unless dependencies were intentionally updated
- Files you cannot explain — if `git status` shows an untracked file you don't recognise, investigate before proceeding

## Commit format

```
type(scope): brief description
```

Optional body explains **why**:

```
fix(payments): resolve retry loop on expired sessions

Token expiry was not checked before retrying, causing 401 loops.
```

| Prefix | Use case |
|--------|----------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Restructure without behavior change |
| `test` | Adding or updating tests |
| `chore` | Maintenance, deps, build config |
| `hotfix` | Emergency production fix |
| `release` | Version bump or release prep |

## Author
NexaCore Engineering Platform Team
