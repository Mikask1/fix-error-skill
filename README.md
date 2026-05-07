# fix-error skill

End-to-end incident response: read Jira ticket -> analyze AWS logs -> clone repo -> explore code -> check DB -> plan -> TDD -> fix -> PR.

---

## Installation

```bash
claude plugin marketplace add Mikask1/fix-error-skill
claude plugin install fix-error@fix-error
```

---

## Prerequisites

- [Jira MCP](#step-1----set-up-the-jira-mcp)
- [AWS CLI](#step-2----set-up-the-aws-cli)
- [GitHub CLI](#step-3----set-up-github-cli)
- [Service mapping](#step-4----edit-the-service-mapping)

---

## Quick Start

1. Install the plugin (see [Installation](#installation))
2. Complete the [Prerequisites](#prerequisites) setup
3. Open Claude Code and type: `fix this error` or `fix ENG-4821`
4. The skill runs all 10 steps automatically and returns a PR URL when done

---

## Full Tutorial

### Step 1 — Set up the Jira MCP

The Jira MCP lets Claude read and create Jira tickets directly without copy-pasting.

**Install:**

```bash
npx @atlassian/atlassian-mcp-server
```

Source: https://github.com/atlassian/atlassian-mcp-server

**Configure** — add to your `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@atlassian/atlassian-mcp-server"],
      "env": {
        "ATLASSIAN_API_TOKEN": "<your-api-token>",
        "ATLASSIAN_EMAIL": "<your-atlassian-email>",
        "ATLASSIAN_DOMAIN": "<your-domain>.atlassian.net"
      }
    }
  }
}
```

**Get your API token:**
1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click **Create API token**, name it `claude-code`, copy the value

**Verify** — ask Claude `read ticket ENG-1`. If it returns ticket details, you're set.

---

### Step 2 — Set up the AWS CLI

```bash
# macOS
brew install awscli

# Windows: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

aws configure              # Enter: Access Key ID, Secret, region, output format
aws sts get-caller-identity  # Verify
```

If your team uses SSO: `aws configure sso`

---

### Step 3 — Set up GitHub CLI

```bash
# macOS:   brew install gh
# Windows: winget install GitHub.cli

gh auth login
gh auth status  # Verify
```

---

### Step 4 — Edit the service mapping

Open [`skills/fix-error/SERVICE_MAPPING.md`](skills/fix-error/SERVICE_MAPPING.md) and replace each `<your-org>` placeholder with your actual GitHub organisation and repo names:

```markdown
| `payment-service` | `https://github.com/your-org/payment-service` |
```

Add or remove rows to match your services. If a service isn't listed when the skill runs, Claude will halt and ask for the repo URL.

---

### Step 5 — Run the skill

```
fix this error
```

Or with a ticket ID:

```
fix ENG-4821
```

What Claude does automatically:

| Step | Action |
|------|--------|
| 1 | Reads your Jira ticket via MCP |
| 2 | Pulls AWS CloudWatch logs and extracts the error |
| 3 | Maps the service name to its GitHub repo and clones it |
| 4 | Reads `CLAUDE.md`, `docs/`, and source files |
| 5 | Checks the database (read-only) for anomalies |
| 6 | Writes `FIX_PLAN.md` with root cause and files to change |
| 7 | Writes failing tests (TDD) |
| 8 | Implements the fix |
| 9 | Runs tests — all must pass, no regressions |
| 10 | Opens a PR, assigns to you |

---

## Service -> Repo Mapping

> **Edit [`skills/fix-error/SERVICE_MAPPING.md`](skills/fix-error/SERVICE_MAPPING.md) to match your actual services and repos before use.**

| Service | Repo |
|---------|------|
| `user-service` | `github.com/<your-org>/user-service` |
| `payment-service` | `github.com/<your-org>/payment-service` |
| `notification-service` | `github.com/<your-org>/notification-service` |
| `auth-service` | `github.com/<your-org>/auth-service` |
| `api-gateway` | `github.com/<your-org>/api-gateway` |
| `order-service` | `github.com/<your-org>/order-service` |
| `inventory-service` | `github.com/<your-org>/inventory-service` |

If your service isn't listed, Claude will ask for the repo URL.
