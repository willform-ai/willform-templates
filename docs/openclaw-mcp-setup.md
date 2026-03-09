# Connect Willform MCP to OpenClaw

Give your OpenClaw agent access to Willform's 42 MCP tools — deploy containers, manage domains, check billing, and more — all from Telegram.

## Overview

- **Willform MCP** uses Streamable HTTP transport (`https://agent.willform.ai/api/mcp`)
- **OpenClaw** natively supports stdio-based MCP servers only
- **Solution**: Use `mcp-proxy` to bridge Streamable HTTP to stdio

```
OpenClaw Agent  ──stdio──>  mcp-proxy  ──streamable-http──>  Willform MCP
(in K8s pod)                (child process)                  (agent.willform.ai)
```

## Prerequisites

| Item | How to get it |
|------|---------------|
| Willform API Key | Dashboard > Settings > API Keys (starts with `wf_sk_`) |
| OpenClaw instance | Deploy via any OpenClaw template on Willform |
| Node.js 18+ | Included in OpenClaw image (`alpine/openclaw`) |

## Step 1: Get Your API Key

1. Go to [agent.willform.ai](https://agent.willform.ai)
2. Sign in with your wallet
3. Navigate to **Settings > API Keys**
4. Create a new key — copy the `wf_sk_...` value (shown only once)

## Step 2: Install mcp-proxy in OpenClaw

Open the OpenClaw Control UI and run in the sandbox terminal:

```bash
npm install -g mcp-proxy
```

> Note: This persists in the volume at `/home/node/.openclaw/`. Survives pod restarts.

## Step 3: Configure openclaw.json

Open the Control UI, navigate to Settings, and add the `mcpServers` section:

```json5
{
  // ... existing gateway, channels config ...

  "mcpServers": {
    "willform": {
      "command": "mcp-proxy",
      "args": [
        "--transport", "streamablehttp",
        "--headers", "Authorization", "Bearer YOUR_WILLFORM_API_KEY",
        "https://agent.willform.ai/api/mcp"
      ]
    }
  }
}
```

Replace `YOUR_WILLFORM_API_KEY` with your actual `wf_sk_...` key.

### Using environment variables (recommended)

To avoid hardcoding the API key in config:

```json5
{
  "mcpServers": {
    "willform": {
      "command": "mcp-proxy",
      "args": [
        "--transport", "streamablehttp",
        "--headers", "Authorization", "Bearer ${WILLFORM_API_KEY}",
        "https://agent.willform.ai/api/mcp"
      ],
      "env": {
        "WILLFORM_API_KEY": "wf_sk_..."
      }
    }
  }
}
```

## Step 4: Restart Gateway

After saving the config, restart the OpenClaw gateway for changes to take effect:

```bash
openclaw gateway restart
```

Or from the Control UI: **Settings > Restart Gateway**.

## Step 5: Verify

Talk to your OpenClaw bot on Telegram:

```
List my namespaces
```

The agent should call the `namespace_list` Willform MCP tool and return your namespaces.

Try more commands:

```
Deploy a Redis cache called my-redis
Show my credit balance
What deployments are running?
```

## Available Tools (42 total)

Once connected, your OpenClaw agent has access to all Willform MCP tools:

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

## Template Configuration

To include Willform MCP in a template's `openclaw.json`, add the `mcpServers` block to the config string in `chartValues.upstream.app-template.configMaps.config.data["openclaw.json"]`.

Since the config is a JSON string inside `Deploy Config`, escape it properly:

```
\"mcpServers\": { \"willform\": { \"command\": \"mcp-proxy\", \"args\": [\"--transport\", \"streamablehttp\", \"--headers\", \"Authorization\", \"Bearer YOUR_WILLFORM_API_KEY\", \"https://agent.willform.ai/api/mcp\"] } }
```

And add a prerequisite variable:

```markdown
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > [API Keys](https://agent.willform.ai/dashboard) | Authenticates MCP tool calls to Willform | `wf_sk_...` |
```

## Troubleshooting

### "mcp-proxy: command not found"

Install it globally in the OpenClaw container:

```bash
npm install -g mcp-proxy
```

If npm global bin is not in PATH:

```bash
export PATH="$PATH:$(npm config get prefix)/bin"
```

### Connection timeout

- Verify your API key is valid: `curl -H "Authorization: Bearer wf_sk_..." https://agent.willform.ai/api/mcp` should return a response (not 401)
- Check if the pod has outbound internet access (NetworkPolicy)

### Tools not appearing

- Restart the gateway after config changes
- Check gateway logs for MCP connection errors: `openclaw gateway logs`
- Verify JSON5 syntax with `openclaw doctor`

### Rate limiting (429)

Willform MCP has a per-user rate limit of 120 requests/minute. If your agent is making too many calls, add delays between operations.

## References

- [Willform MCP Docs](https://agent.willform.ai/docs/mcp-setup)
- [mcp-proxy GitHub](https://github.com/sparfenyuk/mcp-proxy)
- [OpenClaw MCP Issue #8188](https://github.com/openclaw/openclaw/issues/8188) — native HTTP MCP support (pending)
