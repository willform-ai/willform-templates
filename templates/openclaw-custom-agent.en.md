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

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` | Send `/newbot` to [@BotFather](https://t.me/BotFather). [Docs](https://core.telegram.org/bots#how-do-i-create-a-bot) | Connects the bot to Telegram | `110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw` |
| `YOUR_TELEGRAM_USER_ID` | Send any message to [@userinfobot](https://t.me/userinfobot) and copy the numeric ID | Adds you to the DM allowlist | `1234567890` |
| `YOUR_GATEWAY_TOKEN` | Choose any password you like — this is your own secret | Authenticates access to the web Control UI | `my-secret-123` |
| `YOUR_OPENROUTER_API_KEY` | Create at [OpenRouter](https://openrouter.ai/keys). Add credits at [Credits](https://openrouter.ai/credits). | LLM inference — 200+ models (default). Leave blank if not using. | `sk-or-v1-...` |
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys). Add credits at [Billing](https://platform.openai.com/settings/organization/billing). | LLM inference — OpenAI models. Leave blank if not using. | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | Create at [Anthropic Console](https://console.anthropic.com/settings/keys). Add credits at [Billing](https://console.anthropic.com/settings/billing). | LLM inference — Claude models. Leave blank if not using. | `sk-ant-api03-...` |
| `YOUR_GEMINI_API_KEY` | Create at [Google AI Studio](https://aistudio.google.com/apikey). | LLM inference — Gemini models. Leave blank if not using. | `AIza...` |
| `YOUR_IDENTITY_MD` | Write your own (see guide below) | Defines the agent's name, emoji, and role | `Name: ...\nCreature: ...\nVibe: ...\nEmoji: ...` |
| `YOUR_SOUL_MD` | Write your own (see guide below) | Defines the agent's personality, expertise, and tone | `## Core Truths\n## Boundaries\n## Vibe\n## Continuity` |
| `YOUR_AGENTS_MD` | Write your own (see guide below) | Defines the agent's behavior rules and constraints | `## Session Startup\n## Memory Management\n## Analysis Framework` |

> Only one API key is required. Choose the provider you want to use and leave the rest blank. If multiple keys are provided, OpenClaw picks the default in this order: Anthropic > OpenAI > OpenRouter > Gemini.

### How to write IDENTITY.md

Defines who your agent is — name tag, role, and vibe.

Example:
```
# Identity

Name: ...
Creature: ...
Vibe: ...
Emoji: ...
```

### How to write SOUL.md

Defines your agent's personality — core values, boundaries, and communication style.

Example:
```
# Soul

## Core Truths
- ...

## Boundaries
- ...

## Vibe
...

## Continuity
...
```

### How to write AGENTS.md

Defines the agent's operating manual — startup routine, memory management, and task-specific workflows.

Example:
```
# Agents

## Session Startup
1. ...

## Memory Management
- ...

## Analysis Framework
1. ...
2. ...
3. ...
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
    "OPENROUTER_API_KEY": "YOUR_OPENROUTER_API_KEY",
    "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
    "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
    "GEMINI_API_KEY": "YOUR_GEMINI_API_KEY",
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
                  "OPENROUTER_API_KEY": "YOUR_OPENROUTER_API_KEY",
                  "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
                  "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
                  "GEMINI_API_KEY": "YOUR_GEMINI_API_KEY",
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
