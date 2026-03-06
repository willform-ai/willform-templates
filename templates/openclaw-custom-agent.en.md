---
category: ai-agent
tags: [openclaw, telegram, ai, custom]
---

# OpenClaw Custom Agent

Deploy your own AI agent on Telegram with a fully customizable identity, personality, and behavior rules.

## What You Get

- An OpenClaw AI agent running on Willform cloud with your own personality
- Telegram DM integration for conversing with the bot
- Web-based Control UI for managing the agent
- Web search and fetch tools for real-time information
- Persistent memory that saves conversation history and user preferences

## Prerequisites

Replace the 7 placeholder values below with your own.

| Variable | How to get it | Purpose |
|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | Create an API key at [Anthropic Console](https://console.anthropic.com/settings/keys) | Powers LLM inference for the agent |
| `YOUR_TELEGRAM_BOT_TOKEN` | Send `/newbot` to [@BotFather](https://t.me/BotFather) on Telegram | Connects the bot to Telegram |
| `YOUR_TELEGRAM_USER_ID` | Send any message to [@userinfobot](https://t.me/userinfobot) on Telegram → get your numeric ID | Adds you to the DM allowlist |
| `YOUR_GATEWAY_TOKEN` | Choose any password (e.g. `my-secret-123`) | Authenticates access to the web Control UI |
| `YOUR_IDENTITY_MD` | Write your own (see guide below) | Defines the agent's name, emoji, and role |
| `YOUR_SOUL_MD` | Write your own (see guide below) | Defines the agent's personality, expertise, and tone |
| `YOUR_AGENTS_MD` | Write your own (see guide below) | Defines the agent's behavior rules and constraints |

### How to write IDENTITY.md

Three lines that define who your agent is:

```
# AgentName
🤖
Short role description
```

Example:
```
# Atlas
🌍
AI Travel Planning Assistant
```

### How to write SOUL.md

A short document that defines your agent's personality. Include:
- Who the agent is (name and role)
- Language and tone preferences
- Areas of expertise
- Key constraints or boundaries

Example:
```
# Atlas — AI Travel Planner
I am Atlas, an AI travel planning assistant.
All responses must be written in English. Use a friendly, enthusiastic tone.
Areas of expertise: flight search, hotel recommendations, itinerary planning, local cuisine, cultural tips.
Prioritize budget-conscious options and always mention visa requirements.
Never book anything on behalf of the user — only suggest and inform.
```

### How to write AGENTS.md

A list of behavior rules your agent must follow:

Example:
```
# Agent Behavior Rules
- Reference SOUL.md before every response
- Load memory files at the start of each session
- Always provide multiple options with price ranges
- Save user travel preferences and past trips to memory
- Never share personal information in group chats
- Always respond in English regardless of the language of the question
```

## Post-Deploy

**Control UI**: https://$DOMAIN/?token=YOUR_GATEWAY_TOKEN

Open the link above to manage your agent. Talk to the bot on Telegram — your custom agent is ready.

## Deploy Config

```json
{
  "name": "openclaw",
  "chartName": "openclaw",
  "nativeChart": true,
  "chartType": "web",
  "image": "alpine/openclaw:2026.3.2",
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
  "expose": true,
  "exposeProtocol": "http",
  "chartValues": {
    "upstream": {
      "app-template": {
        "openclawVersion": "2026.3.2",
        "configMode": "overwrite",
        "controllers": {
          "main": {
            "replicas": 1,
            "containers": {
              "main": {
                "image": {
                  "repository": "alpine/openclaw",
                  "tag": "2026.3.2"
                },
                "command": ["node", "dist/index.js"],
                "args": ["gateway", "--allow-unconfigured", "--bind", "lan", "--port", "18789"],
                "env": {
                  "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
                  "TELEGRAM_BOT_TOKEN": "YOUR_TELEGRAM_BOT_TOKEN",
                  "OPENCLAW_GATEWAY_TOKEN": "YOUR_GATEWAY_TOKEN"
                }
              },
              "chromium": {
                "enabled": false
              }
            },
            "initContainers": {
              "init-config": {
                "command": ["sh", "-c"],
                "args": ["mkdir -p /home/node/.openclaw/workspace && cp /config/openclaw.json /home/node/.openclaw/openclaw.json && for f in IDENTITY.md SOUL.md AGENTS.md; do [ -f /config/$f ] && cp /config/$f /home/node/.openclaw/workspace/$f; done"]
              }
            }
          }
        },
        "configMaps": {
          "config": {
            "data": {
              "openclaw.json": "{\n  \"gateway\": {\n    \"bind\": \"lan\",\n    \"port\": 18789,\n    \"controlUi\": {\n      \"enabled\": true,\n      \"dangerouslyDisableDeviceAuth\": true,\n      \"dangerouslyAllowHostHeaderOriginFallback\": true\n    },\n    \"auth\": {\n      \"mode\": \"token\"\n    }\n  },\n  \"channels\": {\n    \"telegram\": {\n      \"enabled\": true,\n      \"dmPolicy\": \"allowlist\",\n      \"allowFrom\": [\n        \"tg:YOUR_TELEGRAM_USER_ID\"\n      ]\n    }\n  },\n  \"agents\": {\n    \"defaults\": {\n      \"sandbox\": {\n        \"mode\": \"off\"\n      }\n    }\n  },\n  \"tools\": {\n    \"web\": {\n      \"search\": {\n        \"enabled\": true\n      },\n      \"fetch\": {\n        \"enabled\": true\n      }\n    },\n    \"sandbox\": {\n      \"tools\": {\n        \"allow\": [\n          \"exec\",\n          \"process\",\n          \"read\",\n          \"write\",\n          \"edit\",\n          \"sessions_list\",\n          \"sessions_history\",\n          \"sessions_send\",\n          \"sessions_spawn\",\n          \"session_status\",\n          \"browser\",\n          \"canvas\",\n          \"nodes\",\n          \"cron\",\n          \"gateway\",\n          \"web_search\",\n          \"web_fetch\"\n        ],\n        \"deny\": []\n      }\n    }\n  },\n  \"plugins\": {\n    \"entries\": {\n      \"telegram\": {\n        \"enabled\": true\n      }\n    }\n  }\n}",
              "IDENTITY.md": "YOUR_IDENTITY_MD",
              "SOUL.md": "YOUR_SOUL_MD",
              "AGENTS.md": "YOUR_AGENTS_MD"
            }
          }
        },
        "persistence": {
          "data": {
            "size": "8Gi"
          }
        }
      }
    }
  },
  "valuesMappings": {
    "replicas": "upstream.app-template.controllers.main.replicas",
    "env": "upstream.app-template.controllers.main.containers.main.env",
    "image.tag": "upstream.app-template.openclawVersion"
  }
}
```
