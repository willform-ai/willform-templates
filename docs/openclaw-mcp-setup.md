# Connect Your AI Coding Agent to Willform MCP

Deploy containers, manage domains, check billing — 42 MCP tools, directly from your IDE or CLI agent.

## Overview

Willform exposes a Streamable HTTP MCP server at `https://agent.willform.ai/api/mcp`. All major AI coding agents can connect to it natively.

## Prerequisites

| Item | How to get it |
|------|---------------|
| Willform API Key | Dashboard > Settings > API Keys (starts with `wf_sk_`) |
| AI coding agent | Any of: Claude Code, Cursor, Windsurf, GitHub Copilot, Cline, Continue.dev |

## Step 1: Get Your API Key

1. Go to [agent.willform.ai](https://agent.willform.ai)
2. Sign in with your wallet
3. Navigate to **Settings > API Keys**
4. Create a new key — copy the `wf_sk_...` value (shown only once)

## Step 2: Connect Your Agent

### Claude Code (CLI)

```bash
claude mcp add --transport http --header "Authorization: Bearer wf_sk_YOUR_KEY" willform https://agent.willform.ai/api/mcp
```

Or add to `.mcp.json` in your project:

```json
{
  "mcpServers": {
    "willform": {
      "type": "http",
      "url": "https://agent.willform.ai/api/mcp",
      "headers": {
        "Authorization": "Bearer wf_sk_YOUR_KEY"
      }
    }
  }
}
```

### Cursor

Create `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "willform": {
      "url": "https://agent.willform.ai/api/mcp",
      "transport": "streamable-http",
      "headers": {
        "Authorization": "Bearer wf_sk_YOUR_KEY"
      }
    }
  }
}
```

### Windsurf (Codeium)

Edit `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "willform": {
      "serverUrl": "https://agent.willform.ai/api/mcp",
      "headers": {
        "Authorization": "Bearer wf_sk_YOUR_KEY"
      }
    }
  }
}
```

Note: Windsurf uses `serverUrl` (not `url`).

### GitHub Copilot (VS Code)

Add to VS Code `settings.json`:

```json
{
  "mcp": {
    "servers": {
      "willform": {
        "type": "http",
        "url": "https://agent.willform.ai/api/mcp",
        "headers": {
          "Authorization": "Bearer wf_sk_YOUR_KEY"
        }
      }
    }
  }
}
```

### Cline (VS Code extension)

Edit `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "willform": {
      "url": "https://agent.willform.ai/api/mcp",
      "type": "streamableHttp",
      "headers": {
        "Authorization": "Bearer wf_sk_YOUR_KEY"
      },
      "disabled": false
    }
  }
}
```

Note: Cline uses `streamableHttp` (camelCase).

### Continue.dev

Add to `config.yaml`:

```yaml
mcpServers:
  - name: willform
    type: streamable-http
    url: https://agent.willform.ai/api/mcp
    headers:
      Authorization: "Bearer wf_sk_YOUR_KEY"
```

## Step 3: Verify

Ask your agent:

```
List my Willform namespaces
```

The agent should call `namespace_list` and return your namespaces.

Try more:

```
Deploy nginx:alpine as a web service on port 80
Show my credit balance
What deployments are running?
Create a PostgreSQL database called my-db
```

## Available Tools (42 total)

| Category | Tools |
|----------|-------|
| **Namespace** | `namespace_list`, `namespace_create`, `namespace_get`, `namespace_update`, `namespace_delete` |
| **Deploy** | `deploy_create`, `deploy_plan`, `deploy_status`, `deploy_list`, `deploy_logs`, `deploy_events`, `deploy_stop`, `deploy_restart`, `deploy_delete`, `deploy_scale`, `deploy_update_env`, `deploy_expose`, `deploy_unexpose` |
| **Domain** | `domain_list`, `domain_add`, `domain_remove`, `domain_status` |
| **Build** | `build_upload`, `build_start`, `build_status`, `build_and_deploy_finish` |
| **Template** | `template_list`, `template_get`, `template_deploy` |
| **Billing** | `credit_balance`, `credit_topup`, `transaction_list`, `billing_estimate` |
| **Agent** | `agent_status`, `agent_deploy`, `agent_delete`, `ask_willy`, `agent_chat` |
| **Preflight** | `deploy_preflight` |
| **Account** | `account_info`, `api_key_list`, `api_key_create`, `api_key_revoke` |

## OpenClaw Limitation

OpenClaw (as of v2026.3.8) does **not** support MCP client connections. The `mcpServers` config key is rejected by all versions. OpenClaw is a chatbot gateway (Telegram/Discord/Slack), not a coding IDE — it uses REST API via curl commands instead. See the `openclaw-willform-deploy` E2E scenario for the curl-based approach.

## Troubleshooting

### 401 Unauthorized

- Check that your API key starts with `wf_sk_`
- API keys are SHA-256 hashed in DB — if expired, create a new one

### Connection timeout

- Verify endpoint: `curl -H "Authorization: Bearer wf_sk_..." https://agent.willform.ai/api/mcp`
- Ensure your network allows HTTPS outbound

### Rate limiting (429)

120 requests/minute per user. Add delays between rapid operations.

### transport type mismatch

Each agent uses a different type field name:

| Agent | Type value |
|-------|-----------|
| Claude Code / Copilot | `http` |
| Cursor / Continue.dev | `streamable-http` |
| Cline | `streamableHttp` |
| Windsurf | (uses `serverUrl` instead of `url`) |

## References

- [Willform MCP Docs](https://agent.willform.ai/docs/mcp-setup)
- [MCP Transports Spec](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports)
