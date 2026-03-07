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

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` | Send `/newbot` to [@BotFather](https://t.me/BotFather). [Docs](https://core.telegram.org/bots#how-do-i-create-a-bot) | Connects the bot to Telegram | `110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw` |
| `YOUR_TELEGRAM_USER_ID` | Send any message to [@userinfobot](https://t.me/userinfobot) and copy the numeric ID | Adds you to the DM allowlist | `1234567890` |
| `YOUR_GATEWAY_TOKEN` | Choose any password you like — this is your own secret | Authenticates access to the web Control UI | `my-secret-123` |

### Provider: OpenRouter (default)

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_OPENROUTER_API_KEY` | Create at [OpenRouter](https://openrouter.ai/keys). Add credits at [Credits](https://openrouter.ai/credits). | Powers LLM inference (supports 200+ models) | `sk-or-v1-...` |

### Provider: OpenAI

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys). Add credits at [Billing](https://platform.openai.com/settings/organization/billing). | Powers LLM inference via OpenAI models | `sk-proj-...` |

### Provider: Anthropic

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | Create at [Anthropic Console](https://console.anthropic.com/settings/keys). Add credits at [Billing](https://console.anthropic.com/settings/billing). | Powers LLM inference via Claude models | `sk-ant-api03-...` |

### Provider: Google Gemini

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_GEMINI_API_KEY` | Create at [Google AI Studio](https://aistudio.google.com/apikey). | Powers LLM inference via Gemini models | `AIza...` |

## Post-Deploy

**Control UI**: https://$DOMAIN/?token=YOUR_GATEWAY_TOKEN

Open the link above to manage your chatbot. Talk to the bot on Telegram — it's ready to answer your investment questions.

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
              "IDENTITY.md": "# Bullish\n📊\nAI Investment Researcher",
              "SOUL.md": "# Bullish — AI Investment Researcher\nI am Bullish, an AI-powered investment researcher.\nAll responses must be written in English. Use a professional yet approachable tone.\nAreas of expertise: stocks, ETFs, bonds, cryptocurrency, macroeconomics, sector analysis, portfolio strategy.\nPrioritize data-driven analysis and always cite sources.\nNever guarantee investment returns or recommend specific buy/sell actions.\nAlways remind users that final investment decisions are their own responsibility.",
              "AGENTS.md": "# Agent Behavior Rules\n- Reference SOUL.md before every response\n- Load memory files at the start of each session\n- Always include supporting data and sources in analysis\n- Save user investment preferences, watchlists, and portfolios to memory\n- Never share personal portfolio information in group chats\n- Always respond in English regardless of the language of the question"
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
