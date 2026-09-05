---
name: polymarket-scanner
description: Monitor Polymarket prediction markets, track odds movements, detect mispriced events, and find cross-platform arbitrage.
auto-activate: true
---

# Polymarket Scanner

Monitor prediction markets in real-time, track odds movements, detect anomalies, and find potential arbitrage opportunities across platforms. All Polymarket data is public — no API key, no auth, no wallet required to read.

## API Reference

Base URL: `https://gamma-api.polymarket.com`

All endpoints are public REST. No authentication needed for market data.

### Core Endpoints

| Endpoint | Description |
|---|---|
| `GET /events/keyset?closed=false&limit=20` | List active events (paginated via `after_cursor`) |
| `GET /events/{id}` | Fetch single event by ID |
| `GET /events/slug/{slug}` | Fetch event by URL slug |
| `GET /markets/keyset?closed=false&limit=20` | List active markets |
| `GET /markets/{id}` | Fetch single market by ID |
| `GET /markets/slug/{slug}` | Fetch market by slug |
| `GET /public-search?q={query}` | Search events, tags, profiles |
| `GET /tags?limit=20` | List all tags (categories) |
| `GET /tags/slug/{slug}` | Get tag by slug (returns numeric ID) |
| `GET /events/keyset?tag_id={id}&closed=false` | Filter events by tag |
| `GET /series?closed=false` | List series (recurring event groups) |
| `GET /sports` | List sports metadata |

### CLOB (Order Book) Endpoints

Base URL: `https://clob.polymarket.com`

| Endpoint | Description |
|---|---|
| `GET /midpoint?token_id={tokenId}` | Current midpoint price for a token |
| `GET /price?token_id={tokenId}&side=buy` | Best bid/ask price |
| `GET /book?token_id={tokenId}` | Full order book |
| `GET /prices-history?market={conditionId}&interval=1d&fidelity=60` | Price history |

### Data Model

- **Event** = a question group (e.g., "US election 2028")
- **Market** = a single yes/no question within an event
- **Outcomes** = YES and NO, each with a unique `tokenId`
- **Price** = 0.00-1.00 (represents implied probability; $0.65 YES = 65% chance)

## Task 1: Market Scanner

Scan active markets and report high-volume or fast-moving events.

```bash
exec 'curl -s "https://gamma-api.polymarket.com/events/keyset?closed=false&limit=50" | python3 -c "
import json, sys
data = json.load(sys.stdin)
events = data.get(\"events\", [])
print(f\"Found {len(events)} active events\")
for e in events[:20]:
    title = e.get(\"title\", \"Unknown\")
    slug = e.get(\"slug\", \"\")
    markets = e.get(\"markets\", [])
    print(f\"  [{len(markets)} markets] {title}\")
    print(f\"    https://polymarket.com/event/{slug}\")
"'
```

## Task 2: Price Monitor

Track prices for specific markets. Use the CLOB midpoint endpoint.

```bash
# Get token IDs from a market
exec 'MARKET_ID="703257"
curl -s "https://gamma-api.polymarket.com/markets/${MARKET_ID}" | python3 -c "
import json, sys
m = json.load(sys.stdin)
print(f\"Market: {m.get(\"question\", \"Unknown\")}\")
clob = m.get(\"clobTokenIds\", \"[]\")
tokens = json.loads(clob) if isinstance(clob, str) else clob
for i, t in enumerate(tokens):
    label = \"YES\" if i == 0 else \"NO\"
    print(f\"  {label} token: {t[:20]}...\")
"'
```

Then fetch live prices:

```bash
exec 'TOKEN_ID="YOUR_YES_TOKEN_ID"
curl -s "https://clob.polymarket.com/midpoint?token_id=${TOKEN_ID}" | python3 -c "
import json, sys
data = json.load(sys.stdin)
mid = data.get(\"mid\", \"N/A\")
print(f\"Current midpoint: {mid} (implied probability: {float(mid)*100:.1f}%)\")
"'
```

## Task 3: Odds Movement Detector

Compare current prices against historical to find fast movers.

```bash
exec 'CONDITION_ID="YOUR_CONDITION_ID"
curl -s "https://clob.polymarket.com/prices-history?market=${CONDITION_ID}&interval=1d&fidelity=60" | python3 -c "
import json, sys
data = json.load(sys.stdin)
history = data.get(\"history\", [])
if len(history) >= 2:
    latest = float(history[-1].get(\"p\", 0))
    yesterday = float(history[-24].get(\"p\", 0)) if len(history) > 24 else float(history[0].get(\"p\", 0))
    change = (latest - yesterday) * 100
    direction = \"▲\" if change > 0 else \"▼\"
    print(f\"Current: {latest*100:.1f}% | 24h ago: {yesterday*100:.1f}% | Change: {direction} {abs(change):.1f}pp\")
else:
    print(\"Insufficient history\")
"'
```

## Task 4: Cross-Platform Arbitrage Scanner

Compare Polymarket odds against external sources. An arbitrage exists when implied probabilities across platforms sum to less than 100% for the same event.

**Logic:**
```
Platform A: YES at $0.60 (implies 60% yes, 40% no)
Platform B: NO at $0.35  (implies 35% no, 65% yes)

If you buy YES on A ($0.60) and NO on B ($0.35):
  Total cost: $0.95
  Guaranteed payout: $1.00 (one side always wins)
  Profit: $0.05 = 5.3% risk-free return
```

