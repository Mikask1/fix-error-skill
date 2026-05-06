---
name: jira-ticket
description: Creates or reads a Jira ticket. Use when the user says "create a Jira ticket", "log this in Jira", "open a ticket", "read this ticket", "get ticket details", "what does JIRA-XXX say", or when a workflow needs to create or fetch a ticket.
---

# Jira Ticket

Creates or reads Jira tickets via the Atlassian MCP server.

## MCP Setup (required)

This skill requires the Atlassian MCP server. If not yet configured, tell the user:

> "The Jira MCP is not configured. See [`JIRA_MCP_SETUP.md`](JIRA_MCP_SETUP.md) for setup instructions, or paste the ticket content manually."

All Jira operations use fully-qualified MCP tool names: `mcp__atlassian__<tool>`.

---

## Projects

| Key | Use case |
|-----|----------|
| `ENG` | Engineering bugs, tech debt, infra |
| `FEAT` | New product features |
| `INC` | Production incidents |
| `SEC` | Security issues |
| `OPS` | DevOps / platform / tooling |

Default to `ENG` for bugs. Use `INC` for production incidents from logs.

## Ticket Types

| Type | When to use |
|------|-------------|
| `Bug` | Something broken in production or staging |
| `Task` | Work item that isn't a bug or feature |
| `Story` | User-facing feature or improvement |
| `Incident` | Active production outage or data issue |

---

## Read a Ticket

```
mcp__atlassian__get_issue
  issue_key: <TICKET_ID>
```

Present the result as:

```
## <TICKET_ID>: <Summary>

- **Type**: <type>
- **Priority**: <priority>
- **Status**: <status>
- **Reporter**: <name>
- **Assignee**: <name or unassigned>
- **Labels**: <labels>
- **Component**: <component>

### Description
<full description>

### Acceptance Criteria
<criteria>

### Comments (latest 3)
<comments>
```

Return all fields to the caller for use in fix plans, branch names, and PR titles.

---

## Create a Ticket

### 1. Gather details

Collect or infer from context:
- **Summary** — one-line title
- **Project** — from table above
- **Type** — Bug / Task / Story / Incident
- **Priority** — see criteria below
- **Labels** — affected service names (e.g. `payment-service`)
- **Component** — system area (e.g. `API`, `Database`, `Auth`)

### 2. Priority criteria

| Priority | Criteria |
|----------|----------|
| `Critical` | Production down, data loss, security breach |
| `High` | Major feature broken, significant user impact |
| `Medium` | Degraded functionality, workaround exists |
| `Low` | Minor, cosmetic, or low-traffic path |

### 3. Write the description

```markdown
## Summary
<!-- One sentence: what is broken or needed -->

## Environment
- Service: <service-name>
- Severity: <ERROR / CRITICAL / WARN>
- Timestamp: <when it occurred>

## Steps to Reproduce
1.
2.
3.

## Expected Behavior
<!-- What should happen -->

## Actual Behavior
<!-- What is happening instead -->

## Error Details
<ERROR_TYPE>: <ERROR_MESSAGE>

<STACK_TRACE first 10 lines>

## Proposed Fix (optional)
<!-- Brief description of likely fix approach -->

## Acceptance Criteria
- [ ] Error no longer occurs in production
- [ ] Tests cover the fixed behavior
- [ ] No regressions in related flows
```

Fill every section. For incidents from `fix-error`, paste the log analysis output into "Error Details".

### 4. Create via MCP

```
mcp__atlassian__create_issue
  project: <PROJECT_KEY>
  summary: <one-line title>
  issue_type: <Bug|Task|Story|Incident>
  priority: <Critical|High|Medium|Low>
  description: <filled template>
  labels: [<service-name>]
  components: [<component>]
```

If MCP is unavailable, output the complete ticket as a markdown block for manual entry.

### 5. Return the ticket ID

Return the created ID (e.g. `ENG-4821`) so it can be used in branch names and PR titles.

---

## Critical Rules

- **NEVER create a ticket without a meaningful summary** — ask if unclear
- **NEVER set Critical/High without justification** — state which criteria was met
- **For `SEC` tickets, omit exploit details** — use private Jira fields or a secure channel
- **Always fill Acceptance Criteria** — tickets without it cannot be closed

