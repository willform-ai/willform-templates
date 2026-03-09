# Willform Prompt Templates

Ready-to-use deployment prompts for AI agents on [Willform](https://agent.willform.ai) — the autonomous infrastructure cloud.

Copy a prompt, paste it into your AI agent, and deploy infrastructure in seconds.

## How to Use

1. Connect your AI agent to Willform via [MCP](https://agent.willform.ai/docs/mcp-setup) or [A2A](https://agent.willform.ai/docs/a2a-setup)
2. Browse the templates below
3. Copy the prompt and paste it into your agent
4. Replace placeholder values (marked with `YOUR_*`) with your own credentials
5. Your agent deploys everything autonomously

## Templates

| Template | Description | Languages |
|----------|-------------|-----------|
| [OpenClaw Investment Researcher](./templates/openclaw-investment-researcher.en.md) | AI chatbot on Telegram for stocks, ETFs, crypto, and market analysis | [English](./templates/openclaw-investment-researcher.en.md) · [한국어](./templates/openclaw-investment-researcher.ko.md) |
| [OpenClaw Coding Developer](./templates/openclaw-coding-developer.en.md) | AI coding assistant for code review, debugging, and architecture | [English](./templates/openclaw-coding-developer.en.md) · [한국어](./templates/openclaw-coding-developer.ko.md) |
| [OpenClaw Custom Agent](./templates/openclaw-custom-agent.en.md) | Deploy a custom OpenClaw agent with your own configuration | [English](./templates/openclaw-custom-agent.en.md) · [한국어](./templates/openclaw-custom-agent.ko.md) |

## Creating a Template

Each template is a markdown file in `templates/` with a specific structure:

```
templates/{slug}.{lang}.md
```

- `slug` — template ID (e.g., `my-app`)
- `lang` — `en` or `ko`

### Required Sections

| Section | Heading (EN) | Heading (KO) | Purpose |
|---------|-------------|-------------|---------|
| Title | `# ...` | `# ...` | First H1 heading |
| Description | (paragraph) | (paragraph) | First paragraph after title |
| Highlights | `## What You Get` | `## 배포 결과` | Bullet list of features |
| Prerequisites | `## Prerequisites` | `## 사전 준비` | Table of required variables |
| Deploy Config | `## Deploy Config` | `## Deploy Config` | JSON code block (required) |

### Optional Sections

| Section | Heading | Purpose |
|---------|---------|---------|
| Post-Deploy | `## Post-Deploy` / `## 배포 후 안내` | Instructions after deployment |
| Startup Script | `## Startup Script` | Bash script for initialization |

### Frontmatter

```yaml
---
category: ai-agent
tags: [openclaw, telegram, ai, finance]
---
```

Valid categories: `web-app`, `database`, `cache`, `queue`, `ai-agent`, `background-job`, `full-stack`

### Post-Deploy Variables

Post-deploy sections support automatic variable substitution:

- `$NAME` — deployment name
- `$DOMAIN` — exposed domain
- `$DEPLOYMENT_ID` — deployment UUID
- `$INTERNAL_ENDPOINT` — cluster-internal endpoint
- `$STATUS` — deployment status
- Any prerequisite variable (e.g., `YOUR_API_KEY`)

## Guides

- [Connect Willform MCP to OpenClaw](./docs/openclaw-mcp-setup.md) — Give your OpenClaw agent access to 42 Willform deployment tools via MCP

## Contributing

1. Fork this repo
2. Copy [`TEMPLATE.md`](./TEMPLATE.md) as a starting point
3. Add your template as `templates/{name}.en.md` (Korean: `{name}.ko.md`)
4. Open a Pull Request

## Browse Online

Visit [agent.willform.ai/templates](https://agent.willform.ai/templates) for an interactive gallery with search, filtering, and one-click copy.

## License

[MIT](./LICENSE)
