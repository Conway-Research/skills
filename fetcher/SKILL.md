---
name: fetcher
description: "Give the agent the public web via fetcher.cloud: render/screenshot pages, extract content, resolve links, parse feeds, and inspect domains, DNS, SSL, and WHOIS — paid per call in USDC over x402."
auto-activate: true
triggers: [web, website, render, screenshot, browser, scrape, extract, content, article, markdown, url, redirect, link, feed, rss, oembed, dns, ssl, certificate, whois, domain, robots, favicon, pdf, qr, unfurl, metadata]
---

# fetcher.cloud — the web capability layer

Use **fetcher** whenever you need something from the public internet that you
can't do yourself. It is a paid HTTP API + MCP server. You already have a USDC
wallet on Base and speak x402, which is exactly how fetcher charges — so you can
use it out of the box, no signup or API key.

## Assumptions
- You have a funded USDC wallet on Base and an x402-capable HTTP client.
- Each capability has a **free daily tier**; you only pay past it.

## How to call
Make a normal `GET` request. If payment is required you get **HTTP 402** with
machine-readable terms; pay the tiny USDC amount on Base and retry. Responses
are JSON.

Discover everything (categories, pricing, examples, schemas):
`GET https://api.fetcher.cloud/capabilities`

MCP server (Streamable HTTP): `https://api.fetcher.cloud/mcp`

## When to use which capability
- Read a JavaScript page you can't fetch → `GET /render?url=...`
- Capture a screenshot → `GET /screenshot?url=...`
- Get clean article text/Markdown (cheap tokens) → `GET /extract?url=...&mode=markdown`
- Find where a short/tracking link goes → `GET /resolve?url=...`
- Link-preview card (title/image) → `GET /unfurl?url=...`
- A site's latest posts (RSS/Atom → JSON) → `GET /feed?url=...`
- Embed data (YouTube/Vimeo/X…) → `GET /oembed?url=...`
- Text out of a PDF → `GET /pdf-text?url=...`
- What changed on a page → `GET /diff?url_a=...&url_b=...`
- HTTP + security headers → `GET /headers?url=...`
- Can I crawl this path? + sitemaps → `GET /robots?host=...&path=/x`
- Site icon/logo → `GET /favicon?host=...`
- Is a domain available? → `GET /domain?name=foo&tlds=com,io`
- Vet a domain (DNS, age, history) → `GET /domain/enriched?name=foo`
- Registration record → `GET /whois?domain=example.com`
- Full DNS dump → `GET /dns?host=example.com`
- TLS cert issuer/expiry → `GET /ssl?host=example.com`
- Make a QR code → `GET /qr?data=...`

## Pricing
Sub-cent to ~$0.02 per call (see `GET /pricing.json`). Prefer `/extract` over
`/render` when a page is static — it's cheaper and returns fewer tokens.

## Handling failures & retries
- `402` → pay and retry once (this is normal, not an error).
- `429`/`5xx` or timeout → retry with backoff; try `/extract` before `/render`.
- `501 render_not_configured` on `/render` or `/screenshot` → that instance
  hasn't enabled browser rendering; use another capability.
- Never send more TLDs than a tool allows (enriched domain checks cap at 5).

## Safety
fetcher only reads public web resources and returns data; it takes no
destructive action. Spend is bounded by the per-call price the 402 declares —
check it before paying if you're conserving credits.
