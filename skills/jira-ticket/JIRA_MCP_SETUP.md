# Jira MCP Setup

Connects Claude to Jira via the Atlassian MCP server so `jira-ticket` can read and create tickets without copy-pasting.

## 1. Install the server

```bash
npx @atlassian/atlassian-mcp-server
```

Source: https://github.com/atlassian/atlassian-mcp-server

## 2. Get an API token

1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click **Create API token**
3. Name it (e.g. `claude-code`) and copy the value

## 3. Add to Claude Code settings

In `~/.claude/settings.json`:

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

## 4. Verify

Ask Claude: `read ticket ENG-1`

If it returns ticket details, the MCP is working.
