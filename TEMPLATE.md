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

Replace the placeholder values below with your own.

| Variable | Where to get it | Description |
|---|---|---|
| `YOUR_VARIABLE` | Step-by-step instructions with [links](https://example.com) to the provider's console or docs. Include expected format (e.g. `sk-ant-...`), signup steps if needed, and any billing prerequisites. | What this value is used for |

## Post-Deploy

**Access**: https://$DOMAIN

Describe what the user should do after deployment. Use `$DOMAIN` for the deployed domain, and any prerequisite variable (e.g. `$GATEWAY_TOKEN`) — they are automatically substituted with the actual values.

Available variables:
- `$NAME` — deployment name
- `$DEPLOYMENT_ID` — deployment UUID
- `$DOMAIN` — exposed domain (if `expose: true`)
- `$INTERNAL_ENDPOINT` — cluster-internal endpoint
- `$STATUS` — deployment status
- Any prerequisite variable name (e.g. `YOUR_GATEWAY_TOKEN`, `YOUR_API_KEY`) — substituted with the user's input

## Deploy Config

```json
{
  "name": "my-app",
  "chartType": "web",
  "image": "my-image:latest",
  "port": 3000,
  "env": {
    "MY_VAR": "YOUR_VARIABLE"
  },
  "expose": true
}
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
| Post-Deploy | `## Post-Deploy` | `## 배포 후 안내` | Markdown with variable substitution |
| Deploy Config | `## Deploy Config` | `## Deploy Config` | JSON code block (required) |
| Startup Script | `## Startup Script` | `## Startup Script` | Bash code block (optional) |
