---
category: ai-agent
tags: [langgraph, langchain, graph, ai, mcp, python]
---

# LangGraph Agent Framework

Deploy a stateful LangGraph agent with MCP tools pre-wired to Willform. Build complex agent workflows with branching, loops, and persistent state.

## What You Get

- A Python worker running LangGraph with your custom graph definition
- MCP endpoint pre-configured — agents call Willform's 42 tools out of the box
- A2A endpoint for inter-agent messaging
- Persistent volume for graph state and checkpointing
- Compatible with OpenAI, Anthropic, and any LangChain-supported model

## Prerequisites

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys) | LLM inference for LangGraph nodes | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | Create at [Anthropic Console](https://console.anthropic.com/settings/keys) | LLM inference — Claude models. Leave blank if not using. | `sk-ant-api03-...` |
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > API Keys at [agent.willform.ai](https://agent.willform.ai) | Authenticates MCP tool calls to Willform | `wf_sk_...` |

## Post-Deploy

Your LangGraph worker is running. The `MCP_ENDPOINT` environment variable is pre-injected — load it in your graph:

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

mcp_client = MultiServerMCPClient({
    "willform": {
        "url": os.environ["MCP_ENDPOINT"],
        "transport": "streamable_http",
    }
})
tools = await mcp_client.get_tools()
```

**A2A Endpoint**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "langgraph",
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
