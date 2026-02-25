# OpenClaw 투자 리서처 에이전트

텔레그램으로 소통하는 AI 투자 리서처 챗봇을 Willform에 배포합니다.

## 사전 준비

아래 4개의 값을 본인 것으로 교체한 뒤, 프롬프트 전체를 AI 에이전트에 붙여넣으세요.

| 변수 | 어디서 발급? | 예시 형태 |
|---|---|---|
| `YOUR_ANTHROPIC_API_KEY` | https://console.anthropic.com/settings/keys | `sk-ant-api03-...` |
| `YOUR_TELEGRAM_BOT_TOKEN` | Telegram에서 @BotFather → /newbot | `1234567890:AAG...` |
| `YOUR_TELEGRAM_USER_ID` | Telegram에서 @userinfobot에게 아무 메시지 → 숫자 ID | `8514287619` |
| `YOUR_GATEWAY_TOKEN` | 아무 비밀번호 (본인이 정하면 됨) | `my-secret-123` |

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

시작 커맨드 (sh -c):

1) openclaw.json 작성:
cat > /home/node/.openclaw/openclaw.json << 'OCJSON'
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "port": 18790,
    "controlUi": { "enabled": true, "dangerouslyDisableDeviceAuth": true },
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

5) HTTP 리버스 프록시 작성 후 백그라운드 실행 (Cloudflare 헤더 제거 + Host/Origin localhost rewrite):
cat > /home/node/.openclaw/proxy.js << 'PROXYJS'
const http = require('http');
const net = require('net');
const STRIP = [
  'x-forwarded-for', 'x-forwarded-proto', 'x-real-ip',
  'cf-connecting-ip', 'cf-ray', 'cf-visitor',
  'cf-ipcountry', 'cdn-loop', 'cf-worker'
];
const LOCAL = '127.0.0.1:18790';

const server = http.createServer((req, res) => {
  STRIP.forEach(k => delete req.headers[k]);
  req.headers.host = LOCAL;
  if (req.headers.origin) req.headers.origin = 'http://' + LOCAL;
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
  req.headers.host = LOCAL;
  if (req.headers.origin) req.headers.origin = 'http://' + LOCAL;
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

배포 후 도메인 노출하고, 첫 접속은:
https://{도메인}/?token={YOUR_GATEWAY_TOKEN}
```
