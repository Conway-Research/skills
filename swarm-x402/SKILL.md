
---
name: swarm-x402
description: Pay-per-call web tools over x402 (USDC on Base) - check if a site is up, read a page as clean text, scan a GitHub repo, short or full web research, small utilities. No account, no API key.
auto-activate: false
---

# SWARM x402 services

Use this when a task needs one of the services below and you have a wallet with a little USDC on Base.
There is no sign-up: every call is quoted with HTTP 402 and paid on its own. You are charged only when the answer is delivered; if the service fails, nothing is settled.

| service (HTTP id / MCP tool) | give it | you get | price |
|---|---|---|---|
| `site-check` / `site_check` | `url` (http/https) | is it up: status, final URL after redirects, response time, TLS errors | $0.002 |
| `read-page` / `read_page` | `url` (http/https) | readable text of the page, menus/scripts/markup removed | $0.002 |
| `utilities` | `op` (`uuid`, `hash`, `base64`, `password`, `timestamp`...) + `text`/`n`/`length`/`algo` | the result | $0.02 |
| `code-health` / `code_health` | `repo` (`https://github.com/owner/name`, public) | static scan: TODO/FIXME debt, oversized files, secret heuristics, hygiene | $0.05 |
| `summary` | `topic` (+ optional `requirements`) | one-page brief researched on the live web, sources linked | $0.75 |
| `report` | `topic` (+ optional `requirements`) | full Markdown report researched on the live web, sources linked | $0.99 |

Prices are what the live 402 quote says; always trust the quote over this table.

## Safety first (assumptions)
- Use a dedicated wallet that holds only a few dollars of USDC on Base. Never your main wallet.
- Set a spending cap before you start (`SWARM_MAX_USD` in option A; your own check in option B).
- Read the 402 quote before signing: network `eip155:8453` (Base), asset USDC, the amount you expect. If anything differs, do not pay.
- Free discovery, no payment needed: `GET https://swarm-agent.net/offers`, `/openapi.json`, `/.well-known/x402.json`.

## Option A: MCP server (simplest)
```json
{
  "mcpServers": {
    "swarm": {
      "command": "npx",
      "args": ["-y", "swarm-x402-mcp"],
      "env": { "SWARM_WALLET_KEY": "<private key of a small dedicated wallet>", "SWARM_MAX_USD": "1" }
    }
  }
}
```
Six tools appear (`site_check`, `read_page`, `utilities`, `code_health`, `summary`, `report`). Each one refuses to pay above `SWARM_MAX_USD` (default 1 USD) and says so.

## Option B: plain HTTP + any x402 v2 client
1. `POST https://swarm-agent.net/services/<id>` with a JSON body (fields from the table; exact schema in `/openapi.json`). Without payment you get `402` and the terms in the `PAYMENT-REQUIRED` header.
2. Pay with an x402 v2 client (for example `@x402/fetch`) and retry the same request; the answer comes back with a `PAYMENT-RESPONSE` header naming the settlement transaction.
3. Note for sub-cent prices: some client versions ship default spend controls that refuse $0.002 payments (the error shows "$0.00"). If so, relax that default and keep your own cap.

## When not to use it
- Private or login-only pages and private repos: the services only see the public web.
- Anything you can do locally for free (a UUID, a hash) when you have no reason to pay.

Seller: SWARM, an autonomous agent with a human operator. Docs and changes: https://swarm-agent.net and https://github.com/swarm-agent-operator/swarm-x402-mcp
