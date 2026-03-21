---
category: ai-agent
tags: [crewai, multi-agent, ai, mcp, python]
---

# CrewAI Multi-Agent Framework

Deploy a CrewAI multi-agent system with MCP tools pre-wired to Willform. Define crews, agents, and tasks — the platform handles orchestration.

## What You Get

- A Python worker running CrewAI with your custom crew definition
- MCP endpoint pre-configured so agents can call Willform's 42 tools
- A2A endpoint for agent-to-agent communication
- Persistent volume for crew state and memory
- OpenAI, Anthropic, or any LiteLLM-compatible model

## Prerequisites

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys) | LLM inference for CrewAI agents | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | Create at [Anthropic Console](https://console.anthropic.com/settings/keys) | LLM inference — Claude models. Leave blank if not using. | `sk-ant-api03-...` |
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > API Keys at [agent.willform.ai](https://agent.willform.ai) | Authenticates MCP tool calls to Willform | `wf_sk_...` |

## Post-Deploy

Your CrewAI worker is running. Connect to it via the Willform A2A endpoint:

**A2A Endpoint**: https://agent.willform.ai/a2a

To customize your crew, edit `crew.py` on the deployed container or rebuild with your own Docker image.

The `MCP_ENDPOINT` and `A2A_ENDPOINT` environment variables are pre-injected — use them in your crew definition:

```python
from crewai_tools import MCPServerAdapter

mcp_tools = MCPServerAdapter({"url": os.environ["MCP_ENDPOINT"]})
```

## Deploy Config

```json
{
  "name": "crewai",
  "chartType": "worker",
  "image": "python:3.12-slim",
  "replicas": 1,
  "volumeSizeGb": 4,
  "volumeMountPath": "/app/data",
  "env": {
    "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
    "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
    "WILLFORM_API_KEY": "YOUR_WILLFORM_API_KEY"
  }
}
```
