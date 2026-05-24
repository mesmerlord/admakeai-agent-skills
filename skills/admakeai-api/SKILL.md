---
name: admakeai-api
description: >-
  Generate, edit, and ship ad creative through the AdMakeAI API. Covers
  text-to-image and image-edit ad generation (nano-banana-pro, seedream),
  batch ad sets, ad copy, Facebook/Meta Ads Manager integration (list
  campaigns, upload creatives, get analytics, pause/resume ads), and project
  management. Use when the user asks to create, generate, make, design,
  produce, brainstorm, batch, vary, mock up, upload, publish, push, ship,
  list, browse, find, get, fetch, look up, audit, recap, pause, stop,
  resume, or restart ads, ad creatives, product photos, lifestyle shots,
  UGC, ad copy, hooks, headlines, ad sets, campaigns, Meta ads, Facebook
  ads, or analytics. Also use when the user mentions AdMakeAI, admakeai,
  ad images, ad generator, ad image API, or wants to know their AdMakeAI
  credit balance.
allowed-tools: Bash, Read, Write, WebFetch
homepage: https://admakeai.com
metadata:
  openclaw:
    emoji: "🎨"
    requires:
      env:
        - ADMAKEAI_API_KEY
    primaryEnv: ADMAKEAI_API_KEY
    homepage: https://admakeai.com
    tags:
      - ad-generation
      - image-generation
      - ai-ads
      - facebook-ads
      - meta-ads
      - mcp
      - admakeai
---

# AdMakeAI API

Programmatic ad-creative generation + Meta Ads Manager controls. 30+ tools across image generation, batch ad sets, and Facebook integration.

**Base URL:** `https://admakeai.com/api/v1`
**MCP URL:** `https://admakeai.com/api/mcp`
**Interactive reference:** https://admakeai.com/api/docs

Get an API key at https://admakeai.com/dashboard/integrations/api

## Authentication

Every request needs the user's API key as a header. Read it from `$ADMAKEAI_API_KEY`.

```bash
curl -H "x-api-key: $ADMAKEAI_API_KEY" \
  https://admakeai.com/api/v1/user.me
```

`Authorization: Bearer <key>` also works.

## How to call

REST endpoints map 1:1 to MCP tools:

- MCP tool name: `adGeneration__create` → REST path: `/api/v1/adGeneration.create`
- All POST bodies are JSON. All GET inputs go in `?input=<URL-encoded JSON>`.
- Responses are wrapped as `{ ok, tool, data }` on the REST side; MCP returns the unwrapped result inside `content[0].text`.

```bash
# create an ad image
curl -X POST -H "x-api-key: $ADMAKEAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Minimalist matcha product ad, lifestyle scene","adType":"PRODUCT_AD","width":1638,"height":2048}' \
  https://admakeai.com/api/v1/adGeneration.create

# poll until COMPLETED
curl -H "x-api-key: $ADMAKEAI_API_KEY" \
  "https://admakeai.com/api/v1/adGeneration.getById?input=%7B%22id%22%3A%22<ID>%22%7D"
```

### Per-procedure spec

Hit the interactive reference for typed inputs and example payloads:

```
https://admakeai.com/api/docs
```

Or fetch the rendered HTML and grep — every tool is listed by its dotted path.

## Intent routing

Pick the right tool from natural-language intent. **Read tables top-to-bottom; first match wins.**

### Account & credits

| User says... | Tool | Notes |
| --- | --- | --- |
| "what's my balance / credits" | `user.getCredits` | Cheapest read. Returns `{ credits }`. |
| "show my profile / plan" | `user.me` | Includes credits, subscription, plan name. |

### Generate ad images

