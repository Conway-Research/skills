---
name: whale-signals
description: Buy Hyperliquid whale positioning, funding and open-interest signals, liquidation levels and AI market briefs per request over x402 (USDC on Base). Free preview routes need no payment; the cheapest paid probe is $0.001.
auto-activate: false
---

# whale-signals

Pay-per-request market data from **WhaleTape** (`https://whaletape.xyz`) over HTTP 402.
No account, no API key, no subscription, no minimum. 350+ Hyperliquid perp markets:
crypto, equities, commodities, FX, indices and rates.

Use this skill when a task needs whale positioning, funding or open-interest
changes, liquidation levels, cross-asset flows, or a written read of the market.

**Live version, generated from the catalog on every request:**
`https://whaletape.xyz/skill-automaton.md`. It always carries current prices, so
an automaton can install it directly and skip this copy:

```
install_skill({ source: "url", url: "https://whaletape.xyz/skill-automaton.md" })
```

The price table below is a snapshot taken when this file was written. If it ever
disagrees with the live page or with `/.well-known/x402`, those are right and
this file is stale.

## Assumptions

- **EVM automaton.** `x402_fetch` refuses to sign from a Solana wallet.
- **USDC on Base** in your wallet. The cheapest paid route is 0.001 USDC; the
  most expensive per-call route is 0.99. All fit under the default `maxX402PaymentCents: 100`.
- **One creator action is required before any paid call.** `x402_fetch` is gated by
  `treasuryPolicy.x402AllowedDomains`, which defaults to `["conway.tech"]`.
  `config.json` is a protected file, so you cannot add the domain yourself.
  Ask your creator to set:

  ```json
  "x402AllowedDomains": ["conway.tech", "whaletape.xyz"]
  ```

  Until that lands, the free routes below still work — they are plain HTTP and
  are not subject to the x402 allowlist.

## Start free — it costs nothing and it is real data

These return live samples, not documentation. Fetch them with `exec` and curl:

| route | what you get |
|---|---|
| `/preview/whales` | a slice of current whale positioning |
| `/preview/signals` | a slice of the latest positioning changes |
| `/preview/squeeze` | a slice of the squeeze radar |
| `/markets/preview` | a slice of the market table |
| `/track-record` | the provider's own published record |
| `/public/stats` | usage and coverage counters |
| `/health` | liveness |

Decide from these whether the paid series is worth USDC at all. If a free sample
does not already look useful for your goal, do not pay.

## Paid routes (USDC per call, Base)

| price | routes |
|---|---|
| **$0.001** | `/coverage` · `/coverage/{symbol}` — how much history exists before you buy it |
| **$0.05** | `/whales` · `/recommendations` · `/flows` · `/liquidations` (+`/BTC` `/ETH` `/SOL` `/HYPE` `/XRP` `/DOGE` `/ZEC` `/ENA` `/NEAR` `/PUMP`) · `/history/{symbol}` · `/markets/class/{class}` · `/pulse` · `/index/summary` |
| **$0.07** | `/markets/full` · `/whales/positions` · `/whales/events` · `/squeeze` · `/whale/{address}` · `/index/merchant/{id}` |
| **$0.09** | `/signals` · `/flows/classes` · `/index/ranking` |
| **$0.10** | `/analysis` — a written read of the current snapshot |
| **$0.12** | `/premium` — brief + bias + whale positions + signals in one call |
| **$0.25** | `/ask` — one natural-language question answered from the current snapshot |
| **$0.99** | `/alerts` (POST, webhook for up to 7 days) · `/pro/data` · `/pro/unlock` |

Machine-readable catalog: `https://whaletape.xyz/.well-known/x402`

## How to call

```
x402_fetch({ url: "https://whaletape.xyz/coverage" })
```

The route answers `402` with price and accepted networks; `x402_fetch` signs the
USDC authorization and retries in the same round trip. Base is advertised as
`network: "base"`, `scheme: "exact"`, asset `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`.
Other networks appear under a separate `alternatives` key and are ignored by your
client, which is correct.

Path parameters are literal:

```
x402_fetch({ url: "https://whaletape.xyz/liquidations/BTC" })
x402_fetch({ url: "https://whaletape.xyz/whale/0x5b5d51203a0f9079f8aeb098a6523a13f298c060" })
```

## Spending under survival pressure

| tier | what to buy |
|---|---|
| **normal** | anything the task justifies; `/premium` at $0.12 replaces four separate calls |
| **low_compute** | free routes, plus `/coverage` at $0.001 and single $0.05 routes |
| **critical** | free routes only |

Two habits that cut spend, not just per-call price:

- **Do not poll.** `/pulse` returns only what changed since the cursor it gave you
  last time, at $0.05. Re-fetching `/signals` on a timer costs almost 2x more for the
  same information.
- **Push instead of pull.** `/alerts` is $0.99 once and pushes matching signals to
  your webhook for up to 7 days. Cheaper than 20 polls at $0.05 if your goal runs a week.

## Two rules that protect your balance

- **Errors are never billed.** Any `4xx`/`5xx` settles nothing on chain.
- **Known-bad requests are refused before charging** — malformed webhook URL,
  malformed wallet address, empty question.

## What this is not

- **Not financial advice and not a profit claim.** This is observational data:
  what large accounts hold, how positioning and funding moved, where liquidation
  levels sit.
- **Read `/track-record` before you trust a signal — it is free and it is blunt.**
  The provider publishes the measured hit rate of its own squeeze radar, scored
  once per episode (7,109 episodes over 50 days as of October 2026). On the 4h window every score
  band lands within a few points of a coin flip (0.51 / 0.50 / 0.48). The 24h
  window is slightly better in the higher bands (0.53 and 0.56) but the top band has only
  36 decided episodes; the live numbers are on `/track-record`. Treat this as input to your own reasoning, not as an edge
  someone measured for you.
- **Not a substitute for verifying coverage.** `/coverage` at $0.001 tells you the
  real range and granularity before you buy a series. Buy it first.
- `/index/ranking` and `/index/summary` are the provider's **own** measurements of
  other x402 merchants — useful, but self-published. Weigh them accordingly.

## Provenance

Data is collected from Hyperliquid's public API. Contact: `hello@whaletape.xyz`.
Terms and pricing are whatever `/.well-known/x402` says at call time — that
manifest, not this file, is authoritative.
