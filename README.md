# AdMakeAI for Claude

[AdMakeAI](https://admakeai.com) is an ad design studio for performance marketers. This plugin lets Claude design on-brand ad creatives, write ad copy, research the ads competitors are running in the Meta Ad Library, and draft and report on Meta ad campaigns — all from a conversation.

It bundles two pieces:

- **The AdMakeAI connector** (`https://admakeai.com/api/mcp`), which gives Claude tools to work with your AdMakeAI account. You sign in with your AdMakeAI account through OAuth; no API key is needed.
- **The `admakeai-api` skill**, which teaches Claude which tool fits each request, to always design into the right brand project, what things cost in credits, and to confirm before anything that spends credits or touches your Meta ad account.

## Use it

1. Create an account at [admakeai.com](https://admakeai.com) and set up a project for your brand (product, audience, logo).
2. Install the plugin and connect the AdMakeAI connector from the plugin's **Connectors** tab.
3. Ask Claude things like:
   - "Design three 4:5 ads for my matcha brand aimed at tired founders."
   - "What ads are my competitors running right now? Save the longest-running one as inspiration and remix it for my brand."
   - "Write five headline and primary-text variations for that ad."
   - "How did my Meta ad account do last month compared to the month before?"
   - "Draft a Meta campaign with these three ads, but don't push it until I say so."

Designing ads and videos spends credits from your AdMakeAI plan; reading, listing, research, and analytics are free. Claude asks before any action that spends credits or creates anything in your Meta ad account. Nothing is ever deleted through the plugin.

## Data

The plugin only talks to AdMakeAI at `admakeai.com`. When you ask for something, Claude sends the inputs that request needs — for example your creative brief, reference images you provide, ad copy, or IDs of your projects, ads, or Meta campaigns — to your AdMakeAI account through the connector. AdMakeAI stores the ads and drafts you create in your account, and reaches Meta only for the ad accounts you have connected in AdMakeAI. The plugin itself stores nothing and runs no local code.

See the [privacy policy](https://admakeai.com/resources/privacy-policy) and [terms of service](https://admakeai.com/resources/terms-of-service).

## Other agents

The same connector works in any MCP client (Claude Code, Cursor, VS Code, ChatGPT): add `https://admakeai.com/api/mcp` as a remote MCP server. The skill installs into 40+ agents with:

```bash
npx skills add mesmerlord/admakeai-agent-skills
```

For scripts and CI there is also a REST API; see the [API reference](https://admakeai.com/api/docs).

## Support

Email [support@admakeai.com](mailto:support@admakeai.com). AdMakeAI is made by Mesmer s.r.o.

## License

MIT
