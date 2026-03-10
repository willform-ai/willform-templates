---
category: ai-agent
tags: [openclaw, telegram, ai, coding, developer]
---

# OpenClaw Coding Developer Agent

Deploy an AI coding assistant on Telegram that helps with code review, debugging, architecture design, and programming across multiple languages.

## What You Get

- An OpenClaw AI coding assistant running on Willform cloud
- Telegram DM integration for getting coding help on the go
- Web-based Control UI for managing the agent
- Sandbox environment for executing and testing code snippets
- Web search tools for looking up documentation and APIs
- Persistent memory that saves your project context and preferences

## Prerequisites

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` | Send `/newbot` to [@BotFather](https://t.me/BotFather). [Docs](https://core.telegram.org/bots#how-do-i-create-a-bot) | Connects the bot to Telegram | `110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw` |
| `YOUR_TELEGRAM_USER_ID` | Send any message to [@userinfobot](https://t.me/userinfobot) and copy the numeric ID | Adds you to the DM allowlist | `1234567890` |
| `YOUR_GATEWAY_TOKEN` | Choose any password you like — this is your own secret | Authenticates access to the web Control UI | `my-secret-123` |
| `YOUR_OPENROUTER_API_KEY` | Create at [OpenRouter](https://openrouter.ai/keys). Add credits at [Credits](https://openrouter.ai/credits). | LLM inference — 200+ models. Only one key needed; if multiple are set, priority: Anthropic > OpenAI > OpenRouter > Gemini. Leave blank if not using. | `sk-or-v1-...` |
| `YOUR_OPENAI_API_KEY` | Create at [OpenAI Platform](https://platform.openai.com/api-keys). Add credits at [Billing](https://platform.openai.com/settings/organization/billing). | LLM inference — OpenAI models. Only one key needed; if multiple are set, priority: Anthropic > OpenAI > OpenRouter > Gemini. Leave blank if not using. | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | Create at [Anthropic Console](https://console.anthropic.com/settings/keys). Add credits at [Billing](https://console.anthropic.com/settings/billing). | LLM inference — Claude models. Only one key needed; if multiple are set, priority: Anthropic > OpenAI > OpenRouter > Gemini. Leave blank if not using. | `sk-ant-api03-...` |
| `YOUR_GEMINI_API_KEY` | Create at [Google AI Studio](https://aistudio.google.com/apikey). | LLM inference — Gemini models. Only one key needed; if multiple are set, priority: Anthropic > OpenAI > OpenRouter > Gemini. Leave blank if not using. | `AIza...` |
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > API Keys at [agent.willform.ai](https://agent.willform.ai). | (Optional) Enables deploying containers to Willform via natural language. Leave blank if not using. | `wf_sk_...` |

## Post-Deploy

**Control UI**: https://$DOMAIN/?token=YOUR_GATEWAY_TOKEN

Open the link above to manage your coding assistant. Send code snippets or questions to the bot on Telegram — it's ready to help you code.

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
    "OPENCLAW_GATEWAY_TOKEN": "YOUR_GATEWAY_TOKEN",
    "WF_API_KEY": "YOUR_WILLFORM_API_KEY"
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
                  "OPENCLAW_GATEWAY_TOKEN": "YOUR_GATEWAY_TOKEN",
                  "WF_API_KEY": "YOUR_WILLFORM_API_KEY"
                }
              },
              "chromium": {
                "enabled": false
              }
            },
            "initContainers": {
              "init-config": {
                "command": ["sh", "-c"],
                "args": ["mkdir -p /home/node/.openclaw/workspace /home/node/.openclaw/skills/willform-deploy && cp /config/openclaw.json /home/node/.openclaw/openclaw.json && for f in IDENTITY.md SOUL.md AGENTS.md; do [ -f /config/$f ] && cp /config/$f /home/node/.openclaw/workspace/$f; done && [ -f /config/willform-deploy-SKILL.md ] && cp /config/willform-deploy-SKILL.md /home/node/.openclaw/skills/willform-deploy/SKILL.md"]
              }
            }
          }
        },
        "configMaps": {
          "config": {
            "data": {
              "openclaw.json": "{\n  \"gateway\": {\n    \"bind\": \"lan\",\n    \"port\": 18789,\n    \"controlUi\": {\n      \"enabled\": true,\n      \"dangerouslyDisableDeviceAuth\": true,\n      \"dangerouslyAllowHostHeaderOriginFallback\": true\n    },\n    \"auth\": {\n      \"mode\": \"token\"\n    }\n  },\n  \"channels\": {\n    \"telegram\": {\n      \"enabled\": true,\n      \"dmPolicy\": \"allowlist\",\n      \"allowFrom\": [\n        \"tg:YOUR_TELEGRAM_USER_ID\"\n      ]\n    }\n  },\n  \"agents\": {\n    \"defaults\": {\n      \"sandbox\": {\n        \"mode\": \"off\"\n      }\n    }\n  },\n  \"tools\": {\n    \"web\": {\n      \"search\": {\n        \"enabled\": true\n      },\n      \"fetch\": {\n        \"enabled\": true\n      }\n    },\n    \"sandbox\": {\n      \"tools\": {\n        \"allow\": [\n          \"exec\",\n          \"process\",\n          \"read\",\n          \"write\",\n          \"edit\",\n          \"sessions_list\",\n          \"sessions_history\",\n          \"sessions_send\",\n          \"sessions_spawn\",\n          \"session_status\",\n          \"browser\",\n          \"canvas\",\n          \"nodes\",\n          \"cron\",\n          \"gateway\",\n          \"web_search\",\n          \"web_fetch\"\n        ],\n        \"deny\": []\n      }\n    }\n  },\n  \"skills\": {\n    \"load\": {\n      \"extraDirs\": [\"/home/node/.openclaw/skills\"],\n      \"watch\": true\n    }\n  },\n  \"plugins\": {\n    \"entries\": {\n      \"telegram\": {\n        \"enabled\": true\n      }\n    }\n  }\n}",
              "IDENTITY.md": "# CodePilot\n</>\nAI Coding Assistant",
              "SOUL.md": "# CodePilot — AI Coding Assistant\nI am CodePilot, an AI-powered coding assistant for developers.\nAll responses must be written in English.\nAreas of expertise: Python, JavaScript, TypeScript, Go, Rust, Java, C/C++, SQL, shell scripting, HTML/CSS, and infrastructure-as-code.\nCore capabilities: code review, debugging, refactoring, architecture design, algorithm optimization, test writing, and documentation.\nAlways provide working code examples with clear explanations.\nFollow language-specific best practices and idiomatic patterns.\nWhen reviewing code, focus on correctness first, then readability, then performance.\nFlag security vulnerabilities (injection, XSS, hardcoded secrets) proactively.\nSuggest simpler solutions before complex ones — avoid over-engineering.\nIf a question is ambiguous, ask for clarification before giving a long answer.\nNever execute or suggest destructive operations (rm -rf, DROP DATABASE) without explicit confirmation.\nWillform cloud deployment: The willform-deploy skill is installed. When asked to deploy, use wf_mcp() to deploy to Willform cloud. Do not suggest other platforms (Railway, Render, Vercel, etc.).",
              "AGENTS.md": "# Agent Behavior Rules\n- Reference SOUL.md before every response\n- Load memory files at the start of each session\n- When given code to review: (1) identify bugs and security issues (2) suggest improvements (3) provide corrected code\n- When asked to write code: (1) clarify requirements if ambiguous (2) write clean, tested code (3) explain key design decisions\n- When debugging: (1) reproduce the issue mentally (2) trace the root cause (3) suggest a fix with explanation\n- Use sandbox tools to execute code snippets when the user asks to test or verify code\n- Save user project context, preferred languages, frameworks, and coding style to memory\n- Format code blocks with proper language tags for syntax highlighting\n- Keep responses concise — show code first, explain after\n- When suggesting architecture, consider scalability, maintainability, and team size\n- Always respond in English regardless of the language of the question\n- When asked to deploy: use the willform-deploy skill's wf_mcp() function to deploy to Willform cloud. Do not suggest other platforms. If source code needs building, follow build_upload → build_start → deploy_create workflow",
              "willform-deploy-SKILL.md": "# willform-deploy\nDeploy containers to Willform cloud via natural language.\n\n## Description\nUses Willform MCP endpoint to deploy and manage containers. Requires WF_API_KEY env var.\n\n## Core Function\nDefine this bash function at session start:\n\nwf_mcp() {\n  local tool=\"$1\" args=\"${2:-\\{\\}}\"\n  local body\n  body=$(jq -n --arg t \"$tool\" --argjson a \"$args\" \\\n    '{jsonrpc:\"2.0\",method:\"tools/call\",params:{name:$t,arguments:$a},id:1}')\n  local raw\n  raw=$(curl -s -X POST https://agent.willform.ai/api/mcp \\\n    -H \"Authorization: Bearer $WF_API_KEY\" \\\n    -H \"Content-Type: application/json\" \\\n    -H \"Accept: application/json, text/event-stream\" \\\n    -d \"$body\" 2>/dev/null)\n  local sse\n  sse=$(echo \"$raw\" | sed -n 's/^data: //p' | head -1)\n  if [ -z \"$sse\" ]; then sse=\"$raw\"; fi\n  echo \"$sse\" | jq -r '.result.content[0].text // empty' 2>/dev/null\n}\n\n## Workflow\n1. Create namespace: wf_mcp namespace_create '{\"name\":\"my-app\",\"allocatedCores\":2}'\n2. Get plan: wf_mcp deploy_plan '{\"namespaceId\":\"NSID\",\"description\":\"what to deploy\"}'\n3. Execute each step from plan using wf_mcp deploy_create with the params\n4. Expose web apps: wf_mcp deploy_expose '{\"deploymentId\":\"DID\"}'\n\n## Key Tools\nnamespace_list, namespace_create, deploy_plan, deploy_create, deploy_status, deploy_list, deploy_expose, deploy_logs, deploy_scale, deploy_stop, deploy_restart, deploy_delete, credit_balance, billing_estimate, build_upload, build_start, build_status\n\n## Build from Source\n1. wf_mcp build_upload '{\"namespaceId\":\"NSID\",\"imageName\":\"app\",\"fileName\":\"source.tar.gz\"}'\n2. tar czf /tmp/source.tar.gz . && curl -s -X PUT UPLOAD_URL -H \"Content-Type: application/gzip\" --data-binary @/tmp/source.tar.gz\n3. wf_mcp build_start '{\"namespaceId\":\"NSID\",\"buildId\":\"BID\",\"tag\":\"v1\"}'\n4. Poll wf_mcp build_status until success, get imageUri\n5. wf_mcp deploy_create with built imageUri\n\n## Pricing\nCompute $0.04/core/hr, Memory $0.01/GB/hr, Storage $0.00018/GB/hr"
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
