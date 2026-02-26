---
category: web-app
tags: [example, tag]
---

# Template Name

One-line description of what this deploys.

## What You Get

- What services are deployed
- What integrations are configured
- Key features of this setup

## Prerequisites

Replace the placeholder values below with your own before pasting the prompt.

| Variable | Where to get it | Description |
|---|---|---|
| `YOUR_VARIABLE` | Link or instructions | What this value is used for |

## Prompt

```
Your deployment prompt here.

Use YOUR_VARIABLE placeholders for any user-specific values.
Keep the prompt self-contained — the AI agent should be able to
execute it without asking follow-up questions.
```

## How It Works

The willform-agent platform fetches templates dynamically from this repository.
Adding a new template is as simple as committing a markdown file to `templates/`.

### File naming

`templates/{slug}.{lang}.md`

- `slug` becomes the template ID (e.g., `my-app` in `my-app.en.md`)
- `lang` is `en` or `ko`
- The full ID combines both: `my-app-en`

### Frontmatter

Only `category` and `tags` go in YAML frontmatter. Everything else is parsed from the markdown structure.

```yaml
---
category: ai-agent
tags: [openclaw, telegram, ai, finance]
---
```

Valid categories: `web-app`, `database`, `cache`, `queue`, `ai-agent`, `background-job`, `full-stack`

### Parsed sections

| Section | Heading (EN) | Heading (KO) | Parsed as |
|---|---|---|---|
| Title | `# ...` | `# ...` | First H1 heading |
| Description | (paragraph) | (paragraph) | First paragraph after title |
| Highlights | `## What You Get` | `## 배포 결과` | Bullet list items |
| Prerequisites | `## Prerequisites` | `## 사전 준비` | Table rows (Variable, How to get, Purpose) |
| Prompt | `## Prompt` | `## 프롬프트` | Content inside code block |
