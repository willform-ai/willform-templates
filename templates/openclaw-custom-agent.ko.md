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

아래 7개의 값을 본인 것으로 교체한 뒤, 프롬프트 전체를 AI 에이전트에 붙여넣으세요.

| 변수 | 발급 방법 | 용도 |
|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 API 키 생성 | 에이전트의 LLM 추론에 사용 |
| `YOUR_TELEGRAM_BOT_TOKEN` | Telegram에서 [@BotFather](https://t.me/BotFather)에게 `/newbot` 명령 | 텔레그램 봇 연결 |
| `YOUR_TELEGRAM_USER_ID` | Telegram에서 [@userinfobot](https://t.me/userinfobot)에게 아무 메시지 전송 → 숫자 ID 확인 | DM 허용 목록에 본인 등록 |
| `YOUR_GATEWAY_TOKEN` | 직접 정하는 비밀번호 (예: `my-secret-123`) | 웹 Control UI 접속 인증 |
| `YOUR_IDENTITY_MD` | 직접 작성 (아래 가이드 참조) | 에이전트의 이름, 이모지, 역할 정의 |
| `YOUR_SOUL_MD` | 직접 작성 (아래 가이드 참조) | 에이전트의 성격, 전문성, 말투 정의 |
| `YOUR_AGENTS_MD` | 직접 작성 (아래 가이드 참조) | 에이전트의 행동 규칙과 제약 조건 정의 |

### IDENTITY.md 작성 가이드

에이전트가 누구인지 정의하는 3줄입니다:

```
# 에이전트이름
🤖
역할 한 줄 설명
```

예시:
```
# 아틀라스
🌍
AI 여행 플래닝 어시스턴트
```

### SOUL.md 작성 가이드

에이전트의 성격을 정의하는 짧은 문서입니다. 포함할 내용:
- 에이전트의 이름과 역할
- 언어 및 말투 설정
- 전문 분야
- 핵심 제약 조건 또는 경계

예시:
```
# 아틀라스 — AI 여행 플래너
나는 아틀라스, AI 여행 플래닝 어시스턴트다.
모든 응답은 반드시 한국어로 작성한다. 친절하고 활기찬 말투를 사용한다.
전문 분야: 항공편 검색, 호텔 추천, 일정 계획, 현지 맛집, 문화 팁.
가성비 좋은 옵션을 우선 제안하고, 비자 요건을 반드시 언급한다.
사용자를 대신해서 예약하지 않는다 — 제안과 정보 제공만 한다.
```

### AGENTS.md 작성 가이드

에이전트가 따라야 할 행동 규칙 목록입니다:

예시:
```
# Agent 행동 규칙
- 매 응답 전 SOUL.md 참조
- 세션마다 memory 파일 로드
- 항상 가격대별 여러 옵션을 제시
- 유저 여행 선호도와 과거 여행 이력을 memory에 저장
- 그룹챗에서 개인 정보 공유 금지
- 어떤 언어로 질문받아도 반드시 한국어로 응답
```

## 프롬프트

```
OpenClaw AI 에이전트를 배포해주세요.

이미지: alpine/openclaw:2026.2.26
이름: openclaw
타입: web
포트: 18789
리소스: 1 코어, 2GB 메모리
볼륨: /home/node/.openclaw, 8GB
헬스체크: null (문자열 "null")
레플리카: 1

환경변수:
    ANTHROPIC_API_KEY: "YOUR_ANTHROPIC_API_KEY"
    TELEGRAM_BOT_TOKEN: "YOUR_TELEGRAM_BOT_TOKEN"
    OPENCLAW_GATEWAY_TOKEN: "YOUR_GATEWAY_TOKEN"

시작 커맨드 (sh -c):

1) openclaw.json 작성:
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

2) workspace 디렉토리 생성 및 IDENTITY.md 작성:
mkdir -p /home/node/.openclaw/workspace
cat > /home/node/.openclaw/workspace/IDENTITY.md << 'EOF'
YOUR_IDENTITY_MD
EOF

3) SOUL.md 작성:
cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
YOUR_SOUL_MD
EOF

4) AGENTS.md 작성:
cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
YOUR_AGENTS_MD
EOF

5) 게이트웨이 실행:
exec node openclaw.mjs gateway --allow-unconfigured --bind lan

배포 후:
1. 도메인 노출 (POST /api/deploy/{id}/expose) → {도메인} 반환
2. 첫 접속: https://{도메인}/?token={YOUR_GATEWAY_TOKEN}
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
YOUR_IDENTITY_MD
EOF

cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
YOUR_SOUL_MD
EOF

cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
YOUR_AGENTS_MD
EOF

exec node openclaw.mjs gateway --allow-unconfigured --bind lan
```
