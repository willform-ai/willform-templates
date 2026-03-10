---
category: dev-tool
tags: [terminal, claude-code, codex, gemini, ai, browser]
---

# Web Terminal — AI Coding Tools in Your Browser

A browser-based Linux terminal with Claude Code, Codex, and Gemini CLI pre-installed. No local setup needed — just deploy and start coding with AI.

## What You Get

- A full Linux terminal (Ubuntu 24.04) accessible from any web browser
- Claude Code, Codex, and Gemini CLI pre-installed and ready to use
- Pretty shell (Oh My Zsh + Powerlevel10k) with autosuggestions and syntax highlighting
- Willform plugin with 16 slash commands for cloud deployment
- MCP server pre-configured for both Claude Code and Codex
- Basic auth protection via username/password
- Developer tools: git, gh, python3, ripgrep, fzf, tmux, htop, and more

## Prerequisites

Replace the placeholder values below with your own.

| Variable | How to get it | Purpose | Example |
|---|---|---|---|
| `YOUR_TERMINAL_USER` | Choose any username | Terminal login username | `admin` |
| `YOUR_TERMINAL_PASSWORD` | Choose a strong password | Terminal login password | `my-secret-pw` |
| `YOUR_WILLFORM_API_KEY` | Dashboard > Settings > API Keys at [agent.willform.ai](https://agent.willform.ai). | (Optional) Enables Willform MCP tools inside Claude Code and Codex. Leave blank if not using. | `wf_sk_...` |

## Post-Deploy

**Terminal**: https://$DOMAIN

Open the link above and log in with the username and password you set. Once inside:

1. **Claude Code**: Run `claude auth login` → follow the browser link → sign in to Anthropic → then use `cc` to start coding
2. **Codex**: Run `codex login` → follow the browser link → sign in to OpenAI → then use `codex` to start
3. **Gemini**: Run `gemini` → it will auto-prompt login on first run → follow the browser link → sign in to Google

Type `help-tools` for a quick reference of all available AI tools and commands.

## Deploy Config

```json
{
  "name": "terminal",
  "chartType": "web",
  "image": "registry.willform.ai/library/web-terminal:latest",
  "port": 7681,
  "replicas": 1,
  "env": {
    "TERMINAL_USER": "YOUR_TERMINAL_USER",
    "TERMINAL_PASSWORD": "YOUR_TERMINAL_PASSWORD",
    "TERMINAL_LANG": "en",
    "WF_API_KEY": "YOUR_WILLFORM_API_KEY"
  },
  "expose": true,
  "exposeProtocol": "http"
}
```
