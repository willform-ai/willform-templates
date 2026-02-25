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
| [OpenClaw Investment Researcher](./templates/openclaw-investment-researcher/) | AI-powered investment research chatbot with Telegram integration | [한국어](./templates/openclaw-investment-researcher/prompt-ko.md) · [English](./templates/openclaw-investment-researcher/prompt-en.md) |

## Contributing

1. Fork this repo
2. Add your template under `templates/{your-template-name}/`
3. Include at least one `prompt-en.md` (English) file
4. Open a Pull Request

Each template directory should contain:
- `prompt-en.md` — English version (required)
- `prompt-ko.md` — Korean version (optional)
- Additional language files follow the `prompt-{lang}.md` pattern

## Browse Online

Visit [agent.willform.ai/templates](https://agent.willform.ai/templates) for an interactive gallery with search, filtering, and one-click copy.

## License

[MIT](./LICENSE)
