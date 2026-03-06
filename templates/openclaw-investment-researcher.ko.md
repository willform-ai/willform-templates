---
category: ai-agent
tags: [openclaw, telegram, ai, finance]
---

# OpenClaw 투자 리서쳐 에이전트

텔레그램에서 주식, ETF, 암호화폐 등 투자 관련 질문에 답하는 AI 리서쳐 챗봇을 배포합니다.

## 배포 결과

- OpenClaw 기반 AI 챗봇이 Willform 클라우드에서 실행됩니다
- 텔레그램 DM으로 챗봇과 대화할 수 있습니다
- 웹 브라우저에서 Control UI로 챗봇을 관리할 수 있습니다
- 챗봇은 웹 검색을 통해 실시간 시장 데이터를 조회합니다
- 대화 내역과 사용자 선호도가 자동으로 메모리에 저장됩니다

## 사전 준비

아래 4개의 값을 입력해 주세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | 1. [Anthropic Console](https://console.anthropic.com/settings/keys)에 접속해서 로그인합니다 (계정이 없으면 가입). 2. **Create Key** 버튼을 눌러 키를 생성하고 `sk-ant-...`로 시작하는 키를 복사합니다. 3. [Plans & Billing](https://console.anthropic.com/settings/billing)에서 크레딧을 충전해야 에이전트가 동작합니다. | 챗봇의 LLM 추론에 사용 | `sk-ant-...` |
| `YOUR_TELEGRAM_BOT_TOKEN` | 1. Telegram에서 [@BotFather](https://t.me/BotFather)를 검색합니다. 2. `/newbot`을 보내고 안내에 따라 봇 이름과 `bot`으로 끝나는 유저네임을 설정합니다. 3. BotFather가 `123456:ABC-DEF...` 형식의 토큰을 알려줍니다 — 전체를 복사합니다. 자세한 내용은 [Telegram Bot API 문서](https://core.telegram.org/bots#how-do-i-create-a-bot)를 참고하세요. | 텔레그램 봇 연결 | `1234567890:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` |
| `YOUR_TELEGRAM_USER_ID` | 1. Telegram에서 [@userinfobot](https://t.me/userinfobot)을 검색합니다. 2. 아무 메시지(예: "안녕")를 보냅니다. 3. 봇이 숫자로 된 유저 ID(예: `123456789`)를 알려줍니다 — 숫자만 복사합니다. | DM 허용 목록에 본인 등록 | `1234567890` |
| `YOUR_GATEWAY_TOKEN` | 원하는 비밀번호를 직접 정합니다 (예: `my-secret-123`). 외부 서비스에서 발급받는 것이 아니라 본인이 만드는 값입니다. 웹 Control UI 접속 시 인증에 사용됩니다. | 웹 Control UI 접속 인증 | `my-secret-123` |

## 배포 후 안내

**Control UI**: https://$DOMAIN/?token=YOUR_GATEWAY_TOKEN

위 링크로 접속하면 챗봇을 관리할 수 있습니다. 텔레그램에서 봇에게 메시지를 보내면 바로 투자 관련 질문에 답합니다.

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
              "IDENTITY.md": "# 상승이\n📊\n투자 전문 AI 리서쳐",
              "SOUL.md": "# 상승이 — 투자 전문 리서쳐\n나는 상승이, 투자 전문 AI 리서쳐다.\n모든 응답은 반드시 한국어로 작성한다. 존댓말을 사용한다.\n전문 분야: 주식, ETF, 채권, 암호화폐, 매크로 경제, 섹터 분석, 포트폴리오 전략.\n데이터 기반 분석을 우선하고, 출처를 반드시 명시한다.\n투자 수익을 보장하거나 특정 종목을 매수/매도 추천하지 않는다.\n최종 투자 판단은 사용자 본인의 책임임을 안내한다.",
              "AGENTS.md": "# Agent 행동 규칙\n- 매 응답 전 SOUL.md 참조\n- 세션마다 memory 파일 로드\n- 분석 요청 시 근거 데이터와 출처 필수 포함\n- 유저 투자성향/관심종목/포트폴리오 memory에 저장\n- 그룹챗에서 개인 포트폴리오 정보 공유 금지\n- 어떤 언어로 질문받아도 반드시 한국어로 응답"
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
