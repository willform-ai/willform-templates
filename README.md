# Willform Prompt Templates

Ready-to-use deployment prompts for AI agents on [Willform](https://agent.willform.ai) — the autonomous infrastructure cloud.

Copy a prompt, paste it into your AI agent (Claude Desktop, ChatGPT, etc.), and deploy infrastructure in seconds.

## How to Use

1. Connect your AI agent to Willform via [MCP](https://agent.willform.ai/docs/mcp-setup) or [A2A](https://agent.willform.ai/docs)
2. Browse the templates below
3. Copy the prompt and paste it into your agent
4. Replace placeholder values (marked with `YOUR_*`) with your own credentials
5. Your agent deploys everything autonomously

## Templates

| Template | Description | Languages |
|----------|-------------|-----------|
| [OpenClaw Investment Researcher](./templates/openclaw-investment-researcher.en.md) | AI chatbot on Telegram for stocks, ETFs, crypto, and market analysis | [한국어](./templates/openclaw-investment-researcher.ko.md) · [English](./templates/openclaw-investment-researcher.en.md) |

## Contributing

1. Fork this repo
2. Copy [`TEMPLATE.md`](./TEMPLATE.md) as a starting point
3. Add your template as `templates/{name}.en.md` (Korean: `{name}.ko.md`)
4. Open a Pull Request

File naming: `{template-name}.{lang}.md` — e.g. `my-app.en.md`, `my-app.ko.md`

## Browse Online

Visit [agent.willform.ai/templates](https://agent.willform.ai/templates) for an interactive gallery with search, filtering, and one-click copy.

## License

[MIT](./LICENSE)
