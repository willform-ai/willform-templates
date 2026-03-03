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

Replace the 7 placeholder values below with your own before pasting the prompt.

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

## Prompt

```
Deploy an OpenClaw AI agent on Willform.

Image: ghcr.io/openclaw/openclaw:2026.2.23
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
    OPENCLAW_DOMAIN: (set after expose — see instructions below)

Startup command (sh -c):

1) Write openclaw.json (note: unquoted heredoc so $OPENCLAW_DOMAIN expands):
cat > /home/node/.openclaw/openclaw.json << OCJSON
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "port": 18790,
    "controlUi": {
      "enabled": true,
      "dangerouslyDisableDeviceAuth": true,
      "allowedOrigins": ["https://${OPENCLAW_DOMAIN:-localhost}"]
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

2) Write IDENTITY.md:
cat > /home/node/.openclaw/workspace/IDENTITY.md << 'EOF'
YOUR_IDENTITY_MD
EOF

3) Write SOUL.md:
cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
YOUR_SOUL_MD
EOF

4) Write AGENTS.md:
cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
YOUR_AGENTS_MD
EOF

5) Write HTTP reverse proxy and run in background (strips Cloudflare headers only — do NOT rewrite Host/Origin):
cat > /home/node/.openclaw/proxy.js << 'PROXYJS'
const http = require('http');
const net = require('net');
const STRIP = [
  'x-forwarded-for', 'x-forwarded-proto', 'x-real-ip',
  'cf-connecting-ip', 'cf-ray', 'cf-visitor',
  'cf-ipcountry', 'cdn-loop', 'cf-worker'
];

const server = http.createServer((req, res) => {
  STRIP.forEach(k => delete req.headers[k]);
  const proxy = http.request({
    hostname: '127.0.0.1', port: 18790,
    path: req.url, method: req.method, headers: req.headers
  }, upstream => {
    res.writeHead(upstream.statusCode, upstream.headers);
    upstream.pipe(res);
  });
  req.pipe(proxy);
  proxy.on('error', () => res.destroy());
});

server.on('upgrade', (req, socket, head) => {
  STRIP.forEach(k => delete req.headers[k]);
  const upstream = net.connect(18790, '127.0.0.1', () => {
    let raw = req.method + ' ' + req.url + ' HTTP/1.1\r\n';
    for (const [k, v] of Object.entries(req.headers)) {
      raw += k + ': ' + v + '\r\n';
    }
    raw += '\r\n';
    upstream.write(raw);
    if (head.length) upstream.write(head);
    socket.pipe(upstream);
    upstream.pipe(socket);
  });
  upstream.on('error', () => socket.destroy());
  socket.on('error', () => upstream.destroy());
});

server.listen(18789, '0.0.0.0');
PROXYJS
node /home/node/.openclaw/proxy.js &

6) Start the gateway:
exec node dist/index.js gateway --allow-unconfigured

After deployment:
1. Expose the domain (POST /api/deploy/{id}/expose) → returns {domain}
2. Update env: set OPENCLAW_DOMAIN={domain} (this triggers a restart with the correct allowedOrigins)
3. First access: https://{domain}/?token={YOUR_GATEWAY_TOKEN}
```
