---
category: ai-agent
tags: [openclaw, telegram, ai, custom]
---

# OpenClaw 커스텀 에이전트

나만의 정체성, 성격, 행동 규칙을 가진 AI 에이전트를 텔레그램에 배포합니다.

## 배포 결과

- 사용자가 직접 정의한 성격의 OpenClaw AI 에이전트가 Willform 클라우드에서 실행됩니다
- 텔레그램 DM으로 에이전트와 대화할 수 있습니다
- 웹 브라우저에서 Control UI로 에이전트를 관리할 수 있습니다
- 웹 검색/조회 도구로 실시간 정보를 활용합니다
- 대화 내역과 사용자 선호도가 자동으로 메모리에 저장됩니다

## 사전 준비

아래 값들을 입력해 주세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` | [@BotFather](https://t.me/BotFather)에게 `/newbot` 전송. [가이드](https://core.telegram.org/bots#how-do-i-create-a-bot) | 텔레그램 봇 연결 | `110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw` |
| `YOUR_TELEGRAM_USER_ID` | [@userinfobot](https://t.me/userinfobot)에게 아무 메시지 전송 → 숫자 ID 복사 | DM 허용 목록에 본인 등록 | `1234567890` |
| `YOUR_GATEWAY_TOKEN` | 원하는 비밀번호를 직접 정합니다 | 웹 Control UI 접속 인증 | `my-secret-123` |
| `YOUR_IDENTITY_MD` | 직접 작성 (아래 가이드 참조) | 에이전트의 이름, 이모지, 역할 정의 | `# Identity\nName: 땅선생\nCreature: 부동산 AI 감정평가사...\nVibe: 냉철하고 숫자에 강한...\nEmoji: 🏠` |
| `YOUR_SOUL_MD` | 직접 작성 (아래 가이드 참조) | 에이전트의 성격, 전문성, 말투 정의 | `# Soul\n## Core Truths\n- 데이터 기반 판단만...\n## Boundaries\n- 특정 매물 추천 금지...\n## Vibe\n...` |
| `YOUR_AGENTS_MD` | 직접 작성 (아래 가이드 참조) | 에이전트의 행동 규칙과 제약 조건 정의 | `# Agents\n## 세션 시작 루틴\n1. SOUL.md 읽기...\n## 메모리 관리\n- memory/에 기록...\n## 분석 프레임워크\n...` |

### Provider: OpenRouter (default)

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_OPENROUTER_API_KEY` | [OpenRouter](https://openrouter.ai/keys)에서 생성. [Credits](https://openrouter.ai/credits)에서 크레딧 충전. | LLM 추론 — 200+ 모델 | `sk-or-v1-...` |

### Provider: Anthropic

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 생성. [Billing](https://console.anthropic.com/settings/billing)에서 크레딧 충전. | LLM 추론 — Claude 모델 | `sk-ant-api03-...` |

### Provider: OpenAI

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys)에서 생성. [Billing](https://platform.openai.com/settings/organization/billing)에서 크레딧 충전. | LLM 추론 — OpenAI 모델 | `sk-proj-...` |

### Provider: Google Gemini

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_GEMINI_API_KEY` | [Google AI Studio](https://aistudio.google.com/apikey)에서 생성. | LLM 추론 — Gemini 모델 | `AIza...` |

### IDENTITY.md 작성 가이드

에이전트의 이름표를 정의합니다 — 이름, 역할, 분위기.

예시:
```
# Identity

Name: ...
Creature: ...
Vibe: ...
Emoji: ...
```

### SOUL.md 작성 가이드

에이전트의 성격을 정의합니다 — 핵심 가치관, 경계, 커뮤니케이션 스타일.

예시:
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

### AGENTS.md 작성 가이드

에이전트의 업무 매뉴얼을 정의합니다 — 시작 루틴, 메모리 관리, 업무별 프레임워크.

예시:
```
# Agents

## 세션 시작 루틴
1. ...

## 메모리 관리
- ...

## 분석 프레임워크
1. ...
2. ...
3. ...
```

## 배포 후 안내

**Control UI**: https://$DOMAIN/?token=YOUR_GATEWAY_TOKEN

위 링크로 접속하면 에이전트를 관리할 수 있습니다. 텔레그램에서 봇에게 메시지를 보내면 바로 사용할 수 있습니다.

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
