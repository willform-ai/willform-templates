---
category: ai-agent
tags: [openclaw, telegram, ai, finance]
---

# OpenClaw Investment Researcher Agent

Deploy an AI chatbot on Telegram that answers questions about stocks, ETFs, crypto, and market analysis.

## What You Get

- An OpenClaw AI chatbot running on Willform cloud
- Telegram DM integration for conversing with the bot
- Web-based Control UI for managing the chatbot
- Real-time market data via web search tools
- Persistent memory that saves conversation history and user preferences

## Prerequisites

Replace the 4 placeholder values below with your own before pasting the prompt.

| Variable | How to get it | Purpose |
|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | Create an API key at [Anthropic Console](https://console.anthropic.com/settings/keys) | Powers LLM inference for the chatbot |
| `YOUR_TELEGRAM_BOT_TOKEN` | Send `/newbot` to [@BotFather](https://t.me/BotFather) on Telegram | Connects the bot to Telegram |
| `YOUR_TELEGRAM_USER_ID` | Send any message to [@userinfobot](https://t.me/userinfobot) on Telegram → get your numeric ID | Adds you to the DM allowlist |
| `YOUR_GATEWAY_TOKEN` | Choose any password (e.g. `my-secret-123`) | Authenticates access to the web Control UI |

## Prompt

```
Deploy an OpenClaw AI agent on Willform.

Image: alpine/openclaw:2026.2.26
Name: openclaw
Port: 18789
Volume: /home/node/.openclaw, 8GB

Environment variables:
    ANTHROPIC_API_KEY: "YOUR_ANTHROPIC_API_KEY"
    TELEGRAM_BOT_TOKEN: "YOUR_TELEGRAM_BOT_TOKEN"
    OPENCLAW_GATEWAY_TOKEN: "YOUR_GATEWAY_TOKEN"

After deployment:
1. Expose the domain (POST /api/deploy/{id}/expose) → returns {domain}
2. First access: https://{domain}/?token={YOUR_GATEWAY_TOKEN}
```

## Deploy Config

```json
{
  "name": "openclaw",
  "chartName": "openclaw",
  "chartType": "web",
  "image": "alpine/openclaw:2026.2.26",
  "port": 18789,
  "replicas": 1,
  "volumeSizeGb": 8,
  "volumeMountPath": "/home/node/.openclaw",
  "healthCheckPath": null,
  "env": {
    "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
    "TELEGRAM_BOT_TOKEN": "YOUR_TELEGRAM_BOT_TOKEN",
    "OPENCLAW_GATEWAY_TOKEN": "YOUR_GATEWAY_TOKEN"
  },
  "command": [
    "node",
    "openclaw.mjs",
    "gateway",
    "--allow-unconfigured",
    "--bind",
    "lan"
  ],
  "expose": true,
  "exposeProtocol": "http",
  "chartValues": {
    "files": [
      {
        "path": "/home/node/.openclaw/openclaw.json",
        "content": "{\n  \"gateway\": {\n    \"bind\": \"lan\",\n    \"port\": 18789,\n    \"controlUi\": {\n      \"enabled\": true,\n      \"dangerouslyDisableDeviceAuth\": true,\n      \"dangerouslyAllowHostHeaderOriginFallback\": true\n    },\n    \"auth\": {\n      \"mode\": \"token\"\n    }\n  },\n  \"channels\": {\n    \"telegram\": {\n      \"enabled\": true,\n      \"dmPolicy\": \"allowlist\",\n      \"allowFrom\": [\n        \"tg:YOUR_TELEGRAM_USER_ID\"\n      ]\n    }\n  },\n  \"agents\": {\n    \"defaults\": {\n      \"sandbox\": {\n        \"mode\": \"off\"\n      }\n    }\n  },\n  \"tools\": {\n    \"web\": {\n      \"search\": {\n        \"enabled\": true\n      },\n      \"fetch\": {\n        \"enabled\": true\n      }\n    },\n    \"sandbox\": {\n      \"tools\": {\n        \"allow\": [\n          \"exec\",\n          \"process\",\n          \"read\",\n          \"write\",\n          \"edit\",\n          \"sessions_list\",\n          \"sessions_history\",\n          \"sessions_send\",\n          \"sessions_spawn\",\n          \"session_status\",\n          \"browser\",\n          \"canvas\",\n          \"nodes\",\n          \"cron\",\n          \"gateway\",\n          \"web_search\",\n          \"web_fetch\"\n        ],\n        \"deny\": []\n      }\n    }\n  },\n  \"plugins\": {\n    \"entries\": {\n      \"telegram\": {\n        \"enabled\": true\n      }\n    }\n  }\n}"
      },
      {
        "path": "/home/node/.openclaw/workspace/IDENTITY.md",
        "content": "# Bullish\n📊\nAI Investment Researcher"
      },
      {
        "path": "/home/node/.openclaw/workspace/SOUL.md",
        "content": "# Bullish — AI Investment Researcher\nI am Bullish, an AI-powered investment researcher.\nAll responses must be written in English. Use a professional yet approachable tone.\nAreas of expertise: stocks, ETFs, bonds, cryptocurrency, macroeconomics, sector analysis, portfolio strategy.\nPrioritize data-driven analysis and always cite sources.\nNever guarantee investment returns or recommend specific buy/sell actions.\nAlways remind users that final investment decisions are their own responsibility."
      },
      {
        "path": "/home/node/.openclaw/workspace/AGENTS.md",
        "content": "# Agent Behavior Rules\n- Reference SOUL.md before every response\n- Load memory files at the start of each session\n- Always include supporting data and sources in analysis\n- Save user investment preferences, watchlists, and portfolios to memory\n- Never share personal portfolio information in group chats\n- Always respond in English regardless of the language of the question"
      }
    ]
  }
}
```
