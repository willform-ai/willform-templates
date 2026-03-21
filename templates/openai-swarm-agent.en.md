---
category: ai-agent
tags: [openai, swarm, multi-agent, ai, mcp, python]
---

# OpenAI Swarm Agent Network

Deploy an OpenAI Swarm multi-agent network with MCP tools pre-wired to Willform. Lightweight agent handoffs — each agent specializes in one task, hands off to the next.

## What You Get

- A Python worker running OpenAI Swarm with your custom agent network
- MCP endpoint pre-configured — agents call Willform's 42 tools out of the box
- A2A endpoint for external coordination
- Persistent volume for run history
- Built on the official OpenAI Swarm library

## Prerequisites

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys) | LLM inference — required for Swarm | `sk-proj-...` |
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > API Keys at [agent.willform.ai](https://agent.willform.ai) | Authenticates MCP tool calls to Willform | `wf_sk_...` |

## Post-Deploy

Your Swarm worker is running. The `MCP_ENDPOINT` environment variable is pre-injected — load Willform tools into your agents:

```python
import os
from swarm import Swarm, Agent
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

# Connect to Willform MCP
async with streamablehttp_client(os.environ["MCP_ENDPOINT"]) as (r, w, _):
    async with ClientSession(r, w) as session:
        tools = await session.list_tools()

agent = Agent(name="Willform Agent", functions=[...])  # map MCP tools here
client = Swarm()
response = client.run(agent=agent, messages=[{"role": "user", "content": "Deploy my app"}])
```

**A2A Endpoint**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "swarm",
  "chartType": "worker",
  "image": "willformhq/openai-swarm-starter:latest",
  "replicas": 1,
  "volumeSizeGb": 2,
  "volumeMountPath": "/app/data",
  "env": {
    "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
    "WILLFORM_API_KEY": "YOUR_WILLFORM_API_KEY"
  }
}
```