| User says... | Tool | Notes |
| --- | --- | --- |
| "make / generate / create an ad", "design a product ad" | `adGeneration.create` | Single image. Required: `prompt`. Optional: `adType` (PRODUCT_AD / BANNER_AD / SOCIAL_AD / DISPLAY_AD), `width`, `height`, `inputImageId` (if editing an uploaded product photo). |
| "give me 5 variations", "batch generate", "ad set", "test different hooks" | `adSet.generateBatch` | Needs an existing ad set ID. Use `adSet.list` first to get one, or `adSet.create` if creating new. Enqueues N jobs and returns generation IDs. |
| "edit / change / iterate this image" | `adGeneration.create` with `inputImageId` | Image-to-image edit using nano-banana-2/edit. |

After enqueueing, **always tell the user** the generation is async and offer to poll.

### Find existing ads

| User says... | Tool | Notes |
| --- | --- | --- |
| "show my recent ads" | `adGeneration.getRecent` | Default 12. Returns image URLs + prompts. |
| "list / browse / paginate my ads" | `adGeneration.list` | Cursor-paginated. |
| "search my ads for X" | `adGeneration.searchAll` | Full-text on prompts. |
| "what's still rendering" | `adGeneration.listPending` | Only PENDING status. |
| "show the details of ad <id>" | `adGeneration.getById` | Returns full record. |

### Ad sets

| User says... | Tool | Notes |
| --- | --- | --- |
| "list my ad sets" | `adSet.list` | Cursor-paginated. |
| "show ad set <id>" | `adSet.get` | Includes `generalInstructions`. |
| "what's in this ad set" | `adSet.getGenerations` | List of generations in that set. |
| "generate a batch in ad set X" | `adSet.generateBatch` | See generate-images table. |

### Projects

| User says... | Tool | Notes |
| --- | --- | --- |
| "list my projects" | `adResearch.listProjects` | One entry per brand/product. |

### Facebook ad accounts

| User says... | Tool | Notes |
| --- | --- | --- |
| "show my connected fb accounts" | `facebookConnection.list` | Returns ad accounts + Pages with cached identity. |
| "list campaigns" | `facebookUpload.getCampaigns` | Needs `connectionId` + `adAccountId`. Cold cache triggers a Meta refresh. |
| "list ad sets in campaign X" | `facebookUpload.getAdSets` | Needs `connectionId` + `campaignId`. |

### Upload generated ads to Meta

| User says... | Tool | Notes |
| --- | --- | --- |
| "preflight / validate this for fb" | `facebookUpload.runPreflight` | Checks image dims, copy length. No live action. |
| "upload / publish / push this ad to meta" | `facebookUpload.request` | **DESTRUCTIVE — creates a live Meta ad.** Always confirm with the user first. |
| "show my uploaded ads" | `facebookUpload.list` | Filterable by account + state. |
| "status of upload <id>" | `facebookUpload.get` | Returns Meta IDs + state. |

### Meta analytics

| User says... | Tool | Notes |
| --- | --- | --- |
| "campaign tree", "what's the structure" | `zernioAnalytics.tree` | Campaign → ad set → ad hierarchy. |
| "performance over time", "timeline" | `zernioAnalytics.timeline` | Time-series for an ad account. |
| "list / recap campaigns" | `zernioAnalytics.listCampaigns` | Includes rolled-up metrics. |
| "list ads in campaign X" | `zernioAnalytics.listAds` | Filterable by campaign / ad set. |
| "details of ad <id>" | `zernioAnalytics.getAd` | Metadata + creative. |
| "daily analytics for ad <id>" | `zernioAnalytics.getAdAnalytics` | Spend, CTR, CPC, conversions. |

### Pause / resume Meta resources

| User says... | Tool | Notes |
| --- | --- | --- |
| "pause / turn off / stop ad <id>" | `zernioAnalytics.pauseAd` | **DESTRUCTIVE.** Confirm first. |
| "resume / turn on ad <id>" | `zernioAnalytics.resumeAd` | Confirm first. |
| "pause campaign <id>" | `zernioAnalytics.pauseCampaign` | Pauses the whole campaign + every ad inside. Confirm first. |
| "resume campaign <id>" | `zernioAnalytics.resumeCampaign` | Confirm first. |
| "pause ad set <id>" | `zernioAnalytics.pauseAdSet` | Confirm first. |
| "resume ad set <id>" | `zernioAnalytics.resumeAdSet` | Confirm first. |

