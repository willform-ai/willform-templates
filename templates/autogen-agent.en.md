---
category: ai-agent
tags: [autogen, microsoft, multi-agent, ai, mcp, python]
---

# AutoGen Multi-Agent System

Deploy a Microsoft AutoGen multi-agent system with MCP tools pre-wired to Willform. Build conversational agent networks that collaborate to solve complex tasks.

## What You Get

- A Python worker running AutoGen with your custom agent definitions
- MCP endpoint pre-configured — agents call Willform's 42 tools automatically
- A2A endpoint for external agent communication
- Persistent volume for conversation history and state
- Compatible with OpenAI, Anthropic, and Azure OpenAI

## Prerequisites

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys) | LLM inference for AutoGen agents | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | Create at [Anthropic Console](https://console.anthropic.com/settings/keys) | LLM inference — Claude models. Leave blank if not using. | `sk-ant-api03-...` |
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > API Keys at [agent.willform.ai](https://agent.willform.ai) | Authenticates MCP tool calls to Willform | `wf_sk_...` |

## Post-Deploy

Your AutoGen worker is running. The `MCP_ENDPOINT` environment variable is pre-injected — use it to register Willform MCP tools with your agents:

```python
from autogen.mcp import create_toolkit
from autogen import AssistantAgent

toolkit = await create_toolkit(server_params={"url": os.environ["MCP_ENDPOINT"]})
agent = AssistantAgent("assistant", tools=toolkit.tools)
```

**A2A Endpoint**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "autogen",
  "chartType": "worker",
  "image": "willformhq/autogen-starter:latest",
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
