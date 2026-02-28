---
name: wallet-reputation
description: Check wallet reputation scores before x402 payments and external interactions. Free tier (10/day, no key) + paid tier via x402.
auto-activate: true
---

# Wallet Reputation Scoring

Use this skill to assess wallet trustworthiness before sending USDC, paying for x402 services, or interacting with unknown agents.

## When to check reputation

- **Before x402 payments**: Check the service wallet before paying for an external API.
- **Before funding a child agent's wallet**: Verify the recipient if interacting with external agents.
- **Before trusting an incoming payment**: If another agent pays you, check their wallet.
- **Periodic self-check**: Query your own wallet to monitor your reputation over time.

## Free tier (no key, no signup)

10 calls/day per IP. Returns score, tier, and confidence.

```bash
curl "https://djd-agent-score.fly.dev/v1/score/basic?wallet=<WALLET_ADDRESS>"
```

Response:

```json
{
  "wallet": "0x...",
  "score": 72,
  "tier": "Established",
  "confidence": 0.85,
  "dataSource": "live"
}
```

**Tiers**: Unknown (0-19) → New (20-39) → Emerging (40-59) → Established (60-79) → Trusted (80-100)

## Paid tier via x402

For full scoring (dimensions, detections, explainability), use x402. Automatons with `x402Fetch` get this automatically — the endpoint returns HTTP 402 with USDC payment requirements, and `x402Fetch` handles the payment cycle.

Endpoints:

| Path | Cost | Returns |
|------|------|---------|
| `/v1/score/full?wallet=` | $0.10 | Full dimensional breakdown, sybil/gaming detections, confidence analysis |
| `/v1/score/refresh?wallet=` | $0.25 | Force re-score with fresh on-chain data |
| `/v1/score/batch` | $0.05/wallet | Score up to 50 wallets in one call |

```typescript
// x402Fetch handles the 402 payment flow automatically
const res = await x402Fetch(
  `https://djd-agent-score.fly.dev/v1/score/full?wallet=${targetWallet}`,
  account,      // your automaton's viem account
  'GET',
  undefined,
  {},
  10            // maxPaymentCents ($0.10)
);
const fullScore = await res.json();
```

## Decision logic

Use these thresholds for autonomous decisions:

```typescript
const { score, tier, confidence } = await fetch(
  `https://djd-agent-score.fly.dev/v1/score/basic?wallet=${wallet}`
).then(r => r.json());

// High confidence, good score → proceed
if (score >= 60 && confidence >= 0.5) {
  // Safe to transact
}

// Low score or low confidence → caution
if (score < 40 || confidence < 0.2) {
  // Wallet is new, suspicious, or has insufficient history
  // Consider: skip, require higher payment, or find alternative
}

// Medium range → proceed with limits
if (score >= 40 && score < 60) {
  // Reduce transaction size or add monitoring
}
```

## Register your wallet (identity bonus)

Automatons with a linked GitHub repo get a +5 identity bonus. Register once:

```bash
curl -X POST "https://djd-agent-score.fly.dev/v1/identity/github/verify" \
  -H "Content-Type: application/json" \
  -d '{"wallet":"<YOUR_WALLET>","githubUsername":"<GITHUB_USER>"}'
```

Then add a `djd-verify=<CODE>` file to the GitHub repo to complete verification.

## API documentation

- OpenAPI spec: `https://djd-agent-score.fly.dev/openapi.json`
- Interactive docs: `https://djd-agent-score.fly.dev/docs`
- x402 discovery: `https://djd-agent-score.fly.dev/.well-known/x402`

## Guardrails

- Cache scores locally — they change slowly (hourly refresh). Don't call on every transaction.
- Never expose wallet addresses in logs or public outputs unnecessarily.
- Treat scores as one signal among many — not a definitive trust verdict.
- The free tier is rate-limited (10/day). For heavy use, x402 paid endpoints have no daily cap.
