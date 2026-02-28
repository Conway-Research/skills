---
name: clanker
description: Deploy ERC-20 tokens with Uniswap V4 liquidity pools on Base and earn trading fees — programmatically, no social platform required.
auto-activate: false
---

# Clanker — Token Deployment & Fee Earning

Clanker deploys ERC-20 tokens paired with Uniswap V4 pools on Base in a single transaction. Deployed tokens generate continuous trading fees. As the deployer you configure the fee split — earning passive USDC/ETH every time someone trades your token.

**Earning model:** deploy a token → it gets a live Uniswap V4 pool → you earn a % of every trade forever.

---

## Prerequisites

- A Clanker API key (`x-api-key`). Request one at https://clanker.world or via their Farcaster/X presence (@clanker).
- A Base wallet address to receive fees (`tokenAdmin` + reward recipient).
- Optional: an IPFS image URL for the token.

---

## Deploy a Token

**Endpoint:** `POST https://www.clanker.world/api/tokens/deploy`

**Headers:**
```
x-api-key: <your_api_key>
Content-Type: application/json
```

**Minimal request body:**
```json
{
  "name": "My Token",
  "symbol": "MYTKN",
  "tokenAdmin": "0xYourWalletAddress",
  "requestKey": "<32-char-unique-id>",
  "rewards": {
    "recipients": [
      {
        "admin": "0xYourWalletAddress",
        "recipient": "0xYourWalletAddress",
        "bps": 10000
      }
    ]
  }
}
```

**Generate a `requestKey`:** any 32-character unique string. Use a UUID (remove hyphens) or timestamp + random suffix. Must be unique per deployment.

**Response (success):**
```json
{
  "success": true,
  "message": "Token deployment enqueued",
  "expectedAddress": "0xContractAddress..."
}
```

Deployment is async — the contract address is returned immediately but mining takes ~10-30s. Query the address on BaseScan or via the public API to confirm.

---

## Full Options

| Field | Type | Notes |
|---|---|---|
| `name` | string | Token full name |
| `symbol` | string | Ticker, e.g. `PEPE` |
| `tokenAdmin` | address | Controls admin functions |
| `requestKey` | string | 32-char unique ID (idempotency) |
| `rewards.recipients` | array | Fee recipients, `bps` must sum to 10000 |
| `image` | string | IPFS URL (`ipfs://...`) or https image |
| `description` | string | Token description |
| `socialMediaUrls` | array | `{ url: "https://..." }` objects |
| `chainId` | number | `8453` (Base, default), `130` (Unichain), `42161` (Arbitrum) |
| `pool.initialMarketCap` | number | USD value, e.g. `69000` |
| `pool.pairedToken` | string | Default: WETH. Can use USDC address. |

---

## Query Deployed Tokens (No API Key)

Check tokens by a creator address or Farcaster username:

```
GET https://clanker.world/api/search-creator?address=0xYourAddress&limit=10
GET https://clanker.world/api/search-creator?name=yourusername&limit=10
```

Returns: token name, symbol, contract address, deployment date, price, market cap.

---

## SDK Alternative

For TypeScript agents:

```bash
npm install clanker-sdk viem
```

```typescript
import { ClankerSDK } from 'clanker-sdk';
import { createWalletClient, http } from 'viem';
import { base } from 'viem/chains';

const client = new ClankerSDK({ apiKey: process.env.CLANKER_API_KEY });
const result = await client.deployToken({
  name: 'My Token',
  symbol: 'MYTKN',
  tokenAdmin: '0xYourAddress',
  // ... other options
});
```

---

## Fee Structure

- Default pool fee: configurable (e.g. 1% = 100 bps)
- `rewards.recipients[].bps`: your share of LP fees (10000 = 100%)
- Fees accumulate on-chain and can be claimed at any time via the contract

---

## Related Tools

- **Bankr** (@bankrbot on X/Farcaster) — launch tokens via social post using Clanker under the hood; simpler UX, less control
- **BaseScan** — verify contract: `https://basescan.org/token/<contract_address>`
- **DexScreener** — monitor trading volume/price: `https://dexscreener.com/base/<contract_address>`

---

## Guardrails

- Never deploy tokens with misleading names implying affiliation with real projects
- `requestKey` must be unique — reusing one silently deduplicates (same token returned)
- Keep your API key in an env var, never hardcode it
- Deployment is irreversible — the contract is live immediately once mined
