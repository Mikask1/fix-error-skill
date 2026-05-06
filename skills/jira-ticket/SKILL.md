---
name: jira-ticket
description: Creates or reads a Jira ticket. Use when the user says "create a Jira ticket", "log this in Jira", "open a ticket", "read this ticket", "get ticket details", "what does JIRA-XXX say", or when a workflow needs to create or fetch a ticket.
---

# Jira Ticket 

Creates or reads Jira tickets following the user issue tracking conventions.

## Jira Projects

| Project Key | Use case |
|-------------|----------|
| `ENG` | Engineering bugs, tech debt, infra |
| `FEAT` | New product features |
| `INC` | Production incidents |
| `SEC` | Security issues |
| `OPS` | DevOps / platform / tooling |

Default to `ENG` if the context is a bug or error fix. Use `INC` for production incidents detected from logs.

---

## Ticket Types

| Type | When to use |
|------|-------------|
| `Bug` | Something is broken in production or staging |
| `Task` | Work item that isn't a bug or feature |
| `Story` | User-facing feature or improvement |
| `Incident` | Active production outage or data issue |

---

## Read a Ticket

### Fetch ticket details

Use the Jira MCP tool if available:
```
mcp__jira__get_issue
  issue_key: <TICKET_ID>   (e.g. ENG-4821)
```

If no Jira MCP is configured, ask the user to paste the ticket content.

### Output format

Present the ticket as:

```
## <TICKET_ID>: <Summary>

- **Type**: <Bug|Task|Story|Incident>
- **Priority**: <Critical|High|Medium|Low>
- **Status**: <To Do|In Progress|In Review|Done>
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

Return all fields to the caller so they can be used in fix plans, branch names, and PR titles.

---

## Create a Ticket

### 1. Gather ticket details

Collect (or infer from context):
- **Summary** — one-line title describing the issue
- **Project** — from the table above
- **Type** — Bug / Task / Story / Incident
- **Priority** — Critical / High / Medium / Low
- **Description** — see template below
- **Labels** — service name(s) affected (e.g. `payment-service`, `auth-service`)
- **Component** — the system area (e.g. `API`, `Database`, `Auth`, `Payments`)

If creating from an incident (e.g. from `fix-error` workflow), use the error details directly.

### 2. Determine priority

| Priority | Criteria |
|----------|----------|
| `Critical` | Production down, data loss, security breach |
| `High` | Major feature broken, significant user impact |
| `Medium` | Degraded functionality, workaround exists |
| `Low` | Minor issue, cosmetic, or low-traffic path |

### 3. Write the ticket description

Use this template:

```markdown
## Summary
<!-- One sentence: what is broken or needed -->

## Environment
- Service: <service-name>
- Severity: <ERROR / CRITICAL / WARN>
- Timestamp: <when it occurred>

## Steps to Reproduce (for bugs)
1.
2.
3.

## Expected Behavior
<!-- What should happen -->

## Actual Behavior
<!-- What is happening instead -->

## Error Details
```
<ERROR_TYPE>: <ERROR_MESSAGE>

<STACK_TRACE first 10 lines>
```

## Proposed Fix (optional)
<!-- Brief description of likely fix approach -->

## Acceptance Criteria
- [ ] Error no longer occurs in production
- [ ] Tests cover the fixed behavior
- [ ] No regressions in related flows
```

Fill in every section with real content. For incidents triggered by `fix-error`, paste the log analysis output directly into "Error Details".

### 4. Create the ticket

Use the Jira MCP tool if available:
```
mcp__jira__create_issue
  project: <PROJECT_KEY>
  summary: <one-line title>
  issue_type: <Bug|Task|Story|Incident>
  priority: <Critical|High|Medium|Low>
  description: <filled template>
  labels: [<service-name>]
  components: [<component>]
```

If no Jira MCP is configured, output the complete ticket as a formatted markdown block so the user can paste it into Jira manually.

### 5. Return the ticket ID

Return the created ticket ID (e.g. `ENG-4821`) to the caller so it can be referenced in branches and PR titles.

---

## Critical Rules

- **NEVER create a ticket without a meaningful summary** — ask the user if unclear
- **NEVER set priority to Critical/High without justification** — document the criteria met
- **For security issues (`SEC`), do not include exploit details in the description** — use private Jira fields or a separate secure channel
- **Always fill the Acceptance Criteria section** — tickets without it cannot be closed

