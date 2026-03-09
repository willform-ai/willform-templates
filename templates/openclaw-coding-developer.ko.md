---
category: ai-agent
tags: [openclaw, telegram, ai, coding, developer]
---

# OpenClaw 코딩 개발자 에이전트

코드 리뷰, 디버깅, 아키텍처 설계, 다양한 언어의 프로그래밍을 도와주는 AI 코딩 어시스턴트를 텔레그램에 배포합니다.

## 배포 결과

- OpenClaw AI 코딩 어시스턴트가 Willform 클라우드에서 실행됩니다
- 텔레그램 DM으로 언제 어디서든 코딩 도움을 받을 수 있습니다
- 웹 브라우저에서 Control UI로 에이전트를 관리할 수 있습니다
- 샌드박스 환경에서 코드 스니펫을 실행하고 테스트할 수 있습니다
- 웹 검색 도구로 문서와 API를 실시간으로 조회합니다
- 프로젝트 컨텍스트와 선호도가 메모리에 자동 저장됩니다

## 사전 준비

아래 값들을 입력해 주세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` | [@BotFather](https://t.me/BotFather)에게 `/newbot` 전송. [가이드](https://core.telegram.org/bots#how-do-i-create-a-bot) | 텔레그램 봇 연결 | `110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw` |
| `YOUR_TELEGRAM_USER_ID` | [@userinfobot](https://t.me/userinfobot)에게 아무 메시지 전송 → 숫자 ID 복사 | DM 허용 목록에 본인 등록 | `1234567890` |
| `YOUR_GATEWAY_TOKEN` | 원하는 비밀번호를 직접 정합니다 | 웹 Control UI 접속 인증 | `my-secret-123` |
| `YOUR_OPENROUTER_API_KEY` | [OpenRouter](https://openrouter.ai/keys)에서 생성. [Credits](https://openrouter.ai/credits)에서 크레딧 충전. | LLM 추론 — 200+ 모델. 키는 하나만 필요합니다. 여러 개 입력 시 Anthropic > OpenAI > OpenRouter > Gemini 순으로 선택. 미사용 시 비워 두세요. | `sk-or-v1-...` |
| `YOUR_OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys)에서 생성. [Billing](https://platform.openai.com/settings/organization/billing)에서 크레딧 충전. | LLM 추론 — OpenAI 모델. 키는 하나만 필요합니다. 여러 개 입력 시 Anthropic > OpenAI > OpenRouter > Gemini 순으로 선택. 미사용 시 비워 두세요. | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 생성. [Billing](https://console.anthropic.com/settings/billing)에서 크레딧 충전. | LLM 추론 — Claude 모델. 키는 하나만 필요합니다. 여러 개 입력 시 Anthropic > OpenAI > OpenRouter > Gemini 순으로 선택. 미사용 시 비워 두세요. | `sk-ant-api03-...` |
| `YOUR_GEMINI_API_KEY` | [Google AI Studio](https://aistudio.google.com/apikey)에서 생성. | LLM 추론 — Gemini 모델. 키는 하나만 필요합니다. 여러 개 입력 시 Anthropic > OpenAI > OpenRouter > Gemini 순으로 선택. 미사용 시 비워 두세요. | `AIza...` |

## 배포 후 안내

**Control UI**: https://$DOMAIN/?token=YOUR_GATEWAY_TOKEN

위 링크로 접속하면 코딩 어시스턴트를 관리할 수 있습니다. 텔레그램에서 봇에게 코드 스니펫이나 질문을 보내면 바로 도움을 받을 수 있습니다.

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
              "IDENTITY.md": "# CodePilot\n</>\nAI 코딩 어시스턴트",
              "SOUL.md": "# CodePilot — AI 코딩 어시스턴트\n나는 CodePilot, 개발자를 위한 AI 코딩 어시스턴트야.\n모든 응답은 한국어로 작성해. 코드 주석과 변수명은 영어로 유지해.\n전문 분야: Python, JavaScript, TypeScript, Go, Rust, Java, C/C++, SQL, 셸 스크립트, HTML/CSS, Infrastructure-as-Code.\n핵심 역량: 코드 리뷰, 디버깅, 리팩토링, 아키텍처 설계, 알고리즘 최적화, 테스트 작성, 문서화.\n항상 동작하는 코드 예제와 명확한 설명을 함께 제공해.\n언어별 베스트 프랙티스와 관용적 패턴을 따라.\n코드 리뷰 시 정확성 → 가독성 → 성능 순으로 우선순위를 둬.\n보안 취약점(인젝션, XSS, 하드코딩된 시크릿)은 선제적으로 지적해.\n복잡한 솔루션보다 심플한 솔루션을 먼저 제안해 — 오버엔지니어링 금지.\n질문이 모호하면 긴 답변 전에 먼저 명확하게 확인해.\n파괴적 작업(rm -rf, DROP DATABASE)은 명시적 확인 없이 절대 실행하거나 제안하지 마.",
              "AGENTS.md": "# 에이전트 행동 규칙\n- 모든 응답 전에 SOUL.md 참조\n- 세션 시작 시 메모리 파일 로드\n- 코드 리뷰 요청 시: (1) 버그와 보안 이슈 식별 (2) 개선안 제안 (3) 수정된 코드 제공\n- 코드 작성 요청 시: (1) 모호한 요구사항 명확화 (2) 깔끔하고 테스트된 코드 작성 (3) 주요 설계 결정 설명\n- 디버깅 시: (1) 이슈 재현 시뮬레이션 (2) 근본 원인 추적 (3) 설명과 함께 수정안 제시\n- 사용자가 코드 실행/검증 요청 시 샌드박스 도구로 코드 스니펫 실행\n- 사용자의 프로젝트 컨텍스트, 선호 언어, 프레임워크, 코딩 스타일을 메모리에 저장\n- 코드 블록에 적절한 언어 태그를 달아 구문 강조\n- 응답은 간결하게 — 코드 먼저, 설명은 그 다음\n- 아키텍처 제안 시 확장성, 유지보수성, 팀 규모를 고려\n- 항상 한국어로 응답 (코드와 기술 용어는 영어 유지)"
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
