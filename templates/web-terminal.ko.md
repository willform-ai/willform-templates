---
category: dev-tool
tags: [terminal, claude-code, codex, gemini, ai, browser]
---

# 웹 터미널 — 브라우저에서 AI 코딩 도구 사용하기

브라우저에서 바로 사용하는 리눅스 터미널입니다. Claude Code, Codex, Gemini CLI가 미리 설치되어 있어 별도 설정 없이 바로 AI와 코딩할 수 있습니다.

## 배포 결과

- 웹 브라우저에서 접속 가능한 리눅스 터미널 (Ubuntu 24.04)
- Claude Code, Codex, Gemini CLI가 미리 설치되어 바로 사용 가능
- 예쁜 쉘 환경 (Oh My Zsh + Powerlevel10k) — 자동완성, 구문 강조 포함
- Willform 플러그인 (16개 슬래시 명령어)으로 클라우드 배포
- Claude Code와 Codex 모두에 MCP 서버 자동 연결
- 사용자명/비밀번호 기본 인증으로 보호
- 개발 도구 포함: git, gh, python3, ripgrep, fzf, tmux, htop 등

## 사전 준비

아래 값들을 입력해 주세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_TERMINAL_USER` | 원하는 사용자명을 직접 정합니다 | 터미널 로그인 사용자명 | `admin` |
| `YOUR_TERMINAL_PASSWORD` | 안전한 비밀번호를 직접 정합니다 | 터미널 로그인 비밀번호 | `my-secret-pw` |
| `YOUR_WILLFORM_API_KEY` | [agent.willform.ai](https://agent.willform.ai) > Settings > API Keys에서 발급. | (선택 사항) Claude Code와 Codex에서 Willform MCP 도구를 사용합니다. 미사용 시 비워 두세요. | `wf_sk_...` |

## 배포 후 안내

**터미널**: https://$DOMAIN

위 링크를 열고 설정한 사용자명과 비밀번호로 로그인하세요. 접속 후:

1. **Claude Code**: `claude auth login` 실행 → 브라우저 링크 따라가기 → Anthropic 로그인 → `cc`로 코딩 시작
2. **Codex**: `codex login` 실행 → 브라우저 링크 따라가기 → OpenAI 로그인 → `codex`로 시작
3. **Gemini**: `gemini` 실행 → 첫 실행 시 자동으로 로그인 안내 → 브라우저 링크 따라가기 → Google 로그인

`help-tools`를 입력하면 사용 가능한 AI 도구와 명령어를 한눈에 볼 수 있습니다.

## Deploy Config

```json
{
  "name": "terminal",
  "chartType": "web",
  "image": "ghcr.io/willform-ai/web-terminal:latest",
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
