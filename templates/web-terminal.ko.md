---
category: dev-tool
tags: [terminal, claude-code, codex, gemini, ai, browser, vibe-coding]
---

# AI 터미널 브라우저 — 비개발자를 위한 AI 코딩 환경

브라우저에서 바로 사용하는 AI 터미널입니다. Claude Code, Codex, Gemini CLI가 미리 설치되어 있어 별도 설정 없이 AI에게 말로 설명하면 웹사이트나 앱을 만들 수 있습니다. 만든 결과물은 `/preview/` 경로에서 바로 확인할 수 있습니다.

## 배포 결과

- 웹 브라우저에서 접속 가능한 리눅스 터미널 (Ubuntu 24.04)
- Claude Code, Codex, Gemini CLI가 미리 설치되어 바로 사용 가능
- 바이브코딩 라이브 프리뷰 — AI가 만든 결과물을 `/preview/` 경로에서 즉시 확인
- 예쁜 쉘 환경 (Oh My Zsh + Powerlevel10k) — 자동완성, 구문 강조 포함
- Willform 플러그인과 AIATOR 튜토리얼 내장
- Claude Code와 Codex 모두에 MCP 서버 자동 연결
- 사용자명/비밀번호 기본 인증으로 보호
- 개발 도구 포함: git, gh, python3, ripgrep, fzf, tmux, htop 등

## 사전 준비

아래 값들을 입력해 주세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_TERMINAL_USER` | 원하는 사용자명을 직접 정합니다 | 터미널 로그인 사용자명 | `admin` |
| `YOUR_TERMINAL_PASSWORD` | 안전한 비밀번호를 직접 정합니다 | 터미널 로그인 비밀번호 | `my-secret-pw` |
| `YOUR_WILLFORM_API_KEY` | [agent.willform.ai](https://agent.willform.ai) > Settings > API Keys에서 발급. 페이지 우측 자동 생성 버튼으로도 발급 가능. | Claude Code와 Codex에서 Willform MCP 도구를 사용합니다. | `wf_sk_...` |

## 배포 후 안내

**터미널**: https://$DOMAIN

**라이브 프리뷰**: https://$DOMAIN/preview/

위 링크를 열고 설정한 사용자명과 비밀번호로 로그인하세요. 접속 후:

1. **Claude Code**: `claude` 입력 → 안내에 따라 로그인 → AI에게 원하는 걸 말하면 끝
2. **Codex**: `codex` 입력 → 안내에 따라 로그인 → AI에게 원하는 걸 말하면 끝
3. **Gemini**: `gemini` 입력 → 안내에 따라 로그인 → AI에게 원하는 걸 말하면 끝

**바이브코딩**: AI에게 "포트 3000번으로 웹사이트 만들어줘"라고 하면, 만든 결과를 `/preview/` 경로에서 바로 볼 수 있습니다.

`help-tools`를 입력하면 사용 가능한 명령어를 볼 수 있습니다.

## Deploy Config

```json
{
  "name": "terminal",
  "chartType": "web",
  "image": "registry.willform.ai/library/web-terminal:ko",
  "port": 7681,
  "replicas": 1,
  "env": {
    "TERMINAL_USER": "YOUR_TERMINAL_USER",
    "TERMINAL_PASSWORD": "YOUR_TERMINAL_PASSWORD",
    "TERMINAL_LANG": "ko",
    "WF_API_KEY": "YOUR_WILLFORM_API_KEY"
  },
  "expose": true,
  "exposeProtocol": "http"
}
```
