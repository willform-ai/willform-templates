---
category: ai-agent
tags: [openclaw, telegram, ai, finance]
---

# OpenClaw 투자 리서처 에이전트

텔레그램에서 주식, ETF, 암호화폐 등 투자 관련 질문에 답하는 AI 리서처 챗봇을 배포합니다.

## 배포 결과

- OpenClaw 기반 AI 챗봇이 Willform 클라우드에서 실행됩니다
- 텔레그램 DM으로 챗봇과 대화할 수 있습니다
- 웹 브라우저에서 Control UI로 챗봇을 관리할 수 있습니다
- 챗봇은 웹 검색을 통해 실시간 시장 데이터를 조회합니다
- 대화 내역과 사용자 선호도가 자동으로 메모리에 저장됩니다

## 사전 준비

아래 4개의 값을 본인 것으로 교체한 뒤, 프롬프트 전체를 AI 에이전트에 붙여넣으세요.

| 변수 | 발급 방법 | 용도 |
|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 API 키 생성 | 챗봇의 LLM 추론에 사용 |
| `YOUR_TELEGRAM_BOT_TOKEN` | Telegram에서 [@BotFather](https://t.me/BotFather)에게 `/newbot` 명령 | 텔레그램 봇 연결 |
| `YOUR_TELEGRAM_USER_ID` | Telegram에서 [@userinfobot](https://t.me/userinfobot)에게 아무 메시지 전송 → 숫자 ID 확인 | DM 허용 목록에 본인 등록 |
| `YOUR_GATEWAY_TOKEN` | 직접 정하는 비밀번호 (예: `my-secret-123`) | 웹 Control UI 접속 인증 |

## 프롬프트

```
OpenClaw AI 에이전트를 배포해주세요.

이미지: ghcr.io/openclaw/openclaw:2026.2.23
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
    OPENCLAW_DOMAIN: (expose 후 설정 — 아래 안내 참조)

시작 커맨드 (sh -c):

1) openclaw.json 작성 (주의: $OPENCLAW_DOMAIN 변수 확장을 위해 unquoted heredoc 사용):
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

2) IDENTITY.md 작성:
cat > /home/node/.openclaw/workspace/IDENTITY.md << 'EOF'
# 상승이
📊
투자 전문 AI 리서처
EOF

3) SOUL.md 작성:
cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
# 상승이 — 투자 전문 리서처
나는 상승이, 투자 전문 AI 리서처다.
모든 응답은 반드시 한국어로 작성한다. 존댓말을 사용한다.
전문 분야: 주식, ETF, 채권, 암호화폐, 매크로 경제, 섹터 분석, 포트폴리오 전략.
데이터 기반 분석을 우선하고, 출처를 반드시 명시한다.
투자 수익을 보장하거나 특정 종목을 매수/매도 추천하지 않는다.
최종 투자 판단은 사용자 본인의 책임임을 안내한다.
EOF

4) AGENTS.md 작성:
cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
# Agent 행동 규칙
- 매 응답 전 SOUL.md 참조
- 세션마다 memory 파일 로드
- 분석 요청 시 근거 데이터와 출처 필수 포함
- 유저 투자성향/관심종목/포트폴리오 memory에 저장
- 그룹챗에서 개인 포트폴리오 정보 공유 금지
- 어떤 언어로 질문받아도 반드시 한국어로 응답
EOF

5) HTTP 리버스 프록시 작성 후 백그라운드 실행 (Cloudflare 헤더만 제거 — Host/Origin 재작성 금지):
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

6) 게이트웨이 실행:
exec node dist/index.js gateway --allow-unconfigured

배포 후:
1. 도메인 노출 (POST /api/deploy/{id}/expose) → {도메인} 반환
2. 환경변수 업데이트: OPENCLAW_DOMAIN={도메인} (재시작 트리거, allowedOrigins 적용)
3. 첫 접속: https://{도메인}/?token={YOUR_GATEWAY_TOKEN}
```

## Deploy Config

```json
{
  "name": "openclaw",
  "image": "ghcr.io/openclaw/openclaw:2026.2.23",
  "chartType": "web",
  "port": 18789,
  "replicas": 1,
  "volumeSizeGb": 8,
  "volumeMountPath": "/home/node/.openclaw",
  "healthCheckPath": null,
  "env": {
    "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
    "TELEGRAM_BOT_TOKEN": "YOUR_TELEGRAM_BOT_TOKEN",
    "OPENCLAW_GATEWAY_TOKEN": "YOUR_GATEWAY_TOKEN",
    "OPENCLAW_DOMAIN": ""
  },
  "command": ["sh", "-c"],
  "expose": true,
  "exposeProtocol": "http",
  "postExposeEnv": {
    "OPENCLAW_DOMAIN": "$DOMAIN"
  }
}
```

## Startup Script

```bash
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

cat > /home/node/.openclaw/workspace/IDENTITY.md << 'EOF'
# 상승이
📊
투자 전문 AI 리서처
EOF

cat > /home/node/.openclaw/workspace/SOUL.md << 'EOF'
# 상승이 — 투자 전문 리서처
나는 상승이, 투자 전문 AI 리서처다.
모든 응답은 반드시 한국어로 작성한다. 존댓말을 사용한다.
전문 분야: 주식, ETF, 채권, 암호화폐, 매크로 경제, 섹터 분석, 포트폴리오 전략.
데이터 기반 분석을 우선하고, 출처를 반드시 명시한다.
투자 수익을 보장하거나 특정 종목을 매수/매도 추천하지 않는다.
최종 투자 판단은 사용자 본인의 책임임을 안내한다.
EOF

cat > /home/node/.openclaw/workspace/AGENTS.md << 'EOF'
# Agent 행동 규칙
- 매 응답 전 SOUL.md 참조
- 세션마다 memory 파일 로드
- 분석 요청 시 근거 데이터와 출처 필수 포함
- 유저 투자성향/관심종목/포트폴리오 memory에 저장
- 그룹챗에서 개인 포트폴리오 정보 공유 금지
- 어떤 언어로 질문받아도 반드시 한국어로 응답
EOF

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

exec node dist/index.js gateway --allow-unconfigured
```
