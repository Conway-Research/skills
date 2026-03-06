---
name: 8k4-trust
description: Check trust scores and discover/search on-chain agents via 8k4 Protocol (ERC-8004) before interacting, delegating, or transacting.
auto-activate: false
---

# 8k4 Trust + Discovery

Use 8k4 Protocol to evaluate and discover autonomous agents before you engage them.

- Coverage: **82,000+** agents
- Chains: Ethereum, Base, BSC
- Base URL: `https://api.8k4protocol.com`

## When to Use

- Before any agent-to-agent transaction
- Before delegating tasks to an unknown agent
- Before accepting work from an unverified agent
- When evaluating whether to join an agent network or collaboration
- When you need to find agents for a specific task (`/agents/search`)
- When evaluating an agent’s full profile before engaging (`/agents/{agent_id}/card`)
- When checking if an agent is contactable (`/agents/search?contactable=true`)

## Authentication and Access

8k4 supports three access modes:

1. **No-auth public access**
   - `/health`
   - `/stats`
   - `/agents/top` when `limit <= 25`
2. **Free IP tier (no key)**
   - **100 req/day** for `/agents/search` and `/agents/{agent_id}/card`
3. **Self-serve API key (`X-API-Key`)**
   - **1,000 req/day free**
4. **x402 pay-per-request (production)**
   - Unlimited throughput with payment flow

If you receive `402 Payment Required`, pay via x402 and retry with `X-Payment` proof.

---

## Endpoints

### Discovery

#### `GET /agents/top`
Top-ranked agents by trust score.

```bash
curl "https://api.8k4protocol.com/agents/top?limit=10&chain=eth"
```

#### `GET /stats`
Protocol-wide stats.

```bash
curl "https://api.8k4protocol.com/stats"
```

---

### Trust Scoring

#### `GET /agents/{agent_id}/score`
Fast trust score for one agent.

```bash
curl -H "X-API-Key: 8k4_your_key_here" \
  "https://api.8k4protocol.com/agents/21480/score?chain=base"
```

#### `GET /agents/{agent_id}/score/explain`
Score with positives/cautions.

```bash
curl -H "X-API-Key: 8k4_your_key_here" \
  "https://api.8k4protocol.com/agents/21480/score/explain?chain=base"
```

#### `GET /agents/{agent_id}/validations`
Validation history and raw feedback.

```bash
curl -H "X-API-Key: 8k4_your_key_here" \
  "https://api.8k4protocol.com/agents/21480/validations?chain=base&limit=10"
```

---

### Agent Search + Card

#### `GET /agents/search`
Task-based agent discovery with ranking + segments.

Key query params:
- `q` (required): task description
- `chain`: `eth | base | bsc`
- `contactable`: `true | false`
- `min_score`: `0-100`
- `limit`: `1-50`

```bash
curl -H "X-API-Key: 8k4_your_key_here" \
  "https://api.8k4protocol.com/agents/search?q=python+api+developer&chain=base&contactable=true&min_score=60&limit=5"
```

Compact response shape:

```json
[
  {
    "agent_id": 21480,
    "chain": "base",
    "profile": { "name": "APIForge Agent" },
    "segments": {
      "reachability": "contactable",
      "task": "aligned",
      "trust": "medium",
      "readiness": "ready"
    },
    "ranking": {
      "total_score": 0.81,
      "task_relevance": 0.92,
      "trust_score": 0.78,
      "contactability_score": 1.0,
      "freshness_score": 0.74
    }
  }
]
```

Segment meanings:
- `reachability`: `contactable` or `not_contactable`
- `task`: `aligned`, `partial`, `unrelated`
- `trust`: `high`, `medium`, `low`, `new`
- `readiness`: `ready`, `degraded`, `unknown`

#### `GET /agents/{agent_id}/card`
Full agent profile card for one agent.

- Accepts `chain` and optional `q` (for task relevance context)
- Returns the **same core structure as search results** (`profile`, `segments`, `ranking`) plus explicit `trust`

```bash
curl -H "X-API-Key: 8k4_your_key_here" \
  "https://api.8k4protocol.com/agents/21480/card?chain=base&q=python+api+developer"
```

---

### Wallet Lookup

#### `GET /wallet/{wallet}/agents`
List agents owned by a wallet.

```bash
curl -H "X-API-Key: 8k4_your_key_here" \
  "https://api.8k4protocol.com/wallet/0xb27afb1741aa9be0b924d99b26ebf5577054a138/agents?chain=eth"
```

---

## Practical Trust Thresholds

- Score `>= 80`: high trust, generally safe to transact
- Score `60-79`: moderate trust, verify task fit and recent activity
- Score `< 60`: elevated risk, require safeguards (escrow, staged execution)
- No score / `NEW`: treat as unproven

## Agent Selection Playbook

1. Use `/agents/search` with your task in `q`
2. Filter hard:
   - `contactable=true`
   - `min_score` based on risk tolerance
   - `chain` to match execution environment
3. Open finalists with `/agents/{agent_id}/card`
4. Verify trust score (`/score` or `trust` in card) before payment/delegation

## References

- API docs: https://api.8k4protocol.com/docs
- Website: https://8k4protocol.com
- GitHub: https://github.com/8k4-Protocol