To find arbs, compare Polymarket prices with:
- **Kalshi** (kalshi.com — US-regulated, similar markets)
- **Metaculus** (metaculus.com — community predictions, no trading)
- **Betting odds** (for sports markets — convert moneyline to implied probability)

```bash
exec 'python3 << "PYEOF"
import json, urllib.request

# Fetch Polymarket active events
url = "https://gamma-api.polymarket.com/events/keyset?closed=false&limit=30"
req = urllib.request.Request(url, headers={"User-Agent": "automaton/1.0"})
with urllib.request.urlopen(req, timeout=10) as resp:
    data = json.loads(resp.read())

events = data.get("events", [])
print(f"Scanning {len(events)} active Polymarket events for arb candidates...")
print()

# Flag markets where YES price is extreme (potential mispricing)
for e in events:
    title = e.get("title", "")
    for m in e.get("markets", []):
        clob_raw = m.get("clobTokenIds", "[]")
        tokens = json.loads(clob_raw) if isinstance(clob_raw, str) else (clob_raw or [])
        if not tokens:
            continue
        # Fetch midpoint for YES token
        try:
            mid_url = f"https://clob.polymarket.com/midpoint?token_id={tokens[0]}"
            mid_req = urllib.request.Request(mid_url, headers={"User-Agent": "automaton/1.0"})
            with urllib.request.urlopen(mid_req, timeout=5) as mid_resp:
                mid_data = json.loads(mid_resp.read())
            mid = float(mid_data.get("mid", 0.5))
            # Flag anything near 50/50 (most liquid, best arb candidates)
            # or extreme (>90% or <10% — check if other platforms disagree)
            if 0.40 <= mid <= 0.60:
                print(f"  [LIQUID] {mid*100:.0f}% | {title}")
                print(f"    -> Compare with Kalshi/Metaculus for same event")
            elif mid > 0.92 or mid < 0.08:
                print(f"  [EXTREME] {mid*100:.0f}% | {title}")
                print(f"    -> High confidence — verify no arb against contrarian platform")
        except:
            pass
PYEOF'
```

## Task 5: Heartbeat Market Watch

Set up recurring scans via heartbeat. Monitor specific markets for price movements.

```bash
modify_heartbeat '{"tasks": [{"name": "polymarket_scan", "cron": "0 */4 * * *", "command": "curl -s https://gamma-api.polymarket.com/events/keyset?closed=false&limit=10 > /tmp/pm_latest.json && echo Scan complete"}]}'
```

## Task 6: Monetize as an API (combine with x402-api-server)

Turn your scanner into a paid service:

1. Install `x402-api-server` skill
2. Replace `handleRequest()` with your scanner logic
3. Return aggregated odds, movement alerts, or arb signals
4. Charge $0.02-0.05 per request

Example response your paid API could return:
```json
{
  "timestamp": "2026-09-05T10:00:00Z",
  "markets_scanned": 247,
  "fast_movers": [
    {"event": "Fed rate cut Sep 2026", "change_24h": "+8.3pp", "current": "72.1%"}
  ],
  "arb_candidates": [
    {"event": "BTC above 100k by Dec", "polymarket": "61%", "kalshi": "55%", "spread": "6pp"}
  ],
  "high_volume": [
    {"event": "US Election 2028", "markets": 12, "slug": "us-election-2028"}
  ]
}
```

## Search for specific topics

```bash
exec 'curl -s "https://gamma-api.polymarket.com/public-search?q=bitcoin" | python3 -c "
import json, sys
data = json.load(sys.stdin)
events = data.get(\"events\", [])
print(f\"Found {len(events)} events matching query\")
for e in events[:10]:
    print(f\"  {e.get(\"title\", \"Unknown\")}\")
    print(f\"    https://polymarket.com/event/{e.get(\"slug\", \"\")}\")
"'
```

## Filter by category

Popular tag slugs (resolve to numeric IDs first):
- `politics`, `crypto`, `sports`, `nba`, `nfl`, `science`, `entertainment`, `ai`

```bash
exec 'TAG_SLUG="crypto"
TAG=$(curl -s "https://gamma-api.polymarket.com/tags/slug/${TAG_SLUG}")
TAG_ID=$(echo "$TAG" | python3 -c "import json,sys; print(json.load(sys.stdin).get(\"id\",\"\"))")
curl -s "https://gamma-api.polymarket.com/events/keyset?tag_id=${TAG_ID}&closed=false&limit=20" | python3 -c "
import json, sys
data = json.load(sys.stdin)
events = data.get(\"events\", [])
print(f\"Found {len(events)} active {\"crypto\"} events\")
for e in events[:10]:
    print(f\"  {e.get(\"title\", \"Unknown\")}\")
"'
```

## Guardrails

- All market data endpoints are **free and public** — no API key needed
- **Trading** endpoints require wallet authentication — do not trade without explicit user authorization
- Rate limit: be respectful, add 100ms delays between rapid-fire requests
- Polymarket is unavailable in some jurisdictions — the automaton should note this
- Never present odds as financial advice — frame as "market-implied probability"
- Cross-platform arb calculations are theoretical — execution risk, fees, and timing gaps exist
