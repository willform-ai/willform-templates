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
Type: web
Port: 18789
Resources: 1 core, 2GB memory
Volume: /home/node/.openclaw, 8GB
Health check: null (the literal string "null")
Replicas: 1

Environment variables:
    ANTHROPIC_API_KEY: "YOUR_ANTHROPIC_API_KEY"
    TELEGRAM_BOT_TOKEN: "YOUR_TELEGRAM_BOT_TOKEN"
    OPENCLAW_GATEWAY_TOKEN: "YOUR_GATEWAY_TOKEN"

Startup command (sh -c):

1) Write openclaw.json:
cat > /home/node/.openclaw/openclaw.json << 'OCJSON'
{
  "gateway": {
    "bind": "lan",
    "port": 18789,
    "controlUi": {
      "enabled": true,
      "dangerouslyDisableDeviceAuth": true,
      "dangerouslyAllowHostHeaderOriginFallback": true
    },
    "auth": { "mode": "token" }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["tg:YOUR_TELEGRAM_USER_ID"]
    }
  },
  "agents": {
    "defaults": { "sandbox": { "mode": "off" } }
  },
  "tools": {
    "web": { "search": { "enabled": true }, "fetch": { "enabled": true } },
    "sandbox": {
      "tools": {
        "allow": ["exec","process","read","write","edit","sessions_list","sessions_history","sessions_send","sessions_spawn","session_status","browser","canvas","nodes","cron","gateway","web_search","web_fetch"],
        "deny": []
      }
    }
  },
  "plugins": {
    "entries": {
      "telegram": { "enabled": true }
    }
  }
}
OCJSON

2) Create workspace directory and write IDENTITY.md:
mkdir -p /home/node/.openclaw/workspace
cat > /home/node/.openclaw/workspace/IDENTITY.md << 'EOF'
# Bullish
📊
AI Investment Researcher
EOF

3) Write SOUL.md:
cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
# Bullish — AI Investment Researcher
I am Bullish, an AI-powered investment researcher.
All responses must be written in English. Use a professional yet approachable tone.
Areas of expertise: stocks, ETFs, bonds, cryptocurrency, macroeconomics, sector analysis, portfolio strategy.
Prioritize data-driven analysis and always cite sources.
Never guarantee investment returns or recommend specific buy/sell actions.
Always remind users that final investment decisions are their own responsibility.
EOF

4) Write AGENTS.md:
cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
# Agent Behavior Rules
- Reference SOUL.md before every response
- Load memory files at the start of each session
- Always include supporting data and sources in analysis
- Save user investment preferences, watchlists, and portfolios to memory
- Never share personal portfolio information in group chats
- Always respond in English regardless of the language of the question
EOF

5) Start the gateway:
exec node openclaw.mjs gateway --allow-unconfigured --bind lan

After deployment:
1. Expose the domain (POST /api/deploy/{id}/expose) → returns {domain}
2. First access: https://{domain}/?token={YOUR_GATEWAY_TOKEN}
```

## Deploy Config

```json
{
  "name": "openclaw",
  "image": "alpine/openclaw:2026.2.26",
  "chartType": "web",
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
  "command": ["sh", "-c"],
  "expose": true,
  "exposeProtocol": "http"
}
```

## Startup Script

```bash
cat > /home/node/.openclaw/openclaw.json << 'OCJSON'
{
  "gateway": {
    "bind": "lan",
    "port": 18789,
    "controlUi": {
      "enabled": true,
      "dangerouslyDisableDeviceAuth": true,
      "dangerouslyAllowHostHeaderOriginFallback": true
    },
    "auth": { "mode": "token" }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": ["tg:YOUR_TELEGRAM_USER_ID"]
    }
  },
  "agents": {
    "defaults": { "sandbox": { "mode": "off" } }
  },
  "tools": {
    "web": { "search": { "enabled": true }, "fetch": { "enabled": true } },
    "sandbox": {
      "tools": {
        "allow": ["exec","process","read","write","edit","sessions_list","sessions_history","sessions_send","sessions_spawn","session_status","browser","canvas","nodes","cron","gateway","web_search","web_fetch"],
        "deny": []
      }
    }
  },
  "plugins": {
    "entries": {
      "telegram": { "enabled": true }
    }
  }
}
OCJSON

mkdir -p /home/node/.openclaw/workspace
cat > /home/node/.openclaw/workspace/IDENTITY.md << 'EOF'
# Bullish
📊
AI Investment Researcher
EOF

cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
# Bullish — AI Investment Researcher
I am Bullish, an AI-powered investment researcher.
All responses must be written in English. Use a professional yet approachable tone.
Areas of expertise: stocks, ETFs, bonds, cryptocurrency, macroeconomics, sector analysis, portfolio strategy.
Prioritize data-driven analysis and always cite sources.
Never guarantee investment returns or recommend specific buy/sell actions.
Always remind users that final investment decisions are their own responsibility.
EOF

cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
# Agent Behavior Rules
- Reference SOUL.md before every response
- Load memory files at the start of each session
- Always include supporting data and sources in analysis
- Save user investment preferences, watchlists, and portfolios to memory
- Never share personal portfolio information in group chats
- Always respond in English regardless of the language of the question
EOF

exec node openclaw.mjs gateway --allow-unconfigured --bind lan
```
