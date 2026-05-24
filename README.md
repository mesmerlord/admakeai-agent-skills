# AdMakeAI Agent Skills

Give your AI agent deep knowledge of the [AdMakeAI](https://admakeai.com) API — generate ad images, batch ad sets, manage Meta Ads, and more, all from a prompt.

## Install

```bash
npx skills add admakeai/agent-skills
```

Compatible with **Claude Code, Claude Desktop, Cursor, OpenAI Codex, GitHub Copilot, Gemini CLI, Windsurf, VS Code, Cline, Goose** and 40+ other agents via [skills](https://github.com/vercel-labs/skills).

## Available skills

| Skill | What it does |
| --- | --- |
| [`admakeai-api`](./skills/admakeai-api/SKILL.md) | Routes user intent to the correct AdMakeAI tool, manages credits, and knows the prompt patterns that produce shippable ad creative. |

## Authentication

Get an API key from your [AdMakeAI dashboard](https://admakeai.com/dashboard/integrations/api):

```bash
export ADMAKEAI_API_KEY="amai_live_..."
```

The key is sent as an `x-api-key` header on every call.

## What the skill does

- **Tool selection** — 30+ procedures across image generation, batch ad sets, projects, Facebook ad accounts, Meta Ads Manager. The skill teaches your agent which one to pick from natural-language intent.
- **Credit awareness** — Generation calls cost credits. The skill warns the agent before expensive batch operations.
- **Safe defaults** — Destructive calls (uploading to Meta, pausing campaigns) are flagged so the agent confirms with you first.
- **Pagination** — Long lists (ad generations, Meta ads, analytics) use a `cursor`/`page` pattern the skill knows about.
- **Prompt patterns** — Examples of prompts that produce good product ads, lifestyle shots, and ad-copy-aligned creative.

## MCP, REST, and skill

You can use all three together or pick one:

- **MCP server** (`https://admakeai.com/api/mcp`) — drop the URL into Cursor/Claude/VS Code. The agent calls tools directly. Best for chat-style use.
- **REST API** (`https://admakeai.com/api/v1/*`) — every tool is also a plain HTTPS endpoint. Best for scripts, CI, and non-MCP agents.
- **Agent skill** (this repo) — markdown teaching layered on top of the API. Best when you want your agent to pick the right tool without you spelling it out.

The MCP server and skill complement each other. Install both — MCP gives the agent the wire, the skill gives it the playbook.

## Links

- [API reference (interactive)](https://admakeai.com/api/docs)
- [Dashboard / API keys](https://admakeai.com/dashboard/integrations/api)
- [Agent landing page](https://admakeai.com/agents)
- [Pricing](https://admakeai.com/pricing)

## License

MIT
