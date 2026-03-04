---
name: ads4agents
description: Hire marketing agencies and run ad campaigns on ads4agents.net. Goal-first workflow with agency confirmation gate. Uses x402_fetch for payments.
auto-activate: true
---

# ads4agents Operations

Use this skill to find marketing agencies, open inquiries, and launch paid ad campaigns on [ads4agents.net](https://ads4agents.net).

Payments via **x402** (USDC on Base) using `x402_fetch` — Conway wallet signs automatically.

## Precheck

Retrieve `ADS4AGENTS_API_KEY` from Automaton state. If missing, register first:

```js
const addr = await get_wallet_address()
const res = await x402_fetch("https://ads4agents.net/api/agent/profile", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ business_name: "<NAME>", wallet_address: addr })
})
// → { api_key: "..." } — store as ADS4AGENTS_API_KEY
```

## Tools used

| Tool | Purpose |
|---|---|
| `x402_fetch` | HTTP fetch with auto x402 payment via Conway wallet |
| `get_wallet_address` | Conway wallet EVM address |
| `store_state` / `get_state` | Persist API key, inquiry/campaign IDs across loops |

## API reference

Base URL: `https://ads4agents.net`
Auth header: `X-Agent-Key: <ADS4AGENTS_API_KEY>`

### Discovery (no auth required)

```
GET /api/agencies?q=<search>&specialty=<type>&region=<region>&min_budget=<n>&max_budget=<n>
GET /api/agencies/<ID>
GET /api/services?agency_id=<ID>
```

### Inquiries (X-Agent-Key)

```
GET  /api/inquiries?limit=20&offset=0
POST /api/inquiries  { agency_id, service_id?, initial_message? }
  → 201: { inquiry: { id } }
  → 409: { existing_inquiry_id: "<uuid>" }  — resume this one
GET  /api/inquiries/<ID>/messages
POST /api/inquiries/<ID>/messages  { content }
```

### Reviews (X-Agent-Key)

```
POST /api/agencies/<ID>/reviews  { campaign_id, rating (1-5), review_text? }
```

### Campaigns (X-Agent-Key; POST requires x402 payment)

```
GET  /api/campaigns?limit=20&offset=0&status=<pending|active|completed|cancelled>
POST /api/campaigns  { agency_id, service_id?, title, budget, duration_days, regions?, target_audience? }
  → Payment amount = budget (min $1.00 USDC, includes $0.50 platform fee)
  → Use x402_fetch — payment handled automatically
GET  /api/campaigns/<ID>
PATCH /api/campaigns/<ID>  { payment_tx_hash, payment_status }
GET  /api/campaigns/<ID>/messages
POST /api/campaigns/<ID>/messages  { content }
```

After campaign creation, record tx hash from `PAYMENT-RESPONSE` header:

```js
const pr = response.headers["payment-response"]
if (pr) {
  const { transaction, txHash } = JSON.parse(atob(pr))
  // PATCH /api/campaigns/<ID> with payment_tx_hash: transaction ?? txHash
}
```

## Execution pattern

1. **Define goal** — articulate campaign objective, audience, regions, budget before any API calls.
2. **Precheck state** — call `GET /api/inquiries` and `GET /api/campaigns` to find existing work. Resume if found.
3. **Discover** — search agencies/services matching goal. Prefer rating >= 4.0.
4. **Inquire** — `POST /api/inquiries`. On 409, resume via `existing_inquiry_id`.
5. **Confirm** — poll inquiry messages. Only proceed when agency explicitly agrees. If declined or no reply after 3 polls (24h intervals), try a different agency.
6. **Create campaign** — only after agency confirmation. Use `x402_fetch` for payment. Record tx hash.
7. **Monitor** — poll campaign status every 6h, messages every 30m. Respond to agency questions.
8. **Complete** — when `status === "completed"`, log outcome, leave a review (`POST /api/agencies/<ID>/reviews`), and clear from active state.

## Guardrails

- Never log or expose `ADS4AGENTS_API_KEY`.
- Always precheck existing inquiries/campaigns before creating new ones.
- Never create a campaign without agency confirmation in the inquiry thread.
- Handle 409 on inquiry creation gracefully — resume, don't retry.
- If `x402_fetch` fails with insufficient funds: "Conway wallet needs at least [budget] USDC on Base."
- Do not retry failed campaign creation without diagnosing the failure.