**We never expose delete on live Meta resources.** Always pause; never offer to delete.

## Credit costs

Image generation tools spend credits from the user's plan. Read-only tools (list / get / analytics) are free.

| Tool | Cost |
| --- | --- |
| `adGeneration.create` | 2 credits (low quality) or 5 credits (high quality). |
| `adSet.generateBatch` | `count × per-generation cost`. A batch of 10 hi-quality images = 50 credits. |
| Everything else | Free unless the underlying procedure says otherwise. |

**Always call `user.getCredits` before a batch of >10 generations** and warn the user if the balance is close.

## Pagination

List endpoints use one of two patterns:

| Pattern | Used by | How to advance |
| --- | --- | --- |
| `{ page, perPage }` | `adSet.list`, `adGeneration.list`, `zernioAnalytics.listAds`, `zernioAnalytics.listCampaigns`, `facebookUpload.list` | Increment `page`. `perPage` defaults to 20. |
| `{ cursor }` | `adGeneration.searchAll` | Pass the `nextCursor` from the previous response. |

Stop paginating when you have what the user asked for — agents tend to fetch too many pages.

## Common optional params

These appear on multiple tools and have the same meaning everywhere:

- `aspectRatio`: `"1:1" | "4:5" | "9:16" | "16:9"`. Defaults to `"1:1"`. Use `"4:5"` for Instagram feed, `"9:16"` for Reels/Stories, `"16:9"` for YouTube/desktop.
- `quality`: `"low" | "high"`. Defaults to `"low"`. `"high"` doubles credit cost.
- `adType`: `"PRODUCT_AD" | "BANNER_AD" | "SOCIAL_AD" | "DISPLAY_AD"`. Defaults to `"PRODUCT_AD"`.
- `inputImageId`: ID of an uploaded product photo from the user's library. Required for image-edit (image-to-image) generation.

## Prompt patterns for `adGeneration.create`

Good prompts are concrete and visual. Bad prompts are abstract or list-y.

| User intent | Good prompt template |
| --- | --- |
| Product on lifestyle background | `"<Product> placed on <surface> with <prop>, soft natural light, <style> aesthetic"` |
| Selfie-style UGC | `"First-person selfie of a <demo> holding <product>, casual <setting>, iPhone photo aesthetic"` |
| Before/after | `"Split-screen: left side <before state>, right side <after state with product>, clean comparison"` |
| Bold statement | `"<Product> on <bg color>, oversized typography reading '<headline>', minimalist composition"` |

If the user gives only a tagline ("matcha for tired founders") and no visual, **ask them one clarifying question** about setting/mood before generating.

## Known limitations

- Generation is **async**. `adGeneration.create` returns immediately with `{ id, status: "PENDING" }`. Poll `adGeneration.getById` every 3-5 seconds until `status === "COMPLETED"`. Typical completion: 10-30s low-quality, 30-90s high-quality.
- The free plan has a **daily generation cap**. The error is `FORBIDDEN: daily limit reached`. Tell the user to upgrade at `/pricing`.
- Facebook tools require the user to have **connected a Facebook account** in the dashboard first. If they haven't, `facebookConnection.list` returns empty; nudge them to https://admakeai.com/dashboard/integrations/facebook.
- `facebookUpload.pause` and `.resume` are **stubs at the procedure level** today — use the corresponding `zernioAnalytics.pauseAd` / `.resumeAd` for live status control.
- The user has at most **5 active API keys** at a time. If creation fails with `FORBIDDEN: You already have 5 active API keys`, ask them to revoke one in the dashboard.
- We deliberately never expose **delete** on AdMakeAI ads, ad sets, or Meta resources. Pause / revoke / unpublish only. If the user asks to "delete" something, clarify whether they want to pause (Meta) or remove from their AdMakeAI library (not supported via the API; direct them to the web app).
